---
title: "使用企業整合套件建立 B2B 解決方案 | Microsoft Docs"
description: "了解如何使用企業整合套件的 B2B 功能接收資料"
services: logic-apps
documentationcenter: .net,nodejs,java
author: msftman
manager: erikre
editor: cgronlun
ms.assetid: 20fc3722-6f8b-402f-b391-b84e9df6fcff
ms.service: logic-apps
ms.workload: integration
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 07/08/2016
ms.author: deonhe
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: dbf5e3bcdbaccfb4cd3d25a94d1070d6ca9d0bc6


---
# <a name="learn-about-receiving-data-using-the-b2b-features-of-the-enterprise-integration-pack"></a>了解如何使用企業整合套件的 B2B 功能接收資料
## <a name="overview"></a>Overview
這份文件是 Logic Apps 企業整合套件的一部分。 查看概觀以深入了解[企業整合套件的功能](app-service-logic-enterprise-integration-overview.md)。

## <a name="prerequisites"></a>必要條件
若要使用 AS2 和 X12 動作，您需要企業整合帳戶

[如何建立企業整合帳戶](app-service-logic-enterprise-integration-accounts.md)

## <a name="how-to-use-the-logic-apps-b2b-connectors"></a>如何使用 Logic Apps B2B 連接器
一旦建立整合帳戶並在其中新增夥伴和合約之後，您就可以建立邏輯應用程式來實作企業對企業 (B2B) 工作流程。

在此逐步解說中，您將看到如何使用 AS2 和 X12 動作，來建立企業對企業邏輯應用程式，以接收來自交易夥伴的資料。

1. 建立新的邏輯應用程式，並[將它連結到您的整合帳戶](app-service-logic-enterprise-integration-accounts.md)。  
2. 將 [要求 - 收到 HTTP 要求時] 觸發程序新增到您的邏輯應用程式  
   ![](./media/app-service-logic-enterprise-integration-b2b/flatfile-1.png)  
3. 先選取 [新增動作] 來新增 [將 AS2 解碼] 動作  
   ![](./media/app-service-logic-enterprise-integration-b2b/transform-2.png)  
4. 在搜尋方塊中輸入 **as2**，篩選所有動作以取得您想要使用的動作  
   ![](./media/app-service-logic-enterprise-integration-b2b/b2b-5.png)  
5. 選取 [AS2 - 將 AS2 訊息解碼] 動作  
   ![](./media/app-service-logic-enterprise-integration-b2b/b2b-6.png)  
6. 如上述，新增您將取得來做為輸入的 **本文** 。 在此範例中，選取觸發邏輯應用程式的 HTTP 要求本文。 或者，您可以輸入運算式，以便在 [標頭] 欄位中輸入標頭：
   
    @triggerOutputs()['headers']
7. 新增 AS2 所需的 **標頭** 。 這些項目將位於 HTTP 要求標頭中。 在此範例中，選取觸發邏輯應用程式的 HTTP 要求標頭。
8. 現在，再次選取 [新增動作] 來新增 [將 X12 訊息解碼] 動作  
   ![](./media/app-service-logic-enterprise-integration-b2b/b2b-9.png)   
9. 在搜尋方塊中輸入 **x12**，篩選所有動作以取得您想要使用的動作  
   ![](./media/app-service-logic-enterprise-integration-b2b/b2b-10.png)  
10. 選取 [X12 - 將 X12 解碼訊息] 動作，以將它新增到邏輯應用程式  
    ![](./media/app-service-logic-enterprise-integration-b2b/b2b-as2message.png)  
11. 現在，您需要指定此動作的輸入將會是上述 AS2 動作的輸出。 實際訊息內容是在 JSON 物件中，並以 base64 編碼。 您因而需要指定運算式做為輸入，因此，請在 [要解碼的 X12 一般檔案訊息]  輸入欄位中輸入下列運算式  
    
    @base64ToString(body('Decode_AS2_message')?['AS2Message']?['Content'])  
12. 此步驟會將交易夥伴所傳送的 X12 資料解碼，並將輸出 JSON 物件中的一些項目。 為了讓夥伴得知資料回條，您可以回送一個回應，在 HTTP 回應動作中包含 AS2 訊息處置通知 (MDN)  
13. 選取 [新增動作] 來新增 [回應] 動作   
    ![](./media/app-service-logic-enterprise-integration-b2b/b2b-14.png)  
14. 在搜尋方塊中輸入**回應**，篩選所有動作以取得您想要使用的動作  
    ![](./media/app-service-logic-enterprise-integration-b2b/b2b-15.png)  
15. 選取 [回應] 動作來新增它  
    ![](./media/app-service-logic-enterprise-integration-b2b/b2b-16.png)  
16. 使用下列運算式從 [將 X12 訊息解碼] 動作的輸出中存取 MDN，以設定回應 [本文] 欄位  
    
    @base64ToString(body('Decode_AS2_message')?['OutgoingMdn']?['Content'])  

![](./media/app-service-logic-enterprise-integration-b2b/b2b-17.png)  

1. 儲存您的工作   
   ![](./media/app-service-logic-enterprise-integration-b2b/transform-5.png)  

此時，您已完成設定 B2B 邏輯應用程式。 在真實世界應用程式中，您可能想要在 LOB 應用程式或資料存放區中儲存已解碼的 X12 資料。 您可以輕鬆地新增進一步動作來執行此操作，或撰寫自訂 API 以連接到您自己的 LOB 應用程式，並在您的邏輯應用程式中使用這些 API。

## <a name="features-and-use-cases"></a>功能和使用案例
* AS2 和 X12 解碼和編碼動作可讓您利用 Logic Apps，使用業界標準通訊協定接收來自交易夥伴的資料或將資料傳送給他們  
* 您可以使用 AS2 和 X12，視需要與交易夥伴交換資料
* B2B 動作可輕易地在整合帳戶中建立夥伴和合約，並在邏輯應用程式中使用他們  
* 您可以利用其他動作來擴充您的邏輯應用程式，以便將資料傳送給其他應用程式和服務 (例如 SalesForce)，或從中接收資料  

## <a name="learn-more"></a>詳細資訊
[深入了解企業整合套件](app-service-logic-enterprise-integration-overview.md)  




<!--HONumber=Nov16_HO3-->


