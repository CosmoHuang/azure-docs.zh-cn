---
title: 使用 Azure Analysis Services Web 设计器创建表格模型 | Microsoft Docs
description: 介绍如何在 Azure 门户中使用 Web 设计器创建 Azure Analysis Services 表格模型。
author: minewiskan
manager: kfile
ms.service: analysis-services
ms.topic: conceptual
ms.date: 04/12/2018
ms.author: owend
ms.reviewer: minewiskan
ms.openlocfilehash: 30a7f78e7bf13e6e6197e95b266dfd0d6b8f83c0
ms.sourcegitcommit: 9cdd83256b82e664bd36991d78f87ea1e56827cd
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 04/16/2018
---
# <a name="create-a-model-in-azure-portal"></a>在 Azure 门户中创建模型

Azure 门户的 Azure Analysis Services Web 设计器（预览）功能提供了一种轻松快捷的方式，可用于创建和编辑表格模型以及直接在浏览器中查询模型数据。 

请记住，Web 设计器是预览版。 虽然会不断推出新功能，但预览版中的功能会受到限制。 对于更高级的模型开发和测试，最好使用 Visual Studio (SSDT) 和 SQL Server Management Studio (SSMS)。

## <a name="before-you-begin"></a>开始之前

- 位于标准层或开发人员层的 Azure Analysis Services 服务器。 使用 Web 设计器创建的新模型是 DirectQuery，且仅支持上述这些层。
- 作为数据源的 Azure SQL 数据库、Azure SQL 数据仓库或 Power BI Desktop (.pbix) 文件。 从 Power BI Desktop 文件创建的新模型支持 Azure SQL 数据库、Azure SQL 数据仓库、Oracle 和 Teradata 数据源。
- 用于连接到 Azure SQL 数据库或 Azure SQL 数据仓库数据源的 SQL Server 帐户和密码。

## <a name="sign-in-to-the-azure-portal"></a>登录到 Azure 门户

登录到 [Azure 门户](https://portal.azure.com/)。

## <a name="to-create-a-new-tabular-model"></a>创建新的表格模型

1. 在服务器的“概述” > “Web 设计器”中，单击“打开”。

    ![在 Azure 门户中创建模型](./media/analysis-services-create-model-portal/aas-create-portal-overview-wd.png)

2. 在“Web 设计器” > “模型”中，单击“+ 添加”。

    ![在 Azure 门户中创建模型](./media/analysis-services-create-model-portal/aas-create-portal-models.png)

3. 在“新建模型”中，键入模型名称，然后选择数据源。

    ![Azure 门户中的“新建模型”对话框](./media/analysis-services-create-model-portal/aas-create-portal-new-model.png)

4. 在“连接”中，输入连接属性。 用户名和密码必须是 SQL Server 帐户。

     ![Azure 门户中的“连接”对话框](./media/analysis-services-create-model-portal/aas-create-portal-connect.png)

5. 在“表和视图”中，选择要包括在模型中的表，然后单击“创建”。 将自动使用密钥对创建表之间的关系。

     ![选择表和视图](./media/analysis-services-create-model-portal/aas-create-portal-tables.png)

新模型会显示在浏览器中。 从这里可以：   

- 通过将字段拖动到查询设计器并添加筛选器来查询模型数据。
- 在表中创建新的度量值。
- 通过使用 json 编辑器编辑模型元数据。
- 在 Visual Studio (SSDT)、Power BI Desktop 或 Excel 中打开模型。

![选择表和视图](./media/analysis-services-create-model-portal/aas-create-portal-query.png)

> [!NOTE]
> 在浏览器中编辑模型元数据或创建新的度量值时，会将这些更改保存到 Azure 中的模型。 如果还要在 SSDT、Power BI Desktop 或 Excel 中使用模型，可不同步模型。


## <a name="next-steps"></a>后续步骤 
[管理数据库角色和用户](analysis-services-database-users.md)  
[使用 Excel 进行连接](analysis-services-connect-excel.md)  


