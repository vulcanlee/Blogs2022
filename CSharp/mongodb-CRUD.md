# 如何在 .NET C# 內，使用 MongoDB 提供的套件，針對 MongoDB 來進行 CRUD 的操作

## 建立 C# 專案來存取 MongoDB 內的紀錄

* 打開 Visual Studio 2022
* 點選右下方的 [建立新的專案] 按鈕
* 選擇一個 [主控台應用程式] 的專案範本
* 點選右下方的 [下一步] 按鈕
* 在 [設定新的專案] 對話窗內，在 [專案名稱] 欄位中，輸入 `csMongoCrud`
* 點選右下方的 [下一步] 按鈕
* 在 [其他資訊] 對話窗中
* 取消 [Do not use top-level statements] 這個 checkbox 檢查盒的勾選
* 點選右下方的 [建立] 按鈕
* 透過 NuGet 工具，搜尋到 MongoDB.Driver 與 Newtonsoft.Json 套件，將其安裝到這個專案
* 打開 Program.cs 檔案，將底下內容替換掉這個檔案內容

```csharp
using MongoDB.Bson;
using MongoDB.Driver;
using Newtonsoft.Json;

namespace csMongoCrud;

internal class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine("Hello, World!");

        #region 連線的準備工作
        var settings = MongoClientSettings
        .FromConnectionString(
        "mongodb+srv://vulcan:P%40ssw0rd@vulcanmongo.hptf95d.mongodb.net/?retryWrites=true&w=majority");
        var client = new MongoClient(settings);

        client.DropDatabase("MyCrud");
        var db = client.GetDatabase("MyCrud");
        db.DropCollection("MyCollection");
        var collection = db.GetCollection<MyCrudModel>("MyCollection");
        #endregion

        #region 新增
        Console.WriteLine($"開始進行新增操作");
        int i = 99;
        var newItem = new MyCrudModel()
        {
            Name = $"Auto Name {i}",
            Age = i,
            Birthday = DateTime.Now.AddDays(i * -1),
            IsAudit = i % 3 == 0,
        };
        collection.InsertOne(newItem);
        Console.WriteLine($"完成後的JSON物件:" +
            $"{JsonConvert.SerializeObject(newItem, Formatting.Indented)}");
        #endregion

        #region 查詢
        Console.WriteLine($"開始進行查詢 : Age=99");
        FilterDefinitionBuilder<MyCrudModel> builderFilter = Builders<MyCrudModel>.Filter;
        FilterDefinition<MyCrudModel> filter = 
            builderFilter.Eq(x => x.Age, 99);
        var retriveItem = collection.Find(filter).FirstOrDefault();
        Console.WriteLine($"查詢完成後的JSON物件:" +
            $"{JsonConvert.SerializeObject(retriveItem, Formatting.Indented)}");
        #endregion

        #region 更新
        Console.WriteLine($"開始進行查詢 : Age=168");
        FilterDefinition<MyCrudModel> filterByUpdate =
           builderFilter.Eq(x => x.Id, newItem.Id);
        var update = Builders<MyCrudModel>.Update.Set(x => x.Age, 168);
        collection.UpdateOne(filterByUpdate, update);
        #endregion

        #region 刪除
        Console.WriteLine($"開始進行刪除 : Age=168");
        FilterDefinition<MyCrudModel> filterByDelete =
           builderFilter.Eq(x => x.Id, newItem.Id);
        collection.DeleteOne(filterByDelete);
        #endregion
    }
}
```

在這個主控台的進入點程式碼中，首先會使用 MongoClientSettings.FromConnectionString 方法，傳入雲端的 MongoDB 系統的連線字串，該字串內有該主機資訊、帳號、密碼等宣告。

取得這個物件之後，就可以使用這個物件傳入到 MongoClient 建構式內，建立一個可以呼叫 MongoDB API 的主要物件。

為了要簡化與明瞭整體 CRUD 的操作，在此使用 `client.DropDatabase("MyCrud");` 敘述，將測試用的資料庫刪除掉，接著，執行 `var db = client.GetDatabase("MyCrud");` 敘述，要取得剛剛刪掉的資料庫，雖然資料庫不存在，但是，這個 MangoDB API，將會自動在 MongoDB 主機上，建立起一個新的資料庫；反過來說，若是該資料庫已經存在，則會直接取得該資料庫，並且指定到 db 物件內。

接下來同樣的先使用 `db.DropCollection("MyCollection");` 刪除掉該資料庫內的 MyCollection Collection，而後使用 `var collection = db.GetCollection<MyCrudModel>("MyCollection");` 這個敘述，將可以自動重新在 MongoDB 資料庫內來建立起該 Collection。

對於要新增一筆紀錄到 MongoDB 內，首先先建立一個 MyCrudModel 物件，並且設定該物件內的相關屬性擁有的值。

使用 collection.InsertOne(newItem); 敘述來將該物件新增到 MongoDB 資料庫內，一旦這行敘述執行完畢之後，將會在 MongoDB 內看到這筆紀錄已經產生出來了，而且對於 newItem.Id 這個屬性，也會擁有這筆紀錄在資料庫內實際的唯一 ID 內容值。

![](../Images/net882.png)

想要進行查詢，先透過 Builders<MyCrudModel>.Filter 表示式，0建立一個型別為 FilterDefinitionBuilder<MyCrudModel> 的物件，接著使用剛剛取得的物件，搭配各種邏輯條件方法，建立查詢條件，這裡透過了 Lambda 表示式，建立了一個強型別查詢條件，那就是 Age 欄位的值必須等於 99 這樣的條件，這樣的查詢條件將會以 FilterDefinition<MyCrudModel> 型別來儲存到 filter 物件內

最後，透過 collection.Find(filter).FirstOrDefault(); 表示式來查詢出所要的紀錄。

對於要更新紀錄，則是與傳統資料庫的做法比較不太相同，首先，先要建立起 FilterDefinition<MyCrudModel> filterByUpdate 的物件，在該物件內將會有過濾查詢條件，接著，使用 Builders<MyCrudModel>.Update.Set(x => x.Age, 168) 表示式來宣告要如何進行欄位的更新，最後呼叫 collection.UpdateOne(filterByUpdate, update) 方法，把這兩個物件傳遞進去，完成要更新動作。

要進行刪除的話，還是同樣先要建立一個 FilterDefinition<MyCrudModel> filterByDelete 物件，這裡將會定義查詢條件為何？由於要刪除特定的筆紀錄，因此，使用該物件的唯一 ID 來建立查詢條件，最後呼叫 collection.DeleteOne(filterByDelete) 方法，完成刪除的指定動作。

到現在為止，已經完成了與 MongoDB 的連線準備工作，接下來要進行在 MongoDB 內，進行 CRUD 的操作。

最後，建立該 Collection 內的節點會用到的實際資料類別。

* 在此專案節點，使用滑鼠右擊
* 從彈跳視窗中選擇 [加入] > [新增] 選項
* 在名稱欄位內輸入 ``
* 使用底下程式碼將剛剛建立的內容替換掉

```csharp
using MongoDB.Bson;
using MongoDB.Bson.Serialization.Attributes;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading.Tasks;

namespace csMongoCrud;

[BsonIgnoreExtraElements]
public class MyCrudModel
{
    [BsonId]
    public ObjectId Id { get; set; }
    public string Name { get; set; }
    public int Age { get; set; }
    public DateTime Birthday { get; set; }
    public bool IsAudit { get; set; }
}
```

這裡建立的類別將會是要儲存到 MongoDB 的 Collection 內，而所使用的 MongoDB 套件，則會把 .NET 的物件，轉換成為 BSON 型別的物件，進而儲存到 MongoDB 內。

在這個類別中，將會使用到 [BsonIgnoreExtraElements] 屬性，這表示了將會忽略不在這個類別內的 JSON 屬性，而第一個類別屬性宣告為 ObjectId 型別的 Id，這裡將會儲存 MongoDB 內的 Key Id Value。

底下將會是執行結果，這些程式碼將會先刪除該資料庫，接著使用 GetDatabase("MyCrud") 方法呼叫後，便會自動建立起該資料庫，對於 Collection 也是使用同樣的作法。

而後將會進行 CRUD 新增、查詢、更新、刪除的操作，底下是執行輸出內容

```
Hello, World!
開始進行新增操作
完成後的JSON物件:{
  "Id": "63565be56c74a3733db11c18",
  "Name": "Auto Name 99",
  "Age": 99,
  "Birthday": "2022-07-17T17:33:25.2563571+08:00",
  "IsAudit": true
}
開始進行查詢 : Age=99
查詢完成後的JSON物件:{
  "Id": "63565be56c74a3733db11c18",
  "Name": "Auto Name 99",
  "Age": 99,
  "Birthday": "2022-07-17T09:33:25.256Z",
  "IsAudit": true
}
開始進行查詢 : Age=168
開始進行刪除 : Age=168
```


