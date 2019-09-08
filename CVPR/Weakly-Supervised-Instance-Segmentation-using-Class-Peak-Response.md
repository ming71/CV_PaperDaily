| 创建人 |                       知乎论文阅读专栏                       |              个人博客               |
| :----: | :----------------------------------------------------------: | :---------------------------------: |
| ming71 | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |




<span id="inline-blue">论文发布日期：2018 [CVPR]<p/span>

## 1.Introduction

&emsp;&emsp;本文使用图像级的类别标注监督信息，通过探索类别响应峰值使分类网络能够很好地提取实例分割mask。本工作使用图像级标注进行弱监督学习。（~~CVPR2017有使用到图像级标注信息弱监督分割的相关工作，CVPR2018同期也有~~，但是差不多确实可以称得上是第一批。）

<!-- more -->

* **Motivation**   
在分类监督信息之下，CNN网络会产生一个类别响应图，每个位置是类别置信度分数。其局部极大值往往具有实例很强视觉语义线索。

* **Method**   
首先将类别峰值响应图的信息进行整合，然后反向传播将其映射到物体实例信息量较大的区域如边界。上述从类别极值响应图产生的映射图称为Peak Response Maps (PRMs)，该图提供了实例物体的详细表征，可以很好地用作分割监督信息。直观流程如下：  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Instance-Segmentation-using-Class-Peak-Response/1.png" alt="" style="width:60%" /></center>

* **Advantages**   

  相关的弱监督分割工作其实很多。本方法在分类网络基础上的计算成本小；仅用分类标注就能进行有效的**实例分割**；还有其他使用条件随机场、轮廓学习、RNN方法等。   




## 2.Method 

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Instance-Segmentation-using-Class-Peak-Response/3.png" alt="" style="width:60%" /></center>

&emsp;&emsp;首先将图片经过正常的分类网络训练，其中在类别预测响应图上提取出局部响应极值点，进行增强卷积后预测出PRM。然后结合多种信息进行推断生成mask。

### 2.1 Fully Convolutional Architecture

&emsp;&emsp;分类和分割任务有区别。baseline设置时，去掉分类网络中的全局模块如全局池化、FC层，改为全卷积网络。

### 2.2 Peak Stimulation 

&emsp;&emsp;(~~这部分真的是服了！就是很简单几个mask相乘或者卷积，搞了一堆莫名其妙的公式定义，强行把逼格抬高......~~)这部分论文中的定义和公式归结起来就是，将分类网络输出的类别响应图，获取0-1mask，使得极值点部分的信息进行极值响应的强化。并定义了类别得分计算方式，也就是所有极值点求均值。使用该方法能够进一步剔除背景，更多关注物体上响应最强的部分，效果如下：  

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Instance-Segmentation-using-Class-Peak-Response/4.png" alt="" style="width:60%" /></center>

### 2.3 Peak Back-propagation

&emsp;&emsp;定义极值响应图的反传过程，不是梯度反向传播，这里是将得到的增强极值响应图反向回传生成一张局部响应图也就是RPM。  

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Instance-Segmentation-using-Class-Peak-Response/5.png" alt="" style="width:60%" /></center>

## 2.4 Weakly Supervised Instance Segmentation

&emsp;&emsp;直接计算得分进行排序和NMS。在生成的proposal基础上（~~生成方式未明示~~），分别从实例、边界和类别信息三个角度均衡。  

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Instance-Segmentation-using-Class-Peak-Response/6.png" alt="" style="width:40%" /></center>



## 3. Experiment

* **Ablation Study** 

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Instance-Segmentation-using-Class-Peak-Response/8.png" alt="" style="width:50%" /></center>

  在VOC2012上的实验，轮廓边界的影响相对不大；背景和实例关注部分影响大；PS次之。在VGG-16提升更大，很明显因为本来特征提取能力有限，性能就差所以对结构和算法更加依赖。  

* **Qualitative results** 

  效果自然是吊打前辈，如前一年CVPR同样工作的WILDCAT等，这就没什么好看得了。但是作者也坦言有局限性。如下图的失败分割案例，应该是类别响应的限制。    

  作者的解决办法是在分数计算时将这一因素考虑进去。这一项在Ablation Study效果明显就能理解了，起码能够work。   


<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Instance-Segmentation-using-Class-Peak-Response/10.png" alt="" style="width:50%" /></center>

* **Quality of peak response maps**  

  为了衡量PRM的R和GT的mask G相关性，采用分数如下，该参数越大说明和gt的覆盖效果越好。  
  $$
  \frac{\sum_{}R \bigodot Q}{\sum_{}R}
  $$
  
  结果如下：    
  
  1. 随着物体变多，性能会下降，但是依然可观。  
  2. RPM对于较大尺度变化范围（至少是一般检测框架的物体大小范围内），能够有较好的响应效果。  
  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Weakly-Supervised-Instance-Segmentation-using-Class-Peak-Response/11.png" alt="" style="width:50%" /></center>

 <br>
<br>

<hr />