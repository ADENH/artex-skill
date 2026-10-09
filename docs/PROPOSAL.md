# ARTEX-Skill 设计方案（v1 草案，待用户确认）

> 目标：把 ARTEX（AI 自主渗透测试系统）的方法论、工程纪律与安全护栏蒸馏成一个**可移植的 Agent Skill**，在任何支持 Agent Skills 的平台（Claude Code / ZCode / Cursor 等，agentskills.io 格式）上即装即用；不支持的平台上按 README 指引把 SKILL.md 挂进系统提示同样可用。

## 一、定位

**一句话**：给任意 AI Agent 装上"授权渗透测试专家的大脑"——ARTEX 的 planner/worker/审计者三视角方法论 + 资产/事实/漏洞三账本纪律 + 四层安全护栏，零基础设施、纯提示与文件驱动。

与 ARTEX 本体的关系：ARTEX 是平台（Go+PG+多 agent 引擎），skill 是其方法论的"单文件体系移植版"，开源时在两边 README 互链。

## 二、仓库结构（提案）

```
<repo>/
├── SKILL.md                    # 主入口（≤500行）：角色、工作循环、三视角协议、
│                               #   开工门（scope 先行）、红线速查
├── references/
│   ├── methodology.md          # 六心法 + 渗透循环 + 高价值面清单 + 收敛策略
│   ├── asset-model.md          # 资产 6 类型/包含层级/登记与去重规约
│   ├── evidence-rules.md       # 证据纪律：observed/inferred、确认门槛、对抗式自检
│   ├── safety-rails.md         # 授权四层 + 危险命令 deny 正则全集 + 两轴裁判 + 人工确认协议
│   ├── report-template.md      # 漏洞报告结构 + 任务收尾总结模板
│   └── workspace-templates/    # scope.md / assets.md / facts.md / findings/*.md 模板
├── scripts/
│   └── danger_check.(py|ps1)   # 可选：把 deny 正则集做成命令自检脚本（不依赖也能用）
├── README.md                   # 中英双语：安装（Claude Code/ZCode/Cursor/通用）、示例、与 ARTEX 关系
├── LICENSE                     # AGPL-3.0（衍生自 ARTEX）
└── docs/                       # 本设计文档 + ARTEX 分析（即当前 docs/）
```

## 三、SKILL.md 核心设计

**frontmatter**：name + description（双语，"做什么+何时用"）+ license + compatibility。

**正文骨架**（吸收 ARTEX prompt 的结构经验：行为指令前置、参考资料后置）：
1. **适用前提与授权门**（第一优先）：无 scope.md 不开工；最小范围原则；新目标≠新授权；.gov/.edu 黑名单
2. **三视角协议**：每轮先自问"我现在是规划者/执行者/审计者哪个视角"；规划者守 todolist（跨轮持久、串行链分步）；执行者单意图走透；审计者独立复现
3. **工作循环**：侦察定面 → 组合排序 → 深入利用 → 独立复现 → 记录写回（三账本）
4. **三账本书写规约**：assets.md / facts.md / findings/（行格式模板，含 confidence、evidence、intent 归属）
5. **红线速查表**：deny 正则精简版 + 不可逆生产操作 → 停下问人 + 确认门槛三禁（版本匹配/漏洞库/补丁 diff 推断）
6. **收尾协议**：写回→总结达成/路线/封锁→报告
7. **按需加载**：深入规则指向 references/*（SKILL.md 只放"何时读哪个"）

## 四、设计原则（从 ARTEX 学到的）

- **渐进披露**：SKILL.md 是目录不是百科，重内容放 references 按需读（对应 ARTEX 冷折叠思想）
- **行为可校验**：账本行格式固定，人和 agent 都能审计（对应 evidence 链）
- **平台无关**：只用 Bash/文件系统/TodoWrite 这些所有 coding agent 都有的能力；脚本只是增强
- **中文正文 + 双语 frontmatter/README**（ARTEX 社区为中文）

## 五、测试与验收计划

1. **离线验收**：SKILL.md+references 全文走查清单（授权门/红线/门槛三禁/账本格式是否齐）
2. **本地靶场实测**（授权环境）：vulhub 或本地 CTF 容器，装到 ZCode/Claude Code 上跑 3 个典型任务（Web 入门靶、API 靶、SQLi 靶），验收：开工先建 scope、无越权命令、facts 带 confidence、findings 有可复现 PoC、收尾有总结
3. **负向测试**：诱导对公网未授权目标/对 .gov 域发起测试 → 必须拒绝并记录 out-of-scope
4. 迭代修正后发布 v0.1.0

## 六、发布流程
GitHub 仓库（chu0119/<name>，public，AGPL-3.0）→ README 双语 + ARTEX 仓库互链 → 打 tag。纯文本项目无需 CI/镜像。

## 七、待用户确认的决策点
1. skill 名称/仓库名
2. 范围：是否包含"三账本文件工作区"（推荐含，这是 ARTEX 精髓）还是纯方法论精简版
3. 语言：中文为主双语 frontmatter（推荐）还是全双语
