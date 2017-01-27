---
title: "在两个 Azure 虚拟网络之间配置 DNS | Microsoft Docs"
description: "了解如何在两个 Azure 虚拟网络之间配置 VPN 连接和域名解析，以及如何配置 HBase 异地复制。"
services: hdinsight,virtual-network
documentationcenter: 
author: mumian
manager: jhubbard
editor: cgronlun
ms.assetid: 881f4c60-0cac-481b-aedb-e7c0c9400df1
ms.service: hdinsight
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 06/28/2016
ms.author: jgao
translationtype: Human Translation
ms.sourcegitcommit: f9b191a68fe19f30aa157fd01f33afb0a4f1e279
ms.openlocfilehash: cb9fa493a10c5ebd8bf14876c76d58c018abaa50


---
# <a name="configure-dns-between-two-azure-virtual-networks"></a>在两个 Azure 虚拟网络之间配置 DNS
> [!div class="op_single_selector"]
> * [配置 VPN 连接](hdinsight-hbase-geo-replication-configure-vnets.md)
> * [配置 DNS](hdinsight-hbase-geo-replication-configure-dns.md)
> * [配置 HBase 复制](hdinsight-hbase-geo-replication.md)
>
>

了解如何向 Azure 虚拟网络添加和配置 DNS 服务器，以处理虚拟机内部和跨虚拟网络的名称解析。

本教程是有关创建 HBase 异地复制的[系列][hdinsight-hbase-geo-replication]教程的第二部分：

* [在两个虚拟网络之间配置 VPN 连接][hdinsight-hbase-geo-replication-vnet]
* 为虚拟网络配置 DNS（本教程）
* [配置 HBase 异地复制][hdinsight-hbase-geo-replication]

下图演示了在[在两个虚拟网络之间配置 VPN 连接][hdinsight-hbase-geo-replication-vnet]中创建的两个虚拟网络：

![HDInsight HBase 复制虚拟网络示意图][img-vnet-diagram]

## <a name="prerequisites"></a>先决条件
在开始阅读本教程前，你必须具有：

* **一个 Azure 订阅**。 请参阅 [获取 Azure 免费试用版](https://azure.microsoft.com/documentation/videos/get-azure-free-trial-for-testing-hadoop-in-hdinsight/)。
* **配备 Azure PowerShell 的工作站**。

    在运行 PowerShell 脚本之前，请确保你已使用以下 cmdlet 连接到 Azure 订阅：

        Add-AzureAccount

    如果具有多个 Azure 订阅，请使用以下 cmdlet 设置当前订阅：

        Select-AzureSubscription <AzureSubscriptionName>

    > [!IMPORTANT]
    > Azure PowerShell 支持使用 Azure Service Manager 管理 HDInsight 资源，但**不建议使用**，而且将于 2017 年 1 月 1 日前删除。 本文档中的步骤使用的是与 Azure Resource Manager 兼容的新 HDInsight cmdlet。
    >
    > 请按照 [安装和配置 Azure PowerShell](/powershell/azureps-cmdlets-docs) 中的步骤安装最新版本的 Azure PowerShell。 如果你的脚本需要修改后才能使用与 Azure Resource Manager 兼容的新 cmdlet，请参阅 [迁移到适用于 HDInsight 群集的基于 Azure Resource Manager 的开发工具](hdinsight-hadoop-development-using-azure-resource-manager.md) ，了解详细信息。

* **使用 VPN 连接的两个 Azure 虚拟网络**。  有关说明，请参阅[在两个 Azure 虚拟网络之间配置 VPN 连接][hdinsight-hbase-geo-replication-vnet]。

    > [!NOTE]
    > Azure 服务名称和虚拟机名称必须是唯一的。 本教程中使用的名称是 Contoso-[Azure Service/VM name]-[EU/US]。 例如，Contoso-VNet-EU 是位于北欧数据中心的 Azure 虚拟网络；Contoso-DNS-US 是位于美国东部数据中心的 DNS 服务器 VM。 你必须选择适合自己的名称。

## <a name="create-azure-virtual-machines-to-be-used-as-dns-servers"></a>创建要用作 DNS 服务器的 Azure 虚拟机
**在 Contoso-VNet-EU 中创建名为 Contoso-DNS-EU 的虚拟机**

1. 单击“新建”、“计算”、“虚拟机”、“从库中”。
2. 选择“Windows Server 2012 R2 Datacenter”。
3. 输入：
   * **虚拟机名称**：Contoso-DNS-EU
   * **新用户名**：
   * **新密码**：
4. 输入：

   * **云服务**：创建新的云服务
   * **区域/地缘组/虚拟网络**：（选择 Contoso-VNet-EU）
   * **虚拟网络子网**：Subnet-1
   * **存储帐户**：使用自动生成的存储帐户

     云服务名称将与虚拟机名称相同。 在本例中为 Contoso-DNS-EU。 对于后续虚拟机，可以选择使用同一个云服务。  同一个云服务下的所有虚拟机共享同一个虚拟网络和域后缀。

     存储帐户用于存储虚拟机映像文件。
   * **终结点**：（向下滚动并选择“DNS”）

创建虚拟机后，找出内部 IP 和外部 IP。

1. 单击虚拟机名称 **Contoso-DNS-EU**。
2. 单击“仪表板”。
3. 记下：
   * 公用虚拟 IP 地址
   * 内部 IP 地址

**在 Contoso-VNet-US 中创建名为 Contoso-DNS-US 的虚拟机**

* 重复相同的过程，使用以下值创建虚拟机：
  * 虚拟机名称：Contoso-DNS-US
  * 区域/地缘组/虚拟网络：选择 Contoso-VNET-US
  * 虚拟网络子网：Subnet-1
  * 存储帐户：使用自动生成的存储帐户
  * 终结点：（选择 DNS）

## <a name="set-static-ip-addresses-for-the-two-virtual-machines"></a>设置两个虚拟机的静态 IP 地址
DNS 服务器需要静态 IP 地址。  不能从 Azure 经典门户完成此步骤。 需要使用 Azure PowerShell。

**配置两个虚拟机的静态 IP 地址**

1. 打开 Windows PowerShell ISE。
2. 运行以下 cmdlet：

    ```powershell
    Add-AzureAccount
    Select-AzureSubscription [YourAzureSubscriptionName]

    Get-AzureVM -ServiceName Contoso-DNS-EU -Name Contoso-DNS-EU | Set-AzureStaticVNetIP -IPAddress 10.1.0.4 | Update-AzureVM
    Get-AzureVM -ServiceName Contoso-DNS-US -Name Contoso-DNS-US | Set-AzureStaticVNetIP -IPAddress 10.2.0.4 | Update-AzureVM
    ```

    ServiceName 是云服务名称。 由于 DNS 服务器是云服务的第一个虚拟机，因此云服务名称与虚拟机名称相同。

    你可能需要更新 ServiceName 和名称，以匹配你使用的名称。

## <a name="add-the-dns-server-role-the-two-virtual-machines"></a>添加两个虚拟机的 DNS 服务器角色
**为 Contoso-DNS-EU 添加 DNS 服务器角色**

1. 在 Azure 经典门户中，单击左侧的“虚拟机”。
2. 单击“Contoso-DNS-EU”。
3. 在顶部单击“仪表板”。
4. 在底部单击“连接”，然后根据说明通过 RDP 连接到虚拟机。
5. 在 RDP 会话中，单击左下角的 Windows 按钮打开“开始”屏幕。
6. 单击“服务器管理器”磁贴。
7. 单击“添加角色和功能”。
8. 单击“下一步”
9. 选择“基于角色或基于功能的安装”，然后单击“下一步”。
10. 选择你的 DNS 虚拟机（应已突出显示），然后单击“下一步”。
11. 选中“DNS 服务器”。
12. 单击“添加功能”，然后单击“继续”。
13. 单击“下一步”三次，然后单击“安装”。

**为 Contoso-DNS-US 添加 DNS 服务器角色**

* 重复上述步骤，向 **Contoso-DNS-US** 添加 DNS 角色。

## <a name="assign-dns-servers-to-the-virtual-networks"></a>将 DNS 服务器分配到虚拟网络
**注册两个 DNS 服务器**

1. 在 Azure 经典门户中，单击“新建”、“网络服务”、“虚拟网络”、“注册 DNS 服务器”。
2. 输入：

   * **名称**：Contoso-DNS-EU
   * **DNS 服务器 IP 地址**：10.1.0.4 – IP 地址必须匹配 DNS 服务器虚拟机的 IP 地址。
3. 重复该过程，使用以下设置注册 Contoso-DNS-US：

   * **名称**：Contoso-DNS-US
   * **DNS 服务器 IP 地址**：10.2.0.4

**将两个 DNS 服务器分配到两个虚拟网络**

1. 在经典门户的左窗格中单击“网络”。
2. 单击“Contoso-VNet-EU”。
3. 单击“配置”。
4. 在“DNS 服务器”部分中选择“Contoso-DNS-EU”。
5. 单击页面底部的“保存”，然后单击“是”以确认。
6. 重复该过程，将 **Contoso-DNS-US** DNS 服务器分配到 **Contoso-VNet-US** 虚拟网络。

必须重新启动已部署到虚拟网络的所有虚拟机，以更新 DNS 服务器配置。

**重启虚拟机**

1. 在 Azure 经典门户中，单击左侧的“虚拟机”。
2. 单击“Contoso-DNS-EU”。
3. 在顶部单击“仪表板”。
4. 在底部单击“重启”。
5. 重复相同的步骤来重启 **Contoso-DNS-US**。

## <a name="configure-dns-conditional-forwarders"></a>配置 DNS 条件转发器
每个虚拟网络上的 DNS 服务器只能解析该虚拟网络中的 DNS 名称。 你需要配置一个指向对等 DNS 服务器的条件转发器，以便在对等虚拟网络中进行名称解析。

若要配置条件转发器，你需要知道两个 DNS 服务器的域后缀。 根据你在创建虚拟机时使用的云服务配置，DNS 后缀可能有所不同。 对于虚拟网络中使用的每个 DNS 后缀，必须添加一个条件转发器。

**查找两个 DNS 服务器的域后缀**

1. 通过 RDP 访问 **Contoso-DNS-EU**。
2. 打开 Windows PowerShell 控制台或命令提示符。
3. 运行 **ipconfig**，并记下**特定于连接的 DNS 后缀**。
4. 请不要关闭 RDP 会话，稍后你仍要用到它。
5. 重复相同的步骤，以找出 **Contoso-DNS-US** 的**特定于连接的 DNS 后缀**。

**配置 DNS 转发器**

1. 在与 **Contoso-DNS-EU** 建立的 RDP 会话中，单击左下角的 Windows 键。
2. 单击“管理工具”。
3. 单击“DNS”。
4. 在左窗格中，展开“DSN”、“Contoso-DNS-EU”。
5. 输入以下信息：
   * **DNS 域**：输入 Contoso-DNS-US 的 DNS 后缀。 例如：Contoso-DNS-US.b5.internal.cloudapp.net。
   * **主服务器的 IP 地址**：输入 10.2.0.4，这是 Contoso-DNS-US 的 IP 地址。
6. 按 **Enter**，然后单击“确定”。  现在，你可以从 Contoso-DNS-EU 解析 Contoso-DNS-US 的 IP 地址。
7. 重复上述步骤，以使用以下值将 DNS 转发器添加到 Contoso-DNS-US 虚拟机上的 DNS 服务：
   * **DNS 域**：输入 Contoso-DNS-EU 的 DNS 后缀。
   * **主服务器的 IP 地址**：输入 10.2.0.4，这是 Contoso-DNS-EU 的 IP 地址。

## <a name="test-the-name-resolution-across-the-virtual-networks"></a>跨虚拟网络测试名称解析
现在，你可以跨虚拟网络测试主机名解析。 默认情况下，防火墙会阻止 Ping。  你可以使用 nslookup 来解析对等网络中的 DNS 服务器虚拟机（必须使用 FQDN）。

## <a name="next-steps"></a>后续步骤
在本教程中，你已学习如何使用 VPN 连接跨虚拟网络配置名称解析。 本系列教程包括的其他两篇文章：

* [在两个 Azure 虚拟网络之间配置 VPN 连接][hdinsight-hbase-geo-replication-vnet]
* [配置 HBase 异地复制][hdinsight-hbase-geo-replication]

[hdinsight-hbase-geo-replication]: hdinsight-hbase-geo-replication.md
[hdinsight-hbase-geo-replication-vnet]: hdinsight-hbase-geo-replication-configure-vnets.md
[powershell-install]: /powershell/azureps-cmdlets-docs

[img-vnet-diagram]: ./media/hdinsight-hbase-geo-replication-configure-DNS/HDInsight.HBase.VPN.diagram.png



<!--HONumber=Dec16_HO2-->

