---
title: 专为业务需求打造
description: 必须根据业务需求作出每一个设计决策
author: MikeWasson
ms.openlocfilehash: 768f2298860d91774d93c1917cf95000bb2b873d
ms.sourcegitcommit: 26b04f138a860979aea5d253ba7fecffc654841e
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 06/19/2018
ms.locfileid: "36206527"
---
# <a name="build-for-the-needs-of-the-business"></a>专为业务需求打造

## <a name="every-design-decision-must-be-justified-by-a-business-requirement"></a>必须根据业务需求作出每一个设计决策

此设计原则看似不言自明，但在设计解决方案时请务必遵循此原则。 预期的用户数是数千还是数百万人？ 是否可接受长达一小时的应用程序中断时间？ 希望流量突然增加还是可预测的工作负载？ 最终，必须根据业务需求作出每一个设计决策。 

## <a name="recommendations"></a>建议

**定义业务目标**，包括恢复时间目标 (RTO)、恢复点目标 (RPO) 和可容忍的最长中断时间 (MTO)。 这些数字应为决策提供有关体系结构的信息。 例如，若要实现较低的 RTO，可能会向次要区域实施自动故障转移。 但如果你的解决方案可以接受较高的 RTO，则可能不需要该程度的冗余。

**记录服务级别协议 (SLA) 和服务级别目标 (SLO)**，包括可用性和性能指标。 你可能会构建可用性为 99.95% 的解决方案。 这够了吗？ 答案是需要业务决策。 

**围绕业务领域为应用程序建模**。 首先分析业务需求。 根据这些需求为应用程序建模。 请考虑使用领域驱动设计 (DDD) 方法创建反映业务流程和用例的[域模型][domain-model]。 

**捕获功能性和非功能性需求**。 可以通过功能性需求判断应用程序是否执行了所需操作。 可以通过非功能性需求判断应用程序的操作执行是否则正常。 具体而言，确保你理解自己对可伸缩性、可用性和延迟的需求。 这些需求将影响设计决策和技术选择。

**按工作负荷分解**。 在此语境中，术语“工作负荷”是指某个离散的功能或计算任务，可以将它和其他任务逻辑分离。 不同的工作负荷在可用性、可伸缩性、数据一致性和灾难恢复方面具有不同的需求。 

**规划增长**。 某解决方案可能满足你当前对用户数、事务量和数据存储等方面的需求。 但是，可靠的应用程序可在不进行主要体系结构更改的情况下实现增长。 请参阅[设计为横向扩展](scale-out.md)和[分区避开限制](partition.md)。 也请注意，你的业务模型和业务需求很可能在一段时间后发生更改。 如果应用程序的服务模型和数据模型过于死板，则很难将应用程序用于新的用例和方案。 请参阅 [Design for evolution](design-for-evolution.md)（旨在改进）。

**管理成本**。 在传统的本地应用程序中，需为硬件 (CAPEX) 预付费用。 在云应用程序中，为使用的资源付费。 确保了解所使用服务的定价模型。 总成本包括网络带宽使用量、存储、IP 地址、服务消耗和其他因素。 有关详细信息，请参阅 [Azure 定价][pricing]。 也需考虑操作成本。 在云中，无需管理硬件或其他基础结构，但仍需管理应用程序，包括 DevOps、事件响应和灾难恢复等。 

[domain-model]: https://martinfowler.com/eaaCatalog/domainModel.html
[pricing]: https://azure.microsoft.com/pricing/
