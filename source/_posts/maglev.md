---
title: 改进磁悬浮装置
date: 2018-05-08 23:10:05
tags:
- 项目
typora-root-url: ..
---

![加载失败,请刷新](/img/maglev01.jpg)

> 上一次做磁悬浮应该是大二时候的事情了，那时参加学校的一个电子设计竞赛，做的控制组的题目，就是下推式的磁悬浮装置（结果做的还不错拿了特等奖:D），做完之后我就把项目给开源了，原帖子发在极客工坊和Arduino中文社区。


![加载失败,请刷新](/img/maglev02.jpg)

> * 当时是用的Arduino来做的这个项目，用过Arduino的都知道，它是没有定时器中断的，所以在控制项目中，哪怕是很简单的PID算法，都是无法做到精确地固定周期运行。当然Arduino也有它的好处，就是利用各种方便的库函数进行快速的原型验证。整个装置其实并不复杂，不过是需要控制几个ADC读取霍尔传感器，输出几个PWM控制线圈，中间加点控制算法而已。经典如PID，对于这种低阶系统其实完全够用了，所以当时经过参数的优化之后，整体的运行效果也还可以接受。
> * 但是毕竟Arduino性能有限，运行一些复杂点的算法还是力不从心，而且最明显的缺点是，其PWM频率恰恰是490Hz（有几个口是980Hz，也有第三方库可以改），不偏不倚就是人耳听觉范围最灵敏的频段，也因此整个装置运行时会发出谜之噪音，容易引起强烈不适!😠 
> * **那这一次为什么又重新捡起这个项目呢，因为这是应一个朋友的委托帮忙做的产品方案，也由于是商业项目，所以就不方便开源啦~**

首先这次做的有两个方向，大家都知道磁悬浮分为上拉和下推两种，前者靠一个顶部线圈控制浮子悬浮，而后者则是靠四个线圈控制浮子。

![加载失败,请刷新](/img/maglev03.jpg)

> 其中下推式的由于我之前就做过也有教程，这里就不详细介绍了，重点说一下上拉式的一些特性，相比于那时候的程序只用了PID来控制线圈，这次由于经验丰富了，采用了更复杂的控制算法，最终的悬浮效果非常理想，主要来说，这次这个装置有以下几个特点：
>
> * 悬浮更稳定，几乎看不出浮子有任何抖动，这归功于改进的控制算法 
> * 几乎零噪声，上面说了，由于Arduino的pwm频率为490Hz，这是在人耳的听觉范围内的，所以可以听到之前的装置工作时会发出刺耳的声音，而这次将pwm频率提高到了20kHz，且控制算法进行了内插滤波，所以噪声几乎被消除了
> * 功耗得到了极大的控制，整个装置异常省电，是的你没听错，磁悬浮是很省电的，还是由于改良的控制算法，使用了多个闭环控制，其中包括一个电流环，也就是说算法会调节浮子到一个合适的位置，而这个位置的磁力（不包括线圈的力）是和重力完美平衡的，这样就让磁力抵消了几乎所有的重力，线圈只需要在这个平衡点附近，进行极其轻微的调整，因此电流是非常小的，实测待机电流30ma，加上浮子工作的电流50ma，也就是说线圈消耗的电流只有20ma左右，甚至跟单片机的功耗差不多
> * 浮子重量大大增加，这个装置最大可以悬浮多大的浮子呢？大到夸张，2kg左右都没问题，这是由于上面那一点，电流环的加持，不管浮子多重，其重力都会和磁力相平衡，只不过浮子越重，最终平衡的位置就离线圈的铁芯越近而已。更有意思的是，不管浮子多重，最终的功耗是不会变的，都是50ma左右，因为平衡点附近所需要的线圈磁力非常小
> * 这也是很有意思的一点，装置可以实现断电浮子自动上吸。在实际应用中，断电的时候，浮子如果掉下来可能会损坏挂在上面的物品，所以有必要实现断电的自动上吸。但是，照理说，断电的时候，单片机都不工作了，怎么还能控制浮子呢？如果不控制，由于浮子是稳定在平衡点附近，那么断电时候上吸还是下落完全是随机的啊，怎么保证百分百上吸呢？答案是，不让它工作在平衡点附近，也就是说，让它工作在平衡点上面一点点，这样的话，实际上线圈所提供的不是吸力，而是斥力来使得浮子平衡，于是一旦断电，斥力消失，浮子就自己上吸啦。这也是为什么，待机电流30ma，而工作电流有50ma的原因，其中20ma被用于线圈提供斥力了。实际上，如果不要求这个功能的话，线圈的功耗几乎可以降至0！
> * 也是最核心的一点，所有的参数都是自整定的，所以可以自适应浮子的重量

```
看一下具体的演示视频↓
```

<div style="height: 0;padding-bottom:65%;position: relative;">
<iframe width="760" height="510"  
        src="//player.bilibili.com/player.html?aid=23192890&cid=38606951&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="" style="position: absolute;height: 105%;width: 100%;"> </iframe>
</div>  


<br />

<!--more-->

> 这次的主控换成了性能强很多的STM32，F1系列非常常用的型号

![加载失败,请刷新](/img/maglev04.jpg)

![加载失败,请刷新](/img/maglev05.jpg)

> 继Nano的两轮自平衡、ONE的独轮自平衡、四轴的无轮自平衡之后，加上这次的磁悬浮“莫名其妙自平衡”，New Balance系列就算是齐活啦~

![加载失败,请刷新](/img/maglev06.jpg)