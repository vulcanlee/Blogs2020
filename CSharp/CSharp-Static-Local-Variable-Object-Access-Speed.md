# 在 .NET C# 環境中，要存取靜態變數或者區域變數，是否都是同樣的存取速度

當在進行多執行緒程式設計的時候，往往會需要能夠讓多執行緒可以同時來存取一個共用的資源，也許會利用這個共用資源做為要回傳執行緒執行結果之用；可是，你知道當 .NET C# 程式在進行靜態變數或者區域變數(也就是該變數通常位於某個函式內所宣告的變數)的時候，存取區域變數的執行速度，會優於靜態全域變數，而到底差了多少？

為了要了解這個問題，特別撰寫底下的程式碼，連續存取靜態變數與區域變數數次，這裡將會執行 `int.MaxValue` 次，看看執行結果，不過，這裡的執行結果將會是在 Release 模式下所測試出來的結果。

```
Access Static : 3660ms
Access Static : 3593ms
Access Static : 3625ms
Access Local : 1705ms
Access Local : 1653ms
Access Local : 1692ms
```

可以看到上述的執行及果，對於存取靜態變數的時候，在這個範例將需要花費 359ˇˇ~3660 ms 之間，不過，對於存取區域變數的時候，可以看的出來，存取速度明顯快了許多，大約為 1692 ~1705 ms 之間。

因此，若要在大量迴圈到要存取變數的時候，建議不要使用靜態變數，若有需要，可以將靜態變數設定為區域變數，以便增快執行速度。

```csharp
class Program
{
    static int staticInt = 0;
    static void Main(string[] args)
    {
        int localInt = 0;
        StaticObjectAccess();
        StaticObjectAccess();
        StaticObjectAccess();
        LocalObjectAccess(localInt);
        LocalObjectAccess(localInt);
        LocalObjectAccess(localInt);
    }
 
    private static void LocalObjectAccess(int localInt)
    {
        Stopwatch stopwatch = new Stopwatch();
        stopwatch.Start();
        for (int i = 1; i < int.MaxValue; i++)
        {
            if (i % 2 == 0)
                localInt++;
            else
                localInt--;
        }
        stopwatch.Stop();
        Console.WriteLine($"Access Local : {stopwatch.ElapsedMilliseconds}ms");
        return;
    }
 
    private static void StaticObjectAccess()
    {
        Stopwatch stopwatch = new Stopwatch();
        stopwatch.Start();
        for (int i = 1; i < int.MaxValue; i++)
        {
            if (i % 2 == 0)
                staticInt++;
            else
                staticInt--;
        }
        stopwatch.Stop();
        Console.WriteLine($"Access Static : {stopwatch.ElapsedMilliseconds}ms");
        return;
    }
}
```

