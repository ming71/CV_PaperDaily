|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2017.1.7 [ECCV]<p/span>

## 1. Motivation
* 提出问题  
&emsp;&emsp;现在的很多特征利用都涉及深层网络的组合和预测，时间成本大难以达到实时性；而轻量化的网络精度又不够。似乎优化性能只能在深度特征上做文章，如何优雅地设计一种方案，摆脱唯深度决定的桎梏？

* 来自人类视觉的启发  

<!-- more -->

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/RFBNet/1.png" alt="" style="width:60%" /></center>

&emsp;&emsp;这张图不太好理解。简单谈谈个人体会：  
（1）视觉细胞的（群体）感受野是其偏心率的函数  
（2）针对不同皮层的视觉细胞，偏心率和（群体）感受野大小都是正相关的，差别只在于斜率不同。  
（3）揭示了目标区域要靠近中心的重要性。（*属于引用和直译，没有理解怎么得出这个结论的*）  

## 2. Related Work  
&emsp;&emsp;同样在网络中应用了感受野进行改进的其他网络对比分析（~~他这个分析我没看太懂，所以有一半是自己猜的....~~）（结合3.1分析就可以懂一点）：
* Inception  
&emsp;&emsp;虽然采取了不同感受野卷积核，但是卷积中心是一样的，和带孔卷积相比，需要用到更大的卷积核才能达到相同的采样覆盖率。

* ASPP  
&emsp;&emsp;每个分支上使用不同的膨胀率，最终得到了和中心的采样距离；但是这些特征具有来自相同卷积核大小的先前卷积层的相同分辨率，且在所有位置上平等地处理特征。（没看DeepLab的论文，所以引用网上的解释，~~但是卷积不都是这样的吗....~~）

* Deformable CNN  
&emsp;&emsp;可变形的CNN针对不同对象学习到了完全不同的分辨率位置分布，但是它与ASPP具有相同的缺点

## 3. Method  
&emsp;&emsp;设计了RFB模块，作为基础模块可以应用在单阶段目标检测器上提升性能。

### 3.1 Visual Cortex Revisit  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/RFBNet/2.png" alt="" style="width:90%" /></center>

&emsp;&emsp;结合这张图就能对不同方法的特点和利弊一目了然了。  
&emsp;&emsp;首先是Inception的感受野小，因为和膨胀卷积相比还是远远不够的；然后是ASPP，虽然感受野扩大了，但是由于都是3x3卷积得到的结果，没有考虑图1的视觉皮层细胞的偏心率问题；可变性卷积很好地聚焦了特征；RFB则设计了不同的卷积核和膨胀率，仿照人类视觉皮层。

### 3.2 Receptive Field Block  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/RFBNet/3.png" alt="" style="width:70%" /></center>
&emsp;&emsp;从结构图能看出来，这个多分支卷积模块可以划分为两部分：不同尺度卷积核以及不同膨胀率的卷积（或池化）。<u>分别和图1的生物结构相适应</u>。下面分别介绍。

* Multi-branch convolution layer  
&emsp;&emsp;相比与固定尺度和感受野的网络，采取多尺度感受野更加合理，这部分实现类似Inception，思想就是提供不同感受野的特征来模拟不同细胞。结构采用的Inceptionv4类似，直接改的，打造了下面两种结构：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/RFBNet/4.png" alt="" style="width:80%" /></center>


* Dilated pooling or convolution layer​  
&emsp;&emsp;即采取不同膨胀率的膨胀卷积，如上图最上层的卷积所示，来模仿不同离心率和对感受野的影响，卷积核尺寸与膨胀率之间具有群体感受野的大小和其偏心率一样的正相关关系。最后结果进行concatenate合并，得到类似视觉皮层细胞的阵列。

### 3.3 RFB Net Detection Architecture  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/RFBNet/5.png" alt="" style="width:85%" /></center>

&emsp;&emsp;结构细节不细看了，包括VGG和SSD两版，涉及底层高层使用不同RFB模块，以及高层无法使用等细节问题，不赘述。比较有趣的是，这个网络也能train from scratch，可能是轻量化参数不多的原因，但是DetNet也这一点来说事，这个点可以关注一下。
&emsp;&emsp;就实验结果来看涨点确实挺高的，而且速度也还不错，其实很容易想到TridentNet就能涨点不错，这个工作的insight更足，设计也细节得多，只要合理和巧妙理所当然会不差（~~只是这个启发我是真没弄明白~~）。

### Conclusion
&emsp;&emsp;有空可以review，主要是想弄明白到底是怎么通过那个偏心率的图启发到膨胀率和卷积核的，怎么对另外三种方法进行批评的。









<br>
<br>
<hr />