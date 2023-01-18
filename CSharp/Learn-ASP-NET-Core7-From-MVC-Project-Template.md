# 由 ASP.NET Core MVC (Model-View-Controller) 專案範本來理解與學習 ASP.NET Core

在這個系列文章 由專案範本來理解與學習 ASP.NET Core 架構 中，將會透過 Visual Studio 2022 內建的幾個專案範本所產生的程式碼，了解 ASP.NET Core 7 的運作方式與在這些專案中的差異在哪裡？

1. [ 由 ASP.NET Core 空白專案範本來理解與學習 ASP.NET Core](https://csharpkh.blogspot.com/2023/01/Learn-ASP-NET-Core7-From-Empty-Project-Template.html)
2. [由 ASP.NET Core Web API 專案範本來理解與學習 ASP.NET Core](https://csharpkh.blogspot.com/2023/01/Learn-ASP-NET-Core7-From-Web-API-Project-Template.html)
3. 由 ASP.NET Core 應用程式 (Model-View-Controller) 專案範本來理解與學習 ASP.NET Core

對於更多關於 ASP.NET Core 7 的說明內容，可以參考 [ASP.NET Core 基本概念的概觀](https://learn.microsoft.com/zh-tw/aspnet/core/fundamentals/?view=aspnetcore-7.0&tabs=windows&WT.mc_id=DT-MVP-5002220)

在上一篇文章中，使用 ASP.NET Core Web API 範本建立一個專案，並且從這個專案原始碼中，了解到 ASP.NET Core 的運作方式，並且，了解到 Web API 專案範本與 空白 專案範本的差異在哪裡，不過，透過 Web API 專案範本，卻可以提供這個 Web 網站具有 Web API 服務功能。

在這篇文章中，將會建立同樣名稱的專案，不過將會採用 ASP.NET Core Web 應用程式 (Model-View-Controller) 類型的專案範本，並且來比較這個 Web API 類型的專案與空白類型的專案有何不同。

首先先來建立一個 ASP.NET Core Web 應用程式 (Model-View-Controller) 專案，請依照底下說明來建立這個專案

* 打開 Visual Studio 2022 IDE 應用程式
* 從 [Visual Studio 2022] 對話窗中，點選右下方的 [建立新的專案] 按鈕
* 在 [建立新專案] 對話窗右半部
  * 切換 [所有語言 (L)] 下拉選單控制項為 [C#]
  * 切換 [所有專案類型 (T)] 下拉選單控制項為 [Web]
* 在中間的專案範本清單中，找到並且點選 [ASP.NET Core Web 應用程式 (Model-View-Controller)] 專案範本選項

  ![Visual Studio 2022 建立新專案](../Images/ANC7/anc986.png)

* 點選右下角的 [下一步] 按鈕
* 在 [設定新的專案] 對話窗
* 找到 [專案名稱] 欄位，輸入 `ASPNETCore7` 作為專案名稱
  ![Visual Studio 2022 設定新的專案](../Images/ANC7/anc985.png)
* 點選右下角的 [下一步] 按鈕
* 現在將會看到 [其他資訊] 對話窗
* 找到 [使用控制器 (取消勾選已使用最低 API)] 檢查盒，注意，一定需要勾選這個選項，因為，在此先來觀察傳統的使用 API Controller 建立的專案長成甚麼樣子。
  ![Visual Studio 2022 其他資訊](../Images/ANC7/anc984.png)
* 請點選右下角的 [建立] 按鈕

完成專案創建之後，將會看到 Visual Studio 2022 將這個新專案開啟，從 [方案總管] 視窗內，可以看到這個專案內所建立的檔案

![](../Images/ANC7/anc983.png)

首先，按下 F5 看看這個專案的執行結果會呈現甚麼樣貌

在 Visual Studio 2022 應用程式的上方，將會看到一個綠色三角形，請點選該綠色三角形來執行這個專案。

![](../Images/ANC7/anc995.png)

一旦專案編譯、建置完成後，瀏覽器將會出現這個網頁

![](../Images/ANC7/anc982.png)

在網頁上看出，並沒有像空白專案會顯示出 `Hello World!` 這個文字 或者 Web API 專案顯示出 Swagger 的網頁，這裡將會出現一個採用 MVC 開發框架所設計出來的網頁內容，接下來透過程式碼內容，來理解為什麼可以生成這樣的網頁內容。

## 從專案內容來理解為什麼會有這樣的執行結果

先來看一下 ASP.NET Core Web 應用程式 (Model-View-Controller) 專案的專案定義宣告檔案，也就是 [.csproj]。

可以透過直接點擊方案總管內的專案名稱節點，也就是 [ASPNETCore7] 或者 使用檔案總管找到 [ASPNETCore7.csproj] 這個檔案，便可以看到關於這個專案的定義宣告檔案。

底下的將會是空白專案的 [.csproj] 檔案內容

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net7.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

</Project>
```

這裡可以看到此專案定義宣告檔案，將會與 空白 專案所用到相同，都是採用 [Microsoft.NET.Sdk.Web] 這個 SDK，並且將會採用 .NET7 作為目的框架。既然都是相同，在這裡的專案可以使用 MVC 方式進行開發與顯示出更加豐富的網頁，這樣是怎麼做到的呢？

對於 [Properties] 資料夾內的 [launchSettings.json] 檔案 與 [appsettings.json] 這兩個檔案，其實與空白專案內的用法與意義是相同的。

接下來要來比較程式進入點差異，也就是這個 Web 網站的核心程式碼，這些程式碼將會在 [Program.cs] 檔案內，其內容如下：

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddControllersWithViews();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    // The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts.
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();

app.UseRouting();

app.UseAuthorization();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

從 Web 應用程式 (Model-View-Controller) 專案內產生的 [Program.cs] 檔案，同樣的使用 `var builder = WebApplication.CreateBuilder(args);` 敘述來建立一個 [WebApplicationBuilder] 型別物件到 [builder] 變數內。

在 Web 應用程式 (Model-View-Controller) 專案內，對於這個 builder 並沒有做其他的呼叫，僅呼叫 `builder.Build()` 方法，取得一個型別為 [WebApplication] 物件到 [app] 變數內`,也就是說，這裡沒有明確地進行要進行各種服務註冊到相依性注入容器內。

接下來就是要進行管道與中介軟體 Middleware 的宣告

使用 app.Environment.IsDevelopment() 來判斷此專案此時是否在開發模式下運行，若不是在開發模式下執行，則會加入這兩個中介軟體

`app.UseExceptionHandler` ： 新增例外狀況處理中介軟體

`app.UseHsts()` ： 新增強制執行 HTTPS

接著將會宣告

`app.UseHttpsRedirection()` ： 將 HTTP 要求重新導向至 HTTPS

`app.UseStaticFiles()` ： 可以取得靜態檔案內容

`app.UseRouting()` ： 將路由比對新增至中介軟體管線

`app.UseAuthorization()` ： 授權中介軟可授權使用者存取安全資源

`app.MapControllerRoute(...)` ： 建立單一路由

`app.Run()` ： 執行應用程式並封鎖呼叫執行緒，直到主機關閉為止

可以透過上面的程式碼與 空白 和 Web API 專案來比較，其實就是加入幾行程式碼，瞬間就可以將專案宣告成為採用 MVC 框架方式來設計與執行，其實，這一切功能都包含在 Microsoft.NET.Sdk.Web 內了。

現在來看看在 [Controllers] 資料夾內，有 [HomeController.cs] 檔案，這個檔案內容如下：

```csharp
using ASPNETCore7.Models;
using Microsoft.AspNetCore.Mvc;
using System.Diagnostics;

namespace ASPNETCore7.Controllers
{
    public class HomeController : Controller
    {
        private readonly ILogger<HomeController> _logger;

        public HomeController(ILogger<HomeController> logger)
        {
            _logger = logger;
        }

        public IActionResult Index()
        {
            return View();
        }

        public IActionResult Privacy()
        {
            return View();
        }

        [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
        public IActionResult Error()
        {
            return View(new ErrorViewModel { RequestId = Activity.Current?.Id ?? HttpContext.TraceIdentifier });
        }
    }
}
```

這個控制器繼承 [Controller] 類別，而這個類別又繼承了 [ControllerBase]，也就是，如同 Web API 專案一樣，這些控制器都是繼承於 [ControllerBase] 這個類別

依據 ASP.NET Core MVC 開發框架的設計方式，將會在這個 [HomeController] 類別內設計許多 [Action] 方法，這樣就可以在網頁中使用 URL 來呼叫了。

對於 [Views] & [Models] 這兩個資料夾，也是依據 MVC 開發框架的命名慣例所存在的，用來設計相關檢視頁面內容和資料模型類別。

另外，因為這是 MVC 網頁設計專案，因此，將會額外增加一個 [wwwroot] 目錄，這裡將會儲存網站會用到的靜態資源檔案，例如： .js JavaScript 檔案、 .css 、圖片等檔案

