|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2016 [DICTA]<p/span>


## 1. Introduction

&emsp;&emsp;本文旨在研究如何使用数据增强以及什么时候增强效果好。落实到实际就是在特征域和数据域使用数据增强哪个更好，以及做实验研究了CNN，SVM，ELM的性能差异。  
&emsp;&emsp;得到的结论是：尽管特征空间的增强可行，但无论是性能还是过拟合，数据输入端的增强更好。(~~ICLR那个特征增强发workshop的也太尴尬了吧，估计人家作者也没看过这篇早期论文~~)

<!-- more -->

## 2. Method 

* Related work  
作者这里和之前的光流法Brox大佬文章定义的不同，将增强的数据称作synthetic data，认为这个不是真实数据了，并且举例了一些论文指出synthetic data和real data之间存在synthetic  gap会影响分类性能。  
这里有两点总结：  
（1）synthetic  gap实际是数据集差异，增强数据有gap可能是使用了本没有的变化或其组合导致的（如对车辆检测使用奇怪的扭曲增强）。  
（2）由（1）知。safe augment还是有用的。    
为了减小合成数据集和真实图像之间的synthetic gap，使用稀疏自编码器在两者之间进行训练学习，拉近距离。这和smartaugment的增强其实差不多，只是工具不同。

* Experiment
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Understanding-data-augmentation-for-classification-when-to-warp/1.png" alt="" style="width:60%" /></center>

&emsp;&emsp;选用的数据集是MNIST作者的大致思路就是使用上面的网络结构，分别在数据域和特征域进行增强观察输出的结果。评价指标是分类任务常用的错误率以及过拟合情况（test和train的acc差距来衡量），得到若干组如下的图片进行试验分析。这几个方法用得少，感觉实验比较简单也没什么深入看的意义。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Understanding-data-augmentation-for-classification-when-to-warp/2.png" alt="" style="width:40%" /></center>


## 3. Discussion  

* 数据空间和特征空间：  
数据空间有最为丰富的信息，可以容纳各种变化性提供给神经内网络进行训练。而特征空间由于进行了再组合，语义不明给增强带来困难。这里的增强方法的可解释性不强，效果也很有限（即使是ICLR的workshop那篇文章效果也一般，就像图像金字塔效果会更好一样，在无法理解内部结构的前提下，尽可能保留更多特征也许效果更好）。

* 测试集的增强
直接参考safe augment的观点就行，可以看出这个是可行的。但是对于大数据集而言一般分布不会差太远，尤其是理论实验时会随机划分，所以相对于成本而言，效果不会令人青睐。
当然，很多实际问题往往不是这样，数据集可能比较dirty，这对于只知道拟合的神经网络来说就很糟糕了，需要提高其更广域上的不变性学习能力。测试集增强也无法解决这个问题，只是不会进一步扩大这种呢mismatch。​













<br>
<br>
<hr />