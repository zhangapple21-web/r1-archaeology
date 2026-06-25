# R1 考古：全源码目录扫描报告（第2周）

> 扫描日期：2026-06-25
> 扫描范围：`src/` 下所有目录
> 扫描方法：GitHub Tree API (`?recursive=1`)

## 目录全景

```
src/
├── 01_KERNEL_MOUNT/           # [已分析] 安全域：内核挂载
│   └── security_policy.js     ✅ 已分析（第1周）
├── 05_LINKS/                  # [已分析] 网络域：链接/桥
│   └── Network_Bridge.ps1     ✅ 已分析（第1周）
```

**结论：src/ 下仅存在 2 个目录，且均在 第1周 完成分析。**

目录编号跳跃（01 → 05）暗示中间可能存在计划的但未固化的模块：
- `02_*`：推测为 **SYSTEM_STATE / 系统状态** 域
- `03_*`：推测为 **DATA_FLOW / 数据流** 域
- `04_*`：推测为 **AGENT_CORE / 代理核心** 域

这些目录编号在 R1 原始开发环境（`trae_projects\laozhang_ai`）中可能存在，但未被选入考古固化范围。

## 已分析文件清单

| 文件 | 目录 | 分析报告 | 分析程度 |
|------|------|---------|---------|
| `security_policy.js` | `01_KERNEL_MOUNT` | `security_policy_report.md` | 完整——三层门禁逻辑、设计哲学、矿场启示 |
| `Network_Bridge.ps1` | `05_LINKS` | `network_bridge_comparison.md` | 完整——R1桥 vs zrok公网桥对比、后续路线建议 |

## 第2周新增分析

| 文件 | 说明 |
|------|------|
| `win_broadcast_analysis.md` | P0任务：Broadcast-Presence路线——基于线索的架构重建 |
| `worker_auth_migration_scheme.md` | P2任务：安全策略迁移——三层门禁→Worker鉴权 |

## 缺失目录的推测（基于五域体系）

R1 的设计文档中提到了"五域"概念（`five_realms.loader`）。01 和 05 之外缺失的 02-04 域推测如下：

### 推测：02_SYSTEM_STATE（系统状态域）
- **功能**：系统的运行状态管理、心跳、配置热加载
- **理由**：`security_policy.js` 引用了 `./five_realms.loader`，暗示配置是由独立加载器管理的。02 域很可能就是"加载器"所在位置。
- **可能文件**：`config_loader.js`、`state_manager.ps1`、`health_check.ps1`

### 推测：03_DATA_FLOW（数据流域）
- **功能**：宇宙间的数据交换、消息路由、协议转换
- **理由**：R1 跨 Win/Mac/GPT/TG 四个宇宙运行，必然需要数据流动的管道
- **可能文件**：`message_bus.py`、`protocol_adapter.js`、`router.ps1`

### 推测：04_AGENT_CORE（代理核心域）
- **功能**：AI 代理逻辑、任务调度、决策引擎
- **理由**：R1 本质是 AI 系统，"代理核心"是业务逻辑中心
- **可能文件**：`agent_loop.py`、`task_scheduler.js`、`decision_engine.ps1`

## 关于五域编号的深层解读

`01 → 05` 的跳跃不是随机的，它反映了一种**边界优先**的设计顺序：

1. **01_KERNEL_MOUNT**（安全域）被摆在第一位——先立安全边界
2. **05_LINKS**（网络/链接域）被摆在最后但现存——网络是最终出口
3. **02-04**（中间域）缺失——可能意味着：安全→系统→数据→代理→网络

这种设计哲学与 R1 整体气质一致：**先设防、再运行、最后通联**。

## 后续扫描建议

如果后续找到更多 R1 源码碎片，建议优先补充：
1. `src/02_SYSTEM_STATE/config_loader.js` —— 因为 `security_policy.js` 依赖它
2. `src/02_SYSTEM_STATE/five_realms.loader.js` —— 五域配置的入口点