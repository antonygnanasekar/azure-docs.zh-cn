---
title: "创建第一个 Azure Resource Manager 模板 | Microsoft Docs"
description: "分步指南：创建第一个 Azure Resource Manager 模板。 本指南介绍如何使用存储帐户的模板参考来创建模板。"
services: azure-resource-manager
documentationcenter: 
author: tfitzmac
manager: timlt
editor: tysonn
ms.service: azure-resource-manager
ms.workload: multiple
ms.tgt_pltfrm: na
ms.devlang: na
ms.date: 07/27/2017
ms.topic: get-started-article
ms.author: tomfitz
ms.translationtype: HT
ms.sourcegitcommit: 6e76ac40e9da2754de1d1aa50af3cd4e04c067fe
ms.openlocfilehash: 49086b51e2db1aebed45746306ae14b6f1feb631
ms.contentlocale: zh-cn
ms.lasthandoff: 07/31/2017

---

# <a name="create-and-deploy-your-first-azure-resource-manager-template"></a>创建和部署第一个 Azure 资源管理器模板
本主题介绍如何通过相关步骤创建第一个 Azure Resource Manager 模板。 Resource Manager 模板为 JSON 文件，用于定义针对解决方案进行部署时所需的资源。 若要了解与部署和管理 Azure 解决方案相关联的概念，请参阅 [Azure Resource Manager 概述](resource-group-overview.md)。 如果有现成的资源，需要为这些资源获取模板，请参阅[从现有资源导出 Azure Resource Manager 模板](resource-manager-export-template.md)。

若要创建和修改模板，需要 JSON 编辑器。 [Visual Studio Code](https://code.visualstudio.com/) 是轻量型开源跨平台代码编辑器。 强烈建议使用 Visual Studio Code 来创建资源管理器模板。 本主题假定你使用 VS Code；但是，如果有其他 JSON 编辑器（例如 Visual Studio），可以使用该编辑器。

## <a name="prerequisites"></a>先决条件

* Visual Studio Code。 根据需要从 [https://code.visualstudio.com/](https://code.visualstudio.com/) 安装。
* Azure 订阅。 如果还没有 Azure 订阅，可以在开始前创建一个 [免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。

## <a name="create-template"></a>创建模板

一开始请使用简单的模板将存储帐户部署到订阅。

1. 选择“文件” > “新建文件”。 

   ![新建文件](./media/resource-manager-create-first-template/new-file.png)

2. 将以下 JSON 语法复制并粘贴到文件中：

   ```json
   {
     "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
     "contentVersion": "1.0.0.0",
     "parameters": {
     },
     "variables": {
     },
     "resources": [
       {
         "name": "[concat('storage', uniqueString(resourceGroup().id))]",
         "type": "Microsoft.Storage/storageAccounts",
         "apiVersion": "2016-01-01",
         "sku": {
           "name": "Standard_LRS"
         },
         "kind": "Storage",
         "location": "South Central US",
         "tags": {},
         "properties": {}
       }
     ],
     "outputs": {  }
   }
   ```

   存储帐户名称有多种限制，因此难以设置。 该名称必须为 3 到 24 个字符，只能使用数字和小写字母，而且必需唯一。 前述模板使用 [uniqueString](resource-group-template-functions-string.md#uniquestring) 函数生成哈希值。 此模板通过添加前缀 storage 扩展了该哈希值增的含义。 

3. 在本地文件夹中将该文件另存为 azuredeploy.json。

   ![保存模板](./media/resource-manager-create-first-template/save-template.png)

## <a name="deploy-template"></a>部署模板

已做好部署此模板的准备。 请使用 PowerShell 或 Azure CLI 创建一个资源组。 然后，将存储帐户部署到该资源组。

* 对于 PowerShell，请在包含模板的文件夹中使用以下命令：

   ```powershell
   Login-AzureRmAccount
   
   New-AzureRmResourceGroup -Name examplegroup -Location "South Central US"
   New-AzureRmResourceGroupDeployment -ResourceGroupName examplegroup -TemplateFile azuredeploy.json
   ```

* 若要在本地安装 Azure CLI，请在包含模板的文件夹中使用以下命令：

   ```azurecli
   az login

   az group create --name examplegroup --location "South Central US"
   az group deployment create --resource-group examplegroup --template-file azuredeploy.json
   ```

部署完成后，存储帐户就会存在于资源组中。

## <a name="deploy-template-from-cloud-shell"></a>从 Cloud Shell 部署模板

可以使用 [Cloud Shell](../cloud-shell/overview.md) 来运行 Azure CLI 命令，以便部署模板。 但是，必须先将模板加载到 Cloud Shell 的文件共享。 如果尚未使用过 Cloud Shell，请参阅 [Azure Cloud Shell 概述](../cloud-shell/overview.md)，了解如何设置它。

1. 登录到 [Azure 门户](https://portal.azure.com)。   

2. 选择 Cloud Shell 资源组。 名称模式为 `cloud-shell-storage-<region>`。

   ![选择资源组](./media/resource-manager-create-first-template/select-cs-resource-group.png)

3. 选择适用于 Cloud Shell 的存储帐户。

   ![选择存储帐户](./media/resource-manager-create-first-template/select-storage.png)

4. 选择“文件”。

   ![选择文件](./media/resource-manager-create-first-template/select-files.png)

5. 选择 Cloud Shell 的文件共享。 名称模式为 `cs-<user>-<domain>-com-<uniqueGuid>`。

   ![选择文件共享](./media/resource-manager-create-first-template/select-file-share.png)

6. 选择“添加目录”。

   ![添加目录](./media/resource-manager-create-first-template/select-add-directory.png)

7. 将其命名为“模板”，然后选择“确定”。

   ![为目录命名](./media/resource-manager-create-first-template/name-templates.png)

8. 选择新目录。

   ![选择目录](./media/resource-manager-create-first-template/select-templates.png)

9. 选择**上传**。

   ![选择“上传”](./media/resource-manager-create-first-template/select-upload.png)

10. 找到并上传模板。

   ![上传文件](./media/resource-manager-create-first-template/upload-files.png)

11. 打开提示符。

   ![打开 Cloud Shell](./media/resource-manager-create-first-template/start-cloud-shell.png)

12. 在 Cloud Shell 中输入以下命令：

   ```azurecli
   az group create --name examplegroup --location "South Central US"
   az group deployment create --resource-group examplegroup --template-file clouddrive/templates/azuredeploy.json
   ```

部署完成后，存储帐户就会存在于资源组中。

## <a name="customize-the-template"></a>自定义模板

该模板可以正常使用，但不灵活。 它始终将本地冗余存储部署到美国中南部。 名称始终为存储后跟哈希值。 若要允许将模板用于不同的方案，请向模板添加参数。

以下示例显示带两个参数的 parameters 节。 第一个参数 `storageSKU` 用于指定冗余类型。 它将可以传入的值限制为适用于存储帐户的值。 它还指定默认值。 第二个参数 `storageNamePrefix` 设置为最多允许 11 个字符。 它指定默认值。

```json
"parameters": {
  "storageSKU": {
    "type": "string",
    "allowedValues": [
      "Standard_LRS",
      "Standard_ZRS",
      "Standard_GRS",
      "Standard_RAGRS",
      "Premium_LRS"
    ],
    "defaultValue": "Standard_LRS",
    "metadata": {
      "description": "The type of replication to use for the storage account."
    }
  },
  "storageNamePrefix": {
    "type": "string",
    "maxLength": 11,
    "defaultValue": "storage",
    "metadata": {
      "description": "The value to use for starting the storage account name. Use only lowercase letters and numbers."
    }
  }
},
```

在 variables 节，请添加名为 `storageName` 的变量。 它组合了来自变量的前缀值和来自 [uniqueString](resource-group-template-functions-string.md#uniquestring) 函数的哈希值。 它使用 [toLower](resource-group-template-functions-string.md#tolower) 函数将所有字符转换为小写。

```json
"variables": {
  "storageName": "[concat(toLower(parameters('storageNamePrefix')), uniqueString(resourceGroup().id))]"
},
```

若要将这些新值用于存储帐户，请更改资源定义：

```json
"resources": [
  {
    "name": "[variables('storageName')]",
    "type": "Microsoft.Storage/storageAccounts",
    "apiVersion": "2016-01-01",
    "sku": {
      "name": "[parameters('storageSKU')]"
    },
    "kind": "Storage",
    "location": "[resourceGroup().location]",
    "tags": {},
    "properties": {}
  }
],
```

请注意，存储帐户的名称现在设置为已添加的变量。 SKU 名称设置为参数的值。 位置设置为资源组所在的位置。

保存文件。 

完成本文中的步骤以后，模板将如下所示：

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "storageSKU": {
      "type": "string",
      "allowedValues": [
        "Standard_LRS",
        "Standard_ZRS",
        "Standard_GRS",
        "Standard_RAGRS",
        "Premium_LRS"
      ],
      "defaultValue": "Standard_LRS",
      "metadata": {
        "description": "The type of replication to use for the storage account."
      }
    },   
    "storageNamePrefix": {
      "type": "string",
      "maxLength": 11,
      "defaultValue": "storage",
      "metadata": {
        "description": "The value to use for starting the storage account name. Use only lowercase letters and numbers."
      }
    }
  },
  "variables": {
    "storageName": "[concat(toLower(parameters('storageNamePrefix')), uniqueString(resourceGroup().id))]"
  },
  "resources": [
    {
      "name": "[variables('storageName')]",
      "type": "Microsoft.Storage/storageAccounts",
      "apiVersion": "2016-01-01",
      "sku": {
        "name": "[parameters('storageSKU')]"
      },
      "kind": "Storage",
      "location": "[resourceGroup().location]",
      "tags": {},
      "properties": {}
    }
  ],
  "outputs": {  }
}
```

## <a name="redeploy-template"></a>重新部署模板

请使用不同的值重新部署模板。

对于 PowerShell，请使用：

```powershell
New-AzureRmResourceGroupDeployment -ResourceGroupName examplegroup -TemplateFile azuredeploy.json -storageNamePrefix newstore -storageSKU Standard_RAGRS
```

对于 Azure CLI，请使用：

```azurecli
az group deployment create --resource-group examplegroup --template-file azuredeploy.json --parameters storageSKU=Standard_RAGRS storageNamePrefix=newstore
```

对于 Cloud Shell，请将更改的模板上传到文件共享。 覆盖现有文件。 然后，使用以下命令：

```azurecli
az group deployment create --resource-group examplegroup --template-file clouddrive/templates/azuredeploy.json --parameters storageSKU=Standard_RAGRS storageNamePrefix=newstore
```

## <a name="clean-up-resources"></a>清理资源

不再需要时，请通过删除资源组来清理部署的资源。

对于 PowerShell，请使用：

```powershell
Remove-AzureRmResourceGroup -Name examplegroup
```

对于 Azure CLI，请使用：

```azurecli
az group delete --name examplegroup
```

## <a name="next-steps"></a>后续步骤
* 若要详细了解模板的结构，请参阅 [Authoring Azure Resource Manager templates](resource-group-authoring-templates.md)（创作 Azure Resource Manager 模板）。
* 若要了解存储帐户的属性，请查看[存储帐户模板参考](/azure/templates/microsoft.storage/storageaccounts)。
* 若要查看许多不同类型的解决方案的完整模型，请参阅 [Azure Quickstart Templates](https://azure.microsoft.com/documentation/templates/)（Azure 快速入门模板）。

