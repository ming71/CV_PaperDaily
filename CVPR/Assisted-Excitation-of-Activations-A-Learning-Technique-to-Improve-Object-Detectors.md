
|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2019.6.5 [CVPR]<p/span>

## 1. Motivation  
* YOLOv3的问题  
（1）定位误差大（2）正负样本的极度不均衡（<u>focal loss也不能解决</u>思考一下）

* 原因分析
（1）YOLOv3的分类回归是同时进行的，这个输出就有问题     
（2）和focal loss说的一样，因为单阶段检测器的密集滑窗导致正负样本的不均衡  
<!-- more -->
* 解决方法  
&emsp;&emsp;引入gt的mask的增强指导，借鉴课程学习（curriculum learning）的思想，由易到难逐步提升检测难度。该方案只在训练时使用，并且逐渐下调影响因子，因此不会影响检测速度。下面是一张用bbox增强的效果图：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/YOLO+/exp1.png" alt="" style="width:70%" /></center>   
         
## 2. Related Works  
&emsp;&emsp;通过这部分的阐述可以看出，其实将分割信息作为辅助提高检测性能的思想以前就有很多工作了，而且还有很多其他的辅助信息方法（<u>**多看论文很重要啊！**</u>），所以作者强调自己的novelty是引用了课程学习的思想。          
&emsp;&emsp;这类问题其实也分为两种思路（有点牵强）：
* 将分类和分割任务共同学习和优化（就是MTL），如实例分割任务
* 引入语义分割的特征强化检测的学习  
&emsp;&emsp;本文的方法类似第二种，而且是直接用的bbox作为分割基础，从而避免单独引入分割标注，更加简单。

## 3. Assisted Excitation Process           
### 3.1 Assisted Excitation using Ground-Truth        
&emsp;&emsp;结构图如下，展示了衰减曲线和融合方式：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/YOLO+/str1.png" alt="" style="width:40%" /></center>

&emsp;&emsp;下面再介绍具体计算过程。先定义一个示性函数如下，bbox内的像素位置为1，生成一个0-1mask：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/YOLO+/for1.png" alt="" style="width:40%" /></center>

&emsp;&emsp;最终期望的生成特征如下，其中alpha是关于时间的函数用于控制训练中的强度衰减，式中c为通道数，e是增强特征：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/YOLO+/for2.png" alt="" style="width:30%" /></center>

&emsp;&emsp;增强方法e很多，最后选定的是如下方法，其中d是通道数，可见是在有bbox的分割区进行增强，增强是按照通道去平均等量加上去的（作者的实验证明该效果最好）：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/YOLO+/for3.png" alt="" style="width:30%" /></center>

&emsp;&emsp;选取的时间衰变函数如下：    
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/YOLO+/for4.png" alt="" style="width:30%" /></center>

### 3.2 Inference          
&emsp;&emsp;由于施加分割特征的作用会衰减为0，相当于最终的检测网络和初始的yolo完全一样。 

### 3.3 Assisted Excitation in YOLOv2 and YOLOv3      
&emsp;&emsp;下图展示的是和YOLOv2结合的过程，通过实验发现在16倍降采样的时候添加AE层能取得最好的效果；
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/YOLO+/str2.png" alt="" style="width:90%" /></center>
    
&emsp;&emsp;下图展示的是和YOLOv3结合的过程，得到改进版本的YOLOv3+，通过实验发现在第二个尺度上进行上采样到16倍降采样尺度时候添加AE层能取得最好的效果。（为什么不在全尺度都加？）
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/YOLO+/str3.png" alt="" style="width:90%" /></center>


## 4. Experiments and Results  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/YOLO+/exp2.png" alt="" style="width:80%" /></center>
&emsp;&emsp;从上左边的图可以看到，AE强化过的网络有全面的提升，其中在大尺度上的提升更加明显，推测原因是：大物体上加了分割强化后能够获得更强的辨认度，小物体由于本身尺度不大所以增加后也不明显。结果而言印证了这种强化的有效性，但是也完全地陷入了小目标检测的弊端了--<u>像素内容少而被忽视</u>。        
&emsp;&emsp;右图的信息不太好辨认。先看yolov2的曲线来说，低iou阈值能够得到更高的改进的精度，说明其召回更好了，但是精度一高就趋于重合，改进失效，说明这种增强提高了低质量bbox的精度。再看yolov3，全IoU都有少量的提高，但是不特别大且没有明显的趋势，说明其采用的多尺度预测能一定程度地解决问题，并在其基础上能对全部精度都有增益。

## 5. Discussion  
* Excite object regions vs suppress non-object regions?  
&emsp;&emsp;（这一点我的理解不一定对，看原文）作者认为两种机制不同。如果是直接进行不含目标样本的抑制，那么在检测阶段无法获知哪些样本（bbox）是不含样本的，需要进行打分判断（也就是conf置信）；而AE激励响应是直接通过逐步的减少让最终结果只关注到少量的正样本上，这一点学习起来更加容易。

* What happens during back-propagation?  
&emsp;&emsp;AE增强是会作用到反向传播的，由于它会增强特定区域的响应，所以也会扩大感受野的响应，使得正样本和无定位误差的样本能够更多地贡献loss，降低易分样本的作用。这一点和focal loss背后的思想是类似的。
* Curriculum learning  
&emsp;&emsp;借鉴其从易到难的思想逐步移除增强特征的辅助。（这一点来看是不是和预训练的意义有几分相似？引导网络向正确的方向收敛。但是也不一样，这里修改了特征，开辟了不同的训练路径）

* Applicability  
&emsp;&emsp;这一点不用多说，可以往很多检测去上直接加就完事了。






<br>
<br>
<hr />