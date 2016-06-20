<properties
    pageTitle="创建包含主题和订阅的服务总线命名空间 | Azure"
    description="使用 ARM 模板创建包含主题和订阅的服务总线命名空间"
    services="service-bus"
    documentationCenter=".net"
    authors="sethmanheim"
    manager="timlt"
    editor=""/>

<tags
    ms.service="service-bus"
    ms.date="04/15/2016"
    wacn.date=""/>

# 使用 ARM 模板创建包含主题和订阅的服务总线命名空间

本文介绍如何使用创建包含主题和订阅的服务总线命名空间的 Azure Resource Manager (ARM) 模板。你将了解如何定义要部署的资源以及如何定义执行部署时指定的参数。可将此模板用于自己的部署，或自定义此模板以满足要求

有关创建模板的详细信息，请参阅[创作 Azure Resource Manager 模板][]。

有关完整的模板，请参阅[包含主题和订阅的服务总线命名空间][]模板。

>[AZURE.NOTE] 下列 ARM 模板将可供下载和部署。
>
>-    [创建包含队列和授权规则的服务总线命名空间](/documentation/articles/service-bus-arm-namespace-auth-rule)
>-    [创建包含事件中心和使用者组的服务总线命名空间](/documentation/articles/service-bus-arm-namespace-event-hub)
>-    [创建包含队列的服务总线命名空间](/documentation/articles/service-bus-arm-namespace-queue)
>-    [创建服务总线命名空间](/documentation/articles/service-bus-arm-namespace)
>
>若要检查最新模板，请参阅 [Azure 快速入门模板][]并搜索服务总线。

## 你将部署什么内容？

使用此模板，你将部署包含主题和订阅的服务总线命名空间。

主题和订阅以“发布/订阅”模式提供一对多的通信形式。

[了解有关服务总线主题和订阅的详细信息][]。

若要自动运行部署，请单击以下按钮：

[![部署到 Azure](./media/service-bus-arm-namespace-topic/deploybutton.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F201-servicebus-create-topic-and-subscription%2Fazuredeploy.json)

## Parameters

使用 Azure 资源管理器，可以定义在部署模板时想要指定的值的参数。该模板具有一个名为 `Parameters` 的部分，其中包含所有参数值。你应该为随着要部署的项目或要部署到的环境而变化的值定义参数。不要为永远保持不变的值定义参数。每个参数值可在模板中用来定义所部署的资源。

下面介绍模板中的每个参数。

### serviceBusNamespaceName

要创建的服务总线命名空间的名称。

```
"serviceBusNamespaceName": {
"type": "string"
}
```

### serviceBusTopicName

在服务总线命名空间中创建的主题的名称。

```
"serviceBusTopicName": {
"type": "string"
}
```

### serviceBusSubscriptionName

在服务总线命名空间中创建的订阅的名称。

```
"serviceBusSubscriptionName": {
"type": "string"
}
```

### serviceBusApiVersion

模板的服务总线 API 版本。

```
" serviceBusApiVersion": {
"type": "string"
}
```
## 要部署的资源

创建包含主题和订阅的标准服务总线命名空间。

```
"resources ": [{
        "apiVersion": "[variables('sbVersion')]",
        "name": "[parameters('serviceBusNamespaceName')]",
        "type": "Microsoft.ServiceBus/Namespaces",
        "location": "[variables('location')]",
        "kind": "Messaging",
        "sku": {
            "name": "StandardSku",
            "tier": "Standard"
        },
        "resources": [{
            "apiVersion": "[variables('sbVersion')]",
            "name": "[parameters('serviceBusTopicName')]",
            "type": "Topics",
            "dependsOn": [
                "[concat('Microsoft.ServiceBus/namespaces/', parameters('serviceBusNamespaceName'))]"
            ],
            "properties": {
                "path": "[parameters('serviceBusTopicName')]",
            },
            "resources": [{
                "apiVersion": "[variables('sbVersion')]",
                "name": "[parameters('serviceBusSubscriptionName')]",
                "type": "Subscriptions",
                "dependsOn": [
                    "[parameters('serviceBusTopicName')]"
                ],
                "properties": {}
            }]
        }]
    }]
```

## 运行部署的命令

[AZURE.INCLUDE [app-service-deploy-commands](../includes/app-service-deploy-commands.md)]

## PowerShell

```
New-AzureResourceGroupDeployment -Name <deployment-name> -ResourceGroupName <resource-group-name> -TemplateUri <https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-servicebus-create-topic-and-subscription/azuredeploy.json>
```

## Azure CLI

```
azure config mode arm

azure group deployment create <my-resource-group> <my-deployment-name> --template-uri <https://raw.githubusercontent.com/azure/azure-quickstart-templates/master/201-servicebus-create-topic-and-subscription/azuredeploy.json>
```

## 后续步骤

现在，你已使用 ARM 创建并部署了资源，请通过查看以下文章了解如何管理这些资源：

- [使用 Azure 自动化管理 Azure Service Bus](/documentation/articles/service-bus-automation-manage)
- [使用 PowerShell 管理服务总线](/documentation/articles/service-bus-powershell-how-to-provision)
- [使用服务总线资源管理器管理服务总线资源](https://code.msdn.microsoft.com/Service-Bus-Explorer-f2abca5a)


  [创作 Azure Resource Manager 模板]: /documentation/articles/resource-group-authoring-templates
  [Azure 快速入门模板]: https://azure.microsoft.com/documentation/templates/
  [了解有关服务总线主题和订阅的详细信息]: /documentation/articles/service-bus-queues-topics-subscriptions
  [Using Azure PowerShell with Azure Resource Manager]: /documentation/articles/powershell-azure-resource-manager
  [Using the Azure CLI for Mac, Linux, and Windows with Azure Resource Management]: /documentation/articles/xplat-cli-azure-resource-manager
  [包含主题和订阅的服务总线命名空间]: https://github.com/Azure/azure-quickstart-templates/blob/master/201-servicebus-create-topic-and-subscription/

<!---HONumber=Mooncake_0516_2016-->