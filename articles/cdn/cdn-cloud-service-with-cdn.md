<properties 
	pageTitle="将云服务与 Azure CDN 集成" 
	description="本教程演示如何部署云服务，以便从集成的 Azure CDN 终结点提供内容" 
	services="cdn, cloud-services" 
	documentationCenter=".net" 
	authors="cephalin" 
	manager="wpickett" 
	editor="tysonn"/>

<tags 
	ms.service="cdn" 
	ms.date="09/01/2015" 
	wacn.date=""/>


# <a name="intro"></a> 将云服务与 Azure CDN 集成 

云服务卡可以与 Azure CDN 集成，提供云服务的 `~/CDN` 路径中的任何内容。此方法具有以下优点：

- 轻松部署和更新云服务项目目录中的图像、脚本和样式表。
- 轻松升级云服务中的 NuGet 包，例如 jQuery 或 Bootstrap 版本 
- 在同一 Visual Studio 界面中管理 Web 应用程序和 CDN 提供的内容
- 为 Web 应用程序和 CDN 提供的内容应用统一的部署工作流
- 将 ASP.NET 绑定和缩减与 Azure CDN 集成

## 你要学习的知识 ##

在本教程中，你将了解如何：

-	[将 Azure CDN 终结点与云服务集成，通过 Azure CDN 在网页中提供静态内容](#deploy)
-	[在云服务中配置静态内容的缓存设置](#caching)
-	[通过 Azure CDN 的控制器操作提供内容](#controller)
-	[通过 Azure CDN 提供捆绑的和缩减的内容，同时保留 Visual Studio 中的脚本调试体验](#bundling)
-	[在 Azure CDN 处于脱机状态时通过配置回退脚本和 CSS](#fallback) 

## 要生成的项目 ##

你需要使用默认的 ASP.NET MVC 模板部署云服务 Web 角色，需要添加代码来处理集成 Azure CDN 所提供的内容（例如图像、控制器操作结果、默认的 JavaScript 和 CSS 文件），还需要编写代码来配置回退机制以处理 CDN 脱机时提供的捆绑包。

## 所需的项目 ##

本教程设置了以下前提条件：

-	有效的 [Microsoft Azure 帐户](/account/)
-	Visual Studio 2013 with [Azure SDK](http://go.microsoft.com/fwlink/p/?linkid=323510&clcid=0x409)

> [AZURE.NOTE]完成本教程需要有一个 Azure 帐户：
> + 你可以[建立一个 Azure 帐户](/pricing/1rmb-trial/) - 获取可用来试用付费版 Azure 服务的信用额度，甚至在用完信用额度后，你仍可以保留帐户和使用免费的 Azure 服务（如网站）。
> + 你可以[激活 MSDN 订户权益](/pricing/member-offers/msdn-benefits-details/) - 订阅 MSDN 后，你可以每月获得相应的信用额度，这些信用额度可以用于付费的 Azure 服务。

<a name="deploy"></a>
## 将云服务部署到集成 CDN 终结点 ##

在本部分，你需要将 Visual Studio 2013 中的默认 ASP.NET MVC 应用程序模板部署到云服务 Web 角色，然后将其与新的 CDN 终结点相集成。请根据以下说明进行操作：

1. 在 Visual Studio 2013 中转到“文件 > 新建 > 项目 > 云 > Azure 云服务”，以便从菜单栏创建新的 Azure 云服务。为该应用程序提供一个名称，然后单击“确定”。

	![](media/cdn-cloud-service-with-cdn/cdn-cs-1-new-project.PNG)

2. 选择“ASP.NET Web 角色”，然后单击“>”按钮。单击“确定”。

	![](media/cdn-cloud-service-with-cdn/cdn-cs-2-select-role.PNG)

3. 选择“MVC”，然后单击“确定”。

	![](media/cdn-cloud-service-with-cdn/cdn-cs-3-mvc-template.PNG)

4. 现在，将此 Web 角色发布到 Azure 云服务。右键单击云服务项目，然后选择“发布”。

	![](media/cdn-cloud-service-with-cdn/cdn-cs-4-publish-a.png)

5. 如果你尚未登录到 Microsoft Azure，请单击“登录”按钮。

	![](media/cdn-cloud-service-with-cdn/cdn-cs-5-publish-signin.png)

6. 在登录页中，使用你用来激活 Azure 帐户的 Microsoft 帐户登录。
7. 登录后，单击“下一步”。

	![](media/cdn-cloud-service-with-cdn/cdn-cs-6-publish-signedin.png)

8. 假定你尚未创建云服务或存储帐户，Visual Studio 将帮助你创建这两项。在“创建云服务和帐户”对话框中，键入所需的服务名称。然后单击**创建**。

	![](media/cdn-cloud-service-with-cdn/cdn-cs-7-publish-createserviceandstorage.png)

9. 在发布设置页验证配置并单击“发布”。

	![](media/cdn-cloud-service-with-cdn/cdn-cs-8-publish-finalize.png)

	>[AZURE.NOTE]云服务的发布过程需要很长时间。“为所有角色启用 Web 部署”选项允许你对 Web 角色进行快速的（但却是临时的）更新，从而极大地加快了云服务的调试速度。有关此选项的详细信息，请参阅[使用 Azure Tools 发布云服务](http://msdn.microsoft.com/library/ff683672.aspx)。

	当 **Microsoft Azure 活动日志**显示发布状态为“已完成”时，你就可以创建一个与此云服务集成的 CDN 终结点。

1. 若要创建 CDN 终结点，请登录到 [Azure 管理门户](http://manage.windowsazure.cn/)。
2. 单击“新建 > 应用服务 > CDN > 快速创建”。选择 **http://*&lt;servicename>*.chinacloudapp.cn/cdn/**，然后单击“创建”。

	![](media/cdn-cloud-service-with-cdn/cdn-cs-10-createcdn.png)

	>[AZURE.NOTE]创建 CDN 终结点以后，Azure 门户就会向你显示其 URL 以及所集成的源域。不过，需要一定的时间才能将新 CDN 终结点的配置完全传播到所有 CDN 节点位置。

	请注意，CDN 终结点已绑定到云服务的 **cdn/** 路径。你可以在 **WebRole1** 项目中创建 **cdn** 文件夹，也可以使用 URL 重新来剥离此路径的所有传入链接。在本教程中，你将选择后一种方法。

3. 回到 Azure 门户，在“CDN”选项卡中，单击刚创建的 CDN 终结点的名称。

	![](media/cdn-cloud-service-with-cdn/cdn-cs-11-disablequerya.png)

4. 单击“启用查询字符串”在 CDN 缓存中启用查询字符串。启用此功能后，通过不同的查询字符串进行访问的同一链接就可以缓存为不同的项。

	![](media/cdn-cloud-service-with-cdn/cdn-cs-12-disablequeryb.png)

	>[AZURE.NOTE]虽然启用查询字符串对于教程的此部分来说不是必需的，但为方便起见，最好是尽早启用此功能，因为在这里所做的任何更改都需要一定的时间才能传播到所有 CDN 节点，而你并不想让任何非查询字符串支持的内容充斥 CDN 缓存（稍后将讨论如何更新 CDN 内容）。

5. 对 CDN 终结点执行 ping 操作，确保所做的更改已传播到 CDN 节点。可能需要长达 1 小时的等待才能看到它对 ping 的响应。

	![](media/cdn-cloud-service-with-cdn/cdn-cs-13-testcdn.png)

6. 回到 Visual Studio 2013，在 **WebRole1** 项目中打开 **Web.config**，然后将以下代码添加到 `<system.webServer>` 标记中：

		<system.webServer>
		  <rewrite>
		    <rules>
		      <rule name="RewriteIncomingCdnRequest" stopProcessing="true">
		        <match url="^cdn/(.*)$"/>
		        <action type="Rewrite" url="{R:1}"/>
		      </rule>
		    </rules>
		  </rewrite>
		  ...
		</system.webServer>

7. 重新发布云服务。右键单击云服务项目，然后选择“发布”。

	![](media/cdn-cloud-service-with-cdn/cdn-cs-4-publish-a.png)

8. 当发布状态为“已完成”时，打开一个浏览器窗口，然后导航到 **http://*&lt;cdnName>*.vo.msecnd.net/Content/bootstrap.css**。在我的设置中，此 URL 为：

		http://az632148.vo.msecnd.net/Content/bootstrap.css

	这对应于 CDN 终结点的以下源 URL：

		http://cephalinservice.chinacloudapp.cn/cdn/Content/bootstrap.css

	在 Web 应用中进行 URL 重写后，实际缓存到我的 CDN 缓存中的文件为：

		http://cephalinservice.chinacloudapp.cn/Content/bootstrap.css

	导航到 **http://*&lt;cdnName>*.vo.msecnd.net/Content/bootstrap.css** 时，系统会提示你下载 bootstrap.css，该文件来自你已发布的 Web 应用。

	![](media/cdn-cloud-service-with-cdn/cdn-1-browser-access.PNG)

你可以直接通过 CDN 终结点，采用类似方式访问 **http://*&lt;serviceName>*.chinacloudapp.cn/** 中提供的任何可公开访问的 URL。例如：

-	/Script 路径中的 .js 文件
-	/Content 路径中的任何内容文件
-	任何控制器/操作 
-	任何带查询字符串的 URL（如果在你的 CDN 终结点启用了查询字符串功能）

实际上，你可以使用上述配置托管 **http://*&lt;cdnName>*.vo.msecnd.net/** 中的整个云服务。导航到 ****http://az632148.vo.msecnd.net/** 即可获取 Home/Index 中的操作结果。

![](media/cdn-cloud-service-with-cdn/cdn-2-home-page.PNG)

但是，这并不意味着通过 Azure CDN 提供整个云服务适合所有情况（或是一种通用的作法）。部分注意事项如下：

-	此方法要求你的整个站点都是公共的，因为 Azure CDN 不能提供任何私有内容。
-	如果 CDN 终结点因某种原因而脱机（不管是因为计划的维护，还是因为用户错误），你的整个云服务都会脱机，除非可以将客户重定向到源 URL **http://*&lt;serviceName>*.chinacloudapp.cn/**。 
-	即使使用自定义的 Cache-Control 设置（参见[在云服务中配置静态文件的缓存选项](#caching)），CDN 终结点也不会改善活动度极强的动态内容的性能。请注意，如果你尝试从如上所示的 CDN 终结点加载主页，则第一次操作时至少需要 5 秒钟才能加载默认主页，而该主页是相当简单的主页。想象一下，如果此页包含每分钟必须更新的动态内容，则客户端体验会是一种什么样的情景？从 CDN 终结点提供动态内容要求缓存过期时间短，这会造成在 CDN 终结点处频繁出现缓存未命中的情况。这会损害云服务的性能，与 CDN 的初衷背道而驰。

替代方法是在云服务中按每次的具体情况来判断哪些内容可以由 Azure CDN 提供。就这点来说，我们已向你介绍了如何通过 CDN 终结点访问各个内容文件。我会在[通过 Azure CDN 的控制器操作提供内容](#controller)中向你演示如何处理通过 CDN 终结点进行的特定控制器操作。

你可以指定更严格的 URL 重写规则，限制可以通过 CDN 终结点访问的内容。例如，若要限制对 *\\Scripts* 文件夹进行 URL 重新，可将上述重写规则改为下述代码：
   
	<rule name="RewriteIncomingCdnRequest" stopProcessing="true">
	  <match url="^cdn/Scripts/(.*)$"/>
	  <action type="Rewrite" url="Scripts/{R:1}"/>
	</rule>

<a name="caching"></a>
## 在云服务中配置静态文件的缓存选项 ##

通过在云服务中进行 Azure CDN 集成，你可以指定你所希望的在 CDN 终结点中缓存静态内容的方式。为此，请通过某个 Web 角色项目（例如 WebRole1）打开 *Web.config*，然后将 `<staticContent>` 元素添加到 `<system.webServer>`。以下 XML 将缓存配置为 3 天后过期。

	<system.webServer>
	  <staticContent>
	    <clientCache cacheControlMode="UseMaxAge" cacheControlMaxAge="3.00:00:00"/>
	  </staticContent>
	  ...
	</system.webServer>

执行完此操作后，云服务中的所有静态文件将会遵守 CDN 缓存中的同一规则。若要对缓存设置进行更细致的控制，可将 *Web.config* 文件添加到一个文件夹中，然后在该处添加你的设置。例如，可将 *Web.config* 文件添加到 *\\Content* 文件夹中，然后使用以下 XML 替换其中的内容：

	<?xml version="1.0"?>
	<configuration>
	  <system.webServer>
	    <staticContent>
	      <clientCache cacheControlMode="UseMaxAge" cacheControlMaxAge="15.00:00:00"/>
	    </staticContent>
	  </system.webServer>
	</configuration>

此设置可以让 *\\Content* 文件夹中的所有静态文件缓存 15 天。

有关如何配置 `<clientCache>` 元素的详细信息，请参阅[客户端缓存 &lt;clientCache>](http://www.iis.net/configreference/system.webserver/staticcontent/clientcache)。

在[通过 Azure CDN 的控制器操作提供内容](#controller)中，我还将向你演示如何针对 CDN 缓存中的控制器操作结果配置缓存设置。

<a name="controller"></a>
## 通过 Azure CDN 的控制器操作提供内容 ##

将云服务 Web 角色与 Azure CDN 集成以后，可以相对轻松地通过 Azure CDN 的控制器操作提供内容。直接通过 Azure CDN 来提供云服务（如上所示）并不是必需的，[Maarten Balliauw](https://twitter.com/maartenballiauw) 在[通过 Azure CDN 减少 Web 延迟](http://channel9.msdn.com/events/TechDays/Techdays-2014-the-Netherlands/Reducing-latency-on-the-web-with-the-Windows-Azure-CDN)中向你演示了如何通过有趣的 MemeGenerator 控制器来这样做。在这里，我将简单地再现该过程。

假设在你的云服务中，你希望根据年轻的 Chuck Norris 的照片（由 [Alan Light](http://www.flickr.com/photos/alan-light/218493788/) 拍摄）来制作一个搞笑的迷因，如下所示：

![](media/cdn-cloud-service-with-cdn/cdn-5-memegenerator.PNG)

你提供了一个简单的 `Index` 操作，让客户能够指定照片中的夸张元素，然后待这些元素发布到操作后制作出迷因。由于这是 Chuck Norris，可以想见该页面会在全球热传。这是通过 Azure CDN 提供半动态内容的一个很好的示例。

按照上述步骤设置此控制器操作：

1. 在 *\\Controllers* 文件夹中，创建一个新的名为 *MemeGeneratorController.cs* 的 .cs 文件，然后使用以下代码替换其中的内容。请务必将突出显示的部分替换为你的 CDN 名称。  

		using System;
		using System.Collections.Generic;
		using System.Diagnostics;
		using System.Drawing;
		using System.IO;
		using System.Net;
		using System.Web.Hosting;
		using System.Web.Mvc;
		using System.Web.UI;
		
		namespace WebRole1.Controllers
		{
		    public class MemeGeneratorController : Controller
		    {
		        static readonly Dictionary<string, Tuple<string ,string>> Memes = new Dictionary<string, Tuple<string, string>>();
		
		        public ActionResult Index()
		        {
		            return View();
		        }
		
		        [HttpPost, ActionName("Index")]
		        public ActionResult Index_Post(string top, string bottom)
		        {
		            var identifier = Guid.NewGuid().ToString();
		            if (!Memes.ContainsKey(identifier))
		            {
		                Memes.Add(identifier, new Tuple<string, string>(top, bottom));
		            }
		
		            return Content("<a href="" + Url.Action("Show", new {id = identifier}) + "">here's your meme</a>");
		        }
		
		        [OutputCache(VaryByParam = "*", Duration = 1, Location = OutputCacheLocation.Downstream)]
		        public ActionResult Show(string id)
		        {
		            Tuple<string, string> data = null;
		            if (!Memes.TryGetValue(id, out data))
		            {
		                return new HttpStatusCodeResult(HttpStatusCode.NotFound);
		            }
		
		            if (Debugger.IsAttached) // Preserve the debug experience
		            {
		                return Redirect(string.Format("/MemeGenerator/Generate?top={0}&bottom={1}", data.Item1, data.Item2));
		            }
		            else // Get content from Azure CDN
		            {
		                return Redirect(string.Format("http://<yourCdnName>.vo.msecnd.net/MemeGenerator/Generate?top={0}&bottom={1}", data.Item1, data.Item2));
		            }
		        }
		
		        [OutputCache(VaryByParam = "*", Duration = 3600, Location = OutputCacheLocation.Downstream)]
		        public ActionResult Generate(string top, string bottom)
		        {
		            string imageFilePath = HostingEnvironment.MapPath("~/Content/chuck.bmp");
		            Bitmap bitmap = (Bitmap)Image.FromFile(imageFilePath);
		
		            using (Graphics graphics = Graphics.FromImage(bitmap))
		            {
		                SizeF size = new SizeF();
		                using (Font arialFont = FindBestFitFont(bitmap, graphics, top.ToUpperInvariant(), new Font("Arial Narrow", 100), out size))
		                {
		                    graphics.DrawString(top.ToUpperInvariant(), arialFont, Brushes.White, new PointF(((bitmap.Width - size.Width) / 2), 10f));
		                }
		                using (Font arialFont = FindBestFitFont(bitmap, graphics, bottom.ToUpperInvariant(), new Font("Arial Narrow", 100), out size))
		                {
		                    graphics.DrawString(bottom.ToUpperInvariant(), arialFont, Brushes.White, new PointF(((bitmap.Width - size.Width) / 2), bitmap.Height - 10f - arialFont.Height));
		                }
		            }
		
		            MemoryStream ms = new MemoryStream();
		            bitmap.Save(ms, System.Drawing.Imaging.ImageFormat.Png);
		            return File(ms.ToArray(), "image/png");
		        }
		
		        private Font FindBestFitFont(Image i, Graphics g, String text, Font font, out SizeF size)
		        {
		            // Compute actual size, shrink if needed
		            while (true)
		            {
		                size = g.MeasureString(text, font);
		
		                // It fits, back out
		                if (size.Height < i.Height &&
		                     size.Width < i.Width) { return font; }
		
		                // Try a smaller font (90% of old size)
		                Font oldFont = font;
		                font = new Font(font.Name, (float)(font.Size * .9), font.Style);
		                oldFont.Dispose();
		            }
		        }
		    }
		}

2. 右键单击默认的 `Index()` 操作，然后选择“添加视图”。

	![](media/cdn-cloud-service-with-cdn/cdn-6-addview.PNG)

3.  接受下面的设置，然后单击“添加”。

	![](media/cdn-cloud-service-with-cdn/cdn-7-configureview.PNG)

4. 打开这个新的 *Views\\MemeGenerator\\Index.cshtml*，将其中的内容替换为下面这个简单的 HTML，以便提交夸张元素：

		<h2>Meme Generator</h2>
		
		<form action="" method="post">
		    <input type="text" name="top" placeholder="Enter top text here" />
		    <br />
		    <input type="text" name="bottom" placeholder="Enter bottom text here" />
		    <br />
		    <input class="btn" type="submit" value="Generate meme" />
		</form>

5. 重新发布云服务，然后在浏览器中导航到 **http://*&lt;serviceName>*.chinacloudapp.cn/MemeGenerator/Index**。

当你将窗体值提交到 `/MemeGenerator/Index` 时，`Index_Post` 操作方法会返回一个指向 `Show` 操作方法的链接，其中包含相应的输入标识符。单击该链接即可访问以下代码：

	[OutputCache(VaryByParam = ";*";, Duration = 1, Location = OutputCacheLocation.Downstream)]
	public ActionResult Show(string id)
	{
	    Tuple&lt;string, string&gt; data = null;
	    if (!Memes.TryGetValue(id, out data))
	    {
	        return new HttpStatusCodeResult(HttpStatusCode.NotFound);
	    }
	
	    if (Debugger.IsAttached) // Preserve the debug experience
	    {
	        return Redirect(string.Format(";/MemeGenerator/Generate?top={0}&bottom={1}";, data.Item1, data.Item2));
	    }
	    else // Get content from Azure CDN
	    {
	        return Redirect(string.Format(";http://<mark>&lt;cdnName&gt;</mark>.vo.msecnd.net/MemeGenerator/Generate?top={0}&amp;bottom={1}";, data.Item1, data.Item2));
	    }
	}
	
如果连接了本地调试器，你就可以通过本地重定向获得常规的调试体验。如果是在云服务中运行，则会重定向到：

	http://<yourCDNName>.vo.msecnd.net/MemeGenerator/Generate?top=<formInput>&bottom=<formInput>

这对应于 CDN 终结点的以下源 URL：

	http://<youCloudServiceName>.chinacloudapp.cn/cdn/MemeGenerator/Generate?top=<formInput>&bottom=<formInput>

由于已应用过 URL 重写规则，缓存到 CDN 终结点的实际文件为：

	http://<youCloudServiceName>.chinacloudapp.cn/MemeGenerator/Generate?top=<formInput>&bottom=<formInput>

然后，你可以使用 `Generate` 方法的 `OutputCacheAttribute` 属性来指定 Azure CDN 认可的操作结果缓存方式。以下代码指定缓存在 1 小时（3,600 秒）后过期。

    [OutputCache(VaryByParam = "*", Duration = 3600, Location = OutputCacheLocation.Downstream)]

同样，你可以使用所需的缓存选项，在云服务中通过 Azure CDN 的任何控制器操作提供内容。

在下一部分，我将向你演示如何通过 Azure CDN 提供绑定型和缩减型脚本和 CSS。

<a name="bundling"></a>
## 将 ASP.NET 绑定和缩减与 Azure CDN 集成 ##

脚本和 CSS 样式表很少变化，尤其适合 Azure CDN 缓存。若要将绑定和缩减集成到 Azure CDN，最便捷的方式是通过 Azure CDN 提供整个 Web 角色。当然，你可能不希望这样做，不过我还是会向你演示如何在这样做的同时，保留所需的有关 ASP.NET 绑定和缩减的开发人员体验，例如：

-	理想的调试模式体验
-	简化的部署
-	即时更新客户端，进行脚本/CSS 版本升级
-	在 CDN 终结点故障时适用的回退机制
-	最大程度减少代码修改

在[将 Azure CDN 终结点与 Azure 网站集成，通过 Azure CDN 在网页中提供静态内容](#deploy)部分创建的 **WebRole1** 项目中，打开 *App\_Start\\BundleConfig.cs*，然后查看 `bundles.Add()` 方法调用情况。

    public static void RegisterBundles(BundleCollection bundles)
    {
        bundles.Add(new ScriptBundle("~/bundles/jquery").Include(
                    "~/Scripts/jquery-{version}.js"));
		...
    }

第一个 `bundles.Add()` 语句将脚本捆绑包添加到了虚拟目录 `~/bundles/jquery`。然后，打开 *Views\\Shared\_Layout.cshtml* 以查看脚本捆绑包标记的呈现方式。你应该能够找到以下 Razor 代码行：

    @Scripts.Render("~/bundles/jquery")

当该 Razor 代码在 Azure Web 角色中运行时，它会呈现脚本捆绑包的 `<script>` 标记，如下所示：

    <script src="/bundles/jquery?v=FVs3ACwOLIVInrAl5sdzR2jrCDmVOWFbZMY6g6Q0ulE1"></script>

但是，通过键入 `F5` 而让其在 Visual Studio 中运行时，它会逐一呈现捆绑包中的每个脚本文件（在上面的示例中，捆绑包中只有一个脚本文件）：

    <script src="/Scripts/jquery-1.10.2.js"></script>

这样可以让你在开发环境中调试 JavaScript 代码时，减少生产环境中出现的并发客户端连接（绑定），改进文件下载性能（缩减）。这是一项很适合保留在 Azure CDN 集成中的功能。此外，由于呈现的捆绑包已包含自动生成的版本字符串，因此你希望能够复制该功能，这样当你通过 NuGet 更新 jQuery 版本时，就能够尽快在客户端进行相应的更新。

按照以下步骤操作，将 ASP.NET 绑定和缩减功能集成到 CDN 终结点。

1. 回到 *App\_Start\\BundleConfig.cs*，修改 `bundles.Add()` 方法以使用其他[捆绑包构造函数](http://msdn.microsoft.com/zh-cn/library/jj646464.aspx)来指定 CDN 地址。为此，请使用以下代码替换 `RegisterBundles` 方法定义：  

		public static void RegisterBundles(BundleCollection bundles)
		{
		    bundles.UseCdn = true;
		    var version = System.Reflection.Assembly.GetAssembly(typeof(Controllers.HomeController))
		        .GetName().Version.ToString();
		    var cdnUrl = "http://<yourCDNName>.vo.msecnd.net/{0}?v=" + version;
		
		    bundles.Add(new ScriptBundle("~/bundles/jquery", string.Format(cdnUrl, "bundles/jquery")).Include(
		                "~/Scripts/jquery-{version}.js"));
		
		    bundles.Add(new ScriptBundle("~/bundles/jqueryval", string.Format(cdnUrl, "bundles/jqueryval")).Include(
		                "~/Scripts/jquery.validate*"));
		
		    // Use the development version of Modernizr to develop with and learn from. Then, when you're
		    // ready for production, use the build tool at http://modernizr.com to pick only the tests you need.
		    bundles.Add(new ScriptBundle("~/bundles/modernizr", string.Format(cdnUrl, "bundles/modernizer")).Include(
		                "~/Scripts/modernizr-*"));
		
		    bundles.Add(new ScriptBundle("~/bundles/bootstrap", string.Format(cdnUrl, "bundles/bootstrap")).Include(
		                "~/Scripts/bootstrap.js",
		                "~/Scripts/respond.js"));
		
		    bundles.Add(new StyleBundle("~/Content/css", string.Format(cdnUrl, "Content/css")).Include(
		                "~/Content/bootstrap.css",
		                "~/Content/site.css"));
		}

	请确保将 `<yourCDNName>` 替换为你的 Azure CDN 的名称。

	你将使用纯单词来设置 `bundles.UseCdn = true` 并已经将仔细编写的 CDN URL 添加到了每个捆绑包。例如，代码中的第一个构造函数：

		new ScriptBundle("~/bundles/jquery", string.Format(cdnUrl, "bundles/jquery"))

	相当于：

		new ScriptBundle("~/bundles/jquery", string.Format(cdnUrl, "http://<yourCDNName>.vo.msecnd.net/bundles/jquery?v=<W.X.Y.Z>"))

	进行本地调试时，此构造函数会指示 ASP.NET 绑定和缩减功能呈现各个脚本文件，但会使用指定的 CDN 地址来访问相关脚本。不过，请注意这个仔细编写的 CDN URL 存在两个重要的特征：
	
	-	此 CDN URL 的源是 `http://<yourCloudService>.chinacloudapp.cn/bundles/jquery?v=<W.X.Y.Z>`，这实际上是云服务中脚本捆绑包的虚拟目录。
	-	由于你使用的是 CDN 构造函数，因此捆绑包的 CDN 脚本标记不再包含在呈现的 URL 中自动生成的版本字符串。每次对脚本捆绑包进行修改而造成 Azure CDN 中出现缓存未命中的情况时，你都必须手动生成唯一的脚本字符串。同时，这个唯一的版本字符串在部署过程中必须保持不变，以便在捆绑包部署完以后，最大程度地提高 Azure CDN 中的缓存命中率。
	-	查询字符串 v=<W.X.Y.Z> 的功能是从 Web 角色项目中的 *Properties\\AssemblyInfo.cs* 进行拉取。你可以建立一个部署工作流，这样当你每次将相关内容发布到 Azure 时，程序集版本就会递增一次。你也可以使用通配符“*”直接修改项目中的 *Properties\\AssemblyInfo.cs*，以便每次进行构建时让版本字符串自动递增。例如：

			[assembly: AssemblyVersion("1.0.0.*")]
	
		在这里，你可以使用任何其他的策略来简化部署过程中唯一字符串的生成。

2. 重新发布云服务并访问主页。
 
3. 查看页面的 HTML 代码。每次重新发布对云服务的更改时，你都应该能够看到所呈现的 CDN URL，其中包含唯一版本字符串。例如：

		...
		
		<link href="http://az632148.vo.msecnd.net/Content/css?v=1.0.0.25449" rel="stylesheet"/>
		
		<script src="http://az632148.vo.msecnd.net/bundles/modernizer?v=1.0.0.25449"></script>
		
		...
		
		<script src="http://az632148.vo.msecnd.net/bundles/jquery?v=1.0.0.25449"></script>
		
		<script src="http://az632148.vo.msecnd.net/bundles/bootstrap?v=1.0.0.25449"></script>
		
		...

4. 在 Visual Studio 中，键入 `F5` 即可调试云服务。

5. 查看页面的 HTML 代码。你仍然会看到每个脚本文件独立呈现，因此，你在 Visual Studio 中的调试体验是一致的。

		...
		
		    <link href="/Content/bootstrap.css" rel="stylesheet"/>
		<link href="/Content/site.css" rel="stylesheet"/>
		
		    <script src="/Scripts/modernizr-2.6.2.js"></script>
		
		...
		
		    <script src="/Scripts/jquery-1.10.2.js"></script>
		
		    <script src="/Scripts/bootstrap.js"></script>
		<script src="/Scripts/respond.js"></script>
		
		...   

<a name="fallback"></a>
## CDN URL 的回退机制 ##

你希望你的网页在 Azure CDN 终结点因某种原因而出现故障时，能够表现出相当的智能，即能够访问作为回退选项的源 Web 服务器，以便加载 JavaScript 或 Bootstrap。因 CDN 不可用而丢失网站上的图像是很严重的问题，但更为严重的是失去脚本和样式表提供的重要页面功能。

[捆绑包](http://msdn.microsoft.com/zh-cn/library/system.web.optimization.bundle.aspx)类包含一个名为 [CdnFallbackExpression](http://msdn.microsoft.com/zh-cn/library/system.web.optimization.bundle.cdnfallbackexpression.aspx) 的属性，该属性可以让你配置回退机制以应对 CDN 故障情况。若要使用此属性，请执行以下步骤：

1. 在 Web 角色项目中打开 *App\_Start\\BundleConfig.cs*（你已在其中将 CDN URL 添加到了每个[捆绑包构造函数](http://msdn.microsoft.com/zh-cn/library/jj646464.aspx)），然后进行以下突出显示的更改，以便将回退机制添加到默认捆绑包中。  

		public static void RegisterBundles(BundleCollection bundles)
		{
		    var version = System.Reflection.Assembly.GetAssembly(typeof(BundleConfig))
		        .GetName().Version.ToString();
		    var cdnUrl = ";http://cdnurl.vo.msecnd.net/.../{0}?"; + version;
		    bundles.UseCdn = true;
		
		    bundles.Add(new ScriptBundle(";~/bundles/jquery";, string.Format(cdnUrl, ";bundles/jquery";)) 
						<mark>{ CdnFallbackExpression = ";window.jquery"; }</mark>
		                .Include(";~/Scripts/jquery-{version}.js";));
		
		    bundles.Add(new ScriptBundle(";~/bundles/jqueryval";, string.Format(cdnUrl, ";bundles/jqueryval";)) 
						<mark>{ CdnFallbackExpression = ";$.validator"; }</mark>
		            	.Include(";~/Scripts/jquery.validate*";));
		
		    // Use the development version of Modernizr to develop with and learn from. Then, when you're
		    // ready for production, use the build tool at http://modernizr.com to pick only the tests you need.
		    bundles.Add(new ScriptBundle(";~/bundles/modernizr";, string.Format(cdnUrl, ";bundles/modernizer";)) 
						<mark>{ CdnFallbackExpression = ";window.Modernizr"; }</mark>
						.Include(";~/Scripts/modernizr-*";));
		
		    bundles.Add(new ScriptBundle(";~/bundles/bootstrap";, string.Format(cdnUrl, ";bundles/bootstrap";)) 	
						<mark>{ CdnFallbackExpression = ";$.fn.modal"; }</mark>
		        		.Include(
			              		";~/Scripts/bootstrap.js";,
			              		";~/Scripts/respond.js";));
		
		    bundles.Add(new StyleBundle(";~/Content/css";, string.Format(cdnUrl, ";Content/css";)).Include(
		                ";~/Content/bootstrap.css";,
		                ";~/Content/site.css";));
		}

	当 `CdnFallbackExpression` 不为 null 时，会将脚本注入 HTML 中以测试捆绑包是否已成功加载，而如果没有成功加载，则会直接从源 Web 服务器访问捆绑包。需要将此属性设置成一个 JavaScript 表达式，以便测试相应的 CDN 捆绑包是否已正确加载。根据内容的不同，测试每个捆绑包所需的表达式也会有所不同。对于上面的默认捆绑包：
	
	-	`window.jquery` 在 jquery-{version}.js 中定义
	-	`$.validator` 在 jquery.validate.js 中定义
	-	`window.Modernizr` 在 modernizer-{version}.js 中定义
	-	`$.fn.modal` 在 bootstrap.js 中定义
	
	你可能已注意到，我并没有为 `~/Cointent/css` 捆绑包设置 CdnFallbackExpression。这是因为，目前 [System.Web.Optimization 中的 Bug](https://aspnetoptimization.codeplex.com/workitem/104) 会针对回退 CSS 注入 `<script>` 标记而非预期的 `<link>` 标记。
	
	不过，你可以使用一个不错的[样式捆绑包回退](https://github.com/EmberConsultingGroup/StyleBundleFallback)，是由 [Ember Consulting Group](https://github.com/EmberConsultingGroup) 提供的。

2. 若要将此解决方法用于 CSS，可在 Web 角色项目的 *App\_Start* 文件夹中创建一个新的名为 *StyleBundleExtensions.cs* 的 .cs 文件，然后将其内容替换为 [GitHub 提供的代码](https://github.com/EmberConsultingGroup/StyleBundleFallback/blob/master/Website/App_Start/StyleBundleExtensions.cs)。

3. 在 *App\_Start\\StyleFundleExtensions.cs* 中，将命名空间重命名为你的 Web 角色的名称（例如 **WebRole1**）。

4. 回到 `App_Start\BundleConfig.cs`，使用以下突出显示的代码对最后一个 `bundles.Add` 语句进行修改：

		bundles.Add(new StyleBundle("~/Content/css", string.Format(cdnUrl, "Content/css"))
		    <mark>.IncludeFallback("~/Content/css", "sr-only", "width", "1px")</mark>
		    .Include(
		          "~/Content/bootstrap.css",
		          "~/Content/site.css"));

	这个新的扩展方法使用相同的机制将脚本注入 HTML 中，以便查看 DOM 中是否存在匹配的类名、规则名和规则值（在 CSS 捆绑包中定义），如果没有找到匹配项，则会回退到源 Web 服务器。

5. 重新发布云服务并访问主页。

6. 查看页面的 HTML 代码。你会发现如下所示的已注入脚本：

		...
		
	    <link href="http://az632148.vo.msecnd.net/Content/css?v=1.0.0.25474" rel="stylesheet"/>
		<script>(function() {
		                var loadFallback,
		                    len = document.styleSheets.length;
		                for (var i = 0; i < len; i++) {
		                    var sheet = document.styleSheets[i];
		                    if (sheet.href.indexOf('http://az632148.vo.msecnd.net/Content/css?v=1.0.0.25474') !== -1) {
		                        var meta = document.createElement('meta');
		                        meta.className = 'sr-only';
		                        document.head.appendChild(meta);
		                        var value = window.getComputedStyle(meta).getPropertyValue('width');
		                        document.head.removeChild(meta);
		                        if (value !== '1px') {
		                            document.write('<link href="/Content/css" rel="stylesheet" type="text/css" />');
		                        }
		                    }
		                }
		                return true;
		            }())||document.write('<script src="/Content/css"><\/script>');</script>
		
		    <script src="http://az632148.vo.msecnd.net/bundles/modernizer?v=1.0.0.25474"></script>
		<script>(window.Modernizr)||document.write('<script src="/bundles/modernizr"><\/script>');</script>
		
		... 
		
		    <script src="http://az632148.vo.msecnd.net/bundles/jquery?v=1.0.0.25474"></script>
		<script>(window.jquery)||document.write('<script src="/bundles/jquery"><\/script>');</script>
		
		    <script src="http://az632148.vo.msecnd.net/bundles/bootstrap?v=1.0.0.25474"></script>
		<script>($.fn.modal)||document.write('<script src="/bundles/bootstrap"><\/script>');</script>
		
		...


	请注意，CSS 捆绑包的注入脚本仍包含以下行中 `CdnFallbackExpression` 属性的残存错误：

        }())||document.write('<script src="/Content/css"><\/script>');</script>

	不过，由于 || 表达式的第一部分始终会返回 true（在紧邻其上的行中），因此始终不会运行 document.write() 函数。

## 更多信息 ##
- [Azure 内容交付网络 (CDN) 概述](http://msdn.microsoft.com/zh-cn/library/azure/ff919703.aspx)
- [在 Web 应用程序中从 Azure CDN 提供内容](/documentation/articles/cdn-serve-content-from-cdn-in-your-web-application)
- [将 Azure 网站与 Azure CDN 集成](/documentation/articles/cdn-websites-with-cdn)
- [ASP.NET 绑定和缩减](http://www.asp.net/mvc/tutorials/mvc-4/bundling-and-minification)
- [使用 Azure CDN](/documentation/articles/cdn-how-to-use)

<!---HONumber=74-->