---
title: "使用自助应用程序访问时的问题 | Microsoft Docs"
description: "排除与自助应用程序访问相关的问题"
services: active-directory
documentationcenter: 
author: ajamess
manager: femila
ms.assetid: 
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/11/2017
ms.author: asteen
ms.translationtype: Human Translation
ms.sourcegitcommit: cc9e81de9bf8a3312da834502fa6ca25e2b5834a
ms.openlocfilehash: 4d8c241344485e50a1afde3cd67adf3d0a7c041f
ms.contentlocale: zh-cn
ms.lasthandoff: 04/11/2017

---

<a id="problem-using-self-service-application-access" class="xliff"></a>

# 使用自助应用程序访问时的问题

自助应用程序访问是帮助用户自己发现应用程序的绝佳方式，可选择性地允许业务组批准对这些应用程序的访问。 可允许业务组直接从其访问面板，管理分配给用户的密码单一登录应用程序的凭据。

在用户能够从其访问面板中自行发现应用程序之前，需要先对要允许用户自行发现并请求访问的任何应用程序启用“自助应用程序访问”。

<a id="general-issues-to-check-first" class="xliff"></a>

## 首先要检查的常规问题

-   确保已正确配置自助应用程序访问。 请参阅“如何配置自助应用程序访问”。

-   确保已让用户或组能够请求自助应用程序访问。

-   确保用户在进行自助应用程序访问时所访问的位置正确。 用户可以导航到其[应用程序访问面板](https://myapps.microsoft.com/)，单击“+添加”按钮以查找已启用自助访问的应用。

-   如果最近刚配置了自助应用程序访问，请尝试几分钟后再次登录和登出用户的“访问控制面板”，查看是否已显示自助访问更改。

<a id="how-to-configure-self-service-application-access" class="xliff"></a>

## 如何配置自助应用程序访问

若要启用应用程序的自助应用程序访问，请执行以下步骤：

1.  打开 [**Azure 门户**](https://portal.azure.com/)，以**全局管理员**身份登录

2.  在左侧主导航菜单底部单击“更多服务”，打开“Azure Active Directory 扩展”。

3.  在筛选器搜索框中键入“Azure Active Directory”，选择“Azure Active Directory”项。

4.  在 Azure Active Directory 的左侧导航菜单中，单击“企业应用程序”。

5.  单击“所有应用程序”，查看所有应用程序的列表。

  * 如果未看到想在此处显示的应用程序，请使用“所有应用程序列表”顶部的“筛选器”控件，并将“显示”选项设为“所有应用程序”。

6.  从列表中选择要对其启用自助访问的应用程序。

7.  加载应用程序后，在应用程序的左侧导航菜单中，单击“自助”。

8.  若要对此应用程序启用自助应用程序访问，请将“允许用户请求对此应用程序的访问权限?”切换到“是”。

9.  接下来，要选择向其添加请求此应用程序访问权限的用户的组，请单击“分配的用户应添加到哪个组?”标签旁边的选择器，然后选择一个组。

10. **可选：**如果希望获得业务批准之后才允许用户访问，请将“需要获得批准才能授权访问此应用程序?”切换到“是”。

11. **可选：对于仅使用密码单一登录的应用程序，**如果要允许业务审批人为已批准的用户指定发送到此应用程序的密码，请将“允许审批人设置此应用程序的用户密码?”切换到“是”。

12. **可选：**若要指定有权授予此应用程序访问权限的业务审批人，请单击“谁有权批准此应用程序的访问权限?”标签旁边的选择器，最多可选择 10 个单独业务审批人。

 >[!NOTE]
 > 不支持组。
 >
 >

13. **可选：** **对于公开角色的应用程序**，如果想要向角色分配已被批准使用自助服务的用户，请单击“在此应用程序中应向哪个角色分配用户?”旁边的选择器，选择要向其分配这些用户的角色。

14. 单击边栏选项卡顶部的“保存”按钮以完成操作。

完成自助应用程序配置后，用户可以导航到其[应用程序访问面板](https://myapps.microsoft.com/)，单击“+添加”按钮以查找已启用自助访问的应用。 业务审批人还可以在其[应用程序访问面板](https://myapps.microsoft.com/)中看到通知。 可以启用电子邮件，在用户请求需要审批人批准的应用程序的访问权限时，向审批人发送电子邮件通知。 

这些批准仅支持单个审批工作流，这意味着如果指定了多个审批人，任何一个审批人都可以批准对该应用程序的访问。

<a id="if-these-troubleshooting-steps-do-not-resolve-the-issue" class="xliff"></a>

## 如果这些故障排除步骤未解决此问题 

打开支持票证，并提供以下信息（如有）：

-   相关错误 ID

-   UPN（用户电子邮件地址）

-   TenantID

-   浏览器类型

-   错误发生的时区和时间/时间段

-   Fiddler 跟踪

<a id="next-steps" class="xliff"></a>

## 后续步骤
[为自助组管理设置 Azure Active Directory](active-directory-accessmanagement-self-service-group-management.md)

