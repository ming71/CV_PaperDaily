|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2017.11 [CVPR]<p/span>

## 1. Introduction
### 1.1 区别
&emsp;&emsp;航空图像区别于传统数据集，有其自己的特点，面临很大的数据集偏差问题，例如导致数据集的泛化能力差：  
* 尺度变化性更大（很好理解，如车辆和机场；而且很可能一张大图就一个目标，一个小区域反而有很多密集目标）
* 密集的小物体检测（如港湾、停车场）
* 检测目标的不确定性：方向的随机性和尺度随机性（如桥梁这样极端的长宽比，会使anchor先验的检测效果打折扣）

<!-- more -->

### 1.2 数据集简介
&emsp;&emsp;DOTA数据集包含2806张航空图像，尺寸大约为4kx4k，包含15个类别共计188282个实例。其标注方式为四点确定的任意形状和方向的四边形（区别于传统的对边平行bbox）

## 2. Annotation of DOTA  
* 数据类别  
&emsp;&emsp;plane, ship, storage tank, baseball dia- mond, tennis court, swimming pool, ground track field, har- bor, bridge, large vehicle, small vehicle, helicopter, round- about, soccer ball field , basketball court.共计15个类，其中14个主类，small vehicle 和 large vehicle都是vehicle的子类。
下图是与NWPU数据集相比实例数目。可以看出这个的样本不均衡问题还是稍微好一点的。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/DOTA/1.png" alt="" style="width:50%" /></center>


* 标注方式  
&emsp;&emsp;没有选择(x,y,w,h)和(x,y,w,h.θ)，而是标记四个顶点八个坐标得到不规则四边形。具体是首先标注出一个初始点，为(x1,y1)然后顺时针方向依次标注234。初始点一般选择物体的头部；如果是海港这样没有明显视觉形状的对象，选择左上角为第一个点。如下图abc所示，d是传统方法标注，有很多重叠。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/DOTA/2.png" alt="" style="width:100%" /></center>


* 数据集划分  
&emsp;&emsp;1/6验证集，1/3测试集，1/2训练集。目前发布了训练集和验证集，测试集不会发布。

## 3. Properties of DOTA   
* 图像尺寸  
&emsp;&emsp;从800x800到4000x4000不等，标注直接在原图上进行，不进行裁剪。
* 不同类别的尺寸  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/DOTA/3.png" alt="" style="width:70%" /></center>

&emsp;&emsp;从表中可见实例还是具有很大的尺度变化性的。
* 长宽比的变化性  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/DOTA/4.png" alt="" style="width:90%" /></center>

&emsp;&emsp;AR是aspect ratio，前两张分别表示水平bbox和有方向的bbox的AR比例，可以看出其长宽比的变化也是很大的。
* 目标密度  
&emsp;&emsp;上图第三张反映的是图像包含物体数目的程度，反映了有相当一部分图片的目标十分密集。甚至过千了，这样看来，传统用于COCO检测的模型在NMS只设置100上限降低计算量的方法远远不可行了。

## 4.Evaluations
* 流行框架下的检测结果对比  
&emsp;&emsp;从一些基本的检测网络测试基准来看，还是两阶段的效果可以， 单阶段普遍不行。毕竟两阶段可以一定程度抵御类别不平衡、平移不变性等问题，特征提取更好自然不在话下，还有一种可能就是作者采用的单阶段检测器都没有采用FPN结构，所以小目标不行，而小目标占了很大部分，所以效果差也是情有可原。其中效果最好的是FR-O，也就是可旋转bbox的Faster-RCNN检测器，一方面是Faster RCNN本身好，另一方面也反映了更好的gt能够辅助学到更好的特征（虽然上下文有用比较好，但是明显斜着的舰船车辆这种带来了太大的overap，甚至框到下一辆车了，成为'hard gt'，必然劣化性能）

* 数据集裁剪  
&emsp;&emsp;还有一个问题，就是DOTA数据集的尺寸太大了，普通检测网络输入会计算过慢，实际测试会进行图片的裁剪，得到1024*1024的patch，stride=512。这个过程可能将一些完整的目标分割开来，然后对分割的部分计算IoU，检测之后重新拼接回去。

* 检测器的缺陷  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/DOTA/5.png" alt="" style="width:70%" /></center>

&emsp;&emsp;上图反映的问题：ab对比没啥，OBB好于HBB；cd对比发现OBB不行了，因为OBB方法更贴近真实长宽比，其中就容易出现这种极大长宽比的情况难以回归（推测是anchor的先验偏离）；ef的海港回归都不怎么样，密集样本的检测都有缺陷 （推测加FPN就好点了吧）




<br>
<br>
<hr />
