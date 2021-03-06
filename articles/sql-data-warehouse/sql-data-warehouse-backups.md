---
title: "Azure SQL 数据仓库备份 - 快照，异地冗余 | Microsoft Docs"
description: "了解 SQL 数据仓库的内置数据库备份，该功能可以将 Azure SQL 数据仓库还原到某个还原点或另一地理区域。"
services: sql-data-warehouse
documentationcenter: 
author: Lakshmi1812
manager: jhubbard
editor: 
ms.assetid: b5aff094-05b2-4578-acf3-ec456656febd
ms.service: sql-data-warehouse
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.custom: backup-restore
ms.date: 10/31/2016
ms.author: lakshmir;barbkess
ms.translationtype: HT
ms.sourcegitcommit: 2ad539c85e01bc132a8171490a27fd807c8823a4
ms.openlocfilehash: ef2e6e7a19c7ed1730fdec5eca73c941e1b319c4
ms.contentlocale: zh-cn
ms.lasthandoff: 07/12/2017

---
# <a name="sql-data-warehouse-backups"></a>SQL 数据仓库备份
SQL 数据仓库的数据仓库备份功能分为本地备份和异地备份， 其中包括 Azure 存储 Blob 快照和异地冗余存储。 使用数据仓库备份可以将数据仓库还原到主要区域的某个还原点，或者还原到另一地理区域。 本文介绍了在 SQL 数据仓库中进行备份的细节。

## <a name="what-is-a-data-warehouse-backup"></a>什么是数据仓库备份？
数据仓库备份是指可以用来将数据仓库还原到特定时间的数据。  由于 SQL 数据仓库属于分布式系统，因此数据仓库备份包含许多存储在 Azure blob 中的文件。 

数据库备份是任何业务连续性和灾难恢复策略的基本组成部分，因为数据库备份可以保护数据免遭意外损坏或删除。 有关详细信息，请参阅[业务连续性概述](../sql-database/sql-database-business-continuity.md)。

## <a name="data-redundancy"></a>数据冗余
SQL 数据仓库还可以通过将数据存储在本地冗余 (LRS) Azure 高级存储中来保护数据。 此 Azure 存储功能在本地数据中心存储数据的多个同步副本，确保在发生局部故障时提供透明的数据保护。 数据冗余可确保数据可以经受得住基础结构问题，例如磁盘故障。 数据冗余通过容错和高可用性基础结构来确保业务连续性。

详细了解以下内容：

* 有关 Azure 高级存储的信息，请参阅 [Azure 高级存储简介](../storage/storage-premium-storage.md)。
* 有关本地冗余存储的信息，请参阅 [Azure 存储复制](../storage/storage-redundancy.md#locally-redundant-storage)。

## <a name="azure-storage-blob-snapshots"></a>Azure 存储 Blob 快照
使用 Azure 高级存储的好处是，SQL 数据仓库可以使用 Azure 存储 Blob 快照在本地备份数据仓库。 可以将数据仓库还原到某个快照还原点。 快照至少每 8 小时启动一次，可供使用 7 天。  

详细了解以下内容：

* 有关 Azure Blob 快照的信息，请参阅[创建 Blob 快照](../storage/storage-blob-snapshots.md)。

## <a name="geo-redundant-backups"></a>异地冗余备份
SQL 数据仓库将完整的数据仓库存储在“标准”存储中，每隔 24 小时存储一次。 将根据上次快照的时间创建完整的数据仓库。 标准存储属于具有读取访问权限 (RA-GRS) 的异地冗余存储帐户。 Azure 存储 RA-GRS 功能可将备份文件复制到[配对的数据中心](../best-practices-availability-paired-regions.md)。 此异地复制功能可确保用户能够在无法访问主要区域中的快照的情况下还原数据仓库。 

此功能默认启用。 如果不想要使用异地冗余的备份，可以 [选择禁用] (https://docs.microsoft.com/powershell/resourcemanager/Azurerm.sql/v2.1.0/Set-AzureRmSqlDatabaseGeoBackupPolicy?redirectedfrom=msdn)。 

> [!NOTE]
> 在 Azure 存储中，术语*复制*是指将文件从一个位置复制到另一个位置。 SQL *数据库复制*是指让多个辅助数据库与主数据库保持同步。 
> 
> 

> [!NOTE]
> 无法取消 DWU 9000 和 DWU 18000 的地域冗余备份。 
>
> 

详细了解以下内容：

* 有关异地冗余存储的信息，请参阅 [Azure 存储复制](../storage/storage-redundancy.md)。
* 有关 RA-GRS 存储的信息，请参阅[读取访问异地冗余存储](../storage/storage-redundancy.md#read-access-geo-redundant-storage)。

## <a name="data-warehouse-backup-schedule-and-retention-period"></a>数据仓库备份计划和保留期
SQL 数据仓库每隔 4 到 8 小时创建一次联机数据仓库的快照，每个快照保留 7 天的时间。 可以将联机数据库还原到过去 7 天的某个还原点。 

若要查看上一个快照的启动时间，可对联机 SQL 数据仓库运行以下查询。 

```sql
select top 1 *
from sys.pdw_loader_backup_runs 
order by run_id desc;
```

若需保留某个快照 7 天以上的时间，可将还原点还原到新的数据仓库。 还原完以后，SQL 数据仓库就会开始在新的数据仓库创建快照。 如果不对新的数据仓库进行更改，则快照始终为空，因此快照费用最低。 也可以暂停数据库，让 SQL 数据仓库暂时不再创建快照。

### <a name="what-happens-to-my-backup-retention-while-my-data-warehouse-is-paused"></a>暂停数据仓库时，备份保留会发生什么情况？
暂停数据仓库时，SQL 数据仓库将不会创建快照，也不会让快照过期。 暂停数据仓库时，快照年龄不变。 快照保留期取决于数据仓库处于联机状态的天数，而不是日历日期。

例如，如果快照的启动时间是 10 月 1 日下午 4 点，而数据仓库在 10 月 3 日下午 4 点暂停，则快照的年龄为 2 天。 不管数据仓库何时回到联机状态，快照的年龄始终为 2 天。 如果数据仓库在 10 月 5 日下午 4 点回到联机状态，则快照的年龄为 2 天，还剩 5 天过期。

当数据仓库回到联机状态以后，SQL 数据仓库将继续创建新的快照，并且会让其数据年龄超过 7 天的快照过期。

### <a name="how-long-is-the-retention-period-for-a-dropped-data-warehouse"></a>已删除的数据仓库的保留期有多长？
删除数据仓库时，数据仓库和快照会在保存 7 天后删除。 可以将数据仓库还原到任何已保存的还原点。

> [!IMPORTANT]
> 如果删除某个逻辑 SQL Server 实例，则属于该实例的所有数据库也会删除，且无法恢复。 无法还原已删除的服务器。
> 
> 

## <a name="data-warehouse-backup-costs"></a>数据仓库备份费用
主数据仓库和 7 天 Azure Blob 快照的总费用按最接近的 TB 数进行计算。 例如，如果数据仓库为 1.5 TB，快照使用了 100 GB，则会按 2 TB 的数据以 Azure 高级存储费率计费。 

> [!NOTE]
> 每个快照最初是空的，随着用户对主数据仓库的更改而增大。 所有快照的大小都随数据仓库的更改而增加。 因此，快照的存储费用也会随因更改而导致的费率变化而增加。
> 
> 

如果使用的是异地冗余存储，则会单独收取异地存储费。 异地冗余存储按标准的读取访问异地冗余存储 (RA-GRS) 费率计费。

有关 SQL 数据仓库定价的详细信息，请参阅 [SQL 数据仓库定价](https://azure.microsoft.com/pricing/details/sql-data-warehouse/)。

## <a name="using-database-backups"></a>使用数据库备份
SQL 数据仓库备份的主要用途是在保留期内将数据仓库还原到其中一个还原点。  

* 若要还原 SQL 数据仓库，请参阅[还原 SQL 数据仓库](sql-data-warehouse-restore-database-overview.md)。

## <a name="related-topics"></a>相关主题
### <a name="scenarios"></a>方案
* 有关业务连续性概述，请参阅[业务连续性概述](../sql-database/sql-database-business-continuity.md)

<!-- ### Tasks -->

* 若要还原数据仓库，请参阅[还原 SQL 数据仓库](sql-data-warehouse-restore-database-overview.md)

<!-- ### Tutorials -->


