---
title: "Azure CLI 脚本 - 缩放用于 PostgreSQL 的 Azure 数据库 | Microsoft Docs"
description: "Azure CLI 脚本示例 - 在查询指标后将用于 PostgreSQL 服务器的 Azure 数据库缩放为不同的性能级别。"
services: postgresql
author: salonisonpal
ms.author: salonis
manager: jhubbard
editor: jasonwhowell
ms.service: postgresql-database
ms.devlang: azure-cli
ms.custom: mvc
ms.topic: sample
ms.date: 05/31/2017
ms.translationtype: Human Translation
ms.sourcegitcommit: 4f68f90c3aea337d7b61b43e637bcfda3c98f3ea
ms.openlocfilehash: 75efaa7dd6165fe0a3d3e35928107cae71e23d5a
ms.contentlocale: zh-cn
ms.lasthandoff: 06/20/2017

---
# <a name="monitor-and-scale-a-single-postgresql-server-using-azure-cli"></a>使用 Azure CLI 监视和缩放单个 PostgreSQL 服务器
此示例 CLI 脚本在查询指标后将用于 PostgreSQL 服务器的单个 Azure 数据库缩放为不同的性能级别。 

[!INCLUDE [cloud-shell-try-it](../../../includes/cloud-shell-try-it.md)]

如果选择在本地安装并使用 CLI，本主题要求运行 Azure CLI 2.0 版或更高版本。 运行 `az --version` 即可查找版本。 如果需要进行安装或升级，请参阅[安装 Azure CLI 2.0]( /cli/azure/install-azure-cli)。 

## <a name="sample-script"></a>示例脚本
在此示例脚本中，更改突出显示的行，以自定义管理员用户名和密码。 将 Azure Monitor 中使用的订阅 ID 替换为自己的订阅 ID。
[!code-azurecli-interactive[主要](../../../cli_scripts/postgresql/scale-postgresql-server/scale-postgresql-server.sh?highlight=15-16 "创建并缩放 Azure Database for PostgreSQL。")]

## <a name="clean-up-deployment"></a>清理部署
运行脚本示例后，可以使用以下命令删除资源组以及与其关联的所有资源。
[!code-azurecli-interactive[主要](../../../cli_scripts/postgresql/scale-postgresql-server/delete-postgresql.sh "删除资源组。")]

## <a name="script-explanation"></a>脚本说明
此脚本使用以下命令。 表中的每条命令均链接到特定于命令的文档。

| **命令** | **说明** |
|---|---|
| [az group create](/cli/azure/group#create) | 创建用于存储所有资源的资源组。 |
| [az postgres server create](/cli/azure/postgres/server#create) | 创建托管数据库的 PostgreSQL 服务器。 |
| [az monitor metrics list](/cli/azure/monitor/metrics#list) | 列出资源的指标值。 |
| [az group delete](/cli/azure/group#delete) | 删除资源组，包括所有嵌套的资源。 |

## <a name="next-steps"></a>后续步骤
- 有关 Azure CLI 的详细信息，请参阅 [Azure CLI 文档](/cli/azure/overview)
- 尝试其他脚本：[Azure Database for PostgreSQL 的 Azure CLI 示例](../sample-scripts-azure-cli.md)
- 有关缩放的详细信息，请参阅[服务层](../concepts-service-tiers.md)和[计算单元和存储单元](../concepts-compute-unit-and-storage.md)

