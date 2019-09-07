| 创建人 |                       知乎论文阅读专栏                       |              个人博客               |
| :----: | :----------------------------------------------------------: | :---------------------------------: |
| ming71 | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |

<span id="inline-blue">论文发布日期：2019  [ ICIP ]<p/span>



## 1.Introduction

航拍遥感图像中的小目标检测占据很重要的地位，本文针对遥感目标中的车辆检测提出高效快速的检测网络：simple short and shallow network (SSSDet)。

<!-- more -->

该网络相比SOTA检测器速度快四倍，浮点数运算减少4.4倍，参数少了30倍，内存占用少31倍，然而能够达到更好的检测性能。此外，还制作了一个数据集ABD，在79张遥感图像上标注了1396个目标，进行本文的实验。本结构在公共遥感数据集上都有测试，并且在精度速度计算消耗上都吊打当前的SOTA，十分嚣张。

遥感车辆检测任务的挑战之处在于：1.车辆尺寸的变化性   2.车辆密度的变化性   3.复杂背景的变化性（~~前三点好像也没啥特殊的~~） 4.航空图像的类间相似性大。



## 2.Proposed SSSDet

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/SSSDet/1.png" alt="" style="width:80%" /></center>
主要结构如上图。乍一看没什么新意，仔细一看，也好像没什么特别的地方，模型设计的介绍也不多。该**单阶段检测器**只有十个卷积层，三个maxpooling下采样层，最终输出是5776个列向量（YOLOv3三个尺度共10647个），每个列向量的内容和yolov3一样。

* 下采样数目少，保证小目标检测效果
* 卷积计算的映射参数压缩
* 输入图像的特征扩充



## 3. Experiment Result and  Discussion

* PR曲线

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/SSSDet/2.png" alt="" style="width:50%" /></center>
PR曲线来看，在多个数据集上都能与YOLOv3不相伯仲，印证了该任务上的参数缩减的正确性。有两种可能性：1.部分操作确实work   2.这个任务本身很简单，所以裁剪参数完全可以。此外该任务的尺度变化性其实是有限的，而加了FPN的YOLOv3本身能够适应更大的尺度变化，这部分本来就能省去。

* mAP

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/SSSDet/3.png" alt="" style="width:50%" /></center>
网络设计是为了进行遥感车辆检测的，但是实验这里没有说是不是只针对车辆目标检测（~~也可能是我看的太快没注意~~）。如果只是比较遥感数据集效果有点南辕北辙了，而且没有扣住主题。但是如果真的是整个数据集的效果，这个数据确实有点太好看了，有点匪夷所思？？......

<br>
<br>

<hr />