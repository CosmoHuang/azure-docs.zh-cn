---
title: Azure SQL 数据库 - 读取副本中的查询 | Microsoft Docs
description: Azure SQL 数据库允许使用只读副本的容量（称为“读取横向扩展”）对只读的工作负荷进行负载均衡。
services: sql-database
author: anosov1960
manager: craigg
ms.service: sql-database
ms.custom: monitor & tune
ms.topic: article
ms.date: 04/04/2018
ms.author: sashan
ms.openlocfilehash: 26204d5bd61d193a3d08e26f98faf77ecc367a94
ms.sourcegitcommit: 6fcd9e220b9cd4cb2d4365de0299bf48fbb18c17
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/05/2018
---
# <a name="use-read-only-replicas-to-load-balance-read-only-query-workloads-preview"></a>使用只读副本对只读的查询工作负荷进行负载均衡（预览版）

**读取横向扩展**允许使用只读副本的容量对 Azure SQL 数据库只读工作负荷进行负载均衡。 

## <a name="overview-of-read-scale-out"></a>读取横向扩展的概述

“高级”层（[基于 DTU 的购买模型](sql-database-service-tiers.md#dtu-based-purchasing-model)）或“业务关键”层（[基于 vCore 的购买模型](sql-database-service-tiers.md#vcore-based-purchasing-model-preview)）中的每个数据库中已自动预配多个 Always ON 副本，以支持可用性 SLA。 为这些副本预配的性能级别与常规数据库连接使用的读写副本相同。 **读取横向扩展**功能允许使用只读副本的容量而不是共享读写副本，对 SQL 数据库只读工作负荷进行负载均衡。 这样，只读工作负荷将与主要读写工作负荷相互隔离，并且不会影响后者的性能。此功能面向包含逻辑隔离只读工作负荷（例如分析）的应用程序，因此可以使用这些额外的容量来获得性能优势，且无需额外付费。

若要将读取横向扩展功能用于特定的数据库，必须在创建数据库时或者在之后通过更改其配置来显式启用此功能，可以采用以下方式执行此操作：使用 PowerShell 调用 [Set-AzureRmSqlDatabase](/powershell/module/azurerm.sql/set-azurermsqldatabase) 或 [New-AzureRmSqlDatabase](/powershell/module/azurerm.sql/new-azurermsqldatabase) 命令，或者通过 Azure 资源管理器 REST API 使用[数据库 - 创建或更新](/rest/api/sql/databases/createorupdate)方法。 

为某个数据库启用读取横向扩展后，会根据在应用程序的连接字符串中配置的 `ApplicationIntent` 属性将连接到该数据库的应用程序定向到该数据库的读写副本或只读副本。 有关 `ApplicationIntent` 属性的信息，请参阅[指定应用程序意向](https://docs.microsoft.com/sql/relational-databases/native-client/features/sql-server-native-client-support-for-high-availability-disaster-recovery#specifying-application-intent)。

> [!NOTE]
> 在预览期，查询数据存储和扩展事件不受只读副本的支持。

## <a name="data-consistency"></a>数据一致性

AlwasyON 的优势之一是，副本始终处于事务一致状态，但在不同的时间点，不同的副本之间可能存在较小的延迟。 读取横向扩展支持会话级一致性。 这意味着，如果只读会话在由于副本不可用而出现连接错误后重新连接，可以使用读写副本将其重定向到并非完全处于最新状态的副本。 同样，如果应用程序使用读写会话写入数据，并立即使用只读会话读取该数据，则最新的更新可能不会立即可见。 这是因为，对副本执行的事务日志重做是异步进行的。

> [!NOTE]
> 区域中的复制延迟较低，且这种情况很少见。


## <a name="connecting-to-a-read-only-replica"></a>连接到只读副本

为数据库启用读取横向扩展时，客户端提供的连接字符串中的 `ApplicationIntent` 选项会指示连接是要路由到写入副本还是只读副本。 具体而言，如果 `ApplicationIntent` 值为 `ReadWrite`（默认值），则连接将定向到数据库的读写副本。 这与现有行为相同。 如果 `ApplicationIntent` 值为 `ReadOnly`，则连接将路由到可读副本。

例如，以下连接字符串将客户端连接到只读副本（请将尖括号中的项替换为环境的正确值，并删除尖括号）：

```SQL
Server=tcp:<server>.database.windows.net;Database=<mydatabase>;ApplicationIntent=ReadOnly;User ID=<myLogin>;Password=<myPassword>;Trusted_Connection=False; Encrypt=True;
```

以下连接字符串之一将客户端连接到读写副本（请将尖括号中的项替换为环境的正确值，并删除尖括号）：

```SQL
Server=tcp:<server>.database.windows.net;Database=<mydatabase>;ApplicationIntent=ReadWrite;User ID=<myLogin>;Password=<myPassword>;Trusted_Connection=False; Encrypt=True;

Server=tcp:<server>.database.windows.net;Database=<mydatabase>;User ID=<myLogin>;Password=<myPassword>;Trusted_Connection=False; Encrypt=True;
```

## <a name="enable-and-disable-read-scale-out-using-azure-powershell"></a>使用 Azure PowerShell 启用和禁用读取横向扩展

在 Azure PowerShell 中管理读取横向扩展需要安装 Azure PowerShell 2016 年 12 月版或更高版本。 有关最新的 PowerShell 版本，请参阅 [Azure PowerShell](https://docs.microsoft.com/powershell/azure/install-azurerm-ps)。

在 Azure PowerShell 中调用 [Set-AzureRmSqlDatabase](/powershell/module/azurerm.sql/set-azurermsqldatabase) cmdlet，并为 `-ReadScale` 参数传入所需的值 – `Enabled` 或 `Disabled`，即可启用或禁用读取横向扩展。 或者，可以使用 [New-AzureRmSqlDatabase](/powershell/module/azurerm.sql/new-azurermsqldatabase) cmdlet 来创建已启用读取横向扩展的新数据库。

例如，若要为现有数据库启用读取横向扩展（请将尖括号中的项替换为环境的正确值，并删除尖括号）：

```powershell
Set-AzureRmSqlDatabase -ResourceGroupName <myresourcegroup> -ServerName <myserver> -DatabaseName <mydatabase> -ReadScale Enabled
```

若要为现有数据库禁用读取横向扩展（请将尖括号中的项替换为环境的正确值，并删除尖括号）：

```powershell
Set-AzureRmSqlDatabase -ResourceGroupName <myresourcegroup> -ServerName <myserver> -DatabaseName <mydatabase> -ReadScale Disabled
```

若要创建已启用读取横向扩展的新数据库（请将尖括号中的项替换为环境的正确值，并删除尖括号）：

```powershell
New-AzureRmSqlDatabase -ResourceGroupName <myresourcegroup> -ServerName <myserver> -DatabaseName <mydatabase> -ReadScale Enabled -Edition Premium
```

## <a name="enabling-and-disabling-read-scale-out-using-the-azure-sql-database-rest-api"></a>使用 Azure SQL 数据库 REST API 启用和禁用读取横向扩展

若要创建已启用读取横向扩展的数据库，或者要为现有数据库启用或禁用读取横向扩展，请按以下示例请求中所示，在 `readScale` 属性设置为 `Enabled` 或 `Disabled` 的情况下，创建或更新相应的数据库实体。

```rest
Method: PUT
URL: https://management.azure.com/subscriptions/{SubscriptionId}/resourceGroups/{GroupName}/providers/Microsoft.Sql/servers/{ServerName}/databases/{DatabaseName}?api-version= 2014-04-01-preview
Body:
{
   "properties":
   {
      "readScale":"Enabled"
   }
} 
```

有关更多信息，请参阅[数据库 - 创建或更新](/rest/api/sql/databases/createorupdate)。

## <a name="next-steps"></a>后续步骤

- 有关使用 PowerShell 设置读取横向扩展的信息，请参阅 [Set-AzureRmSqlDatabase](/powershell/module/azurerm.sql/set-azurermsqldatabase) 或 [New-AzureRmSqlDatabase](/powershell/module/azurerm.sql/new-azurermsqldatabase) cmdlet。
- 有关使用 REST API 设置读取横向扩展的信息，请参阅[数据库 - 创建或更新](/rest/api/sql/databases/createorupdate)。
