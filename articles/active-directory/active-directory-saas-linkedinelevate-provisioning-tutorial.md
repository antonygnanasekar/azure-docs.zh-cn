---
title: "教程：使用 Azure Active Directory 为LinkedIn Elevate 配置自动用户预配 | Microsoft Docs"
description: "了解如何配置 Azure Active Directory 以便自动将用户帐户预配到 LinkedIn Elevate 以及取消其预配。"
services: active-directory
documentationcenter: 
author: asmalser-msft
writer: asmalser-msft
manager: stevenpo
ms.assetid: d4ca2365-6729-48f7-bb7f-c0f5ffe740a3
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/15/2017
ms.author: asmalser-msft
ms.translationtype: Human Translation
ms.sourcegitcommit: de674af369080ad7eb608608685e293f2326c8e6
ms.openlocfilehash: 526666301aad1e5284c621024649d9cd52c92d18
ms.contentlocale: zh-cn
ms.lasthandoff: 05/04/2017

---

# <a name="tutorial-configuring-linkedin-elevate-for-automatic-user-provisioning"></a>教程：为 LinkedIn Elevate 配置自动用户预配


本教程的目的是展示为了从 Azure AD 自动将用户帐户预配到 LinkedIn Elevate 以及取消其预配而需要在 LinkedIn Elevate 和 Azure AD 中执行的步骤。 

## <a name="prerequisites"></a>先决条件

在本教程中概述的方案假定您已具有以下各项：

*   一个 Azure Active Directory 租户
*   一个 LinkedIn Elevate 租户 
*   一个有权访问 LinkedIn 帐户中心的 LinkedIn Elevate 管理员帐户

> [!NOTE]
> Azure Active Directory 使用 [SCIM](http://www.simplecloud.info/) 协议与 LinkedIn Elevate 进行了集成。

## <a name="assigning-users-to-linkedin-elevate"></a>将用户分配到 LinkedIn Elevate

Azure Active Directory 使用称为“分配”的概念来确定哪些用户应收到对所选应用的访问权限。 在自动用户帐户预配的上下文中，只有已“分配”到 Azure AD 中的应用程序的用户和组才进行同步。 

在配置和启用预配服务之前，你需要确定 Azure AD 中的哪些用户和/或组表示需要访问 LinkedIn Elevate 应用的用户。 确定后，可以按照此处的说明将这些用户分配到 LinkedIn Elevate 应用：

[向企业应用分配用户或组](active-directory-coreapps-assign-user-azure-portal.md)

### <a name="important-tips-for-assigning-users-to-linkedin-elevate"></a>将用户分配到 LinkedIn Elevate 的重要提示

*    建议将单个 Azure AD 用户分配到 LinkedIn Elevate 来测试预配配置。 其他用户和/或组可以稍后分配。

*    将用户分配到 LinkedIn Elevate 时，必须在分配对话框中选择**用户**角色。 “默认访问权限”角色不可用于预配。


## <a name="configuring-user-provisioning-to-linkedin-elevate"></a>配置 LinkedIn Elevate 的用户预配

本部分将指导你完成以下操作：将 Azure AD 连接到 LinkedIn Elevate 的 SCIM 用户帐户预配 API，配置预配服务以便基于 Azure AD 中的用户和组分配在 LinkedIn Elevate 中创建、更新和禁用所分配的用户帐户。

**提示：**还可以选择按照 [Azure 门户](https://portal.azure.com)中提供的说明为 LinkedIn Elevate 启用基于 SAML 的单一登录。 可以独立于自动预配配置单一登录，尽管这两个功能互相补充。


### <a name="to-configure-automatic-user-account-provisioning-to-linkedin-elevate-in-azure-ad"></a>若要在 Azure AD 中配置 LinkedIn Elevate 的自动用户帐户预配，请执行以下操作：


第一步是检索你的 LinkedIn 访问令牌。 如果你是企业管理员，则可以自己预配一个访问令牌。 在帐户中心内，转到“设置”&gt;“全局设置”并打开“SCIM 设置”面板。

> [!NOTE]
> 如果是要直接（而不是通过某个链接）访问帐户中心，可以使用以下步骤进行访问。

1)  登录到帐户中心。

2)  选择“管理”&gt;“管理设置”。

3)  在左侧的边栏中，单击“高级集成”。 你将被定向到帐户中心。

4)  单击“+ 添加新的 SCIM 配置”并填写每个字段来完成该过程。

> 如果未启用自动分配许可证，则意味着只有用户数据是同步的。

![LinkedIn Elevate 预配](./media/active-directory-saas-linkedin-elevate-provisioning-tutorial/linkedin_elevate1.PNG)

> 如果启用了自动许可证分配，则需要记下应用程序实例和许可证类型。 许可证将按照“先来先服务”原则进行分配，直到所有许可证用完。

![LinkedIn Elevate 预配](./media/active-directory-saas-linkedin-elevate-provisioning-tutorial/linkedin_elevate2.PNG)

5)  单击“生成令牌”。 应当可以看到你的访问令牌显示在“访问令牌”字段下。

6)  离开页面之前，将你的访问令牌保存到剪贴板或计算机。

7) 接下来，登录到[Azure 门户](https://portal.azure.com)，浏览到“Azure Active Directory”>“企业应用”>“所有应用程序”部分。

8) 如果已为 LinkedIn Elevate 配置了单一登录，请使用搜索字段搜索 LinkedIn Elevate 实例。 否则，请选择“添加”并在应用程序库中搜索“LinkedIn Elevate”。 从搜索结果中选择 LinkedIn Elevate，并将其添加到应用程序列表。

9)    选择 LinkedIn Elevate 实例，然后选择“预配”选项卡。

10)    将“预配模式”设置为“自动”。

![LinkedIn Elevate 预配](./media/active-directory-saas-linkedin-elevate-provisioning-tutorial/linkedin_elevate3.PNG)

11)  在“管理员凭据”下填写以下字段：

* 在“租户 URL”字段中，输入 https://api.linkedin.com。

* 在“密钥标记”字段中，输入你在第 1 步中生成的访问令牌并单击“测试连接”。

* 应当会在门户的右上端看到一条成功通知。

12) 在“通知电子邮件”字段中输入应收到预配错误通知的用户或组的电子邮件地址，并选中下面的复选框。

13) 单击“保存” 。 

14) 在“属性映射”部分中，查看将从 Azure AD 同步到 LinkedIn Elevate 的用户和组属性。 请注意，选为**匹配**属性的属性将用于匹配 LinkedIn Elevate 中的用户帐户和组以执行更新操作。 选择“保存”按钮以提交任何更改。

![LinkedIn Elevate 预配](./media/active-directory-saas-linkedin-elevate-provisioning-tutorial/linkedin_elevate4.PNG)

15) 若要为 LinkedIn Elevate 启用 Azure AD 预配服务，请在“设置”部分中将“预配状态”更改为“启用”

16) 单击“保存” 。 

这将开始对在“用户和组”部分中分配给 LinkedIn Elevate 的任何用户和/或组进行初始同步。 请注意，初始同步执行的时间比后续同步长，只要服务正在运行，大约每隔 20 分钟就会进行一次同步。 可以使用“同步详细信息”部分监视进度并跟踪指向预配活动报告的链接，这些报告描述了预配服务对 LinkedIn Elevate 应用执行的所有操作。


## <a name="additional-resources"></a>其他资源

* [管理企业应用的用户帐户预配](active-directory-enterprise-apps-manage-provisioning.md)
* [什么是使用 Azure Active Directory 的应用程序访问和单一登录？](active-directory-appssoaccess-whatis.md)

