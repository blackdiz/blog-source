---
title: "Advent of Code 2021 Day 1"
date: 2022-01-23T16:40:51+08:00
Tags: ["advent of code"]
Categories: ["algorithm"]
---

Advent of Code 是一個很有趣的程式測驗活動，Advent 的意思是「將臨期」指的是為了慶祝耶穌聖誕前的準備期與等待期，所以每年聖誕節前夕，Advent of Code 網站會開始為期 25 天的活動，每天都會有一道題目一直到聖誕節前一天。
Advent of Code 的題目會和聖誕節作連結，而且題目會是一個個相關的情節，玩起來就像在看故事一般，和 LeetCode、Hacker Rank 等知名的刷題網站相比起來比較不會有為了解題而解題的感覺，常常為了看之後的故事而更有動力去解題。
每天的題目會分成兩部分，要先解完 Part 1 才能解 Part 2，通常 Part 1 的難度都滿簡單的有點像是熱身用的題目，而 Part 2 才是重頭戲，另外網站沒有自動檢查輸入測資的機制，所有測資都是純文字必須自己用程式讀取也是和一般像是 LeetCode 等刷題網站相比比較特別的地方。

其實 1、2 年前就知道這個活動，但那個時候對演算法和資料結構不熟覺得解題感覺是一件很可怕的事 😆，2021 年想說有稍微學了一下演算法和資料結構了決定來挑戰一下。一開始想說要符合活動目標拚在聖誕節前夕解完所有題目，但現實果然很殘酷...，不過既然開始了，而且我也很想看到故事的結局所以決定把它解完，順便記錄一下解法，也算是給自己的一個小挑戰。

在寫下這篇時是解到第 10 天，到第 10 天為止的解法都是我自己想的，所以不一定是最佳解，另外如果真的解不出來去~~抄襲~~參考別人的解法會特別註明。而這次解題我想說順便學習一下 Python，所以所有解法都是用 Python 解，同時也有上傳到 Github: [Advent of Code 2021](https://github.com/blackdiz/advent-of-code-2021)，另外這是 [Advent of Code 2021](https://adventofcode.com/2021) 的官網，官網上也有留著歷年的題目可以玩哦。

那麼就讓我們開始冒險的旅程吧~

## Day 1: Sonar Sweep

### Part 1

You're minding your own business on a ship at sea when the overboard alarm goes off! You rush to see if you can help. Apparently, one of the Elves tripped and accidentally sent the sleigh keys flying into the ocean!

Before you know it, you're inside a submarine the Elves keep ready for situations like this. It's covered in Christmas lights (because of course it is), and it even has an experimental antenna that should be able to track the keys if you can boost its signal strength high enough; there's a little meter that indicates the antenna's signal strength by displaying 0-50 stars.

Your instincts tell you that in order to save Christmas, you'll need to get all fifty stars by December 25th.

Collect stars by solving puzzles. Two puzzles will be made available on each day in the Advent calendar; the second puzzle is unlocked when you complete the first. Each puzzle grants one star. Good luck!

As the submarine drops below the surface of the ocean, it automatically performs a sonar sweep of the nearby sea floor. On a small screen, the sonar sweep report (your puzzle input) appears: each line is a measurement of the sea floor depth as the sweep looks further and further away from the submarine.

For example, suppose you had the following report:

```txt
199
200
208
210
200
207
240
269
260
263
```

This report indicates that, scanning outward from the submarine, the sonar sweep found depths of 199, 200, 208, 210, and so on.

The first order of business is to figure out how quickly the depth increases, just so you know what you're dealing with - you never know if the keys will get carried into deeper water by an ocean current or a fish or something.

To do this, count the number of times a depth measurement increases from the previous measurement. (There is no measurement before the first measurement.) In the example above, the changes are as follows:

```txt
199 (N/A - no previous measurement)
200 (increased)
208 (increased)
210 (increased)
200 (decreased)
207 (increased)
240 (increased)
269 (increased)
260 (decreased)
263 (increased)
```

In this example, there are 7 measurements that are larger than the previous measurement.

How many measurements are larger than the previous measurement?

---
題目的意思是我們在一艘潛水艇中，而潛水艇的聲納會回傳前方海溝的深度：

```txt
199
200
208
210
200
207
240
269
260
263
```

每個數字表示一個高低差的變化，而我們要做的是計算總共有幾次變化是增加深度的，以上面的例子來說，200 到 208 是 1 次，而 210 到 200 則是變淺所以不算。

### Solution

我們可以用 2 個 pointer: `first`、`second`，`first` 指向最一開始的數字，`second` 則指向第 2 個數字，兩數相減後如果為正的就表示是增加深度，我們的計數就加 1，接著同時將 `first`、`second` 向後移動一個數字再計算一次：

```txt
199     200 208 210...
 |       |
first  second

199 200     208 210...
     |       |
    first  second
```

一直到 `second` 碰到最後一個數字之後結束。

### Code

```python
# 讀取測資用
with open('./day1-input.txt', 'r') as f:
    increasement_count = 0
    # first 為第 1 個 pointer, 讀取測資第 1 個數字
    first = f.readline()
    # 因為 readline 後讀檔的指標會自動移到下一行, 所以 second = f.readline() 時, second 會指向第 2 個數字
    second = f.readline()
    # 檢查 second 不為 null, 當 second 為 null 時表示已經讀取完所有數字
    while first and second:
        # 相減為正則計數加 1
        if int(second) - int(first) > 0:
            increasement_count += 1
        # 將 first 指向下個數字
        first = second
        # 將 second 指向下個數字
        second = f.readline()
    print(increasement_count)
```

### Part 2

Considering every single measurement isn't as useful as you expected: there's just too much noise in the data.

Instead, consider sums of a three-measurement sliding window. Again considering the above example:

```txt
199  A
200  A B
208  A B C
210    B C D
200  E   C D
207  E F   D
240  E F G
269    F G H
260      G H
263        H
```

Start by comparing the first and second three-measurement windows. The measurements in the first window are marked A (199, 200, 208); their sum is 199 + 200 + 208 = 607. The second window is marked B (200, 208, 210); its sum is 618. The sum of measurements in the second window is larger than the sum of the first, so this first comparison increased.

Your goal now is to count the number of times the sum of measurements in this sliding window increases from the previous sum. So, compare A with B, then compare B with C, then C with D, and so on. Stop when there aren't enough measurements left to create a new three-measurement sum.

In the above example, the sum of each three-measurement window is as follows:

```txt
A: 607 (N/A - no previous sum)
B: 618 (increased)
C: 618 (no change)
D: 617 (decreased)
E: 647 (increased)
F: 716 (increased)
G: 769 (increased)
H: 792 (increased)
```

In this example, there are 5 sums that are larger than the previous sum.

Consider sums of a three-measurement sliding window. How many sums are larger than the previous sum?

---
Part 2 中我們必須每 3 個數字為一組加總後再計算是否有增加，最後統計增加的次數，而這 3 個數字是從最開頭的 3 個數字開始，接著同時往下一個數字後再相加一次，比方上面的例子中，從 199、200、208 開始，將這 3 個數字相加得到 607，接著往下一個數字後新的 3 個數字為 200、208、210，相加得到 618，618 - 607 = 11 所以計數加 1。

### Solution

從範例中我們可以看出每次的移動可以看成是前一組數字加總減去該組數字的第 1 個數字再加上下一個新的數字，比方：

```txt
199  A
200  A B
208  A B C
210    B C D
200  E   C D
207  E F   D
240  E F G
269    F G H
260      G H
263        H
```

第一組數字是 199 + 200 + 208 = 607，而下一組數字可以用 607 - 199 + 210 = 618 算出，所以我們一樣可以用兩個 pointer：`first`、`second` 指向每組數字中的第 1 個數字和第 3 個數字，

```txt
199     200     208     210     200...
 |               |
first          second
```

當加總完後，用加總完的數字減去 `first` 目前指向的數字，將 `first`、`second` 向下移動一個數字後再加上 `second` 指向的新數字來得到下一組總和：

```txt
// 先減去 199, 向後移動, 加上 210
199     200     208     210     200...
         |               |
       first          second
```

這樣就可以算出兩組的差，直到 `second` 碰到最後一個數字之後結束。

### Code

```python
# 讀取測資所有數字並放到 list 中
int_list = []
with open('./day1-input.txt', 'r') as f:
    int_list = [int(i) for i in f]

# 第 1 個 pointer, 指向每組中的第 1 個數字
start = 0
# 第 2 個 pointer, 指向每組中的第 3 個數字, 所以起始位置是 index 2
end = 2
# 記錄第 1 組總和
first_window = 0
# 記錄第 2 組總和
second_window = 0
increasement_count = 0

# 先算出最開始 3 個數字的總和
for i in range(start, end):
    first_window += int_list[i]

# 當 end 碰到最後 1 個數字後計算完就結束
while end < len(int_list) - 1:
    # 下一組數字為前一組減去 start 指向的數字, 再加上 end 向後移動後指向的數字
    second_window = first_window - int_list[start] + int_list[end + 1]
    if second_window - first_window > 0:
        increasement_count += 1

    # second_window 變成下一次計算的 first_window
    first_window = second_window
    # 將 start、end 同時向後移動 1 個數字
    start += 1
    end += 1

print(increasement_count)
```

### 結語

OK，這就是第一天的題目和我的解法啦，希望能堅持到第 25 天 😆，如果有什麼想法或需要指正的地方，歡迎您留言或來信 😄
