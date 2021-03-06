
# 概述
## [什么是 Azure 备份？](backup-introduction-to-azure-backup.md)

# 入门
## [备份 Azure VM](backup-azure-vms-first-look-arm.md)
## [备份 Windows Server 或 Windows 计算机](backup-try-azure-backup-in-10-mins.md)
## [备份 VMware 服务器](backup-azure-backup-server-vmware.md)

# 如何
## Azure VM
### 准备 VM
#### [准备 Resource Manager 部署的虚拟机](backup-azure-arm-vms-prepare.md)
#### [准备 Azure 虚拟机](backup-azure-vms-prepare.md)
### 规划环境
#### [规划 VM 备份基础结构](backup-azure-vms-introduction.md)
### 备份 VM
#### [将 Azure 虚拟机备份到恢复服务保管库](backup-azure-arm-vms.md)
#### [备份加密的虚拟机](backup-azure-vms-encryption.md)
#### [备份 Azure 虚拟机](backup-azure-vms.md)
### 管理和监视 VM
#### [管理 Azure 门户中的 Azure VM 备份](backup-azure-manage-vms.md)
#### [监视 Azure 门户中 Azure VM 备份警报](backup-azure-monitor-vms.md)
#### [管理和监视经典门户中的 Azure VM 备份](backup-azure-manage-vms-classic.md)
### 从 VM 还原数据
#### [从 Azure VM 备份恢复文件](backup-azure-restore-files-from-vm.md)
#### [在 Azure 门户中还原 Resource Manager 部署的 VM](backup-azure-arm-restore-vms.md)
#### [还原加密的虚拟机](backup-azure-vms-encryption.md)
#### [还原 Azure 中的虚拟机](backup-azure-restore-vms.md)
#### [还原加密 VM 的 Key Vault 密钥和机密](backup-azure-restore-key-secret.md)


## Windows Server
### [备份 Windows Server 文件和文件夹](backup-configure-vault.md)
### [备份 Windows Server 系统状态](backup-azure-system-state.md)
### [将文件从 Azure 恢复到 Windows Server](backup-azure-restore-windows-server.md)
### [还原 Windows Server 系统状态](backup-azure-restore-system-state.md)
### [监视和管理恢复服务保管库](backup-azure-manage-windows-server.md)
### 使用经典门户备份和还原
#### [使用经典部署模型备份 Windows Server](backup-configure-vault-classic.md)
#### [使用经典部署模型管理备份保管库](backup-azure-manage-windows-server-classic.md)
#### [使用经典部署模型将文件还原到 Windows Server](backup-azure-restore-windows-server-classic.md)

## Azure 备份服务器
### [Azure 备份服务器保护矩阵](backup-mabs-protection-matrix.md)
### 安装或升级
#### [在 Azure 门户中准备 Azure 备份服务器工作负荷](backup-azure-microsoft-azure-backup.md)
#### [在经典门户中准备 Azure 备份服务器工作负荷](backup-azure-microsoft-azure-backup-classic.md)
#### [将存储添加到 Azure 备份服务器](backup-mabs-add-storage.md)
#### [将 Azure 备份服务器升级到 v.2](backup-mabs-upgrade-to-v2.md)
#### [无人参与的 Azure 备份服务器安装](backup-mabs-unattended-install.md)
### 保护工作负荷
#### [使用 Azure 备份服务器备份 VMware 服务器](backup-azure-backup-server-vmware.md)
#### [使用 Azure 备份服务器备份 Exchange](backup-azure-exchange-mabs.md)
#### [使用 Azure 备份服务器备份 SharePoint 场](backup-azure-backup-sharepoint-mabs.md)
#### [使用 Azure 备份服务器备份 SQL](backup-azure-sql-mabs.md)
#### [保护系统状态和裸机恢复](backup-mabs-system-state-and-bmr.md)

## 数据保护管理器
### [在 Azure 门户中准备 DPM 工作负荷](backup-azure-dpm-introduction.md)
### [在经典 Azure 门户中准备 DPM 工作负荷](backup-azure-dpm-introduction-classic.md)
### [使用 System Center DPM 备份 Exchange 服务器](backup-azure-backup-exchange-server.md)
### [将数据恢复到备用 DPM 服务器](backup-azure-alternate-dpm-server.md)
### [使用 DPM 备份 SQL Server 工作负荷](backup-azure-backup-sql.md)
### [使用 DPM 备份 SharePoint 场](backup-azure-backup-sharepoint.md)

## Azure SQL 数据库
### [配置长期备份保留](../sql-database/sql-database-configure-long-term-retention.md?toc=%2fazure%2fbackup%2ftoc.json)
### [查看恢复服务保管库中的备份](../sql-database/sql-database-view-backups-in-vault.md?toc=%2fazure%2fbackup%2ftoc.json)
### [从长期备份保留存储中还原](../sql-database/sql-database-restore-from-long-term-retention.md?toc=%2fazure%2fbackup%2ftoc.json)
### [删除长期 Azure SQL 备份](../sql-database/sql-database-long-term-retention-delete.md?toc=%2fazure%2fbackup%2ftoc.json)

## 使用 PowerShell
### [Azure 门户中的 Azure VM](backup-azure-vms-automation.md)
### [经典门户中的 Azure VM](backup-azure-vms-classic-automation.md)
### [Azure 门户中的 DPM](backup-dpm-automation.md)
### [经典门户中的 DPM](backup-dpm-automation-classic.md)
### [Azure 门户中的 Windows Server](backup-client-automation.md)
### [经典门户中的 Windows Server](backup-client-automation-classic.md)

## 常见问题
### [恢复服务保管库常见问题解答](backup-azure-backup-faq.md)
### [Azure VM 备份常见问题解答](backup-azure-vm-backup-faq.md)
### [使用 Azure 备份代理进行文件-文件夹备份的常见问题解答](backup-azure-file-folder-backup-faq.md)

## 故障排除
### [在 Azure 门户中排查 Azure VM 备份问题](backup-azure-vms-troubleshoot.md)
### [在经典门户中排查 Azure VM 备份问题](backup-azure-vms-troubleshoot-classic.md)
### [Azure VM 备份失败：无法与 VM 代理通信以获取快照状态 - 快照 VM 子任务超时](backup-azure-troubleshoot-vm-backup-fails-snapshot-timeout.md)
### [在 Azure 备份中备份文件和文件夹时速度缓慢](backup-azure-troubleshoot-slow-backup-performance-issue.md)
### [Azure 备份服务器故障排除](backup-azure-mabs-troubleshoot.md)

# 概念
## [恢复服务保管库概述](backup-azure-recovery-services-vault-overview.md)
## [将备份保管库升级到恢复服务保管库](backup-azure-upgrade-backup-to-recovery-services.md)
## [删除恢复服务保管库](backup-azure-delete-vault.md)
## [基于角色的访问控制](backup-rbac-rs-vault.md)
## [混合备份的安全性](backup-azure-security-feature.md)
## [配置 Azure 备份报表](backup-azure-configure-reports.md)
## [Azure 备份报表的数据模型](backup-azure-reports-data-model.md)
## [Azure 备份的 Log Analytics 数据模型](backup-azure-log-analytics-data-model.md)
## [配置脱机备份](backup-azure-backup-import-export.md)
## [替换磁带库](backup-azure-backup-cloud-as-tape.md)
## [Linux VM 的应用程序一致性备份](backup-azure-linux-app-consistent.md)

# 引用
## [PowerShell](/powershell/module/azurerm.recoveryservices.backup)
## [.NET](/dotnet/api/microsoft.azure.management.recoveryservices.backup)

# 资源
## [Azure 路线图](https://azure.microsoft.com/roadmap/)
## [MSDN 论坛](https://social.msdn.microsoft.com/Forums/en-US/home?forum=windowsazureonlinebackup)
## [定价](https://azure.microsoft.com/pricing/details/backup/)
## [定价计算器](https://azure.microsoft.com/pricing/calculator/)
## [服务更新](https://azure.microsoft.com/updates/?product=backup)
## [视频](https://azure.microsoft.com/documentation/videos/index/?services=backup)
