
|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 



<span id="inline-blue">论文发布日期：2019.8.7 [ICCV]<p/span>

## 1. Introduction
* **Motivation**  
实例分割的需要大量样本的训练，但是标注更难获得。数据增强获取样本停留在传统层面。
<!-- more -->
 
* **Related works**  
为了丰富训练数据和输入特征，现行方法有一部分是利用其它领域的数据进行弱监督学习获取额外信息。  
近两年有相关工作验证了crop-paste的方法在检测和我分割数据增强的效果。

## 2. Our approach

&emsp;&emsp;将目标随机裁剪粘贴到临近的区域，并附加小幅的尺度和旋转抖动；考虑到近邻位置中很多地方是冗余的，以及远处也有可能的相同模式可以进行增强，有必要对这些可能位置结合进行进一步精炼使之概率分布图符合真实世界的先验知识。        
&emsp;&emsp;下图中，a原图，b是随机近邻抖动增强，c是位置概率热图，d是根据热图进行的增强。        
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/InstaBoost-Boosting-Instance-Segmentation-via-Probability-Map-Guided/1.png" alt="" style="width:50%" /></center>

### 2.1  Overview            
&emsp;&emsp;首先是空间中物体平移和旋转的表示，可以通过仿射变换矩阵来表征：            
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/InstaBoost-Boosting-Instance-Segmentation-via-Probability-Map-Guided/2.png" alt="" style="width:40%" /></center>

该矩阵可以提取成一个四维元组：            
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/InstaBoost-Boosting-Instance-Segmentation-via-Probability-Map-Guided/3.png" alt="" style="width:50%" /></center>

对应原图I上的某个物体O，其上任意一初始点定义为(x0,y0)，目标点(x,y)，定义一个概率密度函数f，P是该结果的概率图，表示如下：            
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/InstaBoost-Boosting-Instance-Segmentation-via-Probability-Map-Guided/4.png" alt="" style="width:50%" /></center>

该概率图没有进行任何变化的位置应该是最大概率，小范围内进行缩放和旋转应该具有高的概率，基于这个观察有：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/InstaBoost-Boosting-Instance-Segmentation-via-Probability-Map-Guided/5.png" alt="" style="width:50%" /></center>

&emsp;&emsp;在此基础上，根据概率生成按照仿射变换矩阵H进行变换。  
&emsp;&emsp;但是可能的位置不能只限于附近位置，还需进一步进行扩展，使得即使较远距离的背景，如果具有和档期位置相似的模式，也能够有较大的转移概率。以便最大限度地利用这种裁剪增强。这部分就通过轮廓相似性的热图进行衡量。        

### 2.2  Random InstaBoost            
主要为两步：先分割前景背景，然后根据概率进行放置和转移。

* **Instance and background preparation**  
首先是前景背景的分割，虽然标注提供了label，还是使用matting获得平滑的物体patch；inpainting将背景填充上，如下图分别是原图、inpainting的背景、matting的物体。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/InstaBoost-Boosting-Instance-Segmentation-via-Probability-Map-Guided/6.png" alt="" style="width:70%" /></center>

* **Random transformation**  
变换四元数组(tx,ty,s,r)是服从(0,0,1,0)附近的均匀分布。此外对原图进行少量的模糊增强。然后根据四元数组进行物体的分配即可。    

### 2.3  Appearance consistency heatmap guided InstaBoost              
&emsp;&emsp;上面的概率生成函数f是直接服从均匀分布，为了考量更合适的位置，需要重新定义f。这里将f分解成三个条件概率: fxy(·) , fs(·) , fr(·)，分别是 txty，s，r 的概率密度函数。假设三者的自变量是相互独立的，概率密度函数可以由条件概率进行简化表示为：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/InstaBoost-Boosting-Instance-Segmentation-via-Probability-Map-Guided/format.png" alt="" style="width:50%" /></center>

可以看到，三个概率是相互独立的，可以分别生成。实际上s r直接均匀分布生成。还需要处理xy坐标概率。   
         


#### 2.3.1  Appearance consistency heatmap
* **Appearance descriptor** 
直觉来说距离原始物体越远的背景对物体实例的影响越小，由此定义外观描述子D(·)如下。  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/InstaBoost-Boosting-Instance-Segmentation-via-Probability-Map-Guided/8.png" alt="" style="width:50%" /></center>

实际物体例子如下，可以看到白色轮廓线实际分为三层，由内而外由白色到灰色，表征不同的层次。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/InstaBoost-Boosting-Instance-Segmentation-via-Probability-Map-Guided/9.png" alt="" style="width:50%" /></center>

* **Appearance distance**  
有了轮廓线的定义，再来通过外观描述子来度量不同轮廓线的距离。如度量D1 = D(c1x, c1y)和 D2 = D(c2x, c2y)的距离：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/InstaBoost-Boosting-Instance-Segmentation-via-Probability-Map-Guided/10.png" alt="" style="width:50%" /></center>

采用欧氏距离即可；超出图像的点忽视掉。

* **Heatmap generation**  
有了上述定义，就可以计算热图了。上述距离计算中，将一个定义为D0也就是实例中心位置，另一个遍历图像的其余像素得到D，并计算两描述子的距离d(D0,D)并由对数进行归一化如下。 

其中M和m为度量的极值，得到的概率图和原像素一一对应即可生成热图。效果如下：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/InstaBoost-Boosting-Instance-Segmentation-via-Probability-Map-Guided/11.png" alt="" style="width:70%" /></center>

#### 2.3.2  Heatmap to transformation tuple

* **Coordinate shift**    
热图到xy位移概率之间的映射通过蒙特卡洛方法采样。

* **Scaling and rotation**  
由于三个概率之间是相互独立的，所以可以直接采用均匀分布生成fr fs即可，然后进行对应的变换。

#### 2.3.3  Acceleration
&emsp;&emsp;计算量过大导致无法应用。为了降低时间复杂度~~这个是怎么计算的不太明白，不应该就是WH吗为啥平方了~~，将原始图像先缩放到固定的尺寸，然后计算热图后上采样到原来的尺寸上。权衡了速度与效果。


## 3. Experiments
### 3.1 Result
* **目标检测**  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/InstaBoost-Boosting-Instance-Segmentation-via-Probability-Map-Guided/12.png" alt="" style="width:70%" /></center>

1.对高IoU的高精度目标提升更好
2.小目标的提升没有大目标多
3.越深的backbone效果越好！


* **实例分割**  
和检测的结论类似，不再赘述。

### 3.2  Analysis
* **Comparison with context model**   


和采用背景上下文建模的网络进行比较，自然是本方法效果好。

* **Comparison with random paste**  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/InstaBoost-Boosting-Instance-Segmentation-via-Probability-Map-Guided/13.png" alt="" style="width:40%" /></center>

和随机裁剪粘贴方法相比，反而测试发现后者性能会下降。这很好理解。



* **Substantial Improvement**
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/InstaBoost-Boosting-Instance-Segmentation-via-Probability-Map-Guided/14.png" alt="" style="width:40%" /></center>

Mask RCNN在多轮训练后有过拟合的问题，使得性能有瓶颈；而InstaBoost方法不会。但是实质只是能够提高这种瓶颈，很好理解，因为增强的样本可能性更多了，所以数据多样性更丰富，不容易过拟合。

* **Sensitivity analysis**
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/InstaBoost-Boosting-Instance-Segmentation-via-Probability-Map-Guided/15.png" alt="" style="width:50%" /></center>

由表看出，最佳缩放尺度是0.8-1.2，太大太小都不好（这个就随缘调参了）；平移尺度的敏感度比较低，相对而言在原始位置较小邻域内移动都是合理的。




<br>
<br>
<hr />