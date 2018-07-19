---
title: 智能应用 - Azure 上的图像处理
description: 经验证的解决方案，用于将图像处理内置到 Azure 应用程序中。
author: david-stanford
ms.date: 07/05/2018
ms.openlocfilehash: c5bfb9a929ddddda4336e1cbc8665a0b4d3bbe2c
ms.sourcegitcommit: 5d99b195388b7cabba383c49a81390ac48f86e8a
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 07/06/2018
ms.locfileid: "37891269"
---
# <a name="insurance-claim-image-classification-on-azure"></a><span data-ttu-id="19428-103">Azure 上的保险索赔图像分类</span><span class="sxs-lookup"><span data-stu-id="19428-103">Insurance claim image classification on Azure</span></span>

<span data-ttu-id="19428-104">本示例方案适用于需处理图像的企业。</span><span class="sxs-lookup"><span data-stu-id="19428-104">This example scenario is applicable for businesses that need to process images.</span></span>

<span data-ttu-id="19428-105">可能的应用包括：对时尚网站的图像分类、分析保险索赔的文本和图像，或者理解游戏屏幕截图中的遥测数据。</span><span class="sxs-lookup"><span data-stu-id="19428-105">Potential applications include classifying images for a fashion website, analyzing text and images for insurance claims, or understanding telemetry data from game screenshots.</span></span> <span data-ttu-id="19428-106">传统上，公司需开发机器学习模型方面的专业技术，训练模型，最后再通过自定义过程运行图像，以便从这些图像中获取数据。</span><span class="sxs-lookup"><span data-stu-id="19428-106">Traditionally, companies would need to develop expertise in machine learning models, train the models, and finally run the images through their custom process to get the data out of the images.</span></span>

<span data-ttu-id="19428-107">使用 Azure 服务（例如计算机视觉 API 和 Azure Functions），公司不需管理各个服务器，既减少了成本，又可充分利用 Microsoft 围绕认知服务图像处理开发的专业技术。</span><span class="sxs-lookup"><span data-stu-id="19428-107">By using Azure services such as the Computer Vision API and Azure Functions, companies can eliminate the need to manage individual servers, while reducing costs and leveraging the expertise that Microsoft has already developed around processing images with Cognitive services.</span></span> <span data-ttu-id="19428-108">本方案专门解决图像处理方案问题。</span><span class="sxs-lookup"><span data-stu-id="19428-108">This scenario specifically addresses an image processing scenario.</span></span> <span data-ttu-id="19428-109">如果有各种不同的的 AI 需求，可以考虑全套[认知服务][cognitive-docs]。</span><span class="sxs-lookup"><span data-stu-id="19428-109">If you have different AI needs, consider the full suite of [Cognitive Services][cognitive-docs].</span></span>

## <a name="potential-use-cases"></a><span data-ttu-id="19428-110">可能的用例</span><span class="sxs-lookup"><span data-stu-id="19428-110">Potential use cases</span></span>

<span data-ttu-id="19428-111">以下用例可以考虑本解决方案：</span><span class="sxs-lookup"><span data-stu-id="19428-111">Consider this solution for the following use cases:</span></span>

* <span data-ttu-id="19428-112">对时尚网站上的图像分类。</span><span class="sxs-lookup"><span data-stu-id="19428-112">Classify images on a fashion website.</span></span>
* <span data-ttu-id="19428-113">对保险索赔的图像分类。</span><span class="sxs-lookup"><span data-stu-id="19428-113">Classify images for insurance claims</span></span>
* <span data-ttu-id="19428-114">对游戏屏幕截图中的遥测数据分类。</span><span class="sxs-lookup"><span data-stu-id="19428-114">Classify telemetry data from screenshots of games.</span></span>

## <a name="architecture"></a><span data-ttu-id="19428-115">体系结构</span><span class="sxs-lookup"><span data-stu-id="19428-115">Architecture</span></span>

![智能应用体系结构 - 计算机视觉][architecture-computer-vision]

<span data-ttu-id="19428-117">本解决方案涉及 Web 或移动应用程序的后端组件。</span><span class="sxs-lookup"><span data-stu-id="19428-117">This solution covers the back-end components of a web or mobile application.</span></span> <span data-ttu-id="19428-118">数据流经解决方案的情形如下所示：</span><span class="sxs-lookup"><span data-stu-id="19428-118">Data flows through the solution as follows:</span></span>

1. <span data-ttu-id="19428-119">Azure Functions 充当 API 层。</span><span class="sxs-lookup"><span data-stu-id="19428-119">Azure Functions acts as the API layer.</span></span> <span data-ttu-id="19428-120">应用程序可以通过这些 API 上传图像以及检索 Cosmos DB 中的数据。</span><span class="sxs-lookup"><span data-stu-id="19428-120">These APIs enable the application to upload images and retrieve data from Cosmos DB.</span></span>

2. <span data-ttu-id="19428-121">通过 API 调用上传图像时，该图像会存储在 Blob 存储中。</span><span class="sxs-lookup"><span data-stu-id="19428-121">When an image is uploaded via an API call, it's stored in Blob storage.</span></span>

3. <span data-ttu-id="19428-122">向 Blob 存储添加新文件会触发系统将 EventGrid 通知发送给某个 Azure 函数。</span><span class="sxs-lookup"><span data-stu-id="19428-122">Adding new files to Blob storage triggers an EventGrid notification to be sent to an Azure Function.</span></span>

4. <span data-ttu-id="19428-123">Azure Functions 会向计算机视觉 API 发送新上传文件的链接，供其分析。</span><span class="sxs-lookup"><span data-stu-id="19428-123">Azure Functions sends a link to the newly uploaded file to the Computer Vision API to analyze.</span></span>

5. <span data-ttu-id="19428-124">数据从计算机视觉 API 返回以后，Azure Functions 会在 Cosmos DB 中生成一个条目，以便持久保存分析结果和图像元数据。</span><span class="sxs-lookup"><span data-stu-id="19428-124">Once the data has been returned from the Computer Vision API, Azure Functions makes an entry in Cosmos DB to persist the results of the analysis alongside the image metadata.</span></span>

### <a name="components"></a><span data-ttu-id="19428-125">组件</span><span class="sxs-lookup"><span data-stu-id="19428-125">Components</span></span>

* <span data-ttu-id="19428-126">[计算机视觉 API][computer-vision-docs] 是认知服务套件的一部分，用于检索每个图像的信息。</span><span class="sxs-lookup"><span data-stu-id="19428-126">[Computer Vision API][computer-vision-docs] is part of the Cognitive Services suite and is used to retrieve information about each image.</span></span>

* <span data-ttu-id="19428-127">[Azure Functions][functions-docs] 为 Web 应用程序提供后端 API，并为已上传的图像提供事件处理。</span><span class="sxs-lookup"><span data-stu-id="19428-127">[Azure Functions][functions-docs] provides the backend API for the web application, as well as the event processing for uploaded images.</span></span>

* <span data-ttu-id="19428-128">[事件网格][eventgrid-docs]在新图像上传到 Blob 存储时触发一个事件。</span><span class="sxs-lookup"><span data-stu-id="19428-128">[Event Grid][eventgrid-docs] triggers an event when a new image is uploaded to blob storage.</span></span> <span data-ttu-id="19428-129">该图像然后就会通过 Azure Functions 进行处理。</span><span class="sxs-lookup"><span data-stu-id="19428-129">The image is then processed with Azure functions.</span></span>

* <span data-ttu-id="19428-130">[Blob 存储][storage-docs]存储上传到 Web 应用程序中的所有图像文件，以及 Web 应用程序使用的任何静态文件。</span><span class="sxs-lookup"><span data-stu-id="19428-130">[Blob Storage][storage-docs] stores all of the image files that are uploaded into the web application, as well any static files that the web application consumes.</span></span>

* <span data-ttu-id="19428-131">[Cosmos DB][cosmos-docs] 存储每个已上传图像的元数据，包括计算机视觉 API 的处理结果。</span><span class="sxs-lookup"><span data-stu-id="19428-131">[Cosmos DB][cosmos-docs] stores metadata about each image that is uploaded, including the results of the processing from Computer Vision API.</span></span>

## <a name="alternatives"></a><span data-ttu-id="19428-132">备选项</span><span class="sxs-lookup"><span data-stu-id="19428-132">Alternatives</span></span>

* <span data-ttu-id="19428-133">[自定义视觉服务][custom-vision-docs]。</span><span class="sxs-lookup"><span data-stu-id="19428-133">[Custom Vision Service][custom-vision-docs].</span></span> <span data-ttu-id="19428-134">计算机视觉 API 返回一组[基于分类的类别][cv-categories]。</span><span class="sxs-lookup"><span data-stu-id="19428-134">The Computer Vision API returns a set of [taxonomy-based categories][cv-categories].</span></span> <span data-ttu-id="19428-135">若需处理不是由计算机视觉 API 返回的信息，则可考虑使用自定义视觉服务，以便生成自定义图像分类器。</span><span class="sxs-lookup"><span data-stu-id="19428-135">If you need to process information that isn't returned by the Computer Vision API, consider the Custom Vision Service, which lets you build custom image classifiers.</span></span>

* <span data-ttu-id="19428-136">[Azure 搜索][azure-search-docs]。</span><span class="sxs-lookup"><span data-stu-id="19428-136">[Azure Search][azure-search-docs].</span></span> <span data-ttu-id="19428-137">如果用例需要查询元数据来查找符合特定条件的图像，则可考虑使用 Azure 搜索。</span><span class="sxs-lookup"><span data-stu-id="19428-137">If your use case involves querying the metadata to find images that meet specific criteria, consider using Azure Search.</span></span> <span data-ttu-id="19428-138">[认知搜索][cognitive-search]目前为预览版，可以无缝集成此工作流。</span><span class="sxs-lookup"><span data-stu-id="19428-138">Currently in preview, [Cognitive search][cognitive-search] seamlessly integrates this workflow.</span></span>

## <a name="considerations"></a><span data-ttu-id="19428-139">注意事项</span><span class="sxs-lookup"><span data-stu-id="19428-139">Considerations</span></span>

### <a name="scalability"></a><span data-ttu-id="19428-140">可伸缩性</span><span class="sxs-lookup"><span data-stu-id="19428-140">Scalability</span></span>

<span data-ttu-id="19428-141">大多数情况下，本解决方案的所有组件是托管服务，可以自动进行缩放。</span><span class="sxs-lookup"><span data-stu-id="19428-141">For the most part all of the components of this solution are managed services that will automatically scale.</span></span> <span data-ttu-id="19428-142">一些需注意的例外：Azure Functions 存在限制，最多可以使用 200 个实例。</span><span class="sxs-lookup"><span data-stu-id="19428-142">A couple notable exceptions: Azure Functions has a limit of a maximum of 200 instances.</span></span> <span data-ttu-id="19428-143">如果所需规模超出限制，可以考虑使用多个区域或应用计划。</span><span class="sxs-lookup"><span data-stu-id="19428-143">If you need to scale beyond, consider multiple regions or app plans.</span></span>

<span data-ttu-id="19428-144">Cosmos DB 不会按照预配的请求单位 (RU) 自动缩放。</span><span class="sxs-lookup"><span data-stu-id="19428-144">Cosmos DB doesn’t auto-scale in terms of provisioned request units (RUs).</span></span>  <span data-ttu-id="19428-145">有关如何估算需求的指南，请参阅文档中的[请求单位][request-units]。</span><span class="sxs-lookup"><span data-stu-id="19428-145">For guidance on estimating your requirements see [request units][request-units] in our documentation.</span></span> <span data-ttu-id="19428-146">若要充分利用 Cosmos DB 中的缩放功能，还应参阅[分区键][partition-key]的内容。</span><span class="sxs-lookup"><span data-stu-id="19428-146">To fully take advantage of the scaling in Cosmos DB you should also take a look at [partition keys][partition-key].</span></span>

<span data-ttu-id="19428-147">很多情况下，为了确保可用性、可伸缩性和分区，NoSQL 数据库会牺牲一致性（这就是所谓的 CAP 法则）。</span><span class="sxs-lookup"><span data-stu-id="19428-147">NoSQL databases frequently trade consistency (in the sense of the CAP theorem) for availability, scalability and partition.</span></span>  <span data-ttu-id="19428-148">但就键-值数据模型（在本方案中使用）来说，很少需要事务一致性，因为大多数操作就其定义来说属于原子操作。</span><span class="sxs-lookup"><span data-stu-id="19428-148">However, in the case of key-value data models which is used in this scenario, transaction consistency is rarely needed as most operations are by definition atomic.</span></span> <span data-ttu-id="19428-149">有关如何[选择正确的数据存储](../../guide/technology-choices/data-store-overview.md)的其他指南，请访问体系结构中心。</span><span class="sxs-lookup"><span data-stu-id="19428-149">Additional guidance to [Choose the right data store](../../guide/technology-choices/data-store-overview.md) is available in the architecture center.</span></span>

<span data-ttu-id="19428-150">有关如何设计可缩放解决方案的通用指南，请参阅 Azure 体系结构中心的[可伸缩性核对清单][scalability]。</span><span class="sxs-lookup"><span data-stu-id="19428-150">For general guidance on designing scalable solutions, see the [scalability checklist][scalability] in the Azure Architecture Center.</span></span>

### <a name="security"></a><span data-ttu-id="19428-151">“安全”</span><span class="sxs-lookup"><span data-stu-id="19428-151">Security</span></span>

<span data-ttu-id="19428-152">[托管服务标识][msi] (MSI) 用于访问帐户的其他内部资源，然后会被系统分配给你的 Azure Functions。</span><span class="sxs-lookup"><span data-stu-id="19428-152">[Managed service identities][msi] (MSI) are used to provide access to other resources internal to your account and then assigned to your Azure Functions.</span></span> <span data-ttu-id="19428-153">只允许通过这些标识访问必要的资源，这样可确保不将额外的内容公开给函数（可能还包括客户）。</span><span class="sxs-lookup"><span data-stu-id="19428-153">Only allow access to the requisite resources in those identities to ensure that nothing extra is exposed to your functions (and potentially to your customers).</span></span>  

<span data-ttu-id="19428-154">若需安全解决方案的通用设计指南，请参阅 [Azure 安全性文档][security]。</span><span class="sxs-lookup"><span data-stu-id="19428-154">For general guidance on designing secure solutions, see the [Azure Security Documentation][security].</span></span>

### <a name="resiliency"></a><span data-ttu-id="19428-155">复原</span><span class="sxs-lookup"><span data-stu-id="19428-155">Resiliency</span></span>

<span data-ttu-id="19428-156">本解决方案中的所有组件都是托管的，因此均可在区域级别自动复原。</span><span class="sxs-lookup"><span data-stu-id="19428-156">All of the components in this solution are managed, so at a regional level they are all resilient automatically.</span></span> 

<span data-ttu-id="19428-157">若需可复原解决方案的通用设计指南，请参阅[设计适用于 Azure 的可复原应用程序][resiliency]。</span><span class="sxs-lookup"><span data-stu-id="19428-157">For general guidance on designing resilient solutions, see [Designing resilient applications for Azure][resiliency].</span></span>

## <a name="pricing"></a><span data-ttu-id="19428-158">定价</span><span class="sxs-lookup"><span data-stu-id="19428-158">Pricing</span></span>

<span data-ttu-id="19428-159">为了方便用户了解运行本解决方案的成本，我们已在成本计算器中预配置了所有服务。</span><span class="sxs-lookup"><span data-stu-id="19428-159">To explore the cost of running this solution, all of the services are pre-configured in the cost calculator.</span></span> <span data-ttu-id="19428-160">若要了解自己的特定用例的定价变化情况，请按预期的流量更改相应的变量。</span><span class="sxs-lookup"><span data-stu-id="19428-160">To see how the pricing would change for your particular use case, change the appropriate variables to match your expected traffic.</span></span>

<span data-ttu-id="19428-161">我们已根据流量（假定所有图像的大小均为 100kb）提供了三个示例性的成本配置文件：</span><span class="sxs-lookup"><span data-stu-id="19428-161">We have provided three sample cost profiles based on amount of traffic (we assume all images are 100kb in size):</span></span>

* <span data-ttu-id="19428-162">[小][pricing]：对应于每月处理的图像数 &lt; 5000 的情况。</span><span class="sxs-lookup"><span data-stu-id="19428-162">[Small][pricing]: this correlates to processing &lt; 5000 images a month.</span></span>
* <span data-ttu-id="19428-163">[中][medium-pricing]：对应于每月处理的图像数为 500,000 的情况。</span><span class="sxs-lookup"><span data-stu-id="19428-163">[Medium][medium-pricing]: this correlates to processing 500,000 images a month.</span></span>
* <span data-ttu-id="19428-164">[大][large-pricing]：对应于每月处理的图像数为 5 千万的情况。</span><span class="sxs-lookup"><span data-stu-id="19428-164">[Large][large-pricing]: this correlates to processing 50 million images a month.</span></span>

## <a name="related-resources"></a><span data-ttu-id="19428-165">相关资源</span><span class="sxs-lookup"><span data-stu-id="19428-165">Related Resources</span></span>

<span data-ttu-id="19428-166">如需此解决方案的引导式学习路径，请参阅[在 Azure 中生成无服务器 Web 应用][serverless]。</span><span class="sxs-lookup"><span data-stu-id="19428-166">For a guided learning path of this solution, see [Build a serverless web app in Azure][serverless].</span></span>  

<span data-ttu-id="19428-167">将此解决方案放到生产环境中之前，请参阅 Azure Functions [最佳做法][functions-best-practices]。</span><span class="sxs-lookup"><span data-stu-id="19428-167">Before putting this in a production environment, review the Azure Functions [best practices][functions-best-practices].</span></span>

<!-- links -->
[pricing]: https://azure.com/e/f9b59d238b43423683db73f4a31dc380
[medium-pricing]: https://azure.com/e/7c7fc474db344b87aae93bc29ae27108
[large-pricing]: https://azure.com/e/cbadbca30f8640d6a061f8457a74ba7d
[functions-docs]: /azure/azure-functions/
[computer-vision-docs]: /azure/cognitive-services/computer-vision/home
[storage-docs]: /azure/storage/
[azure-search-docs]: /azure/search/
[cognitive-search]: /azure/search/cognitive-search-concept-intro
[architecture-computer-vision]: ./media/architecture-computer-vision.png
[serverless]: /azure/functions/tutorial-static-website-serverless-api-with-database
[cosmos-docs]: /azure/cosmos-db/
[eventgrid-docs]: /azure/event-grid/
[cognitive-docs]: /azure/#pivot=products&panel=ai
[custom-vision-docs]: /azure/cognitive-services/Custom-Vision-Service/home
[cv-categories]: /azure/cognitive-services/computer-vision/home#the-86-category-concept
[resiliency]: /azure/architecture/resiliency/
[security]: /azure/security/
[scalability]: /azure/architecture/checklist/scalability
[functions-best-practices]: /azure/azure-functions/functions-best-practices
[msi]: /azure/app-service/app-service-managed-service-identity
[request-units]: /azure/cosmos-db/request-units
[partition-key]: /azure/cosmos-db/partition-data