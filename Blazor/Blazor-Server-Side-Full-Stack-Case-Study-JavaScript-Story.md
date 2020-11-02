# Blazor實戰故事經驗分享 - 風起雲湧 如何從無到有建立Blazor團隊與採用全端開發方式設計出給上市企業使用的Web系統

## 摘要

這是一個系列文章，說明作者與好友 Steve 如何從無到有打造出一個使用 Blazor 開發框架技術的3人團隊，沒有前端與後端之分，沒有使用 JavaScript，便可以順利在三個月內設計出 Web 應用程式與 Windows Forms 的平版應用程式，當然，最重要的是將產品交付給客戶，完成此一任務的經驗分享；故事的結局當然是十分完美，要不然也不會有這一系列文章與讀者也不需要繼續來觀看這些文章了。

在這個專案內，共使用到底下的技術與內容 

* Visual Studio 2019
* ASP.NET Core
* Blazor Server-Side
* Entity Framework Core
* Dapper
* Syncfusion for Blazor
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


