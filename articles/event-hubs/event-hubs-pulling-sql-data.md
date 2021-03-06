---
title: "將 SQL 資料提取到 Azure 事件中樞 | Microsoft Docs"
description: "從 SQL 範例匯入之事件中樞的概觀"
services: event-hubs
documentationcenter: na
author: spyrossak
manager: timlt
editor: 
ms.assetid: 3dbc21f2-5e97-463f-85c8-78450369ca48
ms.service: event-hubs
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 08/25/2016
ms.author: spyros;sethm
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 174630ba181a32dc6af30a1a89daac68c4050b74


---
# <a name="pulling-data-from-sql-into-an-azure-event-hub"></a>將資料從 SQL 提取到 Azure 事件中樞
一個針對處理即時資料的典型應用程式架構，必須先將該資料推送到 Azure 事件中樞。 它可能是 IoT 案例 (例如監視高速公路不同區段的車流量)，或是遊戲案列 (監視大量狂熱競賽者的動作)，或是企業案例 (例如監視建築物的使用情形)。 在這些情況中，資料產生者通常是外部物件，產生您需要收集、分析、儲存及處理的時間序列資料，而且您可能已經入大量心力建置這些流程的基礎結構。 如果您想要從類似資料庫，而非從串流資料來源帶入資料，並搭配其他串流資料一起使用，該如何進行？ 請考慮您想要使用 Azure 串流分析、遠端資料總管 (RDX) 或其他工具，來分析及處理來自 Microsoft Dynamics AX 或是自訂設施管理系統之緩慢變更資料的情況？ 為了解決這個問題，我們已撰寫開放原始碼的小型雲端範例，可以讓您加以修改並部署，以從 SQL 資料表提取資料，並將資料推送至 Azure 事件中樞，以做為您下游分析應用程式中的輸入使用。 請務必了解這是個罕見的案例，也和您平常使用事件中樞的方式相反。 不過，如果您發現自己處於必須做出這類動作的情況之下，您可以在 Azure 範例庫中找到該程式碼，位置在 [這裡](https://azure.microsoft.com/documentation/samples/event-hubs-dotnet-import-from-sql/)。  

請注意，此範例中的程式碼只是個範例。 該範例 **不是** 實際執行應用程式，也不適用於這類環境 - 它是讓開發人員留意的 DIY 範例而已。 在決定端對端架構之前，您必須檢閱各種安全性、效能、功能及成本因素。

## <a name="application-structure"></a>應用程式結構
應用程式是以 C# 撰寫，而範例中的 readme.md 檔案則包含您需要修改、建置及發佈應用程式所需的所有資訊。 下列各節提供應用程式用途的高階概觀。

我們一開始假設您擁有存取 SQL Azure 資料表的權限。 您也將需要設定 Azure 事件中樞，並且知道存取它所需的連接字串。

當 SqlToEventHub 解決方案啟動時，會讀取組態檔 (App.config) 來取得一些內容，如同在 readme.md 檔案中所述。 這些內容皆很容易理解，如資料表的名稱等等，因此不需要在此重新敘述它們的解釋。 

應用程式在讀取組態檔之後便會進入迴圈、讀取 SQL 資料表、將記錄推送到事件中樞，然後等候使用者定義的睡眠間隔結束，並將上述步驟全部再做一遍。 有幾件事值得一提：

1. 應用程式是以 SQL 資料表正由某些外部程序進行更新，而且您想要將所有更新 (且僅限更新) 傳送至事件中樞這個假設為基礎。
2. SQL 資料表必須有一個擁有唯一且不斷增加之數字的欄位，例如一個記錄號碼。 這可以單純是個稱為「Id」的欄位，或是任何其他會隨著資料庫新增記錄而增加的項目，例如「Creation_time」或「Sequence_number」。 應用程式會在每次反覆運算時記錄並儲存該欄位的值。 應用程式於後續每次通過迴圈時，基本上便是查詢資料表，並針對所有該欄位的值超過應用程式上一次通過迴圈時所記錄之值的情況做出記錄。 我們將這個最後的值稱為「位移」。
3. 應用程式在啟動時會建立「TableOffsets」資料表，以儲存位移。 資料表是透過在組態檔中定義的「CreateOffsetTableQuery」查詢所建立。 
4. 有數個搭配位移資料表使用的查詢，在組態檔中被定義為「OffsetQuery」、「UpdateOffsetQuery」及「InsertOffsetQuery」。 您不應該變更這些。
5. 最後，在組態檔中定義的「DataQuery」查詢，便是會執行以從您的 SQL 資料表提取記錄的查詢。 目前基於最佳化的目的，該查詢在每次通過迴圈時，被限制只能提取前 1,000 筆記錄 - 例如，如果在上一次查詢之後有 25,000 筆記錄被新增到資料庫，則可能需要花費一段時間才能執行查詢。 藉由將每次的查詢限制在 1,000 筆記錄，查詢會變得更加快速。 選取前 1,000 筆記錄，便能簡單地將後續的 1,000 筆記錄批次推送到事件中樞。    

## <a name="next-steps"></a>後續步驟
若要部署解決方案，請複製或下載 SqlToEventHub 應用程式、編輯 App.config 檔案、加以建置並發佈。 一旦您發佈該應用程式，便可以在雲端服務底下的 Azure 傳統入口網站中查看它的執行情況，並監視傳入您事件中樞的事件。 請注意，頻率將取決於兩件事：SQL 資料表的更新頻率，以及您在應用程式組態檔中指定的睡眠間隔。




<!--HONumber=Nov16_HO3-->


