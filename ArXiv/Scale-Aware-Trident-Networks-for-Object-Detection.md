|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2019.1.1<p/span>

## 1.Motivation  
&emsp;&emsp;为了应对尺度不变性，一般有两种方法，各有优劣:
* 图像金字塔：计算量太大，但是特征尺度可以是连续的  
* 特征金字塔：效率高，但特征尺度受限于backbone结构，一般是不连续、固定的尺寸
两种方法的共同出发点都是**对于不同尺度物体的检测应该提供不同大小的感受野，因此可以从感受野的角度出发更好改进特征提取过程，使模型具有较好的尺度不变性。**
<!-- more --> 

## 2.Investigation of Receptive Field  
&emsp;&emsp;首先做实验研究了一下感受野对检测任务的影响（~~作者自称第一个研究感受野对目标尺度的影响，其实肯定不是，这个idea还是不难想到的~~不过这个insught确实值得思考，和cascade rcnn的IoU分布一样，一说就懂，但是又很有道理，需要洞察力）： 
<img src="http://chaserblog.test.upcdn.net/blogs/paper/Scale-Aware%20Trident%20Networks%20for%20Object%20Detection/exp1.png!" alt="" width = 60%/>  
&emsp;&emsp;从表中可以读出两点信息：
* 使用膨胀卷积增大感受野后，小目标检测效果劣化，大目标效果更佳；这很好理解，小目标尺度本来就小，感受野一大，目标的很多部位直接被0乘了，没有参与特征提取，大目标相反。这说明：**检测特征图的感受野应该和被检测物体尺度match**。  
* 横向对比一下backbone，很有趣的是，越深的backbone提取的特征越充分，对感受野变大的情况下，大目标的性能提升变小，而小目标下降变得更加明显。可能是因为：深度网络的高层语义更加抽象了，因此预测大物体的性能本身就提高了，增大感受野只是锦上添花不会太明显，较浅的ResNet-50本身大尺度还不够好，所以提升明显；而小尺度上，本来加了膨胀卷积就容易丢失小目标，加的越多上层特征图感受野越大丢的越多，那么自然小目标劣化更严重


## 3.Trident Network  
<img src="http://chaserblog.test.upcdn.net/blogs/paper/Scale-Aware%20Trident%20Networks%20for%20Object%20Detection/net.png!//watermark/text/Y2hhc2VycyBibG9nOiBodHRwczovL21pbmc3MS5naXRodWIuaW8v/align/southeast/font/helvetica/animate/true" alt="" width = 100%/>  

### 3.1 Network Structure
* Multi-branch Block
（这个不太好理解）目的是引入不同膨胀率带来的不同感受野的并行卷积分支。具体操作：
1. 在backbone的最后一个stage加上N个trident block膨胀卷积模块，并且其具有不同的膨胀率（选择最后stage因为其更大的降采样步长在加上不同膨胀率后可以带来更大的感受野变化<u>*但是只会在最高层基础上扩大感受野，那小尺度的特征呢？*</u>）trident block结构如下图红色。  
2. 为了共享参数，实际上加的三个block除了膨胀系数不同外，其他参数均共享，loss通过三个分支共同指导卷积核（膨胀卷积见下方，只是卷积的方式变了，卷积核还是不变的）
3. 多个分支对应着不同尺度提取特征的多个输出，如下图：  
<figure class="half">
    <img src="http://chaserblog.test.upcdn.net/blogs/paper/Scale-Aware%20Trident%20Networks%20for%20Object%20Detection/dilatedconv.gif!//watermark/text/Y2hhc2VycyBibG9nOiBodHRwczovL21pbmc3MS5naXRodWIuaW8v/align/southeast/font/helvetica/animate/true" alt="" width = 20%/>
    <img src="http://chaserblog.test.upcdn.net/blogs/paper/Scale-Aware%20Trident%20Networks%20for%20Object%20Detection/branches.png!//watermark/text/Y2hhc2VycyBibG9nOiBodHRwczovL21pbmc3MS5naXRodWIuaW8v/align/southeast/font/helvetica/animate/true" alt="" width = 70%/>
</figure>

* Weight sharing among branches
**不同dilated conv系数的分支之间共享参数，可以降低参数过多带来的过拟合风险**（**~~学套话~~**），其主要思想<u>使用一套参数去适应不同感受野下对不同尺度目标的检测架构，让网络具备通过多重感受野学习不同尺度特征的能力。</u>

### 3.2 Scale-aware Training Scheme  
&emsp;&emsp;借鉴SNIP的思路，将不同尺度的目标用不同尺度的分支进行训练，而不是一股脑用全部尺度目标去训练各个分支。  
&emsp;&emsp;具体操作：以RPN为例，对于三个不同感受野分支，设置各自的面积区间，如果gt的面积wh落入对应的区间，则通过对应的分支进行特征提取和学习；如果是RCNN，则计算的是RPN输出的proposal的wh，同样划分不同的分支然后提取特征和输出。  
&emsp;&emsp;<font color=red>**其中的思想**:</font>使用不同尺度的物体来训练对应尺度的分支（避免出现极端大物体指导小感受野的分支或者极端小物体训练大感受野的分支这类错误指导的情况，这也是基于前面的实验观察：感受野和检测目标尺度匹配可以取得更好效果的思想），使得该分支具有对特定尺度更好的适应性，同时通过一套参数共享实现网络本身对尺度不变性的适应。（如果这个地方用多分支不共享参数，那么就落入FPN的anchor与感受野mismatch的陷阱了） 


### 3.3 Inference and Approximation  
&emsp;&emsp;检测阶段，对不同分支预测出bbox后，筛掉与本分支尺度不符的bbox，剩下的bbox全部进行NMS。  
&emsp;&emsp;Fast Inference Approximation：作者还观察发现，如果只近似取中间分支预测，并且不根据尺度筛bbox（尺度范围 0-+∞），性能只是稍微下降了一点点，但是速度有所提升。  
&emsp;&emsp;原因分析：只取中间分支，解除面积区间限制后效果仍不差，说明该尺度对其他尺度的bbox预测效果也不俗，<u>应该是参数共享的结果</u>，参数本身使得各个分支具有一定的其他尺度的推理能力，所以如果不限制面积也能输出不太差性能的bbox。<u>*那么参数不共享是不是在特定尺度效果更好？*：事实不是这样，作者实验认为是泛化能力不如共享参数来的好，这一点有待考证</u>  


## 4.Ablation Study
&emsp;&emsp;作者的对比实验十分充分，非常详细，做得很好。三个尺度的尺寸为[0, 90], [30, 160] ，[90,∞]，后面不难看出这个是精心设置的。  
* Components of TridentNet
三个部分：多分支预测、参数共享、特定尺度训练，均有所提升其中前两者提升更为明显.
1. 特定尺度训练:  
针对特定尺度设置尺度区间进行区分性训练，会导致大目标的效果降低，小目标增强，作者认为是<u>由于针对特定尺度图片筛选过输入目标后，困难样本被剔除导致模型训练过拟合导致的（<font color=red>**困难/负样本的重要性！**:</font>）</u>。所以在设置范围区间时可以看到，三个区间并不是严格没有交叉，而是比较宽泛，为了减轻这种过拟合 （还有很多其他办法）
2. 参数共享：  
加上了参数共享之后提升了性能，作者认为是三个分支训练不同的尺度，参数共享可以减轻上面的过拟合问题。可见单一尺度的独立特征检测虽然准确但是会面临过拟合的问题（如FPN的在各自尺度的预测），共享参数也有其独特的优势。
* Number of branches
和Cascade RCNN一样，发现是深度到一定程度后就饱和了，三分支最好，~~但是这里的实验不够多，不知道具体的变化是哪个尺度的影响~~。
* Stage of Trident blocks  
<img src="http://chaserblog.test.upcdn.net/blogs/paper/Scale-Aware%20Trident%20Networks%20for%20Object%20Detection/exp2.png!" alt="" width = 60%/>    
对特定的stage加上Trident blocks，这里发现采用低层次的conv2,3效果都不如4（但是他也没试一下conv5我觉得应该变差了），最终选择的也是16降采样的conv4。作者分析是大尺度特征图效果不足以很好的区分不同膨胀系数下的感受野，比如降采样为4，那么加上膨胀卷积后三个分支的感受野差距也不足以覆盖整个检测目标的面积分布范围。（~~那么你加大膨胀系数不就完了？~~）
* Number of trident blocks  
<img src="http://chaserblog.test.upcdn.net/blogs/paper/Scale-Aware%20Trident%20Networks%20for%20Object%20Detection/exp3.png!" alt="" width = 60%/>     
前面说过对每个分支要使用N个trident blocks这里研究发现10-15个差不多了
* Performance of each branch     
<img src="http://chaserblog.test.upcdn.net/blogs/paper/Scale-Aware%20Trident%20Networks%20for%20Object%20Detection/exp2.png!" alt="" width = 60%/>  
<img src="http://chaserblog.test.upcdn.net/blogs/paper/Scale-Aware%20Trident%20Networks%20for%20Object%20Detection/exp2.png!" alt="" width = 60%/>  
不同分支还是具有对不同尺度的区别能力，只是中间分支的效果最强，所以在fast检测时选择branch-2。其实他的尺度设置branch2是30-160，除了中间尺度外，也涵盖了部分的大尺度（>96）与小尺度（<32）所以效果最好也是合理的；
右侧图中，面积覆盖广效果越好，不难理解，因为这样提供的bbox越多，自然NMS后得到的效果越好，但是NMS计算成本就上去了。

## **Conclusion**
<font color=red>**本文亮点在于从感受野的角度合理设计多尺度特征的提取和学习,摆脱了传统的FPN特征融合模式**</font>。  
* **检测特征图的感受野和被检测对象的尺度match时，得到的检测性能更好**
* **用一种不同于FPN的新角度实现对图像尺度不变性的学习--基于检测特征图的感受野应该和被检测物体尺度match的观察设计了不同感受野分支学习不同尺度下的目标。**
* **借鉴SNIPER设计了正样本的选择方法：Scale-aware Training Scheme ，同时调整尺度范围协调正负样本（多尺度训练也是不能只有正样本，否则鲁棒性不强）**
* **参数共享，在一个框架下提高了网络的尺度不变性和泛化能力**
* **由于摆脱FPN结构，所以大尺度效果下降不大；并且采用中尺度预估时计算量也小多了；由于参数共享，即使单尺度预测效果也不像FPN下降明显**


<br>
<hr />