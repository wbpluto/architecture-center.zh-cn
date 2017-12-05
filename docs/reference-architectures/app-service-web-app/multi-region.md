---
title: "多区域 Web 应用程序"
description: "对于在 Microsoft Azure 中运行的具有高可用性的 Web 应用程序，建议使用的体系结构。"
author: MikeWasson
ms.date: 11/23/2016
cardTitle: Run in multiple regions
ms.openlocfilehash: 2d7d0c38bef3efc73a7ba2dd61e4190d07deb1b5
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
# <a name="run-a-web-application-in-multiple-regions"></a><span data-ttu-id="b6bf9-103">在多个区域中运行 Web 应用程序</span><span class="sxs-lookup"><span data-stu-id="b6bf9-103">Run a web application in multiple regions</span></span>
[!INCLUDE [header](../../_includes/header.md)]

<span data-ttu-id="b6bf9-104">此参考体系结构展示了如何在多个区域中运行 Azure 应用服务应用程序以实现高可用性。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-104">This reference architecture shows how to run an Azure App Service application in multiple regions to achieve high availability.</span></span> 

![参考体系结构：具有高可用性的 Web 应用程序](./images/multi-region-web-app-diagram.png) 

<span data-ttu-id="b6bf9-106">下载此体系结构的 [Visio 文件][visio-download]。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-106">*Download a [Visio file][visio-download] of this architecture.*</span></span>

## <a name="architecture"></a><span data-ttu-id="b6bf9-107">体系结构</span><span class="sxs-lookup"><span data-stu-id="b6bf9-107">Architecture</span></span> 

<span data-ttu-id="b6bf9-108">此体系结构基于[提高 Web 应用程序中的可伸缩性][guidance-web-apps-scalability]中显示的体系结构。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-108">This architecture builds on the one shown in [Improve scalability in a web application][guidance-web-apps-scalability].</span></span> <span data-ttu-id="b6bf9-109">它们的主要区别包括：</span><span class="sxs-lookup"><span data-stu-id="b6bf9-109">The main differences are:</span></span>

* <span data-ttu-id="b6bf9-110">**主要和次要区域**。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-110">**Primary and secondary regions**.</span></span> <span data-ttu-id="b6bf9-111">此体系结构使用两个区域来实现更高的可用性。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-111">This architecture uses two regions to achieve higher availability.</span></span> <span data-ttu-id="b6bf9-112">应用程序部署到每个区域。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-112">The application is deployed to each region.</span></span> <span data-ttu-id="b6bf9-113">在正常运行期间，网络流量被路由到主要区域。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-113">During normal operations, network traffic is routed to the primary region.</span></span> <span data-ttu-id="b6bf9-114">如果主要区域变得不可用，则流量将被路由到次要区域。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-114">If the primary region becomes unavailable, traffic is routed to the secondary region.</span></span> 
* <span data-ttu-id="b6bf9-115">**Azure 流量管理器**。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-115">**Azure Traffic Manager**.</span></span> <span data-ttu-id="b6bf9-116">[流量管理器][traffic-manager]将传入请求路由到主要区域。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-116">[Traffic Manager][traffic-manager] routes incoming requests to the primary region.</span></span> <span data-ttu-id="b6bf9-117">如果在该区域中运行的应用程序变得不可用，则流量管理器将故障转移到次要区域。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-117">If the application running that region becomes unavailable, Traffic Manager fails over to the secondary region.</span></span>
* <span data-ttu-id="b6bf9-118">SQL 数据库和 Cosmos DB 的**异地复制**。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-118">**Geo-replication** of SQL Database and Cosmos DB.</span></span> 

<span data-ttu-id="b6bf9-119">与部署到单个区域相比，多区域体系结构可以提供更高的可用性。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-119">A multi-region architecture can provide higher availability than deploying to a single region.</span></span> <span data-ttu-id="b6bf9-120">如果区域性故障影响了主要区域，则可以使用[流量管理器][traffic-manager]故障转移到次要区域。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-120">If a regional outage affects the primary region, you can use [Traffic Manager][traffic-manager] to fail over to the secondary region.</span></span> <span data-ttu-id="b6bf9-121">当应用程序的单个子系统出现故障时，此体系结构可能也比较有用。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-121">This architecture can also help if an individual subsystem of the application fails.</span></span>

<span data-ttu-id="b6bf9-122">有多种常规方法可跨区域实现高可用性：</span><span class="sxs-lookup"><span data-stu-id="b6bf9-122">There are several general approaches to achieving high availability across regions:</span></span> 

* <span data-ttu-id="b6bf9-123">主动/被动（采用热备用模式）。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-123">Active/passive with hot standby.</span></span> <span data-ttu-id="b6bf9-124">流量将前往一个区域，而另一个区域将以热备用模式等待。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-124">Traffic goes to one region, while the other waits on hot standby.</span></span> <span data-ttu-id="b6bf9-125">“热备用模式”意味着次要区域中的 VM 已被分配并总是处于运行状态。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-125">Hot standby means the VMs in the secondary region are allocated and running at all times.</span></span>
* <span data-ttu-id="b6bf9-126">主动/被动（采用冷备用模式）。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-126">Active/passive with cold standby.</span></span> <span data-ttu-id="b6bf9-127">流量将前往一个区域，而另一个区域将以冷备用模式等待。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-127">Traffic goes to one region, while the other waits on cold standby.</span></span> <span data-ttu-id="b6bf9-128">“冷备用模式”意味着次要区域中的 VM 不会被分配，直到故障转移需要它们。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-128">Cold standby means the VMs in the secondary region are not allocated until needed for failover.</span></span> <span data-ttu-id="b6bf9-129">此方法的运行成本较低，但是当发生故障时通常需要花费更长时间才能联机。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-129">This approach costs less to run, but will generally take longer to come online during a failure.</span></span>
* <span data-ttu-id="b6bf9-130">主动/主动。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-130">Active/active.</span></span> <span data-ttu-id="b6bf9-131">两个区域都处于活动状态，并且会在它们之间对请求进行负载均衡。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-131">Both regions are active, and requests are load balanced between them.</span></span> <span data-ttu-id="b6bf9-132">如果一个区域变得不可用，则不再使其参与轮换。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-132">If one region becomes unavailable, it is taken out of rotation.</span></span> 

<span data-ttu-id="b6bf9-133">此参考体系结构侧重于“主动/被动（采用热备用模式）”，使用流量管理器进行故障转移。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-133">This reference architecture focuses on active/passive with hot standby, using Traffic Manager for failover.</span></span> 


## <a name="recommendations"></a><span data-ttu-id="b6bf9-134">建议</span><span class="sxs-lookup"><span data-stu-id="b6bf9-134">Recommendations</span></span>

<span data-ttu-id="b6bf9-135">你的要求可能不同于此处描述的体系结构。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-135">Your requirements might differ from the architecture described here.</span></span> <span data-ttu-id="b6bf9-136">一开始可使用此部分的建议。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-136">Use the recommendations in this section as a starting point.</span></span>

### <a name="regional-pairing"></a><span data-ttu-id="b6bf9-137">区域配对</span><span class="sxs-lookup"><span data-stu-id="b6bf9-137">Regional pairing</span></span>
<span data-ttu-id="b6bf9-138">每个 Azure 区域都与同一地域内的另一个区域配对。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-138">Each Azure region is paired with another region within the same geography.</span></span> <span data-ttu-id="b6bf9-139">通常，请选择同一区域对中的区域（例如“美国东部 2”和“美国中部”）。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-139">In general, choose regions from the same regional pair (for example, East US 2 and Central US).</span></span> <span data-ttu-id="b6bf9-140">这样做的好处包括：</span><span class="sxs-lookup"><span data-stu-id="b6bf9-140">Benefits of doing so include:</span></span>

* <span data-ttu-id="b6bf9-141">如果发生大范围的故障，会优先恢复每个区域对中的至少一个区域。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-141">If there is a broad outage, recovery of at least one region out of every pair is prioritized.</span></span>
* <span data-ttu-id="b6bf9-142">计划内 Azure 系统更新会按顺序提供给配对的区域，以尽可能减少停机时间。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-142">Planned Azure system updates are rolled out to paired regions sequentially to minimize possible downtime.</span></span>
* <span data-ttu-id="b6bf9-143">在多数情况下，区域对位于同一地域内以满足数据驻留要求。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-143">In most cases, regional pairs reside within the same geography to meet data residency requirements.</span></span>

<span data-ttu-id="b6bf9-144">但请确保两个区域都支持应用程序所需的所有 Azure 服务。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-144">However, make sure that both regions support all of the Azure services needed for your application.</span></span> <span data-ttu-id="b6bf9-145">请参阅[每个区域的服务][services-by-region]。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-145">See [Services by region][services-by-region].</span></span> <span data-ttu-id="b6bf9-146">有关区域对的详细信息，请参阅[业务连续性和灾难恢复 (BCDR)：Azure 配对区域][regional-pairs]。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-146">For more information about regional pairs, see [Business continuity and disaster recovery (BCDR): Azure Paired Regions][regional-pairs].</span></span>

### <a name="resource-groups"></a><span data-ttu-id="b6bf9-147">资源组</span><span class="sxs-lookup"><span data-stu-id="b6bf9-147">Resource groups</span></span>
<span data-ttu-id="b6bf9-148">考虑将主要区域、次要区域和流量管理器放置到单独的[资源组][resource groups]中。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-148">Consider placing the primary region, secondary region, and Traffic Manager into separate [resource groups][resource groups].</span></span> <span data-ttu-id="b6bf9-149">这允许你将部署到每个区域的资源作为单个集合进行管理。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-149">This lets you manage the resources deployed to each region as a single collection.</span></span>

### <a name="traffic-manager-configuration"></a><span data-ttu-id="b6bf9-150">流量管理器配置</span><span class="sxs-lookup"><span data-stu-id="b6bf9-150">Traffic Manager configuration</span></span> 

<span data-ttu-id="b6bf9-151">**路由**。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-151">**Routing**.</span></span> <span data-ttu-id="b6bf9-152">流量管理器支持多个[路由算法][tm-routing]。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-152">Traffic Manager supports several [routing algorithms][tm-routing].</span></span> <span data-ttu-id="b6bf9-153">对于本文中所述的情况，请使用“优先级”路由（以前称为“故障转移”路由）。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-153">For the scenario described in this article, use *priority* routing (formerly called *failover* routing).</span></span> <span data-ttu-id="b6bf9-154">使用此设置时，流量管理器将所有请求都发送到主要区域，除非该区域的终结点变得无法访问。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-154">With this setting, Traffic Manager sends all requests to the primary region unless the endpoint for that region becomes unreachable.</span></span> <span data-ttu-id="b6bf9-155">那时，它将自动故障转移到次要区域。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-155">At that point, it automatically fails over to the secondary region.</span></span> <span data-ttu-id="b6bf9-156">请参阅[配置故障转移路由方法][tm-configure-failover]。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-156">See [Configure Failover routing method][tm-configure-failover].</span></span>

<span data-ttu-id="b6bf9-157">**运行状况探测**。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-157">**Health probe**.</span></span> <span data-ttu-id="b6bf9-158">流量管理器使用 HTTP（或 HTTPS）探测来监视每个终结点的可用性。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-158">Traffic Manager uses an HTTP (or HTTPS) probe to monitor the availability of each endpoint.</span></span> <span data-ttu-id="b6bf9-159">探测为流量管理器提供了一个通过/失败测试以用于故障转移到次要区域。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-159">The probe gives Traffic Manager a pass/fail test for failing over to the secondary region.</span></span> <span data-ttu-id="b6bf9-160">它通过将请求发送到指定的 URL 路径来执行工作。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-160">It works by sending a request to a specified URL path.</span></span> <span data-ttu-id="b6bf9-161">如果它在超时期间内收到一个非-200 响应，则探测失败。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-161">If it gets a non-200 response within a timeout period, the probe fails.</span></span> <span data-ttu-id="b6bf9-162">在出现四个失败的请求后，流量管理器会将该终结点标记为已降级并故障转移到另一终结点。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-162">After four failed requests, Traffic Manager marks the endpoint as degraded and fails over to the other endpoint.</span></span> <span data-ttu-id="b6bf9-163">有关详细信息，请参阅[流量管理器终结点监视和故障转移][tm-monitoring]。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-163">For details, see [Traffic Manager endpoint monitoring and failover][tm-monitoring].</span></span>

<span data-ttu-id="b6bf9-164">作为最佳做法，请创建一个运行状况探测终结点，用以报告应用程序的整体运行状况，并使用此终结点进行运行状况探测。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-164">As a best practice, create a health probe endpoint that reports the overall health of the application and use this endpoint for the health probe.</span></span> <span data-ttu-id="b6bf9-165">此终结点应当检查关键依赖项，例如应用服务应用、存储队列和 SQL 数据库。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-165">The endpoint should check critical dependencies such as the App Service apps, storage queue, and SQL Database.</span></span> <span data-ttu-id="b6bf9-166">否则，探测可能会在应用程序的关键部分实际上已出现故障时报告终结点运行状况正常。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-166">Otherwise, the probe might report a healthy endpoint when critical parts of the application are actually failing.</span></span>

<span data-ttu-id="b6bf9-167">另一方面，请不要使用运行状况探测来检查较低优先级的服务。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-167">On the other hand, don't use the health probe to check lower priority services.</span></span> <span data-ttu-id="b6bf9-168">例如，如果某个电子邮件服务发生故障，则应用程序可以切换到辅助提供程序或者只是稍后再发送电子邮件。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-168">For example, if an email service goes down the application can switch to a second provider or just send emails later.</span></span> <span data-ttu-id="b6bf9-169">这不是一个足够高的优先级，不会导致应用程序进行故障转移。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-169">This is not a high enough priority to cause the application to fail over.</span></span> <span data-ttu-id="b6bf9-170">有关详细信息，请参阅[运行状况终结点监视模式][health-endpoint-monitoring-pattern]。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-170">For more information, see [Health Endpoint Monitoring Pattern][health-endpoint-monitoring-pattern].</span></span>
 
### <a name="sql-database"></a><span data-ttu-id="b6bf9-171">SQL 数据库</span><span class="sxs-lookup"><span data-stu-id="b6bf9-171">SQL Database</span></span>
<span data-ttu-id="b6bf9-172">使用[活动异地复制][sql-replication]在一个不同的区域中创建可读取的次要副本。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-172">Use [Active Geo-Replication][sql-replication] to create a readable secondary replica in a different region.</span></span> <span data-ttu-id="b6bf9-173">最多可以有四个可读取的次要副本。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-173">You can have up to four readable secondary replicas.</span></span> <span data-ttu-id="b6bf9-174">如果主数据库失败或者需要使其脱机，请故障转移到辅助数据库。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-174">Fail over to a secondary database if your primary database fails or needs to be taken offline.</span></span> <span data-ttu-id="b6bf9-175">可以为任何弹性数据库池中的任何数据库配置活动异地复制。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-175">Active Geo-Replication can be configured for any database in any elastic database pool.</span></span>

### <a name="cosmos-db"></a><span data-ttu-id="b6bf9-176">Cosmos DB</span><span class="sxs-lookup"><span data-stu-id="b6bf9-176">Cosmos DB</span></span>
<span data-ttu-id="b6bf9-177">Cosmos DB 支持跨区域的异地复制。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-177">Cosmos DB supports geo-replication across regions.</span></span> <span data-ttu-id="b6bf9-178">一个区域被指定为可写入的，其他区域是只读副本。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-178">One region is designated as writable and the others are read-only replicas.</span></span>

<span data-ttu-id="b6bf9-179">如果发生区域性中断，可以通过选择另一个区域作为写入区域来进行故障转移。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-179">If there is a regional outage, you can fail over by selecting another region to be the write region.</span></span> <span data-ttu-id="b6bf9-180">客户端 SDK 会自动将写入请求发送到当前写入区域，因此，在故障转移后不需要更新客户端配置。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-180">The client SDK automatically sends write requests to the current write region, so you don't need to update the client configuration after a failover.</span></span> <span data-ttu-id="b6bf9-181">有关详细信息，请参阅[如何使用 Azure Cosmos DB 在全球范围内分发数据？][docdb-geo]</span><span class="sxs-lookup"><span data-stu-id="b6bf9-181">For more information, see [How to distribute data globally with Azure Cosmos DB?][docdb-geo]</span></span>

> [!NOTE]
> <span data-ttu-id="b6bf9-182">所有副本都属于同一资源组。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-182">All of the replicas belong to the same resource group.</span></span>
>
>

### <a name="storage"></a><span data-ttu-id="b6bf9-183">存储</span><span class="sxs-lookup"><span data-stu-id="b6bf9-183">Storage</span></span>
<span data-ttu-id="b6bf9-184">对于 Azure 存储，请使用[读取访问异地冗余存储][ra-grs] (RA-GRS)。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-184">For Azure Storage, use [read-access geo-redundant storage][ra-grs] (RA-GRS).</span></span> <span data-ttu-id="b6bf9-185">使用 RA-GRS 存储时，数据被复制到次要区域。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-185">With RA-GRS storage, the data is replicated to a secondary region.</span></span> <span data-ttu-id="b6bf9-186">你可以通过一个单独的终结点以只读方式访问次要区域中的数据。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-186">You have read-only access to the data in the secondary region through a separate endpoint.</span></span> <span data-ttu-id="b6bf9-187">如果发生区域性中断或灾难，则 Azure 存储团队可以决定执行到次要区域的异地故障转移。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-187">If there is a regional outage or disaster, the Azure Storage team might decide to perform a geo-failover to the secondary region.</span></span> <span data-ttu-id="b6bf9-188">客户不需要为此故障转移执行任何操作。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-188">There is no customer action required for this failover.</span></span>

<span data-ttu-id="b6bf9-189">对于队列存储，请在次要区域中创建一个备份队列。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-189">For Queue storage, create a backup queue in the secondary region.</span></span> <span data-ttu-id="b6bf9-190">在故障转移期间，应用可以使用备份队列，直到主要区域变得重新可用。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-190">During failover, the app can use the backup queue until the primary region becomes available again.</span></span> <span data-ttu-id="b6bf9-191">这样，应用程序仍然可以处理新请求。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-191">That way, the application can still process new requests.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="b6bf9-192">可用性注意事项</span><span class="sxs-lookup"><span data-stu-id="b6bf9-192">Availability considerations</span></span>


### <a name="traffic-manager"></a><span data-ttu-id="b6bf9-193">流量管理器</span><span class="sxs-lookup"><span data-stu-id="b6bf9-193">Traffic Manager</span></span>

<span data-ttu-id="b6bf9-194">如果主要区域变得不可用，则流量管理器将自动进行故障转移。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-194">Traffic Manager automatically fails over if the primary region becomes unavailable.</span></span> <span data-ttu-id="b6bf9-195">当流量管理器进行故障转移时，一段时间内客户端无法访问应用程序。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-195">When Traffic Manager fails over, there is a period of time when clients cannot reach the application.</span></span> <span data-ttu-id="b6bf9-196">持续时间受以下因素影响：</span><span class="sxs-lookup"><span data-stu-id="b6bf9-196">The duration is affected by the following factors:</span></span>

* <span data-ttu-id="b6bf9-197">运行状况探测必须检测主数据中心是否已变得无法访问。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-197">The health probe must detect that the primary data center has become unreachable.</span></span>
* <span data-ttu-id="b6bf9-198">域名服务 (DNS) 服务器必须更新 IP 地址的已缓存 DNS 记录，这取决于 DNS 生存时间 (TTL)。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-198">Domain name service (DNS) servers must update the cached DNS records for the IP address, which depends on the DNS time-to-live (TTL).</span></span> <span data-ttu-id="b6bf9-199">默认 TTL 为 300 秒（5 分钟），但可以在创建流量管理器配置文件时配置此值。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-199">The default TTL is 300 seconds (5 minutes), but you can configure this value when you create the Traffic Manager profile.</span></span>

<span data-ttu-id="b6bf9-200">相关详细信息，请参阅[关于流量管理器监视][tm-monitoring]。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-200">For details, see [About Traffic Manager Monitoring][tm-monitoring].</span></span>

<span data-ttu-id="b6bf9-201">流量管理器是系统中的一个潜在故障点。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-201">Traffic Manager is a possible failure point in the system.</span></span> <span data-ttu-id="b6bf9-202">如果服务出现故障，则客户端在停机期间无法访问应用程序。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-202">If the service fails, clients cannot access your application during the downtime.</span></span> <span data-ttu-id="b6bf9-203">查看[流量管理器服务级别协议 (SLA)][tm-sla]，然后确定仅使用流量管理器是否能满足业务对高可用性的需求。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-203">Review the [Traffic Manager service level agreement (SLA)][tm-sla] and determine whether using Traffic Manager alone meets your business requirements for high availability.</span></span> <span data-ttu-id="b6bf9-204">如果不能，请考虑添加另一个流量管理解决方案作为故障回复机制。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-204">If not, consider adding another traffic management solution as a failback.</span></span> <span data-ttu-id="b6bf9-205">如果 Azure 流量管理器服务出现故障，请将 DNS 中的规范名称 (CNAME) 记录更改为指向另一个流量管理服务。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-205">If the Azure Traffic Manager service fails, change your canonical name (CNAME) records in DNS to point to the other traffic management service.</span></span> <span data-ttu-id="b6bf9-206">此步骤必须手动执行，并且在 DNS 更改被传播之前，应用程序将不可用。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-206">This step must be performed manually, and your application will be unavailable until the DNS changes are propagated.</span></span>

### <a name="sql-database"></a><span data-ttu-id="b6bf9-207">SQL 数据库</span><span class="sxs-lookup"><span data-stu-id="b6bf9-207">SQL Database</span></span>
<span data-ttu-id="b6bf9-208">[使用 Azure SQL 数据库确保业务连续性的相关概述][sql-rpo]中介绍了 SQL 数据库的恢复点目标 (RPO) 和估计恢复时间 (ERT)。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-208">The recovery point objective (RPO) and estimated recovery time (ERT) for SQL Database are documented in [Overview of business continuity with Azure SQL Database][sql-rpo].</span></span> 

### <a name="storage"></a><span data-ttu-id="b6bf9-209">存储</span><span class="sxs-lookup"><span data-stu-id="b6bf9-209">Storage</span></span>
<span data-ttu-id="b6bf9-210">RA-GRS 存储提供了持久性存储，但请务必了解在中断期间可能会发生什么情况：</span><span class="sxs-lookup"><span data-stu-id="b6bf9-210">RA-GRS storage provides durable storage, but it's important to understand what can happen during an outage:</span></span>

* <span data-ttu-id="b6bf9-211">如果发生存储中断，则在一段时间内无法对数据进行写入访问。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-211">If a storage outage occurs, there will be a period of time when you don't have write-access to the data.</span></span> <span data-ttu-id="b6bf9-212">在中断期间，仍然可以从辅助终结点读取数据。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-212">You can still read from the secondary endpoint during the outage.</span></span>
* <span data-ttu-id="b6bf9-213">如果区域性故障或灾难影响了主位置并且无法从那里恢复数据，则 Azure 存储团队可以决定执行到次要区域的异地故障转移。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-213">If a regional outage or disaster affects the primary location and the data there cannot be recovered, the Azure Storage team may decide to perform a geo-failover to the secondary region.</span></span>
* <span data-ttu-id="b6bf9-214">到次要区域的数据复制是以异步方式执行的。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-214">Data replication to the secondary region is performed asynchronously.</span></span> <span data-ttu-id="b6bf9-215">因此，如果执行异地故障转移，并且无法从主要区域中恢复数据，则可能会丢失一些数据。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-215">Therefore, if a geo-failover is performed, some data loss is possible if the data can't be recovered from the primary region.</span></span>
* <span data-ttu-id="b6bf9-216">暂时性故障（例如网络中断）不会触发存储故障转移。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-216">Transient failures, such as a network outage, will not trigger a storage failover.</span></span> <span data-ttu-id="b6bf9-217">设计应用程序时请使其能够在发生暂时性故障时进行复原。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-217">Design your application to be resilient to transient failures.</span></span> <span data-ttu-id="b6bf9-218">可能的缓解措施：</span><span class="sxs-lookup"><span data-stu-id="b6bf9-218">Possible mitigations:</span></span>
  
  * <span data-ttu-id="b6bf9-219">从次要区域中进行读取。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-219">Read from the secondary region.</span></span>
  * <span data-ttu-id="b6bf9-220">临时切换到另一存储帐户来执行新的写入操作（例如将消息排入队列）。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-220">Temporarily switch to another storage account for new write operations (for example, to queue messages).</span></span>
  * <span data-ttu-id="b6bf9-221">将数据从次要区域复制到另一个存储帐户。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-221">Copy data from the secondary region to another storage account.</span></span>
  * <span data-ttu-id="b6bf9-222">提供降低的功能，直到系统完成故障回复。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-222">Provide reduced functionality until the system fails back.</span></span>

<span data-ttu-id="b6bf9-223">有关详细信息，请参阅[在 Azure 存储中断时该怎么办][storage-outage]。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-223">For more information, see [What to do if an Azure Storage outage occurs][storage-outage].</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="b6bf9-224">可管理性注意事项</span><span class="sxs-lookup"><span data-stu-id="b6bf9-224">Manageability Considerations</span></span>

### <a name="traffic-manager"></a><span data-ttu-id="b6bf9-225">流量管理器</span><span class="sxs-lookup"><span data-stu-id="b6bf9-225">Traffic Manager</span></span>

<span data-ttu-id="b6bf9-226">如果流量管理器进行故障转移，我们建议执行手动故障回复，而不是实施自动故障回复。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-226">If Traffic Manager fails over, we recommend performing a manual failback rather than implementing an automatic failback.</span></span> <span data-ttu-id="b6bf9-227">否则，可能会造成应用程序在区域之间来回转移的情况。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-227">Otherwise, you can create a situation where the application flips back and forth between regions.</span></span> <span data-ttu-id="b6bf9-228">在进行故障回复之前，请验证是否所有应用程序子系统的运行状况都正常。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-228">Verify that all application subsystems are healthy before failing back.</span></span>

<span data-ttu-id="b6bf9-229">请注意，默认情况下，流量管理器会自动进行故障回复。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-229">Note that Traffic Manager automatically fails back by default.</span></span> <span data-ttu-id="b6bf9-230">若要禁止此操作，请在发生故障转移事件后手动降低主要区域的优先级。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-230">To prevent this, manually lower the priority of the primary region after a failover event.</span></span> <span data-ttu-id="b6bf9-231">例如，假设主要区域的优先级为 1，次要区域的优先级为 2。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-231">For example, suppose the primary region is priority 1 and the secondary is priority 2.</span></span> <span data-ttu-id="b6bf9-232">在故障转移后，请将主要区域的优先级设置为 3，以禁止自动故障回复。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-232">After a failover, set the primary region to priority 3, to prevent automatic failback.</span></span> <span data-ttu-id="b6bf9-233">当准备好切换回来时，请将优先级更新为 1。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-233">When you are ready to switch back, update the priority to 1.</span></span>

<span data-ttu-id="b6bf9-234">以下命令用于更新优先级。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-234">The following commands update the priority.</span></span>

<span data-ttu-id="b6bf9-235">**PowerShell**</span><span class="sxs-lookup"><span data-stu-id="b6bf9-235">**PowerShell**</span></span>

```bat
$endpoint = Get-AzureRmTrafficManagerEndpoint -Name <endpoint> -ProfileName <profile> -ResourceGroupName <resource-group> -Type AzureEndpoints
$endpoint.Priority = 3
Set-AzureRmTrafficManagerEndpoint -TrafficManagerEndpoint $endpoint
```

<span data-ttu-id="b6bf9-236">有关详细信息，请参阅 [Azure 流量管理器 Cmdlet][tm-ps]。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-236">For more information, see [Azure Traffic Manager Cmdlets][tm-ps].</span></span>

<span data-ttu-id="b6bf9-237">**Azure 命令行界面 (CLI)**</span><span class="sxs-lookup"><span data-stu-id="b6bf9-237">**Azure command line interface (CLI)**</span></span>

```bat
azure network traffic-manager endpoint set --name <endpoint> --profile-name <profile> --resource-group <resource-group> --type AzureEndpoints --priority 3
```    

### <a name="sql-database"></a><span data-ttu-id="b6bf9-238">SQL 数据库</span><span class="sxs-lookup"><span data-stu-id="b6bf9-238">SQL Database</span></span>
<span data-ttu-id="b6bf9-239">如果主数据库发生故障，请执行到辅助数据库的手动故障转移。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-239">If the primary database fails, perform a manual failover to the secondary database.</span></span> <span data-ttu-id="b6bf9-240">请参阅[还原 Azure SQL 数据库或故障转移到辅助数据库][sql-failover]。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-240">See [Restore an Azure SQL Database or failover to a secondary][sql-failover].</span></span> <span data-ttu-id="b6bf9-241">在进行故障转移之前，辅助数据库将保持只读状态。</span><span class="sxs-lookup"><span data-stu-id="b6bf9-241">The secondary database remains read-only until you fail over.</span></span>


<!-- links -->

[azure-sql-db]: https://azure.microsoft.com/documentation/services/sql-database/
[docdb-geo]: /azure/documentdb/documentdb-distribute-data-globally
[guidance-web-apps-scalability]: ./scalable-web-app.md
[health-endpoint-monitoring-pattern]: https://msdn.microsoft.com/library/dn589789.aspx
[ra-grs]: /azure/storage/storage-redundancy#read-access-geo-redundant-storage
[regional-pairs]: /azure/best-practices-availability-paired-regions
[resource groups]: /azure/azure-resource-manager/resource-group-overview#resource-groups
[services-by-region]: https://azure.microsoft.com/regions/#services
[sql-failover]: /azure/sql-database/sql-database-disaster-recovery
[sql-replication]: /azure/sql-database/sql-database-geo-replication-overview
[sql-rpo]: /azure/sql-database/sql-database-business-continuity#sql-database-features-that-you-can-use-to-provide-business-continuity
[storage-outage]: /azure/storage/storage-disaster-recovery-guidance
[tm-configure-failover]: /azure/traffic-manager/traffic-manager-configure-failover-routing-method
[tm-monitoring]: /azure/traffic-manager/traffic-manager-monitoring
[tm-ps]: https://msdn.microsoft.com/library/mt125941.aspx
[tm-routing]: /azure/traffic-manager/traffic-manager-routing-methods
[tm-sla]: https://azure.microsoft.com/support/legal/sla/traffic-manager/v1_0/
[traffic-manager]: https://azure.microsoft.com/services/traffic-manager/
[visio-download]: https://archcenter.azureedge.net/cdn/app-service-reference-architectures.vsdx