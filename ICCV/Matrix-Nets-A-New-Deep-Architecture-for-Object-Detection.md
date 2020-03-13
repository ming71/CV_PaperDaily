



<span id="inline-blue">论文发布日期：2019.8.4 <p/span>


这篇文章只是挂出来还没发表，写的不详实，看看就行,很多细节没有披露，比如速度估计有问题，对比实验也没做，应该是急着占坑。
<!-- more -->

## 1. Introduction

* 问题：FPN用低级层结合高级语义特征，然后多尺度预测解决了目标检测尺度的变化问题；但比例问题仍未解决。
* 方案：提出Matrix Nets同时敏感尺度和尺寸变化,更好地处理物体的变化性。



## 2. Matrix Nets（xNets）  &emsp;&emsp;
网络结构如下。   
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Matrix-Nets-A-New-Deep-Architecture-for-Object-Detection/1.png" alt="" style="width:40%" /></center>

### 2.1  Layer Generation and  Ranges           
&emsp;&emsp;最大的问题就是相比FPN特征图明显太多了，参数多计算大（FPN已经被诟病多尺度预测计算太大）。实际有多大？论文没说，但是举了个例子：随着降采样的进行，后面的特征图感受野逐渐扩大，如L11的为[24,48]px，[24,48]px，那么L12就是[24,48]px，[48，96]px。实验最终测试缩放到长边900px，说明第一层对应的FPN尺度约900/24~900/48，也就是[18.75,38.5]的降采样，也就是32倍五步降采样的样子，和ResNet+FPN的P5差不多！这么看来确实是会减少点参数，但不知道会少多少，毕竟特征多了太多。
&emsp;&emsp;<u>更奇怪的是，最后一层稍微算一下能发现，**训练时实际上只有一个像素**...还怎么预测角点？？？有大佬能解答一下嘛？</u>

* 太靠近降采样矩阵边缘的部分不降采样，因为现实中极端宽高比的检测物体数目很少。
* 目标的尺寸若处于range边界，放宽layer的边界，实验采用的范围是[0.8, 1.3]，避免尺寸歧义导致不好学。

### 2.2  Advantages of Matrix Nets

&emsp;&emsp;分析这样work的原因是：由于不同的层具有相应的尺度和尺寸特性，因此可以更好地学习对应性质的目标，单个层的属性变化性更小，使得优化问题更容易求解，更容易回归和学习。


## 3. Key-point Based Object Detection 
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Matrix-Nets-A-New-Deep-Architecture-for-Object-Detection/2.png" alt="" style="width:50%" /></center>

&emsp;&emsp;选择CornerNet作为改进目标，先细数CornerNet的不足之处，如角点匹配策略不好，embedding特征的弊端，计算量大等。然后做了改进策略：使用xNets加持backbone，在每个特征图上分别预测角点并匹配；预测中心点坐标不再通过角点embedding进行匹配。
* Experiments

虽然特征图已经算是小了，但是训练还是很耗显存：8块 V100，bs=55.......（相比bs=20能提高0.7，单卡用户：我太难了）。训练阶段将图片裁剪成512*512，也就是最高输出为1x1（~~匪夷所思，这还怎么预测？~~）；测试阶段缩放长边900。数据增强使用了色彩抖动和cutout。

* Result
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/Matrix-Nets-A-New-Deep-Architecture-for-Object-Detection/3.png" alt="" style="width:40%" /></center>

结论而言：            
1. 收敛速度块            
2. 效果好          
3. 没有提到检测速度。（虽然是单阶段，但是囿于这么多的预测层，应该不会很快）
4. 有借鉴学习的地方








<br>
<br>
<hr />