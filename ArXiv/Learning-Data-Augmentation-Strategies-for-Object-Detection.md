|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2019.6.11<p/span>

## 1. Motivation  
&emsp;&emsp;作者的切入点是当前的数据增强都是从分类任务引入的，但是分类任务的增强不一定适用检测任务。因为回归检测任务的样本远不如分类多，所以需要进一步的增强。我理解的是，作者所说的不匹配性无非就是指检测的数据增强形式应该更加丰富多样，而不是简单的几个翻转平移仿射啥的。（~~很简单的一句话的事情google的这个团队非搞得很玄乎一样，出发点弄得可高了，虽然在abstract说后面会demonstrate然而并没有，后面introduction也是直接给的结果~~）

<!-- more -->

&emsp;&emsp;提出办法是采用可学习的数据增强方式。（*其实这篇文章之前已经有过一篇很类似的自学习数据增强方法了，见其参考文献5*）

## 2. Methods 
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Learning-Data-Augmentation-Strategies-for-Object-Detection/exp1.png" alt="" style="width:60%" /></center>

* **定义**  
（1）变换方式        
&emsp;&emsp;共计从色彩变换、图像几何变换、bbox内变换三个角度设计了22种不同的变换方式。每种变换方式定义三个参数：（变换标识符，概率，强度）  
（2）操作子集和搜索空间         
&emsp;&emsp;定义的操作空间子集，每个子集由两个变换操作组成。同时将概率分为M段，强度M段，结合22种变换，总共操作空间数目为(22xLxM)^2个，实验取L=M=6，每次五子集，共计(22x6x6)^(2x5)=9.6x10^28种搜索结果。

* **实现**  
&emsp;&emsp;搜索空间过于巨大，所以采用强化学习和辅助RNN控制器进行结构搜索，和NAS的方法类似，最近谷歌带的节奏都用这个，感觉很有必要看一下相关东西了。

## 3. Experiment
&emsp;&emsp;这些实验可以部分地看作是数据增强的影响。  

* 不同的backbone  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Learning-Data-Augmentation-Strategies-for-Object-Detection/exp2.png" alt="" style="width:50%" /></center>

&emsp;&emsp;数据增强效果提升自然不在话下。但是越好的backbone越能提取好的特征，过多的增强效果并没有提升更明显，说明：       
（1）以后的轻量化小模型特征提取能力必然有限，所以数据增强还是利器  
（2）特征提取和数据增强从两个不同端同时影响效果，如果特征提取足够好，数据增强的效果可以在不那么庞大的样本达到理论很小，摆脱严重的数据驱动依赖。这个就远了。

* 不同增强策略  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Learning-Data-Augmentation-Strategies-for-Object-Detection/exp3.png" alt="" style="width:50%" /></center>
&emsp;&emsp;很好想，bbox变换破坏了图像的整体性画蛇添足，也和自然界中的图像不符，没有必要。  

* 不同大小样本和尺度
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Learning-Data-Augmentation-Strategies-for-Object-Detection/exp4.png" alt="" style="width:40%" /></center>

（1）样本越多，增强效果涨幅越小。这个很好想。  
（2）增强对小目标的检测效果更好。为什么？首先，小物体难检测有backbone的原因，降采样等，但是这里只对比增强与否，这一点排除；推测是COCO小目标最多大概40%+，所以一旦数据集加大，必然会进一步拉开不同尺度训练目标的绝对差距，小目标绝对占优；但是如果大小目标都足够多了达到了算法本身瓶颈，那么两者涨幅都会越来越小，此时受制于算法了。而在小样本时，小目标本来就不好检测样本还不够，因此增强的增幅更大。  
&emsp;&emsp;这里容易有一个误区：小样本既然数目最多，是不是就相当于横轴的右端，那么是不是相对大目标而言增强带来的涨幅相对小，和纵向的小目标涨幅一直凌驾在上方有矛盾？其实不是，横轴的样本是针对整个数据集而言的，因此模型需要拟合大小不同尺度目标，带来性能的折中才具有可比性，如果把大小目标当成单独的数据集割裂来看，就不能用一个模型去评价，这个对比就没有意义了。  

* 迁移到其他数据集 
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Learning-Data-Augmentation-Strategies-for-Object-Detection/exp5.png" alt="" style="width:80%" /></center>

&emsp;&emsp;存疑。这个迁移不就是平平无奇的数据增强吗？涨点是当然的啊，不对比一下手工设置的，怎么知道迁移效果真的就是会好？

* 不同AP结果    
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Learning-Data-Augmentation-Strategies-for-Object-Detection/exp6.png" alt="" style="width:40%" /></center>

&emsp;&emsp;结论：高IoU阈值的结果（AP75）涨幅更明显，高质量检测框更多。        
&emsp;&emsp;分析：因为模型学得更好了，所以AP都会上去。但是AP50不比AP75涨幅多，推测是AP75涨幅来自于很多原来AP60这些，这部分物体大部分都是正确的样本只是框的不准，所以增强后提升明显；而AP50涨幅来自于低于50的，这部分本身有一小部分就是误检所以框不好，学完会踢掉一些，导致整体上升不如AP75明显。    

* 与其他方法并用  
&emsp;&emsp;比较有趣的是，作者自称将自学习数据增强与mixup等其他方法并用，发现你效果没有提升或者性能下降了，这就不懂了。




<br>
<br>
<hr />
