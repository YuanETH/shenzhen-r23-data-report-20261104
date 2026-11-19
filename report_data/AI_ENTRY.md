# 深圳 R23 训练入口（2026-11-18 联合码本）

根目录：`/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256`；总目录的 `CURRENT/` 指向当前版本。先读 `DATASETS.json` 和 `TRAINING_CONTRACT.json`。默认 `k64`，可选 `k32`、`rvq` 和原有 `k256`。

K32/K64 基于深圳与 HM3D 的质检保留轨迹联合拟合，每条轨迹等权基础上将 |左向位移|≥2.5 cm 的真实动作权重提高到 5 倍；RVQ 保持原来的逐轨迹等权联合码本。 K256 保持深圳既有码本。三种联合分支各 80,475 条轨迹、5,167,420 个节点。

K64 于2026-11-19增加多样性：保留56个码字，将前向0.2–0.4m密集直线段的8个码字移到真实左右侧向动作，左右各4个；密集区18码减至10码，侧向至少1cm的码字由10增至18。K64仍为每步1个动作token，K32和RVQ沿用当前版本。

- K32：`589c78646229561178e0fb0251a228dc0e4b2ee68f96bd7f3deee3710d4acac6`；索引 `k32/training_samples/index.json`；码本 `k32/codebook.npz`。
- K64：`b03891724cfab35bc174fab31a20c6e1161df704c0ae65be6961547882f18276`；索引 `k64/training_samples/index.json`；码本 `k64/codebook.npz`。
- RVQ：`8c8f595a61c63f35a83e7935c087debb188bcdc317753a20dcab43b4d9786e18`；索引 `rvq/training_samples/index.json`；码本 `deps/codebook.npz`。

运行：`cd /cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256 && python reader.py --branch k64 --batch-size 2`。训练适配器为 `training_adapter.TrainingDataset`，需训练端注册原子 token 并映射模型词表。K32/K64 的 0 是 STOP；RVQ 只有完整 `[0,0,0]` 是 STOP，单层 0 为普通码字；使用有效 mask，并只执行 `decoded[0]` 后重新观测。

视频在共享 CPFS 的 `/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/source/runs/output`；历史仅含真实保留节点。新码本全量开环误差和码频见 `site/report_data/JOINT_FULL_METRICS.json`，旧码本误差不可沿用。未验证策略训练或真机效果。
