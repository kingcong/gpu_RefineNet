# 目录

<!-- TOC -->

- [目录](#目录)
- [RefineNet描述](#refinenet描述)
    - [描述](#描述)
- [模型架构](#模型架构)
- [数据集](#数据集)
- [特性](#特性)
    - [混合精度](#混合精度)
- [环境要求](#环境要求)
- [快速入门](#快速入门)
- [脚本说明](#脚本说明)
    - [脚本及样例代码](#脚本及样例代码)
    - [脚本参数](#脚本参数)
    - [训练过程](#训练过程)
        - [用法](#用法)
           
            - [GPU处理器环境运行](#gpu处理器环境运行)
        - [结果](#结果)
    - [评估过程](#评估过程)
        - [用法](#用法-1)
            - [处理器环境运行](#处理器环境运行-1)
            - [GPU处理器环境运行](#gpu处理器环境运行-1)
        - [结果](#结果-1)
            - [训练准确率](#训练准确率)
    - [Mindir推理](#Mindir推理)
        - [导出模型](#导出模型)
        - [在310执行推理](#在310执行推理)
        - [结果](#结果)
- [模型描述](#模型描述)
    - [性能](#性能)
        - [评估性能](#评估性能)
- [随机情况说明](#随机情况说明)
- [ModelZoo主页](#modelzoo主页)

<!-- /TOC -->

# RefineNet描述

## 描述

RefineNet是一种通用的多径优化网络，它显式地利用下采样过程中的所有可用信息，利用长程残差连接实现高分辨率预测。通过这种方式，捕获高级语义特征的深层可以使用来自浅层卷积的细粒度特征直接细化。RefineNet的各个组件按照认证映射思想使用残差连接，这允许进行有效的端到端训练。

有关网络详细信息，请参阅[论文][1]
`guosheng.lin，anton.milan，et.al.RefineNet: Multi-Path Refinement Networks for High-Resolution Semantic Segmentation.arXiv:1611.06612v3 [cs.CV] 25 Nov 2016`

[1]: https://arxiv.org/abs/1611.06612v3

# 模型架构

以ResNet-101为骨干，利用不同阶段的多种层次的卷积信息，并将他们融合到一起来获取一个高分辨率的预测,具体请见[链接][2]。

[2]: https://arxiv.org/pdf/1611.06612v3.pdf

# 数据集

Pascal VOC数据集和语义边界数据集（Semantic Boundaries Dataset，SBD）

- 下载分段数据集。  
- 准备Backbone模型。

Pascal VOC数据集官网：[链接](https://host.robots.ox.ac.uk/pascal/VOC/)  
SBD数据集下载地址：[链接](https://www2.eecs.berkeley.edu/Research/Projects/CS/vision/grouping/semantic_contours/benchmark.tgz)  
ResNet101预训练模型下载地址：[链接](https://download.mindspore.cn/model_zoo/r1.2/resnet101__v120_imagenet2012_official_cv_bs32_acc78/resnet101__v120_imagenet2012_official_cv_bs32_acc78.ckpt)  
下载数据集后解压分别得到如下所示目录：

```text
~/data/                            数据集存放根目录
~/data/VOCdevkit/                  Pascal VOC数据集目录
~/data/benchmark_RELEASE/          SBD边界数据集目录
```

- 准备训练数据清单文件。清单文件用于保存图片和标注对的相对路径。如下：

```text
VOCdevkit/VOC2012/JPEGImages/2007_000032.jpg VOCdevkit/VOC2012/SegmentationClassGray/2007_000032.png
VOCdevkit/VOC2012/JPEGImages/2007_000039.jpg VOCdevkit/VOC2012/SegmentationClassGray/2007_000039.png
VOCdevkit/VOC2012/JPEGImages/2007_000063.jpg VOCdevkit/VOC2012/SegmentationClassGray/2007_000063.png
VOCdevkit/VOC2012/JPEGImages/2007_000068.jpg VOCdevkit/VOC2012/SegmentationClassGray/2007_000068.png
......
```

通过运行脚本：`python ~/src/tool/get_dataset_lst.py --data_dir=~/data/` 来自动生成数据清单文件：

```text
voc_train_lst.txtfinetune数据集目录
voc_val_lst.txt验证数据集目录
sbd_train_lst.txt预训练数据集目录
```

- 配置并运行build _MRcd.py，将数据集转换为MindRecords:

运行配置以下参数命令，使用生成的'sbd_train_lst.txt'，生成预训练数据集：

```bash
# build_MRcd.py
Usage: python ~src/tool/build_MRcd.py --data_root=~/data/ --data_lst=~/sbd_train_lst.txt --dst_path=~/data/sbdonly
```

运行配置以下参数命令，使用生成的'voc_train_lst.txt'，生成finetune数据集：

```bash
# build_MRcd.py
Usage: python ~src/tool/build_MRcd.py --data_root=~/data/ --data_lst=~/voc_train_lst.txt --dst_path=~/data/voconly
```

```text
--data_root                 训练数据的根路径（~/data）
--data_lst                  训练数据列表（~/data/sbd_train_lst.txt）
--dst_path                  MindRecord存放路径与MindRecord文件名（eg:~/data/为文件路径，sbdonly为文件名）
--num_shards                MindRecord的分片数（默认为8）
```

# 特性

## 混合精度

采用混合精度的训练方法使用支持单精度和半精度数据来提高深度学习神经网络的训练速度，同时保持单精度训练所能达到的网络精度。混合精度训练提高计算速度、减少内存使用的同时，支持在特定硬件上训练更大的模型或实现更大批次的训练。

# 环境要求

- 硬件（GPU）
    - 准备处理器搭建硬件环境。
- 框架
    - [MindSpore](https://www.mindspore.cn/install)
- 如需查看详情，请参见如下资源：
    - [MindSpore教程](https://www.mindspore.cn/tutorials/zh-CN/master/index.html)
    - [MindSpore Python API](https://www.mindspore.cn/docs/api/zh-CN/master/index.html)
- 安装requirements.txt中的python包。
- 生成config json文件用于8卡训练。

# 快速入门

通过官方网站安装MindSpore后，您可以按照如下步骤进行训练和评估：

- 处理器环境运行

在RefineNet原始论文的基础上，我们对去除与VOC数据集重复部分的边界数据集SBD数据集进行了一次训练实验，再对剩余VOC数据集进行finetune，并对voc_val数据集进行了评估。

运行以下训练脚本配置单卡训练参数：

```bash
run_standalone_train_.sh
```

运行以下训练脚本配置8卡训练参数,微调ResNet_101模型：

```bash
run_distribute_train__r1.sh
```

运行以下训练脚本配置8卡训练参数,微调上一步骤模型：

```bash
run_distribute_train__r2.sh
```

评估步骤如下：

1.使用voc val数据集评估。评估脚本如下：

```bash
run_eval.sh
```

# 脚本说明

## 脚本及样例代码

```shell
.
└──refinenet
  ├── script
    ├── run_standalone_train__r1.sh              # 启动单机预训练（单卡）
    ├── run_standalone_train__r2.sh              # 启动单机finetune（单卡）
    ├── run_distribute_train__r1.sh       # 启动分布式预训练（八卡）
    ├── run_distribute_train__r2.sh       # 启动分布式finetune（八卡）
    ├── run_eval.sh                             # 启动评估
    ├── run_standalone_train_gpu_r1.sh          # 启动GPU单机预训练（单卡）
    ├── run_standalone_train_gpu_r2.sh          # 启动GPU单机finetune（单卡）
    ├── run_distribute_train_gpu_r1.sh          # 启动GPU分布式预训练（八卡）
    ├── run_distribute_train_gpu_r2.sh          # 启动GPU分布式finetune（八卡）
  ├── src
    ├── tools
      ├── get_dataset_lst.py                    # 获取数据清单文件
      ├── build_MRcd.py                         # 获取MindRecord文件
    ├── dataset.py                              # 数据预处理
    ├── refinenet.py                            # RefineNet网络结构
    ├── learning_rates.py                       # 生成学习率
    ├── loss.py                                 # RefineNet的损失定义
  ├── eval.py                                   # 训练时评估网络
  ├── train.py                                  # 训练网络
  ├── requirements.txt                          # requirements文件
  └── README.md
```

## 脚本参数


```

GPU处理器环境默认配置

```bash
"data_file":"~/data/"                             # 数据集路径
"device_target":GPU                               # 训练后端类型
"train_epochs":200                                # 总轮次数
"batch_size":16                                   # 输入张量的批次大小
"crop_size":513                                   # 裁剪大小
"base_lr":0.001                                   # 基础学习率
"lr_type":cos                                     # 用于生成学习率的衰减模式
"min_scale":0.5                                   # 数据增强的最小尺度
"max_scale":2.0                                   # 数据增强的最大尺度
"ignore_label":255                                # 忽略标签
"num_classes":21                                  # 类别数
"ckpt_pre_trained":"/PATH/TO/PRETRAIN_MODEL"      # 加载预训练检查点的路径
"is_distributed":                                 # 分布式训练，设置该参数为True
"save_epochs":5                                   # 用于保存的迭代间隙
"freeze_bn":                                      # 设置该参数freeze_bn为True
"keep_checkpoint_max":200                         # 用于保存的最大检查点
```

## 训练过程

### 用法


```

#### GPU处理器环境运行

参考处理器环境运行方式，使用GPU脚本。同样，首先准备ResNet_101预训练模型：resnet-101.ckpt。

运行以下训练脚本配置单卡训练参数，微调ResNet_101模型：

```bash
# run_standalone_train.sh
Usage: bash scripts/run_standalone_train_gpu_r1.sh [DATASET_PATH] [PRETRAINED_PATH] [DEVICE_ID]
#example: bash scripts/run_standalone_train_gpu_r1.sh ~/data/sbdonly0 /data1/fyf/resnet-101.ckpt 0
```

运行以下训练脚本配置单卡训练参数，微调上一步模型：

```bash
# run_distribute_train.sh
Usage: bash scripts/run_standalone_train_gpu_r2.sh [DATASET_PATH] [PRETRAINED_PATH] [DEVICE_ID]
#example: bash scripts/run_standalone_train_gpu_r2.sh ~/data/voconly0 /data1/fyf/RefineNet/scripts/train2/ckpt_0/refinenet-130_569.ckpt 0
```

运行以下训练脚本配置八卡训练参数，微调ResNet_101模型(注意gpu多卡训练不需要配置rank_table_ip）：

```bash
# run_distribute_train.sh
Usage: bash scripts/run_distribute_train_gpu_r1.sh [DATASET_PATH] [PRETRAINED_PATH] [VISIABLE_DEVICES(0,1,2,3,4,5,6,7)]
#example: bash scripts/run_distribute_train_gpu_r1.sh ~/data/sbdonly0 /data1/fyf/resnet-101.ckpt 0,1,2,3,4,5,6,7
```

运行以下训练脚本配置八卡训练参数，微调上一步模型：

```bash
# run_distribute_train.sh
Usage: bash scripts/run_distribute_train_gpu_r2.sh [DATASET_PATH] [PRETRAINED_PATH] [VISIABLE_DEVICES(0,1,2,3,4,5,6,7)]
#example: bash scripts/run_distribute_train_gpu_r2.sh ~/data/voconly0 /data1/fyf/refinenet-115_1140.ckpt 0,1,2,3,4,5,6,7
```

### 结果


```

GPU处理器环境结果

```bash
# 分布式训练结果（1P）
epoch: 195 step: 569, loss is 0.05817811
epoch time: 410643.678 ms, per step time: 721.694 ms
epoch: 196 step: 569, loss is 0.07650596
epoch time: 409365.036 ms, per step time: 719.446 ms
epoch: 197 step: 569, loss is 0.07034514
epoch time: 409448.961 ms, per step time: 719.594 ms
epoch: 198 step: 569, loss is 0.07419827
epoch time: 409355.774 ms, per step time: 719.430 ms
epoch: 199 step: 569, loss is 0.07571901
epoch time: 409360.690 ms, per step time: 719.439 ms
epoch: 200 step: 569, loss is 0.08345377
epoch time: 410627.769 ms, per step time: 721.666 ms
...
```

## 评估过程

### 用法


```

#### GPU处理器环境运行

使用--ckpt_path配置检查点，运行脚本，在eval_path/log中打印mIOU。

```bash
# run_eval.sh                     # 测试训练结果
Usage: bash scripts/run_eval.sh [DATA_LST] [PRETRAINED_PATH] [DEVICE_TARGET] [DEVICE_TARGET] [DEVICE_ID]
#example: bash scripts/run_eval.sh ~/data/voc_val_lst.txt /data1/fyf/refinenet-115_1140.ckpt GPU 0

per-class IoU [0.95088336 0.90526754 0.62389328 0.90752526 0.77911041 0.79076594
 0.94210807  0.88425516 0.93747317 0.41626388 0.84932021 0.63371361
 0.89109052 0.85608585 0.8491058  0.86728246 0.6983279 0.88386951
 0.47583356 0.8800718 0.78794471]
mean IoU 0.8004853336726656

```

### 结果

运行适用的训练脚本获取结果。要获得相同的结果，请按照快速入门中的步骤操作。

#### 训练准确率

| **网络** | mIOU |论文中的mIOU |
| :----------: | :-----: | :-------------: |
| refinenet | 80.3 | 80.3    |

## Mindir推理

### [导出模型](#contents)

```shell
python export.py --checkpoint [CKPT_PATH] --file_name [FILE_NAME] --file_format [FILE_FORMAT]
```

- 参数`checkpoint`为必填项。
- `file_format` 必须在 ["AIR", "MINDIR"]中选择。

### 在310执行推理

在执行推理前，mindir文件必须通过`export.py`脚本导出。以下展示了使用mindir模型执行推理的示例。
目前仅支持batch_size为1的推理。

```shell
# 310 inference
bash scripts/run_infer_310.sh [MINDIR_PATH] [DATA_ROOT] [DATA_LIST] [DEVICE_ID]
```

- `DATA_ROOT` 表示进入模型推理数据集的根目录。
- `DATA_LIST` 表示进入模型推理数据集的文件列表。
- `DEVICE_ID` 可选，默认值为0。

### 结果

推理结果保存在脚本执行的当前路径，你可以在acc.log中看到以下精度计算结果。

# 模型描述

## 性能

### 评估性能

| 参数 |GPU |
| -------------------------- | -----------|
| 模型版本 | RefineNet |
| 资源 | GForce RTX 3090 |
| 上传日期 | 2022-02-16 |
| MindSpore版本 | 1.2 |
| 数据集 | PASCAL VOC2012 + SBD |
| 训练参数 | epoch=200,batch_size=16 |
| 优化器 |  Momentum |
| 损失函数 |  Softmax交叉熵 |
| 输出 |概率 |
| 损失 | 0.08345377 |
| 性能 |  723.160 ms（GPU单卡）|
| 微调检查点 |  900M（.ckpt文件）|
| 脚本 |  [链接](https://gitee.com/mindspore/models/tree/master/research/cv/RefineNet)

# 随机情况说明

dataset.py中设置了“create_dataset”函数内的种子，同时还使用了train.py中的随机种子。

# ModelZoo主页

 请浏览官网[主页](https://gitee.com/mindspore/models/)。
