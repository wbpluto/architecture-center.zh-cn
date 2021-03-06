---
title: 后台作业指南
description: 有关独立于用户界面运行的后台任务的指南。
author: dragon119
ms.date: 05/24/2017
pnp.series.title: Best Practices
ms.openlocfilehash: 781d616dfcf24775525e2489e7e463174ec9bfa3
ms.sourcegitcommit: e9d9e214529edd0dc78df5bda29615b8fafd0e56
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/28/2018
ms.locfileid: "37091081"
---
# <a name="background-jobs"></a>后台作业
[!INCLUDE [header](../_includes/header.md)]

许多类型的应用程序需要运行与用户界面 (UI) 无关的后台任务。 示例包括批处理作业、密集处理型任务，以及长时间运行的过程，例如工作流。 后台作业无需用户交互就可执行 -- 应用程序可以启动作业，并继续处理来自用户的交互请求。 这有助于减少应用程序 UI 上的负载，从而提高可用性，降低交互响应时间。

例如，如果应用程序需要生成用户上传的图像缩图，可以后台作业的形式执行此操作，并在完成时会缩图保存到存储中，用户无需等待过程完成。 同样地，下单的用户可以启动一个后台工作流来处理订单，同时，UI 可让用户继续浏览 Web 应用。 当后台作业完成时，可以更新存储的订单数据，并将确认订单的电子邮件发送给用户。

考虑是否将任务作为后台作业执行时，主要的准则是：是否可以在无需用户交互的情况下运行该任务，且 UI 无需等待作业完成。 在完成之前需要用户或 UI 等待的任务不适合作为后台作业运行。

## <a name="types-of-background-jobs"></a>后台作业的类型
后台作业通常包括以下类型的一个或多个作业：

* CPU 密集型作业，例如数学计算或结构模型分析。
* I/O 密集型作业，例如执行一系列存储事务或文件索引编制。
* 批处理作业，例如夜间数据更新或计划的处理。
* 长时间运行的工作流，例如订单履行或预配服务和系统。
* 敏感数据处理，其中的任务将转移到更安全的位置以进行处理。 例如，可能不希望处理 Web 应用中的敏感数据。 而想使用[守护程序](http://msdn.microsoft.com/library/dn589793.aspx)等模式将数据传输到有权访问受保护存储的已隔离后台进程。

## <a name="triggers"></a>触发器
可通过多种不同的方式启动后台作业。 这些方式属于以下类别之一：

* [事件驱动的触发器](#event-driven-triggers)。 响应事件时启动任务，这通常是用户或工作流中的步骤执行的操作。
* [计划驱动的触发器](#schedule-driven-triggers)。 基于计时器按计划调用任务。 这可能是定期计划，或者指定在以后运行的一次性调用。

### <a name="event-driven-triggers"></a>事件驱动的触发器
事件驱动的调用使用触发器来启动后台任务。 使用事件驱动的触发器的示例包括：

* UI 或另一个作业将消息放入队列。 该消息包含有关已执行的操作的数据，例如下单的用户。 后台任务将侦听此队列，并检测新消息是否已到达。 它将读取消息，并将其中的数据用作后台作业的输入。
* UI 或另一个作业将保存或更新存储中的值。 后台任务将监视存储并检测更改。 它将读取数据，并将数据用作后台作业的输入。
* UI 或另一个作业向终结点（例如 HTTPS URI，或作为 Web 服务公开的 API）发出请求。 它在请求的过程中传递所需的数据以完成后台任务。 终结点或 Web 服务调用后台任务，将数据用作其输入。

适合事件驱动调用的任务的典型示例包括图像处理、工作流、将信息发送到远程服务、发送电子邮件消息，以及在多租户应用程序中预配新用户。

### <a name="schedule-driven-triggers"></a>计划驱动的触发器
计划驱动的调用使用计时器启动后台任务。 使用计划驱动的触发器的示例包括：

* 在应用程序本地运行的计时器或作为应用程序操作系统一部分的计时器定期调用后台任务。
* 在不同应用程序中运行的计时器或计时器服务（例如 Azure 计划程序）定期将请求发送到 API 或 Web 服务。 API 或 Web 服务调用后台任务。
* 单独的进程或应用程序启动计时器，从而在指定的时间延迟后或在特定时间调用后台任务一次。

适合计划驱动调用的任务的典型示例包括批处理例程（例如，根据用户最新的行为来更新其相关产品列表）、例行数据处理任务（例如更新索引或生成累积结果）、分析每日报告的数据、清理保留的数据和数据一致性检查。

如果使用必须作为单个实例运行的计划驱动任务，请注意以下事项：

* 如果缩放正在运行计划程序的计算实例（例如使用 Windows 计划任务的虚拟机），需要运行计划程序的多个实例。 这些操作可能会启动任务的多个实例。
* 如果任务的运行时间超过了计划程序事件的间隔时间，计划程序可以在前一个任务仍在运行时启动任务的另一个实例。

## <a name="returning-results"></a>返回结果
后台作业通过 UI 或调用后台任务的进程，以异步方式在独立进程甚至不同的位置运行。 在理想情况下，后台任务是“即发即弃”的操作，其执行进度不会影响 UI 或调用进程。 这意味着，调用进程不会等待任务完成。 因此无法自动检测任务的结束时间。

如果需要后台任务与调用任务通信以指示进度或完成状态，则必须为此实施一种机制。 下面是一些示例：

* 将状态指示器值写入可供 UI 或调用方任务访问的存储，这样可以在需要时监视或检查此值。 可将后台任务必须返回给调用方的数据放入同一存储中。
* 建立 UI 或调用方侦听的答复队列。 后台任务可将消息发送到指示状态和完成情况的队列。 可将后台任务必须返回给调用方的数据放入消息中。 如果使用 Azure 服务总线，则可以使用 **ReplyTo** 和 **CorrelationId** 属性来实现此功能。 有关详细信息，请参阅[服务总线中转消息传送中的相关性](http://www.cloudcasts.net/devguide/Default.aspx?id=13029)。
* 从 UI 或调用方可以访问的后台任务公开 API 或终结点以获取状态信息。 可以在响应中包含后台任务必须返回给调用方的数据。
* 让后台任务通过 API 回调 UI 或调用方，以指示预定义时间点或完成时的状态。 这可以通过本地引发的事件或通过发布与订阅机制来实现。 可在请求或事件负载中包含后台任务必须返回给调用方的数据。

## <a name="hosting-environment"></a>宿主环境
可以使用各种不同的 Azure 平台服务来托管后台任务：

* [**Azure Web 应用和 Web 作业**](#azure-web-apps-and-webjobs)。 可以根据 Web 应用上下文中各种不同类型的脚本或可执行程序，使用 Web 作业来执行自定义作业。
* [Azure 虚拟机](#azure-virtual-machines)。 如果有 Windows 服务或想要使用 Windows 任务计划程序，则常见的做法是将后台任务托管在专用虚拟机中。
* [Azure Batch](#azure-batch)。 Batch 是一种平台服务，该服务计划在虚拟机托管集合上运行的计算密集型工作。 它可以自动缩放计算资源。
* [Azure 容器服务](#azure-container-service)。 Azure 容器服务在 Azure 上提供容器托管环境。 
* [Azure 云服务](#azure-cloud-services)。 可以在作为后台任务执行的角色中编写代码。

以下各节详细介绍每一个选项，并提供相关注意事项，帮助你选择适当的选项。

### <a name="azure-web-apps-and-webjobs"></a>Azure Web 应用和 Web 作业

可以使用 Azure Web 作业将自定义作业作为 Azure Web 应用中的后台任务来执行。 Web 作业可在 Web 应用的上下文中作为连续进程运行。 Web 作业还可以在响应来自 Azure 计划程序或外部因素（例如更改存储 Blob 和消息队列）的触发器事件时运行。 作业可按需启动和停止，并可以正常关闭。 如果连续运行的 Web 作业失败，它会自动重新启动。 重试和错误操作可配置。

配置 Web 作业时：

* 如果想要作业响应事件驱动的触发器，应将其配置为“连续运行”。 脚本或进程存储在名为 site/wwwroot/app_data/jobs/continuous 的文件夹中。
* 如果想要作业响应计划驱动的触发器，应将其配置为“按计划运行”。 脚本或进程存储在名为 site/wwwroot/app_data/jobs/triggered 的文件夹中。
* 如果在配置作业时选择了“按需要运行”选项，在启动时，该作业将执行选择了“按计划运行”选项时的相同代码。

Azure Web 作业在 Web 应用沙箱中运行。 这意味着，它们可以访问环境变量，并与 Web 应用共享连接字符串等信息。 作业有权访问运行该作业的计算机的唯一标识符。 名为 **AzureWebJobsStorage** 的连接字符串可用于访问应用程序数据的 Azure 存储队列、Blob 和表，以及用于消息传送和通信的服务总线。 名为 **AzureWebJobsDashboard** 的连接字符串可用于访问作业操作日志文件。

Azure Web 作业具有以下特征：

* **安全性**：Web 作业受 Web 应用的部署凭据保护。
* **支持的文件类型**：可以使用命令脚本 (.cmd)、批处理文件 (.bat)、PowerShell 脚本 (.ps1)、bash shell 脚本 (.sh)、PHP 脚本 (.php)、Python 脚本 (.py)、JavaScript 代码 (.js) 和可执行程序（.exe、.jar 等等）来定义 Web 作业。
* **部署**：可以使用 [Azure 门户](/azure/app-service-web/web-sites-create-web-jobs)、[Visual Studio](/azure/app-service-web/websites-dotnet-deploy-webjobs)、[Azure Web 作业 SDK](/azure/azure-webjobs-sdk) 或直接将它们复制到以下位置以部署脚本和可执行文件：
  * 对于触发的执行：site/wwwroot/app_data/jobs/triggered/{job name}
  * 对于连续执行：site/wwwroot/app_data/jobs/continuous/{job name}
* **日志记录**：Console.Out 被视为（标记为）INFO。 Console.Error 被视为 ERROR。 可以使用 Azure 门户来访问监视和诊断信息。 可以直接从站点下载日志文件。 这些信息保存在以下位置：
  * 对于触发的执行：Vfs/data/jobs/triggered/jobName
  * 对于连续执行：Vfs/data/jobs/continuous/jobName
* **配置**：可以使用门户、REST API 和 PowerShell 配置 Web 作业。 可以使用与作业脚本位于同一根目录的配置文件（名为 settings.job）来提供作业的配置信息。 例如：
  * { "stopping_wait_time": 60 }
  * { "is_singleton": true }

#### <a name="considerations"></a>注意事项

* 默认情况下，Web 作业会随 Web 应用缩放。 但是，可以通过将 **is_singleton** 配置属性设置为 **true**，将作业配置为在单个实例上运行。 单个实例 Web 作业适用于不希望以同时进行多个实例的方式进行缩放或运行的任务（如重建索引、数据分析和类似任务）。
* 要将 Web 应用性能对任务的影响降到最低，请考虑在新的应用服务计划中创建空的 Azure Web 应用，以托管可能长时间运行或资源密集型的 Web 作业。

### <a name="more-information"></a>详细信息
* [有关 Azure Web 作业的推荐资源](/azure/app-service-web/websites-webjobs-resources)列出了 Web 作业的许多有用资源、下载内容和示例。

### <a name="azure-virtual-machines"></a>Azure 虚拟机
实施后台任务时，可以避免将其部署到 Azure Web 应用或云服务，但有时这些选项可能不方便。 典型的示例包括 Windows 服务、第三方实用程序和可执行程序。 另一个示例是针对托管应用程序以外的执行环境所编写的程序。 例如，它可能是你想要从 Windows 或 .NET 应用程序执行的 Unix 或 Linux 程序。 可以为 Azure 虚拟机选择各种操作系统，并在该虚拟机上运行服务或可执行文件。

若要确定何时使用虚拟机，请参阅 [Azure 应用服务s, Cloud Services and Virtual Machines comparison](/azure/app-service-web/choose-web-site-cloud-service-vm/)（Azure 应用程序服务、云服务和虚拟机的比较）。 有关虚拟机选项的信息，请参阅 [Virtual Machine and Cloud Service sizes for Azure](http://msdn.microsoft.com/library/azure/dn197896.aspx)（Azure 的虚拟机和云服务大小）。 有关虚拟机可用的操作系统和预建映像的详细信息，请参阅 [Azure 虚拟机市场](https://azure.microsoft.com/gallery/virtual-machines/)。

若要在独立的虚拟机中启动后台任务，可以从多个选项中进行选择：

* 可以通过将请求发送到任务公开的终结点，来根据需要直接从应用程序执行任务。 这会传入任务所需的任何数据。 此终结点将调用该任务。
* 可以将任务配置为使用所选操作系统中提供的计划程序或计时器按计划运行。 例如，可以在 Windows 上使用 Windows 任务计划程序来执行脚本和任务。 或者，如果已在虚拟机上安装了 SQL Server，则可以使用 SQL Server 代理来执行脚本和任务。
* 可以使用 Azure 计划程序来启动任务，方法是将消息添加到任务侦听的队列，或将请求发送到任务公开的 API。

有关如何启动后台任务的详细信息，请参阅前面的[触发器](#triggers)部分。  

#### <a name="considerations"></a>注意事项
在确定是否要在 Azure 虚拟机中部署后台任务时，请注意以下几点：

* 在单独的 Azure 虚拟机中托管后台任务可提供弹性，并可通过启动、执行、计划和资源分配来实现精确控制。 但是，如果只是出于运行后台任务的目的而必须部署虚拟机，则会增加运行时成本。
* 没有任何工具可以监视 Azure 门户中的任务，并且对于失败的任务没有任何自动重新启动功能 -- 不过，用户可以监视虚拟机的基本状态，并使用[Azure 资源管理器 Cmdlet](https://msdn.microsoft.com/library/mt125356.aspx) 来管理它。 但是，计算节点中没有任何工具可用于控制进程和线程。 通常，使用虚拟机时，需要付出额外的工作量才能实施一个机制用于从任务的检测中收集数据，以及从虚拟机中的操作系统收集数据。 一个适当的解决方案是使用 [System Center Management Pack for Azure](https://www.microsoft.com/download/details.aspx?id=50013)（用于 Azure 的 System Center 管理包）。
* 可以考虑创建通过 HTTP 终结点公开的监视探测。 这些探测器的代码可以执行运行状况检查、收集操作信息和统计信息，或者整理错误信息，并将其返回给管理应用程序。 有关详细信息，请参阅[运行状况终结点监视模式](http://msdn.microsoft.com/library/dn589789.aspx)。

#### <a name="more-information"></a>详细信息
* Azure 上的[虚拟机](https://azure.microsoft.com/services/virtual-machines/)
* [Azure 虚拟机常见问题解答](/azure/virtual-machines/virtual-machines-linux-classic-faq?toc=%2fazure%2fvirtual-machines%2flinux%2fclassic%2ftoc.json)

### <a name="azure-batch"></a>Azure 批处理 

如果需要在数十、数百或数千个 VM 上运行大型、并行的高性能计算 (HPC) 工作负载，请考虑使用 [Azure Batch](/azure/batch/)。  

Batch 服务预配 VM、将任务分配给 VM、运行任务并监视进度。 Batch 可以根据工作负载横向扩展 VM。 Batch 还提供作业计划。 Azure Batch 支持 Linux 和 Windows VM。

#### <a name="considerations"></a>注意事项 

Batch 在固有并行的工作负载上运行良好。 它还可执行并行计算（最后有一个减少单步执行），或者为需要在节点间传递的消息的并行任务，运行[消息传递接口 (MPI) 应用程序](/azure/batch/batch-mpi)。 

Azure Batch 作业在节点池上运行 (VM)。 一种方法是仅在需要时分配池并在作业完成后将其删除。 这可最大化利用率，因为节点不是空闲状态，但是作业必须等待节点分配。 或者，可以提前创建池。 此方法可最小化启动作业的时间，但是会导致节点处于空闲状态。 有关详细信息，请参阅[池和计算节点生存期](/azure/batch/batch-api-basics#pool-and-compute-node-lifetime)。

#### <a name="more-information"></a>详细信息 

* [使用 Batch 运行固有并行的工作负载](/azure/batch/batch-technical-overview) 
* [使用 Batch 开发大规模并行计算解决方案](/azure/batch/batch-api-basics) 
* [适用于大规模计算工作负载的 Batch 和 HPC 解决方案](/azure/batch/batch-hpc-solutions)

### <a name="azure-container-service"></a>Azure 容器服务 

通过 Azure 容器服务，可在 Azure 中配置和管理 VM 以运行容器化的应用程序。 它针对业务流程提供了 Docker Swarm、DC/OS 或 Kubernetes 选项。 

容器有助于运行后台作业。 一些优点包括： 

- 容器支持高密度托管。 可以隔离容器中的后台任务，同时在每个 VM 中放置多个容器。
- 容器业务流程协调程序处理内部负载均衡、配置内部网络和其他配置任务。
- 容器可在需要时启动和停止。 
- 通过Azure 容器注册表，可注册 Azure 边界内的容器。 这会带来安全性、隐私和邻近感应方面的好处。 

#### <a name="considerations"></a>注意事项

- 需要了解如何使用容器业务流程协调程序。 这是否可能成为一个问题取决于 DevOps 团队的技能组合。  
- 容器服务在 IaaS 环境中运行。 它预配专用 VNet 内的 VM 的群集。 

#### <a name="more-information"></a>详细信息 

* [Azure 容器服务的 Docker 容器托管解决方案简介](/azure/container-service/container-service-intro) 
* [专用 Docker 容器注册表简介](/azure/container-registry/container-registry-intro) 

### <a name="azure-cloud-services"></a>Azure 云服务 
可以在 Web 角色或单独的辅助角色中执行后台任务。 是否使用辅助角色的决定应该基于伸缩性和弹性要求、任务生存期、发行安排、安全性、容错、资源争用、复杂性和逻辑体系结构等考虑因素。 有关详细信息，请参阅[计算资源整合模式](http://msdn.microsoft.com/library/dn589778.aspx)。

可通过多种方式在云服务角色中实施后台任务：

* 在角色中创建 **RoleEntryPoint** 类的实现，并使用它的方法来执行后台任务。 任务在 WaIISHost.exe 的上下文中运行。 它们可以使用 **CloudConfigurationManager** 类的 **GetSetting** 方法来加载配置设置。 有关详细信息，请参阅[生命周期](#lifecycle)。
* 应用程序启动时，使用启动任务来执行后台任务。 要强制任务继续在后台运行，请将 **taskType** 属性设置为 **background**（如果不这样做，应用程序启动进程会中止并等待任务完成）。 有关详细信息，请参阅 [Run startup tasks in Azure](/azure/cloud-services/cloud-services-startup-tasks)（在 Azure 中运行启动任务）。
* 可以使用 WebJobs SDK 来实施作为启动任务启动的后台任务（如 Web 作业）。 有关详细信息，请参阅[在 Azure 应用服务中创建 .NET Web 作业](/azure/app-service-web/websites-dotnet-webjobs-sdk-get-started)。
* 使用启动任务可以安装一个 Windows 服务来执行一个或多个后台任务。 必须将 **taskType** 属性设置为 **background**，以便服务在后台执行。 有关详细信息，请参阅 [Run startup tasks in Azure](/azure/cloud-services/cloud-services-startup-tasks)（在 Azure 中运行启动任务）。

在 Web 角色中运行后台任务的主要优势在于可以节省托管成本，因为不要求部署其他角色。

在辅助角色中运行后台任务具有以下优势：

* 可让为每种类型的角色单独管理缩放。 例如，可能需要更多的 Web 角色实例才能支持当前负载，但需要更少的辅助角色实例即可执行后台任务。 从 UI 角色单独缩放后台任务计算实例可以减少托管成本，同时保持可接受的性能。
* 卸载来自 Web 角色的后台任务的处理开销。 提供 UI 的 Web 角色可以保持响应度，这还可能意味着需要更少的实例，即可支持来自用户的给定请求数量。
* 通过它，可实现关注点分离。 每种角色类型可以实施特定一组明确定义的相关任务。 这就简化了代码设计和维护，因为各角色之间的代码和功能依赖性降低。
* 有助于隔离敏感的进程和数据。 例如，实施 UI 的 Web 角色无需访问辅助角色管理和控制的数据。 这可用于增强安全性，尤其是使用[守护程序模式](http://msdn.microsoft.com/library/dn589793.aspx)等模式时。  

#### <a name="considerations"></a>注意事项
如果使用云服务 Web 和辅助角色，请在选择部署后台任务的方式和位置时注意以下要点：

* 在现有 Web 角色中托管后台任务可以节省只对这些任务运行不同辅助角色的成本。 但如果争用进程和其他资源，则可能影响应用程序的性能和可用性。 使用单独的辅助角色可防止长时间运行或资源密集型后台任务影响 Web 角色。
* 如果使用 **RoleEntryPoint** 类托管后台任务，可以轻松地将该类移到另一个角色。 例如，如果在 Web 角色中创建该类，后来确定需要在辅助角色中运行任务，可以将 **RoleEntryPoint** 类实现移入辅助角色。
* 启动任务旨在执行程序或脚本。 将后台作业部署为可执行程序可能更困难，尤其是还需要部署依赖程序集时。 在使用启动任务时，部署和使用脚本定义后台作业可能更容易。
* 导致后台任务失败的异常会造成不同的影响，具体取决于任务的托管方式：
  * 如果使用 **RoleEntryPoint** 类的方式，失败的任务将导致角色重新启动，从而使任务自动重新启动。 这可能会影响应用程序的可用性。 若要避免此问题，请确保在 **RoleEntryPoint** 类和所有后台任务中包含可靠的异常处理。 如果适用，请使用代码来重新启动失败的任务，并且仅当无法正常地从代码失败中恢复时，才引发异常来重新启动角色。
  * 如果使用启动任务，则需要负责管理任务执行，并检查任务是否失败。
* 管理和监视启动任务比使用 **RoleEntryPoint** 类的方式更困难。 但是，Azure Web WebJobs SDK 包含了仪表板，以方便管理通过启动任务启动的 Web 作业。

#### <a name="lifecycle"></a>生命周期 
 如果决定使用 **RoleEntryPoint** 类为使用 Web 角色和辅助角色的云服务应用程序实施后台作业，则必须了解此类的生命周期以正确使用它。

Web 角色和辅助角色在启动、运行和停止时会经历一组不同的阶段。 **RoleEntryPoint** 类将公开一系列事件，用于指示这些阶段何时发生。 可以使用这些事件来初始化、运行和停止自定义后台任务。 整个周期为：

* Azure 加载角色程序集，并在其中搜索派生自 **RoleEntryPoint** 的类。
* 如果找到此类，则调用 **RoleEntryPoint.OnStart()**。 重写此方法可初始化后台任务。
* 完成 **OnStart** 方法后，Azure 将调用应用程序全局文件（如果存在，例如，运行 ASP.NET 的 Web 角色中的 Global.asax）中的 **Application_Start()**。
* Azure 在与 **OnStart()** 对同时执行的新前台线程调用 **RoleEntryPoint.Run()**。 重写此方法可启动后台任务。
* Run 方法结束时，Azure 首先调用应用程序的全局文件中的 **Application_End()**（若存在），然后调用 **RoleEntryPoint.OnStop()**。 可以重写 **OnStop** 方法来停止后台任务、清理资源、处置对象，并关闭任务可能已使用的连接。
* Azure 辅助角色主机进程已停止。 此时，该角色会被回收并重新启动。

有关 **RoleEntryPoint** 类的用法详细信息和示例，请参阅[计算资源整合模式](http://msdn.microsoft.com/library/dn589778.aspx)。

#### <a name="implementation-considerations"></a>实现注意事项

如果正在 Web 或辅助角色中执行后台任务，请注意以下几点：

* **RoleEntryPoint** 类中的默认 **Run** 方法实现包含对 **Thread.Sleep(Timeout.Infinite)**（将角色保持为无限生存）的调用。 若重写 **Run** 方法（执行后台任务通常需要这样做），则不能允许代码从该方法退出，除非想要回收角色实例。
* **Run** 方法的典型实现包含用于启动每个后台任务的代码，以及定期检查所有后台任务状态的循环构造。 它可以重新启动任何失败的任务，或监视用于指示作业已完成的取消标记。
* 如果后台任务引发了未处理的异常，则应该回收该任务，同时允许角色中的任何其他后台任务继续运行。 但是，如果异常是由于任务外部的对象（例如共享存储）损坏所造成的，则应由 **RoleEntryPoint** 类处理异常，应取消所有任务，并允许 **Run** 方法结束。 然后，Azure 将重新启动角色。
* 使用 **OnStop** 方法可以暂停或终止后台任务并清理资源。 这可能涉及到停止长时间运行的任务或多步骤任务。 请务必考虑到这种操作的后果，以避免数据不一致。 如果角色实例出于任何原因（用户启动的关机除外）而停止，**OnStop** 方法中运行的代码必须在五分钟内完成，然后才能将它强行终止。 确保代码可以在这段时间内完成，或者可以容忍无法完成运行。  
* 当 **RoleEntryPoint.OnStart** 方法返回值 **true** 时，Azure 负载均衡器开始将流量定向到角色实例。 因此，请考虑将所有初始化代码置于 **OnStart** 方法中，使未成功初始化的角色实例不会收到任何流量。
* 除了 **RoleEntryPoint** 类的方法以外，还可以使用启动任务。 应该使用启动任务来初始化需要在 Azure 负载均衡器中更改的任何设置，因为在角色接收任何请求之前将执行这些任务。 有关详细信息，请参阅 [Run startup tasks in Azure](/azure/cloud-services/cloud-services-startup-tasks/)（在 Azure 中运行启动任务）。
* 如果启动任务发生错误，它可以强制角色持续重新启动。 这可能阻止虚拟 IP (VIP) 地址交换回之前暂存版本，因为交换需要具有对该角色的独占访问权限。 在角色重新启动时无法做到这一点。 若要解决此问题：
  
  * 将以下代码添加到角色中 **OnStart** 和 **Run** 方法的开头：
    
    ```C#
    var freeze = CloudConfigurationManager.GetSetting("Freeze");
    if (freeze != null)
    {
     if (Boolean.Parse(freeze))
       {
         Thread.Sleep(System.Threading.Timeout.Infinite);
     }
    }
    ```
    
  * 对于角色，请将 **Freeze** 设置的定义作为布尔值添加到 ServiceDefinition.csdef 和 ServiceConfiguration.\*.cscfg 文件，并将它设置为 **false**。 如果角色进入重复重新启动模式，可以将设置更改为 true 以冻结角色的执行，并允许他与以前的版本交换。

#### <a name="more-information"></a>详细信息
* [计算资源整合模式](http://msdn.microsoft.com/library/dn589778.aspx)
* [Azure WebJobs SDK 入门](/azure/app-service-web/websites-dotnet-webjobs-sdk-get-started/)


## <a name="partitioning"></a>分区
如果确定在现有的计算实例（例如 Web 应用、Web 角色、现有辅助角色或虚拟机）中包含后台任务，必须考虑这会如何影响计算实例和后台任务本身的质量属性。 这些因素可帮助你确定是要将任务与现有计算实例放在一起，还是将它们隔离成独立的计算实例：

* **可用性**：后台任务可能无需具有应用程序其他部分所具有的相同可用性级别，特别是直接参与用户交互的 UI 和其他部分。 由于可将操作排入队列，后台任务可能更容许延迟、重试的连接失败，以及影响可用性的其他因素。 但是，必须有足够的容量来防止备份可能阻止队列和影响整个应用程序的请求。
* **伸缩性**：后台任务对 UI 和应用程序的交互部分可能有不同的伸缩性要求。 缩放 UI 可能需要符合需求的高峰，而未完成的后台任务可能在较空闲的时间由较少的计算实例数完成。
* **复原能力**：如果只有托管后台任务的请求可以排入队列或延迟到任务再次可用为止，则这些任务的计算实例失败不会严重影响整个应用程序。 如果计算实例和/或任务可以在适当的间隔内重新启动，则不可以影响应用程序的用户。
* **安全性**：相较于 UI 或应用程序的其他部分，后台任务可能有不同的安全要求或限制。 通过使用单独的计算实例，可以为任务指定不同的安全环境。 还可以使用模式（例如守护程序）将后台计算实例与 UI 相隔离，以最大程度地提供安全性和隔离性。
* **性能**：可以选择专门与任务的性能要求相符的后台任务计算实例类型。 这可能意味着，当任务无需与 UI 具有相同处理功能时，可以使用更便宜的计算选项；或者如果任务需要附加的容量和资源，则使用更大的实例。
* **易管理性**：相较于主应用程序代码或 UI，后台任务可能有不同的开发和部署节奏。 将它们部署到不同的计算实例可简化更新与版本控制。
* **成本**：添加计算实例来执行后台任务会增加托管成本。 应该仔细考虑如何在添加容量与产生的成本之间做出取舍。

有关详细信息，请参阅 [Leader Election Pattern](http://msdn.microsoft.com/library/dn568104.aspx)（领导选拔模式）和 [Competing Consumers Pattern](http://msdn.microsoft.com/library/dn568101.aspx)（使用者竞争模式）。

## <a name="conflicts"></a>冲突
如果有后台作业的多个实例，这些实例可能会争用对资源和服务（例如数据库和存储）的访问权。 这种并发访问可能会导致资源争用情况，从而造成服务可用性及存储中数据完整性的冲突。 可以使用悲观锁定方法来解决资源争用。 这可以防止任务的竞争实例同时访问某个服务或损坏数据。

另一种解决冲突的方法是将后台任务定义为单一实例，以便只有一个运行中的实例。 但是，这会消除多实例配置可提供的可靠性和性能优势。 当 UI 可以提供足够的工作让多个后台任务保持繁忙时尤其如此。

必须确保后台任务可以自动重新启动，并且有足够的容量来应对需求高峰。 这可以通过将足够的资源分配给计算实例、实施队列机制（可存储请求以便日后续需求降低时执行）或使用这些方法的组合来实现此目的。

## <a name="coordination"></a>协调
后台任务可能很复杂，需要执行多个不同的任务来生成结果或满足所有要求。 在这些方案中，住往会将任务分割成可由多个使用者执行的较小离散步骤或子任务。 多步骤操作可能更有效率且更具弹性，因为单个步骤可以在多个作业中重复使用。 还可以轻松地添加、删除或修改步骤的顺序。

协调多个任务和步骤可能相当困难，但可以参考三种常见的模式来实施解决方案：

* **将一个任务分解成多个可重复使用的步骤**。 对于处理的信息，应用程序可能需要执行复杂性不一的各种任务。 实施此应用程序的一种直接但有弹性的方法是将这种处理当作单一模块来执行。 但是，这种方法可能会降低重构代码、优化代码或在应用程序的其他位置需要相同处理的部分时重复使用代码的机会。 有关详细信息，请参阅[管道和筛选器模式](http://msdn.microsoft.com/library/dn568100.aspx)。
* **管理任务的步骤执行**。 应用程序可以执行包含多个步骤的任务（其中有些步骤可以调用远程服务或访问远程资源）。 各个步骤可以彼此独立，但它们由实施任务的应用程序逻辑进行协调。 有关详细信息，请参阅[计划程序代理监督程序模式](http://msdn.microsoft.com/library/dn589780.aspx)。
* **管理失败任务步骤的恢复**。 如果一个或多个步骤失败，应用程序可能需要撤消一系列步骤执行的工作（所有步骤共同定义了最终一致的操作）。 有关详细信息，请参阅[补偿事务模式](http://msdn.microsoft.com/library/dn589804.aspx)。


## <a name="resiliency-considerations"></a>复原注意事项
后台任务必须具有复原能力，以便为应用程序提供可靠的服务。 规划和设计后台任务时，请注意以下几点：

* 后台任务必须能够正常处理角色或服务重新启动，而不会损坏数据或导致应用程序不一致。 对于长时间运行的任务或多步骤任务，请考虑使用*检查点*，方法是在永久性存储中保存作业状态，或者在队列中将作业状态另存为消息（如果适当）。 例如，可以在队列的消息中永久保存状态信息，并根据任务进度增量更新此状态信息，以便从上次已知正常的检查点处理任务，而不必从头重新开始。 使用 Azure 服务总线队列时，可以使用消息会话来实现相同的方案。 会话允许用户使用 [SetState](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.messagesession.setstate?view=azureservicebus-4.0.0) 和 [GetState](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.messagesession.getstate?view=azureservicebus-4.0.0) 方法来保存和检索应用程序处理状态。 有关设计可靠的多步骤过程和工作流的详细信息，请参阅 [Scheduler Agent Supervisor Pattern](http://msdn.microsoft.com/library/dn589780.aspx)（计划程序代理监督程序模式）。
* 使用 Web 角色或辅助角色托管多个后台任务时，请设计 **Run** 方法的重写来监视失败或已停止的任务，并将其重新启动。 如果这不可行，并且你正在使用辅助角色，可从 Run 方法中退出，强制执行辅助角色以重新启动。
* 使用队列来与后台任务通信时，队列可以充当缓冲区，用于在应用程序超过一般负载时，存储发送给任务的请求。 这样，任务便可以在相对空闲期间与 UI 同步。 这也意味着，回收角色不会阻止 UI。 有关详细信息，请参阅[基于队列的负载调节模式](http://msdn.microsoft.com/library/dn589783.aspx)。 如果某些任务比其他任务更重要，请考虑实施[优先级队列模式](http://msdn.microsoft.com/library/dn589794.aspx)，确保这些任务在较不重要的任务之前运行。
* 必须将消息或进程消息启动的后台任务设计为处理不一致情况，例如消息以错误顺序到达、消息重复导致错误（通常称为*有害消息*）和消息传送多次。 请注意以下几点：
  * 必须按特定顺序处理消息，例如，根据数据的现有数据值更改数据的消息（例如，将值添加到现有值）可能不以其原始发送顺序到达。 或者，可能因为每个实例上的负载不同，后台任务的不同实例按不同的顺序处理消息。 必须按特定顺序处理的消息应该包括序号、键，或者可由后台任务用来确保按正确顺序处理这些消息的其他某个指示器。 如果使用 Azure 服务总线，可以使用消息会话来保证传送顺序。 但是，尽可能设计好过程，使消息顺序变得不重要的思路通常更有效率。
  * 一般而言，后台任务会在队列中扫视消息，这会暂时向其他消息使用者隐藏消息。 然后，它会在成功处理消息后，将其删除。 如果后台任务在处理某个消息时失败，该消息会在扫视超时后重新出现在队列中。 该消息由任务的另一个实例处理，或在此实例的下一个处理周期进行处理。 如果消息一直导致使用者出错，则会阻止任务、队列，并最终在队列填满时阻止应用程序本身。 因此，请务必在队列中检测并删除有害消息。 如果使用 Azure 服务总线，导致出错的消息可以自动或手动移到关联的死信队列。
  * 系统为队列保证*至少一次*传送机制，但队列可能会多次传送同一条消息。 此外，如果在处理消息之后、从队列中删除消息之前后台任务失败，消息将可再次处理。 后台任务应该具有幂等性，这意味着多次处理同一条消息不会导致错误，或者使应用程序的数据不一致。 某些操作原生就是幂等的，例如，将存储的值设置为特定的新值。 但是，有些操作（例如，将值添加到现有的存储值而不检查存储值是否仍与最初发送的消息相同）会导致不一致情况。 可将 Azure 服务总线队列配置为自动删除重复消息。
  * 某些消息传送系统（例如 Azure 存储队列和 Azure 服务总线队列）支持用于指示已从队列中读取消息次数的取消排队计数属性。 这在处理重复消息和有害消息时可能很有用。 有关详细信息，请参阅[异步消息传送入门](http://msdn.microsoft.com/library/dn589781.aspx)和[幂等模式](http://blog.jonathanoliver.com/idempotency-patterns/)。

## <a name="scaling-and-performance-considerations"></a>缩放和性能注意事项
后台任务必须提供足够的性能，确保它们不会阻止应用程序，或者不会因系统负载不足而延迟操作时导致不一致。 通常，可以通过缩放托管后台任务的计算实例来提高性能。 规划和设计后台任务时，请注意以下有关伸缩性和性能的要点：

* Azure 根据当前的需求和负载或预定义的计划，支持对 Web 应用、云服务 Web 角色和辅助角色以及虚拟机托管的部署使用自动缩放（向外缩放和向内缩放）。 使用此功能可确保整个应用程序具有足够的性能，同时将运行时成本降到最低。
* 当后台任务具有与云服务应用程序其他部分（例如，UI 或数据访问层等组件）的不同的性能时，将不同辅助角色的后台任务托管在一起可让 UI 和辅助任务角色单独进行缩放以管理负载。 如果多个后台任务彼此有明显不同的性能，请考虑将它们分割成不同的辅助角色并单独缩放每个角色类型。 但请注意，相比将所有任务合并成较少的角色，这可能会增加运行时成本。
* 只是缩放角色可能不足以防止在低负载下损失性能。 还可能需要缩放存储队列和其他资源，以防止整体处理链的单个点变成瓶颈。 另外，请考虑其他限制，例如存储的最大吞吐量，以及应用程序的其他服务和后台任务依赖的服务。
* 必须针对缩放设计后台任务。 例如，后台任务必须能够动态检测正在使用的存储队列数，以侦听相应的队列或向其发送消息。
* 默认情况下，Web 作业会随着其关联的 Azure Web 应用实例进行缩放。 但是，如果只想要将 Web 作业当作单个实例运行，可以创建包含 JSON 数据 { "is_singleton": true } 的 Settings.job 文件。 这会强制 Azure 只运行 Web 作业的一个实例，即使关联的 Web 应用有多个实例。 对于必须以单个实例运行的计划作业而言，这可能是有用的方法。

## <a name="related-patterns"></a>相关模式
* [异步消息传送入门](http://msdn.microsoft.com/library/dn589781.aspx)
* [自动缩放指南](http://msdn.microsoft.com/library/dn589774.aspx)
* [补偿事务模式](http://msdn.microsoft.com/library/dn589804.aspx)
* [使用者竞争模式](http://msdn.microsoft.com/library/dn568101.aspx)
* [计算分区指南](http://msdn.microsoft.com/library/dn589773.aspx)
* [计算资源整合模式](http://msdn.microsoft.com/library/dn589778.aspx)
* [守护程序模式](http://msdn.microsoft.com/library/dn589793.aspx)
* [领导选拔模式](http://msdn.microsoft.com/library/dn568104.aspx)
* [管道和筛选器模式](http://msdn.microsoft.com/library/dn568100.aspx)
* [优先级队列模式](http://msdn.microsoft.com/library/dn589794.aspx)
* [基于队列的负载调节模式](http://msdn.microsoft.com/library/dn589783.aspx)
* [计划程序代理监督程序模式](http://msdn.microsoft.com/library/dn589780.aspx)

## <a name="more-information"></a>详细信息
* [缩放包含辅助角色的 Azure 应用程序](http://msdn.microsoft.com/library/hh534484.aspx#sec8)
* [执行后台任务](http://msdn.microsoft.com/library/ff803365.aspx)
* [Azure 角色启动生命周期](http://blog.syntaxc4.net/post/2011/04/13/windows-azure-role-startup-life-cycle.aspx)（博客文章）
* [Azure 云服务角色生命周期](http://channel9.msdn.com/Series/Windows-Azure-Cloud-Services-Tutorials/Windows-Azure-Cloud-Services-Role-Lifecycle)（视频）
* [什么是 Azure WebJobs SDK](https://docs.microsoft.com/azure/app-service-web/websites-dotnet-webjobs-sdk)
* [使用 WebJobs 运行后台任务](https://docs.microsoft.com/azure/app-service-web/web-sites-create-web-jobs)
* [Azure 队列和服务总线队列 - 比较与对照](https://docs.microsoft.com/azure/service-bus-messaging/service-bus-azure-and-service-bus-queues-compared-contrasted)
* [如何在云服务中启用诊断](https://docs.microsoft.com/azure/cloud-services/cloud-services-dotnet-diagnostics)

