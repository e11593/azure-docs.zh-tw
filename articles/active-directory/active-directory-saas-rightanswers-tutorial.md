---
title: "教學課程：Azure Active Directory 與 RightAnswers 整合 | Microsoft Docs"
description: "了解如何使用 RightAnswers 搭配 Azure Active Directory 來啟用單一登入、自動佈建和更多功能！"
services: active-directory
author: jeevansd
documentationcenter: na
manager: femila
ms.assetid: 7f09e25a-a716-41e1-8ca3-fd00e3d1b8cc
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/26/2016
ms.author: jeedes
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 6c08e0b1349a3eb1bb4876958e36172e87729330


---
# <a name="tutorial-azure-active-directory-integration-with-rightanswers"></a>教學課程：Azure Active Directory 與 RightAnswers 整合
本教學課程的目的是要示範 Azure 與 RightAnswers 的整合。 本教學課程中說明的案例假設您已經具有下列項目：

* 有效的 Azure 訂用帳戶
* 已啟用 RightAnswers 單一登入的訂用帳戶

完成本教學課程之後，您指派給 RightAnswers 的 Azure AD 使用者就能夠使用 [存取面板](active-directory-saas-access-panel-introduction.md)來單一登入應用程式。

本教學課程中說明的案例由下列建置組塊組成：

1. 啟用 RightAnswers 的應用程式整合
2. 設定單一登入
3. 設定使用者佈建
4. 指派使用者

![案例](./media/active-directory-saas-rightanswers-tutorial/IC802925.png "Scenario")

## <a name="enabling-the-application-integration-for-rightanswers"></a>啟用 RightAnswers 的應用程式整合
本節的目的是要說明如何啟用 RightAnswers 的應用程式整合。

### <a name="to-enable-the-application-integration-for-rightanswers-perform-the-following-steps"></a>若要啟用 RightAnswers 的應用程式整合，請執行下列步驟：
1. 在 Azure 傳統入口網站中，按一下左方瀏覽窗格的 [Active Directory] 。
   
   ![Active Directory](./media/active-directory-saas-rightanswers-tutorial/IC700993.png "Active Directory")
2. 從 [目錄]  清單中，選取要啟用目錄整合的目錄。
3. 若要開啟應用程式檢視，請在目錄檢視中，按一下頂端功能表中的 [應用程式]  。
   
   ![應用程式](./media/active-directory-saas-rightanswers-tutorial/IC700994.png "Applications")
4. 按一下頁面底部的 [新增]  。
   
   ![新增應用程式](./media/active-directory-saas-rightanswers-tutorial/IC749321.png "Add application")
5. 在 [欲執行動作] 對話方塊上，按一下 [從資源庫中新增應用程式]。
   
   ![從組件庫新增應用程式](./media/active-directory-saas-rightanswers-tutorial/IC749322.png "Add an application from gallerry")
6. 在**搜尋方塊**中，輸入 **RightAnswers**。
   
   ![應用程式庫](./media/active-directory-saas-rightanswers-tutorial/IC802926.png "Application Gallery")
7. 在結果窗格中選取 [RightAnswers]，然後按一下 [完成] 來新增應用程式。
   
   ## <a name="configuring-single-sign-on"></a>設定單一登入

本節的目的是要說明如何依據 SAML 通訊協定來使用同盟，讓使用者能夠用自己的 Azure AD 帳戶驗證到 RightAnswers。

### <a name="to-configure-single-sign-on-perform-the-following-steps"></a>若要設定單一登入，請執行下列步驟：
1. 在 Azure 傳統入口網站的 [RightAnswers] 應用程式整合頁面上，按一下 [設定單一登入] 來開啟 [設定單一登入] 對話方塊。
   
   ![設定單一登入](./media/active-directory-saas-rightanswers-tutorial/IC802927.png "Configure Single Sign-On")
2. 在 [要如何讓使用者登入 RightAnswers] 頁面上，選取 [Microsoft Azure AD 單一登入]，然後按 [下一步]。
   
   ![設定單一登入](./media/active-directory-saas-rightanswers-tutorial/IC802928.png "Configure Single Sign-On")
3. 在 [設定應用程式 URL] 頁面的 [登入 URL] 文字方塊中，輸入使用者用來登入RightAnswers 應用程式的 URL (例如：https://fortify.rightanswers.com/portal/ss/)，然後按 [下一步]。
   
   ![設定 App 設定](./media/active-directory-saas-rightanswers-tutorial/IC802929.png "Configure App Settings")
4. 於 [設定在 RightAnswers 單一登入] 頁面上，按 [下載中繼資料] 來下載您的中繼資料，然後將中繼資料檔儲存在您的本機電腦上。
   
   ![設定單一登入](./media/active-directory-saas-rightanswers-tutorial/IC802930.png "Configure Single Sign-On")
5. 將下載的中繼資料檔傳送至 RightAnswers 支援小組。
   
   > [!NOTE]
   > RightAnswers 支援小組必須執行實際的 SSO 設定。
   > 當您的訂用帳戶啟用 SSO 之後，您會收到通知。
   > 
   > 
6. 在 Azure 傳統入口網站上，選取單一登入設定確認，然後按一下 [完成] 來關閉 [設定單一登入] 對話方塊。
   
   ![設定單一登入](./media/active-directory-saas-rightanswers-tutorial/IC802931.png "Configure Single Sign-On")
   
   ## <a name="configuring-user-provisioning"></a>設定使用者佈建

若要讓 Azure AD 使用者可以登入 RightAnswers，則必須將他們佈建至 RightAnswers。  
在 RightAnswersd 的情況下，佈建是自動化工作。  
沒有您需要使用的動作項目。

第一次嘗試單一登入時，會視需要自動建立使用者。

> [!NOTE]
> 您可以使用任何其他的 RightAnswers 使用者帳戶建立工具或 RightAnswers 提供的 API 來佈建 AAD 使用者帳戶。
> 
> 

## <a name="assigning-users"></a>指派使用者
若要測試您的組態，則需指派您所允許使用您應用程式的 Azure AD 使用者，藉此授予其存取組態的權限。

### <a name="to-assign-users-to-rightanswers-perform-the-following-steps"></a>若要將使用者指派給 RightAnswers，請執行下列步驟：
1. 在 Azure 傳統入口網站中建立測試帳戶。
2. 在 [RightAnswers] 應用程式整合頁面上，按一下 [指派使用者]。
   
   ![指派使用者](./media/active-directory-saas-rightanswers-tutorial/IC802932.png "Assign Users")
3. 選取測試使用者，按一下 [指派]，然後按一下 [是] 以確認指派。
   
   ![是](./media/active-directory-saas-rightanswers-tutorial/IC767830.png "Yes")

如果要測試您的單一登入設定，請開啟存取面板。 如需 [存取面板] 的詳細資訊，請參閱 [存取面板簡介](active-directory-saas-access-panel-introduction.md)。




<!--HONumber=Nov16_HO3-->


