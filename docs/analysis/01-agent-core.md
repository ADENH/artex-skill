# ARTEX 分析①：Agent 核心架构

> 来源：对 D:\xiangmu\ARTEX 源码的完整勘察（agent/ 包 52 文件），2026-10。

## 1. 整体运行循环与任务生命周期

**三种 agent 分工**（norma SDK 之上薄封装）：mainagent 是"人类接口编排器"，只观察+操舵，不自主生成意图（mainagent.go:16-19）；planner 是**唯一意图生成者**，事件驱动、图一变即被唤醒（planner.go:19-22）；worker 领**一条**意图执行、写回、即停（worker.go:23-27）。

**引擎驱动**（server/engine.go）：每任务一个 plannerLoop goroutine + N 个 workerLoop goroutine（engine.go:763-767），外加 deadline 协调器（engine.go:768）。plannerLoop 是 `select{notify 边沿唤醒 / heartbeat 心跳}` 循环（engine.go:901-913），唤醒先做 **800ms debounce** 把一串变更合并成一轮（engine.go:810-819）；心跳默认 600s 兜底（engine.go:781-787）。workerLoop 轮询 `claimNext`：CAS 领取 frontier 中 open 意图置 running（engine.go:1237-1245）。

**生命周期**：goals 分解器先从任务目标抽出操作约束+资产范围+目标节点（goals.go:19-46, 53-67）→ planner 每轮读态势、`add_intent` 派意图进 frontier → worker 领取执行 → 写回 fact/asset/finding 又触发 Notify → 唤醒 planner（闭环）。终态：planner `prove_goal` 逐个标 met，最后一个目标 met 时引擎落 done 并 cancelExec 杀掉在跑 worker（engine.go:885-894）；非正常终态由 harness.TerminalReason 区分 done/exhausted/blocked/stopped，planner 据此决定续派/重派/换打法（planner.go:326-327）。意图状态机：open→running→(done|paused|exhausted|blocked|stopped)。

## 2. 双图架构

指**资产图（全局共享真值库）+ 探索图（每任务独立推进链）**。探索图节点 = goal/intent/fact/finding/hint/digest，边有向：`spawns/derived_from/yields/proves` 构成血缘（coldgraph.go:11-14）。资产图节点 = root_domain/subdomain/ip/service/app/endpoint，父子关系与去重 key 全由程序计算，agent 只交原始信息。两图靠 `exploration_anchors` 锚点相连，支撑"意图打哪些资产 / 资产被哪些意图测过"双向查询与覆盖度。

**hot/cold 是探索图上的分区+压缩机制**（coldgraph.go）：hot 集 = live intent 及其全部祖先 + 直接子 fact；可折叠类型仅 intent+fact。**去抖**：结构冷节点需连续冷 ≥R=6 轮，复活即清 stamp。**分组**：两趟并查集——冷↔冷真实边连通 + "同父孤点救援"；规模 ≥K=2 才折叠；父节点只做 anchor 绝不做成员。Compactor 每 planner 轮同步维护，未压缩冷节点 ≥20 或活跃 digest ≥8 时后台压缩，签名不变复用旧 digest 免调 LLM。设计思想：**任何活分支保整链热**、保守去抖、语义无损折叠，防上下文爆炸同时不丢因果可达性。

## 3. planner 多轮共享 todolist

planner 每次唤醒都是**全新会话**（无状态），故把 TodoStore 按 exploration id 存在 Planner 实例里跨唤醒复用（planner.go:44-49, 106-117），每轮注入渲染"你的规划待办（跨唤醒保留，上一轮你写的）"并指示：只对前置已满足的下一步派意图、把被 fact 满足的步骤标 completed、不重派 pending 项（planner.go:129-144）。串行利用链"分步派、别并行"写在规划提示正文。解决了事件驱动+会话无状态下多步攻击链"错序/重复/前置缺失空转"。取舍：planner 把易变态势放 user turn 以稳定 system 缓存；worker 反之把意图放 system（永不被压缩丢掉）。

## 4. worker 间过程级信息交换（blackboard）

三层机制：
- **全局态势注入**：worker 启动带 graph_overview（删掉 coverage 防诱发越界），定位"只读知大局、别重复、联想写 fact 交规划者"。
- **过程库检索**：每步 activity 落库；`search_all_worker_traces(q)` 在其他 worker 的过程中检索（自动排除自己），`get_worker_trace` 取具体步骤全文，超 5 步降级并显式告知遗漏——信息以"执行过程"粒度流动。
- **跨任务继承**：SourceTaskIDs 声明，list/get 均带 `inherited=true` 来源标注，**只继承直接来源任务**，写回被拒绝——继承只读单向，避免写穿任务边界。

## 5. worker 自主性边界四件套

- **constraints**：任务级 allow/deny 操作约束（goals 分解自动抽取），注入 prompt 顶部，"凌驾于一切探索启发式"，含"新目标≠新授权"条款。
- **deferred**：延迟工具机制——MCP 工具名渲染成 `<available-deferred-tools>` 块，skill 门控工具在 skill 加载后才出现；resume 后重放 Skill() 调用恢复解锁。
- **noa**：norma SDK 的模型驱动上下文压缩（实验开关），失败回退内置 compaction。
- **terminalreason**：终态翻译：MaxTurns/Timeout→exhausted（已收尾写回，不算失败）；ModelError→blocked（可重派）；Aborted*→区分中断阶段保留取消原因。

## 6. 人在环路（intervention）与旁路提问

**干预通道**：`add_hint`（战略提示挂图）、`add_intent`（直接注入 priority 8-10 意图）、`steer_work`（不打断运行，在下一个工具调用前注入纠偏）、`kill_work`（per-work cancel cause）、`set_goals`（新目标自动拉回 running）。可对暂停中的 worker 直接发消息续跑。

**side_questions（/btw 旁路提问）**：捕获不可变上下文快照，旁路回答用快照+单次请求生成，**不创建 session、不接工具、不影响主上下文**。

## 7. 资产插入与 taskclock

**insert_assets**：统一批量混型插入。task_id 由程序权威赋值；每条先过**资产闸门**（block/allow）；auth/technologies/params 追加合并不覆盖；成功后连锚点、记溯源、**AddAutoScope 保守自动入测试范围**。意图锚定的 asset_ids 在 worker 启动时自动入范围且幂等。enrich 异步补全 DNS/HTTP。

**taskclock**：任务级绝对 deadline + Final 标志。预算夹逼到"距 deadline 剩余"；超时不是硬杀而是**优雅收尾**——SDK 强制一轮 settlement，隐藏 Bash、只写回已识别事实，"事实永不因超时丢失"。
