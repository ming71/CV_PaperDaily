|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

**顺便mark我的数据增强代码工具（持续更新）：https://github.com/ming71/toolbox/tree/master/data_augmentation**

<span id="inline-blue">论文发布日期：2018.1.2<p/span>

## 1. Introduction
&emsp;&emsp;这篇文章就是简单版的mixup，可以直接参考mixup的论文，作为同期投稿ICLR2018的文章，这篇被拒了mixup被接收也是有原因的。  
&emsp;&emsp;介绍了一种用于分类的数据增强方法--SamplePairing 。这种方法实际是mixup的特例，但是作者也不知道为什么这样混合效果就可以提升，没有可解释性，此外作者设计的训练过程也比较复杂，而mixup就有相对较好的理论解释和更广的应用场合，下次填坑mixup。
<!-- more -->


## 2. Method
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Data-Augmentation-by-Pairing-Samples-for-Images-Classification/1.png" alt="" style="width:60%" /></center>

* **实现方法**  
&emsp;&emsp;从训练集取一张A，再随机取另一张B，对两者进行基础数据增强后采取像素平均化叠加，得到的label是A的，然后送到网络训练。

* **对称性**   
&emsp;&emsp;如果数据集大小为N，扩增之后最大为NxN，对于小数据集很有利。虽然label取得是一个，但是比如取label A融合B，后面在label B的时候也会融合A，从而是对称的。

* **问题**   
&emsp;&emsp;由于同样两张图片融合却可能输出不同的label，对于网络的学习肯定还是有影响，导致理论无法得到极高（100%）的AP。但是作者设计了复杂的训练过程，发现能够大幅度减少分类误差，也算是比较好地弥补了这一点问题。   

* **训练过程**   
&emsp;&emsp;先采用普通数据增强训练，完成一定轮数之后加入samplepairing，同时间歇性调用该增强，直到loss比较稳定后停止samplepairing的使用。实验来看效果肯定没的说，确实可以，有意思的是这个增强和其他方法一样，会造成loss很大的波动，但是整体趋势是下降的，同时在最终fine-tune会有稳定的收敛loss，其他方法如果最后取消augmentation效果应该也类似。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Data-Augmentation-by-Pairing-Samples-for-Images-Classification/2.png" alt="" style="width:70%" /></center>


## 3. Conclusion 
&emsp;&emsp;关于这个方法的思考，暂不记录在这篇ICLR的弃子上，留到mixup进行总结，毕竟两篇文章几乎就是子集关系了（~~尽管这篇作者还在related works里面大篇幅澄清他和mixup的关系~~）。简单从使用角度来看，对于检测而言如果想用这个samplepairing可以将图片直接融合然后输出，至于label（bbox）可以考虑只选取一个或者两个都是正解，因为检测任务的遮挡和背景干扰更严重，而两个label都用可以抵抗这一点。
&emsp;&emsp;此外，关于这种图像叠加可行性有一个**正则化的解释**，通过学习线性函数来简化获得更具鲁棒性的模型的角度很有趣，在mixup在做分析。





<br>
<br>
<hr />