# 如何解決 Android 模擬器無法透過 Internet 打開任何網頁與看到網站內容

![](../Images/0MAUI/Maui9962.png)

也許之前在使用 .NET MAUI 開發的時候，大多是用於進行各種教學課程的練習專案設計，還沒有進入到需要連線到網路上的應用範例；然而，上週需要進行使用 .NET MAUI 開發的專案，進行 OAuth2 的身分驗證開發需求，這裡首先需要用到的是要與 Azure AD 來進行身分驗證。

經過一番奮戰，首先決定先採用 [Web authenticator](https://learn.microsoft.com/en-us/dotnet/maui/platform-integration/communication/authentication?view=net-maui-7.0&tabs=windows&WT.mc_id=DT-MVP-5002220) 這個由 .NET MAUI 平台在 [平臺整合](https://learn.microsoft.com/zh-tw/dotnet/maui/platform-integration/?view=net-maui-7.0&WT.mc_id=DT-MVP-5002220) 內所提供的 API 來嘗試能夠完成 OAuth2 與 Azure AD 的身分驗證。

在解決了大部分的問題之後，突然發現到，我經常使用的 Android Pixel 5 - API 33 模擬器，要使用裡面內建的 Chrome 瀏覽器來開啟自己設計的 Web API 服務的時候，卻無法開啟這個服務端點。當遇到這個問題，當然先去網路上來搜尋看看有沒有類似的問題發生，首先看到的就是，很多人都指向這樣的問題是與 DNS 設定有關，因此，根據網路上查詢到的資料，進行本機或者模擬器端的 DNS 修正，結果是沒有任何效果，之後回想，當第一次在模擬器開啟 Chrome 瀏覽器的時候，將會看到如下圖的畫面，在下方的 [Discovery] 的內容，可以看到這個瀏覽器似乎有正常運作，因為可以抓取到最新的網頁內容；另外，我也嘗試打開 [Youtube] App，發現到這個 App 可以正常運作，這表示了該模擬器的網路與 DNS 運作是沒有問題的。

![](../Images/0MAUI/Maui9963.png)

其實，我是可以忽略掉這個問題，因為，在我的桌機上面，有個之前很早之前就安裝的模擬器，我用他來開啟任何網頁都可以正常運作，不過，當我沒有使用家裡的桌機時候，就必須面對到上面所提到的問題，因此，還是需要燃燒自己的生命，再次來進行探索與嘗試解決這個問題。

再次透過網路來搜尋，無意見看到一個這段文字

```
It's caused by vulkan. To fix it, you must turn vulkan off on emulator or chrome.
```

其中 [Vulkan](https://zh.wikipedia.org/zh-tw/Vulkan) 代表甚麼與為什麼會造成這樣的問題，其實我並不關心，所以，我根據相關網頁提到的線索，進行底下的操作

* 找到這個目錄 [C:\Users\%USER%\.android] 下的 [advancedFeatures.ini]
* 使用任何文字編輯器工具來開啟這個檔案
* 底下將會是我這台電腦上所看到的預設內容

```
WindowsHypervisorPlatform=on
```

* 在這個檔案內，加入底下兩行敘述

```
WindowsHypervisorPlatform=on
Vulkan = off
GLDirectMem = on
```

* 儲存並且關閉這個檔案
* 重新開啟 Android 模擬器
* 打開模擬器上的 Chrome 瀏覽器
* 打開任何網頁 ，這裡打開聯合報的網站
* 此時便可以正常運作了

![](../Images/0MAUI/Maui9961.png)


