---
title: "开发测试实验室概念 | Microsoft Docs"
description: "了解开发测试实验室的基本概念及其如何轻松地创建、管理和监视 Azure 虚拟机"
services: devtest-lab,virtual-machines
documentationcenter: na
author: tomarcher
manager: douge
editor: 
ms.assetid: 105919e8-3617-4ce3-a29f-a289fa608fb2
ms.service: devtest-lab
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/25/2016
ms.author: tarcher
ms.translationtype: Human Translation
ms.sourcegitcommit: e0c999b2bf1dd38d8a0c99c6cdd4976cc896dd99
ms.openlocfilehash: 1caea59e71126e934e2e52a1ad7f533ffa7d4b03
ms.contentlocale: zh-cn
ms.lasthandoff: 04/20/2017


---
# 开发测试实验室概念
<a id="devtest-labs-concepts" class="xliff"></a>
## 概述
<a id="overview" class="xliff"></a>
下表包含关键开发测试实验室概念和定义：

## 实验室
<a id="labs" class="xliff"></a>
实验室是包含一组资源（例如，虚拟机 (VM)）的基础结构，通过指定限制和配额可以更好地管理这些资源。

## 虚拟机
<a id="virtual-machine" class="xliff"></a>
Azure VM 是 Azure 提供的多种[可缩放按需分配计算资源](https://docs.microsoft.com/azure/app-service-web/choose-web-site-cloud-service-vm)之一。 无需购买和维护运行 Azure VM 的物理硬件，Azure VM 即可为你提供虚拟化灵活性，尽管你仍需要执行某些任务（如配置、修补和安装在 Azure VM 上运行的软件）来维护 VM。

[Azure 中的 Windows 虚拟机概述](https://docs.microsoft.com/azure/virtual-machines/virtual-machines-windows-overview)提供有关在创建 VM 之前应考虑的信息、如何创建 VM 以及如何管理 VM。

## 可认领 VM
<a id="claimable-vm" class="xliff"></a>
Azure 可认领 VM 是可供具备权限的任何实验室用户使用的虚拟机。 实验室管理员可准备具有特定基本映像和项目的 VM，并将其保存到共享池。 此后，在需要这种特定配置时，实验室用户可从池中认领一台有用的 VM。

可认领 VM 最初并未分配给任何特定用户，但会显示在“可认领虚拟机”下每个用户的列表中。 用户认领 VM 后，该 VM 会移动到“我的虚拟机”区域，并且任何其他用户都无法再对其进行认领。

## 环境
<a id="environment" class="xliff"></a>
在开发测试实验室中，环境是指实验室中 Azure 资源的集合。 [此博客文章](https://blogs.msdn.microsoft.com/devtestlab/2016/11/16/connect-2016-news-for-azure-devtest-labs-azure-resource-manager-template-based-environments-vm-auto-shutdown-and-more/)讨论如何基于 Azure Resource Manager 模板创建多 VM 环境。

## 基础映像
<a id="base-images" class="xliff"></a>
基础映像为 VM 映像，包含预安装和配置为快速创建 VM 的所有工具和设置。 可通过选取现有基并添加项目来对 VM 进行预配，进而安装测试代理。 然后可将预配的 VM 保存为基，这样无需为每个 VM 的预配重新安装测试代理即可使用该基。

## 项目
<a id="artifacts" class="xliff"></a>
项目用于在预配 VM 后部署和配置应用程序。 项目可以是：

* 想要在 VM 上安装的工具，例如，代理、Fiddler 和 Visual Studio。
* 想要在 VM 上运行的操作，例如，克隆存储库。
* 想要测试的应用程序。

项目是 [Azure Resource Manager](../azure-resource-manager/resource-group-overview.md) JSON 文件，包含执行部署和应用配置的说明。

## 项目存储库
<a id="artifact-repositories" class="xliff"></a>
项目存储库是其中已签入项目的 Git 存储库。 可将项目存储库添加到组织中的多个实验室，从而允许重用和共享。

## 公式
<a id="formulas" class="xliff"></a>
除基础映像外，还可使用公式提供用于快速预配 VM 的机制。 开发测试实验室中的公式是用于创建实验室 VM 的默认属性值的列表。
利用公式，可以创建具有相同属性（例如，基础映像、VM 大小、虚拟网络和项目）组的 VM，而无需每次都指定这些属性。 使用公式创建 VM 时，可以按原样使用默认值，也可以修改默认值。

## 策略
<a id="policies" class="xliff"></a>
策略有助于控制实验室中的成本。 例如，可以创建策略来自动关闭基于定义计划的 VM。

## 上限
<a id="caps" class="xliff"></a>
上限是一种可最大程度减少实验室浪费的机制。 例如，可以将上限设置为限制每个用户或在实验室中可创建的 VM 数量。

## 安全级别
<a id="security-levels" class="xliff"></a>
安全访问权限由 Azure 基于角色的访问控制 (RBAC) 决定。 若要了解如何访问，最好先了解权限、角色和由 RBAC 定义的作用域之间的差异。

* 权限 - 权限是对特定操作的定义访问，例如，对所有虚拟机的读取访问权限。
* 角色 - 角色是一组可进行分组、可分配给用户的权限。 例如，“订阅所有者”角色对订阅中所有资源都具有访问权限。
* 作用域 - 作用域是 Azure 资源层次结构中的级别，例如，资源组、单个实验室或整个订阅。

开发测试实验室的作用域中存在两种定义用户权限的角色：实验室所有者和实验室用户。

* 实验室所有者 - 实验室所有者具有对实验室中所有资源的访问权限。 因此，实验室所有者可以修改策略、读取和写入任意 VM、更改虚拟网络等。
* 实验室用户 - 实验室用户可查看实验室中的所有资源（例如 VM、策略和虚拟网络），但不能修改其他用户创建的策略和 VM。

若要查看如何在开发测试实验室中创建自定义角色，可参阅本文[向用户授予特定实验室策略的权限](devtest-lab-grant-user-permissions-to-specific-lab-policies.md)。

由于作用域是分层的，因此用户在某个作用域具有权限时，将会自动授予他所包含的每个较低级别的作用域的权限。 例如，如果用户分配到订阅所有者的角色，那么他可访问订阅中的所有资源，包括所有虚拟机、所有虚拟网络和所有实验室。 因此，订阅所有者将自动继承实验室所有者的角色。 但是反过来则不适用。 实验室所有者具有访问实验室的权限，其低于订阅级别的作用域。 因此，实验室所有者将不能查看虚拟机或虚拟网络或实验室之外的任何资源。

## Azure Resource Manager 模板
<a id="azure-resource-manager-templates" class="xliff"></a>
本文中所述的所有概念都可以使用 Azure Resource Manager 模板配置，这些模板可用于定义 Azure 解决方案的基础结构/配置，并以一致的状态反复部署该解决方案。

[了解 Azure Resource Manager 模板的结构和语法](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-authoring-templates#template-format)描述了 Azure Resource Manager 模板的结构，以及模板的不同节中提供的属性。

[!INCLUDE [devtest-lab-try-it-out](../../includes/devtest-lab-try-it-out.md)]

## 后续步骤
<a id="next-steps" class="xliff"></a>
[在开发测试实验室中创建实验室](devtest-lab-create-lab.md)

