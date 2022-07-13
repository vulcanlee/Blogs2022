# C# : 將 EF Core 的資料模型類別庫，打包成為 NuGet 套件，並且上傳到公開 NuGet 伺服器上

在這篇文章，將會延續上一篇 [EF Core : 使用資料庫反向工程，取得 EF Core 的資料模型](https://csharpkh.blogspot.com/2022/07/Database-Reverse-Engineer-Model-Scaffold-DbContext.html) 的開發結果，準備要將已經開發好的一個 Entity Framework Core 模型類別庫，打包成為 [NuGet 套件](https://docs.microsoft.com/zh-tw/nuget/what-is-nuget?WT.mc_id=DT-MVP-5002220)，不過，在這裡將不會把這個 NuGet 套件上傳到 [www.nuget.org](https://www.nuget.org/) 網站上，而是會上傳到另外一個方便管理的 [MyGet](https://www.myget.org/) 伺服器上

## 將類別庫專案設定可以產生 NuGet 套件

* 再度開啟上篇文章所建立的 [DBReverse] 類別庫專案
* 滑鼠右擊 [DBReverse] 專案節點
* 從彈出功能表中，點選 [屬性] 選項
* 此時，將會看到該專案的 [屬性] 視窗顯示在螢幕上
* 請點選 [屬性] 視窗左方的清單選項 [套件] > [一般]

  ![Visual Studio 專案套件屬性](../Images/net9932.png)

* 請勾選 [在建置時產生 NuGet 套件] 下方的 Checkbox 檢查盒。
* 對於下方的其他屬性設定值，可以依照自己的需要來做調整
* 底下將會是這個類別庫專案的設定內容

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <Authors>Vulcan Lee</Authors>
    <Product>EF Core 動手實作課程</Product>
    <GeneratePackageOnBuild>True</GeneratePackageOnBuild>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="6.0.7" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Tools" Version="6.0.7">
      <PrivateAssets>all</PrivateAssets>
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
    </PackageReference>
  </ItemGroup>

</Project>
```

* 切換到 Release 模式下，建置這個類別庫專案
* 現在，可以從 Visual Studio 2022 中，看到 [bin] > [Release] 資料夾下有個 [DBReverse.1.0.0.nupkg] 檔案產生出來

  ![](../Images/net9931.png)






?WT.mc_id=DT-MVP-5002220
