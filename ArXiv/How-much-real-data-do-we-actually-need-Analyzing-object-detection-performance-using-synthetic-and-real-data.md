|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2019.7.7<p/span>

## 1. Introduction

* 背景和出发点  
探究在自动驾驶任务上的合成数据集和真实数据集性能差异，探究如何最小成本地利用少量真实数据集达到较好的效果。

* 方法和结论  
通过大量的实验，论证了采用合成数据训练并在real data上进行fine-tune的效果很好，效费比高。但是这个并不是关注的焦点，更关注对不同数据集偏置的分析，怎样的数据集才是更好的，以及实验中的一些方法和结论分析。
<!-- more -->
## 2. Related works  

&emsp;&emsp;数据集的性能不只表现在其大小。还有多样性、完整性、表现、物体出现的分布等因素。本文的只选取了一个角度：不同（合成与真实）数据集的数量（比重）。

* Real data augmentation   
这种方式其实最为常见，在真实帧上进行处理，增强物体或者其变换性，得到的结果也自然是real data，因为其环境物体等都是真实的。

* Synthetic data generation through simulation  
以Virtual KITTI和Synscapes为例，有很多的虚拟数据集，通过虚拟游戏引擎的环境建模等方式进行虚拟数据建模。尽管可以省去人工标注的麻烦，但是研究发现似乎在真实数据集上进行fine-tune效果更好。  
虚拟数据集囿于其建模方式，变化性可能会单调（~~只看过一篇合成数据集的论文，所以不敢断言~~），但是也有其独特的便利性和特殊性，尽管有些是没用的，下图很明显看出，合成数据可以提供一定的相似性  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/How-much-real-data-do-we-actually-need/1.png" alt="" style="width:70%" /></center>

* 域适应  
和这个的想法有点类似。本文不采取这种做法，而是纯粹从数据集的角度出发进行比较。


## 3. Experiment

### 3.1 实验手法和安排的借鉴  
* 选用car,person作为两类基本对象，在不同的数据集上进行测试。原因是具有可比性和一致性。  
* 作者舍掉了占图像面积4%以下的小目标，一是因为他们大多数情况这些目标不会出现在特征图上，而且带来很高的loss，给性能的比较带来困难；二是小目标说明目标很远，一般不被优先考虑，因此可以不作为重点。  
 

### 3.2 实验结果  
* 合成和真实数据集的不同比例效果    
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/How-much-real-data-do-we-actually-need/2.png" alt="" style="width:60%" /></center>


&emsp;&emsp;(1) real data占比越大，曲线都是往右上方走的，说明性能越好          
&emsp;&emsp;(2) 真实数据减少变化上，100%到10%减少带来的效果下降比10%-5%的小，反过来看，说明real-data数据越少的时候real-data的增强很有用，这和普遍的数据增强方法是一致的。（同时，5%-2.5%带来的减幅不大，应该是数据过少整个都欠拟合导致的）         
&emsp;&emsp;(3) 数据增强/数据集的容量增大，带来的效果并不线性加增，而是在衰减。          


* 数据集相似性  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/How-much-real-data-do-we-actually-need/3.png" alt="" style="width:60%" /></center>

&emsp;&emsp;推测是因为不同的虚拟数据集在实现环境建模和多样性时采取的方法是各不相同的，偏置不同，所以数据集直接迁移效果差。而真实数据集这个问题相对不大。        
&emsp;&emsp;（1）由于实际上训练的模型都是要迁移的，所以类似域适应、one-shot、few-shot这样的研究还是很有必要        
&emsp;&emsp;（2）工业界更注重效果，所以从数据集本身出发就很重要了，抓住数据集分布的同一性进行构造。



<br>
<br>
<hr />