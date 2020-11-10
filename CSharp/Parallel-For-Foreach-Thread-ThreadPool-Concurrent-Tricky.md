# C# 平行 / 並行計算 Parallel.For 隱藏在細節背後的惡魔，你所不瞭解的平行與併行計算

幾乎所有使用 C# 程式語言的開發者，都會於 平行 / 並行計算 parallel / concurrent computing 存在著少女心的崇景，多麼希望能夠擁有與掌握些技術呀？可是，老天爺是殘酷的，對於你的願望，老天爺是有聽到，也就給你一盞明燈 [Parallel 類別](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.tasks.parallel?view=netcore-3.1&WT.mc_id=DT-MVP-5002220)，滿心歡喜地來學習這項指引，該 Parallel 類別開宗明義地說到： `提供平行迴圈和區域的支援` ，神呀，這就是我要的功能與技能，請賜與我神奇的力量吧～。

順手打開 [Parallel.For 方法](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.tasks.parallel.for?view=netcore-3.1&WT.mc_id=DT-MVP-5002220)，看到這個方法為：執行可平行執行反覆項目的 for 迴圈 (請大家務必先閱讀一下這個連結的文章內容) ，而且在這篇文章中也附上一個相當清楚的使用範例，相信你已經躍躍欲試，並且可以把大量重複性的工作，使用這個具有平行處理的方法，做到大幅提升效能的目的，因為大家可以同時一起來執行呀~~。

舉例來說，若這些重複性的工作每個需要約 500ms 才能夠執行完成，現在有 100 這樣的工作要來處理，你確信神明不會騙你，因為你也閱讀過這篇文章，你也相信你的眼睛不會看錯，眼見為憑，此時信信滿滿的和你老闆說，現在我可以把 100 個重複性工作，透過最新學成絕技，在 500ms 內把他們都執行完畢。

二話不說，寫段程式碼來測試看看，讓你老闆看到你的高超技能，先來平行執行 4 個工作，在這裡，每個工作會使用 Thread.Sleep 方法來模擬正在進行處理相關工作，這裡的休息時間將會使用變數 processCost 來代表，其單位是 ms；另外，使用變數 N 來代表要同時執行幾個相同計算作業。在這裡使用到了 [Parallel.For 方法](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.tasks.parallel.for?view=netcore-3.1&WT.mc_id=DT-MVP-5002220) 來做到這樣平行執行能力；最後，要來量測整個大量同時執行的工作花費了多少時間，這裡使用了 [Stopwatch 類別](https://docs.microsoft.com/zh-tw/dotnet/api/system.diagnostics.stopwatch?view=netcore-3.1) 來做到這樣的需求，這個模擬程式於執行完成之後，會輸出總共花費了多少時間。

底下是完成的程式碼

```csharp
int processCost = 500;
int N = 4;
Stopwatch stopwatch = new Stopwatch();
stopwatch.Start();
Parallel.For(0, N, i =>
{
    Thread.Sleep(processCost);
});
stopwatch.Stop();
Console.WriteLine($"平行處理 {N} 個工作，共需 {stopwatch.ElapsedMilliseconds} ms");
```

不過，請先不要觀看底下結過

看到程式碼之後，你覺到這樣的程式碼執行完成之後

會用到多少時間呢？

底下是在作者電腦上執行完成的結果

```
平行處理 4 個工作，共需 675 ms
```

感覺上還不錯，雖然有著 175 ms 的落差，似乎還不錯，不要緊張，還可以做到更好，這裡是使用除厝組態來執行，現在切換成為 Release 模式來執行

```
平行處理 4 個工作，共需 524 ms
```

哇，更佳完美了，幾乎同時完成了這四個工作，這下子更有信心了

(思考 : 不過，你知道為什麼會有這樣的大幅改善嗎？)

現在把同時要處理的工作改成 16 個，也就是把 N 改成 16 ，你覺得全部都執行完成，需要花費多少時間呢？

```csharp
int processCost = 500;
int N = 16;
Stopwatch stopwatch = new Stopwatch();
stopwatch.Start();
Parallel.For(0, N, i =>
{
    Thread.Sleep(processCost);
});
stopwatch.Stop();
Console.WriteLine($"平行處理 {N} 個工作，共需 {stopwatch.ElapsedMilliseconds} ms");
```

既然是平行處理，當然是大約 5xx ms，現在的執行結果，全部都使用 Release 模式下，採用不除錯的方式來執行。

```
平行處理 16 個工作，共需 1045 ms
```

我的老天鵝，這是在作弄我嗎？這裡只不過把同時要處理的工作提升到 16 個，就需要 1045 ms 才能夠執行完成，不過，最後是要能夠同時處理 100 個呀，那麼不是無法做到僅需要 5xx ms 同時完成的境界嗎？

(思考 : 為什麼會產生出其他額外的時間，這些時間究竟是在執行那些程式碼呢？)

(思考 : 那麼，真的使用這樣平行設計方法，可以做到約 5xx ms 來完成所有結果嗎？ ==> 是第)

現在在來提升同時處理工作數量，現在提升為 50 個 (N=50)

請開始評估與猜測，你認為需要多久的時間呢?

```
平行處理 50 個工作，共需 3111 ms
```

最後讓 N=100，所得到的結果是 

```
平行處理 100 個工作，共需 6194 ms
```

看樣子，100平行執行的工作，處理時間約50個平行處理工作的一倍

(思考 : 真的是每增加一倍的平行處理工作量，就會需要額外一倍的處理時間？)

再來個挑戰，讓每個處理工作設定約 5000 ms 才能夠完成，並且設定同時處理 8 個工作，所得到的結果如下

```
```

再來個挑戰，讓每個處理工作設定約 5000 ms 才能夠完成，並且設定同時處理 8 個工作，所得到的結果如下

```
```

再來個挑戰，讓每個處理工作設定約 5000 ms 才能夠完成，並且設定同時處理 8 、 50 、 100 個工作，所得到的結果如下

```csharp
int processCost = 5000;
int N = 8;
Stopwatch stopwatch = new Stopwatch();
stopwatch.Start();
Parallel.For(0, N, i =>
{
    Thread.Sleep(processCost);
});
stopwatch.Stop();
Console.WriteLine($"平行處理 {N} 個工作，共需 {stopwatch.ElapsedMilliseconds} ms");
```

同時平行執行 8 個工作的執行結果
```
平行處理 8 個工作，共需 5041 ms
```

同時平行執行 50 個工作的執行結果
```
平行處理 50 個工作，共需 20108 ms
```

同時平行執行 100 個工作的執行結果
```
平行處理 100 個工作，共需 35115 ms
```

不知道當你看到這樣的結果出來，是不是又推翻掉你之前對於 [Parallel 類別](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.tasks.parallel?view=netcore-3.1&WT.mc_id=DT-MVP-5002220) 理解，不要認為這些數值的變動是不正常的，當您了解到背後的運作原理與許多核心功能，這些執行結果都是可以預測出來的，你無須撞牆、踩雷、等這彩蛋挑出來給你驚喜。

這篇文章先針對幾乎所有 C# 開發者都想要具備 平行 / 並行計算 parallel / concurrent computing 希望能夠輕鬆駕馭這個技能的心情，讓大家了解到其實要學會這樣的技能並不困難，困難的在於你是否準備好要去理解隱藏在背後的原理、小心假設並且逐步來驗證、要能夠區分所看到、聽到、碰到的人所告訴你的知識與方法是否是不正確的，畢竟，網路上很多自稱為大神的人，當包裝的光環退去之後，剩下沒有穿衣服的大神，相信這樣的裸身大神並不使養眼的，也不是你想要看到的。

若對於這裡所提到的問題，歡迎大家在這裡進行討論，看看大家是否可以推敲出問題在哪裡，畢竟

名偵探柯南最常說的一句話 : 真相只有一個



