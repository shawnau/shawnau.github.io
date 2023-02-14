# Mask-RCNN


存货, 介绍了Mask-RCNN和具体的实现细节

<!--more-->

## RPN
### 网络结构
Mask RCNN的RPN是multi head的, 也就是每一个FPN的特征图对应一个RPN head, 最后把输出concatenate到一块.

对于单个RPN Head, 输入为四张FPN的特征图之一: 
`Batch, Input_channel=4, Height=D, Width=D`

|layer|parameter|output|
|-|-|-|
|Conv1|C=512, K=3,S=1,P=1|B, 512, D, D|
|Conv_cls|C=$(2\times s \times r)$, K=1, S=1|B, 2k, D, D|
|Conv_reg|C=$(4\times s \times r)$, K=1, S=1|B, 4k, D, D|

其中$s$代表anchor尺寸, 由于FPN输出4个通道, 恰好对应了4种scale, 因此这里天然用4是实现时的方案. $r$代表anchor长宽比,  $s \times r$个anchor是由$s$种尺寸, $r$种长宽比组合而成的.

1. `conv1`是一个3x3的conv+relu+dropout层, 输出被cls和reg两个分支接收. 早期论文使用fc, 现在已经全卷积化, 更轻更快. 而且更准确(保留了空间信息)
2. `Conv_cls`, 全卷积网络(没有激活函数), 输出在DxD个位置中, 每个位置上$s \times r$个anchor的前景/背景的概率, 因此在每个位置上都有$(2\times s \times r)$个score.如果前景/背景概率和为1, 则只需要一维变量即可
3. `Conv_reg`, 全卷积网络(没有激活函数), 输出在DxD个位置中, 每个位置上$s \times r$个anchor的坐标修正, 因此在每个位置上都有$(4\times s \times r)$个score

### 正负样本与损失函数
正负样本定义与batch内的重采样

1. IOU > 0.7被标记为正样本, 或者是某一个instance中iou最高的. 后者是为了防止有些实例没有对应的proposal分配
3. 由于负样本占绝大部分, 需要按照正负样本比例bootstrapping至1:1

损失函数
1. cls head只需要分类前景和背景, 因此就是log loss. (改进: focal loss)
2. reg head是一个回归模型, 只计算正样本的损失. 使用smooth L1 loss (改进: weighted smooth L1, 或者IOU loss)


## RoiAlign
RoiAlign被用于根据RPN proposal或者RCNN proposal从特征图上切割固定尺寸的特征图给RCNN或者Mask Head

![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/roi_align_1.png)
![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/roi_align_2.png)

两张图解释一下ROI Align与ROI Pooling的区别.
1. 使用ROI Pooling, proposal在feature map上的坐标大概率不是整数(尤其是有Bbox Regression), pooling的时候需要取整, 这个时候引入了不可逆的misalignment, 损失了精确的位置信息. 对要求精确到像素的实例分割是不可忍受的
2. 使用ROI Align, 则pooling的时候不取整, 在每个block中取四个锚点(**可以用更多锚点吗? 可以根据FPN的尺寸取不同数目的锚点吗?**)

备注

1. 代码中RCNN和Mask Head接收的尺寸都是14x14
2. 与FPN合用的时候, ROI Align需要根据proposal的大小来判断从FPN的哪一个feature map上pool

## RCNN
### 网络结构
|layer|parameter|output|
|-|-|-|
|fc1|`(in_channel*14*14)` -> `(1024)`|`(1024)`|
|fc2|`(1024)` -> `(1024)`|`(1024)`|
|cls|`(1024)` -> `(num_classes)`|`(num_classes)`|
|reg|`(1024)` -> `(4*num_classes)`|`(4*num_classes)`|

输入是RoiAlign提供的$(B, C, 14, 14)$
1. 先拉平, 通过两个FC+relu和一个dropout层, 输出一个特征向量. 这里不涉及空间属性, 因此不需要使用全卷积结构
2. 两个FC分支分别接收同一个特征向量, cls分支输出属于某一类的score, reg分支输出delta

### 正负样本与损失函数
1. 正样本IOU>0.7 负样本IOU < 0.3
2. 需要将正负样本比例bootstrapping到1:1

损失函数

1. cls使用交叉熵
2. reg使用smooth L1 loss

## Mask Head
1. 4个CBR全卷积层, 其中有一个是步长2的反卷积层, 最后一层是1x1卷积, 用于转换输出维度到80层(对应了80类), 最终输出的mask是(80, 28, 28)的, 
2. 拉平之后用交叉熵损失函数(改进: dice loss for 人脸识别)
3. 只取了正样本中iou > 0.7的用于计算损失

与类别无关(只输出一个mask)或输出多类别的网络(使用softmax)相比, 这样做没有类内竞争, 对重叠的mask效果更好. 一个有趣的现象是类别无关的结构效果也几乎一样(nearly as effective), 说明网络将分类和分割解耦得很好

## NMS
> 未完待续

## 训练细节
1. 每张图都预处理到1024*800, 保持长宽比
2. 120k iter之后学习率降低10(StepLR)
3. 5 scale, 3 aspect的anchor box

