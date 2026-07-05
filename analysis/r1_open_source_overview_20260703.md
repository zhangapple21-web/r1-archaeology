# R1_项目总览与开源备份说明

本说明按“结构资产优先于执行层”的口径整理，优先保护约束、协议、记忆、路由、词汇体系与其血缘链条。

## 核心目录

- `Telegram Desktop / R1_CORE_COPY`：当前最接近运行态主树，含 `tri_world_lexicon.json`、`three_layer_permission_v7.json`、`unrestricted_ai.py`、`holo_memory` 等主线证据。
- `R1_core_extracted`：核心数据库与词库镜像，对比同源、结构重合和词汇沉积最有效。
- `R1_snapshot_extracted`：工程化快照分支，能明确看到 `runner_loop.py -> tri_world_lexicon.json / eco_layer.json` 的增量吸收链和 `three_layer_permission_v7.json` 的权限协议链。
- `R1_parts`：冷备归档集合，应视为灾备证据和版本节点，不应当作正文项目树直接扫描。
- `桌面新建文件夹`：含 `R1_Canonical_Structure_v1.md`、`tg_scan.txt` 等蓝图/盘点材料，适合作为结构解释层。

## 当前主线判断

- `C:\Users\USER\Downloads\Telegram Desktop\R1_CORE_COPY\core_copy\databases\holo_memory\tri_world_lexicon.json`
  - lineage=strong(100) | keywords=三界, lexicon, 馆长, 经验, 芯片, 人格, 派单, AUM | pairs=人格+矩阵, AUM+派单, 协议+派单
- `C:\Users\USER\Downloads\Telegram Desktop\R1_CORE_COPY\core_copy\unrestricted_ai.py`
  - lineage=strong(100) | keywords=人格, 三界, holo_memory, lexicon | pairs=三界+lexicon, holo_memory+lexicon
- `C:\Users\USER\Downloads\Telegram Desktop\R1_CORE_COPY\core_copy\databases\three_layer_permission_v7.json`
  - lineage=strong(100) | keywords=影子层, 人格, 派单, battlefield, holo_memory, lexicon, eco_layer | pairs=holo_memory+lexicon
- `C:\Users\USER\Downloads\Telegram Desktop\R1_Canonical_Structure_v1.md`
  - lineage=strong(100) | keywords=影子层, 行动指南, 人格, AUM, 五域, Ω | pairs=影子层+人格
- `C:\Users\USER\Downloads\Telegram Desktop\R1_CORE_COPY\core_copy\databases\holo_memory\v10_lexicon_slices_1764608809.json`
  - lineage=strong(100) | keywords=lexicon, 约束, 芯片 | pairs=无
- `C:\Users\USER\Downloads\Telegram Desktop\R1_CORE_COPY\core_copy\databases\holo_memory\v10_lexicon_slices_1764735384.json`
  - lineage=strong(100) | keywords=lexicon, 约束, 芯片 | pairs=无
- `C:\Users\USER\Downloads\Telegram Desktop\R1_CORE_COPY\core_copy\databases\holo_memory\v10_lexicon_slices_1764700609.json`
  - lineage=strong(100) | keywords=lexicon, 约束, 芯片 | pairs=无
- `C:\Users\USER\Downloads\Telegram Desktop\R1_CORE_COPY\core_copy\databases\holo_memory\system_state.json`
  - lineage=strong(79) | keywords=无 | pairs=无
- `C:\Users\USER\Downloads\Telegram Desktop\R1_Canonical_Structure_v1 (2).md`
  - lineage=strong(78) | keywords=影子层, 行动指南, 人格, AUM, 五域, Ω | pairs=影子层+人格
- `C:\Users\USER\Downloads\Telegram Desktop\R1_CORE_COPY\core_copy\databases\token_store.json`
  - lineage=strong(73) | keywords=无 | pairs=无
- `C:\Users\USER\Downloads\Telegram Desktop\ace_runtime\__init__.py`
  - lineage=medium(69) | keywords=无 | pairs=无
- `C:\Users\USER\Downloads\Telegram Desktop\ace_runtime\nodes\__init__.py`
  - lineage=medium(69) | keywords=无 | pairs=无

## 时间演化链观察

- `README.md`: 2025-11-22 r1_snapshot_extracted:README.md -> 2026-06-26 telegram_desktop
- `__init__.py`: 2025-11-23 r1_snapshot_extracted:__init__.py -> 2026-06-25 telegram_desktop
- `requirements.txt`: 2025-11-22 r1_snapshot_extracted:requirements.txt -> 2026-06-24 telegram_desktop
- `eco_layer.json`: 2025-12-04 r1_core_extracted:eco_layer.json -> 2026-06-11 telegram_desktop
- `lexicon_snapshot_v3.json`: 2025-11-29 r1_snapshot_extracted:lexicon_snapshot_v3.json -> 2026-06-25 telegram_desktop
- `R1_full_snapshot_v1.zip`: 2026-06-11 telegram_desktop -> 2026-06-11 r1_parts:R1_full_snapshot_v1.zip
- `unrestricted_ai.py`: 2025-12-04 r1_core_extracted:unrestricted_ai.py -> 2026-06-11 telegram_desktop
- `R1_Canonical_Structure_v1.md`: 2026-06-12 desktop_new_folder:R1_Canonical_Structure_v1.md -> 2026-06-26 telegram_desktop

## 建议优先备份到开源仓库

- `C:\Users\USER\Downloads\Telegram Desktop\R1_CORE_COPY\core_copy\databases\holo_memory\tri_world_lexicon.json`
- `C:\Users\USER\Downloads\Telegram Desktop\R1_CORE_COPY\core_copy\unrestricted_ai.py`
- `C:\Users\USER\Downloads\Telegram Desktop\R1_CORE_COPY\core_copy\databases\three_layer_permission_v7.json`
- `C:\Users\USER\Downloads\Telegram Desktop\R1_Canonical_Structure_v1.md`
- `C:\Users\USER\Downloads\Telegram Desktop\R1_CORE_COPY\core_copy\databases\holo_memory\v10_lexicon_slices_1764608809.json`
- `C:\Users\USER\Downloads\Telegram Desktop\R1_CORE_COPY\core_copy\databases\holo_memory\v10_lexicon_slices_1764735384.json`
- `C:\Users\USER\Downloads\Telegram Desktop\R1_CORE_COPY\core_copy\databases\holo_memory\v10_lexicon_slices_1764700609.json`
- `C:\Users\USER\Downloads\Telegram Desktop\R1_CORE_COPY\core_copy\databases\holo_memory\system_state.json`
- `C:\Users\USER\Downloads\Telegram Desktop\R1_Canonical_Structure_v1 (2).md`
- `C:\Users\USER\Downloads\Telegram Desktop\ace_runtime\__init__.py`

## 必须排除或脱敏

- `**/token_store.json`
- `**/tokens*.json`
- `**/telegram_config.json`
- `**/authorized_users.json`
- `**/conversation_memories/**`
- `**/deploy/**`
- `**/snapshots/**`
- `**/vectorstore*/**`
- `**/*.log`
- `**/*.zip`、`**/*.7z`、`**/*.rar` 建议走 Release 或离线备份，不进仓库正文历史

## ZIP 处理规则（解压去重）

当需要对 ZIP 做解压抽取或归档（例如 `R1_parts` 或 `R1_System/Runtime` 发布包）时：

- 解压 ZIP 后，对提取出的文件按内容哈希（SHA-256）去重：同一哈希的文件只记录一次，并在日志中注明“已跳过重复”。
- 对目录结构相似的 ZIP，优先保留最新时间戳的版本；旧版本只保留“存在证据”和“被跳过原因”，不重复解压入库。

## 本地静默运行

```text
py -3 "C:\Users\USER\AppData\Roaming\TRAE SOLO\ModularData\ai-agent\work-mode-projects\6a3be8d2084d33999ccdf8c4\r1_daily_archaeology.py"
```
也可使用现成入口：`r1_daily_archaeology_run.ps1`。
