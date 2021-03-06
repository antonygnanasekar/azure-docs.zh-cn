---
title: Deploy a virtual machine with a securely stored certificate on Azure Stack | Microsoft Docs
description: Learn how to deploy a virtual machine and push a certificate onto it by using a key vault in Azure Stack
services: azure-stack
documentationcenter: 
author: SnehaGunda
manager: byronr
editor: 
ms.assetid: 46590eb1-1746-4ecf-a9e5-41609fde8e89
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 08/03/2017
ms.author: sngun
ms.translationtype: HT
ms.sourcegitcommit: 80fd9ee9b9de5c7547b9f840ac78a60d52153a5a
ms.openlocfilehash: 11be2ac2a06f5a8dacf701d8c0112d3274eb95cb
ms.contentlocale: zh-cn
ms.lasthandoff: 08/14/2017

---
# <a name="create-a-virtual-machine-and-include-certificate-retrieved-from-a-key-vault"></a>Create a virtual machine and include certificate retrieved from a key vault

This article helps you to create a virtual machine in Azure Stack and push certificates onto it. 

## <a name="prerequisites"></a>Prerequisites

* Azure Stack cloud administrators must have [created an offer](azure-stack-create-offer.md) that includes the Azure Key Vault service.  
* Users must [subscribe to an offer](azure-stack-subscribe-plan-provision-vm.md) that includes the Key Vault service.  
* [Install PowerShell for Azure Stack.](azure-stack-powershell-install.md)  
* [Configure PowerShell for use with Azure Stack.](azure-stack-powershell-configure.md)

A key vault in Azure Stack is used to store certificates. Certificates are helpful in many different scenarios. For example, consider a scenario where you have a virtual machine in Azure Stack that is running an application that needs a certificate. This certificate can be used for encrypting, for authenticating to Active Directory, or for SSL on a website. Having the certificate in a key vault helps make sure that it's secure.

In this article, we walk you through the steps required to push a certificate onto a Windows virtual machine in Azure Stack. You can use these steps either from the Azure Stack Development Kit, or from a Windows-based external client if you are connected through VPN.

The following steps describe the process required to push a certificate onto the virtual machine:

1. Create a Key Vault secret.
2. Update the azuredeploy.parameters.json file.
3. Deploy the template

## <a name="create-a-key-vault-secret"></a>Create a Key Vault secret

The following script creates a certificate in the .pfx format, creates a key vault, and stores the certificate in the key vault as a secret. You must use the `-EnabledForDeployment` parameter when you're creating the key vault. This parameter makes sure that the key vault can be referenced from Azure Resource Manager templates.

```powershell

# Create a certificate in the .pfx format
New-SelfSignedCertificate `
  -certstorelocation cert:\LocalMachine\My `
  -dnsname contoso.microsoft.com

$pwd = ConvertTo-SecureString `
  -String "<Password used to export the certificate>" `
  -Force `
  -AsPlainText

Export-PfxCertificate `
  -cert "cert:\localMachine\my\<Your certificate Thumbprint>" `
  -FilePath "<Fully qualified path to the certificate>" `
  -Password $pwd

# Create a key vault and upload the certificate into the key vault as a secret
$vaultName = "contosovault"
$resourceGroup = "contosovaultrg"
$location = "local"
$secretName = "servicecert"
$fileName = "<Fully qualified path to the certificate>"
$certPassword = "<Password used to export the certificate>"

$fileContentBytes = get-content $fileName `
  -Encoding Byte

$fileContentEncoded = [System.Convert]::ToBase64String($fileContentBytes)
$jsonObject = @"
{
"data": "$filecontentencoded",
"dataType" :"pfx",
"password": "$certPassword"
}
"@
$jsonObjectBytes = [System.Text.Encoding]::UTF8.GetBytes($jsonObject)
$jsonEncoded = [System.Convert]::ToBase64String($jsonObjectBytes)

New-AzureRmResourceGroup `
  -Name $resourceGroup `
  -Location $location

New-AzureRmKeyVault `
  -VaultName $vaultName `
  -ResourceGroupName $resourceGroup `
  -Location $location `
  -sku standard `
  -EnabledForDeployment

$secret = ConvertTo-SecureString `
  -String $jsonEncoded `
  -AsPlainText -Force

Set-AzureKeyVaultSecret `
  -VaultName $vaultName `
  -Name $secretName `
   -SecretValue $secret

```

When you run the previous script, the output includes the secret URI. Make a note of this URI. You have to reference it in the [Push certificate to Windows Resource Manager template](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-push-certificate-windows). Download the [vm-push-certificate-windows template](https://github.com/Azure/azure-quickstart-templates/tree/master/201-vm-push-certificate-windows) folder onto your development computer. This folder contains the `azuredeploy.json` and `azuredeploy.parameters.json` files, which you will need in the next steps.

Modify the `azuredeploy.parameters.json` file according to your environment values. The parameters of special interest are the vault name, the vault resource group, and the secret URI (as generated by the previous script). The following file is an example of a parameter file:

## <a name="update-the-azuredeployparametersjson-file"></a>Update the azuredeploy.parameters.json file

Update the azuredeploy.parameters.json file with the vaultName, secret URI, VmName, and other values as per your environment. The following JSON file shows an example of the template parameters file: 

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "newStorageAccountName": {
      "value": "kvstorage01"
    },
    "vmName": {
      "value": "VM1"
    },
    "vmSize": {
      "value": "Standard_D1_v2"
    },
    "adminUserName": {
      "value": "demouser"
    },
    "adminPassword": {
      "value": "demouser@123"
    },
    "vaultName": {
      "value": "contosovault"
    },
    "vaultResourceGroup": {
      "value": "contosovaultrg"
    },
    "secretUrlWithVersion": {
      "value": "https://testkv001.vault.local.azurestack.external/secrets/testcert002/82afeeb84f4442329ce06593502e7840"
    }
  }
}
```

## <a name="deploy-the-template"></a>Deploy the template

Now deploy the template by using the following PowerShell script:

```powershell
# Deploy a Resource Manager template to create a VM and push the secret onto it
New-AzureRmResourceGroupDeployment `
  -Name KVDeployment `
  -ResourceGroupName $resourceGroup `
  -TemplateFile "<Fully qualified path to the azuredeploy.json file>" `
  -TemplateParameterFile "<Fully qualified path to the azuredeploy.parameters.json file>"
```

When the template is deployed successfully, it results in the following output:

![Deployment output](media\azure-stack-kv-push-secret-into-vm/deployment-output.png)

When this virtual machine is deployed, Azure Stack pushes the certificate onto the virtual machine. In Windows, the certificate is added to the LocalMachine certificate location, with the certificate store that the user provided. In Linux, the certificate is placed under the /var/lib/waagent directory, with the file name &lt;UppercaseThumbprint&gt;.crt for the X509 certificate file and &lt;UppercaseThumbprint&gt;.prv for the private key.

## <a name="retire-certificates"></a>Retire certificates

In the preceding section, we showed you how to push a new certificate onto a virtual machine. Your old certificate is still on the virtual machine, and it can't be removed. However, you can disable the older version of the secret by using the `Set-AzureKeyVaultSecretAttribute` cmdlet. The following is an example usage of this cmdlet. Make sure to replace the vault name, secret name, and version values according to your environment:

```powershell
Set-AzureKeyVaultSecretAttribute -VaultName contosovault -Name servicecert -Version e3391a126b65414f93f6f9806743a1f7 -Enable 0
```

## <a name="next-steps"></a>Next steps

* [Deploy a VM with a Key Vault password](azure-stack-kv-deploy-vm-with-secret.md)
* [Allow an application to access Key Vault](azure-stack-kv-sample-app.md)



