# ARTEX 分析⑤：产品功能面与数据模型

> 来源：db/schema.sql（~45 表）、server/ 文件、web/src/app 路由树。

## 1. 数据库核心表（分类速览）
- **资产**：companies（甲方主体）/ assets（6 类型主表）/ company_scope（domain/ip/cidr/icp/keyword 范围，自动推导归属）
- **探索**：explorations（每任务一个）/ exploration_nodes（begin/goal/intent/fact/finding/hint/digest + 状态机 + priority）/ exploration_edges（spawns/derived_from/yields/proves/covers）/ exploration_anchors（节点↔资产）
- **任务**：tasks（状态机 created→running→paused/done/failed/timeout，超限排队，LLM 双绑定）/ task_categories / task_templates（含拦截规则快照）/ task_scope / task_constraints / task_intercept_rules / task_archives（冷归档 tar.zst）
- **发现**：findings（vulnclass/severity/9 态处置/Markdown 报告）/ finding_retests
- **证据**：traffic_evidence_snapshots / finding_traffic_bindings（role: baseline/proof/verification/supporting）
- **会话/活动**：conversations / conversation_activities / activity（工具调用+token 计量流水）
- **Agent**：agents（5 角色）/ agent_prompts（版本）/ agent_visibility / agent_triggers
- **LLM**：llm_profiles / llm_profile_health / llm_usage / llmretry
- **平台**：mcp_servers / mcp_tools_cache / tools / skill_usage / tool_usage / commands
- **拦截**：intercept_rules / intercept_pending / asset_intercept_rules
- **通知**：notification_channels / events / deliveries（钉钉/飞书/企业微信/webhook/Telegram/Email；realtime/digest；过滤+限流 20/min）
- **运维**：settings / scheduler_state / server_logs / side_question_*

## 2. Web 页面路由
login/setup；dashboard（总览/Token/活动流）；chat（人在环路）；tasks（列表/详情：会话/工具调用/探索链路 tab）；findings（列表/详情）；assets（+覆盖力导向图）；traffic；sync（ScopeSentry）；workspace/commands/llm-records；system/agents/llm/mcp/skills/tools/intercept(+approvals/assets)/notify/logs/settings。

## 3. 任务模型要点
状态机+软删+归档三轨；并发准入（queued FIFO）；模板带拦截规则快照；覆盖度开关；LLM 默认+当前双绑定链降级。

## 4. 资产模型与覆盖图
6 类型：root_domain/subdomain/ip/service/app/endpoint，每类独立唯一索引；字段覆盖 DNS 解析、C 段、ICP、APP bundle、HTTP 服务（标题/状态码/技术栈/favicon mmh3）、端点（url+method+params+auth）。**覆盖图不存、实时算**：节点=范围内资产+公司+合成根域，边=包含关系（endpoint→service→subdomain/ip→root_domain→company），节点带 Tested/InScope 标记。覆盖度分母来自 task_scope，分子来自 task_asset_links。

## 5. ScopeSentry 同步
作为 HTTP MCP 数据源，按项目/任务两维度分页拉取（单类型上限 5000），映射 subdomain/app/service 三类入库。

## 6. 蒸馏取舍结论（供 skill 设计）
- **简化为 markdown 文件**：任务状态机语义、发现 9 态处置流、资产 6 类型与包含层级（覆盖图本质）、拦截规则全集、审批语义。
- **砍掉**：冷归档、LLM 健康度/计量、cold-digest 压缩、usage 统计、CIDR/GIST 索引细节、旁路提问、通知渠道（skill 场景直接对话就是通知）。
