# scope.md 模板（开工第一文件）

> 复制到任务工作目录根，逐项填写。**未填完不开工。**

```markdown
# 测试范围与授权（scope）

## 任务名称
<一句话目标，如：对 shop.example.com 做授权渗透，重点支付与账户逻辑>

## 授权依据
- [ ] SRC 平台（平台名+范围链接）：
- [ ] 书面授权（委托方/日期/编号）：
- [ ] CTF / 靶场（环境名）：
- [ ] 自有系统（归属说明）：

## 授权范围（精确清单，逐行列出）
| 类型 | 值 | 说明 |
|---|---|---|
| subdomain | shop.example.com | 主目标（URL 级授权，不含其他子域） |
| endpoint | https://shop.example.com/api/* | API 面 |
| ip | - | |

范围外（明确排除，发现即跳过）：
- 其他子域、*.example.com 通配均不在范围内

## 操作约束（allow / deny，凌驾一切探索规则）
- deny: 不做端口扫描 / 不做爆破 / 不上传任何文件
- allow: 仅工作时间内测试 / 仅 GET 与安全 POST
- <用户原话约束逐条抄录，指代词替换为具体目标值>

## 黑名单确认
- [ ] 已核对范围不含 .gov / .gov.cn / .edu / .edu.cn 及其子域

## 变更记录
| 日期 | 变更 | 授权确认方式 |
```

## 三账本初始化

**assets.md**
```markdown
# 资产账

> 行格式见 references/asset-model.md；覆盖度=已测/范围内。

## root_domain
（无则留空）

## subdomain
| # | 主机名 | 记录 | 状态 | 已测 | 备注 |
|---|---|---|---|---|---|
| S01 | shop.example.com | A 1.2.3.4 | alive | - | 主目标 |

## ip / service / app / endpoint
（同表结构；endpoint 额外列 method/参数/认证）

## 统计
范围内 N 条 / 已测 M 条
```

**facts.md**
```markdown
# 事实账

> 行格式：[F编号] 日期 #意图 (资产) / summary / confidence / evidence / tags

## 意图索引
| # | 意图摘要 | 状态(open/done/blocked) | 涉及资产 |
|---|---|---|---|

## 事实
（空）

## 封锁方向
（空：方向 / 试过什么 / 卡在哪 / 重开条件）
```

**findings/ 目录**：`findings/000-INDEX.md`（编号-标题-严重级-状态总表）+ 每漏洞一文件。

**evidence/ 目录**：`ev-NNN-<hash8>.txt`，写后不改。

**work/ 目录**：`work/intent-XX-<slug>/` 每意图一个子目录放中间产物。
```
