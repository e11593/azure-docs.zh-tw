---
title: "使用 Azure IoT 中樞從裝置上傳檔案 | Microsoft Docs"
description: "如何使用適用於 .NET 的 Azure IoT 裝置 SDK 將檔案從裝置上傳至雲端。 上傳的檔案會儲存在 Azure 儲存體 blob 容器中。"
services: iot-hub
documentationcenter: .net
author: fsautomata
manager: timlt
editor: 
ms.assetid: 4759d229-f856-4526-abda-414f8b00a56d
ms.service: iot-hub
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 11/16/2016
ms.author: elioda
translationtype: Human Translation
ms.sourcegitcommit: a243e4f64b6cd0bf7b0776e938150a352d424ad1
ms.openlocfilehash: ddc181bde6a154cb3fb35254da6b76a91450ba6b


---
# <a name="upload-files-from-devices-to-the-cloud-with-iot-hub"></a>使用 IoT 中心從裝置將檔案上傳至雲端
## <a name="introduction"></a>簡介
Azure IoT 中樞是一項完全受管理的服務，可在數百萬個裝置和一個解決方案後端之間啟用可靠且安全的雙向通訊。 先前的教學課程 ([開始使用 IoT 中樞]和[使用 IoT 中樞傳送雲端到裝置訊息]) 說明了 IoT 中樞基本的裝置到雲端和雲端到裝置訊息功能，而[處理裝置到雲端訊息]教學課程會討論如何在 Azure Blob 儲存體中可靠地儲存裝置到雲端訊息的方式。 不過，在某些情況下，您無法輕易地將裝置傳送的資料對應到 IoT 中樞接受且相對較小的裝置到雲端訊息。 範例涵蓋包含圖片、影片、高頻率震動資料範例，或是包含部分前置處理資料形式的大型檔案。 這些檔案通常會使用工具 (例如 [Azure Data Factory] 或 [Hadoop] 堆疊) 在雲端中進行批次處理。 若偏好從裝置上傳檔案來傳送事件，您仍然可以使用 IoT 中樞安全性與可靠性功能。

本教學課程是以 [使用 IoT 中樞傳送雲端到裝置訊息] 教學課程中的程式碼來建置，示範如何使用 IoT 中樞的檔案上傳功能。 這會說明如何：

- 安全地將 Azure Blob URI 提供給裝置，以便上傳檔案。
- 您可以使用 IoT 中樞檔案上傳通知來觸發在您的應用程式後端中處理此檔案。

在本教學課程結尾處，您會執行兩個 .NET 主控台應用程式：

* **SimulatedDevice**，此為[使用 IoT 中樞傳送雲端到裝置訊息]教學課程中建立的應用程式修改版本。 此應用程式可以使用 IoT 中樞提供的 SAS URI，將檔案上傳到儲存體。
* **ReadFileUploadNotification**，它會接收來自 IoT 中樞的檔案上傳通知。

> [!NOTE]
> IoT 中樞透過 Azure IoT 裝置 SDK 來支援許多裝置平台和語言 (包括 C、Java 及 Javascript)。 如需如何將您的裝置連接到 Azure IoT 中樞的逐步指示，請參閱 [Azure IoT 開發人員中心]。
> 
> 

若要完成此教學課程，您需要下列項目：

* Microsoft Visual Studio 2015，
* 使用中的 Azure 帳戶。 (如果您沒有帳戶，只需要幾分鐘的時間就可以建立[免費帳戶][lnk-free-trial]。)

## <a name="associate-an-azure-storage-account-to-iot-hub"></a>讓 Azure 儲存體帳戶與 IoT 中樞產生關聯
由於模擬裝置應用程式會將檔案上傳至 Blob，因此您必須擁有和 IoT 中樞關聯的 [Azure 儲存體]帳戶。 當您在建立 Azure 儲存體帳戶與 IoT 中樞的關聯時，IoT 中樞會產生可供裝置安全地將檔案上傳至 Blob 容器的 SAS URI。 IoT 中樞服務和裝置 SDK 會協調產生 SAS URI 的程序，並使其可供裝置用來上傳檔案。

依照[使用 Azure 入口網站設定檔案上傳][lnk-configure-upload]中的指示，讓 Azure 儲存體帳戶與 IoT 中樞產生關聯。

## <a name="upload-a-file-from-a-simulated-device-app"></a>從模擬裝置應用程式上傳檔案
在本節中，您將修改在[使用 IoT 中樞傳送雲端到裝置訊息]中建立的模擬裝置應用程式，以接收來自 IoT 中樞的雲端到裝置訊息。

1. 在 Visual Studio 中，以滑鼠右鍵按一下 **SimulatedDevice** 專案，按一下 [新增]，然後按一下 [現有項目]。 瀏覽至影像檔並將它包含在您的專案中。 本教學課程假設影像名稱為 `image.jpg`。
2. 以滑鼠右鍵按一下影像，然後按一下 [內容] 。 確定 [複製到輸出目錄] 是設為 [一律複製]。
   
    ![][1]
3. 在 **Program.cs** 檔的頂端，新增下列陳述式：
   
        using System.IO;
4. 將下列方法加入至 **Program** 類別：
   
        private static async void SendToBlobAsync()
        {
            string fileName = "image.jpg";
            Console.WriteLine("Uploading file: {0}", fileName);
            var watch = System.Diagnostics.Stopwatch.StartNew();
   
            using (var sourceData = new FileStream(@"image.jpg", FileMode.Open))
            {
                await deviceClient.UploadToBlobAsync(fileName, sourceData);
            }
   
            watch.Stop();
            Console.WriteLine("Time to upload file: {0}ms\n", watch.ElapsedMilliseconds);
        }
   
    `UploadToBlobAsync` 方法會取得要上傳之檔案的檔案名稱與資料流來源，然後處理上傳至儲存體的工作。 主控台應用程式會顯示上傳檔案所花費的時間。
5. 將下列方法新增到 **Main** 方法中緊接在 `Console.ReadLine()` 行前面：
   
        SendToBlobAsync();

> [!NOTE]
> 為了簡單起見，本教學課程不會實作任何重試原則。 在生產環境程式碼中，您應該如 MSDN 文章 [暫時性錯誤處理]所建議，實作重試原則 (例如指數型輪詢)。
> 
> 

## <a name="receive-a-file-upload-notification"></a>接收檔案上傳通知
在本節中，您將撰寫 .NET 主控台應用程式，以接收來自 IoT 中樞的檔案上傳通知訊息。

1. 在目前的 Visual Studio 方案中，使用 [主控台應用程式] 專案範本來建立 Visual C# Windows 專案。 將專案命名為 **ReadFileUploadNotification**。
   
    ![Visual Studio 中的新專案][2]
2. 在 [方案總管] 中，以滑鼠右鍵按一下 **ReadFileUploadNotification** 專案，然後按一下 [管理 NuGet 封裝]。
   
    此動作會顯示 [管理 NuGet 套件] 視窗。
3. 搜尋 `Microsoft.Azure.Devices`，然後按一下 [ **安裝**] 並接受使用條款。 
   
    此動作會下載和安裝 [Azure IoT 服務 SDK NuGet 套件]，並在 **ReadFileUploadNotification** 專案中新增此套件的參考。

4. 在 **Program.cs** 檔的頂端，新增下列陳述式：
   
        using Microsoft.Azure.Devices;
5. 將下列欄位新增到 **Program** 類別。 將預留位置的值從 IoT 中樞連接字串替換成[開始使用 IoT 中樞]：
   
        static ServiceClient serviceClient;
        static string connectionString = "{iot hub connection string}";
6. 將下列方法加入至 **Program** 類別：
   
        private async static Task ReceiveFileUploadNotificationAsync()
        {
            var notificationReceiver = serviceClient.GetFileNotificationReceiver();
   
            Console.WriteLine("\nReceiving file upload notification from service");
            while (true)
            {
                var fileUploadNotification = await notificationReceiver.ReceiveAsync();
                if (fileUploadNotification == null) continue;
   
                Console.ForegroundColor = ConsoleColor.Yellow;
                Console.WriteLine("Received file upload noticiation: {0}", string.Join(", ", fileUploadNotification.BlobName));
                Console.ResetColor();
   
                await notificationReceiver.CompleteAsync(fileUploadNotification);
            }
        }
   
    請注意，這裡的接收模式，與用來從裝置應用程式接收雲端到裝置訊息的模式相同。
7. 最後，將下列幾行加入至 **Main** 方法：
   
        Console.WriteLine("Receive file upload notifications\n");
        serviceClient = ServiceClient.CreateFromConnectionString(connectionString);
        ReceiveFileUploadNotificationAsync().Wait();
        Console.ReadLine();

## <a name="run-the-applications"></a>執行應用程式
現在您已經準備好執行應用程式。

1. 在 Visual Studio 中，在您的方案上按一下滑鼠右鍵，然後選取 [設定啟始專案]。 選取 [多個啟始專案]，然後針對 **ReadFileUploadNotification** 和 **SimulatedDevice** 選取 [啟動]。
2. 按 **F5**。 這兩個應用程式應該都會啟動。 您應該會在其中一個主控台應用程式中看到上傳已完成，以及另一個主控台應用程式所收到的上傳通知訊息。 您可以使用 [Azure 入口網站]或 Visual Studio 伺服器總管，在您的 Azure 儲存體帳戶中檢查上傳的檔案是否存在。
   
   ![][50]

## <a name="next-steps"></a>後續步驟
在本教學課程中，您已學到如何運用 IoT 中樞的檔案上傳功能來簡化從裝置上傳檔案。 您可以利用下列文章繼續探索 IoT 中樞功能和案例：

* [以程式設計方式建立 IoT 中樞][lnk-create-hub]
* [C SDK 簡介][lnk-c-sdk]
* [Azure IoT SDK][lnk-sdks]

若要進一步探索 IoT 中樞的功能，請參閱︰

* [使用 IoT 閘道 SDK 來模擬裝置][lnk-gateway]

<!-- Images. -->

[50]: ./media/iot-hub-csharp-csharp-file-upload/run-apps1.png
[1]: ./media/iot-hub-csharp-csharp-file-upload/image-properties.png
[2]: ./media/iot-hub-csharp-csharp-file-upload/create-identity-csharp1.png

<!-- Links -->

[Azure 入口網站]: https://portal.azure.com/

[Azure Data Factory]: https://azure.microsoft.com/documentation/services/data-factory/
[Hadoop]: https://azure.microsoft.com/documentation/services/hdinsight/

[使用 IoT 中樞傳送雲端到裝置訊息]: iot-hub-csharp-csharp-c2d.md
[處理裝置到雲端訊息]: iot-hub-csharp-csharp-process-d2c.md
[開始使用 IoT 中樞]: iot-hub-csharp-csharp-getstarted.md
[Azure IoT 開發人員中心]: http://www.azure.com/develop/iot

[暫時性錯誤處理]: https://msdn.microsoft.com/library/hh680901(v=pandp.50).aspx
[Azure 儲存體]: ../storage/storage-create-storage-account.md#create-a-storage-account
[lnk-configure-upload]: iot-hub-configure-file-upload.md
[Azure IoT 服務 SDK NuGet 套件]: https://www.nuget.org/packages/Microsoft.Azure.Devices/
[lnk-free-trial]: http://azure.microsoft.com/pricing/free-trial/

[lnk-create-hub]: iot-hub-rm-template-powershell.md
[lnk-c-sdk]: iot-hub-device-sdk-c-intro.md
[lnk-sdks]: iot-hub-devguide-sdks.md

[lnk-gateway]: iot-hub-linux-gateway-sdk-simulated-device.md





<!--HONumber=Dec16_HO1-->


