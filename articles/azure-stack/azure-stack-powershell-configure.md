---
title: Configure PowerShell for use with Azure Stack | Microsoft Docs
description: Learn how to configure PowerShell for Azure Stack.
services: azure-stack
documentationcenter: 
author: SnehaGunda
manager: byronr
editor: 
ms.assetid: 
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/16/2017
ms.author: sngun
ms.translationtype: HT
ms.sourcegitcommit: 94d1d4c243bede354ae3deba7fbf5da0652567cb
ms.openlocfilehash: 3c6fc6af0588ae2e796d50f2d7bfdafaea81e2a1
ms.contentlocale: zh-cn
ms.lasthandoff: 07/18/2017

---

# <a name="configure-powershell-for-use-with-azure-stack"></a>Configure PowerShell for use with Azure Stack 

This article describes the steps required to connect to an Azure Stack Development Kit instance by using PowerShell. After you connect, you can access the portal and deploy resources through PowerShell. You can use the steps described in this article either from the development kit, or from a Windows-based external client if you are connected through VPN.

This article has detailed instructions to configure PowerShell for Azure Stack. However, if you want to quickly install and configure PowerShell, you can use the script provided in the [Get up and running with PowerShell](azure-stack-powershell-configure-quickstart.md) topic. 

## <a name="prerequisites"></a>Prerequisites
* Install [Azure Stack-compatible Azure PowerShell modules](azure-stack-powershell-install.md).  
* Download the [tools required to work with Azure Stack](azure-stack-powershell-download.md).  

## <a name="import-the-connect-powershell-module"></a>Import the Connect PowerShell module

After you download the required tools, navigate to the downloaded folder and import the **Connect** PowerShell module. To import the Connect module, run the following command in an elevated PowerShell session:

```PowerShell
Set-ExecutionPolicy RemoteSigned
Import-Module .\Connect\AzureStack.Connect.psm1
```

## <a name="configure-the-powershell-environment"></a>Configure the PowerShell environment

To configure your Azure Stack environment, do the following:

1. Register an AzureRM environment that targets your Azure Stack instance by using one of the following cmdlets:

   * **Cloud administrative environment**

       ```PowerShell
       Add-AzureRMEnvironment `
         -Name "AzureStackAdmin" `
         -ArmEndpoint "https://adminmanagement.local.azurestack.external"
       ```

   * **User environment**

       ```PowerShell
       Add-AzureRMEnvironment `
         -Name "AzureStackUser" `
         -ArmEndpoint "https://management.local.azurestack.external" 
       ```
   
   After you've registered the AzureRM environment, you can use all the AzureRM cmdlets in your Azure Stack environment. The output of the previous cmdlet is shown in the following screenshot:

   ![Get environment details](media/azure-stack-powershell-configure/getenvdetails.png)

2. Set the GraphEndpointResourceId value by using one of the following cmdlets:
   
   * **Azure Active Directory (Azure AD)**
   
      * For the **cloud administrative environment**, use:
        ```powershell
        Set-AzureRmEnvironment `
          -Name "AzureStackAdmin" `
          -GraphAudience "https://graph.windows.net/"
        ```
        
      * For the **user environment**, use:
        ```powershell
        Set-AzureRmEnvironment `
          -Name "AzureStackUser" `
          -GraphAudience "https://graph.windows.net/"
        ```

   * **Active Directory Federation Services**
   
      * For the **cloud administrative environment**, use:
        ```powershell
        Set-AzureRmEnvironment `
          -Name "AzureStackAdmin" `
          -GraphAudience "https://graph.local.azurestack.external/" `
          -EnableAdfsAuthentication:$true
        ```
        
      * For the **user environment**, use:
        ```powershell
        Set-AzureRmEnvironment `
          -Name "AzureStackUser" `
          -GraphAudience "https://graph.local.azurestack.external/" `
          -EnableAdfsAuthentication:$true
        ```

3. Get the GUID value of the Active Directory tenant that is used to deploy Azure Stack. If your Azure Stack environment is deployed by using:  

   * **Azure Active Directory (Azure AD)**
   
      * To access the **cloud administrative environment**, use:
        ```PowerShell
        $TenantID = Get-AzsDirectoryTenantId `
          -AADTenantName "<myDirectoryTenantName>.onmicrosoft.com" `
          -EnvironmentName "AzureStackAdmin"
        ```

      * To access the **user environment**, use:
        ```PowerShell
        $TenantID = Get-AzsDirectoryTenantId `
          -AADTenantName "<myDirectoryTenantName>.onmicrosoft.com" `
          -EnvironmentName "AzureStackUser"
        ```

   * **Active Directory Federation Services**
   
      * To access the **cloud administrative environment**, use:
        ```PowerShell
        $TenantID = Get-AzsDirectoryTenantId `
          -ADFS `
          -EnvironmentName "AzureStackAdmin"
        ```

      * To access the **user environment**, use:
        ```PowerShell 
        $TenantID = Get-AzsDirectoryTenantId `
          -ADFS `
          -EnvironmentName "AzureStackUser" 
        ```

## <a name="sign-in-to-azure-stack"></a>Sign in to Azure Stack

Sign in to the Azure Stack environment by using one of the following two cmdlets:

   * To sign in to the **administrative portal**, use:
    
       ```PowerShell
       Login-AzureRmAccount `
         -EnvironmentName "AzureStackAdmin" `
         -TenantId $TenantID 
       ```

   * To sign in to the **user portal**, use:

       ```PowerShell
       Login-AzureRmAccount `
         -EnvironmentName "AzureStackUser" `
         -TenantId $TenantID 
       ```

## <a name="register-resource-providers"></a>Register resource providers

After you sign in to the administrator or user portal, you can issue operations against the registered resource providers. By default, all the foundational resource providers are registered in the Default Provider Subscription (the cloud administrator's subscription).

When you operate on a newly created user subscription, which doesn’t have any resources deployed through the portal, the resource providers aren't automatically registered. You should explicitly register the resource providers by using the following script:

```powershell

foreach($s in (Get-AzureRmSubscription)) {
        Select-AzureRmSubscription -SubscriptionId $s.SubscriptionId | Out-Null
        Write-Progress $($s.SubscriptionId + " : " + $s.SubscriptionName)
Get-AzureRmResourceProvider -ListAvailable | Register-AzureRmResourceProvider -Force
    } 
```


## <a name="next-steps"></a>Next steps
* [Develop templates for Azure Stack](azure-stack-develop-templates.md)
* [Deploy templates with PowerShell](azure-stack-deploy-template-powershell.md)

