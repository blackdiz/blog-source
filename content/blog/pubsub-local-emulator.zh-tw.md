---
title: "GCP Pub/Sub Local Emulator"
date: 2021-08-26T23:38:30+08:00
tags: ["GCP", "Pub/Sub"]
categories: ["note"]
---

Pub/Sub 是 GCP 上類似 Message Queue 的服務，為了測試方便，Google 提供了可在本機端啟動的模擬程式讓開發者可以不用連上 GCP 直接使用本機的 Pub/Sub service。

Pub/Sub 的基本概念是我們在 Pub/Sub 上建立一個 `Topic`，`Topic` 算是訊息通道的概念，接著我們要在程式中建立 `Subscription` 去訂閱這個 `Topic`，每當有訊息進入 `Topic` 時，有訂閱的程式就接收這個訊息進行處理。

`Subscription` 分為 `Pull Subscription` 和 `Push Subscription`，`Pull Subscription` 是程式必須主動去跟 `Topic` 取回訊息來處理，而 `Push Subscription` 則是我們在建立 `Subscription` 時預先指定一個 HTTPS 的 URL，當有訊息進到 `Topic` 時，Pub/Sub 會將訊息推送到這個 URL，更詳細的說明可以參照 [Subscriber overview](https://cloud.google.com/pubsub/docs/subscriber#)。

除了 `Topic` 和 `Subscription`，最後我們需要 `Publisher` 負責產生訊息並傳送到 `Topic` 中，整個架構圖為：

<div class="img-wrapper">
    <img src="../../img/blog/pubsub-many-to-many.svg" />
    <div class="cp">
    Picture from Google Cloud
    </div>
</div>

這篇算是記錄一下整個安裝過程和使用 Java 程式連接本機 Pub/Sub 的程式片段。

### 安裝 Google Cloud SDK
首先需要安裝 Google Cloud SDK，目前只有在 MAC 上安裝，所以下面只簡單記錄一下 MAC 下的安裝步驟，其他環境可以到 [Google Cloud SDK Guide](https://cloud.google.com/sdk/docs/install) 有詳細的說明。

如果 Mac 是 M1 版本下載 [google-cloud-sdk-354.0.0-darwin-arm.tar.gz](https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-354.0.0-darwin-arm.tar.gz)，非 M1 則下載 [google-cloud-sdk-354.0.0-darwin-x86_64.tar.gz](https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-354.0.0-darwin-x86_64.tar.gz)，解壓縮後執行：
```bash
./google-cloud-sdk/install.sh && ./google-cloud-sdk/bin/gcloud init
```

### 安裝 Pub/Sub emulator
執行：
```bash
gcloud components install pubsub-emulator
gcloud components update
```

### 啟動 Pub/Sub emulator
執行，`PUBSUB_PROJECT_ID` 可以自由命名，在後面連線到 Pub/Sub 時會用到。另外 Pub/Sub 每次啟動時的 listen port 是隨機的，為了方便起見，我們用 `--host-port=0.0.0.0:8085` 指定 listen port 為 8085：
```bash
gcloud beta emulators pubsub start --host-port=0.0.0.0:8085 --project=PUBSUB_PROJECT_ID
```

接著設定環境變數 `PUBSUB_EMULATOR_HOST`，執行：
```bash
$(gcloud beta emulators pubsub env-init)
```
會自動取得 Pub/Sub 啟動時的 listen port 設為環境變數。

這邊為了方便寫了簡單的 script 一次做完並將 Pub/Sub 的 log 輸出到檔案中：
```bash
#!/bin/bash
# 將 log 輸出到 ~/pubsub.log，可以依需要自行修改路徑
gcloud beta emulators pubsub start --host-port=0.0.0.0:8085 --project=PUBSUB_PROJECT_ID >> ~/pubsub.log 2>&1 &

echo "Set PUBSUB_EMULATOR_HOST"

# 這裡不確定原因但用 $(gcloud beta emulators pubsub env-init) 無法正常設置環境變數。因為我們已經指定了 listen port，所以直接用 export 指定即可
export PUBSUB_EMULATOR_HOST=0.0.0.0:8085
```

### 使用 Java 程式連線
建立 `Topic` 和 `Subscription`
```java
 // 從環境變數 PUBSUB_EMULATOR_HOST 取得 Pub/Sub 的 listen port
String hostport = System.getenv("PUBSUB_EMULATOR_HOST");
ManagedChannel channel = ManagedChannelBuilder.forTarget(hostport).usePlaintext().build();
TransportChannelProvider channelProvider = FixedTransportChannelProvider.create(GrpcTransportChannel.create(channel));

CredentialsProvider credentialsProvider = NoCredentialsProvider.create();
TopicAdminClient topicClient =
        TopicAdminClient.create(
                TopicAdminSettings.newBuilder()
                        .setTransportChannelProvider(channelProvider)
                        .setCredentialsProvider(credentialsProvider)
                        .build());

// 這裡的 PUBSUB_PROJECT_ID 即之前啟動 Pub/Sub 時設定的 PUBSUB_PROJECT_ID，而 PUBSUB_TOPIC_ID 則是 Topic 的 id 可以自行設定不同名稱
TopicName topicName = TopicName.of(PUBSUB_PROJECT_ID, PUBSUB_TOPIC_ID);

// 先確定同樣 id 的 Topic 未建立過再建立新的 Topic，否則就重用之前建立 Topic 就好
Topic topic = topicClient.getTopic(topicName);
if (topic == null) {
    topic = topicClient.createTopic(topicName);
}

SubscriberStubSettings subscriberStubSettings = SubscriberStubSettings.newBuilder()
    .setTransportChannelProvider(channelProvider)
    .setCredentialsProvider(credentialsProvider)
    .build();

SubscriptionAdminSettings subscriptionAdminSettings = SubscriptionAdminSettings.newBuilder()
        .setCredentialsProvider(credentialsProvider)
        .setTransportChannelProvider(channelProvider)
        .build();
SubscriptionAdminClient client = SubscriptionAdminClient.create(subscriptionAdminSettings);

// 因為我們要建立的是 Push Subscription，所以要指定 endpoint，這裡的 https://localhost:8080/endpoint 依實際需要修改
PushConfig pushConfig = PushConfig.newBuilder().setPushEndpoint("https://localhost:8080/endpoint").build();

// 同樣的，PUBSUB_PROJECT_ID 為啟動時設定的值，而每個 Subscription 需要指定名稱，這裡用 subscription-name 可依需要自行修改
ProjectSubscriptionName subscriptionName =
        ProjectSubscriptionName.of(PUBSUB_PROJECT_ID, "subscription-name");

client.createSubscription(subscriptionName, topicName, pushConfig, 0);
```

建立 `Publisher` 並傳送訊息：
```java
 // 從環境變數 PUBSUB_EMULATOR_HOST 取得 Pub/Sub 的 listen port
String hostport = System.getenv("PUBSUB_EMULATOR_HOST");
ManagedChannel channel = ManagedChannelBuilder.forTarget(hostport).usePlaintext().build();

// 這裡的 PUBSUB_PROJECT_ID 即之前啟動 Pub/Sub 時設定的 PUBSUB_PROJECT_ID，而 PUBSUB_TOPIC_ID 則要使用剛剛建立 Subscription 時的相同 Topic id
TopicName topicName = ProjectTopicName.of(PUBSUB_PROJECT_ID, PUBSUB_TOPIC_ID);
Publisher publisher = null;

try {
    TransportChannelProvider channelProvider =
            FixedTransportChannelProvider.create(GrpcTransportChannel.create(channel));
    CredentialsProvider credentialsProvider = NoCredentialsProvider.create();

    TopicAdminClient topicClient =
            TopicAdminClient.create(
                    TopicAdminSettings.newBuilder()
                            .setTransportChannelProvider(channelProvider)
                            .setCredentialsProvider(credentialsProvider)
                            .build());

    publisher = Publisher.newBuilder(topicName).setChannelProvider(channelProvider).setCredentialsProvider(credentialsProvider).build();

    // data 即為我們要傳送的訊息
    String data = "test message"
    PubsubMessage pubsubMessage = PubsubMessage.newBuilder().setData(data).build();

    ApiFuture<String> future = publisher.publish(pubsubMessage);

} finally {
    if (publisher != null) {
        // When finished with the publisher, shutdown to free up resources.
        publisher.shutdown();
    }
}
```

以上即是用 Java 連線 Pubsub 傳送訊息的程式片段。

---
參考資料：
- [Google Cloud Pub/Sub Doc](https://cloud.google.com/pubsub/docs/overview)
- [Google Cloud Pub/Sub Local Emulator Doc](https://cloud.google.com/pubsub/docs/emulator)

如果有什麼想法或需要指正的地方，歡迎您留言或來信 😄
