---
title: "服務恢復指引 | Microsoft Azure"
description: "Microsoft Azure 服務的災害復原和主動式恢復和可用性指引的連結。"
services: 
documentationcenter: na
author: adamglick
manager: saladki
editor: 
ms.assetid: 919a197d-ba47-4e49-a64c-118e27d5a7df
ms.service: resiliency
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/18/2016
ms.author: aglick
translationtype: Human Translation
ms.sourcegitcommit: aafb03c86884d1af3e2cae1134c9e6d7ac42397b
ms.openlocfilehash: 4186211af944c0c0109799a94e4163cfceeee4c4


---
# <a name="microsoft-azure-service-resiliency-guidance"></a>Microsoft Azure 服務恢復指引
Microsoft Azure 的設計目的是要提供您所需的資源 (需要時)。 作為良好的設計與操作實務的一部分，您應該知道同時如何架構對於 Azure 服務的使用，才能達到高可用性，以及您的應用程式受到服務中斷的影響時該怎麼做。 為了協助您進行此程序，本文件包含災害復原指引以及各種 Azure 服務的設計指引的連結。

## <a name="disaster-recovery-guidance"></a>災害復原指引
下列的災害復原指引連結可以提供您所需的資訊，以便在您受到 Azure 服務中斷影響時，協助您快速讓應用程式恢復上線。 建立這些連結的目的是要協助您回答這個問題，「我正受到 Azure 服務中斷的影響，我該怎麼辦？」

## <a name="design-guidance"></a>設計指引
以下的設計指引連結為設計和架構指引，其建立目的是要協助您了解如何以能夠充分發揮您的應用程式執行時間的方式，來使用每個 Azure 服務。 建立這些連結的目的是要協助您回答這個問題，「我如何確定錯誤、硬體故障、服務中斷或其他故障不會影響我的應用程式整體的可用性？ 」 如果沒有適用於您目前正在尋找之服務的專用指引，則 [建置在 Microsoft Azure 上之應用程式的高可用性](resiliency-high-availability-azure-applications.md) 一文可能含有其他可協助您進行設計的資訊。

## <a name="service-guidance"></a>服務指引
| 服務 | 災害復原指引 | 設計指引 |
|:--- |:---:|:---:|
| [雲端服務](https://azure.microsoft.com/services/cloud-services/ "Azure 雲端服務") |[link](../cloud-services/cloud-services-disaster-recovery-guidance.md "Azure 雲端服務災害復原指引") |無法使用 |
| [金鑰保存庫](https://azure.microsoft.com/services/key-vault/ "Azure 金鑰保存庫") |[link](../key-vault/key-vault-disaster-recovery-guidance.md "Azure 金鑰保存庫災害復原指引") |無法使用 |
| [儲存體](https://azure.microsoft.com/services/storage/ "Azure 儲存體") |[link](../storage/storage-disaster-recovery-guidance.md "Azure 儲存體災害復原指引") |無法使用 |
| [SQL Database](https://azure.microsoft.com/services/sql-database/ "Azure SQL Database") |[link](../sql-database/sql-database-disaster-recovery.md "Azure SQL Database 災害復原指引") |[link](../sql-database/sql-database-performance-guidance.md "Azure SQL Database 設計指引") |
| [虛擬機器](https://azure.microsoft.com/services/virtual-machines/ "Azure 虛擬機器") |[link](../virtual-machines/virtual-machines-disaster-recovery-guidance.md "Azure 虛擬機器災害復原指引") |無法使用 |
| [虛擬網路](https://azure.microsoft.com/services/virtual-network/ "Azure 虛擬網路") |[link](../virtual-network/virtual-network-disaster-recovery-guidance.md "Azure 虛擬網路災害復原指引") |無法使用 |

## <a name="next-steps"></a>後續步驟
如果您正在尋找著重在更廣泛的系統與解決方案的指引，請參閱 [建置在 Microsoft Azure 上之應用程式的災害復原和高可用性](https://aka.ms/drtechguide)。



<!--HONumber=Nov16_HO3-->


