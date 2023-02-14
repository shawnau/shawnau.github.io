# 目标检测综述-从RCNN到Mask-RCNN


存货, 介绍了RCNN系列的发展历史以及基本理念

<!--more-->

## RCNN
### 预测
[Rich feature hierarchies for accurate object detection and semantic segmentation](https://arxiv.org/abs/1311.2524)(2013)
1. **Selective Search** 生成大约2000个 **Region Proposals**, 这是category-independent的, 也就是只有object/background之分

3. 特征提取:
    1. 将Region Proposals切割的图片缩放(crop/warp) 到固定尺寸(227*227)
    2. 通过(**Alexnet**) 生成4096维特征向量
4. 对每一类:
    1. 通过$k$个SVM(二分类), 输出属于各个类的score
    2. 给出score和proposal之后, 对每一类用**non-maximum suppresion** over a **learned** threshold来去重
4. 通过训练好的LR模型**Bounding Box Regression**对proposal进行精修

### 训练
1. 载入ILSVRC2012上预训练的Alexnet
2. 训练(Fine-Tune)CNN:
    1. 数据: region proposals切割好的图片
    2. 模型: 将网络最后一层改为$R^{N+1}$层, 代表了N+1(背景)类
    3. **IoU > 0.5**是正样本, 否则是负样本(第N+1类)
    4. 优化: SGD, batch大小为128, 每个batch中正负样本比为1/3, 学习率为预训练的1/10
3. 训练 SVM:
    1. 正样本: **Ground Truth** 通过CNN的特征向量
    2. 负样本: **IoU < 0.3** 的proposal通过CNN的特征向量
 4. 训练LR进行Bounding Box Regression
    1. 输入: CNN输出的4096维向量
    2. 正样本: 计算出bbox delta

### 备注

Q: 为何不用Fine-tune的时候的CNN直接检测物体类别? 最后一层不就是softmax分类器吗? 
A: 因为准确率低, 为了获得充足的样本训练CNN, IoU阈值(>0.5)取得比较低, 检测出来的物体往往不包含整个样本. 将提取出来的特征给SVM并用更严格的策略(Ground Truth为正样本, <0.3为负样本, 其他丢弃, 这是验证集测试mAP出来的结果)来判断包括物体的bbox, 效果更好. 因为SVM适合小样本训练.

Q: 什么是Bounding Box Regression?
A: 有四个模型, 两个是回归坐标的, 两个是回归长宽的. 输入是CNN提取Regional Proposals的特征向量, 输出是四种model要回归的目标中的一个:
$$
\begin{aligned}
\hat G_x &= P_w d_x(P) + P_x \\\
\hat G_y &= P_h d_y(P) + P_y \\\
\hat G_w &= P_w \exp(d_w(P)) \\\
\hat G_h &= P_h \exp(d_h(P)) \\\
\end{aligned}
$$

Q: 什么是Hard negative mining?
A: [rcnn中的Hard negative mining方法是如何实现的](https://www.zhihu.com/question/46292829)

## SPPNet
[Spatial Pyramid Pooling in Deep Convolutional Networks for Visual Recognition](https://arxiv.org/abs/1406.4729)(2014/6)
SPPNet主要做了两件事:
1. 从特征图上裁剪ROI
    1. 在RCNN里, 通过Selective Search生成2000个Region Proposals, 然后crop/warp, 再喂进CNN, 太慢
    2. 在SPPNet里, 将整张图片喂进CNN, 然后从特征图上直接裁剪出ROI. 这样只需要一次前向传播, 比之前要2000次快得多

2. 提出Spatial Pyramid Pooling Layer(SPP Layer)解决了把不同尺度的feature map转换为一个尺度的方案, 不再需要crop/warp了:
    1. 将feature map按不同维度划分并max pooling
    2. 拼接各个维度的输出, 产生固定维度的输出

3. 训练上, 由于可以输入任何尺寸的图片, 所以是分几个阶段用不同分辨率的图训练的. 其他细节看[这篇专栏](https://zhuanlan.zhihu.com/p/27485018)吧....

## Fast RCNN
[Fast R-CNN](https://arxiv.org/abs/1504.08083)(2015/4)
### 预测

1. **Selective Search** 生成大约2000个 **Region Proposals**, 和RCNN一样
2. 特征提取: 
    1. 将图片输入CNN, 得到特征图
    2. 用**Region Proposals**直接从特征图上得到切割过的特征
    3. 使用**ROI pooling layer**得到特征向量, 所以从Fast RCNN开始, 训练和预测对图片的尺寸没有要求了
3. 特征向量 -> FC1 -> SoftMax Probability ($K+1$)
4. 特征向量 -> FC2 -> bbox offsets ($4 \times K$)
5. 对每一类用nms得到最终结果

### 训练

1. 预训练CNN (Imagenet):
    1. 将最后一层pooling layer换成ROI pooling layer
    2. 将最后一个全连接层换成两个并行的全连接层FC1, FC2
    3. 网络需要同时输入图片和ROI
2. 同时训练bbox regressor和softmax分类器
    1. bbox regressor用Smooth L1损失函数
    2. Softmax for probability
    3. 将两个损失相加之后回传
3. 一个Mini-Batch中重采样正负样本:
    1. 一个batch包含两张图
    2. 每张图贡献64个ROI, 一共128个
    3. 正样本: IoU>0.5
    4. 负样本: IoU $\in$ [0.1, 0.5) (**hard negative mining**)
 
### Notes
1. ROI pooling: 其实就是单层SPP layer, 详细机制参考[region-of-interest-pooling-explained](https://blog.deepsense.ai/region-of-interest-pooling-explained/)
2. [原始图片中的ROI如何映射到到feature map?](https://zhuanlan.zhihu.com/p/24780433)这坑挺深的

## Faster RCNN
[Faster R-CNN: Towards Real-Time Object Detection](https://arxiv.org/abs/1506.01497)(2015/6)
核心是把Selective Search交给了RPN做, 而RPN和fast rcnn是可以合并的

### RPN
https://www.quora.com/How-does-the-region-proposal-network-RPN-in-Faster-R-CNN-work

1. 输入: CNN产生的固定尺寸的特征图
2. 第一层: $n \times n$ 卷积层, 输出256维特征向量. ($3 \times 3$ in the paper)
3. 两个并行的分支:
    1. $cls$ 分支: $(2k, H, W)$ 每个像素位置产生$2k$个anchor boxes, 其中k为不同比例/尺寸的anchor数目
    2. $reg$分支: $(4k, H, W)$每个像素位置产生$4k$个delta, bbox regression
4. RPN's loss function:
$$
L(p\_i, t\_i) = \frac{1}{N\_{cls}} \sum\limits\_{i} L\_{cls}(p\_i, p^{\star}\_{i}) + \lambda \frac{1}{N\_{reg}} \sum\limits\_{i} p\_{i}^{\star}L\_{reg}(t\_i, t^{\star}\_{i})
$$
    - $i$: index of anchor in a mini-batch
    - $p_i$: predicted probability of foreground
    - $p^*_i$: grund truth. 1 for postive, 0 for negative
    - $t_i$: bbox regression offset
    - $L_{cls}$: log loss
    - $L_{reg}$: Smooth L1 loss

### Notes
1. 后续RPN产生的ROIs扔进fast rcnn即可, RPN和RCNN共享backbone, 训练过程和细节不表, 见[Faster R-CNN](https://zhuanlan.zhihu.com/p/24916624)
2. 有关RPN的细节见[faster rcnn中rpn的anchor，sliding windows，proposals](https://www.zhihu.com/question/42205480/answer/155759667)


## Feature Pyramid Network
[Feature Pyramid Networks for Object Detection](https://arxiv.org/abs/1612.03144)(2016/11)
核心是Lateral Block. 解决了位置精度和语义信息的冲突问题:
1. 越靠近输入的特征图语义信息越少, 但是位置信息越精确(特征图大)
2. 越远离输入的特征图语义信息越强, 但是位置信息越弱(各种卷积, pooling)

![fpn](http://my-imgshare.oss-cn-shenzhen.aliyuncs.com/fpn.png)

4. 左边是resnet的四层`stage`输出的特征图, 简称C2, C3, C4, C5
5. 右边是P层, 每个P层通过上一层P层的upscale和相邻C层的smooth(统一通道数)之后相加得到. 每层通道数都是256, 其中最高层P4直接由C5 smooth而来.

```python
class LateralBlock(nn.Module):
    """
    Feature Pyramid LateralBlock
    """
    def __init__(self, c_planes):
        """
        c_planes: channels of the C layer to smooth
        """
        super(LateralBlock, self).__init__()
        self.lateral = nn.Conv2d(c_planes, 256, 1)
        self.smooth = nn.Conv2d(256, 256, 3, padding=1)

    def forward(self, c, p):
        """
        :param c: c layer before conv 1x1
        :param p: p layer before upsample
        :return: conv3x3( conv1x1(c) + upsample(p) )
        """
        _, _, H, W = c.size()
        c = self.lateral(c)
        p = F.upsample(p, scale_factor=2, mode='nearest')
        # p = F.upsample(p, size=(H, W), mode='bilinear')
        p = p[:, :, :H, :W] + c
        p = self.smooth(p)
        return p
```

## RetinaNet
[Focal Loss for Dense Object Detection](https://arxiv.org/abs/1708.02002): 对正负样本不平衡的深入挖掘

## YOLO
[人家讲的很好了, 懒得复制粘贴了](https://zhuanlan.zhihu.com/p/24916786)
简单说几点
1. 直接在7x7的特征图上输出每个格子的检测/分类结果, 每个格子输出两个proposal$(C_x, C_y, w, h, score, \vec c)$, 这个score相当于RPN的score, $\vec c$是一个one-hot的类概率向量.
2. 这个输出是先拉平之后经过**fc层**再还原成7x7的! 如此简单粗暴, 惊不惊喜, 意不意外? PANet有更优雅的做法, 分出一个branch来指导. 但总之有fc层会对速度影响挺大的.
3. 这么多loss混在一起, 需要给予不同的权重防止某一种dominate. 有/没有object的权重是不同的(这个有没有需要用iou去判断)
4. 小box的损失应该比大box大, 设计损失函数的时候直接用长宽的根号差来处理...

嗯, 这种把所有损失一锅端的方法非常粗暴

## SSD
1. 这玩意就是个class-specified RPN.
2. SSD取了backbone不同阶段的特征图, 大概当时FPN还没横空出世吧?

[比较一下这两兄弟](https://www.zhihu.com/question/50910763)

## Mask RCNN
详见blog中关于mask-rcnn的文章...懒得再复制粘贴了

## DetNet(SOTA)
![](http://my-imgshare.oss-cn-shenzhen.aliyuncs.com/detnet1.png)
[DetNet: A Backbone network for Object Detection](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/1804.06215)[ECCV18]
主要是对ResNet50的魔改

1. 加入了空洞卷积
2. 唔 为了防止越卷越小加入了一些1x1卷积(似乎?

没有看到特别有趣的insight...

