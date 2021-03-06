---
title: 将警报从 OMS 门户扩展（复制）到 Azure - 概述 | Microsoft Docs
description: 概述将警报从 OMS 门户复制到 Azure 警报的过程，围绕常见的客户顾虑进行了详述。
author: msvijayn
manager: kmadnani1
editor: ''
services: monitoring-and-diagnostics
documentationcenter: monitoring-and-diagnostics
ms.service: monitoring-and-diagnostics
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 05/14/2018
ms.author: vinagara
ms.openlocfilehash: 25dcbad8607a651a7dd4b79f4f418cc473a2bf0e
ms.sourcegitcommit: d78bcecd983ca2a7473fff23371c8cfed0d89627
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/14/2018
ms.locfileid: "34166484"
---
# <a name="extend-copy-alerts-from-oms-portal-into-azure"></a>将警报从 OMS 门户扩展（复制）到 Azure
Operations Management Suite (OMS) 门户仅显示 Log Analytics 警报。  新的警报体验现在已在 Microsoft Azure 中的各种服务和部件中集成了警报体验。 新体验在 Azure 门户中通过 Azure Monitor 下的“警报”提供，包含活动日志警报、指标警报以及有关 Log Analytics 和 Application Insights 的日志警报。 


但是，某些用户已在通过 [Microsoft Operation Management Suite (OMS) 门户](../operations-management-suite/operations-management-suite-overview.md)使用 Log Analytics 和类似警报的功能。 因此，为了使这些用户能够轻松管理他们的其他 Azure 资源并同时使用 Log Analytics，Microsoft 已在按部就班地确保 OMS 门户的功能也可以在 Azure 门户中使用。 在这方面，Azure 警报已允许用户为 Log Analytics 管理基于查询的警报，有关详细信息，请参阅 [Azure 警报中的日志警报](monitor-alerts-unified-log.md)。 在 Azure Monitor 下的“警报”中，在 OMS 门户中创建的警报已列在相应的 Log Analytics 工作区下。 但是，编辑或更改在 OMS 门户中创建的这类警报时，用户需要离开 Azure 并使用 OMS 门户；然后，如果他们需要管理任何其他服务，则需要返回到 Azure。 为降低此困难，Microsoft 现在已允许用户将其警报从 OMS 门户扩展到 Azure。

## <a name="benefits-of-extending-your-alerts"></a>对警报进行扩展的好处
除了不必导航到 Azure 门户外所带来的好处之外，将警报从 OMS 门户扩展到 Azure 还有其他重大优点。

- 在 OMS 门户中，只能创建和查看 250 个警报，与之不同，在 Azure 警报中不存在此限制。
- 从 Azure 警报中，可以管理、枚举和查看所有警报类型，而不是像使用 OMS 门户时那样仅限于 Log Analytics 警报。
- 使用 [Azure Monitor 角色](monitoring-roles-permissions-security.md)将用户的访问权限控制到仅监视和警报
- Azure 警报利用[操作组](monitoring-action-groups.md)，这允许针对每个警报采取多个操作，包括 SMS、语音呼叫、自动化 Runbook、Webhook、ITSM 连接器，等等。 

## <a name="process-of-extending-your-alerts"></a>对警报进行扩展的流程
将警报从 OMS 门户扩展到 Azure 中的流程**不**涉及以任何方式更改警报定义、查询或配置。 唯一需要进行的更改是，在 Azure 中，诸如电子邮件通知、Webhook 调用、运行自动化 Runbook 或连接到 ITSM 工具之类的所有操作都是通过操作组执行的。 因此，如果合适的操作组与警报关联，则它们将扩展到 Azure 中。

因为扩展流程是无损的并且不会造成中断，所以，从 **2018 年 5 月 14 日**开始，Microsoft 会自动将在 OMS 门户中创建的警报扩展到 Azure 警报。 从这天开始，Microsoft 将开始计划将警报扩展到 Azure 中并逐步使 OMS 门户中存在的所有警报也可以在 Azure 门户中进行管理。 

> [!NOTE]
> 从 2018 年 5 月 14 日开始，Microsoft 将开始自动将警报扩展到 Azure 的过程。 并非所有工作区和警报都将在这一天扩展；相反，Microsoft 将开始在未来的几周内自动分批扩展警报。 因此，OMS 门户中的警报不会在 2018 年 5 月 14 日当天自动扩展到 Azure，在此期间用户仍可以[手动扩展其警报](monitoring-alerts-extend-tool.md)。

在计划将 Log Analytics 工作区中的警报扩展到 Azure 中时，它们将继续工作，并且**不会**以任何方式影响你的监视。 在计划时，警报可能暂时不可修改/编辑；但是，在此短暂的时间内，可以继续创建新的 Azure 警报。 在此短暂期限内，如果从 OMS 门户中编辑或创建了警报，则用户可以选择继续在 Azure Log Analytics 中还是在 Azure 警报中进行操作。

 ![在计划的期限内，用户对警报的操作会重定向到 Azure](./media/monitor-alerts-extend/ScheduledDirection.png)

> [!NOTE]
> 将警报从 OMS 门户扩展到 Azure 是免费的，并且，当在 [Azure Monitor 定价策略](https://azure.microsoft.com/pricing/details/monitor/)中声明的限制和条件范围内使用时，将 Azure 警报用于基于查询的 Log Analytics 警报也不会计费。  

在此日期之前，用户可以通过主动选择使其警报可在 Azure 中进行管理，来享受对警报进行扩展的好处。

### <a name="how-to-voluntarily-extending-your-alerts"></a>如何主动扩展警报
为了使 OMS 用户能够轻松进入 Azure 警报，Microsoft 已创建了相关工具来用于实现警报扩展。 Microsoft OMS 门户客户可以通过 OMS 门户中的向导或使用新的 API 以编程方式将其警报扩展到 Azure。 有关详细信息，请参阅[使用 OMS 门户和 API 将警报扩展到 Azure](monitoring-alerts-extend-tool.md)。


## <a name="usage-after-extending-your-alerts"></a>警报在扩展后的使用
如上所述，在 Microsoft Operation Management Suite 中创建的警报将扩展到 Azure 警报中，此后，可以从 Azure 中管理它们。 警报将继续在 OMS 门户 - 警报设置部分中列出。 如下图所示：

 ![OMS 门户在警报扩展到 Azure 中之后仍然列出警报](./media/monitor-alerts-extend/PostExtendList.png)

对于在 OMS 门户中对警报执行的任何操作（例如编辑或创建），都会以透明方式将用户定向到 Azure 警报。 

> [!NOTE]
> 由于对 OMS 中的警报执行任何添加或编辑操作时，用户将以透明方式转到 Azure，因此请确保将用户正确地映射到相应的[使用 Azure Monitor 和警报的权限](monitoring-roles-permissions-security.md)

与之前一样，警报创建仍然基于现有的 [Log Analytics API](../log-analytics/log-analytics-api-alerts.md) 和[资源模板](../monitoring/monitoring-solutions-resources-searches-alerts.md)，唯一的微小变化是，在警报扩展到 Azure 后，需要在计划中关联操作组。

## <a name="next-steps"></a>后续步骤

* 了解[启动将警报从 OMS 扩展到 Azure 的流程](monitoring-alerts-extend-tool.md)时使用的工具
* 详细了解新的 [Azure 警报体验](monitoring-overview-unified-alerts.md)。
* 了解 [Azure 警报中的日志警报](monitor-alerts-unified-log.md)。
