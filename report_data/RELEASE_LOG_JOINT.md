# 深圳办公室数据发版日志（2026-11-20 K64 短窗口更新）

## 数据与版本

总目录：`/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028`；`START_HERE.txt` 为总入口，`CURRENT/` 指向 `/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256`。原始成功采集 **93,319** 条，质检保留 **80,475** 条、约 **954.0 小时**。

2026-11-20，K64 以未来5步和10步重叠窗口的终点FDE为主要目标，联合优化码本与整条离线标签序列；5/10步使用同一条序列，每步仍为1个运动token、64个运动码字加STOP=0，近直行前进距离间隔至少5cm；K32、RVQ和深圳K256沿用当前版本。 K32、K64 标签和码本已替换。每分支 **80,475 条轨迹、5,167,420 个节点、80,475 个 STOP**。原有 K256 保留，共 5,167,420 个宏节点；历史 `v1/v2/v3-refit` 的 RVQ 入口现指向联合 RVQ。

| 分支 | 码本路径 | SHA256 |
|---|---|---|
| K32 | `/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256/k32/codebook.npz` | `0429002c179e4223b21a5faf5fd00ae18326f7cd5b2384be22dea9799cc0c16b` |
| K64 | `/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256/k64/codebook.npz` | `f0cc68c7892d8b43b1c1a9aa5a309db0b9053b6864b3fd73f96aa94d8249950b` |
| RVQ | `/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256/deps/codebook.npz` | `8c8f595a61c63f35a83e7935c087debb188bcdc317753a20dcab43b4d9786e18` |

K256 保留码本 SHA256：`685a9d717360a3ab6776657dd1920fd10efddcce393a94131f58b90c848b9c9d`。各联合分支索引为 `<branch>/training_samples/index.json` 与 `<branch>/index.json`。先读 `AI_ENTRY.md`、`DATASETS.json`、`TRAINING_CONTRACT.json`，默认 K64。

## K32 版本说明

K32 保持2026-11-19版本：13个近直行码字、最小前进间距5cm，仍为32个运动码字加STOP。K64见本次5/10步FDE更新。

## 2026-11-20 K64 未来5/10步FDE更新

K64 以未来5步和10步重叠窗口的终点FDE为主要目标，联合优化码本与整条离线标签序列；5/10步使用同一条序列，每步仍为1个运动token、64个运动码字加STOP=0，近直行前进距离间隔至少5cm；K32、RVQ和深圳K256沿用当前版本。

以下覆盖全部有效重叠窗口；每个窗口仅在真实起点对齐，中间不做GT位姿纠正。步是保留节点之间的运动边，不是原始视频帧或秒数。

| 窗口 | 全量窗口数 | FDE均值 cm：前→后 | P95 cm：前→后 |
|---|---:|---:|---:|
| 5步 | 4,765,045 | 4.54 → 2.79 | 12.20 → 6.86 |
| 10步 | 4,362,880 | 10.54 → 4.30 | 31.34 → 12.29 |

K64标签由整条离线序列联合搜索生成，不能用单步最近码字重新生成相同标签。离线编码器和参数在 k64/encoder/；解码仍使用原有三维局部动作。真实未来路径上的编码重建误差不等于模型预测或机器人执行误差。

## 全量开环误差与码频

覆盖全部 **80,475 条轨迹、5,167,420 个节点**，每条轨迹等权。以真实起点开始逐个累积动作，中途不以真实位姿纠正；RVQ 每步使用 decoded[0]。ADE 包含起点，FDE 为终点水平位置误差。属于离线重建，不代表闭环策略或真机效果。

| 分支 | ADE 均值 m | FDE 均值 m | FDE P95 m |
|---|---:|---:|---:|
| K32 | 1.646 | 3.400 | 11.404 |
| K64 | 0.301 | 0.607 | 2.003 |
| RVQ | 0.171 | 0.327 | 1.053 |
| K256 | 0.292 | 0.591 | 1.951 |

侧向覆盖改善与普通节点误差的取舍见编码报告。全量码字使用频率及其他误差统计见 [统计文件](https://yuaneth.github.io/shenzhen-r23-data-report-20261104/report_data/JOINT_FULL_METRICS.json) 和编码报告。

## 训练与资源

读取示例：`cd /cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256 && python reader.py --branch k64 --batch-size 2`。训练适配器为 `training_adapter.TrainingDataset`；需训练端自行注册原子 token 并映射模型词表。K32/K64 的 0 为 STOP；RVQ 仅完整 `[0,0,0]` 为 STOP，单层 0 是普通码字。RVQ 使用有效 mask，只执行 decoded[0] 后重新观测。

视频位于共享 CPFS 的 `/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/source/runs/output`，不是旧 `/data/pool` 路径；跨机器需保证可访问相同路径。地图、原始采集、位姿缓存、索引分别位于总目录 `maps/`、`source/`、`trajectories/`、`encoded/`。标注和 QC_R_SI 的原始说明仍位于 `/cpfs-normal/ziruicheng/shenzhen-r23-20261028/README.md`。

全量编码和安装文件哈希校验通过，训练适配器 0、2 worker 样本读取通过；尚未验证策略训练和真机效果。

[质检报告](https://yuaneth.github.io/shenzhen-r23-data-report-20261104/) · [联合编码报告](https://yuaneth.github.io/shenzhen-r23-data-report-20261104/tokenizer.html) · [训练入口](https://yuaneth.github.io/shenzhen-r23-data-report-20261104/training.html)
