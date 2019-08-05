|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2019.4.6 <p/span>


## 1. Motivation  
* 来源：COCO2018检测挑战冠军获得者发现，实例分割任务中将bbox回归与分割在同一个卷积head上学习，效果比将bbox回归与分类在同一个fc head上学习好。
* 设想：检测任务中使用不同的头部分别完成分类和回归
<!-- more -->


## 2. Double-Head

### 2.1 Motivation and Hypothesis  
&emsp;&emsp;如上所述，前提假设是全连接和卷积的head具有不同的功能偏向性。为了验证，后面设计对比实验，对比看出只选择全用fc或conv或者反用效果都不如分别对应使用效果好，从而验证了假设。

### 2.2 Network Structure  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Double-Head-RCNN/str1.png" alt="" style="width:70%" /></center>

&emsp;&emsp;安排的很明白了。值得注意一下的是b的向量拉长使用的是均值池化，这个有意思。c图是标准的Double-Head结构很简单。d图是泛化的Double-Head结构，相比之下不难看出就是在进行分类时，不仅用到了分类head的结果，也加入回归。loss设置如下，可见是融合不同分支信息，也不难看出令对应lambda=0就是两种普通head之一。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Double-Head-RCNN/for1.png" alt="" style="width:30%" /></center>

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Double-Head-RCNN/for2.png" alt="" style="width:40%" /></center>

分类器输出结果的监督信息融合规则：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Double-Head-RCNN/for3.png" alt="" style="width:50%" /></center>

&emsp;&emsp;实验中设计的backbone是FPN，具体不赘述；fc-head就是两个1024fc层；conv-head有三种设计如下，分别是残差通道增加模块、bottleneck、non-local模块，进行组合；loss很简单，就是RPN-loss+w1x fc-loss+w2 x conv-loss，w1,w2是权重。  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Double-Head-RCNN/srt2.png" alt="" style="width:70%" /></center>


## 3. Experiment

* 假设验证  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Double-Head-RCNN/exp1.png" alt="" style="width:85%" /></center>

可见单独用一种head处理两个任务是不如分别处理的好（体现了结构与功能系相适应）。

* Depth of conv-head    
结果来看，趋势是越深越好，为了权衡速度，最后选择3残差2non-local共5个模块。

* Balance Weights
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Double-Head-RCNN/exp2.png" alt="" style="width:90%" /></center>

首先看ab和cd对比，明显后面两个的整体map高，效果好，印证不同head的有效性；再看c，表示采取不同比例loss的互监督效果，右上角是不采取互监督两head完全独立，效果也不赖（相比最好的40.0差不多）；最后一张是在分类器输出采取信息融合，得到的效果更好。

* ​Fusion of Classifiers  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Double-Head-RCNN/exp3.png" alt="" style="width:90%" /></center>

权重就不说了，按照上面对比实验选的。有意思的是其实仔细看会发现这里的差别都没有特别大，硬要说大，那就是一个是融合的有无，一个是信息全局利用的方式好过max。

* ​检测质量分析  

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Double-Head-RCNN/exp4.png" alt="" style="width:60%" /></center>

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Double-Head-RCNN/exp5.png" alt="" style="width:60%" /></center>
印证假设，其实可以看看还有东西可以想。




<br>
<br>
<hr />