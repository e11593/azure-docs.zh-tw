---
title: "Azure AD Connect：後續步驟及如何管理 Azure AD Connect | Microsoft Docs"
description: "了解如何針對 Azure AD Connect 擴充預設組態和作業工作。"
services: active-directory
documentationcenter: 
author: billmath
manager: femila
editor: curtand
ms.assetid: c18bee36-aebf-4281-b8fc-3fe14116f1a5
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/01/2016
ms.author: billmath
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 20195cb0d653cf0b1dc87af0fedbf9e49f259737


---
# <a name="next-steps-and-how-to-manage-azure-ad-connect"></a>接下來的步驟，以及如何管理 Azure AD Connect
以下這些進階的選擇性主題可讓您自訂 Azure Active Directory Connect，以符合您組織的需要和需求。  

## <a name="add-additional-sync-administrators"></a>加入額外的同步處理管理員
預設只有執行安裝的使用者和本機系統管理員才能夠管理已安裝的同步處理引擎。 如想讓其他人存取及管理同步處理引擎，請找到本機伺服器上名為 ADSyncAdmins 的群組，將這些人加入這個群組中。

## <a name="assigning-licenses-to-azure-ad-premium-and-enterprise-mobility-users"></a>指派授權給 Azure AD Premium 和 Enterprise Mobility 使用者
您的使用者現在已同步到雲端，您將需要指派授權給他們，才能繼續使用雲端應用程式 (例如 Office 365)。

### <a name="to-assign-an-azure-ad-premium-or-enterprise-mobility-suite-license"></a>指派 Azure AD Premium 或 Enterprise Mobility Suite 授權
- - -
1. 以系統管理員身分登入 Azure 入口網站。
2. 選取左邊的 [Active Directory] 。
3. 在 [Active Directory] 頁面上，在有您要啟用的使用者之目錄上按兩下。
4. 在目錄頁面頂端，選取 [授權] 。
5. 在 [授權] 頁面上，選取 [Active Directory Premium] 或 [Enterprise Mobility Suite]，然後按一下 [指派] 。
6. 在對話方塊中，選取您要對其指派授權的使用者，然後按一下核取記號圖示，以儲存變更。

## <a name="verifying-the-scheduled-synchronization-task"></a>確認已排程的同步處理工作
如果想要查看同步處理的狀態，您可以在 Azure 入口網站查看。

### <a name="to-verify-the-scheduled-synchronization-task"></a>確認已排程的同步處理工作
- - -
1. 以系統管理員身分登入 Azure 入口網站。
2. 選取左邊的 [Active Directory] 。
3. 在 [Active Directory] 頁面上，在有您要啟用的使用者之目錄上按兩下。
4. 在 [目錄] 頁面的頂端，選取 [目錄整合] 。
5. 請注意與本機 Active Directory 整合下方的上次同步處理時間。

<center>![雲端](./media/active-directory-aadconnect-whats-next/verify.png)</center>

## <a name="starting-a-scheduled-synchronization-task"></a>啟動已排程的同步處理工作
如果您需要執行同步處理工作，可以再次執行「Azure AD Connect 精靈」。  您需要提供 Azure AD 認證。  在精靈中，選取 [自訂同步處理選項]  工作，並依精靈指示繼續按 [下一步]。 在最後步驟中，確定已核取 [設定一完成，即開始同步處理程序]  方塊。

<center>![雲端](./media/active-directory-aadconnect-whats-next/startsynch.png)</center>

如需 Azure AD Connect 同步：排程器的詳細資訊，請參閱 [Azure AD Connect 排程器](active-directory-aadconnectsync-feature-scheduler.md)

## <a name="additional-tasks-available-in-azure-ad-connect"></a>Azure AD Connect 中可用的其他工作
在初始 Azure AD Connect 的安裝之後，您隨時可以從 Azure AD Connect 開始頁面或桌面捷徑啟動精靈。  您會在再次執行精靈的過程中發現，一些新的選項以「其他工作」的形式提供。  

下表提供這些工作的摘要和個別的簡短描述。

![加入規則](./media/active-directory-aadconnect-whats-next/addtasks.png)

| 其他工作 | 說明 |
| --- | --- |
| 檢視所選取的案例 |可讓您檢視目前的 Azure AD Connect 解決方案。  這包括一般設定、同步處理的目錄、同步處理設定等等。 |
| 自訂同步處理選項 |可讓您變更目前的組態，包括新增其他 Active Directory 樹系到組態或啟用同步處理選項，例如使用者、群組、裝置或密碼回寫。 |
| 啟用預備模式 |這可讓您預備稍後將要同步處理的資訊，但不會將任何項目匯出到 Azure AD 或 Active Directory。  這可讓您在同步處理發生之前先預覽。 |

## <a name="next-steps"></a>後續步驟
深入了解 [整合內部部署身分識別與 Azure Active Directory](active-directory-aadconnect.md)。




<!--HONumber=Nov16_HO3-->


