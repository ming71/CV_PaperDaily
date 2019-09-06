| 创建人 |                       知乎论文阅读专栏                       |              个人博客               |
| :----: | :----------------------------------------------------------: | :---------------------------------: |
| ming71 | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |



<span id="inline-blue">论文发布日期：2016 [ECCV]<p/span>


## 1. Introduction
&emsp;&emsp;强调人类视觉中的自顶向上信息（？）和反馈机制，并且将之融入CNN改进Faster R-CNN结构。做出的工作有：
* 通过语义分割网络强化Faster R-CNN的检测效果（很多人做过，这个还是直接用的分割标注，实用性很弱）
* 自顶向下上下文分割
* 通过分割提供自顶向下的迭代反馈
<!-- more -->


## 2. Our Approach

### 2.1  Augmenting Faster R-CNN with Segmentation  

&emsp;&emsp;首先通过额外的分割信息来加强检测性能。要求有三点：速度快、便于整合进网络、少用或不用后处理以便于和网络进行端到端的训练。为了模型的轻量化，选用的分割子网络是ParseNet架构，网络结构图如下。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Contextual-Priming-and-Feedback-for-Faster-R-CNN/1.png" alt="" style="width:60%" /></center>
### 2.2  Contextual Priming via Segmentation
&emsp;&emsp;出发点是分割能够更好地帮助检测器聚焦于有物体的部分上下文。采用的做法十分简单，直接将分割结果池化降采样后分别加到RPN和检测头的特征图上去。~~这样操作显然效果是存疑的,没有说服力~~
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Contextual-Priming-and-Feedback-for-Faster-R-CNN/2.png" alt="" style="width:60%" /></center>
### 2.3  Iterative Feedback via Segmentation
&emsp;&emsp;上面的方法只是将分割结果加到两个地方，作者认为还不够——要在每层特征图上都加这样的信息加强监督。给的理由是能够帮助底层特征图更好地定位物体。
&emsp;&emsp;做法就很自然地将分割结果处理后较大特检测模型的每个stage开始阶段，然后重新进行预测，方法比较笨重，具体处理方法无关紧要（也可以关注一下，毕竟能让这个模型work需要将两个网络的很多细节进行处理）。这个方式有问题的，甚至可能后面的修补迭代是因为发现这个idea不work才做的。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Contextual-Priming-and-Feedback-for-Faster-R-CNN/3.png" alt="" style="width:60%" /></center>
### 2.4  Joint Model
&emsp;&emsp;综合上面两个组件就是联合训练的模型，模型体量比较大，不够简单。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Contextual-Priming-and-Feedback-for-Faster-R-CNN/4.png" alt="" style="width:60%" /></center>

## 3. Result
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Contextual-Priming-and-Feedback-for-Faster-R-CNN/5.png" alt="" style="width:50%" /></center>
&emsp;&emsp;效果没什么好看的，关注一下ablation study。发现和想象的不太一样，加上分割特征之后效果涨点还是比每层都加的 效果更好的。如果变量控制地好的话，说明这部分信息的有无比添加的强度更加重要。但是另一方面来看，虽然在多层上都添加了，方式仍然是一样的。但是这也不全是坏事，因为如果分割特征太强反而不好，可以参考其他工作。













<br>
<br>

<hr />