|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

**顺便mark我的数据增强代码工具（持续更新）：https://github.com/ming71/toolbox/tree/master/data_augmentation**

<span id="inline-blue">论文发布日期：2018 [ICANN]<p/span>

## 1. Introduction

&emsp;&emsp;~~引用之前也是德国人的一篇文章观点，这里站队了,不会是一个组的相互引用吧。~~  现在的正则化方法不是必要的；而数据增强不能被简单认为是一种正则化方法，但也能起到同等的正则化作用。同时通过实验对比发现数据增强的正则化效果比采用单纯的正则化方法如dropout,weight decay(L2)效果好，加上其具有普适性且无需调参，所以优势很大。  
<!-- more -->

&emsp;&emsp;参考文献部分：**深层网络比浅层网络在数据增强上的收益更大**。应该是因为：深层网络更容易过拟合。这个地方注意区别，**小数据集数据增强比大数据集效果好**，别弄混了。  

## 2. Experimental setup

* 网络选择  
&emsp;&emsp;一个复杂一个简单的，分别是WRN(Wide Residual Network，可以看做加强的残差网络)和All-CNN（可以看做简单的全卷积网络）。后面会在All-CNN上设计不同深度的三个版本进行对比实验。

* 数据集设置  
&emsp;&emsp;采用了两个数据集CIFAR-10和CIFAR-100，每个数据集内设置的对照试验有三个：不同的增强程度强度、不同的训练样本、数据增强与正则化方法。      
&emsp;&emsp;关于不同的增强强度，分为no augmentation, light and heavier augmentation。

## 3. Results        

### 3.1 减少数据集训练样本
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Further-advantages-of-data-augmentation-on-convolutional-neural-networks/2.png" alt="" style="width:70%" /></center>

* 数据增强强度对比  
&emsp;&emsp;light的涨点基础上heavier的涨幅并不高。一方面是light已经足够拟合bias内情况了；另一方面也可能反而引起过拟合了；        
&emsp;&emsp;越是大数据集，heavier提升越大，因为其超越bias的部分越多，这种增强就能起到作用；        
&emsp;&emsp;越深度网络学习和拟合能力都越强，所以更多增强越好；        
&emsp;&emsp;训练样本越少，数据增强的提升效果越是显著性地增强，这一点上训练样本多少和数据集大小具有一致性。

* 正则化与增强对比  
&emsp;&emsp;最浅的红色是只用正则化的结果，可见哪怕不用正则化只要加了增强（两个深紫色bar）效果都要比他好；        
&emsp;&emsp;正则化的效果不稳定，如CIFAR-100在All-CNN上有两个情况是加了反而效果会变差，效果而言却确实不明显也不稳定，但实际也有他自己的优势。        
&emsp;&emsp;还有一点就是，正则化需要调参比较麻烦。                

### 3.2 不同深度的网络架构    
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Further-advantages-of-data-augmentation-on-convolutional-neural-networks/3.png" alt="" style="width:70%" /></center>          

&emsp;&emsp;选用的All-CNN设计了不同的深度进行对比。观察发现：
* 数据集大小要和网络容量match（欠拟合/过拟合）
* 越深效果越好，和前面分析一致不赘述（可以一定程度认为就是上面的两个网络的对比）

### 3.3 Discussion            
&emsp;&emsp;上面还有一个观察~~也是作者认为自己的创新所在~~，可以看出同时使用正则化和数据增强并不会有所助益，甚至可能削弱正则作用。个人认为：数据增强和正则化之间具有一定的一致性，其相互的抵消就和bbox_oly的部分增强效果差原因是类似的。但是这个比较难验证和下手。


<br>
<br>
<hr />
