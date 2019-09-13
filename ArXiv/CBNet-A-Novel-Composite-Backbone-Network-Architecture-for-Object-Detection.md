| 创建人 |                       知乎论文阅读专栏                       |              个人博客               |
| :----: | :----------------------------------------------------------: | :---------------------------------: |
| ming71 | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |



<!-- more -->

<span id="inline-blue">论文发布日期：2019.9.3 <p/span>

最高单模型map COCO 53.3

## 1. Introduction

&emsp;&emsp;名义上是单模型，实际是多模型的特征融合，只是和真正的多模型策略略有不同。作者的起点是，设计新的模型往往需要在ImageNet上进行预训练，比较麻烦。因而提出的Composite Backbone Network (CBNet)，采用经典网络的多重组合的方式构建网络，一方面可以提取到更有效的特征，另一方面也能够直接用现成的预训练参数（如ResNet，ResNeXt等）比较简单高效。（~~这个出发点也太凑合了吧.....~~）



## 2. Proposed method
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/CBNet-A-Novel-Composite-Backbone-Network-Architecture-for-Object-Detection/1.png" alt="" style="width:50%" /></center>

### 2.1  **Architecture of CBNet**  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/CBNet-A-Novel-Composite-Backbone-Network-Architecture-for-Object-Detection/2.png" alt="" style="width:50%" /></center>

&emsp;&emsp;如上图，模型中采用K个（K>1）相同的结构进行紧密联结。其中两个相同backbone的叫Dual-Backbone (DB)，三个叫Triple- Backbone (TB)；L代表backbone的stage数目，这里统一设置为L=5。其中，和前任工作不同的地方在于，这里将不同的stage信息进行复用回传，以便获取更好的特征（为什么work不好说）。        

### 2.2  Other possible composite styles          
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/CBNet-A-Novel-Composite-Backbone-Network-Architecture-for-Object-Detection/3.png" alt="" style="width:50%" /></center>

&emsp;&emsp;相关工作的其他类似结构，大同小异。要么是前面backbone的stage往后传播，要么是往前一个传播，每个都有一篇论文，应该都会给出不同的解释；第四个结构不太一样，是类似densnet的结构，但是密集连接+多backbone assemble的内存消耗不出意外会非常大。但是脱离这些体系来看，多backbone的结构类似多模型的assemble，和单模型有点不公平。



## 3. Experiment


* **result**  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/CBNet-A-Novel-Composite-Backbone-Network-Architecture-for-Object-Detection/4.png" alt="" style="width:65%" /></center>

COCO数据集上的结果。看来提升还是有的。但是也能看出，大趋势上，三阶级联效果不如两阶的提升大，也是这部分的特征提升空间有限的缘故，到底哪部分在work不好说。下图的研究就更说明这一点了，斜率逐渐减小。


* **Comparisons of different composite styles**  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/CBNet-A-Novel-Composite-Backbone-Network-Architecture-for-Object-Detection/5.png" alt="" style="width:50%" /></center>

他的级联网络相比，作者的阐述点只落脚于特征的利用情况，但是这个东西本身就很玄乎，不好说到底怎么算利用得好。硬要说这种做法的解释性，大概就是将backbone方向的后面高级语义特征传播回前面进行加强，相当于横向的FPN传播。


* **Number of backbones in CBNet**  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/CBNet-A-Novel-Composite-Backbone-Network-Architecture-for-Object-Detection/6.png" alt="" style="width:50%" /></center>

速度慢是必然的，FPN+ResNeXt为8fps，加上两个backboen后为5.5FPS；如果减去backbone的前两个stage，可以节省部分参数达到6.9FPS，而精度下降不大（整体速度太低，这个实验意义不大）

* **Sharing weights for CBNet**    
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/CBNet-A-Novel-Composite-Backbone-Network-Architecture-for-Object-Detection/7.png" alt="" style="width:50%" /></center>
* 
从中可以看出其实权重是否share区别不大， 不到一个点的降幅，参数量减少。


* **Effectiveness of basic feature enhancement by CBNet**   
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/CBNet-A-Novel-Composite-Backbone-Network-Architecture-for-Object-Detection/8.png" alt="" style="width:50%" /></center>

从中可以看出激活响应效果更好，确实是能够提取到更为有效的特征，对物体的响应更加敏感。










<br>
<br>
<hr />