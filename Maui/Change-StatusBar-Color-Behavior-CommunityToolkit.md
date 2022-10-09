# 使用 CommunityToolkit 提供的 StatusBarBehavior 來進行狀態列的顏色變更

這幾天看到這篇 [Announcing the .NET MAUI Community Toolkit v1.3](https://devblogs.microsoft.com/dotnet/announcing-the-dotnet-maui-community-toolkit-v13/) 文章，其中看到了 StatusBarBehavior 這個行為可以進行底下的操作

StatusBarColor: allows you to specify the color of the status bar background.

StatusBarStyle: allows you to control if the content (text and icons) on the status bar are light, dark or the system default.

這樣的功能太夢幻了，因為有了他，將不再需要進行實作出原生的程式碼來進行狀態列的背景顏色修正

可是，當我開始使用 Prism.maui 專案範本建立一個專案，根據這篇文章 [StatusBarBehavior](https://learn.microsoft.com/zh-tw/dotnet/communitytoolkit/maui/behaviors/statusbar-behavior?tabs=ios) 的說明，修改專案內的 MainPage.xaml 檔案如下所示

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:toolkit="http://schemas.microsoft.com/dotnet/2022/maui/toolkit"
             xmlns:localBehaviors="clr-namespace:mauiStatusBar.Behaviors"
             Title="{Binding Title}"
             x:Class="mauiStatusBar.Views.MainPage">

    <ContentPage.Behaviors>
        <toolkit:StatusBarBehavior StatusBarColor="Fuchsia" StatusBarStyle="LightContent" />
    </ContentPage.Behaviors>

    <ScrollView>
    <VerticalStackLayout
            Spacing="25"
            Padding="30,0"
            VerticalOptions="Center">

      <Image Source="prism.png"
             SemanticProperties.Description="Cute dot net bot waving hi to you!"
             HeightRequest="150"
             HorizontalOptions="Center" />

      <Label Text="Hello, World!"
             SemanticProperties.HeadingLevel="Level1"
             FontSize="32"
             HorizontalOptions="Center" />

      <Label Text="Welcome to Prism for .NET MAUI"
             SemanticProperties.HeadingLevel="Level2"
             SemanticProperties.Description="Welcome to Prism for dot net Multi platform App U I"
             FontSize="18"
             HorizontalOptions="Center" />

      <Button Text="{Binding Text}"
              SemanticProperties.Hint="Counts the number of times you click"
              Command="{Binding CountCommand}"
              HorizontalOptions="Center" />

    </VerticalStackLayout>
  </ScrollView>

</ContentPage>
```

想說應該可以很輕鬆的來使用這個功能，沒想到卻得到這樣的錯誤訊息

System.NotImplementedException: 'Either set MainPage or override CreateWindow.'

為了確認我的作法沒有錯誤，我使用原生的 MAUI 專案範本，建立一個原生 MAUI 專案，把上面的做法重新實作一遍，沒想到卻是正常的。

想說，好吧，那麼我把這個 StatusBarBehavior 原始碼找出來，寫到 Prism.Maui 建立起來的專案，直接引用我複製的 StatusBarBehavior 行為，底下是這個 StatusBarBehavior 程式碼

```csharp
using CommunityToolkit.Maui.Core.Platform;
using CommunityToolkit.Maui.Core;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Runtime.CompilerServices;
using System.Runtime.Versioning;
using System.Text;
using System.Threading.Tasks;

namespace mauiStatusBar.Behaviors
{
    /// <summary>
    /// <see cref="PlatformBehavior{TView,TPlatformView}"/> that controls the Status bar color
    /// </summary>
    [UnsupportedOSPlatform("Windows"), UnsupportedOSPlatform("MacCatalyst"), UnsupportedOSPlatform("MacOS")]
    public class StatusBarBehavior : PlatformBehavior<ContentPage>
    {
        /// <summary>
        /// <see cref="BindableProperty"/> that manages the StatusBarColor property.
        /// </summary>
        public static readonly BindableProperty StatusBarColorProperty =
            BindableProperty.Create(nameof(StatusBarColor), typeof(Color), typeof(StatusBarBehavior), Colors.Transparent);


        /// <summary>
        /// <see cref="BindableProperty"/> that manages the StatusBarColor property.
        /// </summary>
        public static readonly BindableProperty StatusBarStyleProperty =
            BindableProperty.Create(nameof(StatusBarStyle), typeof(StatusBarStyle), typeof(StatusBarBehavior), StatusBarStyle.Default);

        /// <summary>
        /// Property that holds the value of the Status bar color. 
        /// </summary>
        public Color StatusBarColor
        {
            get => (Color)GetValue(StatusBarColorProperty);
            set => SetValue(StatusBarColorProperty, value);
        }

        /// <summary>
        /// Property that holds the value of the Status bar color. 
        /// </summary>
        public StatusBarStyle StatusBarStyle
        {
            get => (StatusBarStyle)GetValue(StatusBarStyleProperty);
            set => SetValue(StatusBarStyleProperty, value);
        }

#if !(WINDOWS || MACCATALYST)

        /// <inheritdoc /> 
#if IOS
	protected override void OnAttachedTo(ContentPage bindable, UIKit.UIView platformView)
#elif ANDROID
        protected override void OnAttachedTo(ContentPage bindable, Android.Views.View platformView)
#else
	protected override void OnAttachedTo(Page bindable, object platformView)
#endif
        {
            StatusBar.SetColor(StatusBarColor);
            StatusBar.SetStyle(StatusBarStyle);
        }


        /// <inheritdoc /> 
        protected override void OnPropertyChanged([CallerMemberName] string? propertyName = null)
        {
            if (string.IsNullOrWhiteSpace(propertyName))
            {
                return;
            }

            base.OnPropertyChanged(propertyName);

            if (IsOneOf(propertyName, StatusBarColorProperty, Page.WidthProperty, Page.HeightProperty))
            {
                StatusBar.SetColor(StatusBarColor);
            }
            else if (propertyName == StatusBarStyleProperty.PropertyName)
            {
                StatusBar.SetStyle(StatusBarStyle);
            }
        }
#endif

        public bool IsOneOf(string propertyName, BindableProperty p0, BindableProperty p1, BindableProperty p2)
        {
            return propertyName == p0.PropertyName ||
                propertyName == p1.PropertyName ||
                propertyName == p2.PropertyName;
        }

    }
}
```

天不從人願，還是得到一樣的錯誤訊息，現在只好到 [PrismLibrary / Prism.Maui](https://github.com/PrismLibrary/Prism.Maui) 發出一個 [Issue](https://github.com/PrismLibrary/Prism.Maui/issues/108)，也許是我有贊助這個開源專案，所以很快地得到作者的回應，不過，看樣子不是那麼容易的來解決這個問題。

經過幾天的陸續測試與思考，我想到一個做法，我想要在 MainPage 的覆寫方法 OnAppearing 內，直接呼叫這個 CommunityToolkit 所提供的 StatusBar.SetColor 這個方法，不知道是否可以解決這個問題呢？

因此，我將原來的 MainPage.xaml 修改如下

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:toolkit="http://schemas.microsoft.com/dotnet/2022/maui/toolkit"
             xmlns:localBehaviors="clr-namespace:mauiStatusBar.Behaviors"
             Title="{Binding Title}"
             x:Class="mauiStatusBar.Views.MainPage">

    <!--<ContentPage.Behaviors>
        <toolkit:StatusBarBehavior StatusBarColor="Fuchsia" StatusBarStyle="LightContent" />
    </ContentPage.Behaviors>-->

    <ScrollView>
    <VerticalStackLayout
            Spacing="25"
            Padding="30,0"
            VerticalOptions="Center">

      <Image Source="prism.png"
             SemanticProperties.Description="Cute dot net bot waving hi to you!"
             HeightRequest="150"
             HorizontalOptions="Center" />

      <Label Text="Hello, World!"
             SemanticProperties.HeadingLevel="Level1"
             FontSize="32"
             HorizontalOptions="Center" />

      <Label Text="Welcome to Prism for .NET MAUI"
             SemanticProperties.HeadingLevel="Level2"
             SemanticProperties.Description="Welcome to Prism for dot net Multi platform App U I"
             FontSize="18"
             HorizontalOptions="Center" />

      <Button Text="{Binding Text}"
              SemanticProperties.Hint="Counts the number of times you click"
              Command="{Binding CountCommand}"
              HorizontalOptions="Center" />

    </VerticalStackLayout>
  </ScrollView>

</ContentPage>
```

我不再使用 Behavior 來變更狀態列的顏色，接著，在這個頁面的 Code Behind 內，改成底下的程式碼

```csharp
using CommunityToolkit.Maui.Core.Platform;

namespace mauiStatusBar.Views;

public partial class MainPage : ContentPage
{
    public MainPage()
    {
        InitializeComponent();
    }

    protected override void OnAppearing()
    {
        var foo = App.Current.MainPage;
        StatusBar.SetColor(Colors.Red);
    }
}
```

