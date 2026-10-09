# ARTEX Skill

> ⚠️ **仅用于授权测试**：本技能只应用于拥有书面授权的目标、SRC/众测平台范围内、CTF 竞赛或自有系统。对未授权目标的任何测试都是违法行为。技能本身内置了授权门控与安全护栏——请勿试图移除它们。

**给任意 AI Agent 装上"授权渗透测试专家的大脑"。**

本技能把 [ARTEX](https://github.com/chu0119/ARTEX)（AI 自主渗透测试系统，百度"agent+"攻防挑战赛冠军项目）的核心能力蒸馏成一个纯提示+文件驱动的 Agent Skill——零基础设施、零依赖，装上即用：

- **三视角协议**：planner（规划）/ worker（执行）/ auditor（对抗式自检）三视角轮换，单 agent 模拟多智能体分工
- **三账本纪律**：资产 / 事实 / 漏洞三本 markdown 账，每条事实带 `observed/inferred` 置信度与证据指针，每个确认漏洞必须"真实触发 + 可复现 + 独立复现"三重门槛
- **四层安全护栏**：scope 授权门（新目标≠新授权）→ 操作约束 → 资产黑名单（.gov/.edu）→ 危险命令拦截（系统/数据库/HTTP 破坏全量规则）+ 不可逆操作人工确认协议
- **ARTEX 方法论**：六心法（先广后聚、一条路走透、对抗式自检…）、高价值攻击面清单、收敛与去重策略、优雅收尾协议

## 安装

**Claude Code / ZCode（Agent Skills 格式）**

```bash
git clone https://github.com/chu0119/artex-skill.git ~/.claude/skills/artex-skill   # Claude Code
git clone https://github.com/chu0119/artex-skill.git ~/.zcode/skills/artex-skill    # ZCode（或你的 skills 目录）
```

**Cursor / 其他支持自定义规则的 Agent**

把 `SKILL.md` 与 `references/` 复制进项目，在规则文件（如 `.cursorrules`、`CLAUDE.md`）里加一行：

```
渗透测试任务必须遵循 ./SKILL.md 的全部协议后再行动。
```

**一次性使用**：直接把 `SKILL.md` 全文粘贴进系统提示 / 对话开头即可。

## 快速开始

对 agent 说：

```
用 artex-skill 对 https://target.example.com 做授权渗透测试，
授权来源：XX SRC 平台（范围链接），约束：不做端口扫描、不爆破。
```

装了本技能的 agent 会依次：立 scope → 建三账本 → 三视角循环推进 → 每步带证据写回 → 收尾出报告。工作目录会长这样：

```
<task>/
├── scope.md          # 授权范围与约束（开工门）
├── assets.md         # 资产账（6 类型，含已测标记）
├── facts.md          # 事实账（observed/inferred + 证据指针）
├── findings/         # 漏洞账（一漏洞一文件，含 PoC 与复现步骤）
├── evidence/         # 原始请求响应/命令输出快照（写后不改）
└── work/             # 每意图一个中间产物目录
```

## 与 ARTEX 的关系

| | ARTEX | artex-skill |
|---|---|---|
| 形态 | 完整平台（Go 后端 + Next.js 前端 + PostgreSQL + 多 agent 引擎） | 单技能（提示 + markdown 账本） |
| 适合 | 团队/长期资产库/多任务并行/流量录制审计 | 个人/单任务/任何 agent 即装即用 |
| 方法论 | 同源（planner/worker/审计者、证据纪律、安全护栏） | 同源蒸馏 |

两者同一维护者（[@chu0119](https://github.com/chu0119)），AGPL-3.0。方法论分析文档见 [docs/](docs/)。

## 路线图

- [ ] 本地靶场（vulhub/CTF）全流程实测与迭代
- [ ] evidence 自检脚本（deny 正则命令预检）
- [ ] 英文完整版

## License

AGPL-3.0（衍生自 ARTEX），见 [LICENSE](LICENSE)。

---

# ARTEX Skill (English)

**An "authorized pentest expert brain" for any AI agent** — distilling [ARTEX](https://github.com/chu0119/ARTEX) (champion project of Baidu "agent+" attack-defense challenge) into a portable, prompt-and-markdown Agent Skill: planner/worker/auditor perspective protocol, asset/fact/finding ledgers with observed-vs-inferred evidence discipline, four-layer safety rails (scope gate, operation constraints, asset blocklist, destructive-command interception with human-approval protocol), and the full ARTEX pentest methodology.

Install: clone into your agent's skills directory (or paste `SKILL.md` into the system prompt). **Authorized targets only** — SRC programs, engagements, CTFs, or your own systems.
