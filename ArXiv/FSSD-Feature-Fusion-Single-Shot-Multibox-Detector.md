|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2017.12 <p/span>

&emsp;&emsp;这篇论文行文很一般，结构组织也不是很好，比如relate work恨不得像本科毕设的绪论一样把目标检测介绍个遍，抓不住重点，没有意识到自己到底要做什么；最重要的是他在介绍relate work时没有进行评判和分析，真的只是单纯提了一下.....英语表达也比较中式。说的一些观点其实都是已经反复提到见怪不怪的，所以没有什么特别的intuition（这也正常，毕竟快2017年12月的东西），提出的这个结构有点强行改进的意思，作者也没分析这个结构work的思想和道理，不过这个角度还是值得思考。

<!-- more -->

## 1. Motivation  
&emsp;&emsp;文章自称Motivation是改进特征融合做法，也值得指出了SSD和FPN的缺点（~~没什么新颖的，都是老调重弹就不说了~~），在SSD上只用较小的速度下降换取更好的效果，号称达到SOTA。  
&emsp;&emsp;比较有意思的是，本文提到了多尺度特征的感受野限制检测精度的问题，这么早之前就有人提出了，何况还是个北航硕士，说明他之前应该可能更早就有提了。

## 2. Feature Fusion Module     

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/FSSD/str.png" alt="" style="width:90%" /></center>

&emsp;&emsp;简单来说本文工作一张图就够了。先concatenate融合各层特征到一个较大的尺度，然后在得到的特征图上构造特征金字塔。这么看来其实很像将TDM的最底层多加了几部降采样得到特征金字塔而已。  
&emsp;&emsp;思考1：这篇文章的有趣之处在于，初步有一个**base feature**的概念。像M2Det一样通过将多尺度信息暴力融合得到基础特征，在此基础上进行多尺度的构建。只不过M2Det只用了两个层就够了，这里做实验也发现concatenate下选取的层并不是越多就能提升越大，提升是有瓶颈的，这说明了骨干网络的多尺度特征实际是有很大冗余的。  
&emsp;&emsp;思考2：利用base feature构造多尺度特征，是对原来的尺度信息进行了破坏和重组，然后进行深度的构建，在深度上进行多尺度预测。这就有个问题，所谓利用多尺度信息，究竟是深度信息还是原来尺度信息？  
 

## 3. Ablation Study  

&emsp;&emsp;直接说几点结论：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/FSSD/exp.png" alt="" style="width:90%" /></center>

* 选用哪些层的信息合适？  
&emsp;&emsp;作者首先直接武断地抛弃VGG很高的层，认为信息过于抽象，其实未必准确，thunernet还直接加了个单像素的全局平均池化结果呢，STDN甚至还有1*1，至于这个抽象信息有没有用？怎样利用这个抽象信息？应该经过实验或者分析推导进行验证；  
&emsp;&emsp;从实验结果对比，345行来看，在concatenate下信息是冗余的，没必要加太多的特征图；这也<u>证明base feature进行多尺度构建的可行性</u>。   
* ele-sum还是没有concatenate有效。  
&emsp;&emsp;这里可以思考一下concatenate的作用和机制到底是怎样的，参见笔记。


## **Conclusion**
&emsp;&emsp;本文的做法没什么可圈可点之处，几个思考和自己的理解值得尝试和再思考。
* **base feature基础上构建多尺度特征的可行性和特点**
* **concatenate方法的理解**



<br>

<hr />