| 创建人 |                       知乎论文阅读专栏                       |              个人博客               |
| :----: | :----------------------------------------------------------: | :---------------------------------: |
| ming71 | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |

<span id="inline-blue">论文发布日期：2019 [CVPR]<p/span>


## 1. Introduction

&emsp;&emsp;使用分类标注作为弱监督信息，在CAM提取到特征的基础上，进一步设计IRNet学习额外的特征约束，从而到达更好的弱监督实例分割效果。
<!-- more -->

* CAM用于分割任务的问题
  * CAM响应不准确，对于分割任务是很不利的
  * CAM无法区分实例个体
* IRNet  
组成为两部分：（1）不分类别的实例响应图  （2）pairwise semantic affinitie。其中通过不分类别的实例响应图和CAM结合，约束后得到instance-wise CAMS；另一个分支预先预测物体的边界然后得到pairwise semantic affinitie（关于这个的论文参考Related Work的对应部分，有相应的方法，~~暂不深究~~）进行融合和处理得到最终的分割。整体流程如下：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Learning-of-Instance-Segmentation-with-Inter-pixel-Relations/1.png" alt="" style="width:70%" /></center>


## 2. Related Work
* Weakly Supervised Semantic Segmentation
* Weakly Supervised Instance Segmentation
* Pixel-wise Prediction of Instance Location
* Semantic Affinities Between Pixels


## 3. Class Attention Maps  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Learning-of-Instance-Segmentation-with-Inter-pixel-Relations/2.png" alt="" style="width:20%" /></center>

&emsp;&emsp;将CAM的特征提取和计算写成公式的。为了扩大CAM的分辨率，backbone的最后一个stage的下采样stride从2改为1，整体降采样只有1/16。

## 4. Inter-pixel Relation Network

### 4.1  IRNet Architecture             
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Learning-of-Instance-Segmentation-with-Inter-pixel-Relations/3.png" alt="" style="width:60%" /></center>

&emsp;&emsp;结构而言IRNet很简单，利用的特征都是R-50的五个level特征图，只是进行的操作略有不同，Displacement Field分支先将小尺度特征图融合一次再和大尺度的融合。赋予特征以意义和学习约束的是目标函数gt的制定。        

### 4.2  Inter-pixel Relation Mining from CAMs     
~~好用回头再填坑~~        

### 4.3  Loss for Displacement Field Prediction            
&emsp;&emsp;作者的观察是：同一个实例上的实例中心应该是一样的，即有公式如下，式中x代表像素坐标，D(x)是预测的质心位置：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Learning-of-Instance-Segmentation-with-Inter-pixel-Relations/4.png" alt="" style="width:30%" /></center>

因此针对该部分没有标椎gt的情况，设计的前景loss如下，分别计算像素和质心的差，其实就是上面公式移项得到的：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Learning-of-Instance-Segmentation-with-Inter-pixel-Relations/5.png" alt="" style="width:30%" /></center>

对于背景而言，无法预测出准确的质心：        
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Learning-of-Instance-Segmentation-with-Inter-pixel-Relations/6.png" alt="" style="width:30%" /></center>


### 4.4  Loss for Class Boundary Detection            
&emsp;&emsp;同样面临的问题是没有GT监督，作者的观察假设是：边界出现的位置其两边像素对的分类标签不同。（好用回头再填坑）                

### 4.5  Joint Learning of the Two Branches         
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Learning-of-Instance-Segmentation-with-Inter-pixel-Relations/7.png" alt="" style="width:20%" /></center>

直接将三个loss相加即可。

## 5. Label Synthesis Using IRNet
~~好用回头再填坑~~

## 6. Experiments
&emsp;&emsp;该方法的思路比较明确，方法略微繁琐一点，涉及的超参数相对多。  

* **Instance Segmentation**  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Learning-of-Instance-Segmentation-with-Inter-pixel-Relations/8.png" alt="" style="width:60%" /></center>

弱监督实例分割的工作相对较少。数据上来看，效果确实还不错远高于PRM，甚至也高于bbox监督的效果很好、吊打很多算法~~但是比较复杂的SDI~~。也可见和谁比很重要，SDI号称达到全监督模型的95%，却是和DeepLabv1比的....和Mask RCNN相比就差远了。


* **Semantic Segmentation**  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Learning-of-Instance-Segmentation-with-Inter-pixel-Relations/9.png" alt="" style="width:60%" /></center>

整体来看，bbox监督的效果要在class监督之上的。PRM不算是占优；IRNet效果甚至高于bbox监督的算法，如经典的BoxSup；和SDI差不多持平，但是后者在BSDS上的训练这一点来说普适性受限，而且较为麻烦。

分割效果：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Learning-of-Instance-Segmentation-with-Inter-pixel-Relations/10.png" alt="" style="width:80%" /></center>

<br>
<br>
<hr />