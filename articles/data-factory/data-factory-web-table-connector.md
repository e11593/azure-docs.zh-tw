---
title: "從 Web 資料表移動資料 | Microsoft Docs"
description: "了解如何使用 Azure Data Factory，來移動網頁上內部部署資料表的資料。"
services: data-factory
documentationcenter: 
author: linda33wj
manager: jhubbard
editor: monicar
ms.assetid: f54a26a4-baa4-4255-9791-5a8f935898e2
ms.service: data-factory
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 12/12/2016
ms.author: jingwang
translationtype: Human Translation
ms.sourcegitcommit: 7b55f6730c6a2bf8637f312c452fe552f82dbaeb
ms.openlocfilehash: 4adfd82a0dea0aa46607b3cc528c922cd46ab7d5


---
# <a name="move-data-from-a-web-table-source-using-azure-data-factory"></a>使用 Azure Data Factory 來移動 Web 資料表的資料
本文章將概述如何使用 Azure 資料處理站中的複製活動，來把網頁上資料表中的資料複製到另一個資料存放區。 本文是根據 [資料移動活動](data-factory-data-movement-activities.md) 一文，該文呈現使用複製活動移動資料的一般概觀以及支援的資料存放區組合。

Data factory 目前只支援把 Web 資料表的資料移動到其他資料存放區，但不支援把其他資料存放區的資料移動到 Web 資料表。

> [!NOTE]
> 此 Web 連接器目前只支援從 HTML 網頁擷取資料表內容。
>
>

## <a name="sample-copy-data-from-web-table-to-azure-blob"></a>範例：把 Web 資料表的資料複製到 Azure Blob
下列範例顯示：

1. [Web](#web-linked-service-properties)類型的連結服務。
2. [AzureStorage](data-factory-azure-blob-connector.md#azure-storage-linked-service)類型的連結服務。
3. [WebTable](#WebTable-dataset-properties) 類型的輸入[資料集](data-factory-create-datasets.md)。
4. [AzureBlob](data-factory-azure-blob-connector.md#azure-blob-dataset-type-properties) 類型的輸出[資料集](data-factory-create-datasets.md)。
5. 具有使用 [WebSource](#websource-copy-activity-type-properties) 和 [BlobSink](data-factory-azure-blob-connector.md#azure-blob-copy-activity-type-properties) 之複製活動的[管線](data-factory-create-pipelines.md)。

範例會每隔一小時就把 Web 資料表的資料複製到 Azure blob。 範例後面的各節會說明這些範例中使用的 JSON 屬性。

下列範例示範如何把 Web 資料表的資料複製到 Azure Blob。 不過，您可以使用 Azure Data Factory 中的複製活動，把資料直接複製到 [資料移動活動](data-factory-data-movement-activities.md) 一文中所述的任何接收器。

**Web 連結服務** 這個範例會使用有匿名驗證的 Web 連結服務。 請參閱 [Web 連結服務](#web-linked-service-properties) 一節，來了解您可以使用的不同驗證類型。

```JSON
{
    "name": "WebLinkedService",
    "properties":
    {
        "type": "Web",
        "typeProperties":
        {
            "authenticationType": "Anonymous",
            "url" : "https://en.wikipedia.org/wiki/"
        }
    }
}
```

**Azure 儲存體連結服務**

```JSON
{
  "name": "AzureStorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>"
    }
  }
}
```

**WebTable 輸入資料集** 將 **external** 設定為 **true**，等於是通知 Data Factory 服務：這是 Data Factory 外部的資料集而且不是由 Data Factory 中的活動所產生。

> [!NOTE]
> 如需如何取得 HTML 網頁中資料表索引的步驟，請參閱 [取得 HTML 網頁中資料表的索引](#get-index-of-a-table-in-an-html-page) 一節。  
>
>

```JSON
{
    "name": "WebTableInput",
    "properties": {
        "type": "WebTable",
        "linkedServiceName": "WebLinkedService",
        "typeProperties": {
            "index": 1,
            "path": "AFI's_100_Years...100_Movies"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval":  1
        }
    }
}
```


**Azure Blob 輸出資料集**

資料會每小時寫入至新的 Blob (頻率：小時，間隔：1)。

```JSON
{
    "name": "AzureBlobOutput",
    "properties":
    {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties":
        {
            "folderPath": "adfgetstarted/Movies"
        },
        "availability":
        {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```



**具有複製活動的管線**

此管線包含複製活動，該活動已設定為使用上述輸入和輸出資料集並排定為每小時執行。 在管線 JSON 定義中，**source** 類型設為 **WebSource**，而 **sink** 類型設為 **BlobSink**。

如需 WebSource 支援的屬性清單，請參閱 [WebSource 類別屬性](#websource-copy-activity-type-properties) 。

```JSON
{  
    "name":"SamplePipeline",
    "properties":{  
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline with copy activity",
    "activities":[  
      {
        "name": "WebTableToAzureBlob",
        "description": "Copy from a Web table to an Azure blob",
        "type": "Copy",
        "inputs": [
          {
            "name": "WebTableInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureBlobOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "WebSource"
          },
          "sink": {
            "type": "BlobSink"
          }
        },
       "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
      ]
   }
}
```

## <a name="web-linked-service-properties"></a>Web 連結服務屬性
下表提供 Web 連結服務專屬 JSON 元素的說明。

| 屬性 | 說明 | 必要 |
| --- | --- | --- |
| 類型 |類型屬性必須設為： **Web** |是 |
| Url |Web 來源的 URL |是 |
| authenticationType |匿名或基本。 |是 |
| userName |基本驗證用的使用者名稱 |是 (用於基本驗證) |
| password |基本驗證用的密碼 |是 (用於基本驗證) |

### <a name="using-anonymous-authentication"></a>使用匿名驗證

```JSON
{
    "name": "web",
    "properties":
    {
        "type": "Web",
        "typeProperties":
        {
            "authenticationType": "Anonymous",
            "url" : "https://en.wikipedia.org/wiki/"
        }
    }
}
```

### <a name="using-basic-authentication"></a>使用基本驗證

```JSON
{
    "name": "web",
    "properties":
    {
        "type": "Web",
        "typeProperties":
        {
            "authenticationType": "basic",
            "url" : "http://myit.mycompany.com/",
            "userName": "Administrator",
            "password": "password"
        }
    }
}
```

## <a name="webtable-dataset-properties"></a>WebTable 資料集屬性
如需定義資料集的區段和屬性完整清單，請參閱[建立資料集](data-factory-create-datasets.md)一文。 資料集 JSON 的結構、可用性和原則等區段類似於所有的資料集類型 (SQL Azure、Azure Blob、Azure 資料表等)。

每個資料集類型的 **typeProperties** 區段都不同，可提供資料存放區中的資料位置資訊。 **WebTable** 類型資料集的 typeProperties 區段有下列屬性

| 屬性 | 說明 | 必要 |
|:--- |:--- |:--- |
| 類型 |資料集的類型。 必須設定為 **WebTable** |是 |
| 路徑 |包含資料表之資源的相對 URL。 |否。 當路徑未指定時，則只會使用在連結服務定義中指定的 URL。 |
| index |資源中資料表的索引。 如需如何取得 HTML 網頁中資料表索引的步驟，請參閱 [取得 HTML 網頁中資料表的索引](#get-index-of-a-table-in-an-html-page) 一節。 |是 |

**範例：**

```JSON
{
    "name": "WebTableInput",
    "properties": {
        "type": "WebTable",
        "linkedServiceName": "WebLinkedService",
        "typeProperties": {
            "index": 1,
            "path": "AFI's_100_Years...100_Movies"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval":  1
        }
    }
}
```

## <a name="websource---copy-activity-type-properties"></a>WebSource：複製活動類型屬性
如需定義活動的區段和屬性完整清單，請參閱[建立管線](data-factory-create-pipelines.md)一文。 屬性 (例如名稱、描述、輸入和輸出資料表，以及原則) 適用於所有類型的活動。

另一方面，活動的 typeProperties 區段中可用的屬性會隨著每個活動類型而有所不同。 就「複製活動」而言，這些屬性會根據來源和接收器的類型而有所不同。

當複製活動中的來源類型為 **WebSource**，目前並未支援任何其他屬性。

## <a name="get-index-of-a-table-in-an-html-page"></a>取得 HTML 網頁中資料表的索引
1. 啟動 **Excel 2016**，然後切換到 [資料] 索引標籤。  
2. 按一下工具列上的 [開新查詢]、指向 [從其他來源]，然後按一下 [從 Web]。

    ![Power Query 功能表](./media/data-factory-web-table-connector/PowerQuery-Menu.png)
3. 在 [從 Web] 對話方塊中，輸入您要在連結服務 JSON 中使用的 **URL** (例如：https://en.wikipedia.org/wiki)，以及您為資料集指定的路徑 (例如：AFI%27s_100_Years...100_Movies)，然後按一下 [確定]。

    ![[從 Web] 對話方塊](./media/data-factory-web-table-connector/FromWeb-DialogBox.png)

    此範例中使用的 URL：https://en.wikipedia.org/wiki/AFI%27s_100_Years...100_Movies
4. 當您看到 [存取 Web 內容] 對話方塊時，選取右側的 **URL**、**驗證方式**，然後按一下 [連線]。

   ![[存取 Web 內容] 對話方塊](./media/data-factory-web-table-connector/AccessWebContentDialog.png)
5. 按一下樹狀檢視中的某個**資料表**項目來查看資料表內容，然後按一下底部的 [編輯] 按鈕。  

   ![[導覽器] 對話方塊](./media/data-factory-web-table-connector/Navigator-DialogBox.png)
6. 在 [查詢編輯器] 視窗中，按一下工具列上的 [進階編輯器] 按鈕。

    ![[進階編輯器] 按鈕](./media/data-factory-web-table-connector/QueryEditor-AdvancedEditorButton.png)
7. 在 [進階編輯器] 對話方塊中，「Source」旁的數字就是索引。

    ![進階編輯器及索引](./media/data-factory-web-table-connector/AdvancedEditor-Index.png)

如果您使用的是 Excel 2013，請使用 [Microsoft Power Query for Excel](https://www.microsoft.com/download/details.aspx?id=39379) 來取得索引。 如需詳細資訊，請參閱 [連線至網頁](https://support.office.com/article/Connect-to-a-web-page-Power-Query-b2725d67-c9e8-43e6-a590-c0a175bd64d8) 一文。 如果您使用 [Microsoft Power BI for Desktop](https://powerbi.microsoft.com/desktop/)，步驟就很類似。

[!INCLUDE [data-factory-column-mapping](../../includes/data-factory-column-mapping.md)]

[!INCLUDE [data-factory-structure-for-rectangualr-datasets](../../includes/data-factory-structure-for-rectangualr-datasets.md)]

## <a name="performance-and-tuning"></a>效能和微調
請參閱[複製活動的效能及微調指南](data-factory-copy-activity-performance.md)一文，以了解在 Azure Data Factory 中會影響資料移動 (複製活動) 效能的重要因素，以及各種最佳化的方法。



<!--HONumber=Nov16_HO3-->


