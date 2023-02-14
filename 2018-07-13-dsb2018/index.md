# Data Science Bowl 2018 比赛小结


Kaggle 实例分割比赛小结. 比赛花了我1800在阿里云上训练模型, 最后还因为没提交模型的checksum而取消了成绩.
> 包大人, 冤啊!

<!--more-->

## 难点
0. 训练数据少. 黑白600, 彩色100
1. 细胞核形状多样, 有的十分细长
2. 细胞核紧挨, 有重叠部分, 难以使用unet区分(unet先预测mask再用watershed(分水岭算法)分割)
3. 黑白染色和彩色染色的图片细胞核区别显著. 彩色图片训练数据少, 颜色种类不同, 模型表现差
4. 测试集具备训练集没有的细胞核类型. 甚至教科书中的手绘插图

## 解决方案
实例分割可用选项:
1. U-Net, 比赛初期的主流选择. 这是二分类网络, 且只能用于语义分割而不是instance level的实例分割(无法区分不同细胞核), 需要用后处理(watershed算法)分割拥挤细胞核. 直觉上比Mask RCNN这种自带目标检测的有劣势
2. FCIS(全卷积网络用来做实例分割), 比U-Net的优势是可以做实例分割, 实际上就是先detection之后, 开一个新分支, 把ROI进一步划分(成9块), 专门判断ROI里面那些内容是前景. 这么做还是比较粗糙, Mask RCNN论文里也提到 But FCIS exhibits systematic errors on overlapping instances. 对重叠部分表现很差.

敲定Mask RCNN之后, 选择合适的Backbone.

### Mask-RCNN base model
1. SE-ResNeXt50-FPN BackBone, 训练集不大, 所以需要一个尽量小巧并且高效的Backbone. 最初想到的是ResNet50. 做了一些research之后发现还可以用Xception里depthwise conv进一步减肥, 于是使用**ResNeXt50**. 这场比赛比较大的困难是不同图片里的细胞核大小天差地别, 因此很自然上**FPN**. 最后实验性地加入了SE 模块, 免费提分的工具何不试试.
2. 图片尺寸只有256, 而coco数据集图片一般是1024*1024尺寸的. 因此取消ResNeXt的conv1的pooling层获得类似尺寸的FPN-Feature Map
3. 更多细长比例Anchor Boxes捕捉细长并紧挨的细胞核, 并且根据mask绘制更紧密的gt(训练集只提供了mask). 但是考虑到太过细长的anchorbox超出了3x3卷积核的感受野, 是不是需要更丰富的卷积核呢?(**提升点**)  
4. RPN的cls head使用了**focal loss**, 这样也避免了OHEM的复杂性. reg head使用了**weighted_smooth_l1**, 加上了权重, 给foreground和scale较小的实例更高的权重
5. **UnitBox**用于bbox regression, [UnitBox: An Advanced Object Detection Network](https://arxiv.org/abs/1608.01471): RPN的Reg Head原来使用的是weighted_smooth_l1, (未来得及使用**iou loss**替代, **提升点**. 因为有大量紧密重叠的细胞核, 类似于人群中人脸识别的特点, iou loss对小的细胞核有帮助, 可以回归得更准确) 使用UnitBox的encoding方式主要是因为没有exp, 而训练伊始exp这部分很容易爆炸, 强制做clip收敛又慢, 因此做了这样的改进.   
6. 因为是细胞核(偏向于圆形)且为二分类, Mask Head使用[**Dice Loss**](https://dev.to/andys0975/what-is-dice-loss-for-image-segmentation-3p85), 比cross entropy效果更好一些. dice loss蕴含了强调了中心点的重要性. 因为最难处理的情况是proposal包含了拥挤的细胞核, mask head的任务是找出中心的细胞核.此时mask的每一个pixel都是相同的权重, 这不符合直觉, 需要dice loss来进行强调  


### 预处理
1. CLAHE(限制对比度自适应直方图均衡算法), 对医学图像增强效果好
2. Stain Normalize: A. Vahadane et al., ‘Structure-Preserving Color Normalization and Sparse Stain Separation for Histological Images 维持不同染色细胞颜色统一

### Train Time Augment

1. 随机裁剪到256*256, 若不够则zero padding维持长宽比
2. 0.5~1.5的随机缩放
3. 水平/垂直翻转 
4. 90°旋转
5. 随机高斯模糊
6. 高斯噪音
7. 随机对比度
8. [random_hue_transform做数据增强](http://lamda.nju.edu.cn/weixs/project/CNNTricks/CNNTricks.html): 有彩色和黑白图同时存在, 彩色图片染色风格不同  


### Test Time Augment
1. padding 到anchor的整数倍, 保证anchor可以覆盖到边缘
2. **scale**: 0.6, 0.8, 1.0, 1.2, 1.4, **rotate**: 90, 180, 270
3. 把做了各种增强之后的rpn proposal ensemble, 之后再nms
4. mask instance **clustering**. 以iou为依据进行聚类, 聚类核心为所有mask的交集(按交集/并集处理之后发现交集比较好)
5. mask instance ensemble, 叠加之后取均值. 最佳阈值根据验证集计算出
6. 根据mask 尺寸进行false positive suppression, **由于基本策略是先提升recall, 再抑制FP,** 由于同一张图里的细胞大小差异不会过于悬殊, 因为是显微镜拍摄的图片, 显微镜中是切片, 细胞核距离镜头都是一样的. 直接使用一维异常值检测算法MAD(平均绝对偏差)过滤小的mask. 一开始使用DBSCAN, 后来发现MAD更快而且效果差不多. 因为这个任务比较简单. 

### 训练策略
1. [Cyclical Learning Rates for Training Neural Networks](https://arxiv.org/abs/1506.01186), SetpLR, DecayLR, etc, 最后还是用了StepLR  
2. 训练时的run-in: 初始阶段每个batch中掺入了小部分gt, 既保证了RCNN能有输出, 也保证了Mask head快速收敛. 否则需要很长时间才能收敛. 后期撤除gt
3. 训练流程是依次训练三个head, 最后一起训练
  

### Winner Strategy  

1. [1st](https://www.kaggle.com/c/data-science-bowl-2018/discussion/54741)  
2. [4th](https://www.kaggle.com/c/data-science-bowl-2018/discussion/55118)  
3. [一文介绍3篇无需Proposal的_实例分割_论文](https://zhuanlan.zhihu.com/p/35770716)

> 待续
