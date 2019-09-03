
|  创建人   |  知乎论文阅读专栏 | 个人博客 | 
|  ----  | ----  | ----  | 
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |    




<span id="inline-blue">论文发布日期：2019.6.29[IJAC]<p/span>

## 1. Introduction  
&emsp;&emsp;轮廓的概念来自人类视觉的直观感受，没有精确的数学定义，根据生成方式和相关定义略有不同。例如下图是四种经典的轮廓分类，一二的光照纹理边界还好理解后面的两种感觉在通用目标检测的范畴中应该不常见，但是实际轮廓检测任务很可能是这几种轮廓的组合。 
<!-- more -->
  
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/An-Overview-of-Contour-Detection-Approaches/1.png" alt="" style="width:50%" /></center>


&emsp;&emsp;轮廓检测可以分为三种类型：基于像素、基于边缘、基于区域。
* 基于像素：根据提取的特征逐像素判断是否属于边界轮廓。
* 基于边缘：首先边缘检测，然后根据结果进行优化得到轮廓。
* 基于区域：将轮廓线为感兴趣区域的边界，因此需要考虑轮廓内部的信息进行识别。


## 2. Pixel-based approaches   
&emsp;&emsp;最早得到关注的特征是边界灰度的不连续性，为了提取这种特性有Sobel、Canny等线性滤波算子，此外还有非线性滤波器，考虑不同尺度、不同方向的滤波核等进行更完备的特征描述。虽然这些方法应用很广，但是单独检测边缘轮廓时仍容易受到歧义纹理等的干扰导致效果不好。所以最近提出的很多方法关注如何有效利用高阶特征，主要分为两类：基于大脑工作机制延伸的方法和基于自然特征的方法。      

### 2.1  Methods based on brain-inspired feature  
&emsp;&emsp;涉及人类视觉感受机制V1区，CRF/NCRF等响应机制，略。相关工作如用二维Gabor滤波器模拟CRF的刺激响应， 研究表明Gabor滤波器可以近似模拟CRF如方向和频率的神经元响应特性；还有大量关于NCRF调制机理的研究和改进工作， 实验表明，NCRF相关方法可以显著抑制纹理产生的平凡边缘，从而提高轮廓检测性能。（过于复杂无关，不细看，但是根据参考文献来看，传统方法没有完全销声匿迹，在近几年还是有相关文章的）  

### 2.2  Methods based on natural features   
&emsp;&emsp;基于自然特征的方法主要是提取图像的亮度、色彩等属性，进行建模、特征选择和判断。纹理和噪声是轮廓检测的主要干扰因素，将之与颜色、纹理梯度等属性相结合产生概率检测器，由此可以生成一些新的特征，如SCG；应用频谱图理论融入全局信息并扩展到图像分割方法的（这个挺广的）；....方法总结如下（传统方法体系略）：
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/An-Overview-of-Contour-Detection-Approaches/2.png" alt="" style="width:80%" /></center>        


### 2.3   Discussions            
&emsp;&emsp;介绍的大体特点都是概率检测器模型的基础上，进一步设计更优的特征进行描述。例如很多研究都在做的多尺度信息的综合利用和融合，从不同角度进行改进，仍存在比较大的改进空间。这部分的工作在深度学习应用上能够极大降低人工设计成本，并且由于CNN的强大特征提取能力，随便加上FPN等多尺度融合效果可能不逊色于精心设计的人工结构（~~个人推测~~）。


## 3. Edge-based approaches 
### 3.1  Contour extraction            
&emsp;&emsp;基于边缘的轮廓提取可以分为两步：边缘检测，轮廓提取。这部分主要阐述了基于边缘检测结果的分组方法。边缘元素作为度量单元，其属性服从Gestalt laws，具有similarity, proximity, continuity, symmetry, parallelism, closure and familiarity等性质。该类方法有加入贝叶斯推断分组、显著性增度量、基于图论的方法。     

### 3.2  Contour completion           
&emsp;&emsp;轮廓的闭合性，闭合原则具有全局性能够更好地提取需要的轮廓。但是边缘检测一般都是碎片化的，而且由于噪声干扰使得闭合轮廓在实现上并不简单。解决方法如采用条件随机场CRF（只能解决小的间隙） ；也有文献采用了较大轮廓的填充方案，但是填充曲线明显会偏离原来的图案。


## 4. Region-based approaches    
&emsp;&emsp;这种方法会关注轮廓图像的内部信息，实际上和分割任务有几分相似。这方面的工作相对较少，主要是从度量方法定义和能量函数的构造出发。这种思想的优点是考虑全局的影响（轮廓内外），但对噪声干扰更具有鲁棒性。

## 5. Deep-learning based approach
&emsp;&emsp;不感兴趣部分，略。



<br>
<br>
<hr />