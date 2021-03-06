---
title: "在 HDInsight 群集创建过程中添加 Hive 库 — Azure | Microsoft Docs"
description: "了解如何在群集创建中将 Hive 库（jar 文件）添加到 HDInsight 群集中。"
services: hdinsight
documentationcenter: 
author: Blackmist
manager: jhubbard
editor: cgronlun
ms.assetid: 2fd74b8d-c006-45c6-a9e2-72ff5d2d978a
ms.service: hdinsight
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: big-data
ms.date: 07/12/2017
ms.author: larryfr
ms.custom: H1Hack27Feb2017,hdinsightactive
ms.translationtype: HT
ms.sourcegitcommit: 54774252780bd4c7627681d805f498909f171857
ms.openlocfilehash: 3412864384961e8820d6700c1bf22a4cae64ba4b
ms.contentlocale: zh-cn
ms.lasthandoff: 07/28/2017

---
# <a name="add-custom-hive-libraries-when-creating-your-hdinsight-cluster"></a>创建 HDInsight 群集时添加自定义 Hive 库

如果你有经常与 HDInsight 上的 Hive 配合使用的库，本文档包含有关在群集创建期间使用脚本操作预加载库的信息。 使用本文档中的步骤添加的库在 Hive 中全局可用 - 不需使用 [ADD JAR](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+Cli) 来加载。

## <a name="how-it-works"></a>工作原理

创建群集时，可以选择指定脚本操作，以便在创建群集节点时在其上运行脚本。 本文档中的脚本接受单个参数，即包含将预加载的库 （存储为 jar 文件）的 WASB 位置。

在群集创建过程中，该脚本将枚举文件、将这些文件复制到头节点和工作节点上的 `/usr/lib/customhivelibs/` 目录，然后将它们添加到 `core-site.xml` 文件中的 `hive.aux.jars.path` 属性。 在基于 Linux 的群集中，它还会针对文件位置更新 `hive-env.sh` 文件。

> [!NOTE]
> 使用本文中的脚本操作使库可用于以下方案：
>
> * **基于 Linux 的 HDInsight** - 使用 Hive 客户端、**WebHCat** 和 **HiveServer2** 时。
> * **基于 Windows 的 HDInsight** - 使用 Hive 客户端和 **WebHCat** 时。

## <a name="the-script"></a>脚本

**脚本位置**

对于**基于 Linux 的群集**：[https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v01.sh](https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v01.sh)

对于**基于 Windows 的群集**：[https://hdiconfigactions.blob.core.windows.net/setupcustomhivelibsv01/setup-customhivelibs-v01.ps1](https://hdiconfigactions.blob.core.windows.net/setupcustomhivelibsv01/setup-customhivelibs-v01.ps1)

> [!IMPORTANT]
> Linux 是 HDInsight 3.4 或更高版本上使用的唯一操作系统。 有关详细信息，请参阅 [HDInsight 在 Windows 上停用](hdinsight-component-versioning.md#hdinsight-windows-retirement)。

**要求**

* 这些脚本必须同时应用于“头节点”和“辅助角色节点”。

* 要安装的 jar 必须存储在**单个容器**中的 Azure Blob 存储中。

* 在创建期间，包含 jar 文件的库的存储帐户**必须**链接到 HDInsight 群集。 它必须是默认的存储帐户，或者是通过__可选配置__添加的帐户。

* 必须指定容器的 WASB 路径作为脚本操作的参数。 例如，如果 jar 存储在名为 **mystorage** 的存储帐户上名为 **libs** 的容器中，则该参数应为 **wasb://libs@mystorage.blob.core.windows.net/**。

  > [!NOTE]
  > 本文档假定你已创建存储帐户、blob 容器，并已将文件上传到该容器。
  >
  > 如果尚未创建存储帐户，可以通过 [Azure 门户](https://portal.azure.com)创建该帐户。 然后可以使用实用程序（如 [Azure 存储资源管理器](http://storageexplorer.com/)）在帐户中创建一个容器并将文件上传到该容器。

## <a name="create-a-cluster-using-the-script"></a>使用脚本创建群集。

> [!NOTE]
> 以下步骤创建基于 Linux 的 HDInsight 群集。 若要创建基于 Windows 的群集，创建群集时请选择 **Windows** 作为群集 OS，并使用 Windows (PowerShell) 脚本而不是 bash 脚本。
>
> 也可以使用 Azure PowerShell 或 HDInsight .NET SDK 来使用此脚本创建群集。 有关使用这些方法的详细信息，请参阅[使用脚本操作自定义 HDInsight 群集](hdinsight-hadoop-customize-cluster-linux.md)。

1. 使用[预配 Linux 上的 HDInsight 群集](hdinsight-hadoop-provision-linux-clusters.md)中的步骤开始预配群集，但不要完成预配。

2. 在“可选配置”边栏选项卡上，选择“脚本操作”，并提供以下信息：

   * **名称**：输入脚本操作的友好名称。

   * **脚本 URI**：https://hdiconfigactions.blob.core.windows.net/linuxsetupcustomhivelibsv01/setup-customhivelibs-v01.sh

   * **标头**：选中此选项。

   * **辅助角色**：选中此选项。

   * **ZOOKEEPER**：将此选项留空。

   * **参数**：输入包含 jar 的容器和存储帐户的 WASB 地址。 例如，**wasb://libs@mystorage.blob.core.windows.net/**。

3. 在“脚本操作”的底部，使用“选择”按钮保存配置。

4. 在“可选配置”边栏选项卡上，选择“链接存储帐户”，然后选择“添加存储密钥”链接。 选择包含 jar 的存储帐户，然后使用“选择”按钮保存设置并返回“可选配置”边栏选项卡。

5. 使用“可选配置”边栏选项卡底部的“选择”按钮以保存可选配置信息。

6. 继续按[预配 Linux 上的 HDInsight 群集](hdinsight-hadoop-provision-linux-clusters.md)中所述预配群集。

群集创建完成后，将能够使用通过此脚本从 Hive 添加的 jar，而无需使用 `ADD JAR` 语句。

## <a name="next-steps"></a>后续步骤

有关使用 Hive 的详细信息，请参阅[将 Hive 与 HDInsight 配合使用](hdinsight-use-hive.md)

