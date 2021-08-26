---
title: "PubSub Local Emulator"
date: 2021-08-26T23:38:30+08:00
tag: ["GCP"]
catory: ["note"]
---

PubSub 是 GCP 上類似 Message Queue 的服務，為了測試方便，Google 提供了可在本機端啟動的模擬程式讓開發者可以不用連上 GCP 直接使用本機的 PubSub service。

這篇算是記錄一下整個安裝過程和使用 Java 程式連接本機 PubSub 的程式片段。

### 安裝 Google Cloud SDK
首先需要安裝 Google Cloud SDK，目前只有在 MAC 上安裝，所以下面只簡單記錄一下 MAC 下的安裝步驟，其他環境可以到 [Google Cloud SDK Guide](https://cloud.google.com/sdk/docs/install) 有詳細的說明。

如果 Mac 是 M1 版本下載 [google-cloud-sdk-354.0.0-darwin-arm.tar.gz](https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-354.0.0-darwin-arm.tar.gz)，非 M1 則下載 [google-cloud-sdk-354.0.0-darwin-x86_64.tar.gz](https://dl.google.com/dl/cloudsdk/channels/rapid/downloads/google-cloud-sdk-354.0.0-darwin-x86_64.tar.gz)，解壓縮後執行：
```bash
./google-cloud-sdk/install.sh && ./google-cloud-sdk/bin/gcloud init
```

### 安裝 PubSub emulator
執行：
```bash
gcloud components install pubsub-emulator
gcloud components update
```

### 啟動 PubSub emulator
執行，`PUBSUB_PROJECT_ID` 可以自由命名，在後面連線到 PubSub 時會用到。另外 PubSub 每次啟動時的 listen port 是隨機的，為了方便起見，我們用 `--host-port=0.0.0.0:8085` 指定 listen port 為 8085：
```bash
gcloud beta emulators pubsub start --host-port=0.0.0.0:8085 --project=PUBSUB_PROJECT_ID
```

接著設定環境變數 `PUBSUB_EMULATOR_HOST`，執行：
```bash
$(gcloud beta emulators pubsub env-init)
```
會自動取得 PubSub 啟動時的 listen port 設為環境變數。

這邊為了方便寫了簡單的 script 一次做完並將 PubSub 的 log 輸出到檔案中：
```bash
#!/bin/sh
gcloud beta emulators pubsub start --host-port=0.0.0.0:8085 --project=PUBSUB_PROJECT_ID 2&1> ~/pubsub.log
echo "Set PUBSUB_EMULATOR_HOST"
# 這裡不確定原因但用 $(gcloud beta emulators pubsub env-init) 無法正常設置環境變數。因為我們已經指定了 listen port，所以直接用 export 指定即可
export PUBSUB_EMULATOR_HOST=0.0.0.0:8085
```