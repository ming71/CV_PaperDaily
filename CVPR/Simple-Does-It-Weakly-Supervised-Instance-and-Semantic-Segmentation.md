| 创建人 |                       知乎论文阅读专栏                       |              个人博客               |
| :----: | :----------------------------------------------------------: | :---------------------------------: |
| ming71 | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |



<span id="inline-blue">论文发布日期：2017 [CVPR]<p/span>


## 1. Introduction

&emsp;&emsp;本文做的是bbox弱监督语义/实例分割任务，号称能达到全监督分割效果的95%。主要工作为：  
* 讨论了使用弱监督语义标签进行迭代训练的方法
* 证明了通过类似GrabCut的算法能通过bbox生成分割训练标签方法的可行性
* 在VOC数据集上逼近监督学习的分割任务效果
* 首个弱监督实例分割方法（~~居然之前没有，CAM问世之后应该会有一系列的算法才对~~）
<!-- more -->


## 2. From boxes to semantic labels

&emsp;&emsp;目的是从bbox标注获取尽可能高质量的分割label，而提供的现有信息包括：bbox标注、物体之间的先验关系。具体而言：
* C1 Background  
bbox标注之外的全都是背景。

* C2 Object extent  
bbox提供的实例标注中，大致能够确定物体的位置，还能得到关于形状的先验知识。

* C3 Objectness  
除了上面两条，还能利用形状先验通过一定的手段设计（例举某论文的segment proposal techniques方法）得到物体的可能形状。


### 2.1  Box baselines 
&emsp;&emsp;先拟定baseline。对于给定的bbox，将其内所有pix定位给定box的类别；如果两个box重叠，取面积小的在前面；bbox之外的全部定为背景。。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Simple-Does-It-Weakly-Supervised-Instance-and-Semantic-Segmentation/1.png" alt="" style="width:30%" /></center>

* **Recursive training**  
作者有一个观察：将bbox level的mask送入网络训练后得到分割maskg会要好。因此启发的操作是：将bbox level标注作为初始mask输入，每次将得到的标注作为gt进行下一轮迭代。这种方式的baseline称为Naive。效果如下：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Simple-Does-It-Weakly-Supervised-Instance-and-Semantic-Segmentation/2.png" alt="" style="width:80%" /></center>


&emsp;&emsp;其中，如果每次迭代得到mask后不直接送入下一轮，而是经过降噪处理，结合物体的先验和额外信息优化后再送进去，这样的pipeline称为Box。该部分的后处理有三条：

  * C1：将预测出的mask中，不再bbox gt内的像素设为0（背景）；
  * C2：如果分割得到的mask区域与bbox标注的IoU<0.5，则将重置bbox标注，继续迭代训练；
  * C3：DenseCRF后处理。

* **Ignore regions**  
作者的intuition是：通过更低的召回换取更高的像素标注精度（？）。因此初始阶段只用bbox mask内的20%像素，这部分像素不易错分，能有效地降低数据噪声干扰。除此之外其他步骤和Box一样，该方法取名Boxi。效果如下：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Simple-Does-It-Weakly-Supervised-Instance-and-Semantic-Segmentation/3.png" alt="" style="width:60%" /></center>


### 2.2  Box-driven segments   
&emsp;&emsp;尽管就作者的说法来看，使用bbox的baseline就已经能够达到stoa的水平，但是探索更好的segment仍有必要。
* **GrabCut baselines**  
采用HED边界增强的GrabCut（~~HED是在BSDS500训练的，这个方法不具普适性，而且麻烦~~），称为GrabCut+。每个标注box生成约150个分割输出（~~不会很慢？~~），对于特定像素，70%和20%为proposal的像素投票比例决定当前像素的前景背景界限。

* **Adding objectness**  
加入MCG算法生成分割proposal再和GrabCut联合优化，pipeline更复杂了，这部分没什么意思，只是用了别的技术融合，也没有特别要解决的问题，略过。


# 3. Semantic labelling results
&emsp;&emsp;按作者的说法，bbox弱监督实例分割没人做，所以只比较了语义分割的效果。
### 3.1 Main Result
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Simple-Does-It-Weakly-Supervised-Instance-and-Semantic-Segmentation/4.png" alt="" style="width:55%" /></center>

* 只用bbox作迭代效果逐渐劣化，很好理解，必然会这样。
* 后处理的工作其实都是保障分割准确性，才能在多轮迭代中提升性能的。例如Boxi比Box提升很多，原因就是其用更少的像素初始化bbox label，否则必然发散效果变差。

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Simple-Does-It-Weakly-Supervised-Instance-and-Semantic-Segmentation/5.png" alt="" style="width:50%" /></center>

&emsp;&emsp;中间的box和boxi是第10轮的训练结果；下面的复杂后处理只有一轮。可见后处理的效果还是很明显的，但是计算成本和流程复杂度也相应地提升上去了。可以看到结果居然十分逼近监督学习的效果。
比较奇怪的是Boxi相比Box提升居然没有那么大（论文也说提升很大，但是数值来看没有体现）。

### 3.2  Semi-supervised case
&emsp;&emsp;加入了10%VOC分割标注，但是性能没有很大的提升，说明弱监督分割的性能已经很好，逼近监督学习了。













<br>
<br>
<hr />