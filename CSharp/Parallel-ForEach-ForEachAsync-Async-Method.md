# 如何設計有效的 Parallel.ForEachAsync 迴圈 - 需要使用平行計算方式來執行 async Method 非同步方法

對於要使用 [Parallel.ForEach] 來設計出一個具有平行作業的應用程式，相信對於許多 .NET C# 開發者而言，應該不是個很大的問題，此時，可以參考 [如何：撰寫簡單的 Parallel.ForEach 迴圈](https://docs.microsoft.com/zh-tw/dotnet/standard/parallel-programming/how-to-write-a-simple-parallel-foreach-loop?WT.mc_id=DT-MVP-5002220) 這篇文章中的說明與範例程式碼，相信可以很容易得輕鬆上手。

可是對於 [Parallel.ForEach 方法](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.tasks.parallel.foreach?WT.mc_id=DT-MVP-5002220) 的使用，需要傳入至少一個列舉 IEnumerable 與一個委派方法，不過，在這裡若是設計使用同步方法來進行設計程式碼，相信執行上一切都沒有問題， Parallel.ForEach 方法將會使用平行計算的方式，使用列舉物件內的值，平行執行與傳入到這個委派方法，但是，當這個委派方法已經改成一個非同步方法，也就是在這個委派方法有加入了 async 這個修飾詞，那麼將會發現到，一旦執行到 Parallel.ForEach 之後，將會馬上就執行到下一行敘述，不會等到所有的列舉物件都執行完成後，才會繼續往下執行

對於使用同步委派方法來設計 Parallel.ForEach 的應用，可以參考底下的範例程式碼

```csharp
#region 若在 Parallel.Foreach 內使用同步方式進行委派方法的設計，會等到所有委派方法都執行完畢後，才會繼續往下執行
Console.WriteLine($"使用 Parallel.Foreach 與同步委派方法 開始 {DateTime.Now}");
Parallel.ForEach(Enumerable.Range(0, 3), async (x, t) =>
{
    Console.WriteLine("  Bpfsync");
    Thread.Sleep(3000);
    Console.WriteLine("  Cpfsync");
});
// 當看到這行敘述，表示 Parallel.ForEach 已經結束執行，不過，將還沒看到所有的 Cpf 文字輸出
Console.WriteLine($"使用 Parallel.Foreach 與同步委派方法 結束 {DateTime.Now}");
```

在這裡，將會使用 `Enumerable.Range(0, 3)` 一個列舉物件，且這個列舉物件內有三個值，並使用 Lambda 指定一個委派方法，該方法將會使用 `Thread.Sleep(3000)` 模擬該方法需要花費三秒鐘的時間來執行某些工作，整個 Lambda 委派方法將會是使用同步方式來進行運行。

從底下的執行結果可以看出，一旦 `Parallel.ForEach` 開始執行之後，將會平行執行 `Enumerable.Range(0, 3)` 列舉物件內的每個值，

```
使用 Parallel.Foreach 與同步委派方法 開始 2000/9/2 上午 09:23:09
  Bpfsync
  Bpfsync
  Bpfsync
  Cpfsync
  Cpfsync
  Cpfsync
使用 Parallel.Foreach 與同步委派方法 結束 2000/9/2 上午 09:23:12
```

若將程式碼改成如下，將每個平行執行的傳入物件值顯示出來

```csharp
#region 若在 Parallel.Foreach 內使用同步方式進行委派方法的設計，會等到所有委派方法都執行完畢後，才會繼續往下執行
Console.WriteLine($"使用 Parallel.Foreach 與同步委派方法 開始 {DateTime.Now}");
Parallel.ForEach(Enumerable.Range(0, 3), async (x, t) =>
{
    Console.WriteLine($"  Bpfsync {x}");
    Thread.Sleep(3000);
    Console.WriteLine($"  Cpfsync {x}");
});
// 當看到這行敘述，表示 Parallel.ForEach 已經結束執行，不過，將還沒看到所有的 Cpf 文字輸出
Console.WriteLine($"使用 Parallel.Foreach 與同步委派方法 結束 {DateTime.Now}");
#endregion
```

這裡將會是上面程式碼執行後的結果，從這裡可以再度驗證與得到一個結論， Parallel.ForEach 確實逐一平行來執行美個委派方法，可以在平行執行作業過程中，是每有說哪個執行緒必須一定要先執行，或者要按著當初啟動的順序來逐一平行執行的慣例或者說法。

```
使用 Parallel.Foreach 與同步委派方法 開始 2000/9/2 上午 09:34:12
  Bpfsync 0
  Bpfsync 1
  Bpfsync 2
  Cpfsync 0
  Cpfsync 2
  Cpfsync 1
使用 Parallel.Foreach 與同步委派方法 結束 2000/9/2 上午 09:34:15
```

現在將要平行運行的委派方法改成非同步委派方法，因此，原先使用 Thread.Sleep(3000) 這個敘述，將會改成使用 await Task.Delay(3000) 這樣的敘述，然而，因為使用了 await 運算子，所以，就需要在委派方法前面加上 async 修飾詞，底下將會是這樣的程式碼

```csharp
#region 若在 Parallel.Foreach 內使用 async 方法，將會立即結束平行敘述，相關程式碼會在背景執行中
Console.WriteLine($"使用 Parallel.Foreach 開始 {DateTime.Now}");
Parallel.ForEach(Enumerable.Range(0, 3), async (x, t) =>
{
    Console.WriteLine($"  Bpf {x}");
    await Task.Delay(3000);
    Console.WriteLine($"  Cpf {x}");
});
// 當看到這行敘述，表示 Parallel.ForEach 已經結束執行，不過，將還沒看到所有的 Cpf 文字輸出
Console.WriteLine($"使用 Parallel.Foreach 結束 {DateTime.Now}");
#endregion


#region 故意休息五秒，等待上述的平行作業全部都結束
Console.WriteLine();
Console.WriteLine($"休息 五秒鐘");
await Task.Delay(5000);
Console.WriteLine();
```

因為若在 Parallel.Foreach 內使用 async 方法，將會立即結束平行敘述，相關委派方法內程式碼會仍在背景執行中，因此，無法透過 Parallel.ForEach 來得知是否所有的平行運算都已經全部完成了；由於平行計算模擬花費 3 秒計算時間，而這三秒將會在背景下運行，因此，這裡使用 await Task.Delay(5000) 這樣的敘述，故意休息五秒，等待上述的所有背景平行作業全部都結束，因此將會看到底下的輸出結果；從執行結果可以看出，當 [休息 五秒鐘] 文字顯示之後，並且真的讓當前執行緒強制睡眠五秒鐘之後，約在三秒之後，就會看到每個委派方法執行結束的文字輸出。

```
使用 Parallel.Foreach 開始 2022/9/2 上午 09:34:15
  Bpf 0
  Bpf 2
  Bpf 1
使用 Parallel.Foreach 結束 2022/9/2 上午 09:34:15

休息 五秒鐘
  Cpf 2
  Cpf 0
  Cpf 1
  ```

為了解決這樣的應用，在 .NET 6 的 BCL 中，將會提供了 [Parallel.ForEachAsync 方法](https://docs.microsoft.com/zh-tw/dotnet/api/system.threading.tasks.parallel.foreachasync?WT.mc_id=DT-MVP-5002220) ，透過這個方法，將會可以做到使用 Parallel 類別提供的功能，並且使用非同步的方法來平行執行委派方法

```csharp
#region 這裡使用 Parallel.ForEachAsync 來平行非同步方法，將不會有上述問題，全部的非同步作業都平行執行完畢，該行敘述才會繼續往下執行，這可以從時間戳記看出
Console.WriteLine($"使用 Parallel.ForEachAsync 開始 {DateTime.Now}");
await Parallel.ForEachAsync(Enumerable.Range(0, 3), async (x, t) =>
{
    Console.WriteLine($"  Bpfa {x}");
    await Task.Delay(3000);
    Console.WriteLine($"  Cpfa {x}");
});
Console.WriteLine($"使用 Parallel.ForEachAsync 結束 {DateTime.Now}");
#endregion
```

在這裡將會使用 `await Parallel.ForEachAsync` 來使用非封鎖方式來等待所有的平行作業都執行完畢，而且這些要採用平行執行的委派方法，都將採用非同步方法(有 async 修飾詞的方法)來設計。

從底下的執行結果，應該是當初需求所期望能夠設計出來的功能

```
使用 Parallel.ForEachAsync 開始 2000/9/2 上午 09:34:20
  Bpfa 0
  Bpfa 1
  Bpfa 2
  Cpfa 2
  Cpfa 1
  Cpfa 0
使用 Parallel.ForEachAsync 結束 2000/9/2 上午 09:34:27
```
