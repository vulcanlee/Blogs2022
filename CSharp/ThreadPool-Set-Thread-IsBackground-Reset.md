# 當從執行緒集區取得執行緒後，將執行緒宣告為前景執行緒，若再次取得，還是會有前景執行緒嗎？

```csharp
using System.Collections.Concurrent;
using System.Diagnostics;

namespace Change_ThreadPool_Thread_Property_Question
{
    internal class Program
    {
        //static Dictionary<int, Thread> threads = new Dictionary<int, Thread>();
        static ConcurrentBag<Thread> queue = new ConcurrentBag<Thread>();
        static void Main(string[] args)
        {
            Console.WriteLine($"邏輯處理器數量 : {Environment.ProcessorCount}");

            Console.WriteLine("請求邏輯處理器數量的執行緒");
            for (int i = 0; i < Environment.ProcessorCount; i++)
            {
                int idx = i;
                ThreadPool.QueueUserWorkItem(_ =>
                {
                    // 設定執行緒的相關參數
                    Console.Write($"{idx}");
                    Thread thread = Thread.CurrentThread;
                    thread.Name = $"重新設定{idx}";
                    thread.IsBackground = (idx % 3 == 0) ? false : true;
                    queue.Add(thread);
                    Console.Write($"*");
                    Thread.Sleep(10000);
                });
            }

            Thread.Sleep(8000);
            Console.WriteLine($"查看所取得的執行緒狀態");

            Thread getThread = null;
            while (queue.Count > 0)
            {
                queue.TryTake(out getThread);
                Console.WriteLine($"ID : {getThread.ManagedThreadId}");
                Console.WriteLine($"Name : {getThread.Name}");
                Console.WriteLine($"IsBackground : {getThread.IsBackground}");
                Console.WriteLine();
            }

            Console.WriteLine($"休息10秒鐘，重新配置相同數量執行緒");
            Thread.Sleep(10000);
            queue.Clear();
            for (int i = 0; i < Environment.ProcessorCount; i++)
            {
                int idx = i;
                ThreadPool.QueueUserWorkItem(_ =>
                {
                    Console.Write($"{idx}");
                    Thread thread = Thread.CurrentThread;
                    queue.Add(thread);
                    Thread.Sleep(10000);
                });
            }

            Thread.Sleep(8000);
            Console.WriteLine($"第二次查看所取得的執行緒狀態");
            while (queue.Count > 0)
            {
                queue.TryTake(out getThread);
                Console.WriteLine($"ID : {getThread.ManagedThreadId}");
                Console.WriteLine($"Name : {getThread.Name}");
                Console.WriteLine($"IsBackground : {getThread.IsBackground}");
                Console.WriteLine();
            }

            Console.WriteLine("Press any key for continuing...");
            Console.ReadKey();
        }
    }
}
```

執行後的輸出結果

```
邏輯處理器數量 : 8
請求邏輯處理器數量的執行緒
0132456*******7*查看所取得的執行緒狀態
ID : 15
Name : 重新設定7
IsBackground : True

ID : 11
Name : 重新設定2
IsBackground : True

ID : 12
Name : 重新設定4
IsBackground : True

ID : 6
Name : 重新設定0
IsBackground : False

ID : 14
Name : 重新設定5
IsBackground : True

ID : 10
Name : 重新設定3
IsBackground : False

ID : 9
Name : 重新設定1
IsBackground : True

ID : 13
Name : 重新設定6
IsBackground : False

休息10秒鐘，重新配置相同數量執行緒
23146750第二次查看所取得的執行緒狀態
ID : 16
Name : .NET ThreadPool Worker
IsBackground : True

ID : 15
Name : .NET ThreadPool Worker
IsBackground : True

ID : 11
Name : .NET ThreadPool Worker
IsBackground : True

ID : 12
Name : .NET ThreadPool Worker
IsBackground : True

ID : 6
Name : .NET ThreadPool Worker
IsBackground : True

ID : 14
Name : .NET ThreadPool Worker
IsBackground : True

ID : 9
Name : .NET ThreadPool Worker
IsBackground : True

ID : 13
Name : .NET ThreadPool Worker
IsBackground : True

Press any key for continuing...
```

