# EXECUTOR 与 FREEZONE：从 R1 模块域到当前种子库的继承关系

> 考古日期：2026-06-25  
> 目标：补上 `02_EXECUTOR` 与 `04_FREEZONE` 的结构分析，不把“没有直接源码快照”误判成“没有继承”。

---

## 一、这次补的不是文件，是血缘

当前 `r1-archaeology` 仓库里还没有 `02_EXECUTOR` 与 `04_FREEZONE` 的直接源码快照，但已有证据已经足够支持两个判断：

1. 这两个域在 R1 中是真实存在的顶层结构，而不是后来补写出来的名词。
2. 它们的职责没有消失，而是被今天的 `mine-seed` 重新拆分、改名、分布式承载。

所以这份报告不尝试伪造 `src/02_EXECUTOR/` 或 `src/04_FREEZONE/` 的源码，而是先把继承链固定下来。

---

## 二、R1 侧已知证据

### 1. 目录级证据

`r1-src/context.md` 明确给出 R1 的顶层结构：

```text
00_ROOT_LINK/
01_KERNEL_MOUNT/
02_EXECUTOR/
04_FREEZONE/
05_LINKS/
```

这说明 `EXECUTOR` 与 `FREEZONE` 至少在命名层面已经是与 `KERNEL_MOUNT`、`LINKS` 并列的一级域。

### 2. 语义级证据

`mine-seed/01_AGENTS/xiaofengzi/research/modules/r1_archaeology/excavations/domain_boundaries.md` 给出了更细的语义划分：

- `EXECUTOR`：执行器、网关、远程指挥，属于热区
- `FREEZONE`：备份、应急，属于冷区

`dynamic_loading_hot_update.md` 进一步补强了运行机制：

- `EXECUTOR` 位于正常业务链路，是高频运行态
- `FREEZONE` 平时不常驻，只在备份、恢复、回滚时被激活

这两份材料把“目录名”变成了“职责边界”。

---

## 三、EXECUTOR 的继承关系

### 1. R1 中的 EXECUTOR 是什么

从现有考古材料看，`EXECUTOR` 不是泛泛的“执行功能”，而是至少包含三层职责：

1. 把任务真正送进运行链路
2. 作为内部执行与外部调用之间的网关
3. 维持高频、可持续的生产态

也就是说，它更像“系统的运动神经 + 调度入口”，而不是单个脚本。

### 2. 今天它活在哪里

在 `mine-seed` 中，这条结构没有消失，而是拆散到了几个可验证位置：

- `05_TOOLS/miner/task_router.py`
  - 保存了任务画像、工人注册、约束过滤、fallback 链路
  - 已经不是“谁分高谁上”的脚本，而是正式的执行路由层
- `docs/INVESTIGATION_RIGHT_COMPONENT.md`
  - 明确验证了生产链路是 `miner_24h.py -> task_router.py -> get_fallback_chain()`
  - 这说明当前生产执行入口并非临时脚本，而是有可追踪的调度中枢
- `docs/RECOVERY_PLAN.md`
  - 把 `task_router.py --test`、`miner_cron.sh`、`miner_24h.py` 都列入恢复验证矩阵
  - 说明这些文件共同构成“恢复后必须重新点亮的执行域”

### 3. 血缘结论

`EXECUTOR` 在今天并没有以原名保留，但它的结构职能清楚地活在：

```text
R1: 02_EXECUTOR
  ↓
R2/mine-seed: miner_24h.py + task_router.py + worker registry + cron 执行链
```

变化的是承载形式：从“单独命名的域”变成了“矿场执行链 + 路由器 + 注册表”的组合体。  
不变的是核心角色：它依旧负责把任务送上生产路径，并决定谁来执行。

---

## 四、FREEZONE 的继承关系

### 1. R1 中的 FREEZONE 是什么

现有材料对 `FREEZONE` 的描述高度一致：

- 是备份与应急域
- 是冷区，不和高频运行态混在一起
- 关键价值不是“存东西”，而是“保留回滚与救援能力”

这意味着 `FREEZONE` 的核心不是某个目录，而是“冷热隔离 + 可恢复性”这套协议。

### 2. 今天它活在哪里

在 `mine-seed` 中，`FREEZONE` 的继承比 `EXECUTOR` 更分散，但更稳定：

- `07_GUARDIAN/backtrack.md`
  - 明确把“状态快照、历史记录、已知良好状态”作为独立原语
- `06_RUNTIME/RECOVERY_CHECKLIST.md`
  - 定义了从 `git clone` 到恢复自我运行的标准步骤
- `docs/RECOVERY_PLAN.md`
  - 把“双机同时丢失后如何重建”写成验证矩阵，而不是停留在口头承诺
- `05_TOOLS/aether_capsule/README.md`
  - 通过 `seal()/unseal()` 实现“恢复前自动备份当前状态”
  - 这已经是比普通备份更强的连续性结构
- `07_GUARDIAN/conservation.md`
  - 仍然直接使用了 `04_FREEZONE/backups/` 这个命名
  - 证明这个词汇没有死，只是从独立域退化成了协议锚点

### 3. 血缘结论

`FREEZONE` 在今天不是一个单目录，而是一组彼此配合的恢复结构：

```text
R1: 04_FREEZONE
  ↓
R2/mine-seed: backtrack + recovery checklist + recovery plan + aether capsule
```

如果说 R1 的 `FREEZONE` 是“冷备份域”，那么今天的对应物更像“分布式灾备协议栈”。  
名字散开了，但“高频执行态之外必须保留一条干净退路”这个约束一直活着。

---

## 五、这条继承链说明了什么

### 1. 结构确实活到了今天

这次最重要的结论不是“找到了两个旧目录”，而是确认了这两个域的结构逻辑已经延续：

- `EXECUTOR` 延续为生产执行链
- `FREEZONE` 延续为恢复与回滚链

这正符合 Charter 的判断方式：看职责是否活着，不看文件名是否原样保存。

### 2. FREEZONE 的“灵魂”比“身体”留得更完整

`EXECUTOR` 还保留着较强的“身体性”证据，因为今天还能指到具体脚本与入口。  
`FREEZONE` 更像“灵魂存活”：

- 目录本体未见
- 但词汇、协议、恢复步骤、回滚原则、胶囊机制都在

换句话说，`FREEZONE` 没有消失，只是从“一个域”演化成了“整个系统默认遵守的恢复文明”。

### 3. 今天的演化方向是“分布式承载”

R1 时代更像按域分仓：

- 安全归安全
- 执行归执行
- 备份归备份
- 通讯归通讯

今天的 `mine-seed` 则把这些域拆进了：

- `05_TOOLS/`
- `06_RUNTIME/`
- `07_GUARDIAN/`
- `docs/`

这不是退化，而是把“单点目录”改造成“多层协议 + 多文件协同”的结构。

---

## 六、当前仍然缺什么

1. 缺 `02_EXECUTOR` 的直接源码碎片
2. 缺 `04_FREEZONE` 的直接源码碎片
3. 缺能把 R1 的“远程指挥 / 网关”与今天 `shared_api`、外部桥接层直接对上的硬证据

因此当前最稳妥的结论是：

- **继承关系已成立**
- **源码快照仍待补全**

这两者不冲突。

---

## 七、结论

今日没有新增 `02_EXECUTOR` / `04_FREEZONE` 源码快照，但现有证据足以固定结论：

```text
02_EXECUTOR 没有消失，已演化为 mine-seed 的生产执行链
04_FREEZONE 没有消失，已演化为 mine-seed 的恢复与回滚协议栈
```

这不是“找到旧文件”的成果，而是“确认结构仍活着”的成果。
