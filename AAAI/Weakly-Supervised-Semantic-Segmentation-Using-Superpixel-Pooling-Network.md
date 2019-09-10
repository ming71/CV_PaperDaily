

| 创建人 |                       知乎论文阅读专栏                       |              个人博客               |
| :----: | :----------------------------------------------------------: | :---------------------------------: |
| ming71 | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |

<span id="inline-blue">论文发布日期：2017 [AAAI]<p/span>

## 1. Introduction
&emsp;&emsp;通过类别标注的标签实现弱监督语义分割的方法。该方法在语义分割mask生成和使用生成mask学习分割生成网络之间反复交替（和GraphCut分割的那个论文有点像）。提出了Superpixel Pooling Network (SPN)，将输入图像的超像素分割结果作为低阶结构的表征，辅助语义分割的推断。

<!-- more -->


## 2. Superpixel Pooling Network
### 2.1 Architecture
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Semantic-Segmentation-Using-Superpixel-Pooling-Network/1.png" alt="" style="width:70%" /></center>
* **Backbone**  
特征提取的部分是全卷积网络，选用VGG-16的卷积部分。参数是ImageNet预训练的，其后加上一个额外的卷积层finetune。  

* **Upsample**  
基础输出的特征会有分辨率过低的问题。这里非线性上采样参数trained from scratch.  

* **SP Layer + GAP/FC/cls**  
输入是上采样的特征图和超像素结果。将上采样的特征图按照超像素划分进行均值池化，输出NxK矩阵，N代表超像素个数K是通道数（如512）；后面会在超像素方向均值池化得到1xK的向量，送入全连接局进行分类和训练。  


### 2.2 Forward and Backward Propagations via SP Layer
&emsp;&emsp;git渲染latex公式有问题，不打公式，直接参考[SP Layer的前向传播](https://zhuanlan.zhihu.com/p/27911739)中的定义。  
公式之一：  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Semantic-Segmentation-Using-Superpixel-Pooling-Network/2.png" alt="" style="width:30%" /></center>
&emsp;&emsp;固定当前某一个超像素，将之池化。式中的r存在是因为存在尺度差异，是感受野大小。作者的池化方式是固定超像素，遍历特征图和超像素内的元素，计算当前超像素元素在位置j感受野所占的比例，然后加权上去。  
公式之二：  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Semantic-Segmentation-Using-Superpixel-Pooling-Network/3.png" alt="" style="width:30%" /></center>
就是将上面的结果再遍历超像素进行一次pooling。  

### 2.3 Learning SPN
* **Loss function**  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Semantic-Segmentation-Using-Superpixel-Pooling-Network/4.png" alt="" style="width:40%" /></center>
&emsp;&emsp;SPN采用分类标注和学习，C是类别数，${f_c (x)}$和$y_c\in(0,1)$分别是对于特定类的特征图输出和label标注；其中SPN网络的输出有两个数值的，在loss中同等计算。

* **Multi-scale learning**  
&emsp;&emsp;为了适应尺度变化性，mini-batch的缩放尺度随机为${250^2,300^2,350^2,400^2,450^2,500^2}$

### 2.4 Generating Initial Annotations with SPN
* **Superpixel-pooled class activation map**  
&emsp;&emsp;在inference阶段时，将从SP层输出的每个超像素的特征向量后，计算出每个超像素的各类的分类得分，最后得到WxHxC的张量，其中通道方向是类别数。（每个类别得分图分割的最小单元就是成片的超像素）    
&emsp;&emsp;在训练的时候，这张图会被缩放成上面的六个尺寸，分别计算不同尺度下的响应张量，然后取max pooling得到的图就是Superpixel-Pooled Class Activation Map (SP-CAM)。  

* **Generating initial annotation with SP-CAM**  
&emsp;&emsp;有了上述SP-CAM直接将响应强度低于0.5的位置置0得到的mask就是初始分割。如下图所示：  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Semantic-Segmentation-Using-Superpixel-Pooling-Network/5.png" alt="" style="width:40%" /></center>
## 3. Iterative Learning of DecoupledNet
&emsp;&emsp;进一步分割采用的网络是DecoupledNet，包含分类和分割两个分支。其中分类通过标注监督训练；分割的gt采用SPN生成的mask监督。为了避免噪声导致的不完整分割mask影响结果，所选择进行训练的mask子集特点是**the degree of scatter**比较小。  

## 4. Experiments  
* **the degree of scatter**  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Semantic-Segmentation-Using-Superpixel-Pooling-Network/6.png" alt="" style="width:60%" /></center>
&emsp;&emsp;可以看出：（1）训练多轮效果更好，不在话下  （2）选择优质分割mask确实有好处。

* **Comparison to other methods**  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Semantic-Segmentation-Using-Superpixel-Pooling-Network/7.png" alt="" style="width:50%" /></center>
&emsp;&emsp;从性能来看是吊打之前提出的如基于EM算法等的弱监督分割。和用到gt的相比还是有一定的差距，但是作为无监督来说已经很不容易了。下面是部分结果：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Semantic-Segmentation-Using-Superpixel-Pooling-Network/8.png" alt="" style="width:40%" /></center>
&emsp;&emsp;虽然效果看上去还可以，但是展示的大多都是些比较容易区分的案例。稍微难一点的如摩托车，还是会有边缘的问题。可能是输入的分割就比较粗糙，如果细致，虽然超像素边缘保留好，额外的时间成本加上后就太大了。


<br>
<br>

<hr />