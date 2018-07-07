﻿---
title: '负载平衡: Exchange 2013 Help'
TOCTitle: 负载平衡
ms:assetid: f572c193-6f3a-400e-9085-a9d3e5e18c59
ms:mtpsurl: https://technet.microsoft.com/zh-cn/library/JJ898588(v=EXCHG.150)
ms:contentKeyID: 51408293
ms.date: 01/11/2018
mtps_version: v=EXCHG.150
ms.translationtype: HT
---

# 负载平衡

 

_**适用于：**Exchange Server 2013_

_**上一次修改主题：**2016-12-09_

负载平衡是一种管理由哪些服务器接收通信的方法。负载平衡有助于通过各终结点（例如客户端访问服务器）分发传入客户端连接，以确保没有哪个终结点承担不成比例的份额。负载平衡还可以在一个或多个终结点出现故障时，提供故障转移冗余。通过在 Exchange Server 2013 中使用负载均衡，可以确保用户在计算机出现故障时能够继续接收 Exchange 服务。负载平衡还使您的部署能够处理比一台服务器能够处理的通信还多的通信，同时为客户端提供单一主机名。

负载平衡有两种主要用途。当您的一个 Active Directory 站点中的某个客户端访问服务器出现故障时，负载平衡可以降低该故障造成的影响。此外，负载平衡可确保每个客户端访问服务器上的负载分配均匀。

Exchange 2013 还包括以下用于切换和故障转移冗余的解决方案：

  - **高可用性**Exchange 2013 使用数据库可用性组 (DAG) 保留同步的不同服务器上邮箱中的多个副本。这样，如果一台服务器上的邮箱数据库出现故障，用户可以与另一服务器上已同步的数据库副本连接。

  - **站点恢复** 您可以在不同的地理位置部署两个 Active Directory 站点，使两者之间的邮箱数据保持同步，并且让一个站点在另一个站点出现故障后负担所有负载。

  - **联机邮箱移动**   在联机邮箱移动中，用户在移动过程中可以访问他们的电子邮件帐户。只有在过程结束时（发生最终同步时），才会短暂锁定用户帐户。您可以跨林或在同一林中执行联机邮箱移动。

  - **卷影冗余**   卷影冗余保护邮件在传输过程中的可用性和可恢复性。使用卷影冗余时，如果要从传输数据库中删除邮件，将延迟删除操作，直至传输服务器确认针对该邮件的所有下一跃点均已完成。如果在报告成功传递之前有任何下一跃点失败，则会重新提交该邮件，以便传递到未完成的跃点。

## Exchange Server 2013 负载平衡中的体系结构更改

在 Exchange Server 2010 中，客户端连接和处理由客户端访问服务器角色来进行。这需要外部和内部 Outlook 连接以及移动设备和第三方客户端连接跨部署中的客户端访问服务器阵列取得负载平衡，以实现服务器的容错功能和有效利用。很多 Exchange 2010 客户端访问协议都需要相关性，这是客户端和特定客户端访问服务器之间的一种关系。具体而言，Outlook Web App、Exchange 控制面板、Exchange Web 服务、Outlook 无处不在、Outlook TCP/IP MAPI 连接、Exchange ActiveSync、Exchange 通讯簿服务以及远程 PowerShell 都需要或受益于客户端到客户端访问服务器相关性。Exchange 2010 中的负载平衡包含以下选项：

  - 具有源 IP 相关性的 Windows 网络负载平衡

  - 硬件负载平衡

由于 Exchange 2010 中客户端协议的需求不同，建议使用第 7 层负载平衡解决方案。第 7 层也称为应用层负载平衡，假定客户端和服务器之间的整个会话都可用于负载平衡器逻辑，允许负载平衡解决方案使用复杂的规则，以确定如何平衡进入系统的每个请求。这些复杂的规则可确保来自特定客户端的所有请求到达同一个客户端访问服务器终结点。在 Exchange 2010 中，如果来自特定客户端的所有请求因需要相关性的协议而无法到达同一终结点，则用户体验会受到负面影响。有关 Exchange 2010 负载平衡选项的详细信息，请参阅[了解 Exchange 2010 中的负载平衡](https://go.microsoft.com/fwlink/p/?linkid=196447)。

在 Exchange Server 2013 中，主要有两种类型的服务器——客户端访问服务器和邮箱服务器。Exchange 2013 中的客户端访问服务器为轻量型无状态代理服务器，允许客户端连接到 Exchange 2013 邮箱服务器。Exchange 2013 客户端访问服务器提供了统一的命名空间和身份验证。此外，Exchange 2013 客户端访问服务器还可以：

  - 支持客户端协议的代理和重定向。

  - 支持使用第 4 层负载平衡。

通过会话相关性和第 7 层负载平衡，客户端和服务器之间的所有请求根据各种协议的要求发送到同一终结点。请求在应用层进行分发。通过第 4 层负载平衡，请求可在传输层进行分发。负载平衡解决方案分发来自客户端的请求，它可感知执行工作的一组服务器的单个 IP 地址（有时也称为虚拟 IP 地址或 VIP）。客户端和服务器之间的连接必须在确定请求内容之前建立，以使负载平衡器先选择一个服务器来接收请求，再检查请求内容。目标服务器的选择可通过多种方式实现，如“轮询机制”，其中每个入站连接会到达循环列表中的下一个目标服务器；或“最少连接”，其中负载平衡器会向当时建立了最少连接的服务器发送每个新的连接。由于不再需要会话相关性，对于部署的负载平衡体系结构，您可拥有更多的灵活性、选择性和简单性。通过无会话相关性的负载平衡，您可以增加负载平衡器的容量和利用率，这是因为处理无需用于维护更复杂的相关性选项，如基于 cookie 的负载平衡或安全套接字层 (SSL) 会话 ID。

## 客户端访问服务器阵列和 Exchange 2013

在 Exchange 2010 中，我们引入了客户端访问阵列的概念。为 Active Directory 站点配置客户端访问阵列后，站点中的所有客户端访问服务器会自动成为阵列的成员。在 Exchange 2013 的当前内部版本中，不需要进行客户端访问阵列的配置，因为经过负载平衡且高度可用的服务部署要简单得多。

## 负载平衡解决方案

Exchange 2013 仍然支持使用硬件负载平衡器。有关已在 Exchange 2010 中完成解决方案测试并将可能在 Exchange 2013 中达到相同效果的硬件负载平衡解决方案的详细信息，请参阅 [Exchange Server 2010 负载平衡器部署](https://go.microsoft.com/fwlink/p/?linkid=261834)。请记住，该页面会显示 Exchange 2010 中硬件负载平衡器的更复杂的第 7 层配置。假如出现本主题前面所述的体系结构变化，实现 Exchange 2013 通信负载平衡可能会更加简单。不同于为每个 Exchange 协议配置会话相关性，到 Exchange 2013 客户端访问服务器的入站连接可以通过负载平衡器直接到达可用服务器，无需进行额外的相关性处理。硬件负载平衡器在提供 Exchange 服务的高可用性方面仍然具有重要作用，因为它可以检测特定客户端访问服务器何时不可用，并将其从要处理入站连接的一组服务器中删除。

## Windows 网络负载平衡

Windows 网络负载平衡 (WNLB) 是用于 Exchange 服务器的常见软件负载平衡器。为 Microsoft Exchange 部署 WNLB 时有一些限制。

  - 无法在使用邮箱 DAG 的 Exchange 服务器上使用 WNLB，因为 WNLB 与 Windows 故障转移群集不兼容。如果您现在使用的是 Exchange 2013 DAG，但又想使用 WNLB，则需要在不同的服务器上运行客户端访问服务器角色和邮箱服务器角色。

  - WNLB 不会检测服务中断。WNLB 只按 IP 地址检测服务器中断。这就是说，如果一个特定 Web 服务（如 Outlook Web App）出现故障，但服务器仍在运行，则 WNLB 不会检测到该故障，并且仍会将请求路由到该客户端访问服务器。需要手动干预来从负载平衡池中删除出现中断的客户端访问服务器。

  - 使用 WNLB 会导致端口泛滥，从而导致网络崩溃。

  - 由于 WNLB 仅使用源 IP 地址执行客户端相关性，因此当源 IP 池较小时，这不是有效的解决方案。当源 IP 池来自远程网络子网或组织在使用网络地址转换时，可能会发生这种情况。
