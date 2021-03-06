---
title: "Microsoft Azure IaaS 上軟體更新的最佳作法 | Microsoft Docs"
description: "本文針對 Microsoft Azure IaaS 環境中的軟體更新提供最佳作法集合。  它的適用對象為 IT 專業人員和安全性分析師，他們每天都在處理變更控制、軟體更新和資產管理，包括那些負責其組織安全性和法規工作的人員。"
services: security
documentationcenter: na
author: YuriDio
manager: swadhwa
editor: 
ms.assetid: df6598ee-de5b-48cd-b321-0b2f19d7eeaa
ms.service: security
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 01/04/2017
ms.author: yurid
translationtype: Human Translation
ms.sourcegitcommit: 219dcbfdca145bedb570eb9ef747ee00cc0342eb
ms.openlocfilehash: c52a2d6be31b1a8f581313932c0af87120ccd9fe


---
# <a name="best-practices-for-software-updates-on-microsoft-azure-iaas"></a>Microsoft Azure IaaS 上軟體更新的最佳作法
在探究 Azure [IaaS](https://azure.microsoft.com/overview/what-is-iaas/) 環境的任何最佳作法討論之前，請務必先了解可讓您管理軟體更新的案例和責任。 下圖可協助您了解這些界限︰

![雲端模型和職責](./media/azure-security-best-practices-software-updates-iaas/sec-cloudstack-new.png)

最左邊資料行中顯示組織應該考量的七個責任 (定義於下列各節)，全部都與運算環境的安全性和隱私權有關。

資料的分類與責任和用戶端與端點保護完全是客戶的責任，而實體、主機和網路的責任則在於 PaaS 和 SaaS 模型中的雲端服務提供者。 

其餘的責任則是由客戶和雲端服務提供者共同分擔。 某些責任需要 CSP 和客戶一同管理責任，包括其網域的稽核。 例如，使用 Azure Active Directory 服務時的身分識別與存取管理；如 Multi-Factor Authentication 等服務的組態是由客戶負責，但是確保功能有效是 Microsoft Azure 的責任。

> [!NOTE]
> 如需雲端中共同責任的詳細資訊，請閱讀 [Shared Responsibilities for Cloud Computing](https://gallery.technet.microsoft.com/Shared-Responsibilities-81d0ff91/file/153019/1/Shared%20responsibilities%20for%20cloud%20computing.pdf) (雲端運算的共同責任) 
> 
> 

這些相同的原則會套用在混合式案例中，其中貴公司使用 Azure IaaS VM 與內部部署資源通訊，如下圖所示。

![Microsoft Azure 的典型混合式案例](./media/azure-security-best-practices-software-updates-iaas/sec-azconnectonpre.png)

## <a name="initial-assessment"></a>初步評估
即使您的公司已採用更新管理系統，而且您已採用軟體更新原則，請務必經常檢閱先前的原則評估並根據目前的需求加以更新。 這表示您需要熟悉您的公司中資源的目前狀態。 若要達到此狀態，您必須了解：

* 您企業中的實體和虛擬電腦。
* 這些實體和虛擬電腦所執行的作業系統和版本。
* 每部電腦目前安裝的軟體更新 (Service pack 版本、軟體更新和其他修改)。
* 每部電腦在您的企業中執行的功能。
* 在每部電腦上執行的應用程式和程式。
* 每部電腦的擁有權和連絡人資訊。
* 您的環境中出現的資產，以及用來判斷哪些區域需要最多注意和保護的相對值。
* 已知的安全性問題以及您的企業在安全性層級用來識別新安全性問題或變更的程序。
* 為了保護您的環境所部署的安全措施。

您應該定期更新此資訊，而此資訊應該可隨時提供給那些參與軟體更新管理程序的人員使用。

## <a name="establish-a-baseline"></a>建立基準
軟體更新管理程序中的重要部分就是為您企業中的電腦建立作業系統版本、應用程式和硬體的初始標準安裝；這些稱之為基準。 基準是在特定時間點建立的產品或系統的組態。 例如，應用程式或作業系統基準可供將電腦或服務重建至特定狀態。

基準可提供尋找及修正潛在問題的基礎並簡化軟體更新管理程序，其作法是減少您必須在企業中部署的軟體更新數目，以及增強您監視相符性的能力。

在執行您企業的初始稽核之後，您應該使用取自稽核的資訊來定義您的生產環境中 IT 元件的作業基準。 視生產環境中部署的各類硬體和軟體而定，可能一些需要基準。

例如，某些伺服器需要軟體更新，以免在執行 Windows Server 2012 的情況下進入關機程序時懸置。 這些伺服器的基準應該包含此軟體更新。

在大型組織中，這通常有助於將您企業中的電腦分成各種資產類別，並使用相同版本的軟體和軟體更新讓每個類別保持在標準基準。 然後，您可以在設定軟體更新散發的優先順序時使用這些資產類別。

## <a name="subscribe-to-the-appropriate-software-update-notification-services"></a>訂閱適當的軟體更新通知服務
在您執行企業中所用軟體的初始稽核之後，您應該針對每個軟體產品和版本決定用於接收新軟體更新通知的最佳方法。 視軟體產品而定，最佳的通知方法可能是電子郵件通知、網站或電腦發佈。

例如，Microsoft 安全性回應中心 (MSRC) 會回應 Microsoft 產品的所有安全性相關疑慮，並提供 Microsoft 安全性公告服務、新發現弱點的免費電子郵件通知，以及為了解決這些弱點所發行的軟體更新。 您可以在 http://www.microsoft.com/technet/security/bulletin/notify.mspx 訂閱此服務。

## <a name="software-update-considerations"></a>軟體更新考量
在您執行企業中所用軟體的初始稽核之後，您應該決定設定軟體更新管理系統的需求，這取決於您使用的軟體更新管理系統。 對於 WSUS，請參閱 [Windows Server Update Services 的最佳做法](https://technet.microsoft.com/library/Cc708536)，對於系統中心，請參閱[在 Configuration Manager 中規劃軟體更新](https://technet.microsoft.com/library/gg712696)。

不過，不論您正在使用的解決方案為何，您可以套用一些一般考量和最佳作法，如後續各節所示。

### <a name="setting-up-the-environment"></a>設定環境
在打算設定軟體更新管理環境時，請考量下列作法：

* **建立以穩定準則為基礎的生產軟體更新集合**：一般而言，使用穩定準則來建立可供清查和散發軟體更新的集合，有助於簡化軟體更新管理程序的各個階段。 穩定準則可以包含已安裝的用戶端作業系統版本和 Service Pack 層級、系統角色或目標組織。
* **建立包含參考電腦的預先生產集合**：預先生產集合應該包含作業系統版本、企業營運軟體以及您企業中執行的其他軟體的代表性組態。

您也應該考慮軟體更新伺服器將位於何處︰位於雲端的 Azure IaaS 基礎結構中或在內部部署中。 這是一項重要決策，因為您需要評估內部部署資源與 Azure 基礎結構之間的流量。 如需有關如何將內部部署基礎結構連接至 Azure 的詳細資訊，請參閱[將內部部署網路連接到 Microsoft Azure 虛擬網路](https://technet.microsoft.com/library/Dn786406.aspx)。

根據您目前的基礎結構以及您目前使用的軟體更新系統而定，用來決定更新伺服器將位於何處的設計選項也會有所不同。 對於 WSUS，請參閱[在您的組織中部署 Windows Server Update Services](https://technet.microsoft.com/library/hh852340.aspx)，對於 System Center Configuration Manager，請參閱[在 Configuration Manager 中規劃網站和階層](https://technet.microsoft.com/library/Gg712681.aspx)。

### <a name="backup"></a>備份
定期備份很重要，不只是對軟體更新管理平台本身而言，對即將更新的伺服器而言也很重要。 採用[變更管理程序](https://technet.microsoft.com/library/cc543216.aspx)的組織會要求 IT 部門合理解釋需要更新伺服器的原因、預估的停機時間以及可能的影響。 為了確保在更新失敗時能採用回復組態，請務必定期備份系統。

Azure IaaS 的某些備份選項包括：

* [使用 Data Protection Manager 的 Azure IaaS 工作負載保護](https://azure.microsoft.com/blog/2014/09/08/azure-iaas-workload-protection-using-data-protection-manager/)
* [備份 Azure 虛擬機器](../backup/backup-azure-vms.md)

### <a name="monitoring"></a>監視
您應該執行定期報告，以針對每個已獲授權的軟體更新，監視遺漏或已安裝的更新數目，或狀態不完整的更新。 同樣地，報告未獲授權的軟體更新可讓部署決策變得更容易。

您也應考量下列工作：

* 進行公司內所有電腦的適用和已安裝安全性更新的稽核。
* 授權更新並部署到適當的電腦。
* 追蹤公司內所有電腦的清查和更新安裝狀態和進度。

除了本文中說明的一般考量以外，您也應該考慮每項產品的最佳作法，例如︰如果您在採用 SQL Server 的 Azure 中有 VM，請確保遵循該產品的軟體更新建議。

## <a name="next-steps"></a>後續步驟
使用本文所述的指導方針，可協助您判斷適用於 Azure IaaS 中虛擬機器軟體更新的最佳選項。 傳統資料中心與 Azure IaaS 之間的軟體更新最佳作法有許多相似之處，因此建議您評估目前的軟體更新原則，以包含 Azure VM 並將本文中的相關最佳作法章納入整體軟體更新程序中。




<!--HONumber=Nov16_HO3-->


