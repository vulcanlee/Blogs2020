# Entity Framework Core EF Core 的 紀錄新增

在上一篇 EF Core 討論文章 [Entity Framework Core 一對多的資料讀取](https://csharpkh.blogspot.com/2020/10/Entity-Framework-Core-Load-Include-Multiple-Many-One-Relation-Read.html) ，說明了如何使用 EF Core 來取得導航屬性內的關聯紀錄做法。

在這篇文章中，來了解如何使用 EF Core 來進行記錄新增動作的作法，關於更多這方面的應用，可以參考 [基本儲存](https://docs.microsoft.com/zh-tw/ef/core/saving/basic/?WT.mc_id=DT-MVP-5002220) 這份文件內容。

請按照底下的步驟來進行操作

## 建立練習專案

* 打開 Visual Studio 2019
* 點選 [建立新的專案] 按鈕
* 在 [建立新專案] 對話窗內，選擇 [Blazor 應用程式] 專案樣板
* 在 [設定新的專案] 對話窗內，於 [專案名稱] 欄位內輸入 `efAddRecord`
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
Department department1 = new Department()
{
    Name = "新增的科系1",
};
context.Department.Add(department1);
context.SaveChanges();
Department department2 = new Department()
{
    Name = "新增的科系2",
};
context.Entry(department2).State = Microsoft.EntityFrameworkCore.EntityState.Added;
context.SaveChanges();
foreach (var item in context.Department.ToList())
{
    Console.WriteLine($"科系名稱:{item.Name}");
}
```

上面的程式碼說明了兩種新增記錄到後端資料庫內的作法，其實這兩種作法都是相同的，不論是哪一種做法，首先需要建立一個 Entity 類別的執行個體，這裡是要新增兩筆科系的紀錄，因此，需要使用 `new` 運算子來建立 `Department` 類別的物件。

一旦 Entity 的物件產生完成之後，便可以使用這樣的作法 `context.Department.Add(department1)` 將剛剛建立的物件，通知 EF Core 要準備把這個物件新增到資料庫內，最後，呼叫 `context.SaveChanges();` 便會把 Entity Framework Core 內的變更追蹤的紀錄比對一次，便會發現到有要新增一筆紀錄的請求，接著便會產生 Insert SQL 敘述，把這個物件新增到資料庫內，成為一筆新的紀錄。

第二種作法那就是使用 `context.Entry(department2).State = Microsoft.EntityFrameworkCore.EntityState.Added;` 這樣的作法，宣告這個物件需要在下次呼叫 SaveChanges 的時候，將這個物件新增到資料庫內。

上面的程式會再度查詢資料庫內所有科系的紀錄，並且顯示在螢幕上，底下是執行結果

```
科系名稱:新增的科系1
科系名稱:新增的科系2
```


