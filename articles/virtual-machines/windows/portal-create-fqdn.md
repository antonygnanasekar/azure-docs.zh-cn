---
title: "在 Azure 门户中为 Windows VM 创建 FQDN | Microsoft 文档"
description: "了解如何在 Azure 门户中为基于 Resource Manager 的虚拟机创建完全限定域名或 FQDN。"
services: virtual-machines-windows
documentationcenter: 
author: iainfoulds
manager: timlt
editor: tysonn
tags: azure-resource-manager
ms.assetid: a2ae5887-76df-485e-ae19-0efd96df8600
ms.service: virtual-machines-windows
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure-services
ms.date: 07/05/2017
ms.author: iainfou
ms.custom: H1Hack27Feb2017
ms.translationtype: Human Translation
ms.sourcegitcommit: bb794ba3b78881c967f0bb8687b1f70e5dd69c71
ms.openlocfilehash: da91ce0f14f904530a1a53af00485f5a2ac1757b
ms.contentlocale: zh-cn
ms.lasthandoff: 07/06/2017


---
# <a name="create-a-fully-qualified-domain-name-in-the-azure-portal-for-a-windows-vm"></a>在 Azure 门户中为 Windows VM 创建完全限定的域名

在 [Azure 门户](https://portal.azure.com)中创建虚拟机 (VM) 时，该门户会为虚拟机自动创建一个公共 IP 资源。 可以使用此 IP 地址远程访问 VM。 虽然门户不会创建[完全限定的域名](https://en.wikipedia.org/wiki/Fully_qualified_domain_name)（简称 FQDN），但可以在创建 VM 后创建一个完全限定的域名。 本文演示了创建 DNS 名称或 FQDN 的步骤。

## <a name="create-fqdn"></a>创建 FQDN
本文假设你已创建一个 VM。 如果需要，可以[在门户中创建 VM](quick-create-portal.md)或[通过 Azure PowerShell 创建 VM](quick-create-powershell.md)。 在启动和运行 VM 后，请执行下列步骤：

[!INCLUDE [virtual-machines-common-portal-create-fqdn](../../../includes/virtual-machines-common-portal-create-fqdn.md)]

你现在可以使用此 DNS 名称（例如远程桌面协议 (RDP)）远程连接至 VM。

## <a name="next-steps"></a>后续步骤
你的 VM 已经有公共 IP 和 DNS 名称，现在可以部署通用应用程序框架或服务，例如 IIS、SQL 或 SharePoint。

你也可以深入了解[使用 Resource Manager](../../azure-resource-manager/resource-group-overview.md)，以获取生成 Azure 部署的相关提示。


