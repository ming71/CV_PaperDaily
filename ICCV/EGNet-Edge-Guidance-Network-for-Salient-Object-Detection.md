| 创建人 |                       知乎论文阅读专栏                       |              个人博客               |
| :----: | :----------------------------------------------------------: | :---------------------------------: |
| ming71 | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |

<span id="inline-blue">论文发布日期：2019.8.8[ICCV]<p/span>

ICCV 2019南开程明明组的工作

## 1. Introduction  
* 出发点：基于FCN方法的显著性检测任务（分割也有是类似的）由于是像素级的判别，缺少结构信息，导致显著性目标检测的边界不够精确。
<!-- more -->

* 解决方案：引入边缘信息作为监督，将边缘信息和显著性目标检测任务共同学习，并且互相特征复用、优势互补，能够取得更好的效果。


## 2. Salient Edge Guidance Network
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/EGNet-Edge-Guidance-Network-for-Salient-Object-Detection/1.png" alt="" style="width:70%" /></center>
* **PSFEM**  
采用的backbone是U-Net，在提取不同尺度的特征图，并且通过类似FPN的结构将上层的语义信息向下传播；每一层特征图经过一层卷积后输出显著性检测结果；

* **NLSEM**      
加入边缘信息，底层融合了高层语义信息的特征图加入边缘信息学习边缘，用于后面的特征融合；

* **O2OGM**  
将NLSEM的卷积特征和PSFEM每个特征图的上采样结果进行像素相加，得到边缘和显著性检测的融合特征；该特征最终输出预测显著性目标结果。


## 3. Result 

&emsp;&emsp;不太了解显著性检测，咱也不知道，咱也不好说，但是效果应该是拔萃的，直接和SOTA比了个遍，指标上全面碾压：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/EGNet-Edge-Guidance-Network-for-Salient-Object-Detection/2.png" alt="" style="width:70%" /></center>
PR曲线在三个数据集上也是凌驾于其他检测器，高高在上，十分嚣张的样子：        
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/EGNet-Edge-Guidance-Network-for-Salient-Object-Detection/3.png" alt="" style="width:70%" /></center>
&emsp;&emsp;比较有意思的是引入的方式和融合中相关问题的讨论。纯粹引入边缘信息并不一定能带来很好的效果，这在目标检测中已经有很多工作做过了，从结果来看效果不怎么突出，远不像能取得这里很亮眼的表现。这里能work可能归因于他的一些融合考量和学习方法，加上边缘和显著性检测两个任务本身相似性很高，更符合MTL的方式。        



<br>
<br>

<hr />