# Entity Framework Core 體驗 ChangeTracker 運作模式

在上一篇 EF Core 討論文章 [EF Core 的 紀錄刪除](https://csharpkh.blogspot.com/2020/10/Entity-Framework-Core-delete-Save-Changes-EntityState-State.html) ，說明了如何使用 EF Core 來刪除資料庫紀錄的做法。

在這篇文章中，來了解如何使用 EF Core 來進行記錄刪除動作的作法，關於更多這方面的應用，可以參考 [ChangeTracker](https://docs.microsoft.com/zh-tw/ef/core/saving/?WT.mc_id=DT-MVP-5002220) & [追蹤與 No-Tracking 的查詢](https://docs.microsoft.com/zh-tw/ef/core/querying/tracking?WT.mc_id=DT-MVP-5002220) 這份文件內容。

請按照底下的步驟來進行操作

## 建立練習專案

* 打開 Visual Studio 2019
* 點選 [建立新的專案] 按鈕
* 在 [建立新專案] 對話窗內，選擇 [Blazor 應用程式] 專案樣板
* 在 [設定新的專案] 對話窗內，於 [專案名稱] 欄位內輸入 `efChangeTracker`
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
    using (var context = new SchoolContext())
    {
        Console.WriteLine($"執行任何資料庫存取動作前的 ChangeTracker");
        DisplayStates(context.ChangeTracker.Entries());
        Console.WriteLine($"查詢第一筆 Person 紀錄，但使用 AsNoTracking");
        var person = context.Person.AsNoTracking().FirstOrDefault();
        DisplayStates(context.ChangeTracker.Entries());
        Console.WriteLine($"查詢第一筆 Person 紀錄 ");
        person = context.Person.FirstOrDefault();
        DisplayStates(context.ChangeTracker.Entries());
        Console.WriteLine($"修改與更新 Person 紀錄 ");
        person.LastName = $"{person.LastName}1";
        DisplayStates(context.ChangeTracker.Entries());
    }
}
```

這裡將要說明 Entity Framework Core 的變更追蹤的使用方式，這裡設計一個方法 [DisplayStates] ，該方法會接收一個 `IEnumerable<EntityEntry>` 的參數，接著，會將該列舉的 EntityEntry 集合物件內的名稱與狀態，顯示出來，更多這方面的資訊，可以參考 [EntityEntry](https://docs.microsoft.com/en-us/dotnet/api/microsoft.entityframeworkcore.changetracking.entityentry?view=efcore-3.1&WT.mc_id=DT-MVP-5002220)

在主程式那哩，首先會先呼叫 [DisplayStates] 方法，確認 Entity Framework Core 的變更追蹤內沒有任何的紀錄存在，接著會取得 Person 這個資料表內的第一筆紀錄出來，不過，這裡會呼叫 [AsNoTracking() 無追蹤查詢](https://docs.microsoft.com/zh-tw/ef/core/querying/tracking#no-tracking-queries?WT.mc_id=DT-MVP-5002220) 方法，指定此次資料查詢動作，不需要做異動追蹤；因此，可以得知，當執行完成之後，Entity Framework Core 的變更追蹤是沒有任何資料存在的

```csharp
private static void DisplayStates(IEnumerable<EntityEntry> entries)
{
    foreach (var entry in entries)
    {
        Console.WriteLine($"Entity: {entry.Entity.GetType().Name}," +
            $"State: { entry.State.ToString()}");
    }
}

```

接著同樣的再度呼叫 `context.Person.FirstOrDefault()` 方法，查詢出 Person 資料庫內的第一筆紀錄，不過，這次將會採用預設的 Entity Framework Core 的變更追蹤功能，因此，當這個敘述執行完成之後，Entity Framework Core 的變更追蹤內將會有一筆紀錄存在。這筆紀錄的狀態值將會是 `ntity: Person,State: Unchanged`

最後，嘗試將剛剛查詢出來的人員物件的 LastName 屬性作變更，並且再度呼叫 [DisplayStates] 方法，將會看到這樣的輸出內容： `Entity: Person,State: Modified` 代表該紀錄已經被修改過了，也就是說，當呼叫 SaveChanges 方法之後，Entity Framework Core 將會產生一筆 Update 的 SQL 敘述到資料庫內，以便更新該筆紀錄。

底下是執行結果

```
執行任何資料庫存取動作前的 ChangeTracker
查詢第一筆 Person 紀錄，但使用 AsNoTracking
查詢第一筆 Person 紀錄
Entity: Person,State: Unchanged
修改與更新 Person 紀錄
Entity: Person,State: Modified

```


