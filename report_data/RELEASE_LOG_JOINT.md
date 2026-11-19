# 深圳办公室数据发版日志（2026-11-18 联合编码更新）

## 数据与版本

总目录：`/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028`；`START_HERE.txt` 为总入口，`CURRENT/` 指向 `/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256`。原始成功采集 **93,319** 条，质检保留 **80,475** 条、约 **954.0 小时**。

2026-11-18，K32、K64 与 HM3D 和深圳的质检保留轨迹联合拟合，以每条轨迹等权为基础，真实动作 |左向位移| ≥ 2.5 cm 的拟合权重为 5 倍；RVQ 沿用原有逐轨迹等权的联合码本。 K32、K64 标签和码本已替换。每分支 **80,475 条轨迹、5,167,420 个节点、80,475 个 STOP**。原有 K256 保留，共 5,167,420 个宏节点；历史 `v1/v2/v3-refit` 的 RVQ 入口现指向联合 RVQ。

| 分支 | 码本路径 | SHA256 |
|---|---|---|
| K32 | `/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256/k32/codebook.npz` | `589c78646229561178e0fb0251a228dc0e4b2ee68f96bd7f3deee3710d4acac6` |
| K64 | `/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256/k64/codebook.npz` | `b03891724cfab35bc174fab31a20c6e1161df704c0ae65be6961547882f18276` |
| RVQ | `/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256/deps/codebook.npz` | `8c8f595a61c63f35a83e7935c087debb188bcdc317753a20dcab43b4d9786e18` |

K256 保留码本 SHA256：`685a9d717360a3ab6776657dd1920fd10efddcce393a94131f58b90c848b9c9d`。各联合分支索引为 `<branch>/training_samples/index.json` 与 `<branch>/index.json`。先读 `AI_ENTRY.md`、`DATASETS.json`、`TRAINING_CONTRACT.json`，默认 K64。

## 2026-11-19 K64 多样性调整

K64 保留56个码字，将密集直线区8个码字重新拟合到真实左右侧向动作，各侧4个。前向0.2–0.4m密集直线区由18码减为10码，侧向位移至少1cm的码字由10增至18；K64仍为每步1个动作token。K32、RVQ和深圳K256沿用当前版本。直线区平均位置误差由约2.8mm升至约5.2–5.3mm，HM3D和深圳整轨迹FDE均小幅下降，详见编码报告的完整对照。

## 全量开环误差与码频

覆盖全部 **80,475 条轨迹、5,167,420 个节点**，每条轨迹等权。以真实起点开始逐个累积动作，中途不以真实位姿纠正；RVQ 每步使用 decoded[0]。ADE 包含起点，FDE 为终点水平位置误差。属于离线重建，不代表闭环策略或真机效果。

| 分支 | ADE 均值 m | FDE 均值 m | FDE P95 m |
|---|---:|---:|---:|
| K32 | 1.613 | 3.324 | 11.203 |
| K64 | 1.083 | 2.166 | 7.336 |
| RVQ | 0.171 | 0.327 | 1.053 |
| K256 | 0.292 | 0.591 | 1.951 |

侧向覆盖改善与普通节点误差的取舍见编码报告。全量码字使用频率及其他误差统计见 [统计文件](https://yuaneth.github.io/shenzhen-r23-data-report-20261104/report_data/JOINT_FULL_METRICS.json) 和编码报告。

## 训练与资源

读取示例：`cd /cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256 && python reader.py --branch k64 --batch-size 2`。训练适配器为 `training_adapter.TrainingDataset`；需训练端自行注册原子 token 并映射模型词表。K32/K64 的 0 为 STOP；RVQ 仅完整 `[0,0,0]` 为 STOP，单层 0 是普通码字。RVQ 使用有效 mask，只执行 decoded[0] 后重新观测。

视频位于共享 CPFS 的 `/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/source/runs/output`，不是旧 `/data/pool` 路径；跨机器需保证可访问相同路径。地图、原始采集、位姿缓存、索引分别位于总目录 `maps/`、`source/`、`trajectories/`、`encoded/`。标注和 QC_R_SI 的原始说明仍位于 `/cpfs-normal/ziruicheng/shenzhen-r23-20261028/README.md`。

全量编码和安装文件哈希校验通过，训练适配器 0、2 worker 样本读取通过；尚未验证策略训练和真机效果。

[质检报告](https://yuaneth.github.io/shenzhen-r23-data-report-20261104/) · [联合编码报告](https://yuaneth.github.io/shenzhen-r23-data-report-20261104/tokenizer.html) · [训练入口](https://yuaneth.github.io/shenzhen-r23-data-report-20261104/training.html)
