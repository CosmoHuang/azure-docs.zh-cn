---
title: "轮转存储访问密钥后更新媒体服务 | Microsoft Docs"
description: "此文将提供有关在轮转存储访问密钥后如何更新媒体服务的指导。"
services: media-services
documentationcenter: 
author: Juliako
manager: erikre
editor: 
ms.assetid: a892ebb0-0ea0-4fc8-b715-60347cc5c95b
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/26/2016
ms.author: milanga;cenkdin;juliako
translationtype: Human Translation
ms.sourcegitcommit: e126076717eac275914cb438ffe14667aad6f7c8
ms.openlocfilehash: 8321f677d344109e35da3d8ba1109d8bece70db1


---
# <a name="update-media-services-after-rolling-storage-access-keys"></a>轮转存储访问密钥后更新媒体服务
当你创建新的 Azure 媒体服务帐户时，系统还会要求你选择用来存储媒体内容的 Azure 存储帐户。 请注意，可以[将多个存储帐户添加到](meda-services-managing-multiple-storage-accounts.md)媒体服务帐户。

在创建新的存储帐户后，Azure 将生成两个 512 位存储访问密钥，用于对你存储帐户的访问进行身份验证。 若要保持存储连接的安全性，我们建议定期重新生成并轮转你的存储访问密钥。 将提供两个访问密钥（主密钥和辅助密钥），以便在你重新生成其中一个访问密钥时，始终能够使用另一个访问密钥连接到存储帐户。 此过程也称为“轮转访问密钥”。

媒体服务依赖于为它提供的存储密钥。 具体而言，用于流式传输或下载资产的定位符依赖于指定的存储访问密钥。 创建 AMS 帐户时，媒体服务默认依赖于主存储访问密钥，但用户可以更新 AMS 的存储密钥。 必须遵照本主题中所述的步骤，确保媒体服务知晓要使用的密钥。 此外，在轮转存储访问密钥时，需要确保更新定位符，使流式传输服务不会中断（本主题中也描述了此步骤）。

> [!NOTE]
> 如果你有多个存储帐户，请对每个存储帐户执行此过程。
>
> 在对生产帐户执行本主题中所述的步骤之前，请确保对生产前帐户测试这些步骤。
>
>

## <a name="step-1-regenerate-secondary-storage-access-key"></a>步骤 1：重新生成辅助存储访问密钥
首先，请重新生成辅助存储密钥。 默认情况下，媒体服务不使用辅助密钥。  有关如何轮转存储密钥的信息，请参阅[如何：查看、复制和重新生成存储访问密钥](../storage/storage-create-storage-account.md#view-and-copy-storage-access-keys)。

## <a name="a-idstep2astep-2--update-media-services-to-use-the-new-secondary-storage-key"></a><a id="step2"></a>步骤 2：将媒体服务更新为使用新的辅助存储密钥
将媒体服务更新为使用辅助存储访问密钥。 可以使用以下两种方法之一，将重新生成的存储密钥同步到媒体服务。

* 使用 Azure 门户：若要查找名称和密钥值，请转到 Azure 门户并选择你的帐户。 “设置”窗口显示在右侧。 在“设置”窗口中，选择“密钥”。 根据要与媒体服务同步的存储密钥，选择同步主密钥按钮或同步辅助密钥按钮。 在本例中，我们将使用辅助密钥。
* 使用媒体服务管理 REST API。

以下代码示例演示了如何构造 https://endpoint/*subscriptionId*/services/mediaservices/Accounts/*accountName*/StorageAccounts/*storageAccountName*/Key 请求，以便将指定的存储密钥与媒体服务同步。 在本例中，我们将使用辅助存储密钥值。 有关详细信息，请参阅[如何：使用媒体服务管理 REST API](https://docs.microsoft.com/rest/api/media/management/how-to-use-media-services-management-rest-api)。

    public void UpdateMediaServicesWithStorageAccountKey(string mediaServicesAccount, string storageAccountName, string storageAccountKey)
    {
        var clientCert = GetCertificate(CertThumbprint);

        HttpWebRequest request = (HttpWebRequest)WebRequest.Create(string.Format("{0}/{1}/services/mediaservices/Accounts/{2}/StorageAccounts/{3}/Key",
        Endpoint, SubscriptionId, mediaServicesAccount, storageAccountName));
        request.Method = "PUT";
        request.ContentType = "application/json; charset=utf-8";
        request.Headers.Add("x-ms-version", "2011-10-01");
        request.Headers.Add("Accept-Encoding: gzip, deflate");
        request.ClientCertificates.Add(clientCert);


        using (var streamWriter = new StreamWriter(request.GetRequestStream()))
        {
            streamWriter.Write("\"");
            streamWriter.Write(storageAccountKey);
            streamWriter.Write("\"");
            streamWriter.Flush();
        }

        using (var response = (HttpWebResponse)request.GetResponse())
        {
            string jsonResponse;
            Stream receiveStream = response.GetResponseStream();
            Encoding encode = Encoding.GetEncoding("utf-8");
            if (receiveStream != null)
            {
                var readStream = new StreamReader(receiveStream, encode);
                jsonResponse = readStream.ReadToEnd();
            }
        }
    }

完成此步骤后，请按以下步骤中所示，更新现有（依赖于旧存储密钥）的定位符。

> [!NOTE]
> 在对媒体服务执行任何操作（例如，创建新定位符）之前等待 30 分钟，以防止对挂起的作业产生任何影响。
>
>

## <a name="step-3-update-locators"></a>步骤 3：更新定位符
> [!NOTE]
> 在轮转存储访问密钥时，你需要确保更新现有的定位符，使流式处理服务不会中断。
>
>

将新存储密钥与 AMS 同步后，至少等待 30 分钟。 然后，你可以重新创建 OnDemand 定位符，使其依赖于指定的存储密钥并保留现有的 URL。

请注意，当你更新（或重新创建）SAS 定位符时，URL 始终会变化。

> [!NOTE]
> 若要确保保留 OnDemand 定位器的现有 URL，你需要删除现有定位符并新建一个具相同 ID 的定位符。
>
>

以下 .NET 示例演示如何重新创建具相同 ID 的定位符。

    private static ILocator RecreateLocator(CloudMediaContext context, ILocator locator)
    {
    // Save properties of existing locator.
    var asset = locator.Asset;
    var accessPolicy = locator.AccessPolicy;
    var locatorId = locator.Id;
    var startDate = locator.StartTime;
    var locatorType = locator.Type;
    var locatorName = locator.Name;

    // Delete old locator.
    locator.Delete();

    if (locator.ExpirationDateTime <= DateTime.UtcNow)
        {
            throw new Exception(String.Format(
                "Cannot recreate locator Id={0} because its locator expiration time is in the past",
                locator.Id));
        }

        // Create new locator using saved properties.
        var newLocator = context.Locators.CreateLocator(
            locatorId,
            locatorType,
            asset,
            accessPolicy,
            startDate,
            locatorName);



        return newLocator;
    }


## <a name="step-5-regenerate--primary-storage-access-key"></a>步骤 5：重新生成主存储访问密钥
重新生成主存储访问密钥。 有关如何轮转存储密钥的信息，请参阅[如何：查看、复制和重新生成存储访问密钥](../storage/storage-create-storage-account.md#view-and-copy-storage-access-keys)。

## <a name="step-6-update-media-services-to-use-the-new-primary-storage-key"></a>步骤 6：将媒体服务更新为使用新的主存储密钥
按照[步骤 2](media-services-roll-storage-access-keys.md#step2) 中所述的相同过程操作，仅此次会将新的主存储访问密钥与媒体服务帐户同步。

> [!NOTE]
> 在对媒体服务执行任何操作（例如，创建新定位符）之前等待 30 分钟，以防止对挂起的作业产生任何影响。
>
>

## <a name="step-7-update-locators"></a>步骤 7：更新定位符
在 30 分钟后，你可以重新创建 OnDemand 定位符，使其依赖于新的主存储密钥并保留现有的 URL。

使用[步骤 3](media-services-roll-storage-access-keys.md#step-3-update-locators) 中所述的相同过程。

## <a name="media-services-learning-paths"></a>媒体服务学习路径
[!INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供反馈
[!INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]

### <a name="acknowledgments"></a>致谢
我们诚挚地向以下人员表达谢意，是他们协助完成了本文档的写作工作：Cenk Dingiloglu、Milan Gada 和 Seva Titov。



<!--HONumber=Jan17_HO2-->

