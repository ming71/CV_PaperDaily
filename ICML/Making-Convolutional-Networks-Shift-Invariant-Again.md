|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2019.4 [ICML]<p/span>  

## 1. Motivation    
&emsp;&emsp;问题引入和一篇ICLR2018的相同，如下图的实验，对不变性进行研究和解决。并紧接着提出的不变性问题提出解决方案，下图是效果，可见整体表现非常平滑，有效地缓解了分类问题中的平移不变性问题。（实验为了对比效果明显，采用的分类网络都比较简单，一方面确实能体现这种方法的优良性能，但是在比较新的网络上是否有这么大的提升就令人怀疑，可能其他方法在网络结构上能够对不变性进行补偿，这一点论文没有实验，尚不明确）
<!-- more -->
&emsp;&emsp;前文提出降采样步骤会引入高频信号导致信号成分中存在大于奈奎斯特频率的部分从而导致无法还原信息，实际上也就是混叠效应（~~多尺度融合那个怎么能理解成混叠？？~~）。常见的抗混叠方法是在降采样之前加一个低通滤波器，滤除信号中大于奈奎斯特频率的成分，但是这种操作往往导致性能劣化（可能是滤掉太多信息导致失真），所以现在不常用。
&emsp;&emsp;本文通过合理安排抗混叠滤波器的位置和频率，在不去掉最大值池化等的前提下降低混叠效应，使网络具有不变性特点。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Making-Convolutional-Networks-Shift-Invariant-Again/exp1.png" alt="" style="width:80%" /></center>


## 2. Methods  
### 2.1. Preliminaries
* Shift-equivariance and invariance：描述平移不变性和平移变化性。
* Periodic-N shift-equivariance/invariance：描述周期平移不变性和变化性，例如固定步长降采样对应的严格不变性。
* Circular convolution and shifting：出于卷积边缘的考虑，一般处理边缘时，为了保证输出尺度的不变，会采用padding方式填充，但是循环卷积(圆周卷积)是回到始边（也可以理解为用始边填充）；对其不变性的分析也是类似的，如果平移到了边沿，也是填充到始边方向；似乎是很有道理，不过作者这里自己承认实际应用的意义不大，padding就够了....这里使用循环卷积只是为了保证数据全部来源于原始特征 ，以便排除padding的干扰，进行更好的测试

### 2.2. Anti-aliasing to improve shift-equivariance
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Making-Convolutional-Networks-Shift-Invariant-Again/str.png" alt="" style="width:80%" /></center>

&emsp;&emsp;主要思想是比较巧妙地将最大值池化进行了分解：（1）密集最大值选择（2）降采样。然后在两者中间插入一个低通滤波器（二维图像就是卷积运算）。其中，第一步的Max操作是通过密集滑窗进行的，因此具有平移变化性，而后面的降采样不具备平移敏感性。  
&emsp;&emsp;为了便于理解作者还用一维向量的一步平移进行举例论述这种操作具体怎么进行的，挺简单的随便算算就能了解具体的操作，见论文注释。最后可见作者的功夫确实做得很足，为了这个实验进行评测，还提出一种比较科学的指标进行衡量，甚至考虑的padding的影响；最后的附录实验也很细致，甚至还尝试让网络学习低通滤波器参数的设置。
&emsp;&emsp;不是很感兴趣，就不细看了。

<br>
<br>
<hr />