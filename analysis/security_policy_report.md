# R1 `security_policy.js` 安全策略考古（黑名单优先 → 宇宙对齐 → 白名单准入）

> 依据：`r1-src/security_policy.js`  
> 背景：R1 LOCK SYSTEM，位于 `01_KERNEL_MOUNT/` 安全域的气质（“五域 + 安全”）  
> 注：`loadConfig()` 的具体实现不在本次快照内，以下只基于策略模块本身做“门禁逻辑”与设计哲学考古。

## 模块一句话

这是一个“可配置的三段式门禁（guard）”：**先按身份黑名单一票否决，再做来源宇宙合法性校验，最后在启用白名单保护时做白名单准入**。

## 关键对象与输入输出

- 配置来源：`const config = loadConfig();`（来自 `./five_realms.loader`，暗示配置与“五域/五域装载器”同源）
- 三组集合：
  - `blacklist`: `config.security.blacklist`
  - `whitelist`: `config.security.whitelist`
  - `universeKeys`: `config.security.universeKeys`
- 核心 API：
  - `isAllowed(identity, sourceUniverse) -> { allowed: boolean, reason?: string }`
  - `getPolicySummary() -> { mode, universes, protected }`

输出设计很“事件化”：拒绝时给出明确 `reason`（可用于日志、审计、或上层策略路由）。

## 三层逻辑拆解（按执行顺序）

`isAllowed()` 的判定顺序就是策略哲学本体：

### 1) 黑名单优先（Blacklist Priority）

```js
if (blacklist.has(identity)) {
  return { allowed: false, reason: "IDENTITY_BLACK-LISTED" };
}
```

含义：**身份的“绝对否决权”高于一切**，即便来源宇宙合法、甚至后续白名单可能允许，也不会继续评估。

这种选择通常意味着：
- 运营侧/应急侧需要一个“立即止血”的闸刀
- 风险模型偏向“宁可误杀，不可漏放”（在攻防高压期很常见）

### 2) 宇宙对齐校验（Universe Alignment）

```js
if (!universeKeys.has(sourceUniverse)) {
  return { allowed: false, reason: "UNIVERSE_UNALIGNED" };
}
```

含义：R1 的系统不是单宇宙运行，它明确承认并管理多个来源宇宙（注释里写了 `Win, Mac, GPT, TG`），因此先问一句：**你来自哪个宇宙？这个宇宙是否被认可？**

这一步和传统“来源 IP / Origin / Client 类型”很像，但它用更抽象的“宇宙”概念统一了多端/多代理/多入口的身份来源。

关键哲学点：
- R1 不是只做“谁（identity）”的治理，也做“从哪里来（sourceUniverse）”的治理
- 把跨平台、跨代理、跨协议的差异，压缩成一个可配置枚举集合 `universeKeys`

### 3) 白名单准入（Whitelist Admission，可选）

```js
if (whitelist.size > 0 && !whitelist.has(identity)) {
  return { allowed: false, reason: "NOT_IN_WHITE-LIST" };
}
```

含义：白名单不是默认强制，而是一个**可开启的“保护态”**：

- `whitelist.size === 0`：系统处于“开放态”（只要宇宙对齐且不在黑名单，就放行）
- `whitelist.size > 0`：系统切换到“保护态”（只允许白名单内的身份）

这里的细节很关键：它不是 “blacklist vs whitelist 二选一”，而是 **blacklist 永远有效，whitelist 是可选的额外收紧**。

## 设计哲学总结：拒绝优先、边界前置、保护态可切换

把三层拼起来，会得到一个很 R1 的世界观：

1. **拒绝优先**：先给“立即拒绝”的路径（黑名单），保障事故响应速度。
2. **边界前置**：在谈身份授权前，先确认“宇宙边界”是否对齐；这相当于把“跨域/跨端”的风险集中在一个入口处解决。
3. **保护态开关**：白名单不强迫常开，但一旦需要“锁仓”，往配置里塞一个 whitelist 就能把系统切到强保护模式。

## `getPolicySummary()`：策略可观测性的伏笔

```js
getPolicySummary: () => ({
  mode: "Blacklist-Priority",
  universes: Array.from(universeKeys),
  protected: whitelist.size > 0
})
```

这个摘要接口是很实用的“自描述”：

- `mode` 明确上层该怎么理解拒绝顺序（避免误读）
- `universes` 能直接用于展示/审计（系统当前承认哪些宇宙）
- `protected` 把“白名单保护态”显式暴露出来，利于排障：为什么突然全拒了？因为进入 protected 了。

## 对矿场的启示（可直接迁移的点）

1. **把“来源维度”抽象成可配置枚举**
   - 现在矿场的公网入口（zrok + Worker）天然跨环境；R1 的 `sourceUniverse` 抽象可以迁移成“入口类型/调用来源/代理层级”的统一字段（例如 `worker|cli|dashboard|tg-bot`）。

2. **白名单作为“保护态开关”非常适合矿场**
   - 在需要临时封仓、事故演练、或数据敏感期时，只要 whitelist 非空就进入“只许通过”的模式，运维成本低。

3. **拒绝原因码是必需品**
   - `IDENTITY_BLACK-LISTED / UNIVERSE_UNALIGNED / NOT_IN_WHITE-LIST` 这类 reason 非常利于 Worker 侧做可观测与分流（记录、告警、分级响应）。

## 潜在风险/缺口（从代码里能看出来的）

- **identity 的格式与强度未知**：如果 identity 是可伪造的字符串，黑/白名单价值会下降；更理想的是 identity 绑定签名或不可伪造凭证（token / key / mTLS 等）。
- **universeKeys 是硬边界但缺少降级策略**：一旦配置漏了某个宇宙，会全量拒绝（可能是设计目标，也可能需要灰度/告警）。
- **缺少速率/时间维度**：这是纯“允许/拒绝”策略，没有次数限制、TTL、会话状态等，说明它更像基础门禁层，可能上层还有更复杂的安全域。

## 彩蛋式解读：为什么叫 Multi-Universe

注释写了 “Multi-Universe (Win, Mac, GPT, TG)”，这不是装饰，而是在告诉你：R1 的安全边界不是“一个服务一个入口”，而是**一套系统跨多个入口宇宙共用同一门禁**。这和你正在考古的“五域边界划分法”在气质上是一致的：先立边界，再谈流动。

