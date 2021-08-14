---
title: "Linux 2、1、& 和 >"
date: 2021-07-21T20:28:15+08:00
Tags: ["linux", "server side"]
Categories: ["note"]
---

小記一下 `command 2>&1 > /dev/null` 和 `command > /dev/null 2>&1` 的差別。首先先看 `2>&1` 的意思。在 Linux 的 file descriptor 中，1 表示 stdout (標準輸出) 表示程式執行中輸出訊息的地方，預設為 terminal。而 2 表示 stderr (標準錯誤輸出) 表示程式執行中發生錯誤時輸出錯誤訊息的地方，預設也是 terminal。

`>` 表示把 `>` 前方的輸出都導到 `>` 後方的目標。

而 `>&` 表示把 `>&` 前的 file descriptor 指向 `>&` 後方的 file descriptor，`&` 表示目標是 file descriptor 而不是檔案。

`/dev/null` 是個特殊的目標，任何輸入到它的訊息都會消失不會輸出到任何其他地方。

在了解各部分的作用後，現在把它們組合起來看，`command 2>&1 > /dev/null` 表示執行 command，把 stderr 輸出的目標指向 stdout 目前輸出的目標，因為 stdout 目前指向的目標是 terminal，所以 stderr 指向的目標就變成 terminal，最後 `> /dev/null` 表示把 stdout 輸出的目標指向 `/dev/null`，所以最終執行的結果就是 stderr 會輸出到 terminal 而 stdout 會輸出到 `/dev/null`。
`command > /dev/null 2>&1` 則是執行 command，`> /dev/null` 會把 stdout 輸出目標指向 `/dev/null`，而最後面的 `2>&1` 會再把 stderr 輸出的目標指向目前 stdout 的輸出目標，所以也同樣變成 `/dev/null`，因此最終執行結果會是 stdout 和 stderr 都會輸出到 `/dev/null` 中。

所以結論是 `command 2>&1 > /dev/null` 依然會輸出錯誤訊息到 terminal 上，而 `command > /dev/null 2>&1` 則不會輸出任何訊息。

---
參考資料：
- [Bash One-Liners Explained, Part III: All about redirections](https://catonmat.net/bash-one-liners-explained-part-three)

如果有什麼想法或需要指正的地方，歡迎您留言或來信 😄