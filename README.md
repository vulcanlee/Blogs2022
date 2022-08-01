# Vulcan Lee 2022 部落格文章清單 C# / .NET Core / Blazor / MAUI

|類型|文章檔案|文章名稱|部落格網址|
|-|-|-|-|
|Blazor|[](Blazor/.md)||https://csharpkh.blogspot.com/2022/04/.html|
|||||
|||||
|Blazor|[](Blazor/.md)||https://csharpkh.blogspot.com/2022/04/.html|
|CSharp|[](CSharp/.md)||https://csharpkh.blogspot.com/2022/04/.html|
|MAUI|[](MAUI/.md)||https://csharpkh.blogspot.com/2022/04/.html|
|ASPNETCore|[](ASPNETCore/.md)||https://csharpkh.blogspot.com/2022/04/.html|
|||||

嚴重性	程式碼	說明	專案	檔案	行	隱藏項目狀態
錯誤	XA5301	Failed to generate Java type for class: AndroidX.Core.View.Accessibility.AccessibilityManagerCompat/ITouchExplorationStateChangeListenerImplementor due to MAX_PATH: System.IO.DirectoryNotFoundException: 找不到路徑 'C:\Vulcan\Github\Using-Prism-Create-MAUI-App-Project\2 Build Core Service and Model\PrismMonkey\PrismMonkey\obj\Debug\net6.0-android\android\src\mono\androidx\core\view\accessibility\AccessibilityManagerCompat_TouchExplorationStateChangeListenerImplementor.java' 的一部分。
   於 System.IO.__Error.WinIOError(Int32 errorCode, String maybeFullPath)
   於 System.IO.FileStream.Init(String path, FileMode mode, FileAccess access, Int32 rights, Boolean useRights, FileShare share, Int32 bufferSize, FileOptions options, SECURITY_ATTRIBUTES secAttrs, String msgPath, Boolean bFromProxy, Boolean useLongPath, Boolean checkHost)
   於 System.IO.FileStream..ctor(String path, FileMode mode, FileAccess access, FileShare share, Int32 bufferSize)
   於 Microsoft.Android.Build.Tasks.Files.CopyIfStreamChanged(Stream stream, String destination) 於 /Users/runner/work/1/s/xamarin-android/external/xamarin-android-tools/src/Microsoft.Android.Build.BaseTasks/Files.cs: 行 170
   於 Xamarin.Android.Tasks.GenerateJavaStubs.CreateJavaSources(IEnumerable`1 javaTypes, TypeDefinitionCache cache)	PrismMonkey	C:\Program Files\dotnet\packs\Microsoft.Android.Sdk.Windows\32.0.440\tools\Xamarin.Android.Common.targets	1438	
