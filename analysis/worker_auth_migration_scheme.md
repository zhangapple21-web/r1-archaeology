# R1 安全策略迁移方案：三层门禁 → zrok Cloudflare Worker 鉴权

> 基于：`analysis/security_policy_report.md` 对 R1 `security_policy.js` 的三层门禁分析
> 目标：将 R1 的 "黑名单优先 → 宇宙对齐 → 白名单保护态" 设计迁移到矿场当前的 zrok + Cloudflare Worker 架构

## 1. 映射关系：R1 概念 → Worker 实现

| R1 概念 | Worker 实现载体 |
|---------|----------------|
| `identity` | `request.cf?.tlsClientAuth?.cert?.fingerprint` / JWT sub / Token Header |
| `sourceUniverse` | `request.headers.get('x-source-universe')` / URL 路径前缀 / `cf.continent` |
| `blacklist` | KV/Env 中的 `BLACKLISTED_IDS` JSON 数组 |
| `whitelist` | KV/Env 中的 `WHITELISTED_IDS` JSON 数组（空 = 关闭保护态） |
| `universeKeys` | KV/Env 中的 `UNIVERSE_KEYS` JSON 数组 |
| `getPolicySummary()` | Worker 的 `/__policy` 管理端点 |

## 2. 核心 Worker 代码

```javascript
// === R1-Inspired Worker Auth Guard ===
// 移植自 security_policy.js 的三层门禁
// 策略顺序：黑名单优先 → 宇宙对齐 → 白名单保护态

// 配置来源（env / KV）
const AUTH_CONFIG = {
  // 使用环境变量或 KV 存储，支持热更新
  get blacklist() { return JSON.parse(env.BLACKLIST || '[]'); },
  get whitelist() { return JSON.parse(env.WHITELIST || '[]'); },
  get universeKeys() { return JSON.parse(env.UNIVERSE_KEYS || '["worker","cli"]'); },
  get protectionMode() { return JSON.parse(env.WHITELIST || '[]').length > 0 ? 'PROTECTED' : 'OPEN'; }
};

// === 第一层：黑名单优先 ===
function checkBlacklist(identity) {
  if (AUTH_CONFIG.blacklist.includes(identity)) {
    return { passed: false, reason: 'IDENTITY_BLACK-LISTED', status: 403 };
  }
  return { passed: true };
}

// === 第二层：宇宙对齐 ===
function checkUniverseAlignment(sourceUniverse) {
  if (!AUTH_CONFIG.universeKeys.includes(sourceUniverse)) {
    return { passed: false, reason: 'UNIVERSE_UNALIGNED', status: 403 };
  }
  return { passed: true };
}

// === 第三层：白名单保护态（可选） ===
function checkWhitelistProtection(identity) {
  if (AUTH_CONFIG.protectionMode === 'PROTECTED' && 
      !AUTH_CONFIG.whitelist.includes(identity)) {
    return { passed: false, reason: 'NOT_IN-WHITELIST', status: 403 };
  }
  return { passed: true };
}

// === 主门禁 ===
function r1AuthGuard(identity, sourceUniverse) {
  // 严格按 R1 顺序执行
  const checks = [
    checkBlacklist(identity),
    checkUniverseAlignment(sourceUniverse),
    checkWhitelistProtection(identity)
  ];
  
  for (const check of checks) {
    if (!check.passed) return check;
  }
  
  return { passed: true, reason: 'ALLOWED' };
}

// === Worker fetch handler ===
export default {
  async fetch(request, env) {
    // 1. 提取身份
    const identity = extractIdentity(request);
    // 2. 提取来源宇宙
    const sourceUniverse = extractSourceUniverse(request);
    
    // 3. 执行门禁
    const auth = r1AuthGuard(identity, sourceUniverse);
    if (!auth.passed) {
      return new Response(JSON.stringify({
        error: auth.reason,
        policy: getPolicySummary()
      }), { 
        status: auth.status,
        headers: { 'Content-Type': 'application/json' }
      });
    }
    
    // 4. 放行，传递身份上下文
    request.r1 = { identity, sourceUniverse, auth };
    return await fetch(request);
  }
};
```

## 3. 身份与来源提取策略

### 3.1 `extractIdentity()` 实现选项

| 方案 | 提取源 | 安全等级 | 适用场景 |
|------|--------|---------|---------|
| **Token-Based** | `Authorization: Bearer <jwt>` 的 `sub` 字段 | ⭐⭐⭐⭐⭐ | 生产推荐 |
| **mTLS** | `request.cf.tlsClientAuth.certificate.fingerprint` | ⭐⭐⭐⭐⭐ | 高安全场景 |
| **API Key** | `X-API-Key` 请求头 | ⭐⭐⭐⭐ | 兼容现有系统 |
| **IP-Based** | `request.headers.get('CF-Connecting-IP')` | ⭐⭐ | 仅内网/LAN |
| **混合** | Token + IP 双重绑定 | ⭐⭐⭐⭐⭐ | 推荐方案 |

**推荐实现（JWT + 签名）**：

```javascript
function extractIdentity(request) {
  // 优先 JWT
  const authHeader = request.headers.get('Authorization');
  if (authHeader?.startsWith('Bearer ')) {
    try {
      const token = authHeader.slice(7);
      const payload = verifyAndDecodeJWT(token, env.JWT_SECRET);
      return payload.sub;  // 用户/服务身份
    } catch {
      return null;
    }
  }
  
  // 降级到 API Key
  const apiKey = request.headers.get('X-API-Key');
  if (apiKey && env.API_KEY_MAP?.[apiKey]) {
    return env.API_KEY_MAP[apiKey];
  }
  
  return null;  // 无法识别身份
}
```

### 3.2 `extractSourceUniverse()` 实现选项

```javascript
function extractSourceUniverse(request) {
  // 显式声明（客户端自报来源）
  const explicit = request.headers.get('X-Source-Universe');
  if (explicit && AUTH_CONFIG.universeKeys.includes(explicit)) {
    return explicit;
  }
  
  // 自动推断（基于入口特征）
  const url = new URL(request.url);
  const pathPrefix = url.pathname.split('/')[1];
  
  // URL 路径映射宇宙
  const PATH_UNIVERSE_MAP = {
    'worker': 'worker',      // 默认 Worker 入口
    'cli': 'cli',            // CLI 工具调用
    'api': 'worker',         // API 请求
    'dashboard': 'dashboard', // 管理面板
    'tg': 'tg-bot'           // Telegram Bot
  };
  
  if (PATH_UNIVERSE_MAP[pathPrefix]) {
    return PATH_UNIVERSE_MAP[pathPrefix];
  }
  
  // 兜底：检查 User-Agent 或 cf 特征
  const ua = request.headers.get('User-Agent') || '';
  if (ua.includes('curl') || ua.includes('wget')) return 'cli';
  
  return 'unknown';  // 宇宙未对齐会触发第二层拒绝
}
```

## 4. 策略热更新设计

R1 的 `loadConfig()` 暗示了可热加载的配置。Worker 侧实现：

```javascript
// === 策略热更新端点（受保护的管理接口）===
async function handlePolicyUpdate(request, env) {
  // 仅允许 dashboard 宇宙的管理员操作
  if (request.method !== 'POST') {
    return new Response('Use POST', { status: 405 });
  }
  
  const update = await request.json();
  
  // 支持部分更新
  if (update.blacklist) {
    await env.KV.put('BLACKLIST', JSON.stringify(update.blacklist));
  }
  if (update.whitelist) {
    await env.KV.put('WHITELIST', JSON.stringify(update.whitelist));
  }
  if (update.universeKeys) {
    await env.KV.put('UNIVERSE_KEYS', JSON.stringify(update.universeKeys));
  }
  
  return new Response(JSON.stringify({
    status: 'updated',
    summary: getPolicySummary(env)
  }), { headers: { 'Content-Type': 'application/json' } });
}
```

### 策略管理端点一览

| 端点 | 方法 | 功能 | 需要保护？ |
|------|------|------|-----------|
| `GET /__policy` | GET | 查看当前策略摘要 | ✅ 仅 dashboard 宇宙 |
| `POST /__policy` | POST | 更新策略配置 | ✅ 仅 dashboard + 管理员 |
| `GET /__policy/audit` | GET | 查看拒绝日志 | ✅ 仅 dashboard + 管理员 |
| `POST /__policy/reset` | POST | 重置为开放态（清空白名单） | ✅ 需要双重确认 |

## 5. 审计日志 & 可观测性

```javascript
// === 审计日志 ===
function auditLog(request, authResult, env) {
  const identity = request.r1?.identity || 'unknown';
  const universe = request.r1?.sourceUniverse || 'unknown';
  
  console.log(JSON.stringify({
    timestamp: new Date().toISOString(),
    identity,
    universe,
    path: new URL(request.url).pathname,
    method: request.method,
    action: authResult.passed ? 'ALLOW' : 'DENY',
    reason: authResult.reason || 'ALLOWED',
    cfRay: request.headers.get('CF-Ray')
  }));
  
  // 拒绝事件可触发告警
  if (!authResult.passed) {
    incrementMetric('r1_auth_deny_total', {
      reason: authResult.reason,
      identity
    });
  }
}
```

## 6. 渐进式部署方案

### 阶段一：审计模式（安全验证期）
- Worker 部署 R1 门禁逻辑，但 **只记录不拒绝**
- 观察 72 小时，验证身份/宇宙提取是否正确

```javascript
function r1AuthGuard(identity, sourceUniverse) {
  const result = _internalCheck(identity, sourceUniverse);
  // 审计模式：记录结果但不拒绝
  auditLog({ identity, sourceUniverse, ...result }, env);
  return { passed: true };  // 始终放行
}
```

### 阶段二：软拒绝期
- 仅拒绝明确的黑名单身份（最低风险）
- 宇宙未对齐和白名单保护态仅告警，不阻止

### 阶段三：全量保护期
- 三层完整生效
- 保留 `/__policy` 紧急降级能力

## 7. 紧急降级流程

当事故排障需要绕过门禁：

```
1. 设置 env.FORCE_OPEN = "true"
   → r1AuthGuard 跳过所有检查
   
2. 排障完成后：
   wrangler kv:key put --binding=AUTH_CONFIG FORCE_OPEN false
   
3. 确认恢复：
   GET /__policy → 确认 mode 恢复为 Blacklist-Priority
```

## 8. 对比 R1 原版：迁移等价性

| R1 原生特性 | Worker 迁移实现 | 等价度 |
|-------------|----------------|--------|
| `blacklist.has(identity)` | `env.BLACKLIST.includes(identity)` | ✅ 完全等价 |
| `universeKeys.has(sourceUniverse)` | `env.UNIVERSE_KEYS.includes(sourceUniverse)` | ✅ 完全等价 |
| `whitelist.size > 0` 判断保护态 | `env.WHITELIST.length > 0` | ✅ 完全等价 |
| `getPolicySummary()` | `GET /__policy` 端点 | ✅ 功能等价（更丰富） |
| `loadConfig()` 热加载 | KV 动态读取 | ⬆️ 更强（实时生效） |
| identity 强度依赖调用方 | JWT 签名验证 | ⬆️ 更强（不可伪造） |

## 9. 部署约束与注意事项

1. **KV 延迟**：Worker 读取 KV 可能有 200ms+ 延迟。高频策略建议用 `env.XXX` 环境变量（需重新部署）。**推荐混合**：静态策略用 env，动态更新用 KV 并配合 `wrangler publish` 刷新。
2. **身份提取的可靠性**：JWT 需要验证签名，引入依赖（jsonwebtoken polyfill）。建议用 Cloudflare 原生 `crypto.subtle` 验签，避免第三方库。
3. **宇宙对齐的粒度**：R1 只有 4 个宇宙值（Win/Mac/GPT/TG），Worker 场景可能需要更细：`worker|cli|dashboard|tg-bot|internal-cron`。
4. **审计日志上限**：Cloudflare Workers 免费版 `console.log` 有 128KB/请求限制，生产环境建议通过 `fetch` 写入外部日志服务或使用 `cf.waitUntil` 异步写入 R2。

## 10. 与 win_broadcast 的联动设计

如果将广播发现与 Worker 鉴权结合：

```
内网广播发现（win_broadcast 模式）
        ↓
节点发现 R1 服务，获取 universe + identity
        ↓
通过 Worker 公网入口调用时携带
  - Header: X-Source-Universe: <universe>
  - Authorization: Bearer <JWT with identity>
        ↓
Worker 三层门禁判定
  - 黑名单优先 → 宇宙对齐 → 白名单保护态
```

这种联动使得**内网发现 + 公网鉴权**形成完整链路：内网广播解决"谁在哪"，Worker 鉴权解决"谁可以"。