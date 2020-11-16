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



