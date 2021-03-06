---
title: "Azure AD Connect：單一登入 | Microsoft Docs"
description: "本主題提供您所需資訊，以讓您了解如何使用單一登入從內部部署 Active Directory (AD) 同步處理登入至雲端式 Azure Active Directory (Azure AD) 和已連線服務。"
services: active-directory
keywords: "何謂 Azure AD Connect、安裝 Active Directory、Azure AD、SSO、單一登入的必要元件"
documentationcenter: 
author: billmath
manager: femila
ms.assetid: 9f994aca-6088-40f5-b2cc-c753a4f41da7
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/06/2016
ms.author: billmath
translationtype: Human Translation
ms.sourcegitcommit: bddd581b5164b03ccba75370cd89a7831101cf37
ms.openlocfilehash: f0133aaf735533159d72b1e63b952258707ad8b9


---

# <a name="what-is-single-sign-on-sso-preview"></a>何謂單一登入 (SSO) (預覽)
單一登入是可在 Azure Active Directory Connect 中使用[密碼雜湊同步處理](active-directory-aadconnectsync-implement-password-synchronization.md)或[傳遞驗證](active-directory-aadconnect-pass-through-authentication.md)啟用的一個選項。  啟用之後，當使用者在他們的公司電腦上且已經連線至公司網路時，使用者不需要輸入他們的密碼就能登入 Azure Active Directory (Azure AD) 或其他雲端服務。  

透過啟用單一登入，當您的使用者存取已連線至 Azure Active Directory 服務 (例如 Office 365、Microsoft Intune、CRM Online 及 SaaS 服務) 的資源時，使用者不需要輸入他們的密碼，而是改為使用自動從您內部部署 Active Directory 取得的 Kerberos 票證。

![單一登入](./media/active-directory-aadconnect-sso/sso1.png)

透過為使用者提供 SSO，來提供更熟悉的雲端式服務存取方式，並為組織提供安全、簡單，且不需要任何其他內部部署元件的程序。

SSO 是一個透過 AAD Connect 啟用的功能，且會和密碼雜湊同步處理或傳遞驗證及您的內部部署 Active Directory 搭配運作。  若要讓您的使用者在您的環境中使用單一登入，您需要確定使用者：


- 在已加入網域的電腦上
- 直接連線至網域控制站，例如在企業有線或無線網路上，或透過遠端存取連線 (例如 VPN 連線) 連線。
- 將雲端的 Kerberos 端點定義為內部網路區域的一部分。

如果缺少上述任何一個項目，例如電腦已經離開公司網路且 Active Directory 無法使用，將會提示使用者輸入他們的密碼，就像在沒有使用單一登入時一樣。

## <a name="supported-clients"></a>支援的用戶端
單一登入是透過能夠使用 Kerberos 驗證的電腦 (例如 Windows) 上，支援新式驗證的 Web 瀏覽器型用戶端和支援 Office 用戶端來支援。  下面的矩陣提供多種作業系統上瀏覽器架構用戶端的詳細資料。

| 作業系統\瀏覽器 |Internet Explorer|Chrome|Firefox|Edge
| --- | --- |--- | --- |--- |
|Windows 10|是|是|是*|否
|Windows 8.1|是|是|是*|N/A
|Windows 8|是|是|是*|N/A
|Windows 7|是|是|是*|N/A
|Mac|N/A|N/A|N/A|N/A

*需要個別組態。

>[!NOTE]
>對於 Windows 10 架構用戶端，建議使用 [Azure AD Join](../active-directory-azureadjoin-overview.md) 以獲得 Azure AD 最佳使用體驗。 

## <a name="how-single-sign-on-works"></a>單一登入運作方式

當您在 Azure AD Connect 中啟用單一登入時，會在內部部署 Active Directory 中建立名為 AZUREADSSOACCT 的電腦帳戶，並和 Azure AD 安全地共用 Kerberos 解密金鑰。  此外，會建立兩個 Kerberos 服務主體名稱 (SPN)，以代表在用戶端和 Azure AD 之間進行驗證證期間使用的雲端 URL。  

在此設定完成後，驗證程序就和任何其他整合式 Windows 驗證 (IWA) 架構應用程式一樣。  如果您熟悉 IWA 的運作方式，那麼您已經知道單一登入和 Azure AD 的搭配運作方式。  如果您不熟悉，那麼 IWA 的程序如下：

![單一登入](./media/active-directory-aadconnect-sso/sso2.png)

首先，使用者嘗試存取資源。  資源可以是電腦或 URL。  在使用 Azure AD 的情況下，它是服務 (例如 SharePoint Online) 的 URL，以下稱為「Azure AD 資源」。

1.  Azure AD 資源會透過 401 未授權回應挑戰用戶端，以提供 Kerberos 票證。
2.  用戶端向 Active Directory 要求 Azure AD 資源的票證。 
3.  Active Directory 會找出和 Azure AD 資源相關聯的電腦帳戶，並傳回 Kerberos 票證給用戶端，並使用電腦帳戶的密碼加密。 票證包括目前已登入電腦之使用者的身分識別。
4.  用戶端將它向 Active Directory 取得的 Kerberos 票證傳遞至 Azure AD。
5.  Azure AD 會使用之前共用的金鑰將 Kerberos 票證解密，然後將權杖傳回給使用者，或依照資源的要求來要求使用者提供其他證明，例如多重要素驗證。

單一登入是一個隨機功能，這表示如果它因為某些原因而失敗，使用者只需要像平常一樣在登入頁面上輸入他們的密碼即可。

## <a name="enabling-sso-with-pass-through-authentication-or-password-hash-sync"></a>搭配傳遞驗證或密碼雜湊同步處理來啟用 SSO
Azure AD Connect 提供簡單的程序來搭配傳遞驗證或密碼雜湊同步處理啟用單一驗證。  您將需要確保您擁有每個樹系內要同步處理的其中一個網域的網域系統管理員權限，才能允許在電腦帳戶上使用 Kerberos 服務主體名稱 (SPN) 的組態。  使用者名稱和密碼不會儲存在 Azure AD Connect 或 Azure AD 中，並且僅用於這項作業。

安裝 Azure AD Connect 時，請選取自訂安裝，您才能夠在使用者登入頁面上設定單一登入。 如需詳細資訊，請參閱[自訂 Azure AD Connect 安裝](active-directory-aadconnect-get-started-custom.md)。

![單一登入](./media/active-directory-aadconnect-sso/sso3.png)

啟用單一登入之後，您可以繼續執行精靈，直到您到達單一登入頁面為止。

![單一登入](./media/active-directory-aadconnect-sso/sso4.png)

針對每個列出的樹系，提供適當的帳戶詳細資料，就會為您的 Azure 目錄啟用單一登入。

## <a name="ensuring-clients-sign-in-automatically"></a>確保用戶端自動登入
依照預設，瀏覽器不會嘗試傳送認證至 Web 伺服器，除非 URL 是定義為位於內部網路區域中。  一般而言，瀏覽器可以透過查看 URL 來計算正確的區域。  例如，如果 URL 為 http://intranet/，瀏覽器將會自動傳送認證，因為它將會把 URL 對應到內部網路區域。  但是，如果 URL 包含句號，例如 http://intranet.contoso.com/，伺服器將不會自動傳送認證，並且會將 URL 視為任何網際網路網站。

因為 Azure AD 中用於單一登入的 URL 包含句號 (因為它們是可全域路由主機名稱)，所以必須將它們明確加入電腦的內部網路區域，瀏覽器才會以 Kerberos 票證的形式，將目前登入之使用者的認證自動傳送到 Azure AD。  將所需 URL 加入到內部網路區域的最簡單方式是，在 Active Directory 中建立群組原則。

1.  開啟 [群組原則管理工具]
2.  編輯會套用至所有使用者的群組原則。  例如，預設網域原則。
3.  瀏覽至 **User Configuration\Administrative Templates\Windows Components\Internet Explorer\Internet Control Panel\Security 頁面**，並選取 [指派網站到區域清單]。
![單一登入](./media/active-directory-aadconnect-sso/sso6.png) </br>
4.  啟用原則，並在對話方塊中輸入下列值/資料。</br>
        
        Value: https://autologon.microsoftazuread-sso.com
        Data: 1
        Value: https://aadg.windows.net.nsatc.net 
        Data: 1'
5.  看起來應該會像下面這樣：![單一登入](./media/active-directory-aadconnect-sso/sso7.png)

6.  按一下 [確定]，然後再按一下 [確定]。

您的使用者現在已經可以開始進行單一登入。

>[!NOTE]
>依照預設，Chrome 將會使用和 Internet Explorer 相同的一組信任的網站 URL。  如果您已經為 Chrome 設定不同的設定，您將需要個別更新這些設定。

## <a name="troubleshooting-single-sign-on-issues"></a>針對單一登入問題進行疑難排解
請務必確定已針對單一登入正確設定用戶端，包括下列項目：

1.  已在內部網路區域內定義 https://autologon.microsoftazuread-sso.com 和 https://aadg.windows.net.nsatc.net。  
2.  確定工作站已經加入網域。
3.  確定使用者已經使用網域帳戶登入。
4.  確定電腦已經連線到公司網路
5.  確定電腦的時間已經與 Active Directory 同步，且網域控制站時間是在正確時間的 5 分鐘內。

如果您已經能夠確認上述需求，您就可以檢閱瀏覽器的主控台記錄檔，以了解其他資訊。  主控台記錄檔可以在開發人員工具底下找到。  這將會協助您判斷潛在問題。

## <a name="event-log-entries"></a>事件記錄檔項目
如果已經成功啟用稽核，使用者每次使用單一登入登入時，就會在網域控制站的事件記錄檔中記錄一個項目。  若要尋找這些事件，您可以檢閱和電腦帳戶 AzureADSSOAcc$ 相關聯之安全性事件 4769 的事件記錄檔。  下面的篩選可找出和電腦帳戶相關聯的所有安全性事件：

    <QueryList>
      <Query Id="0" Path="Security">
    <Select Path="Security">*[EventData[Data[@Name='ServiceName'] and (Data='AZUREADSSOACC$')]]</Select>
      </Query>
    </QueryList>






<!--HONumber=Dec16_HO3-->


