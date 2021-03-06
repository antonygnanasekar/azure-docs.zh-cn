---
title: "编写使用 Azure 服务总线队列的应用程序 | Microsoft Docs"
description: "如何编写一个基于队列的、使用 Azure 服务总线的简单应用程序。"
services: service-bus-messaging
documentationcenter: na
author: sethmanheim
manager: timlt
editor: 
ms.assetid: 754d91b3-1426-405e-84b4-fd36d65b114a
ms.service: service-bus-messaging
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/07/2017
ms.author: sethm
ms.translationtype: Human Translation
ms.sourcegitcommit: d987aa22379ede44da1b791f034d713a49ad486a
ms.openlocfilehash: 40414edebcd76fc93136cbc14d7a6436fc7f6da5
ms.contentlocale: zh-cn
ms.lasthandoff: 02/16/2017

---
# <a name="create-applications-that-use-service-bus-queues"></a>创建使用服务总线队列的应用程序
本主题介绍了服务总线队列，并说明了如何编写一个基于队列的、使用服务总线的简单应用程序。

考虑零售业中的这样一种场景，其中来自各销售点 (POS) 终端的销售数据必须路由到库存管理系统，该系统使用此数据来确定何时需要补充库存。 此解决方案使用服务总线进行消息传送，以便在终端与库存管理系统之间进行通信，如下图所示：

![服务总线队列映像 1](./media/service-bus-create-queues/IC657161.gif)

每个 POS 终端通过将消息发送到 **DataCollectionQueue** 来报告其销售数据。 这些消息将保留在此队列中，直到库存管理系统检索到它们。 此模式通常称为*异步消息传送*，因为 POS 终端无需等待库存管理系统的回应即可继续进行处理。

## <a name="why-queuing"></a>为什么要使用队列？
在查看设置此应用程序所需的代码之前，请考虑在此方案中使用队列而不是让 POS 终端库存管理系统直接对话（同步）的优点。

### <a name="temporal-decoupling"></a>暂时分离
使用异步消息传送模式，创建方和使用方不需要在同一时间联机。 消息传送基础结构可靠地存储消息，直到使用方准备好接收它们。 这意味着分布式应用程序的组件可以断开连接，例如，为进行维护而自动断开，或因组件故障断开连接，而不会影响整个系统。 此外，使用方应用程序可能只需在一天的特定时段内联机。 例如，在此零售方案中，库存管理系统可能只需要在营业日结束后进入联机状态。

### <a name="load-leveling"></a>负载分级
在许多应用程序中，系统负载随时间而变化，而每个工作单元所需的处理时间通常为常量。 使用队列在消息创建方与使用方之间中继意味着，只需将使用方应用程序（辅助角色）预配为处理平均负载而非最大负载。 队列深度将随传入负载的变化而加大和减小。 这将直接根据为应用程序加载提供服务所需的基础结构的数目来节省成本。

![服务总线队列映像 2](./media/service-bus-create-queues/IC657162.gif)

### <a name="load-balancing"></a>负载平衡
随着负载增加，可添加更多的辅助角色以从辅助角色队列中读取。 每条消息仅由一个辅助进程处理。 另外，可通过此基于拉取的负载平衡来以最合理的方式使用辅助计算机，即使这些辅助计算机具有不同的处理能力（因为它们将以其最大速率拉取消息）也是如此。 此模式通常称为“使用者竞争模式”。

![服务总线队列映像 3](./media/service-bus-create-queues/IC657163.gif)

### <a name="loose-coupling"></a>松散耦合
使用消息队列在消息创建方与使用方之间中继可在个组件之间提供固有的松散耦合。 由于创建方和使用方互不相识，因此，可升级使用方，不会对创建方产生任何影响。 此外，消息传送拓扑结构还可以演化而不影响现有终结点。 我们将在讲解发布/订阅时介绍此内容的详细信息。

## <a name="show-me-the-code"></a>显示相关代码
以下部分介绍了使用服务总线生成此应用程序的方法。

### <a name="sign-up-for-an-azure-account"></a>注册 Azure 帐户
需要 Azure 帐户才能开始使用服务总线。 如尚无帐户，可在[此处](https://azure.microsoft.com/pricing/free-trial/?WT.mc_id=A85619ABF)注册免费帐户。

### <a name="create-a-namespace"></a>创建命名空间
拥有订阅后，即可[创建服务命名空间](service-bus-create-namespace-portal.md)。 每个命名空间都可充当一组服务总线实体的作用域容器。 请为新命名空间指定在所有服务总线帐户中唯一的名称。 

### <a name="install-the-nuget-package"></a>安装 NuGet 包
若要使用服务总线命名空间，应用程序必须引用服务总线程序集，特别是 Microsoft.ServiceBus.dll。 可在 Microsoft Azure SDK 的一部分中找到此程序集，也可在 [Azure SDK ](https://azure.microsoft.com/downloads/)下载页下载。 但是，[服务总线 NuGet 包](https://www.nuget.org/packages/WindowsAzure.ServiceBus)是获取服务总线 API 并为应用程序配置所有服务总线依赖项的最简单方法。

### <a name="create-the-queue"></a>创建队列
服务总线消息传送实体（队列和发布/订阅主题）的管理操作通过 [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) 类执行。 服务总线使用基于安全模型的[共享访问签名 (SAS)](service-bus-sas.md)。 [TokenProvider](/dotnet/api/microsoft.servicebus.tokenprovider) 类代表具有内置工厂方法的安全令牌提供程序，这些方法可返回一些众所周知的令牌提供程序。 我们将使用 [CreateSharedAccessSignatureTokenProvider](/dotnet/api/microsoft.servicebus.tokenprovider#Microsoft_ServiceBus_TokenProvider_CreateSharedAccessSignatureTokenProvider_System_String_) 方法来保留 SAS 凭据。 然后使用服务总线命名空间和令牌提供程序的基址构建 [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) 实例。

[NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) 类提供了创建、枚举和删除消息传送实体的方法。 此处显示的代码介绍了创建 [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) 实例并用其创建 **DataCollectionQueue** 队列的方法。

```csharp
Uri uri = ServiceBusEnvironment.CreateServiceUri("sb", 
                "test-blog", string.Empty);
string name = "RootManageSharedAccessKey";
string key = "abcdefghijklmopqrstuvwxyz";

TokenProvider tokenProvider = 
    TokenProvider.CreateSharedAccessSignatureTokenProvider(name, key);
NamespaceManager namespaceManager = 
    new NamespaceManager(uri, tokenProvider);
namespaceManager.CreateQueue("DataCollectionQueue");
```

请注意，存在 [CreateQueue](/dotnet/api/microsoft.servicebus.namespacemanager#Microsoft_ServiceBus_NamespaceManager_CreateQueue_System_String_) 方法的过载，其允许调整队列的属性。 例如，可为发送给队列的消息设置默认生存期 (TTL) 值。

### <a name="send-messages-to-the-queue"></a>将消息发送到队列
为了对服务总线实体进行运行时操作（例如发送和接收消息），应用程序必须首先创建 [MessagingFactory](/dotnet/api/microsoft.servicebus.messaging.messagingfactory) 对象。 类似于 [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) 类，[MessagingFactory](/dotnet/api/microsoft.servicebus.messaging.messagingfactory) 实例也从服务命名空间和令牌提供程序的基址创建。

```csharp
 BrokeredMessage bm = new BrokeredMessage(salesData);
 bm.Label = "SalesReport";
 bm.Properties["StoreName"] = "Redmond";
 bm.Properties["MachineID"] = "POS_1";
```

在服务总线队列中发送和接收的消息是 [BrokeredMessage](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage) 类的实例。 此类包含一组标准属性（如 [Label](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_Label) 和 [TimeToLive](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_TimeToLive)）、一个用来保存应用程序属性的词典以及大量随机应用程序数据。 应用程序可以通过传入任何可序列化对象来设置正文（下面的示例传入 **SalesData** 对象，表示来自 POS 终端的销售数据），它将使用 [DataContractSerializer](https://msdn.microsoft.com/library/system.runtime.serialization.datacontractserializer.aspx) 来序列化该对象。 或者，也可以提供 [Stream](https://msdn.microsoft.com/library/system.io.stream.aspx) 对象。

将消息发送到队列（在本例中即 **DataCollectionQueue**）的最简单方法是使用 [CreateMessageSender](/dotnet/api/microsoft.servicebus.messaging.messagingfactory#Microsoft_ServiceBus_Messaging_MessagingFactory_CreateMessageSender_System_String_) 从 [MessagingFactory](/dotnet/api/microsoft.servicebus.messaging.messagingfactory) 实例直接创建 [MessageSender](/dotnet/api/microsoft.servicebus.messaging.messagesender) 对象。

```csharp
MessageSender sender = factory.CreateMessageSender("DataCollectionQueue");
sender.Send(bm);
```

### <a name="receiving-messages-from-the-queue"></a>从队列接收消息
若要从队列接收消息，可使用 [MessageReceiver](/dotnet/api/microsoft.servicebus.messaging.messagereceiver) 对象，该对象是使用 [CreateMessageReceiver](/dotnet/api/microsoft.servicebus.messaging.messagingfactory#Microsoft_ServiceBus_Messaging_MessagingFactory_CreateMessageReceiver_System_String_) 从 [MessagingFactory](/dotnet/api/microsoft.servicebus.messaging.messagingfactory) 直接创建的。 消息接收方可在两种不同模式下工作：**ReceiveAndDelete** 和 **PeekLock**。 创建消息接收方时，[ReceiveMode](/dotnet/api/microsoft.servicebus.messaging.receivemode) 将会被设置为 [CreateMessageReceiver](/dotnet/api/microsoft.servicebus.messaging.messagingfactory?redirectedfrom=MSDN#Microsoft_ServiceBus_Messaging_MessagingFactory_CreateMessageReceiver_System_String_Microsoft_ServiceBus_Messaging_ReceiveMode_) 调用的一个参数。

当使用 **ReceiveAndDelete** 模式时，接收是一个单次操作；即，服务总线接收请求时，会将该消息标记为“正在使用”并将其返回给应用程序。 **ReceiveAndDelete** 模式是最简单的模式，最适合出现故障时应用程序允许不处理消息的方案。 为了理解这一点，可以考虑这样一种情形：使用方发出接收请求，但在处理该请求前发生了崩溃。 由于服务总线将消息标记为“已使用”，因此当应用程序重新启动并重新开始使用消息时，它会漏掉在发生崩溃前使用的消息。

在 **PeekLock** 模式下，接收变成了一个两阶段操作，从而可以支持无法容忍遗漏消息的应用程序。 当服务总线收到请求时，它会查找下一条要使用的消息，锁定该消息以防其他使用方接收，然后将该消息返回到应用程序。 应用程序完成消息处理（或可靠地存储消息以供将来处理）后，它将通过对收到的消息调用 [Complete](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_Complete) 完成接收过程的第二个阶段。 服务总线发现 [Complete](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_Complete) 调用时会将消息标记为“正在使用”。

可能会出现另外两种结果。 第一种，如果应用程序出于某种原因无法处理消息，可对接收的消息调用 [Abandon](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_Abandon)（而不是 [Complete](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_Complete)）。 这将引起服务总线解锁消息，使得同一个使用方或其他竞争使用方能够重新接收它。 第二种，还存在与锁定关联的超时，并且如果应用程序无法在锁定超时到期之前处理消息（例如，如果应用程序崩溃），服务总线将解锁该消息并使其可再次被接收（实质上是默认执行一个 [Abandon](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_Abandon) 操作）。

请注意，如果应用程序在处理消息之后，但在发出 [Complete](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_Complete) 请求之前发生崩溃，则在应用程序重新启动时会将该消息重新传送给应用程序。 这通常称为“至少一次”处理。 这表示每条消息将至少被处理一次，但在某些情况下，同一消息可能会被重新传送。 如果方案不容许重复处理，则应用程序中需要用于检测重复的其他逻辑。 可以根据消息的 [MessageId](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_MessageId) 属性实现此要求。 在多次传送尝试中，此属性的值将保持不变。 这称为“仅一次”处理。

此处显示的代码使用 **PeekLock** 模式接收和处理消息，如果未显式提供 [ReceiveMode](/dotnet/api/microsoft.servicebus.messaging.receivemode) 值，则这是默认设置。

```csharp
MessageReceiver receiver = factory.CreateMessageReceiver("DataCollectionQueue");
BrokeredMessage receivedMessage = receiver.Receive();
try
{
    ProcessMessage(receivedMessage);
    receivedMessage.Complete();
}
catch (Exception e)
{
    receivedMessage.Abandon();
}
```

### <a name="use-the-queue-client"></a>使用队列客户端
此部分中前面的示例从 [MessagingFactory](/dotnet/api/microsoft.servicebus.messaging.messagingfactory) 直接创建了 [MessageSender](/dotnet/api/microsoft.servicebus.messaging.messagesender) 和 [MessageReceiver](/dotnet/api/microsoft.servicebus.messaging.messagereceiver) 对象，分别用于向队列发送消息和从队列接收消息。 另一种方法是使用 [QueueClient](/dotnet/api/microsoft.servicebus.messaging.queueclient) 对象，该类支持发送和接收操作，并且还具有会话等更高级的功能。

```csharp
QueueClient queueClient = factory.CreateQueueClient("DataCollectionQueue");
queueClient.Send(bm);

BrokeredMessage message = queueClient.Receive();

try
{
    ProcessMessage(message);
    message.Complete();
}
catch (Exception e)
{
    message.Abandon();
} 
```

## <a name="next-steps"></a>后续步骤
了解了队列的基础知识后，请参阅[创建使用服务总线主题和订阅的应用程序](service-bus-create-topics-subscriptions.md)，以使用服务总线主题和订阅的发布/订阅功能继续这一讨论。


