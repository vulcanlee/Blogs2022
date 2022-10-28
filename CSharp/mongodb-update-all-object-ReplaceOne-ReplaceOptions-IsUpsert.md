# 在 mongodb 內要來更新整個物件，無須個別指定要更新的欄位屬性

一開始學習 mongodb 的時候，看到要更新紀錄的時候，文件或者文章內將會告訴你先要建立一個過濾器 ，如這樣的用法： 

```
FilterDefinition<MyCrudModel> filterByUpdate = builderFilter.Eq(x => x.Id, newItem.Id);
```

在這裡將會過濾出所傳送進來的物件 Id 相符合的所有物件出來

而後，將會建立一個更新器，在此同樣的透過 Builder 類別，不過使用 Update 方法與 Set 方法來指定那些屬性要進行更新與更新成為甚麼內容，如底下程式碼所示：

```
var update = Builders<MyCrudModel>.Update.Set(x => x.Age, 168);
```

準備好上述兩個物件之後，便可以呼叫 UpdateOne 這個方法，通知 Mongodb 進行相關紀錄的更新

```
collection.UpdateOne(filterByUpdate, update);
```

然而，我突然有個想法，若此時我想要更新者個物件內容，而不想個別指定物件內的不同屬性來進行更新，此時，又該怎麼辦呢？


## 建立 C# 專案來將某個物件全部屬性更新到 MongoDB 內的紀錄

* 打開 Visual Studio 2022
* 點選右下方的 [建立新的專案] 按鈕
* 選擇一個 [主控台應用程式] 的專案範本
* 點選右下方的 [下一步] 按鈕
* 在 [設定新的專案] 對話窗內，在 [專案名稱] 欄位中，輸入 `csMongoUpdate`
* 點選右下方的 [下一步] 按鈕
* 在 [其他資訊] 對話窗中
* 取消 [Do not use top-level statements] 這個 checkbox 檢查盒的勾選
* 點選右下方的 [建立] 按鈕
* 透過 NuGet 工具，搜尋到 MongoDB.Driver 與 Newtonsoft.Json 套件，將其安裝到這個專案
* 打開 Program.cs 檔案，將底下內容替換掉這個檔案內容

```csharp
using MongoDB.Driver;
using Newtonsoft.Json;

namespace csMongoUpdate;

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

        #region 更新，直接更新整個物件，無須個別指定要更新的欄位
        Console.WriteLine($"開始進行查詢 : Age=168");
        newItem.Birthday = DateTime.Now.AddDays(7);
        newItem.Name = "Vulcan Lee";
        newItem.Age = 168;
        FilterDefinitionBuilder<MyCrudModel> builderFilter = Builders<MyCrudModel>.Filter;
        FilterDefinition<MyCrudModel> filterByUpdate =
           builderFilter.Eq(x => x.Id, newItem.Id);
        collection.ReplaceOne(filterByUpdate, newItem, new ReplaceOptions { IsUpsert = true });
        Console.WriteLine($"完成後的JSON物件:" +
            $"{JsonConvert.SerializeObject(newItem, Formatting.Indented)}");
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

現在要來進行直接更新整個物件，無須個別指定要更新的欄位的設計做法，首先，把要修改的內容進行調整，也就是更新 newItem 物件內的屬性值

同樣的，還是要先建立一個過濾器物件 `FilterDefinitionBuilder<MyCrudModel> builderFilter = Builders<MyCrudModel>.Filter;` ，接著將這個過濾器物件宣告要過濾的條件，也就是要找出與 newItem.Id 屬性值具有同樣 Id 的物件出來，這裡將會使用這樣的程式碼 `FilterDefinition<MyCrudModel> filterByUpdate = builderFilter.Eq(x => x.Id, newItem.Id);`

在每個 Collection 物件內，其實有提供一個方法 ReplaceOne ，該方法就是可以做到這篇文章想要做的事情，因此，可以使用 `collection.ReplaceOne(filterByUpdate, newItem, new ReplaceOptions { IsUpsert = true });` 這樣的方法來完成需求。

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

namespace csMongoUpdate;

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

執行這個專案，將會在命令提示字元視窗內，看到底下的輸出內容

```
Hello, World!
開始進行新增操作
完成後的JSON物件:{
  "Id": "635b31d7ec445751d6934e51",
  "Name": "Auto Name 99",
  "Age": 99,
  "Birthday": "2022-07-21T09:35:19.2551566+08:00",
  "IsAudit": true
}
開始進行查詢 : Age=168
完成後的JSON物件:{
  "Id": "635b31d7ec445751d6934e51",
  "Name": "Vulcan Lee",
  "Age": 168,
  "Birthday": "2022-11-04T09:35:19.6053779+08:00",
  "IsAudit": true
}
```

