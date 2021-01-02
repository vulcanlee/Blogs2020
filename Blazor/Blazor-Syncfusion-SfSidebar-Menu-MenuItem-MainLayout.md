# 使用 Syncfusion SfSidebar 建立一個版面功能表選項清單

當建立一個 Blazor 專案時，並且開始執行的時候，會在瀏覽器畫面的左邊看到功能選項清單，對於 Syncfusion 提供了一個功能表的元件， SfMenu，可以做到類似彈出只功能選項清單的效果，當然也可以使用這個元件做出其他不同的需求出來，在這篇文章中就來看看如何使用 SfMenu 元件來設計出具有彈出子功能選項清單的效果。

這個說明專案的原始碼位於 [bzSidebar](https://github.com/vulcanlee/CSharp2020/tree/master/bzSidebar)

## 建立 Blazor Server-Side 的專案

* 打開 Visual Studio 2019
* 點選右下方的 [建立新的專案] 按鈕
* [建立新專案] 對話窗將會顯示在螢幕上
* 從[建立新專案] 對話窗的中間區域，找到 [Blazor 應用程式] 這個專案樣板選項，並且選擇這個項目
* 點選右下角的 [下一步] 按鈕
* 現在 [設定新的專案] 對話窗將會出現
* 請在這個對話窗內，輸入適當的 [專案名稱] 、 [位置] 、 [解決方案名稱]

  在這裡請輸入 [專案名稱] 為 `bzSidebar`

* 完成後，請點選 [建立] 按鈕
* 當出現 [建立新的 Blazor 應用程式] 對話窗的時候
* 請選擇最新版本的 .NET Core 與 [Blazor 伺服器應用程式]
* 完成後，請點選 [建立] 按鈕

稍微等會一段時間，Blazor 專案將會建立起來

## 進行 Syncfusion 元件的安裝

* 滑鼠右擊 Blazor 專案的 [相依性] 節點
* 選擇 [管理 NuGet 套件]
* 切換到 [瀏覽] 標籤頁次
* 搜尋 `Syncfusion.Blazor` 這個元件名稱
* 選擇搜尋到的 [Syncfusion.Blazor] 元件，並且安裝起來

## 進行 Syncfusion 元件的設定

* 打開專案根目錄下的 [Startup.cs] 這個檔案
* 找到 [ConfigureServices] 這個方法
* 在這個方法的最後面，加入底下程式碼，已完成 Blazor 元件會用到的服務註冊

```csharp
#region Syncfusion 元件的服務註冊
services.AddSyncfusionBlazor();
#endregion
```

* 在同一個檔案內，找到 [Configure] 這個方法
* 在這個方法的最前面，加入底下程式碼，宣告合法授權的金鑰 (License Key)

```csharp
#region 宣告所使用 Syncfusion for Blazor 元件的使用授權碼
Syncfusion.Licensing.SyncfusionLicenseProvider.RegisterLicense("YOUR LICENSE KEY");
#endregion
```

* 打開 [Pages] 資料夾內的 [_Host.cshtml] 檔案
* 在 `<head>` 標籤內，加入需要的 CSS 宣告，如底下內容
 
  >若沒有加入底下的宣告，將無法正常看到 Syncfusion 的元件樣貌

```XML
<link href="_content/Syncfusion.Blazor/styles/bootstrap4.css" rel="stylesheet" />
```

## 建立 SfDataManager 會用到的類別

* 在專案的 [Shared] 資料夾
* 打開 [MainLayout.razor] 檔案
* 使用底下程式碼替換到 [MainLayout.razor] 檔案內容

找到 <!-- sidebar element declaration --> 這個註解文字，在底下的程式碼就是用來宣告彈出功能表的標記語言代碼，其中在這裡使用了 SfMenu 來宣告這個功能表要有哪一些項目，期資料定義來源將會是在 Items="@MainMenuItems" 這個屬性來宣告，因此在底下可以看到 MainMenuItems 這個 .NET 物件相關初始化地址 。

MainMenuItems 是一個集合屬性，使用這樣的程式碼來宣告：public List<MenuItem> MainMenuItems = new List<MenuItem>。因此在這裡需要使用 MenuItem 所生成的物件來定義功能表上所看到的每一個項目，而在這個 MenuItem 物件內，可以透過 Text 屬性來定義該功能表項目所要顯示的文字，並且該物件那還有一個 Items 屬性，這也是一個  List<MenuItem> 集合型別的物件，所以便可以在這裡宣 定義出仔功能表的各個項目 。

底下為執行結果的螢幕截圖

![](../Images/CS2020-9923.png)

```XML
@inherits LayoutComponentBase
@using Syncfusion.Blazor
@using Syncfusion.Blazor.Navigations
@inject NavigationManager NavigationManager
@*<div class="sidebar">
        <NavMenu />
    </div>

    <div class="main">
        <div class="top-row px-4">
            <a href="https://docs.microsoft.com/aspnet/" target="_blank">About</a>
        </div>

        <div class="content px-4">
            @Body
        </div>
    </div>*@

<div id="wrapper" style="width:100%">
    <div class="col-lg-12 col-sm-12 col-md-12">
        <div class="header-section dock-menu" id="header">
            <ul class="header-list">
                <li id="hamburger" class="icon-menu icon list" @onclick="@Toggle"></li>
                <input type="text" placeholder="Search..." class="search-icon list">
                <li class="right-header list">
                    <div class="horizontal-menu">
                        <SfMenu CssClass="dock-menu" Items="@AccountMenuItems" Orientation="@Orientation">
                            @*<MenuEvents ItemSelected="itemSelected"></MenuEvents>*@
                        </SfMenu>
                    </div>
                </li>
                <li class="right-header list support">Support</li>
                <li class="right-header list tour">Tour</li>
            </ul>
        </div>
        <!-- sidebar element declaration -->
        <SfSidebar HtmlAttributes="@HtmlAttribute" Target=".main-content" Width="220px" DockSize="50px" EnableDock="true" @ref="Sidebar">
            <ChildContent>
                <div class="main-menu">
                    <p class="main-menu-header">功能清單</p>
                    <div>
                        <SfMenu CssClass="dock-menu" Items="@MainMenuItems"
                                Orientation="@VerOrientation">
                            <MenuEvents ItemSelected="itemSelected"></MenuEvents>
                        </SfMenu>
                    </div>
                </div>
                <div class="action">
                    <p class="main-menu-header">ACTION</p>
                    <button class="e-btn action-btn" id="action-button">+ Button</button>
                </div>
            </ChildContent>
        </SfSidebar>
        <!-- end of sidebar element -->
        <!-- main content declaration -->
        <div class="main-content" id="maintext">
            <div class="content">
                @Body
            </div>
        </div>
        <!--end of main content declaration -->
    </div>
</div>

@code {
    SfSidebar Sidebar;
    public Orientation Orientation = Orientation.Horizontal;
    public Orientation VerOrientation = Orientation.Vertical;
    Dictionary<string, object> HtmlAttribute = new Dictionary<string, object>()
{
        {"class", "sidebar-menu" }
    };
    public List<MenuItem> AccountMenuItems = new List<MenuItem> {
        new MenuItem {
            Text = "Account",
            Items = new List<MenuItem> {
                new MenuItem { Text = "Profile" ,Url= $"Welcome/e.Item.Text"},
                new MenuItem { Text = "Sign out" }
            }
        }
    };

    public List<MenuItem> MainMenuItems = new List<MenuItem>{
        new MenuItem {
            Text = "Overview", IconCss = "icon-user icon",
                Items = new List<MenuItem> {
                    new MenuItem{ Text = "All Data"  },
                    new MenuItem{ Text = "Category2" },
                    new MenuItem{ Text = "Category3" }
                }
            },
        new MenuItem {
            Text = "Notification",
            IconCss = "icon-bell-alt icon",
            Items = new List<MenuItem> {
                new MenuItem{ Text = "Change Profile" },
                new MenuItem{ Text = "Add Name" },
                new MenuItem{ Text = "Add Details" }
            }
        },
        new MenuItem {
            Text = "Info",
            IconCss = "icon-tag icon",
            Items = new List<MenuItem> {
                new MenuItem{ Text = "Message" },
                new MenuItem{ Text = "Facebook" },
                new MenuItem{ Text = "Twitter" }
            }
        },
        new MenuItem {
            Text = "Comments",
            IconCss = "icon-comment-inv-alt2 icon",
            Items = new List<MenuItem> {
                new MenuItem{ Text = "Category1 " },
                new MenuItem{ Text = "Category2" },
                new MenuItem{ Text = "Category3" }
            }
        },
        new MenuItem {
            Text = "Bookmarks",
            IconCss = "icon-bookmark icon",
            Items = new List<MenuItem> {
                new MenuItem{ Text = "All Comments" },
                new MenuItem{ Text = "Add Comments" },
                new MenuItem{ Text = "Delete Comments" }
            }
        },
        new MenuItem {
            Text = "Images",
            IconCss = "icon-picture icon",
            Items = new List<MenuItem> {
                new MenuItem{ Text = "Add Name" },
                new MenuItem{ Text = "Add Mobile Number" }
            }
        },
        new MenuItem {
            Text = "Users",
            IconCss = "icon-user icon",
            Items = new List<MenuItem> {
                new MenuItem{ Text = "Mobile User" },
                new MenuItem{ Text = "Laptop User" },
                new MenuItem{ Text = "Desktop User" }
            }
        },
        new MenuItem {
            Text = "Settings",
            IconCss = "icon-eye icon",
            Items = new List<MenuItem> {
                new MenuItem{ Text = "Change Profile" },
                new MenuItem{ Text = "Add Name" },
                new MenuItem{ Text = "Add Details" }
            }
        },
        new MenuItem {
            Text = "Info",
            IconCss = "icon-tag icon",
            Items = new List<MenuItem> {
                new MenuItem{ Text = "Facebook" },
                new MenuItem{ Text = "Mobile" }
            }
        }
    };

    public void Toggle()
    {
        this.Sidebar.Toggle();
    }

    private void itemSelected(MenuEventArgs e)
    {
        if (e.Item.Items.Count == 0)
        {
            string Url = $"Welcome/{e.Item.Text}";
            NavigationManager.NavigateTo(Url);
        }
    }}

<style>
    /* header-section styles */
    #header.header-section,
    #header .search-icon {
        height: 50px;
    }

    #header #hamburger.icon-menu {
        font-size: 24px;
        float: left;
        line-height: 50px;
    }

    #header .right-header {
        height: 35px;
        padding: 7px;
        float: right;
    }

    #header .list {
        list-style: none;
        cursor: pointer;
        font-size: 16px;
        line-height: 35px;
    }

    #header .header-list {
        padding-left: 15px;
        margin: 0;
    }

    @@media(max-width:500px) {

        #header .right-header.list.support,
        #header .right-header.list.tour {
            display: none;
        }
    }

    /* text input styles */
    #header .search-icon {
        float: left;
        padding-left: 15px;
        border: 0px solid #33383e !important;
        background-color: #33383e;
        cursor: text;
        width: 10em;
    }

        #header .search-icon:focus {
            outline: none;
            cursor: default;
        }

    /* end of text input styles */
    /* end of header-section styles */

    /* content area styles */
    #maintext.main-content {
        height: 100vh;
        z-index: 1000;
    }

    #maintext .content {
        /*margin-top: 230px;
        text-align: center;
        font-size: 32px;
        color: #1784c7;*/
    }

    /* end of content area styles */

    /* menu styles */
    /* horizontal-menu styles */
    #header .header-list .horizontal-menu .e-menu-item {
        height: 35px;
        vertical-align: middle;
        font-size: 16px;
        line-height: 35px;
    }

    #header .e-menu-item .e-caret {
        line-height: 35px;
    }

    /* end of horizontal-menu styles */
    /* vertical-menu styles */

    .sidebar-menu .e-menu-wrapper ul .e-menu-item.e-menu-caret-icon {
        width: 220px;
    }

    .sidebar-menu .e-menu-wrapper ul .e-menu-item:hover, .e-menu-wrapper ul .e-menu-item.e-focused:hover {
        background-color: #3e454c !important;
    }

    .e-menu-wrapper ul .e-menu-item.e-selected {
        background-color: #3e454c !important;
    }
    /* end of vertical-menu styles */
    /* end of menu styles */
    /* Sidebar styles */
    /* docksidebar styles */
    .e-menu-wrapper ul .e-menu-item .e-caret,
    #header .search-icon,
    .sidebar-menu .action-btn,
    #header .e-menu-item .e-caret,
    .e-menu-wrapper ul .e-menu-item {
        color: #fff !important;
    }

    .e-close .e-menu-wrapper ul .e-menu-item {
        width: 50px;
    }

    .e-close ul .e-menu-item.e-menu-caret-icon {
        padding-right: 12px;
    }

    .sidebar-menu.e-dock.e-close .e-menu-wrapper ul .e-menu-item .e-caret,
    .sidebar-menu.e-dock.e-close .main-menu-header,
    .sidebar-menu.e-dock.e-close .action-btn {
        display: none;
    }

    .sidebar-menu.e-dock.e-close .e-menu-wrapper ul .e-menu-item.e-menu-caret-icon,
    .sidebar-menu.e-dock.e-close .e-menu-wrapper ul.e-vertical {
        min-width: 0;
        width: 50px !important;
    }

    .sidebar-menu.e-dock.e-close .e-menu-wrapper ul.e-menu {
        font-size: 0;
    }

    .sidebar-menu.e-dock.e-close .e-menu-item .e-menu-icon {
        font-size: 20px;
        padding: 0;
    }

    .e-menu-wrapper ul .e-menu-item.e-focused {
        background-color: #33383e !important;
    }

    .sidebar-menu, #header ul, .e-menu-wrapper, .e-menu-wrapper ul {
        background-color: #33383e !important;
        color: #fff !important;
        overflow: hidden;
    }
        /* end of docksidebar styles */
        /*end of  Sidebar styles */
        /*main-menu-header  styles */
        .sidebar-menu .main-menu-header {
            padding: 4px 0 0 18px;
            color: #656a70;
        }

        /*end of main-menu-header  styles */

        /*button styles */
        .sidebar-menu .action-btn {
            margin-left: 16px;
            width: 165px;
            height: 30px;
            font-size: 13px;
            border-radius: 5px;
        }

        .sidebar-menu .action-btn {
            background-color: #1784c7;
        }

    /*end of button styles */
    /* custom code start */
    .center {
        text-align: center;
        display: none;
        font-size: 13px;
        font-weight: 400;
        margin-top: 20px;
    }

    .sb-content-tab .center {
        display: block;
    }

    #sb-content-header {
        display: none
    }

    .sb-content-section {
        border: 0;
    }

    .col-md-12, body {
        padding: 0;
    }

    .sidebar-menu {
        margin-left: -1px;
    }
    /*body styles */
    body {
        margin: 0;
        overflow-y: hidden;
        font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
        -webkit-tap-highlight-color: transparent;
    }

    /*end of body styles */
    /* custom code end */
    /*icon styles */
    @@font-face {
        font-family: 'fontello';
        src: url('data:application/octet-stream;base64,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') format('truetype');
    }

    .sidebar-menu .icon-up-hand:before {
        content: '\e801';
    }

    .sidebar-menu .icon-bell-alt:before {
        content: '\e802';
    }

    .sidebar-menu .icon-user:before {
        content: '\e803';
    }

    .sidebar-menu .icon-picture:before {
        content: '\e804';
    }

    .sidebar-menu .icon-comment-alt:before {
        content: '\e805';
    }

    .sidebar-menu .icon-tag:before {
        content: '\e806';
    }

    .sidebar-menu .icon-comment-inv-alt2:before {
        content: '\e807';
    }

    .sidebar-menu .icon-eye:before {
        content: '\e808';
    }

    .sidebar-menu .icon-globe:before {
        content: '\e809';
    }

    .sidebar-menu .icon-bookmark-empty:before {
        content: '\e810';
    }

    .sidebar-menu .icon-bookmark:before {
        content: '\e811';
    }

    #header .icon-menu:before {
        content: '\e812';
    }

    .sidebar-menu .icon,
    #header #hamburger.icon-menu {
        font-family: 'fontello';
    }

    .sidebar-menu .e-menu-icon::before {
        color: #656a70;
    }

    /*icon styles */
    /* custom code start */
    .sf-new .sb-header,
    .sf-new .sb-bread-crumb,
    .sf-new #action-description,
    .sf-new .sb-action-description,
    .sf-new .e-tab-header,
    .sf-new .description-section,
    .sf-new #description-section,
    .sf-new #description,
    .sf-new #navigation-btn,
    .sf-new .sb-toolbar-splitter,
    .sf-new .sb-footer, .sf-new #left-sidebar, .sb-component-name {
        display: none
    }

    .sf-new .sb-right-pane.e-view {
        margin-left: 0px !important;
    }

    .sb-action-description.sb-rightpane-padding {
        padding-bottom: 0;
    }

    .description-section {
        padding-top: 0;
    }

    #content-tab.sb-content-tab {
        height: 100% !important;
    }

    .sf-new .container-fluid,
    .sf-new .container-fluid .control-section,
    #sidebar-section, description-section sb-rightpane-padding {
        padding: 0;
    }

    .sb-component-name.sb-rightpane-padding {
        margin-top: -56px;
    }

    .sb-right-pane.e-view {
        left: 0;
        padding-left: 0;
        padding-right: 0;
        top: 0;
        overflow-y: hidden;
    }

    .sb-desktop-wrapper {
        height: 100%;
    }

    .sb-component-name h1 {
        padding-top: 0;
    }

    .sf-new .sb-content.e-view {
        top: 0;
    }
    /* custom code end */
    @@font-face {
        font-family: 'e-icons';
        src: url(data:application/x-font-ttf;charset=utf-8;base64,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) format('truetype');
        font-weight: normal;
        font-style: normal;
    }
    /* end of newTab support */

</style>
```

* 滑鼠右擊專案的 [Data] 資料夾
* 點選 [加入] > [類別] 選項
* 在 [新增項目] 對話窗的 [名稱] 欄位輸入 `OrderService`
* 點選 [新增] 按鈕
* 使用底下程式碼替換到 [OrderService.cs] 檔案內容

```csharp
public class OrderService
{
    List<Order> Items { get; set; } = new List<Order>();
    public OrderService()
    {
        Items = Enumerable.Range(1, 75).Select(x => new Order()
        {
            OrderID = 1000 + x,
            CustomerID = (new string[] { "ALFKI", "ANANTR", "ANTON", "BLONP", "BOLID" })[new Random().Next(5)],
            Freight = 2.1 * x,
        }).ToList();
    }
 
    public IEnumerable<Order> GetItems()
    {
        return Items;
    }
    public Task<Order> GetAsync(int orderID)
    {
        var item = Items.FirstOrDefault(x => x.OrderID == orderID);
        return Task.FromResult(item);
    }
    public Task AddAsync(Order order)
    {
        Items.Add(order);
        return Task.CompletedTask;
    }
    public Task UpdateAsync(Order order)
    {
        var item = Items.FirstOrDefault(x => x.OrderID == order.OrderID);
        if (item != null)
        {
            item.Freight = order.Freight;
            item.CustomerID = order.CustomerID;
        }
        return Task.CompletedTask;
    }
    public Task RemoveAsync(Order order)
    {
        var item = Items.FirstOrDefault(x => x.OrderID == order.OrderID);
        if (item != null)
        {
            Items.Remove(item);
        }
        return Task.CompletedTask;
    }
    public void RemoveSomeRecordAsync()
    {
        Items.RemoveRange(1, 50);
        Items[0].CustomerID = "Vulcan Lee";
    }
}
```

在上面的程式碼中，設計了一個 OrderService 類別，在 OrderService 建構函式內將會隨機產生出 75 筆的 Order 類別的紀錄，這些紀錄將會要顯示在網頁上面。

該 OrderService 類別提供一個 [GetItems] 方法將會取得該服務內所有的集合物件，另外，將會有相關 查詢、更新、新增與刪除 的方法定義在這個類別內，這些 CRUD 的方法分別是 GetAsync, UpdateAsync, AddAsync, DeleteAsync，使用者可以點選 DataGrid 元件上的任何一筆紀錄，便可以進行刪除動作，也可以點選新增按鈕，進行新增一筆紀錄的能力。這些方法將會於使用者對 DataGrid 紀錄做一度的時候來呼叫。

對於這個 [RemoveSomeRecordAsync()] 沒有參數的方法，則是會讓網頁上的 [CRUD] 按鈕被點選之後，可以呼叫這個方法，以便刪除 50 筆紀錄，接著將第一筆紀錄，更新為 Vulcan Lee

* 滑鼠右擊專案的 [Shared] 資料夾
* 點選 [加入] > [類別] 選項
* 在 [新增項目] 對話窗的 [Razor元件] 欄位輸入 `OrderServiceAdapter`
* 點選 [新增] 按鈕
* 使用底下程式碼替換到 [OrderServiceAdapter.razor] 檔案內容

```XML
@using Syncfusion.Blazor;
@using Syncfusion.Blazor.Data;
@using Newtonsoft.Json
@using bzsfCustomBindingCRUD.Data

@inherits DataAdaptor<OrderService>

<CascadingValue Value="@this">
    @ChildContent
</CascadingValue>

@code {
    [Parameter]
    [JsonIgnore]
    public RenderFragment ChildContent { get; set; }

    // Performs data Read operation
    public override async Task<object> ReadAsync(DataManagerRequest dataManagerRequest, string key = null)
    {
        IEnumerable<Order> DataSource = (IEnumerable<Order>)Service.GetItems();
        if (dataManagerRequest.Search != null && dataManagerRequest.Search.Count > 0)
        {
            // Searching
            DataSource = DataOperations.PerformSearching(DataSource, dataManagerRequest.Search);
        }
        if (dataManagerRequest.Sorted != null && dataManagerRequest.Sorted.Count > 0)
        {
            // Sorting
            DataSource = DataOperations.PerformSorting(DataSource, dataManagerRequest.Sorted);
        }
        if (dataManagerRequest.Where != null && dataManagerRequest.Where.Count > 0)
        {
            // Filtering
            DataSource = DataOperations.PerformFiltering(DataSource, dataManagerRequest.Where, dataManagerRequest.Where[0].Operator);
        }
        int count = DataSource.Cast<Order>().Count();
        if (dataManagerRequest.Skip != 0)
        {
            //Paging
            DataSource = DataOperations.PerformSkip(DataSource, dataManagerRequest.Skip);
        }
        if (dataManagerRequest.Take != 0)
        {
            DataSource = DataOperations.PerformTake(DataSource, dataManagerRequest.Take);
        }
        var item = dataManagerRequest.RequiresCounts ? new DataResult() { Result = DataSource, Count = count } : (object)DataSource;
        await Task.Yield();
        return item;
    }

    public override async Task<object> InsertAsync(DataManager dataManager, object data, string key)
    {
        await Service.AddAsync(data as Order);
        return data;
    }

    public override async Task<object> UpdateAsync(DataManager dataManager, object data, string keyfield, string key)
    {
        await Service.UpdateAsync(data as Order);
        return data;
    }

    public override async Task<object> RemoveAsync(DataManager dataManager, object data, string keyField, string key)
    {
        var item = await Service.GetAsync(Convert.ToInt32(data));
        await Service.RemoveAsync(item);
        return data;
    }
}
```

