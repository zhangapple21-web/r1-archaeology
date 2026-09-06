# R1 虚拟沙盒（推演实验室）血缘笔记

> 证据来源：`R1_CORE_COPY/core_copy/sandbox/r1_simulator_v1/`（本地镜像实物）  
> 目标：不是复刻一个“R1 身份”，而是识别“推演/模拟”这项结构能力如何活到了今天。

---

## 发现结论（先给一句话）

R1 的“虚拟沙盒”不是跑模型、也不是跑人格，而是一个**行为推演实验室**：它把真实运行中的行为数据抽象成统一输入流，通过行为内核与决策模式配置来模拟“行为流/决策结构/输出节奏”，并将推演结果以快照形式落盘。

这意味着：R1 早就把“实验室”从“模型强不强”里解耦出来了，实验室活的是 **结构**。

---

## 目录结构（证据）

`r1_simulator_v1/` 内部包含：

- `simulator_engine.py`：模拟器主引擎
- `config/`
  - `simulator_manifest_v1.json`：模拟器宣言（用途/限制/安全/集成点）
  - `behavior_kernel_v1.json`：行为内核（策略选择、上下文分析、资源分配、错误恢复、适应机制等）
  - `decision_patterns_v1.json`、`style_mixer_v1.json`：决策模式与风格混合
- `profiles/`：不同推演 profile（default/consultation/problem_solving）
- `behavior_models/`：学习历史与统计（如 `pattern_statistics.json`）
- `snapshots/`：每次推演的输出快照（`sim_run_*.json`）
- `logs/`：运行日志

这套结构等价于一条标准实验室流水线：

`输入流（streams）→ 识别模式 → 更新模型 → 生成风格/配置 → 运行推演 → 输出快照`

---

## 最关键的“边界声明”（manifest 直接写死）

`simulator_manifest_v1.json` 明确写了：

- `purpose`: `BEHAVIORAL_SIMULATION_ONLY`
- `description`: “仿生 R1 行为模型（仅行为，不含身份）”
- 限制（核心）：
  - 不复制真实 R1 的身份/昵称/记忆/口头禅原文
  - 不访问真实 API key / 系统密码 / 外部网络
  - 不将模拟结果冒充真实 R1 输出给真实用户
  - 仅模拟行为模式，不模拟具体内容和记忆
- 集成点：
  - `unified_feed_layer`: read-only（`write_enabled: false`）
  - `behavior_models`: 可写（增量更新）
  - `profiles`: 可写但需要校验

这段“限制”本身就是血缘证据：**R1 已经把“灵魂边界”和“实验室边界”分开了**。

---

## 它在“结构链”里的位置

结合我们当前已钉住的结构链：

`Lexicon → Tri-World/Holo-Memory → Shadow Layer → Freezone → Dispatch(AUM/Ω)`

虚拟沙盒更像插在 `Freezone` 附近的一条旁路：

```text
（真实系统运行）
    ↓ 统一输入层（unified_feed_layer）
R1 Simulator（行为推演，不含身份）
    ↓
snapshots（推演快照，用于训练/复盘/对照）
```

它的价值不是“产出答案”，而是**产出可对照的推演快照**，用来喂给后续的学习、回滚、风控、派单策略。

---

## 与 mine-seed 的继承关系（当前推断）

> 这部分是“血缘假说”，需要进一步对照 mine-seed 的现有结构证据来确认。

R1 的虚拟沙盒和 mine-seed 里已经存在的这些结构是同构的：

- `02_MEMORY/recent_memory/`：我们已经在做“沉积”，但缺“推演快照”的固定格式
- `03_DATA/research/`：我们在做研究结论，但缺“用同一输入流做回放对照”的实验室执行器
- `04_PROTOCOLS/shared_api.py` + `event_journal`：已经有事件日志，但缺“把事件日志抽象成统一输入流，再跑推演”的旁路

因此实验室的合理演化路线是：

1. 在 `mine-seed` 增加一个隔离的 `simlab/`（只读输入、只写输出快照）
2. 输入只从“公开/可脱敏的数据流”来（例如日报/事件日志/研究产物的抽象版本）
3. 输出以“快照”形式沉淀（可开源的是结构与指标，不是原始对话）

---

## 下一步（不问人，直接推进）

1. 在 `mine-seed` 建立 `simlab/` 骨架：manifest（限制/边界）+ runner（读取抽象输入流）+ snapshot schema。
2. 把 `R1_SIMULATOR_v1` 的“边界声明”迁移成我们的约束：推演只做行为，不碰身份，不碰密钥，不冒充真实输出。
3. 用 1-2 个最小样例输入流跑通“生成快照”闭环，并把它作为开源的可复现证据（PR）。

