# 深圳 R23 训练入口（2026-11-19 直行间距更新）

根目录：`/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256`；总目录的 `CURRENT/` 指向当前版本。先读 `DATASETS.json` 和 `TRAINING_CONTRACT.json`。默认 `k64`，可选 `k32`、`rvq` 和原有 `k256`。

K32/K64 使用深圳与 HM3D 的质检保留轨迹联合拟合，本版增加近直行至少5cm间距约束；RVQ 保持原来的逐轨迹等权联合码本。 K256 保持深圳既有码本。三种联合分支各 80,475 条轨迹、5,167,420 个节点。

K32/K64 近直行码字（|左向|<0.5cm且|转角|<3°）前进距离间隔至少5cm，分别保留13和16个；其他既有码字向量保持一致，释放的1和9个ID重新拟合到真实侧向动作；原始归一化、STOP及每步1个运动token保持一致；RVQ沿用当前联合码本。

- K32：`0429002c179e4223b21a5faf5fd00ae18326f7cd5b2384be22dea9799cc0c16b`；索引 `k32/training_samples/index.json`；码本 `k32/codebook.npz`。
- K64：`5f633256e06b28b78da1a16c370c37711212d7146c20c12078e5cd38a446aeb7`；索引 `k64/training_samples/index.json`；码本 `k64/codebook.npz`。
- RVQ：`8c8f595a61c63f35a83e7935c087debb188bcdc317753a20dcab43b4d9786e18`；索引 `rvq/training_samples/index.json`；码本 `deps/codebook.npz`。

运行：`cd /cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256 && python reader.py --branch k64 --batch-size 2`。训练适配器为 `training_adapter.TrainingDataset`，需训练端注册原子 token 并映射模型词表。K32/K64 的 0 是 STOP；RVQ 只有完整 `[0,0,0]` 是 STOP，单层 0 为普通码字；使用有效 mask，并只执行 `decoded[0]` 后重新观测。

视频在共享 CPFS 的 `/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/source/runs/output`；历史仅含真实保留节点。新码本全量开环误差和码频见 `site/report_data/JOINT_FULL_METRICS.json`，旧码本误差不可沿用。未验证策略训练或真机效果。
