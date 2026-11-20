# 深圳 R23 训练入口（2026-11-20 K32/K64/RVQ 短窗口FDE版）

根目录：`/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256`；总目录的 `CURRENT/` 指向当前版本。先读 `DATASETS.json` 和 `TRAINING_CONTRACT.json`。默认 `k64`，可选 `k32`、`rvq` 和原有 `k256`。

深圳与HM3D的质检保留轨迹共用码本，K32/K64/RVQ均为未来5/10步FDE版。 K256 保持深圳既有码本。三种联合分支各 80,475 条轨迹、5,167,420 个节点。

深圳与HM3D共用联合码本。K32、K64和RVQ均以未来5步、10步执行轨迹的终点FDE为主要目标，联合拟合码本和离线标签序列，两个步长使用同一条序列；整轨迹FDE和每步ADE只作辅助诊断。K32/K64每步1个运动token，分别32/64个运动码字加STOP=0，近直行前进距离间隔至少5cm。RVQ每步仍为3个残差token、每层256个选项，保持未来5节点的加和解码与小动作检查。本次更新K32和RVQ，K64沿用已完成的窗口FDE版。

K64 编码约定（2026-11-20）：标签使用真实未来路径，联合搜索同一条码字序列来减小5步和10步FDE，不能用单步最近码字复现当前标签。每步1个运动token、64个运动码字及STOP=0，解码仍为码字的局部[前向、左向、偏航]。离线编码器、源码和参数在 k64/encoder/。

K32/RVQ 编码约定（2026-11-20）：直接使用当前码本和GT，确定性联合搜索同一条序列以减小未来5步、10步FDE；无需历史标签或码本。K32每步1个运动token，RVQ每步3个残差token。完整离线源码和参数分别在 k32/encoder/ 与 rvq/encoder/；解码接口保持一致。

- K32：`6f94bedcd6e1330e796c3cd71ae9d1d18ffb4156e41ef01ef665ddf1938816b2`；索引 `k32/training_samples/index.json`；码本 `k32/codebook.npz`。
- K64：`f0cc68c7892d8b43b1c1a9aa5a309db0b9053b6864b3fd73f96aa94d8249950b`；索引 `k64/training_samples/index.json`；码本 `k64/codebook.npz`。
- RVQ：`0966c2ba8d04e8fd868a5df92366c2693bf9f1ff1226e74d65c07ad5a562eeee`；索引 `rvq/training_samples/index.json`；码本 `deps/codebook.npz`。

运行：`cd /cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256 && python reader.py --branch k64 --batch-size 2`。训练适配器为 `training_adapter.TrainingDataset`，需训练端注册原子 token 并映射模型词表。K32/K64 的 0 是 STOP；RVQ 只有完整 `[0,0,0]` 是 STOP，单层 0 为普通码字；使用有效 mask，并只执行 `decoded[0]` 后重新观测。

视频在共享 CPFS 的 `/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/source/runs/output`；历史仅含真实保留节点。主要5/10步FDE见 `site/report_data/K32_WINDOW_COMPARISON.json`、`RVQ_WINDOW_COMPARISON.json`、`K64_WINDOW_COMPARISON.json`；辅助整轨迹诊断和码频见 `site/report_data/JOINT_FULL_METRICS.json`，旧码本误差不可沿用。未验证策略训练或真机效果。
