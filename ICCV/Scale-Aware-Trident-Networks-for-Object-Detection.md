|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2019.1.1 [ICCV] <p/span>

## 1.Motivation  
&emsp;&emsp;为了应对尺度不变性，一般有两种方法，各有优劣:
* 图像金字塔：计算量太大，但是特征尺度可以是连续的  
* 特征金字塔：效率高，但特征尺度受限于backbone结构，一般是不连续、固定的尺寸

<!-- more --> 

## 2.Investigation of Receptive Field  
&emsp;&emsp;首先做实验研究了一下感受野对检测任务的影响（~~作者自称第一个研究感受野对目标尺度的影响，其实肯定不是，这个idea还是不难想到的~~不过这个insight确实值得思考，和cascade rcnn的IoU分布一样需要洞察力）： 
<img src="http://chaserblog.test.upcdn.net/blogs/paper/Scale-Aware%20Trident%20Networks%20for%20Object%20Detection/exp1.png!" alt="" width = 60%/>  
&emsp;&emsp;从表中可以读出两点信息：
* 使用膨胀卷积增大感受野后，小目标检测效果劣化，大目标效果更佳；这很好理解，小目标尺度本来就小，感受野一大，目标的很多部位直接被0乘了，没有参与特征提取，大目标相反。      
* 横向对比一下backbone，很有趣的是，越深的backbone提取的特征越充分，对感受野变大的情况下，大目标的性能提升变小，而小目标下降变得更加明显。可能是因为：深度网络的高层语义更加抽象了，因此预测大物体的性能本身就提高了，增大感受野只是锦上添花不会太明显，较浅的ResNet-50本身大尺度还不够好，所以提升明显。  


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
**不同dilated conv系数的分支之间共享参数，可以降低参数过多带来的过拟合风险**

### 3.2 Scale-aware Training Scheme   
&emsp;&emsp;借鉴SNIP的思路，将不同尺度的目标用不同尺度的分支进行训练，而不是一股脑用全部尺度目标去训练各个分支。  
&emsp;&emsp;具体操作：以RPN为例，对于三个不同感受野分支，设置各自的面积区间，如果gt的面积wh落入对应的区间，则通过对应的分支进行特征提取和学习；如果是RCNN，则计算的是RPN输出的proposal的wh，同样划分不同的分支然后提取特征和输出。  
） 


### 3.3 Inference and Approximation   
&emsp;&emsp;检测阶段，对不同分支预测出bbox后，筛掉与本分支尺度不符的bbox，剩下的bbox全部进行NMS。  
&emsp;&emsp;Fast Inference Approximation：作者还观察发现，如果只近似取中间分支预测，并且不根据尺度筛bbox（尺度范围 0-+∞），性能只是稍微下降了一点点，但是速度有所提升。  



## 4.Ablation Study
&emsp;&emsp;作者的对比实验十分充分，非常详细，做得很好。三个尺度的尺寸为[0, 90], [30, 160] ，[90,∞]，后面不难看出这个是精心设置的。  
* Components of TridentNet  
三个部分：多分支预测、参数共享、特定尺度训练，均有所提升其中前两者提升更为明显.
1. 特定尺度训练:  
针对特定尺度设置尺度区间进行区分性训练，会导致大目标的效果降低，小目标增强。在设置范围区间时可以看到，三个区间并不是严格没有交叉，而是比较宽泛，为了减轻这种过拟合 （还有很多其他办法）
2. 参数共享：  
加上了参数共享之后提升了性能，作者认为是三个分支训练不同的尺度，参数共享可以减轻上面的过拟合问题。

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



<br>
<hr />