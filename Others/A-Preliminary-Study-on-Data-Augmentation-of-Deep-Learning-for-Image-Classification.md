|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2019.6.11<p/span>


## 1. Introduction
&emsp;&emsp;这篇文章挺简单（~~简陋~~）的。~~语言组织和结构安排很糟糕，就是为了先挂在arxiv上的，不是什么正经文章~~，看看结论就行然后讨论一下。  
&emsp;&emsp;文章研究了一些简单的数据增强方法的效果对比，然后归纳出数据增强的几个指导方向，结合上次看到谷歌的learned-augmentation能有点思考。结论：  
（1）几何变换作用好  
（2）2-3倍的增强足够  
（3）小数据集更好  
<!-- more -->

## 2. Experiment and Conclusion
### 2.1 Experiment Design  
&emsp;&emsp;作者的实验选取增强方式除了常见的裁剪缩放平移等，还采取了直方图均衡、曝光等多种复杂的色彩变换如下表：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/A-Preliminary-Study-on-Data-Augmentation-of-Deep-Learning-for-Image-Classification/1.png" alt="" style="width:60%" /></center>

&emsp;&emsp;以及效果图（CIFAR-10）如下。实验做得挺一般没什么好看的，直接看结论。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/A-Preliminary-Study-on-Data-Augmentation-of-Deep-Learning-for-Image-Classification/2.png" alt="" style="width:70%" /></center>

### 2.2 Result and Rethink
* 小数据集的效果更佳  
&emsp;&emsp;小数据集提升效果明显，主要问题在于其容易**过拟合**。

* 几何变换效果更好  
&emsp;&emsp;数据增强是为了拟合实际出现的可能情况，而均衡、曝光等情况不常见，所以不如简单的平移裁剪等多，效果自然也不如其好。一方面有测试集的局限性，另一方面也能一定程度反映真实世界的图像变化情况。

* 关于增强能力是“一定程度”的思考：
1. 《Why do deep convolutional networks generalize so poorly to small image transformations? Aharon》把这个叫做photographer bias，大部分照片（人眼视觉也是）都是有焦点角度的，可以认为是某种同一性，数据增强如果拟合这些同一性，就能很好地实现效果提高。（极端如distortion类似的增强就基本不为photographer关注）

2. 增强个人感觉不是作者说的adding more feature，而应该是adding more useful feature，和上面我理解的同一性类似，正因此不同方案的增强才有了显著的性能差异。

3. 一个悖论：检测数据集的这种变化性也有体现，没有真实世界强。涉及对数据集的理解，为了更高的mAP，理论上可以追求同一性更强的检测框架即可，但是现实世界变化性远多于这个，这其中就有矛盾。



<br>
<br>
<hr />