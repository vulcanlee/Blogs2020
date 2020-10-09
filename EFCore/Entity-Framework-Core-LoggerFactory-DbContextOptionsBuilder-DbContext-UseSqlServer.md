# 使用 LoggerFactory 觀察 EF Core 送出的 SQL Statement

根據 上一篇 EF Core 討論文章 [使用 DbContextOptionsBuilder 來指定連線字串與觀察 EF Core 產生的 SQL 指令](https://csharpkh.blogspot.com/2020/10/Entity-Framework-Core-DbContextOptionsBuilder-DbContext-connection-String-UseSqlServer.html) ，透過 SQL Server Profiler 來觀察使用 EF Core 的程式，究竟產生了甚麼 SQL 指令到 SQL Server 上，現在，使用另外一種方式來觀察究竟執行了甚麼 SQL指令，在這裡需要使用 [LoggerFactory](https://docs.microsoft.com/zh-tw/dotnet/api/microsoft.extensions.logging.loggerfactory?view=dotnet-plat-ext-3.1&WT.mc_id=DT-MVP-5002220) 這個類別來做到這件事情。根據官方文件上提到的，這個類別的目的是 : `根據指定的提供者，產生 ILogger 類別的執行個體。`

根據這份文件內容 [.NET Core 與 ASP.NET Core 中的記錄](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/logging/?view=aspnetcore-3.1&WT.mc_id=DT-MVP-5002220) 可以得知， [ILogger] 是用於提供支援記錄 API。

請按照底下的步驟來進行操作

## 建立練習專案

* 打開 Visual Studio 2019
* 點選 [建立新的專案] 按鈕
* 在 [建立新專案] 對話窗內，選擇 [主控台應用程式 (.NET Core)] 專案樣板
* 在 [設定新的專案] 對話窗內，於 [專案名稱] 欄位內輸入 `efLoggerFactory`
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
using efLoggerFactory.Models;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Logging;
using System;
using System.Linq;

namespace efLoggerFactory
{
    class Program
    {
        public static readonly ILoggerFactory MyLoggerFactory
            = LoggerFactory.Create(builder =>
            {
                builder.AddConsole();
            });
        static void Main(string[] args)
        {
            string connectionString = @"Data Source=(localdb)\MSSQLLocalDB;Initial Catalog=School";
            DbContextOptions<SchoolContext> options = new DbContextOptionsBuilder<SchoolContext>()
                .UseLoggerFactory(MyLoggerFactory)
                .UseSqlServer(connectionString)
                .Options;
            using (var context = new SchoolContext(options))
            {
                Console.WriteLine($"取得 StudentGrade 第一筆紀錄");
                var aStudentGrade = context.StudentGrade.FirstOrDefault();
                Console.WriteLine($"更新成績為 4.99");
                aStudentGrade.Grade = 4.99m;
                context.SaveChanges();
            }
        }
    }
}
```

從上面的程式碼中，宣告了一個靜態的 ILoggerFactory 型別變數 ， ILoggerFactory ， 透過 [LoggerFactory.Create] 這個工廠方法，產生出一個物件，在呼叫這個工廠方法的時候，也宣告了使用螢幕 Console 作為輸出日誌的目的地，這裡可以參考 [ConsoleLoggerExtensions.AddConsole 方法](https://docs.microsoft.com/zh-tw/dotnet/api/microsoft.extensions.logging.consoleloggerextensions.addconsole?view=dotnet-plat-ext-3.1&WT.mc_id=DT-MVP-5002220) 文件。

接著，如同上一篇文章 [使用 DbContextOptionsBuilder 來指定連線字串與觀察 EF Core 產生的 SQL 指令](https://csharpkh.blogspot.com/2020/10/Entity-Framework-Core-DbContextOptionsBuilder-DbContext-connection-String-UseSqlServer.html) 做法，先建立一個 [DbContextOptions] 物件，不過，當該物件建立之後，會呼叫 [UseLoggerFactory] 這個方法，並且把 MyLoggerFactory 物件傳送進去，如此，便可以透過這個 [ILogger] 執行個體來觀察 EF Core 產生的 SQL 敘述。

執行結果如下

```
取得 StudentGrade 第一筆紀錄
info: Microsoft.EntityFrameworkCore.Infrastructure[10403]
      Entity Framework Core 3.1.7 initialized 'SchoolContext' using provider 'Microsoft.EntityFrameworkCore.SqlServer' with options: None
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (29ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
      SELECT TOP(1) [s].[EnrollmentID], [s].[CourseID], [s].[Grade], [s].[StudentID]
      FROM [StudentGrade] AS [s]
更新成績為 4.99
info: Microsoft.EntityFrameworkCore.Database.Command[20101]
      Executed DbCommand (37ms) [Parameters=[@p1='?' (DbType = Int32), @p0='?' (DbType = Decimal)], CommandType='Text', CommandTimeout='30']
      SET NOCOUNT ON;
      UPDATE [StudentGrade] SET [Grade] = @p0
      WHERE [EnrollmentID] = @p1;
      SELECT @@ROWCOUNT;
```

觀察上面的輸出內容，可以但看到當 SchoolContext 進行初始化的時候，會使用 Microsoft.EntityFrameworkCore.SqlServer 這個提供者，接著便會執行底下的 SQL 指令，查詢出 StudentGrade 資料表內的紀錄

```SQL
Executed DbCommand (29ms) [Parameters=[], CommandType='Text', CommandTimeout='30']
SELECT TOP(1) [s].[EnrollmentID], [s].[CourseID], [s].[Grade], [s].[StudentID]
FROM [StudentGrade] AS [s]
```

最後執行底下的 SQL Command ，更新該筆紀錄

```SQL
 Executed DbCommand (37ms) [Parameters=[@p1='?' (DbType = Int32), @p0='?' (DbType = Decimal)], CommandType='Text', CommandTimeout='30']
 SET NOCOUNT ON;
 UPDATE [StudentGrade] SET [Grade] = @p0
 WHERE [EnrollmentID] = @p1;
 SELECT @@ROWCOUNT;
```




