---
title: "使用 Azure CLI 2.0 打开 Linux VM 的端口 | Microsoft Docs"
description: "了解如何使用 Azure Resource Manager 部署模型和 Azure CLI 2.0 在 Linux VM 上打开端口/创建终结点"
services: virtual-machines-linux
documentationcenter: 
author: iainfoulds
manager: timlt
editor: 
ms.assetid: eef9842b-495a-46cf-99a6-74e49807e74e
ms.service: virtual-machines-linux
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 05/11/2017
ms.author: iainfou
ms.translationtype: Human Translation
ms.sourcegitcommit: 97fa1d1d4dd81b055d5d3a10b6d812eaa9b86214
ms.openlocfilehash: f1b4910ed0f63fd4914113e83e9fe4b483147c3a
ms.contentlocale: zh-cn
ms.lasthandoff: 05/11/2017


---
# <a name="open-ports-and-endpoints-to-a-linux-vm-with-the-azure-cli"></a>使用 Azure CLI 打开 Linux VM 的端口和终结点
通过在子网或 VM 网络接口上创建网络筛选器可为 Azure 中的虚拟机 (VM) 打开端口或创建终结点。 将这些筛选器（控制入站和出站流量）放在网络安全组中，并附加到将接收流量的资源。 让我们在端口 80 上使用 Web 流量的常见示例。 本文说明如何使用 Azure CLI 2.0 打开 VM 的端口。 还可以使用 [Azure CLI 1.0](nsg-quickstart-nodejs.md) 执行这些步骤。


## <a name="quick-commands"></a>快速命令
若要创建网络安全组和规则，需要安装最新的 [Azure CLI 2.0](/cli/azure/install-az-cli2)，并使用 [az login](/cli/azure/#login) 登录到 Azure 帐户。

在以下示例中，请将示例参数名称替换为自己的值。 示例参数名称包括 myResourceGroup、myNetworkSecurityGroup 和 myVnet。

使用 [az network nsg create](/cli/azure/network/nsg#create) 创建网络安全组。 以下示例在 eastus 位置创建名为 myNetworkSecurityGroup 的网络安全组：

```azurecli
az network nsg create \
    --resource-group myResourceGroup \
    --location eastus \
    --name myNetworkSecurityGroup
```

借助 [az 网络 nsg 规则创建](/cli/azure/network/nsg/rule#create)添加规则以允许 HTTP 流量流向 Web 服务器（或者根据自己的情况（例如 SSH 访问或数据库连接）来调整此规则）。 以下示例创建一个名为 myNetworkSecurityGroupRule 的规则，以允许端口 80 上的 TCP 流量：

```azurecli
az network nsg rule create \
    --resource-group myResourceGroup \
    --nsg-name myNetworkSecurityGroup \
    --name myNetworkSecurityGroupRule \
    --protocol tcp \
    --priority 1000 \
    --destination-port-range 80
```

借助 [az 网络 nic 更新](/cli/azure/network/nic#update)将网络安全组与 VM 的网络接口 (NIC) 相关联。 以下示例将名为 myNic 的现有 NIC 与名为 myNetworkSecurityGroup 的网络安全组相关联：

```azurecli
az network nic update \
    --resource-group myResourceGroup \
    --name myNic \
    --network-security-group myNetworkSecurityGroup
```

或者，也可以借助 [az 网络 vnet 子网更新](/cli/azure/network/vnet/subnet#update)将网络安全组与虚拟网络的子网相关联，而不是只与单个 VM 上的网络接口相关联。 以下示例将 myVnet 虚拟网络中名为 mySubnet 的现有子网与名为 myNetworkSecurityGroup 的网络安全组相关联：

```azurecli
az network vnet subnet update \
    --resource-group myResourceGroup \
    --vnet-name myVnet \
    --name mySubnet \
    --network-security-group myNetworkSecurityGroup
```

## <a name="more-information-on-network-security-groups"></a>有关网络安全组的详细信息
利用此处的快速命令，可以让流向 VM 的流量开始正常运行。 网络安全组提供许多出色的功能和粒度来控制资源的访问。 可以[在此处详细了解如何创建网络安全组和 ACL 规则](../../virtual-network/virtual-networks-create-nsg-arm-cli.md)。

可以将网络安全组和 ACL 规则定义为 Azure Resource Manager 模板的一部分。 详细了解如何[使用模板创建网络安全组](../../virtual-network/virtual-networks-create-nsg-arm-template.md)。

如果需要使用端口转发将唯一的外部端口映射至 VM 上的内部端口，则需要使用负载均衡器和网络地址转换 (NAT) 规则。 例如，你可能想对外公开 TCP 端口 8080，然后让流量定向到 VM 上的 TCP 端口 80。 可以了解如何[创建面向 Internet 的负载均衡器](../../load-balancer/load-balancer-get-started-internet-arm-cli.md)。

## <a name="next-steps"></a>后续步骤
在本示例中，你创建了简单的规则来允许 HTTP 流量。 你可以从下列文章中，找到有关创建更详细环境的信息：

* [Azure Resource Manager 概述](../../azure-resource-manager/resource-group-overview.md)
* [什么是网络安全组 (NSG)？](../../virtual-network/virtual-networks-nsg.md)
* [Azure Resource Manager 中负载均衡器的概述](../../load-balancer/load-balancer-arm.md)


