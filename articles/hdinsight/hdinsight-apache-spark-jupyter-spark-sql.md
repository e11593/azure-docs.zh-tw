---
title: "在 Azure HDInsight 中建立 Spark 叢集，並使用 Spark SQL 從 Jupyter 執行互動式分析 | Microsoft Docs"
description: "關於如何在 HDInsight 中快速建立 Apache Spark 叢集，然後使用 Spark SQL 從 Jupyter Notebook 執行互動式查詢的逐步指示。"
services: hdinsight
documentationcenter: 
author: nitinme
manager: jhubbard
editor: cgronlun
tags: azure-portal
ms.assetid: 91f41e6a-d463-4eb4-83ef-7bbb1f4556cc
ms.service: hdinsight
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: get-started-article
ms.date: 01/06/2017
ms.author: nitinme
translationtype: Human Translation
ms.sourcegitcommit: 791b6a5a07bb87302cb382290a355c9a14c63ff0
ms.openlocfilehash: cc1d484d40dce0b1c64f2e8cdebb9377a38705cb


---
# <a name="get-started-create-apache-spark-cluster-in-azure-hdinsight-and-run-interactive-queries-using-spark-sql"></a>開始使用：在 Azure HDInsight 中建立 Apache Spark 叢集並使用 Spark SQL 執行互動式查詢
了解在 HDInsight 中建立 [Apache Spark](hdinsight-apache-spark-overview.md) 叢集，然後使用 [Jupyter](https://jupyter.org) Notebook 對 Spark 叢集執行 Spark SQL 互動式查詢。

   ![開始使用 HDInsight 中的 Apache Spark](./media/hdinsight-apache-spark-jupyter-spark-sql/hdispark.getstartedflow.png "開始使用 HDInsight 中的 Apache Spark 教學課程。說明的步驟：建立儲存體帳戶、建立叢集、執行 Spark SQL 陳述式")

[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="prerequisites"></a>必要條件
* **Azure 訂用帳戶**。 開始進行本教學課程之前，您必須擁有 Azure 訂用帳戶。 請參閱[立即建立免費的 Azure 帳戶](https://azure.microsoft.com/free)。
* **安全殼層 (SSH) 用戶端**：Linux、Unix 和 OS X 系統透過 `ssh` 命令提供 SSH 用戶端。 若為 Windows 系統，請參閱[從具有 PuTTY 的 Windows 在 HDInsight 上搭配以 Linux 為基礎的 Hadoop 使用 SSH](hdinsight-hadoop-linux-use-ssh-windows.md)；若為 Linux、Unix 或 OS X，請參閱[從 Linux、Unix 或 OS X 在 HDInsight 上搭配以 Linux 為基礎的 Hadoop 使用 SSH](hdinsight-hadoop-linux-use-ssh-unix.md)

> [!NOTE]
> 本文使用 Azure Resource Manager 範本來建立使用 [Azure 儲存體 Blob 做為叢集儲存體](hdinsight-hadoop-use-blob-storage.md)的 Spark 叢集。 除了使用 Azure 儲存體 Blob 做為預設儲存體外，您也可以建立使用 [Azure Data Lake Store](../data-lake-store/data-lake-store-overview.md) 做為額外儲存體的 Spark 叢集。 如需指示，請參閱 [建立具有 Data Lake Store 的 HDInsight 叢集](../data-lake-store/data-lake-store-hdinsight-hadoop-use-portal.md)。
>
>

### <a name="access-control-requirements"></a>存取控制需求
[!INCLUDE [access-control](../../includes/hdinsight-access-control-requirements.md)]

## <a name="create-spark-cluster"></a>建立 Spark 叢集
在本節中，您會在 HDInsight 中使用 [Azure Resource Manager 範本](https://azure.microsoft.com/resources/templates/101-hdinsight-spark-linux/)建立 Spark 叢集。 如需不同 HDInsight 版本及其 SLA 的相關資訊，請參閱〈 [HDInsight 元件版本設定](hdinsight-component-versioning.md)〉。 如需其他叢集建立方法，請參閱 [建立 HDInsight 叢集](hdinsight-hadoop-provision-linux-clusters.md)。

1. 按一下以下影像，在 Azure 入口網站中開啟範本。         

    <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-hdinsight-spark-linux%2Fazuredeploy.json" target="_blank"><img src="./media/hdinsight-apache-spark-jupyter-spark-sql/deploy-to-azure.png" alt="Deploy to Azure"></a>

2. 輸入下列值：

    ![在 HDInsight 中使用 Azure Resource Manager 範本建立 Spark 叢集](./media/hdinsight-apache-spark-jupyter-spark-sql/create-spark-cluster-in-hdinsight-using-azure-resource-manager-template.png "在 HDInsight 中使用 Azure Resource Manager 範本建立 Spark 叢集")

   * **訂用帳戶**：針對此叢集選取您的 Azure 訂用帳戶。
   * **資源群組**：建立新的資源群組，或選取現有的資源群組。 資源群組用來管理專案的 Azure 資源。
   * **位置**：選取資源群組的位置。  這個位置也用於預設叢集儲存體和 HDInsight 叢集。
   * **ClusterName**：輸入您將建立的 Hadoop 叢集的名稱。
   * 叢集登入名稱和密碼：預設登入名稱是 admin。
   * SSH 使用者名稱和密碼。

   請記下這些值。  稍後在教學課程中需要這些資訊。

3. 選取 [我同意上方所述的條款及條件]，選取 [釘選到儀表板]，然後按一下 [購買]。 您可以看到新的圖格，標題為「提交範本部署的部署」。 大約需要 20 分鐘的時間來建立叢集。

## <a name="run-spark-sql-queries-using-a-jupyter-notebook"></a>使用 Jupyter Notebook 執行 Spark SQL 查詢
在本節中，您會使用 Jupyter Notebook 來針對 Spark 叢集執行 Spark SQL 查詢。 HDInsight Spark 叢集提供兩種核心，可讓您用於 Jupyter Notebook。 它們是：

* **PySpark** (適用於以 Python 撰寫的應用程式)
* **Spark** (適用於以 Scala 撰寫的應用程式)

在本文中，您將使用 PySpark 核心。 在 [使用 Spark HDInsight 叢集之 Jupyter Notebook 上可用的核心](hdinsight-apache-spark-jupyter-notebook-kernels.md#why-should-i-use-the-pyspark-or-spark-kernels) 一文中，您可以詳細閱讀使用 PySpark 核心的優點。 但是，使用 PySpark 核心的幾個主要優點包括：

* 您不需要設定 Spark 和 Hive 的內容。 這些內容會自動為您設定。
* 您可以使用 cell magic (例如 `%%sql`) 直接執行 SQL 或 Hive 查詢，而不需要任何前置的程式碼片段。
* SQL 或 Hive 查詢的輸出會自動視覺化。

### <a name="create-jupyter-notebook-with-pyspark-kernel"></a>使用 PySpark 核心建立 Jupyter Notebook

1. 開啟 [Azure 入口網站](https://portal.azure.com/)。
2. 從左功能表中，按一下 [資源群組]。
3. 按一下您在上一節中建立的資源群組名稱。 如果有太多的資源群組，您可以使用搜尋功能。 您可以在群組、HDInsight 叢集及預設儲存體帳戶中看見兩個資源。
4. 按一下叢集加以開啟。
 
2. 從 [快速連結]，按一下 [叢集儀表板]，然後按一下 [Jupyter Notebook]。 出現提示時，輸入叢集的系統管理員認證。

   ![HDInsight 叢集儀表板](./media/hdinsight-apache-spark-jupyter-spark-sql/hdinsight-azure-portal-cluster-dashboards.png "HDInsight 叢集儀表板")

   > [!NOTE]
   > 您也可以在瀏覽器中開啟下列 URL，來連接到您的叢集的 Jupyter Notebook。 使用您叢集的名稱取代 **CLUSTERNAME** ：
   >
   > `https://CLUSTERNAME.azurehdinsight.net/jupyter`
   >
   >
3. 建立新的 Notebook。 按一下 [新增]，然後按一下 [PySpark]。

   ![建立新的 Jupyter Notebook](./media/hdinsight-apache-spark-jupyter-spark-sql/hdispark.note.jupyter.createnotebook.png "建立新的 Jupyter Notebook")

   新的 Notebook 隨即建立並以 Untitled(Untitled.pynb) 名稱開啟。 

4. 按一下頂端的 Notebook 名稱，然後輸入好記的名稱。

    ![提供 Notebook 的名稱](./media/hdinsight-apache-spark-jupyter-spark-sql/hdispark.note.jupyter.notebook.name.png "提供 Notebook 的名稱")
5. 將以下程式碼貼入空白儲存格，然後按下 **SHIFT + ENTER** 鍵以執行此程式碼。 此程式碼會匯入此案例所需的類型：

        from pyspark.sql.types import *

    您使用 PySpark 核心建立 Notebook，因此不需要明確建立任何內容。 當您執行第一個程式碼儲存格時，系統會自動為您建立 Spark 和 Hive 內容。

    ![Jupyter Notebook 作業的狀態](./media/hdinsight-apache-spark-jupyter-spark-sql/hdispark.jupyter.job.status.png "Jupyter Notebook 作業的狀態")

    每當您在 Jupyter 中執行作業時，網頁瀏覽器視窗標題將會顯示 Notebook 標題和 **(忙碌)** 狀態。 您也會在右上角的 **PySpark** 文字旁看到一個實心圓。 工作完成後，實心圓將變成空心圓。

6. 執行下列程式碼，將一些範例資料註冊到名為 **hvac** 的暫存資料表。

        # Load the data
        hvacText = sc.textFile("wasbs:///HdiSamples/HdiSamples/SensorSampleData/hvac/HVAC.csv")

        # Create the schema
        hvacSchema = StructType([StructField("date", StringType(), False),StructField("time", StringType(), False),StructField("targettemp", IntegerType(), False),StructField("actualtemp", IntegerType(), False),StructField("buildingID", StringType(), False)])

        # Parse the data in hvacText
        hvac = hvacText.map(lambda s: s.split(",")).filter(lambda s: s[0] != "Date").map(lambda s:(str(s[0]), str(s[1]), int(s[2]), int(s[3]), str(s[6]) ))

        # Create a data frame
        hvacdf = sqlContext.createDataFrame(hvac,hvacSchema)

        # Register the data fram as a table to run queries against
        hvacdf.registerTempTable("hvac")

    HDInsight 中的 Spark 叢集隨附範例資料檔案 **hvac.csv** (位於 **\HdiSamples\HdiSamples\SensorSampleData\hvac**)。
    
7. 執行下列程式碼以查詢資料：

        %%sql
        SELECT buildingID, (targettemp - actualtemp) AS temp_diff, date FROM hvac WHERE date = \"6/1/13\"

   由於您使用的是 PySpark 核心，因此現在可直接在您剛才使用 `%%sql` magic 建立的暫存資料表 **hvac** 上執行 SQL 查詢。 如需 `%%sql` magic 及 PySpark 核心提供的其他 magic 的詳細資訊，請參閱 [使用 Spark HDInsight 叢集之 Jupyter Notebook 上可用的核心](hdinsight-apache-spark-jupyter-notebook-kernels.md#why-should-i-use-the-pyspark-or-spark-kernels)。

   預設會顯示下列表格式輸出。

     ![查詢結果的資料表輸出](./media/hdinsight-apache-spark-jupyter-spark-sql/tabular.output.png "查詢結果的資料表輸出")

    您也可以查看其他視覺效果中的結果。 例如，相同輸出的區域圖看起來會如下所示。

    ![查詢結果的區域圖](./media/hdinsight-apache-spark-jupyter-spark-sql/area.output.png "查詢結果的區域圖")

9. 應用程式執行完畢之後，您可以關閉 Notebook 來釋放資源。 若要這樣做，請從 Notebook 的 [檔案] 功能表中，按一下 [關閉並停止]。 這樣就能夠結束並關閉 Notebook。

## <a name="delete-the-cluster"></a>刪除叢集
[!INCLUDE [delete-cluster-warning](../../includes/hdinsight-delete-cluster-warning.md)]

## <a name="see-also"></a>另請參閱
* [概觀：Azure HDInsight 上的 Apache Spark](hdinsight-apache-spark-overview.md)

### <a name="scenarios"></a>案例
* [Spark 和 BI：在 HDInsight 中搭配使用 Spark 和 BI 工具執行互動式資料分析](hdinsight-apache-spark-use-bi-tools.md)
* [Spark 和機器學習服務：使用 HDInsight 中的 Spark，利用 HVAC 資料來分析建築物溫度](hdinsight-apache-spark-ipython-notebook-machine-learning.md)
* [Spark 和機器學習服務：使用 HDInsight 中的 Spark 來預測食品檢查結果](hdinsight-apache-spark-machine-learning-mllib-ipython.md)
* [Spark 串流：使用 HDInsight 中的 Spark 來建置即時串流應用程式](hdinsight-apache-spark-eventhub-streaming.md)
* [使用 HDInsight 中的 Spark 進行網站記錄分析](hdinsight-apache-spark-custom-library-website-log-analysis.md)
* [HDInsight 中使用 Spark 的 Application Insight 遙測資料分析](hdinsight-spark-analyze-application-insight-logs.md)

### <a name="create-and-run-applications"></a>建立及執行應用程式
* [使用 Scala 建立獨立應用程式](hdinsight-apache-spark-create-standalone-application.md)
* [利用 Livy 在 Spark 叢集上遠端執行作業](hdinsight-apache-spark-livy-rest-interface.md)

### <a name="tools-and-extensions"></a>工具和擴充功能
* [Use HDInsight Tools Plugin for IntelliJ IDEA to create and submit Spark Scala applicatons (使用 IntelliJ IDEA 的 HDInsight Tools 外掛程式來建立和提交 Spark Scala 應用程式)](hdinsight-apache-spark-intellij-tool-plugin.md)
* [使用 IntelliJ IDEA 的 HDInsight Tools 外掛程式遠端偵錯 Spark 應用程式](hdinsight-apache-spark-intellij-tool-plugin-debug-jobs-remotely.md)
* [利用 HDInsight 上的 Spark 叢集來使用 Zeppelin Notebook](hdinsight-apache-spark-use-zeppelin-notebook.md)
* [HDInsight 的 Spark 叢集中 Jupyter Notebook 可用的核心](hdinsight-apache-spark-jupyter-notebook-kernels.md)
* [搭配 Jupyter Notebook 使用外部套件](hdinsight-apache-spark-jupyter-notebook-use-external-packages.md)
* [在電腦上安裝 Jupyter 並連接到 HDInsight Spark 叢集](hdinsight-apache-spark-jupyter-notebook-install-locally.md)

### <a name="manage-resources"></a>管理資源
* [在 Azure HDInsight 中管理 Apache Spark 叢集的資源](hdinsight-apache-spark-resource-manager.md)
* [追蹤和偵錯在 HDInsight 中的 Apache Spark 叢集上執行的作業](hdinsight-apache-spark-job-debugging.md)

[hdinsight-versions]: hdinsight-component-versioning.md
[hdinsight-upload-data]: hdinsight-upload-data.md
[hdinsight-storage]: hdinsight-hadoop-use-blob-storage.md

[azure-purchase-options]: http://azure.microsoft.com/pricing/purchase-options/
[azure-member-offers]: http://azure.microsoft.com/pricing/member-offers/
[azure-free-trial]: http://azure.microsoft.com/pricing/free-trial/
[azure-management-portal]: https://manage.windowsazure.com/
[azure-create-storageaccount]: storage-create-storage-account.md



<!--HONumber=Jan17_HO2-->


