---
layout: post
title: 关于返回零长度的集合还是null的一点思考
comments: true
author: "Bao Jie"
date: 2016-08-01 12:00:00
header-img: 
tags:
    - Java
    - 代码规范
---

《Effective Java》中第43条建议：返回零长度的数组，而不是null。这一条也是Java编码规范中最常见的法则之一。遵守这条规范的种种好处书中原文已描述的很清楚，此处不再细述。笔者对这条规范也非常忍痛，甚至还一度认为所有返回类型为数组或者集合的方法都没有理由返回null。但最近想到的一个Case使笔者重新对该条规范进行审视，认为在某些特殊场景下返回null不能被返回emptyList所取代。

# 1. Case：修改接口前端json数据的转化

考虑这样一种业务场景：提供一个可供广告主（用户）修改广告的接口。我们知道，广告有很多属性，而广告主可能只想修改其中的一个或多个属性，那么考虑接口可以足够方便使用，让用户只用提供要修改的属性即可，其余不提供的属性视为不修改。比如用户只想修改一个叫标签的属性，而业务上允许一个广告有0至多个标签，那么用户调用接口时只需提供"tag"字段即可。若数据的传输采用json格式，那么用户传输的数据可以是：

    "tag":["tag1", "tag2", "tag3"]

即将标签改为3个，分别为"tag1"、"tag2"、"tag3"。

一般我们后端的服务习惯采用java来编写，当然目前有现成的json转化工具供转化使用，但要若让我们自己来实现一个json值到java bean的转化，将"tag"属性的值转化为List<String>的对象，若遵守上述规则，则实现逻辑的第一步为：如果无"tag"参数，则返回emptyList。咋一看好像没啥问题，但是仔细想想就会看出歧义来了：系统拿到转化后的emptyList，是不修改标签呢，还是将原有的标签修改为0个标签？

# 2. 重新审视

或许上述的Case不够有说服力，但是我们可以想想一些json的转化工具，如jackson、fastjson等默认都没有将null转化为emptyList。因此笔者认为，在这种场景下，将null转化为emptyList是不明智的，虽然遵守了规范，但却破坏了应有的业务逻辑。

现在总结下该条规则适用及不适用的场景：

*   如果只是做数据转化，并不需要理解返回数据本身（比如jackson不需要了解数据本身含义），那么笔者不建议返回emptyList来替代返回null。
*   如果是对外提供的功能性的方法，此时一般开发者都会了解返回数据的含义，建议遵守规则。比如方法返回还剩余多少张火车票，对于这种场景没有任何理由返回null，必须遵守规则。
*   其他场景：在没有对功能产生破坏力的情况下建议遵守规则。

以上是笔者对『返回零长度的数组，而不是null』这条规则的一点点思考，或许举出的case不那么有说服力，或许得出的结论也可能是不合理的，那么就期待读者有更好的case、更好的想法、更好的观点。

# 参考
Joshua Bloch：《Effective Java》, Second Edition.
