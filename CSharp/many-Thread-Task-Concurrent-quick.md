# C# 用最快的速度完成他，不考慮CPU記憶體，完成 10000 工作單元，你所不瞭解的多執行緒計算

這兩天看到臉書社團上有篇[討論文章](https://www.facebook.com/groups/DotNetUserGroupTaiwan/permalink/2529212007371902/?__cft__[0]=AZVjLCAjclLXi_wGI2-cX3YmQ9e0gT6mnwDUJlpepA3bLG8ouzkUD3Ut-zMHakKbnb0A4LyIkHGK9tBVvyWjsoiSGp3ENvCt6Fk1cjZpVyZQIjLrsr95fxHoS4bDUueBn3jYRYanMXevB3w3ZR5oIGb81IYaxUHd_t0zr3gz96Lp9GZxg5xnNKU9-xd2pBknniA&__tn__=%2CO%2CP-R)，那就是提問的人提出一個問題：`用最快的速度完成他，不考慮CPU記憶體` 同時執行 10000 次

這裡原先是要透過底下的程式碼，產生出10000個併行工作單元，並且使用 [Parallel 類別](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.tasks.parallel?view=netcore-3.1&WT.mc_id=DT-MVP-5002220) 提供的[Parallel.For 方法](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.tasks.parallel.for?view=netcore-3.1&WT.mc_id=DT-MVP-5002220)來同時執行這些工作單元，在每個委派方法內，使用休息5秒的做法，模擬需要執行的處理時間，而提問的人遇到瓶頸，希望採用 `用最快的速度完成他，不考慮CPU記憶體` 的方式來解決此一問題。

```csharp
Parallel.For(0, 10000, (i) =>
{
Thread.Sleep(5 * 1000);
});
```

不知道大家是否有看到提問人提出的這個簡單又明瞭的訴求 `用最快的速度完成他，不考慮CPU記憶體`，又姑且不論大家有著許多額外的建議與批評，這包括討論到 Thread 與 Task 是否有不同、有差異嗎？Thread.Sleep 也是浪費那條thread、thread 新增太慢、您應該先考慮有沒有了解 Thread 的意思、你要先搞清楚你要處理的事件是CPU密集型任務還是IO密集型任務，task本質上只是在同一個時間可以做更多事，不會加快處理事件的時間、這種程式我一輩子都不會寫到也不會遇到有這種需求，請問你能得到甚麼等等。

首先，我想要先針對提問人的需求，不管有著潛在問題或者後遺症，先來看看是否能夠做到且滿足他的需求，那就是同時啟動10000並行工作單元，能否在 5 秒內完成。

先使用最基本的 C# [Thread 執行緒類別](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.thread?view=net-5.0&WT.mc_id=DT-MVP-5002220)，看看能否做到同時執行10000個相同的工作，並且在五秒左右同時完成，底下是採用的程式碼

```csharp
int MAX = 10000;
int SLEEP = 5 * 1000;
List<Thread> threads = new List<Thread>();
CountdownEvent cde = new CountdownEvent(MAX);
Console.WriteLine($"starting {MAX} threads...");
Stopwatch stopwatch = new Stopwatch();
stopwatch.Start();
for (int i = 0; i < MAX; i++)
{
    int idx = i;
    Thread thread = new Thread(x =>
    {
        Thread.Sleep(SLEEP);
        cde.Signal();
    });
    thread.IsBackground = true;
    threads.Add(thread);
}
 
 
for (int i = 0; i < MAX; i++)
{
    threads[i].Start();
}
 
cde.Wait();
stopwatch.Stop();
Console.WriteLine();
Console.WriteLine($"{stopwatch.ElapsedMilliseconds} ms");
```

這裡透過 for 迴圈，產生出 10000 個執行緒物件，並且設定這些執行緒都是背景執行緒，這是可選擇性的作法，當然也可以產生 10000 個前景執行緒；為了要能夠確認這 10000 個執行是否都執行完畢，在此使用了 [CountdownEvent 類別](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.countdownevent?view=net-5.0&WT.mc_id=DT-MVP-5002220)，根據這篇[文章](https://docs.microsoft.com/zh-tw/dotnet/standard/threading/countdownevent?WT.mc_id=DT-MVP-5002220) 提到 ： System.Threading.CountdownEvent 是一個同步處理基本類型，在發出了特定次數的訊號給它之後，就會解除封鎖其等候中的執行緒。所以，就使用了 new CountdownEvent(MAX) 來進行 10000 次的倒數計時，只要執行緒執行完成之後，便會透過 cde.Signal(); 敘述，送出訊號，這樣就會完成倒數加一的工作。

再透過另外一個迴圈，將這些執行緒一次全部啟動執行，因此，理論上這台電腦中將會有 10000 個同時執行的工作，在此迴圈之後，使用 cde.Wait() 方法來等待 10000 委派方法的執行完成，因為這裡有使用 [Stopwatch 類別](https://docs.microsoft.com/zh-tw/dotnet/api/system.diagnostics.stopwatch?view=netcore-3.1&WT.mc_id=DT-MVP-5002220) 要來量測整個大量同時執行的工作花費了多少時間，最後便會顯示出總共執行時間大約是多少。

這裡將會是分別執行 3 次的輸出結果

```
starting 10000 threads...

6239 ms

starting 10000 threads...

6198 ms

starting 10000 threads...

6480 ms
```

先使用最基本的 C# [Task 工作類別](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.tasks.task?view=net-5.0&WT.mc_id=DT-MVP-5002220)，看看能否做到同時執行10000個相同的工作，並且在五秒左右同時完成，底下是採用的程式碼

```csharp
int MAX = 10000;
int SLEEP = 5 * 1000;
List<Task> tasks = new List<Task>();
Console.WriteLine($"starting {MAX} tasks...");
Stopwatch stopwatch = new Stopwatch();
stopwatch.Start();
for (int i = 0; i < MAX; i++)
{
    int idx = i;
    Task task = Task.Factory.StartNew(() =>
      {
          Thread.Sleep(SLEEP);
      }, TaskCreationOptions.LongRunning);
    tasks.Add(task);
}
Task.WaitAll(tasks.ToArray());
stopwatch.Stop();
Console.WriteLine();
Console.WriteLine($"{stopwatch.ElapsedMilliseconds} ms");
```

這裡透過 for 迴圈，產生出 10000 個 Task 物件，不過，這裡使用了 [TaskFactory.StartNew 方法](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.tasks.taskfactory.startnew?view=net-5.0&WT.mc_id=DT-MVP-5002220) 來產生這些大量工作物件，原因很簡單，因為想要這些大量的工作物件，不需要透過 [執行緒集區](https://docs.microsoft.com/zh-tw/dotnet/standard/threading/the-managed-thread-pool?WT.mc_id=DT-MVP-5002220) 來取得執行緒，而是要透過直接建立一個新的執行緒來處理該工作所指派的委派程式碼 

>(聽起來也些模糊、有些詭異、有些迷糊，反過頭來說，若你看懂了這段話，其實，這篇文章的問題你也就會解了；另外，許多人，甚至自視為大神的人，似乎對於執行緒與工作間的差異與本質不同，存在著許多問題，請大家在學習或者觀看網路文章的時候，要多多 停、看、聽)

當啟動完成 10000 個工作之後，便使用了 [Task.WaitAll 方法](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.tasks.task.waitall?view=net-5.0&WT.mc_id=DT-MVP-5002220) 來等待全部的 10000 個工作都完成。

> (從這裡，聰明的你應該已經看得出 Thread 執行緒 與 Task 工作 之間的差異點了嗎？但是，你可以分辨與看得出相同點嗎？)

這裡有使用 [Stopwatch 類別](https://docs.microsoft.com/zh-tw/dotnet/api/system.diagnostics.stopwatch?view=netcore-3.1&WT.mc_id=DT-MVP-5002220) 要來量測整個大量同時執行的工作花費了多少時間，最後便會顯示出總共執行時間大約是多少。

這裡將會是分別執行 3 次的輸出結果

```
starting 10000 tasks...

6550 ms

starting 10000 tasks...

6547 ms

starting 10000 tasks...

6533 ms
```

說明到這裡，你應該也看得出來 Thread 執行緒 與 Task 工作 之間的相同點了嗎？其中一個是，不論是使用執行緒，或者工作來同時執行 10000 工作單元，每個工作單元預計約 5 秒鐘執行時間，而整個執行完成的時間大約是 6.1~6.6 秒，這應該與原提問人想要做到的目標有些接近吧～

若對於這裡所提到的內容，歡迎大家在這裡進行討論，看看大家是否可以推敲出問題在哪裡，畢竟

名偵探柯南最常說的一句話 : 真相只有一個

請繼續參考更精采的 [C# 平行 / 並行計算 Parallel.For 隱藏在細節背後的惡魔，你所不瞭解的平行與併行計算](https://csharpkh.blogspot.com/2020/11/Parallel-For-Foreach-Thread-ThreadPool-Concurrent-Tricky.html)

## 彩蛋

上面兩種做法為單純僅使用執行序，或者工作來滿足這個題目的需求，不過，不論是哪種作法，都可以看到要耗損將近 10000 個執行序來完成這個需求任務。

在此，稍微修改一下原先 Task 的程式碼如下

```csharp
int MAX = 10000;
int SLEEP = 5 * 1000;
List<Task> tasks = new List<Task>();
Console.WriteLine($"starting {MAX} tasks...");
Stopwatch stopwatch = new Stopwatch();
stopwatch.Start();
for (int i = 0; i < MAX; i++)
{
    int idx = i;
    Task task = Task.Run(async () =>
    {
        await Task.Delay(SLEEP);
    });
    tasks.Add(task);
}
Task.WaitAll(tasks.ToArray());
stopwatch.Stop();
Console.WriteLine();
Console.WriteLine($"{stopwatch.ElapsedMilliseconds} ms");
```

這裡將會是分別執行 3 次的輸出結果

```
starting 10000 tasks...

5024 ms

starting 10000 tasks...

5035 ms

starting 10000 tasks...

5036 ms
```

甚麼？上面的程式碼幾乎與前面採用 [TaskFactory.StartNew 方法](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.tasks.taskfactory.startnew?view=net-5.0&WT.mc_id=DT-MVP-5002220) 的做法大致相同，但是，為什麼這樣的寫法卻更接近 5 秒的時間，幾乎是 5.0xx 秒左右。

到這裡，讀者您應該更能夠了解到直接使用執行緒或工作來強制產生大量執行緒所帶來的後遺症與副作用，之前有聽到某位自稱大神說過，想要讓程式跑的更快，就要使用更多的執行緒，殊不知嗎啡可用於幫人麻醉的緩解疼痛藥品，但是長期經常服用，而不知道嗎啡具有成癮性，將會形成吸食毒品問題與造成身體器官發生問題；用多了執行緒，到時候會很麻煩地。隨意聽信偏方、江湖術士的話，受騙的將會是你自己，因此，唯有對於整個基本知識與運作方式的徹底明瞭，才會有助於這接高階技術的學習與未來進行除錯與思考的依據。

## 炸(詐)彈

好的，大部分的看完這篇文章之後，再度回到原先的問題

那就是提問的人提出一個問題：`用最快的速度完成他，不考慮CPU記憶體` 同時執行 10000 次，這裡指名 使用 [Parallel 類別](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.tasks.parallel?view=netcore-3.1&WT.mc_id=DT-MVP-5002220) 提供的[Parallel.For 方法](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.tasks.parallel.for?view=netcore-3.1&WT.mc_id=DT-MVP-5002220) 來完成


```csharp
Parallel.For(0, 10000, (i) =>
{
Thread.Sleep(5 * 1000);
});
```


也就想說，沒問題，我也會解決此一問題，那就是把原先的 `Thread.Sleep(5 * 1000);` 敘述，改成 `await Task.Delay(5 * 1000);` 那不就好了。而且許多大神也都是這麼順利成章的說，想要使用非同步處理，就直接使用 [Parallel.For 方法](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.tasks.parallel.for?view=netcore-3.1&WT.mc_id=DT-MVP-5002220) 就可以做到了(也許你已經成為歐陽鋒，而所學成的九陰真經是黃蓉瞎掰給你的，最後結果是如何，要你自己去看那本書)，現在也是驗證這些人說明的時候。

```csharp
Stopwatch stopwatch = new Stopwatch();
stopwatch.Start();
Parallel.For(0, 10000, async (i) =>
{
    //Thread.Sleep(5 * 1000);
    await Task.Delay(5 * 1000);
});
stopwatch.Stop();
Console.WriteLine();
Console.WriteLine($"{stopwatch.ElapsedMilliseconds} ms");
```

OK OK 很 OK ，那就開始執行吧，將會得到底下的結果

```

23 ms

xxxx.exe (處理序 37520) 已結束，出現代碼 0。
按任意鍵關閉此視窗…
```

一看到執行結果，不要以為你練成神功了，若以剛剛的例子，還沒五秒鐘，只花費了 23 ms ，整個程式就結束執行了，這樣似乎與之前使用 Thread.Sleep 方法有些不同，因為，在這個例子中，程式一結束，那 10000 個等候 5 秒的工作單元 Unit of Work 在還沒執行完成前，也就直接提前終止執行了。

這樣的結果不是所預期的，因此，再度修改程式碼，使用執行緒同步 [CountdownEvent 類別](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.countdownevent?view=net-5.0&WT.mc_id=DT-MVP-5002220) 來同時等待這 10000 個工作單元的完成時刻來臨。

```csharp
CountdownEvent cde = new CountdownEvent(10000);
Stopwatch stopwatch = new Stopwatch();
stopwatch.Start();
Parallel.For(0, 10000, async (i) =>
{
    //Thread.Sleep(5 * 1000);
    await Task.Delay(5 * 1000);
    cde.Signal();
});
 
cde.Wait();
stopwatch.Stop();
Console.WriteLine();
Console.WriteLine($"{stopwatch.ElapsedMilliseconds} ms");
```

稍做小小修正，完成上述程式碼，二話不囉嗦，再來執行3次，得到底下結果

```

5039 ms

xxxx.exe (處理序 37520) 已結束，出現代碼 0。

5043 ms

xxxx.exe (處理序 37520) 已結束，出現代碼 0。

5034 ms

xxxx.exe (處理序 37520) 已結束，出現代碼 0。

```

首先，整個處理程序 Process 沒有提早 5 秒鐘前就結束，接著，竟然使用 [Parallel.For 方法](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.tasks.parallel.for?view=netcore-3.1&WT.mc_id=DT-MVP-5002220) 也可以做到僅需要 5 秒鐘就可以完成 10000 個工作單元的預期目標，現在，觀看這篇文章的讀者能夠知道發生了甚麼問題嗎？

若對於這裡所看到的各種疑問，歡迎大家在這裡進行討論，看看大家是否可以推敲出問題在哪裡，畢竟

名偵探柯南最常說的一句話 : 真相只有一個


## 請繼續參考更精采的

[C# 平行 / 並行計算 Parallel.For 隱藏在細節背後的惡魔，你所不瞭解的平行與併行計算](https://csharpkh.blogspot.com/2020/11/Parallel-For-Foreach-Thread-ThreadPool-Concurrent-Tricky.html)

[Blazor實戰故事經驗分享 1 - 風起雲湧 如何從無到有建立Blazor團隊與採用全端開發方式設計出給上市企業使用的Web系統](https://csharpkh.blogspot.com/2020/11/Blazor-Server-Side-Full-Stack-Case-Study-JavaScript-Story.html)

[Blazor實戰故事經驗分享 2 - 風雲再現 探究 Blazor 可以快速開發出來內部細節](https://csharpkh.blogspot.com/2020/11/Blazor-Server-Side-Layer-Data-Case-Study-Story.html)


