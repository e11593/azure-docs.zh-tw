---
title: "在串流分析中設定查詢的警示 | Microsoft Docs"
description: "了解串流分析警示"
keywords: "設定警示"
services: stream-analytics
documentationcenter: 
author: jeffstokes72
manager: jhubbard
editor: cgronlun
ms.assetid: 9952e2cf-b335-4a5c-8f45-8d3e1eda2e20
ms.service: stream-analytics
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: data-services
ms.date: 01/24/2017
ms.author: jeffstok
translationtype: Human Translation
ms.sourcegitcommit: 219dcbfdca145bedb570eb9ef747ee00cc0342eb
ms.openlocfilehash: c8463e68600edefecd0f3bdd19c13b13f676b82b


---
# <a name="set-up-alerts-for-azure-stream-analytics-jobs"></a>設定 Azure 串流分析工作的警示
## <a name="introduction-monitor-page"></a>簡介：監視頁面
您可以設定會在計量達到您指定的條件時觸發的警示。

例如，「如果過去 15 分鐘內的輸出事件 <100，則將電子郵件通知傳送給電子郵件識別碼： xyz@company.com」。

您可以透過入口網站來針對計量設定規則，或是透過作業記錄檔資料以 [程式設計方式](https://code.msdn.microsoft.com/windowsazure/Receive-Email-Notifications-199e2c9a) 來設定規則。

## <a name="set-up-alerts-through-the-azure-classic-portal"></a>透過 Azure 傳統入口網站設定警示
在 Azure 傳統入口網站中設定警示的方式有兩種：  

1. 串流分析工作的 [監視]  索引標籤  
2. 管理服務中的作業記錄檔  

## <a name="set-up-alert-through-the-monitor-tab-of-the-job-in-the-portal"></a>透過入口網站中工作的 [監視] 索引標籤來設定警示
1. 在 [監視] 索引標籤中選取計量，然後按一下儀表板底部的 [新增規則]  按鈕，並設定您的規則。  
   
   ![儀表板](./media/stream-analytics-set-up-alerts/01-stream-analytics-set-up-alerts.png)  
2. 定義此警示的名稱和說明  
   
   ![建立規則](./media/stream-analytics-set-up-alerts/02-stream-analytics-set-up-alerts.png)  
3. 輸入臨界值、警示評估視窗與警示的動作  
   
   ![定義狀況](./media/stream-analytics-set-up-alerts/03-stream-analytics-set-up-alerts.png)  

## <a name="set-up-alerts-through-the-operations-logs"></a>透過作業記錄檔來設定警示
1. 移至 [Azure 傳統入口網站](https://manage.windowsazure.com)中 [管理服務] 的 [警示] 索引標籤。  
2. 按一下 [新增規則]   
   
   ![準則](./media/stream-analytics-set-up-alerts/04-stream-analytics-set-up-alerts.png)  
3. 定義此警示的名稱和說明。 選取「串流分析」做為服務類型，並選取工作名稱以做為服務名稱。  
   
   ![定義警示](./media/stream-analytics-set-up-alerts/05-stream-analytics-set-up-alerts.png)  

## <a name="set-up-alerts-in-the-azure-portal"></a>在 Azure 入口網站中設定警示
在 Azure 入口網站中，瀏覽至您有興趣設定警示的串流分析工作，然後按一下 [監視]  區段。  在開啟的 [度量] 刀鋒視窗中，按一下 [新增警示] 命令。

  ![Azure 入口網站設定](./media/stream-analytics-set-up-alerts/06-stream-analytics-set-up-alerts.png)  

您可以為您的警示規則命名，並選擇將會顯示在電子郵件通知的描述。

當您選取 [計量] 時，您將會為計量選擇條件和閾值標準。

  ![Azure 入口網站選取度量](./media/stream-analytics-set-up-alerts/07-stream-analytics-set-up-alerts.png)  

如需有關在 Azure 入口網站中設定警示的更多詳細資料，請參閱[接收警示通知](../monitoring-and-diagnostics/insights-receive-alert-notifications.md)。  

## <a name="get-help"></a>取得說明
如需進一步的協助，請參閱我們的 [Azure Stream Analytics 論壇](https://social.msdn.microsoft.com/Forums/en-US/home?forum=AzureStreamAnalytics)

## <a name="next-steps"></a>後續步驟
* [Azure Stream Analytics 介紹](stream-analytics-introduction.md)
* [開始使用 Azure Stream Analytics](stream-analytics-get-started.md)
* [調整 Azure Stream Analytics 工作](stream-analytics-scale-jobs.md)
* [Azure Stream Analytics 查詢語言參考](https://msdn.microsoft.com/library/azure/dn834998.aspx)
* [Azure 串流分析管理 REST API 參考](https://msdn.microsoft.com/library/azure/dn835031.aspx)




<!--HONumber=Nov16_HO3-->


