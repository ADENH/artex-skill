# ARTEX 分析③：工具体系

> 来源：agent/toolcatalog.go、tools*.go、tools_insert.go、server/platform_tools.go、mcpdiscover.go、customtool.go、traffic/traffic.go、enrich/enrich.go。

## 1. 工具总清单（7 层）

**A. SDK 通用工具**：Read/Write/Edit/MultiEdit/LS/Glob/Grep/Bash/Sleep。worker 刻意裁掉 MultiEdit/Glob/Grep 收敛工具面；interactive_shell 开启时加 5 个交互会话工具。

**B. 探索图工具**：graph_overview（态势蒸馏）/ list_findings / list_facts（分页+关键词）/ node_detail / expand_digest / **add_intent**（批量写方向，锚定 fact/finding）/ **record_fact**（强制 evidence+confidence observed/inferred）/ **report_finding**（vulnclass/severity/summary/evidence/traffic_refs）/ add_hint / set_goals / set_constraints / list_goals / prove_goal / goal_met / kill_work / steer_work / get_worker_output / get_worker_trace / search_all_worker_traces / list_worker_traces。

**C. 资产工具**：**insert_assets**（批量混合登记 6 类，过资产闸门，自动入 task_scope）/ list_assets（DSL 检索，禁无条件全量）/ list_companies / add_company_scope（拒裸 TLD、IPv4≤/16）/ add_task_scope（主动扩大授权边界）/ list_untested_assets。

**D. 流量工具**：traffic_search（必填 host 防全库扫）/ traffic_get（req 截 2500、resp 截 4000）/ traffic_blob（超大 body 8KB 分段读）。

**E. 跨任务编排**：list_tasks / spawn_task / pause_task / get_task_graph / add_task_hint / get_task_node_detail / update_finding_report 等。

**F. 漏洞证据/复测**：get_finding_traffic / bind_finding_traffic / get_finding_retest_context / record_finding_retest_result（verdict: reproduced/fixed/inconclusive）。

**G. 平台自服务**：create_skill / update_skill_file / create_custom_tool / create_mcp / delete_assets_by_host。

## 2. 执行封装要点
- Bash 底层复用：自定义 command/script 工具复用 Bash Call，继承安全 floor/超时/代理/输出溢出。
- 超时：command/script 120s、http 30s；预算耗尽触发 Settlement 收尾轮（隐藏 Bash，先写回再总结）。
- 工作目录：**每意图独占** `<workDir>/tasks/<taskID>/i<intentID>`；大输出溢出 `cmd-output/`（30000 字符上限，全文留盘）。
- Python script 工具：临时 .py 到 `.tools/`，参数 JSON 走 stdin。

## 3. 上下文控制（tools_digest / overview）
- **冷折叠**：旧节点折 digest，概览只给前 15 条，`expand_digest(id)` 是冷区唯一出口；被新意图引用即复活。
- **窗口+游标+总数**：列表默认 20/上限 100、summary 截 160 字；open_intents≤30、recent_facts≤20，并配真实总数防止"没显示=没做过"。
- 关联任务文本预算：总量 48000 runes、每源≤8000。

## 4. 流量录制（MITM）
嵌入式 go-mitmproxy，失败 host 降级透传不中断；worker 侧全局注入 HTTP(S)_PROXY + 各生态 CA 变量（SSL_CERT_FILE/REQUESTS_CA_BUNDLE/CURL_CA_BUNDLE/GIT_SSL_CAINFO/NODE_EXTRA_CA_CERTS），工具无需 -x/-k；WebFetch 同代理。

## 5. MCP / 自定义工具
stdio/http/sse 三协议发现+缓存；`mcp__server__tool` 展开进工具面；skill 门控 MCP 延迟加载（schema 扣留、调用解锁）；自定义工具 4 类（shell 声明/command/script/http）；DB 只能改描述/schema，handler 永远来自代码。

## 6. enrich（资产自动富化）
非 AI 异步 worker 池（4 worker、5 分钟冷却去重）：DNS 解析（dnsx 查 A/AAAA/CNAME 回写）与 HTTP 探测（status/长度/title 回写）。

## 7. 结果回流纪律
统一 JSON 文本结果；错误作为正常工具结果返回带纠正建议；大内容二段式取（id/hash → 分页原文）；执行全程 Activity 流捕获（summary≤200 字/行，全 detail 存库）供回看与审计。

**可移植经验**：分层工具面（通用 shell/文件 + 领域读写 + 证据工具）；写回三件套（fact/asset/finding）强制证据与置信度；列表一律"紧凑行+分页+总数"；重内容按 id/hash 二段式取；代理+CA 全局注入。
