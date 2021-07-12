---
title: "Java: Compartion Violation 問題小記"
date: 2021-07-11T23:58:09+08:00
Tags: ["java"]
Categories: ["note"]
---

前陣子和朋友討論一個奇妙的錯誤訊息，說在執行 `Collections.sort(list)` 時會出現。第一眼看到這個錯誤訊息，腦中只冒出無限多個問號，心中只有 OS：「這些工程師什麼時候才能好好說句人話...」(~~話說你自己不也是工程師嘛~~)，沒辦法只好請出 Google 大神，誰叫我們都是 Google 工程師嘛。
原來在 JDK 1.7 後引入了 Tim Sort 這個排序法，Tim Sort 是 Merge Sort 的變型但更有效率，至於 Tim Sort 和 Merge Sort 具體的排序細節為何則不在本文範圍內(~~其實是我自己也還沒搞清楚 Tim Sort…~~)。
雖然比較有效率，但對於排序時如何比較被排序元素大小的邏輯正確性就更要求，要能使用 `Collections.sort` 的元素必須實作 `Comparable` 的 `toCompare` method，根據 [API 說明](https://docs.oracle.com/javase/7/docs/api/java/lang/Comparable.html#compareTo(T))，實作 `compareTo` 時，比較元素大小的邏輯必須符合下列原則：

1. 相反性： A > B 時，必須符合 -A < B
2. 遞延性：A > B 且 B > C 時，必須符合 A > C
3. 必須符合 A == B 時，A == C 且 B == C

如果有違反這 3 個原則的話，有可能會在排序過程中拋出 `IllegalArgumentException: Comparison method violates its general contract!` 的錯誤訊息，但這邊要注意的是也只是有可能，有時候會在毫無錯誤的情況下完成排序，但此時也別高興得太早，古語有云：沉默錯誤最可怕…(~~我自己說的…~~)，在違反排序原則的情況排出來的結果，可想而知排序的正確性是有問題，但可怕的是我們這時候反而不知道啊，所以知錯能改善莫大焉，你看說程式還能學到人生道理，多划算 😆。
知道出現錯誤的可能原因後，下一步就是如何找出邏輯的問題，這一點其實滿麻煩的，因為實作的邏輯可能有顯而易見的錯誤，但也可能要看傳入的資料而定，並不是任何資料都有可能讓排序邏輯違反原則，像這次遇到問題的排序邏輯：

```java
// Element.val 是 String
public int compareTo(Elment e) {
    if (e == null || e.val == null) {
        return -1;
    }
    if (this.val == null) {
        return 1;
    }
    if (this.val.matches("\\\\+d") && this.val.matches("\\\\+d")) {
        return Intger.parseInt(this.val) - Integer.parseInt(e.val);
    }

    return this.val.compareTo(e.val);
}

```

我們來分析一下，首先

- 如果和此元素比較的元素 `e` 是 `null` 或 `e` 的 `val` 是 `null` 就回傳 `1` 表示 `e` > 此元素
- 如果此元素比較的元素 `e` 和 `e.val` 非 `null`，但此元素的 `val` 為 `null` 則回傳 `1` 表示 `e` < 此元素

上面這兩個邏輯處理了元素的 `val` 為 `null` 或者一起比較的元素 `e` 為 `null` 或 `e.val` 為 `null` 的狀況，從回傳值可以看出這個邏輯是想把 `null` 值往後排序。

那如果此元素的 `val` 和 `e.val` 都有值呢? 這時候就套用下面的邏輯：

- 當此元素的 `val` 和 `e.val` 都是純數字的字串時則以轉換成數字比大小
- 而如果有一邊的 `val` 字串不是純數字則進直接呼叫 `String.compareTo` 以字串的方式比較。

這幾個邏輯各別來看都沒什麼問題，數字就用數字比大小，有牽涉到字串就改用字串方式比較，也有考慮到 `null` 的情況，但是既然有這篇文章當然就是出事啦。
如果您已經看出問題了，那我真心佩服，在當下想了很久但把除錯的目標集中在錯誤的 `null` 處理邏輯段落，直到 YouTube 上剛好看到某埸演講主題就是 Java 的排序問題，講者提供了一個方式來 debug，在 `compareTo` 裡將元素值印出來，直到發生錯誤而停下來的前一刻所印出來的數值有可能就是觸發異常邏輯的資料：

```java
// Element.val 是 String
public int compareTo(Elment e) {
    System.out.println("this: " + null + ", e: " + (e == null ? "null" : e));
    System.out.println("this.val: " + this.val + ", e.val: " + e.val);
    if (e == null || e.val == null) {
        return -1;
    }
    if (this.val == null) {
        return 1;
    }
    if (this.val.matches("\\\\+d") && this.val.matches("\\\\+d")) {
        return Intger.parseInt(this.val) - Integer.parseInt(e.val);
    }

    return this.val.compareTo(e.val);
}

```

當看到印出來的訊息時，突然間謎題全部都解開了(請下 BGM)

<div class="img-wrapper">
    <img src="../../img/blog/buster.jpg" />
    <div class="cp">
    我就是想用到這張圖
    </div>
</div>

最終抓到的問題資料是 `8`、`19`、`5Z`(為保護機密，此資料非實際當事資料)，這三個值有什麼問題呢? 我們來分別套用到排序邏輯上看看。
首先是 `8` 和 `19`，這兩個字串都是純數字，所以我們會轉成 `int` 後比較大小，得到 `8 < 19`。
接著是 `8` 和 `5Z`，`5Z` 是字串，因此我們會用 `"8".compareTo("5Z")` 的方式比大小，在 Java 中字串的排序是從最前面的字元開始比較它轉成 Unicode 後的數值，而 `"8" > "5"` 所以得到 `8 > 5Z`。
最後是 `5Z` 和 `19`，同樣的這裡用字串的方式排序，因為 `"5" > "1"` 所以得到 `5Z > 19`。
我們把三個結果擺一起看看：

- `8 < 19`
- `8 > 5Z`
- `19 < 5Z`
結果明顯是矛盾的，如果 `8 < 19` 且 `19 < 5Z` 那應該是 `8 < 5z` 才對，所以這個排序邏輯違反了第一項原則。

經過兩個晚上的奮鬥運氣不錯地找到了實作邏輯上的錯誤，在 debug 的過程中很神奇的是同樣的資料在我這邊的測試程式中不會出現錯誤，但最終的排序結果看起來滿奇怪的，也印證了開頭提到的，如果沒有遇到錯誤不一定是好事，因為排序結果可能不是你想要的樣子。
因為之前很少自己實作排序邏輯，通常直接仰賴原生的排序邏輯或者直接在 SQL 中就先排好，所以也是第一次遇上這個問題，現在比較了解在排序實作上必須注意的點，要考慮好 `null` 的情況還有三原則的部分，另外非常推薦去看看文中提到的演影片，講者示範了很多排序的方式和可能出現的實作錯誤，連結在下方。

參考資料：

- [Comparison Method Violates Its General Contract! (Part 1) by Stuart Marks](https://youtu.be/Enwbh6wpnYs)
- [Comparison Method Violates Its General Contract! (Part 2) by Stuart Marks](https://youtu.be/bvnmbRo7a1Y)

如果有什麼想法或需要指正的地方，歡迎您留言或來信 😄