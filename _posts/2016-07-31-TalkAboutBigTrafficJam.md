---
layout: post
title: 再谈大塞车游戏活动（基于neoremind大神博文的思考）
comments: true
author: "Bao Jie"
date: 2016-07-31 12:00:00
header-img: 
tags:
    - 算法
    - 逻辑思维
---

一直没想好第一篇博客写什么，正好闲逛neoremind大神博客时看到一个有意思的问题——[大塞车游戏活动的算法解](http://neoremind.com/2016/07/%E5%A4%A7%E5%A1%9E%E8%BD%A6%E6%B8%B8%E6%88%8F%E6%B4%BB%E5%8A%A8%E7%9A%84%E7%AE%97%E6%B3%95%E8%A7%A3/)，仔细思考后发现回味很深，我的处女作就借这个为题来聊一聊我的整个解题思路及证明方法，也是对neoremind大神这篇文章的一个补充吧。

# 1. 问题描述

这里再对问题进行简单的描述，为方便理解，这里对问题做了等价转化，使其更贴近学术性，原问题的描述可参考neoremind大神的原文。问题描述如下：

A、B两队各有n名队员，分列队伍两头，两队之间有一把空椅子。要求两队队员只能通过空椅子来交换位置（即队员与椅子做交换），只允许两种方式的交换，第一种是椅子旁边的队员与椅子的交换，第二种是一方队员隔着对方队员跳一步与椅子做交换，最终的目标是交换若干次后A、B两队交换队伍，即A队队员都换至原B队队员的位置，B队队员都换至原A队队员的位置，如以下A、B队各有2名队员，椅子用*字符描述，则：

    初始状态：AA*BB
    第一步后：A*ABB （右边的A与右边的椅子交换）
    第二步后：ABA*B （左边的B跳过左边的A与椅子交换）
    第三步后：ABAB* （右边的B与左边的椅子交换）
    第四步后：AB*BA （右边的A跳过右边的B与椅子交换）
    第五步后：*BABA （左边的A跳过左边的B与椅子交换）
    第六步后：B*ABA （左边的B与左边的椅子交换）
    第七步后：BBA*A （右边的B跳过左边的A与椅子交换）
    第八步后：BB*AA （左边的A与右边的椅子交换）

neoremind大神在原文中提供了一种解决方案，并附有程序的实现，有兴趣的读者可以先阅览下原文，本文接下去的章节尝试给出一种更直观通用的解决方案，并论证方案的可行性，最后再试图论证方案的最优性（步数最少）。

# 2. 解决方案

要解决这类问题，一般先手动模拟（针对n=1、n=2、n=3、n=4等情况分别进行手动模拟），来试图找到通用性的规律，然后再对发现的规律进行归纳和总结，最后来进行论证。

模拟的过程不再繁述，neoremind大神针对n<=5的情况也有gif图的动画演示，那么我们从模拟的过程中发现了什么规律呢？

从初始状态到最终状态分为了两个阶段：

*   第一阶段：交叉阶段。即从AAAAAA......A*BBBBBB......B经过若干步骤后，变成了*BABABA......BA或BABABA......BA*，即A队和B队的队员互相交叉。
*   第二阶段：完成阶段。即从*BABABA......BA或BABABA......BA*经过若干步骤后，变成了BBBBBB......B*AAAAAA......A，即完成最终任务。

## 2.1 第一阶段——交叉阶段

还是通过模拟后的发现，在交换的过程中总会出现如下两种状态：

    AAA......AA BABA......BA* BBB......BB
    AAA......AA *BABA......BA BBB......BB

上述两种状态中的空格只是为了把队伍切成3部分来看：

*   第1部分：若干个A队员
*   第2部分：若干组BABA队员互相交叉
*   第3部分：若干个B队员

初始状态也可以看成是特殊的以上两种状态之一（即第2部分BA对为0队），第一阶段的结束状态也可以看作是特殊的以上两种状态之一（第1部分和第3部分为空），那么接下去的工作只要说明如何通过若干步骤后使第2部分增加一个BA对即可。

这里针对第2部分为BABA......BA*来讲，*BABA......BA这种情况方法类似，请读者自行推导。

我们在BABA......BA*的前后各补上一个A来看，由以下步骤：

    初始状态：ABABA......BABA*B
    第一步后：ABABA......BABAB* （最右边的B与左边的椅子交换）
    第二步后：ABABA......BAB*BA （最右边的A跳过右边的B与椅子交换）
    第三步后：ABABA......B*BABA （第二右边的A跳过右边的B与椅子交换）
    ......
    若干步后：*BABA......BABABA （BA对比操作前多了一对）

由此，通过若干次以上的操作后，可由初始状态编程交叉状态。

## 2.2 第二阶段——完成阶段

仍然是通过模拟后的发现，在第二阶段的过程中总会出现如下两种状态：

    BBB......BB BABABA......BA* AAAAA......AA
    BBB......BB *BABABA......BA AAAAA......AA

变换过程和第一阶段类似，无非每一次的变换则第2部分少了一个BA对，请读者自行推导。

至此，两阶段的解决方案已描述完毕，并通过状态的推导论证了方案的可行性。

# 3. 方案的最优性证明

有了以上的解决方案后，我就产生了另一个问题，以上方案是最优方案么（步数最少），如果是最优，则如何证明呢？最优的步数是多少步呢？

以上几个问题困扰了我一个晚上，因为通过模拟，无法看出以上问题的答案，但最后换了种思路，问题迎刃而解。

还记得基于比较的排序最优的时间复杂度O(nlgn)是如何证明的么？它并没有证明那些O(nlgn)的排序方法时间复杂度是最优的，而是换了个角度，证明了基于比较的排序，最坏情况下元素之间的交换总次数不少于nlngn次，即证明了时间复杂度的下限是O(nlgn)。

回到我们讨论的问题，借鉴证明排序时间复杂度方法的思路，那么我们只要证明所有队员的步数总和的下限和上节方案中的总步数相同就可以了。

首先我们观察每个队员的初始位置和最终位置，发现它们之间的距离是n+1，即所有队员要走的总距离为2n(n+1)。当然这不是最终的总步骤，因为规则中有个跳跃的规则，每次跳跃相当于走的距离是2。那么总共跳跃了多少次呢？

通过思考不难得到，每个A队员都要和每个B队员来一次左右的互换（不是A翻越B就是B翻越A）。那么总共翻越的次数为n*n=n^2。剩下的就都是一步一步走的，为2n(n+1)-n^2=2n步。因此理论上总步数的下限是n^2+2n步。

下面看一下我们上节给出的方案。方案中每个A队员也和每个B队员进行过有且仅有一次的互换，即总共跳跃的总次数也为n^2次。剩下的步数通过方案也不难发现，方案中所有的A队员每次都是向右走，所有的B队员每次都向左走，不走回头路和冤枉路，因此最终一步一步走的总步数也一定是最佳的2n步。综上，我们的方案是最优方案。

# 4. 最优方案的唯一解（不考虑对称性）

这是之后加的一节。原文发出后，有热心的网友提出：如果不考虑对称性，应该只有唯一最优解。简单想想，该命题应该是正确的，但一直找不出证明的思路及方向。后又有热心的网友给我提供了一些证明思路，最终我找到了一个比较好理解的证明方法。

这里提供一个证明的思路和轮廓，具体的细节请读者自行脑补。

考虑第2节解决方案中的各个状态，发现解决方案的整体过程无非就是以下几种状态之间的转换，我们只要证明那些转换的唯一可行性就行了（其它都是死路）

   AAA......AA BABA......BA* BBB......BB
   AAA......AA *BABA......BA BBB......BB
   BBB......BB BABA......BA* AAA......AA
   BBB......BB *BABA......BA AAA......AA

# 5. 总结

本文基于neoremind大神一篇博文提出的塞车游戏活动问题进行补充，对问题进行了深入的讨论，提通过模拟、找规律、归纳、论证的思路来得出最终的解决方案，随后借鉴极限论证法来证明方案的最优性。希望本文解决问题及论证的思路能给读者在思考问题上带来一点点帮助及启发，来解决更多类似的问题。同时也感谢neoremind大神分享出这个很有意思的问题。

# 参考资料
1. [naoremind](http://neoremind.com/)：[大塞车游戏活动的算法解](http://neoremind.com/2016/07/%E5%A4%A7%E5%A1%9E%E8%BD%A6%E6%B8%B8%E6%88%8F%E6%B4%BB%E5%8A%A8%E7%9A%84%E7%AE%97%E6%B3%95%E8%A7%A3/)
