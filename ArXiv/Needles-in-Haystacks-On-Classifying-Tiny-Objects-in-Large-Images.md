

|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 



<span id="inline-blue">论文发布日期：2019.8.6<p/span>



## 1. Introduction  
&emsp;&emsp;FB的工作，关注的是大尺寸图像中小目标的分类问题。目前的通用数据集大多数目标占比都很大，对于微小目标检测的能力有限。为了解决这个问题：
* 提出两个具有宽信噪比的微小目标分类的数据集（主要是针对低信噪比图像分类的问题）  
* 设计一个分类网络验证相关结论。（结论中有很多点其实是容易推理得到的常识，这里将其与微小目标的分类结合起来了而已；此外也有有趣的点）
<!-- more -->


&emsp;&emsp;先定义一个O2I（Object-to-Image）参数衡量目标占比，如下例举了几个数据集。ImageNet的O2I均在1%以上，用于训练fine-tune微小目标的分类是有问题的；相较而言病理学等图像往往可能是很小的病变组织就能决定图像的类别最小可达 10e-6。

&emsp;&emsp;可能的应用场景。如医学影像中小微病灶的关注，远距离航拍遥感目标的检测、地形勘探等会有用。但是基于极小像素做出的判断一者需要更多的样本训练，这样的标注不具备迁移性、难以获得；二者纹理相似的部分很容易引起误检，应该需要进行进一步优化（推测）。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Needles-in-Haystacks-On-Classifying-Tiny-Objects-in-Large-Images/1.png" alt="" style="width:60%" /></center>


## 2. Experimental testbed 
&emsp;&emsp;关于提出的数据集和相关论述略。设计的分类pipeline借鉴BagNet结构，采用patch训练。进行特征embedding；IN规范化；全局池化（多种比较）；接分类器。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Needles-in-Haystacks-On-Classifying-Tiny-Objects-in-Large-Images/2.png" alt="" style="width:70%" /></center>

简单关注要验证的结论：

* O2I limit vs. dataset size   
数据越多泛化性和精度越高；随着O2I的减小，需要的训练样本更多。（让人联想到关于合成数据的论文，同样的结论，但是性能并不是线性的）
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Needles-in-Haystacks-On-Classifying-Tiny-Objects-in-Large-Images/3.png" alt="" style="width:50%" /></center>

* Inductive bias  
小的感受野效果要好于大的。这一点很好理解，因为太大的感受野包含过多的上下文含有了过多的无关信息。

* Global pooling operations  
不同全局池化的对比发现，较大的O2I上不同池化的性能相似；但是当O2I比较小的时候，全局最大值池化效果最好；对于更小的O2I，soft attention poolings效果最好。  
现在基本都不用pooling，但是如果在多尺度或者多层次信息以及减小计算为前提时，也可以用到；此外也有论文指出全局平均池化的优势。在需要pooling时，这部分的对比可以作为对小目标的特征效果的参考，况且小目标在pooling中经常丢失，更需要被注意。

* Optimization  
随着 O2I的减少，优化任务难度逐渐上升，而最大值池化能够一定程度上对这种变化比较鲁棒。比较好理解，目标占比变小之后，均值等方式只会更严重抹平特征，最大值池化不受域的限制会保留最明显的特征，一定程度能抵御特征丢失的问题。





<br>
<br>
<hr />