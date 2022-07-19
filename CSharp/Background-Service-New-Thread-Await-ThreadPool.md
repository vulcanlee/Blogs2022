# 當要進行背景服務程式碼設計的時候，是否要使用 await 運算子呢？

這篇文章是記錄我之前在進行 [ASP.NET Core 中使用託管服務的背景工作](https://docs.microsoft.com/zh-tw/aspnet/core/fundamentals/host/hosted-services?WT.mc_id=DT-MVP-5002220) 程式碼設計的時候，想要設計一個背景服務，這個程式將會每隔一段時間來檢查是否有使用者的帳號已經密碼過期，或者需要強制登出的請求出現，此時，這個背景服務程式將會設定這個帳號需要強制登出；若是帳號密碼過期的問題，則該帳號再重新登入的時候，將會要求重新變更帳號密碼，否則無法進行登入。

對於這樣的需求，一開始的想法是，我需要自己 new [Thread](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.thread?WT.mc_id=DT-MVP-5002220) 這個執行個體出來，產生出一個前景執行緒，因為，在這裡需要進行長時間的處理工作。另外，不去使用執行緒集區來取得一個被景執行緒的原因，那就是這個執行緒會長時間運行，若該執行緒來自於執行緒集區，則該集區內的執行緒便會少了一個可用執行緒資源。

然而，這樣的設計理念雖然很好，很快地就遇到問題，那就是在這個前景執行緒中，因為某些需要，便要呼叫一些非同步的 API ，而這些 API 是可以適用於 [async](https://docs.microsoft.com/zh-tw/dotnet/csharp/language-reference/keywords/async?WT.mc_id=DT-MVP-5002220) / await 的方式，理所當然的就直接使用這樣的作法把他呼叫下去。

當專案設計完成與正式執行之後，也沒有發現到甚麼異常現象產生，畢竟，在這個前景執行緒的程式碼內，比較沒有存取共用資源的競賽問題，也就是沒有執行緒安全的問題產生。可是，總是感覺怪怪的，畢竟，大家都知道，當程式碼執行到 [await](https://docs.microsoft.com/zh-tw/dotnet/csharp/language-reference/operators/await?WT.mc_id=DT-MVP-5002220) 運算子這個關鍵字後，將會做三件事情 

1. 記住離開的執行程式碼位置 
2. 捕捉當前執行內容 
3. 立即 return 返回。

當 await 所等待的這個非同步工作或者方法完成之後，將會做三件事情 

1. 從工作排程器取得一個可以繼續執行緒 
2. 還原之前執行內容 
3. 從剛剛中斷的地方繼續往下執行。

一旦看完剛剛所提到的事情之後，那麼一開始建立一個前景執行緒的動作不就白做了嗎？

因為只要在這個前景執行緒所指定的 Unit Of Work 工作單元 / 委派方法內，使用 await 關鍵字，並且在這個委派方法外加入 async 關鍵字，那麼，執行完成 await 關鍵字之後，前景執行緒也就不再存在了，之後的程式碼都會來自於 [工作排成器 Task Scheduler](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.tasks.taskscheduler?WT.mc_id=DT-MVP-5002220) 預設將會從 執行緒集區 取得一個可用的執行緒來繼續執行下去。

為了要驗證這樣的情況，特別設計一個主控台應用程式的專案，其程式碼如下

```csharp
namespace Background_Service_New_Thread_Await
{
    internal class Program
    {
        static void Main(string[] args)
        {
            Thread thread = new Thread(async () =>
            {
                Thread.CurrentThread.Name = "新 new Thread 的處理服務用執行緒(非來自執行緒集區)";
                Console.WriteLine($"開始進行背景服務程式執行");
                // 模擬這個背景服務要處理的同步程式碼執行動作
                Thread.Sleep(2000);
                Console.WriteLine($"準備進行非同步 await 呼叫");
                Console.WriteLine();

                ShowThreadInformation("呼叫 await 前，new Thread 的相關資訊");

                // 模擬要使用 await 來進行非同步呼叫
                await Task.Delay(3000);

                ShowThreadInformation("呼叫 await 後，new Thread 的相關資訊");
            });
            thread.Start();

            Console.WriteLine("Press any key for continuing...");
            Console.ReadKey();
        }

        // 將當前的執行緒資訊顯示出來
        static void ShowThreadInformation(string message)
        {
            Console.WriteLine(message);
            Console.WriteLine($"執行緒 Id : {Thread.CurrentThread.ManagedThreadId}");
            Console.WriteLine($"執行緒名稱 : {Thread.CurrentThread.Name}");
            Console.WriteLine($"來自集區 : {Thread.CurrentThread.IsThreadPoolThread}");
            Console.WriteLine($"為背景執行緒 : {Thread.CurrentThread.IsBackground}");
            Console.WriteLine();
        }
    }
}
```

在上面的程式碼，設計的一個 [ShowThreadInformation] 方法，這個方法會將當前執行緒的 Id ， 名稱 Name ， 是否該執行緒來自於執行緒集區 、 該執行緒是否為背景執行緒 資訊顯示在螢幕上。

在主執行緒內，首先使用 `new Thread` 運算式來建立一個執行緒物件，在該執行緒所綁定的委派方法內，將會使用 `await Task.Delay(3000);` 敘述來執行 await 的工作，並且在這個 Lambda 委派方法上，使用了 async 修飾詞 `async ()=> {...}` 標明這個委派方法，是一個非同步方法。

而在執行 await 運算式的前後，將會呼叫 [ShowThreadInformation] 方法，顯示當前執行緒的資訊。

現在可以執行這個專案，將會看到底下的執行內容

```
Press any key for continuing...
開始進行背景服務程式執行
準備進行非同步 await 呼叫

呼叫 await 前，new Thread 的相關資訊
執行緒 Id : 10
執行緒名稱 : 新 new Thread 的處理服務用執行緒(非來自執行緒集區)
來自集區 : False
為背景執行緒 : False

呼叫 await 後，new Thread 的相關資訊
執行緒 Id : 6
執行緒名稱 : .NET ThreadPool Worker
來自集區 : True
為背景執行緒 : True
```

從執行結果可以看的出來，在呼叫 await 之前，當前的執行緒卻是為前景執行緒，並且不是來自於執行緒集區內，另外，在該執行緒執行後，就會執行 `Thread.CurrentThread.Name = "新 new Thread 的處理服務用執行緒(非來自執行緒集區)";` 敘述，設定這個執行緒 Name 的屬性值，這樣的設定結果，可以從執行結果看的出來。

當 await 執行完成後，再度顯示當前執行緒的狀態，發現到現在的執行緒 ID 已經改變了，名稱也不對了，最重要的是這個執行緒來自於執行緒集區，而且是個背景執行緒。

