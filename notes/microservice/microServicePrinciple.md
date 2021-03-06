---
layout: post
notes: true
subtitle: "【技术博客】华为内部如何实施微服务架构？基本就靠这5大原则"
comments: false
author: "李林锋"
date: 2016-08-23 12:00:00

---


原文：[http://www.yunweipai.com/archives/9172.html](http://www.yunweipai.com/archives/9172.html)

*   目录
{:toc }

MVC单体架构问题：

*   **研发成本高**：代码重复率高，需求变更困难，无法满足新业务快速上线和敏捷交付。
*   **测试、部署成本高**：业务运行在一个进程中，因此系统中任何程序的改变，都需要对整个系统重新测试并部署。
*   **可伸缩性差**：水平扩展只能基于整个系统进行扩展，无法针对某一个功能模块按需扩展。
*   **可靠性差**：某个应用BUG，例如死循环、OOM等，会导致整个进程宕机，影响其它合设的应用。
*   **代码维护成本高**：本地代码在不断的迭代和变更，最后形成了一个个垂直的功能孤岛，只有原来的开发者才理解接口调用关系和功能需求，新加入人员或者团队其它人员很难理解和维护这些代码。
*   **依赖关系无法有效管理**：服务间依赖关系变得错踪复杂，甚至分不清哪个应用要在哪个应用之前启动，架构师都不能完整的描述应用的架构关系。

以上问题的应对策略，就是服务化。

*   首先，需要对业务进行**拆分**。
    *   横向拆分：按照不同的业务域进行拆分，例如订单、商品、库存、号卡资源等。形成独立的业务领域微服务集群。
    *   纵向拆分：把一个业务功能里的不同模块或者组件进行拆分。例如把公共组件拆分成独立的原子服务，下沉到底层，形成相对独立的原子服务层。
*   其次，要做好微服务的**分层**：梳理和抽取核心应用、公共应用，作为独立的服务下沉到核心和公共能力层，逐渐形成稳定的服务中心，使前端应用能更快速的响应多变的市场需求。
*   随后是服务的**部署和调用**。

# 服务化架构演进历史

![](/img/notes/microservice/microServicePrinciple/architecture_history.png)

## MVC

MVC架构大部分人都用过，它主要用来解决前后端、界面、控制逻辑和业务逻辑分层问题。比较流行的技术堆栈就是Spring + Struts + iBatis（Hibernate）+ Tomcat（JBoss）。

## RPC

随着业务特别是互联网的发展，业务规模的扩大，模块化逐步成为一种趋势，此时解决模块之间远程调用的RPC框架应运而生。RPC需要解决模块之间跨进程通信的问题，不同的团队开发不同的模块，通过一个RPC框架实现远程调用，RPC框架帮业务把通信细节给屏蔽掉，但是RPC框架也有自身的缺点。

RPC本身不负责服务化，例如：服务的自动发现不管、服务的应用和发布不管、服务的运维和治理也不管。没有透明化、服务化的能力，对整个应用层的侵入还是比较深的。

## SOA

SOA服务化架构，企业级资产重用和异构系统间的集成对接，SOA架构的现状：在传统企业IT领域，主要是解决异构系统之间的互通和粗粒度的标准化（WebService）。互联网领域，提供一套高效支撑应用快速开发迭代的服务化架构。例如各个互联网公司自研或者开源的分布式服务框架。

## 微服务架构

微服务（MSA）是一种架构风格，旨在通过将功能分解到各个离散的服务中以实现对解决方案的解耦。它有如下几个特征：

*   小，且只干一件事情。
*   独立部署和生命周期管理。
*   异构性
*   轻量级通信，RPC或者Restful。

微服务拆分原则： 围绕业务功能进行垂直和水平拆分。大小粒度是难点，也是团队争论的焦点。

*   功能完整性、职责单一性。
*   粒度适中，团队可接受。
*   迭代演进，非一蹴而就。
*   API的版本兼容性优先考虑。

# 微服务架构的开发原则

*   接口先行，语言中立
*   **管理+技术**双管齐下：
    *   允许接口变更，但是对变更的频度要做严格管控。
    *   提供全在线的API文档服务（例如Swagger UI），将离线的API文档转成全在线、互动式的API文档服务。
    *   API变更的主动通知机制，要让所有消费该API的消费者能够及时感知到API的变更。
    *   契约驱动测试，用于对兼容性做回归测试。

# 微服务架构的测试原则

微服务的测试包括单元测试、接口测试、集成测试和行为测试等，其中最重要的就是契约测试。

# 微服务架构的部署原则

**独立部署和生命周期管理、基础设施自动化。**

微部署可以部署在Dorker容器、PaaS平台（VM）或者物理机上。使用Docker部署微服务会带来很多优先：

*   一致的环境，线上线下环境一致。
*   避免对特定云基础设施提供商的依赖。
*   降低运维团队负担。
*   高性能接近裸机性能。
*   多租户。
*   云的弹性和敏捷。
*   云的动态性和资源隔离。

# 微服务架构的治理原则

微服务治理原则：**线上治理、实时动态生效**。

常用治理策略：

*   流量控制：动态、静态流控制。
*   服务降级。
*   超时控制。
*   优先级调度。
*   流量迁移。
*   调用链跟踪和分析。
*   服务路由。
*   服务上线审批、下线通知。
*   SLA策略控制。

# 微服务最佳实践

服务路由：本地短路策略。关键技术点：优先调用本JVM内部服务提供者，其次是相同主机或者VM的，最后是跨网络调用。通过本地短路，可以避免远程调用的网络开销，降低服务调用时延、提升成功率。

服务调用方式：同步调用、异步调用、并行调用。

微服务故障隔离：线程级、进程级、容器级、VM级、物理机级等。关键技术点：

*   支持服务部署到不同线程/线程池中。
*   核心服务和非核心服务隔离部署。
*   为了防止线程膨胀，支持共享和独占两种线程池策略。

事务一致性问题：大部分业务可以通过最终一致性来解决，极少部分需要采用强一致性。

*   最终一致性，可以基于消息中间件实现。
*   强一致性，使用TCC框架。服务框架本身不会直接提供“分布式事务”，往往根据实际需要迁入分布式事务框架来支持分布式事务。

微服务的性能三要素：

*   I/O模型，这个通常会选用非堵塞的，Java里面可能用java原生的。
*   线程调度模型。
*   序列化方式。