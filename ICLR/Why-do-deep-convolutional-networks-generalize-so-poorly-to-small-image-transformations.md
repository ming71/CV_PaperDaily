|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2018.5.30 [ICLR]<p/span>

## 1. Introduction   
&emsp;&emsp;切入点是卷积网络对不变性的抵御能力差，并通过实验和推导分析归因这种不变性差的根源，以及初步得出解决方案。    
<!-- more -->
## 2. Failures of modern CNNs  
&emsp;&emsp;结果而言，现在的CNN结构不具有严格的不变性，即使是微不足道的像素变化也可能带来很大检测结果的差别。通过下面的图片可以看出（更详细和直观的展示在视频 https://www.youtube.com/watch?v=MpUdRacvkWk&feature=youtu.be ）。极小的像素变化会带来很大的识别率波动，因此很容易受到对抗样本的攻击。（实际上他这个实验没说用的什么网络，只介绍是modern deep convolutional neural networks，推测应该没有用一些复杂的结构和方法。yolov3为例，视频检测中这种波动虽然存在，但是也比较稳定，没几个这样跌倒谷底的精度点）
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Why-do-deep-convolutional-networks-generalize-so-poorly-to-small-image-transformations/exp1.png" alt="" style="width:100%" /></center>

## 3. Ignoring the Sampling Theorem
&emsp;&emsp;（~~这章比较头大，细节公式没看懂...~~）  
&emsp;&emsp;首先是直观分析来看，如果是完全卷积网络（不带任何非线性和降采样），其表征编码方式最终是线性的，在最后输出加一个全局平均池化后的得到分类特征，这种方式下理论上来说应该具有平移不变性，最终特征是相同的。（理解为卷积对固定特征的响应是固定的，所以平移过程响应也能很好地保留，只是位置变化了，那非线性为什么就不一样？抛开香浓定理，单看逐元素操作得到的特征应该不变才是，那最后的特征为什么不同？）
* **降采样**  
&emsp;&emsp;一般来说，降采样是为了降低参数，同时也引入了平移不变性。但是这种平移不变性是不可靠的，即只有当平移的尺度是降采样步长的整数倍时，这种不变性才能得以最好地体现（这个很好理解）。  
&emsp;&emsp;对于最终的特征图而言，如果最后的降采样步长为64，那么只有1/64x64的概率是保持严格平移不变性的（其他情况效果会衰减）（1/64x64计算：比如取最后特征图一个像素，对应原图是64x64个像素，这张图上每一个像素而言，平移后会落到其他的64x64的cell中去，有64*64种可能，其中只有落在和本cell位置一样的另一个cell，才能保证精确的平移不变性，所以概率是1/64x64。而其他点不用计算，因为只要这个点落到对应的位置，由于是平移操作，其他点的位置也都是对应的，就被固定了）  
&emsp;&emsp;所以对于严格的平移不变性很难成立，于是作者又验证了一个shiftability性质，来近似平移不变性，并且以此证明全局均值池化具有不变性（证明过程没仔细看）。并且验证了网络越深，降采样步长越大，这种平移不变性越差。如下图，同一只狗，到后面慢慢就丢了，响应位置偏的厉害。  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Why-do-deep-convolutional-networks-generalize-so-poorly-to-small-image-transformations/exp2.png" alt="" style="width:70%" /></center>

* **非线性**  
&emsp;&emsp;根据采样定理，系统包含的频率不应该大于奈奎斯特频率，而非线性会引入高频成分，造成可能的混叠效应，并且削弱不变性。

## 4. Why don't modern CNNs learn to be invariant from data?  
&emsp;&emsp;这部分内容好理解，就是论述了为什么不能指望通过数据集直接训练来获得网络的不变性。
* dataset bias：数据集的分布比较有限，存在“摄影师偏差”问题，导致数据集偏置，不能很好地帮助网络训练出不变性特征。下图展示ImageNet的Photographer's biases现象，大多属性集中呈现高斯分布。
* 通过数据集本身让网络学会不变性是不切实际的，现实世界中不可能为网络提供所有的不变性数据。同时，数据增强是很有用的，只是拟合的很充分比较难和不经济而已。对于小数据集非常有必要，而在大数据集上，这种问题就凸显出来了。（这部分叙述混乱，其实没必要分两大节，就是数据集和增强，可以分开阐述没有本质的不同）  
  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Why-do-deep-convolutional-networks-generalize-so-poorly-to-small-image-transformations/exp3.png" alt="" style="width:60%" /></center>

## 5. Quantifying partial invariance


* **photographer's biases**   
&emsp;&emsp;摄影师偏差，和数据偏置一样，描述大多数样本的高斯分布属性。严格的不变性是编码后两张图片的结果一模一样，这在实现起来是难以想象、极其困难的。所以也不用真的去着手解决严格不变性的问题：  
1. 现实捕捉到的图片，如大型数据集，都是存在摄影师偏差的，花费很大功夫去解决极少情况的严格不变性获得的回报很低  
2. 即使编码方式不能实现严格不变性，但也能减弱不变性带来的干扰影响，因此仍具有实用意义，效费比相对也较高。
​
* **小目标的不变性更难处理**  
&emsp;&emsp;很好理解，同样的1个像素位移，对于小目标而言不变性更难维持，大目标则不然。可以从这个角度思考不同尺度特征图的融合。但是人类视觉受这种影响不大，而CNN在不同尺度目标面对同样的变化会得到很大的差别。

## Conclusion  
* 降采样和非线性是导致不变性不能维持的原因
* 摄影师偏差的存在：难以拟合严格非线性，也没必要拟合严格不变性
* 从不变性的角度，计算层面来思考一些调整融合的问题。  



<br>
<br>
<hr />