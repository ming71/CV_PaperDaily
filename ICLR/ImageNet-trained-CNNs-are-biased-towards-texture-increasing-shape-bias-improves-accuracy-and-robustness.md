
|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 



<span id="inline-blue">论文发布日期：2018.11.12[ICLR2019]<p/span>

&emsp;&emsp;最近刚好看到别的东西有关纹理的观点，回到这篇文章上再看一遍发现其被ICLR录用了。有关的观点挺有意思的，算是挖了个小坑。

## 1.Introduction 
&emsp;&emsp;CNN训练学习到的实际是纹理特征而不是形状特征，这和人类的认知方式有所区别；增强神经网络学习形状的能力，能够增强网络的鲁棒性。比如下面这只带有大象纹理的象皮猫，CNN会学习纹理而得出这是象的判断，而人类则是通过形状判定为猫。
<!-- more -->

<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/ImageNet-trained-CNNs-are-biased-towards-texture-increasing-shape-bias-improves-accuracy-and-robustness/1.png" alt="" style="width:60%" /></center>


&emsp;&emsp;至于出现上述情况的原因也好理解（~~作者主要探讨了数据集的问题，可能是method基于数据的工作就没怎么深入提吧~~），当下网络多使用固定小感受野的卷积核进行特征提取，只能获取局部的小范围纹理特征。但是却足以取得非常好的分类和回归效果，这说明对于CNN而言，学习小范围的纹理特征是他们胜任分类等任务的主要手段，而且已经足够了。  

## 2.Discussion  
&emsp;&emsp;在本文证明CNN主要通过纹理识别物体之前，相关工作已经证明，诸如尺寸、色彩是不起决定性的作用，甚至也有工作表明：CNN可以识别形状扭曲混乱的物体，但无法辨别不带纹理信息的物体，比如下面图中三四的猫的轮廓。**用mmdetection自己做实验找形状轮廓勾勒较多、细节纹理不多、甚至没有纹理的图像，进行试验发现其实效果没有那么不堪**，比如马的二值化图像都能检测出来，使用的仅仅是faster rcnn(R-50backbone)（可能本身具备提取形状的能力，而且分类和检测任务不一样，如FPN等结构能够从更广的尺度上对图像进行分析和理解（~~记忆~~），依赖纹理可能不像分类那样严重）。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/ImageNet-trained-CNNs-are-biased-towards-texture-increasing-shape-bias-improves-accuracy-and-robustness/2.png" alt="" style="width:60%" /></center>


&emsp;&emsp;但是对于人类而言，人类感知多是对形状的感觉和判断，而不是尺寸或纹理特点。 可以通过对训练数据集合适的改动，使得CNN由偏向于纹理学习的模式转变为偏向于形状学习的模式。
最后采取的方案是，同时训练ImageNet和Stylized-ImageNet同时学习形状和纹理特点，可以得到优于单独学习某一个的效果（其中单独学习形状好于纹理，~~这个结果让人有点不解，难道是分类任务的全局性来说形状特征的鲁棒性更强？至少在检测任务很明显现有的纹理具有更强的指导作用~~）。


## 3.Experiment

根据实验的分析：
&emsp;&emsp;好几处如下面的图，以一个为例。图中越往左形状分辨能力越强。可以看出，形状分辨能力是人类>AlexNet>GoogLeNet>ResNet-50≈ VGG-16。

&emsp;&emsp;原因的话，猜测是GoogLeNet含有inception模块可以增大感受野（因为他用的是Inceptionv1，包含大尺寸5x5和7x7）所以不差；这几种都直接输出分类信息，所以网络越深对提取的纹理细节特征的组合越强，得到的语义信息越抽象，加强了特定纹理和特定分类的联系，而VGG就差很多，至于ResNet-50更差，可能是纹理学习饱和或者形状学习有下限又或者残差模块可以改善性能？导致他和VGG差不多.
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/ImageNet-trained-CNNs-are-biased-towards-texture-increasing-shape-bias-improves-accuracy-and-robustness/3.png" alt="" style="width:60%" /></center>

&emsp;&emsp;结果而言这个用SIN训练的方法并不算是solid的新方法，挖坑的意义大于实际应用意义，看引用次数和引用的文献来说好像确实是这样。




     

<br>
<br>
<hr />