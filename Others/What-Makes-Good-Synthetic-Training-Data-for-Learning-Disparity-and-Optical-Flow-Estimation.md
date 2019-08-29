|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2018 [IJCV]<p/span>

## 1. Introduction   
&emsp;&emsp;光流追踪大佬Brox的大作。最近应该是读水文章读多了...突然读到好点的文章发现够看的地方很多。   
<!-- more -->

&emsp;&emsp;作者从光流估计和视差估计的角度出发，研究了合成数据集和真实数据集的效果对照，较为深入地探讨了使数据集很好地为网络训练助力的因素所在，并提出一些训练策略和增强方法，增强数据以及让网络更好地学习到数据集的特征，达到更好的估计效果。最终还强调了，本实验得到的结论只针对光流估计和视差估计任务，对于其他任务效果有待考证。        


## 2. Synthetic Data

* 合成数据集的难点（定向数据增强的难点）：合成（增强）得到照片真实感的数据不简单，因为我们不知道现实世界的哪些方面是相关而必须建模的部分。这些问题的根源**在于我们无法很好地描述客观世界**。
* 数据增强的基本假设就是网络会把增强数据当做新的数据。理论上增强越多，效果线性增长，但是实际没有，因为重复了很多样本，针对特定的不变性不够；或者可能是学习还不够。


## 3. Experiment

### 3.1 What Makes a Good Training Dataset?    

* **Object Shape and Motion** 
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/What-Makes-Good-Synthetic-Training-Data-for-Learning-Disparity-and-Optical-Flow-Estimation/1.png" alt="" style="width:50%" /></center>

结果而言，带洞的增强基本没用，旋转涨幅最高，这和数据集本身相适应的。  
如果能知道特定目标数据集的特点，就能在针对其做出训练的调整，但是遗憾的是，这就说明没有一种泛化普适的方法来针对任意数据集都能取得很好的效果。那么类似谷歌的AutoAugment其实就可以某种程度上是泛化的，但是RNN搜索太大了不实在。

* **Textures** 
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/What-Makes-Good-Synthetic-Training-Data-for-Learning-Disparity-and-Optical-Flow-Estimation/2.png" alt="" style="width:60%" /></center>

三种纹理对比，第三个最好。这个实验的结论是显而易见的，纹理对于训练当然很重要，所以卡通数据集的纹理完全不够，导致其上训练的模型效果不好。  

* **Lighting**  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/What-Makes-Good-Synthetic-Training-Data-for-Learning-Disparity-and-Optical-Flow-Estimation/3.png" alt="" style="width:60%" /></center>

光照情况分三种：动态（考虑光照角度和阴影），静态（统一亮度变化），无阴影（没有光线和阴影的变化，只有形状特征）.结果而言，静态相对好，因为动态的变化过于复杂，没为了区分不同角度的光照，需要庞大的光照数据显然效费比低也不现实，这里样本不够所以学的效果没有那么突出。        
如果真的这么细，搜索空间太大了。哪怕简单如谷歌的RNN控制器也是10^28数量级搜索空间，过于粗暴和直接。

* **Data Augmentation**  
这里对色彩和几何的研究比较浅，直接一块做实验发现效果有提升。归纳出对于色彩变化不足的数据集，增加这样的变换可以更好地学习到色彩变换。  
总结增强两点：<u>（1）如果是和数据集目标域一致，可以有效增强尤其是小数据集的学习效果（2）如果是非目标域的增强可以学习到更广域的特征。但是有问题：1.花费权重学习这些很少的广域特征虽然增强了泛化性，但是也损失了对测试集更多出现特征的辨别能力。 2. 评价标准都是在数据集进行，这个广域输入的识别能力并不能被反映。</u>       

### 3.2 Learning Schedules with Multiple Datasets   
&emsp;&emsp;这部分主要应用课程学习的思想，上次看到“课程学习”还是在YOLOv3+。随着训练进程改变，适应性调整变化量。一开始加太多的增强可能面临难收敛，用课程学习的思想逐渐加入。也可以设置多个（至少两个）不同难度的数据集进行逐级学习。        

### 3.3 Synthesizing the Defects of Real Imaging

* “realistic的重要性被高估了”这个观点的意义在于，将图像的realistic属性变得透明化，把原来认为不可知的真实世界分解，从而能实现对这种“真实性”的可控性。
* 光照为例，就是realistic的特征，但是往往变化性不大，因此没必要去特地增强，增强带来的效益也不高，更应该着重于简单的有无而不是角度强度等复杂因素。





<br>
<br>
<hr />