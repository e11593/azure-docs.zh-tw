---
title: "搭配進階編碼器使用多個輸入檔案和元件屬性 | Microsoft Docs"
description: "本主題說明如何使用 setRuntimeProperties 來使用多個輸入檔案，並將自訂資料傳遞給「媒體編碼器高階工作流程」媒體處理器。"
services: media-services
documentationcenter: 
author: xpouyat
manager: erikre
editor: 
ms.assetid: 7fb35bdd-9891-4401-a65b-ef3cc8190e8a
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/10/2016
ms.author: xpouyat;anilmur;juliako
translationtype: Human Translation
ms.sourcegitcommit: f8b0917b6eb0295641360c4e0a80e81100809f6e
ms.openlocfilehash: 1d7f5aeb5fcdb6b80289bbd77c12957148d03732


---
# <a name="using-multiple-input-files-and-component-properties-with-premium-encoder"></a>搭配進階編碼器使用多個輸入檔案和元件屬性
## <a name="overview"></a>概觀
在某些情況下，您可能需要在使用「媒體編碼器高階工作流程」  媒體處理器提交工作時，自訂元件屬性、指定剪輯清單 XML 內容，或傳送多個輸入檔案。 部分範例如下：

* 在每個輸入視訊的執行階段在視訊上覆疊文字並設定文字值 (例如，目前的日期)。
* 自訂剪輯清單 XML (以指定一或多個包含或不含修剪的來源檔案等)。
* 視訊編碼時在輸入視訊上覆疊標誌影像。

若要在建立工作或傳送多個輸入檔案時，讓**「媒體編碼器高階工作流程」**知道您要變更工作流程中的某些屬性，您必須使用包含 **setRuntimeProperties** 及/或 **transcodeSource** 的組態字串。 本主題說明如何使用它們。

## <a name="configuration-string-syntax"></a>組態字串語法
在編碼工作中設定的組態字串會使用類似下面的 XML 文件︰

    <?xml version="1.0" encoding="utf-8"?>
    <transcodeRequest>
      <transcodeSource>
      </transcodeSource>
      <setRuntimeProperties>
        <property propertyPath="Media File Input/filename" value="MyInputVideo.mp4" />
      </setRuntimeProperties>
    </transcodeRequest>


以下 C# 程式碼從檔案讀取 XML 設定，並以作業的形式傳遞給工作︰

    XDocument configurationXml = XDocument.Load(xmlFileName);
    IJob job = _context.Jobs.CreateWithSingleTask(
                                                  "Media Encoder Premium Workflow",
                                                  configurationXml.ToString(),
                                                  myAsset,
                                                  "Output asset",
                                                  AssetCreationOptions.None);


## <a name="customizing-component-properties"></a>自訂元件屬性
### <a name="property-with-a-simple-value"></a>具有簡單值的屬性
在某些情況下，適合在「媒體編碼器高階工作流程」將要執行的工作流程檔案中一起自訂元件屬性。

假設您設計了一個在視訊上覆疊文字的工作流程，要在執行階段設定文字 (例如，目前的日期)。 您可以從編碼工作傳送要設定為覆疊元件文字屬性新值的文字，執行此動作。 您可以使用這項機制來變更工作流程中元件的其他屬性 (例如覆疊的位置或色彩、AVC 編碼器的位元速率等等)。

**setRuntimeProperties** 可用來覆寫工作流程元件中的屬性。

範例：

    <?xml version="1.0" encoding="utf-8"?>
      <transcodeRequest>
        <setRuntimeProperties>
          <property propertyPath="Media File Input/filename" value="MyInputVideo.mp4" />
          <property propertyPath="/primarySourceFile" value="MyInputVideo.mp4" />
          <property propertyPath="Optional Overlay/Overlay/filename" value="MyLogo.png"/>
          <property propertyPath="Optional Text Overlay/Text To Image Converter/text" value="Today is Friday the 13th of May, 2016"/>
      </setRuntimeProperties>
    </transcodeRequest>


### <a name="property-with-an-xml-value"></a>具有 XML 值的屬性
若要設定預期會有 XML 值的屬性，請使用 `<![CDATA[ and ]]>`進行封裝。

範例：

    <?xml version="1.0" encoding="utf-8"?>
      <transcodeRequest>
        <setRuntimeProperties>
          <property propertyPath="/primarySourceFile" value="start.mxf" />
          <property propertyPath="/inactiveTimeout" value="65" />
          <property propertyPath="clipListXml" value="xxx">
          <extendedValue><![CDATA[<clipList>
            <clip>
              <videoSource>
                <mediaFile>
                  <file>start.mxf</file>
                </mediaFile>
              </videoSource>
              <audioSource>
                <mediaFile>
                  <file>start.mxf</file>
                </mediaFile>
              </audioSource>
            </clip>
            <primaryClipIndex>0</primaryClipIndex>
            </clipList>]]>
          </extendedValue>
          </property>
          <property propertyPath="Media File Input Logo/filename" value="logo.png" />
        </setRuntimeProperties>
      </transcodeRequest>

> [!NOTE]
> 在 `<![CDATA[` 後面請務必不要放入歸位字元。
>
>

### <a name="propertypath-value"></a>propertyPath 值
在上一個範例中，propertyPath 是 "/Media File Input/filename" 或 "/inactiveTimeout" 或 "clipListXml"。
這通常是元件的名稱，然後是屬性的名稱。 路徑可以有更多或更少層級，例如 "/primarySourceFile" (因為屬性是在工作流程的根目錄)，或 "/Video Processing/Graphic Overlay/Opacity" (因為覆疊是在群組中)。    

若要檢查路徑和屬性名稱，請使用緊鄰每個屬性的動作按鈕。 您可以按一下這個動作按鈕，然後選取 [編輯] 。 這會對您顯示屬性的實際名稱，並在其正上方顯示命名空間。

![動作/編輯](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture6_actionedit.png)

![屬性](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture7_viewproperty.png)

## <a name="multiple-input-files"></a>多個輸入檔案
提交給「媒體編碼器高階工作流程」  的每個工作都需要兩個資產︰

* 第一個是「工作流程資產」  ，其中包含工作流程檔案。 您可以使用 [工作流程設計工具](media-services-workflow-designer.md)設計工作流程檔案。
* 第二個是「媒體資產」  ，其中包含您想要編碼的媒體檔案。

傳送多個媒體檔案給「媒體編碼器高階工作流程」  編碼器時有以下限制︰

* 所有媒體檔案必須位於同一個「媒體資產」 。 不支援使用多個媒體資產。
* 您必須在此媒體資產中設定主要檔案 (在理想情況下，這是要求編碼器處理的主要視訊檔案)。
* 必須將包含 **setRuntimeProperties** 及/或 **transcodeSource** 元素的組態資料傳遞至處理器。
  * **setRuntimeProperties** 可用來覆寫工作流程元件中的檔案名稱屬性或其他屬性。
  * **transcodeSource** 可用來指定剪輯清單 XML 內容。

工作流程中的連接：

* 如果您使用一或多個媒體檔案輸入元件，並計劃使用 **setRuntimeProperties** 來指定檔案名稱，則請勿將主要檔案元件接點連接到這些元件。 確認主要檔案物件和媒體檔案輸入之間沒有連接。
* 如果您想要使用剪輯清單 XML 和一個媒體來源元件，則您可以將兩者連接在一起。

![主要來源檔案未連接至媒體檔案輸入](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture0_nopin.png)

如果您使用 setRuntimeProperties 來設定檔案名稱屬性，則主要檔案與媒體檔案輸入元件之間不會有連接。

![剪輯清單 XML 連接至剪輯清單來源](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture1_pincliplist.png)

您可以將剪輯清單 XML 連接到媒體來源並使用 transcodeSource。

### <a name="clip-list-xml-customization"></a>剪輯清單 XML 自訂
您可以在組態字串 XML 中使用 **transcodeSource** ，以在執行階段於工作流程中指定剪輯清單 XML。 這需要剪輯清單 XML 接點才能連接到工作流程中的媒體來源元件。

    <?xml version="1.0" encoding="utf-16"?>
      <transcodeRequest>
        <transcodeSource>
          <clipList>
            <clip>
              <videoSource>
                <mediaFile>
                  <file>video-part1.mp4</file>
                </mediaFile>
              </videoSource>
              <audioSource>
                <mediaFile>
                  <file>video-part1.mp4</file>
                </mediaFile>
              </audioSource>
            </clip>
            <primaryClipIndex>0</primaryClipIndex>
          </clipList>
        </transcodeSource>
        <setRuntimeProperties>
          <property propertyPath="Media File Input Logo/filename" value="logo.png" />
        </setRuntimeProperties>
      </transcodeRequest>

如果您想要指定 /primarySourceFile 使用此屬性，以使用 'Expressions' 來命名輸出檔案，我們建議在 /primarySourceFile 屬性「之後」  將剪輯清單 XML 傳遞為屬性，以避免剪輯清單遭到 /primarySourceFile 設定覆寫。

    <?xml version="1.0" encoding="utf-8"?>
      <transcodeRequest>
        <setRuntimeProperties>
          <property propertyPath="/primarySourceFile" value="c:\temp\start.mxf" />
          <property propertyPath="/inactiveTimeout" value="65" />
          <property propertyPath="clipListXml" value="xxx">
          <extendedValue><![CDATA[<clipList>
            <clip>
              <videoSource>
                <mediaFile>
                  <file>c:\temp\start.mxf</file>
                </mediaFile>
              </videoSource>
              <audioSource>
                <mediaFile>
                  <file>c:\temp\start.mxf</file>
                </mediaFile>
              </audioSource>
            </clip>
            <primaryClipIndex>0</primaryClipIndex>
            </clipList>]]>
          </extendedValue>
          </property>
          <property propertyPath="Media File Input Logo/filename" value="logo.png" />
        </setRuntimeProperties>
      </transcodeRequest>

具有其他框架精確修剪︰

    <?xml version="1.0" encoding="utf-8"?>
      <transcodeRequest>
        <setRuntimeProperties>
          <property propertyPath="/primarySourceFile" value="start.mxf" />
          <property propertyPath="/inactiveTimeout" value="65" />
          <property propertyPath="clipListXml" value="xxx">
          <extendedValue><![CDATA[<clipList>
            <clip>
              <videoSource>
                <trim>
                  <inPoint fps="25">00:00:05:24</inPoint>
                  <outPoint fps="25">00:00:10:24</outPoint>
                </trim>
                <mediaFile>
                  <file>start.mxf</file>
                </mediaFile>
              </videoSource>
              <audioSource>
               <trim>
                  <inPoint fps="25">00:00:05:24</inPoint>
                  <outPoint fps="25">00:00:10:24</outPoint>
                </trim>
                <mediaFile>
                  <file>start.mxf</file>
                </mediaFile>
              </audioSource>
            </clip>
            <primaryClipIndex>0</primaryClipIndex>
            </clipList>]]>
          </extendedValue>
          </property>
          <property propertyPath="Media File Input Logo/filename" value="logo.png" />
        </setRuntimeProperties>
      </transcodeRequest>


## <a name="example"></a>範例
請設想視訊編碼時要在輸入視訊上覆疊標誌影像的範例。 在此範例中，輸入視訊名為 "MyInputVideo.mp4"，標誌名為 "MyLogo.png"。 您應執行下列步驟：

* 使用工作流程檔案建立工作流程資產 (參閱下列範例)。
* 建立媒體資產，其中包含兩個檔案︰MyInputVideo.mp4 (主要檔案) 和 MyLogo.png。
* 使用上述輸入資產將工作傳送至「媒體編碼器高階工作流程」媒體處理器，並指定下列組態字串。

組態:

    <?xml version="1.0" encoding="utf-8"?>
      <transcodeRequest>
        <setRuntimeProperties>
          <property propertyPath="Media File Input/filename" value="MyInputVideo.mp4" />
          <property propertyPath="/primarySourceFile" value="MyInputVideo.mp4" />
          <property propertyPath="Media File Input Logo/filename" value="MyLogo.png" />
        </setRuntimeProperties>
      </transcodeRequest>


在上述範例中，視訊檔案的名稱會傳送至媒體檔案輸入元件及 primarySourceFile 屬性。 標誌檔案的名稱會傳送至另一個連接至圖形覆疊元件的媒體檔案輸入。

> [!NOTE]
> 視訊檔案名稱會傳送至 primarySourceFile 屬性。 其原因是要在工作流程中使用這個屬性，以便 (舉例來說) 使用運算式建置正確的輸出檔案名稱。
>
>

### <a name="step-by-step-workflow-creation-that-overlays-a-logo-on-top-of-the-video"></a>在視訊頂端覆疊標誌的工作流程建立逐步說明
以下步驟會建立工作流程來將兩個檔案做為輸入︰視訊和影像。 它會在視訊頂端覆疊影像。

開啟**工作流程設計工具**，然後選取 [檔案] > [新增工作區] > [轉碼藍圖]。

新的工作流程會顯示三個元素︰

* 主要來源檔案
* 剪輯清單 XML
* 輸出檔案/資產  

![新編碼工作流程](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture9_empty.png)

*新編碼工作流程*

為了接受輸入媒體檔案，請從加入媒體檔案輸入元件開始。 若要將元件加入至工作流程，請在 [儲存機制] 搜尋方塊中尋找它，然後將所需的項目拖曳至設計工具窗格。

接著，新增要用於設計工作流程的視訊檔案。 若要這樣做，請按一下工作流程設計工具的背景窗格，並在右手邊的屬性窗格中尋找 [主要來源檔案] 屬性。 按一下資料夾圖示，然後選取適當的視訊檔案。

![主要檔案來源](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture10_primaryfile.png)

*主要檔案來源*

接下來，在媒體檔案輸入元件中指定視訊檔案。   

![媒體檔案輸入來源](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture11_mediafileinput.png)

*媒體檔案輸入來源*

完成此動作之後，媒體檔案輸入元件會檢查檔案，並填入其輸出接點，以反映它檢查的檔案。

下一個步驟是新增「視訊資料類型更新器」以對 Rec.709 指定色彩空間。 新增「資料配置/配置類型 = 可設定平面」的「視訊格式轉換器」。 這會將視訊串流轉換成可以當做覆疊元件來源的格式。

![視訊資料類型更新器和格式轉換器](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture12_formatconverter.png)

*視訊資料類型更新器和格式轉換器*

![配置類型 = 可設定平面](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture12_formatconverter2.png)

*Layout type is Configurable Planar*

接下來，新增視訊覆疊元件，並將 (未壓縮的) 視訊接點連接至媒體檔案輸入的 (未壓縮的) 視訊接點。

新增另一個媒體檔案輸入 (以載入標誌檔案)、按一下這個元件、將它重新命名為「媒體檔案輸入標誌」，然後在檔案屬性中選取影像 (例如 .png 檔案)。 將未壓縮的影像接點連接到覆疊的未壓縮影像接點。

![覆疊元件和影像檔案來源](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture13_overlay.png)

*覆疊元件和影像檔案來源*

如果您想要修改視訊的標誌位置 (例如，您要將它放在距離視訊左上角 10% 之處)，請清除 [手動輸入] 核取方塊。 由於您使用媒體檔案輸入來提供標誌檔案給覆疊元件，您可以執行此動作。

![覆疊位置](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture14_overlay_position.png)

*覆疊位置*

若要將視訊串流編碼成 H.264，請將 AVC 視訊編碼器和 AAC 編碼器元件加入至設計工具介面。 連接接點。
設定 AAC 編碼器，然後選取「音訊格式轉換/預設值 : 2.0 (L, R)」。

![音訊和視訊編碼器](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture15_encoders.png)

*音訊和視訊編碼器*

現在，新增 **ISO MPEG-4 多工器**和**檔案輸出**元件並連接接點，如下所示。

![MP4 多工器和輸出檔案](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture16_mp4output.png)

*MP4 多工器和輸出檔案*

您必須設定輸出檔案的名稱。 按一下 [檔案輸出]  元件，然後編輯檔案的運算式︰

    ${ROOT_outputWriteDirectory}\${ROOT_sourceFileBaseName}_withoverlay.mp4

![檔案輸出名稱](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture17_filenameoutput.png)

*檔案輸出名稱*

您可以在本機執行工作流程，以檢查它是否正確運作。

完成之後，您就可以在 Azure 媒體服務中執行它。

首先，為 Azure 媒體服務中的資產準備兩個檔案︰視訊檔案和標誌。 您可以使用 .NET 或 REST API 進行。 您也可以使用 Azure 入口網站或 [Azure 媒體服務總管](https://github.com/Azure/Azure-Media-Services-Explorer) (AMSE) 來進行。

本教學課程示範如何使用 AMSE 管理資產。 有兩種方式可將檔案新增至資產：

* 建立本機資料夾、在其中複製這兩個檔案，然後將資料夾拖放至 [資產]  索引標籤。
* 上傳視訊檔案做為資產、顯示資產資訊、移至檔案索引標籤，並上傳其他檔案 (標誌)。

> [!NOTE]
> 請務必在資產中設定主要檔案 (主要的視訊檔案)。
>
>

![AMSE 中的資產檔案](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture18_assetinamse.png)

*AMSE 中的資產檔案*

選取資產，並選擇使用進階編碼器將其編碼。 上傳工作流程，並加以選取。

按一下按鈕以將資料傳遞給處理器，然後新增下列 XML 來設定執行階段屬性︰

![AMSE 中的進階編碼器](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture19_amsepremium.png)

*AMSE 中的進階編碼器*

然後，貼上下列 XML 資料。 您必須指定媒體檔案輸入和 primarySourceFile 的視訊檔案名稱。 也請指定標誌的檔案名稱。

    <?xml version="1.0" encoding="utf-16"?>
      <transcodeRequest>
        <setRuntimeProperties>
          <property propertyPath="Media File Input/filename" value="Microsoft_HoloLens_Possibilities_816p24.mp4" />
          <property propertyPath="/primarySourceFile" value="Microsoft_HoloLens_Possibilities_816p24.mp4" />
          <property propertyPath="Media File Input Logo/filename" value="logo.png" />
        </setRuntimeProperties>
      </transcodeRequest>

![setRuntimeProperties](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture20_amsexmldata.png)

*setRuntimeProperties*

如果您使用 .NET SDK 來建立和執行工作，必須將此 XML 資料傳遞為組態字串。

    public ITask AddNew(string taskName, IMediaProcessor mediaProcessor, string configuration, TaskOptions options);

作業完成之後，輸出資產中的 MP4 檔案就會顯示覆疊！

![視訊上的覆疊](./media/media-services-media-encoder-premium-workflow-multiplefilesinput/capture21_resultoverlay.png)

*視訊上的覆疊*

您可以從 [GitHub](https://github.com/Azure/azure-media-services-samples/tree/master/Encoding%20Presets/VoD/MediaEncoderPremiumWorkfows/)下載範例工作流程。

## <a name="see-also"></a>另請參閱
* [介紹 Azure 媒體服務中的 Premium 編碼](http://azure.microsoft.com/blog/2015/03/05/introducing-premium-encoding-in-azure-media-services)
* [How to use Premium Encoding in Azure Media Services (如何使用 Azure 媒體服務中的 Premium 編碼)](http://azure.microsoft.com/blog/2015/03/06/how-to-use-premium-encoding-in-azure-media-services)
* [使用 Azure 媒體服務編碼隨選內容](media-services-encode-asset.md#media-encoder-premium-workflow)
* [媒體編碼器高階工作流程格式和轉碼器](media-services-premium-workflow-encoder-formats.md)
* [範例工作流程檔案](https://github.com/AzureMediaServicesSamples/Encoding-Presets/tree/master/VoD/MediaEncoderPremiumWorkfows)
* [Azure 媒體服務總管工具](http://aka.ms/amse)

## <a name="media-services-learning-paths"></a>媒體服務學習路徑
[!INCLUDE [media-services-learning-paths-include](../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>提供意見反應
[!INCLUDE [media-services-user-voice-include](../../includes/media-services-user-voice-include.md)]



<!--HONumber=Nov16_HO3-->


