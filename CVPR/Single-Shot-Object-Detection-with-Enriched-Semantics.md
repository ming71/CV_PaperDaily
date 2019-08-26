

|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 


<span id="inline-blue">论文发布日期：2017.12.1 [CVPR] <p/span>




## 1. Introduction

&emsp;&emsp;就思想而言，和本文四个月后发表且同在CVPR的PAD基本类似，采用bbox自监督增强检测结果。
* 出发点：低级特征图不含有高级语义信息。如SSD多级预测在底层预测时只用了低级特征没有结合高级语义信息（~~实际上FPN可以部分地解决，而且现在特征融合的方法非常多样，这个立足点实际已经不算新颖了~~）。
<!-- more -->

* 主要工作：  
提出DES检测网络，通过共享基础特征的两个分支——检测和自监督分割，以后者的学习信息加强前者的底层低级语义信息特征，采用全局激活模块学习特征图通道和物体类别之间的语义关系抽象出高级语义信息。


## 2. Proposed method 
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Single-Shot-Object-Detection-with-Enriched-Semantics/1.png" alt="" style="width:60%" /></center>

&emsp;&emsp;该单阶段检测框架分为三个部分：检测分支、分割分支、全局激活模块。实验中采用的检测backbone是SSD。        
### 2.1  Semantic enrichment at low level layer            
&emsp;&emsp;先看分割分支。

* 运算  
原论文定义地花里胡哨搞了几个公式，其实很简单。共计定义了三个运算：F G H ，其中输入特征为X，有Y=F(G(X))；局部响应特征Z=H(G(X))；最终输出X'是X和Z运算结果。  
具体来说，G运算定义的某个卷积层后会分化成两支：  
  1. 计算loss；loss分支的最后mask是类别得分概率图，通道数为类别N+1                  
  2. 输出mask X'，首先G(X)经过运算得到Z，将Z与X相乘得到mask
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Single-Shot-Object-Detection-with-Enriched-Semantics/2.png" alt="" style="width:65%" /></center>

* mask生成

这个mask十分简单......：对于每个像素，如果落在某个bbox内，就分配对应的类别给这个像素；若一个像素落在多个box内，就分配给最小的那个bbox。一张图就能看明白：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Single-Shot-Object-Detection-with-Enriched-Semantics/3.png" alt="" style="width:40%" /></center>


### 2.2  Semantic enrichment at higher level layers        

&emsp;&emsp;引入该模块的目的是学习通道方向的信息与物体类别的关系。（~~但是C≠N，这个假设能成立吗？~~）。全局激活模块其实就是用的SENet的基本结构。该过程分为三部分：spatial pooling, channel-wise learning and broadcasted multiplying。  
* spatial pooling  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Single-Shot-Object-Detection-with-Enriched-Semantics/5.png" alt="" style="width:19%" /></center>

通道池化，将整个特征图按照通道方向进行均值池化。

* channel-wise learning  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Single-Shot-Object-Detection-with-Enriched-Semantics/6.png" alt="" style="width:40%" /></center>

将上一步的原始向量映射到高维空间再映射回来，得到的该向量即为权重。其中W1维度是C'xC，W2是CxC'，这里取C'=0.25C

* broadcasted multiplying  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Single-Shot-Object-Detection-with-Enriched-Semantics/7.png" alt="" style="width:21%" /></center>

加权相乘。将上一步得到的向量按照逐通道顺序乘到原始特征图上去。        


### 2.3  Multi-task training           
&emsp;&emsp;必然是联合训练，主要看损失：分割loss+检测loss。检测loss很常规，分割loss如下图，就是简单的负对数，相比之下Pseudo Mask Augmented Object Detection的工作确实很足。  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Single-Shot-Object-Detection-with-Enriched-Semantics/8.png" alt="" style="width:32%" /></center>

&emsp;&emsp;然后加个系数α进行权衡：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Single-Shot-Object-Detection-with-Enriched-Semantics/9.png" alt="" style="width:40%" /></center>

## 3. Experiments  
* Result  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Single-Shot-Object-Detection-with-Enriched-Semantics/10.png" alt="" style="width:100%" /></center>

实验结果很意外！和PAD同是VOC2007test，相同的traindataset相比，Faster RCNN在R-101都是76.4说明数据具有可比性。但是，这里能达到81.7，而后者只有77.0。也许这就是后者不敢和SSD比较的原因，因为同backbone甚至不如SSD。不排除这里可能是SENet的模块起作用，但是SSD裸分就和PAD方法差不多了....很是尴尬，应该有哪里条件不一样。

* Ablation Study  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Single-Shot-Object-Detection-with-Enriched-Semantics/11.png" alt="" style="width:40%" /></center>

  都没开源，包括CVPR2019的。



<br>
<br>
<hr />