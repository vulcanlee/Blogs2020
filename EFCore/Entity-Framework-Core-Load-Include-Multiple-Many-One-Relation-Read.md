# Entity Framework Core 一對多的資料讀取

在上一篇 EF Core 討論文章 [在 ASP.NET Core 專案內，建立其他日誌輸出設備](https://csharpkh.blogspot.com/2020/10/Entity-Framework-Core-Blazor-ConfigureLogging-AddConsole-AddDebug-AddEventSourceLogger-AddEventLog.html) ，說明了如何透過 ILogger 來觀察 EF Core 產出了哪些 SQL 命到到 SQL Server 內。

在這篇文章中，來了解如何使用 EF Core 來讀取不同資料表內的關聯紀錄。

關於更多這方面的應用，可以參考 [載入相關資料](https://docs.microsoft.com/zh-tw/ef/core/querying/related-data/?WT.mc_id=DT-MVP-5002220) 這份文件內容。

請按照底下的步驟來進行操作

## 建立練習專案

* 打開 Visual Studio 2019
* 點選 [建立新的專案] 按鈕
* 在 [建立新專案] 對話窗內，選擇 [Blazor 應用程式] 專案樣板
* 在 [設定新的專案] 對話窗內，於 [專案名稱] 欄位內輸入 `EFRelation`
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
SchoolContext SchoolContext = new SchoolContext();
var allCourse = await SchoolContext.Course
    .AsNoTracking()
    .ToListAsync();
foreach (var itemCourse in allCourse)
{
    Console.WriteLine($"Course : {itemCourse.Title}");
    if(itemCourse.Department!= null)
    {
        Console.WriteLine($"Department : {itemCourse.Department.Name}");
    }
}
```

## 沒有指定使用何種方式載入關聯資料

從上面的程式碼將會看到，這裡會將所有的 課程 紀錄都讀取到 allCourse 這個集合物件內，並且使用 foreach 逐一將該課程紀錄與每個課程所指派的科系名稱顯示在螢幕上，不過，若該課程沒有指派任何科系紀錄，將不會看到這些科系名稱紀錄(在資料庫內因為有設定限制約束條件，因此，不會有這樣的課程與科系關聯存在)。在這裡的程式碼也會檢查，若科系物件不存在，則不會列印出科系名稱的資料。

實際執行這樣的程式碼，會發現到如下執行結果

```
Course : Calculus
Course : Chemistry
Course : Physics
Course : Composition
Course : Poetry
Course : Literature
Course : Trigonometry
Course : Microeconomics
Course : Macroeconomics
Course : Quantitative
```

這樣的執行結果與實際在資料庫到的紀錄有所不同，因為，在資料庫內，每個課程都會對應到一個科系。

## 使用明確式載入 Explicit Loading

首先，來使用 [明確式載入](https://docs.microsoft.com/zh-tw/ef/core/querying/related-data/explicit#explicit-loading?WT.mc_id=DT-MVP-5002220) 來解決此一問題。

請將程式碼修改成為如下

```csharp
SchoolContext SchoolContext = new SchoolContext();
var allCourse = await SchoolContext.Course
    .ToListAsync();
foreach (var itemCourse in allCourse)
{
    Console.WriteLine($"Course : {itemCourse.Title}");
    SchoolContext.Entry(itemCourse)
        .Reference(b => b.Department)
        .Load();
 
    if(itemCourse.Department!= null)
    {
        Console.WriteLine($"Department : {itemCourse.Department.Name}");
    }
}
```

實際執行這樣的程式碼，會發現到如下執行結果

```
Course : Calculus
Department : Mathematics
Course : Chemistry
Department : Engineering
Course : Physics
Department : Engineering
Course : Composition
Department : English
Course : Poetry
Department : English
Course : Literature
Department : English
Course : Trigonometry
Department : Mathematics
Course : Microeconomics
Department : Economics
Course : Macroeconomics
Department : Economics
Course : Quantitative
Department : Economics
```

現在終於可以看到科系資料了，這裡對於程式碼做了兩個修正，首先原先程式碼在讀取課程資料的時候，有使用了 `.AsNoTracking()` 這個呼叫方法 ，這裡宣告了不要使用 EF Core 的追蹤功能，也就是 [追蹤與 No-Tracking 的查詢](https://docs.microsoft.com/zh-tw/ef/core/querying/tracking?WT.mc_id=DT-MVP-5002220) 所說明的內容。可是，若加入了這個方法呼叫，將會造成底下的例外異常。

會有這樣的情況發生，這是因為有使用了 .Load() 方法，告知要使用明確式載入關聯紀錄，不過，若要使用這樣的功能，所查詢出來的紀錄，必須要納入 EF Core 變更追蹤內，否則，就會拋出這樣的例外異常。因此，這裡就會將 `.AsNoTracking()` 方法呼叫移除了。

```
System.InvalidOperationException
  HResult=0x80131509
  Message=Navigation property 'Department' on entity of type 'Course' cannot be loaded because the entity is not being tracked. Navigation properties can only be loaded for tracked entities.
  Source=Microsoft.EntityFrameworkCore
  StackTrace: 
   at Microsoft.EntityFrameworkCore.Internal.EntityFinder`1.Load(INavigation navigation, InternalEntityEntry entry)
   at Microsoft.EntityFrameworkCore.ChangeTracking.NavigationEntry.Load()
   at EFRelation.Program.<Main>d__0.MoveNext() in D:\Vulcan\GitHub\Blazor-HOL\Labs\EFRelation\EFRelation\Program.cs:line 20
```

另外，當每一筆課程紀錄讀取出來之後，使用 Collection 或者 Reference 這兩個方法，說明要明確地載入集合或這單一導覽屬性，最後，呼叫 Load 方法，就會送出 SQL 敘述到資料庫內，讀取相關科系紀錄出來。

## 使用積極式載入 Eager Loading

對於 [積極式載入](https://docs.microsoft.com/zh-tw/ef/core/querying/related-data/eager#eager-loading?WT.mc_id=DT-MVP-5002220) 這樣的做法則是另外一種不錯的選擇，可以在讀取全部課程紀錄的時候，一併讀取出相關的科系記錄出來。

請將程式碼修改成為如下

```csharp
SchoolContext SchoolContext = new SchoolContext();
var allCourse = await SchoolContext.Course
    .AsNoTracking()
    .Include(x=>x.Department)
    .ToListAsync();
foreach (var itemCourse in allCourse)
{
    Console.WriteLine($"Course : {itemCourse.Title}");
    if(itemCourse.Department!= null)
    {
        Console.WriteLine($"Department : {itemCourse.Department.Name}");
    }
}
```

實際執行這樣的程式碼，會發現到如下執行結果

```
Course : Calculus
Department : Mathematics
Course : Chemistry
Department : Engineering
Course : Physics
Department : Engineering
Course : Composition
Department : English
Course : Poetry
Department : English
Course : Literature
Department : English
Course : Trigonometry
Department : Mathematics
Course : Microeconomics
Department : Economics
Course : Macroeconomics
Department : Economics
Course : Quantitative
Department : Economics
```

在使用 [積極式載入](https://docs.microsoft.com/zh-tw/ef/core/querying/related-data/eager#eager-loading?WT.mc_id=DT-MVP-5002220) 做法中，將會使用 Include 方法指定要同時載入那些導航屬性的紀錄。


