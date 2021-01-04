# ThreadStatic 與 ThreadLocal 預設值執行結果差異

對於執行緒本身的儲存空間，也就是 執行緒區域儲存區 [Thread Local Storage](https://docs.microsoft.com/zh-tw/dotnet/standard/threading/thread-local-storage-thread-relative-static-fields-and-data-slots?WT.mc_id=DT-MVP-5002220) ，在 .NET 中可以使用 [ThreadStaticAttribute 類別](https://docs.microsoft.com/zh-tw/dotnet/api/system.threadstaticattribute?view=net-5.0&WT.mc_id=DT-MVP-5002220) 與 [ThreadLocal<T> 類別](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.threadlocal-1?view=net-5.0&WT.mc_id=DT-MVP-5002220) 來做到，前者為 屬性 Attribute 宣告方式，後者為泛型型別；這兩者對於在多執行緒執行下，會有一個明顯的差異，那就是使用前者的話，若有指定預設值，並不會在每個執行緒內第一次取得 ThreadStatic 標示的靜態物件，都會使用宣告的預設值來呈現。

而對於 ThreadLocal 的泛行型別宣告的用法，則是每個執行緒在取用的時候，都會擁有宣告的預設值。

首先，宣告兩個類別，這兩個類別都宣告一個靜態整數成員，分別使用 ThreadStatic 與 ThreadLocal 來定義，而且前者設定預設值為 1 ，而後者宣告預設值為 2

```csharp
class MyThreadStaticClass
{
    [ThreadStatic]
    public static int MyInt = 1;
}

class MyThreadLocalcClass
{
    public static ThreadLocal<int> MyInt =
        new ThreadLocal<int>(() => 2);
}
```

現在，首先先來針對 MyThreadStaticClass.MyInt 這個使用 ThreadStatic 宣告的靜態成員來進行測試，在這裡將會使用 Task.Run 產生出 500 個任務，也就是會有 500 執行緒來執行同樣的委派方法，在指定的匿名 [Lambda 運算式](https://docs.microsoft.com/zh-tw/dotnet/csharp/language-reference/operators/lambda-expressions?WT.mc_id=DT-MVP-5002220) 方法內，將會顯示出 MyThreadStaticClass.MyInt 這個靜態成員的值。

```csharp
List<Task> allTasks = new List<Task>();
Console.WriteLine("多執行使用 ThreadStatic 執行結果");
for (int i = 0; i < 500; i++)
{
    Task task = Task.Run(() =>
    { Console.Write($"{MyThreadStaticClass.MyInt} "); });
    allTasks.Add(task);
}
Task.WaitAll(allTasks.ToArray());
Console.WriteLine();
```

這裡是執行結果，這裡可以看到使用 ThreadStatic 的方式，在多執行緒執行下，並沒有正確地顯示出預期的預設值

```
多執行使用 ThreadStatic 執行結果
1 0 0 0 0 0 0 0 0 1 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 1 1 1 1 1 1 1 0 0 0 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 1 1 1 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 0 0 0 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 1 1 1 1 1 1 1 1 1 1 1 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 1 1 0 0 0 1 1 0 0 0 0 0 0 0
```

接著來針對 MyThreadLocalcClass.MyInt 這個使用 ThreadLocal 宣告的靜態成員來進行測試，在這裡將會使用 Task.Run 產生出 500 個任務，也就是會有 500 執行緒來執行同樣的委派方法，在指定的匿名 Lambda 方法內，將會顯示出 MyThreadLocalcClass.MyInt 這個靜態成員的值。

```csharp
Console.WriteLine("多執行使用 ThreadLocal 執行結果");
allTasks.Clear();
for (int i = 0; i < 500; i++)
{
    Task task = Task.Run(() =>
    { Console.Write($"{MyThreadLocalcClass.MyInt} "); });
    allTasks.Add(task);
}
Task.WaitAll(allTasks.ToArray());
Console.WriteLine();
```

這裡是執行結果，這裡可以看到使用 ThreadLocal 的方式，在多執行緒執行下，都會有預期的預設值出現

```
多執行使用 ThreadLocal 執行結果
2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2 2
```


