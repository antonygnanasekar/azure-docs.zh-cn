﻿<properties
   pageTitle="确认已恢复的 Azure SQL Database"
   description="时间点还原, Microsoft Azure SQL Database, 还原数据库, 恢复数据库, Azure 管理门户, Azure 门户"
   services="sql-database"
   documentationCenter=""
   authors="elfisher"
   manager="jeffreyg"
   editor=""/>

<tags
   ms.service="sql-database"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="storage-backup-recovery"
   ms.date="04/13/2015"
   ms.author="elfish"/>

# 确认已恢复的 Azure SQL Database

> [AZURE.SELECTOR]
- [时间点还原 - 门户](sql-database-point-in-time-restore-tutorial-management-portal.md)
- [时间点还原 - PowerShell](sql-database-point-in-time-restore-tutorial-powershell.md)
- [时间点还原 - REST API](sql-database-point-in-time-restore-tutorial-rest.md)
- [地域还原 - 门户](sql-database-geo-restore-tutorial-management-portal.md)
- [地域还原 - PowerShell](sql-database-geo-restore-tutorial-powershell.md)
- [地域还原 - REST API](sql-database-geo-restore-tutorial-rest.md)

## 概述

在将最近恢复的 Azure SQL Database 部署回到生产环境之前，你需要浏览本文中提供的任务清单。此清单适用于通过地域复制故障转移、已删除的数据库还原、时间点还原或地域还原恢复的数据库。

## 更新连接字符串

验证应用程序的连接字符串指向最近恢复的数据库。如果存在以下情况之一，请更新你的连接字符串：

  + 已恢复的数据库使用的名称不同于源数据库名称
  + 已恢复的数据库所在的服务器不同于源服务器

有关更改连接字符串的详细信息，请参阅[以编程方式连接到 Azure SQL 数据库时的准则](https://msdn.microsoft.com/library/azure/ee336282.aspx)和[到 Azure SQL 数据库的连接：中心建议](sql-database-connect-central-recommendations.md)。
 
## 修改防火墙规则
在服务器级别和数据库级别验证防火墙规则，并确保已启用从客户端计算机或 Azure 到服务器以及最近恢复的数据库的连接。有关详细信息，请参阅 [Azure SQL Database 防火墙](https://msdn.microsoft.com/library/azure/ee621782.aspx)和[如何：配置防火墙设置 (Azure SQL Database)](https://msdn.microsoft.com/library/azure/jj553530.aspx)。

## 验证服务器登录名和数据库用户

验证应用程序使用的所有登录名是否在托管已恢复数据库的服务器上存在。重新创建缺少的登录名，并向其授予对已恢复数据库的适当权限。有关详细信息，请参阅[在 Azure SQL Database 中管理数据库和登录名](https://msdn.microsoft.com/library/azure/ee336235.aspx)。

验证已恢复数据库中的每个数据库用户是否与有效的服务器登录名关联。使用 ALTER USER 语句将用户映射到有效的服务器登录名。有关详细信息，请参阅 [ALTER USER](http://go.microsoft.com/fwlink/?LinkId=397486)。 


## 设置遥测警报

验证现有的警报规则设置是否映射到已恢复的数据库。如果存在以下情况之一，请更新设置：

  + 已恢复的数据库使用的名称不同于源数据库名称
  + 已恢复的数据库所在的服务器不同于源服务器

有关数据库警报规则的详细信息，请参阅[如何：在 Azure 中接收警报通知和管理警报规则](https://msdn.microsoft.com/library/azure/dn306638.aspx)。


## 启用审核

如果需要通过审核来访问数据库，你需要在恢复数据库后启用审核。如果客户端应用程序使用了 *.database.secure.windows.net 模式的安全连接字符串，则就充分表明需要审核。有关详细信息，请参阅 [SQL Database 审核入门](sql-database-auditing-get-started.md)。 

<!--HONumber=55-->