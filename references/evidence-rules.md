# 证据规则：置信度、确认门槛与账本书写格式

> ARTEX record_fact / report_finding 工具纪律的单文件版。每次写 facts.md 或
> findings/ 之前重读。这是本 skill 的灵魂章节。

## 一、置信度：只有两个值

| 值 | 定义 | 例子 |
|---|---|---|
| `observed` | 我亲手执行并看到结果 | curl 返回了 admin 的手机号 |
| `inferred` | 我基于证据推断，未亲手触发 | 该版本存在已知 RCE（版本来自指纹） |

不存在 "confirmed-ish"、"高概率"、"基本确定"。要么看到，要么推断。

## 二、确认漏洞的唯一门槛

写成 confirmed finding 必须**同时满足**：

1. **真实触发**：拿到目标的行为改变——数据外带/回显/状态变更/权限提升。
2. **可复现证据**：完整请求+响应（或命令+输出）已存 `evidence/`，他人照做能复现。
3. **独立复现通过**：换路径/换会话再验证过一次（审计者视角，§methodology 心法4）。

**三禁**（只能产生 inferred 嫌疑，永不直接确认）：
- 禁：版本号/CPE 匹配到 CVE 就确认
- 禁：漏洞库/扫描器报告就确认
- 禁：补丁 diff 反推就确认

嫌疑的处理：记 facts.md（inferred）+ 归入待验证路线，由后续验证意图升格或降级。

## 三、facts.md 行格式（硬约束）

```markdown
- [F012] 2026-10-10 14:32 #intent-03 (subdomain api.example.com)
  summary: /api/v1/user 接口未鉴权，可拉取任意用户资料（换 id 测试 3 个账号均成功）
  confidence: observed
  evidence: evidence/ev-012-a3f8c2d1.txt (curl 请求+响应全文)
  tags: idor, auth-bypass
```

规则：
- **summary 写结论不写过程**：一句话说清"什么资产有什么性质"，多观察合并成一条，
  过程细节放 evidence 文件。
- **否定结论必须记**：`[F015] sub.example.com 无 80/443 以外端口（全端口扫描完毕，
  confidence: observed）`——封锁方向是后续规划的依据。
- **只写增量**：新观察更新旧条目（追加行），不重写旧内容。
- 每条事实归属一个意图编号（#intent-XX），血缘可追溯。

## 四、findings/ 文件规约

一漏洞一文件：`findings/001-shop-idor.md`，编号递增不复用。内容模板见
report-template.md。硬性要求：
- severity：critical / high / medium / low（判定标准见 report-template.md）
- 证据文件先存后写：finding 里引用的每个 evidence 文件必须已存在
- **证据变更不改历史**：复测推翻旧结论时新建 `002-…retest.md` 引用旧文件，
  旧 finding 标注"已被 002 复测修正"，原文保留
- 复测结论三值：reproduced / fixed / inconclusive（环境变化打不出结果≠已修复）

## 五、evidence/ 目录纪律

- 命名：`ev-NNN-<内容hash前8位>.txt`，写完**永不修改**
- 内容：原始请求响应 / 命令与完整输出 / 截图路径，附一行时间戳与来源命令
- **缺失结果记 unknown 不记成功**：命令输出丢失/截断关键部分 → 重跑取证，
  禁止凭记忆补写"应该是成功了的"

## 六、账本审计自查（收尾前过一遍）

```
□ 每条 fact 都有 confidence 与 evidence 指针？
□ 每个 confirmed finding 都有独立复现记录？
□ 封锁方向都写了"试过什么、卡在哪"？
□ 有没有把 inferred 混写成 confirmed 的条目？
□ scope 外发现都标了 out-of-scope 而不是展开测试？
```
