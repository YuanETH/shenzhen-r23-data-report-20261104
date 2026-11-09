# 深圳 R23 训练入口

默认使用 K64。先读 DATASETS.json、TRAINING_CONTRACT.json 和 k64/README.md。
K64 与 K256、RVQ-H5 平行存放，均使用质检保留的80,475条轨迹、同一组已去除细碎动作的真实观测节点，共5,167,420个监督样本。没有增加过滤或恢复被跳过的原始帧。

- 默认标签：k64/parts/*.jsonl.gz；索引 k64/training_samples/index.json；码本 k64/codebook.npz；编码/解码 k64.py。
- 运动ID 1..64，原始真实终点 STOP=0；codec_ids保持uint16；模型词表是65个独立字符串 <k64_0> 至 <k64_64>，必须按字符串注册并映射，不能沿用K256模型ID。
- reader.samples()、reader.NavigationDataset()、training_adapter.TrainingDataset()默认均为k64；显式branch='k256'或'rvq'可读取保留的对照分支。
- 图像、历史和动作只来自frame_indices，不能补回被合并的原始帧；时间戳间隔可变，不能当固定控制时长。K64执行一个解码宏目标后重新观测；RVQ只执行decoded[0]。
- 视频、地图和位姿均在统一CPFS目录内；视频通过source_video引用，没有在k64目录重复复制。挂载相同CPFS路径即可读取。
- source数据只读保留；原始成功93,319条，QC过滤12,844条，合并跳过1,701,277帧属于采样变化，并非删除轨迹。只有train，没有eval/probe；报告误差为训练数据离线重建误差。
- 训练器需显式导入Dataset、注册词表并resize模型embedding，再接模型processor/collator；没有声称已修改任何外部训练器。

实际读取命令：
```bash
python /cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256/reader.py --batch-size 4 --images
python /cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256/run_k64_release.py --from-stage fit
```

拟合参数、SHA256、全量校验及真实图像batch验证分别见k64/CODEBOOK.json、K64_VALIDATION.json、TRAINING_ADAPTER_VALIDATION.json和k64_reader_sequence_validation.json。
编码报告：公开tokenizer.html#k64。旧K256/RVQ标签和码本保持原样，不能跨分支混用。

<!-- K32 OPTIONAL -->
新增可选 K32：/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256/k32/README.md。显式 branch='k32'；码本k32/codebook.npz，运动1..32、STOP0，33个模型词表token。默认继续K64，节点/历史/终点与其它分支相同。
