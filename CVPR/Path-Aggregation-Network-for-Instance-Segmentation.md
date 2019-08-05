|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2018.5  [CVPR]<p/span>

## 1. Motivation
&emsp;&emsp;在信息自底向上流通过程中，较深的backbone会经过近百层的卷积，底层的定位信息很容易丢失；虽然后来有SSD和FPN，但是前者只是在特定的level上进行detection，后者虽然有融合，较好地改善之前严格层级检测的桎梏，但是也存在一次只在一个尺度特征图上预测的限制。（其实像DenseNet和FPN解决得已经还算可以，所以不好谈及。这里只是想强调利用底层信息的重要性，提出新的改进）
<!-- more -->
&emsp;&emsp;针对更好利用底层特征，proposals的多尺度特征图预测，提出PANet，结构如下：  
<img src="http://chaserblog.test.upcdn.net/blogs/paper/Path%20Aggregation%20Network%20for%20Instance%20Segmentation/PANet.png!//watermark/text/Y2hhc2VycyBibG9nOiBodHRwczovL21pbmc3MS5naXRodWIuaW8v/align/southeast/font/helvetica/animate/true" alt="" style="width:85%" />  

&emsp;&emsp;特点在于：  
* 在FPN基础上添加自底向上的通道融合路径  
* 自适应特征图池化，充分利用更多尺度特征进行proposals的detection，避免固定分配level带来的信息断层
* 不同方式得到的特征进行FC融合，预测出更好的mask


## 2. Related Work
Insight:
* 基于语义分割的实例分割进展也很大，主要工作是从平移变换、实例边界、RNN、图模型等
* **大范围的池化可以提供更多的上下文信息（也有使用大卷积核的全局卷积）**。全局信息可以提供更好的分割效果。


## 3. Framework  
### 3.1 Bottom-up Path Augmentation  
&emsp;&emsp;高层语义信息响应的是物体的整体信息，底层信息响应局部和问题信息；和FPN的区别在于，FPN和CNN的骨干网络都是跨很多层进行信息的重组和融合，底层信息想要传到最顶层的P5需要跨越100+层，而新的分支只需要跨越不到10层，信息流通更快，可以更加有效利用底层语义信息的定位信息便于精确分割（~~个人认为十分牵强，也许是我没看懂，如果只是因为在FPN基础上融合所以跨层少，简直是毫无逻辑的空谈，没FPN的跨层就没有这个path~~）。特点是增加局部信息的上传路径。
&emsp;&emsp;融合方式不赘述，很简答的下采样、卷积、element wise sum，然后卷积消除混叠效应

### 3.2 Adaptive Feature Pooling  
#### 3.2.1 Motivation  
&emsp;&emsp;FPN的proposal会被分配到不同特征图尺寸（level）进行分别预测，这样每个尺度的预测只用到了当前一个level的特征图信息（~~其实FPN已经融合了多尺度信息，这样说不够准确~~,应该说这种自适应的特征池化是进一步地融合和利用多尺度特征才严谨，**不过这个打破anchor固定尺度预测的思路确实是当下一个主流**，anhcor-free正火），而每个实际上有的很相似的proposal相差不到10像素，但是被分配到不同的level，是很不合理的；此外，<u>**特征图的level和信息的重要程度也不是直接相关的，这么划分依据性也不强**（这个说的挺在理，也是显然的）</u>。所以提出自适应特征池化，采用max pooling方式对所有尺度的proposal进行池化，**自适应特征池化按需要选择不同level特征图，让网络逐元素选择有用的信息。** 进一步效果见下图：    

<img src="http://chaserblog.test.upcdn.net/blogs/paper/Path%20Aggregation%20Network%20for%20Instance%20Segmentation//exp1.png" alt="" style="width:70%" />

&emsp;&emsp;Insight：上图为自适应feature pooling的结果，从中可以分析来自不同level的特征池化的因子（占比）。图中四条线是子是自适应特征池化的proposal/RoI在不同level特征图信息的的占比（~~不知道怎么分析得到的~~），横坐标代表不同的level，纵坐标是某个level下不同RoI的占比。例如蓝色线条中的RoI约0.3+0.3来自较大特征图，应该是识别小尺度的目标，属于较小的RoI。  
&emsp;&emsp;可见<u>自适应池化时，摆脱了固定分配的RoI Pooling，会倾向选择多尺度特征，而不是原来的固定尺度特征</u>。**从RoI角度诠释了对不同尺度特征的选择趋向。**

#### 3.2.2 Adaptive Feature Pooling Structure  
具体操作：
1. 先在增强的FPN上对应区域选择RoI，以灰色的为例，在N2-N5经过缩放得到各层对应该RoI的大小区域（这四个灰色阴影实际是一个proposal，只是在不同level的大小不同，从而提取了该RoI的多尺度信息）；
2. RoIAlign到固定的尺寸（如7*7）,对多尺度信息进行裁剪选择；
3. 全连接拉长；
4. element wise sum/max（实验验证其实差不多） ，得到融合向量，用于分类回归  
<img src="http://chaserblog.test.upcdn.net/blogs/paper/Path%20Aggregation%20Network%20for%20Instance%20Segmentation//stru1.png" alt="" style="width:80%" />

### 3.3 Fully-connected Fusion  

#### 3.3.1 Motivation  
&emsp;&emsp;Insigt：FCN的卷积像素级预测利用的是局部信息，而FC预测能够利用到整个proposal的全局信息，参数多，是位置敏感的，这对实例分割区分不同的物体是很有用的。为了结合卷积的局部信息和FC的全局信息特点，在mask预测时采取将两种信息进行融合的方式预测mask分支。    

<img src="http://chaserblog.test.upcdn.net/blogs/paper/Path%20Aggregation%20Network%20for%20Instance%20Segmentation//stru2.png" alt="" style="width:80%" />


#### 3.3.2Mask Prediction Structure  
&emsp;&emsp;相较传统的Mask RCNN的mask head，上面支路保持不变，依然是转置卷积后再接卷积压缩，不过这里只输出80类不含背景（Mask RCNN是81含背景）预测28x28的mask；下面支路是对conv3进行两次卷积减半深度然后FC拉成28*28=784长度的向量，reshape成28x28x1单层输出预测前景背景；至于融合的方式也不是简单的选择，而是进行两个层的像素叠加融合，作为mask输出。（推测是直接将fc的分支逐层加到FCN分支上）  
&emsp;&emsp;个人理解：这里用前景背景预测实际是MTL的一种形式，用辅助任务提高分类mask的精度，进一步将mask与classification decouple。  


## 4. Ablation Study  
<img src="http://chaserblog.test.upcdn.net/blogs/paper/Path%20Aggregation%20Network%20for%20Instance%20Segmentation//exp2.png" alt="" style="width:850%" />
&emsp;&emsp;不难看出，作者加的三个创新点（BPA , AFP , FF）各自都提升了约0.5，0.6，0.7个点；多尺度训练提升1.9个点；跨GPU训练+0.4；检测头部不压缩+0.2.（**不得不思考**，为什么技术上的三个改进加起来还不如一个多尺度训练？**猜测**：多尺度训练给网络输入提供了根本的多尺度特征，而后面的改进只是在特征提取和融合上进行重组和优化，一来这种优化还是试探性的，不能最大发挥特征的作用；二来特征组合的形式有限，而多尺度训练提供了极大的尺度变化性，所以效果更好。）

* Ablation Studies on Fusion Style  
在AFP上，sum和max效果相差无几；FF的融合上整体而言sum>max>product。 

## Conclusion
**创新点：**  
* **增强的FPN，自底向上再融合特征**
* **自适应特征融合，让网络自己选择用什么尺度的信息，而不是强制分配level**
* **mask分支的FC和FCN分支融合（类似MTL）增强mask质量**

<br>
<hr />