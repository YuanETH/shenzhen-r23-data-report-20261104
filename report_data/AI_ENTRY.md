# AI_ENTRY

从 DATASETS.json 取绝对路径，先读 TRAINING_CONTRACT.json 和 QC_CONFIG.json。两分支是同一 QC 冻结集的不同监督粒度；RVQ 再排除未解决整轨迹。

- K256：k256/parts/*.jsonl.gz；原始相邻真实帧，motion1..256、STOP0，uint16；码本 k256/codebook.npz；编码/解码 k256.py。
- Pointing+Object+RVQ-H5：rvq/parts/*.jsonl.gz；连续真实宏节点，五点绝对局部位姿；固定码本 deps/codebook.npz；完整000停止。
- 默认读取：reader.samples 或 reader.NavigationDataset。历史严格来自 frame_indices 切片；不能补回原始中间帧。
- 运行示例：python /cpfs-normal/xuyuan/data-workspace/shenzhen-r23-qc-k256-rvq-h5-20261104-v2/reader.py --branch rvq --split train --batch-size 4 --images
- 统一续跑：python /cpfs-normal/xuyuan/data-workspace/shenzhen-r23-qc-k256-rvq-h5-20261104-v2/run_pipeline.py --from-stage correction --workers 24
- 查看本地 report.html、中文报告.md、COUNTS.tsv、VERIFICATION.json、reader_validation.json。

不要把源码词表 ID 当目标模型 ID；用实际 tokenizer 按字符串注册和映射。部署只执行 decoded[0]，重新观测后再决策。禁止把记录时间当控制时长。

本次源数据仅有 train，没有 eval/probe；离线指标属于训练数据重建误差。这些产物验证的是数据/读取与离线量化误差，没有策略训练或真机性能结论。所有原数据只读引用，过滤体现在实体分片和默认索引中。

公开报告三页位于 site/index.html、site/tokenizer.html、site/training.html；实际公开地址与验收状态以 PUBLICATION_STATE.json、PUBLIC_REPORT_VALIDATION.json 为准。
训练适配器为 training_adapter.TrainingDataset；配置和0/2worker真实图片batch检查见 TRAINING_ADAPTER_CONFIG.json、TRAINING_ADAPTER_VALIDATION.json。

源视频位于本次工作主机 /data/pool 的本地XFS盘（独立恢复源）；标签和码本位于CPFS。跨主机使用需同步源视频并更新清单/分片source_video路径；只复制标签目录不能读取图像。当前主机已实际通过完整标签与真实batch检查。
