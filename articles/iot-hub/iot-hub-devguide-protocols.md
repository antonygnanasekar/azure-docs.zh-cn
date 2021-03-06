---
title: "Azure IoT 中心通信协议和端口 | Microsoft Docs"
description: "开发人员指南 - 介绍了设备到云和云到设备通信支持的通信协议和必须打开的端口号。"
services: iot-hub
documentationcenter: .net
author: dominicbetts
manager: timlt
editor: 
ms.assetid: 3fc5f1a3-3711-4611-9897-d4db079b4250
ms.service: iot-hub
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/25/2017
ms.author: dobett
ms.translationtype: Human Translation
ms.sourcegitcommit: 5edc47e03ca9319ba2e3285600703d759963e1f3
ms.openlocfilehash: 98004a48734e33f85eebf8f6213d9f0751dea843
ms.contentlocale: zh-cn
ms.lasthandoff: 06/01/2017


---
# <a name="reference---choose-a-communication-protocol"></a>参考 - 选择通信协议

IoT 中心允许设备使用 [MQTT][lnk-mqtt]、基于 WebSocket 的 MQQT、[AMQP][lnk-amqp]、基于 WebSocket 的 AMQP 和 HTTP 协议进行设备端通信。 若要了解这些协议如何支持特定的 IoT 中心功能，请参阅[设备到云通信指南][lnk-d2c-guidance]和[云到设备通信指南][lnk-c2d-guidance]。

下表提供了针对协议选取的高水平建议：

| 协议 | 何时应选择此协议 |
| --- | --- |
| MQTT <br> 基于 WebSocket 的 MQTT |用于无需使用相同的 TLS 连接来连接多台设备（各有自己的设备凭据）的所有设备。 |
| AMQP <br> 基于 WebSocket 的 AMQP |用于利用跨设备连接复用的字段和云网关。 |
| HTTP |用于不可支持其他协议的设备。 |

在选择设备端通信协议时，请考虑以下几点：

* **从云到设备模式**。 HTTP 没有用于实现服务器推送的有效方法。 因此，使用 HTTP 时，设备会在 IoT 中心轮询从云到设备的消息。 此方法对于设备和 IoT 中心而言是低效的。 根据当前 HTTP 准则，每台设备应每 25 分钟或更长时间轮询一次消息。 而 MQTT 和 AMQP 支持服务器在收到云到设备的消息时进行推送。 它们将启用从 IoT 中心到设备的直接消息推送。 如果传送延迟是考虑因素，最好使用 MQTT 或 AMQP 协议。 对于很少连接的设备，HTTP 也适用。
* **现场网关**。 使用 MQTT 和 HTTP 时，无法使用相同的 TLS 连接来连接多台设备（各有自己的设备凭据）。 因此，对于[现场网关方案][lnk-azure-gateway-guidance]，这些并不是最理想的协议，因为对于每个连接到现场网关的设备，这些协议需要在现场网关和 IoT 中心之间建立一个 TLS 连接。
* **低资源设备**。 MQTT 和 HTTP 库的占用空间比 AMQP 库的占用空间小。 因此，如果设备的资源很少（如低于 1 MB RAM），可能只可实现这些协议。
* **网络遍历**。 标准 AMQP 协议使用端口 5671，而 MQTT 在端口 8883 上侦听，这可能会导致对非 HTTP 协议关闭的网络发生问题。 基于 WebSocket 的 MQTT、基于 WebSocket 的 AMQP 和 HTTP 均可用于此方案。
* **有效负载大小**。 MQTT 和 AMQP 是二进制协议，因此其有效负载比 HTTP 的有效负载更精简。

> [!WARNING]
> 使用 HTTP 时，每台设备应每 25 分钟或更长时间轮询一次从云到设备的消息。 但在开发期间，可按低于 25 分钟的更高频率进行轮询。

## <a name="port-numbers"></a>端口号

设备可在 Azure 中使用各种协议来与 IoT 中心通信。 通常，选择的协议根据解决方案的具体要求而定。 下表列出了必须打开的、使设备能够使用特定协议的出站端口：

| 协议 | 端口 |
| --- | --- |
| MQTT |8883 |
| 基于 WebSocket 的 MQTT |443 |
| AMQP |5671 |
| 基于 WebSockets 的 AMQP |443 |
| HTTP |443 |

在 Azure 区域创建 IoT 中心后，该 IoT 中心将在生存期内保留同一 IP 地址。 但为保证服务质量，如果 Microsoft 调整 IoT 中心的大小，则向其分配新的 IP 地址。


## <a name="next-steps"></a>后续步骤

若要深入了解 IoT 中心如何实现 MQTT 协议，请参阅[使用 MQTT 协议与 IoT 中心通信][lnk-mqtt-support]。

[lnk-d2c-guidance]: iot-hub-devguide-d2c-guidance.md
[lnk-c2d-guidance]: iot-hub-devguide-c2d-guidance.md
[lnk-mqtt-support]: iot-hub-mqtt-support.md
[lnk-amqp]: http://docs.oasis-open.org/amqp/core/v1.0/os/amqp-core-complete-v1.0-os.pdf
[lnk-mqtt]: http://docs.oasis-open.org/mqtt/mqtt/v3.1.1/mqtt-v3.1.1.pdf
[lnk-azure-gateway-guidance]: iot-hub-devguide-endpoints.md#field-gateways

