---
title: "从数据损坏或意外删除中恢复"
description: "本文可帮助了解如何从数据损坏或意外数据删除中恢复，并可帮助设计有复原能力和高可用性的容错应用程序，以及对灾难恢复进行规划"
author: adamglick
ms.date: 08/18/2016
ms.openlocfilehash: b75c774f85c42f64472167897f08a7302ab50a3f
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/14/2017
---
[!INCLUDE [header](../_includes/header.md)]
# <a name="azure-resiliency-technical-guidance-recovery-from-data-corruption-or-accidental-deletion"></a><span data-ttu-id="59a75-103">Azure 复原技术指南 - 数据损坏或意外删除后进行恢复</span><span class="sxs-lookup"><span data-stu-id="59a75-103">Azure resiliency technical guidance: recovery from data corruption or accidental deletion</span></span>
<span data-ttu-id="59a75-104">健全的业务连续性计划的一部分是针对数据损坏或意外删除制定计划。</span><span class="sxs-lookup"><span data-stu-id="59a75-104">Part of a robust business continuity plan is having a plan if your data gets corrupted or accidentally deleted.</span></span> <span data-ttu-id="59a75-105">下面提供有关应用程序错误或操作人员失误造成数据损坏或意外删除后进行恢复的信息。</span><span class="sxs-lookup"><span data-stu-id="59a75-105">The following is information about recovery after data has been corrupted or accidentally deleted, due to application errors or operator error.</span></span>

## <a name="virtual-machines"></a><span data-ttu-id="59a75-106">虚拟机</span><span class="sxs-lookup"><span data-stu-id="59a75-106">Virtual Machines</span></span>
<span data-ttu-id="59a75-107">若要在发生应用程序错误或意外删除后保护 Azure 虚拟机（有时称基础结构即服务 VM），可以使用 [Azure 备份](https://azure.microsoft.com/services/backup/)。</span><span class="sxs-lookup"><span data-stu-id="59a75-107">To protect your Azure Virtual Machines (sometimes called infrastructure-as-a-service VMs) from application errors or accidental deletion, use [Azure Backup](https://azure.microsoft.com/services/backup/).</span></span> <span data-ttu-id="59a75-108">使用 Azure 备份可跨多个 VM 磁盘创建一致的备份。</span><span class="sxs-lookup"><span data-stu-id="59a75-108">Azure Backup enables the creation of backups that are consistent across multiple VM disks.</span></span> <span data-ttu-id="59a75-109">此外，可以跨区域复制备份保管库，以便在发生区域服务中断时进行恢复。</span><span class="sxs-lookup"><span data-stu-id="59a75-109">In addition, the Backup Vault can be replicated across regions to provide recovery from region loss.</span></span>

## <a name="storage"></a><span data-ttu-id="59a75-110">存储</span><span class="sxs-lookup"><span data-stu-id="59a75-110">Storage</span></span>
<span data-ttu-id="59a75-111">请注意，尽管 Azure 存储通过自动化副本提供了数据复原功能，但这不会防止应用程序代码（或开发人员/用户）因为意外或无心的删除、更新等操作而造成数据损坏。</span><span class="sxs-lookup"><span data-stu-id="59a75-111">Note that while Azure Storage provides data resiliency through automated replicas, this does not prevent your application code (or developers/users) from corrupting data through accidental or unintended deletion, update, and so on.</span></span> <span data-ttu-id="59a75-112">在遇到应用程序或用户错误时保持数据保真需要更为先进的技术，如将数据和审核日志复制到辅助存储位置。</span><span class="sxs-lookup"><span data-stu-id="59a75-112">Maintaining data fidelity in the face of application or user error requires more advanced techniques, such as copying the data to a secondary storage location with an audit log.</span></span> <span data-ttu-id="59a75-113">开发人员可以利用 blob [快照功能](https://msdn.microsoft.com/library/azure/ee691971.aspx)，它可以创建 blob 内容的只读时间点快照。</span><span class="sxs-lookup"><span data-stu-id="59a75-113">Developers can take advantage of the blob [snapshot capability](https://msdn.microsoft.com/library/azure/ee691971.aspx), which can create read-only point-in-time snapshots of blob contents.</span></span> <span data-ttu-id="59a75-114">这可以用作 Azure 存储 Blob 数据保真解决方案的基础。</span><span class="sxs-lookup"><span data-stu-id="59a75-114">This can be used as the basis of a data-fidelity solution for Azure Storage blobs.</span></span>

### <a name="blob-and-table-storage-backup"></a><span data-ttu-id="59a75-115">Blob 和表存储备份</span><span class="sxs-lookup"><span data-stu-id="59a75-115">Blob and Table Storage Backup</span></span>
<span data-ttu-id="59a75-116">尽管 Blob 和表具有很高的持久性，但它们始终代表数据的当前状态。</span><span class="sxs-lookup"><span data-stu-id="59a75-116">While blobs and tables are highly durable, they always represent the current state of the data.</span></span> <span data-ttu-id="59a75-117">从意外的数据修改或删除中恢复可能需要将数据还原到以前的状态。</span><span class="sxs-lookup"><span data-stu-id="59a75-117">Recovery from unwanted modification or deletion of data may require restoring data to a previous state.</span></span> <span data-ttu-id="59a75-118">这可以通过利用 Azure 提供的功能来存储和保留时间点副本来实现。</span><span class="sxs-lookup"><span data-stu-id="59a75-118">This can be achieved by taking advantage of the capabilities provided by Azure to store and retain point-in-time copies.</span></span>

<span data-ttu-id="59a75-119">对于 Azure Blob，可以使用 [blob 快照功能](https://msdn.microsoft.com/library/ee691971.aspx)执行时间点备份。</span><span class="sxs-lookup"><span data-stu-id="59a75-119">For Azure Blobs, you can perform point-in-time backups using the [blob snapshot feature](https://msdn.microsoft.com/library/ee691971.aspx).</span></span> <span data-ttu-id="59a75-120">对于每个快照，只需要支付存储自上次快照状态以来 Blob 内的更改所需的存储空间费用。</span><span class="sxs-lookup"><span data-stu-id="59a75-120">For each snapshot, you are only charged for the storage required to store the differences within the blob since the last snapshot state.</span></span> <span data-ttu-id="59a75-121">快照依赖于它们所基于的原始 Blob 的存在，因此建议复制到另一个 Blob 甚至另一个存储帐户。</span><span class="sxs-lookup"><span data-stu-id="59a75-121">The snapshots are dependent on the existence of the original blob they are based on, so a copy operation to another blob or even another storage account is advisable.</span></span> <span data-ttu-id="59a75-122">这可以确保正确保护备份数据不受意外删除的影响。</span><span class="sxs-lookup"><span data-stu-id="59a75-122">This ensures that backup data is properly protected against accidental deletion.</span></span> <span data-ttu-id="59a75-123">对于 Azure 表，可以将时间点备份保存到另一个表或 Azure Blob。</span><span class="sxs-lookup"><span data-stu-id="59a75-123">For Azure Tables, you can make point-in-time copies to a different table or to Azure Blobs.</span></span> <span data-ttu-id="59a75-124">有关执行表和 Blob 的应用程序级备份的更详细指南和示例，请参阅以下文章：</span><span class="sxs-lookup"><span data-stu-id="59a75-124">More detailed guidance and examples of performing application-level backups of tables and blobs can be found here:</span></span>

* [<span data-ttu-id="59a75-125">防止表受到应用程序错误的影响</span><span class="sxs-lookup"><span data-stu-id="59a75-125">Protecting Your Tables Against Application Errors</span></span>](https://blogs.msdn.microsoft.com/windowsazurestorage/2010/05/03/protecting-your-tables-against-application-errors/)
* [<span data-ttu-id="59a75-126">防止 Blob 受到应用程序错误的影响</span><span class="sxs-lookup"><span data-stu-id="59a75-126">Protecting Your Blobs Against Application Errors</span></span>](https://blogs.msdn.microsoft.com/windowsazurestorage/2010/04/29/protecting-your-blobs-against-application-errors/)

## <a name="database"></a><span data-ttu-id="59a75-127">数据库</span><span class="sxs-lookup"><span data-stu-id="59a75-127">Database</span></span>
<span data-ttu-id="59a75-128">Azure SQL 数据库提供了多个[业务连续性](/azure/sql-database/sql-database-business-continuity/)（备份、还原）选项。</span><span class="sxs-lookup"><span data-stu-id="59a75-128">There are several [business continuity](/azure/sql-database/sql-database-business-continuity/) (backup, restore) options available for Azure SQL Database.</span></span> <span data-ttu-id="59a75-129">可以使用[数据库复制](/azure/sql-database/sql-database-copy/)功能或通过[导出](/azure/sql-database/sql-database-export/)和[导入](https://msdn.microsoft.com/library/hh710052.aspx) SQL Server bacpac 文件来复制数据库。</span><span class="sxs-lookup"><span data-stu-id="59a75-129">Databases can be copied by using the [Database Copy](/azure/sql-database/sql-database-copy/) functionality, or by  [exporting](/azure/sql-database/sql-database-export/) and [importing](https://msdn.microsoft.com/library/hh710052.aspx) a SQL Server bacpac file.</span></span> <span data-ttu-id="59a75-130">数据库复制提供事务一致的结果，而 bacpac（通过导入/导出服务）则不会提供事务一致的结果。</span><span class="sxs-lookup"><span data-stu-id="59a75-130">Database Copy provides transactionally consistent results, while a bacpac (through the import/export service) does not.</span></span> <span data-ttu-id="59a75-131">这两种选项都在数据中心中作为基于队列的服务运行，当前不提供完成时间 SLA。</span><span class="sxs-lookup"><span data-stu-id="59a75-131">Both of these options run as queue-based services within the data center, and they do not currently provide a time-to-completion SLA.</span></span>

> [!NOTE]
> <span data-ttu-id="59a75-132">数据库复制和导入/导出服务会对源数据库形成极大的负载。</span><span class="sxs-lookup"><span data-stu-id="59a75-132">The database copy and import/export options place a significant degree of load on the source database.</span></span> <span data-ttu-id="59a75-133">它们可能会触发资源争用或限制事件。</span><span class="sxs-lookup"><span data-stu-id="59a75-133">They can trigger resource contention or throttling events.</span></span>
> 
> 

### <a name="sql-database-backup"></a><span data-ttu-id="59a75-134">SQL 数据库备份</span><span class="sxs-lookup"><span data-stu-id="59a75-134">SQL Database Backup</span></span>
<span data-ttu-id="59a75-135">Microsoft Azure SQL 数据库的时间点备份是通过[复制 Azure SQL 数据库](/azure/sql-database/sql-database-copy/)来实现的。</span><span class="sxs-lookup"><span data-stu-id="59a75-135">Point-in-time backups for Microsoft Azure SQL Database are achieved by [copying your Azure SQL database](/azure/sql-database/sql-database-copy/).</span></span> <span data-ttu-id="59a75-136">可以使用此命令在同一逻辑数据库服务器或不同服务器上创建数据库的事务一致副本。</span><span class="sxs-lookup"><span data-stu-id="59a75-136">You can use this command to create a transactionally consistent copy of a database on the same logical database server or to a different server.</span></span> <span data-ttu-id="59a75-137">无论是哪种情况，数据库副本都完全独立于源数据库并发挥全部功能。</span><span class="sxs-lookup"><span data-stu-id="59a75-137">In either case, the database copy is fully functional and completely independent of the source database.</span></span> <span data-ttu-id="59a75-138">创建的每个副本表示一个时间点恢复选项。</span><span class="sxs-lookup"><span data-stu-id="59a75-138">Each copy you create represents a point-in-time recovery option.</span></span> <span data-ttu-id="59a75-139">可以通过将新数据库重命名为源数据库名称，完全恢复数据库状态。</span><span class="sxs-lookup"><span data-stu-id="59a75-139">You can recover the database state completely by renaming the new database with the source database name.</span></span> <span data-ttu-id="59a75-140">或者，可以通过使用 Transact-SQL 查询从新数据库恢复数据的特定子集。</span><span class="sxs-lookup"><span data-stu-id="59a75-140">Alternatively, you can recover a specific subset of data from the new database by using Transact-SQL queries.</span></span> <span data-ttu-id="59a75-141">有关 SQL Database 的其他详细信息，请参阅[使用 Azure SQL 数据库确保业务连续性的相关概述](/azure/sql-database/sql-database-business-continuity/)。</span><span class="sxs-lookup"><span data-stu-id="59a75-141">For additional details about SQL Database, see [Overview of business continuity with Azure SQL Database](/azure/sql-database/sql-database-business-continuity/).</span></span>

### <a name="sql-server-on-virtual-machines-backup"></a><span data-ttu-id="59a75-142">虚拟机上的 SQL Server 备份</span><span class="sxs-lookup"><span data-stu-id="59a75-142">SQL Server on Virtual Machines Backup</span></span>
<span data-ttu-id="59a75-143">对于在 Azure 基础结构即服务虚拟机（通常称为 IaaS 或 Iaas VM）上使用的 SQL Server，有两个选项可供使用：传统备份和日志传送。</span><span class="sxs-lookup"><span data-stu-id="59a75-143">For SQL Server used with Azure infrastructure as a service virtual machines (often called IaaS or IaaS VMs), there are two options: traditional backups and log shipping.</span></span> <span data-ttu-id="59a75-144">通过使用传统备份可以还原到某个特定时间点，但恢复过程较慢。</span><span class="sxs-lookup"><span data-stu-id="59a75-144">Using traditional backups enables you to restore to a specific point in time, but the recovery process is slow.</span></span> <span data-ttu-id="59a75-145">还原传统备份要求从初始的完整备份开始，然后应用在该备份之后创建的所有备份。</span><span class="sxs-lookup"><span data-stu-id="59a75-145">Restoring traditional backups requires starting with an initial full backup, and then applying any backups taken after that.</span></span> <span data-ttu-id="59a75-146">第二个选项是配置日志传送会话，延迟日志备份的还原（例如，延迟两小时）。</span><span class="sxs-lookup"><span data-stu-id="59a75-146">The second option is to configure a log shipping session to delay the restore of log backups (for example, by two hours).</span></span> <span data-ttu-id="59a75-147">这就为从主要数据库发生的错误恢复提供了时间窗口。</span><span class="sxs-lookup"><span data-stu-id="59a75-147">This provides a window to recover from errors made on the primary.</span></span>

## <a name="other-azure-platform-services"></a><span data-ttu-id="59a75-148">其他 Azure 平台服务</span><span class="sxs-lookup"><span data-stu-id="59a75-148">Other Azure platform services</span></span>
<span data-ttu-id="59a75-149">某些 Azure 平台服务将信息存储在用户控制的存储帐户或 SQL 数据库中。</span><span class="sxs-lookup"><span data-stu-id="59a75-149">Some Azure platform services store information in a user-controlled storage account or Azure SQL Database.</span></span> <span data-ttu-id="59a75-150">如果帐户或存储资源遭到删除或损毁，这可能会造成服务的严重错误。</span><span class="sxs-lookup"><span data-stu-id="59a75-150">If the account or storage resource is deleted or corrupted, this could cause serious errors with the service.</span></span> <span data-ttu-id="59a75-151">在这些情况下，请务必维护在这些资源已删除或损坏时可用于重新创建它们的备份。</span><span class="sxs-lookup"><span data-stu-id="59a75-151">In these cases, it is important to maintain backups that would enable you to re-create these resources if they were deleted or corrupted.</span></span>

<span data-ttu-id="59a75-152">对于 Azure 网站和 Azure 移动服务，必须备份和维护关联的数据库。</span><span class="sxs-lookup"><span data-stu-id="59a75-152">For Azure Web Sites and Azure Mobile Services, you must backup and maintain the associated databases.</span></span> <span data-ttu-id="59a75-153">对于 Azure 媒体服务和虚拟机，必须维护相关的 Azure 存储帐户和该帐户中的所有资源。</span><span class="sxs-lookup"><span data-stu-id="59a75-153">For Azure Media Service and Virtual Machines, you must maintain the associated Azure Storage account and all resources in that account.</span></span> <span data-ttu-id="59a75-154">例如，对于虚拟机，必须备份和管理 Azure Blob 存储中的 VM 磁盘。</span><span class="sxs-lookup"><span data-stu-id="59a75-154">For example, for Virtual Machines, you must back up and manage the VM disks in Azure blob storage.</span></span>

## <a name="checklists-for-data-corruption-or-accidental-deletion"></a><span data-ttu-id="59a75-155">数据损坏和意外删除清单</span><span class="sxs-lookup"><span data-stu-id="59a75-155">Checklists for data corruption or accidental deletion</span></span>
## <a name="virtual-machines-checklist"></a><span data-ttu-id="59a75-156">虚拟机清单</span><span class="sxs-lookup"><span data-stu-id="59a75-156">Virtual Machines checklist</span></span>
1. <span data-ttu-id="59a75-157">查看本文档的“虚拟机”部分。</span><span class="sxs-lookup"><span data-stu-id="59a75-157">Review the Virtual Machines section of this document.</span></span>
2. <span data-ttu-id="59a75-158">使用 Azure 备份（或者在自己的备份系统中使用 Azure Blob 存储和 VHD 快照）来备份和维护 VM 磁盘。</span><span class="sxs-lookup"><span data-stu-id="59a75-158">Back up and maintain the VM disks with Azure Backup (or your own backup system by using Azure blob storage and VHD snapshots).</span></span>

## <a name="storage-checklist"></a><span data-ttu-id="59a75-159">存储清单</span><span class="sxs-lookup"><span data-stu-id="59a75-159">Storage checklist</span></span>
1. <span data-ttu-id="59a75-160">查看本文档的“存储”部分。</span><span class="sxs-lookup"><span data-stu-id="59a75-160">Review the Storage section of this document.</span></span>
2. <span data-ttu-id="59a75-161">定期备份重要的存储资源。</span><span class="sxs-lookup"><span data-stu-id="59a75-161">Regularly back up critical storage resources.</span></span>
3. <span data-ttu-id="59a75-162">考虑对 Blob 使用快照功能。</span><span class="sxs-lookup"><span data-stu-id="59a75-162">Consider using the snapshot feature for blobs.</span></span>

## <a name="database-checklist"></a><span data-ttu-id="59a75-163">数据库清单</span><span class="sxs-lookup"><span data-stu-id="59a75-163">Database checklist</span></span>
1. <span data-ttu-id="59a75-164">查看本文档的“数据库”部分。</span><span class="sxs-lookup"><span data-stu-id="59a75-164">Review the Database section of this document.</span></span>
2. <span data-ttu-id="59a75-165">使用“数据库复制”命令创建时间点备份。</span><span class="sxs-lookup"><span data-stu-id="59a75-165">Create point-in-time backups by using the Database Copy command.</span></span>

## <a name="sql-server-on-virtual-machines-backup-checklist"></a><span data-ttu-id="59a75-166">虚拟机上的 SQL Server 备份清单</span><span class="sxs-lookup"><span data-stu-id="59a75-166">SQL Server on Virtual Machines Backup checklist</span></span>
1. <span data-ttu-id="59a75-167">查看本文档的“虚拟机上的 SQL Server 备份”部分。</span><span class="sxs-lookup"><span data-stu-id="59a75-167">Review the SQL Server on Virtual Machines Backup section of this document.</span></span>
2. <span data-ttu-id="59a75-168">使用传统备份和还原技术。</span><span class="sxs-lookup"><span data-stu-id="59a75-168">Use traditional backup and restore techniques.</span></span>
3. <span data-ttu-id="59a75-169">创建延迟的日志传送会话。</span><span class="sxs-lookup"><span data-stu-id="59a75-169">Create a delayed log shipping session.</span></span>

## <a name="web-apps-checklist"></a><span data-ttu-id="59a75-170">Web 应用清单</span><span class="sxs-lookup"><span data-stu-id="59a75-170">Web Apps checklist</span></span>
1. <span data-ttu-id="59a75-171">备份和维护关联的数据库（如果有）。</span><span class="sxs-lookup"><span data-stu-id="59a75-171">Back up and maintain the associated database, if any.</span></span>

## <a name="media-services-checklist"></a><span data-ttu-id="59a75-172">媒体服务清单</span><span class="sxs-lookup"><span data-stu-id="59a75-172">Media Services checklist</span></span>
1. <span data-ttu-id="59a75-173">备份和维护关联的存储资源。</span><span class="sxs-lookup"><span data-stu-id="59a75-173">Back up and maintain the associated storage resources.</span></span>

## <a name="more-information"></a><span data-ttu-id="59a75-174">详细信息</span><span class="sxs-lookup"><span data-stu-id="59a75-174">More information</span></span>
<span data-ttu-id="59a75-175">有关 Azure 中的备份和还原功能的详细信息，请参阅[存储、备份和恢复方案](https://azure.microsoft.com/documentation/scenarios/storage-backup-recovery/)。</span><span class="sxs-lookup"><span data-stu-id="59a75-175">For more information about backup and restore features in Azure, see [Storage, backup and recovery scenarios](https://azure.microsoft.com/documentation/scenarios/storage-backup-recovery/).</span></span>

