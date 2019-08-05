|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2017.12.4 <p/span>


## 1. Introduction
本文工作大致有：
* 研究对比了各种增强方式的有效性和性能
* 尝试用GAN生成不同风格的网络的效果
* 提出一种让网络自身去学习提高分类性能的方法，并通过其在不同的数据集上的表现优劣进行讨论。
<!-- more -->

## 2. Method
&emsp;&emsp;作者提供尝试了两种思路：a.直接进行常规增强和GAN生成，获取更大的数据集喂给网络 b.通过网络学习增强策略。目的也：不是为了获得最佳性能的分类器（~~也可能是他实验不够或者发现提不上去懒得提了~~），而是为了探究增强策略是如何降低过拟合和提高分类精度，帮助网络更快收敛。       

### 2.1  Traditional Transformations  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/The-Effectiveness-of-Data-Augmentation-in-Image-Classification-using-Deep-Learning/1.png" alt="" style="width:60%" /></center>

就是使用常规的仿射、色彩、翻转等变换，将数据集大小从N扩充为2N。  

### 2.2  Generative Adversarial Networks  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/The-Effectiveness-of-Data-Augmentation-in-Image-Classification-using-Deep-Learning/2.png" alt="" style="width:60%" /></center>

使用GAN生成数据的6种不同风格。        

### 2.3  Learning the Augmentation    
整个网络包含两部分：增强子网络和分类子网络。loss包含两部分：分类网络的交叉熵损失；增强网络的loss——用于衡量输入两张图像的相似度。


## 3. Experiment  
* **SmallNet**  
&emsp;&emsp;输入两张图片，进行concatenate得到两倍通道图，然后学习压缩得到输出三通道的同尺寸图像（灰度图的话输入2通道输出1通道）。采用的loss是将输出的图像与同类内的另一张进行比较得到，最小化loss的过程是使得输出增强图尽可能与第三张图像类似。这样做的好处也是数据集为N时，能够通过学习组合得到NXN的更大数据集，而且学习后的效果会更好。        
&emsp;&emsp;训练时通过两个loss反向传播共同指导分类和增强网络；在inference阶段去掉增强网络。示意图如下。        
&emsp;&emsp;~~【这个思路方法和目的与Smart Augmentation一模一样，都是2017挂出来的，两个人中可能有一个是抄的，或者刚好都想到这个吧，但是相互都没引用】~~。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/The-Effectiveness-of-Data-Augmentation-in-Image-Classification-using-Deep-Learning/3.png" alt="" style="width:60%" /></center>


* **loss设计**  
大体上有两个loss：增强和分类。其中针对增强网络的损失，有三种方式。如下介绍，两个loss通过系数alpha和beta进行权衡。    

（1）content loss     
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/The-Effectiveness-of-Data-Augmentation-in-Image-Classification-using-Deep-Learning/4.png" alt="" style="width:25%" /></center> 

&emsp;&emsp;输出图像和随机选择的目标图像的像素均方差。上式中D是图像的宽高（正方形），A是增强图，T是目标图。           

（2）style loss         
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/The-Effectiveness-of-Data-Augmentation-in-Image-Classification-using-Deep-Learning/5.png" alt="" style="width:20%" /></center>
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/The-Effectiveness-of-Data-Augmentation-in-Image-Classification-using-Deep-Learning/6.png" alt="" style="width:25%" /></center>

形式和上面类似，但是计算的不再是像素内容而是Gram矩阵的content loss（在风格迁移中经常用到Gram矩阵）。图中C通道数，F是特征图。  
（3）no loss        
一个对比实验，即不加增强网络的loss。


## 4. Result
* 效果对比
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/The-Effectiveness-of-Data-Augmentation-in-Image-Classification-using-Deep-Learning/7.png" alt="" style="width:45%" /></center>
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/The-Effectiveness-of-Data-Augmentation-in-Image-Classification-using-Deep-Learning/8.png" alt="" style="width:45%" /></center>

&emsp;&emsp;control实验中，效果反而不如未增强的。作者总结的原因可能是：1.调参不好，如学习率，所以没取得较好的效果  2.加入的增强网络过于复杂但数据量不够，反而欠拟合，没有达到很好的学习效果。 
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/The-Effectiveness-of-Data-Augmentation-in-Image-Classification-using-Deep-Learning/9.png" alt="" style="width:45%" /></center>


&emsp;&emsp;在MNIST上没明显的提升作用，原因分析：1.简单的CNN分类就足够，再加增强网络没有助益  2.数字特征足够简单，所以学习增强结合特征实际上也不会加入新的信息        
&emsp;&emsp;从loss角度出发，没loss甚至perform better。也很简单。

* 增强网络的输出
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/The-Effectiveness-of-Data-Augmentation-in-Image-Classification-using-Deep-Learning/10.png" alt="" style="width:55%" /></center>
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/The-Effectiveness-of-Data-Augmentation-in-Image-Classification-using-Deep-Learning/11.png" alt="" style="width:55%" /></center>

&emsp;&emsp;有视觉含义的和无视觉含义的。其实可以看出增强网络实际是以一种CNN的方式学习增强，提取他想要的特征，所以学到的东西可视化效果一般也是可以理解的，~~这个才是小网络该学到的东西，Smart Augmentation的网络更小，而且还加了池化，学到的合成图像精确无比！很难让人不怀疑它的输出是不是用GAN做的假结果。~~        


<br>
<br>
<hr />