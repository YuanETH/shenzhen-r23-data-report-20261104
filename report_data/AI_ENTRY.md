# 深圳 R23 训练入口（2026-11-20 K64 短窗口FDE更新）

根目录：`/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256`；总目录的 `CURRENT/` 指向当前版本。先读 `DATASETS.json` 和 `TRAINING_CONTRACT.json`。默认 `k64`，可选 `k32`、`rvq` 和原有 `k256`。

深圳与 HM3D 质检保留轨迹共享码本。K64 本版主要优化未来5步和10步FDE；K32保持5cm间距版，RVQ保持原联合码本。 K256 保持深圳既有码本。三种联合分支各 80,475 条轨迹、5,167,420 个节点。

K64 以未来5步和10步重叠窗口的终点FDE为主要目标，联合优化码本与整条离线标签序列；5/10步使用同一条序列，每步仍为1个运动token、64个运动码字加STOP=0，近直行前进距离间隔至少5cm；K32、RVQ和深圳K256沿用当前版本。

K64 编码约定（2026-11-20）：标签使用真实未来路径，联合搜索同一条码字序列来减小5步和10步FDE，不能用单步最近码字复现当前标签。每步1个运动token、64个运动码字及STOP=0，解码仍为码字的局部[前向、左向、偏航]。离线编码器、源码和参数在 k64/encoder/。

- K32：`0429002c179e4223b21a5faf5fd00ae18326f7cd5b2384be22dea9799cc0c16b`；索引 `k32/training_samples/index.json`；码本 `k32/codebook.npz`。
- K64：`f0cc68c7892d8b43b1c1a9aa5a309db0b9053b6864b3fd73f96aa94d8249950b`；索引 `k64/training_samples/index.json`；码本 `k64/codebook.npz`。
- RVQ：`8c8f595a61c63f35a83e7935c087debb188bcdc317753a20dcab43b4d9786e18`；索引 `rvq/training_samples/index.json`；码本 `deps/codebook.npz`。

运行：`cd /cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256 && python reader.py --branch k64 --batch-size 2`。训练适配器为 `training_adapter.TrainingDataset`，需训练端注册原子 token 并映射模型词表。K32/K64 的 0 是 STOP；RVQ 只有完整 `[0,0,0]` 是 STOP，单层 0 为普通码字；使用有效 mask，并只执行 `decoded[0]` 后重新观测。

视频在共享 CPFS 的 `/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/source/runs/output`；历史仅含真实保留节点。主要5/10步FDE见 `site/report_data/K64_WINDOW_COMPARISON.json`；辅助整轨迹诊断和码频见 `site/report_data/JOINT_FULL_METRICS.json`，旧码本误差不可沿用。未验证策略训练或真机效果。
