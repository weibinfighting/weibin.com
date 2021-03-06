---
title: 绘图软件——R
author: 瞿伟斌
date: '2016-05-20'
slug: 绘图软件-r
categories:
  - R
tags: []
---

&emsp;&emsp;累死累活的数据挖掘比赛终于结束了，虽然勉强在截止时间内交上了论文报告，但是感觉依然是有好多东西可以做的最后没有做到。如果要问本次比赛最大的感受的话，或许就是通过这次比赛，学会了一款强大的绘图软件——R :)）。

&emsp;&emsp;其实在开始之前也想过，既然是数据挖掘比赛嘛，无非就是多套几个高大上的模型，写几个非专人士看都看不懂的公式，然后P值、$R^2$、AIC之类的参数一摆，然后大功告成坐等拿奖……然后就是啪啪啪的打脸。在面对海量数据的时候第一步根本不是选择什么模型，而是先把数据洗白了再说。人生第一次面对五十万的数据，第一感觉就是打不开……无论是用Excel还是R，要直接打开一个五十万格式不明确，有一堆不明字符的文字数据还是太过勉强了。最终还是在连续删除了几行报错数据之后，利用R分批读取成功。然后就是所谓的套模型。先试一下主流的主成分，嗯不行，再试一下因子分析，典型相关分析，依然没什么结果。看着五十万的数据对各种模型油盐不进，维数死活降不下来，我的内心是极度淡定的，只是强烈要求先算别的题。然后就是不断的聚类分类，聚类分类……然后就没有然后了。后来仔细想想主要的原因在于数据清洗的还不够，有太多的无关数据，所以无论套什么模型，白噪声的量远远超过了含有信息的量，在这种情况下，不管套什么模型都分析不出什么所以然出来。只是在我领悟到这一点时距离比赛结束仅剩下不到五天时间……

&emsp;&emsp;在常规套路，模型、公式、图表装逼失败之后，为了能够写出论文，我就不得不换一种方法来完成比赛。而我手上还剩下的就只有所谓的数据可视化的这个利器了。作为一名有职业修养的R程序员，在无法用代码提升逼格的时候就要用各种闪瞎双眼的图表来征服对方。还好在“不能画好图的编程语言不是好的统计软件”的观点的影响之下R语言的绘图功能可以说是傲绝群雄，独孤求败了。仅用自带的绘图函数就可以展示各式各样的统计图形，更勿论基于图层的ggplot2、主攻三维图像的scatterplot3d、地图专用的map等R包。在这里就分别说几个用到的图形吧。

1. 散点图（线图）。要说R语言自带的绘图函数中，最为实用同时也是最常用的还是基础的plot()函数。利用最简单的线图和点图，最为直接的展示数据的基础数据，而在此基础之上，可以通过添加其他元素来丰富图表。于是一张最为简单的线图（或点图）就展示了数据间最为直接的关系。尽管有很多人觉得这样实在太简单了，但是对于很多的问题，这样简单的图其实就可以解决很多问题，大致的联系，分类或聚类情况等。有的时候简单并不代表不重要。
2. 核密度函数图。在后期写论文的时候最苦恼的就是缺少那种“单个拆开都认识，合在一起却不懂的‘学术段落’。”还好有核密度函数。尽管这种只是模拟变量分布的图看起来是如此的便于理解，但是其理论深度真的是写论文者的福音。而最让人拍手叫绝的是，如此构成原理如此复杂的图，在使用R绘图时却也只是简简单单的几行代码，这无疑减轻了大量的计算和模拟工作。尤其是在选择合适带宽的时候，一个for循环就可以轻松解决。不得不感叹，R语言果然不负“绘图软件”之名。
3. 玫瑰图。什么你说条形图？但是在这种逼格很重要的比赛中，自然要适时的显示自己不是简简单单的只知道条形图啊，当然是搬出拼图的先祖玫瑰图来最为自己略懂统计图史的证明。只是遗憾的是，像玫瑰图这种花哨的图，用R自带的绘图函数显然是难以完成任务。还好有强大的ggplot2包。这个基于图层进行作图的ggplot2包甚至是很多人选择在电脑上保留R语言这个软件的首要原因。利用ggplot2对各个元素的操作，你几乎可以得到你想要的所有图形。至于由条形图演变而来的玫瑰图就更简单不过了。只需画出先用geom_bar()函数画出条形图，在用coord_polar()函数旋转坐标轴即可。如此简单易懂的操作实在是让R作为绘图软件一骑绝尘，将其他竞争对手远远地甩在身后。
4. 地图。空间和时间是影响数据的重要因素，对于时间来说，有时间序列图可以完美展示，而对于空间来说，做好的方式，莫过于使用地图进行展示。以二维或三维的地图方式，我们就可以表现空间特点上的数据特征。而如果再结合动画等表现形式，就可以几乎完美的展现对象在空间时间上的特性。作为一个专治各种不服的“绘图软件”，R语言也可以轻松地画出各式各样的地图，借助于map包、sp包和maptool包以及网络上官方的地图资源就可以实现地图的完美表现。而但这一起都以图形的方式表达出来之后，有时往往会有许多出乎意料的结果。

&emsp;&emsp;在处理数据之前，我们应该将远不直观的数字格式的数据转化为图像数据。因为通过更为直观的图像方式，我们往往会发现许多之前被我们忽视的东西，而这些常常是我们一直在寻找或者需要深入解析的东西。同时利用图像来展示数据往往能被更多的人所理解。

&emsp;&emsp;只是遗憾的是我在离交论文还有不到3天的时候才画完了所有的图，因此最后的报告几乎沦落为看图说话了……
