# Entity Framework Core EF Core 的 紀錄修改

在上一篇 EF Core 討論文章 [EF Core 的 紀錄新增](https://csharpkh.blogspot.com/2020/10/Entity-Framework-Core-Add-Insert-Save-Changed-EntityState-State.html) ，說明了如何使用 EF Core 來建立產生一筆新的資料庫紀錄的做法。

在這篇文章中，來了解如何使用 EF Core 來進行記錄修改動作的作法，關於更多這方面的應用，可以參考 [基本儲存](https://docs.microsoft.com/zh-tw/ef/core/saving/basic/?WT.mc_id=DT-MVP-5002220) 這份文件內容。

請按照底下的步驟來進行操作

## 建立練習專案

* 打開 Visual Studio 2019
* 點選 [建立新的專案] 按鈕
* 在 [建立新專案] 對話窗內，選擇 [Blazor 應用程式] 專案樣板
* 在 [設定新的專案] 對話窗內，於 [專案名稱] 欄位內輸入 `efUpdateRecord`
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
var context = new DataContext();
context.Database.EnsureDeleted();
context.Database.EnsureCreated();
 
Department department1 = new Department() { Name = "新增的科系1", };
context.Department.Add(department1);
context.SaveChanges();
Department department2 = context.Department.First();
department2.Name = "修改該科系名稱";
context.SaveChanges();
Reset(context); // 清除 ChangeTracker 內的科系紀錄
Department department3 = new Department()
{
    Id = department1.Id,
    Name = "使用狀態來修改的科系名稱",
};
context.Entry(department3).State = Microsoft.EntityFrameworkCore.EntityState.Modified;
context.SaveChanges();
foreach (var item in context.Department.ToList())
{
    Console.WriteLine($"科系名稱:{item.Name}");
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

上面的程式碼使用 `new` 運算子來建立 `Department` 類別的物件，也就是科系名稱為 [新增的科系1] 的紀錄，並且呼叫了 SaveChanges 告知 Entity Framework Core 將這筆紀錄新增到資料庫，不過，記得要呼叫 `context.SaveChanges()` 方法，這樣的異動請求才會更新到資料庫內；接著，使用了 `context.Department.First()` 表示式，取得科系資料表內的第一筆紀錄，也就是剛剛建立的紀錄。

現在要來變更剛剛取得的紀錄，也就是 department2 這個物件，使用這樣的敘述 `department2.Name = "修改該科系名稱";` ，當想要同步這個 .NET 物件與資料庫內的紀錄時候，還是一樣要使用 呼叫 `context.SaveChanges()` 方法，因為 Entity Framework Core 發現到在變更追蹤內的紀錄，與剛剛修改的物件不相同，因此，將會產生出 Update 的 SQL 敘述，並且把這個敘述送到資料庫內。

由於在取得科系資料表內的第一筆紀錄的時候，使用了 `Department department2 = context.Department.First();` 這樣的敘述，因此，在 Entity Framework Core 內的變更追蹤內，將會有剛剛取得的紀錄物件，現在將會呼叫一個客製方法 `Reset(context)`，這個方法將會清除 ChangeTracker 內的有關科系紀錄，這樣在 Entity Framework Core 內的變更追蹤系統內，就都沒有任何有關科系資料表相關的最新紀錄了。

在這裡使用 C# new 運算子，建立一個 department3 的物件

接下來，將使用 C# new 運算子，建立一個 department3 .NET 物件，不過，該物件的 Id 屬性值將會與 department1的物件值相同，不過科系名稱欄位卻是不同的內容，由於由於 Entity Framework Core 內的變更追蹤系統內，沒有任何有關科系資料表相關的最新紀錄，因此，當下達 `context.SaveChanges()` 方法，這個物件的異動，是不會同步到資料庫內，因此，需要使用下面兩種做法的其中一個。

第一個做法是下達 `context.Department.Update(department3);` 這樣的敘述，如同前一篇文章要新增一筆紀錄相同，只不過這裡要呼叫的 Update 這樣的方法；第二種做法是使用 `context.Entry(department3).State = Microsoft.EntityFrameworkCore.EntityState.Modified;` 這樣的敘述，通知 Entity Framework Core 的變更追蹤系統，有一筆紀錄要更新到資料庫內。

不論是哪種做法，最後都要 呼叫 `context.SaveChanges()` 方法

上面的程式會再度查詢資料庫內所有科系的紀錄，並且顯示在螢幕上，底下是執行結果

```
科系名稱:使用狀態來修改的科系名稱

```


