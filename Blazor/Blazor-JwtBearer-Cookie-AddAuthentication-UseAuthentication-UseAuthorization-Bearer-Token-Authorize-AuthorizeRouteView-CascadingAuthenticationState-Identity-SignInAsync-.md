# Blazor Server Side 專案從無到有做到認證 Authentication 與授權 Authorization 的完整攻略教學

這是一篇關於如何在 Blazor Server-Side 的專案內，從無到有的做到使用者的身分驗證與授權的實際操作過程文件，在該文件中首先將會針對如何保護 Blazor 的頁面，也就是可以限制某些頁面僅能夠讓適當授權的使用者可以來存取，當然，沒有通過身分驗證的使用者，當然也是無法存取的，這裡將會使用到 Cookie Based 的方式來實作出來；另外一種就是針對 RESTful Web API 資源進行授權保護，這裡將會使用 JWT ，也就是 Token Based 的方式來做到保護。

關於這裡所討論到的內容，可以參考 [ASP.NET Core Blazor 驗證與授權](https://docs.microsoft.com/zh-tw/aspnet/core/blazor/security/?view=aspnetcore-3.1&WT.mc_id=DT-MVP-5002220)

## 建立 Blazor Server-Side 專案

- 打開 Visual Studio 2019
- 點選 [建立新的專案] 按鈕
- 在 [建立新專案] 對話窗內，選擇 [Blazor應用程式] 專案樣板
- 在 [設定新的專案] 對話窗內，在專案名稱欄位輸入 `Backend`
- 點選 [建立] 按鈕
- 在 [建立新的 Blazor 應用程式] 對話窗內，點選 [Blazor伺服器應用程式]
- 點選 [建立] 按鈕，以便建立這個新專案

## 設定 Blazor 要啟用 Cookie 基礎的身分驗證功能

- 在專案根目錄下，打開 [Startup.cs] 檔案
- 找到 ConfigureServices 方法，在這個方法內加入底下程式碼

```csharp
#region 加入使用 Cookie 認證需要的宣告
services.Configure<CookiePolicyOptions>(options =>
{
    options.CheckConsentNeeded = context => true;
    options.MinimumSameSitePolicy = Microsoft.AspNetCore.Http.SameSiteMode.None;
});
services.AddAuthentication(
    CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie();
#endregion
```

- 找到 Configure 方法
- 找到 `app.UseRouting();` 敘述，在其後面加入底下程式碼

```csharp
#region 指定要使用 Cookie & 使用者認證的中介軟體
app.UseCookiePolicy();
app.UseAuthentication();
#endregion
```

## 設定路由與驗證狀態的設定

- 在專案根目錄下，打開 [App.razor] 檔案
- 找到 [Found] 標籤，將其內容替換成為底下宣告

```XML
<Found Context="routeData">
    <AuthorizeRouteView RouteData="@routeData" DefaultLayout="@typeof(MainLayout)">
        <NotAuthorized>
            <p class="text-danger">你沒有授權可以存取該服務</p>
        </NotAuthorized>
    </AuthorizeRouteView>
</Found>
```

- 找到 [NotFound] 標籤，將其內容替換成為底下宣告

```XML
<NotFound>
    <CascadingAuthenticationState>
        <LayoutView Layout="@typeof(MainLayout)">
            <h2 class="text-danger">找不到此頁面</h2><br>
            <h3 class="text-danger">對不起，未能找到您要的網頁。</h3>
            <h3 class="text-danger">可能該網頁已被移除或被移到其他的網址。</h3>
        </LayoutView>
    </CascadingAuthenticationState>
</NotFound>
```

## 建立登入與登出 Razor 頁面

- 滑鼠右擊 [Pages] 資料夾，點選 [加入] > [Razor頁面]
- 在 [新增 Scaffold 項目] 對話窗內，選擇 [Razor 頁面]
- 點選 [加入] 按鈕
- 在 [新增項目] 對話窗內，在名稱欄位內輸入 `Login`
- 點選 [新增] 按鈕
- 打開 [Login.cshtml] 檔案，替換為底下內容

```XML
@page
@model Backend.Pages.LoginModel
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@{
    ViewData["Title"] = "身分驗證";
}
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width">
    <title>請進行 身分驗證</title>
    <link href="css/bootstrap/bootstrap.min.css" rel="stylesheet" />
    <link href="css/site.css" rel="stylesheet" />
    <style>
        .centered {
            position: fixed;
            top: 50%;
            left: 50%;
            margin-top: -150px;
            margin-left: -250px;
        }
    </style>
</head>
<body>
    <div class="container centered">
        <div class="row ">
            <div class="card " style="width: 30rem;">
                <article class="card-body">
                    <h4 class="card-title text-center mb-4 mt-1">請輸入帳號與密碼</h4>
                    <hr>
                    <p class="text-danger text-center">@Model.Msg</p>
                    <form method="post">
                        <div class="form-group">
                            <div class="input-group">
                                <div class="input-group-prepend">
                                    <span class="input-group-text oi oi-person"> </span>
                                </div>
                                <input class="form-control" asp-for="Username" placeholder="使用者的帳號">
                            </div> <!-- input-group.// -->
                        </div> <!-- form-group// -->
                        <div class="form-group">
                            <div class="input-group">
                                <div class="input-group-prepend">
                                    <span class="input-group-text oi oi-lock-locked"> </span>
                                </div>
                                <input class="form-control" asp-for="Password" placeholder="使用者的密碼" type="@Model.PasswordType">
                            </div> <!-- input-group.// -->
                        </div> <!-- form-group// -->
                        <div class="form-group">
                            <button type="submit" class="btn btn-primary btn-block"> 登入  </button>
                        </div> <!-- form-group// -->
                    </form>
                </article>
            </div>
        </div>
    </div>
</body>
</html>
```

- 打開 [Login.cshtml.cs] 檔案，替換為底下內容

```csharp
using System;
using System.Collections.Generic;
using System.Security.Claims;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace Backend.Pages
{
    [AllowAnonymous]
    public class LoginModel : PageModel
    {
        public LoginModel()
        {
#if DEBUG
            Username = "user";
            Password = "123";
            PasswordType = "";
#endif
        }
        [BindProperty]
        public string Username { get; set; } = "";

        [BindProperty]
        public string Password { get; set; } = "";
        public string PasswordType { get; set; } = "password";
        public string Msg { get; set; }
        public async Task OnGetAsync()
        {
            try
            {
                // 清除已經存在的登入 Cookie 內容
                await HttpContext
                    .SignOutAsync(
                    CookieAuthenticationDefaults.AuthenticationScheme);
            }
            catch { }
        }
        public string ReturnUrl { get; set; }
        public async Task<IActionResult> OnPostAsync()
        {
            if (string.IsNullOrEmpty(Username) == false && string.IsNullOrEmpty(Password) == false)
            {
                bool result = true;
                string msg = "";
                if (Username != "admin" && Username != "user")
                {
                    result = false;
                    Msg = "帳號或密碼不正確";
                }
                //(bool result, string msg, Person person) = await personService.LoginAsync(Username, Password, true);
                if (result == true)
                {
                    string returnUrl = Url.Content("~/");

                    #region 加入這個使用者需要用到的 宣告類型 Claim Type
                    var claims = new List<Claim>
                    {
                        new Claim(ClaimTypes.Role, "User"),
                        new Claim(ClaimTypes.Name, Username),
                    };
                    if (Username == "admin")
                    {
                        claims.Add(new Claim(ClaimTypes.Role, "Administrator"));
                    }
                    #endregion

                    #region 建立 宣告式身分識別
                    // ClaimsIdentity類別是宣告式身分識別的具體執行, 也就是宣告集合所描述的身分識別
                    var claimsIdentity = new ClaimsIdentity(
                        claims, CookieAuthenticationDefaults.AuthenticationScheme);
                    #endregion

                    #region 建立關於認證階段需要儲存的狀態
                    var authProperties = new AuthenticationProperties
                    {
                        IsPersistent = true,
                        RedirectUri = this.Request.Host.Value
                    };
                    #endregion

                    #region 進行使用登入
                    try
                    {
                        await HttpContext.SignInAsync(
                        CookieAuthenticationDefaults.AuthenticationScheme,
                        new ClaimsPrincipal(claimsIdentity),
                        authProperties);
                    }
                    catch (Exception ex)
                    {
                        string error = ex.Message;
                    }
                    #endregion

                    return LocalRedirect(returnUrl);
                }
            }
            else
            {
                Msg = "帳號與密碼不可為空白";
            }
            return Page();
        }
    }
}
```

- 滑鼠右擊 [Pages] 資料夾，點選 [加入] > [Razor頁面]
- 在 [新增 Scaffold 項目] 對話窗內，選擇 [Razor 頁面]
- 點選 [加入] 按鈕
- 在 [新增項目] 對話窗內，在名稱欄位內輸入 `Logout`
- 點選 [新增] 按鈕
- 打開 [Logout.cshtml.cs] 檔案，替換為底下內容

```csharp
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authentication;
using Microsoft.AspNetCore.Authentication.Cookies;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace Backend.Pages
{
    public class LogoutModel : PageModel
    {
        public async Task<IActionResult> OnGetAsync()
        {
            string returnUrl = Url.Content("~/");
            try
            {
                // 清除已經存在的登入 Cookie 內容
                await HttpContext.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
            }
            catch
            {
            }
            return LocalRedirect(Url.Content("~/Login"));
        }
    }
}
```

## 建立登入與登出的 Razor元件

- 滑鼠右擊 [Shared] 資料夾，點選 [加入] > [Razor元件]
- 在 [新增項目] 對話窗內，在名稱欄位內輸入 `SigninView.razor`
- 點選 [新增] 按鈕
- 打開 [SigninView.razor] 檔案，替換為底下內容

```XML
<AuthorizeView>
    <Authorized>
        <span>
            <b>你好, @context.User.Identity.Name!</b>
            <a class="ml-md-auto btn btn-primary"
               href="/Logout"
               target="_top"> 登出 </a>
        </span>
    </Authorized>
    <NotAuthorized>
        <a class="ml-md-auto btn btn-primary"
           href="/Login"
           target="_top"> 登入 </a>
    </NotAuthorized>
</AuthorizeView>

@code {

}
```

## 修正使用建立登入與登出的 Razor元件

- 從 [Shared] 資料夾中，找到並且打開 [MainLayout.razor] 檔案
- 照到該標籤 `<div class="main">`，並將該標籤內容替換為底下內容

```XML
<div class="main">
    <div class="top-row px-4">
        <SigninView />
    </div>

    <div class="content px-4">
        @Body
    </div>
</div>
```

## 執行該專案，觀看使用者認證過程

- 執行這個專案

![](../Images/xBHOL961.png)

- 點選右上角的 [登入] 按鈕
- 在帳號欄位，故意輸入不存在的使用者 `abc`
- 點選 [登入] 按鈕
- 將會出現底下畫面

![](../Images/xBHOL960.png)

- 在帳號欄位，輸入存在的使用者 `user`
- 點選 [登入] 按鈕
- 將會出現底下畫面

![](../Images/xBHOL959.png)

- 點選右上角的 [登出] 按鈕
- 在帳號欄位，輸入存在的使用者 `admin`
- 點選 [登入] 按鈕
- 將會出現底下畫面

![](../Images/xBHOL958.png)

![](../Images/xBHOL957.png)

- 點選右上角的 [登出] 按鈕

## 建立僅限 Administrator 角色可以存取的 Blazor 頁面

- 滑鼠右擊 [Pages] 資料夾，點選 [加入] > [Razor元件]
- 在 [新增項目] 對話窗內，在名稱欄位內輸入 `OnlyAdministrator`
- 點選 [新增] 按鈕
- 打開 [OnlyAdministrator.razor] 檔案，替換為底下內容

```XML
@page "/OnlyAdministrator"
@using Microsoft.AspNetCore.Authentication.Cookies
@attribute [Authorize(Roles = "Administrator")]

<h2 class="text-primary">該頁面僅限具有管理者權限才能看到</h2>

@code {

}
```

## 建立僅限 User 角色可以存取的 Blazor 頁面

- 滑鼠右擊 [Pages] 資料夾，點選 [加入] > [Razor元件]
- 在 [新增項目] 對話窗內，在名稱欄位內輸入 `OnlyUser`
- 點選 [新增] 按鈕
- 打開 [OnlyUser.razor] 檔案，替換為底下內容

```XML
@page "/OnlyAdministrator"
@using Microsoft.AspNetCore.Authentication.Cookies
@attribute [Authorize(Roles = "Administrator")]

<h2 class="text-primary">該頁面僅限具有管理者權限才能看到</h2>

@code {

}
```

## 執行專案，確認頁面有受到保護

- 執行專案
- 確認使用者已經登出
- 開啟網址 https://localhost:5001/OnlyUser
- 確認沒有成功登入的時候，是無法存取 OnlyUser 頁面

![](../Images/xBHOL956.png)

- 開啟網址 https://localhost:5001/OnlyAdministrator
- 確認沒有成功登入的時候，是無法存取 OnlyAdministrator 頁面

![](../Images/xBHOL955.png)

- 請登入 user 使用者
- 開啟網址 https://localhost:5001/OnlyUser
- 確認成功登入 user 時候，是可以存取 OnlyUser 頁面

![](../Images/xBHOL954.png)

- 開啟網址 https://localhost:5001/OnlyAdministrator
- 確認成功登入 user 時候，是無法存取 OnlyAdministrator 頁面

![](../Images/xBHOL953.png)

- 確認使用者已經登出
- 請登入 admin 使用者
- 開啟網址 https://localhost:5001/OnlyAdministrator
- 確認成功登入 admin 時候，是無法存取 OnlyAdministrator 頁面

![](../Images/xBHOL952.png)

- 開啟不存在的網址，例如 https://localhost:5001/Administrator
- 確認會提示該網址所指定的頁面不存在

![](../Images/xBHOL951.png)

## 設定 Blazor 要啟用 JWT Token 基礎的身分授權功能

## 加入 NuGet 套件

- 滑鼠右擊 [Backend] 專案內的 [相依性] 節點
- 在彈出功能表點選 [管理 NuGet 套件] 選項
- 在 [NuGet: Backend] 視窗內，切換到 [瀏覽] 頁次
- 在 [搜尋] 文字輸入盒內，輸入 `Microsoft.AspNetCore.Authentication.JwtBearer`
- 從搜尋結果內找到 [Microsoft.AspNetCore.Authentication.JwtBearer] 這個套件
- 點選右上方的 [安裝] 按鈕，進行安裝這個套件

## 設定 Blazor 要啟用 JWT 基礎的身分驗證功能

- 在專案根目錄下，打開 [Startup.cs] 檔案
- 找到 ConfigureServices 方法，找到 `#region 加入使用 Cookie 認證需要的宣告` ，把這個 region 區塊內的程式碼，替換為底下程式碼

```csharp
#region 加入使用 Cookie 認證需要的宣告
services.Configure<CookiePolicyOptions>(options =>
{
    options.CheckConsentNeeded = context => true;
    options.MinimumSameSitePolicy = Microsoft.AspNetCore.Http.SameSiteMode.None;
});
services.AddAuthentication(
    CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie()
    .AddJwtBearer(JwtBearerDefaults.AuthenticationScheme, options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = Configuration["Tokens:ValidIssuer"],
            ValidAudience = Configuration["Tokens:ValidAudience"],
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(Configuration["Tokens:IssuerSigningKey"])),
            RequireExpirationTime = true,
        };
        options.Events = new JwtBearerEvents()
        {
            OnAuthenticationFailed = context =>
            {
                context.NoResult();
 
                context.Response.StatusCode = 401;
                context.Response.HttpContext.Features.Get<IHttpResponseFeature>().ReasonPhrase = context.Exception.Message;
                Debug.WriteLine("OnAuthenticationFailed: " + context.Exception.Message);
                return Task.CompletedTask;
            },
            OnTokenValidated = context =>
            {
                Console.WriteLine("OnTokenValidated: " +
                    context.SecurityToken);
                return Task.CompletedTask;
            }
 
        };
    });
//JwtBearerDefaults.AuthenticationScheme
#endregion
```

- 找到 ConfigureServices 方法，在這個方法內加入底下程式碼

```csharp
#region 新增控制器和 API 相關功能的支援，但不會加入 views 或 pages
services.AddControllers();
#endregion
 
#region 修正 Web API 的 JSON 處理
services.AddControllers().AddJsonOptions(config =>
{
    config.JsonSerializerOptions.PropertyNamingPolicy = null;
});
#endregion
```

- 找到 Configure 方法
- 找到 `app.UseEndpoints` 敘述，將該敘述修改成為底下程式碼

```csharp
app.UseEndpoints(endpoints =>
{
    #region Adds endpoints for controller actions to the IEndpointRouteBuilder without specifying any routes.
    endpoints.MapControllers();
    #endregion
 
    endpoints.MapBlazorHub();
    endpoints.MapFallbackToPage("/_Host");
});
```

## 修改 appsettings.json

- 在專案根目錄下，打開 [appsettings.json] 檔案
- 修正該檔案的內容如下

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*",
  "Tokens": {
    "ValidIssuer": "XamarinFormsWS.vulcan.net",
    "ValidAudience": "Xamarin.Forms App",
    "JwtExpireMinutes": 15,
    "JwtRefreshExpireDays": 7,
    "IssuerSigningKey": "F-JaNdRgUkXp2s5v8x/A?D(G+KbPeShVmYq3t6w9z$B&E)H@McQfTjWnZr4u7x!A"
  }
}
```

## 建立 API 控制器

- 滑鼠右擊 [Backend] 專案節點
- 從彈出功能表中，選擇 [加入] > [新增資料夾]
- 定義該新資料夾的名稱為 `Controllers`
- 滑鼠右擊 [Backend] 專案 > [Controllers] 資料夾
- 從彈出功能表中，選擇 [加入] > [新增項目]
- 確認在 [新增項目] 對話窗選取 [API 控制器類別 - 空白] 項目
- 在 名稱 欄位，輸入 `OnlyAdministratorController`
- 點選 [新增] 按鈕
- 當 [OnlyAdministratorController.cs] 檔案建立完成後，使用底下程式碼取代剛剛產生的內容

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace Backend.Controllers
{
    [Authorize(AuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme, Roles = "Administrator")]
    [Produces("application/json")]
    [Route("api/[controller]")]
    [ApiController]
    public class OnlyAdministratorController : ControllerBase
    {
        [HttpGet]
        public string Get()
        {
            return "Hello Administrator~~";
        }
    }
}
```

- 滑鼠右擊 [Backend] 專案 > [Controllers] 資料夾
- 從彈出功能表中，選擇 [加入] > [新增項目]
- 確認在 [新增項目] 對話窗選取 [API 控制器類別 - 空白] 項目
- 在 名稱 欄位，輸入 `OnlyUserController`
- 點選 [新增] 按鈕
- 當 [OnlyUserController.cs] 檔案建立完成後，使用底下程式碼取代剛剛產生的內容

```csharp
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;

namespace Backend.Controllers
{
    [Authorize(AuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme, Roles = "User")]
    [Produces("application/json")]
    [Route("api/[controller]")]
    [ApiController]
    public class OnlyUserController : ControllerBase
    {
        [HttpGet]
        public string Get()
        {
            return "Hello User~~";
        }
    }
}
```

## 執行與測試，並修正錯誤

- 執行專案
- 在網址列輸入 https://localhost:5001/api/OnlyUser
- 此時將會看到底下的錯誤畫面

![](../Images/xBHOL950.png)

- 錯誤訊息文字如下

```
An unhandled exception occurred while processing the request.
InvalidOperationException: Endpoint Backend.Controllers.OnlyUserController.Get (Backend) contains authorization metadata, but a middleware was not found that supports authorization.
Configure your application startup by adding app.UseAuthorization() inside the call to Configure(..) in the application startup code. The call to app.UseAuthorization() must appear between app.UseRouting() and app.UseEndpoints(...).
Microsoft.AspNetCore.Routing.EndpointMiddleware.ThrowMissingAuthMiddlewareException(Endpoint endpoint)
```

- 在專案根目錄下，打開 [Startup.cs] 檔案
- 找到 ConfigureServices 方法，找到 `#region 加入使用 Cookie 認證需要的宣告` ，把這個 region 區塊內的程式碼，替換為底下程式碼
- 找到 Configure 方法
- 找到 `app.UseEndpoints` 敘述，在該敘述之前，加入底下程式碼

```csharp
#region 指定使用授權檢查的中介軟體
app.UseAuthorization();
#endregion
```

- 重新執行專案
- 在網址列輸入 https://localhost:5001/api/OnlyUser
- 現在將會看到底下畫面

![](../Images/xBHOL949.png)

```
此頁面目前無法運作
如果問題持續發生，請連絡網站擁有者。
HTTP ERROR 401
```

## 安裝與啟用 Postman

- 開啟 [Postman](https://www.postman.com/) 官網，下載與安裝 Postman 軟體
- 啟動 Postman 軟體
- 在 Postman 的 API 網址列上，輸入 `https://localhost:5001/api/OnlyUser`
- HTTP 動作 (Action) 選擇 [GET]
- 點選 [Send] 按鈕，將會看到底下畫面

![](../Images/xBHOL948.png)

- 點選 Postman 功能表上的 [View] > [Show Postman Console]
- 將會看到如下截圖畫面

![](../Images/xBHOL947.png)

- 點選右上方的 `Show raw log` 橘色文字
- 將會看到如下截圖畫面

![](../Images/xBHOL946.png)

```
GET https://localhost:5001/api/OnlyUser
401
42 ms
Warning: Unable to verify the first certificate
GET /api/OnlyUser HTTP/1.1
User-Agent: PostmanRuntime/7.26.5
Accept: */*
Postman-Token: 02cbc911-64eb-4d8c-83a0-a316d2eea310
Host: localhost:5001
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
HTTP/1.1 401 Unauthorized
Date: Sun, 25 Oct 2020 07:32:27 GMT
Server: Kestrel
Content-Length: 0
WWW-Authenticate: Bearer
```

## 加入取得 JWT Token 的登入頁面

- 滑鼠右擊 [Backend] 專案 
- 從彈出功能表中，選擇 [加入] > [新增資料夾]
- 定義該新資料夾的名稱為 `DTOs`
- 滑鼠右擊 [Backend] 專案 
- 從彈出功能表中，選擇 [加入] > [新增資料夾]
- 定義該新資料夾的名稱為 `Enums`
- 滑鼠右擊 [Backend] 專案 
- 從彈出功能表中，選擇 [加入] > [新增資料夾]
- 定義該新資料夾的名稱為 `Factories`
- 滑鼠右擊 [Backend] 專案 > [Dtos] 資料夾
- 從彈出功能表中，選擇 [加入] > [類別]
- 確認在 [新增項目] 對話窗選取 [類別] 項目
- 在 名稱 欄位，輸入 `APIResult`
- 點選 [新增] 按鈕
- 當 [APIResult.cs] 檔案建立完成後，使用底下程式碼取代剛剛產生的內容

```csharp
/// <summary>
/// 呼叫 API 回傳的制式格式
/// </summary>
public class APIResult
{
    /// <summary>
    /// 此次呼叫 API 是否成功
    /// </summary>
    public bool Status { get; set; } = true;
    public int HTTPStatus { get; set; } = 200;
    public int ErrorCode { get; set; }
    /// <summary>
    /// 呼叫 API 失敗的錯誤訊息
    /// </summary>
    public string Message { get; set; } = "";
    /// <summary>
    /// 呼叫此API所得到的其他內容
    /// </summary>
    public object Payload { get; set; }
}
```

- 滑鼠右擊 [Backend] 專案 > [Dtos] 資料夾
- 從彈出功能表中，選擇 [加入] > [類別]
- 確認在 [新增項目] 對話窗選取 [類別] 項目
- 在 名稱 欄位，輸入 `LoginRequestDTO`
- 點選 [新增] 按鈕
- 當 [LoginRequestDTO.cs] 檔案建立完成後，使用底下程式碼取代剛剛產生的內容

```csharp
public class LoginRequestDTO
{
    [Required]
    public string Account { get; set; }
    [Required]
    public string Password { get; set; }
}
```

- 滑鼠右擊 [Backend] 專案 > [Dtos] 資料夾
- 從彈出功能表中，選擇 [加入] > [類別]
- 確認在 [新增項目] 對話窗選取 [類別] 項目
- 在 名稱 欄位，輸入 `LoginResponseDTO`
- 點選 [新增] 按鈕
- 當 [LoginResponseDTO.cs] 檔案建立完成後，使用底下程式碼取代剛剛產生的內容

```csharp
public class LoginResponseDTO
{
    public int Id { get; set; }
    public string Account { get; set; }
    public string Name { get; set; }
    public string Token { get; set; }
    public int TokenExpireMinutes { get; set; }
    public string RefreshToken { get; set; }
    public int RefreshTokenExpireDays { get; set; }
}
```

- 滑鼠右擊 [Backend] 專案 > [Enums] 資料夾
- 從彈出功能表中，選擇 [加入] > [類別]
- 確認在 [新增項目] 對話窗選取 [類別] 項目
- 在 名稱 欄位，輸入 `ErrorMessageEnum`
- 點選 [新增] 按鈕
- 當 [ErrorMessageEnum.cs] 檔案建立完成後，使用底下程式碼取代剛剛產生的內容

```csharp
public enum ErrorMessageEnum
{
    None = 0,
    SecurityTokenExpiredException,
    帳號或密碼不正確 = 1000,
    權杖中沒有發現指定使用者ID,
    沒有發現指定的該使用者資料,
    傳送過來的資料有問題,
    沒有任何符合資料存在,
    沒有發現指定的請假單,
    權杖Token上標示的使用者與傳送過來的使用者不一致,
    沒有發現指定的請假單類別,
    要更新的紀錄_發生同時存取衝突_已經不存在資料庫上,
    紀錄更新時_發生同時存取衝突,
    紀錄更新所指定ID不一致,
    使用者需要強制登出並重新登入以便進行身分驗證,
    原有密碼不正確,
    新密碼不能為空白,
    沒有發現指定的發票,
    沒有發現指定的發票明細項目,
    Exception = 2147483647,
}
```

- 滑鼠右擊 [Backend] 專案 > [Enums] 資料夾
- 從彈出功能表中，選擇 [加入] > [類別]
- 確認在 [新增項目] 對話窗選取 [類別] 項目
- 在 名稱 欄位，輸入 `ErrorMessageMapping`
- 點選 [新增] 按鈕
- 當 [ErrorMessageMapping.cs] 檔案建立完成後，使用底下程式碼取代剛剛產生的內容

```csharp
public class ErrorMessageMapping
{
    private Dictionary<ErrorMessageEnum, string> ErrorMessages { get; set; }
    private static ErrorMessageMapping instance;
    private ErrorMessageMapping()
    {
        BuildErrorMessages();
    }
 
    private void BuildErrorMessages()
    {
        ErrorMessages = new Dictionary<ErrorMessageEnum, string>();
        ErrorMessages.Add(ErrorMessageEnum.None, "");
        ErrorMessages.Add(ErrorMessageEnum.SecurityTokenExpiredException, "存取權杖可用期限已經逾期超過");
        ErrorMessages.Add(ErrorMessageEnum.權杖中沒有發現指定使用者ID, "權杖中沒有發現指定使用者ID");
        ErrorMessages.Add(ErrorMessageEnum.帳號或密碼不正確, "帳號或密碼不正確");
        ErrorMessages.Add(ErrorMessageEnum.沒有發現指定的該使用者資料, "沒有發現指定的該使用者資料");
        ErrorMessages.Add(ErrorMessageEnum.傳送過來的資料有問題, "傳送過來的資料有問題");
        ErrorMessages.Add(ErrorMessageEnum.沒有任何符合資料存在, "沒有任何符合資料存在");
        ErrorMessages.Add(ErrorMessageEnum.沒有發現指定的請假單, "沒有發現指定的請假單");
        ErrorMessages.Add(ErrorMessageEnum.權杖Token上標示的使用者與傳送過來的使用者不一致, "權杖 Token 上標示的使用者與傳送過來的使用者不一致");
        ErrorMessages.Add(ErrorMessageEnum.沒有發現指定的請假單類別, "沒有發現指定的請假單類別");
        ErrorMessages.Add(ErrorMessageEnum.要更新的紀錄_發生同時存取衝突_已經不存在資料庫上, "要更新的紀錄，發生同時存取衝突，已經不存在資料庫上");
        ErrorMessages.Add(ErrorMessageEnum.紀錄更新時_發生同時存取衝突, "紀錄更新時，發生同時存取衝突");
        ErrorMessages.Add(ErrorMessageEnum.紀錄更新所指定ID不一致, "紀錄更新所指定 ID 不一致");
        ErrorMessages.Add(ErrorMessageEnum.使用者需要強制登出並重新登入以便進行身分驗證, "系統存取政策違反，使用者需要強制登出，並重新登入，以便進行身分驗證");
        ErrorMessages.Add(ErrorMessageEnum.原有密碼不正確, "原有密碼不正確");
        ErrorMessages.Add(ErrorMessageEnum.新密碼不能為空白, "新密碼不能為空白");
        ErrorMessages.Add(ErrorMessageEnum.沒有發現指定的發票, "沒有發現指定的發票");
        ErrorMessages.Add(ErrorMessageEnum.沒有發現指定的發票明細項目, "沒有發現指定的發票明細項目");
        ErrorMessages.Add(ErrorMessageEnum.Exception, "發生例外異常：");
    }
 
    public static ErrorMessageMapping Instance
    {
        get
        {
            // 若 instance 並沒有持有一個單例物件，則需要在這個時候，進行產生出來
            // ?? 若這個 單例物件 需要能夠在多執行緒環境下正確執行，又該如何設計呢？
            if (instance == null)
            {
                instance = new ErrorMessageMapping();
            }
            return instance;
        }
    }
    public string GetErrorMessage(ErrorMessageEnum errorMessageEnum)
    {
        string fooMsg = "";
        if (ErrorMessages.ContainsKey(errorMessageEnum) == true)
        {
            fooMsg = ErrorMessages[errorMessageEnum];
        }
        return fooMsg;
    }
}
```

- 滑鼠右擊 [Backend] 專案 > [Factories] 資料夾
- 從彈出功能表中，選擇 [加入] > [類別]
- 確認在 [新增項目] 對話窗選取 [類別] 項目
- 在 名稱 欄位，輸入 `APIResult`
- 點選 [新增] 按鈕
- 當 [APIResult.cs] 檔案建立完成後，使用底下程式碼取代剛剛產生的內容

```csharp
public static class APIResultFactory
{
    public static APIResult Build(bool aPIResultStatus,
        int statusCodes = StatusCodes.Status200OK, ErrorMessageEnum errorMessageEnum = ErrorMessageEnum.None,
        object payload = null, string exceptionMessage = "", bool replaceExceptionMessage = true)
    {
        APIResult apiResult = new APIResult()
        {
            Status = aPIResultStatus,
            ErrorCode = (int)errorMessageEnum,
            Message = (errorMessageEnum == ErrorMessageEnum.None) ? "" : $"錯誤代碼 {(int)errorMessageEnum}, {ErrorMessageMapping.Instance.GetErrorMessage(errorMessageEnum)}",
            HTTPStatus = statusCodes,
            Payload = payload,
        };
        if (apiResult.ErrorCode == (int)ErrorMessageEnum.Exception)
        {
            apiResult.Message = $"{apiResult.Message}{exceptionMessage}";
        }
        else if (string.IsNullOrEmpty(exceptionMessage) == false)
        {
            if (replaceExceptionMessage == true)
            {
                apiResult.Message = $"{exceptionMessage}";
            }
            else
            {
                apiResult.Message += $"{exceptionMessage}";
            }
        }
        return apiResult;
    }
}
```

- 滑鼠右擊 [Backend] 專案 > [Controllers] 資料夾
- 從彈出功能表中，選擇 [加入] > [新增項目]
- 確認在 [新增項目] 對話窗選取 [API 控制器類別 - 空白] 項目
- 在 名稱 欄位，輸入 `LoginController`
- 點選 [新增] 按鈕
- 當 [LoginController.cs] 檔案建立完成後，使用底下程式碼取代剛剛產生的內容

```csharp
[Produces("application/json")]
[Route("api/[controller]")]
[ApiController]
[AllowAnonymous]
public class LoginController : ControllerBase
{
    private readonly IConfiguration configuration;
    int UserID;
    int TokenVersion;
 
    public LoginController(IConfiguration configuration)
    {
        this.configuration = configuration;
    }
    [AllowAnonymous]
    [HttpPost]
    public async Task<IActionResult> Post(LoginRequestDTO loginRequestDTO)
    {
        if (ModelState.IsValid == false)
        {
            APIResult apiResult = APIResultFactory.Build(false, StatusCodes.Status200OK,
             ErrorMessageEnum.傳送過來的資料有問題);
            return Ok(apiResult);
        }
        if (loginRequestDTO.Account != "admin" && loginRequestDTO.Account != "user")
        {
            APIResult apiResult = APIResultFactory.Build(false, StatusCodes.Status400BadRequest,
             ErrorMessageEnum.帳號或密碼不正確);
            return BadRequest(apiResult);
        }
 
        {
            string token = GenerateToken(loginRequestDTO);
            string refreshToken = GenerateRefreshToken(loginRequestDTO);
 
            LoginResponseDTO LoginResponseDTO = new LoginResponseDTO()
            {
                Account = loginRequestDTO.Account,
                Id = 0,
                Name = loginRequestDTO.Account,
                Token = token,
                TokenExpireMinutes = Convert.ToInt32(configuration["Tokens:JwtExpireMinutes"]),
                RefreshToken = refreshToken,
                RefreshTokenExpireDays = Convert.ToInt32(configuration["Tokens:JwtRefreshExpireDays"]),
            };
 
            APIResult apiResult = APIResultFactory.Build(true, StatusCodes.Status200OK,
                ErrorMessageEnum.None, payload: LoginResponseDTO);
            return Ok(apiResult);
        }
 
    }
 
    [Authorize(Roles = "RefreshToken")]
    [Route("RefreshToken")]
    [HttpGet]
    public async Task<IActionResult> RefreshToken()
    {
        APIResult apiResult;
 
        LoginRequestDTO loginRequestDTO = new LoginRequestDTO()
        {
            Account = User.FindFirst(JwtRegisteredClaimNames.Sid)?.Value,
        };
        string token = GenerateToken(loginRequestDTO);
        string refreshToken = GenerateRefreshToken(loginRequestDTO);
 
        LoginResponseDTO LoginResponseDTO = new LoginResponseDTO()
        {
            Account = loginRequestDTO.Account,
            Id = 0,
            Name = loginRequestDTO.Account,
            Token = token,
            TokenExpireMinutes = Convert.ToInt32(configuration["Tokens:JwtExpireMinutes"]),
            RefreshToken = refreshToken,
            RefreshTokenExpireDays = Convert.ToInt32(configuration["Tokens:JwtRefreshExpireDays"]),
        };
 
        apiResult = APIResultFactory.Build(true, StatusCodes.Status200OK,
           ErrorMessageEnum.None, payload: LoginResponseDTO);
        return Ok(apiResult);
 
    }
 
    public string GenerateToken(LoginRequestDTO loginRequestDTO)
    {
        var claims = new List<Claim>()
        {
            new Claim(JwtRegisteredClaimNames.Sid, loginRequestDTO.Account),
            new Claim(ClaimTypes.Name, loginRequestDTO.Account),
            new Claim(ClaimTypes.Role, "User"),
        };
        if (loginRequestDTO.Account == "admin")
        {
            claims.Add(new Claim(ClaimTypes.Role, "Administrator"));
        }
 
        var token = new JwtSecurityToken
        (
            issuer: configuration["Tokens:ValidIssuer"],
            audience: configuration["Tokens:ValidAudience"],
            claims: claims,
            expires: DateTime.Now.AddMinutes(Convert.ToDouble(configuration["Tokens:JwtExpireMinutes"])),
            //notBefore: DateTime.Now.AddMinutes(-5),
            signingCredentials: new SigningCredentials(new SymmetricSecurityKey
                        (Encoding.UTF8.GetBytes(configuration["Tokens:IssuerSigningKey"])),
                    SecurityAlgorithms.HmacSha512)
        );
        string tokenString = new JwtSecurityTokenHandler().WriteToken(token);
 
        return tokenString;
 
    }
 
    public string GenerateRefreshToken(LoginRequestDTO loginRequestDTO)
    {
        var claims = new[]
        {
            new Claim(JwtRegisteredClaimNames.Sid, loginRequestDTO.Account),
            new Claim(ClaimTypes.Name, loginRequestDTO.Account),
            new Claim(ClaimTypes.Role, "User"),
            new Claim(ClaimTypes.Role, $"RefreshToken"),
        };
 
        var token = new JwtSecurityToken
        (
            issuer: configuration["Tokens:ValidIssuer"],
            audience: configuration["Tokens:ValidAudience"],
            claims: claims,
            expires: DateTime.Now.AddDays(Convert.ToDouble(configuration["Tokens:JwtRefreshExpireDays"])),
            //notBefore: DateTime.Now.AddMinutes(-5),
            signingCredentials: new SigningCredentials(new SymmetricSecurityKey
                        (Encoding.UTF8.GetBytes(configuration["Tokens:IssuerSigningKey"])),
                    SecurityAlgorithms.HmacSha512)
        );
        string tokenString = new JwtSecurityTokenHandler().WriteToken(token);
 
        return tokenString;
 
    }
}
```

## 測試取得 JWT Token 並存取限制保護資源

- 啟動與執行專案
- 在 Postman 中，輸入 URL 為 `https://localhost:5001/api/Login`
- HTTP 動作 (Action) 設定為 POST
- 點選 [Body] 標籤頁次
- 點選 [raw] Radio 按鈕
- 在 [raw] Radio 按鈕最右方的下拉選單，選擇 `JSON`
- 在其下方輸入使用者身分驗證用到的帳號與密碼

```json
{
    "Account" :"admin",
    "Password" :"123"
}
```
- 點選 [Send] 按鈕
- 將會看到成功登入後的回傳結果，如下面節圖

![](../Images/xBHOL945.png)

- 回傳內容如下

```json
{
    "Status": true,
    "HTTPStatus": 200,
    "ErrorCode": 0,
    "Message": "",
    "Payload": {
        "Id": 0,
        "Account": "admin",
        "Name": "admin",
        "Token": "eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJzaWQiOiJhZG1pbiIsImh0dHA6Ly9zY2hlbWFzLnhtbHNvYXAub3JnL3dzLzIwMDUvMDUvaWRlbnRpdHkvY2xhaW1zL25hbWUiOiJhZG1pbiIsImh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vd3MvMjAwOC8wNi9pZGVudGl0eS9jbGFpbXMvcm9sZSI6WyJVc2VyIiwiQWRtaW5pc3RyYXRvciJdLCJleHAiOjE2MDM2MTUwMTgsImlzcyI6IlhhbWFyaW5Gb3Jtc1dTLnZ1bGNhbi5uZXQiLCJhdWQiOiJYYW1hcmluLkZvcm1zIEFwcCJ9.r-bwy7birRotQ6U9vSSvBfTmHk7lvRc0EMxRjP-MkB8oMbLAIFokaWL9yF5LCJ0Oo7hNpP5R6v75UCWEJsLGEQ",
        "TokenExpireMinutes": 15,
        "RefreshToken": "eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJzaWQiOiJhZG1pbiIsImh0dHA6Ly9zY2hlbWFzLnhtbHNvYXAub3JnL3dzLzIwMDUvMDUvaWRlbnRpdHkvY2xhaW1zL25hbWUiOiJhZG1pbiIsImh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vd3MvMjAwOC8wNi9pZGVudGl0eS9jbGFpbXMvcm9sZSI6WyJVc2VyIiwiUmVmcmVzaFRva2VuIl0sImV4cCI6MTYwNDIxODkxOCwiaXNzIjoiWGFtYXJpbkZvcm1zV1MudnVsY2FuLm5ldCIsImF1ZCI6IlhhbWFyaW4uRm9ybXMgQXBwIn0._tntwCVwC0XF2h3TRSJdKnn4Rs-pqOboFFd5pSFATwhMi5TfAbd7nCnLtJWujDRuC6xxWrnGBKjJWjZOUI2igg",
        "RefreshTokenExpireDays": 7
    }
}
```

- 從 [Postman Console] 視窗內，可以看到此次 HTTP 通訊協定的原始內容

```
POST /api/Login HTTP/1.1
Content-Type: application/json
User-Agent: PostmanRuntime/7.26.5
Accept: */*
Postman-Token: 613b2e5b-f617-4f91-a721-8b264577a75f
Host: localhost:5001
Accept-Encoding: gzip, deflate, br
Connection: keep-alive
Content-Length: 52
{
    "Account" :"admin",
    "Password" :"123"
}
HTTP/1.1 200 OK
Date: Sun, 25 Oct 2020 08:21:57 GMT
Content-Type: application/json; charset=utf-8
Server: Kestrel
Transfer-Encoding: chunked
{"Status":true,"HTTPStatus":200,"ErrorCode":0,"Message":"","Payload":{"Id":0,"Account":"admin","Name":"admin","Token":"eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJzaWQiOiJhZG1pbiIsImh0dHA6Ly9zY2hlbWFzLnhtbHNvYXAub3JnL3dzLzIwMDUvMDUvaWRlbnRpdHkvY2xhaW1zL25hbWUiOiJhZG1pbiIsImh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vd3MvMjAwOC8wNi9pZGVudGl0eS9jbGFpbXMvcm9sZSI6WyJVc2VyIiwiQWRtaW5pc3RyYXRvciJdLCJleHAiOjE2MDM2MTUwMTgsImlzcyI6IlhhbWFyaW5Gb3Jtc1dTLnZ1bGNhbi5uZXQiLCJhdWQiOiJYYW1hcmluLkZvcm1zIEFwcCJ9.r-bwy7birRotQ6U9vSSvBfTmHk7lvRc0EMxRjP-MkB8oMbLAIFokaWL9yF5LCJ0Oo7hNpP5R6v75UCWEJsLGEQ","TokenExpireMinutes":15,"RefreshToken":"eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJzaWQiOiJhZG1pbiIsImh0dHA6Ly9zY2hlbWFzLnhtbHNvYXAub3JnL3dzLzIwMDUvMDUvaWRlbnRpdHkvY2xhaW1zL25hbWUiOiJhZG1pbiIsImh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vd3MvMjAwOC8wNi9pZGVudGl0eS9jbGFpbXMvcm9sZSI6WyJVc2VyIiwiUmVmcmVzaFRva2VuIl0sImV4cCI6MTYwNDIxODkxOCwiaXNzIjoiWGFtYXJpbkZvcm1zV1MudnVsY2FuLm5ldCIsImF1ZCI6IlhhbWFyaW4uRm9ybXMgQXBwIn0._tntwCVwC0XF2h3TRSJdKnn4Rs-pqOboFFd5pSFATwhMi5TfAbd7nCnLtJWujDRuC6xxWrnGBKjJWjZOUI2igg","RefreshTokenExpireDays":7}}
```

- 若將使用者身分驗證用到的帳號與密碼宣告為不正確，如下內容

```json
{
    "Account" :"adminXXX",
    "Password" :"123"
}
```

- 將會看到 Postman 執行結果如下

![](../Images/xBHOL944.png)

## 使用 JWT Token 來存取受保護的資源

- 取得剛剛成功登入回傳結果內的 [Token] 欄位值，也就是 `eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJzaWQiOiJhZG1pbiIsImh0dHA6Ly9zY2hlbWFzLnhtbHNvYXAub3JnL3dzLzIwMDUvMDUvaWRlbnRpdHkvY2xhaW1zL25hbWUiOiJhZG1pbiIsImh0dHA6Ly9zY2hlbWFzLm1pY3Jvc29mdC5jb20vd3MvMjAwOC8wNi9pZGVudGl0eS9jbGFpbXMvcm9sZSI6WyJVc2VyIiwiQWRtaW5pc3RyYXRvciJdLCJleHAiOjE2MDM2MTUwMTgsImlzcyI6IlhhbWFyaW5Gb3Jtc1dTLnZ1bGNhbi5uZXQiLCJhdWQiOiJYYW1hcmluLkZvcm1zIEFwcCJ9.r-bwy7birRotQ6U9vSSvBfTmHk7lvRc0EMxRjP-MkB8oMbLAIFokaWL9yF5LCJ0Oo7hNpP5R6v75UCWEJsLGEQ`
- 點選 [Authentication] 標籤頁次
- 從 [TYPE] 下拉選單中選擇 `Bearer Token`
- 將剛剛的 [Token] 內容，輸入到右方的 [Token] 欄位中
- 在 Postman 的 URL 上輸入 `https://localhost:5001/api/OnlyUser`
- HTTP 動作 (Action) 為 [GET]
- 點選 [Send] 按鈕
- 此時，將會看到這個 URL 將會成功的回應

![](../Images/xBHOL943.png)

- 在 Postman 的 URL 上輸入 `https://localhost:5001/api/OnlyAdministrator`
- HTTP 動作 (Action) 為 [GET]
- 點選 [Send] 按鈕
- 此時，將會看到這個 URL 將會成功的回應

![](../Images/xBHOL942.png)

- 將使用者身分驗證用到的帳號與密碼宣告為 user，如下內容

```json
{
    "Account" :"user",
    "Password" :"123"
}
```

- 當成功登入之後，從回傳結果取得 [Token] 的值
- 點選 [Authentication] 標籤頁次
- 從 [TYPE] 下拉選單中選擇 `Bearer Token`
- 將剛剛的 [Token] 內容，輸入到右方的 [Token] 欄位中
- 在 Postman 的 URL 上輸入 `https://localhost:5001/api/OnlyAdministrator`
- 點選 [Send] 按鈕
- 此時，因為該使用者為一般使用者，而不是 Administrator，所以，將會得到 403 的 HTTP 狀態碼，也就是禁止存取，當然，已無法取得這個受保護的資源

![](../Images/xBHOL941.png)




