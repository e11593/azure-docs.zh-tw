---
title: "教學課程：Azure Active Directory 與 Freshdesk 整合 | Microsoft Docs"
description: "了解如何使用 Freshdesk 搭配 Azure Active Directory 來啟用單一登入、自動化佈建和更多功能！"
services: active-directory
author: jeevansd
documentationcenter: na
manager: femila
ms.assetid: 31177994-7910-4a72-bd76-5fa6260242fb
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/29/2016
ms.author: jeedes
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 7043787c74c138984fdf5370784678d6dfcaa81e


---
# <a name="tutorial-azure-active-directory-integration-with-freshdesk"></a>教學課程：Azure Active Directory 與 Freshdesk 整合
本教學課程的目的是要示範 Azure 與 Freshdesk 的整合。  
本教學課程中說明的案例假設您已經具有下列項目：

* 有效的 Azure 訂閱
* Freshdesk 租用戶

完成本教學課程之後，您指派給 Freshdesk 的 Azure AD 使用者就能夠單一登入您 Freshdesk 公司網站 (服務提供者起始登入) 的應用程式，或是使用 [存取面板簡介](active-directory-saas-access-panel-introduction.md)。

本教學課程中說明的案例由下列建置組塊組成：

1. 啟用 Freshdesk 的應用程式整合
2. 設定單一登入
3. 設定使用者佈建
4. 指派使用者

![案例](./media/active-directory-saas-freshdesk-tutorial/IC776761.png "Scenario")

## <a name="enabling-the-application-integration-for-freshdesk"></a>啟用 Freshdesk 的應用程式整合
本節的目的是要說明如何啟用 Freshdesk 的應用程式整合。

### <a name="to-enable-the-application-integration-for-freshdesk-perform-the-following-steps"></a>若要啟用 Freshdesk 的應用程式整合，請執行下列步驟：
1. 在 Azure 傳統入口網站中，按一下左方瀏覽窗格的 [Active Directory] 。
   
   ![Active Directory](./media/active-directory-saas-freshdesk-tutorial/IC700993.png "Active Directory")
2. 從 [目錄]  清單中，選取要啟用目錄整合的目錄。
3. 若要開啟應用程式檢視，請在目錄檢視中，按一下頂端功能表中的 [應用程式]  。
   
   ![應用程式](./media/active-directory-saas-freshdesk-tutorial/IC700994.png "Applications")
4. 按一下頁面底部的 [新增]  。
   
   ![新增應用程式](./media/active-directory-saas-freshdesk-tutorial/IC749321.png "Add application")
5. 在 [欲執行動作] 對話方塊上，按一下 [從資源庫中新增應用程式]。
   
   ![從組件庫新增應用程式](./media/active-directory-saas-freshdesk-tutorial/IC749322.png "Add an application from gallerry")
6. 在**搜尋方塊**中，輸入 **Freshdesk**。
   
   ![應用程式庫](./media/active-directory-saas-freshdesk-tutorial/IC776762.png "Application gallery")
7. 在結果窗格中，選取 [Freshdesk]，然後按一下 [完成] 以加入應用程式。
   
   ![Freshdesk!](./media/active-directory-saas-freshdesk-tutorial/IC776763.png "Freshdesk")
   
   ## <a name="configuring-single-sign-on"></a>設定單一登入

本節的目的是要說明如何依據 SAML 通訊協定來使用同盟，讓使用者能夠用自己的 Azure AD 帳戶驗證至 Freshdesk。  
設定 Freshdesk 的單一登入需要您從憑證擷取指紋值。  
如果您不熟悉這個程序，請參閱 [如何擷取憑證的指紋值](http://youtu.be/YKQF266SAxI)。

### <a name="to-configure-single-sign-on-perform-the-following-steps"></a>若要設定單一登入，請執行下列步驟：
1. 在 Azure 傳統入口網站的 [Freshdesk] 應用程式整合頁面上，按一下 [設定單一登入] 來開啟 [設定單一登入] 對話方塊。
   
   ![設定單一登入](./media/active-directory-saas-freshdesk-tutorial/IC776764.png "Configure Single Sign-On")
2. 在 [您希望使用者如何登入 Freshdesk] 頁面上，選取 [Microsoft Azure AD 單一登入]，然後按 [下一步]。
   
   ![設定單一登入](./media/active-directory-saas-freshdesk-tutorial/IC776765.png "Configure Single Sign-On")
3. 在 [設定應用程式 URL] 頁面的 [Freshdesk 登入 URL] 文字方塊中，使用下列模式輸入您的 URL："*https://\<tenant-name\>.Freshdesk.com*"，然後按 [下一步]。
   
   ![設定應用程式 URL](./media/active-directory-saas-freshdesk-tutorial/IC776766.png "Configure App URL")
4. 於 [在 Freshdesk 設定單一登入] 頁面上，按 [下載憑證] 以下載您的憑證，然後在本機電腦上將憑證檔案儲存為 **c:\\Freshdesk.cer**。
   
   ![設定單一登入](./media/active-directory-saas-freshdesk-tutorial/IC776767.png "Configure Single Sign-On")
5. 在不同的網頁瀏覽器視窗中，以系統管理員身分登入您的 Freshdesk 公司網站。
6. 在頂端的功能表中，按一下 [系統管理員] 。
   
   ![Admin](./media/active-directory-saas-freshdesk-tutorial/IC776768.png "Admin")
7. 在 [一般設定] 索引標籤上，按一下 [安全性]。
   
   ![安全性](./media/active-directory-saas-freshdesk-tutorial/IC776769.png "Security")
8. 在 [安全性]  區段中，執行下列步驟：
   
   ![單一登入](./media/active-directory-saas-freshdesk-tutorial/IC776770.png "Single Sign On")
   
   1. 針對 [單一登入 (SSO)] 選取 [啟用]。
   2. 選取 [SAML SSO] 。
   3. 在 Azure 傳統入口網站中的 [在 Freshdesk 設定單一登入] 對話頁面上，複製 [遠端登入 URL] 值，然後將它貼至 [SAML 登入 URL] 文字方塊中。
   4. 在 Azure 傳統入口網站中的 [在 Freshdesk 設定單一登入] 對話頁面上，複製 [遠端登出 URL] 值，然後將它貼至 [登出 URL] 文字方塊中。
   5. 從匯出的憑證複製**指紋**值，然後將它貼入 [安全性憑證指紋] 文字方塊。  
      
      > [!TIP]
      > 如需詳細資訊，請參閱 [如何抓取憑證的指紋值](http://youtu.be/YKQF266SAxI)
      > 
      > 
   6. 按一下 [儲存] 。
9. 在 Azure 傳統入口網站上，選取單一登入設定確認，然後按一下 [完成] 來關閉 [設定單一登入] 對話方塊。
   
   ![設定單一登入](./media/active-directory-saas-freshdesk-tutorial/IC776771.png "Configure Single Sign-On")
   
   ## <a name="configuring-user-provisioning"></a>設定使用者佈建

若要讓 Azure AD 使用者可以登入 Freshdesk，則必須將他們佈建到 Freshdesk。  
Freshdesk 需以手動的方式佈建。

### <a name="to-provision-a-user-accounts-perform-the-following-steps"></a>若要佈建使用者帳戶，請執行下列步驟：
1. 登入您的 **Freshdesk** 租用戶。
2. 在頂端的功能表中，按一下 [系統管理員] 。
   
   ![Admin](./media/active-directory-saas-freshdesk-tutorial/IC776772.png "Admin")
3. 在 [一般設定] 索引標籤上，按一下 [代理程式]。
   
   ![代理程式](./media/active-directory-saas-freshdesk-tutorial/IC776773.png "Agents")
4. 按一下 [新增代理程式] 。
   
   ![新代理程式](./media/active-directory-saas-freshdesk-tutorial/IC776774.png "New Agent")
5. 在 [代理程式資訊] 對話方塊中，執行下列步驟：
   
   ![代理程式資訊](./media/active-directory-saas-freshdesk-tutorial/IC776775.png "Agent Information")
   
   1. 在 [全名]  文字方塊中，輸入您要佈建的 Azure AD 帳戶名稱。
   2. 在 [電子郵件]  文字方塊中，輸入您要佈建的 Azure AD 帳戶的電子郵件地址。
   3. 在 [職稱]  文字方塊中，輸入您要佈建的 Azure AD 帳戶的職稱。
   4. 選取 [代理程式角色]，然後按一下 [指派]。
   5. 按一下 [儲存] 。
      
      > [!NOTE]
      > Azure AD 帳戶的持有者會收到一封包含連結的電子郵件，以在啟用帳戶前進行確認。
      > 
      > 

> [!NOTE]
> 您可以使用任何其他的 Freshdesk 使用者帳戶建立工具或 Freshdesk 提供的 API 來佈建 AAD 使用者帳戶。
> 
> 

## <a name="assigning-users"></a>指派使用者
若要測試您的組態，則需指派您所允許使用您應用程式的 Azure AD 使用者，藉此授予其存取組態的權限。

### <a name="to-assign-users-to-freshdesk-perform-the-following-steps"></a>若要指派使用者給 Freshdesk，請執行下列步驟：
1. 在 Azure 傳統入口網站中建立測試帳戶。
2. 在 [Freshdesk] 應用程式整合頁面上，按一下 [指派使用者]。
   
   ![指派使用者](./media/active-directory-saas-freshdesk-tutorial/IC776776.png "Assign Users")
3. 選取測試使用者，按一下 [指派]，然後按一下 [是] 以確認指派。
   
   ![是](./media/active-directory-saas-freshdesk-tutorial/IC767830.png "Yes")

如果要測試您的單一登入設定，請開啟存取面板。 如需 [存取面板] 的詳細資訊，請參閱 [存取面板簡介](active-directory-saas-access-panel-introduction.md)。




<!--HONumber=Nov16_HO3-->


