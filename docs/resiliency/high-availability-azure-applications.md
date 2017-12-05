---
title: "Azure 应用程序的高可用性"
description: "有关在 Microsoft Azure 上设计和构建应用程序以实现高可用性的技术概述与深入信息。"
author: adamglick
ms.date: 05/31/2017
ms.openlocfilehash: 46b7b802326a8de03546528aaeb1a1c6419d41db
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
[!INCLUDE [header](../_includes/header.md)]
# <a name="high-availability-for-applications-built-on-microsoft-azure"></a><span data-ttu-id="5db52-103">构建在 Microsoft Azure 基础之上的应用程序高可用性</span><span class="sxs-lookup"><span data-stu-id="5db52-103">High availability for applications built on Microsoft Azure</span></span>
<span data-ttu-id="5db52-104">高度可用的应用程序可消减依赖的服务和硬件在可用性、负载和临时故障方面的波动。</span><span class="sxs-lookup"><span data-stu-id="5db52-104">A highly available application absorbs fluctuations in availability, load, and temporary failures in dependent services and hardware.</span></span> <span data-ttu-id="5db52-105">应用程序以可接受的性能继续运行，如业务要求或应用程序服务级别协议 (SLA) 所定义。</span><span class="sxs-lookup"><span data-stu-id="5db52-105">The application continues to perform acceptably, as defined by business requirements or application service-level agreements (SLAs).</span></span>

## <a name="azure-high-availability-features"></a><span data-ttu-id="5db52-106">Azure 高可用性功能</span><span class="sxs-lookup"><span data-stu-id="5db52-106">Azure high-availability features</span></span>
<span data-ttu-id="5db52-107">Azure 在平台中内置了多种功能，用于支持高度可用的应用程序。</span><span class="sxs-lookup"><span data-stu-id="5db52-107">Azure has many built-in platform features that support highly available applications.</span></span> <span data-ttu-id="5db52-108">本部分介绍其中一些重要功能。</span><span class="sxs-lookup"><span data-stu-id="5db52-108">This section describes some of those key features.</span></span>

### <a name="fabric-controller"></a><span data-ttu-id="5db52-109">结构控制器</span><span class="sxs-lookup"><span data-stu-id="5db52-109">Fabric controller</span></span>
<span data-ttu-id="5db52-110">Azure 结构控制器负责预配和监视 Azure 计算实例的状况。</span><span class="sxs-lookup"><span data-stu-id="5db52-110">The Azure fabric controller provisions and monitors the condition of Azure compute instances.</span></span> <span data-ttu-id="5db52-111">结构控制器监视主机和来宾计算机实例的硬件和软件的状态。</span><span class="sxs-lookup"><span data-stu-id="5db52-111">The fabric controller monitors the status of the hardware and software of the host and guest machine instances.</span></span> <span data-ttu-id="5db52-112">检测到故障后，它通过自动重新定位 VM 实例来维持 SLA。</span><span class="sxs-lookup"><span data-stu-id="5db52-112">When it detects a failure, it maintains SLAs by automatically relocating the VM instances.</span></span> <span data-ttu-id="5db52-113">容错域和升级域的概念进一步为计算 SLA 提供支持。</span><span class="sxs-lookup"><span data-stu-id="5db52-113">The concept of fault and upgrade domains further supports the compute SLA.</span></span>

<span data-ttu-id="5db52-114">部署多个云服务角色实例时，Azure 将这些实例部署到不同的容错域。</span><span class="sxs-lookup"><span data-stu-id="5db52-114">When multiple Cloud Service role instances are deployed, Azure deploys these instances to different fault domains.</span></span> <span data-ttu-id="5db52-115">容错域边界本质上就是同一区域内的其他硬件机架。</span><span class="sxs-lookup"><span data-stu-id="5db52-115">A fault domain boundary is essentially a different hardware rack in the same region.</span></span> <span data-ttu-id="5db52-116">容错域可降低局部硬件故障将中断应用程序服务的可能性。</span><span class="sxs-lookup"><span data-stu-id="5db52-116">Fault domains reduce the probability that a localized hardware failure interrupts the service of an application.</span></span> <span data-ttu-id="5db52-117">无法管理辅助角色或 Web 角色的容错域的数量。</span><span class="sxs-lookup"><span data-stu-id="5db52-117">You cannot manage the number of fault domains of your worker roles or web roles.</span></span> <span data-ttu-id="5db52-118">结构控制器采用与 Azure 托管的应用程序分离的专用资源。</span><span class="sxs-lookup"><span data-stu-id="5db52-118">The fabric controller uses dedicated resources that are separate from Azure-hosted applications.</span></span> <span data-ttu-id="5db52-119">由于它充当 Azure 系统的核心，因此其运行时间需要是 100%。</span><span class="sxs-lookup"><span data-stu-id="5db52-119">It requires 100 percent uptime because it serves as the nucleus of the Azure system.</span></span> <span data-ttu-id="5db52-120">它监视和管理各个容错域中的角色实例。</span><span class="sxs-lookup"><span data-stu-id="5db52-120">It monitors and manages role instances across fault domains.</span></span>

<span data-ttu-id="5db52-121">以下示意图显示了结构控制器在各个容错域中部署和管理的 Azure 共享资源。</span><span class="sxs-lookup"><span data-stu-id="5db52-121">The following diagram shows Azure shared resources that the fabric controller deploys and manages across different fault domains.</span></span>

![容错域隔离的简化视图](./images/high-availability-azure-applications/fault-domain-isolation.png)

<span data-ttu-id="5db52-123">虽然容错域是用于缓解故障的物理分隔，但升级域是实例分隔的逻辑单元，它们决定了将在特定的时间升级服务的哪些实例。</span><span class="sxs-lookup"><span data-stu-id="5db52-123">While fault domains are physical separations to mitigate failure, upgrade domains are logical units of instance separation that determine which instances of a service will be upgraded at a specific time.</span></span> <span data-ttu-id="5db52-124">默认情况下会为托管服务部署定义五个升级域。</span><span class="sxs-lookup"><span data-stu-id="5db52-124">By default, five upgrade domains are defined for your hosted service deployment.</span></span> <span data-ttu-id="5db52-125">不过，可以在服务定义文件中更改该值。</span><span class="sxs-lookup"><span data-stu-id="5db52-125">However, you can change that value in the service definition file.</span></span> <span data-ttu-id="5db52-126">例如，如果你的 web 角色有八个实例，则三个升级域中分别有两个实例，一个升级域中有两个实例。</span><span class="sxs-lookup"><span data-stu-id="5db52-126">For example, if you have eight instances of your web role, there are two instances in three upgrade domains and two instances in one upgrade domain.</span></span> <span data-ttu-id="5db52-127">Azure 基于升级域的数量定义更新序列。</span><span class="sxs-lookup"><span data-stu-id="5db52-127">Azure defines the update sequence based on the number of upgrade domains.</span></span> <span data-ttu-id="5db52-128">有关详细信息，请参阅[更新云服务](/azure/cloud-services/cloud-services-update-azure-service/)。</span><span class="sxs-lookup"><span data-stu-id="5db52-128">For more information, see [Update a cloud service](/azure/cloud-services/cloud-services-update-azure-service/).</span></span>

### <a name="features-in-other-services"></a><span data-ttu-id="5db52-129">其他服务中的功能</span><span class="sxs-lookup"><span data-stu-id="5db52-129">Features in other services</span></span>
<span data-ttu-id="5db52-130">除了这些支持计算资源高可用性的平台功能之外，Azure 还将高可用性功能嵌入到它的其他服务中。</span><span class="sxs-lookup"><span data-stu-id="5db52-130">In addition to the platform features that support high availability of compute resources, Azure embeds high-availability features in its other services.</span></span> <span data-ttu-id="5db52-131">例如，Azure 存储会维护 Azure 存储帐户中所有数据的至少三个副本。</span><span class="sxs-lookup"><span data-stu-id="5db52-131">For example, Azure Storage maintains at least three replicas of all data in your Azure storage account.</span></span> <span data-ttu-id="5db52-132">它还启用了异地复制来将数据副本存储在次要区域中。</span><span class="sxs-lookup"><span data-stu-id="5db52-132">It also enables geo-replication to store copies of your data in a secondary region.</span></span> <span data-ttu-id="5db52-133">通过 Azure 内容交付网络，可在世界各地缓存 Blob 以实现冗余、可伸缩性和更低的延迟。</span><span class="sxs-lookup"><span data-stu-id="5db52-133">The Azure Content Delivery Network allows blobs to be cached around the world for redundancy, scalability, and lower latency.</span></span> <span data-ttu-id="5db52-134">Azure SQL 数据库也保留多个副本。</span><span class="sxs-lookup"><span data-stu-id="5db52-134">Azure SQL Database maintains multiple replicas as well.</span></span>

<span data-ttu-id="5db52-135">有关 Azure 平台可用性功能的更深入讨论，请参阅[复原技术指南](index.md)。</span><span class="sxs-lookup"><span data-stu-id="5db52-135">For a deeper discussion of Azure platform availability features, see [Resiliency technical guidance](index.md).</span></span> <span data-ttu-id="5db52-136">另请参阅 [Best practices for designing large-scale services on Windows Azure](https://azure.microsoft.com/blog/best-practices-for-designing-large-scale-services-on-windows-azure/)（在 Windows Azure 上设计大型服务的最佳做法）。</span><span class="sxs-lookup"><span data-stu-id="5db52-136">Also see [Best practices for designing large-scale services on Windows Azure](https://azure.microsoft.com/blog/best-practices-for-designing-large-scale-services-on-windows-azure/).</span></span>

<span data-ttu-id="5db52-137">尽管 Azure 提供多项支持高可用性的功能，但了解其限制也很重要：</span><span class="sxs-lookup"><span data-stu-id="5db52-137">Although Azure provides multiple features that support high availability, it's important to understand their limitations:</span></span>

* <span data-ttu-id="5db52-138">对于计算，Azure 保证角色可用并正在运行，但它无法检测应用程序是否正在运行或过载。</span><span class="sxs-lookup"><span data-stu-id="5db52-138">For compute, Azure guarantees that your roles are available and running, but it cannot detect whether your application is running or overloaded.</span></span>
* <span data-ttu-id="5db52-139">对于 Azure SQL 数据库，在区域内同步复制数据。</span><span class="sxs-lookup"><span data-stu-id="5db52-139">For Azure SQL Database, data is replicated synchronously within the region.</span></span> <span data-ttu-id="5db52-140">可以选择活动异地复制，以便在同一个区域（或不同的区域中）最多创建四个附加的数据库副本。</span><span class="sxs-lookup"><span data-stu-id="5db52-140">You can choose active geo-replication, which allows up to four additional database copies in the same region (or different regions).</span></span> <span data-ttu-id="5db52-141">虽然这些数据库副本不是时间点备份，但 SQL 数据库提供了时间点备份功能。</span><span class="sxs-lookup"><span data-stu-id="5db52-141">While these database replicas are not point-in-time backups, SQL Database does provide point-in-time backup capabilities.</span></span> <span data-ttu-id="5db52-142">有关详细信息，请参阅[使用自动化数据备份恢复 Azure SQL 数据库：时间点还原](/azure/sql-database/sql-database-recovery-using-backups#point-in-time-restore)。</span><span class="sxs-lookup"><span data-stu-id="5db52-142">For more information, see [Recover an Azure SQL Database using automated data backups: Point-in-time restore](/azure/sql-database/sql-database-recovery-using-backups#point-in-time-restore).</span></span>
* <span data-ttu-id="5db52-143">对于 Azure 存储，默认情况下会将表数据和 Blob 数据复制到备用区域。</span><span class="sxs-lookup"><span data-stu-id="5db52-143">For Azure Storage, table data and blob data are replicated by default to an alternate region.</span></span> <span data-ttu-id="5db52-144">但是，直到 Microsoft 选择将故障转移到备用站点，才能访问这些副本。</span><span class="sxs-lookup"><span data-stu-id="5db52-144">However, you cannot access the replicas until Microsoft chooses to fail over to the alternate site.</span></span> <span data-ttu-id="5db52-145">通常只有在发生区域范围的长时间服务中断时才执行区域故障转移，并且不存在针对异地故障转移时间的 SLA。</span><span class="sxs-lookup"><span data-stu-id="5db52-145">A region failover occurs only during a prolonged region-wide service disruption, and there is no SLA for geo-failover time.</span></span> <span data-ttu-id="5db52-146">另外，必须注意，任何数据损坏将迅速扩散到副本。</span><span class="sxs-lookup"><span data-stu-id="5db52-146">It's also important to note that any data corruption quickly spreads to the replicas.</span></span> <span data-ttu-id="5db52-147">由于这些原因，你必须使用特定于应用程序的可用性功能补充平台可用性功能，包括用来创建 blob 数据的时间点备份的 blob 快照功能。</span><span class="sxs-lookup"><span data-stu-id="5db52-147">For these reasons, you must supplement platform availability features with application-specific availability features, including the blob snapshot feature to create point-in-time backups of blob data.</span></span>

### <a name="availability-sets-for-azure-virtual-machines"></a><span data-ttu-id="5db52-148">Azure 虚拟机的可用性集</span><span class="sxs-lookup"><span data-stu-id="5db52-148">Availability sets for Azure Virtual Machines</span></span>
<span data-ttu-id="5db52-149">本文档重点介绍了云服务，这些服务使用平台即服务 (PaaS) 模型。</span><span class="sxs-lookup"><span data-stu-id="5db52-149">This document primarily focuses on cloud services, which use a platform-as-a-service (PaaS) model.</span></span> <span data-ttu-id="5db52-150">Azure 虚拟机也有一些特定的可用性功能，它们使用基础结构即服务 (IaaS) 模型。</span><span class="sxs-lookup"><span data-stu-id="5db52-150">There are also specific availability features for Azure Virtual Machines, which use an infrastructure-as-a-service (IaaS) model.</span></span> <span data-ttu-id="5db52-151">若要实现虚拟机的高可用性，必须使用可用性集，它们提供与容错域和升级域相似的功能。</span><span class="sxs-lookup"><span data-stu-id="5db52-151">To achieve high availability with Virtual Machines, you must use availability sets, which serve a similar function to fault and upgrade domains.</span></span> <span data-ttu-id="5db52-152">在可用性集中，Azure 会以某种方式定位虚拟机，避免局部硬件错误和维护活动导致该组的所有虚拟机停机。</span><span class="sxs-lookup"><span data-stu-id="5db52-152">Within an availability set, Azure positions the virtual machines in a way that prevents localized hardware faults and maintenance activities from bringing down all the machines in that group.</span></span> <span data-ttu-id="5db52-153">可用性集必须达到针对虚拟机可用性的 Azure SLA。</span><span class="sxs-lookup"><span data-stu-id="5db52-153">Availability sets are required to achieve the Azure SLA for the availability of Virtual Machines.</span></span>

<span data-ttu-id="5db52-154">下图显示了分别适用于 Web 和 SQL Server 虚拟机的两个可用性集。</span><span class="sxs-lookup"><span data-stu-id="5db52-154">The following diagram shows two availability sets for web and SQL Server virtual machines, respectively.</span></span>

![Azure 虚拟机的可用性集](./images/high-availability-azure-applications/availability-set-for-azure-virtual-machines.png)

> [!NOTE]
> <span data-ttu-id="5db52-156">在上图中，SQL Server 安装并运行在虚拟机上。</span><span class="sxs-lookup"><span data-stu-id="5db52-156">In the preceding diagram, SQL Server is installed and running on virtual machines.</span></span> <span data-ttu-id="5db52-157">这不同于 Azure SQL 数据库，后者将数据库作为托管服务提供。</span><span class="sxs-lookup"><span data-stu-id="5db52-157">This is different from Azure SQL Database, which provides a database as a managed service.</span></span>
> 
> 

## <a name="application-strategies-for-high-availability"></a><span data-ttu-id="5db52-158">高可用性的应用程序策略</span><span class="sxs-lookup"><span data-stu-id="5db52-158">Application strategies for high availability</span></span>
<span data-ttu-id="5db52-159">高可用性的大多数应用程序策略均涉及冗余或消除应用程序组件之间的硬性依赖。</span><span class="sxs-lookup"><span data-stu-id="5db52-159">Most application strategies for high availability involve either redundancy or the removal of hard dependencies between application components.</span></span> <span data-ttu-id="5db52-160">应用程序设计应在 Azure 或第三方服务偶发的停机时间内支持容错。</span><span class="sxs-lookup"><span data-stu-id="5db52-160">Application design should support fault tolerance during sporadic downtime of Azure or third-party services.</span></span> <span data-ttu-id="5db52-161">以下各部分介绍了几种用于提高云服务可用性的应用程序模式。</span><span class="sxs-lookup"><span data-stu-id="5db52-161">The following sections describe application patterns for improving the availability of your cloud services.</span></span>

### <a name="asynchronous-communication-and-durable-queues"></a><span data-ttu-id="5db52-162">异步通信和持久队列</span><span class="sxs-lookup"><span data-stu-id="5db52-162">Asynchronous communication and durable queues</span></span>
<span data-ttu-id="5db52-163">若要提高 Azure 应用程序的可用性，请考虑在松散耦合的服务之间使用异步通信。</span><span class="sxs-lookup"><span data-stu-id="5db52-163">To increase availability in Azure applications, consider asynchronous communication between loosely coupled services.</span></span> <span data-ttu-id="5db52-164">在此模式下，会将消息写入到存储队列或 Azure 服务总线队列供以后处理。</span><span class="sxs-lookup"><span data-stu-id="5db52-164">In this pattern, messages are written to either storage queues or Azure Service Bus queues for later processing.</span></span> <span data-ttu-id="5db52-165">将消息写入到队列后，控制权会立即归还给发送者。</span><span class="sxs-lookup"><span data-stu-id="5db52-165">When a message is written to the queue, control immediately returns to the sender.</span></span> <span data-ttu-id="5db52-166">应用程序的另一项服务（通常实现为辅助角色）对消息进行处理。</span><span class="sxs-lookup"><span data-stu-id="5db52-166">Another service of the application (typically implemented as a worker role) processes the message.</span></span> <span data-ttu-id="5db52-167">如果处理服务停止工作，则消息会累积在队列中，直到处理服务得以还原。</span><span class="sxs-lookup"><span data-stu-id="5db52-167">If the processing service stops working, the messages accumulate in the queue until the processing service is restored.</span></span> <span data-ttu-id="5db52-168">前端发送者与消息处理程序之间不存在直接的依赖关系。</span><span class="sxs-lookup"><span data-stu-id="5db52-168">There is no direct dependency between the front-end sender and the message processor.</span></span> <span data-ttu-id="5db52-169">这消除了在分布式应用程序中可能会导致瓶颈的同步服务调用。</span><span class="sxs-lookup"><span data-stu-id="5db52-169">This eliminates synchronous service calls that can cause bottlenecks in distributed applications.</span></span>

<span data-ttu-id="5db52-170">此模式的一种变体在 Azure 存储（Blob、表或队列）或服务总线队列中存储关于失败的数据库调用的信息。</span><span class="sxs-lookup"><span data-stu-id="5db52-170">A variation of this pattern stores information about failed database calls in Azure Storage (blobs, tables, or queues) or Service Bus queues.</span></span> <span data-ttu-id="5db52-171">例如，在应用程序中同步调用另一个服务（如 Azure SQL 数据库）反复失败。</span><span class="sxs-lookup"><span data-stu-id="5db52-171">For example, a synchronous call within an application to another service (such as Azure SQL Database) fails repeatedly.</span></span> <span data-ttu-id="5db52-172">可以将该请求以序列化方式存储到持久存储中。</span><span class="sxs-lookup"><span data-stu-id="5db52-172">You might be able to serialize that request into durable storage.</span></span> <span data-ttu-id="5db52-173">以后在服务或数据库恢复联机时，应用程序可重新提交来自存储的请求。</span><span class="sxs-lookup"><span data-stu-id="5db52-173">At some later point when the service or database is back online, the application can resubmit the request from storage.</span></span> <span data-ttu-id="5db52-174">此模型中的区别在于中间位置仅在发生故障期间使用，不是应用程序工作流的常规部分。</span><span class="sxs-lookup"><span data-stu-id="5db52-174">The difference in this model is that the intermediate location is used only during failures, not as a regular part of the application workflow.</span></span>

<span data-ttu-id="5db52-175">在两种方案中，异步通信和中间存储均可防止后端服务中断导致整个应用程序停止运行。</span><span class="sxs-lookup"><span data-stu-id="5db52-175">In both scenarios, asynchronous communication and intermediate storage prevent a downed back-end service from bringing down the entire application.</span></span> <span data-ttu-id="5db52-176">队列充当逻辑中介。</span><span class="sxs-lookup"><span data-stu-id="5db52-176">Queues serve as a logical intermediary.</span></span> <span data-ttu-id="5db52-177">有关在队列服务之间进行选择的详细信息，请参阅 [Azure 队列和 Azure 服务总线队列 &mdash; 比较与对照](/azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted/)。</span><span class="sxs-lookup"><span data-stu-id="5db52-177">For more information on choosing between queuing services, see [Azure queues and Azure Service Bus queues &mdash; compared and contrasted](/azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted/).</span></span>

### <a name="fault-detection-and-retry-logic"></a><span data-ttu-id="5db52-178">故障检测和重试逻辑</span><span class="sxs-lookup"><span data-stu-id="5db52-178">Fault detection and retry logic</span></span>
<span data-ttu-id="5db52-179">高度可用的应用程序设计的一个关键方面是在代码中使用重试逻辑来正常处理暂时不可用的服务。</span><span class="sxs-lookup"><span data-stu-id="5db52-179">A key aspect of the design of highly available applications is the use of retry logic within code to gracefully handle a service that is temporarily unavailable.</span></span> <span data-ttu-id="5db52-180">适用于 Azure 存储和 Azure 服务总线的最新 SDK 版本本身就支持重试。</span><span class="sxs-lookup"><span data-stu-id="5db52-180">Recent versions of SDKs for both Azure Storage and Azure Service Bus natively support retries.</span></span> <span data-ttu-id="5db52-181">有关为应用程序提供自定义重试逻辑的详细信息，请参阅[重试模式](../patterns/retry.md)。</span><span class="sxs-lookup"><span data-stu-id="5db52-181">For more information on providing custom retry logic for your application, see the [Retry pattern](../patterns/retry.md).</span></span>

### <a name="reference-data-pattern-for-high-availability"></a><span data-ttu-id="5db52-182">高可用性的引用数据模式</span><span class="sxs-lookup"><span data-stu-id="5db52-182">Reference data pattern for high availability</span></span>
<span data-ttu-id="5db52-183">引用数据是应用程序的只读数据。</span><span class="sxs-lookup"><span data-stu-id="5db52-183">Reference data is the read-only data of an application.</span></span> <span data-ttu-id="5db52-184">此数据提供一个业务上下文，应用程序在业务运行期间将在其中生成事务数据。</span><span class="sxs-lookup"><span data-stu-id="5db52-184">This data provides the business context within which the application generates transactional data during a business operation.</span></span> <span data-ttu-id="5db52-185">事务数据的完整性依赖于在事务完成时引用数据的快照。</span><span class="sxs-lookup"><span data-stu-id="5db52-185">The integrity of transactional data depends on a snapshot of the reference data at the time the transaction was completed.</span></span>

<span data-ttu-id="5db52-186">引用数据是应用程序正确操作所必需的。</span><span class="sxs-lookup"><span data-stu-id="5db52-186">Reference data is necessary for the proper operation of the application.</span></span> <span data-ttu-id="5db52-187">各种应用程序创建并维护引用数据；主控数据管理 (MDM) 系统通常负责执行此功能。</span><span class="sxs-lookup"><span data-stu-id="5db52-187">Various applications create and maintain reference data; master data management (MDM) systems often perform this function.</span></span> <span data-ttu-id="5db52-188">这些系统对引用数据的整个生命周期负责。</span><span class="sxs-lookup"><span data-stu-id="5db52-188">These systems are responsible for the life cycle of the reference data.</span></span> <span data-ttu-id="5db52-189">引用数据的示例包括产品目录、雇员主控数据、部件主控数据和设备主控数据。</span><span class="sxs-lookup"><span data-stu-id="5db52-189">Examples of reference data include product catalog, employee master, parts master, and equipment master.</span></span> <span data-ttu-id="5db52-190">引用数据也可来自组织以外，如邮政编码或税率。</span><span class="sxs-lookup"><span data-stu-id="5db52-190">Reference data can also originate from outside the organization, for example, postal codes or tax rates.</span></span> <span data-ttu-id="5db52-191">提高引用数据可用性的策略之外通常要比提高事务数据可用性的策略简单。</span><span class="sxs-lookup"><span data-stu-id="5db52-191">Strategies for increasing the availability of reference data are typically less difficult than those for transactional data.</span></span> <span data-ttu-id="5db52-192">引用数据具有最为持久的优点。</span><span class="sxs-lookup"><span data-stu-id="5db52-192">Reference data has the advantage of being mostly immutable.</span></span>

<span data-ttu-id="5db52-193">通过将引用数据连同应用程序一起部署，可以让使用引用数据的 Azure Web 角色和辅助角色在运行时成为自治的。</span><span class="sxs-lookup"><span data-stu-id="5db52-193">Azure web and worker roles that consume reference data can be made autonomous at runtime by deploying the reference data along with the application.</span></span> <span data-ttu-id="5db52-194">如果本地存储的大小允许这样的部署，则此方法最理想。</span><span class="sxs-lookup"><span data-stu-id="5db52-194">This approach is ideal if the size of the local storage allows such a deployment.</span></span> <span data-ttu-id="5db52-195">在本地部署的嵌入式 SQL 数据库、NoSQL 数据库或 XML 文件有助于实现 Azure 计算缩放单位的自治。</span><span class="sxs-lookup"><span data-stu-id="5db52-195">Embedded SQL databases, NoSQL databases, or XML files deployed locally help with the autonomy of Azure compute scale units.</span></span> <span data-ttu-id="5db52-196">但是，应该制定一种机制，以便无需重新部署即可更新每个角色中的数据。</span><span class="sxs-lookup"><span data-stu-id="5db52-196">However, you should have a mechanism to update the data in each role without requiring redeployment.</span></span> <span data-ttu-id="5db52-197">为此，请将对引用数据的任何更新放置到云存储终结点（例如，Azure Blob 存储或 SQL 数据库）。</span><span class="sxs-lookup"><span data-stu-id="5db52-197">To do this, place any updates to the reference data at a cloud storage endpoint (for example, Azure Blob storage or SQL Database).</span></span> <span data-ttu-id="5db52-198">向每个角色添加在角色启动时会数据更新下载到计算节点中的代码。</span><span class="sxs-lookup"><span data-stu-id="5db52-198">Add code to each role that downloads the data updates into the compute nodes at role startup.</span></span> <span data-ttu-id="5db52-199">或者添加使管理员可在角色实例中执行强制下载的代码。</span><span class="sxs-lookup"><span data-stu-id="5db52-199">Alternatively, add code that allows an administrator to perform a forced download into the role instances.</span></span>

<span data-ttu-id="5db52-200">若要提高可用性，角色还应包含一组引用数据以防存储失灵。</span><span class="sxs-lookup"><span data-stu-id="5db52-200">To increase availability, the roles should also contain a set of reference data in case storage is down.</span></span> <span data-ttu-id="5db52-201">角色可以先使用一组基本的引用数据，直到有存储资源可供更新使用。</span><span class="sxs-lookup"><span data-stu-id="5db52-201">Roles can start with a basic set of reference data until the storage resource becomes available for the updates.</span></span>

![通过自治计算节点使应用程序获得高可用性](./images/high-availability-azure-applications/application-high-availability-through-autonomous-compute-nodes.png)

<span data-ttu-id="5db52-203">在此模式下，如果部署或下载大量的引用数据，则新部署或角色实例可能需要花费更长时间才能启动。</span><span class="sxs-lookup"><span data-stu-id="5db52-203">With this pattern, new deployments or role instances may take longer to start if you are deploying or downloading large amounts of reference data.</span></span> <span data-ttu-id="5db52-204">要在每个角色上立即有引用数据可用而不依赖于外部存储服务，此折衷也许是可接受的。</span><span class="sxs-lookup"><span data-stu-id="5db52-204">This tradeoff might be acceptable for the autonomy of having the reference data immediately available on each role, rather than depending on external storage services.</span></span>

### <a name="transactional-data-pattern-for-high-availability"></a><span data-ttu-id="5db52-205">高可用性的事务数据模式</span><span class="sxs-lookup"><span data-stu-id="5db52-205">Transactional data pattern for high availability</span></span>
<span data-ttu-id="5db52-206">事务数据是应用程序在某种业务上下文中生成的数据。</span><span class="sxs-lookup"><span data-stu-id="5db52-206">Transactional data is the data that the application generates in a business context.</span></span> <span data-ttu-id="5db52-207">事务数据由应用程序实现的一组业务流程与支持这些流程的引用数据组合而成。</span><span class="sxs-lookup"><span data-stu-id="5db52-207">Transactional data is a combination of the set of business processes that the application implements and the reference data that supports these processes.</span></span> <span data-ttu-id="5db52-208">事务数据的示例包括订单、预先发货通知、发票和客户关系管理 (CRM) 机会。</span><span class="sxs-lookup"><span data-stu-id="5db52-208">Examples of transactional data can include orders, advanced shipping notices, invoices, and customer relationship management (CRM) opportunities.</span></span> <span data-ttu-id="5db52-209">事务数据将被提供到外部系统进行记录保留或进一步处理。</span><span class="sxs-lookup"><span data-stu-id="5db52-209">Transactional data is supplied to external systems for record keeping or for further processing.</span></span>

<span data-ttu-id="5db52-210">引用数据在负责这些数据的系统中可能会发生更改。</span><span class="sxs-lookup"><span data-stu-id="5db52-210">Reference data can change within the systems that are responsible for that data.</span></span> <span data-ttu-id="5db52-211">因此，事务数据必须保存时间点引用数据上下文，以使其对外部的依赖程度降至最低，保持其语义一致性。</span><span class="sxs-lookup"><span data-stu-id="5db52-211">Therefore, transactional data must save the point-in-time reference data context to minimize external dependencies for its semantic consistency.</span></span> <span data-ttu-id="5db52-212">例如，可以在履行订单几个月后从目录中删除某个产品。</span><span class="sxs-lookup"><span data-stu-id="5db52-212">For example, a product may be removed from the catalog several months after an order is fulfilled.</span></span> <span data-ttu-id="5db52-213">建议随事务存储尽可能多的引用数据上下文。</span><span class="sxs-lookup"><span data-stu-id="5db52-213">Storing as much reference data context as possible with the transaction is recommended.</span></span> <span data-ttu-id="5db52-214">此方法可保留与事务相关的语义，即使要在捕获事务之后更改引用数据也是如此。</span><span class="sxs-lookup"><span data-stu-id="5db52-214">This approach preserves the semantics associated with the transaction, even if the reference data changes after the transaction is captured.</span></span>

<span data-ttu-id="5db52-215">如前所述，使用松散耦合和异步通信的体系结构可以提供更高级别的可用性。</span><span class="sxs-lookup"><span data-stu-id="5db52-215">As mentioned previously, architectures that use loose coupling and asynchronous communication can provide higher levels of availability.</span></span> <span data-ttu-id="5db52-216">对于事务数据也是这样，但实现起来更为复杂。</span><span class="sxs-lookup"><span data-stu-id="5db52-216">This is true for transactional data as well, but the implementation is more complex.</span></span> <span data-ttu-id="5db52-217">传统的事务模式通常依靠数据库来确保事务的正确处理。</span><span class="sxs-lookup"><span data-stu-id="5db52-217">Traditional transactional patterns usually rely on the database for guaranteeing the transaction.</span></span> <span data-ttu-id="5db52-218">引入中间层后，应用程序代码必须在各层正确处理数据，以确保足够的一致性和持久性。</span><span class="sxs-lookup"><span data-stu-id="5db52-218">When you introduce intermediate layers, the application code must correctly handle the data at various layers to ensure sufficient consistency and durability.</span></span>

<span data-ttu-id="5db52-219">以下序列描述了一个工作流，其中将事务数据的捕获与其处理相分离：</span><span class="sxs-lookup"><span data-stu-id="5db52-219">The following sequence describes a workflow that separates the capture of transactional data from its processing:</span></span>

1. <span data-ttu-id="5db52-220">Web 计算节点：提供引用数据。</span><span class="sxs-lookup"><span data-stu-id="5db52-220">Web compute node: Present reference data.</span></span>
2. <span data-ttu-id="5db52-221">外部存储：保存中间事务数据。</span><span class="sxs-lookup"><span data-stu-id="5db52-221">External storage: Save intermediate transactional data.</span></span>
3. <span data-ttu-id="5db52-222">Web 计算节点：完成最终用户事务。</span><span class="sxs-lookup"><span data-stu-id="5db52-222">Web compute node: Complete the end-user transaction.</span></span>
4. <span data-ttu-id="5db52-223">Web 计算节点：将完成的事务数据及其引用数据上下文一起发送到可保证做出可预测响应的临时持久存储。</span><span class="sxs-lookup"><span data-stu-id="5db52-223">Web compute node: Send the completed transactional data with its reference data context to temporary durable storage that is guaranteed to give a predictable response.</span></span>
5. <span data-ttu-id="5db52-224">Web 计算节点：通知最终用户事务已完成。</span><span class="sxs-lookup"><span data-stu-id="5db52-224">Web compute node: Signal the end-user completion of the transaction.</span></span>
6. <span data-ttu-id="5db52-225">后台计算节点：提取事务数据，如有必要，进一步处理这些数据，然后将其发送到它在当前系统中的最终存储位置。</span><span class="sxs-lookup"><span data-stu-id="5db52-225">Background compute node: Extract the transactional data, process it further if necessary, and send it to its final storage location in the current system.</span></span>

<span data-ttu-id="5db52-226">下图显示了在 Azure 云服务中实现这种设计的一种可行方式。</span><span class="sxs-lookup"><span data-stu-id="5db52-226">The following diagram shows one possible implementation of this design in an Azure-hosted cloud service.</span></span>

![通过松耦合获得高可用性](./images/disaster-recovery-high-availability-azure-applications/application-high-availability-through-loose-coupling.png)

<span data-ttu-id="5db52-228">上图中的虚线箭头表示异步处理。</span><span class="sxs-lookup"><span data-stu-id="5db52-228">The dashed arrows in the preceding diagram indicate asynchronous processing.</span></span> <span data-ttu-id="5db52-229">前端 Web 角色不了解这种异步处理。</span><span class="sxs-lookup"><span data-stu-id="5db52-229">The front-end web role is not aware of this asynchronous processing.</span></span> <span data-ttu-id="5db52-230">这样导致将事务存储在其最终目标，并引用当前系统。</span><span class="sxs-lookup"><span data-stu-id="5db52-230">This leads to the storage of the transaction at its final destination with reference to the current system.</span></span> <span data-ttu-id="5db52-231">由于此异步模型会导致延迟，因此无法立即查询事务数据。</span><span class="sxs-lookup"><span data-stu-id="5db52-231">Due to the latency that this asynchronous model introduces, the transactional data is not immediately available for query.</span></span> <span data-ttu-id="5db52-232">因此，需要将事务数据的每个单位保存在缓存或用户会话中，以满足即时 UI 的需要。</span><span class="sxs-lookup"><span data-stu-id="5db52-232">Therefore, each unit of the transactional data needs to be saved in a cache or a user session to meet the immediate UI needs.</span></span>

<span data-ttu-id="5db52-233">Web 角色将脱离基础结构的其余部分进行自治。</span><span class="sxs-lookup"><span data-stu-id="5db52-233">The web role is autonomous from the rest of the infrastructure.</span></span> <span data-ttu-id="5db52-234">其可用性配置文件是 Web 角色与 Azure 队列而非整个基础结构的结合。</span><span class="sxs-lookup"><span data-stu-id="5db52-234">Its availability profile is a combination of the web role and the Azure queue and not the entire infrastructure.</span></span> <span data-ttu-id="5db52-235">除了高可用性之外，这种方法还使 Web 角色可独立于后端存储进行水平缩放。</span><span class="sxs-lookup"><span data-stu-id="5db52-235">In addition to high availability, this approach allows the web role to scale horizontally, independent of the back-end storage.</span></span> <span data-ttu-id="5db52-236">这种高可用性模型可能会影响运营的经济状况。</span><span class="sxs-lookup"><span data-stu-id="5db52-236">This high-availability model can have an impact on the economics of operations.</span></span> <span data-ttu-id="5db52-237">Azure 队列和辅助角色等其他组件可能影响到每月的使用成本。</span><span class="sxs-lookup"><span data-stu-id="5db52-237">Additional components like Azure queues and worker roles can affect monthly usage costs.</span></span>

<span data-ttu-id="5db52-238">上图显示了针对事务数据的这种去耦方法的一种实现。</span><span class="sxs-lookup"><span data-stu-id="5db52-238">The previous diagram shows one implementation of this decoupled approach to transactional data.</span></span> <span data-ttu-id="5db52-239">还有许多其他实现可用。</span><span class="sxs-lookup"><span data-stu-id="5db52-239">There are many other possible implementations.</span></span> <span data-ttu-id="5db52-240">以下列表提供一些备用的变化形式：</span><span class="sxs-lookup"><span data-stu-id="5db52-240">The following list provides some alternatives:</span></span>

* <span data-ttu-id="5db52-241">可在 Web 角色与队列存储之间放置辅助角色。</span><span class="sxs-lookup"><span data-stu-id="5db52-241">A worker role might be placed between the web role and the storage queue.</span></span>
* <span data-ttu-id="5db52-242">可使用服务总线队列代替 Azure 存储队列。</span><span class="sxs-lookup"><span data-stu-id="5db52-242">A Service Bus queue can be used instead of an Azure Storage queue.</span></span>
* <span data-ttu-id="5db52-243">最终目标可能是 Azure 存储或其他数据库提供商。</span><span class="sxs-lookup"><span data-stu-id="5db52-243">The final destination might be Azure Storage or a different database provider.</span></span>
* <span data-ttu-id="5db52-244">可在 Web 层使用 Azure 缓存满足完成事务后的立即缓存要求。</span><span class="sxs-lookup"><span data-stu-id="5db52-244">Azure Cache can be used at the web layer to provide the immediate caching requirements after the transaction.</span></span>

### <a name="scalability-patterns"></a><span data-ttu-id="5db52-245">伸缩性模式</span><span class="sxs-lookup"><span data-stu-id="5db52-245">Scalability patterns</span></span>
<span data-ttu-id="5db52-246">请务必注意，云服务的伸缩性会直接影响到可用性。</span><span class="sxs-lookup"><span data-stu-id="5db52-246">It's important to note that the scalability of a cloud service directly affects availability.</span></span> <span data-ttu-id="5db52-247">如果负载增加导致服务无法响应，则给用户的感觉就是应用程序故障。</span><span class="sxs-lookup"><span data-stu-id="5db52-247">If increased load causes your service to be unresponsive, the user perception is that the application is down.</span></span> <span data-ttu-id="5db52-248">根据预期的应用程序负载和未来的预期，遵照用于实现可伸缩性的经验证做法进行操作。</span><span class="sxs-lookup"><span data-stu-id="5db52-248">Follow proven practices for scalability based on your expected application load and future expectations.</span></span> <span data-ttu-id="5db52-249">最大程度地扩大规模需要考虑许多因素，如使用单个还是多个存储帐户、在多个数据库之间共享以及缓存策略。</span><span class="sxs-lookup"><span data-stu-id="5db52-249">Maximizing scale involves many considerations, such as single versus multiple storage accounts, sharing across multiple databases, and caching strategies.</span></span> <span data-ttu-id="5db52-250">有关这些模式的详细信息，请参阅 [Best practices for designing large-scale services on Microsoft Azure](https://azure.microsoft.com/blog/best-practices-for-designing-large-scale-services-on-windows-azure/)（在 Microsoft Azure 上设计大型服务的最佳做法）。</span><span class="sxs-lookup"><span data-stu-id="5db52-250">For in-depth information about these patterns, see [Best practices for designing large-scale services on Microsoft Azure](https://azure.microsoft.com/blog/best-practices-for-designing-large-scale-services-on-windows-azure/).</span></span>

## <a name="next-steps"></a><span data-ttu-id="5db52-251">后续步骤</span><span class="sxs-lookup"><span data-stu-id="5db52-251">Next steps</span></span>
<span data-ttu-id="5db52-252">此系列文档包括了在 Microsoft Azure 上构建的应用程序的灾难恢复和高可用性。</span><span class="sxs-lookup"><span data-stu-id="5db52-252">This series of documents covers disaster recovery and high availability for applications built on Microsoft Azure.</span></span> <span data-ttu-id="5db52-253">此系列文档的下一篇文章是[在 Microsoft Azure 上构建的应用程序的灾难恢复](disaster-recovery-azure-applications.md)。</span><span class="sxs-lookup"><span data-stu-id="5db52-253">The next article in the series is [Disaster recovery for applications built on Microsoft Azure](disaster-recovery-azure-applications.md).</span></span>
