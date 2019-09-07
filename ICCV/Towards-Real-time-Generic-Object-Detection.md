|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2019.3 [ ICCV 2019]<p/span>

## 1. Motivation  
&emsp;&emsp;进一步减小计算开销，使得两阶段的检测网络能够在移动嵌入式设备上实现实时运行。CNN可分为backbone和detect head两部分，从两个角度同时改进，提取更好的特征，更有效检测目标的同时轻量化网络。


## 2. ThunderNet  
<img src="http://chaserblog.test.upcdn.net/blogs/paper/ThunderNet/Net.png" alt="" style="width:90%" />

### 2.1 Backbone Part
* **Input Resolution**
&emsp;&emsp;两阶段检测器为了获得较好的效果输入一般尺寸大于800像素，这里为了速度起见输入为320*320。作者还有一个观察：**输入分辨率应该match模型的体量，两者不匹配效果都不好**。（很好理解也很关键，在设计输入resize时需要注意这一点） 
<!-- more --> 
* **Backbone Networks**

&emsp;&emsp;语义差异：分类和回归对于backbone的需求各不相同，前者需要的是高层特征图抽象的语义信息，后者更注重低层特征图精细的定位信息。（对于一般的网络而言，误差更多出现在底层的定位上，经过实验很容易发现回归损失是不容易收敛的，分类损失则比较容易，所以可以采取增强底层信息的表达能力，加深通道的方法）。所以在之前SNet中采取移除C5而**在早期阶段加更多的通道提升定位性能**；但是这样会导致高层语义信息编码的不足，这里<u>采取保留C5但是压缩其通道为512，然后增加底层通道以权衡两者的性能</u>。         
&emsp;&emsp;感受野：采用深度分离5*5卷积替代所有3*3，实际运行中这样的替换并没有太大加长运行时间，但是获得了更大的感受野，对于定位也是有帮助的。              
&emsp;&emsp;很多轻量化的网络只专注了底层或高层一个方面，作者这里通过加深底层特征同时保留高层语义，兼顾了两者。        

### 2.2 Detection Part

#### 2.2.1 Compressing RPN and Detection Head
&emsp;&emsp;将RPN的256通道3*3卷积压缩为5*5深度可分离卷积和1*1卷积，使用了5尺度5比例25个anchor；        
&emsp;&emsp;与light-head类似，沿用R-FCN的结构，进一步位置敏感特征图通道数压缩到5；        
减少了测试阶段的RoI数目。

#### 2.2.2 Context Enhancement Module
&emsp;&emsp;设计了CEM模块用于扩大感受野（FPN可以融合上下文做到，但是多分支卷积核预测计算量大；GCN也是计算量大）。        
<img src="http://chaserblog.test.upcdn.net/blogs/paper/ThunderNet/str1.png" alt="" style="width:60%" />
&emsp;&emsp;在C5上全局均值池化，然后将特征图1*1通道扩展到245上采样进行融合，融合多尺度的局部和全局特征。

#### 2.2.3 Spatial Attention Module  
&emsp;&emsp;思想：希望检测的特征图能够更多关注前景忽略背景，而<u>**RPN训练了物体的前景背景二分类，融合利用RPN的分类信息可以帮助特征图聚焦前景**</u>。        
&emsp;&emsp;实现方法：对RPN通道变换(这里为了减少计算用的1x1)和CEM输出的245通道一致，然后接BN，再用sigmoid将参数归一化后作为权值乘到CEM特征图上进行加权。实现了对RPN前景背景信息的利用。额外的好处在于由于信息的融合，反向传播时，RCNN子网络也会帮助监督RPN的训练。  
<img src="http://chaserblog.test.upcdn.net/blogs/paper/ThunderNet/str2.png" alt="" style="width:70%" />          




## 3. Ablation Study


### 3.1 Input Resolution
<img src="http://chaserblog.test.upcdn.net/blogs/paper/ThunderNet/exp1.png" alt="" style="width:65%" />
&emsp;&emsp;**模型大小和输入分辨率应该match**。很好理解，大模型小输入，提供的基础特征就不够细节，特征表达上细节特征很容易丢失，模型容量很难补救；小模型大输入，模型的特征提取和组合能力有限，对信息的抽象和重组不足。        
&emsp;&emsp;所以应该追求模型体积和输入尺寸的匹配。
### 3.2 Backbone Networks

* 5×5 Depthwise Convolutions：

&emsp;&emsp;大尺寸卷积核的有效感受野大，这个大家都知道。不过也没涨多少点。

* Early-stage and Late-stage Features：

&emsp;&emsp;最高的conv5去掉了会影响分类和回归的精度，可见高级语义特征还是要保留着；减少底层的通道数会损害检测精度；较好的权衡是高维通道压缩，低维通道加深，兼顾分类和回归需要的语义信息。
### 3.3 Detection Part
<img src="http://chaserblog.test.upcdn.net/blogs/paper/ThunderNet/exp2.png" alt="" style="width:70%" />
<img src="http://chaserblog.test.upcdn.net/blogs/paper/ThunderNet/exp3.png" alt="" style="width:60%" />
&emsp;&emsp;可见CEM融合多尺度的还有点用，SAM也有点用效果稍微差点。（但是都没敢用trick，说不定用了效果就微乎其微了）
&emsp;&emsp;右图是SAM聚焦的前景，看上去还有点样子。

### 3.4 Balance between Backbone and Detection Head


&emsp;&emsp;一般而言**backbone和head的大小也要match**。作者还对比了一下mismatch的backbone和head：large-backbone-small- head 和 small-backbone-large-head，结论是前者更好，也很容易想，巧妇难为无米之炊，特征都不够怎么进行检测。当然，还是match一点的好，不然要么浪费了计算backbone提取的特征，要么特征不够检测效果不好。


## Conclusion:
* **输入分辨率应该match模型的大小，两者不匹配效果都不好**
* **backbone和head的大小也要match**
* **很有意思的思路：利用RPN的二分类信息训练前景监督器，使模型更加集中关注前景（但是怎么更有效地利用是个问题）**







<hr />