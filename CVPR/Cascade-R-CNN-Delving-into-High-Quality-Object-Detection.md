|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2017.12 [CVPR]<p/span>

## 1. Motivation 
### 1.1 问题：  
&emsp;&emsp;从IoU处找到样本划分的矛盾，并指出症结所在以解决问题：通常使用的IoU作为正负样本的划分依据，一般性地设置u=0.5，问题在于：   
（1）IoU阈值设置过低，背景太多了，不容易学习到准确的特征，容易出现较多的误检  
（2）IoU阈值设置过高，正样本被大量筛掉了导致过少，容易过拟合；  
&emsp;&emsp;而且训练采取的IoU和检测最优的IoU可能不一致，这会导致mismatch（下面实验分析这种情况）
<!-- more -->

### 1.2 实验：  
<img src="http://chaserblog.test.upcdn.net/blogs/paper/CascadeRCNN/exp1.png" alt="" style="width:70%" />  

实验分析：  
&emsp;&emsp;（c）横轴是proposal与gt的IoU，纵轴是回归后输出的bbox与gt的IoU，显然希望纵轴越大越好。分析得出结论：a.所有的IoU都比baseline好，说明输出的回归结果IoU都比输入高，选择IoU阈值筛选proposal再回归有意义 b.在选取阈值附近IoU的proposal表现性能最佳.(如u=0.5时，会筛掉小于0.5IoU的proposals，发现与gt IoU在0.5附近的proposal经过回归的效果甚至比那些具有更高IoU的proposal更好，其他阈值也是对应各自的区间，**mismatch现象**说明proposal与gt的IoU与选择阈值对应性能最好，而不是越高越好)    
**proposal IoU分布的mismatch问题是这篇paper的Insight**   
&emsp;&emsp;（d）右图曲线中u=0.7反而在0.5以下，说明一味设置高的IoU效果未必好，原因可能是高的IoU筛掉了过多的正样本，导致过拟合。  
<font color=red>**Insight:**</font>     
&emsp;&emsp;既然proposal与gt的IoU和筛选的阈值IoU比较p匹配才能获得更好的回归效果，而一次又不能设置太高，那么多设计几个检测器，分别针对proposal不同IoU进行单独检测，较低级的IoU输出效果一定比输入的IoU高（都在baseline上面），这个高IoU的作为proposal喂给下一级进行更高IoU阈值的检测。**原则是一个检测器只负责一个IoU范围的proposals筛选**。


## 2. 其他的级联结构比较


&emsp;&emsp;另外两个的结构可以说真的是一塌糊涂没什么创新和可比性，作者的这个cascade结构其实也很容易想到，只是他新颖地发现了IoU的mismatch问题，在不同stage设置不同的IoU进一步提升性能，非常独特的insight，基于观察的结果。

<img src="http://chaserblog.test.upcdn.net/blogs/paper/CascadeRCNN/str1.png" alt="" style="width:95%" />

* Iterative BBox at inference  
&emsp;&emsp;采取容易想到的级联方式，问题在于：1.不同stage的IoU阈值相同，没有解决mismatch问题  2.每个stage共用一个head，没有考虑到前一个stage输出时已经改变了输出的proposal分布  
* Integral Loss  
&emsp;&emsp;考虑了IoU不同的矛盾，所以采取多IoU设计，但是是在一个proposal stage完成的，矛盾还是存在
* Cascade R-CNN  
<img src="http://chaserblog.test.upcdn.net/blogs/paper/CascadeRCNN/exp2.png" alt="" style="width:70%" />  
&emsp;&emsp;就很好理解了，RPN+3 stage(Iou=0.5 , 0.6 , 0.7)设计不同的IoU适应不同stage的proposal分布，取得更好的检测效果。同时上图proposal分布可以看出，在RPN送到stage 1的proposal IoU集中较低水平，越往后越高，因此<u>设置递增的IoU阈值不会导致正样本减少的问题</u>。

## 3. Experiment


### 3.1 Quality Mismatch  
<img src="http://chaserblog.test.upcdn.net/blogs/paper/CascadeRCNN/exp3.png" alt="" style="width:70%" />  

&emsp;&emsp;左侧实线显示单检测器的效果而言IoU阈值越高效果反而越差，为了探究原因**在proposal中加入gt提高proposal的高IoU分布**情况，发现IoU较大的检测器提升更加明显（但是IoU不大提升不明显，应该是mismatch的原因，但是它也提升了,这是符合一开始的图片实验结果的：u=0.5-0.7情况下输入IoU在0.5-0.7区间都会提升的，太大就不一定了，比如输入Iou在0.7以上时，增长斜率小于1，说明开始劣化，可见加gt改分布也要结合设置的IoU进行match）；虚线是用多阶段级联效果，自然是提升了。
### 3.2 Number of Stages
&emsp;&emsp;经过实验发现三阶级联的效果最好，四阶会导致略微的性能退化，而计算成本反而上去了。

<br>

## Conclusion：
* **发现了<u>IoU分布和阈值设定的mismatch现象</u>：输入的proposal IoU分布需要与筛选的阈值相近才能获得较好的回归输出效果（过高会导致过拟合正样本少；过低容易误检）**
* **设计多级级联检测器，用不同的IoU递进优化检测性能。不同IoU能够很好match输出的proposal的IoU分布**
* **<u>一个trick</u>：将gt作为正样本加入proposal提高高IoU的正样本分布，从而容许了更高IoU阈值设置，得到更好的检测效果**

<br>

<hr />