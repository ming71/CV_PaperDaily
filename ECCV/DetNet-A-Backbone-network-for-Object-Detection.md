|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2018.4.19 [ECCV]<p/span>

## 1. Motivation  
&emsp;&emsp;这个论文的insight不错。作者在related works观察总结到，很多backbone的提出都是用于挑战ImageNet分类任务后被应用到检测上来，因此鲜有单独<u>针对检测任务设计的backbone</u>。          
<!-- more -->
&emsp;&emsp;**检测和分类有明显的区别**：（1）不仅需要分类，还需要精确的定位 （2）最近的检测器都是基于类似FPN结构，在分类网络基础上加额外多尺度特征进行检测，应对不同尺度变化的目标。（总感觉总结有点平淡，少了东西）。这两点又是相互补充，共同协助网络完成分类到检测任务的转变。例如分类任务是检测的一环所以必不可少，但是传统分类采用的最高级特征定位细节不够，因此很多最近网络设法用类似FPN的结构去处理尺度变化的问题，就将分类较好地过渡到检测任务上了。

## 2. DetNet  
### 2.1 Motivation  
&emsp;&emsp;主要着眼点是**分辨率**，从大目标和小目标分别阐述保持分辨率的重要性（可以对照<u>图像金字塔</u>来思考，降采样降低分辨率会丢弃信息）。所以DetNet也是从分辨率的保持着手，解决多尺度物体的识别问题。

* Weak visibility of large objects  
&emsp;&emsp;网络在较深层如P6（FPN）P7（RetinaNet）大目标的边界不明确使精确定位困难。（~~还没谁说过这个问题的，因为如果认为大目标都不行，那网络没法用了，这里也只是说“weak”，至于是不是weak，不好说。况且相比于VGG , FPN的大目标下降不一定是深度导致的，和他的连接有关系~~）

* Invisibility of small objects  
&emsp;&emsp;小目标就很惨了，降采样容易丢。这个就不赘述了，所以只要避开降采样就能防止目标丢失，但是这种方法又会导致抽象能力不够 ~~，难道只有膨胀卷积一种答案？~~

### 2.2 DetNet Design  
&emsp;&emsp;保持分辨率有两个麻烦的问题：（1）内存消耗大，计算大 （2）降采样减少导致高层的抽象特征不足以很好地进行分类任务。下面设计时会同时考虑时间和高层抽象信息两点。  
&emsp;&emsp;先放出DetNet的多尺度各stage的尺寸如下图， 可以看到，相比前两种方式，DetNet在P4之后就不再进一步降采样了，进行分辨率的保持。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/DetNet/str1.png" alt="" style="width:80%" /></center>

&emsp;&emsp;实现细节如下图：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/DetNet/str2.png" alt="" style="width:80%" /></center>

* 采用的backbone是ResNet-50，改进设计了DetNet-59。
* 对bottlenecks进行了改进，传统的其实不止C，也包含两种，即将AB的膨胀卷积换成普通卷积。AB是新的基础模块。
* 为了减少分辨率保持带来的时间和内存成本消耗，通道数固定为256（思考：降采样和膨胀卷积都会有信息丢失，这里可以想想）。
* DetNet也可以加FPN结构，方法类似。



## 3. Experiments  
&emsp;&emsp;检测和训练的细节配置就不看了。

### 3.1 Main Results
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/DetNet/exp1.png" alt="" style="width:60%" /></center>

* 在FPN基础上明显有大物体涨点，同时由于高分辨率，小物体也有不错的提升。（~~但是FPN为什么不行还是没有探究~~）
* 膨胀卷积提供的大感受野使得分类也不逊色
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/DetNet/exp2.png" alt="" style="width:60%" /></center>

### 3.2 Results analysis
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/DetNet/exp3.png" alt="" style="width:60%" /></center>

* **AP指标评价很有意思**，这里的解读耐人寻味
* 从AP50看出，高好1.7；从AP80看出，高了3.7。由此可以看出确实提高了检测性能。（那么CascadeRCNN的IoU分布能不能从这个角度进行解读？这个解读可能更贴近本质）
* 从定位性能来看，大物体的提升比小物体更多。作者认为是高分辨率解决了大物体边界模糊的问题（个人认为没有说服力，加膨胀卷积确实提升了，但是到底是因为新的方法更有效利用了特征带来的提升，还是触及了FPN的弊端改变了权值更新的方式带来的提升。感觉应该是前者）。其实有一种解释：小目标没有大目标明显，因为膨胀卷积核降采样都会丢失小目标，只是膨胀卷积可能离散采样不至于像降采样直接给到后面没了，但是没有根本性的解决，所以小目标不大。至于大目标，需要进一步理解膨胀卷积的操作和意义才能理解。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/DetNet/exp4.png" alt="" style="width:60%" /></center>


* AR指标也有类似结论
* AR50体现了小目标的查全率更好，这也印证上面分析的：相对降采样，膨胀卷积丢失会好点。此下大目标效果虽然提升不大但是也很高了，作者表示DetNet擅长找到更精确的定位目标（也是检测网络设计的初衷），在AR85的高指标就能看出。
* AR85看大目标丢失少，说明能够像 VGG一样对大目标效果优良。关于小目标的效果平平，作者认为没有必要太高，因为FPN结构对小目标已经利用地很充分了，这里即使不高也没事。（~~是否暗示可以调高IoU来获得更好的检测效果？~~）



### 3.3 Discussion
* 关于stage  
&emsp;&emsp;为了研究backbone对检测的影响，首先研究stage的作用。前4个还好说，和ResNet一样，但是P5 P6就不同，没有尺度的变化，和传统意义的stage不一样了，需要重新定义。这里DetNet也是类似ResNet的方法，虽然没有尺度变化，但是AB模块的位置还是保持了，B开启一个stage（~~听上去有点牵强~~）。如下图，认为新加的仍属于P5。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/DetNet/str3.png" alt="" style="width:70%" /></center>

&emsp;&emsp;验证方法是做了实验，将P6开始的block换成上图所示的A模块对比效果如下图。 发现还是加了B效果更好。（但是这个stage和传统意义很不一样，所以很多性质不能相提并论，只是B模块的改变也不好判定什么）
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/DetNet/exp5.png" alt="" style="width:60%" /></center>

* 关于训练（没看懂想表达什么...）



## Conclusion
* 很有趣的insight：改进适合检测任务的backbone，使结构与功能相适应
* 提高了大目标定位能力和缓解了小目标丢失问题
* 一些其他理解















<br>
<br>
<hr />