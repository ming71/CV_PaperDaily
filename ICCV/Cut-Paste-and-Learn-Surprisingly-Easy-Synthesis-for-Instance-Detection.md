
| 创建人 |                       知乎论文阅读专栏                       |              个人博客               |
| :----: | :----------------------------------------------------------: | :---------------------------------: |
| ming71 | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |



<span id="inline-blue">论文发布日期：2017 [ICCV]<p/span>

## 1. Introduction   
&emsp;&emsp;本文是室内环境检测的工作，从数据合成角度提出一种简单的cut-paste方法实现bbox level的标注下提取物体mask并paste到可能的场景中去，从而得到新的更具realism的合成数据。    
<!-- more -->

&emsp;&emsp;在此之前已经有几个同样做室内目标定位领域的数据合成工作，和这个类似。大致可以看出室内目标定位的特点是：
* 物体的变化性小（尺度，形状上）
* 场景比较单一，前景背景的可分离度大
* 物体的出现上下文相对比较简单（都是桌面,床等场景）


## 2. Approach Overview 
&emsp;&emsp;大体流程如下图：首先随机采样样本patch和背景；通过卷积网络预测出patch中的物体mask；然后进行一定的增强（如旋转）；最后将随机选择的物体粘贴到随机目标背景上即可。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Cut-Paste-and-Learn-Surprisingly-Easy-Synthesis-for-Instance-Detection/1.png" alt="" style="width:60%" /></center>


## 3. Approach Details and Analysis 

### 3.1 Collecting images  
&emsp;&emsp;待检测的目标是从BigBIRD数据集中进行选择的；背景从UW Scene数据集选择背景样本图；前景背景分割是通过FCN分割。但是gt采用的图像深度（该数据集是RGBD数据仅）作为mask标注，直接可用。

### 3.2  Adding Objects to Images
&emsp;&emsp;测试合成数据性能比较的方法是，在合成数据集上进行训练，真实数据集上测试，然后与真实数据集上训练测试的结果作对比。该部分选用的是GMU Kitchen数据集验证。下面的实验分析结果提前列出：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Cut-Paste-and-Learn-Surprisingly-Easy-Synthesis-for-Instance-Detection/2.png" alt="" style="width:85%" /></center>

* **Blending**  
这里提出的一个问题是paste会出现边界和环境的不相容问题，对边界进行处理，使得图像更加自然能够取得更好的结果。（~~不太好说~~）从视觉上来看，是否处理边界的区别不大，但是作者认为还是很重要的，而且有实验支撑和相关的一篇文献作为引证，也许会有点用。效果如下。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Cut-Paste-and-Learn-Surprisingly-Easy-Synthesis-for-Instance-Detection/3.png" alt="" style="width:45%" /></center>


* **Data Augmentation**  
采用的增强方式有2D/3D旋转，阶段和遮挡。后面实验会看出确实有效果。



## 4. Experiments  
&emsp;&emsp;实验中选取的物体来自BigBIRD，GMU Kitchen，ACV三个数据集；背景来自UW Scene。最后生成的合成数据6000张，验证集为GMU Kitchen的6728张，ACV的17556张。

* Training and Evaluation on the GMU Dataset  
在两个数据集的训练样本容量差不多的情况下，这里看来合成的76.2和真实的86.3差别还是有的。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Cut-Paste-and-Learn-Surprisingly-Easy-Synthesis-for-Instance-Detection/4.png" alt="" style="width:65%" /></center>


* Evaluation on the Active Vision Dataset  
研究了合成数据和真实数据一起训练时的效果情况，这个ablation study做的不错。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Cut-Paste-and-Learn-Surprisingly-Easy-Synthesis-for-Instance-Detection/5.png" alt="" style="width:55%" /></center>

1. 只用real的情况。可以看出在10%-30%时涨幅很大，但是继续到70%就不行了，应该是数据的瓶颈。同样的情况，在real+syn也能看到。
2. 为什么不管什么比例的real数据，只要加上syn都能涨点很多，不受瓶颈的限制了？推测是因为，real数据来自GMU，而syn有ACV，所以这里可能存在不公平。
3. 为什么syn一般，real一般，syn+real就很好？一是上面说的可能存在的不公平，而是ACV的数据量更大任务更复杂（33objects），所以可以容纳更大的训练样本不至于过早达到数据提升的瓶颈，加数据效果仍然明显。realdata的瓶颈应该是数据的问题。
4. 作者结论是验证了patch level的处理就能很好地合成新图像，不用更多的上下文考量。感觉不太对，只是这个任务比较简单，上下文还是很重要的。









<br>
<br>
<hr />