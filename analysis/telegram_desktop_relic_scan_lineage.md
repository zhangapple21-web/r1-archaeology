# Telegram Desktop 遗迹扫描：结构血缘速记（v0）

> 这份笔记只保留“能指向血缘”的结构线索，不保留大量文件清单。  
> 原则：**找继承关系，不做目录观光。**

---

## 这轮扫描的意义

“R1”这个名字是会换的，但结构不会：  
当扫描命中集中出现在 `holo_memory / lexicon / tri_world / freezone / shadow_layer / dispatch(AUM/Ω)` 这些词上时，就意味着我们抓到的是“文明构件”，不是某个项目名。

---

## 三个祖坟级锚点（可反向溯源）

### 1) `fuse_content.txt`：把“五域词汇”拼成可执行骨架

扫描结果显示它把这些东西串在一条链上：

- `Five-Realms`
- `local_lexicon_priority`
- `影子层 / 安全屋 / 自由区`
- `语义压缩`

从内容片段可确认它不是随手笔记，而是**带目录映射、带开关、带压缩词表的可执行配置/脚本化文本**：

- 有“出身目录”列表（含 `安全屋/影子层/自由区/图书馆/废墟/沙盒` 等）
- 有 `ENABLE` 开关，包含 `disk_scan / index_absorb / world_absorb` 这类“吸收/扫描”动作
- 有 `语义压缩` 词表，把“影子层、派单”等作为压缩符号进行检测

**血缘指向**：这更像“R1 的考古/吸收器”，是“观察 → 压缩 → 沉积”那条链的工程形态。

### 2) `tri_world_lexicon.json`：词库不是词典，是任务与路由的种子库

扫描结果显示 `tri_world_lexicon.json` 里同时出现：

- `lexicon`（词库升级/快照/切片）
- `人格`（人格路由/权重/风控）
- `派单`（任务分发与风控）
- `AUM-TASK`（把任务标注为资产/资源管理的一部分）

从片段能看到：它包含“词库升级包说明 + 大量正文语料/规则/任务模板”，而不是纯 JSON 配置。

**血缘指向**：词库在 R1 里并不只是“知识”，而是**路由矩阵与派单体系的输入**，属于“记忆高于参数 / 路由高于能力”的落地形态。

### 3) `offshore_dispatch_config.json`：派单体系把“Ω / Router∞ / AUM”写进配置

该文件结构上明确出现：

- `dispatch_mode: offshore`
- `core_components` 含 `Ω-Core / L∞-Core / Router∞ / Self-Field`
- `aum_module`（资产/资源管理）

同时也出现了明显敏感字段（例如 `invite_url`、`bot_secret` 等），证明它属于“可运行配置”，不是纯文档。

**血缘指向**：这就是“经验 → 约束 → 派单 → 执行”的配置侧证据：  
R1 的派单不是单点脚本，而是一个带治理视角（offshore）的系统化模块。

---

## 一条可工作的继承链（当前能被证据支撑）

在证据充分之前，先用“最小链”固定血缘：

```text
Lexicon（词库升级/切片/语料/任务模板）
  ↓ 提供语义材料与路由输入
Tri-World / Holo-Memory（多界记忆结构）
  ↓ 形成可检索/可拼接的语义底座
Shadow Layer（影子层：只观察不写/错误记录/老版本）
  ↓ 提供压缩、回滚、校验、叙述竞争
Freezone（自由区：演化/试验/人格材料）
  ↓ 允许突变与试错
Dispatch（AUM/Ω/Router∞：派单与治理视角）
```

如果这条链成立，那么“R2/mine-seed”里出现的：

- 约束系统（constraints）
- 路由（router）
- 研究域（research modules）
- 档案/记忆（02_MEMORY）

就不是“新设计”，而是对上述链条的再实现与再命名。

---

## 下一步怎么挖才是“找血缘”

1. **把 `语义压缩` 词表当作祖先 DNA**
   - 把它抽出来，对照 mine-seed 中的 `principles/constraints/router` 词汇，找“同义改名”的映射表。
2. **从 `tri_world_lexicon` 抽 “人格×派单×风控” 结构**
   - 不要搬运语料，只抽结构：角色、权重、触发条件、风控阈值、路由矩阵。
3. **从 `offshore_dispatch_config` 抽 “组件图”**
   - 只抽 key 和组件名，不带任何 secret/value：`Ω-Core / Router∞ / AUM / 自治层 layers`。

完成以上三步，就能把“R1 的名词”翻译成“今天还能识别的结构”。

