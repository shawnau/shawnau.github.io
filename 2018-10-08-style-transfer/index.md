# 风格迁移(Style Transfer)


如何将艺术风格量化? 基于深度学习的风格迁移是个很有趣也很浪漫的领域
<!--more-->

## [Texture Synthesis Using Convolutional Neural Networks](https://arxiv.org/pdf/1505.07376.pdf)
2015a, Gatys
![enter image description here](http://my-imgshare.oss-cn-shenzhen.aliyuncs.com/gatys-1.png)

1. 将图片输入预训练好的神经网络(VGG-19)中， 生成每层的特征
2. 将灰度图同样输入预训练好的神经网络中，生成每层的特征
3. 计算每层特征的Gram矩阵，计算两张图片对应层的Gram矩阵的平方误差，再将每层的误差按权求和，获得损失函数
4. 计算损失函数对灰度图每个像素的梯度，进行梯度下降，最终得到输出图。这里进行梯度下降的不是模型参数，而是输入像素。这也是Deep Dream的主要思路。

其中最重要的贡献是定义了纹理，即特征图的Gram矩阵。网络第$l$层的纹理误差为

$$
E\_{l}(S, \hat{X}) = \frac{1}{4C\_{in}^2C\_{out}^2}\sum\limits\_{i,j}(G\_{ij} - \hat{G}\_{ij})^2
$$

其中$S$是需要学习的纹理(Texture Style)的图片，$\hat{X}$代表了需要生成的图片，总误差就是各层误差的加权和：

$$
\mathcal{L}\_{style}(S, \hat{X}) = \sum\limits\_{l}w\_{l}E\_{l}
$$

计算Gram矩阵的方法如下。Gram矩阵可以视为有偏的协方差矩阵
![enter image description here](http://my-imgshare.oss-cn-shenzhen.aliyuncs.com/gram-1.jpg)
![enter image description here](http://my-imgshare.oss-cn-shenzhen.aliyuncs.com/gram-2.jpg)

## [A Neural Algorithm of Artistic Style](https://arxiv.org/pdf/1508.06576.pdf)
2015b, Gatys
本文增加了计算内容相似性的部分。内容相似性就是特征图的平方误差。
$$
\mathcal{L}\_{content}(P, \hat{X}) = \frac{1}{2}\sum\limits\_{i,j}(F\_{ij} - \hat{F}\_{ij})^2
$$
其中$P$代表了需要渲染的图片，$\hat{X}$代表了需要生成的图片，$F\_{ij}$和$\hat{F}\_{ij}$代表了他们在某一层的特征图。总的损失是内容和风格损失的加权和：
$$
\mathcal{L}(P, S, \hat{X}) = \alpha\mathcal{L}\_{content}(P, \hat{X}) + \beta\mathcal{L}\_{style}(S, \hat{X})
$$
其中alpha和beta可以使用L-BFGS训练
## [Perceptual Losses for Real-Time Style Transfer and Super-Resolution](https://arxiv.org/pdf/1603.08155v1.pdf)
Mar 2016, Justin Johnson
以上方法生成一张图是通过梯度下降的方法，也就是一次训练过程，耗时较长，计算资源消耗也大。本工作将生成图片的过程变成了一次inference，降低了计算量，提升了速度。

本文对风格和内容损失有更现代的定义：
### Feature Reconstruction Loss
网络$j$层对应的content loss
$$
\mathcal{L^{j}\_{content}}(y, \hat{y}) = \frac{1}{C\_{j}H\_{j}W\_{j}} \Vert \phi\_{j}(y) - \phi\_{j}(\hat{y}) \Vert^{2}\_{2}
$$

### Style Reconstruction Loss
网络$j$层对应的style loss
$$
\mathcal{L^{j}\_{style}}(y, \hat{y}) = \frac{1}{C\_{j}H\_{j}W\_{j}}\Vert G^{\phi}\_{j}(y) - G^{\phi}\_{j}({\hat{y}}) \Vert^{2}\_{F}
$$

![enter image description here](http://my-imgshare.oss-cn-shenzhen.aliyuncs.com/Perceptual-Losses.png)

本文使用一个resnet来学习生成图片的变换，而将内容和风格损失作为损失函数，其计算方式通过一个固定参数的VGG16来生成。去掉风格损失之后还可以作为超分辨率模型使用。此外3.3节还提到了两个额外的损失函数，Pixel Loss和Total Variation Regularization。后者在风格迁移任务中也被使用到。

## [Fast Patch-based Style Transfer of Arbitrary Style](https://arxiv.org/pdf/1612.04337.pdf)
Dec 2016, Tian Qi Chen
实现了任意风格任意图片的迁移，
![enter image description here](http://my-imgshare.oss-cn-shenzhen.aliyuncs.com/Fast-Patch-based-style-transfer.png)
### Style Swap
1. Content图片产生特征图$\phi(C)$
2. Style图片产生特征图$\phi(S)$
3. 用$\phi(S)$做卷积核扫过$\phi(C)$,寻找最相似的

[参考blog](https://blog.csdn.net/hungryof/article/details/61195783)

