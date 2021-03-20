---
title: "Cache Policy"
date: 2020-12-08T23:40:40+08:00
draft: true
Tags: []
Categories: []
math: true
DisableComments: false
---

# Cache 機制

快取 (Cache) 是在儲存資料和程式中間做為一個暫存式的區塊存放暫時資料，這樣做的好處是免除每次都必須在儲存資料的地方存取資料以加快處理速度和減少儲存資料處的 IO 處理。

以 web application 來說，常見的作法就是在程式端和資料庫中間建立像 Redis 這類的 cache server，當程式要讀取某特定資料時會先到 Redis 中尋找，如果沒有找到 (稱為 cache miss) 再在資料庫中查找，但如果資料已存在於 Redis 中 (稱為 cache hit) 就直接回傳資料，這樣就可以減少到資料庫查詢的 IO 消耗。

在 cache miss 的情況下，除了從資料庫查詢資料回傳給程式端外還必須把資料寫入 Redis 中，這樣下次同樣的查詢就可以從 Redis 取得資料。

而在寫入資料時就涉及到一個問題：我們寫入資料到 Redis 後，在什麼時間點要把資料也寫到資料庫中？

主要的做法有兩種：
1. **write-through**：當寫入資料到 Redis 時同步也寫到資料庫中
2. **write-back** (或 **write-behind**)：寫入資料到 Redis 後，在特定時間後再寫到資料庫中