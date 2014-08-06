﻿## <a name="what-is"> </a>什么是队列存储？

Windows Azure 队列存储是一项可存储大量消息的服务，
用户可以通过经验证的呼叫，使用 HTTP 或 HTTPS 从
世界任何地方访问这些消息。一条队列消息的大小可达 64 KB，
一个队列中可以包含数百万条消息，而一个存储帐户的总容量限值高达 100 TB。对于 2012 年 6 月 8 日之后创建的存储帐户，总容量为 200TB；对于此日期之前创建的存储帐户，总容量为 100TB。有关存储帐户容量的详细信息，请参见 [Windows Azure 存储可伸缩性和性能目标](http://msdn.microsoft.com/zh-cn/library/dn249410.aspx)。

队列存储的常见用途包括：

-   <span>创建积压工作以进行异步处理</span>
-   将消息从 Windows Azure Web 角色传递到 Windows Azure 辅助角色

## <a name="concepts"> </a>概念

队列服务包含以下组件：

![队列 1](./media/howto-queue-storage/queue1.png)


- **URL 格式：**可使用以下 URL 格式对队列寻址：
	http://`<storage account>`.queue.core.windows.net/`<queue>` 
      
可使用以下 URL 对图中的某个队列寻址：
	http://myaccount.queue.core.windows.net/imagesToDownload

- **存储帐户**：对 Windows Azure 存储服务的所有访问都要通过存储帐户完成。有关存储帐户容量的详细信息，请参见 [Windows Azure 存储可伸缩性和性能目标](http://msdn.microsoft.com/zh-cn/library/dn249410.aspx)。

- **队列：**一个队列包含一组消息。所有消息必须位于相应的队列中。

- **消息：** 一条消息（不管采用何种格式）的最大大小为 64 KB。


