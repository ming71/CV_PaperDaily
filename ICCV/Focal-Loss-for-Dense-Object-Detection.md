
|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2017.8.7 [ICCV]<p/span>

## 1. Introduction  
* 出发点：单阶段的检测网络能否达到两阶段的效果，如果不行，问题在哪。
* 类别不平衡问题：  <!-- more -->
（1）大多数负样本对损失的贡献是无效的 （2）简单负样本的大量训练会劣化模型性能        
&emsp;&emsp;现有网络的解决方法：两阶段如RCNN中可以通过一阶段的RPN过滤掉大多数的背景，从而减轻了类别不平衡的问题；而单阶段检测器是直接在特征图上密集滑窗预测的，因此会产生大量的候选框，其中就包含很大部分的背景，导致类别不均衡问题。这就是单阶段效果不好的原因所在。        
&emsp;&emsp;类别不平衡可以出现在样本的任何形式和阶段，如输入的训练图片的类别不平衡、生成的RoI（proposal）的类别不平衡等。可以理解为一种**训练容量的mismatch**。针对可能出现的情况，改进单阶段或者两阶段可以采用OHEM采样降低不平衡问题，再如yolov3，只选择最高的IoU作为正样本，对负样本进行很大的压制，得到了较低的背景误检，以至于加focal loss没什么提升（<u>所以不要好东西就随便加，看看到底为什么work，是否符合当前任务</u>）

* 与OHEM的区别    
&emsp;&emsp;OHEM是采样训练样本实现正负样本的均衡，而这个是全部样本都训练，只是loss不同。OHEM重视了困难的样本，但是忽视了更多存在的简单易分的负样本，FL都考虑到了。所以FL的<u>优势在于既考虑了正负样本也考虑了难分样本的损失贡献</u>。

## 2. Focal Loss  
&emsp;&emsp;首先是最原始的二分类交叉熵，在label为1和非1两种情况分别采用不同的loss。gt为1时，预测的1概率越大loss越小；gt不为1时，预测的1概率越大loss越大；
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Focal-Loss/format1.png" alt="" style="width:40%" /></center>
&emsp;&emsp;然后采用了一个标记，简化了上述表示：   
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Focal-Loss/format2.png" alt="" style="width:30%" /></center>
&emsp;&emsp;使得CE损失直接为：        
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Focal-Loss/format3.png" alt="" style="width:35%" /></center>
&emsp;&emsp;改进CE使得loss能够对正负样本区别对待如下。其中at和pt一样是分段函数，形式也是相同的（y=1时，at=a，否则at=1-a），在类别1的样本特别多时，选取a在[0,0.5]降低其loss贡献权重，实际实验取得0.25。  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Focal-Loss/format4.png" alt="" style="width:25%" /></center>
&emsp;&emsp;上述公式可以敏感正负样本，但是无法处理难分样本，于是有了对CE的下面改进。实验中发现gamma=2效果最好。易分样本的得分要么很高（正样本）要么很低（负样本），但是统一在pt上都表现为pt很大，那么乘以权值1-pt后，下面loss中贡献的损失就很小；而难分样本往往pt在0.5左右摇摆，其loss贡献相对就大了。（确实是这样，难分样本可以分为正负两种，正的来说，是网络不确定是不是正类，那么给分例如会在0.5-0.7；负的错分样本既然是错分，也不会太相信，所以也是0.3-0.6这样，而不大可能是0.1得分的正类）        
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Focal-Loss/format5.png" alt="" style="width:35%" /></center>
&emsp;&emsp;实际使用是综合正负样本和难易分样本得到的FL如下：  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Focal-Loss/format6.png" alt="" style="width:40%" /></center>

## 3. RetinaNet Detector

&emsp;&emsp;没有特别的创新，就是简单的ResNet+FPN单阶段检测器。实现细节和参数设置不赘述。实验也不解读了，暂时不感兴趣。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Focal-Loss/str1.png" alt="" style="width:90%" /></center>
## Conclusion
* 注意不平衡问题，这个应该有启发
* 单阶段检测网络使用focal loss会有一定效果，而两阶段本身能够通过RPN，OHEM等方式对类别不均衡具有一定的抵抗能力
* 为了避免密集的训练样本带来不均衡问题，改进单阶段也能达到类似focal loss的目的，如yolov3（目前见过的loss设计到调参都很鸡肋啊）



<br>
<br>

<hr />