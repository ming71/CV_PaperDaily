|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 


<span id="inline-blue">论文发布日期：2017.6.29 [CVPR]<p/span>

## 1. Introduction  
* 文本检测的特点和挑战  
&emsp;&emsp;尺度变化性、特殊的宽高比例、方向、不同字体风格、光照、角度等，其中尤其是**方向检测**十分重要。

* 本文方案
<!-- more -->
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/R2CNN/str.png" alt="" style="width:90%" /></center>

&emsp;&emsp;如上图，**首先RPN生成正常的水平proposal（~~论文用的axis-aligned，应该是表述不清，实际是水平竖直检测框~~），二分类筛掉，然后保留的每个proposal回归斜框和水平框坐标，最后对斜框进行斜向NMS得到最终检测结果**。效果如下图：        
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/R2CNN/img1.png" alt="" style="width:50%" /></center>

&emsp;&emsp;a是原图，b是RPN生成的proposal，c是回归的正框和斜框，d是斜的NMS保留的斜框。注意这个和RRPN的区别，RRPN是直接在RPN阶段就生成斜框了。


## 2. Proposed Approach

### 2.1 Problem definition
* gt定义  
&emsp;&emsp;文本检测的ICDAR竞赛数据集是不规则的四边形，用顺时针的四个点坐标进行表征，不是严格的矩形框。这种不规则四边形一般可以用斜矩形来近似拟合，所以后面的bbox都采用斜矩形进行bbox预测。

* 旋转表征  
&emsp;&emsp;关于旋转方向的表征，很多方法是直接选择的角度回归来直接表征，本文的旋转表征不带角度如下所示：  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/R2CNN/img2.png" alt="" style="width:50%" /></center>

### 2.2 Rotational Region CNN (R2CNN) 
* RPN for proposing axis-aligned boxes  
&emsp;&emsp;RPN和常规的相似，加了更小的anchor尺寸以便获得更好的效果。其他配置均同Faster R-CNN。

* ROIPoolings of different pooled sizes  
&emsp;&emsp;为了适应文本不同方向的特殊长宽比，除了标准7x7外，还采用了不同比例的RoIpooling为11x3和3x11的信息进行获取融合

* Regression for text/non-text scores, axis-aligned boxes, and inclined minimum area boxes  
&emsp;&emsp;在回归上，<u>不仅回归斜框坐标，还回归正框</u>，实验表明这样的效果能提高。

* Inclined non-maximum suppression    
&emsp;&emsp;实验表明，斜的NMS效果会比正的好。很好理解，对于<u>斜向密集目标的检测</u>，常规NMS很容易出现IoU过大而被抑制，漏检很大，斜向NMS就不会。        

### 2.3 Training objective (Multi-task loss) 
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/R2CNN/format.png" alt="" style="width:60%" /></center>
* 细节如函数具体形式之类的就不说了
* RPN就是个二分类就行
* proposal的loss如上图公式。分为三部分：分类、正框reg、斜框reg。可见同时回归了正框和斜框的位置这个看似多余效果还行。


## 3. Experiments
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/R2CNN/exp.png" alt="" style="width:90%" /></center>

&emsp;&emsp;一张表就够了，可以看出：    
1. 加入正框预测的效果涨点值得一看，应该是让他更好学习    
2. 加入小尺度anchor效果不错（加一个大尺度anchor的效果劣化）    
3. 多尺度pooling的效果也还行，不是特别明显    
4. 斜框的NMS效果也还行，比3大比12小     
     

## Conclusion
* 采用多尺度pooling提取不同宽高比的信息
* 斜框的NMS解决传统NMS的密集漏检问题
* 正斜框的MTL（RRPN的思路不同）




<br>
<br>
<hr />