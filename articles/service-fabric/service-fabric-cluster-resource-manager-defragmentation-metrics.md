---
title: "Azure Service Fabric 中度量的重組 | Microsoft Docs"
description: "使用重組或封裝作為 Service Fabric 中度量策略的概觀"
services: service-fabric
documentationcenter: .net
author: masnider
manager: timlt
editor: 
ms.assetid: e5ebfae5-c8f7-4d6c-9173-3e22a9730552
ms.service: Service-Fabric
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: NA
ms.date: 08/19/2016
ms.author: masnider
translationtype: Human Translation
ms.sourcegitcommit: 219dcbfdca145bedb570eb9ef747ee00cc0342eb
ms.openlocfilehash: 68183597e19347f10a0ffe24a418dbcac409cb14


---
# <a name="defragmentation-of-metrics-and-load-in-service-fabric"></a>度量的重組和 Service Fabric 中的負載
Service Fabric 叢集 Resource Manager 主要與分散負載方面的平衡有關 - 確定會平均使用叢集中的所有節點。 這對於存留失敗而言通常是最安全且最聰明的配置，因為它可確保任何指定的失敗都不會取得大多數的指定工作負載。 Service Fabric 叢集 Resource Manager 也支援不同的策略 (重組)。 重組通常表示，我們應該實際嘗試合併計量，而不是嘗試將計量的使用量分散到叢集中。 這是我們一般策略的幸運反轉 – 我們是透過增加偏差來開始最佳化，而不是以針對指定度量將度量負載的平均標準偏差最小化為基礎，來將叢集最佳化。 但是我們為什麼要用這項策略？

嗯，如果您已經在叢集中的節點間平均分散負載，則您已耗用了一些節點必須提供的資源。 通常這不會產生問題，但有時某些工作負載會建立特別大型的服務並耗用絕大部分的節點 – 假設 75% 至 95% 的節點資源最終會專屬於單一服務執行個體或複本。 這不是問題，叢集資源管理員將會在建立服務時偵測到它需要重新組織叢集，以便為這個大型工作負載騰出空間，並進行要使其成真的設定，但同時該工作負載必須等待在叢集中排程。

假設排程新的工作負載通常至少會有一點延遲，如果我們未執行一些不同的動作，若有許多要移動的服務和狀態，有時我們會透過這些 SLA 適當地提供，特別是如果叢集中的工作負載很大 (因此需要較長時間在叢集中四處移動) 時。 事實上，當我們在模擬中根據真正的叢集資料來測量建立時間時，發現如果服務夠大且會妥善利用叢集，則建立這些大型服務的速度會變慢，而我們可以引進重組度量的原則來改善這一點。

舉例來說，如果電腦的硬碟分散，建立或存取檔案的速度就會變慢，而您可藉由磁碟重組以提供較大的連續區塊來加速，您就可以設定重組度量，讓叢集資源管理員主動嘗試將服務的負載壓縮至較少數的節點，如此一來，(幾乎) 永遠都會有空間可供非常大型的服務使用，讓它們能夠快速建立。 大部分的人員都不需要這樣做，因為服務通常應該都很小，因此能夠很容易地為它們找到空間，但如果您有大型服務且需要快速建立服務 (且願意在其他方面妥協，例如較高的失敗影響力，以及某些資源在等待工作負載排程時未被利用)，您就適合使用重組策略。

下圖提供兩個不同叢集的視覺表示法，一個已重組，另一個未重組。 在平衡的情況下，如果要建立一個新的叢集，相較於已重組的叢集 (可能立即放置在節點 4 或 5)，請考慮是否需要放置其中一個最大服務物件的移動。

![比較平衡和重組叢集][Image1]

## <a name="defragmentation-pros-and-cons"></a>重組的優缺點
因此，有哪些其他概念性的代價？ 我們建議先徹底測量您的工作負載，然後再開啟重組計量。 以下是要考慮事項的一覽表︰

| 重組優點 | 重組缺點 |
| --- | --- |
| 能夠更快速建立大型服務 |將負載集中到較少數的節點，提高爭用 |
| 在建立期間啟用較低的資料移動 |失敗會影響更多服務，並導致更多流失 |
| 能夠豐富描述需求和空間的回收 |較複雜的整體資源管理組態 |

您可以在同一個叢集中混合重組和一般計量，而資源管理員將盡力確保您會得到下列配置：在嘗試分散到其他地方時，盡可能合併最多數量的重組計量。 您將得到的確切結果取決於相較於重組度量數量的平衡度量數量，以及它們的權數、目前負載等項目。

## <a name="configuring-defragmentation-metrics"></a>設定重組度量
設定重組計量是叢集中的全域決策，而您可以選取個別的計量進行重組︰

ClusterManifest.xml：

```xml
<Section Name="DefragmentationMetrics">
    <Parameter Name="Disk" Value="true" />
    <Parameter Name="CPU" Value="false" />
</Section>
```

## <a name="next-steps"></a>後續步驟
* 叢集資源管理員有許多描述叢集的選項。 若要深入了解這些選項，請查看關於 [描述 Service Fabric 叢集](service-fabric-cluster-resource-manager-cluster-description.md)
* 度量是 Service Fabric 叢集資源管理員管理叢集中的耗用量和容量的方式。 若要深入了解度量及其設定方式，請查看 [這篇文章](service-fabric-cluster-resource-manager-metrics.md)

[Image1]:./media/service-fabric-cluster-resource-manager-defragmentation-metrics/balancing-defrag-compared.png



<!--HONumber=Nov16_HO3-->


