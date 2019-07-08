## BASNet：一种能关注边缘的显著性检测算法，已开源！

autocyz [CVer](javascript:void(0);) *昨天*

点击上方“**CVer**”，选择加"星标"或“置顶”

重磅干货，第一时间送达![img](https://mmbiz.qpic.cn/mmbiz_jpg/ow6przZuPIENb0m5iawutIf90N2Ub3dcPuP2KXHJvaR1Fv2FnicTuOy3KcHuIEJbd9lUyOibeXqW8tEhoJGL98qOw/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

> 作者：autocyz
>
> https://zhuanlan.zhihu.com/p/71538356
>
> 本文已授权，未经允许，不得二次转载

https://webdocs.cs.ualberta.ca/~xuebin/BASNet.pdf

代码：https://github.com/NathanUA/BASNet



**主要贡献**

1）将显著性检测网络两个部分，一部分是predict网络，可以得到coarse saliency region，一部分是紧跟在预测网络后后面的fine网络，用来对上一步得到的coarse saliency区域进行进一步的细化，得到更加精确地显著性图。这两个网络的网络结构大致相同，都是经典的Encode-Decode网络，只不过predict网络的结构更加深一些，而fine网络则浅一些。

2）提出了hybrid loss。通过将Binary Cross Entropy (BCE)，Structural SIMilarity (SSIM)和Intersection-over-Union (IoU)三种loss进行结合，让模型能够关注到图像的pixel-level，patch-level和map-level三个不同层级的显著性信息。从而获得更加精确的显著性结果。

## **BASNet网络结构**

![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oViaWtdlWVoVRnXA5l51vmpoFQusTLS3ufdbbgdnFAxwUib2PP1x4Y0C3UicKficrNfP2MjuUCTJf0YyQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

上面的网络结构即整个BASNet算法的网络结构，同时也是整个算法的pipeline。

整个BASNet的网络结构分为两个部分：

一部分是Predict Module，这部分网络输入一张图像，然后经过encode和decode层，输出初步预测的显著性图。这部分网络就是毕竟经典的Ecode-Decode网络，前面的Encode对图像进行提取特征，使用Pooling方法得到了分辨率逐步变小的高层语义特征，后面的Decode部分则负责将高层语义信息逐步还原放大，从而逐步获得大分辨率的feature map图，最终输出和原图一样大小的显著性图。在Encode和Decode之间，会有shortcut，将相同分辨率的feature map图相加，从而让最终的输出的feature map能够兼顾low-level和high-level的特征。值得一提的是， 在decode的过程中，共有 6 种不同分辨率的feature map图，再加上encode阶段最后一层的feature map，一共使用了7个feature map进行loss算，这种多层多loss的方法有点类似于中继loss，一方面可以帮助网络更好的收敛，另一方面可以让网络关注到不同尺度的显著性图。

另一部分是Residual Refinement Module，这部分的网络结构其实和前面的Predict Module模块网络结构一样，使用conv、BN、RELU构造encode和decode，只不过与前面的Predict Module相比，这部分的网络结构要简单一些，网络深度低一些。另外，这部分的loss只用最后一层的输出作为loss，中间层的输出则没有。

## **Loss**

总的loss等于每层的loss的加权和： ![img](https://mmbiz.qpic.cn/mmbiz_svg/qd3u5IHSYT8HPjOtqjPUianWW2ebmD5XJvibo3n3myZy7vXFndN6hbPa7KAqIYaKXFh3Tia9ePC50NiamhIjxOer9iblPiciaggwZicd/640?wx_fmt=svg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

而每层的loss又由三部分loss组成： ![img](https://mmbiz.qpic.cn/mmbiz_svg/qd3u5IHSYT8HPjOtqjPUianWW2ebmD5XJVJ8vBiaIuz9yzItibv3uZCNxToaXZIJDNjxicLCJP7vjb838RgMjAcymXV0o2mzEErz/640?wx_fmt=svg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

三部分loss分别：

1. pixel-level的Binary Cross Entropy (BCE）loss： ![img](https://mmbiz.qpic.cn/mmbiz_svg/qd3u5IHSYT8HPjOtqjPUianWW2ebmD5XJe2KFZfd1pAJJgJfxl5Ak2fk05bYZm3kVrBqUlv6hHlWUGTyNA1ZhaM9u4jI5sJ6G/640?wx_fmt=svg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)![img](https://mmbiz.qpic.cn/mmbiz_svg/qd3u5IHSYT8HPjOtqjPUianWW2ebmD5XJNW4ICp6phjgRrB7HYhib502wj95LHIsibNrkedicaBCl6PgazR6Ksea9oQ6tbUGI6Ja/640?wx_fmt=svg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_svg/qd3u5IHSYT8HPjOtqjPUianWW2ebmD5XJ2NxsAVeuMooGpevvFwrQLVz7Rggd2Lfqo9bzDgyq55d0icMdWxZia6uSkUEnBpIeud/640?wx_fmt=svg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

其中 ![img](https://mmbiz.qpic.cn/mmbiz_svg/qd3u5IHSYT8HPjOtqjPUianWW2ebmD5XJNW4ICp6phjgRrB7HYhib502wj95LHIsibNrkedicaBCl6PgazR6Ksea9oQ6tbUGI6Ja/640?wx_fmt=svg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1) 表示在第r行第c列处的真实值， ![img](https://mmbiz.qpic.cn/mmbiz_svg/qd3u5IHSYT8HPjOtqjPUianWW2ebmD5XJgxOb5piaFU4ibILXPiavibd4MLic0FxlpiaHPt6jANiaSibfxNwU6q3Lo0ZYV2lbabROydNG/640?wx_fmt=svg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1) 则表示的是预测值。从公式形式可以看出，loss与每个像素都有关，因此是pixel-level的loss。

\2. patch-level的Structural SIMilarity (SSIM) loss： ![img](https://mmbiz.qpic.cn/mmbiz_svg/qd3u5IHSYT8HPjOtqjPUianWW2ebmD5XJls8HgpW1EYmPV6lTqGFMN2ticMgZ5QkYud1ewMricP5cEibU3RB7uTyjlvRwfH769kY/640?wx_fmt=svg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_svg/qd3u5IHSYT8HPjOtqjPUianWW2ebmD5XJKBn2FKqa7vlyia1p5KGHogOY5buW5Su8Pqqqia72MBTs19wibYmbyicV3wstTC3OS9DL/640?wx_fmt=svg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

其中 ![img](https://mmbiz.qpic.cn/mmbiz_svg/qd3u5IHSYT8HPjOtqjPUianWW2ebmD5XJAkbcFG4k98zJZia5064znHh8TmSeZBMDOvNe5kkjLRktkCK8W9T0icWibCs3xHaTicSf/640?wx_fmt=svg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1) 表示从预测的显著性图和groundtruth上抠出的N*N区域。 ![img](https://mmbiz.qpic.cn/mmbiz_svg/qd3u5IHSYT8HPjOtqjPUianWW2ebmD5XJQTOIE6ztxJ1QgKJicYictmiaGKjsBNu2LFunaLXrmlQ9FQ9Vq4MErMK6tSFXnBn99wN/640?wx_fmt=svg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1) 分别表示 ![img](https://mmbiz.qpic.cn/mmbiz_svg/qd3u5IHSYT8HPjOtqjPUianWW2ebmD5XJLoo7qbHHpq9Zvp3Ddk3riaa9E2CWT2ZJhIXrybLzmoTCCdz6pjEWGGl1KMXFmP4IG/640?wx_fmt=svg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1) 的均值和方差， ![img](https://mmbiz.qpic.cn/mmbiz_svg/qd3u5IHSYT8HPjOtqjPUianWW2ebmD5XJLGDv5gt8SkYQwLbNlz80V6RibsbnADGXvzoS9tFxwBazZkz4vQF5x3fDr2FdibIE50/640?wx_fmt=svg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1) 则表示他们的协方差。 ![img](https://mmbiz.qpic.cn/mmbiz_svg/qd3u5IHSYT8HPjOtqjPUianWW2ebmD5XJB4alic1pAicD71QYicVYuFgKwQOKX5IlYZEjL61mqAbffSdfkDJAkSTSibK6pPXV5Pae/640?wx_fmt=svg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1) 则都为了防止出现除以0 的情况。分析loss可以发现，每个像素点的产生的loss都与其附近的局部patch有关（这里是N*N的patch），因此在训练的过程中，会对物体边缘部分的loss值加强，对非边缘部分抑制。正式因为这个loss的存在，使得该算法可以关注到更多的目标显著性的边缘细节信息，

\3. map-level的IoU loss： ![img](https://mmbiz.qpic.cn/mmbiz_svg/qd3u5IHSYT8HPjOtqjPUianWW2ebmD5XJ69N3CoRRw3THI9tpZ1e3oicZ7cjByypKk8Ox1icYL8yjicyXBkJFKiaUoh6CPRfmXKIm/640?wx_fmt=svg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

![img](https://mmbiz.qpic.cn/mmbiz_svg/qd3u5IHSYT8HPjOtqjPUianWW2ebmD5XJib6Xf0dbpWTkdfIUERKuge43Upxcib8piahX4oUvUUdN9GvYDwc0se71eRTpZdUEFu8/640?wx_fmt=svg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

这里的S、G和BCE loss的含义是一样的。

三种loss中，BCE loss和IOU loss都是比较常见的目标检测、分割的loss。而SSIM loss在分割则不常用。这个loss一般用于衡量两幅图像的结构相似性，其对局部结构变化比较敏感。

## **实验结果**

下图是实验结果对比，可以发现性能还是比较强悍的，在GPU上，256*256的图像也可以达到25fps。虽然网络结构的改进只是对Encode-Decode结构进行叠加，loss的改进创新也只是对三种loss进行组合，但是最终的结果来看还是比较work的，也说明作者的改进点简单有效。

![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oViaWtdlWVoVRnXA5l51vmpoKeg7hFMsyLQPpkyEdNiahvYB3JxqLGALPYiaUeLdFTmMIBj6ibSWyy2OQ/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

## **总结**

两大贡献：

1、构造了Coarse-to-Fine的Encode-Decode网络结构

2、借鉴目标检测、目标分割、图像相似度匹配的思想，构造了混合的loss，此loss能够关注图像的pixel-level、patch-level、map-level的显著性，从而获得更加精细的显著性结果。



**CVer-显著性目标检测交流群**



扫码添加CVer助手，可申请加入**CVer-显著性目标检测群。****一定要备注：****研究方向+地点+学校/公司+昵称**（如显著性+上海+上交+卡卡）

![img](https://mmbiz.qpic.cn/mmbiz_jpg/yNnalkXE7oX7pdpBKibicSnmb8wRGicbT0Rhr61k0f922lbXcowibk5DTRibROvFB1yMCAZQvj1iaEe6Qsia9bU0UMJCA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

▲长按加群



![img](https://mmbiz.qpic.cn/mmbiz_png/e1jmIzRpwWg3jTWCAZ4BrnvIuN20lLkhIjtg4GRSDhTk9NpeF0GGTJwUpKPatscIQU7Ndj9hgl8BPpGj2BJoFw/640?tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

▲长按关注我们

**麻烦给我一个在看****！**

[阅读原文](https://mp.weixin.qq.com/s?__biz=MzUxNjcxMjQxNg==&mid=2247490221&idx=3&sn=7a82a5641c1dd4a20272296c10f32ddd&chksm=f9a26822ced5e1340449ef98e3d3734dd418f37a5776583bf513db3d8ce7022c56ba1e822662&mpshare=1&scene=1&srcid=0708mXXzEKXopoYNFFJI0KRT&key=cfad420b0c7e89f9ecfca1471649069fc07d736d9de93ad699a50164fac7814a700885f235d9b4704ffbb86420630e8c151d7c090c66f33d6a8f7241705fe94f31ca7257bb215edaab29827a25e17368&ascene=1&uin=MjMzNDA2ODYyNQ%3D%3D&devicetype=Windows+10&version=62060833&lang=zh_CN&pass_ticket=TJZ2x%2BCeLcNXILoA2fzlvgGCucD2AExSAq4kcuqUm1grb%2BD3%2FE%2FG0iYjqRlvhdTC##)



![img](https://mp.weixin.qq.com/mp/qrcode?scene=10000004&size=102&__biz=MzUxNjcxMjQxNg==&mid=2247490221&idx=3&sn=7a82a5641c1dd4a20272296c10f32ddd&send_time=)

微信扫一扫
关注该公众号