---
title: Azure Stack 集成系统的 Azure Stack 公钥基础结构证书要求 | Microsoft Docs
description: 介绍 Azure Stack 集成系统的 Azure Stack PKI 证书部署要求。
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/10/2018
ms.author: jeffgilb
ms.reviewer: ppacent
ms.openlocfilehash: b1dcbfc51e63a5bca9186b62c871b2623653bbab
ms.sourcegitcommit: d98d99567d0383bb8d7cbe2d767ec15ebf2daeb2
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 05/10/2018
---
# <a name="azure-stack-public-key-infrastructure-certificate-requirements"></a>Azure Stack 公钥基础结构证书要求

Azure Stack 有一个公共基础结构网络，该网络使用分配给少量 Azure Stack 服务，并可能分配给租户 VM 的外部可访问公共 IP 地址。 在部署 Azure Stack 期间，需要使用这些 Azure Stack 公共基础结构终结点的、具有适当 DNS 名称的 PKI 证书。 本文提供以下方面的信息：

- 部署 Azure Stack 时需要哪些证书
- 获取与这些规范匹配的证书的过程是什么
- 如何在部署期间准备、验证和使用这些证书

> [!NOTE]
> 在部署期间，必须将证书复制到与要部署的标识提供者（Azure AD 或 AD FS）匹配的部署文件夹中。 如果将单个证书用于所有终结点，必须将该证书文件复制到下表所述的每个部署文件夹。 该文件夹的结构已预先在部署虚拟机中构建，路径为：C:\CloudDeployment\Setup\Certificates。 

## <a name="certificate-requirements"></a>证书要求
以下列表描述了部署 Azure Stack 时需要满足的证书要求： 
- 证书必须由内部证书颁发机构或公共证书颁发机构颁发。 如果使用公共证书颁发机构，它必须作为 Microsoft 信任根颁发机构计划的一部分包含在基础操作系统映像中。 在以下网页中查看完整列表：https://gallery.technet.microsoft.com/Trusted-Root-Certificate-123665ca 
- 你 Azure 堆栈的基础结构必须具有网络访问权限的证书中发布的证书颁发机构的证书吊销列表 (CRL) 位置。 此 CRL 必须是一个 http 终结点
- 轮换证书时，证书必须由签署部署时提供的证书的同一内部证书颁发机构颁发，或者由上述任何公共证书颁发机构颁发
- 不支持使用自签名证书
- 证书可以是单个通配符证书，其中涵盖使用者可选名称 (SAN) 字段中的所有命名空间。 或者，可以针对需要证书的终结点（例如 **acs**和 Key Vault）使用采用通配符的单个证书。 
- 证书签名算法不能是 SHA1，因为算法必须更可靠。 
- 证书格式必须是 PFX，因为安装 Azure Stack 时需要公钥和私钥。 
- 证书 pfx 文件的“密钥用途”字段中必须包含“数字签名”和“KeyEncipherment”值。
- 证书 pfx 文件的“增强型密钥使用”字段中必须包含“服务器身份验证(1.3.6.1.5.5.7.3.1)”和“客户端身份验证(1.3.6.1.5.5.7.3.2)”值。
- 证书的“颁发给:”字段不能与其“颁发者:”字段相同。
- 部署时，所有证书 pfx 文件的密码都必须相同
- 访问的证书 pfx 的密码必须是一个复杂的密码。
- 确保所有证书的“使用者名称”和“使用者可选名称”匹配本文中所述的规范，以免部署失败。

> [!NOTE]
> 不支持自签名证书。

> [!NOTE]
> 支持在证书的信任链 IS 中包含中间证书颁发机构。 

## <a name="mandatory-certificates"></a>必需的证书
本部分中的表格描述部署 Azure AD 和 AD FS Azure Stack 时所需的 Azure Stack 公共终结点 PKI 证书。 证书要求已根据区域以及所用命名空间和每个命名空间所需的证书分组。 此表格还描述了解决方案提供程序将每个公共终结点的不同证书所复制到的文件夹。 

需要使用每个 Azure Stack 公共基础结构终结点的、具有适当 DNS 名称的证书。 每个终结点的 DNS 名称使用以下格式表示：*&lt;prefix>.&lt;region>.&lt;fqdn>*。 

对于部署，[region] 和 [externalfqdn] 值必须与针对 Azure Stack 系统选择的区域和外部域名相匹配。 例如，如果区域名称为 *Redmond*，外部域名为 *contoso.com*，则 DNS 名称的格式为 *&lt;prefix>.redmond.contoso.com*。*&lt;prefix>* 值由 Microsoft 预先指定，描述证书保护的终结点。 此外，外部基础结构终结点的 *&lt;prefix>* 值取决于使用特定终结点的 Azure Stack 服务。 

> [!note]  
> 可以提供证书作为已复制到所有目录的单个通配符证书，其中涵盖“使用者”和“使用者可选名称”(SAN) 字段中的所有命名空间；也可以作为每个终结点的、已复制到相应目录的单个证书。 请记住，这两个选项都要求对 **acs** 和 Key Vault 等需要通配符证书的终结点使用此类证书。 

| 部署文件夹 | 所需的证书使用者和使用者可选名称 (SAN) | 范围（按区域） | 子域命名空间 |
|-------------------------------|------------------------------------------------------------------|----------------------------------|-----------------------------|
| 公共门户 | portal.&lt;region>.&lt;fqdn> | 门户 | &lt;region>.&lt;fqdn> |
| 管理门户 | adminportal.&lt;region>.&lt;fqdn> | 门户 | &lt;region>.&lt;fqdn> |
| Azure 资源管理器公共门户 | management.&lt;region>.&lt;fqdn> | Azure 资源管理器 | &lt;region>.&lt;fqdn> |
| Azure 资源管理器管理门户 | adminmanagement.&lt;region>.&lt;fqdn> | Azure 资源管理器 | &lt;region>.&lt;fqdn> |
| ACSBlob | *.blob.&lt;region>.&lt;fqdn><br>（通配符 SSL 证书） | Blob 存储 | blob.&lt;region>.&lt;fqdn> |
| ACSTable | *.table.&lt;region>.&lt;fqdn><br>（通配符 SSL 证书） | 表存储 | table.&lt;region>.&lt;fqdn> |
| ACSQueue | *.queue.&lt;region>.&lt;fqdn><br>（通配符 SSL 证书） | 队列存储 | queue.&lt;region>.&lt;fqdn> |
| KeyVault | *.vault.&lt;region>.&lt;fqdn><br>（通配符 SSL 证书） | Key Vault | vault.&lt;region>.&lt;fqdn> |
| KeyVaultInternal | *.adminvault.&lt;region>.&lt;fqdn><br>（通配符 SSL 证书） |  内部 Key Vault |  adminvault.&lt;region>.&lt;fqdn> |

### <a name="for-azure-stack-environment-on-pre-1803-versions"></a>适用于 1803 以前版本上的 Azure Stack 环境

|部署文件夹|所需的证书使用者和使用者可选名称 (SAN)|范围（按区域）|子域命名空间|
|-----|-----|-----|-----|
|公共门户|portal.*&lt;region>.&lt;fqdn>*|门户|*&lt;region>.&lt;fqdn>*|
|管理门户|adminportal.*&lt;region>.&lt;fqdn>*|门户|*&lt;region>.&lt;fqdn>*|
|Azure 资源管理器公共门户|management.*&lt;region>.&lt;fqdn>*|Azure 资源管理器|*&lt;region>.&lt;fqdn>*|
|Azure 资源管理器管理门户|adminmanagement.*&lt;region>.&lt;fqdn>*|Azure 资源管理器|*&lt;region>.&lt;fqdn>*|
|ACS<sup>1</sup>|一个包含使用者可选名称的多子域通配符证书：<br>&#42;.blob.*&lt;region>.&lt;fqdn>*<br>&#42;.queue.*&lt;region>.&lt;fqdn>*<br>&#42;.table.*&lt;region>.&lt;fqdn>*|存储|blob.*&lt;region>.&lt;fqdn>*<br>table.*&lt;region>.&lt;fqdn>*<br>queue.*&lt;region>.&lt;fqdn>*|
|KeyVault|&#42;.vault.*&lt;region>.&lt;fqdn>*<br>（通配符 SSL 证书）|Key Vault|vault.*&lt;region>.&lt;fqdn>*|
|KeyVaultInternal|&#42;.adminvault.*&lt;region>.&lt;fqdn>*<br>（通配符 SSL 证书）|内部 Key Vault|adminvault.*&lt;region>.&lt;fqdn>*|
|
<sup>1</sup> ACS 证书要求单个证书中包含三个通配符 SAN。 并非所有公共证书颁发机构都支持在单个证书中包含多个通配符 SAN。 

如果使用 Azure AD 部署模式来部署 Azure Stack，只需请求上表中所列的证书。 但是，如果使用 AD FS 部署模式来部署 Azure Stack，则还必须请求下表中所述的证书：

|部署文件夹|所需的证书使用者和使用者可选名称 (SAN)|范围（按区域）|子域命名空间|
|-----|-----|-----|-----|
|ADFS|adfs.*&lt;region>.&lt;fqdn>*<br>（SSL 证书）|ADFS|*&lt;region>.&lt;fqdn>*|
|图形|graph.*&lt;region>.&lt;fqdn>*<br>（SSL 证书）|图形|*&lt;region>.&lt;fqdn>*|
|

> [!IMPORTANT]
> 本部分所列的所有证书必须使用相同的密码。 

## <a name="optional-paas-certificates"></a>可选的 PaaS 证书
如果打算在部署和配置 Azure Stack 之后部署其他 Azure Stack PaaS 服务（SQL、MySQL 和 应用服务），则需要请求额外的证书来涵盖 PaaS 服务的终结点。 

> [!IMPORTANT]
> 用于应用服务、SQL 和 MySQL 资源提供程序的证书需要包含与用于全局 Azure Stack 终结点的证书相同的根颁发机构。 

下表描述了 SQL 和 MySQL 适配器以及应用服务所需的终结点与证书。 无需将这些证书复制到 Azure Stack 部署文件夹。 应在安装其他资源提供程序时提供这些证书。 

|范围（按区域）|证书|所需的证书使用者和使用者可选名称 (SAN)|子域命名空间|
|-----|-----|-----|-----|
|SQL、MySQL|SQL 和 MySQL|&#42;.dbadapter.*&lt;region>.&lt;fqdn>*<br>（通配符 SSL 证书）|dbadapter.*&lt;region>.&lt;fqdn>*|
|应用服务|Web 流量默认 SSL 证书|&#42;.appservice.*&lt;region>.&lt;fqdn>*<br>&#42;.scm.appservice.*&lt;region>.&lt;fqdn>*<br>&#42;.sso.appservice.*&lt;region>.&lt;fqdn>*<br>（多域通配符 SSL 证书<sup>1</sup>）|appservice.*&lt;region>.&lt;fqdn>*<br>scm.appservice.*&lt;region>.&lt;fqdn>*|
|应用服务|API|api.appservice.*&lt;region>.&lt;fqdn>*<br>（SSL 证书<sup>2</sup>）|appservice.*&lt;region>.&lt;fqdn>*<br>scm.appservice.*&lt;region>.&lt;fqdn>*|
|应用服务|FTP|ftp.appservice.*&lt;region>.&lt;fqdn>*<br>（SSL 证书<sup>2</sup>）|appservice.*&lt;region>.&lt;fqdn>*<br>scm.appservice.*&lt;region>.&lt;fqdn>*|
|应用服务|SSO|sso.appservice.*&lt;region>.&lt;fqdn>*<br>（SSL 证书<sup>2</sup>）|appservice.*&lt;region>.&lt;fqdn>*<br>scm.appservice.*&lt;region>.&lt;fqdn>*|

<sup>1</sup> 需要一个包含多个通配符使用者可选名称的证书。 并非所有公共证书颁发机构都支持在单个证书中包含多个通配符 SAN 

<sup>2</sup> 不能使用 &#42;.appservice.*&lt;region>.&lt;fqdn>* 通配符证书来取代这三个证书（api.appservice.*&lt;region>.&lt;fqdn>*、ftp.appservice.*&lt;region>.&lt;fqdn>* 和 sso.appservice.*&lt;region>.&lt;fqdn>*）。 应用服务明确要求对这些终结点使用不同的证书。 

## <a name="learn-more"></a>了解详细信息
了解如何[为 Azure Stack 部署生成 PKI 证书](azure-stack-get-pki-certs.md)。 

## <a name="next-steps"></a>后续步骤
[标识集成](azure-stack-integrate-identity.md)

