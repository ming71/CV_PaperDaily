|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2017.2  [WACV]<p/span>
## 1. Motivation

&emsp;&emsp;针对现在分割任务中的两个步骤提出改进:
* 上采样。上采样过程多用双线性插值，不可避免地有信息损失。
* 膨胀卷积。膨胀卷积会有gridding问题，感受野只来自部分非0窗格，临近信息全部丢失
<!-- more -->

## 2. Approach
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Understanding-Convolution-for-Semantic-Segmentation/str1.png" alt="" style="width:90%" /></center>

### 2.1 Dense Upsampling Convolution (DUC) 
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Understanding-Convolution-for-Semantic-Segmentation/str2.png" alt="" style="width:80%" /></center>

&emsp;&emsp;从上图最后一张图不难看出，实际上也是R-FCN、STDN一派的做法，将通道方向的信息进行抹平得到高分辨率的特征图。最后输出的层数是类别数，经过一个softmax每类一张mask分数，然后做通道方向的argmax得到每个像素所属的类别进行标注。作者认为相对于双线性插值，例如16降采样步长，原来不到16*16的物体经过降采样可能信息就丢了，那么即使双线性插值也无法还原信息；而这种DUC操作使原图大小上的操作，不会有这种问题（？？？~~基础特征都是一样的，要丢都会丢才是~~）    

### 2.2 Hybrid Dilated Convolution (HDC)
&emsp;&emsp;这个不太好理解，大概就是指<u>膨胀卷积在dilated convolution在高层使用的rate变大时，对输入的采样将变得很稀疏，将不利于学习：一是因为一些局部信息完全丢失，二是长距离上的一些信息可能并不相关；并且gridding效应可能会打断局部信息之间的连续性</u>。如下图：  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Understanding-Convolution-for-Semantic-Segmentation/exp1.png" alt="" style="width:50%" /></center>

&emsp;&emsp;作者设计了不同的膨胀率的膨胀卷积进行信息组合，在一组卷积内（若干层）通过锯齿波形的变化，避免信息的重叠（因此设置膨胀率也不能有最大公约数）， 从而使输出可以获取更宽阔的区域信息，在保持接收野大小不变的情况下提高信息利用率。对于大型目标的识别有好处。  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Understanding-Convolution-for-Semantic-Segmentation/exp2.png" alt="" style="width:50%" /></center>

&emsp;&emsp;实际参数选择和膨胀率的设置是根据经验公式来做的，具体如何在ResNet中将不同bottlenecks划分成哪些组参见原论文的experiment部分。此外，就作者实验来看，膨胀率设置到很大了，感受野非常大，理论上甚至可能出现信息不相关，但是效果还能上提，没有出现类似FPN结构的语义鸿沟问题，这点值得关注一下。说明膨胀卷积的感受野提升和卷积降采样的提升还是有所不同的（但本质而言只是采样位置不一样，仍然都是信息的深组合），可能是卷积具有很强的局部相关性，而膨胀卷积则一定程度破坏了这种局部相关性，从而在同样学习参数下获得了更大的感受野。推测这种结构相对于卷积更不容易过拟合，但是对小物体的学习可能不算友善。  

## **Conclusion**  
&emsp;&emsp;这对分割任务中的两种常见操作提出改进：双线性插值上采样、膨胀卷积  
* 用类似R-FCN的通道信息填补得到高分辨率特征图  
* **通过变化的膨胀率组合解决膨胀卷积的稀疏学习问题：长距离信息不相关、局部信息丢失、信息不连续** 


<br>



<hr />