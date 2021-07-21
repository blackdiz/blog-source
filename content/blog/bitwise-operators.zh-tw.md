---
title: "Bitwise Operators"
date: 2021-07-21T20:12:53+08:00
Tags: ["java", "programming"]
Categories: ["note"]
---
以前覺得位元邏輯操作很難記，但昨天仔細看了一下後突然了解到之前沒有特別去理解 `0` 和 `1` 代表的意思而是用死背的方式，所以才容易忘記規則。
其實只要記住 `0 == false`、`1 == true` 就會發現規則其實和程式中的 `&&` 和 `||` 意思相同。

雖然我想其實應該是反過來，先有 `&` (**AND**)、`|` (**OR**) 的定義，程式語言才用 `&&` 和 `||` 來作出 shortcut logic，但對像我這種半路直接學程式語言的人來說，借用程式語言的語義來記憶比較容易。
所以 `&` 和 `&&` 相同，表示要兩邊都為 `true` 才會輸出 `true`，否則為 `false`。而 `|` 和 `||` 相同，只要有一邊為 `true` 就會輸出 `true`：


|    &     |    0     |    1     |
| -------- | -------- | -------- |
|    0     |    0     |    0     |
|    1     |    0     |    1     |


|    \|    |    0     |    1     |
| -------- | -------- | -------- |
|    0     |    0     |    1     |
|    1     |    1     |    1     |

而 `^` (XOR) 表示只有完全符合 OR 也就除了要有一邊是 `true` 以外，另一邊**必須**是 `false` 才會輸出 `true`，而兩邊都是 `true` 的情況下依然輸出 `false`：

|    ^     |    0     |    1     |
| -------- | -------- | -------- |
|    0     |    0     |    1     |
|    1     |    1     |    0     |

如果有什麼想法或需要指正的地方，歡迎您留言或來信 😄