# Blazor實戰故事經驗分享 1 - 風起雲湧 如何從無到有建立Blazor團隊與採用全端開發方式設計出給上市企業使用的Web系統

## 摘要

這是一個系列文章，說明作者與好友 Steve 如何從無到有打造出一個使用 [Blazor](https://docs.microsoft.com/zh-tw/aspnet/core/blazor/?view=aspnetcore-3.1&WT.mc_id=DT-MVP-5002220) 開發框架技術的3人團隊，沒有前端與後端之分，沒有使用 JavaScript，便可以順利在三個月內設計出 Web 應用程式與 Windows Forms 的平版應用程式，當然，最重要的是將產品交付給客戶，完成此一任務的經驗分享；故事的結局當然是十分完美，要不然也不會有這一系列文章與讀者也不需要繼續來觀看這些文章了。

在這個專案內，共使用到底下的技術與內容 

* Visual Studio 2019
* Github
* [ASP.NET Core](https://docs.microsoft.com/zh-tw/aspnet/core/?view=aspnetcore-3.1&WT.mc_id=DT-MVP-5002220)
* [Blazor Server-Side](https://docs.microsoft.com/en-us/aspnet/core/blazor/hosting-models?view=aspnetcore-3.1#blazor-server&WT.mc_id=DT-MVP-5002220)
* [Entity Framework Core](https://docs.microsoft.com/zh-tw/ef/core/?WT.mc_id=DT-MVP-5002220)
* Dapper
* [Syncfusion for Blazor](https://www.syncfusion.com/blazor-components)
* AutoMapper
* EFCore.BulkExtensions
* ncrontab
* 從取 SQL Server / Oracle / SQLite
* Windows Forms
* RESTful Web API
* 使用者的身分認證與授權(Blazor & Web API)
* Cookie base / JWT base
* Excel 匯入

## 起源

作者於今年四月離開了待了快要八年的公司，不要問我問甚麼要離開、是哪裡不爽快、有甚麼恩怨，沒有，沒有，沒有，甚麼都沒有，要對號入座，請自己挑好位置，最好準備好觀落陰的工具，也許你會有更好的八卦主題。

而在今年因為寫了一本 [Blazor 快速體驗](https://leanpub.com/Blazor-Quick-Overview) 電子書，趁著離職後的閒暇時間，也把這本書的各個實作過程，錄製成為 Youtube 影片 [Blazor 快速體驗 之 線上操作教學影片](https://www.youtube.com/watch?v=SfXJf7Q3dg4&list=PLUHhEA6x1LH-MTqXJTNElSPkYFTMFHvsi)，有興趣的人，可以[訂閱本頻道](https://www.youtube.com/channel/UCowlBYQMT3O6whJwNHkFGVA?view_as=subscriber)。

作者從 2018 年底開始接觸 Blazor，一邊研究 Blazor 實驗性版本，也一邊寫出許多關於 [Blazor 應用的文章](https://csharpkh.blogspot.com/search/label/Blazor) 約 28 篇，而在研究過程中，讓我想起想要能夠充分掌握 Blazor 的設計精神與發揮這個開發框架的價值，據以傳統的 Desktop 設計經驗，也就是類似 Client / Server 經驗的人，更能夠進入狀況；這讓我突然想起快要 30 年的好友，我們當初一起研究與使用 Delphi 這套開發工具 (而我本身是從事於 Delphi 的推廣與教育訓練課程的規劃與授課) ，那就是人稱 陳少 的 Steve。

找個時間到高軟與陳少碰面，談起了 Blazor & Delphi 的經驗，瞬間找回當初在研究 Delphi 的時光，而我們兩人都是從來無緣接觸過 Web 開發的人，而我本身對於 JavaScript 這個程式語言，從他一發表來，都沒有任何好感 (我依稀記得，當初 Borland 也有推出關於 JavaScript 的開發工具，無奈無緣)，所以，當我一提起 Blaror，陳少瞬間了解 Blazor 的價值，也看出對於許多開發者而言，透過 Blazor 這套框架，是可以獨立一個人開發出 Web 應用程式，根本不需要做甚麼前後端分離的做法。

對於為什麼要做到前後端分離，這需要各位看官自行去了解當初的背景，以及這樣的作法是要解決甚麼樣的問題，並且透過這樣的做法會帶來甚麼樣的好處，以及要付出甚麼樣的代價。想想，當初在使用 Client / Server 開發環境下，每個 Delphi 程式設計師不都是甚麼前後端工作都一手包辦，而且這些專案也都有開發出來，那麼，為什麼現在不能夠在 Web 開發環境下，一樣可以做到這樣的事情呢？ (你知道問題在哪裡嗎？沒錯，那就是 JavaScript，我從來沒有喜歡過這個語言與其相關環境，更不用說他在不同瀏覽器下造成的許多不一致現象與許多詭異問題)。

好了，既然英雄有志一同，就請他根據 先看過 [Blazor 快速體驗](https://leanpub.com/Blazor-Quick-Overview) 電子書 與 [Blazor 快速體驗 之 線上操作教學影片](https://www.youtube.com/watch?v=SfXJf7Q3dg4&list=PLUHhEA6x1LH-MTqXJTNElSPkYFTMFHvsi) ，之後，順理成章也就在五月份針對如何使用 Blazor 開發的過程與相關應用，針對陳少做了一場一天的訓練，不過，請注意到，陳少其實對於 C# 並不是十分孰悉，不過，對於整場的訓練下來之後，陳少發現到原來他也可以輕鬆自在的開發出 Web 應用程式。

所謂的機緣就是這麼奇妙，突然間 陳少 被邀請到另外一家公司內，負責整個專案研發部門的管理工作；然而，實際的狀況是這個部門就僅有他一個人，他要負責把原先公司使用 PHP & MySQL 開發出來的多套產品，重新設計出來(為什麼不去改善原有的 PHP 系統，這是因為原有系統已經值得打掉重練的五顆星評價)，更為險峻的是原有系統的許多開發文件、系統架構都極為混亂與不見了；看樣子是無法看著原先的程式碼，逐一轉換成為另外一套系統，順而求其次，只好看著原有使用手冊畫面，猜測與理解原先系統背後的運作方式。另外，這些產品都有 Windows Forms 的用戶端與手機 App 也需要與新系統能夠串接起來，當然，二話不說， Windows Forms 與原生 Android / iOS App 也是需要打掉重練。

現在，問題來了，要選擇甚麼樣的開發環境來進行這樣的專案開發，要找到多少人來重新打造這樣的系統出來，當初接收到的指示是期望能夠在 2020 年底，打造出新一代的產品出來。

既然身為好友，現在又沒有工作，我就想說要不要我來幫忙一起設計與打造出這樣的團隊與開發出這樣的產品，對我而言，這是一個自我挑戰，也是驗證我對於 Blazor 這個開發框架的眼光是否準確，不過，我僅能夠每周投入個1~2天的時間，幫忙一起進行這樣的挑戰，，其餘的時間，我想要把之前沒有開發完成的 .NET C# 相關教育訓練教材完成，例如，多執行緒方面方面的課程(現在這個課程已經整理到快要 400 頁的程度了，這其中還包含了來自於微軟官方技術支援的方想)；因此，這個 Blazor實戰故事經驗分享 應該要就此開始囉...

## 人員招募

由於陳少與我都是高雄人，他所待的公司自然是在高雄，不過，對於要找到可以具備擁有開發 Blazor 的程式設計師，在高雄真的滿大的挑戰；經過分析過後，我決定使用之前設計 Xamarin.Forms 教育訓練課程的技巧，在最短的時間內，設計出讓具備 .NET / C# 能力的程式設計師，就算沒有甚麼 Web 開發經驗的人，也可以進行 Blazor 專案開發。

>不要以為我瘋了，或者我話唬爛，因為，最後證明這樣做法，可以讓程式設計師都可以進行 Blazor 專案開發，並且開發出交付給客戶產品。

另外，有鑑於整個系統的 UI/UX 的設計考量

* UI
  
  讓畫面更好看

* UX

  讓系統更好用

根據上面的準則，首先要解決 UI 上的問題，雖然，在 Blazor 還在實驗性階段，就已經有許多免費的 UI 套件，當然，到現在為止，原先的 UI 套件更加的豐富與完整，而且還有更多的 UI 套件也陸續推出；為了避免選擇或者整合不同 UI 套件的困擾，以及這些 UI 套件在升級過程中，產生了中斷變更 Break-Change 的問題，陳少決定使用 Syncfusion 這家的 Blazor 付費套件。

其實，選擇哪家付費 Blazor UI 套件並不是十分難選擇，先分析這個專案會有哪些的 UI ，這家的原件是否可以滿足需求即可，最重要的是，雖然 Blazor 正式版本在今年才推出，幾乎市面上聽得過名字的有名元件公司，都有提供 Blazor 的原件，這些元件當然都是原生的，若沒有問題，使用上完全使用 C# 來設定與呼叫，幾乎不會用到任何一行 JavaScript 程式碼。

>從這裡可以看的出來，各家付費元件廠商早在 Blazor 在實驗性產品的時候，就已經推出相對應的 Blazor 加值元件，從此可以看的出來，各家元件廠商都相當看好 Blazor 這個開發平台，要不然，賠錢的生意沒人做，殺頭的生意有人做，各家元件廠商是不會傻到投入一個沒有前途的開發平台元件上。

有了讓應用程式是否好看的決定性要素，接下來要解決是否好用的因素，這裡將會設計一個通用型的 CRUD SS 與新增、取得、更新、刪除、搜尋、排序等應用的程式寫法，這部分也在決定要使用 Syncfusion元件之後，於五月底完成相關範例程式碼的開發工作。

在五月中旬，招募了第一個程式設計師，本身大約有業界 3 年開發經驗，之前是在上海工作，因為疫情無法回去上海，因此，決定到 陳少 這裡來上班；這位程式設計師的經歷大部分是在 Windows Forms 上，對於 Web 專案開發的經驗並沒有很多；雖然人已經招募進來了，因為新的辦公室還在裝潢中，因此，大約等到六月中旬，他才開始進駐新辦公室，準備進行相關 Blazor 開發訓練。

另外一位工程是在六月的時候招募進來，剛好是前一位程式設計師的同校、同屆、同科系、但沒有同班，算是有機緣可以一起共事；這樣程式設計之前是在新竹友達駐廠，使用 ASP.NET MVC 負責開發系統。

這兩位工程是共同的特色那就是

* 都有 .NET C# 的開發經驗
* 程式設計年資都差不多
* 不過，開發領域不相同
* 沒有任何 Blazor 的開發經驗
* 沒有任何 ASP.NET Core 的開發經驗
* 對於相關近代的設計模式或者技術，例如 相依性注入、非同步程式設計等技能，也沒有經驗
* 對於資料庫存取、觀念與 Entity Framework Core 也沒有經驗
* JavaScript 應該功力不深

至於第三位程式設計師大約是在八月份招募進來，大約有超過八年以上的經驗，不過，最近10年的工作，大多是在前端開發，使用 Vue 開發框架，當然，JavaScript 有一定的經驗，不過，對於 .NET / C# 則是很早之前工作上用到的。

好了，就在 7月11日，陳少接收到公司總經理的訊息，詢問是否可以提前先完成已經成交的某上市公司案子，這個案子預計要能夠在 10月27日交付，並且所開發出來的產品，要能夠經過客戶的確認與驗收，更悲慘的是，這個產品案子要能夠存 SQL Server、Oracle、SQLite 三個資料庫系統，而且也需要包含一個 Windows Forms 的前台應用程式，透過 SQLite 資料庫做到離線與同步處理的能力。

## 結論

看到這裡，若你是陳少，你會怎麼解決這樣的問題

* 要能夠使用新的工具與平台，打造出公司新一代的產品
* 大約只有三個月的時間可以來支配
* 相關要開發的程式設計師具備的經驗與經歷都不盡相同
* 要開發該產品的領域知識 Domain-How 也還沒摸清楚
* 這個新一代的產品，究竟要使用甚麼樣的開發技術
* 所選定的技術，以現有人力，是否足夠呢
* 對於所選定的技術架構，對於日後開發產品上又會有何影響

相信大家對 陳少 一定捏一把冷汗，他究竟會如何解決這些問題？反觀，若是你承接了這樣的工作，你又會怎麼處理。

最重要的是，這樣的運作模式決定之後，對於日後相關新系統的開發與人力支配，又會該如何成長，如何做到最佳經濟成本，獲得絕佳的效益。

而且，未來將會要做到超全端 Hyper Full Stack 的開發模式，讓一個工程師可以獨力完成 Web App的前後台系統、跨平台手機 App，這也是要納入思考的。

繼續觀賞第二部分

## 相關文章

[Blazor實戰故事經驗分享 1 - 風起雲湧 如何從無到有建立Blazor團隊與採用全端開發方式設計出給上市企業使用的Web系統](https://csharpkh.blogspot.com/2020/11/Blazor-Server-Side-Full-Stack-Case-Study-JavaScript-Story.html)

[Blazor實戰故事經驗分享 2 - 風雲再現 探究 Blazor 可以快速開發出來內部細節](https://csharpkh.blogspot.com/2020/11/Blazor-Server-Side-Layer-Data-Case-Study-Story.html)



