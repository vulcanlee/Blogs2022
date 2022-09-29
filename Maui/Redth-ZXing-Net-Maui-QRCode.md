# 在 .NET MAUI 專案內進行讀取 QR Code 二維條碼內容

對於如何透過手機 App 來讀取 QRCode 二維條碼這樣的需求，向來是要學習手機 App 開發學員的前三名最想要知道的技術，在這篇文章中，將會來說明如何透過手機鏡頭，讀取 QR Code 二維條碼上的內容是甚麼？

## 建立 .NET MAUI 應用程式 專案

* 開啟 Visual Studio 2022 版本
* 點選螢幕右下角的 [建立新的專案] 按鈕
* 切換右上角的 [所有專案類型] 下拉選單控制項
* 找到並且點選 [MAUI] 這個選項
* 從清單中找到並選擇 [.NET MAUI 應用程式] 這個專案範本

  > 此專案可用於建立適用於 iOS、Android、Mac Catalyst、Tizen和WinUI 的 .NET MAUI 應用程式
* 點選右下角的 [下一步] 按鈕
* 當出現了 [設定新的專案] 對話窗
* 在 [專案名稱] 欄位內，輸入 `mauiScanQRCode`
* 點選右下角的 [下一步] 按鈕
* 當出現了 [其他資訊] 對話窗
* 對於 [架構] 的下拉選單控制項，使用預設值
* 點選右下角的 [建立] 按鈕

## 加入 PropertyChanged.Fody 的 NuGet 套件

* 滑鼠右擊該專案的 [相依性] 節點
* 從彈出功能表中選擇 [管理 NuGet 套件] 功能選項
* 此時，[NuGet: mauiScanQRCode] 視窗將會出現
* 點選 [瀏覽] 標籤頁次
* 在左上方的搜尋文字輸入盒內輸入 `ZXing.Net.Maui` 關鍵字
* 現在，將會看到 ZXing.Net.Maui 套件出現在清單內
* 點選這個 ZXing.Net.Maui 套件，並且點選右上方的 [安裝] 按鈕，安裝這個套件到這個專案內。

## 進行 ZXing.NET.Maui 套件的服務註冊

* 在專案根目錄下
* 找到並且打開 MauiProgram.cs 檔案
* 找到 `.UseMauiApp<App>()` 程式碼
* 在其下方加入 `.UseBarcodeReader()` Fluent 呼叫程式碼

## 對 Android 專案進行需要用到權限宣告

* 在專案內，找到  Platforms\Android 目錄
* 在其目錄內找到並且打開 MainApplication.cs 檔案
* 找到 namespace 關鍵字
* 在其上方加入 `[assembly: UsesPermission(Android.Manifest.Permission.Camera)]` 敘述，說明這個 Android App 需要使用到 Camera 這個權限，如此，才可以讓這個 Zxing 軟體透過鏡頭來讀取到條碼的圖片

## 對 Android 專案進行需要用到權限宣告

* 在專案內，找到  Platforms\iOS 目錄
* 找到 Info.plist 檔案，並且選擇使用 XML (文字) 編輯器工具，打開這個檔案
* 找到 `</dict>` 這個關鍵字
* 在其上方，加入底下的 XML 宣告

```XML
<key>NSCameraUsageDescription</key>
<string>This app uses barcode scanning to...</string>
```

## 將 QR Code 讀取檢視控制項放到 XAML 頁面內

* 在專案根目錄下
* 找到並且打開 MainPage.xaml 這個檔案
* 使用底下 XAML 內容將其替換掉

```xml
<?xml version="1.0" encoding="utf-8" ?>
<ContentPage xmlns="http://schemas.microsoft.com/dotnet/2021/maui"
             xmlns:x="http://schemas.microsoft.com/winfx/2009/xaml"
             xmlns:zxing="clr-namespace:ZXing.Net.Maui.Controls;assembly=ZXing.Net.MAUI"
             x:Class="mauiScanQRCode.MainPage">

    <ScrollView>
        <VerticalStackLayout
            Spacing="25"
            Padding="30,0"
            VerticalOptions="Center">

            <Label x:Name="labelResult"
                   FontSize="24" TextColor="Red"/>
            
            <zxing:CameraBarcodeReaderView
                x:Name="cameraBarcodeReaderView"
                BarcodesDetected="BarcodesDetected" />
            
            <Button
                x:Name="CounterBtn"
                Text="Click me"
                SemanticProperties.Hint="Counts the number of times you click"
                Clicked="OnCounterClicked"
                HorizontalOptions="Center" />

        </VerticalStackLayout>
    </ScrollView>

</ContentPage>
```

在上面的 XAML 頁面宣告中，首先加入了一個新的 zxing 命名空間宣告 : `xmlns:zxing="clr-namespace:ZXing.Net.Maui.Controls;assembly=ZXing.Net.MAUI"` ，如此，在這個頁面中，便可以透過這個 zxing 這個前置詞，存取到 ZXing.Net.MAUI 這個套件所提供的功能與檢視了。

在 Button 按鈕前，加入了 `<zxing:CameraBarcodeReaderView x:Name="cameraBarcodeReaderView" BarcodesDetected="BarcodesDetected" />` 這個 CameraBarcodeReaderView 檢視控制項，這個控制項將會把鏡頭看到的影像，顯示到螢幕上，並且會針對每個讀取進來的影像，進行條碼分析，一旦發現到有條碼圖片存在，並且該條碼也被解碼了，將會呼叫觸發這個 BarcodesDetected 事件，並且可以透該事件內提供的事件參數，取得所掃描到的條碼內容。

由於是採用事件觸發方式來設計，所以，需要透過 Code Behind 的方式來設計相關商業邏輯

* 找到並且打開 MainPage.xaml.cs 檔案
* 使用底下 C# 程式碼來替換掉這個檔案內容

```csharp
using ZXing.Net.Maui;

namespace mauiScanQRCode;

public partial class MainPage : ContentPage
{
	int count = 0;

	public MainPage()
	{
		InitializeComponent();

        cameraBarcodeReaderView.Options = new BarcodeReaderOptions
        {
            Formats = BarcodeFormats.All,
            AutoRotate = true,
            Multiple = true
        };
    }

	private void OnCounterClicked(object sender, EventArgs e)
	{
		count++;

		if (count == 1)
			CounterBtn.Text = $"Clicked {count} time";
		else
			CounterBtn.Text = $"Clicked {count} times";

		SemanticScreenReader.Announce(CounterBtn.Text);
	}

	private void BarcodesDetected(object sender, ZXing.Net.Maui.BarcodeDetectionEventArgs e)
	{
		string Result = "";
        foreach (var barcode in e.Results)
            Result += $"Barcodes: {barcode.Format} -> {barcode.Value}  ";

		MainThread.BeginInvokeOnMainThread(() =>
		{
			labelResult.Text = Result;
		});
    }
}
```

在這個頁面建構式內，首先建立一個 BarcodeReaderOptions 物件，用來宣告這個 QRCode 的掃描行為，這裡使用了 Formats 屬性，宣告要掃描所有類型的一維或者二維條碼圖片，使用 AutoRotate 將會允許旋轉，使用 Multiple 屬性，宣告可以讀取多個條碼內容。完成之後，設定到這個 QR Code 讀取元件內的 cameraBarcodeReaderView.Options 屬性內。

對於一旦條碼讀取到之後，將會觸發這裡的事件委派方法，其中，該事件傳遞進來的參數將會是 ZXing.Net.Maui.BarcodeDetectionEventArgs 型別，由於上面宣告了可以一次讀取一個以上的條碼圖片，因此，在 e.Results 將會有可能擁有多筆條碼內容，所以，在此使用迴圈，將所有讀到的條碼內容，都放到 Result 這個變數內。

當蒐集到所有的條碼內容之後，便可以將這些文字內容，顯示到螢幕上，這裡使用了 `labelResult.Text = Result;` 這樣的敘述，不過，因為這個觸發所執行的事件方法，不會在 UI (或 主) 執行緒上運行，為了要能夠更新 labelResult 這個 UI 元件的 Text 屬性值，所以，需要透過 `MainThread.BeginInvokeOnMainThread` 敘述，讓這些程式碼能夠在主執行緒上來運行。

## 執行與測試

底下影片將會是在實體手機上的執行結果

![](../Images/net899.gif)
