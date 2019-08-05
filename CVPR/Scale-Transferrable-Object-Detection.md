|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2018 [CVPR]<p/span>


## 1. Motivation  
* 提出问题：
&emsp;&emsp;<u>提出不同尺度特征图的感受野固定的问题（和TridentNet的角度类似）</u>，相比真实世界的目标尺度变化，固定感受野特征图出现了<font color=red>**inconsistency**</font>的问题（也可以理解为特征图固定的感受野无法很好match真实多尺度变化的物体）;        
&emsp;&emsp;指出FPN类的问题：融合全部特征，时间开销大。（很多文章论述FPN缺点都会说这个，但是实际性能上问题也是有短板的，如大目标检测的tradeoff，应该还有，可以思考一下 ）（作者提到了<u>all scale的融合方式</u>，但是没有质疑这样对不对，我觉得这其中**是有问题的**！参考NAS-FPN就知道了，这个地方能做工作） 

<!-- more -->
* 个人思考：  
&emsp;&emsp;这个问题一直都是检测的难点，就目前我所了解的来看，有两种思路和路线：          
&emsp;&emsp;（1）<u>**从特征角度改进**</u>。主要是探索更好表达能力的特征，使特征表示能够应对尺度的变化，如FPN，TDM等，使不同尺度特征图本身能够更好融合多尺度特征的不同语义信息。        
&emsp;&emsp;（2）<u>**从感受野角度改进**</u>。主要着眼于解决特征图和多尺度目标的inconsistency问题，核心多是剑指anchor，如当下很火的各种anchor-free框架，或者是anchor特征选择上采取自适应融合的PANet；思路比较有趣的是TridentNet，摆脱FPN从感受野上设计多分支提取多尺度特征（至于效果...不见得那么神奇，不过思路值得赞一下）

## 2. Related work
&emsp;&emsp;这部分有两点值得关注和思考。  
* 多尺度信息融合方式讨论。
&emsp;&emsp;目前像yolov3，DenseNet，MS RCNN用的都是concat，即便是DSSD说product好，他也只是和sum比较，没做concat的实验（已经很慢了，没法concat了）。综合来看，<u>concat融合的效果肯定是最好的</u>。那么有个问题：concat引入额外的学习参数，所以效果更好，这是一种解释，那么真的只是参数多的量变吗？其中会不会有什么质变，也就是简单的参数引入背后能达到怎样的改变数据分布的效果？（当然也不用抱太大希望给，TDM能学的很多了，也没产生惊世骇俗的效果，虽然和他单层预测也许有关系）大道至简，这一点可以思考一下。

* **浅层信息不能单独使用的思考**。
&emsp;&emsp;说玄乎点，就是高层信息的抽象语义很关键，可以进行检测分类等，而浅层的信息虽然定位信息多，但是没有语义信息，导致网络还不知道这些东西到底是干什么的；所以单独用高层特征可以做一些工作，而单独用底层特征就不行了；这也印证了FPN的top-down合理性（将必要的高层信息向下传播）。我自己做的自适应实验也发现这个问题，网络会自动搜索高级特征图，而不会怎么会向下搜索。        
&emsp;&emsp;说简单点，很好理解，浅层就好使了，要那么深的网络干吗？？所以单独用浅层必然不行，高层才是正解，但是结合起来有额外好处（也有坏处，如FPN的大尺度问题）

## 3. Scale-Transferrable Detection Network 
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/STDN/net.png" alt="" style="width:100%"/></center>
&emsp;&emsp;先简单说一下网络结构。采用的是DenseNet-169作backbone，取其最后一个block作为特征组合基础；选取其中某一层的输入作为基准直接当做单位1的特征图，比他小的特征图生成方式是采用上面层的输入作max/mean pooling，比他大的特征图生成采取后面更深的特征图进行通道展平，相当于reshape的操作得到。（参加下图最后一张，倒数第二层为基准层，最高一层为reshape得到的大尺度图。）然后每个尺度进行anchor-based检测分类和回归。    

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/STDN/str1.png" alt="" style="width:70%"/></center>

### 3.1 Base Network : DenseNet    
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/STDN/str2.png" alt="" style="width:100%"/></center>

#### 3.1.1 DenseNet回顾                
&emsp;&emsp;先回顾一下DenseNet的部分知识，很有必要，之前论文看的不仔细，导致在这里重新看了半天DenseNet。明确一下几个观点： 
* growth rate：设置一个block内的每层输出通道数，本文的DenseNet-169为32，意味着每个层输出都是32通道
* block内输入：输入通道数是k0+(l-1)*growth rate，因为组合特征的方式是concat所以越往后输入通道越多。
* bottlenecks：DenseNet-169一个block内设置了32个bottlenecks，其中的1*1卷积会对输入的超厚特征图进行通道压缩为32.

#### 3.1.2 DenseNet-169                     
&emsp;&emsp;结合上面结构图，DBstage4_concat5意思是第五个bottlenecks，这里是800层，可以推断其输入叠加了前面五个层共计32x5=160，也就是说初始输入到本block的k0=800-160=640，结合论文给的结构图来看确实如此（Denseblock4的输入是640x9x9）。最后的输出是DBstage4_concat32为1664x9x9，结合了前面的32x32+660=1664.               
&emsp;&emsp;提一句，上面的网络结构图只展示了最后一个block，但是实际上前面还有其他block进行融合和压缩的。            
#### 3.1.3 改动                
&emsp;&emsp;在backbone上将原来的输入7x7/2卷积层+3x3/2最大值池化替换为3x3/2卷积+3x3/1卷积+3x3/1卷积+2x2/2均值池化。经过实验发现这样的改进效果非常好，涨点很明显。                
&emsp;&emsp;**思考和分析**：
* 作者的观点：  
&emsp;&emsp;连续的两次下采样会导致大量的信息坍缩和丢失，而在其间插入卷积进行信息重组有利于缓和这种信息丢失。<u>中心思想是网络层之间要避免过度急剧的特征压缩（参数减少）</u>。这个观点在inceptionv2也提出过。
* 我的理解：  
&emsp;&emsp;还有可能是因为输入是300x300的小尺度特征图，采用7x7大感受野可能从一开始就造成了尺度mismatch的问题。即：如果按照thundenet模型320x320输入的标准，DenseNet也应该算小模型了，所以输入自然要小，而感受野也不应该选大才对，ThunderNet就用的3x3，所以这里也许小卷积核比较合适。  
【<u>注意一个区别</u>：DenseNet内存占用大，但是参数少模型小，两者不矛盾！因为是相同的特征concat所以内存消耗大，但是参数是一样的】

#### 3.1.4 Backbone的分析                
&emsp;&emsp;选择DenseNet是STDN展开的核心。全部工作都基于<u>一个DenseNet的假设上进行：DenseNet具有很好的多尺度特征复用能力，因此可以直接选择最后一个block进行多尺度特征构建</u>。这个假设持保留意见（<u>我认为还是存在inconsistency的问题，作者并没有解决</u>），有机会再review原paper再作答（~~如果记得起来~~），但是需要明确的是，这个观点是大前提。        
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/STDN/str3.png" alt="" style="width:50%"/></center>

### 3.2 High Efficiency Scale-Transfer Module                        
&emsp;&emsp;该模块用于在denseblock内生成大尺度特征图。结合上图和最开始的网络结构图分析，以最后一层1664通道的4x特征图为例，1x特征图是直接利用denseblock内的第20个bottleneck输入：1280x9x9，那么4x特征图大小应为36x36，所以将1664分成1664/32=104份，每份一组厚度36（和R-FCN位置敏感特征图很像），该36x9x9的一份通过reshape进行调整成为1x36x36即可。            
&emsp;&emsp;其它大尺度层类似处理，而小尺度特征图就很简单了直接在对应位置进行池化降采样，得到不同尺度的特征图，然后分配anchor进行检测。可见整个过程没有数学插值运算或者转置卷积，所以不引入额外的计算开销，速度快，但是也完全依赖DenseNet本身的特征，即上面提到的前提假设。  

## 4. Experiment

&emsp;&emsp;实验没什么好说的，作者也没公布具体和ssd比的时候怎么修改的结构，所以看不出到底能差多少，毕竟傍上了DenseNet这个大腿，效果不会太差。比较值得关注的是，stem block将大尺度7x7卷积核和连续下采样换成3x3分别下采样后涨点不少，至于到底是那个原因可以试验分析一下。        
&emsp;&emsp;速度来看还可以，有单阶段检测器的样子，但是看tradeoff效果，其实并没有太多的提升，也可以说有一点平平无奇。

## Conclusion  
我是没有太get到这篇文章实质性的intuition，只说说个人的感受。  
* **提到了固定感受野特征图面对多尺度物体的inconsistency问题**（但是没解决）
* 利用DenseNet的优良性能，采用一种巧妙的方式在不增加额外计算前提下生成了大尺度的特征图。（该方法往往用于SR任务中）
* <u>**前面的个人分析总结和思考比这篇论文更重要**</u>

<br>


<hr />