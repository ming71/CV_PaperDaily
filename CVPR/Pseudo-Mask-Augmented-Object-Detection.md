|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2018.3.15 [CVPR] <p/span>






## 1. Introduction
* 核心思想：通过检测目标的mask辅助检测任务的完成
* PAD方法：
  * 将目标mask作为未知的隐变量，并且只用bbox标注，生成pseudo ground truth masks
  * 分割和检测是不同的分支，但是两者具有共通性，因此将分割的信息自顶向下加到检测中去，辅助检测任务
<!-- more -->

## 2. Pseudo-mask Augmented Detection
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Pseudo-Mask-Augmented-Object-Detection/1.png" alt="" style="width:70%" /></center>

&emsp;&emsp;大致检测流程：分为检测和分割两个子网络，两个网络在backbone部分是参数共享的。其中分割子网络的全局位敏信息以及实例信息分别反馈到检测网络的相应部分来丰富检测特征；分割子网络的mask是通过bbox的gt进行学习和图分割推理得到的。算法流程如下：      
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Pseudo-Mask-Augmented-Object-Detection/2.png" alt="" style="width:60%" /></center>

&emsp;&emsp;实际上实验中选择的是三迭代效果如下：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Pseudo-Mask-Augmented-Object-Detection/3.png" alt="" style="width:70%" /></center>

&emsp;&emsp;损失函数分为两部分：检测和分割。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Pseudo-Mask-Augmented-Object-Detection/4.png" alt="" style="width:50%" /></center>

### 2.1  Network Architecture
* **Object Segmentation with Pseudo Masks**    
&emsp;&emsp;分割子网络实际采取任意实例分割的网络都可以，这里借鉴的Instance-FCN和FCIS，并且将位敏特征图乘以C像R-FCN一样能够预测不同的类别。训练时，在每个类的层后面加sigmoid层生成概率图。

* **Segmentation Loss** 
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Pseudo-Mask-Augmented-Object-Detection/5.png" alt="" style="width:50%" /></center>

&emsp;&emsp;无监督的mask最大的问题就是不准，直接拿去用很可能使网络不收敛。对于传统mask交叉熵损失（这里称为2D-loss），定义mask与bbox的IoU在0.5以上的mask与上一轮迭代得到的mask作为gt计算交叉熵；除此之外，还额外引入约束--两个一维向量（1D-loss）。        
&emsp;&emsp;一维向量loss的思路来自LocNet，先看看LocNet怎么做的。如下图，首先将得到的proposal放大得到更大的region，在region内执行行列搜索，得到边界响应的概率图，从而实现box的定位。响应方式有两种：响应边界和响应区域。以下图为例，右边是行搜索，得到长度为h的1-d vector，其中响应最大的区域是bbox内；列搜索类似。        
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Pseudo-Mask-Augmented-Object-Detection/6.png" alt="" style="width:50%" /></center>

&emsp;&emsp;这里的分割1D loss采用相同的思想，对前景mask的RoI预测输出两个一维向量，分别是行列搜索的结果，加上sigmoid转化成概率图然后找到目标区域与bbox gt计算损失。（**直接用mask像素来判断目标位置不就完事了？这花里胡哨的有什么好处？反正mask都是不准的，这样再多一层回归更不准了才是。**）

* **Object Detection with Top-down Segmentation Feedback**  
&emsp;&emsp;增强两个任务的相关性，信息复用进行反馈。其实就是将分割子网络的对应信息和检测网络相加。至于检测的loss就很简单，直接是分类和回归的loss相加。        

### 2.2  Pseudo Mask Refinement Accurate            
&emsp;&emsp;这部分涉及GraphCut知识，不太清楚。大致流程还是定义一个目标函数，其中一元项指定像素和bbox的关系，二元项统计颜色直方图和梯度直方图从色彩和纹理寻找相似性来度量，最终通过学习最小化目标函数实现寻优。

## 3. Experiment
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Pseudo-Mask-Augmented-Object-Detection/7.png" alt="" style="width:70%" /></center>

&emsp;&emsp;实验效果自然是有的，不过没什么特别大的亮点。比较有意思的是没和MaskRCNN比，可能是理论上是比不过，而且实际而言差距还不小吧。关于推理速度，时间耗费大，显而易见的，因为它额外加了个分割网络，而且相比Mask RCNN的轻量化的分割头而言，这个还是类似R-FCN的位敏图，一定更慢。但是作者避开了这个不谈，只说加了融合连接的参数多了，所以影响不大，不知道为什么。




<br>
<br>
<hr />