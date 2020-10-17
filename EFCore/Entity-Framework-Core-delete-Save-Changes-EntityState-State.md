# Entity Framework Core EF Core 的 紀錄刪除

在上一篇 EF Core 討論文章 [EF Core 的 紀錄修改](https://csharpkh.blogspot.com/2020/10/Entity-Framework-Core-Update-Modified-Save-Changes-EntityState-State.html) ，說明了如何使用 EF Core 來修改資料庫紀錄的做法。

在這篇文章中，來了解如何使用 EF Core 來進行記錄刪除動作的作法，關於更多這方面的應用，可以參考 [基本儲存](https://docs.microsoft.com/zh-tw/ef/core/saving/basic/?WT.mc_id=DT-MVP-5002220) 這份文件內容。

請按照底下的步驟來進行操作

## 建立練習專案

* 打開 Visual Studio 2019
* 點選 [建立新的專案] 按鈕
* 在 [建立新專案] 對話窗內，選擇 [Blazor 應用程式] 專案樣板
* 在 [設定新的專案] 對話窗內，於 [專案名稱] 欄位內輸入 `efDeleteRecord`
* 在 [建立新的 Blazor 應用程式] 對話窗內，選擇 [Blazor 伺服器應用程式] 這個選項

  > 在該對話窗右半部的其他選項，可以不用變更

* 點選 [建立] 按鈕，以便開始建立這個專案

## 加入 Entity Framework Core 要使用到的 NuGet 套件

* 滑鼠右擊專案內的 [相依性] 節點
* 選擇 [管理 NuGet 套件]
* 點選 [瀏覽] 標籤分頁頁次
* 在 [搜尋] 文字輸入盒內，輸入 [Microsoft.EntityFrameworkCore.SqlServer]
* 點選 [安裝] 按鈕以便安裝這個套件
* 在 [搜尋] 文字輸入盒內，輸入 [Microsoft.EntityFrameworkCore.Tools]
* 點選 [安裝] 按鈕以便安裝這個套件

## 使用反向工程來產生 Entity Framework 要用到的 Entity 模型相關類別

* 切換到 [套件管理器主控台] 視窗

  > 若沒有看到 [套件管理器主控台] 視窗，點選功能表 [工具] > [NuGet 套件管理員] > [套件管理器主控台]

* 在 [套件管理器主控台] 輸入底下內容

  > 因為都在同一個專案內，所以，這裡可以省略 `StartupProject` & `Project` 這兩個參數，因此，底下的指令會更為精簡

```
Scaffold-DbContext "Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=School" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -f
```

請打開這個 [Program.cs] 檔案，完成底下的程式碼

```csharp
static void Main(string[] args)
{
    var context = new DataContext();
    context.Database.EnsureDeleted();
    context.Database.EnsureCreated();
 
    context.Department.Add(new Department());
    context.Entry(new Department()).State = EntityState.Added;
    context.SaveChanges();
    Console.WriteLine($"科系記錄數量:{context.Department.ToList().Count}");
    var departments = context.Department.ToList();
    Reset(context); // 清除 ChangeTracker 內的科系紀錄
    context.Department.Remove(departments[0]);
    context.Entry(departments[1]).State = EntityState.Deleted;
    context.SaveChanges();
    Console.WriteLine($"科系記錄數量:{context.Department.ToList().Count}");
}
public static void Reset(DataContext context)
{
    #region 解除快取紀錄
    foreach (var item in context
        .Set<Department>().Local.ToList())
    {
        context.Entry(item).State = 
            Microsoft.EntityFrameworkCore.EntityState.Detached;
    }
    #endregion
}
```

上面的程式碼使用 `new` 運算子來建立兩個 `Department` 類別的物件，分別呼叫了 `context.Department.Add(new Department());` 與 `context.Entry(new Department()).State = EntityState.Added;` 這兩個敘述，並且呼叫了 SaveChanges 告知 Entity Framework Core 將這筆紀錄新增到資料庫，接著使用了 `context.Department.ToList().Count` 敘述，顯示出 科系 Department 資料表內究竟有多少筆記錄存在，不用猜想，這裡當然會顯示出僅有2筆紀錄存在。

現在使用了 `var departments = context.Department.ToList()` 敘述，將科系資料表內的所有紀錄都取回到 .NET 環境內，並且這些紀錄都會在 Entity Framework Core 的變更追蹤系統內有份紀錄，而這裡也會呼叫一個客製方法 `Reset(context)`，這個方法將會清除 ChangeTracker 內的有關科系紀錄，這樣在 Entity Framework Core 內的變更追蹤系統內，就都沒有任何有關科系資料表相關的最新紀錄了。

使用這個敘述 `context.Department.Remove(departments[0])` 將第一筆科系紀錄刪除掉，而使用這個敘述 `context.Entry(departments[1]).State = EntityState.Deleted` 將第二個科系紀錄也刪除掉，這兩種作法都是得到相同的結果。


最後呼叫 `context.SaveChanges()` 來通知 EF Core 產生 Delete 的 SQL 敘述，讓資料庫刪除這兩筆紀錄，而最後的 `Console.WriteLine($"科系記錄數量:{context.Department.ToList().Count}")` 敘述，將會得到沒有任何紀錄的輸出結果。

底下是執行結果

```
科系記錄數量:2
科系記錄數量:0

```


