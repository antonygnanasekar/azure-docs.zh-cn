---
title: "使用 Xamarin.Forms 移动应用入门"
description: "按照本教程进行操作，开始使用 Azure 移动应用进行 Xamarin.Forms 开发"
services: app-service\mobile
documentationcenter: xamarin
author: ggailey777
manager: syntaxc4
editor: 
ms.assetid: 5e692220-cc89-4548-96c8-35259722acf5
ms.service: app-service-mobile
ms.workload: mobile
ms.tgt_pltfrm: mobile-xamarin
ms.devlang: dotnet
ms.topic: hero-article
ms.date: 10/01/2016
ms.author: glenga
ms.translationtype: HT
ms.sourcegitcommit: 99523f27fe43f07081bd43f5d563e554bda4426f
ms.openlocfilehash: cb959867ccfc85993694bd810f08e2f8150b44f3
ms.contentlocale: zh-cn
ms.lasthandoff: 08/05/2017

---
# <a name="create-a-xamarinforms-app"></a>创建 Xamarin.Forms 应用
[!INCLUDE [app-service-mobile-selector-get-started](../../includes/app-service-mobile-selector-get-started.md)]

## <a name="overview"></a>概述
本教程说明如何使用 Azure 移动应用后端向 Xamarin.Forms 移动应用添加基于云的后端服务。 将创建一个新的移动应用后端以及一个简单的*待办事项列表* Xamarin.Forms 应用，此应用将应用数据存储在 Azure 中。

只有在完成本教程后，才可以学习有关 Xamarin.Forms 的所有其他移动应用教程。

## <a name="prerequisites"></a>先决条件
要完成本教程，需要以下各项：

* 有效的 Azure 帐户。 如果没有帐户，可以注册 Azure 试用版并获取多达 10 个免费的移动应用，即使在试用期结束之后仍可继续使用这些应用。 有关详细信息，请参阅 [Azure 免费试用](https://azure.microsoft.com/pricing/free-trial/)。
* Visual Studio with Xamarin。 有关说明，请参阅 [设置和安装 Visual Studio 和 Xamarin](https://msdn.microsoft.com/library/mt613162.aspx) 。
* 安装了 Xcode v7.0 版或更高版本以及 Xamarin Studio Community 的 Mac。 请参阅[设置和安装 Visual Studio 和 Xamarin](https://msdn.microsoft.com/library/mt613162.aspx) 以及 [Mac 用户的设置、安装和验证](https://msdn.microsoft.com/library/mt488770.aspx) (MSDN)。

## <a name="create-a-new-azure-mobile-app-backend"></a>创建新的 Azure 移动应用后端
按照下列步骤创建新的移动应用后端。

[!INCLUDE [app-service-mobile-dotnet-backend-create-new-service](../../includes/app-service-mobile-dotnet-backend-create-new-service.md)]

现已预配可供移动客户端应用程序使用的 Azure 移动应用后端。 接下来，将为简单的“待办事项列表”后端下载服务器项目并将其发布到 Azure。

## <a name="configure-the-server-project"></a>配置服务器项目
按照下列步骤将服务器项目配置为使用 Node.js 或 .NET 后端。

[!INCLUDE [app-service-mobile-configure-new-backend](../../includes/app-service-mobile-configure-new-backend.md)]

## <a name="download-and-run-the-xamarinforms-solution"></a>下载并运行 Xamarin.Forms 解决方案
在此有两个选项。 可以将解决方案下载到 Mac 并在 Xamarin Studio 中打开它，也可以将解决方案下载到 Windows 计算机并使用联网的 Mac 在 Visual Studio 中打开它以生成 iOS 应用。 如果需要有关 Xamarin 设置方案的更详细说明，请参阅 [设置和安装 Visual Studio 和 Xamarin](https://msdn.microsoft.com/library/mt613162.aspx) 。

让我们继续：

1. 在 Mac 或 Windows 计算机上，在浏览器窗口中打开 [Azure 门户] 。
2. 在移动应用的“设置”边栏选项卡上，单击“开始使用”（在“移动”下）>“Xamarin.Forms”。 在步骤 3 下，单击“创建新应用”（如果尚未选择它）。  接下来，单击“下载”  按钮。

   此时会下载包含已连接到移动应用的客户端应用程序的项目。 将压缩的项目文件保存到本地计算机，并记下保存位置。
3. 解压缩下载的项目，并在 Xamarin Studio 或 Visual Studio 中打开它。

   ![][9]

   ![][8]

## <a name="optional-run-the-ios-project"></a>（可选）运行 iOS 项目
本部分用于运行适用于 iOS 设备的 Xamarin iOS 项目。 如果不使用 iOS 设备，可以跳过本部分。

#### <a name="in-xamarin-studio"></a>在 Xamarin Studio 中
1. 右键单击 iOS 项目，并单击“设为启动项目” 。
2. 在“运行”菜单上，单击“开始调试”以生成项目，并在 iPhone 模拟器中启动应用。

#### <a name="in-visual-studio"></a>在 Visual Studio 中
1. 右键单击 iOS 项目，并单击“设为启动项目” 。
2. 在“生成”菜单上，单击“配置管理器”。
3. 在“配置管理器”对话框中，选中 iOS 项目的“生成”和“部署”复选框。
4. 按 **F5** 键生成项目，并在 iPhone 模拟器中启动应用。

   > [!NOTE]
   > 如果遇到生成问题，请运行 NuGet 包管理器并更新到 Xamarin 支持包的最新版本。 有时快速入门项目可能会在更新到最新版本时滞后。    
   >
   >

在应用中键入有意义的文本（例如 *Learn Xamarin*），并单击“+”按钮。

![][10]

这样可向在 Azure 中托管的新移动应用后端发送 POST 请求。 来自请求的数据被插入到 TodoItem 表。 移动应用后端返回存储在表中的项，数据显示在列表中。

> [!NOTE]
> 可在解决方案的可移植类库项目的 TodoItemManager.cs C# 文件中找到用于访问移动应用后端的代码。
>
>

## <a name="optional-run-the-android-project"></a>（可选）运行 Android 项目
本部分用于运行适用于 Android 的 Xamarin droid 项目。 如果不使用 Android 设备，可以跳过本部分。

#### <a name="in-xamarin-studio"></a>在 Xamarin Studio 中
1. 右键单击 Android 项目，并单击“设为启动项目” 。
2. 在“运行”菜单上，单击“开始调试”以生成项目，并在 Android 模拟器中启动应用。

#### <a name="in-visual-studio"></a>在 Visual Studio 中
1. 右键单击 Android (Droid) 项目，并单击“设为启动项目” 。
2. 在“生成”菜单上，单击“配置管理器”。
3. 在“配置管理器”对话框中，选中 Android 项目的“生成”和“部署”复选框。
4. 按 **F5** 键生成项目，并在 Android 模拟器中启动应用。

   > [!NOTE]
   > 如果遇到生成问题，请运行 NuGet 包管理器并更新到 Xamarin 支持包的最新版本。 有时快速入门项目可能会在更新到最新版本时滞后。    
   >
   >

在应用中键入有意义的文本（例如 *Learn Xamarin*），并单击“+”按钮。

![][11]

这样可向在 Azure 中托管的新移动应用后端发送 POST 请求。 来自请求的数据被插入到 TodoItem 表。 移动应用后端返回存储在表中的项，数据显示在列表中。

> [!NOTE]
> 可在解决方案的可移植类库项目的 TodoItemManager.cs C# 文件中找到用于访问移动应用后端的代码。
>
>

## <a name="optional-run-the-windows-project"></a>（可选）运行 Windows 项目
本部分用于运行适用于 Windows 设备的 Xamarin WinApp 项目。 如果不使用 Windows 设备，可以跳过本部分。

#### <a name="in-visual-studio"></a>在 Visual Studio 中
1. 右键单击任一 Windows 项目，并单击“设为启动项目” 。
2. 在“生成”菜单上，单击“配置管理器”。
3. 在“配置管理器”对话框中，选中所选 Windows 项目的“生成”和“部署”复选框。
4. 按 **F5** 键生成项目，并在 Windows 模拟器中启动应用。

   > [!NOTE]
   > 如果遇到生成问题，请运行 NuGet 包管理器并更新到 Xamarin 支持包的最新版本。 有时快速入门项目可能会在更新到最新版本时滞后。    
   >
   >

在应用中键入有意义的文本（例如 *Learn Xamarin*），并单击“+”按钮。

这样可向在 Azure 中托管的新移动应用后端发送 POST 请求。 来自请求的数据被插入到 TodoItem 表。 移动应用后端返回存储在表中的项，数据显示在列表中。

![][12]

> [!NOTE]
> 可在解决方案的可移植类库项目的 TodoItemManager.cs C# 文件中找到用于访问移动应用后端的代码。
>
>

## <a name="next-steps"></a>后续步骤
* [向应用添加身份验证](app-service-mobile-xamarin-forms-get-started-users.md)  
  了解如何使用标识提供者对应用的用户进行身份验证。
* [向应用添加推送通知](app-service-mobile-xamarin-forms-get-started-push.md)  
  了解如何为应用添加推送通知支持，以及如何将移动应用后端配置为使用 Azure 通知中心发送推送通知。
* [为应用启用脱机同步](app-service-mobile-xamarin-forms-get-started-offline-data.md)  
  了解如何使用移动应用后端向应用添加脱机支持。 脱机同步允许最终用户与移动应用进行交互（查看、添加或修改数据），即使在没有网络连接时也是如此。
* [如何使用 Azure 移动应用的托管客户端](app-service-mobile-dotnet-how-to-use-client-library.md)  
  了解如何在 Xamarin 应用中使用托管客户端 SDK。

<!-- Anchors. -->
[Getting started with mobile app backends]:#getting-started
[Create a new mobile app backend]:#create-new-service
[Next Steps]:#next-steps


<!-- Images. -->
[6]: ./media/app-service-mobile-xamarin-forms-get-started/xamarin-forms-quickstart.png
[8]: ./media/app-service-mobile-xamarin-forms-get-started/xamarin-forms-quickstart-vs.png
[9]: ./media/app-service-mobile-xamarin-forms-get-started/xamarin-forms-quickstart-xs.png
[10]: ./media/app-service-mobile-xamarin-forms-get-started/mobile-quickstart-startup-ios.png
[11]: ./media/app-service-mobile-xamarin-forms-get-started/mobile-quickstart-startup-android.png
[12]: ./media/app-service-mobile-xamarin-forms-get-started/mobile-quickstart-startup-windows.png


<!-- URLs. -->
[Visual Studio Professional 2013]: https://go.microsoft.com/fwLink/p/?LinkID=257546
[Mobile app SDK]: http://go.microsoft.com/fwlink/?LinkId=257545
[Azure 门户]: https://portal.azure.com/

