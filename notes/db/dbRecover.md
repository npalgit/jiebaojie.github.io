---
layout: post
notes: true
subtitle: "【技术博客】即使删了全库，保证半小时恢复"
comments: false
author: "58沈剑"
date: 2017-03-12 00:00:00

---


原文：[http://zhuanlan.51cto.com/art/201611/522786.htm](http://zhuanlan.51cto.com/art/201611/522786.htm)

*   目录
{:toc }

# 总结

保证数据的安全性是DBA第一要务，需要进行：

1.	全量备份+增量备份，并定期进行恢复演练，但该方案恢复时间较久，对系统可用性影响大
2.	1小时延时从，双份1小时延时从能极大加速数据库恢复时间
3.	个人建议1小时延时从足够，后台只读服务可以连1小时延时从，提高资源利用率