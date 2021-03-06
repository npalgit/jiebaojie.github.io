---
layout: post
notes: true
subtitle: "【技术博客】爬虫原理及反爬虫技术"
comments: false
author: "私募工场"
date: 2016-11-26 00:00:00

---


原文：[http://www.weixinla.com/document/61937703.html](http://www.weixinla.com/document/61937703.html)

*   目录
{:toc }

# 1. 爬虫技术概述

网络爬虫(Web crawler)，是一种按照一定的规则，自动地抓取万维网信息的程序或者脚本，它们被广泛用于互联网搜索引擎或其他类似网站，可以自动采集所有其能够访问到的页面内容，以获取或更新这些网站的内容和检索方式。从功能上来讲，爬虫一般分为**数据采集，处理，储存**三个部分。

*   传统爬虫从一个或若干初始网页的URL开始，获得初始网页上的URL，在抓取网页的过程中，不断从当前页面上抽取新的URL放入队列，直到满足系统的一定停止条件。
*   聚焦爬虫的工作流程较为复杂，需要根据一定的网页分析算法过滤与主题无关的链接，保留有用的链接并将其放入等待抓取的URL队列。然后，它将根据一定的搜索策略从队列中选择下一步要抓取的网页URL，并重复上述过程，直到达到系统的某一条件时停止。

相对于通用网络爬虫，聚焦爬虫还需要解决三个主要问题：

1.  对抓取目标的描述或定义；
2.  对网页或数据的分析与过滤；
3.  对URL的搜索策略。

# 2. 爬虫原理

## 2.1 网络爬虫原理

Web网络爬虫系统正是通过网页中的超连接信息不断获得网络上的其它网页。

![](/img/notes/search/crawlerAndAntiCrawler/search_engine.jpg)

## 2.2 网络爬虫系统的工作原理

在网络爬虫的系统框架中，主过程由控制器，解析器，资源库三部分组成：

*   控制器：控制器是网络爬虫的中央控制器，它主要是负责根据系统传过来的URL链接，分配一线程，然后启动线程调用爬虫爬取网页的过程。
*   解析器：解析器是负责网络爬虫的主要部分，其负责的工作主要有：下载网页的功能，对网页的文本进行处理，如过滤功能，抽取特殊HTML标签的功能，分析数据功能。
*   资源库：主要是用来存储网页中下载下来的数据记录的容器，并提供生成索引的目标源。

![](/img/notes/search/crawlerAndAntiCrawler/crawler_component.jpg)

网络爬虫的基本工作流程：

1.  首先选取一部分精心挑选的种子URL；
2.  将这些URL放入待抓取URL队列；
3.  从待抓取URL队列中取出待抓取在URL，解析DNS，并且得到主机的ip，并将URL对应的网页下载下来，存储进已下载网页库中。此外，将这些URL放进已抓取URL队列；
4.  分析已抓取URL队列中的URL，分析其中的其他URL，并且将URL放入待抓取URL队列，从而进入下一个循环。

![](/img/notes/search/crawlerAndAntiCrawler/crawler_workflow.jpg)

![](/img/notes/search/crawlerAndAntiCrawler/crawler_process.jpg)

## 2.3 抓取策略

### 2.3.1 深度优先遍历策略

### 2.3.2 宽度优先遍历策略

### 2.3.3 反向链接数策略

反向链接数是指一个网页被其他网页链接指向的数量。反向链接数表示的是一个网页的内容受到其他人的推荐的程度。因此，很多时候搜索引擎的抓取系统会使用这个指标来评价网页的重要程度，从而决定不同网页的抓取先后顺序。

### 2.3.4 Partial PageRank策略

Partial PageRank算法借鉴了PageRank算法的思想：对于已经下载的网页，连同待抓取URL队列中的URL，形成网页集合，计算每个页面的PageRank值，计算完之后，将待抓取URL队列中的URL按照PageRank值的大小排列，并按照该顺序抓取页面。

### 2.3.5 OPIC策略

该算法实际上也是对页面进行一个重要性打分。在算法开始前，给所有页面一个相同的初始现金（cash）。当下载了某个页面P之后，将P的现金分摊给所有从P中分析出的链接，并且将P的现金清空。对于待抓取URL队列中的所有页面按照现金数进行排序。

### 2.3.6 大站优先策略

对于待抓取URL队列中的所有网页，根据所属的网站进行分类。对于待下载页面数多的网站，优先下载。这个策略也因此叫做大站优先策略。

# 3. 爬虫分类

1.  分布式爬虫：Nutch
2.  JAVA爬虫：Crawler4j、WebMagic、WebCollector
3.  非JAVA爬虫：scrapy（基于Python语言开发）

## 3.1 分布式爬虫

爬虫使用分布式，主要是解决两个问题：

1.  海量URL管理
2.  网速

现在比较流行的分布式爬虫，是Apache的Nutch。但是对于大多数用户来说，Nutch是这几类爬虫里，最不好的选择，理由如下：

1.  Nutch是为搜索引擎设计的爬虫，大多数用户是需要一个做精准数据爬取（精抽取）的爬虫。
2.  Nutch依赖hadoop运行，hadoop本身会消耗很多的时间。如果集群机器数量较少，爬取速度反而不如单机爬虫快。
3.  Nutch虽然有一套插件机制，而且作为亮点宣传。可以看到一些开源的Nutch插件，提供精抽取的功能。但是开发过Nutch插件的人都知道，Nutch的插件系统有多蹩脚。利用反射的机制来加载和调用插件，使得程序的编写和调试都变得异常困难，更别说在上面开发一套复杂的精抽取系统了。
4.  用Nutch进行爬虫的二次开发，爬虫的编写和调试所需的时间，往往是单机爬虫所需的十倍时间不止。
5.  很多人说Nutch2有gora，可以持久化数据到avro文件、hbase、mysql等。很多人其实理解错了，这里说的持久化数据，是指将URL信息（URL管理所需要的数据）存放到avro、hbase、mysql。并不是你要抽取的结构化数据。其实对大多数人来说，URL信息存在哪里无所谓。
6.  Nutch2的版本目前并不适合开发。官方现在稳定的Nutch版本是nutch2.2.1，但是这个版本绑定了gora-0.3。如果想用hbase配合nutch（大多数人用nutch2就是为了用hbase)，只能使用0.90版本左右的hbase，相应的就要将hadoop版本降到hadoop 0.2左右。

所以，如果你不是要做搜索引擎，尽量不要选择Nutch作为爬虫。

如果你是要做搜索引擎，Nutch1.x是一个非常好的选择。Nutch1.x和solr或者es配合，就可以构成一套非常强大的搜索引擎了。如果非要用Nutch2的话，建议等到Nutch2.3发布再看。目前的Nutch2是一个非常不稳定的版本。

![](/img/notes/search/crawlerAndAntiCrawler/crawler_architect.jpg)

## 3.2 JAVA爬虫

使用开源爬虫的意义在哪里？其实是要用开源爬虫的线程池和URL管理功能（比如断点爬取）。

用户关心的问题：

1）爬虫支持多线程么、爬虫能用代理么、爬虫会爬取重复数据么、爬虫能爬取JS生成的信息么？

不支持多线程、不支持代理、不能过滤重复URL的，那都不叫开源爬虫，那叫循环执行http请求。

能不能爬js生成的信息和爬虫本身没有太大关系。爬虫主要是负责遍历网站和下载页面。爬js生成的信息和网页信息抽取模块有关，往往需要通过模拟浏览器(htmlunit,selenium)来完成。这些模拟浏览器，往往需要耗费很多的时间来处理一个页面。所以一种策略就是，使用这些爬虫来遍历网站，遇到需要解析的页面，就将网页的相关信息提交给模拟浏览器，来完成JS生成信息的抽取。

2) 爬虫可以爬取ajax信息么？

网页上有一些异步加载的数据，爬取这些数据有两种方法：使用模拟浏览器（问题1中描述过了），或者分析ajax的http请求，自己生成ajax请求的url，获取返回的数据。

3) 爬虫怎么爬取要登陆的网站？

这些开源爬虫都支持在爬取时指定cookies，模拟登陆主要是靠cookies。至于cookies怎么获取，不是爬虫管的事情。你可以手动获取、用http请求模拟登陆或者用模拟浏览器自动登陆获取cookie。

4) 爬虫怎么抽取网页的信息？

开源爬虫一般都会集成网页抽取工具。主要支持两种规范：CSS SELECTOR和XPATH。

5) 爬虫怎么保存网页的信息？

有一些爬虫，自带一个模块负责持久化。比如webmagic，有一个模块叫pipeline。通过简单地配置，可以将爬虫抽取到的信息，持久化到文件、数据库等。还有一些爬虫，并没有直接给用户提供数据持久化的模块。比如crawler4j和webcollector。让用户自己在网页处理模块中添加提交数据库的操作。

6) 爬虫被网站封了怎么办？

爬虫被网站封了，一般用多代理（随机代理）就可以解决。但是这些开源爬虫一般没有直接支持随机代理的切换。所以用户往往都需要自己将获取的代理，放到一个全局数组中，自己写一个代理随机获取（从数组中）的代码。

7) 网页可以调用爬虫么？

爬虫的调用是在Web的服务端调用的，平时怎么用就怎么用，这些爬虫都可以使用。

8) 爬虫速度怎么样？

单机开源爬虫的速度，基本都可以将本机的网速用到极限。爬虫的速度慢，往往是因为用户把线程数开少了、网速慢，或者在数据持久化时，和数据库的交互速度慢。而这些东西，往往都是用户的机器和二次开发的代码决定的。这些开源爬虫的速度，都很可以。

9) 明明代码写对了，爬不到数据，是不是爬虫有问题，换个爬虫能解决么？

如果代码写对了，又爬不到数据，换其他爬虫也是一样爬不到。遇到这种情况，要么是网站把你封了，要么是你爬的数据是javascript生成的。爬不到数据通过换爬虫是不能解决的。

10) 哪个爬虫可以判断网站是否爬完、那个爬虫可以根据主题进行爬取？

爬虫无法判断网站是否爬完，只能尽可能覆盖。

至于根据主题爬取，爬虫之后把内容爬下来才知道是什么主题。所以一般都是整个爬下来，然后再去筛选内容。如果嫌爬的太泛，可以通过限制URL正则等方式，来缩小一下范围。

11) 哪个爬虫的设计模式和架构比较好？

对于JAVA开源爬虫，我觉得，随便找一个用的顺手的就可以。如果业务复杂，拿哪个爬虫来，都是要经过复杂的二次开发，才可以满足需求。

## 3.3 非JAVA爬虫

python可以用30行代码，完成JAVA 50行代码干的任务。python写代码的确快，但是在调试代码的阶段，python代码的调试往往会耗费远远多于编码阶段省下的时间。使用python开发，要保证程序的正确性和稳定性，就需要写更多的测试模块。当然如果爬取规模不大、爬取业务不复杂，使用scrapy这种爬虫也是蛮不错的，可以轻松完成爬取任务。

![](/img/notes/search/crawlerAndAntiCrawler/scrapy.jpg)

上图是Scrapy的架构图，绿线是数据流向，首先从初始URL 开始，Scheduler 会将其交给 Downloader 进行下载，下载之后会交给 Spider 进行分析，需要保存的数据则会被送到Item Pipeline，那是对数据进行后期处理。另外，在数据流动的通道里还可以安装各种中间件，进行必要的处理。 因此在开发爬虫的时候，最好也先规划好各种模块。我的做法是单独规划下载模块，爬行模块，调度模块，数据存储模块。

对于C++爬虫来说，学习成本会比较大。软件的调试也不是那么容易。

的确有一些非常小型的数据采集任务，用ruby或者php很方便。但是选择这些语言的开源爬虫，一方面要调研一下相关的生态圈，还有就是，这些开源爬虫可能会出一些你搜不到的BUG（用的人少、资料也少）

# 4. 反爬虫技术

## 4.1 通过Headers反爬虫

从用户请求的Headers反爬虫是最常见的反爬虫策略。很多网站都会对Headers的User-Agent进行检测，还有一部分网站会对Referer进行检测（一些资源网站的防盗链就是检测Referer）。如果遇到了这类反爬虫机制，可以直接在爬虫中添加Headers，将浏览器的User-Agent复制到爬虫的Headers中；或者将Referer值修改为目标网站域名。对于检测Headers的反爬虫，在爬虫中修改或者添加Headers就能很好的绕过。

## 4.2 基于用户行为反爬虫

还有一部分网站是通过检测用户行为，例如同一IP短时间内多次访问同一页面，或者同一账户短时间内多次进行相同操作。

大多数网站都是前一种情况，对于这种情况，使用IP代理就可以解决。可以专门写一个爬虫，爬取网上公开的代理ip，检测后全部保存起来。这样的代理ip爬虫经常会用到，最好自己准备一个。有了大量代理ip后可以每请求几次更换一个ip，这在requests或者urllib2中很容易做到，这样就能很容易的绕过第一种反爬虫。

对于第二种情况，可以在每次请求后随机间隔几秒再进行下一次请求。有些有逻辑漏洞的网站，可以通过请求几次，退出登录，重新登录，继续请求来绕过同一账号短时间内不能多次进行相同请求的限制。对于账户做防爬限制，一般难以应对，随机几秒请求也往往可能被封，如果能有多个账户，切换使用，效果更佳。

## 4.3 动态页面的反爬虫

上述的几种情况大多都是出现在静态页面，还有一部分网站，我们需要爬取的数据是通过ajax请求得到，或者通过Java生成的。首先用Firebug或者HttpFox对网络请求进行分析。如果能够找到ajax请求，也能分析出具体的参数和响应的具体含义，我们就能采用上面的方法，直接利用requests或者urllib2模拟ajax请求，对响应的json进行分析得到需要的数据。

能够直接模拟ajax请求获取数据固然是极好的，但是有些网站把ajax请求的所有参数全部加密了。遇到这样的网站，我们就不能用上面的方法了，我用的是selenium + phantomJS框架，调用浏览器内核，并利用phantomJS执行js来模拟人为操作以及触发页面中的js脚本。从填写表单到点击按钮再到滚动页面，全部都可以模拟，不考虑具体的请求和响应过程，只是完完整整的把人浏览页面获取数据的过程模拟一遍。

# 5. 参考资料

【1】网络爬虫基本原理：http://www.cnblogs.com/wawlian/archive/2012/06/18/2553061.html

【2】基于HttpClient4.0的网络爬虫基本框架：http://www.xuebuyuan.com/177356.html

【3】开源Python网络爬虫框架Scrapy：http://blog.sina.com.cn/s/blog_72995dcc0101kgty.html

【4】开源爬虫框架各有什么优缺点：http://www.36dsj.com/archives/36491

【5】社会化海量数据采集爬虫框架搭建：http://www.lanceyan.com/tech/arch/snscrawler.html

【6】网站常见的反爬虫和应对方法：http://www.cnblogs.com/data2value/p/5197060.html