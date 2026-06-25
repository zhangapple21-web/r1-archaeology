# r1-archaeology

R1 考古固化仓库：把 R1 时代的关键源码碎片按原始目录语义保存，同时把每次考古产出（对比报告、策略分析等）沉淀到 `analysis/`。

## 目录结构

```text
r1-archaeology/
├── src/                    # R1 源码（按原始语义组织）
│   ├── 01_KERNEL_MOUNT/
│   ├── 05_LINKS/
│   └── ...
├── analysis/               # 考古分析成果
│   ├── security_policy_report.md
│   ├── network_bridge_comparison.md
│   └── ...
└── README.md
```

## 当前已固化内容

- `src/05_LINKS/Network_Bridge.ps1`：R1 网络桥（内网穿透 + 广播宣告）
- `src/01_KERNEL_MOUNT/security_policy.js`：R1 安全策略门禁（黑名单优先 → 宇宙对齐 → 可选白名单保护态）
- `analysis/security_policy_report.md`：对 `security_policy.js` 的考古分析报告
- `analysis/network_bridge_comparison.md`：R1 Bridge vs 当前矿场 zrok 公网桥对比报告

## 第2周（本周）新增

- `analysis/win_broadcast_analysis.md`：Broadcast-Presence 路线深挖，`win_broadcast.ps1` 溯源与架构重建
- `analysis/full_src_scan_report.md`：全源码目录扫描报告（含缺失目录推测）
- `analysis/worker_auth_migration_scheme.md`：R1 三层门禁 → zrok Cloudflare Worker 鉴权迁移方案

## 备注

仓库默认以"可复盘"为第一原则：每个 `analysis/` 文档都应能指回 `src/` 中的具体文件与符号，便于后续追溯与社区公开记录同步。