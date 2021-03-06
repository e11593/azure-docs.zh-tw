---
title: "服務匯流排和 Java 與 AMQP 1.0 | Microsoft Docs"
description: "搭配使用 Java 的服務匯流排與 AMQP"
services: service-bus-messaging
documentationcenter: na
author: sethmanheim
manager: timlt
editor: 
ms.assetid: 1848f58d-1b40-43e0-89c4-ab14ba34c032
ms.service: service-bus-messaging
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/29/2016
ms.author: sethm
translationtype: Human Translation
ms.sourcegitcommit: 219dcbfdca145bedb570eb9ef747ee00cc0342eb
ms.openlocfilehash: eb41df70551bc327473caea4fab95cd269bc0175


---
# <a name="use-service-bus-from-java-with-amqp-10"></a>搭配使用 Java 的服務匯流排與 AMQP 1.0
[!INCLUDE [service-bus-selector-amqp](../../includes/service-bus-selector-amqp.md)]

Java 訊息服務 (JMS) 是在 Java 平台上搭配訊息導向中介軟體一起使用的標準 API。 我們已根據 Apache Qpid 專案開發的 JMS 用戶端程式庫，使用 AMQP 1.0 測試過 Microsoft Azure 服務匯流排。 此程式庫支援完整的 JMS 1.1 API，可以搭配 AMQP 1.0 相容的任何訊息服務一起使用。 在 [Windows Server 服務匯流排](https://msdn.microsoft.com/library/dn282144.aspx) (內部部署服務匯流排) 中也支援這種情況。 如需詳細資訊，請參閱 [Windows Server 服務匯流排中的 AMQP][Windows Server 服務匯流排中的 AMQP]。

## <a name="download-the-apache-qpid-amqp-10-jms-client-library"></a>下載 Apache Qpid AMQP 1.0 JMS 用戶端程式庫
如需有關下載最新版 Apache Qpid JMS AMQP 1.0 用戶端程式庫的資訊，請造訪 [http://people.apache.org/~rgodfrey/qpid-java-amqp-1-0-client-jms.html](http://people.apache.org/~rgodfrey/qpid-java-amqp-1-0-client-jms.html)。

您使用服務匯流排建立和執行 JMS 應用程式時，必須從 Apache Qpid JMS AMQP 1.0 散發封裝將下列 4 個 JAR 檔加入 Java CLASSPATH：

* geronimo-jms\_1.1\_spec-[version].jar
* qpid-amqp-1-0-client-[version].jar
* qpid-amqp-1-0-client-jms-[version].jar
* qpid-amqp-1-0-common-[version].jar

## <a name="work-with-service-bus-queues-topics-and-subscriptions-from-jms"></a>從 JMS 使用服務匯流排佇列、主題和訂用帳戶
### <a name="java-naming-and-directory-interface-jndi"></a>Java 命名及目錄介面 (JNDI)
JMS 使用 Java 命名及目錄介面 (JNDI) 建立邏輯名稱與實際名稱之間的區別。 使用 JNDI 可以解析兩種 JMS 物件：**ConnectionFactory** 和 **Destination**。 JNDI 使用提供者模型，您可以在其中插入不同的目錄服務處理名稱解析作業。 Apache Qpid JMS AMQP 1.0 程式庫隨附一個簡單的屬性檔 JNDI 提供者，可使用文字檔來設定。

Qpid 屬性檔 JNDI 提供者是透過下列格式的屬性檔來設定：

```
# servicebus.properties – sample JNDI configuration

# Register a ConnectionFactory in JNDI using the form:
# connectionfactory.[jndi_name] = [ConnectionURL]
connectionfactory.SBCONNECTIONFACTORY = amqps://[username]:[password]@[namespace].servicebus.windows.net

# Register some queues in JNDI using the form
# queue.[jndi_name] = [physical_name]
# topic.[jndi_name] = [physical_name]
topic.TOPIC = topic1
queue.QUEUE = queue1
```

#### <a name="configure-the-connection-factory"></a>設定連接工廠
在 Qpid 內容檔案 JNDI 提供者中用來定義 **ConnectionFactory** 的項目使用下列格式：

```
connectionfactory.[jndi_name] = [ConnectionURL]
```

其中 `[jndi\_name]` 和 `[ConnectionURL]` 具有下列意義：

| 名稱 | 意義 |  |  |  |  |
| --- | --- | --- | --- | --- | --- |
| `[jndi\_name]` |連接工廠的邏輯名稱。 Java 應用程式中使用 JNDI `IntialContext.lookup()` 方法解析這個名稱。 | | | | |
| `[ConnectionURL]` |將包含所需資訊的 JMS 程式庫提供給 AMQP 代理人的 URL。 | | | | |

連接 URL 的格式如下：

```
amqps://[username]:[password]@[namespace].servicebus.windows.net
```

其中 `[namespace]`、`[username]` 和 `[password]` 具有下列意義：

| 名稱 | 意義 |  |  |  |  |
| --- | --- | --- | --- | --- | --- |
| `[namespace]` |從 [Azure 入口網站][Azure 入口網站]取得的服務匯流排命名空間。 | | | | |
| `[username]` |從 [Azure 入口網站][Azure 入口網站]取得的服務匯流排 SAS 金鑰名稱。 | | | | |
| `[password]` |從 [Azure 入口網站][Azure 入口網站]取得的服務匯流排 SAS 金鑰，為 URL 編碼格式。 | | | | |

> [!NOTE]
> 您必須手動使用 URL 將密碼編碼。 [http://www.w3schools.com/tags/ref_urlencode.asp](http://www.w3schools.com/tags/ref_urlencode.asp) 中提供實用的 URL 編碼公用程式。
> 
> 

例如，如果從入口網站取得的資訊如下：

| 命名空間： | test.servicebus.windows.net |
| --- | --- |
| 發行者名稱： |RootManageSharedAccessKey |
| 發行者金鑰： |abcdefg |

然後為了定義名為 `SBCONNECTIONFACTORY` 的 **ConnectionFactory**，組態字串如下：

```
connectionfactory.SBCONNECTIONFACTORY = amqps://RootManageSharedAccessKey:abcdefg@test.servicebus.windows.net
```

#### <a name="configure-destinations"></a>設定目的地
在 Qpid 屬性檔 JNDI 提供者中用來定義目的地的項目使用下列格式：

```
queue.[jndi_name] = [physical_name]
topic.[jndi_name] = [physical_name]
```

其中 `[jndi\_name]` 和 `[physical\_name]` 具有下列意義：

| 名稱 | 意義 |
| --- | --- |
| `[jndi\_name]` |目的地的邏輯名稱。 我們已使用 JNDI `IntialContext.lookup()` 方法在 Java 應用程式中解析此名稱。 |
| `[physical\name]` |應用程式傳送或接收訊息的服務匯流排實體名稱。 |

請注意：

* `[physical\name]` 值可以是服務匯流排佇列或主題。
* 從服務匯流排主題訂用帳戶收到在 JNDI 中指定的實體名稱應該是主題的名稱。 以 JMS 應用程式程式碼建立持續性訂用帳戶時，將建立訂用帳戶名稱。
* 也可以將服務匯流排主題訂用帳戶視為 JMS 佇列。 這種方法有幾個好處：相同的接收者程式碼可用於佇列和主題訂用帳戶，而屬性檔中的所有位址資訊 (主題和訂用帳戶名稱) 都外部化。
* 若要將服務匯流排主題訂用帳戶視為 JMS 佇列，屬性檔案中的項目格式應為：`queue.[jndi\_name] = [topic\_name]/Subscriptions/[subscription\_name]`。|

若要定義名為 "TOPIC" 的邏輯 JMS 目的地來對應至名為 "topic1" 的服務匯流排主題，屬性檔中的項目應該如下：

```
topic.TOPIC = topic1
```

### <a name="send-messages-using-jms"></a>使用 JMS 傳送訊息
下列程式碼示範如何將訊息傳送至服務匯流排主題。 當中假設 `SBCONNECTIONFACTORY` 和 `TOPIC` 在前一節所述的 **servicebus.properties** 組態檔中進行定義。

```
Hashtable<String, String> env = new Hashtable<String, String>(); 
env.put(Context.INITIAL_CONTEXT_FACTORY, 
        "org.apache.qpid.amqp_1_0.jms.jndi.PropertiesFileInitialContextFactory"); 
env.put(Context.PROVIDER_URL, "servicebus.properties"); 

InitialContext context = new InitialContext(env); 

ConnectionFactory cf = (ConnectionFactory) context.lookup("SBCONNECTIONFACTORY");
Topic topic = (Topic) context.lookup("TOPIC");
Connection connection = cf.createConnection();
Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
MessageProducer producer = session.createProducer(topic);
TextMessage message = session.createTextMessage("This is a text string"); 
producer.send(message);
```

### <a name="receive-messages-using-jms"></a>使用 JMS 接收訊息
下列程式碼示範 `how` 從服務匯流排主題訂用帳戶收到訊息。 當中假設 `SBCONNECTIONFACTORY` 和 TOPIC 在前一節所述的 **servicebus.properties** 組態檔中進行定義。 也假設訂用帳戶名稱為 `subscription1`。

```
Hashtable<String, String> env = new Hashtable<String, String>(); 
env.put(Context.INITIAL_CONTEXT_FACTORY, 
        "org.apache.qpid.amqp_1_0.jms.jndi.PropertiesFileInitialContextFactory"); 
env.put(Context.PROVIDER_URL, "servicebus.properties"); 

InitialContext context = new InitialContext(env);

ConnectionFactory cf = (ConnectionFactory) context.lookup("SBCONNECTIONFACTORY");
Topic topic = (Topic) context.lookup("TOPIC");
Connection connection = cf.createConnection();
Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
TopicSubscriber subscriber = session.createDurableSubscriber(topic, "subscription1");
connection.start();
Message message = messageConsumer.receive();
```

### <a name="guidelines-for-building-robust-applications"></a>建置健全應用程式的指導方針
JMS 規格定義如何撰寫 API 方法和應用程式碼的例外狀況合約來處理這類例外狀況。 以下是有關例外狀況處理的一些其他考慮事項：

* 使用 **connection.setExceptionListener** 向 JMS 連接註冊 **ExceptionListener**。 這樣可透過非同步方式向用戶端通知問題。 此通知對於只取用訊息的連接特別重要，因為它們沒有其他方法可得知連接已失敗。 如果基礎 AMQP 連接、工作階段或連結發生問題，將呼叫 **ExceptionListener**。 在此情況下，應用程式應該從頭重新建立 **JMS Connection**、**Session**、**MessageProducer** 和 **MessageConsumer** 物件。
* 若要確認訊息已成功從 **MessageProducer** 傳送至服務匯流排實體，請確定已透過 **qpid.sync\_publish** 系統屬性集設定該應用程式。 作法是啟動應用程式時，透過在命令列上設定的 **-Dqpid.sync\_publish=true** Java VM 選項來啟動程式。 設定此選項可在接獲確認服務匯流排已收到訊息之前，不要讓程式庫從傳送呼叫返回。 如果傳送作業期間發生問題，將引發 **JMSException**。 有兩個可能的原因： 
  
  1. 如果問題是因為服務匯流排拒絕讓特定訊息進行傳送而造成的，便會引發 **MessageRejectedException** 例外狀況。 這個錯誤可能是暫時性，或由於訊息的某些問題而引起。 建議是動作是以某種後退邏輯來多次重試作業。 如果問題持續發生，就應該放棄訊息並在本機記錄錯誤。 在此情況下，不需要重新建立 **JMS Connection**、**Session** 或 **MessageProducer** 物件。 
  2. 如果問題是因為服務匯流排關閉 AMQP 連結而造成的，將引發 **InvalidDestinationException** 例外狀況。 這可能是因為暫時性問題或已刪除訊息實體。 無論是哪一種狀況，都應該重新建立 **JMS Connection**、**Session** 及 **MessageProducer** 物件。 如果錯誤狀況是暫時性，此作業最終還是會成功。 如果已刪除實體，則為永久失敗。

## <a name="messaging-between-net-and-jms"></a>在 .NET 與 JMS 之間傳訊
### <a name="message-bodies"></a>訊息主體
JMS 可定義五種不同的訊息類型：**BytesMessage**、**MapMessage**、**ObjectMessage**、**StreamMessage** 和 **TextMessage**。 服務匯流排 .NET API 只有一種訊息類型：[BrokeredMessage][BrokeredMessage]。

#### <a name="jms-to-service-bus-net-api"></a>JMS 至服務匯流排 .NET API
下列各節顯示如何從 .NET 取用每一種 JMS 訊息類型的訊息。 尚未包含 **ObjectMessage** 範例，因為 **ObjectMessage** 的主體包含採用 Java 程式設計語言中的可序列化物件，但 .NET 應用程式無法解譯此物件。

##### <a name="bytesmessage"></a>BytesMessage
下列程式碼示範如何使用服務匯流排 .NET API 取用 **BytesMessage** 物件的主體。

```
Stream stream = message.GetBody<Stream>();
int streamLength = (int)stream.Length;

byte[] byteArray = new byte[streamLength];
stream.Read(byteArray, 0, streamLength);

Console.WriteLine("Length = " + streamLength);
for (int i = 0; i < stream.Length; i++)
{
  Console.Write("[" + (sbyte) byteArray[i] + "]");
}
```

##### <a name="mapmessage"></a>MapMessage
下列程式碼示範如何使用服務匯流排 .NET API 取用 **MapMessage** 物件的主體。 此程式碼會逐一檢查對應的元素，顯示每個元素的名稱和值。

```
Dictionary<String, Object> dictionary = message.GetBody<Dictionary<String, Object>>();

foreach (String mapItemName in dictionary.Keys)
{
  Object mapItemValue = null;
  if (dictionary.TryGetValue(mapItemName, out mapItemValue))
  {
    Console.WriteLine(mapItemName + ":" + mapItemValue);
  }
}
```

##### <a name="streammessage"></a>StreamMessage
下列程式碼示範如何使用服務匯流排 .NET API 取用 **StreamMessage** 物件的主體。 此程式碼會列出串流中的每個項目及其類型。

```
List<Object> list = message.GetBody<List<Object>>();

foreach (Object item in list)
{
  Console.WriteLine(item + " (" + item.GetType() + ")");
}
```

##### <a name="textmessage"></a>TextMessage
下列程式碼示範如何使用服務匯流排 .NET API 取用 **TextMessage** 物件的主體。 此程式碼會顯示訊息主體所包含的文字字串。

```
Console.WriteLine("Text: " + message.GetBody<String>());
```

#### <a name="service-bus-net-apis-to-jms"></a>服務匯流排 .NET API 至 JMS
下列各節以每一種不同的 JMS 訊息類型示範 .NET 應用程式如何建立 JMS 中接收的訊息。 尚未包含 **ObjectMessage** 範例，因為 **ObjectMessage** 的主體包含採用 Java 程式設計語言中的可序列化物件，但 .NET 應用程式無法解譯此物件。

##### <a name="bytesmessage"></a>BytesMessage
下列程式碼示範如何在 .NET 中建立 [BrokeredMessage][BrokeredMessage] 物件 (以 **BytesMessage** 形式傳送給 JMS 用戶端)。

```
byte[] bytes = { 33, 12, 45, 33, 12, 45, 33, 12, 45, 33, 12, 45 };
message = new BrokeredMessage(bytes);
```

##### <a name="streammessage"></a>StreamMessage
下列程式碼示範如何在 .NET 中建立 [BrokeredMessage][BrokeredMessage] 物件 (以 **StreamMessage** 形式傳送給 JMS 用戶端)。

```
List<Object> list = new List<Object>();
list.Add("String 1");
list.Add("String 2");
list.Add("String 3");
list.Add((double)3.14159);
message = new BrokeredMessage(list);
```

##### <a name="textmessage"></a>TextMessage
下列程式碼示範如何使用服務匯流排 .NET API 取用 **TextMessage** 的主體。 此程式碼會顯示訊息主體所包含的文字字串。

```
message = new BrokeredMessage("this is a text string");
```

### <a name="application-properties"></a>應用程式屬性
#### <a name="jms-to-service-bus-net-apis"></a>JMS 至服務匯流排 .NET API
JMS 訊息支援下列類型的應用程式屬性︰**boolean**、**byte**、**short**、**int**、**long**、**float**、**double** 和 **String**。 下列 Java 程式碼使用以上每一種屬性類型示範如何設定訊息的屬性。

```
message.setBooleanProperty("TestBoolean", true); 
message.setByteProperty("TestByte", (byte) 33); 
message.setDoubleProperty("TestDouble", 3.14159D); 
message.setFloatProperty("TestFloat", 3.13159F); 
message.setIntProperty("TestInt", 100); 
message.setStringProperty("TestString", "Service Bus");
```

在服務匯流排 .NET API 中，訊息應用程式屬性包含在 [BrokeredMessage][BrokeredMessage] 的 **Properties** 集合中。 下列程式碼示範如何讀取從 JMS 用戶端接收之訊息的應用程式屬性。

```
if (message.Properties.Keys.Count > 0)
{
  foreach (string name in message.Properties.Keys)
  {
    Object value = message.Properties[name];
    Console.WriteLine(name + ": " + value + " (" + value.GetType() + ")" );
  }
  Console.WriteLine();
}
```

下表顯示 JMS 屬性類型如何對應至 .NET 屬性類型。

| JMS 屬性類型 | .NET 屬性類型 |
| --- | --- |
| Byte |sbyte |
| Integer |int |
| Float |float |
| Double |double |
| Boolean |布林 |
| String |string |

[BrokeredMessage][BrokeredMessage] 類型支援下列類型的應用程式屬性：**byte****sbyte**、**char**、**short**、**ushort**、**int**、**uint**、**long**、**ulong**、**float****double****decimal****bool****Guid****string****Uri****DateTime****DateTimeOffset** 和 **TimeSpan**。 下列 .NET 程式碼示範如何使用每一個屬性類型來設定 [BrokeredMessage][BrokeredMessage] 物件的屬性。

```
message.Properties["TestByte"] = (byte)128;
message.Properties["TestSbyte"] = (sbyte)-22;
message.Properties["TestChar"] = (char) 'X';
message.Properties["TestShort"] = (short)-12345;
message.Properties["TestUshort"] = (ushort)12345;
message.Properties["TestInt"] = (int)-100;
message.Properties["TestUint"] = (uint)100;
message.Properties["TestLong"] = (long)-12345;
message.Properties["TestUlong"] = (ulong)12345;
message.Properties["TestFloat"] = (float)3.14159;
message.Properties["TestDouble"] = (double)3.14159;
message.Properties["TestDecimal"] = (decimal)3.14159;
message.Properties["TestBoolean"] = true;
message.Properties["TestGuid"] = Guid.NewGuid();
message.Properties["TestString"] = "Service Bus";
message.Properties["TestUri"] = new Uri("http://www.bing.com");
message.Properties["TestDateTime"] = DateTime.Now;
message.Properties["TestDateTimeOffSet"] = DateTimeOffset.Now;
message.Properties["TestTimeSpan"] = TimeSpan.FromMinutes(60);
```

下列 Java 程式碼示範如何讀取從服務匯流排 .NET 用戶端接收之訊息的應用程式屬性。

```
Enumeration propertyNames = message.getPropertyNames(); 
while (propertyNames.hasMoreElements()) 
{ 
  String name = (String) propertyNames.nextElement(); 
  Object value = message.getObjectProperty(name); 
  System.out.println(name + ": " + value + " (" + value.getClass() + ")"); 
}
```

下表顯示 .NET 屬性類型如何對應至 JMS 屬性類型。

| .NET 屬性類型 | JMS 屬性類型 | 注意事項 |
| --- | --- | --- |
| byte |UnsignedByte |- |
| sbyte |位元組 |- |
| char |Character |- |
| short |簡短 |- |
| ushort |UnsignedShort |- |
| int |Integer |- |
| uint |UnsignedInteger |- |
| long |long |- |
| ulong |UnsignedLong |- |
| float |Float |- |
| double |Double |- |
| 十進位 |BigDecimal |- |
| 布林 |Boolean |- |
| Guid |UUID |- |
| 字串 |String |- |
| DateTime |日期 |- |
| Datetimeoffset |DescribedType |DateTimeOffset.UtcTicks 會對應至 AMQP 類型：<type name=”datetime-offset” class=restricted source=”long”> <descriptor name=”com.microsoft:datetime-offset” /></type> |
| TimeSpan |DescribedType |Timespan.Ticks 會對應至 AMQP 類型：<type name=”timespan” class=restricted source=”long”> <descriptor name=”com.microsoft:timespan” /></type> |
| Uri |DescribedType |Uri.AbsoluteUri 會對應至 AMQP 類型：<type name=”uri” class=restricted source=”string”> <descriptor name=”com.microsoft:uri” /></type> |

### <a name="standard-headers"></a>標準標頭
下表顯示如何使用 AMQP 1.0 對應 JMS 標準標頭和 [BrokeredMessage][BrokeredMessage] 標準屬性。

#### <a name="jms-to-service-bus-net-apis"></a>JMS 至服務匯流排 .NET API
| JMS | 服務匯流排 .NET | 注意事項 |
| --- | --- | --- |
| JMSCorrelationID |Message.CorrelationID |- |
| JMSDeliveryMode |目前無法使用 |不論指定何者，服務匯流排僅支援持久的訊息；例如 DeliveryMode.PERSISTENT。 |
| JMSDestination |Message.To |- |
| JMSExpiration |Message. TimeToLive |轉換 |
| JMSMessageID |Message.MessageID |根據預設，JMSMessageID 在 AMQP 訊息中會以二進位格式編碼。 收到二進位訊息識別碼時，.NET 用戶端程式庫會根據位元組的 unicode 值，將它轉換為字串表示法。 若要將 JMS 程式庫切換為使用字串訊息識別碼，請將 “binary-messageid=false” 字串附加至 JNDI ConnectionURL 的查詢參數。 例如：“amqps://[username]:[password]@[namespace].servicebus.windows.net? binary-messageid=false”。 |
| JMSPriority |目前無法使用 |服務匯流排不支援訊息優先順序。 |
| JMSRedelivered |目前無法使用 |- |
| JMSReplyTo |Message. ReplyTo |- |
| JMSTimestamp |Message.EnqueuedTimeUtc |轉換 |
| JMSType |Message.Properties[“jms-type”] |- |

#### <a name="service-bus-net-apis-to-jms"></a>服務匯流排 .NET API 至 JMS
| 服務匯流排 .NET | JMS | 注意事項 |
| --- | --- | --- |
| ContentType |- |目前無法使用 |
| CorrelationId |JMSCorrelationID |- |
| EnqueuedTimeUtc |JMSTimestamp |轉換 |
| 標籤 |n/a |目前無法使用 |
| MessageId |JMSMessageID |- |
| ReplyTo |JMSReplyTo |- |
| ReplyToSessionId |n/a |目前無法使用 |
| ScheduledEnqueueTimeUtc |n/a |目前無法使用 |
| SessionId |n/a |目前無法使用 |
| TimeToLive |JMSExpiration |轉換 |
| 收件人 |JMSDestination |- |

## <a name="unsupported-features-and-restrictions"></a>不支援的功能和限制
JMS over AMQP 1.0 和服務匯流排一起使用時有下列限制：

* 對於各個工作階段僅允許一個 **MessageProducer** 或 **MessageConsumer**。 如果您想要在應用程式中建立多個 **MessageProducer** 或 **MessageConsumer** 物件，請分別建立專用的工作階段。
* 目前不支援可變更的主題訂閱。
* 不支援 **MessageSelector** 物件。
* 不支援 **TemporaryQueue** 或 **TemporaryTopic** 這些暫時目的地，也不支援使用它們的 **QueueRequestor** 和 **TopicRequestor** API。
* 不支援交易式工作階段。
* 不支援分散式工作階段。

## <a name="next-steps"></a>後續步驟
準備好進行深入了解嗎？ 請造訪下列連結：

* [服務匯流排 AMQP 概觀]
* [Windows Server 服務匯流排中的 AMQP]

[Windows Server 服務匯流排中的 AMQP]: https://msdn.microsoft.com/library/dn574799.aspx
[BrokeredMessage]: https://msdn.microsoft.com/library/azure/microsoft.servicebus.messaging.brokeredmessage.aspx

[服務匯流排 AMQP 概觀]: service-bus-amqp-overview.md
[Azure 入口網站]: https://portal.azure.com



<!--HONumber=Nov16_HO3-->


