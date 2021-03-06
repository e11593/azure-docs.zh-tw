---
title: "使用 Mahout 與以 Windows 為基礎的 HDInsight 產生推薦 | Microsoft Docs"
description: "了解如何搭配以 Windows 為基礎的 HDInsight (Hadoop) 使用 Apache Mahout 機器學習庫來產生電影推薦。"
services: hdinsight
documentationcenter: 
author: Blackmist
manager: jhubbard
editor: cgronlun
tags: azure-portal
ms.assetid: 07b57208-32aa-4e59-900a-6c934fa1b7a7
ms.service: hdinsight
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/11/2016
ms.author: larryfr
translationtype: Human Translation
ms.sourcegitcommit: fc79b8017f2184091f2473a0ff9cdfbd0a4cbdf8
ms.openlocfilehash: c7499dba829e24e957cae47dae88599013c3de95


---
# <a name="generate-movie-recommendations-by-using-apache-mahout-with-hadoop-in-hdinsight"></a>透過在 HDInsight 上將 Apache Mahout 與 Hadoop 搭配使用來產生電影推薦
[!INCLUDE [mahout-selector](../../includes/hdinsight-selector-mahout.md)]

了解如何使用搭配 Azure HDInsight 的 [Apache Mahout](http://mahout.apache.org) 機器學習庫產生電影推薦。

> [!NOTE]
> 此文件中的步驟需要 Windows 用戶端和 Windows 架構的 HDInsight 叢集。 如需搭配使用來自 Linux、OS X 或 Unix 用戶端的 Mahout 與 Linux 架構之 HDInsight 叢集的相關資訊，請參閱 [透過在 HDInsight 上將 Apache Mahout 與 Linux 架構的 Hadoop 搭配使用來產生電影推薦清單](hdinsight-hadoop-mahout-linux-mac.md)

## <a name="a-namelearnawhat-you-will-learn"></a><a name="learn"></a>您將了解
Mahout 是 Apache Hadoop 的[機器學習服務][ml]程式庫。 Mahout 包含可處理資料的演算法，例如篩選、分類和叢集化。 在本文中，您將使用推薦引擎，其將根據朋友看過的電影來產生電影推薦。 您也將了解如何利用決策森林來進行分類。 本文將會告訴您：

* 如何使用 Windows PowerShell 執行 Mahout 工作
* 如何從 Hadoop 命令列執行 Mahout 工作
* 如何在 HDInsight 3.0 和 HDInsight 2.0 叢集上安裝 Mahout

  > [!NOTE]
  > Mahout 提供 HDInsight 3.1 版的叢集。 如果您正在使用舊版的 HDInsight，請參閱 [安裝 Mahout](#install) 後再繼續。

## <a name="prerequisites"></a>必要條件
* **HDInsight 中 Windows 架構的 Hadoop 叢集**。 如需建立此叢集的相關資訊，請參閱[開始在 HDInsight 中使用 Hadoop][getstarted]。
* **具有 Azure PowerShell 的工作站**。

    > [!IMPORTANT]
    > 使用 Azure Service Manager 管理 HDInsight 資源的 Azure PowerShell 支援已**被取代**，將會在 2017 年 1 月 1 日前移除。 本文件中的步驟會使用可與 Azure Resource Manager 搭配使用的新 HDInsight Cmdlet。
    >
    > 請遵循 [安裝和設定 Azure PowerShell](/powershell/azureps-cmdlets-docs) 中的步驟來安裝最新版的 Azure PowerShell。 如果您需要修改指令碼才能使用適用於 Azure Resource Manager 的新 Cmdlet，請參閱 [移轉至以 Azure Resource Manager 為基礎的開發工具 (適用於 HDInsight 叢集)](hdinsight-hadoop-development-using-azure-resource-manager.md) ，以取得詳細資訊。

## <a name="a-namerecommendationsagenerate-recommendations-by-using-windows-powershell"></a><a name="recommendations"></a>使用 Windows PowerShell 產生推薦
> [!NOTE]
> 雖然本節中使用的工作能夠利用 Windows PowerShell 來執行，但 Mahout 隨附的許多類別目前仍無法搭配 Windows PowerShell 運作，而必須使用 Hadoop 命令列來執行。 如需無法搭配 Windows PowerShell 使用的類別清單，請參閱 [疑難排解](#troubleshooting) 一節。
>
> 如需使用 Hadoop 命令列執行 Mahout 工作的範例，請參閱 [使用 Hadoop 命令列將資料分類](#classify)。

Mahout 提供的其中一項功能是推薦引擎。 這個引擎接受 `userID``itemId` 和 `prefValue` (使用者偏好的項目) 格式的資料。 Mahout 接著可以執行共生分析來判斷：「偏好某項目的使用者同時也偏好其他這些項目」。 接著 Mahout 會以偏好的類似項目判斷使用者，並以此做出推薦。

以下使用電影做一個很簡單的範例：

* **共生**：Joe、Alice 和 Bob 都喜歡*《星際大戰》*、*《帝國大反擊》*和*《絕地大反攻》*。 Mahout 將判斷喜歡上述任何一部電影的使用者，也會喜歡另外兩部電影。
* **共生**：Bob 和 Alice 同時也喜歡*《威脅潛伏》*、*《複製人全面進攻》*和*《西斯大帝的復仇》*。 Mahout 將判斷喜歡前三部電影的使用者，也會喜歡這三部電影。
* **相似性推薦**：因為 Joe 喜歡前三部電影，Mahout 會查看具有相似偏好的其他使用者所喜歡但 Joe 還沒看過 (喜歡/評價) 的電影。 在此情況下，Mahout 將會推薦*《威脅潛伏》*、*《複製人全面進攻》*和*《西斯大帝的復仇》*。

### <a name="understanding-the-data"></a>了解資料
[GroupLens 研究][movielens]提供 Mahout 相容格式的電影評價資料，相當方便。 您可在位於 `/HdiSamples/MahoutMovieData`的叢集預設儲存體取得這份資料。

有兩份檔案：`moviedb.txt` (影片相關資訊) 和 `user-ratings.txt`。 分析期間使用的是 user-ratings.txt 檔案，moviedb.txt 則是在顯示分析結果時用來提供使用者易懂的文字資訊。

user-ratings.txt 內包含的資料具有 `userID`、`movieID`、`userRating` 和 `timestamp` 結構，可告訴我們每位使用者對於影片的評價為何。 以下是資料範例：

    196    242    3    881250949
    186    302    3    891717742
    22    377    1    878887116
    244    51    2    880606923
    166    346    1    886397596

### <a name="run-the-job"></a>執行工作
使用下列 Windows PowerShell 指令碼執行工作，以透過 Mahout 推薦引擎來處理影片資料：

```powershell
# The HDInsight cluster name.
$clusterName = "the cluster name"

#Get HTTPS/Admin credentials for submitting the job later
$creds = Get-Credential
#Get the cluster info so we can get the resource group, storage, etc.
$clusterInfo = Get-AzureRmHDInsightCluster -ClusterName $clusterName
$resourceGroup = $clusterInfo.ResourceGroup
$storageAccountName=$clusterInfo.DefaultStorageAccount.split('.')[0]
$container=$clusterInfo.DefaultStorageContainer
$storageAccountKey=(Get-AzureRmStorageAccountKey `
    -Name $storageAccountName `
    -ResourceGroupName $resourceGroup)[0].Value

#Create a storage content and upload the file
$context = New-AzureStorageContext `
    -StorageAccountName $storageAccountName `
    -StorageAccountKey $storageAccountKey

# NOTE: The version number in the file path
# may change in future versions of HDInsight.
$jarFile =  "file:///C:/apps/dist/mahout-0.9.0.2.2.9.1-8/examples/target/mahout-examples-0.9.0.2.2.9.1-8-job.jar"
#
# If you are using an earlier version of HDInsight,
# set $jarFile to the jar file you
# uploaded.
# For example,
# $jarFile = "wasbs:///example/jars/mahout-core-0.9-job.jar"

# The arguments for this job
# * input - the path to the data uploaded to HDInsight
# * output - the path to store output data
# * tempDir - the directory for temp files
$jobArguments = "--similarityClassname", "recommenditembased", `
                "-s", "SIMILARITY_COOCCURRENCE", `
                "--input", "wasbs:///HdiSamples/MahoutMovieData/user-ratings.txt",
                "--output", "wasbs:///example/out",
                "--tempDir", "wasbs:///example/temp"

# Create the job definition
$jobDefinition = New-AzureRmHDInsightMapReduceJobDefinition `
    -JarFile $jarFile `
    -ClassName "org.apache.mahout.cf.taste.hadoop.item.RecommenderJob" `
    -Arguments $jobArguments

# Start the job
$job = Start-AzureRmHDInsightJob `
    -ClusterName $clusterName `
    -JobDefinition $jobDefinition `
    -HttpCredential $creds

# Wait on the job to complete
Write-Host "Wait for the job to complete ..." -ForegroundColor Green
Wait-AzureRmHDInsightJob `
        -ClusterName $clusterName `
        -JobId $job.JobId `
        -HttpCredential $creds
# Download the output
Get-AzureStorageBlobContent `
        -Blob example/out/part-r-00000 `
        -Container $container `
        -Destination output.txt `
        -Context $context

# Write out any error information
Write-Host "STDERR"
Get-AzureRmHDInsightJobOutput `
        -Clustername $clusterName `
        -JobId $job.JobId `
        -DefaultContainer $container `
        -DefaultStorageAccountName $storageAccountName `
        -DefaultStorageAccountKey $storageAccountKey `
        -HttpCredential $creds `
        -DisplayOutputType StandardError
```

> [!NOTE]
> Mahout 工作不會移除處理工作時所建立的暫存資料。 範例工作中所指定的 `--tempDir` 參數會將暫存檔隔離到特定的目錄。

Mahout 工作不會將輸出傳回 STDOUT。 相反地，其會將該輸出儲存在指定的輸出目錄 **part-r-00000**中。 指令碼會下載這個檔案到您工作站上目前目錄的 **output.txt** 檔。

以下是此檔案的內容範例：

    1    [234:5.0,347:5.0,237:5.0,47:5.0,282:5.0,275:5.0,88:5.0,515:5.0,514:5.0,121:5.0]
    2    [282:5.0,210:5.0,237:5.0,234:5.0,347:5.0,121:5.0,258:5.0,515:5.0,462:5.0,79:5.0]
    3    [284:5.0,285:4.828125,508:4.7543354,845:4.75,319:4.705128,124:4.7045455,150:4.6938777,311:4.6769233,248:4.65625,272:4.649266]
    4    [690:5.0,12:5.0,234:5.0,275:5.0,121:5.0,255:5.0,237:5.0,895:5.0,282:5.0,117:5.0]

第一欄是 `userID`。 '[' 和 ']' 中包含的值是 `movieId`:`recommendationScore`。

### <a name="view-the-output"></a>檢視輸出
雖然產生的輸出可以在應用程式中使用，但非常難以讓人判讀。 伺服器的 `moviedb.txt` 可用來將 `movieId` 解析為影片名稱，但是您必須先使用下列指令碼，從伺服器下載此項目與分級檔案︰

```powershell
# The HDInsight cluster name.
$clusterName = "the cluster name"

#Get the cluster info so we can get the resource group, storage, etc.
$clusterInfo = Get-AzureRmHDInsightCluster -ClusterName $clusterName
$resourceGroup = $clusterInfo.ResourceGroup
$storageAccountName=$clusterInfo.DefaultStorageAccount.split('.')[0]
$container=$clusterInfo.DefaultStorageContainer
$storageAccountKey=(Get-AzureRmStorageAccountKey `
    -Name $storageAccountName `
    -ResourceGroupName $resourceGroup)[0].Value
#Create a storage content and upload the file
$context = New-AzureStorageContext `
    -StorageAccountName $storageAccountName `
    -StorageAccountKey $storageAccountKey
#Download the files
Get-AzureStorageBlobContent -blob "HdiSamples/MahoutMovieData/moviedb.txt" `
-Container $container `
-Destination moviedb.txt `
-Context $context
Get-AzureStorageBlobContent -blob "HdiSamples/MahoutMovieData/user-ratings.txt" `
-Container $container `
-Destination user-ratings.txt `
-Context $context
```

下載檔案之後，即可使用下列 PowerShell 指令碼來顯示建議的影片名稱︰

```powershell
<#
.SYNOPSIS
    Displays recommendations for movies.
.DESCRIPTION
    Displays recommendations generated by Mahout
    with HDInsight example in a human readable format.
.EXAMPLE
    .\Show-Recommendation -userId 4
        -userDataFile "user-ratings.txt"
        -movieFile "moviedb.txt"
        -recommendationFile "output.txt"
#>

[CmdletBinding(SupportsShouldProcess = $true)]
param(
    #The user ID
    [Parameter(Mandatory = $true)]
    [String]$userId,

    [Parameter(Mandatory = $true)]
    [String]$userDataFile,

    [Parameter(Mandatory = $true)]
    [String]$movieFile,

    [Parameter(Mandatory = $true)]
    [String]$recommendationFile
)
# Read movie ID & description into hash table
Write-Host "Reading movies descriptions" -ForegroundColor Green
$movieById = @{}
foreach($line in Get-Content $movieFile)
{
    $tokens = $line.Split("|")
    $movieById[$tokens[0]] = $tokens[1]
}
# Load movies user has already seen (rated)
# into a hash table
Write-Host "Reading rated movies" -ForegroundColor Green
$ratedMovieIds = @{}
foreach($line in Get-Content $userDataFile)
{
    $tokens = $line.Split("`t")
    if($tokens[0] -eq $userId)
    {
        # Resolve the ID to the movie name
        $ratedMovieIds[$movieById[$tokens[1]]] = $tokens[2]
    }
}
# Read recommendations generated by Mahout
Write-Host "Reading recommendations" -ForegroundColor Green
$recommendations = @{}
foreach($line in get-content $recommendationFile)
{
    $tokens = $line.Split("`t")
    if($tokens[0] -eq $userId)
    {
        #Trim leading/treailing [] and split at ,
        $movieIdAndScores = $tokens[1].TrimStart("[").TrimEnd("]").Split(",")
        foreach($movieIdAndScore in $movieIdAndScores)
        {
            #Split at : and store title and score in a hash table
            $idAndScore = $movieIdAndScore.Split(":")
            $recommendations[$movieById[$idAndScore[0]]] = $idAndScore[1]
        }
        break
    }
}

Write-Host "Rated movies" -ForegroundColor Green
Write-Host "---------------------------" -ForegroundColor Green
$ratedFormat = @{Expression={$_.Name};Label="Movie";Width=40}, `
                @{Expression={$_.Value};Label="Rating"}
$ratedMovieIds | format-table $ratedFormat
Write-Host "---------------------------" -ForegroundColor Green

write-host "Recommended movies" -ForegroundColor Green
Write-Host "---------------------------" -ForegroundColor Green
$recommendationFormat = @{Expression={$_.Name};Label="Movie";Width=40}, `
                        @{Expression={$_.Value};Label="Score"}
$recommendations | format-table $recommendationFormat
```

以下是執行此指令碼的範例：

    PS C:\> show-recommendation.ps1 -userId 4 -userDataFile .\user-ratings.txt -movieFile .\moviedb.txt -recommendationFile .\output.txt

輸出應該如下所示：

    Reading movies descriptions
    Reading rated movies
    Reading recommendations
    Rated movies
    ---------------------------
    Movie                                    Rating
    -----                                    ------
    Devil's Own, The (1997)                  1
    Alien: Resurrection (1997)               3
    187 (1997)                               2
    (lines ommitted)

    ---------------------------
    Recommended movies
    ---------------------------

    Movie                                    Score
    -----                                    -----
    Good Will Hunting (1997)                 4.6504064
    Swingers (1996)                          4.6862745
    Wings of the Dove, The (1997)            4.6666665
    People vs. Larry Flynt, The (1996)       4.834559
    Everyone Says I Love You (1996)          4.707071
    Secrets & Lies (1996)                    4.818182
    That Thing You Do! (1996)                4.75
    Grosse Pointe Blank (1997)               4.8235292
    Donnie Brasco (1997)                     4.6792455
    Lone Star (1996)                         4.7099237

## <a name="a-nameclassifyaclassify-data-by-using-the-hadoop-command-line"></a><a name="classify"></a>使用 Hadoop 命令列將資料分類
Mahout 隨附的其中一個分類方法是建置[隨機樹系][forest]。 這是多重步驟的程序，其包含使用訓練資料來產生決策樹，再用以將資料分類。 此程序使用 Mahout 提供的 **org.apache.mahout.classifier.df.tools.Describe** 類別。 目前必須使用 Hadoop 命令列來執行。

### <a name="load-the-data"></a>載入資料
1. 從 [NSL-KDD 資料集](http://nsl.cs.unb.ca/NSL-KDD/)下載下列檔案。

   * [KDDTrain+.ARFF](http://nsl.cs.unb.ca/NSL-KDD/KDDTrain+.arff)：訓練檔案
   * [KDDTest+.ARFF](http://nsl.cs.unb.ca/NSL-KDD/KDDTest+.arff)：測試資料
2. 開啟每一個檔案並移除頂端以 '@', 開頭的各行，然後儲存檔案。 如果未移除，則在 Mahout 中使用此資料時會收到錯誤訊息。
3. 將檔案上傳至 **example/data**。 您可以使用下列指令碼來達成目的： 將 **CLUSTERNAME** 取代為 HDInsight 叢集的名稱。 使用要上傳檔案的名稱取代 FILENAME。

    ```powershell
    #Get the cluster info so we can get the resource group, storage, etc.
    $clusterName="CLUSTERNAME"
    $fileToUpload="FILENAME"
    $blobPath="example/data/FILENAME"
    $clusterInfo = Get-AzureRmHDInsightCluster -ClusterName $clusterName
    $resourceGroup = $clusterInfo.ResourceGroup
    $storageAccountName=$clusterInfo.DefaultStorageAccount.split('.')[0]
    $container=$clusterInfo.DefaultStorageContainer
    $storageAccountKey=(Get-AzureRmStorageAccountKey `
        -Name $storageAccountName `
    -ResourceGroupName $resourceGroup)[0].Value

    #Create a storage content and upload the file
    $context = New-AzureStorageContext `
        -StorageAccountName $storageAccountName `
        -StorageAccountKey $storageAccountKey

    Set-AzureStorageBlobContent `
        -File $fileToUpload `
        -Blob $blobPath `
        -Container $container `
        -Context $context
    ```

### <a name="run-the-job"></a>執行工作
1. 這項工作需要 Hadoop 命令列。 依照 [使用 RDP 連線到 HDInsight 叢集](hdinsight-administer-use-management-portal.md#connect-to-clusters-using-rdp)中的指示，為 HDInsight 叢集啟用遠端桌面，然後進行連線。
2. 連線之後，使用 [Hadoop 命令列]  圖示開啟 Hadoop 命令列：

    ![hadoop cli][hadoopcli]
3. 使用下列命令，並利用 Mahout 產生檔案描述元 (**KDDTrain+.info**)。

        hadoop jar "c:/apps/dist/mahout-0.9.0.2.2.9.1-8/examples/target/mahout-examples-0.9.0.2.2.9.1-8-job.jar" org.apache.mahout.classifier.df.tools.Describe -p "wasbs:///example/data/KDDTrain+.arff" -f "wasbs:///example/data/KDDTrain+.info" -d N 3 C 2 N C 4 N C 8 N 2 C 19 N L

    `N 3 C 2 N C 4 N C 8 N 2 C 19 N L` 描述檔案中的資料屬性。 例如，L 表示標籤。
4. 使用下列命令建置決策樹的森林：

        hadoop jar c:/apps/dist/mahout-0.9.0.2.2.9.1-8/examples/target/mahout-examples-0.9.0.2.2.9.1-8-job.jar org.apache.mahout.classifier.df.mapreduce.BuildForest -Dmapred.max.split.size=1874231 -d wasbs:///example/data/KDDTrain+.arff -ds wasbs:///example/data/KDDTrain+.info -sl 5 -p -t 100 -o nsl-forest

    此作業的輸出會儲存在 **nsl-forest** 目錄，其位於您的 HDInsight 叢集的儲存體中，位於：__wasbs://user/&lt;username>/nsl-forest/nsl-forest.seq。 &lt;username> 是您遠端桌面工作階段的使用者名稱。 此檔案無法讓人判讀。
5. 將 **KDDTest+.arff** 資料集分類來測試森林。 使用下列命令：

        hadoop jar c:/apps/dist/mahout-0.9.0.2.2.9.1-8/examples/target/mahout-examples-0.9.0.2.2.9.1-8-job.jar org.apache.mahout.classifier.df.mapreduce.TestForest -i wasbs:///example/data/KDDTest+.arff -ds wasbs:///example/data/KDDTrain+.info -m nsl-forest -a -mr -o wasbs:///example/data/predictions

    此命令會傳回分類流程的摘要資訊，類似下列所示：

        14/07/02 14:29:28 INFO mapreduce.TestForest:

        =======================================================
        Summary
        -------------------------------------------------------
        Correctly Classified Instances          :      17560       77.8921%
        Incorrectly Classified Instances        :       4984       22.1079%
        Total Classified Instances              :      22544

        =======================================================
        Confusion Matrix
        -------------------------------------------------------
        a       b       <--Classified as
        9437    274      |  9711        a     = normal
        4710    8123     |  12833       b     = anomaly

        =======================================================
        Statistics
        -------------------------------------------------------
        Kappa                                       0.5728
        Accuracy                                   77.8921%
        Reliability                                53.4921%
        Reliability (standard deviation)            0.4933

   此作業也會產生一個位於 **wasbs:///example/data/predictions/KDDTest+.arff.out** 的檔案。 不過，此檔案無法讓人判讀。

> [!NOTE]
> Mahout 工作不會覆寫檔案。 如果您想要重新執行這些工作，則必須刪除先前的工作所建立的檔案。

## <a name="a-nametroubleshootingatroubleshooting"></a><a name="troubleshooting"></a>疑難排解
### <a name="a-nameinstallainstall-mahout"></a><a name="install"></a>安裝 Mahout
Mahout 安裝於 HDInsight 3.1 叢集上，且可使用下列步驟來手動安裝到 HDInsight 3.0 或 HDInsight 2.1 叢集上：

1. 使用的 Mahout 版本視叢集的 HDInsight 版本而定。 您可以透過在 Azure portal 中檢閱叢集屬性來尋找叢集版本。

   * **針對 HDInsight 2.1**，您可以下載包含 [Mahout 0.9](http://repo2.maven.org/maven2/org/apache/mahout/mahout-core/0.9/mahout-core-0.9-job.jar)的 Java 封存 (JAR) 檔案。
   * **針對 HDInsight 3.0**，您必須[從原始檔建置 Mahout][build]，並指定 HDInsight 所提供的 Hadoop 版本。 安裝建置頁面上列出的必要條件、下載原始檔，然後使用下列命令建立 Mahout jar 檔案：

            mvn -Dhadoop2.version=2.2.0 -DskipTests clean package

        建置完成之後，您可以在 **mahout\mrlegacy\target\mahout-mrlegacy-1.0-SNAPSHOT-job.jar** 找到 JAR 檔案。

     > [!NOTE]
     > 當 Mahout 1.0 發行時，您應能搭配 HDInsight 3.0 使用預先建置的封裝。

2. 將 jar 檔案上傳至叢集預設儲存庫中的 **example/jars** 。 使用您 HDInsight 叢集的名稱來取代下列程式碼中的 CLUSTERNAME，並且使用指向 **mahout-coure-0.9-job.jar** 檔案的路徑來取代 FILENAME。

        #Get the cluster info so we can get the resource group, storage, etc.
        $clusterName = "CLUSTERNAME"
        $fileToUpload = "FILENAME"
        $clusterInfo = Get-AzureRmHDInsightCluster -ClusterName $clusterName
        $resourceGroup = $clusterInfo.ResourceGroup
        $storageAccountName=$clusterInfo.DefaultStorageAccount.split('.')[0]
        $container=$clusterInfo.DefaultStorageContainer
        $storageAccountKey=(Get-AzureRmStorageAccountKey `
            -Name $storageAccountName `
        -ResourceGroupName $resourceGroup)[0].Value

        #Create a storage content and upload the file
        $context = New-AzureStorageContext `
            -StorageAccountName $storageAccountName `
            -StorageAccountKey $storageAccountKey

        Set-AzureStorageBlobContent `
            -File $fileToUpload `
            -Blob "example/jars/mahout-core-0.9-job.jar" `
            -Container $container `
            -Context $context

### <a name="cannot-overwrite-files"></a>無法覆寫檔案
Mahout 工作不會清除在處理期間所建立的暫存檔。 此外，工作也不會覆寫現有的輸出檔。

為了避免執行 Mahout 工作時發生錯誤，請在每次執行之前刪除暫存檔和輸出檔，或使用唯一的暫存和輸出目錄名稱。

### <a name="cannot-find-the-jar-file"></a>找不到 JAR 檔案
HDInsight 3.1 叢集包含 Mahout。 路徑和檔案名稱包含叢集上安裝之 Mahout 的版本號碼。 本教學課程中的 Windows PowerShell 範例指令碼使用在 2015 年 11 月時有效的路徑，但版本號碼將隨著未來 HDInsight 更新而有所變更。 若要判斷您叢集的 Mahout JAR 檔案的目前路徑，請使用下列 Windows PowerShell 命令，然後將指令碼修改為參考傳回的檔案路徑：

```powershell
Use-AzureRmHDInsightCluster -ClusterName $clusterName
#Get the cluster info so we can get the resource group, storage, etc.
    $clusterInfo = Get-AzureRmHDInsightCluster -ClusterName $clusterName
    $resourceGroup = $clusterInfo.ResourceGroup
    $storageAccountName=$clusterInfo.DefaultStorageAccount.split('.')[0]
    $container=$clusterInfo.DefaultStorageContainer
    $storageAccountKey=(Get-AzureRmStorageAccountKey `
        -Name $storageAccountName `
    -ResourceGroupName $resourceGroup)[0].Value
Invoke-AzureRmHDInsightHiveJob `
        -StatusFolder "wasbs:///example/statusout" `
        -DefaultContainer $container `
        -DefaultStorageAccountName $storageAccountName `
        -DefaultStorageAccountKey $storageAccountKey `
        -Query '!${env:COMSPEC} /c dir /b /s ${env:MAHOUT_HOME}\examples\target\*-job.jar'
```

### <a name="a-namenopowershellaclasses-that-do-not-work-with-windows-powershell"></a><a name="nopowershell"></a>不適用於 Windows PowerShell 的類別
如果從 Windows PowerShell 中使用的 Mahout 工作利用到下列類別，則會傳回各種錯誤訊息：

* org.apache.mahout.utils.clustering.ClusterDumper
* org.apache.mahout.utils.SequenceFileDumper
* org.apache.mahout.utils.vectors.lucene.Driver
* org.apache.mahout.utils.vectors.arff.Driver
* org.apache.mahout.text.WikipediaToSequenceFile
* org.apache.mahout.clustering.streaming.tools.ResplitSequenceFiles
* org.apache.mahout.clustering.streaming.tools.ClusterQualitySummarizer
* org.apache.mahout.classifier.sgd.TrainLogistic
* org.apache.mahout.classifier.sgd.RunLogistic
* org.apache.mahout.classifier.sgd.TrainAdaptiveLogistic
* org.apache.mahout.classifier.sgd.ValidateAdaptiveLogistic
* org.apache.mahout.classifier.sgd.RunAdaptiveLogistic
* org.apache.mahout.classifier.sequencelearning.hmm.BaumWelchTrainer
* org.apache.mahout.classifier.sequencelearning.hmm.ViterbiEvaluator
* org.apache.mahout.classifier.sequencelearning.hmm.RandomSequenceGenerator
* org.apache.mahout.classifier.df.tools.Describe

若要執行用到這些類別的工作，請連接到 HDInsight 叢集，然後使用 Hadoop 命令列執行工作。 相關範例請參閱 [使用 Hadoop 命令列將資料分類](#classify) 。

## <a name="next-steps"></a>後續步驟
您現在已了解如何使用 Mahout，請繼續探索在 HDInsight 上使用資料的其他方法：

* [搭配 HDInsight 使用 Hive](hdinsight-use-hive.md)
* [搭配 HDInsight 使用 Pig](hdinsight-use-pig.md)
* [搭配 HDInsight 使用 MapReduce](hdinsight-use-mapreduce.md)

[build]: http://mahout.apache.org/developers/buildingmahout.html
[aps]: /powershell/azureps-cmdlets-docs
[movielens]: http://grouplens.org/datasets/movielens/
[100k]: http://files.grouplens.org/datasets/movielens/ml-100k.zip
[getstarted]: hdinsight-hadoop-linux-tutorial-get-started.md
[upload]: hdinsight-upload-data.md
[ml]: http://en.wikipedia.org/wiki/Machine_learning
[forest]: http://en.wikipedia.org/wiki/Random_forest
[management]: https://manage.windowsazure.com/
[enableremote]: ./media/hdinsight-mahout/enableremote.png
[connect]: ./media/hdinsight-mahout/connect.png
[hadoopcli]: ./media/hdinsight-mahout/hadoopcli.png
[tools]: https://github.com/Blackmist/hdinsight-tools



<!--HONumber=Dec16_HO2-->


