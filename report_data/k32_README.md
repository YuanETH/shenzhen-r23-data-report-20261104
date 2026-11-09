# K32 可选训练分支

K32 与 K64、K256、RVQ 同级。默认训练仍为 K64；显式选择 branch='k32'。
根目录：/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256/k32
监督索引：/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256/k32/training_samples/index.json
码本：/cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256/k32/codebook.npz
SHA256：2cd130dc5ae806c3334e5780fb4d529c5cacb6aa08cffc403321cd5b5e36d66a

使用质检保留80,475条train轨迹、5,167,420个真实宏节点（含80,475个STOP）。在5,086,945个已经合并细碎动作的真实运动动作上从零拟合32个码字，每条轨迹总权重1；STOP不参与拟合，没有eval/probe。没有重新引入原始中间帧或增加过滤。
运动1..32，STOP0；codec_ids为uint16；词表是33个原子字符串<k32_0>至<k32_32>。按字符串注册实际模型词表并调整embedding，不能混用K64/K256词表或码本。
图像和历史只取保留节点；原始首末帧不变。每次执行一个解码宏目标后重新观测，记录时长不是固定控制时长。细碎动作定义为平移≤10cm且绝对转角≤10°（STOP除外），GT和全部32个解码中心均无细碎残留。
视频和位姿只读引用统一CPFS目录；没有重复复制视频。接训练器仍需显式导入Dataset、注册tokenizer并连接processor/collator，没有自动注册外部训练器。

```bash
cd /cpfs-normal/xuyuan/data-workspace/shenzhen-r23-20261028/encoded/v4-macro-k256
python reader.py --branch k32 --batch-size 4 --images
```
```python
import numpy as np
import k32
from training_adapter import TrainingDataset, required_tokens
tokens = required_tokens('k32')  # 33个原子token，注册到实际tokenizer
row = next(iter(TrainingDataset(branch='k32', load_images=True)))
action = k32.decode(row['codec_ids'], k32.load_book())  # (1,3)，前向米/左向米/逆时针弧度
```
验证：K32_VALIDATION.json、k32_reader_sequence_validation.json及TRAINING_ADAPTER_VALIDATION.json。
报告：tokenizer.html#k32；训练入口：training.html#k32-training。
统计属于训练数据离线重建误差，没有策略训练或真机效果验证。
