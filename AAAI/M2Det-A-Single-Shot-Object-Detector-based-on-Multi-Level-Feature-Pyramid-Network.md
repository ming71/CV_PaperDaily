|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2018.11.12 [AAAI]<p/span>

## 1. Motivation  

* 现有的FPN特征融合结构都是基于最初设计为分类网络的backbone上进行的，这种分类表征能力的特征用于目标检测不一定准确(这个intuition本身其实挺好，但是分类特征为什么用于检测不好？哪里不好？作者没有进一步挖掘)
* 也提到了特征预测尺度**inconsistency**的问题  
<!-- more -->

&emsp;&emsp;作者对高低层信息的表述：低层特征更容易响应简单的物体外形；高层特征更多的响应复杂的外形。这个说法的角度比较直观挺有意思，虽然不难理解。如低层响应的车轮、车窗、眼睛等，稍微高一点会有狗，人，车等，这也能用高层语义信息具有更多的信息组合方式来解释。而从外形解释更直观地联想到为什么大尺度特征图对小物体检测重要。

## 2. Multi-level Feature Pyramid Network    
&emsp;&emsp;先上一张对比图这个对结构描述比较直观。这张图有个小错误，最后其实U型网络后半段输出的并不单纯是上采样的结果，而是上采样后和前半段对应尺度特征图融合后的结果。     
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/M2Det/str1.png" alt="" style="width:80%" /></center>

&emsp;&emsp;再放一张细致的结构图，可以看出，网络自己搭建的地方其实分成三个部分：FFM、TUM、SFAM三模块。总体流程大概是首先经过FFMv1融合VGG的两个层得到base feature，然后送入级联的TUM模块进行U型下采样和上采样并融合，得到从浅层到深层递进的多个多尺度特征图组，然后送入SFAM不同特征图组之间进行对应尺度的concat，加上注意力模块，得到多level的输出。实验中实际采取了6个不同尺度，8个不同深度的特征图组。（<u>我的结论是，这是一篇在改进层面毫无亮眼insight的论文。或者说，在论文表现不明显，也许我个人对特征表达和融合的理解还不够有待学习。但是至少他完全没有FPN、ResNet等这种大道至简、一针见血的观点</u>）  
。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/M2Det/str2.png" alt="" style="width:100%" /></center>

### 2.1 Multi-level Feature Pyramid Network            
&emsp;&emsp;简单补充两句，其基础特征融合选择的是VGG的conv4_3和conv5_3；可以看出TUM的级联是向下的，所以最浅层的输出只用了基础特征，逐渐往下更多地复用了前面的特征。

#### 2.1.1 FFM            
&emsp;&emsp;FFM分为两种，FFMv1是融合基础特征的，方法是concat，结构比较简单，直接看图就行。Conv模块的第三个参数是降采样步长注意一下，其他都很简单略。 
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/M2Det/str3.png" alt="" style="width:50%" /></center>

&emsp;&emsp;FFMv2用在TUM内，主要是融合上一组特征图中最大尺度的那张特征图和基础特征，方法是concat，这个更简单。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/M2Det/str4.png" alt="" style="width:50%" /></center>

#### 2.1.2 TUM     
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/M2Det/str5.png" alt="" style="width:90%" /></center>

&emsp;&emsp;看上去很复杂，其实就是将输入特征图连续五次下采样，然后五次上采样，并且把对应尺度相同的部分融合起来，注意这里方法是sum。原图尺寸在内可以得到六张不同尺度特征图。也没什么好说的。  

#### 2.1.3 SFAM      
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/M2Det/str6.png" alt="" style="width:90%" /></center>

&emsp;&emsp;用于融合各尺度特征，将对应尺度（6个）在不同level上取出来，进行concat然后引入channel-wise注意力模块，再引用了**全局均值池化**（很多论文都在说这个，可以试着用用），加两个FC层和ReLU出来即可。

### 2.2 Network Configurations             
&emsp;&emsp;TUM通道用的256以减少参数（和FPN一样），输入尺寸可选300/512/800。（啰嗦一句，这里作者还描述了一些阈值的选择和设计，只能说太细致了，参数这么调效果当然差不到哪儿去）


## 3. Experiment

&emsp;&emsp;就不作一一的解读了，不过可以发现作者的实验做的真的细致，可见参数调试必然花了不少功夫，那效果就反而不好说到底是谁work了，毕竟这个思路在方向上是对的，加的trick也不少。简单放一些结论（也许自己理解不够，过段时间再review看看有没有新的发现和理解）：
* TUM的通道数经过实验测试得出，和FPN一样为256时效益最高。加大该参数必然能取得更好的效果（512以内的实验），但是参数量增长过剩。
* SFAM和TUM都存在类似FPN的问题，大目标精度下降（尤其TUM很明显）
* 在每个TUM输入都融合一次base feature效果更佳。作者认为是浅层特征的定位信息优化了检测结果。（问题：这里的基础特征只用了两层，就开始强调浅层特征好，到底怎样定义浅层？究竟是位置还是尺度？我认为偏向后者，前提是信息冗余或者没有大幅度坍缩的情况下）

## Conclusion    
&emsp;&emsp;本文暂不作conclusion，因为感觉自己的解读有问题，不做过早定论。总感觉这篇文章的起点很好，但是insight的理解和所做工作没有抓住要点。毕竟是发在AAAI上的文章，应该还是有东西的，文章的工作也很多，应该有不少我理解不到位的地方，放一段时间再review。

<br>

<hr />