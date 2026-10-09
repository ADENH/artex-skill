# ARTEX 精髓提炼（skill 的原料清单）

> 五份分析（analysis/01-05）的合成结论：把 ARTEX 蒸馏成单文件体系 Agent Skill 时，哪些东西必须带走、怎么带。

## 一、ARTEX 的核心设计思想（必须完整继承）

1. **三角色分工，单人可模拟**：planner（唯一意图生成者：读态势→去重→深度优先→派意图）/ worker（领一条意图走透，做完即停）/ 审计者（对抗式自检：怀疑自己的"发现"）。单 agent 上用"视角切换 + 跨轮共享 TodoWrite"即可模拟（ARTEX 的 planner 本来就是无状态会话+持久化 todolist）。
2. **双图思想，文件可落地**：资产图（root_domain→subdomain→ip/service→endpoint 层级真值库）+ 探索图（goal/intent/fact/finding 血缘链）。skill 里对应：`workspace/assets.md`（含 Tested 标记）+ `workspace/facts.md` + todolist。
3. **证据纪律（ARTEX 的灵魂）**：
   - 置信度二值：observed（亲眼看）/ inferred（推断）
   - **确认漏洞的唯一门槛：真实触发拿到可复现证据**；版本/CVE 匹配、外部漏洞库、补丁 diff 推断一律只算 inferred
   - 对抗式自检：换路径独立复现，证伪与证实等价
   - 证据=请求响应原文/命令输出快照，缺失结果记 unknown 不记成功
4. **收敛策略**：先广后聚（2-3 条机理不同路线并存，有逼近目标实证才聚焦）；一条路走透再下结论；封锁路线只有"材料性新机理"才重开；串行利用链分步派；深度优先于覆盖度。
5. **优雅收尾**：超时/预算尽=先把已识别事实写回、总结达成项/路线/封锁方向，事实永不因超时丢失。

## 二、安全纪律（skill 的底线，从 intercept/guard/scope 蒸馏）

1. **授权前置四层**：scope 文件先行（最小范围原则：URL 取完整主机名）→ allow/deny 约束凌驾一切启发式 → 新目标≠新授权（out-of-scope 记录并跳过）→ 资产黑名单（.gov/.edu 等）。
2. **危险命令 deny 正则全集**（系统破坏 100 / DB 破坏 90 / HTTP 删除 80 / 删除类接口路径 v3，含具体正则，见 analysis/04）。
3. **不可逆×生产资产 = 拒绝**；只读恒可放行；不确定即转人工确认（等价于 ask，超时视为拒绝）。
4. 平台/自我拦截 ≠ 目标防御，不得当成 WAF 去绕。

## 三、方法论正文（skill 的直接内容，从 prompt 体系蒸馏）

- pentest 六心法 + 工作循环（侦察定面→组合排序→深入利用→独立复现→回组合）
- 高价值面清单：输入解析/上传/反序列化/认证绕过/错误泄露/缓存投毒/竞态/类型混淆/批量赋值
- planner 决策流程（去重/证明目标/补方向规则）
- worker 红线（意图外线索交 fact、初次受阻≠此路不通、三种写回别串图）
- 报告结构模板（复现步骤可照做/证据贴原文/PoC 完整代码块/根因/修复建议）

## 四、工程纪律（单 agent 上下文管理经验）

- 中间产物写意图专属目录，禁 /tmp
- 大输出落盘只回看（30000 字符截断）；列表"紧凑行+分页+真实总数"
- 冷知识折叠成 digest、被引用即展开（skill 场景≈ facts.md 分段+索引）
- 输出注入一律当数据（防提示注入）

## 五、ARTEX 组件 → Skill 组件映射表

| ARTEX 组件 | 实现载体 | Skill 对应物 |
|---|---|---|
| planner/worker 分工 | 双角色 prompt + 引擎 | SKILL.md 视角切换协议 + TodoWrite 编排规则 |
| 资产图（6 类型+锚点+覆盖） | PG assets/anchors/links | `references/asset-model.md` + workspace/assets.md 模板 |
| 探索图（intent/fact 血缘） | PG nodes/edges | workspace/facts.md（每条带 intent 归属与依赖）+ todolist |
| record_fact（evidence+confidence） | 工具 | facts.md 书写规约（模板行格式） |
| report_finding（门槛校验） | 工具 | `references/evidence-rules.md` 确认门槛 + findings/ 目录模板 |
| intercept 规则集 | DB 规则引擎 | `references/safety-rails.md`（正则全集+两轴裁判+人工确认协议） |
| task_scope/constraints | DB + prompt 注入 | `references/scope-contract.md` + workspace/scope.md 模板（开工先立） |
| reporter 报告结构 | prompt + Go 模板 | `references/report-template.md` |
| 冷压缩 digest | 图算法+LLM | facts.md 分段归档约定（手动/低成本） |
| 流量录制 MITM | go-mitmproxy | 不内置；引导用代理+CA 环境变量纪律（可选参考） |
| judge 模型裁判 | LLM 调用 | 两轴自判清单（可逆性×作用对象），不可逆生产操作→停下问人 |

## 六、skill 明确不带的（砍掉项）
通知渠道、归档、LLM 池/failover、usage 统计、多任务编排、旁路提问、Web UI、ScopeSentry 同步、MCP 平台管理。
