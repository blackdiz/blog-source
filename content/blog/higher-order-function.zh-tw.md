---
title: "Higher Order Function 小記"
date: 2021-08-14T22:33:38+08:00
Tags: ["program", "function programming"]
Categories: ["note"]
---

在 **Functional Programming** (函式程式設計) 的世界中，function (函式) 本身可以接受另一個 function 的做為參數，或是返回一個 function 做為回傳值，這就是所謂 **Higher Order Function**。

以 JavaScript 為例，比方我們想在每次執行 function 時附帶執行 1 個 function，我們可以把要附帶執行的 function 做為參數傳入：
```javascript
let f = function() {
    console.log("I'm callback");
}

// 這裡 callback function 做為參數傳入
function add(a, b, callback) {
    // 所以我們可以在 a + b 之前執行傳入的 function
    callback();
    return a + b
}

let sum = add(3, 4, f);
console.log(sum);
```
會印出：
```
I'm callback
7
```

另一方面我們也可以回傳 function，這裡的 `addThree()` 回傳一個 function 是接受一個參數 `x` 回傳 `x + 3`，所以 `sum(4)`，會得到 `7`：
```javascript
function addThree() {
    return function(x) {
        return x + 3;
    }
}

/*
 * sum = function (x) {
 *   return x + 3;
 * }
 */
let sum = addThree();
```
因此會印出：
```
7
```

## 應用

在 FP 的世界中，我們可以把 function 用 Higher Order Function 做組合，所以我們可以把常見的**執行流程**抽象成 function，把**執行邏輯**做為 function 傳入。常見的有 `forEach`、`map`、`filter`、`reduce` 等流程 function。

### forEach
我們很常在程式中做的流程之一就是用迴圈對一連串資料比方 Array 或 List 中的元素一個個進行操作，所以這個迴圈流程就可以抽象成一個 function，而執行的流程很簡單：對傳入的一連串資料中 (比方 Array 或 List) 的一個個元素對其執行傳入的 function：
```javascript
function print(x) {
    console.log(x);
}

[1, 2, 3].forEach((i) => print(i));
```
上面的程式中會分別對 Array 中的 `1`、`2`﹑`3` 分別執行 `print` function 所以會印出：
```
1
2
3
```
而通常我們可以傳入一個 anonymous function (匿名函式)，也就是不用先宣告好 function 直接傳入 function，所以我們可以把上面的程式改寫成：
```javascript
// anonymous function 可以不用宣告名字
[1, 2, 3].forEach(function(i) {
    console.log(i);
});

// 在 JavaScript 中可以用 arrow function (箭頭函式) 讓程式更簡潔
[1, 2, 3].forEach((i) => console.log(i));
```

### map
如果今天我們傳入的 function 是想要把資料操作後再回傳，比方我們想把 `[1, 2, 3]` 中的元素每個都乘上 3，我們可以用 `forEach` 來完成：
```javascript
// 用來儲存乘 3 後的結果
let newArray = [];
[1, 2, 3].forEach((i) => {
    newArray.push(i * 3)
});

newArray.forEach((i) => console.log(i));
```
這樣會印出：
```
3
6
9
```
但因為這種操作其實也很常見，所以就有 `map` 這個 function，其執行流程為：對元素執行傳入的 function 並`回傳`執行後的新元素的列表 (以 JavaScript 來說是 Array )。

因此上面 `forEach` 版本可以改寫成：
```javascript
/* 因為 map 回傳的是執行後的新元素的 Array ，所以我們可以不用另外建立新的 Array 來儲存。
 * 另外因為在 arrow function 中如果只有一行程式比方下面 i * 3，則可以不用 {} 包住，且預設會回傳執行後的
 * 結果，所以下面的程式相同於：
 * let newArray = [1, 2, 3].map((i) {
 *    return i * 3
 * });
 */
let newArray = [1, 2, 3].map((i) => i * 3);

newArray.forEach((i) => console.log(i));
```

### filter
另一個很常見的邏輯是，我們需要過濾元素，比方我們想要過濾掉 `[1, 2, 3]` 中比 2 小的元素，如果用 `forEach` 的話可以寫成：
```javascript
let newArray = [];

[1, 2, 3].forEach((i) => {
    if (i > 1) {
        newArray.push(i);
    }
});

newArray.forEach((i) => console.log(i));
```
這樣會印出：
```
2
3
```
但我們可以用 `filter` 改寫，`filter` 的執行邏輯是：傳入一個 function，該 function 對元素執行後回傳的是 `true` 或 `false`，根據回傳的值，如果是 `true` 就放入回傳的列表中，如果是 `false` 就略過。
比方上面的程式我們可以改寫成：
```javascript
let newArray = [1, 2, 3].filter((i) => i > 1);

newArray.forEach((i) => console.log(i));
```

### reduce
有時候我們會想對操作完的元素再做統整的處理，比方我們想對 `[1, 2, 3]` 的每個數字乘 3 後再全部加總，如果用 `forEach` 的寫法會是：
```javascript
let sum = 0;
[1, 2, 3].forEach((i) => {
    i = i * 3;
    sum = sum + i;
});

console.log(sum);
```
這樣會印出 `18`。
而這個操作可以用 `reduce` 取代，`reduce` 我感覺是比之前的幾個 function 複雜，也比較難用文字描述，MDN 上的 `reduce` 的說明是：
> reduce() 方法將一個累加器及 Array 中每項元素（由左至右）傳入回呼函式，將 Array 化為單一值。

基本上我覺得光看文字應該是看不懂在說什麼，所以我們直接透過 API 說明搭配上面的例子來解釋可能比較清楚。
`reduce` 的 API 是
> arr.reduce(callback[accumulator, currentValue, currentIndex, array], initialValue)

接下來我們個別套到上面的例子中來看，首先 `arr` 就是目標要操作的 Array ，在這裡是 `[1, 2, 3]`，這應該沒什麼問題。

接下來 `reduce` 的參數就比較複雜，先說 `initialValue`，`initialValue` 指的是初始值，也就是當 `callback` 第一次執行時可以傳入 `initialValue` 使用。所以如果應用到上面的例子的話，我們的目的是想把每個元素加到 `sum`，而 `sum` 一開始是 `0`，因此我們可以傳入 0 做為 `initialValue`

而 `callback` 是針對元素要執行的 function，所以在這邊我們的 `callback` 中要對每個元素乘 3 而 `callback` 本身能接受的參數有：
- `accumulator`：這是指累加值，每次針對一個元素執行完 `callback` 後會把回傳值傳入下次執行 `callback` 時的 `accumulator`，而在第一次執行時如果有傳入 `initialValue` 的話就會是 `initialValue` 的值。在我們的例子中，我們想把每個元素加到 `sum` 中，所以我們將 0 做為 `initialValue` 傳入，而在每次加完元素後的總和就會做為 `accumulator` 傳入下次 `callback` 的執行
- `currentValue`：這是指執行 `callback` 當下時的目標元素，在這個例子中第一次執行為就會是 `1`，第二次會是 `2`
- `currentIndex`：執行 `callback` 當下時目標元素的 index，在這個例子中第一次執行為就會是 `0`，第二次會是 `1`，可以不傳入，在我們的例子不需要使用
- `array`：呼叫 `reduce` 的 Array ，在這個例子中為 `[1, 2, 3]`，可以不傳入，在我們的例子用不到

所以這樣套完後，我們可以用 `reduce` 改寫成：
```javascript
let total = [1, 2, 3].reduce((sum, i) => {
    return sum * 3 + i;
}, 0);

console.log(total);
```
我們可以把 `callback` 執行過程拆開來看會更清楚，第一次執行 `callback` 時會執行像是這樣的 function：
```javascript
function(0, 1) {
    return 0 * 3 + 1;
}
```
0 是我們傳入的 `initialValue` 因為是第一次執行所以會變成 `callback` 的第一個參數 `accumulator`，而 1 就是當下處理的元素。

第二次執行時會把第一次執行的回傳值 1 做為 `accumulator` 傳入 `callback`，所以第二次的 function 會是：
```javascript
function(1, 2) {
    return 1 * 3 + 2;
}
```
第三次執行則把第二次執行的回傳值 5 傳入，因此第三次的 function 會是：
```javascript
function(5, 3) {
    return 5 * 3 + 3;
}
```
在執行完第三次後因為沒有元素了，所以回傳最終結果 18。

除了這些以外還有很多其他的流程型 function，比方在 JavaScript 中有 `find` 傳入一個回傳值為 `boolean` 的 function 並回傳第一個符合 `true` 的元素，`some` 傳入回傳值為 `boolean` 的 function 並回傳 `boolean` 如果為 `true` 表示至少有一個元素會讓傳入 function 回傳 `true`。

### 回傳 function
上面提到的都是將 function 做為參數傳入另一個 function 中可做的應用，而應用可回傳 function 的特性我們就可以依據不同參數產生新的 function。

比方我們今天想要建立一個傳入數字 `a` 後回傳 `a + 3` 的 function，這很簡單：
```javascript
function addThree(a) {
    return a + 3;
}
```
然後我們想要新的加 4 的 function，這也很簡單：
```javascript
function addFour(a) {
    return a + 4;
}
```
但如果我們想要能產生 `a` 加上任意數字的 function，總不能一一去宣告，這時就可以用上這個特性：
```javascript
function addNumber(b) {
    return function(a) {
        return a + b;
    };
}

let addThree = addNumber(3);
console.log(addThree(4)); // 此行印出 7

let addFive = addNumber(5);
console.log(addFive(5)); // 此行印出 10
```
`addNumber(b)` 這個 function 所做的是將傳入的 `b` 的值綁定到要回傳的 function 內部的 `b` 中再回傳，所以 `addNumber(3)` 會得到：
```javascript
function(a) {
    return a + 3;
}
```
而 `addNubmer(5)` 則會得到：
```javascript
function(a) {
    return a + 5;
}
```
上述的應用稱為 **Partial Application** 也就是我們可以將 function 中的某些參數給予固定值來得到新的 function，新的 function 只需給予剩下的尚未給值的參數，如此我們就可以做更多抽象化來減少程式碼的重複。另一個例子，比方我們可以寫一個 function：
```javascript
function url(scheme, host, path) {
    return function(path) {
        return scheme + "://" + host + "/" + path;
    };
}
```
我們可以使用這個 function 產生 `function twitter(path)` 和 `function facebook(path)`，只要傳入 `path` 就可以得到 `https://www.twitter.com/{path}` 或 `https://www.facebook.com/{path}`：
```javascript
let twitter = url("https", "www.twitter.com");
console.log(twitter("user")); // 印出 https://www.twitter.com/user

let facebook = url("https", "www.facebook.com")
console.log(facebook("posts")); // 印出 https://www.facebook.com/posts
```

另一個可能的簡單應用是我們可以把讀取檔案的程式和處理檔案的程式分開：
```javascript
function readFile(callback) {
    let data = read("{path}") // 假使 read 是讀取 path 路徑的檔案

    return function(data) {
        callback(data);
    };
}

// 如此 printFile 會在讀取檔案後印出它的內容
let printFile = readFile((data) => {
    console.log(data);
})
```
如此一來，我們就可以重復使用讀取檔案的程式，只需要傳入不同的處理邏輯就可以得到不同的處理檔案方式。

## 結語
上述雖然是以 JavaScript 做為例子，但主要是描述 Higer Order Function 的概念，其他語言可能語法不同，但基本概念相同，在現在大多數語言都支援多範式下，通常也支援程度不一的 Functional Programming，比方 Java 雖然是 Object Oriented Programming (物件導向語言 OOP) 但在 Java 8 後也加入了 Functional Programming 的概念讓我們不必像從前一樣任何實作邏輯都必須先宣告成 `class` 而讓我們可以寫出更簡潔同時易讀性更高更專注在實作邏輯上的程式。

此篇主要是記錄我自己對 Higher Order Function 的了解和比較常使用的部分，除了此篇提到的以外還有很多 Higher Order Function 的應用，比方可以拿來做 **Dependency Injection** (依賴注入)，或者是實現如 **Aspect Oriented Programming** (剖面導向 AOP) 的設計，有機會的話再來另外記錄。

---
參考資料
- [Wiki - 高階函式](https://zh.wikipedia.org/wiki/%E9%AB%98%E9%98%B6%E5%87%BD%E6%95%B0)
- [MDN forEach](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Array/forEach)
- [MDN map](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Array/map)
- [MDN filter](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Array/filter)
- [MDN reduce](https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Array/Reduce)
- [Curry and Function Composition](https://medium.com/javascript-scene/curry-and-function-composition-2c208d774983)

如果有什麼想法或需要指正的地方，歡迎您留言或來信 😄