---
title: "利用堆積傾印偵錯和分析與 Hadoop 服務 |Microsoft Docs"
description: "自動為 Hadoop 服務收集堆積傾印，並放在 Azure Blob 儲存體帳戶內以供偵錯和分析。"
services: hdinsight
documentationcenter: 
tags: azure-portal
author: mumian
manager: jhubbard
editor: cgronlun
ms.assetid: e4ec4ebb-fd32-4668-8382-f956581485c4
ms.service: hdinsight
ms.workload: big-data
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 10/19/2016
ms.author: jgao
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: 2382b1a752312511ebe29403acfbed2242c08f74


---
# <a name="collect-heap-dumps-in-blob-storage-to-debug-and-analyze-hadoop-services"></a>收集 Blob 儲存體中的堆積傾印以偵錯和分析 Hadoop 服務
[!INCLUDE [heapdump-selector](../../includes/hdinsight-selector-heap-dump.md)]

堆積傾印含有應用程式記憶體的快照，其中包括建立傾印時的變數值。 因此堆積傾印非常有助於診斷在執行階段發生的問題。 系統可以自動為 Hadoop 服務收集堆積傾印，並放在使用者之 Azure Blob 儲存體帳戶內的 HDInsightHeapDumps/ 下。 

啟用各種服務的堆積傾印收集時，必須針對個別叢集上的服務啟用。 叢集的這項功能預設為關閉。 這些堆積傾印的大小可能會很大，因此一旦啟用收集，建議您監視儲存這些傾印的 Blob 儲存體帳戶。

> [!NOTE]
> 本文的資訊僅適用於以 Windows 為基礎的 HDInsight。 如需以 Linux 為基礎的 HDInsight 資訊，請參閱 [在以 Linux 為基礎的 HDInsight 上啟用 Hadoop 服務的堆積傾印](hdinsight-hadoop-collect-debug-heap-dump-linux.md)
> 
> 

## <a name="eligible-services-for-heap-dumps"></a>堆積傾印的合格服務
您可以啟用下列服務的堆積傾印：

* **hcatalog** - tempelton
* **hive** - hiveserver2、metastore、derbyserver
* **mapreduce** - jobhistoryserver
* **yarn** - resourcemanager、nodemanager、timelineserver
* **hdfs** - datanode、secondarynamenode、namenode

## <a name="configuration-elements-that-enable-heap-dumps"></a>可啟用堆積傾印的組態元素
為了開啟服務的堆積傾印，您必須在該服務的區段 (由 **service_name** 指定) 中，設定適當的組態元素。

    "javaargs.<service_name>.XX:+HeapDumpOnOutOfMemoryError" = "-XX:+HeapDumpOnOutOfMemoryError",
    "javaargs.<service_name>.XX:HeapDumpPath" = "-XX:HeapDumpPath=c:\Dumps\<service_name>_%date:~4,2%%date:~7,2%%date:~10,2%%time:~0,2%%time:~3,2%%time:~6,2%.hprof"

**service_name** 的值可以是上述所列的任何服務：tempelton、hiveserver2、metastore、derbyserver、jobhistoryserver、resourcemanager、nodemanager、timelineserver、datanode、secondarynamenode 或 namenode。

## <a name="enable-using-azure-powershell"></a>使用 Azure PowerShell 啟用
例如，若要使用 Azure PowerShell 來開啟 jobhistoryserver 的堆積傾印，您必須執行下列作業：

[!INCLUDE [upgrade-powershell](../../includes/hdinsight-use-latest-powershell.md)]

    $MapRedConfigValues = new-object 'Microsoft.WindowsAzure.Management.HDInsight.Cmdlet.DataObjects.AzureHDInsightMapReduceConfiguration'

    $MapRedConfigValues.Configuration = @{ "javaargs.jobhistoryserver.XX:+HeapDumpOnOutOfMemoryError"="-XX:+HeapDumpOnOutOfMemoryError" ; "javaargs.jobhistoryserver.XX:HeapDumpPath" = "-XX:HeapDumpPath=c:\\Dumps\\jobhistoryserver_%date:~4,2%_%date:~7,2%_%date:~10,2%_%time:~0,2%_%time:~3,2%_%time:~6,2%.hprof" }

## <a name="enable-using-net-sdk"></a>使用 .NET SDK 啟用
例如，若要使用 Azure HDInsight .NET SDK 來開啟 jobhistoryserver 的堆積傾印，您必須執行下列作業：

    clusterInfo.MapReduceConfiguration.ConfigurationCollection.Add(new KeyValuePair<string, string>("javaargs.jobhistoryserver.XX:+HeapDumpOnOutOfMemoryError", "-XX:+HeapDumpOnOutOfMemoryError"));

    clusterInfo.MapReduceConfiguration.ConfigurationCollection.Add(new KeyValuePair<string, string>("javaargs.jobhistoryserver.XX:HeapDumpPath", "-XX:HeapDumpPath=c:\\Dumps\\jobhistoryserver_%date:~4,2%_%date:~7,2%_%date:~10,2%_%time:~0,2%_%time:~3,2%_%time:~6,2%.hprof"));



<!--HONumber=Nov16_HO3-->


