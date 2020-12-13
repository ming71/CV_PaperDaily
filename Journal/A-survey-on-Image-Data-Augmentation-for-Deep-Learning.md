|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71 , [HesseSummer](https://github.com/HesseSummer)  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

**顺便mark我的数据增强代码工具（持续更新）：https://github.com/ming71/toolbox/tree/master/data_augmentation**

<span id="inline-blue">论文发布日期：2019.6.1[Big Data]<p/span>

## 1. Introduction  
* 数据增强与过拟合  
验证是否过拟合的方法：画出loss曲线，如果训练集loss持续减小但是验证集loss增大，就说明是过拟合了。
<!-- more -->

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/A-survey-on-Image-Data-Augmentation-for-Deep-Learning/1.png" alt="" style="width:60%" /></center>

* 数据增强目的  
通过数据增强实现数据更复杂的表征，从而减小验证集和训练集以及最终测试集的差距，让网络更好地学习迁移数据集上的数据分布。这也说明网络不是真正地理解数据，而是记忆数据分布。

* 数据增强的方法  
（1）数据变换增强          
包括几何变换、色彩空间变换，随机擦除，对抗训练，神经风格迁移等    
（2）重采样增强        
主要侧重于新的实例合成。如图像混合（mixup），特征空间的增强，GAN生成图片。一张图看明白：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/A-survey-on-Image-Data-Augmentation-for-Deep-Learning/2.png" alt="" style="width:60%" /></center>

## 2. Image Data Augmentation techniques

### 2.1 Data Augmentations based on basic image manipulations  
* Geometric transformations          
&emsp;&emsp;如果数据集潜在的表征能够被观察和分离，那么简单的几何变换就能取得很好的效果。对于复杂的数据集如医学影像，数据小而且训练集和测试集的偏差大，几何变换等增强的合理运用就很关键。

  * Flipping  
作者提到了要衡量普遍性的观点。但是这种变换对于数字数据集不具有安全性。

  * Color space  
主要提及的识别RGB通道上的变换，将三通道图进行分离，以及直方图变换增强等。（颜色空间更多增强方式可以参考A Preliminary Study on Data Augmentation of Deep Learning for Image Classification）

  * Cropping  
通常在输入图片的尺寸不一时会进行按中心的裁剪操作。裁剪某种程度上和平移操作有相似性。根据裁剪幅度变化，该操作具有一定的不安全性。

  * Rotation  
大幅度的旋转对数字集会有不安全性的考虑。

  * Translation  
平移也需要合理设计。如车站人脸检测，只需要中心检测时，就可以加合适的平移增强。平移后空出部分填0或者255，或用高斯分布噪声。

  * Noise injection  
在像素上叠加高斯分布的随机噪声。

* Color space transformations    
&emsp;&emsp;由于实际图像中一定存在光线偏差，所以光线的增强十分有必要（但是IJCV的光流文章指出，3D建模的灯光增强实在是很难学习到，所以对于光线增强的效果不如几何也可能因为**光线的复杂度更高，数据样本远远不够**）。色彩变换十分多样，如像素限制、像素矩阵变换、像素值颠倒等；灰度图和彩图相比，计算时间成本大大较少，但是据实验效果会下降一些，很明显因为特征的维度被降维了；还有尝试将RGB映射到其他的色彩空间进行学习，YUV,CMY.HSV等。        
&emsp;&emsp;除了计算大内存消耗和时间长等缺点，色彩变换也面临不安全性，比如识别人脸的关键信息是黄白黑，但是大量增强出红绿蓝，会丢信息。颜色变换的增强方法是从色彩空间角度拟合偏置，效果有限的可能性是多样的：1. 真实几何多样性比颜色更简单  2. 色彩的变化多样性更多，导致增强不够反而学不好，颜色空间的欠拟合 3. **变换不安全**


* Experiment
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/A-survey-on-Image-Data-Augmentation-for-Deep-Learning/3.png" alt="" style="width:40%" /></center>  

**随机裁剪**效果最好。        


### 2.2 Geometric versus photometric transformations 
* Kernel filter   
滤波器核在图像处理用的比较广，这里提到用这种方法来增强。还提到了一种正则化增强方法PatchShuffle，在一个patch内随机交换像素值，使得对噪声的抵抗更强以及避免过拟合。        
文章指出关于应用滤波器增强的工作尚且不多，因为这种方法其实和CNN的机制是一样的，这么做也许还不如直接在原始CNN上加层加深网络。

* Mixing images  
~~就是那篇被ICLR拒稿的采样方法~~直接均值相加混合。  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/A-survey-on-Image-Data-Augmentation-for-Deep-Learning/4.png" alt="" style="width:50%" /></center>

&emsp;&emsp;还有非线性的mixup裁剪如下：  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/A-survey-on-Image-Data-Augmentation-for-Deep-Learning/5.png" alt="" style="width:50%" /></center>

&emsp;&emsp;以及随机裁剪的图像混合：  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/A-survey-on-Image-Data-Augmentation-for-Deep-Learning/6.png" alt="" style="width:50%" /></center>

&emsp;&emsp;这些混合方式是十分反人类直觉的，因此可解释性不强。只能说是可能增强了对底层低级特征如线条边缘等的鲁棒性。其实有点没有抓住关键点。

* Random erasing  
随机擦除就是类似cutout的思想，通过mask的遮挡使得网络能够提高遮挡情况的鲁棒性。需要手工设计的部分包括mask的大小以及生成方式。是一种比较有效的方法。这种方式也需要考量增强的安全性，比如MNIST数据集8cutout后可能出问题。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/A-survey-on-Image-Data-Augmentation-for-Deep-Learning/7.png" alt="" style="width:50%" /></center>

* A note on combining augmentations  
组合的增强方式往往是连续变化的，导致数据集的容量会迅速扩大，这对于小数据集领域来说容易发生过拟合 ，所以需要设计合理的搜索算法设计恰当的训练数据集。        

### 2.3 Data Augmentations based on Deep Learning
*  Feature space augmentation  
之前刚看的基于SMOTE类别不平衡的过采样法来进行特征空间的插值操作进行数据增强，就实验效果而言不算特别出众。

* Adversarial training  
对抗样本训练可以提高鲁棒性，但是实际应用中其实提高不一定明显，因为自然对抗样本的数目没有那么多。而NIPS的对抗攻击大赛很多从神经网络的学习策略下手，进行梯度攻击，更加偏向于人为的攻击了，对于普适的检测性能提高意义反而不大，更强调安全需求高的场合。

* GAN‑based Data Augmentation
  
* Neural Style Transfer

不觉得这个效果会普遍很好，应该来说是针对特定域会有效（如白天黑夜），实际效果应该有限。

* Meta learning Data Augmentations 
  * Neural augmentation
  * Smart Augmentation  
  两个东西差不多，就是上次看到SmartAugment方法。随机采样类内图片进行通道叠加然后输出融合图像，学通过梯度下降使得输出图像的类内差距减小（没考虑类间关系，可能也不便处理）。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/A-survey-on-Image-Data-Augmentation-for-Deep-Learning/8.png" alt="" style="width:50%" /></center>

* AutoAugment  
谷歌最早做的自学习增强方法，走的NAS的思路RL+RNN搜索增强空间，还有后来最近发的检测增强也是大同小异，基本就是换汤不换药，问题在于**搜索空间太大**，复现搜索过于依赖硬件条件（~~普通实验室玩不起~~）

## 3. Design considerations for image Data Augmentation

### 3.1 Test-time augmentation             
&emsp;&emsp;许多都论文指出在检测阶段进行同等的数据增强能够获得较好的效果。归结可以认为是训练检测阶段的一致性。当然，这种手段时间成本太高，只在如医学影像等追求精度的关键领域可以使用。        

### 3.2 Curriculum learning            
&emsp;&emsp;Bengio团队早年在ICML提出的观点，确实合理，一开始就进行大量的增强容易导致网络不收敛。
从一个数据集学习到的数据增强也可以迁移到其他数据集。


### 3.3 Resolution impact
高清（1920×1080×3）或4K（3840×2160×3）等高分辨率图像需要更多的处理和内存来训练深度CNN。然而下一代模型更倾向于使用这样更高分辨率的图像。因为模型中常用的下采样会造成图像中信息的丢失，使图像识别更困难。
研究人员发现，高分辨率图像和低分辨率图像一起训练的模型集合，比单独的任何一个模型都要好。
某个实验（这里就不注明引用了）在256×256图像和512×512图像上训练的模型分别获得7.96%和7.42%的top-5 error。汇总后，他们的top-5 error变低，为6.97%。
随着超分辨率网络的发展，将图像放大到更高的分辨率后训练模型，能够得到更好更健壮的图像分类器。

### 3.4 Final dataset size            
&emsp;&emsp;数据增强的形式可以分为在线和离线增强。前者是在加载数据时增强，可能造成额外的内存消耗（现在都是数据容量不变的随机增强）。               
&emsp;&emsp;此外作者提到了一个比较有意思的点：当前数据集尤其是进行增广后是十分庞大的，明显能够在一定程度上缩小数据集但是保持性能下降不多的子集效率会高得多。

### 3.5 Alleviating class imbalance with Data Augmentation            
&emsp;&emsp;这也是值得借鉴的一点。通过增强在一定程度上解决类别不平衡问题。但增强需要仔细设计，否则会面对已经学习较好的类别或者场景造成过拟合等问题。




<br>
<br>
<hr />
