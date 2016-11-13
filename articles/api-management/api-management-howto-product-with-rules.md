---
title: 使用 Azure API 管理保护 API | Microsoft Docs
description: 了解如何使用配额和限制（速率限制）策略保护 API。
services: api-management
documentationcenter: ''
author: steved0x
manager: erikre
editor: ''

ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 08/24/2016
ms.author: sdanie

---
# 使用 Azure API 管理借助速率限制保护 API
本指南演示如何通过使用 Azure API 管理配置速率限制和配额策略轻松地为后端 API 添加保护。

在本教程中，将使用[限制每个订阅的调用速率](https://msdn.microsoft.com/library/azure/dn894078.aspx#LimitCallRate)和[设置每个订阅的使用配额](https://msdn.microsoft.com/library/azure/dn894078.aspx#SetUsageQuota)策略创建一个“免费试用版”API 产品，该产品允许开发人员对 API 每分钟最多进行 10 次调用且每周最多进行 200 次调用。然后，将发布 API，并测试速率限制策略。

有关使用 [rate-limit-by-key](https://msdn.microsoft.com/library/azure/dn894078.aspx#LimitCallRateByKey) 和 [quota-by-key](https://msdn.microsoft.com/library/azure/dn894078.aspx#SetUsageQuotaByKey) 策略的更高级限制方案，请参阅[使用 Azure API 管理设置高级请求限制](api-management-sample-flexible-throttling.md)。

## <a name="create-product"> </a>创建产品
在此步骤中，您将创建一种免费试用版产品，不需要订阅审批。

> [!NOTE]
> 如果已配置产品并想要在本教程中使用它，可以跳转到[配置调用速率限制和配额策略][配置调用速率限制和配额策略]，并按照那里的教程使用产品来代替免费试用版产品。
> 
> 

若要开始，请在 Azure 经典门户中单击 API 管理服务所对应的“管理”。这将转到 API 管理发布者门户。

![发布者门户][api-management-management-console]

> 如果尚未创建 API 管理服务实例，请参阅[在 Azure API 管理中管理第一个 API][在 Azure API 管理中管理第一个 API] 教程中的[创建 API 管理服务实例][创建 API 管理服务实例]。
> 
> 

单击左侧“API 管理”菜单中的“产品”，显示“产品”页。

![添加产品][api-management-add-product]

单击“添加产品”，显示“添加新产品”对话框。

![添加新产品][api-management-new-product-window]

在“标题”框中，键入“免费试用版”。

在“说明”框中，键入以下文本：“Subscribers will be able to run 10 calls/minute up to a maximum of 200 calls/week after which access is denied.”

可以保护或打开 API 管理中的产品。受保护的产品必须先订阅，然后才能使用。打开的产品无需订阅即可使用。请确保选中“需要订阅”，才能创建需要订阅的受保护产品。此设置为默认设置。

如果希望管理员审查并接受或拒绝此产品的订阅尝试，请选中“需要订阅审批”。如果未选中该复选框，则订阅尝试将自动批准。在此示例中，将自动批准订阅，因此未选中该框。

若要允许开发人员帐户多次订阅新产品，请选中“允许多个同时订阅”复选框。本教程不使用多个同时订阅，因此将它保留为未选中状态。

输入所有值后，单击“保存”创建产品。

![添加的产品][api-management-product-added]

默认情况下，新产品对“管理员”组中的用户可见。我们要添加“开发人员”组。单击“免费试用版”，然后单击“可见性”选项卡。

> 在 API 管理中，使用组来管理产品对开发人员的可见性。产品向组授予可见性，并且开发人员可以查看和订阅对他们所属的组可见的产品。有关详细信息，请参阅[如何创建和使用 Azure API 管理中的组][如何创建和使用 Azure API 管理中的组]。
> 
> 

![添加开发人员组][api-management-add-developers-group]

选中“开发人员”复选框，然后单击“保存”。

## <a name="add-api"> </a>将 API 添加到产品
在教程的此步骤中，我们将 Echo API 添加到新的免费试用版产品。

> 每个预先配置 Echo API 的 API 管理服务实例，都可用于试验和了解 API 管理。有关详细信息，请参阅[在 Azure API 管理中管理第一个 API][在 Azure API 管理中管理第一个 API]。
> 
> 

单击左侧“API 管理”菜单中的“产品”，然后单击“免费试用版”以配置该产品。

![配置产品][api-management-configure-product]

单击“将 API 添加到产品”。

![将 API 添加到产品][api-management-add-api]

选择“Echo API”，然后单击“保存”。

![添加 Echo API][api-management-add-echo-api]

## <a name="policies"> </a>配置调用速率限制和配额策略
在策略编辑器中配置速率限制和配额。单击左侧“API 管理”菜单下的“策略”。在“产品”列表中，单击“免费试用版”。

![产品策略][api-management-product-policy]

单击“添加策略”导入策略模板，并开始创建速率限制和配额策略。

![添加策略][api-management-add-policy]

要插入策略，请将光标放置到策略模板的 **inbound** 或 **outbound** 节中。速率限制和配额策略是入站的策略，因此将光标定位在入站元素中。

![策略编辑器][api-management-policy-editor-inbound]

我们将在本教程中添加的两个策略是[限制每个订阅的调用速率](https://msdn.microsoft.com/library/azure/dn894078.aspx#LimitCallRate)和[设置每个订阅的使用配额](https://msdn.microsoft.com/library/azure/dn894078.aspx#SetUsageQuota)策略。

![策略语句][api-management-limit-policies]

将光标放置在 **inbound** 策略元素中后，请单击“限制每个订阅的调用速率”旁边的箭头以插入其策略模板。

    <rate-limit calls="number" renewal-period="seconds">
    <api name="name" calls="number">
    <operation name="name" calls="number" />
    </api>
    </rate-limit>

**限制每个订阅的调用速率**可在产品级别使用，也可在 API 和单个操作名称级别使用。在本教程中，仅使用产品级别策略，因此请从 **rate-limit** 元素中删除 **api** 和 **operation** 元素，因此，只有外部的 **rate-limit** 元素仍然保留，如以下示例所示。

    <rate-limit calls="number" renewal-period="seconds">
    </rate-limit>

在“免费试用版”产品中，最大允许调用速率是每分钟调用 10 次，因此请键入 **10** 作为 **calls** 属性的值，键入 **60** 作为 **renewal-period** 属性的值。

    <rate-limit calls="10" renewal-period="60">
    </rate-limit>

若要配置“设置每个订阅的使用配额”策略，请直接在 **inbound** 元素中新添加的 **rate-limit** 元素下面定位光标，然后单击“设置每个订阅的使用配额”左侧的箭头。

    <quota calls="number" bandwidth="kilobytes" renewal-period="seconds">
    <api name="name" calls="number" bandwidth="kilobytes">
    <operation name="name" calls="number" bandwidth="kilobytes" />
    </api>
    </quota>

由于此策略还设计用在产品级别，因此请删除 **api** 和 **operation** 名称元素，如下例中所示。

    <quota calls="number" bandwidth="kilobytes" renewal-period="seconds">
    </quota>

配额可基于每个时间间隔、带宽或这两者的调用次数。在本教程中不基于带宽进行限制，因此请删除 **bandwidth** 属性。

    <quota calls="number" renewal-period="seconds">
    </quota>

在免费试用版产品中，该配额是每周调用 200 次。指定 **200** 作为 **calls** 属性的值，然后指定 **604800** 作为 **renewal-period** 属性的值。

    <quota calls="200" renewal-period="604800">
    </quota>

> 以秒为单位指定策略间隔。若要计算一周的时间间隔，可将天数 (7) 乘以一天的小时数 (24) 乘以每小时的分钟数 (60) 再乘以一分钟的秒数 (60)： 7 * 24 * 60 * 60 = 604800。
> 
> 

完成配置策略后，它应与下面的示例匹配。

    <policies>
        <inbound>
            <rate-limit calls="10" renewal-period="60">
            </rate-limit>
            <quota calls="200" renewal-period="604800">
            </quota>
            <base />

    </inbound>
    <outbound>

        <base />

        </outbound>
    </policies>

配置所需的策略后，请单击“保存”。

![保存策略][api-management-policy-save]

## <a name="publish-product"> </a> 发布产品
现在，已添加 API 且已配置策略，必须发布该产品才可供开发人员使用。单击左侧“API 管理”菜单中的“产品”，然后单击“免费试用版”以配置该产品。

![配置产品][api-management-configure-product]

单击“发布”，然后“单击是，发布”确认。

![发布产品][api-management-publish-product]

## <a name="subscribe-account"> </a>为开发人员帐户订阅产品
现在产品已发布，它可用于开发人员订阅和使用。

> API 管理实例的管理员自动订阅到每个产品。在此教程步骤中，将为非管理员开发人员帐户之一订阅“免费试用版”产品。如果开发人员帐户属于管理员角色，则可以遵循此步骤，即使已订阅也是如此。
> 
> 

单击左侧“API 管理”菜单上的“用户”，然后单击开发人员帐户的名称。在此示例中，将使用 **Clayton Gragg** 开发人员帐户。

![配置开发人员][api-management-configure-developer]

单击“添加订阅”。

![添加订阅][api-management-add-subscription-menu]

选择“免费试用版”，然后单击“订阅”。

![添加订阅][api-management-add-subscription]

> [!NOTE]
> 在本教程中，未为“免费试用版”产品启用“多个同时订阅”。如果是这样，会提示用户命名该订阅，如下面的示例所示。
> 
> 

![添加订阅][api-management-add-subscription-multiple]

单击“订阅”后，该产品将出现在用户的“订阅”列表中。

![添加的订阅][api-management-subscription-added]

## <a name="test-rate-limit"> </a>调用操作并测试速率限制
现在已配置和发布免费试用版产品，我们可以调用某些操作并测试速率限制策略。通过单击右上方菜单中的“开发人员门户”切换到开发人员门户。

![开发人员门户][api-management-developer-portal-menu]

单击顶部菜单中的 **API**，然后单击 **Echo API**。

![开发人员门户][api-management-developer-portal-api-menu]

单击“获取资源”，然后单击“试用”。

![打开控制台][api-management-open-console]

保留默认参数值，然后选择“免费试用版”产品的订阅密钥。

![订阅密钥][api-management-select-key]

> [!NOTE]
> 如果有多个订阅，请务必选择**免费试用版**的密钥，否则在前面步骤中配置的策略不会生效。
> 
> 

单击“发送”，然后查看响应。请注意响应状态为“200 正常”。

![操作结果][api-management-http-get-results]

以大于速率限制策略的每分钟 10 次调用的速率单击“发送”。超出速率限制策略后，将返回响应状态“429 太多请求”。

![操作结果][api-management-http-get-429]

“响应内容”指示重试次数成功前的剩余时间间隔。

每分钟 10 次调用的速率限制策略生效时，在超出速率限制前对产品进行前 10 次成功调用经过 60 秒之前，后续调用将失败。此示例中的剩余时间间隔为 54 秒。

## <a name="next-steps"></a>后续步骤
* 观看以下视频中设置速率限制和配额的演示。

> [!VIDEO https://channel9.msdn.com/Blogs/AzureApiMgmt/Rate-Limits-and-Quotas/player]
> 
> 

[api-management-management-console]: ./media/api-management-howto-product-with-rules/api-management-management-console.png
[api-management-add-product]: ./media/api-management-howto-product-with-rules/api-management-add-product.png
[api-management-new-product-window]: ./media/api-management-howto-product-with-rules/api-management-new-product-window.png
[api-management-product-added]: ./media/api-management-howto-product-with-rules/api-management-product-added.png
[api-management-add-policy]: ./media/api-management-howto-product-with-rules/api-management-add-policy.png
[api-management-policy-editor-inbound]: ./media/api-management-howto-product-with-rules/api-management-policy-editor-inbound.png
[api-management-limit-policies]: ./media/api-management-howto-product-with-rules/api-management-limit-policies.png
[api-management-policy-save]: ./media/api-management-howto-product-with-rules/api-management-policy-save.png
[api-management-configure-product]: ./media/api-management-howto-product-with-rules/api-management-configure-product.png
[api-management-add-api]: ./media/api-management-howto-product-with-rules/api-management-add-api.png
[api-management-add-echo-api]: ./media/api-management-howto-product-with-rules/api-management-add-echo-api.png
[api-management-developer-portal-menu]: ./media/api-management-howto-product-with-rules/api-management-developer-portal-menu.png
[api-management-publish-product]: ./media/api-management-howto-product-with-rules/api-management-publish-product.png
[api-management-configure-developer]: ./media/api-management-howto-product-with-rules/api-management-configure-developer.png
[api-management-add-subscription-menu]: ./media/api-management-howto-product-with-rules/api-management-add-subscription-menu.png
[api-management-add-subscription]: ./media/api-management-howto-product-with-rules/api-management-add-subscription.png
[api-management-developer-portal-api-menu]: ./media/api-management-howto-product-with-rules/api-management-developer-portal-api-menu.png
[api-management-open-console]: ./media/api-management-howto-product-with-rules/api-management-open-console.png
[api-management-http-get]: ./media/api-management-howto-product-with-rules/api-management-http-get.png
[api-management-http-get-results]: ./media/api-management-howto-product-with-rules/api-management-http-get-results.png
[api-management-http-get-429]: ./media/api-management-howto-product-with-rules/api-management-http-get-429.png
[api-management-product-policy]: ./media/api-management-howto-product-with-rules/api-management-product-policy.png
[api-management-add-developers-group]: ./media/api-management-howto-product-with-rules/api-management-add-developers-group.png
[api-management-select-key]: ./media/api-management-howto-product-with-rules/api-management-select-key.png
[api-management-subscription-added]: ./media/api-management-howto-product-with-rules/api-management-subscription-added.png
[api-management-add-subscription-multiple]: ./media/api-management-howto-product-with-rules/api-management-add-subscription-multiple.png

[How to add operations to an API]: api-management-howto-add-operations.md
[How to add and publish a product]: api-management-howto-add-products.md
[Monitoring and analytics]: ../api-management-monitoring.md
[Add APIs to a product]: api-management-howto-add-products.md#add-apis
[Publish a product]: api-management-howto-add-products.md#publish-product
[在 Azure API 管理中管理第一个 API]: api-management-get-started.md
[如何创建和使用 Azure API 管理中的组]: api-management-howto-create-groups.md
[View subscribers to a product]: api-management-howto-add-products.md#view-subscribers
[Get started with Azure API Management]: api-management-get-started.md
[创建 API 管理服务实例]: api-management-get-started.md#create-service-instance
[Next steps]: #next-steps

[Create a product]: #create-product
[配置调用速率限制和配额策略]: #policies
[Add an API to the product]: #add-api
[Publish the product]: #publish-product
[Subscribe a developer account to the product]: #subscribe-account
[Call an operation and test the rate limit]: #test-rate-limit

[Limit call rate]: https://msdn.microsoft.com/library/azure/dn894078.aspx#LimitCallRate
[Set usage quota]: https://msdn.microsoft.com/library/azure/dn894078.aspx#SetUsageQuota

<!---HONumber=AcomDC_0921_2016-->