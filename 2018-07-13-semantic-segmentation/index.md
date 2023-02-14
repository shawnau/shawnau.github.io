# 语义分割综述


存货, 介绍了语义分割模型的发展历史以及基本理念

<!--more-->

## FCN
[Fully Convolutional Networks for Semantic Segmentation](https://people.eecs.berkeley.edu/~jonlong/long_shelhamer_fcn.pdf) (2014)


1. 全卷积架构, 不带全连接层
2. 解决pooling过程中增大感受野的同时丢失位置信息的问题, 这对segment影响很大
    - Encoder-Decoder结构, 用可学习的transpose-conv来上采样, 同时还伴随前后跳跃连接(U-NET采用concat,  FPN采用加法)
    - 空洞卷积, 见下节

## SegNet
![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/segnet.png)
[SegNet: A Deep Convolutional Encoder-Decoder Architecture for Image Segmentation](https://arxiv.org/abs/1511.00561) (2015/11/2)
1. 缓存了max pooling的位置(memorized max-pooling), 这样先upsampling之后获得稀疏特征图, 再卷积获得dense的特征图
2. 最后一层输出$class_number$层, 再使用带权重的pixel level的softmax, 即class balancing, 按照类的中位数的频率来给权重(median frequency balancing), 稀少的类权重更高
> The final decoder output is fed to a multi-class soft-max classifier to produce class probabilities for each pixel independently.

## 空洞卷积(dilated convolution)
![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/dilation.gif)
[Multi-Scale Context Aggregation by Dilated Convolutions](https://arxiv.org/abs/1511.07122) (2015/11/23)
**空洞卷积**(dilated convolution)替代pooling: 既能够增大感受野, 也避免了down-sampling保留了位置信息

## DeepLab v1/v2
(2014v1, 2016v2)
![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/deeplab1.png)
1. 空洞池化(atrous spatial pyramid pooling, ASPP), 多尺度的空洞卷积并行处理, 这个和inception v1的理念差不多哇
2. 全连接的CRF进行后处理


## PSPNet
![](https://hszhao.github.io/projects/pspnet/figures/pspnet.png)

[Pyramid Scene Parsing Network](https://arxiv.org/abs/1612.01105) (2016/12/4)
还是多尺寸特征融合. deeplab中用不同尺寸的空洞卷积并行处理, 这里就用不同尺寸的pooling, 再上采样到相同尺寸后concat...之后的卷积层就能获得不同尺度的信息. 换汤不换药啊

## DeepLab v3
加点BN, 又是篇文章

## Hybrid Dilated Convolution, HDC
[Understanding Convolution for Semantic Segmentation](https://arxiv.org/abs/1702.08502) 
 (2017/2)
![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/dilated1.jpg)
用更加奇形怪状的空洞卷积兼顾大物体和小物体的检查, 避免gridding effect

## Mask RCNN
[Mask RCNN](https://arxiv.org/abs/1703.06870)[ICCV17 Best Paper]
1. 用了Faster-RCNN作为前端
2. 用了encoder-decoder结构做mask head
3. 解耦了classification和segmentation, classification只需要RCNN去做就行了, 而FCN等模型是需要同时解决两个问题的. 带来提升的原因是做segmentation的时候解决了class competition

详见blog中关于mask-rcnn的文章...

## Learning to Segment Every Thing
![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/segeverything.png)
[Learning to Segment Every Thing](https://arxiv.org/abs/1711.10370)[CVPR18]
1. 解决了一下有box却没有对应mask的情况下如何处理分类的问题: 学习一个模型, 把检测网络的权重转换成mask网络的权重. 
2. 因为detection和segmentation使用的都是同一份特征, 很自然的, 这篇文章说明两者是可以互相迁移的. 下文的PANet也考虑到了这一点, 不过是用额外的一条high-level的向量来指导mask的预测

## PANet (SOTA)
![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/panet1.png)
[Path Aggregation Network for Instance Segmentation](http://link.zhihu.com/?target=https%3A//arxiv.org/abs/1803.01534)[CVPR 18]

 - 在FPN的基础上又加了一个Bottom-Up的金字塔. 用conv3x3进行S=2的downsample之后与平行的P层相加, 再过一个3x3conv(这个FPN里没有, 倒是平行的C层需要先过conv1x1统一channel数). 作者认为:
 > high response to edges or instance parts is a strong indicator to accurately localize instances

![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/panet2.png)
 - 其中Detection分支是拉平成fc之后经过两个fc层

![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/panet3.png)
 - Mask分支是将四个特征图concat到一起. 然后利用conv3的输出辅助一个fc向量, 希望使用high level的特征指导mask. 因为fc层的好处就是可以学到交叉特征.


## 一些个人小结
 - 可以发现, 在segmentation中大多数时候我们在和pooling作斗争. 那么能不能不用pooling? [pooling能不能用conv代替](https://www.zhihu.com/question/270777218)中有讨论过可行性
 - Segmentation里面有很多deconv, [deconv和upsample的区别](https://www.zhihu.com/question/63890195)中提到了其实deconv可以做成bilnear的upsample.
 - 另外, [如何理解深度学习中的deconv](https://www.zhihu.com/question/43609045)中的高赞答案从计算角度给出了deconv的数学解释, 顺便还可以复习一下conv的实现
 - 如果segmentation能一步到位, 那就不需要detection了! 有没有类似的工作? [一文介绍3篇无需Proposal的实例分割论文](https://zhuanlan.zhihu.com/p/35770716)带来很多干货. 我认为这是未来趋势

下面介绍一些非主流方法

## GAN
![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/ganseg.png)
[Semantic Segmentation using Adversarial Networks](https://arxiv.org/abs/1611.08408)(2016/11)
FAIR的大作. 其中Soumith Chintala哥哥是pytorch的主力, 他的名字我老是读成吃蛋挞

大体思路是用一个segmentation网络生成mask, 然后再用判别网络区分真实mask和生成mask. 损失函数是生成网络的逐像素softmax损失和判别网络的二分类logloss

![](https://my-imgshare.oss-cn-shenzhen.aliyuncs.com/semiseg.png)
[Semi Supervised Semantic Segmentation Using Generative Adversarial Network](http://link.zhihu.com/?target=https%3A//arxiv.org/abs/1703.09695)(ICCV17)
将分割网络(FCN)变成了判别器, 判别的是逐像素是不是instance.
> 假设分割类别数为K，那么判别器则有K+1个类别的输出。多出来的分类类别为”该像素为假像素”。训练时，使用标记的分割图像训练前K个通道，使用（真实图片，生成图片）图片组按照adversarial loss的定义训练”该像素为假像素”的通道。真是图片既有分割数据库中的图片，也有大量未标注的图片。或者也可以理解为判断“真/假”的分类器，其“真”的这一类扩展成了K类具体的语义类别。


## Pixel Embedding
![](https://pic4.zhimg.com/80/v2-e1e0958468e1b3ac4a776d2b2838582f_hd.jpg)
![](https://pic4.zhimg.com/80/v2-96e7e28ab2727f2d544a9eb8c35c10a3_hd.jpg)
[Semantic Instance Segmentation with a Discriminative Loss Function](http://link.zhihu.com/?target=https%3A//arxiv.org/abs/1708.02551)(CVPR2017)

1. 假设全卷积网络最终输出为$(C, H, W)$传统方法将输出的每个像素位置变成一个$C$维向量, 维度等于类别数. 最终使用softmax损失会将每个向量embedding到一个one-hot向量上去
2. 这里使用的Pixel Embedding是指将输出的向量在embedding空间中计算损失函数(如上图)
3. 这么做并不需要
