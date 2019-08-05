|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2019.7.16<p/span>


## 1. Introduction

介绍了一个数据集ImageNet-A，包含7500个自然界中存在出现（相对于人工合成的对抗样本）的易错分的实例。该数据集使得DenseNet-121的分类精度从92%降低到2%。
<!-- more -->
反映两个问题：    
（1）分类器过于依赖颜色、纹理和背景信息（如下图），很难在这些自然对抗样本上取得好的效果。    
（2）当前分类器精度很高并不意味着可以很好地解决这些对抗样本的问题。因为自然对抗样本在ImageNet中的比例很小，即使全部分不对，精度也可以达到很高。    
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Natural-Adversarial-Examples/1.png" alt="" style="width:60%" /></center>
 
【注意】**分类和检测任务是不同的**。后面自己做实验会说明这一点。

## 2. ImageNet-A

* 数据集  
选择了ImageNet的200个类，解压出来是200个文件夹，每个文件夹下单独类对抗样本几十张。

* 实验结论  
作者通过实验改进发现传统的训练方法在对抗样本数据集上不奏效，分类涨点不多。部分对网络结构的调整能够有较好的提升。   

* 数据集部分图片  
我用cascade mask RCNN在COCO上训练的模型对其进行分割测试，发现有的结果确实不好；但是有的挺好的：推测原因是：分割检测和分类不同，可以聚焦局部信息。  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Natural-Adversarial-Examples/2.jpg" alt="" style="width:45%" /></center>

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Natural-Adversarial-Examples/3.jpg" alt="" style="width:45%" /></center>

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Natural-Adversarial-Examples/4.jpg" alt="" style="width:45%" /></center>

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Natural-Adversarial-Examples/5.jpg" alt="" style="width:45%" /></center>

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Natural-Adversarial-Examples/6.jpg" alt="" style="width:45%" /></center>

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Natural-Adversarial-Examples/7.jpg" alt="" style="width:45%" /></center>
<br>
<br>
<hr />