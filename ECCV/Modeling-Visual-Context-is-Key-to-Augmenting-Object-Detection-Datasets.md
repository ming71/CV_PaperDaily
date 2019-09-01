
|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 




<span id="inline-blue">论文发布日期：2018.7.7 [ECCV]<p/span>


## 1. Introduction  
* 视觉上下文   
在检测目标出现遮挡、晦暗、噪声、截断等情况，可以通过目标的上下文推理得到物体的类型。此外，不同类别之间有的也是密切相关。
<!-- more -->

* 相关方法和存在问题  
这里着重研究的是crop-paste和上下文信息的利用。之前的随机裁剪和粘贴，会使得物体与环境上下文的关系变得错综不明如下图所示。  

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Modeling-Visual-Context-is-Key-to-Augmenting-Object-Detection-Datasets/1.png" alt="" style="width:70%" /></center>


注意：所以之前有的直接将图像合成的增强方式其实idea上从出发点就并不work，只是借助CNN的强大学习能力work而已。这种增强的文章还不少...

* 解决思想
模型预测图中各个位置的实例出现可能性；然后自动找到合适的位置，放置物体实例完成数据增强。如下图。


## 2. Related Works  
&emsp;&emsp;对空间上下文信息利用的相关文章介绍；数据增强的部分，不只是传统方法，也关注了其他类似的新样本合成方法，以及和他类似的方法（室内上下文监督）。


## 3. Modeling Visual Context for Data Augmentation  
&emsp;&emsp;大体流程是两部分：由一个上下文网络根据图像中包含bbox标注的区域，抽取物体得到上下文作为输入，进行训练让该网络自行判断上下文的目标情况；然后用这个网络判断待检测图像得到物体出现概率的上下文情况，并以此作为依据完成实例的增强放置。详细过程建参见论文和下图说明。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Modeling-Visual-Context-is-Key-to-Augmenting-Object-Detection-Datasets/2.png" alt="" style="width:70%" /></center>



&emsp;&emsp;想法：这种思路的落脚点其实一样，只是实现方式途径不同。这里是利用的上下文学习了上下文建模网络，思路比较有意思很新颖。





<br>
<br>
<hr />