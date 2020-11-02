# Blazor實戰故事經驗分享 - 風起雲湧 如何從無到有建立Blazor團隊與採用全端開發方式設計出給上市企業使用的Web系統

## 摘要

這是一個系列文章，說明作者與好友 Steve 如何從無到有打造出一個使用 Blazor 開發框架技術的3人團隊，沒有前端與後端之分，沒有使用 JavaScript，便可以順利在三個月內設計出 Web 應用程式與 Windows Forms 的平版應用程式，當然，最重要的是將產品交付給客戶，完成此一任務的經驗分享；故事的結局當然是十分完美，要不然也不會有這一系列文章與讀者也不需要繼續來觀看這些文章了。

在這個專案內，共使用到底下的技術與內容 

* Visual Studio 2019
* Github
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

找個時間到高軟與陳少碰面，談起了 Blazor & Delphi 的經驗，瞬間找回當初在研究 Delphi 的時光，而我們兩人都是從來無緣接觸過 Web 開發的人，而我本身對於 JavaScript 這個程式語言，從他一發表來，都沒有任何好感 (我依稀記得，當初 Borland 也有推出關於 JavaScript 的開發工具，無奈無緣)，所以，當我一提起 Blaror，陳少瞬間了解 Blazor 的價值，也看出對於許多開發者而言，透過 Blazor 這套框架，是可以獨立一個人開發出 Web 應用程式，根本不需要做甚麼前後端分離的做法。

對於為什麼要做到前後端分離，這需要各位看官自行去了解當初的背景，以及這樣的作法是要解決甚麼樣的問題，並且透過這樣的做法會帶來甚麼樣的好處，以及要付出甚麼樣的代價。想想，當初在使用 Client / Server 開發環境下，每個 Delphi 程式設計師不都是甚麼前後端工作都一手包辦，而且這些專案也都有開發出來，那麼，為什麼現在不能夠在 Web 開發環境下，一樣可以做到這樣的事情呢？ (你知道問題在哪裡嗎？沒錯，那就是 JavaScript，我從來沒有喜歡過這個語言與其相關環境，更不用說他在不同瀏覽器下造成的許多不一致現象與許多詭異問題)。

好了，既然英雄有志一同，就請他根據 先看過 [Blazor 快速體驗](https://leanpub.com/Blazor-Quick-Overview) 電子書 與 [Blazor 快速體驗 之 線上操作教學影片](https://www.youtube.com/watch?v=SfXJf7Q3dg4&list=PLUHhEA6x1LH-MTqXJTNElSPkYFTMFHvsi) ，之後，順理成章也就在五月份針對如何使用 Blazor 開發的過程與相關應用，針對陳少做了一場一天的訓練，不過，請注意到，陳少其實對於 C# 並不是十分孰悉，不過，對於整場的訓練下來之後，陳少發現到原來他也可以輕鬆自在的開發出 Web 應用程式。

所謂的機緣就是這麼奇妙，突然間 陳少 被邀請到另外一家公司內，負責整個專案研發部門的管理工作；然而，實際的狀況是這個部門就僅有他一個人，他要負責把原先公司使用 PHP & MySQL 開發出來的多套產品，重新設計出來(為什麼不去改善原有的 PHP 系統，這是因為原有系統已經值得打掉重練的五顆星評價)，更為險峻的是原有系統的許多開發文件、系統架構都極為混亂與不見了；看樣子是無法看著原先的程式碼，逐一轉換成為另外一套系統，順而求其次，只好看著原有使用手冊畫面，猜測與理解原先系統背後的運作方式。另外，這些產品都有 Windows Forms 的用戶端與手機 App 也需要與新系統能夠串接起來，當然，二話不說， Windows Forms 與原生 Android / iOS App 也是需要打掉重練。

現在，問題來了，要選擇甚麼樣的開發環境來進行這樣的專案開發，要找到多少人來重新打造這樣的系統出來，當初接收到的指示是期望能夠在 2020 年底，打造出新一代的產品出來。


