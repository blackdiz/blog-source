---
title: "從 Server 端輸出下載的檔名含有中文會變成亂碼問題"
date: 2021-03-22T15:15:46+08:00
Tags: ["backend", "server side"]
Categories: ["note"]
DisableComments: false
---
## 從 Server 端輸出下載的檔名含有中文會變成亂碼問題

朋友遇到一個問題是如果檔名有中文，在瀏覽器下載時檔名會變成亂碼。

首先，瀏覽器是用 header 中的 `Content-Disposition=attachment;filename=${檔名}` 做為預設的下載檔名，但 header 中並不支援 UTF-8 編碼，所以如果在程式中直接拿中文檔名放在 `filename` 中就會變成亂碼。

解決方法是先把檔名 encode 成 URL-encoded 編碼，在 Java 中可以用 `URLEncoder` 處理：

```java
response.setHeader("Content-Disposition", "attachment;filename=" + URLEncoder.encode(fileName, "UTF-8");
```