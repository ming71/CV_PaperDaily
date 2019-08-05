|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2019.1.10 [CVPR]<p/span>

## 1. Motivation
* anchor人为经验设置引入超参数，设计复杂；而且一旦设置不合理会影响效果     
* 检测依赖密集的anchor，计算量大
* 绝大多数的anchor是负样本，不参与回归预测  

&emsp;&emsp;探究了anchor-based框架设计anchor两个重要因素：<u><font color=red>**alignment and consistency**</font>。**前者指的是anchor的中心和特征图像素进行对齐，从而可以滑窗产生密集的anchor确保对物体的覆盖；后者是不同尺度的特征图使用不同尺度的anchor检测才能达到很精确的效果**。</u>如果使用学习生成的anchor，需要解决如何满足上述两个条件。  
<!-- more -->
## 2. Guided Anchoring 
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Region-Proposal-by-Guided-Anchoring/str1.png" alt="" style="width:100%" /></center>

### 2.1 Anchor Location Prediction             
&emsp;&emsp;该模块使用子网络<u>预测特征图每一点作为物体中心点的可能性，这些响应区域会作为anchor中心</u>，（和RPN的前景背景二分类不同！看loss部分的label就知道，label是取得中心位置为1而无关前景背景）.<u>解决了alignment的问题</u>。由于进行了位置的响应，可以剔除背景，因而需要放置的anchor可以大大减少。            
&emsp;&emsp;位置预测对应右边上面的location预测，将特征图通过1x1卷积压缩到单通道，然后接像素级sigmoid对每个像素有无物体进行打分。（作者提到，<u>尽管这里用深层的子网络效果更好，但是经验来说直接卷积+sigmoid可以更好平衡精度和速度</u>）；            
&emsp;&emsp;对得分图按照一定阈值进行筛选，得到物体位置响应图，可以滤去90%的背景（如下图）； <center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Region-Proposal-by-Guided-Anchoring/exp1.png" alt="" style="width:50%" /></center>             
&emsp;&emsp;在inference阶段不用考虑背景，预测完位置之后使用mask卷积替代普通卷积，只在有anchor的地方计算，进行加速   

### 2.2 Anchor Shape Prediction            
&emsp;&emsp;在上面响应了位置信息，知道哪些位置是anchor后，需要确定anchor的形状，进一步解决consistency的问题。   
&emsp;&emsp;对于shape prediction分支，输入不是上面的位置得分图而是直接从原图进行学习，直接学习每个像素点的wh信息。这里和RPN不同，不需要考虑定位和特征图尺度适配问题，只负责预测形状。但是<u>由于物体尺度变化范围很大直接学习wh仍可能导致计算不稳定</u>，采取学习如下dw,dh：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Region-Proposal-by-Guided-Anchoring/format1.png" alt="" style="width:50%" /></center>
其中缩放因子sigma=8，s为特征图的降采样步长，这样一来将训练目标范围从[0,1000]约束到[-1,1],(例如，考虑s为32倍下采样的尺度图sigma=8，[-1,1]能负责检出[94,696]的尺度，其他尺度计算类似)。             
&emsp;&emsp;最后是两个1*1卷积压缩得到两张特征图预测的dw和dh。通过上面公式的变换可以得到每个位置的anchor形状（一个位置只有一个anchor）。

### 2.3 Anchor-Guided Feature Adaptation            
&emsp;&emsp;**anchor和特征图匹配的问题**：某个尺度特征图的感受野是固定的，但是要基于它生成的anchor大小是通过shape预测分支得到的，所以形状并不一样（RPN中anchor都一样），这就产生了矛盾：理论上大的anchor应该在更大感受野的特征图上预测，小的反之，而不是放到统一尺度特征图预测。            
&emsp;&emsp;解决办法：<u>把anchor的形状信息直接融入到特征图中，这样新得到的特征图就可以去适应每个位置anchor的形状，**从而解决consistency的问题**。</u>我们利用一个3x3的deformable convolution来修正原始的特征图，而 deformable convolution 的 offset 是通过 anchor 的 w 和 h 经过一个 1x1 conv 得到的。通过这样的操作，达到了让 feature 的有效范围和 anchor 形状更加接近的目的，同一个 conv 的不同位置也可以代表不同形状大小的 anchor 了。        
### 2.4 Training
#### 2.4.1 联合训练目标函数：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Region-Proposal-by-Guided-Anchoring/format2.png" alt="" style="width:60%" /></center>
&emsp;&emsp;除了传统的分类回归外，还引入了anchor位置和形状两项。
#### 2.4.2 Anchor location targets
关于位置损失的计算：       
(1) label：      
&emsp;&emsp;训练Location Prediction分支时，label为特征图的二进制mask，正样本是1 ；        
&emsp;&emsp;样本划分：将原图映射到特征图大小后，希望尽可能中心的位置分配多anchor，边缘的少分配，这样放置的anchor更容易回归。按照这个原则如下图最上面特征图为例，红色框是gt对应的位置，而正样本定义是（xywh）中xy固定，wh同时缩小一定系数sigma_1，得到的绿色区域为正样本；而按照另一个系数sigma_2缩放得到黄色区域是忽略的样本，剩下的才是负样本。其中正样本范围的二进制label为1，负样本0，忽略样本不计算。<br>     
(2) loss: focal loss
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Region-Proposal-by-Guided-Anchoring/str2.png" alt="" style="width:80%" /></center>

#### 2.4.3 Anchor shape targets（借用作者的描述）
&emsp;&emsp;形状预测分支的目标是给定 anchor 中心点，预测最佳的长和宽，这是一个回归问题。按照往常做法，先算出target，也就是该中心点的anchor最优的w和h然后用L1/L2/Smooth L1这类loss来监督。然而这玩意的target并不好计算，而且实现起来也会比较困难，所以我们直接使用IoU作为监督，来学习w和h。既然我们算不出来最优的w和h，而计算IoU又是可导的操作，那就让网络自己去优化使得IoU最大吧。      
&emsp;&emsp;损失函数借鉴了bounded IoU Loss，而不是直接优化wh，如图： 
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Region-Proposal-by-Guided-Anchoring/format3.png" alt="" style="width:70%" /></center>        

&emsp;&emsp;这里面还有个问题，就是对于某个anchor，应该优化和哪个ground truth的IoU，也就是说应该把这个anchor分配给哪个ground truth。对于以前常规的anchor，我们可以直接计算它和所有 ground truth的IoU，然后将它分配给IoU最大的那个gt。但是很不幸现在的anchor的w和h是不确定的，是一个需要预测的变量。    
&emsp;&emsp;当然我们不可能真的把所有可能的w和h遍历一遍然后求IoU的最大值，所以采用了近似的方法，也就是 *sample一些可能的w和h*。理论上sample得越多，近似效果越好，但出于效率的考虑，我们 sample了常见的9组w和h。我们通过实验发现，最终结果对sample的组数这个超参并不敏感，也就是说不管sample多少组，近似效果已经足够。          
&emsp;&emsp;（实际上还是手动设置wh，但是他说对这个超参数不敏感，如果真的如此，那就算还行）

## 3. Ablation Study
### 3.1 Model design
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Region-Proposal-by-Guided-Anchoring/exp2.png" alt="" style="width:80%" /></center>
&emsp;&emsp;可以看出anchor形状的预测占了主导性的优势，因为他是从九个anchor中脱颖而出的，位置预测带来的proposal召回率提升不大，但是其意义在于减少背景anchor的生成，加速计算。

### 3.2 Anchor location
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Region-Proposal-by-Guided-Anchoring/exp3.png" alt="" style="width:80%" /></center>
&emsp;&emsp;设置很小的阈值就能剔除大量的背景anchor，效果拔群，而且对于精度的降低并不大。

### 3.3 其他实验
* Anchor shape.
* Feature adaption
* Alignment and consistency rule
* The use of high-quality proposals   

&emsp;&emsp;使用GA之后带来两个优势：  
&emsp;&emsp;(1) 可以使用更高的IoU阈值，因为相比RPN GA可以生成更多的高质量proposal   
&emsp;&emsp;(2) 训练时采用更少的proposal。往往proposal数目少会带来召回率降低查全低下，但是GA-RPN的查全率已经很高了吗，因此使用更少的proposal就能达到效果，这对训练和检测都是有益的。

### 3.4 Hyper-parameters
&emsp;&emsp;取的anchor默认wh对依次为3-9-15，AR为68.3%-68.5%-68.5%，可见取多了确实没大的用处（实际上，在Faster-RCNN中，作者实验证明，加了更多尺度作用也没那么大，只有0.1%这就值得思考了）


## Conclusion:
* **创新点**：
1. <u>减少了大量的anchor负样本生成，背景anchor的剔除，降低计算；并且生成的proposal召回率更高</u>
2. 不用过于依赖手动设置的anchor(~~这一点有待考证，因为它实际是通过更多组的anchor进行候选的~~)
3. 使用更少的proposal但是获得更好的效果
4. <u>**提出了 alignment 和 consistency 这两个设计anchor的思想**，并且尝试解决：前者通过预测中心概率图实现，后者通过学习anchor形状实现，这种思想很具有参考价值。</u>更进一步，也是多尺度特征图检测的inconsistency问题。

* 实现结构  
&emsp;&emsp;为了实现anchor的自生成，通过anchor的中心预测概率图确定xy，anchor形状预测wh（~~但是这里有手动参数，尽管作者说不敏感，但是证据不充分~~）；为了解决尺度匹配的问题引入可变性卷积，融合anchor shape和特征图信息。
* 个人感觉：  
&emsp;&emsp;生成wh的方法还是比较因循守旧的，不过提出的中心点xy预测思路很不错，预测中心概率图来剔除大量的背景样本，大大提高检测效率；并且进一步通过可变性卷积特征融合以及改进的shape预测将proposal质量提高了；为网络提供更少、质量更高的proposal  


<br>


<hr />