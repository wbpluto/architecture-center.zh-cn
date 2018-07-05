---
title: 说明：什么是云调控？
description: 解释 Azure 和云的资源调控概念
author: petertay
ms.openlocfilehash: 3404beaa719177ee7638feed8a8442b5c3b455c6
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206474"
---
# <a name="explainer-what-is-cloud-resource-governance"></a><span data-ttu-id="dba31-103">说明：什么是云资源调控？</span><span class="sxs-lookup"><span data-stu-id="dba31-103">Explainer: what is cloud resource governance?</span></span>

<span data-ttu-id="dba31-104">在 [Azure 工作原理](azure-explainer.md)说明文章中，我们已知道 Azure 是在虚拟化硬件和软件中以用户身份运行的服务器和网络硬件的集合。</span><span class="sxs-lookup"><span data-stu-id="dba31-104">In the [how does Azure work](azure-explainer.md) explainer, you learned that Azure is a collection of servers and networking hardware running virtualized hardware and software on behalf of users.</span></span> <span data-ttu-id="dba31-105">Azure 可让组织根据需要轻松创建、读取、更新和删除资源，使开发工作和 IT 部门变得更加轻盈。</span><span class="sxs-lookup"><span data-stu-id="dba31-105">Azure enables your organization's development and IT departments to be agile by making it easy to create, read, update, and delete resources as needed.</span></span>

<span data-ttu-id="dba31-106">但是，为开发人员授予不受限制的资源访问权限可能会使他们变得随心所欲，此外还可能导致意外的成本。</span><span class="sxs-lookup"><span data-stu-id="dba31-106">However, while giving unrestricted resource access to developers can make them very agile, it can also lead to unintended cost consequences.</span></span> <span data-ttu-id="dba31-107">例如，某个开发团队获批部署一组用于测试的资源，但完成测试后忘记删除这些资源。</span><span class="sxs-lookup"><span data-stu-id="dba31-107">For example, a development team might be approved to deploy a set of resources for testing but forget to delete them when testing is complete.</span></span> <span data-ttu-id="dba31-108">这些资源会不断加大成本，即使其用途不再受批准，或者不再有需要。</span><span class="sxs-lookup"><span data-stu-id="dba31-108">These resources will continue to accrue costs even though their use is no longer approved or necessary.</span></span> 

<span data-ttu-id="dba31-109">解决此问题的方法就是采用资源访问**调控**。</span><span class="sxs-lookup"><span data-stu-id="dba31-109">The solution to this problem is resource access **governance**.</span></span> <span data-ttu-id="dba31-110">调控是指根据组织的目标和要求，对 Azure 资源的使用持续进行管理、监视和审核的过程。</span><span class="sxs-lookup"><span data-stu-id="dba31-110">Governance refers to the ongoing process of managing, monitoring, and auditing the use of Azure resources to meet the goals and requirements of your organization.</span></span> 

> [!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2ii94] 

<span data-ttu-id="dba31-111">每家组织的目标和要求都是独特的，因此，不存在以一应百的调控方法。</span><span class="sxs-lookup"><span data-stu-id="dba31-111">These goals and requirements are unique to each organization so it's not possible to have a one-size-fits-all approach to governance.</span></span> <span data-ttu-id="dba31-112">Azure 实施两个主要的管理工具：**基于资源的访问控制 (RBAC)** 和**资源策略**，每家组织可以使用这些工具设计自己的调控模型。</span><span class="sxs-lookup"><span data-stu-id="dba31-112">Rather, Azure implements two primary governance tools, **resource based access control (RBAC)**, and **resource policy**, and it's up to each organization to design their governance model using them.</span></span>

<span data-ttu-id="dba31-113">RBAC 定义角色，而角色定义获得该角色的用户的功能。</span><span class="sxs-lookup"><span data-stu-id="dba31-113">RBAC defines roles, and roles define the capabilities for a user that is assigned the role.</span></span> <span data-ttu-id="dba31-114">例如，**所有者**角色可以资源的所有功能（创建、读取、更新和删除），**读取者**角色只能启用读取功能。</span><span class="sxs-lookup"><span data-stu-id="dba31-114">For example, the **owner** role enables all capabilites (create, read, update, and delete) for a resource, while the  **reader** roles enables only the read capability.</span></span> <span data-ttu-id="dba31-115">可以使用应用到许多资源类型的宽泛范围定义角色，也可以使用应用到少量资源类型的狭窄范围定义角色。</span><span class="sxs-lookup"><span data-stu-id="dba31-115">Roles can be defined with a broad scope that applies to many resources types, or a narrow scope that applies to a few.</span></span> 

<span data-ttu-id="dba31-116">资源策略定义有关创建资源的规则。</span><span class="sxs-lookup"><span data-stu-id="dba31-116">Resource policies define rules for resource creation.</span></span> <span data-ttu-id="dba31-117">例如，资源策略可将 VM 的 SKU 限制为特定的预批准大小。</span><span class="sxs-lookup"><span data-stu-id="dba31-117">For example, a resource policy can limit the SKU of a VM to a particular pre-appproved size.</span></span> <span data-ttu-id="dba31-118">或者，在发出创建资源的请求时，资源策略可以强制添加包含成本中心的标记。</span><span class="sxs-lookup"><span data-stu-id="dba31-118">Or, a resource policy can enforce the addition of a tag with a cost center when the request is made to create the resource.</span></span> 

<span data-ttu-id="dba31-119">配置这些工具时，一个重要的考虑因素是在调控与组织灵活性之间实现平衡。</span><span class="sxs-lookup"><span data-stu-id="dba31-119">When configuring these tools, an important consideration is balancing governance versus organizational agility.</span></span> <span data-ttu-id="dba31-120">也就是说，调控策略的限制性越强，开发人员和 IT 工作人员的灵活性就越低。</span><span class="sxs-lookup"><span data-stu-id="dba31-120">That is, the more restrictive your governance policy, the less agile your developers and IT workers become.</span></span> <span data-ttu-id="dba31-121">这是因为，限制性的调控策略可能需要更多的手动步骤，例如，要求开发人员填写表单，或者将电子邮件发送到管理团队中的某人，以手动创建资源。</span><span class="sxs-lookup"><span data-stu-id="dba31-121">This is because a restrictive goverance policy may require more manual steps, such as requiring a developer to fill out a form or send an email to a person on the governance team to manually create a resource.</span></span> <span data-ttu-id="dba31-122">调控团队的功能有限，可能会积压工作，导致工作效率不高的开发团队等待创建其资源，并且在等待删除不需要的资源过程中，这些资源持续产生成本。</span><span class="sxs-lookup"><span data-stu-id="dba31-122">The goverance team has finite capabilities and may become backlogged, resulting in unproductive development teams waiting for their resources to be created and unneeded resources accruing costs while they wait to be deleted.</span></span>

## <a name="next-steps"></a><span data-ttu-id="dba31-123">后续步骤</span><span class="sxs-lookup"><span data-stu-id="dba31-123">Next steps</span></span>

<span data-ttu-id="dba31-124">采用 Azure 的下一步骤是[了解 Azure 中的数字标识](tenant-explainer.md)以及[在 Azure AD 中创建第一个用户][docs-add-users-to-aad]。</span><span class="sxs-lookup"><span data-stu-id="dba31-124">Your next step in adopting Azure is to [understand digital identity in Azure](tenant-explainer.md) and [create your first user in Azure AD][docs-add-users-to-aad].</span></span>

<!-- Links -->

[docs-add-users-to-aad]: /azure/active-directory/add-users-azure-active-directory?toc=/azure/architecture/cloud-adoption-guide/toc.json