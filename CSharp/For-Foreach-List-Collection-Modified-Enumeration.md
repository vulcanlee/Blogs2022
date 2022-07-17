# 多執行的集合型別物件存取

```csharp
namespace ConsoleApp5
{
    internal class Program
    {
        static List<string> list = new List<string>();
        static void Main(string[] args)
        {

            new Thread(() =>
            {
                Run1();
            })
            { IsBackground = true }.Start();
            new Thread(() =>
            {
                Run2();
                //Run3();
            })
            { IsBackground = true }.Start();

            Console.ReadKey();
        }


        static void Run1()
        {
            while (true)
            {
                list.Add("A");
            }
        }
        static void Run2()
        {
            while (true)
            {
                foreach (var item in list)
                {

                }
                list.Clear();
            }
        }

        static void Run3()
        {
            while (true)
            {
                for (int i = 0; i < list.Count; i++)
                {
                    var b = list[i];
                }
                list.Clear();
            }
        }
    }
}
```

```csharp
namespace ConsoleApp6
{
    internal class Program
    {
        static List<string> list = new List<string>();
        static void Main(string[] args)
        {
            for (int i = 0; i < 100; i++)
            {
                list.Add("A");
            }
            ThreadPool.QueueUserWorkItem(_ =>
            {
                foreach (var item in list)
                {
                    Thread.Sleep(500);
                    Console.Write(".");
                }
                list.Clear();
            });

            Console.WriteLine("Press any key for continuing...");
            Console.ReadKey();

            for (int i = 0; i < 100; i++)
            {
                list.Add("A");
            }

            Console.WriteLine("Press any key for continuing...");
            Console.ReadKey();
        }
    }
}
```
