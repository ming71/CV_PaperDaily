
|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2019.2.4<p/span>

 
## 1. Introduction

&emsp;&emsp;上次亚马逊发了个分类的训练trick在CVPR上，这次是检测的，还没发表。就没什么多说的了，下面直接介绍。先看效果如下，其实摘要声称的5%是单阶段的yolov3的提升，说明：单阶段没有RoIPooling阶段很多性质确实不如两阶段，因此采用trick很有必要；相反，两阶段本身结构优于单阶段所以外加的trick提供的如不变性等网络自身能够学习和适应就不起作用了。 
<!-- more -->

&emsp;&emsp;拿来主义直接用就行，就不多去想了。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Bag-of-Freebies-for-Training-Object-Detection-Neural-Networks/1.png" alt="" style="width:45%" /></center>

## 2. Bag of Freebies 

&emsp;&emsp;提出了一种基于mixup的视觉联系图像混合方法，以及一些数据处理和训练策略。        

### 2.1  Visually Coherent Image Mixup for Object Detection            
&emsp;&emsp;先介绍图像分类中的mixup方法，作用是提供了训练的正则化，应用到图像上如下图，将图像作简单的像素值输入mixup的凸函数中得到合成图；然后将one-hot编码类似处理得到新的label。       
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Bag-of-Freebies-for-Training-Object-Detection-Neural-Networks/2.png" alt="" style="width:60%" /></center>

&emsp;&emsp;技术细节：         
* 相比于分类的resize，为了保证检测图像不畸变影响效果，作者选择直接叠加，取最大的宽高，空白进行灰度填充，不进行缩放。        
* 选择ab较大（如1.5,1.5）的Beta分布作为系数来混合图像，作者说是相干性视觉图像的更强；loss是两张图像物体的loss之和，loss计算权重分别是beta分布的系数  <center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Bag-of-Freebies-for-Training-Object-Detection-Neural-Networks/3.png" alt="" style="width:60%" /></center>


### 2.2  Classification Head Label Smoothing 
&emsp;&emsp;标签平滑在检测的分类任务常有用到，最早是Inceptionv2中提出。        
&emsp;&emsp;如果标签中有的是错的，或者不准，会导致网络过分信任标签而一起错下去。为了提高网络泛化能力，避免这种错误，在one-hot的label进行计算loss时，真实类别位置乘以一个系数（1-e），e很小如0.05，以0.95的概率送进去；非标注的类别原来为0，现在改为e=0.05送进去计算loss。网络的优化方向不变，但是相比0-1label会更加平滑。      
（标签平滑这个讲的不错：https://juejin.im/post/5a29fd4051882534af25dc92）
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Bag-of-Freebies-for-Training-Object-Detection-Neural-Networks/4.png" alt="" style="width:40%" /></center>        
&emsp;&emsp;这里进一步改进了一下label smooth的公式而已，在原来基础上除了个类别数。        

### 2.3  Data Preprocessing
&emsp;&emsp;就是数据增强，没什么其他的。至于分类也是几何变换和色彩变换。这么分区别其实是是否变换label。但是将真实世界就这么简单地分解过于粗糙了。好不容易谷歌的增强考虑到了如何学习一下检测任务的增强，但是也只是加了bbox_only的增强，就效果而言，一般；而且就实际来说，合理性和有效性有待商榷。     
&emsp;&emsp;作者认为，两阶段网络的RPN生成就是对输入的任意裁剪，所以这个增强就够了；这老哥膨胀了，two-stage就不用裁剪的增强，虽然两阶段能提供一些不变性，但是用了一般来说都是更好的。 

### 2.4  Training Schedule Revamping
训练策略上：余弦学习率调整+warmup      

### 2.5  Synchronized Batch Normalization 
跨多卡同步正则化，土豪专区，穷人退避     

### 2.6  Random shapes training for single-stage object detection networks        
多尺度训练，每经过一定的iteration更换一种尺度。举例是yolov3的尺度范围。


## 3. Experiment  
不看了，好用就成。





<br>
<br>
<hr />