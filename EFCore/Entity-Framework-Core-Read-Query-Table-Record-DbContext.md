# Console 專案與 EF Core 讀取已經存在的資料庫

在前一個文章 [Entity Framework Core  - 反向工程 建立 Entity Model建立 Entity Model](https://csharpkh.blogspot.com/2020/09/Entity-Framework-Core-Model-DbContext-Reverse-Engineer.html) ，說明了如何使用反向工程與 [Scaffold-DbContext](https://docs.microsoft.com/zh-tw/ef/core/miscellaneous/cli/powershell?WT.mc_id=DT-MVP-5002220) 指令來產生出 Entity Framework Core 需要的 Entity & DbContext 相關類別。

因為有了這些 Entity 定義類別，便可以進行資料庫的操作，在這裡，將會來練習如何開始使用 Entity Framework Core 來讀取資料庫，這裡將會顯示 Person 資料表內的紀錄清單，並且請按照 LastName , FirstName 來排序。

請按照底下的步驟來進行操作

## 建立練習專案

* 打開 Visual Studio 2019
* 點選 [建立新的專案] 按鈕
* 在 [建立新專案] 對話窗內，選擇 [主控台應用程式 (.NET Core)] 專案樣板
* 在 [設定新的專案] 對話窗內，於 [專案名稱] 欄位內輸入 `DBEntityFrameworkCore`
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

現在 Entity Model 相關資料已經建立完成

由於在 [Models] 資料夾內，建立了一個 [SchoolContext] 類別，這個類別將會指向 SQL Server 上的 [School] 資料庫，也就是說，想要對這個資料庫進行任何操作：新增、讀取、修改、刪除，都可以透過這個 [SchoolContext] 類別來做到；而想要知道有那些 Entity 物件對應到資料庫的那些資料表內，請查看 [SchoolContext] 類別的 DbSet<T> 屬性，在這裡將會有這些 Entity 可以存取 Course , CourseInstructor , Department , OfficeAssignment , OnsiteCourse , Outline , Person , StudentGrade ，同樣的，這些名稱也會出現在該資料庫內的資料表。

```csharp
using System;
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata;

namespace EFCoreReverseEngineering.Models
{
    public partial class SchoolContext : DbContext
    {
        ...
        public virtual DbSet<Course> Course { get; set; }
        public virtual DbSet<CourseInstructor> CourseInstructor { get; set; }
        public virtual DbSet<Department> Department { get; set; }
        public virtual DbSet<OfficeAssignment> OfficeAssignment { get; set; }
        public virtual DbSet<OnsiteCourse> OnsiteCourse { get; set; }
        public virtual DbSet<Outline> Outline { get; set; }
        public virtual DbSet<Person> Person { get; set; }
        public virtual DbSet<StudentGrade> StudentGrade { get; set; }
        ...
    }
}
```

請打開這個 [Program.cs] 檔案，完成底下的程式碼

```csharp
using DBEntityFrameworkCore.Model;
using Microsoft.EntityFrameworkCore;
using System;
using System.Linq;
using System.Threading.Tasks;

namespace DBEntityFrameworkCore
{
    class Program
    {
        static async Task Main(string[] args)
        {
            SchoolContext context = new SchoolContext();
            var people = await context.Person
                .OrderBy(x=>x.LastName)
                .ThenBy(x=>x.FirstName)
                .ToListAsync();
            foreach (var item in people)
            {
                Console.WriteLine($"人員:{item.LastName} {item.FirstName}");
            }
        }
    }
}
```

從上面的程式碼中，可以看到首先建立了一個 SchoolContext 型別的 context 物件，這個物件就代表了遠端的資料庫，當想要讀取 Person 資料表內的紀錄，只需要使用 `context.Person` 這樣的方式，就可以取得 Person 資料表內的紀錄，這樣的用法對於 C# 開發者並不陌生，因為，就把 Person 這個屬性，當作是一個集合 Collection 類型的物件，因為是集合類型，所以在 Person 這個屬性(DbSet)內，就會擁有了多筆的 Person 型別的物件。

另外，對於要使用 Entity Framework Core 來對資料庫紀錄操作的時候，通常會搭配著 [LINQ](https://docs.microsoft.com/zh-tw/dotnet/csharp/programming-guide/concepts/linq?WT.mc_id=DT-MVP-5002220) 功能來進行查詢，對於開發者而言，是不需學習 SQL 語言，就可以存取資料庫了。

因此，因為這裡使用了這樣的敘述，將會把 Person 資料表內的所有紀錄，按照 LastName 先做排序，接著按照 FirstName 來排序，最後顯示在螢幕上。

```csharp
var people = await context.Person
    .OrderBy(x=>x.LastName)
    .ThenBy(x=>x.FirstName)
    .ToListAsync();
```

這裡將會是輸出結果

```
人員:Abercrombie Kim
人員:Alexander Carson
人員:Alonso Meredith
人員:Anand Arturo
人員:Barzdukas Gytis
人員:Browning Meredith
人員:Bryant Carson
人員:Carlson Robyn
人員:Fakhouri Fadi
人員:Gao Erica
人員:Griffin Rachel
人員:Harui Roger
人員:Holt Roger
人員:Jai Damien
人員:Justice Peggy
人員:Kapoor Candace
人員:Li Yan
人員:Lopez Sophia
人員:Martin Randall
人員:Morgan Isaiah
人員:Norman Laura
人員:Olivotto Nino
人員:Powell Carson
人員:Rogers Cody
人員:Serrano Stacy
人員:Shan Alicia
人員:Stewart Jasmine
人員:Suarez Robyn
人員:Tang Wayne
人員:Van Houten Roger
人員:Walker Alexandra
人員:White Anthony
人員:Xu Kristen
人員:Zheng Roger
```
