# AI_ENTRY

从 DATASETS.json 取绝对路径，先读 TRAINING_CONTRACT.json 和 QC_CONFIG.json。两分支使用同一 QC 冻结集和同一真实宏节点，动作表示不同；RVQ 再排除未解决整轨迹。

- K256：k256/parts/*.jsonl.gz；与RVQ相同的真实宏节点，motion1..256、STOP0，uint16；码本 k256/codebook.npz；编码/解码 k256.py。
- Pointing+Object+RVQ-H5：rvq/parts/*.jsonl.gz；连续真实宏节点，五点绝对局部位姿；本批重新拟合码本 deps/codebook.npz；完整000停止。
- 默认读取：reader.samples 或 reader.NavigationDataset。历史严格来自 frame_indices 切片；不能补回原始中间帧。
- 运行示例：python /cpfs-normal/xuyuan/data-workspace/shenzhen-r23-qc-k256-rvq-h5-20261104-v4-macro-k256/reader.py --branch rvq --split train --batch-size 4 --images
- 统一续跑：python /cpfs-normal/xuyuan/data-workspace/shenzhen-r23-qc-k256-rvq-h5-20261104-v4-macro-k256/run_pipeline.py --from-stage k256 --workers 24
- 查看本地 report.html、中文报告.md、COUNTS.tsv、VERIFICATION.json、reader_validation.json。

不要把源码词表 ID 当目标模型 ID；用实际 tokenizer 按字符串注册和映射。部署只执行 decoded[0]，重新观测后再决策。禁止把记录时间当控制时长。

本次源数据仅有 train，没有 eval/probe；离线指标属于训练数据重建误差。这些产物验证的是数据/读取与离线量化误差，没有策略训练或真机性能结论。所有原数据只读引用，过滤体现在实体分片和默认索引中。

训练适配器：training_adapter.TrainingDataset，使用 required_tokens(branch) 注册实际模型词表并调整 embedding；尚需将 Dataset/collator 接到实际训练器。0/2 worker 真实图片 batch 检查见 TRAINING_ADAPTER_VALIDATION.json。
保留集的 80,475 个视频已复制并逐文件 SHA256 核验至共享 CPFS：/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-videos-20261105。两分支分片中的 source_video、source_root 及训练索引已更新；挂载同一 CPFS 路径的节点可直接读取，无需访问原主机 /data/pool。共享副本只包含本次保留集；本地原件保留供旧版本和离线重建使用。迁移与验证见 VIDEO_STORAGE.json、VIDEO_STORAGE_VALIDATION.json，视频清单见共享目录 VIDEO_MANIFEST.jsonl。K256 和 RVQ 均为 5,167,420 个监督节点；6,868,697 是保留轨迹原始帧数。config.json 的 source_root/scratch_root 仍用于本机离线重建；跨节点训练使用 DATASETS.json 和分支训练索引，不使用这些重建缓存路径。
K256本版按宏动作重新拟合；RVQ复用v3基于同一80,475条数据拟合的码本，绑定 DATASETS.json 中各自 SHA256；旧标签不能与新码本混用。拟合参数与轨迹清单见 k256/CODEBOOK.json、rvq/CODEBOOK.json 和对应 fit_trajectories.json。
