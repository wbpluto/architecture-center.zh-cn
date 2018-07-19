---
title: 使用 Jenkins 和 Azure Kubernetes 服务进行 DevOps
description: 经验证的解决方案，所生成的 DevOps 管道适合使用 Jenkins、Azure 容器注册表、Azure Kubernetes 服务、Cosmos DB 和 Grafana 的 Node.js Web 应用。
author: iainfoulds
ms.date: 07/05/2018
ms.openlocfilehash: 9dbd1cb335f1c03c54ea342466594048e907d040
ms.sourcegitcommit: 5d99b195388b7cabba383c49a81390ac48f86e8a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/06/2018
ms.locfileid: "37891270"
---
# <a name="deploy-a-container-based-devops-pipeline-for-modern-application-development-with-jenkins-and-azure-kubernetes-service"></a><span data-ttu-id="c1257-103">部署基于容器的 DevOps 管道，适合使用 Jenkins 和 Azure Kubernetes 服务进行的现代应用程序开发</span><span class="sxs-lookup"><span data-stu-id="c1257-103">Deploy a container-based DevOps pipeline for modern application development with Jenkins and Azure Kubernetes Service</span></span>

<span data-ttu-id="c1257-104">本示例方案适用于需要通过容器和 DevOps 工作流实现应用程序开发现代化的企业。</span><span class="sxs-lookup"><span data-stu-id="c1257-104">This example scenario is applicable to businesses that want to modernize application development by using containers and DevOps workflows.</span></span> <span data-ttu-id="c1257-105">在本解决方案中，将会通过 Jenkins 生成一个 Node.js Web 应用并将其部署到 Azure 容器注册表和 Azure Kubernetes 服务中。</span><span class="sxs-lookup"><span data-stu-id="c1257-105">In this solution, a Node.js web app is built and deployed by Jenkins into an Azure Container Registry and Azure Kubernetes Service.</span></span> <span data-ttu-id="c1257-106">为了实现全局分布式数据库层，将使用 Azure Cosmos DB。</span><span class="sxs-lookup"><span data-stu-id="c1257-106">For a globally distributed database tier, Azure Cosmos DB is used.</span></span> <span data-ttu-id="c1257-107">为了监视应用程序性能并排查其问题，Azure Monitor 会与 Grafana 实例及仪表板集成。</span><span class="sxs-lookup"><span data-stu-id="c1257-107">To monitor and troubleshoot application performance, Azure Monitor integrates with a Grafana instance and dashboard.</span></span>

<span data-ttu-id="c1257-108">示例应用程序方案包括提供自动化开发环境、验证新的代码提交，以及将新部署推送到过渡环境或生产环境中。</span><span class="sxs-lookup"><span data-stu-id="c1257-108">Example application scenarios include providing an automated development environment, validating new code commits, and pushing new deployments into staging or production environments.</span></span> <span data-ttu-id="c1257-109">传统上，企业必须手动生成和编译应用程序和更新，维持一个大型代码库。</span><span class="sxs-lookup"><span data-stu-id="c1257-109">Traditionally, businesses had to manually build and compile applications and updates, and maintain a large, monolithic code base.</span></span> <span data-ttu-id="c1257-110">现代的应用程序开发方法使用持续集成 (CI) 和持续部署 (CD)，可以更快速地生成、测试和部署服务。</span><span class="sxs-lookup"><span data-stu-id="c1257-110">With a modern approach to application development that uses continuous integration (CI) and continuous deployment (CD), you can more quickly build, test, and deploy services.</span></span> <span data-ttu-id="c1257-111">此现代方法可以更快速地将应用程序和更新发布给客户，更灵活地响应不断变化的业务需求。</span><span class="sxs-lookup"><span data-stu-id="c1257-111">This modern approach lets you release applications and updates to your customers faster, and respond to changing business demands in a more agile manner.</span></span>

<span data-ttu-id="c1257-112">有了 Azure Kubernetes 服务、容器注册表和 Cosmos DB 之类的 Azure 服务，公司就可以使用最新的应用程序开发技术和工具来简化高可用性的实现过程。</span><span class="sxs-lookup"><span data-stu-id="c1257-112">By using Azure services such as Azure Kubernetes Service, Container Registry, and Cosmos DB, companies can use the latest in application development techniques and tools to simplify the process of implementing high availability.</span></span>

## <a name="potential-use-cases"></a><span data-ttu-id="c1257-113">可能的用例</span><span class="sxs-lookup"><span data-stu-id="c1257-113">Potential use cases</span></span>

<span data-ttu-id="c1257-114">以下用例可以考虑本解决方案：</span><span class="sxs-lookup"><span data-stu-id="c1257-114">Consider this solution for the following use cases:</span></span>

* <span data-ttu-id="c1257-115">改革应用程序开发做法，采用微服务式的基于容器的方法。</span><span class="sxs-lookup"><span data-stu-id="c1257-115">Modernizing application development practices to a microservices, container-based approach.</span></span>
* <span data-ttu-id="c1257-116">缩短应用程序开发和部署生命周期。</span><span class="sxs-lookup"><span data-stu-id="c1257-116">Speeding up application development and deployment lifecycles.</span></span>
* <span data-ttu-id="c1257-117">自动部署到测试或验收环境进行验证。</span><span class="sxs-lookup"><span data-stu-id="c1257-117">Automating deployments to test or acceptance environments for validation.</span></span>

## <a name="architecture"></a><span data-ttu-id="c1257-118">体系结构</span><span class="sxs-lookup"><span data-stu-id="c1257-118">Architecture</span></span>

![从体系结构的角度概要说明某个使用 Jenkins、Azure 容器注册表和 Azure Kubernetes 服务的 DevOps 解决方案中涉及的 Azure 组件][architecture]

<span data-ttu-id="c1257-120">本解决方案涉及一个用于 Node.js Web 应用程序和数据库后端的 DevOps 管道。</span><span class="sxs-lookup"><span data-stu-id="c1257-120">This solution covers a DevOps pipeline for a Node.js web application and database backend.</span></span> <span data-ttu-id="c1257-121">数据流经解决方案的情形如下所示：</span><span class="sxs-lookup"><span data-stu-id="c1257-121">The data flows through the solution as follows:</span></span>

1. <span data-ttu-id="c1257-122">开发人员更改 Node.js Web 应用程序源代码。</span><span class="sxs-lookup"><span data-stu-id="c1257-122">A developer makes changes to the Node.js web application source code.</span></span>
2. <span data-ttu-id="c1257-123">所做的代码更改提交到某个源代码管理存储库，例如 GitHub。</span><span class="sxs-lookup"><span data-stu-id="c1257-123">The code change is committed to a source control respository, such as GitHub.</span></span>
3. <span data-ttu-id="c1257-124">为了启动持续集成 (CI) 过程，某个 GitHub Webhook 会触发一个 Jenkins 项目生成。</span><span class="sxs-lookup"><span data-stu-id="c1257-124">To start the continuous integration (CI) process, a GitHub webhook triggers a Jenkins project build.</span></span>
4. <span data-ttu-id="c1257-125">Jenkins 生成作业使用 Azure Kubernetes 服务中的动态生成代理来执行容器生成过程。</span><span class="sxs-lookup"><span data-stu-id="c1257-125">The Jenkins build job uses a dynamic build agent in Azure Kubernetes Service to perform a container build process.</span></span>
5. <span data-ttu-id="c1257-126">系统会根据源代码管理中的代码创建容器映像并将其推送到 Azure 容器注册表。</span><span class="sxs-lookup"><span data-stu-id="c1257-126">A container image is created from the code in source control, and is then pushed to an Azure Container Registry.</span></span>
6. <span data-ttu-id="c1257-127">Jenkins 通过持续部署 (CD) 将这个更新的容器映像部署到 Kubernetes 群集。</span><span class="sxs-lookup"><span data-stu-id="c1257-127">Through continuous deployment (CD), Jenkins deploys this updated container image to the Kubernetes cluster.</span></span>
7. <span data-ttu-id="c1257-128">Node.js Web 应用程序将 Azure Cosmos DB 用作其后端。</span><span class="sxs-lookup"><span data-stu-id="c1257-128">The Node.js web application uses Azure Cosmos DB as it's backend.</span></span> <span data-ttu-id="c1257-129">Cosmos DB 和 Azure Kubernetes 服务都会将指标报告给 Azure Monitor。</span><span class="sxs-lookup"><span data-stu-id="c1257-129">Both Cosmos DB and Azure Kubernetes Service report metrics to Azure Monitor.</span></span>
8. <span data-ttu-id="c1257-130">Grafana 实例根据 Azure Monitor 提供的数据在仪表板上直观显示应用程序性能。</span><span class="sxs-lookup"><span data-stu-id="c1257-130">A Grafana instance provides visual dashboards of the application performance based on the data from Azure Monitor.</span></span>

### <a name="components"></a><span data-ttu-id="c1257-131">组件</span><span class="sxs-lookup"><span data-stu-id="c1257-131">Components</span></span>

* <span data-ttu-id="c1257-132">[Jenkins][jenkins] 是一种开源的自动化服务器，与 Azure 服务集成后即可进行持续集成 (CI) 和持续部署 (CD)。</span><span class="sxs-lookup"><span data-stu-id="c1257-132">[Jenkins][jenkins] is an open-source automation server that can integrate with Azure services to enable continuous integration (CI) and continuous deployment (CD).</span></span> <span data-ttu-id="c1257-133">在本解决方案中，Jenkins 会根据提交到源代码管理中的内容协调新容器映像的创建过程，接着将这些映像推送到 Azure 容器注册表，然后更新 Azure Kubernetes 服务中的应用程序实例。</span><span class="sxs-lookup"><span data-stu-id="c1257-133">In this solution, Jenkins orchestrates the creation of new container images based on commits to source control, pushes those images to Azure Container Registry, then updates application instances in Azure Kubernetes Service.</span></span>
* <span data-ttu-id="c1257-134">[Azure Linux 虚拟机][azurevm-docs]用于运行 Jenkins 和 Grafana 实例。</span><span class="sxs-lookup"><span data-stu-id="c1257-134">[Azure Linux Virtual Machines][azurevm-docs] are used to run the Jenkins and Grafana instances.</span></span>
* <span data-ttu-id="c1257-135">[Azure 容器注册表][azureacr-docs]存储和管理 Azure Kubernetes 服务群集使用的容器映像。</span><span class="sxs-lookup"><span data-stu-id="c1257-135">[Azure Container Registry][azureacr-docs] stores and manages container images that are used by the Azure Kubernetes Service cluster.</span></span> <span data-ttu-id="c1257-136">映像会以安全的方式进行存储，并可通过 Azure 平台复制到其他区域以加快部署速度。</span><span class="sxs-lookup"><span data-stu-id="c1257-136">Images are securely stored, and can replicated to other regions by the Azure platform to speed up deployment times.</span></span>
* <span data-ttu-id="c1257-137">[Azure Kubernetes 服务][azureaks-docs]是一种托管的 Kubernetes 平台，可以让用户在没有容器业务流程专业知识的情况下部署和管理容器化的应用程序。</span><span class="sxs-lookup"><span data-stu-id="c1257-137">[Azure Kubernetes Service][azureaks-docs] is a managed Kubernetes platform that lets you deploy and manage containerized applications without container orchestration expertise.</span></span> <span data-ttu-id="c1257-138">作为一个托管 Kubernetes 服务，Azure 可以自动处理运行状况监视和维护等关键任务。</span><span class="sxs-lookup"><span data-stu-id="c1257-138">As a hosted Kubernetes service, Azure handles critical tasks like health monitoring and maintenance for you.</span></span>
* <span data-ttu-id="c1257-139">[Azure Cosmos DB][azurecosmosdb-docs] 是一种全局分布式多模型数据库，允许用户根据需要选择不同的数据库和一致性模型。</span><span class="sxs-lookup"><span data-stu-id="c1257-139">[Azure Cosmos DB][azurecosmosdb-docs] is a globally distributed, multi-model database that allows you to choose from various database and consistency models to suit your needs.</span></span> <span data-ttu-id="c1257-140">Cosmos DB 允许全局复制数据，且不需部署和配置群集管理或复制组件。</span><span class="sxs-lookup"><span data-stu-id="c1257-140">With Cosmos DB, your data can be globally replicated, and there is no cluster management or replication components to deploy and configure.</span></span>
* <span data-ttu-id="c1257-141">[Azure Monitor][azuremonitor-docs] 可以跟踪性能、维护安全和确定趋势。</span><span class="sxs-lookup"><span data-stu-id="c1257-141">[Azure Monitor][azuremonitor-docs] helps you track performance, maintain security, and identify trends.</span></span> <span data-ttu-id="c1257-142">Monitor 获得的指标可供其他资源和工具（例如 Grafana）使用。</span><span class="sxs-lookup"><span data-stu-id="c1257-142">Metrics obtained by Monitor can be used by other resources and tools, such as Grafana.</span></span>
* <span data-ttu-id="c1257-143">[Grafana][grafana] 是一种用于查询、可视化、警示和了解指标的开源解决方案。</span><span class="sxs-lookup"><span data-stu-id="c1257-143">[Grafana][grafana] is an open-source solution to query, visualize, alert, and understand metrics.</span></span> <span data-ttu-id="c1257-144">Grafana 可以通过 Azure Monitor 的数据源插件创建直观的仪表板，以便监视在 Azure Kubernetes 服务中运行并使用 Cosmos DB 的应用程序的性能。</span><span class="sxs-lookup"><span data-stu-id="c1257-144">A data source plugin for Azure Monitor allows Grafana to create visual dashboards to monitor the performance of your applications running in Azure Kubernetes Service and using Cosmos DB.</span></span>

### <a name="alternatives"></a><span data-ttu-id="c1257-145">备选项</span><span class="sxs-lookup"><span data-stu-id="c1257-145">Alternatives</span></span>

* <span data-ttu-id="c1257-146">[Visual Studio Team Services][vsts] 和 Team Foundation Server 可以为任何应用实现一个持续集成 (CI)、测试和部署 (CD) 管道。</span><span class="sxs-lookup"><span data-stu-id="c1257-146">[Visual Studio Team Services][vsts] and Team Foundation Server help you implement a continuous integration (CI), test, and deployment (CD) pipeline for any app.</span></span>
* <span data-ttu-id="c1257-147">若要对群集进行更多的控制，可以直接在 Azure VM 上运行 [Kubernetes][kubernetes]，不必通过托管服务来运行。</span><span class="sxs-lookup"><span data-stu-id="c1257-147">[Kubernetes][kubernetes] can be run directly on Azure VMs instead of via a managed service if you would like more control over the cluster.</span></span>
* <span data-ttu-id="c1257-148">[Service Fabric][service-fabric] 是另一个可以替代 AKS 的容器业务流程协调程序。</span><span class="sxs-lookup"><span data-stu-id="c1257-148">[Service Fabric][service-fabric] is another alternate container orchestrator that can replace AKS.</span></span>

## <a name="considerations"></a><span data-ttu-id="c1257-149">注意事项</span><span class="sxs-lookup"><span data-stu-id="c1257-149">Considerations</span></span>

### <a name="availability"></a><span data-ttu-id="c1257-150">可用性</span><span class="sxs-lookup"><span data-stu-id="c1257-150">Availability</span></span>

<span data-ttu-id="c1257-151">为了监视应用程序性能并报告问题，本解决方案将 Azure Monitor 和 Grafana 组合在一起，以便创建直观的仪表板。</span><span class="sxs-lookup"><span data-stu-id="c1257-151">To monitor your application performance and report on issues, this solution combines Azure Monitor with Grafana for visual dashboards.</span></span> <span data-ttu-id="c1257-152">可以通过这些工具监视和排查性能问题，这些问题可能需要代码更新，而这些代码更新均可通过 CI/CD 管道进行部署。</span><span class="sxs-lookup"><span data-stu-id="c1257-152">These tools let you monitor and troubleshoot performance issues that may require code updates, which can all then be deployed with the CI/CD pipeline.</span></span>

<span data-ttu-id="c1257-153">作为 Azure Kubernetes 服务群集的一部分，负载均衡器可以将应用程序流量分发到一个或多个运行应用程序的容器 (Pod)。</span><span class="sxs-lookup"><span data-stu-id="c1257-153">As part of the Azure Kubernetes Service cluster, a load balancer distributes application traffic to one or more containers (pods) that run your application.</span></span> <span data-ttu-id="c1257-154">可以通过这种在 Kubernetes 中运行容器化应用程序的方法，为客户提供高度可用的基础结构。</span><span class="sxs-lookup"><span data-stu-id="c1257-154">This approach to running containerized applications in Kubernetes provides a highly-available infrastructure for your customers.</span></span>

<span data-ttu-id="c1257-155">若要了解其他可伸缩性主题，请参阅体系结构中心提供的[可用性核对清单][availability]。</span><span class="sxs-lookup"><span data-stu-id="c1257-155">For other scalability topics, see the [availability checklist][availability] available in the architecure center.</span></span>

### <a name="scalability"></a><span data-ttu-id="c1257-156">可伸缩性</span><span class="sxs-lookup"><span data-stu-id="c1257-156">Scalability</span></span>

<span data-ttu-id="c1257-157">Azure Kubernetes 服务允许按照应用程序的需求调整群集节点的数目。</span><span class="sxs-lookup"><span data-stu-id="c1257-157">Azure Kubernetes Service lets you scale the number of cluster nodes to meet the demands of your applications.</span></span> <span data-ttu-id="c1257-158">应用程序增加时，可以扩大运行服务的 Kubernetes 节点数。</span><span class="sxs-lookup"><span data-stu-id="c1257-158">As your application increases, you can scale out the number of Kubernetes nodes that run your service.</span></span>

<span data-ttu-id="c1257-159">应用程序数据存储在 Azure Cosmos DB 中，后者是一种全局分布式多模型数据库，可以进行全局缩放。</span><span class="sxs-lookup"><span data-stu-id="c1257-159">Application data is stored in Azure Cosmos DB, a globally distributed, multi-model database that can scale globally.</span></span> <span data-ttu-id="c1257-160">Cosmos DB 可以使用传统的数据库组件按需求来缩放基础结构，你可以根据客户的需求选择对 Cosmos DB 进行全局复制。</span><span class="sxs-lookup"><span data-stu-id="c1257-160">Cosmos DB abstracts the need to scale your  infrastructure as with traditional database components, and you can choose to replicate your Cosmos DB globally to meet the demands of your customers.</span></span>

<span data-ttu-id="c1257-161">若要了解其他可伸缩性主题，请参阅体系结构中心提供的[可伸缩性核对清单][scalability]。</span><span class="sxs-lookup"><span data-stu-id="c1257-161">For other scalability topics, see the [scalability checklist][scalability] available in the architecure center.</span></span>

### <a name="security"></a><span data-ttu-id="c1257-162">“安全”</span><span class="sxs-lookup"><span data-stu-id="c1257-162">Security</span></span>

<span data-ttu-id="c1257-163">为了尽量减少受攻击面，本解决方案不通过 HTTP 公开 Jenkins VM 实例。</span><span class="sxs-lookup"><span data-stu-id="c1257-163">To minimize the attack footprint, this solutions does not expose the Jenkins VM instance over HTTP.</span></span> <span data-ttu-id="c1257-164">若要执行需要与 Jenkins 交互的管理任务，请使用本地计算机的 SSH 隧道创建安全的远程连接。</span><span class="sxs-lookup"><span data-stu-id="c1257-164">For any management tasks that require you to interact with Jenkins, you create a secure remote connection using an SSH tunnel from your local machine.</span></span> <span data-ttu-id="c1257-165">Jenkins 和 Grafana VM 实例仅允许 SSH 公钥身份验证。</span><span class="sxs-lookup"><span data-stu-id="c1257-165">Only SSH public key authentication is allowed for the Jenkins and Grafana VM instances.</span></span> <span data-ttu-id="c1257-166">基于密码的登录已禁用。</span><span class="sxs-lookup"><span data-stu-id="c1257-166">Password-based logins are disabled.</span></span> <span data-ttu-id="c1257-167">有关详细信息，请参阅[在 Azure 上运行 Jenkins 服务器](../../reference-architectures/jenkins/index.md)。</span><span class="sxs-lookup"><span data-stu-id="c1257-167">For more information, see [Run a Jenkins server on Azure](../../reference-architectures/jenkins/index.md).</span></span>

<span data-ttu-id="c1257-168">为了将凭据和权限隔离，本解决方案使用专用的 Azure Active Directory (AD) 服务主体。</span><span class="sxs-lookup"><span data-stu-id="c1257-168">For separation of credentials and permissions, this solution uses a dedicated Azure Active Directory (AD) service principal.</span></span> <span data-ttu-id="c1257-169">此服务主体的凭据以安全的凭据对象形式存储在 Jenkins 中，因此不会在脚本或生成管道中直接公开和可见。</span><span class="sxs-lookup"><span data-stu-id="c1257-169">The credentials for this service principal are stored as a secure credential object in Jenkins so that they are not directly exposed and visible within scripts or the build pipeline.</span></span>

<span data-ttu-id="c1257-170">若需安全解决方案的通用设计指南，请参阅 [Azure 安全性文档][security]。</span><span class="sxs-lookup"><span data-stu-id="c1257-170">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="c1257-171">复原</span><span class="sxs-lookup"><span data-stu-id="c1257-171">Resiliency</span></span>

<span data-ttu-id="c1257-172">本解决方案使用适合应用程序的 Azure Kubernetes 服务。</span><span class="sxs-lookup"><span data-stu-id="c1257-172">This solution uses Azure Kubernetes Service for your application.</span></span> <span data-ttu-id="c1257-173">Kubernetes 内置了复原组件，用于监视容器 (Pod) 并在出现问题时重启容器。</span><span class="sxs-lookup"><span data-stu-id="c1257-173">Built into Kubernetes are resiliency components that monitor and restart the containers (pods) if there is an issue.</span></span> <span data-ttu-id="c1257-174">应用程序可以运行多个 Kubernetes 节点，可以容忍一个 Pod 或节点不可用的情况。</span><span class="sxs-lookup"><span data-stu-id="c1257-174">Combined with running multiple Kubernetes nodes, your application can tolerate a pod or node being unavailable.</span></span>

<span data-ttu-id="c1257-175">若需可复原解决方案的通用设计指南，请参阅[设计适用于 Azure 的可复原应用程序][resiliency]。</span><span class="sxs-lookup"><span data-stu-id="c1257-175">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="c1257-176">部署解决方案</span><span class="sxs-lookup"><span data-stu-id="c1257-176">Deploy the solution</span></span>

<span data-ttu-id="c1257-177">**先决条件：**</span><span class="sxs-lookup"><span data-stu-id="c1257-177">**Prerequisites.**</span></span>

* <span data-ttu-id="c1257-178">必须已经有 Azure 帐户。</span><span class="sxs-lookup"><span data-stu-id="c1257-178">You must have an existing Azure account.</span></span> <span data-ttu-id="c1257-179">如果没有 Azure 订阅，请在开始之前创建一个[免费帐户](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)。</span><span class="sxs-lookup"><span data-stu-id="c1257-179">If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.</span></span>
* <span data-ttu-id="c1257-180">需要一个 SSH 公钥对。</span><span class="sxs-lookup"><span data-stu-id="c1257-180">You need an SSH public key pair.</span></span> <span data-ttu-id="c1257-181">有关如何创建公钥对的步骤，请参阅[创建和使用适合 Linux VM 的 SSH 公钥对][sshkeydocs]。</span><span class="sxs-lookup"><span data-stu-id="c1257-181">For steps on how to create a public key pair, see [Create and use an SSH key pair for Linux VMs][sshkeydocs].</span></span>
* <span data-ttu-id="c1257-182">需要一个 Azure Active Directory (AD) 服务主体，以便对服务和资源进行身份验证。</span><span class="sxs-lookup"><span data-stu-id="c1257-182">You need an Azure Active Directory (AD) service principal for the authentication of service and resources.</span></span> <span data-ttu-id="c1257-183">可以根据需要使用 [az ad sp create-for-rbac][createsp] 创建一个服务主体</span><span class="sxs-lookup"><span data-stu-id="c1257-183">If needed, you can create a service principal with [az ad sp create-for-rbac][createsp]</span></span>

    ```azurecli-interactive
    az ad sp create-for-rbac --name myDevOpsSolution
    ```

    <span data-ttu-id="c1257-184">请记下此命令输出中的 *appId* 和 *password*。</span><span class="sxs-lookup"><span data-stu-id="c1257-184">Make a note of the *appId* and *password* in the output from this command.</span></span> <span data-ttu-id="c1257-185">部署解决方案时，请将这些值提供给模板。</span><span class="sxs-lookup"><span data-stu-id="c1257-185">You provide these values to the template when you deploy the solution.</span></span>

<span data-ttu-id="c1257-186">若要通过 Azure 资源管理器模板部署此解决方案，请执行以下步骤。</span><span class="sxs-lookup"><span data-stu-id="c1257-186">To deploy this solution with an Azure Resource Manager template, perform the following steps.</span></span>

1. <span data-ttu-id="c1257-187">单击“部署到 Azure”按钮：</span><span class="sxs-lookup"><span data-stu-id="c1257-187">Click the **Deploy to Azure** button:</span></span><br><a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmspnp%2Fsolution-architectures%2Fmaster%2Fapps%2Fdevops-with-aks%2Fazuredeploy.json" target="_blank"><img src="https://azuredeploy.net/deploybutton.png"/></a>
2. <span data-ttu-id="c1257-188">等待模板部署在 Azure 门户中打开，然后完成以下步骤：</span><span class="sxs-lookup"><span data-stu-id="c1257-188">Wait for the template deployment to open in the Azure portal, then complete the following steps:</span></span>
   * <span data-ttu-id="c1257-189">选择“新建”资源组，然后在文本框中提供一个名称，例如 *myAKSDevOpsSolution*。</span><span class="sxs-lookup"><span data-stu-id="c1257-189">Choose to **Create new** resource group, then provide a name such as *myAKSDevOpsSolution* in the text box.</span></span>
   * <span data-ttu-id="c1257-190">从“位置”下拉框中选择区域。</span><span class="sxs-lookup"><span data-stu-id="c1257-190">Select a region from the **Location** drop-down box.</span></span>
   * <span data-ttu-id="c1257-191">通过 `az ad sp create-for-rbac` 命令输入服务主体应用 ID 和密码。</span><span class="sxs-lookup"><span data-stu-id="c1257-191">Enter your service principal app ID and password from the `az ad sp create-for-rbac` command.</span></span>
   * <span data-ttu-id="c1257-192">提供用于 Jenkins 实例和 Grafana 控制台的用户名和安全密码。</span><span class="sxs-lookup"><span data-stu-id="c1257-192">Provide a username and secure password for the Jenkins instance and Grafana console.</span></span>
   * <span data-ttu-id="c1257-193">提供 SSH 密钥，确保安全地登录到 Linux VM。</span><span class="sxs-lookup"><span data-stu-id="c1257-193">Provide an SSH key to secure logins to the Linux VMs.</span></span>
   * <span data-ttu-id="c1257-194">查看条款和条件，然后勾选“我同意上述条款和条件”。</span><span class="sxs-lookup"><span data-stu-id="c1257-194">Review the terms and conditions, then check **I agree to the terms and conditions stated above**.</span></span>
   * <span data-ttu-id="c1257-195">选择“购买”按钮。</span><span class="sxs-lookup"><span data-stu-id="c1257-195">Select the **Purchase** button.</span></span>

<span data-ttu-id="c1257-196">部署可能需要 15-20 分钟才能完成。</span><span class="sxs-lookup"><span data-stu-id="c1257-196">It can take 15-20 minutes for the deployment to complete.</span></span>

## <a name="pricing"></a><span data-ttu-id="c1257-197">定价</span><span class="sxs-lookup"><span data-stu-id="c1257-197">Pricing</span></span>

<span data-ttu-id="c1257-198">为了方便用户了解运行本解决方案的成本，我们已在成本计算器中预配置了所有服务。</span><span class="sxs-lookup"><span data-stu-id="c1257-198">To explore the cost of running this solution, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="c1257-199">若要了解自己的特定用例的定价变化情况，请按预期的流量</span><span class="sxs-lookup"><span data-stu-id="c1257-199">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span> <span data-ttu-id="c1257-200">更改相应的变量。</span><span class="sxs-lookup"><span data-stu-id="c1257-200">match your expected traffic.</span></span>

<span data-ttu-id="c1257-201">我们已根据要存储的容器映像数以及运行应用程序所需的 Kubernetes 节点数提供了三个示例性的成本配置文件。</span><span class="sxs-lookup"><span data-stu-id="c1257-201">We have provided three sample cost profiles based on the number of container images to store and Kubernetes nodes to run your applications.</span></span>

* <span data-ttu-id="c1257-202">[小][small-pricing]：对应于每月 1000 个容器生成的情况。</span><span class="sxs-lookup"><span data-stu-id="c1257-202">[Small][small-pricing]: this correlates to 1000 container builds per month.</span></span>
* <span data-ttu-id="c1257-203">[中][medium-pricing]：对应于每月 100,000 个容器生成的情况。</span><span class="sxs-lookup"><span data-stu-id="c1257-203">[Medium][medium-pricing]: this correlates to 100,000 container builds per month.</span></span>
* <span data-ttu-id="c1257-204">[大][large-pricing]：对应于每月 1,000,000 个容器生成的情况。</span><span class="sxs-lookup"><span data-stu-id="c1257-204">[Large][large-pricing]: this correlates to 1,000,000 container builds per month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="c1257-205">相关资源</span><span class="sxs-lookup"><span data-stu-id="c1257-205">Related Resources</span></span>

<span data-ttu-id="c1257-206">本解决方案使用了 Azure 容器注册表和 Azure Kubernetes 服务来存储和运行基于容器的应用程序。</span><span class="sxs-lookup"><span data-stu-id="c1257-206">This solution used Azure Container Registry and Azure Kubernetes Service to store and run your container-based applications.</span></span> <span data-ttu-id="c1257-207">Azure 容器实例也可用于运行基于容器的应用程序，不需预配任何业务流程组件。</span><span class="sxs-lookup"><span data-stu-id="c1257-207">Azure Container Instances can also be used to run container-based applications, without having to provision any orchestration components.</span></span> <span data-ttu-id="c1257-208">有关详细信息，请参阅 [Azure 容器实例概述][azureaci-docs]。</span><span class="sxs-lookup"><span data-stu-id="c1257-208">For more information, see [Azure Container Instances overview][azureaci-docs].</span></span>

<!-- links -->
[architecture]: ./media/devops-with-aks/architecture-devops-with-aks.png
[autoscaling]: ../../best-practices/auto-scaling.md
[availability]: ../../checklist/availability.md
[azureaci-docs]: /azure/container-instances/container-instances-overview
[azureacr-docs]: /azure/container-registry/container-registry-intro
[azurecosmosdb-docs]: /azure/cosmos-db/introduction
[azureaks-docs]: /azure/aks/intro-kubernetes
[azuremonitor-docs]: /azure/monitoring-and-diagnostics/monitoring-overview
[azurevm-docs]: /azure/virtual-machines/linux/overview
[createsp]: /cli/azure/ad/sp#az-ad-sp-create
[grafana]: https://grafana.com/
[jenkins]: https://jenkins.io/
[resiliency]: ../../resiliency/index.md
[resource-groups]: /azure/azure-resource-manager/resource-group-overview
[security]: /azure/security/
[scalability]: ../../checklist/scalability.md
[sshkeydocs]: /azure/virtual-machines/linux/mac-create-ssh-keys
[vsts]: /vsts/?view=vsts
[kubernetes]: https://kubernetes.io/
[service-fabric]: /azure/service-fabric/

[small-pricing]: https://azure.com/e/841f0a75b1ea4802ba1ac8f7918a71e7
[medium-pricing]: https://azure.com/e/eea0e6d79b4e45618a96d33383ec77ba
[large-pricing]: https://azure.com/e/3faab662c54c473da55a1e93a27e0e64