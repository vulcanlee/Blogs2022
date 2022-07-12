# Task.Run 要在哪裡傳入與檢查 CancellationToken

這篇文章將會說明，對於一開始學習 Task.Run 方法與想要透過 CancellationToken 取消權杖來取消工作的開發者，將會有個迷失點，這個問題在於，當查看 [Task.Run 方法](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.tasks.task.run?WT.mc_id=DT-MVP-5002220) 文件的時候，將會看到 `Run(Action, CancellationToken)` 這個函式簽章，而這個方法將透過執行緒集區佇列內進行排隊，取得一個可用執行緒，並且將 `Action` 這個委派方法交由這個執行緒來執行，並傳回代表該工作的 Task 物件；這個方法的第二個參數，則是一個 [CancellationToken] 型別的參數，代表一個取消權杖可用來在工作尚未開始之前取消工作。

若沒有看到上面最後一段說明，絕大部分的開發者都會以為，只要在呼叫 Task.Run 方法的時候，將取消權杖傳入進去，在任何時候就可以透過產生該取消權證的 CancellationTokenSource 這個型別的物件，發出呼叫 [CancellationTokenSource.Cancel 方法](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.cancellationtokensource.cancel?WT.mc_id=DT-MVP-5002220) ， 送出與傳遞取消要求，此時，這個 Task.Run 這個非同步工作，就會取消執行了。

從上面的說明可以看出不是這樣的運作的，當呼叫 Task.Run 方法所傳入的取消權杖，僅會在要呼叫 Task.Run 方法前，這個取消權杖就已將發出取消請求了，所以，一旦呼叫了 Task.Run 的方法，就不會去取得一個非同步的執行緒，並且開始執行所指定的委派方法，如此，可以節省系統資源的使用。

現在來測試這樣的說明，首先，建立一個 主控台應用程式，其程式碼如下：

```csharp
static void Main(string[] args)
{
    CancellationTokenSource cts = new CancellationTokenSource();
    CancellationToken token = cts.Token;
    cts.Cancel();
    Console.WriteLine($"1 建立非同步工作");
    var task = Task.Run(() =>
    {
        Console.WriteLine($"  2 非同步工作已經開始執行");
        Thread.Sleep(3000);
        Console.WriteLine($"  3 非同步工作已經結束執行");
    }, token);
    Console.WriteLine($"4 主執行緒休息 2 秒");
    Thread.Sleep(2000);
    Console.WriteLine($"5 主執行緒即將結束執行");
    Console.WriteLine("Press any key for continuing...");
    Console.ReadKey();
}
```

這是一個非常經典的取消權杖的程式設計準則，在程式要開始執行之前，首先建立一個 CancellationTokenSource 物件，這個物件可用於取得取消權杖與可以發出取消請求的方法，接著，將取消權杖透過 CancellationToken token = cts.Token 敘述來取得，有了取消權杖，之後面可以將取消權杖傳遞到其他的非同步方法或者非同工作內。

在這個時候，將會呼叫 cts.Cancel() 方法，對於該取消權杖發出取消工作的請求，此時，該取消權杖處於已經取消的狀態下，接著，透過 Task.Run 方法建立一個非同步工作，這個非同步工作所指定的委派方法將會休息三秒鐘後，就會結束執行。

同時，在主執行緒端，一旦非同步工作建立與開始執行後，將會休息 2 秒鐘

現在，執行這個專案，將會得到底下的執行結果

```
1 建立非同步工作
4 主執行緒休息 2 秒
5 主執行緒即將結束執行
Press any key for continuing...
```

從執行結果的輸出內容可以看出，這個非同步工作物件所指定的委派方法，是沒有被執行的，也就是說，雖然呼叫了 Task.Run 方法，系統並沒有從執行緒集區內取得一個執行緒來執行指定的委派方法程式碼。

現在將測試程式碼修改成為如下：

```csharp
static void Main(string[] args)
{
    CancellationTokenSource cts = new CancellationTokenSource();
    CancellationToken token = cts.Token;
    Console.WriteLine($"1 建立非同步工作");
    var task = Task.Run(() =>
    {
        Console.WriteLine($"  2 非同步工作已經開始執行");
        Thread.Sleep(3000);
        Console.WriteLine($"  3 非同步工作已經結束執行");
    }, token);
    Console.WriteLine($"4 主執行緒休息 1 秒");
    cts.CancelAfter(2000);
    Thread.Sleep(1000);
    Console.WriteLine($"5 主執行緒即將結束執行");
    Console.WriteLine("Press any key for continuing...");
    Console.ReadKey();
}
```

前半段幾乎沒有變動，除了將一開始就發出取消權杖的呼叫，將其程式碼移除了。

在非同步工作建立之後，使用了 `cts.CancelAfter(2000);` 敘述，將會於 2 秒鐘之後，對取消權杖發出取消的請求通知，這個敘述執行完後，並不會暫停2秒鐘，而會繼續往下執行，現在，在主執行緒內，將會休息一秒鐘後，準備要結束執行。

底下是執行這個專案的螢幕輸出結果

```
1 建立非同步工作
4 主執行緒休息 1 秒
  2 非同步工作已經開始執行
5 主執行緒即將結束執行
Press any key for continuing...
  3 非同步工作已經結束執行
```

從執行結果可以看出，雖然在呼叫 Task.Run 方法的時候，有傳入取消權杖，不過，請求取消的動作，是在呼叫 Task.Run 之後，因此，在 Task.Run 所傳入的取消權杖是沒有效用的，結論就是，想要建立一個非同步工作，讓這個非同步工作具有取消權杖的效果，建議如下：

* 在 Task.Run 方法內，傳入取消權杖
* 在 Task.Run 的委派方法內，也要輪詢檢查取消權杖是否已經發出取消請求

接著，將測試程式碼修改如下：

```csharp
static void Main(string[] args)
{
    CancellationTokenSource cts = new CancellationTokenSource();
    CancellationToken token = cts.Token;
    Console.WriteLine($"1 建立非同步工作");
    var task = Task.Run(() =>
    {
        Console.WriteLine($"  2 非同步工作已經開始執行");
        token.ThrowIfCancellationRequested();
        Thread.Sleep(3000);
        token.ThrowIfCancellationRequested();
        Console.WriteLine($"  3 非同步工作已經結束執行");
    }, token);
    Console.WriteLine($"4 主執行緒休息 1 秒");
    cts.CancelAfter(2000);
    Thread.Sleep(1000);
    Console.WriteLine($"5 主執行緒即將結束執行");
    Console.WriteLine("Press any key for continuing...");
    Console.ReadKey();
}
```

在這裡將會在非同步工作內的委派方法中，使用輪詢 Polling 方式，檢查取消權杖的狀態，這裡使用了 token.ThrowIfCancellationRequested(); 敘述，一旦有取消請求發出，則將會拋出例外異常，結束這個非同步工作的執行。

底下是執行結果

```
1 建立非同步工作
4 主執行緒休息 1 秒
  2 非同步工作已經開始執行
5 主執行緒即將結束執行
Press any key for continuing...
```

從執行結果可以看出，當發出取消請求之後，這個取消權杖被設定為取消狀態，而這個非同的工作也就拋出例外異常，而中止了。

最後，再來將程式碼改成底下內容：

```csharp
static async Task Main(string[] args)
{
    CancellationTokenSource cts = new CancellationTokenSource();
    CancellationToken token = cts.Token;
    Console.WriteLine($"1 建立非同步工作 {DateTime.Now.TimeOfDay}");
    var task = Task.Run(async () =>
    {
        Console.WriteLine($"  2 非同步工作已經開始執行 {DateTime.Now.TimeOfDay}");
        token.ThrowIfCancellationRequested();
        await Task.Delay(3000, token);
        token.ThrowIfCancellationRequested();
        Console.WriteLine($"  3 非同步工作已經結束執行 {DateTime.Now.TimeOfDay}");
    }, token);
    Console.WriteLine($"4 主執行緒休息 1 秒 {DateTime.Now.TimeOfDay}");
    cts.CancelAfter(2000);
    try
    {
        await task;
    }
    catch (Exception ex)
    {
        Console.WriteLine($"5 非同步工作取消了 {ex.GetType().ToString()} : {DateTime.Now.TimeOfDay}");
    }
    Console.WriteLine($"6 主執行緒即將結束執行 {DateTime.Now.TimeOfDay}");
    Console.WriteLine("Press any key for continuing...");
    Console.ReadKey();
}
```

在上面的程式碼，對 Console.WriteLine 方法內，將會加入當時呼叫的時間點內容，這裡是要來判斷在非同步工作內委派方法，在等待 3 秒鐘的時候，是否不會在 3 秒內的任何時間點，若接收到取消通知，都會立即中止等候，直接結束這個非同步工作。

對於在非同步工作內的委派方法，原先使用 Thread.Sleep(3000) 的封鎖等待的呼叫，將會改成 await Task.Delay(3000, token); 非封鎖式的呼叫，並且在 Delay 方法的後面，傳入取消權杖，要 Task.Delay 這個方法要能夠關注取消權杖是否發出請求。

在主執行緒端，也將休息2秒的敘述，改成 await task; 這個敘述，並且將其包裹在 try catch 敘述內，因為，一旦這個非同步工作有取消動作產生，將會拋出例外異常，所以，將會在此捕捉這個例外異常，查看非同步工作內的委派方法，何時觸發了取消動作。

現在，可以來執行這個專案，得到底下的執行結果

```
1 建立非同步工作 16:22:37.3960409
4 主執行緒休息 1 秒 16:22:37.4027808
  2 非同步工作已經開始執行 16:22:37.4033487
5 非同步工作取消了 System.Threading.Tasks.TaskCanceledException : 16:22:39.4463615
6 主執行緒即將結束執行 16:22:39.4464650
Press any key for continuing...
```

從執行結果可以看出，非同步工作內的委派方法於 37 秒的時候開始執行，接著要休息 3 秒鐘，因此，這個非同步工作應該會於 40 秒的時候結束執行。

不過，在主執行緒端，將會於 2 秒鐘之後發出取消請求，因此，這個非同步工作將會於 39 秒的時候拋出例外異常，這裡看到的例外異常是 [TaskCanceledException](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.tasks.taskcanceledexception?WT.mc_id=DT-MVP-5002220) ，這代表表示用來傳達工作取消的例外狀況；從捕捉到例外異常的時間點，也正好是 39 秒，符合這個程式的設計。
