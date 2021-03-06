---
title: "使用 Azure PowerShell 配置文件上传 | Microsoft Docs"
description: "如何使用 Azure PowerShell cmdlet 配置 IoT 中心，以便从连接的设备上传文件。 包括有关配置目标 Azure 存储帐户的信息。"
services: iot-hub
documentationcenter: 
author: dominicbetts
manager: timlt
editor: 
ms.assetid: 915f1597-272d-4fd4-8c5b-a0ccb1df0d91
ms.service: iot-hub
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/08/2017
ms.author: dobett
ms.translationtype: HT
ms.sourcegitcommit: f9003c65d1818952c6a019f81080d595791f63bf
ms.openlocfilehash: 66bb54927f3137aa7bfeb467962655a51bf71b4b
ms.contentlocale: zh-cn
ms.lasthandoff: 08/09/2017

---
# <a name="configure-iot-hub-file-uploads-using-powershell"></a>使用 PowerShell 配置 IoT 中心文件上传

[!INCLUDE [iot-hub-file-upload-selector](../../includes/iot-hub-file-upload-selector.md)]

要使用 [IoT 中心的文件上传功能][lnk-upload]，必须先将 Azure 存储帐户与 IoT 中心关联。 可以使用现有存储帐户，也可以创建新的存储帐户。

若要完成本教程，需要以下各项：

* 有效的 Azure 帐户。 如果没有帐户，只需几分钟即可创建一个[免费帐户][lnk-free-trial]。
* [Azure PowerShell cmdlet][lnk-powershell-install]。
* Azure IoT 中心。 如果没有 IoT 中心，可以使用 [New-AzureRmIoTHub cmdlet][lnk-powershell-iothub] 创建一个，或使用门户[创建 IoT 中心][lnk-portal-hub]。
* 一个 Azure 存储帐户。 如果没有 Azure 存储帐户，可以使用 [Azure 存储 PowerShell cmdlet][lnk-powershell-storage] 创建一个，或使用门户[创建存储帐户][lnk-portal-storage]。

## <a name="sign-in-and-set-your-azure-account"></a>登录并设置 Azure 帐户

登录到 Azure 帐户，并选择订阅。

1. 在 PowerShell 提示符下，运行 **Login-AzureRmAccount** cmdlet：

    ```powershell
    Login-AzureRmAccount
    ```

1. 如果有多个 Azure 订阅，则访问 Azure 即有权访问与凭据关联的所有 Azure 订阅。 使用以下命令，列出可供使用的 Azure 订阅：

    ```powershell
    Get-AzureRMSubscription
    ```

    使用以下命令，选择想要用于运行命令以管理 IoT 中心的订阅。 可使用上一命令输出中的订阅名称或 ID：

    ```powershell
    Select-AzureRMSubscription `
        -SubscriptionName "{your subscription name}"
    ```

## <a name="retrieve-your-storage-account-details"></a>检索存储帐户详细信息

以下步骤假设已使用 **Resource Manager** 部署模型而不**经典**部署模型创建了存储帐户。

若要从设备配置文件上传，需要 Azure 存储帐户的连接字符串。 存储帐户必须与 IoT 中心位于同一订阅中。 还需要存储帐户中 Blob 容器的名称。 使用以下命令检索存储帐户密钥：

```powershell
Get-AzureRmStorageAccountKey `
  -Name {your storage account name} `
  -ResourceGroupName {your storage account resource group}
```

记下 **key1** 存储帐户密钥值。 在后续步骤中需要用到它。

可将现有的 Blob 容器用于文件上传，或新建一个容器：

* 若要列出存储帐户中的现有 Blob 容器，请使用以下命令：

    ```powershell
    $ctx = New-AzureStorageContext `
        -StorageAccountName {your storage account name} `
        -StorageAccountKey {your storage account key}
    Get-AzureStorageContainer -Context $ctx
    ```

* 若要在存储帐户中创建 Blob 容器，请使用以下命令：

    ```powershell
    $ctx = New-AzureStorageContext `
        -StorageAccountName {your storage account name} `
        -StorageAccountKey {your storage account key}
    New-AzureStorageContainer `
        -Name {your new container name} `
        -Permission Off `
        -Context $ctx
    ```

## <a name="configure-your-iot-hub"></a>配置 IoT 中心

现在可以配置 IoT 中心启用[文件上传功能][lnk-upload]使用存储帐户详细信息。

配置需要以下值：

**存储容器**：当前 Azure 订阅中要与 IoT 中心关联的 Azure 存储帐户中的 Blob 容器。 检索在上一部分中必要的存储帐户信息。 IoT 中心会自动生成对此 Blob 容器具有写入权限的 SAS URI，以供设备上传文件时使用。

接收已上传文件的通知：启用或禁用文件上传通知。

**SAS TTL**：此设置是 IoT 中心返回给设备的 SAS URI 生存时间。 默认设置为一小时。

**文件通知设置默认 TTL**：文件上传通知到期前的生存时间。 默认设置为一天。

**文件通知最大传送数**：IoT 中心将尝试传送文件上传通知的次数。 默认设置为 10。

使用以下 PowerShell cmdlet 在 IoT 中心内配置上传文件设置：

```powershell
Set-AzureRmIotHub `
    -ResourceGroupName "{your iot hub resource group}" `
    -Name "{your iot hub name}" `
    -FileUploadNotificationTtl "01:00:00" `
    -FileUploadSasUriTtl "01:00:00" `
    -EnableFileUploadNotifications $true `
    -FileUploadStorageConnectionString "DefaultEndpointsProtocol=https;AccountName={your storage account name};AccountKey={your storage account key};EndpointSuffix=core.windows.net" `
    -FileUploadContainerName "{your blob container name}" `
    -FileUploadNotificationMaxDeliveryCount 10
```

## <a name="next-steps"></a>后续步骤

有关 IoT 中心文件上传功能的详细信息，请参阅[从设备上传文件][lnk-upload]。

若要了解有关如何管理 Azure IoT 中心的详细信息，请参阅以下链接：

* [批量管理 IoT 设备][lnk-bulk]
* [IoT 中心度量值][lnk-metrics]
* [操作监视][lnk-monitor]

若要进一步探索 IoT 中心的功能，请参阅：

* [IoT 中心开发人员指南][lnk-devguide]
* [使用 IoT Edge 模拟设备][lnk-iotedge]
* [从根本上保护 IoT 解决方案][lnk-securing]

[lnk-upload]: iot-hub-devguide-file-upload.md

[lnk-bulk]: iot-hub-bulk-identity-mgmt.md
[lnk-metrics]: iot-hub-metrics.md
[lnk-monitor]: iot-hub-operations-monitoring.md

[lnk-devguide]: iot-hub-devguide.md
[lnk-iotedge]: iot-hub-linux-iot-edge-simulated-device.md
[lnk-securing]: iot-hub-security-ground-up.md
[lnk-powershell-install]: https://docs.microsoft.com/powershell/azure/install-azurerm-ps
[lnk-powershell-storage]: https://docs.microsoft.com/powershell/module/azurerm.storage/
[lnk-powershell-iothub]: https://docs.microsoft.com/powershell/module/azurerm.iothub/new-azurermiothub
[lnk-portal-hub]: iot-hub-create-through-portal.md
[lnk-free-trial]: http://azure.microsoft.com/pricing/free-trial/
[lnk-portal-storage]: ../storage/storage-create-storage-account.md
