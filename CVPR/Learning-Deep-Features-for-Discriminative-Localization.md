
| 创建人 |                       知乎论文阅读专栏                       |              个人博客               |
| :----: | :----------------------------------------------------------: | :---------------------------------: |
| ming71 | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |


<span id="inline-blue">论文发布日期：2016 [CVPR]<p/span>


## 1. Introduction
* **Insight**        
作者在之前的工作发现，分类网络即使不进行检测定位的监督学习，神经网络依然具有一定的目标定位能力。但是这种定位能力在分类任务中不能体现出来，因为分类网络最后都会用FC层进行全局特征的提取，破坏了卷积提取的局部定位信息。下图是分类训练的网络得到的CAM响应热图，可以看出响应位置特性。
<!-- more -->

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Learning-Deep-Features-for-Discriminative-Localization/1.png" alt="" style="width:50%" /></center>

* **Contribution**  
（1）神经网络提取的特征本身具有一定的定位能力，为了利用这部分特征，将分类网络去掉FC层，替换为GAP保持对整个物体的响应,得到CAM。        
（2）使用CAM对定位特征进行响应，验证了这种定位特征能够被迁移用到很多相关领域，如弱监督检测、弱监督分割等。



## 2. Class Activation Mapping 
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Learning-Deep-Features-for-Discriminative-Localization/2.png" alt="" style="width:60%" /></center>

&emsp;&emsp;公式用的挺多的，实际上归纳起来就是：backbone成全卷积，输出加上GAP，softmax归一化，系数加权到卷积。然后相加得到CAM图，上采样原图就能很好地观察分类网络响应物体位置的关系。       
&emsp;&emsp; 一个有趣的例子可以说明不同类别的响应不同。下图是一个gt为dome的图片，分类时有五个比较高得分的预测，可以看到不同类别预测下的激活特征图，会响应当前channel所属类的最大响应区域。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Learning-Deep-Features-for-Discriminative-Localization/3.png" alt="" style="width:50%" /></center>



## 3. Weakly-supervised Object Localization 

&emsp;&emsp;实验安排从网络结构（AlexNet、GoogleNet），池化方式（GAP、GMP），不同视觉任务之间进行设计和对比。

* **Classification**  
可以看出去掉FC层对分类有影响，但问题不大；GMP和GAP在分类任务上的性能相仿，没很大的区别。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Learning-Deep-Features-for-Discriminative-Localization/4.png" alt="" style="width:40%" /></center>

* **Localization**  
定位方法比较简单，用最大的外接框框住一定阈值的响应联通区域即可。这部分实验的结果表明，确实是GAP效果优于GMP，后者而言，低分段的信息也能参与精确定位但是被忽视了。从下图可以看出，这种分类网络提供的定位能力普遍存在于各种网络中：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Learning-Deep-Features-for-Discriminative-Localization/5.png" alt="" style="width:70%" /></center>



## 4. Deep Features for Generic Localization  
&emsp;&emsp;为了验证这种能力的普适性，在多个其他数据集上也做了类似的实验，发现效果也依旧客观，说明神经网络的这种定位能力具有一定的普遍性。        
&emsp;&emsp;此外，还在多个任务上验证了CAM响应的有效性，但实际都是基于分类网络学习和聚焦的内容得到的，本质差不多，可以用但是有效性不一定能保证。    <center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Learning-Deep-Features-for-Discriminative-Localization/6.png" alt="" style="width:70%" /></center>            

## 5. Visualizing Class-Specific Units

为了验证这种能力的普适性，在多可视化卷积层的响应区域并且用分割mask显示后，可以看出不同的卷积单元响应的是物体的不同区域（也是该作者ICLR2015的论文结论），将其进行组合才对完整的物体具有判别能力。这也会是其迁移应用需要解决的一个问题。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Learning-Deep-Features-for-Discriminative-Localization/7.png" alt="" style="width:60%" /></center>


<br>
<br>
<hr />