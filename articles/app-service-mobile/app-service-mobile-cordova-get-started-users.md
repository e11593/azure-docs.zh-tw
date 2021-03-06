---
title: "使用 Mobile Apps 在 Apache Cordova 上新增驗證 | Microsoft Docs"
description: "了解如何在 Azure App Service 中使用 Mobile Apps，透過眾多識別提供者驗證 Apache Cordova 應用程式使用者，包括 Google、Facebook、Twitter 和 Microsoft。"
services: app-service\mobile
documentationcenter: javascript
author: adrianhall
manager: erikre
editor: 
ms.assetid: 10dd6dc9-ddf5-423d-8205-00ad74929f0d
ms.service: app-service-mobile
ms.workload: na
ms.tgt_pltfrm: mobile-html
ms.devlang: javascript
ms.topic: article
ms.date: 10/01/2016
ms.author: adrianha
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: a5da863c7cbd5760a15f5e72fa7a884973ed38af


---
# <a name="add-authentication-to-your-apache-cordova-app"></a>新增驗證至您的 Apache Cordova 應用程式
[!INCLUDE [app-service-mobile-selector-get-started-users](../../includes/app-service-mobile-selector-get-started-users.md)]

## <a name="summary"></a>Summary
在本教學課程中，您可以使用支援的身分識別提供者，將驗證加入 Apache Cordova 上的 TodoList 快速入門專案。 本教學課程以 [開始使用 Mobile Apps] 為基礎，您必須先完成該教學課程。

## <a name="a-nameregisteraregister-your-app-for-authentication-and-configure-the-app-service"></a><a name="register"></a>註冊應用程式進行驗證，並設定應用程式服務
[!INCLUDE [app-service-mobile-register-authentication](../../includes/app-service-mobile-register-authentication.md)]

[觀看示範類似步驟的影片](https://channel9.msdn.com/series/Azure-connected-services-with-Cordova/Azure-connected-services-task-8-Azure-authentication)

## <a name="a-namepermissionsarestrict-permissions-to-authenticated-users"></a><a name="permissions"></a>限制只有通過驗證的使用者具有權限
[!INCLUDE [app-service-mobile-restrict-permissions-dotnet-backend](../../includes/app-service-mobile-restrict-permissions-dotnet-backend.md)]

現在，您可以驗證是否已停用後端的匿名存取。 當您完成 [開始使用 Mobile Apps]教學課程時，請在 Visual Studio 中開啟您建立的專案，然後在 **Google Android 模擬器** 中執行您的應用程式，並驗證「未預期的連線錯誤」會在啟動應用程式之後顯示。

接下來，您要將應用程式更新為在要求行動應用程式後端的資源之前必須驗證使用者。

## <a name="a-nameadd-authenticationaadd-authentication-to-the-app"></a><a name="add-authentication"></a>將驗證新增至應用程式
1. 在 **Visual Studio** 中開啟您的專案，然後開啟 `www/index.html` 檔案進行編輯。
2. 找出 head 區段中的 `Content-Security-Policy` 中繼標籤。  您必須將 OAuth 主機新增至允許的來源清單。
   
   | 提供者 | SDK 提供者名稱 | OAuth 主機 |
   |:--- |:--- |:--- |
   | Azure Active Directory |aad |https://login.windows.net |
   | Facebook |Facebook |https://www.facebook.com |
   | Google |Google |https://accounts.google.com |
   | Microsoft |microsoftaccount |https://login.live.com |
   | Twitter |Twitter |https://api.twitter.com |
   
    content-security-policy (針對 Azure Active Directory 實作) 的範例如下所示：
   
        <meta http-equiv="Content-Security-Policy" content="default-src 'self'
            data: gap: https://login.windows.net https://yourapp.azurewebsites.net; style-src 'self'">
   
    您應該使用上表中的 OAuth 主機取代 `https://login.windows.net` 。  如需此中繼標籤的詳細資訊，請參閱 [Content-Security-Policy 文件] 。
   
    請注意，在適當的行動裝置上使用時，某些驗證提供者不需要 Content-Security-Policy 變更。  例如，在 Android 裝置上使用 Google 驗證時不需要 Content-Security-Policy 變更。
3. 開啟 `www/js/index.js` 檔案進行編輯，找出 `onDeviceReady()` 方法，並在用戶端建立程式碼底下加入下列內容：
   
        // Login to the service
        client.login('SDK_Provider_Name')
            .then(function () {
   
                // BEGINNING OF ORIGINAL CODE
   
                // Create a table reference
                todoItemTable = client.getTable('todoitem');
   
                // Refresh the todoItems
                refreshDisplay();
   
                // Wire up the UI Event Handler for the Add Item
                $('#add-item').submit(addItemHandler);
                $('#refresh').on('click', refreshDisplay);
   
                // END OF ORIGINAL CODE
   
            }, handleError);
   
    請注意，此程式碼會取代現有用於建立資料表參考和重新整理 UI 的程式碼。
   
    login () 方法會開始向提供者驗證。 login() 方法是會傳回 JavaScript Promise 的非同步函式。  初始化作業的其餘部分會置於承諾回應中，如此就不會在 login() 方法完成之前執行。
4. 在您剛才加入的程式碼中，使用您的登入提供者名稱取代 `SDK_Provider_Name` 。 例如，針對 Azure Active Directory，請使用 `client.login('aad')`。
5. 執行專案。  當專案完成初始化時，您的應用程式會針對選擇的驗證提供者顯示 OAuth 登入頁面。

## <a name="a-namenext-stepsanext-steps"></a><a name="next-steps"></a>後續步驟
* 深入了解 Azure App Service [驗證相關資訊] 。
* 將 [推播通知] 新增至 Apache Cordova 應用程式，以繼續本教學課程。

了解如何使用 SDK。

* [Apache Cordova SDK]
* [ASP.NET Server SDK]
* [Node.js Server SDK]

<!-- URLs. -->
[開始使用 Mobile Apps]: app-service-mobile-cordova-get-started.md
[Content-Security-Policy 文件]: https://cordova.apache.org/docs/en/latest/guide/appdev/whitelist/index.html
[推播通知]: app-service-mobile-cordova-get-started-push.md
[驗證相關資訊]: app-service-mobile-auth.md
[Apache Cordova SDK]: app-service-mobile-cordova-how-to-use-client-library.md 
[ASP.NET Server SDK]: app-service-mobile-dotnet-backend-how-to-use-server-sdk.md
[Node.js Server SDK]: app-service-mobile-node-backend-how-to-use-server-sdk.md



<!--HONumber=Nov16_HO3-->


