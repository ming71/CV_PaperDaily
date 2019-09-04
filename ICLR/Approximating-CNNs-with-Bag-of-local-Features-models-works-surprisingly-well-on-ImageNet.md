
|  创建人   |  知乎论文阅读专栏 | 个人博客 | 
|  ----  | ----  | ----  | 
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |    





<span id="inline-blue">论文发布日期：2019.4.20 [ICLR]<p/span>


## 1. Introduction  
&emsp;&emsp;基于一次分类任务上的尝试：将图片划分为小的patch只关注局部信息，训练后仍能取得很好的效果。采用的模型是基于ResNet-50改进的BagNets。发现该小模型能够在利用局部patch信息分类上等同于深度网络的效果，作者由此认为过去很多深度网络实现更好性能是通过调参实现的，而不是更好的决策方式。（~~有点偏激了~~）
<!-- more -->


* 神经网络的决策复杂性  
作者认为神经网络决策的难以理解的原因是输入层和隐层之间的复杂依赖关系，中间隐层到输出之间又经过了很多的计算单元，所以对于输出很难理解神经网络是如何决策的。

* BagNet  
借鉴bag-of-feature的思想赋予网络的设计一定的可解释性，通过图像不同的patch内容进行训练，探索分析网络的分类任务识别特点。

## 2. Network Architecture
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Approximating-CNNs-with-Bag-of-local-Features-models-works-surprisingly-well-on-ImageNet/1.png" alt="" style="width:50%" /></center>

&emsp;&emsp;流程也比较简单：先将图像分割为很多patch；然后每个patch接卷积生成向量进行类别打分判断，综合每个小块的得分可以生成原图像的响应得分热图；然后求所有字块的得分均值得到对应类别概率输出，完成分类任务。显然，该过程中唯一的可控制变量就是patch的尺寸，也是验证的切入点所在。


## 3. Result


* **heatmap**  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Approximating-CNNs-with-Bag-of-local-Features-models-works-surprisingly-well-on-ImageNet/2.png" alt="" style="width:70%" /></center>

从热图来看，最为集中的部分集中于容易识别、信息特征丰富的区域，如：企鹅的头、拖拉机的车前部，因为这些地方含有关键信息可以进行有效分类。关注两点：（1）非目标patch是否有误分类。问题较小，但是随着感受野（patch）变大，有效信息更多，误分类会明显减少  （2）目标patch是否错分类。问题很大，比如企鹅的肚子，马桶的盖子，当感受野不够提取的信息不足时，再精良的分类器也无法判断（人都做不到）。        但是可以看到，企鹅的头不管多大的patch（只要能覆盖到），都能分类地较好。  

* **RF**  
可以看到下面的patches，当感受野过小，图片信息过少，连人都很难进行物体分类，分类器就更难了。只能凭借纹理进行局部推理，所含有的信息量本身是十分有限的，作为输入的信息本身不足以进行准确的判断，断然不可能得到更好的分类效果。（~~除非CNN胜过人类，能够推理出隐更丰富的信息关联和知识，这么简单的网络显然应该是达不到的~~）
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Approximating-CNNs-with-Bag-of-local-Features-models-works-surprisingly-well-on-ImageNet/3.png" alt="" style="width:70%" /></center>


* **Comprehension**   
从BagNet的patch选择来看，扩大感受野能包含更多特征，增大了有效重点信息的容量，自然能够进一步提升性能；但是如果包含了所有重点特征，再扩大感受野去容纳更多的上下文是否有必要？从应用来看，是有必要的。检测任务中加入上下文可以减少过拟合，以及赋予网络更强的鲁棒性；即使是简单的分类任务，如果只接受关键特征作为判别信息，在复杂纹理下会很容易受到干扰和误判；复杂任务上需要更精确的定位，因此仅仅有重要特征是不够的。（要是有检测任务的BagNet，我收回这句话）



<br>
<br>
<hr />