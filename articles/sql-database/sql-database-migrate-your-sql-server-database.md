---
title: "将 SQL Server DB 迁移到 Azure SQL 数据库 | Microsoft Docs"
description: "了解如何将 SQL Server 数据库迁移至 Azure SQL 数据库。"
services: sql-database
documentationcenter: 
author: janeng
manager: jhubbard
editor: 
tags: 
ms.assetid: 
ms.service: sql-database
ms.custom: mvc,load & move data
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: 
ms.date: 06/27/2017
ms.author: janeng
ms.translationtype: Human Translation
ms.sourcegitcommit: 3716c7699732ad31970778fdfa116f8aee3da70b
ms.openlocfilehash: 375d3ea0230e7d3fd0fc02ca7e0b8a7a76c24a27
ms.contentlocale: zh-cn
ms.lasthandoff: 06/30/2017


---

# <a name="migrate-your-sql-server-database-to-azure-sql-database"></a>将 SQL Server 数据库迁移至 Azure SQL 数据库

将 SQL Server 数据库移到 Azure SQL 数据库的过程由三个部分组成 - 首先准备、然后导出、最后导入数据库。 在本教程中，你将学习：

> [!div class="checklist"]
> * 使用[数据迁移助手](https://www.microsoft.com/download/details.aspx?id=53595) (DMA) 在 SQL Server 中准备要迁移到 Azure SQL 数据库的数据库
> * 将数据库导出到 BACPAC 文件
> * 将 BACPAC 文件导入 Azure SQL 数据库

如果还没有 Azure 订阅，可以在开始前[创建一个免费帐户](https://azure.microsoft.com/free/)。

## <a name="prerequisites"></a>先决条件

若要完成本教程，请确保已完成了以下先决条件：

- 安装最新版本的 [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) (SSMS)。 安装 SSMS 就会安装最新版本的 SQLPackage，这是一个可用于自动执行一系列数据库开发任务的命令行实用工具。 
- 安装 [Data Migration Assistant](https://www.microsoft.com/download/details.aspx?id=53595) (DMA)。
- 已被识别并有权访问要迁移的数据库。 本教程在 SQL Server 2008R2 或更高版本的实例上使用 [SQL Server 2008R2 AdventureWorks OLTP 数据库](https://msftdbprodsamples.codeplex.com/releases/view/59211)，但你可以使用你选择的任何数据库。 若要解决兼容性问题，请使用 [SQL Server Data Tools](https://docs.microsoft.com/sql/ssdt/download-sql-server-data-tools-ssdt)

## <a name="prepare-for-migration"></a>准备迁移

你已做好准备进行迁移。 按照以下步骤使用 **[Data Migration Assistant](https://www.microsoft.com/download/details.aspx?id=53595)** 来评估你的数据库迁移到 Azure SQL 数据库的准备情况。

1. 打开 **Data Migration Assistant**。 可以在任何连接到包含计划迁移的数据库的 SQL Server 实例的计算机上运行 DMA，且不需要将其安装在托管 SQL Server 实例的计算机上。

     ![打开 Data Migration Assistant](./media/sql-database-migrate-your-sql-server-database/data-migration-assistant-open.png)

2. 在左侧菜单中，单击“+ 新建”以创建**评估**项目。 将**项目名称**填入表单（其他所有值都应保留为默认值），然后单击“创建”。 “选项”页随即打开。

     ![新 Data Migration Assistant 项目](./media/sql-database-migrate-your-sql-server-database/data-migration-assistant-new-project.png)

3. 在“选项”页上，单击“下一步”。 “选择源”页面随即打开。

     ![新数据迁移选项](./media/sql-database-migrate-your-sql-server-database/data-migration-assistant-options.png) 

4. 在“选择源”页面上，输入包含要迁移的服务器的 SQL Server 实例的名称。 如有必要，请更改此页上的其他值。 单击“连接”。

     ![新数据迁移选择源](./media/sql-database-migrate-your-sql-server-database/data-migration-assistant-sources.png)

5. 在“选择源”页面的“添加源”部分，选择要测试兼容性的数据库的复选框。 单击 **“添加”**。

     ![新数据迁移选择源](./media/sql-database-migrate-your-sql-server-database/data-migration-assistant-sources-add.png)

6. 单击“启动评估”。

     ![新数据迁移开始评估](./media/sql-database-migrate-your-sql-server-database/data-migration-assistant-start-assessment.png)

7. 完成评估后，首先确认数据库是否具有迁移所需的足够兼容性。 查找具有绿圈的复选标记。

     ![新数据迁移评估结果兼容](./media/sql-database-migrate-your-sql-server-database/data-migration-assistant-assessment-results-compatible.png)

8. 查看结果。 显示的 SQL Server 功能奇偶校验结果是默认查看结果。 特别查看有关不受支持和部分支持的功能的信息和有关推荐操作的信息。 

     ![新数据迁移评估奇偶校验](./media/sql-database-migrate-your-sql-server-database/data-migration-assistant-assessment-results-parity.png)

9. 单击左上角的选项，查看兼容性问题。 特别查看有关每个兼容级别的迁移阻止程序、行为更改和已弃用功能的信息。 对于 AdventureWorks2008R2 数据库，请查看自 SQL Server 2008 以来对全文搜索的更改以及自 SQL Server 2000 以来对 SERVERPROPERTY('LCID') 的更改。 有关这些更改的详细信息，已提供链接。 已更改全文搜索的许多搜索选项和设置 

   > [!IMPORTANT] 
   > 将数据库迁移到 Azure SQL 数据库后，可选择以当前兼容级别（对于 AdventureWorks2008R2 数据库是级别 100）或更高级别操作数据库。 有关在特定兼容级别操作数据库的影响和选项的详细信息，请参阅 [ALTER DATABASE Compatibility Level](https://docs.microsoft.com/sql/t-sql/statements/alter-database-transact-sql-compatibility-level)（更改数据库兼容级别）。 有关与兼容级别相关的其他数据库级别设置的信息，另请参阅 [ALTER DATABASE SCOPED CONFIGURATION](https://docs.microsoft.com/sql/t-sql/statements/alter-database-scoped-configuration-transact-sql)（更改数据库范围的配置）。
   >

10. 或者，单击“导出报表”将报表另存为 JSON 文件。
11. 关闭 Data Migration Assistant。

## <a name="export-to-bacpac-file"></a>导出到 BACPAC 文件 

BACPAC 文件是扩展名为 BACPAC 的 ZIP 文件，其中包含来自 SQL Server 数据库的元数据和数据。 BACPAC 文件可以存储在 Azure Blob 存储或本地存储中，以进行存档或迁移（例如从 SQL Server 到 Azure SQL 数据库的迁移）。 若要使导出在事务上保持一致，必须确保在导出期间不会发生任何写入活动。

请按照以下步骤使用 SQLPackage 命令行实用工具将 AdventureWorks2008R2 数据库导出到本地存储。

1. 打开 Windows 命令提示符并将目录更改为包含 **130**  版本的 SQLPackage 的文件夹，例如 **C:\Program Files (x86)\Microsoft SQL Server\130\DAC\bin**。

2. 在命令提示符处执行以下 SQLPackage 命令，将 **AdventureWorks2008R2** 数据库从 **localhost** 导出到  **AdventureWorks2008R2.bacpac**。 根据你的环境更改这些值。

    ```SQLPackage
    sqlpackage.exe /Action:Export /ssn:localhost /sdn:AdventureWorks2008R2 /tf:AdventureWorks2008R2.bacpac
    ```

    ![导出 sqlpackage](./media/sql-database-migrate-your-sql-server-database/sqlpackage-export.png)

执行完成后，生成的 BACPAC 文件将存储在 sqlpackage 可执行文件所在的目录中。 在此示例中为 C:\Program Files (x86)\Microsoft SQL Server\130\DAC\bin。 

## <a name="log-in-to-the-azure-portal"></a>登录到 Azure 门户

登录到 [Azure 门户](https://portal.azure.com/)。 从运行 SQLPackage 命令行实用工具的计算机登录有助于步骤 5 中的防火墙规则创建。

## <a name="create-a-sql-server-logical-server"></a>创建 SQL Server 逻辑服务器

[SQL Server 逻辑服务器](sql-database-features.md)充当多个数据库的中心管理点。 按照以下步骤创建 SQL Server 逻辑服务器以包含已迁移的 AdventureWorks OLTP SQL Server 数据库。 

1. 单击 Azure 门户左上角的“新建”按钮。

2. 在“新建”页面的搜索窗口中键入“sql server”，然后从筛选列表中选择“SQL 数据库(逻辑服务器)”。

    ![选择逻辑服务器](./media/sql-database-migrate-your-sql-server-database/logical-server.png)

3. 在“所有内容”页面上，单击“SQL Server (逻辑服务器)”，然后在“SQL Server (逻辑服务器)”页面上单击“创建”。

    ![创建逻辑服务器](./media/sql-database-migrate-your-sql-server-database/logical-server-create.png)

4. 如上图所示，在“SQL Server (逻辑服务器)”窗体中填写以下信息：     

   | 设置       | 建议的值 | 说明 | 
   | ------------ | ------------------ | ------------------------------------------------- | 
   | **服务器名称** | 输入任何全局唯一名称 | 如需有效的服务器名称，请参阅 [Naming rules and restrictions](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions)（命名规则和限制）。 | 
   | 服务器管理员登录名 | 输入任何有效的名称 | 如需有效的登录名，请参阅 [Database Identifiers](https://docs.microsoft.com/sql/relational-databases/databases/database-identifiers)（数据库标识符）。 |
   | **密码** | 输入任何有效的密码 | 密码必须至少有 8 个字符，且必须包含以下类别中的三个类别的字符：大写字符、小写字符、数字以及非字母数字字符。 |
   | **订阅** | 选择一个订阅 | 有关订阅的详细信息，请参阅[订阅](https://account.windowsazure.com/Subscriptions)。 |
   | **资源组** | 选择现有资源组或创建新资源组，如“myResourceGroup” |  如需有效的资源组名称，请参阅 [Naming rules and restrictions](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions)（命名规则和限制）。 |
   | **位置** | 输入新服务器的任何有效位置 | 有关区域的信息，请参阅 [Azure 区域](https://azure.microsoft.com/regions/)。 |

   ![创建逻辑服务器完成窗体](./media/sql-database-migrate-your-sql-server-database/logical-server-create-completed.png)

5. 单击“创建”以预配逻辑服务器。 预配需要数分钟。 

> [!IMPORTANT]
> 请记住服务器名称、服务器管理员登录名和密码。 在本教程后面的步骤中，你需要用到这些值。
>

## <a name="create-a-server-level-firewall-rule"></a>创建服务器级防火墙规则

SQL 数据库服务[在服务器级别创建一个防火墙](sql-database-firewall-configure.md)。除非创建了防火墙规则来为特定的 IP 地址打开防火墙，否则会阻止外部应用程序和工具连接到服务器或服务器上的任何数据库。 按照以下步骤为运行 SQLPackage 命令行实用工具的计算机的 IP 地址创建 SQL 数据库服务器级防火墙规则。 这使 SQLPackage 能够通过 Azure SQL 数据库防火墙连接到 SQL Server 数据库逻辑服务器。 

1. 单击左侧菜单中的“所有资源”，然后在“所有资源”页面上单击新服务器。 服务器的“概述”页面随即打开，其中提供了用于进一步配置的选项。

     ![逻辑服务器概述](./media/sql-database-migrate-your-sql-server-database/logical-server-overview.png)

2. 在概述页面的“设置”下面，单击左侧菜单中的“防火墙”。 此时会打开 SQL 数据库服务器的“防火墙设置”页。 

3. 单击工具栏上的“添加客户端 IP”以添加当前使用的计算机的 IP 地址，然后单击“保存”。 此时会针对此 IP 地址创建服务器级防火墙规则。

     ![设置服务器防火墙规则](./media/sql-database-migrate-your-sql-server-database/server-firewall-rule-set.png)

4. 单击 **“确定”**。

现在可以使用之前创建的服务器管理员帐户通过 SQL Server Management Studio 或其他所选工具从此 IP 地址连接到此服务器上的所有数据库。

> [!NOTE]
> 通过端口 1433 进行的 SQL 数据库通信。 如果尝试从企业网络内部进行连接，则该网络的防火墙可能不允许经端口 1433 的出站流量。 如果是这样，则无法连接到 Azure SQL 数据库服务器，除非 IT 部门打开了端口 1433。
>

## <a name="import-a-bacpac-file-to-azure-sql-database"></a>将 BACPAC 文件导入 Azure SQL 数据库 

SQLPackage 命令行实用工具的最新版本支持在指定[服务层和性能级别](sql-database-service-tiers.md)创建 Azure SQL 数据库。 为了在导入过程中获得最佳性能，请选择一个较高的服务层和性能级别，然后在导入后降低级别（如果此服务层和性能级别高于当前所需级别）。

请按照以下步骤使用 SQLPackage 命令行实用工具将 AdventureWorks2008R2 数据库导入 Azure SQL 数据库。 虽然可使用 SQL Server Management Studio 执行此任务，但在大多数生产环境中，SQLPackage 是首选方法，因为它有最大的灵活性和最佳性能。 请参阅 [Migrating from SQL Server to Azure SQL Database using BACPAC Files](https://blogs.msdn.microsoft.com/sqlcat/2016/10/20/migrating-from-sql-server-to-azure-sql-database-using-bacpac-files/)（使用 BACPAC 文件从 SQL Server 迁移到 Azure SQL 数据库）。

- 在命令提示符下执行以下 SQLPackage 命令，将“AdventureWorks2008R2”数据库从本地存储导入先前创建到新数据库的 SQL 服务器逻辑服务器（服务层为“高级”、服务目标为“P6”）。 将尖括号中的值替换为 SQL Server 逻辑服务器的相应值，并指定新数据库的名称（也替换尖括号）。 还可根据环境选择更改数据库版本和服务目标的值。 就本教程而言，迁移的数据库称为“myMigratedDatabase”。

    ```
    SqlPackage.exe /a:import /tcs:"Data Source=<your_server_name>.database.windows.net;Initial Catalog=<your_new_database_name>;User Id=<change_to_your_admin_user_account>;Password=<change_to_your_password>" /sf:AdventureWorks2008R2.bacpac /p:DatabaseEdition=Premium /p:DatabaseServiceObjective=P6
    ```

   ![sqlpackage 导入](./media/sql-database-migrate-your-sql-server-database/sqlpackage-import.png)

> [!IMPORTANT]
> SQL Server 逻辑服务器侦听端口 1433。 如果尝试在企业防火墙内连接到 SQL Server 数据库逻辑服务器，则必须在企业防火墙中打开此端口，否则无法成功进行连接。
>

## <a name="connect-using-sql-server-management-studio-ssms"></a>使用 SQL Server Management Studio (SSMS) 连接

使用 SQL Server Management Studio 建立到 Azure SQL 数据库服务器和新迁移的数据库的连接，本教程中数据库名为“myMigratedDatabase”。 如果你不在运行 SQLPackage 的计算机上运行 SQL Server Management Studio，请使用前面过程中的步骤为此计算机创建防火墙规则。

1. 打开 SQL Server Management Studio。

2. 在“连接到服务器”对话框中，输入以下信息：
   - **服务器类型**：指定数据库引擎
   - **服务器名称**：输入完全限定的服务器名称，例如 **mynewserver20170403.database.windows.net**
   - **身份验证**：指定 SQL Server 身份验证
   - **登录名**：输入服务器管理员帐户
   - **密码**：输入服务器管理员帐户的密码
 
    ![使用 ssms 进行连接](./media/sql-database-migrate-your-sql-server-database/connect-ssms.png)

3. 单击“连接”。 此时会打开“对象资源管理器”窗口。 

4. 在对象资源管理器中展开“数据库”，然后展开 **myMigratedDatabase**，查看示例数据库中的对象。

## <a name="change-database-properties"></a>更改数据库属性

可以使用 SQL Server Management Studio 更改服务层、性能级别和兼容级别。 在导入阶段中，建议你导入到更高性能层数据库，以便获得最佳性能。但在导入后，可降低级别以节省资金，直到准备好积极使用导入的数据库为止。 更改兼容性级别可能有助于提高性能和访问 Azure SQL 数据库服务的最新功能。 迁移较旧的数据库时，其数据库兼容性级别将保持在与要导入的数据库兼容的最低支持级别。 有关详细信息，请参阅[在 Azure SQL 数据库中使用兼容性级别 130 改进查询性能](sql-database-compatibility-level-query-performance-130.md)。

1. 在“对象资源管理器”中，右键单击“myMigratedDatabase”**，然后单击“新建查询”**。 此时会打开一个连接到数据库的查询窗口。

2. 执行以下命令将服务层设置为“标准”，将性能级别设置为“S1”。

    ```
    ALTER DATABASE myMigratedDatabase 
    MODIFY 
        (
        EDITION = 'Standard'
        , MAXSIZE = 250 GB
        , SERVICE_OBJECTIVE = 'S1'
    );
    ```

    ![更改服务层](./media/sql-database-migrate-your-sql-server-database/service-tier.png)

3. 执行以下命令将数据库兼容级别更改为 **130**。

    ```
    ALTER DATABASE myMigratedDatabase  
    SET COMPATIBILITY_LEVEL = 130;
    ```

   ![更改兼容级别](./media/sql-database-migrate-your-sql-server-database/compat-level.png)

## <a name="next-steps"></a>后续步骤 
本教程已介绍如何准备、导出和导入数据库。 你已了解：

> [!div class="checklist"]
> * 在 SQL Server 中准备要迁移到 Azure SQL 数据库的数据库
> * 将数据库导出到 BACPAC 文件
> * 将 BACPAC 文件导入 Azure SQL 数据库

请转到下一教程，了解如何保护数据库。

> [!div class="nextstepaction"]
> [保护 Azure SQL 数据库](sql-database-security-tutorial.md)。



