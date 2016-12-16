---
title: "服务总线高级和标准消息传送定价层概述 | Microsoft Docs"
description: "服务总线高级和标准消息传送"
services: service-bus-messaging
documentationcenter: .net
author: djrosanova
manager: timlt
editor: 
ms.assetid: e211774d-821c-4d79-8563-57472d746c58
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 09/02/2016
ms.author: darosa,sethm
translationtype: Human Translation
ms.sourcegitcommit: 9ace119de3676bcda45d524961ebea27ab093415
ms.openlocfilehash: f75dcb6fd74ca2db7944ac76363801f5ee96650d


---
# <a name="service-bus-premium-and-standard-messaging-tiers"></a>服务总线高级和标准消息传送层
服务总线消息传送（包括队列和主题等消息传送实体）将企业消息传送功能与具有云规模丰富的发布-订阅语义结合在一起。 对于许多完善的云解决方案，服务总线消息传送会用作通信基础结构。

服务总线消息传送的 *高级* 层将解决有关任务关键应用程序的规模、性能和可用性的常见客户请求。 虽然功能集几乎完全相同，但这两个层的服务总线消息传送旨在用于满足不同的使用情形。

下表突出显示了部分高层差异。

| 高级 | 标准 |
| --- | --- |
| 高吞吐量 |可变吞吐量 |
| 可预测性能 |可变滞后时间 |
| 可预测定价 |即用即付可变定价 |
| 增加和缩减工作负荷的能力 |不适用 |
| 消息大小 > 256 KB |消息大小为 256 KB |

**服务总线高级消息传送** 在 CPU 和内存层提供资源隔离，以便每个客户工作负荷以隔离方式运行。 此资源容器称为 *消息传送单元*。 每个高级命名空间至少会分配一个消息传送单元。 你可以为每个服务总线高级命名空间购买 1、2 或 4 个消息传送单元。 单一工作负荷或实体可以跨多个消息传送单元，尽管计费以 24 小时或每天的费率收取，但仍然可以随意更改消息传送单元数。 从而为基于服务总线的解决方案提供可预测和稳定的性能。

此性能不仅更易于预测和实现，而且速度更快。 服务总线高级消息传送以在 [Azure 事件中心](https://azure.microsoft.com/services/event-hubs/)引入的存储引擎为基础。 使用高级消息传送，峰值性能比使用标准层快得多。

## <a name="premium-messaging-technical-differences"></a>高级消息传送技术差异
以下是高级和标准消息传送层之间的部分差异。

### <a name="partitioned-queues-and-topics"></a>分区的队列和主题
分区的队列和主题在高级消息传送中受支持，但它们的作用方式不同于标准和基本层的服务总线消息传送方式。 高级消息传送不使用 SQL 作为数据存储，并且也不再具有与共享平台相关联的可能的资源竞争。 因此，不必进行分区。 此外，分区计数已从标准消息传送的 16 个减少到高级中的 2 个分区。 使用两个分区可确保可用性并且更适合高级运行时环境所要求的分区数。 有关分区的详细信息，请参阅 [分区的队列和主题](service-bus-partitioning.md)。

### <a name="express-entities"></a>快速实体
由于高级消息传送在一个完全隔离的运行时环境中运行，因此高级命名空间中不支持快速实体。 有关快速功能的详细信息，请参阅 [Microsoft.ServiceBus.Messaging.QueueDescription.EnableExpress](https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.queuedescription.enableexpress.aspx) 属性。

## <a name="next-steps"></a>后续步骤
若要了解有关服务总线消息传送的详细信息，请参阅以下主题。

* [Azure 服务总线高级消息传送简介（博客文章）](http://azure.microsoft.com/blog/introducing-azure-service-bus-premium-messaging/)
* [Azure 服务总线高级消息传送简介（Channel9）](https://channel9.msdn.com/Blogs/Subscribe/Introducing-Azure-Service-Bus-Premium-Messaging)
* [服务总线消息传送概述](service-bus-messaging-overview.md)
* [如何使用 Service Bus 队列](service-bus-dotnet-get-started-with-queues.md)




<!--HONumber=Nov16_HO3-->

