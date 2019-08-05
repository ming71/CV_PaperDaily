|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2019.5.21 [CVPR]<p/span>

## 1. Insight
&emsp;&emsp;本文的工作是进行红外热图的目标检测改进。红外热图如下。我感觉效果差在，由于颜色空间的压缩，红外图的很多细节纹理丢失了，所以精度不可避免地下跌。
<!-- more -->
  
&emsp;&emsp;为了解决这一问题，本文采取从RGB图像“借”特征来补充红外图像特征不明显的劣势，进而提高检测精度。        
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Pseudo-Multi-modal-Object-Detection-in-Thermal-Imagery/exp.png" alt="" style="width:75%" /></center>

## 2. Methodology
### 2.1 结构介绍    
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Pseudo-Multi-modal-Object-Detection-in-Thermal-Imagery/str.png" alt="" style="width:90%" /></center>    

&emsp;&emsp;pipeline比较简单。首先将输入红外图通过GAN生成RGB图像，然后同时进行输入，将最后的特征图concatenate，送到head检测。        
### 2.2 相关细节和实验分析
* 参数初始化  
&emsp;&emsp;两张图片的各自特征提取分支选用对应的预训练模型，如红外输入分支用红外预训练backbone，RGB输入分支选择ImageNet预训练参数初始化；因为最终还是回到红外图的检测上，所以RPN选择红外权重初始化。但遗憾的是，论文没说怎么训练，是不是端到端的。
* I2I结构  
&emsp;&emsp;GAN使用了两种网络分别实验，均能够完成红外到彩色图像生成（一个是NIPS的，一个是ICCV的，被引挺高），最后效果都比不融合RGB特征的效果好很多，证明这个idea确实work。
* 模型大小  
&emsp;&emsp;作者自己都没意识到还有一个好处（也是一个疑点）：红外数据不如常见的RGB数据容易获取，数据集不大，为了match数据，模型选择也不能太深，所以效果一般不会显著。但是将红外转为RGB后，本质就是RGB图像的检测了，这个就能采用更深的目标检测模型了。（当然，红外图像还原成RGB这部分分支的设计还是不能过大，因为它本质来自于红外小数据）但是作者并没有加深下面的分支，而是保守地使两个分支参数容量一致。其实可以进一步实验的，甚至可以尝试一下参数共享。  

## Conclusion  
&emsp;&emsp;我关注这篇文章也正是想看看他是不是在**形状识别**上做了工作，不过看完才发现本文并没有想到这一点，而是从简单的纹理生成再检测的思路着手。虽然这样也是一种思路，但若只是想利用形状特征来增强检测，这个流程的计算代价和复杂度显然太大了。  
&emsp;&emsp;容易产生一个误解：既然红外和彩色图像的目标位置不变，这篇文章怎么不直接选择GAN输出的彩色图片去检测？如果用这种办法，就把性能的提高完全依赖在GAN设计的合理性上了，这样未尝不可尝试；而作者强调的是“borrow”，RGB图像只是为红外图提供更丰富的特征，并不是完全依赖其定位的，两者出发点不同。  
&emsp;&emsp;这篇文章是CVPR 2019的workshop，虽然方法很简单，很多地方明显还能有更多的尝试，不过做出了该方向比较大的突破，所以被放到workshop也不奇怪。  
&emsp;&emsp;<u>最近像这种，以及压缩重建抵御对抗样本这样的还挺有意思的，思路简单，方法简单，但是效果拔群。</u>

<br>
<br>

<hr />