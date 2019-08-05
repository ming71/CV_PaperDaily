|  创建人   |  知乎论文阅读专栏 | 个人博客 | 其他相关链接 |
|  ----  | ----  | ----  | ----  |
| ming71  | [论文笔记入口](https://zhuanlan.zhihu.com/c_1113860303082704896) | [chaser](https://ming71.github.io/) |   [CSDN](https://blog.csdn.net/mingqi1996) 

<span id="inline-blue">论文发布日期：2018.5.18 [TIP]<p/span>

&emsp;&emsp;只读了不到半小时，没仔细看，理解也有限，但我实在没看出这篇能在2018年中旬发在TIP的文章有任何的insight和能拿得出的idea，但是<u>**老说别人论文灌水可能反而是自己水平不够**</u>，希望有看过这篇论文并且有启发的人能分享一下感想，并且指出我的不足和短见....  
<!-- more -->
email: mq_chaser@126.com  
qq: 1343545543 （加我请备注）   


## 1. Motivation  
&emsp;&emsp;为了相比SSD提高小目标检测效果，同时保持检测速度不至于像DSSD一样个位数。  

## 2. Implemention
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/MDSSD/str1.png" alt="" style="width:80%" /></center>
&emsp;&emsp;这篇论文一张图就够了。采用的VGG16作backbone，因为ResNet-152的降采样步长太大小目标丢失。相比SSD检测分支，作者仅仅引入了底层和高层的跨级连接，进行信息融合，拼接方式如下。上采样是转置卷积，Norm采用的L2，ele sum融合，然后加3x3抹平。仔细读了下这个融合模块，确实平平无奇，也没什么insight....
<center><img src="http://chaserblog.test.upcdn.net/blogs/paper/MDSSD/str2.png" alt="" style="width:50%" /></center>

## 3. Question  
&emsp;&emsp;对这个看上去很直接不讲道理也没什么创新的工作，有这么几个问题：
* 融合层选择的降采样步长跨越为6-8倍，这么大的跨度不怕语义分歧吗？
* 是不是转置卷积的效果比双线性插值好？（肯定不会好很多，不然都不用双线性插值了）类似沙漏网络的上采样然后和降采样信息融合的方法优势在哪里？和这种上采样融合多尺度特征有什么异同？


<br>
<br>
<hr />