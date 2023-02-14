# 在 .NET MAUI 專案內使用 AutoMapper 套件，將 DTO 轉換成為 Model

![](../Images/0MAUI/Maui9964.png)

在進行 .NET MAUI 專案開發的時候，通常會使用 Web API 的方式與外部系統進行通訊，進行相關資料的處理工作，為了要讓這兩個系統(.NET MAUI App 與 Web API 的系統)能夠具有鬆散耦合的協同運作方式，對於要進行 請求 Request 與 回應 Response 的資料模型，會抽取出來另外進行設計，兩個系統將會透過這個 DTO , Data Transfer Object 資料傳輸物件模型定義內容，進行彼此間的溝通。可是，對於各自系統內，將會有屬於自己的資料模型、商業模型、檢視模型等等，這個時候若自行進行這些模型物件的轉換工作，將會是相當耗費時間與人力成本的，因此，可以透過類似 AutoMapper 這樣的套件來完成這樣的需求，不用重新再次發明輪子。

AutoMapper 是一個 .NET C# 中的物件映射套件，它可以自動將一個類別的屬性映射到另一個類別的屬性。這個套件可以幫助您省去手動映射類別屬性的麻煩，並且可以大大簡化您的程式碼。

使用 AutoMapper，您只需要建立一個映射配置檔案，然後就可以讓 AutoMapper 自動地將一個類別的屬性映射到另一個類別的屬性。例如，如果您有一個 User 類別和一個 UserDTO 類別，您可以使用 AutoMapper 將 User 物件的屬性映射到 UserDTO 物件的屬性，而不需要手動逐個設定屬性。

接下來要來看看如何在 .NET MAUI 專案內，如何使用 AutoMapper 這個套件。

## 建立採用 Prism 開發框架的 MAUI 專案

* 打開 Visual Studio 2022 IDE 應用程式
* 從 [Visual Studio 2022] 對話窗中，點選右下方的 [建立新的專案] 按鈕
* 在 [建立新專案] 對話窗右半部
  * 切換 [所有語言 (L)] 下拉選單控制項為 [C#]
  * 切換 [所有專案類型 (T)] 下拉選單控制項為 [MAUI]
* 在中間的專案範本清單中，找到並且點選 [Vulcan Custom Prism .NET MAUI App] 專案範本選項
* 點選右下角的 [下一步] 按鈕
* 現在顯示出 [設定新的專案] 對話窗
* 在 [專案名稱] 欄位內輸入 `MA54` 作為此專案的名稱
* 請點選右下角的 [建立] 按鈕
* 此時，將會建立一個可以用於 MAUI 開發的專案

## 安裝 AutoMapper 套件

* 滑鼠右擊專案根目錄下的 [相依性] 節點
* 選擇 [管理 NuGet 套件] 選項
* 在 NuGet 視窗內，點選 [瀏覽] 標籤頁次
* 在 [搜尋] 文字輸入盒內，輸入 `AutoMapper.Extensions.Microsoft.DependencyInjection`
* 當搜尋到這個套件，點選這個套件，接著點選右上方的 [安裝] 按鈕，進行這個套件的安裝

  ![](../Images/0MAUI/Maui9960.png)

## 建立 DTO 模型類別

* 滑鼠右擊專案節點
* 在彈出功能清單視窗內，選擇 [加入] > [資料夾]
* 將剛剛產生的新資料夾命名為 `Dtos`
* 滑鼠右擊專案根目錄節點下的 [Dtos]
* 在彈出功能清單視窗內，選擇 [加入] > [類別]
* 在 [新增項目] 視窗的下方 [名稱] 欄位內輸入 `APIResult.cs`
* 點選視窗右下方的 [新增] 按鈕
  
  ![](../Images/0MAUI/Maui9959.png)

* 使用底下程式碼替換掉剛剛建立檔案內容

```csharp
namespace MA54.Dtos;

/// <summary>
/// 呼叫 API 回傳的制式格式
/// </summary>
public class APIResult : ICloneable
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

    #region 介面實作
    public APIResult Clone()
    {
        return ((ICloneable)this).Clone() as APIResult;
    }
    object ICloneable.Clone()
    {
        return this.MemberwiseClone();
    }
    #endregion
}
```

* 滑鼠右擊專案根目錄節點下的 [Dtos]
* 在彈出功能清單視窗內，選擇 [加入] > [類別]
* 在 [新增項目] 視窗的下方 [名稱] 欄位內輸入 `ProductDto.cs`
* 點選視窗右下方的 [新增] 按鈕
* 使用底下程式碼替換掉剛剛建立檔案內容

```csharp
namespace MA54.Dtos;

public class ProductDto : ICloneable
{
    public int Id { get; set; }
    public string Name { get; set; }
    public short ModelYear { get; set; }
    public decimal ListPrice { get; set; }

    #region 介面實作
    public ProductDto Clone()
    {
        return ((ICloneable)this).Clone() as ProductDto;
    }
    object ICloneable.Clone()
    {
        return this.MemberwiseClone();
    }
    #endregion
}
```

## 建立 產品模型 類別

* 滑鼠右擊專案節點
* 在彈出功能清單視窗內，選擇 [加入] > [資料夾]
* 將剛剛產生的新資料夾命名為 `Models`
* 滑鼠右擊專案根目錄節點下的 [Models]
* 在彈出功能清單視窗內，選擇 [加入] > [類別]
* 在 [新增項目] 視窗的下方 [名稱] 欄位內輸入 `Product.cs`
* 點選視窗右下方的 [新增] 按鈕
* 使用底下程式碼替換掉剛剛建立檔案內容

```csharp
using CommunityToolkit.Mvvm.ComponentModel;

namespace MA54.Models;

public partial class Product : ObservableObject, ICloneable
{
    [ObservableProperty]
    int id = 0;
    [ObservableProperty]
    string name = string.Empty;
    [ObservableProperty]
    short modelYear = 0;
    [ObservableProperty]
    decimal listPrice = 0;

    #region 介面實作
    public Product Clone()
    {
        return ((ICloneable)this).Clone() as Product;
    }
    object ICloneable.Clone()
    {
        return this.MemberwiseClone();
    }
    #endregion
}
```

## 建立 AutoMapper 設定 類別

* 滑鼠右擊專案節點
* 在彈出功能清單視窗內，選擇 [加入] > [資料夾]
* 將剛剛產生的新資料夾命名為 `Helpers`
* 滑鼠右擊專案根目錄節點下的 [Helpers]
* 在彈出功能清單視窗內，選擇 [加入] > [類別]
* 在 [新增項目] 視窗的下方 [名稱] 欄位內輸入 `AutoMapping.cs`
* 點選視窗右下方的 [新增] 按鈕
* 使用底下程式碼替換掉剛剛建立檔案內容

```csharp
using AutoMapper;
using MA54.Dtos;
using MA54.Models;

namespace MA54.Helpers;

public class AutoMapping : Profile
{
    public AutoMapping()
    {
        #region DTO - Model 對應關係宣告
        CreateMap<Product, ProductDto>();
        CreateMap<ProductDto, Product>();
        #endregion
    }
}
```

## 在程式進入點宣告 AutoMapper 服務

* 在專案根目錄下，找到並且打開 [MauiProgram.cs] 檔案
* 在該檔案最上方，加入底下的命名空間宣告

```csharp
using MA53.Helpers;
```

* 找到 `var builder = MauiApp.CreateBuilder();` 敘述
* 在其下方加入底下程式碼

```csharp
#region AutoMapper 服務註冊
builder.Services.AddAutoMapper(c => c.AddProfile<AutoMapping>());
#endregion
```

* 底下將會是完成後的程式碼

```csharp
using MA54.ViewModels;
using MA54.Views;
using MA54.Helpers;

namespace MA54;

public static class MauiProgram
{
    public static MauiApp CreateMauiApp()
    {
        var builder = MauiApp.CreateBuilder();

        #region AutoMapper 服務註冊
        builder.Services.AddAutoMapper(c => c.AddProfile<AutoMapping>());
        #endregion

        builder
            .UseMauiApp<App>()
            .UsePrism(prism =>
            {

                prism.RegisterTypes(container =>
                      {
                          container.RegisterForNavigation<MainPage, MainPageViewModel>();
                      })
                     .OnInitialized(() =>
                      {
                          // Do some initializations here
                      })
                     .OnAppStart(async navigationService =>
                     {
                         // Navigate to First page of this App
                         var result = await navigationService
                         .NavigateAsync("NavigationPage/MainPage");
                         if (!result.Success)
                         {
                             System.Diagnostics.Debugger.Break();
                         }
                     });
            })
            .ConfigureFonts(fonts =>
            {
                fonts.AddFont("OpenSans-Regular.ttf", "OpenSansRegular");
                fonts.AddFont("OpenSans-Semibold.ttf", "OpenSansSemibold");
            });

        return builder.Build();
    }
}
```

## 開始使用 AutoMapper 功能

* 在專案根目錄下，打開 [ViewModels] > [MainPageViewModel.cs] 檔案
* 在程式碼最上方加入底下命名空間的宣告

```csharp
using MA54.Dtos;
using MA54.Models;
```

* 使用底下程式碼，建立一個 [IMapper] 型別的私有欄位

```csharp
private readonly IMapper mapper;
```

* 找到這個 ViewModel 類別的建構式，也就是 `public MainPageViewModel(INavigationService navigationService)`
* 將這個建構式使用底下程式碼來取代

```csharp
public MainPageViewModel(INavigationService navigationService,
    IMapper mapper)
{
    this.navigationService = navigationService;
    this.mapper = mapper;
}
```

* 找到 `private void Count()` 方法宣告
* 將這個 Count() 方法程式碼，使用底下程式碼來替換

```csharp
private async Task Count()
{
    APIResult apiReslut = new();
    Text = "請稍後 ...";
    HttpClient client = new HttpClient();
    var responseMessage = await client
        .GetAsync("https://blazortw.azurewebsites.net/api/SampleAutoMapper");
    apiReslut = await responseMessage.Content.ReadFromJsonAsync<APIResult>();
    if (responseMessage.IsSuccessStatusCode)
    {
        if (apiReslut.Status == true)
        {
            List<ProductDto> productDtos = JsonConvert
                .DeserializeObject<List<ProductDto>>(
                apiReslut.Payload.ToString());
            List<Product> products = mapper.Map<List<Product>>(productDtos);
            Text = $"取得 Product 筆數 : {products.Count}";

            foreach (var item in products)
            {
                Text += $",{item.Name}";
            }
        }
    }
}
```

## 執行結果

* 切換到 [Android Emulator] 模式，選擇一個適合的模擬器，開始執行此專案，將會看到底下結果

  ![](../Images/0MAUI/Maui9958.png)

* 點選 [Click me] 按鈕，將會出現底下畫面結果

  ![](../Images/0MAUI/Maui9957.png)



