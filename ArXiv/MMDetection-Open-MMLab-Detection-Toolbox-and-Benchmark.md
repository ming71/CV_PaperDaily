|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2019.6.17 <p/span>


## 1. Introduction

MMdetection的特点：
* 模块化设计：将不同网络的部分进行切割，模块之间具有很高的复用性和独立性（十分便利，可以任意组合）
* 高效的内存使用
* SOTA


## 2. Support Frameworks  
* 单阶段检测器  
SSD、RetinaNet、FCOS、FSAF

* 两阶段检测器  
Faster R-CNN、R-FCN、Mask R-CNN、Mask Scoring R-CNN、Grid R-CNN

* 多阶段检测器  
Cascade R-CNN、Hybrid Task Cascade

* 通用模块和方法  
soft-NMS、DCN、OHEN、Train from Scratch 、M2Det 、GN 、HRNet 、Libra R-CNN

## 3. Architecture

模型表征：划分为以下几个模块：          
Backbone（ResNet等）、Neck（FPN）、DenseHead（AnchorHead）、RoIExtractor、RoIHead（BBoxHead/MaskHead）       
结构图如下：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/mmdetection/1.png" alt="" style="width:60%" /></center>

## 4. Notice
* 1x代表12epoch的COCO训练，2x类似推导
* 由于batch-size一般比较小（1/2这样的量级），所以大多数地方默认冻结BN层。可以使用GN代替。

<br>
<br>

<p id="div-border-top-blue">相关地址链接</p> 

OpenMMLab官方工具箱地址：[mmdetection官方github](https://github.com/open-mmlab/mmdetection)  

个人注释的mmdetection代码（**推荐**）：[mmdetection注释github](https://github.com/ming71/mmdetection-annotated)






<br>
<br>
<hr />