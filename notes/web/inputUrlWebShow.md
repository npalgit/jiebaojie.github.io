---
layout: post
notes: true
subtitle: "【技术博客】老生常谈——从输入url到页面展示到底发生了什么"
comments: false
author: "咸鱼老弟"
date: 2017-09-01 00:00:00

---


原文：[http://www.cnblogs.com/xianyulaodi/p/6547807.html](http://www.cnblogs.com/xianyulaodi/p/6547807.html)

*   目录
{:toc }

**本文的目的是通过输入url之后发生的事情来做知识的总结和扩展。所以文章可能会很杂。**

# 1、输入地址

当我们开始在浏览器中输入网址的时候，浏览器其实就已经在智能的匹配可能得 url 了，他会从历史记录，书签等地方，找到已经输入的字符串可能对应的 url，然后给出智能提示，让你可以补全url地址。对于 google的chrome 的浏览器，他甚至会直接从缓存中把网页展示出来，就是说，你还没有按下 enter，页面就出来了。

# 2、浏览器查找域名的 IP 地址

1.	请求一旦发起，浏览器首先要做的事情就是解析这个域名，一般来说，浏览器会首先查看本地硬盘的 hosts 文件，看看其中有没有和这个域名对应的规则，如果有的话就直接使用 hosts 文件里面的 ip 地址。
2.	如果在本地的 hosts 文件没有能够找到对应的 ip 地址，浏览器会发出一个 DNS请求到本地DNS服务器 。本地DNS服务器一般都是你的网络接入服务器商提供，比如中国电信，中国移动。
3.	查询你输入的网址的DNS请求到达本地DNS服务器之后，本地DNS服务器会首先查询它的缓存记录，如果缓存中有此条记录，就可以直接返回结果，此过程是递归的方式进行查询。如果没有，本地DNS服务器还要向DNS根服务器进行查询。
4.	根DNS服务器没有记录具体的域名和IP地址的对应关系，而是告诉本地DNS服务器，你可以到域服务器上去继续查询，并给出域服务器的地址。这种过程是迭代的过程。
5.	本地DNS服务器继续向域服务器发出请求，在这个例子中，请求的对象是.com域服务器。.com域服务器收到请求之后，也不会直接返回域名和IP地址的对应关系，而是告诉本地DNS服务器，你的域名的解析服务器的地址。
6.	最后，本地DNS服务器向域名的解析服务器发出请求，这时就能收到一个域名和IP地址对应关系，本地DNS服务器不仅要把IP地址返回给用户电脑，还要把这个对应关系保存在缓存中，以备下次别的用户查询时，可以直接返回结果，加快网络访问。

下面这张图很完美的解释了这一过程：


![](/img/notes/web/inputUrlWebShow/dns.jpg)

## 知识扩展

### 1)什么是DNS？

DNS（Domain Name System，域名系统），因特网上作为域名和IP地址相互映射的一个分布式数据库，能够使用户更方便的访问互联网，而不用去记住能够被机器直接读取的IP数串。通过主机名，最终得到该主机名对应的IP地址的过程叫做域名解析（或主机名解析）。

通俗的讲，我们更习惯于记住一个网站的名字，比如www.baidu.com,而不是记住它的ip地址，比如：167.23.10.2。而计算机更擅长记住网站的ip地址，而不是像www.baidu.com等链接。因为，DNS就相当于一个电话本，比如你要找www.baidu.com这个域名，那我翻一翻我的电话本，我就知道，哦，它的电话（ip）是167.23.10.2。

### 2)DNS查询的两种方式：递归查询和迭代查询

#### 1、递归解析

当局部DNS服务器自己不能回答客户机的DNS查询时，它就需要向其他DNS服务器进行查询。此时有两种方式，如图所示的是递归方式。局部DNS服务器自己负责向其他DNS服务器进行查询，一般是先向该域名的根域服务器查询，再由根域名服务器一级级向下查询。最后得到的查询结果返回给局部DNS服务器，再由局部DNS服务器返回给客户端。

![](/img/notes/web/inputUrlWebShow/dns_recursive.png)

#### 2、迭代解析

当局部DNS服务器自己不能回答客户机的DNS查询时，也可以通过迭代查询的方式进行解析，如图所示。局部DNS服务器不是自己向其他DNS服务器进行查询，而是把能解析该域名的其他DNS服务器的IP地址返回给客户端DNS程序，客户端DNS程序再继续向这些DNS服务器进行查询，直到得到查询结果为止。也就是说，迭代解析只是帮你找到相关的服务器而已，而不会帮你去查。比如说：baidu.com的服务器ip地址在192.168.4.5这里，你自己去查吧，本人比较忙，只能帮你到这里了。

![](/img/notes/web/inputUrlWebShow/dns_iter.png)

### 3)DNS域名空间的组织方式

我们在前面有说到根DNS服务器，域DNS服务器，这些都是DNS域名称空间的组织方式。按其功能命名空间中用来描述 DNS 域名称的五个类别的介绍详见下表中，以及与每个名称类型的示例：

| 名称类型 | 说 明 | 示 例 |
| -------- | ----- | ----- |
| 根域 | DNS域名中使用时，规定由尾部句点(.)来指定名称位于根或更高级别的域层次结构 | 单个据点(.)或句点用于末尾的名称 |
| 顶级域 | 用来指示某个国家/地区或组织使用的名称的类型名称 | .com |
| 第二层域 | 个人组织在Internet上使用的注册名称 | qq.com |
| 子域 | 已注册的二级域名派生的域名，通俗的讲就是网站名 | www.qq.com |
| 主机名 | 通常情况下，DNS域名的最左侧的标签标识网络上的特定计算机，如h1 | h1.www.qq.com |