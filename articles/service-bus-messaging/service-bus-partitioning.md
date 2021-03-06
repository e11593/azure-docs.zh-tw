---
title: "分割的佇列和主題 | Microsoft Docs"
description: "說明如何使用多個訊息代理程式分割服務匯流排佇列和主題。"
services: service-bus-messaging
documentationcenter: na
author: sethmanheim
manager: timlt
editor: 
ms.assetid: a0c7d5a2-4876-42cb-8344-a1fc988746e7
ms.service: service-bus-messaging
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 12/22/2016
ms.author: sethm;hillaryc
translationtype: Human Translation
ms.sourcegitcommit: 6b169b3501ac938c1c0d621909dfc82418a7146e
ms.openlocfilehash: 7da16e2bca1332b63709ecff39af7124656db98a


---
# <a name="partitioned-queues-and-topics"></a>分割的佇列和主題
Azure 服務匯流排會採用多個訊息代理人來處理訊息，並採用多個訊息存放區來儲存訊息。 傳統的佇列或主題由單一訊息代理程式處理並儲存在一個訊息存放區中。 服務匯流排也可讓佇列或主題分割到多個訊息訊息代理程式以及訊息存放區。 這表示分割佇列或主題的整體輸送量不會再受到單一訊息代理程式或訊息存放區的效能所限制。 此外，即使訊息存放區暫時中斷也不會讓分割的佇列或主題無法使用。 分割的佇列和主題可以包含所有進階的服務匯流排功能，例如支援交易和工作階段。

如需有關服務匯流排內部的詳細資料，請參閱[服務匯流排架構][Service Bus architecture]一文。

## <a name="how-it-works"></a>運作方式
每個分割的佇列或主題都包含多個片段。 每個片段儲存在不同的訊息存放區中，並由不同的訊息代理人處理。 當訊息傳送至分割的佇列或主題時，服務匯流排會指派訊息到其中一個片段。 選取作業由服務匯流排或使用傳送者可指定的分割索引鍵隨機進行。

當用戶端想要從分割的佇列或從分割主題的訂用帳戶接收訊息時，服務匯流排會查詢所有片段的訊息，然後將取自任何訊息存放區的第一個訊息傳回給接收者。 服務匯流排會快取其他訊息，然後在它收到其他接收要求時將其傳回。 接收的用戶端並不知道分割。分割佇列或主題的用戶端對向行為 (例如讀取、完成、延遲、無效化、預先擷取) 和一般實體的行為相同。

傳送訊息給分割的佇列或主題，或從該處接收訊息時，不需要額外成本。

## <a name="enable-partitioning"></a>啟用分割
若要搭配 Azure 服務匯流排使用分割的佇列和主題，請使用 Azure SDK 2.2 版或更新版本，或在您的 HTTP 要求中指定 `api-version=2013-10`。

您可以建立 1、2、3、4 或 5 GB 大小的服務匯流排佇列及主題 (預設值為 1 GB)。 啟用分割時，服務匯流排會為您指定的每 GB 建立 16 個資料分割。 因此，如果您建立 5 GB 大小的佇列，每 GB 有 16 個資料分割，則佇列大小上限會變成 (5 \* 16) = 80 GB。 您可以在 [Azure 入口網站][Azure portal]上檢視分割的佇列或主題項目，藉此查看其大小上限。

有多種方式可以建立分割的佇列或主題。 當您從應用程式建立佇列或主題時，您可以分別將 [QueueDescription.EnablePartitioning][QueueDescription.EnablePartitioning] 或 [TopicDescription.EnablePartitioning][TopicDescription.EnablePartitioning] 屬性設為 **true** 來啟用佇列或主題的分割。 這些屬性必須在建立佇列或主題時設定。 您不可以在現有的佇列或主題上變更這些屬性。 例如：

```csharp
// Create partitioned topic
NamespaceManager ns = NamespaceManager.CreateFromConnectionString(myConnectionString);
TopicDescription td = new TopicDescription(TopicName);
td.EnablePartitioning = true;
ns.CreateTopic(td);
```

或者，您可以在 Visual Studio 或 [Azure 入口網站][Azure portal]中建立分割的佇列或主題。 當您在入口網站建立新的佇列或主題時，請在佇列或主題 [設定] 視窗的 [一般設定] 刀鋒視窗中，將 [啟用分割] 選項設定為 **true**。 在 Visual Studio 中，按一下 [新增佇列] 或 [新增主題] 對話方塊中的 [啟用分割] 核取方塊。

## <a name="use-of-partition-keys"></a>分割索引鍵的用途
當訊息加入佇列至分割的佇列或主題時，服務匯流排會檢查分割區索引鍵是否存在。 如果找到，它會根據該索引鍵選取片段。 如果找不到分割索引鍵，它會根據內部演算法選取片段。

### <a name="using-a-partition-key"></a>使用分割區索引鍵
某些案例，例如工作階段或交易，需要儲存在特定片段的訊息。 這些案例都需要使用分割索引鍵。 使用相同分割索引鍵的所有訊息都會指派給相同的片段。 若片段暫時無法使用，服務匯流排會傳回錯誤。

根據這個案例，會使用不同的訊息屬性做為分割索引鍵：

**SessionId**：若訊息已設定 [BrokeredMessage.SessionId][BrokeredMessage.SessionId] 屬性，則服務匯流排會使用這個屬性做為分割區索引鍵。 如此一來，所有屬於相同工作階段的訊息都會由相同的訊息代理人處理。 這樣可讓服務匯流排保證訊息的排序以及工作階段狀態的一致性。

**PartitionKey**：若訊息已設定 [BrokeredMessage.PartitionKey][BrokeredMessage.PartitionKey] 屬性而非 [BrokeredMessage.SessionId][BrokeredMessage.SessionId] 屬性，則服務匯流排會使用 [PartitionKey][PartitionKey] 屬性做為分割索引鍵。 如果訊息已設定 [SessionId][SessionId] 和 [PartitionKey][PartitionKey] 屬性，這兩個屬性必須相同。 如果 [PartitionKey][PartitionKey] 屬性設為和 [SessionId][SessionId] 屬性不同的值，服務匯流排會傳回無效作業例外狀況。 如果傳送者傳送非工作階段感知的交易訊息，應使用 [PartitionKey][PartitionKey] 屬性。 分割索引鍵可確保在交易內傳送的所有訊息都由相同的訊息代理人處理。

**MessageId**：如果佇列或主題已將 [QueueDescription.RequiresDuplicateDetection][QueueDescription.RequiresDuplicateDetection] 屬性設為 **true**，且未設定 [BrokeredMessage.SessionId][BrokeredMessage.SessionId] 或 [BrokeredMessage.PartitionKey][BrokeredMessage.PartitionKey] 屬性，則 [BrokeredMessage.MessageId][BrokeredMessage.MessageId] 屬性可做為分割索引鍵。 (請注意，如果傳送應用程式沒有指派訊息識別碼，Microsoft .NET 和 AMQP 程式庫會自動指派)。在此情況下，相同訊息的所有複本會由相同的訊息代理人處理。 這樣可讓服務匯流排偵測並排除重複的訊息。 如果 [QueueDescription.RequiresDuplicateDetection][QueueDescription.RequiresDuplicateDetection] 屬性未設為 **true**，服務匯流排不會將 [MessageId][MessageId] 屬性視為分割索引鍵。

### <a name="not-using-a-partition-key"></a>不使用分割索引鍵
沒有分割區索引鍵時，服務匯流排會以循環配置的方式將訊息分配到分割區佇列或主題的所有片段。 如果找不到所選的片段，服務匯流排會將訊息指派至不同的片段。 如此一來，儘管訊息存放區暫時無法使用，傳送作業仍會成功。

若要讓服務匯流排有足夠的時間將訊息加入佇列的不同片段中，由傳送訊息之用戶端所指定的 [MessagingFactorySettings.OperationTimeout][MessagingFactorySettings.OperationTimeout] 值必須大於 15 秒。 建議將 [OperationTimeout][OperationTimeout] 屬性設為預設值 60 秒。

請注意，分割索引鍵會將訊息「釘選」到指定的片段。 如果保留此片段的訊息存放區無法使用，服務匯流排會傳回錯誤。 沒有分割索引鍵時，服務匯流排可以選擇不同的片段，而作業就會成功。 因此，建議您若非必要請勿提供分割索引鍵。

## <a name="advanced-topics-use-transactions-with-partitioned-entities"></a>進階主題：搭配交易使用分割的實體
傳送做為交易一部分的訊息必須指定資料分割索引鍵。 這可以是下列屬性之一：[BrokeredMessage.SessionId][BrokeredMessage.SessionId]、[BrokeredMessage.PartitionKey][BrokeredMessage.PartitionKey] 或 [BrokeredMessage.MessageId][BrokeredMessage.MessageId]。 傳送做為相同交易一部分的所有訊息必須指定相同的分割索引鍵。 如果您嘗試在交易內傳送沒有分割索引鍵的訊息，服務匯流排會傳回無效作業例外狀況。 如果您嘗試在相同交易內傳送多個具有不同分割索引鍵的訊息，服務匯流排會傳回無效作業例外狀況。 例如：

```csharp
CommittableTransaction committableTransaction = new CommittableTransaction();
using (TransactionScope ts = new TransactionScope(committableTransaction))
{
    BrokeredMessage msg = new BrokeredMessage("This is a message");
    msg.PartitionKey = "myPartitionKey";
    messageSender.Send(msg); 
    ts.Complete();
}
committableTransaction.Commit();
```

如果設定任何做為分割索引鍵的屬性，服務匯流排會將訊息釘選到特定片段。 無論是否使用交易，都會發生這個行為。 建議您若非必要請勿指定分割索引鍵。

## <a name="using-sessions-with-partitioned-entities"></a>搭配工作階段使用分割的實體
若要將交易訊息傳送至工作階段感知的主題或佇列，該訊息必須設定 [BrokeredMessage.SessionId][BrokeredMessage.SessionId] 屬性。 如果也指定 [BrokeredMessage.PartitionKey][BrokeredMessage.PartitionKey] 屬性，它必須與 [SessionId][SessionId] 屬性相同。 如果兩者不同，服務匯流排會傳回無效作業例外狀況。

不同於一般 (非分割) 的佇列或主題，無法使用單一交易將多則訊息傳送到不同的工作階段。 如果嘗試這樣做，服務匯流排會傳回無效作業例外狀況。 例如：

```csharp
CommittableTransaction committableTransaction = new CommittableTransaction();
using (TransactionScope ts = new TransactionScope(committableTransaction))
{
    BrokeredMessage msg = new BrokeredMessage("This is a message");
    msg.SessionId = "mySession";
    messageSender.Send(msg); 
    ts.Complete();
}
committableTransaction.Commit();
```

## <a name="automatic-message-forwarding-with-partitioned-entities"></a>使用分割實體的自動訊息轉送
服務匯流排支援往返於分割實體或在它們之間自動轉送訊息。 若要啟用自動訊息轉送，請在來源佇列或訂用帳戶上設定 [QueueDescription.ForwardTo][QueueDescription.ForwardTo] 屬性。 如果訊息指定分割索引鍵 ([SessionId][SessionId]、[PartitionKey][PartitionKey] 或 [MessageId][MessageId])，該分割索引鍵會用於目的地實體。

## <a name="considerations-and-guidelines"></a>考量和指導方針
* **高度一致性功能**︰如果實體使用工作階段、重複偵測或明確控制資料分割區索引鍵等功能，則傳訊作業一定會路由至特定的片段。 如果任何片段遇到過高的流量，或基礎存放區的狀況不良，這些作業將會失敗，而且可用性會降低。 整體來說，一致性仍然遠高於非分割實體，只有一部分流量會遭遇問題，而不是所有的流量。
* **管理**︰必須在實體的所有片段上執行建立、更新及刪除等作業。 如果任何片段的狀況不良，可能會造成這些作業失敗。 以「取得」作業來說，必須彙總來自所有片段的資訊，例如訊息計數。 如果任何片段的狀況不良，則實體可用性狀態會報告為受限制。
* **少量訊息案例**︰對於這類案例，尤其是當使用 HTTP 通訊協定時，您可能必須執行多次接收作業，才能取得所有訊息。 對於接收要求，前端會在所有片段上執行接收，並快取所有收到的回應。 相同連接上的後續接收要求將受益於此快取，而且接收延遲將會縮短。 不過，如果您有多個連線或使用 HTTP，則會針對每個要求建立新的連接。 因此，不保證抵達相同的節點。 如果所有現有的訊息遭鎖定，而且在另一個前端中快取，接收作業會傳回 **null**。 訊息最後會到期，您可以再次接收它們。 建議使用 HTTP 持續作用。
* **瀏覽/查看訊息**：[PeekBatch](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_PeekBatch_System_Int32_) 不一定會傳回 [MessageCount](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queuedescription#Microsoft_ServiceBus_Messaging_QueueDescription_MessageCount) 屬性中指定的訊息數目。 這有兩個常見的原因。 其中一個原因是訊息集合的彙總大小超過大小上限 256KB。 另一個原因是，如果佇列或主題的 [EnablePartitioning 屬性](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queuedescription#Microsoft_ServiceBus_Messaging_QueueDescription_EnablePartitioning)設為 **true**，分割區可能沒有足夠的訊息來完成所要求的訊息數目。 一般而言，如果應用程式想要接收一定數目的訊息，它應該重複呼叫 [PeekBatch](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_PeekBatch_System_Int32_)，直到取得該數目的訊息，或已沒有更多訊息可查看為止。 如需詳細資訊，包括程式碼範例，請參閱 [QueueClient.PeekBatch](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_PeekBatch_System_Int32_) 或 [SubscriptionClient.PeekBatch](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.subscriptionclient#Microsoft_ServiceBus_Messaging_SubscriptionClient_PeekBatch_System_Int32_)。

## <a name="latest-added-features"></a>最新加入的功能
* 分割實體現在支援新增或移除規則。 不同於非分割實體，交易情況下不支援這些作業。 
* AMQP 現在支援往返於分割實體傳送和接收訊息。
* AMQP 現在支援下列作業：[批次傳送](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_SendBatch_System_Collections_Generic_IEnumerable_Microsoft_ServiceBus_Messaging_BrokeredMessage__)、[批次接收](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_ReceiveBatch_System_Int32_)、[依序號接收](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_Receive_System_Int64_)、[查看](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_Peek)、[更新鎖定](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_RenewMessageLock_System_Guid_)、[排定訊息](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_ScheduleMessageAsync_Microsoft_ServiceBus_Messaging_BrokeredMessage_System_DateTimeOffset_)、[取消排定的訊息](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_CancelScheduledMessageAsync_System_Int64_)、[新增規則](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.ruledescription)、[移除規則](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.ruledescription)、[工作階段更新鎖定](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.messagesession#Microsoft_ServiceBus_Messaging_MessageSession_RenewLock)、[設定工作階段狀態](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.messagesession#Microsoft_ServiceBus_Messaging_MessageSession_SetState_System_IO_Stream_)、[取得工作階段狀態](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.messagesession#Microsoft_ServiceBus_Messaging_MessageSession_GetState)和[列舉工作階段](https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queueclient#Microsoft_ServiceBus_Messaging_QueueClient_GetMessageSessionsAsync)。

## <a name="partitioned-entities-limitations"></a>分割實體限制
目前服務匯流排會為分割的佇列和主題設定下列限制：

* 分割的佇列及主題不支援在單一交易中傳送屬於不同工作階段的訊息。
* 服務匯流排目前每個命名空間允許最多 100 個分割佇列或主題。 每個分割佇列或主題的配額計數為每個命名空間 10,000 個實體 (不適用於高階層)。

## <a name="next-steps"></a>後續步驟
請參閱[適用於服務匯流排分割的佇列和主題的 AMQP 1.0 支援][AMQP 1.0 support for Service Bus partitioned queues and topics]，深入了解分割訊息實體。 

[Service Bus architecture]: service-bus-architecture.md
[Azure portal]: https://portal.azure.com
[QueueDescription.EnablePartitioning]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queuedescription#Microsoft_ServiceBus_Messaging_QueueDescription_EnablePartitioning
[TopicDescription.EnablePartitioning]: https://docs.microsoft.com/en-us/dotnet/api/microsoft.servicebus.messaging.topicdescription#Microsoft_ServiceBus_Messaging_TopicDescription_EnablePartitioning
[BrokeredMessage.SessionId]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_SessionId
[BrokeredMessage.PartitionKey]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_PartitionKey
[SessionId]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_SessionId
[PartitionKey]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_PartitionKey
[QueueDescription.RequiresDuplicateDetection]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queuedescription#Microsoft_ServiceBus_Messaging_QueueDescription_RequiresDuplicateDetection
[BrokeredMessage.MessageId]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_MessageId
[MessageId]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.brokeredmessage#Microsoft_ServiceBus_Messaging_BrokeredMessage_MessageId
[MessagingFactorySettings.OperationTimeout]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.messagingfactorysettings#Microsoft_ServiceBus_Messaging_MessagingFactorySettings_OperationTimeout
[OperationTimeout]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.messagingfactorysettings#Microsoft_ServiceBus_Messaging_MessagingFactorySettings_OperationTimeout
[QueueDescription.ForwardTo]: https://docs.microsoft.com/dotnet/api/microsoft.servicebus.messaging.queuedescription#Microsoft_ServiceBus_Messaging_QueueDescription_ForwardTo
[AMQP 1.0 support for Service Bus partitioned queues and topics]: service-bus-partitioned-queues-and-topics-amqp-overview.md



<!--HONumber=Dec16_HO4-->


