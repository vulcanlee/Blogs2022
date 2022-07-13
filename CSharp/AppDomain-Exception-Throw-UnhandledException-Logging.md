# C# 非同步 : 在 .NET 程式中，如何讀取到未攔截到例外異常 Exception 發生的時候，當時的詳細狀況

當 .NET 處理程序在執行過程中，產生無法捕捉到的例外異常情況，將會造成該應用程式崩潰，此時，透過 AppDomain 物件內所提供的 UnhandledException 事件 ([AppDomain.UnhandledException 事件](https://docs.microsoft.com/zh-tw/dotnet/api/system.appdomain.unhandledexception?WT.mc_id=DT-MVP-5002220))，將會於該處理程序要結束執行前幾秒，觸發與執行這裡所綁定的委派事件方法，因此，程式設計師可以在這裡進行任何的紀錄或者補救措施，不過，很重要的是，只有短短的幾秒鐘的時間可以來執行。

首先，先來設計一個非同步的多執行緒應用程式，其程式碼如下

```csharp
Console.WriteLine($"啟動一個非同步執行緒");
ThreadPool.QueueUserWorkItem(_ =>
{
    Console.WriteLine($"   非同步委派方法開始執行");
    Task.Delay(3000).Wait();
    Console.WriteLine($"   模擬呼叫的 API 拋出例外異常");
    throw new Exception("喔喔，發生不明的錯誤");
    Console.WriteLine($"   非同步委派方法結束執行");
});
Console.WriteLine($"等候5秒鐘");
Thread.Sleep(5000);
Console.WriteLine("Press any key for continuing...");
Console.ReadKey();
```

底下將會是這個程式碼的執行螢幕輸出結果

```
啟動一個非同步執行緒
等候5秒鐘
   非同步委派方法開始執行
   模擬呼叫的 API 拋出例外異常
Unhandled exception. System.Exception: 喔喔，發生不明的錯誤
   at TD017__未攔截到例外狀況.Program.<>c.<Main>b__0_0(Object _) in C:\Vulcan\Github\CSharp-Thread-Quick-Launch\CSharpThread\TD017  未攔截到例外狀況\Program.cs:line 24
   at System.Threading.QueueUserWorkItemCallbackDefaultContext.Execute()
   at System.Threading.ThreadPoolWorkQueue.Dispatch()
   at System.Threading.PortableThreadPool.WorkerThread.WorkerThreadStart()
   at System.Threading.Thread.StartCallback()
```

在這個範例程式碼中，首先將會透過執行緒集區 ThreadPool 來取得一個執行緒，執行這裡所傳入的 Lambda 委派方法

這裡非同步要執行的委派方法將會模擬要處理大量的工作，因此，使用 `Task.Delay(3000).Wait()` 方法來進行封鎖等待 3 秒鐘，接著，將會使用 `throw new Exception("喔喔，發生不明的錯誤");` 敘述來拋出一個例外異常，這裡將會是模擬這個應用程式，在一個非同步的執行緒內，產生一個未知的錯誤，而當時的委派方法內，卻有沒有相對應的 `try{...} catch {...}` 敘述存在，導致當時無法補中到這樣的例外異常。

因為處理程序中發現到未捕捉到的例外異常，此時，處理程序將會開始進行強制結束執行，結果就是該應用程式突然的結束且不見追跡，並且沒有留下任何的線索，可以得知為什麼這個應用程式突然的掛掉了。

為了要解決這樣的問題，將會把底下的程式碼放到這個應用程式的最前面，使其優先來執行

```csharp
 AppDomain appDomain = AppDomain.CurrentDomain;
 appDomain.UnhandledException += (s, e) =>
 {
     Console.WriteLine($"接收到 未攔截例外狀況的通知");
     Exception exception = e.ExceptionObject as Exception;
     Console.WriteLine($" > 例外異常訊息 : {exception.Message}");
     Console.WriteLine($" > 當時呼叫堆疊 : {exception.StackTrace}");
 };
```

在此，先會透過 `AppDomain.CurrentDomain` 取得當前的 AppDomain 這個類別的執行個體，而 [AppDomain](https://docs.microsoft.com/zh-tw/dotnet/api/system.appdomain?WT.mc_id=DT-MVP-5002220) 表示應用程式定義域，也就是應用程式執行的獨立環境。

接著透過取得的物件，訂閱了 `appDomain.UnhandledException` 這個事件，每當預設應用程式域中擲回未處理的例外狀況時，就會叫用這個事件處理常式。

在該事件的委派方法或者處理常式內，可以將當時沒有捕捉到的例外異常訊息記錄到自己常用的 Logger 日誌系統內，或者也可以寫入到本機的檔案內，如此，日後便可以依據這裡的訊息，追查到當時究竟發生了甚麼問題。

在這個例子中，將會把錯誤訊息顯示在螢幕上，底下是執行的螢幕輸出結果

```
啟動一個非同步執行緒
等候5秒鐘
   非同步委派方法開始執行
   模擬呼叫的 API 拋出例外異常
接收到 未攔截例外狀況的通知
 > 例外異常訊息 : 喔喔，發生不明的錯誤
 > 當時呼叫堆疊 :    at TD017__未攔截到例外狀況.Program.<>c.<Main>b__0_1(Object _) in C:\Vulcan\Github\CSharp-Thread-Quick-Launch\CSharpThread\TD017  未攔截到例外狀況\Program.cs:line 24
   at System.Threading.QueueUserWorkItemCallbackDefaultContext.Execute()
   at System.Threading.ThreadPoolWorkQueue.Dispatch()
   at System.Threading.PortableThreadPool.WorkerThread.WorkerThreadStart()
   at System.Threading.Thread.StartCallback()
Unhandled exception. System.Exception: 喔喔，發生不明的錯誤
   at TD017__未攔截到例外狀況.Program.<>c.<Main>b__0_1(Object _) in C:\Vulcan\Github\CSharp-Thread-Quick-Launch\CSharpThread\TD017  未攔截到例外狀況\Program.cs:line 24
   at System.Threading.QueueUserWorkItemCallbackDefaultContext.Execute()
   at System.Threading.ThreadPoolWorkQueue.Dispatch()
   at System.Threading.PortableThreadPool.WorkerThread.WorkerThreadStart()
   at System.Threading.Thread.StartCallback()
```

