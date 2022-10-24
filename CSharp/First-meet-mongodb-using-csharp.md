# 第一次使用 C# 來存取 NoSQL 的 MongoDB 資料庫

## 第一次使用的準備工作

為了簡化第一次體驗使用 MongoDB 的經驗，這裡將會採用 https://www.mongodb.com/atlas/database 上面提供的雲端 MongoDB 資料庫服務，因此，此時需要先連上這個網站，並且登入或者新註冊一個帳號。

一旦開啟這個網站，可以看到該網頁右上方有個 Sign in 連結，我是直接點選這個連結來開始操作

![](../Images/net883.png)

從下面的畫面上可以看到，可以直接使用 Google 或者 GitHub 帳號來進行登入，或者點選右下方的 Sign Up 連結，自行註冊一個新的帳號；在此，我是使用 Google 帳號來登入到 MongoDB 雲端資料庫服務內。

![](../Images/net897.png)

一旦登入完成後，就會看到底下的畫面

![](../Images/net896.png)

此時，點選 [Build a Database] 按鈕，根據網頁上的描述，這個按鈕將會 : Create a database , Choose your cloud provider, region, and spaces.

豪不猶豫的點選這個 [Build a Database] 按鈕

現在將會出現下面畫面，準備要部署一個雲端資料庫，這裡提供了三種方案，對於新手小白而言，可以先選擇 [Starting at FREE] 這個免費方案來先開始。

對於 Shared 這個方案，官方的描述為： For learning and exploring MongoDB in a clound environment. Basic configuration options.

* No credit card required to start
* Explorer with sample datasets
* Upgrade to dedicated clusters for full functionality

因此，沒有問題的話，點選 [Shared] 這個方案內的 [Create] 按鈕

![](../Images/net895.png)

現在進入到了 [Create a Shared Cluster] 頁面，在這裡可以決定要把這個雲端資料放在哪個雲端服務供應商與哪個區域內，在這裡，先不做相關設定，因此，根據預設值，使用的是 AWS 這個供應商。

![](../Images/net894.png)

現在捲動到該畫面的最下方，將會看到 [Cluster Name] 這個項目區塊，在這裡我自己指定了 `VulcanMongo` 這個叢集 Cluster 名稱，最後，點選該網頁右下方的 [Create Cluster] 按鈕

![](../Images/net893.png)

在 [Security Quickstart] 頁面下，首先要先來設定使用這個 MongoDB 雲端資料庫要使用到的帳號與密碼，請在 [Username] & [Password] 這兩個欄位內輸入想要使用的帳號與密碼，接著，點選 [Create User] 按鈕

完成帳號與密碼設定之後，將會看到類似下面的畫面截圖

![](../Images/net892.png)

![](../Images/net891.png)

最後一個設定步驟，那將是要設定可以從哪裡來連線到這個 Mongo 雲端資料庫，因為是要用於測試開發之用，可以能隨時從不同 IP 來源連線到這個 MongoDB 資料庫上，所以，使用 0.0.0.0/0 ，指定所有的 IP 都可以連上這個資料庫。

建立完成這個 [IP Access List] 後，請點選該網頁右下方的 [Finish and Close] 按鈕

![](../Images/net890.png)

此時，網頁顯示了 Congratulations on setting up access rules 這樣的訊息，請點選該對話窗右下方的 [Go to Database] 按鈕

![](../Images/net889.png)

好的，現在資料庫已經建立與部署好了，請根據下面畫面，找到 [VulcanMongo] 這個名詞的最右邊，找到 ... 按鈕

點選這個 ... 按鈕，將會看到一個彈出子視窗，從這裡找到並且點選 [Load Sample Dataset] 這個選項

![](../Images/net888.png)

此時，網頁將會顯示一個對話窗，如下圖所示，請點選 [Load Sample Dataset] 這個按鈕，根據這個對話窗顯示的內容，這些範例資料集 Dataset，將會使用約 350MB 空間

![](../Images/net887.png)

現在，請在 [Database Deployments] 網頁中，找到 [Connect] 按鈕

![](../Images/net886.png)

這個時候，將會出現一個對話窗 [Connect to VulcanMongo] ，由於這裡需要使用 C# 程式語言來讀取雲端 MongoDB 內的資料，所以，在此要選擇 [Connect your application] Connect your application to your cluster using MongoDB's native drivers.

![](../Images/net885.png)

在接下來的對話窗中，在 [DRIVER] 下拉選單內，選擇 [C# / .NET] 這個選項，而在 [VERSION] 下拉選單中，選擇了 [2.5 or later] 這個項目。

在 [Add your connection string into your application code] 文字區塊的下方，將會列出要連線到這台雲端 MongoDB 資料庫的時候，需要用到的連線字串參數，取得該字串之後，請將 `<password>` 文字替換成為剛剛所設定的密碼，在此，要特別注意的是，這裡所指定的密碼文字，將會是要有經過 URL encode 處理過的。

![](../Images/net884.png)

若有勾選 Include full driver code example 檢查盒，將會出現底下的 C# 使用 MongoDB 的代碼

```csharp

var settings = MongoClientSettings.FromConnectionString("mongodb+srv://vulcan:<password>@vulcanmongo.hptf95d.mongodb.net/?retryWrites=true&w=majority");
var client = new MongoClient(settings);
var database = client.GetDatabase("test");

```

取得與複製要連線到雲端 MongoDB 的連線資訊或者範例代碼，點選該對話窗下的 [Close] 按鈕

## 建立 C# 專案來存取 MongoDB 內的紀錄

* 打開 Visual Studio 2022
* 點選右下方的 [建立新的專案] 按鈕
* 選擇一個 [主控台應用程式] 的專案範本
* 點選右下方的 [下一步] 按鈕
* 在 [設定新的專案] 對話窗內，在 [專案名稱] 欄位中，輸入 `csMongoFirst`
* 點選右下方的 [下一步] 按鈕
* 在 [其他資訊] 對話窗中
* 取消 [Do not use top-level statements] 這個 checkbox 檢查盒的勾選
* 點選右下方的 [建立] 按鈕
* 透過 NuGet 工具，搜尋到 MongoDB.Driver 這個套件，將其安裝到這個專案
* 打開 Program.cs 檔案，將底下內容替換掉這個檔案內容

```csharp
using MongoDB.Bson;
using MongoDB.Driver;

namespace csMongoFirst;

internal class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("Hello, World!");
        var settings = MongoClientSettings
            .FromConnectionString(
            "mongodb+srv://vulcan:P%40ssw0rd@vulcanmongo.hptf95d.mongodb.net/?retryWrites=true&w=majority");
        var client = new MongoClient(settings);
        var database = client.GetDatabase("sample_mflix");
        var collection = database.GetCollection<BsonDocument>("movies");
        var result = collection.Find("{title:'The Princess Bride'}").FirstOrDefault();
        Console.WriteLine(result);
    }
}
```

現在，可以實際執行這個專案，將可以看到讀取到的 MongoDB 內的 JSON 資料

```
Hello, World!
{ "_id" : ObjectId("573a1398f29313caabcea974"), "plot" : "While home sick in bed, a young boy's grandfather reads him a story called The Princess Bride.", "genres" : ["Adventure", "Comedy", "Family"], "runtime" : 98, "metacritic" : 77, "rated" : "PG", "cast" : ["Cary Elwes", "Mandy Patinkin", "Chris Sarandon", "Christopher Guest"], "poster" : "https://m.media-amazon.com/images/M/MV5BMGM4M2Q5N2MtNThkZS00NTc1LTk1NTItNWEyZjJjNDRmNDk5XkEyXkFqcGdeQXVyMjA0MDQ0Mjc@._V1_SY1000_SX677_AL_.jpg", "title" : "The Princess Bride", "fullplot" : "A kindly grandfather sits down with his ill grandson and reads him a story. The story is one that has been passed down from father to son for generations. As the grandfather reads the story, the action comes alive. The story is a classic tale of love and adventure as the beautiful Buttercup, engaged to the odious Prince Humperdinck, is kidnapped and held against her will in order to start a war, It is up to Westley (her childhood beau, now returned as the Dread Pirate Roberts) to save her. On the way he meets a thief and his hired helpers, an accomplished swordsman and a huge, super strong giant, both of whom become Westley's companions in his quest.", "languages" : ["English"], "released" : ISODate("1987-10-09T00:00:00Z"), "directors" : ["Rob Reiner"], "writers" : ["William Goldman (book)", "William Goldman (screenplay)"], "awards" : { "wins" : 7, "nominations" : 8, "text" : "Nominated for 1 Oscar. Another 6 wins & 8 nominations." }, "lastupdated" : "2015-09-10 17:41:34.337000000", "year" : 1987, "imdb" : { "rating" : 8.1999999999999993, "votes" : 267597, "id" : 93779 }, "countries" : ["USA"], "type" : "movie", "tomatoes" : { "website" : "http://www.theprincessbride-themovie.com/", "viewer" : { "rating" : 4.0, "numReviews" : 523369, "meter" : 95 }, "dvd" : ISODate("1999-01-26T00:00:00Z"), "critic" : { "rating" : 8.3000000000000007, "numReviews" : 63, "meter" : 97 }, "lastUpdated" : ISODate("2015-09-12T18:16:37Z"), "consensus" : "A delightfully postmodern fairy tale, The Princess Bride is a deft, intelligent mix of swashbuckling, romance, and comedy that takes an age-old damsel-in-distress story and makes it fresh.", "rotten" : 2, "production" : "20th Century Fox", "fresh" : 61 }, "num_mflix_comments" : 0 }
```


