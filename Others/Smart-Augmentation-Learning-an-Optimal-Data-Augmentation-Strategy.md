|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2017 [IEEE Acess]<p/span>

## 1. Introduction  
* 关于本文：Acess是个SCI三区最近升二区，但是据说灌水挺多。本文的引用次数还算有点人气，不过论文很多地方逻辑安排不太好，让人看得不明不白（比如abstract都没讲明白到底这个方法做什么从哪里入手；第三章的B开头提了loss也没说清楚到底参数什么含义不明不白的，直到最后才说明）。但是思路可圈可点 。
<!-- more -->

* 出发点：图像的增强是有方向性和针对性的。本文采取的自动增强手段是将任意图片的输入增强学习网络和最终任务网络一起学习，用共同loss指导图片采样增强方式。


## 2. Smart Augmentation

* 网络结构  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Smart-Augmentation-Learning-an-Optimal-Data-Augmentation-Strategy/1.png" alt="" style="width:70%" /></center>
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Smart-Augmentation-Learning-an-Optimal-Data-Augmentation-Strategy/2.png" alt="" style="width:70%" /></center>

&emsp;&emsp;大致流程：大意就是多了一个子网A生成样本，在一个类内，任意采样（或者聚类）多样本，然后合成得到输出。生成的样本作为任务网络B的训练样本输入。各个子网络通过共同进行误差的反向传播，以共同的loss指导学习。
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Smart-Augmentation-Learning-an-Optimal-Data-Augmentation-Strategy/3.png" alt="" style="width:70%" /></center>
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Smart-Augmentation-Learning-an-Optimal-Data-Augmentation-Strategy/4.png" alt="" style="width:70%" /></center>

&emsp;&emsp;多类别的话设计比较简单，采取了多个类内采样合成网络，有几个类就针对其设计几个采样网络，这样针对特定类别的样本混合能达到最好的效果。

* 损失函数  
上图不难看出，通过系数进行控制，LB是目标任务的loss，LA是合成图片与采样集的均方差。这个A的loss很奇怪，可能是想使得类内数据趋向同一，但是：1. 这样盲目均方差操作是不是能够提取类内典型特征有待商榷；2. 没有类间比较，类间差距能否识别。

## 3. Result  
实验设计和论述的多余的话太多了，就不看了，直接看结果。
* 合成效果   
左一是A子网络的合成输出，BC是随机采样的两张图。可以看到A的输出也是有一定的直观意义（并不是纯粹特征空间叠加，这样的效果供应该更好）。但是这个和GAN数据增强有什么区别？而且真的这个小网络能普遍性地实现这个效果吗，还是这个结果是特例或者做出来的？  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Smart-Augmentation-Learning-an-Optimal-Data-Augmentation-Strategy/5.png" alt="" style="width:60%" /></center>


* 训练和检测  
训练阶段AB网络一起训练，共同学习。检测阶段去掉A网络，直接输入样本到B。  
这样统一学习的好处是可以有效利用最终结果进行合成的指导，如果独立开来，最终结果将无法反应到合成网络的输入端；但是联合学习有mismatch的问题。




<br>
<br>
<hr />