|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

**顺便mark我的数据增强代码工具（持续更新）：https://github.com/ming71/toolbox/tree/master/data_augmentation**

<span id="inline-blue">论文发布日期：2017.8.4<p/span>

## 1. Introduction

&emsp;&emsp;数据增强普遍认为是正则化手段，减少过拟合，提高网络的泛化能力。介绍了一种数据增强方式--cutout。方法很简单就是图片上的随机crop像素块（如下图），但是这个思路表达的比这个简单的方法要深多了（就像第一次看到FPN一样）~~会编故事很重要~~,会洞察简单操作的背后思想和用途很重要。

<!-- more -->        
&emsp;&emsp;此外，有对于这个简单方法的一些拓展思考，比如分类和检测的增强等。


## 2. Cutout
&emsp;&emsp;需要注意的是，由于这个是在分类数据集CIFAR-10/100上测试的，必然有很多问题。

* Operation  
&emsp;&emsp;在图像上进行随机位置和一定大小的patch进行0-mask裁剪。一开始使用裁剪上采样等变换出复杂轮廓的patch后来发现简单的固定像素patch就效果不赖，所以直接采用正方形patch。        
&emsp;&emsp;作者为了论证~~讲故事~~丰富，认为这种操作相当于连续的dropout，只是后者是对神经元操作而且是离散的，而cutout是操作输入像素而且连续，可以减少噪声。

* Motivation  
&emsp;&emsp;通过patch的遮盖让网络学习到遮挡的特征。cutout不仅能够让模型学习到如何辨别他们，同时还能更好地结合上下文从而关注一些局部次要的特征。


## 3. Rethink

&emsp;&emsp;一点想法和思考，结合之前的一些论文增强对比实验。

* cutout效果不如几何变换   
&emsp;&emsp;在CIFAR上效果平平（在之前一篇论文的对比实验看出，只有仿射的 一般涨点）应该是摄像师偏差的缘故，这里的CIFAR自然 有这个问题。                    


* 数据集的问题   
&emsp;&emsp;收回之前对谷歌论文《Learning Data Augmentation Strategies for Object Detection》的肤浅评价，别人确实揭示了这一点，我当时没看出来而已。
&emsp;&emsp;CIFAR是图像分类，但是移植到检测上，还要考虑bbox的问题：裁剪应该在bbox内进行。        
1. 有bbox，增强要考虑是否交于或者只进行bbox的变化        
2. 图像的有用特征和无用特征的距离更大        


* cutout的尺寸问题  
&emsp;&emsp;这个涉及对遮挡问题定义的思考。       
1. patch尺寸首先最好是可变的，这样对大目标和大遮挡也有效    
2. 大目标和大遮挡是否有检测出的必要？如果没必要，那就按比例只是用小mask就行了；如果有必要，可以学习不同大小gt的不同mask比例进行增强

* 实现方式   
&emsp;&emsp;patch的mask不全在图像内的方式相比整个mask必须融入图像而言，增强能取得的效果更好。作者解释：这种小patch的增强图片能保证图像上更多样例被看到。如果真是这样，还有其他解决办法：1.设置不同size的patch，加入遮挡尺度的适应性  2.设置增强比，不让增强太多，避免学不到主要特征


## 4. Experiment

&emsp;&emsp;分类的实验上cutout没什么特别的参考性，可以简单看看：          
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Improved-Regularization-of-Convolutional-Neural-Networks-with-Cutout/2.png" alt="" style="width:75%" /></center>

&emsp;&emsp;通共32像素的图像，patch居然能达到这么高。但是检测任务就比这个比复杂多了，需要考虑遮挡的摄影师偏差，不太好直接统一处理

        

<br>
<br>
<hr />