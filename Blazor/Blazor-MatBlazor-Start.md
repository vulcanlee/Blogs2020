# 使用 MatBlazor 來開發 Blazor 專案

[MatBlazor](https://github.com/SamProf/MatBlazor) 提供了相當豐富的 Blazor UI 原件來協助專案開發，這篇文章將會說明如何在 Blazor 專案中來啟用這些 UI 功能。

這個說明專案的原始碼位於 [bzMatBlazorStart](https://github.com/vulcanlee/CSharp2020/tree/master/bzMatBlazorStart)

## 建立 Blazor Server-Side 的專案

* 打開 Visual Studio 2019
* 點選右下方的 [建立新的專案] 按鈕
* [建立新專案] 對話窗將會顯示在螢幕上
* 從[建立新專案] 對話窗的中間區域，找到 [Blazor 應用程式] 這個專案樣板選項，並且選擇這個項目
* 點選右下角的 [下一步] 按鈕
* 現在 [設定新的專案] 對話窗將會出現
* 請在這個對話窗內，輸入適當的 [專案名稱] 、 [位置] 、 [解決方案名稱]

  在這裡請輸入 [專案名稱] 為 `bzMatBlazorStart`

* 完成後，請點選 [建立] 按鈕
* 當出現 [建立新的 Blazor 應用程式] 對話窗的時候
* 請選擇最新版本的 .NET Core 與 [Blazor 伺服器應用程式]
* 完成後，請點選 [建立] 按鈕

稍微等會一段時間，Blazor 專案將會建立起來

## 建立可以取得 EditContext 的元件

* 滑鼠右擊 [Pages] 資料夾
* 選擇 [加入] > [類別]
* 在名稱欄位中，輸入 `InputWatcher`
* 使用底下 C# 程式碼替換掉這個檔案中的內容

```csharp
using Microsoft.AspNetCore.Components;
using Microsoft.AspNetCore.Components.Forms;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace bzEditContext.Pages
{
    public class InputWatcher : ComponentBase
    {
        private EditContext editContext;

        [CascadingParameter]
        protected EditContext EditContext
        {
            get => editContext;
            set
            {
                editContext = value;
                EditContextActionChanged?.Invoke(editContext);
            }
        }

        [Parameter]
        public Action<EditContext> EditContextActionChanged { get; set; }
    }
}
```

這裡建立一個 C# 類別，並且繼承 ComponentBase，使其成為 Blazor 可以使用的元件，當然，也可以建立一個 Razor 元件，完成同樣的工作。

另外，為了要能夠捕捉到 EditForm 元件中 EditContext 物件，在此將會需要使用到 [CascadingParameter](https://docs.microsoft.com/zh-tw/aspnet/core/blazor/components?view=aspnetcore-3.1#cascading-values-and-parameters) 這個串聯式參數屬性宣告，如此才能夠捕捉到 EditContext 物件。

最後，將捕捉到的 EditContext 物件，透過一個委派方法，傳送回給其他的 Blazor 元件內

> ## 注意事項
> 請不要使用 EventCallback 的機制來建立一個回報事件機制，因為，這將會造成無窮循環的執行狀況，因為，若使用 EventCallback 機制，會在接收該事件端的原件，強制再次產生最新的 Render Tree，而因為這樣，又會再度執行到這個 InputWatcher 物件內設定的機制了。

## 建立具有 Form Validation Balzor 頁面

在這裡，初步需求的設計程式碼與 HTML 標記，將會採用 Index.razor 這個檔案來設計，因此

* 打開 [Pages] 資料夾內的 [Index.razor] 檔案
* 使用底下 Razor 元件標記與程式碼，替換該檔案內的原有內容

```html
@page "/"
@using bzEditContext.Data

<h1>Hello, 客製化進行表單資料輸入驗證 EditForm - EditContext !</h1>

<EditForm Model="@Customer">
    <DataAnnotationsValidator />

    <InputWatcher EditContextActionChanged="@OnEditContestChanged" />

    <div class="form-group row mb-1">
        <label class="col-sm-3 col-form-label" for="FirstName">Name:</label>
        <div class="col-sm-9">
            <InputText class="form-control"
                       @bind-Value="@Customer.Name" />
            <ValidationMessage For="@(() => Customer.Name)" />
        </div>
    </div>
    <div class="form-group row mb-1">
        <label class="col-sm-3 col-form-label"
               for="LastName">Street:</label>
        <div class="col-sm-9">
            <InputText class="form-control"
                       @bind-Value="@Customer.Street" />
            <ValidationMessage For="@(() => Customer.Street)" />
        </div>
    </div>
    <div class="form-group row mb-1">
        <label class="col-sm-3 col-form-label"
               for="Birthday">City:</label>
        <div class="col-sm-9">
            <InputText class="form-control"
                       @bind-Value="@Customer.City" />
            <ValidationMessage For="@(() => Customer.City)" />
        </div>
    </div>
</EditForm>

<div>
    <button class="btn btn-primary" @onclick="OnOK">確定</button>
</div>

<div class="display-4 text-secondary">
    @ValidationMessage
</div>

@code {
    public Customer Customer { get; set; }
    public EditContext LocalEditContext { get; set; }
    public string ValidationMessage { get; set; }

    protected override void OnInitialized()
    {
        Customer = new Customer();
    }
    void OnOK()
    {
        if (LocalEditContext.Validate() == false)
        {
            ValidationMessage = "資料有錯，請重新修正";
        }
        else
        {
            ValidationMessage = "表單驗證正確無誤";
        }
    }
    private void OnEditContestChanged(EditContext context)
    {
        LocalEditContext = context;
    }

}
```

在這個 Blazor 頁面中，將剛剛設計的 InputWatcher 元件，放置到 EditForm 元件內，並且記得要綁定一個委派方法，透過該委派方法，將取得的 EditContext 儲存取來，以便可以呼叫執行 Form Validation 資料檢查狀態結果。

該頁面將會要輸入一個 Customer 類別的物件，該類別將會有三個屬性，這些屬性值都需要強制輸入。

在 EditForm 的外部，將會到有個 button 按鈕，當使用者按下這個按鈕之後，將會觸發 OnOK 這個委派方法；該方法將會呼叫 EditContext 的 Validate() 方法，此時，可以根據回傳結果來檢查使用者輸入的資料內容，是否有違反 Form Validation 的定義。
