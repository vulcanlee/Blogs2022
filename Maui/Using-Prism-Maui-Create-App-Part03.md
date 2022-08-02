# 顯示單頁面的集合清單資料

## 建立新的猴子清單頁面 View

* 滑鼠右擊 [Views] 資料夾節點
* 從彈出功能表中，點選 [加入] > [新增項目]
* 當 [新增項目 - PrismMonkey] 對話窗出現後
* 點選對話窗左邊的項目清單 [已安裝] > [C# 項目] > [.NET MAUI]
* 在該對話窗中間區域，選擇 [.NET MAUI ContentPage (XAML)] 這個項目

  > 請不要選擇 [.NET MAUI ContentPage (C#)] ，因為這個項目是採用 C# 語言來開發頁面，而不是採用 XAML 標記宣告語言來設計頁面

* 在此對話窗下方的 [名稱] 欄位內，出入 `MonkeyListPage.xaml`

  ![](../Images/x984.png)
* 點選此對話窗右下方的 [新增] 按鈕

## 建立新的猴子清單頁面檢視類別 ViewModel

* 滑鼠右擊 [ViewModels] 資料夾節點
* 從彈出功能表中，點選 [加入] > [類別]
* 當 [新增項目 - PrismMonkey] 對話窗出現後
* 在此對話窗下方的 [名稱] 欄位內，出入 `MonkeyListPageViewModel.cs`

  ![](../Images/x983.png)
* 點選此對話窗右下方的 [新增] 按鈕

## 註冊該頁面到相依性注入容器內

在以往使用 Prism Template Pack 開發 Xamarin.Forms 專案的時候，這裡要做的事情，其實 Prism Template Pack 這個擴充套件已經幫忙做完了，不過，現在進行 MAUI 專案開發的時候，在這裡需要養成習慣，也就是，每次建立一個新的頁面 View 的時候，需要同時建立一個相對應的 ViewModel 類別，最後，要來把這個剛剛設計好的頁面 View，註冊到 DI / IoC Container 相依性注入容器內

* 在此專案的根目錄下
* 找到並且打開 [PrismStartup.cs] 檔案
* 找到 [RegisterTypes] 這個方法
* 在該方法內加入 `containerRegistry.RegisterForNavigation<MonkeyListPage>();` 敘述

  > 在此是透過 containerRegistry 這個物件，告知相依性注入容器要註冊一個 [MonkeyListPage] 這個頁面，當要進行頁面導航的時候，可以透過這裡個宣告，產生並且注入到需要的類別物件內。

* 底下是完成後的 [PrismStartup.cs] 程式碼內容

```csharp
using PrismMonkey.Services;
using PrismMonkey.Views;

namespace PrismMonkey;

internal static class PrismStartup
{
    public static void Configure(PrismAppBuilder builder)
    {
        builder.RegisterTypes(RegisterTypes)
                .OnAppStart("NavigationPage/MainPage");
    }

    private static void RegisterTypes(IContainerRegistry containerRegistry)
    {
        containerRegistry.RegisterForNavigation<MainPage>()
                     .RegisterInstance(SemanticScreenReader.Default);

        // 註冊 猴子集合紀錄 頁面
        containerRegistry.RegisterForNavigation<MonkeyListPage>();

        // 註冊 猴子服務
        containerRegistry.RegisterSingleton<MonkeyService>();
    }
}
```

## 進行猴子清單頁面的 ViewModel 設計

因為這個專案採用了 Prism.Maui 開發框架來進行設計，並且搭配使用 PropertyChanged.Fody 套件來幫忙產生與注入資料綁定時候需要用到的屬性變更通知的程式碼，因此，首先須要完成這個 ViewModel 類別的基本程式碼架構

* 在 [ViewModels] 資料夾下
* 找到並且打開 [MonkeyListPageViewModel.cs] 檔案
* 使用底下的 C# 程式碼，替換掉原有這個檔案內的內容

首先，進行這個 ViewModel 類別的設計，對於 MonkeyListPageViewModel 這個類別，將會實作 INotifyPropertyChanged 與 INavigationAware 這兩個介面。

前面的 INotifyPropertyChanged 介面是要做到在 MVVM 設計模式下，需要使用資料綁定機制取得屬性變更的通知，也就是說，當 MonkeyListPageViewModel 這個 ViewModel 內的屬性有異動產生的時候，將會觸發 PropertyChanged 事件，通知 View (在這裡將會是頁面 MonkeyListPage 上面所綁定的該屬性的控制項知道這樣的情況已經發生了)

而後者 INavigationAware 這個介面，是屬於 Prism.Maui 這個開發框架下所提供的，這是用於當在進行頁面導航的時候，對於要離開頁這個頁面的時候，將會觸發這個 OnNavigatedFrom 方法，而當要導航進入此頁面的時候，則會觸發與執行這個 OnNavigatedTo 方法。

一般來說，當在 MAUI 內建立一個頁面，通常會需要導航到其他的頁面內，因此，作者在進行 Xamarin.Forms 或者 MAUI 專案開發的時候，通常會把導航服務這個物件先準備好；想要產生這個導航服務 NavigationServer 物件，不用自己來建立，此時，將會透過 DI Container 相依性注入容器來幫助產生即可。

因此，在這裡將會透過建構式注入方式，將 Prism.Maui 提供的 INavigationService 介面要用到的實作執行個體，注入到建構式的參數內，在這個類別內，已經宣告了一個唯讀欄位 `private readonly INavigationService navigationService`，因此，在建構式內，便可以將注入進來參數物件，設定給這個類別的欄位，這樣，對於該類別的其他方法想要進行頁面導航工作的時候，便可以直接使用這個欄位來進行操作。

另外，在這個 MonkeyListPageViewModel 類別內，將會看到有許多的 `#region ... #endregion` 敘述，這裡是提示在未來要進行其他程式碼設計的時候，可以把相關類型的程式碼寫到適當 Region 區域內，方便管理與維護。

```csharp
namespace PrismMonkey.ViewModels
{
    using System.ComponentModel;
    using Prism.Navigation;
    public class MonkeyListPageViewModel : INotifyPropertyChanged, INavigationAware
    {
        // 這裡是實作 INotifyPropertyChanged 介面需要用到的事件成員
        // 這是要用於屬性變更的時候，將會觸發這個事件通知
        public event PropertyChangedEventHandler PropertyChanged;

        #region 透過建構式注入的服務
        // 這是透過建構式注入的頁面導航的實作執行個體
        private readonly INavigationService navigationService;
        #endregion

        #region 在此設計要進行資料綁定的屬性

        #endregion

        #region 在此設計要進行命令物件綁定的屬性

        #endregion

        public MonkeyListPageViewModel(INavigationService navigationService)
        {
            #region 將透過建構式注入進來的物件，指派給這個類別內的欄位或者屬性
            this.navigationService = navigationService;
            #endregion

            #region 在此將命令屬性進行初始化，建立命令物件與指派委派方法

            #endregion
        }

        #region 在此設計該 ViewModel 的其他商業邏輯程式碼

        #endregion

        #region 頁面導航將會觸發的方法
        // 因為實作 INavigationAware 介面，需要建立這個方法
        // 該方法將會用於當要離開此頁面的時候，會被觸發執行
        public void OnNavigatedFrom(INavigationParameters parameters)
        {
        }

        // 因為實作 INavigationAware 介面，需要建立這個方法
        // 該方法將會用於當要導航到此頁面的時候，會被觸發執行
        public void OnNavigatedTo(INavigationParameters parameters)
        {
        }
        #endregion

    }
}
```




