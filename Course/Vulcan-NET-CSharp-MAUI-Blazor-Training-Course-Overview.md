# Vulcan Lee 在 .NET / C# 開發環境下提供的教育訓練教學課程

若對於這些課程有興趣，或者想要企業內訊者，可以在此留言，或者到 [Xamarin Blazor 實驗室](https://www.facebook.com/vulcanlabtw) 粉絲團來私訊給我

## 初探 .NET 平行程式設計 (非同步程式設計系列 之 1 / 6)

這年頭手機都多核心了我們寫的程式還跑在單核上嗎？

一台電腦8核16緒但我們的程式就是跑不快？

平行程式設計是近年來一個很務實的議題，之前 SkillTree 有開過較進階的「勇闖非同步程式設計」，收到許多開發人員的好評但有開發人員反應希望能夠開設更初階一點的入門主題，於是本活動來了！這是專門為了「沒實際摸過平行運算、非同步的開發人員」所設計的，讓你短時間掌握平行程式設計基本概念與觀念。

### 課程大綱

* 使用同步程式設計來解決問題
* 了解同步程式設計的瓶頸與使用非同步程式設計要解決的問題
* 為什麼要有平行程式設計與微軟提出的解決方案
* 將待解決問題採用平行、並行非同步程式設計 - 使用 執行緒 與 執行緒集區
* 了解平行與並行計算的不同
* 介紹什麼是非同步程式設計
* 使用 TPL Task Parallel Library 工作平行類別庫 來解決問題
* 使用 Timer，Background，委派來進行非同步程式設計
* 使用資料平行處理程式設計來解決問題
* 使用 PLINQ 來解決問題

### 等級 

入門

### 需求

* 具備 C# 程式語言開發經驗
* 具有 委派 Delegate 的知識與使用技術

### 想要參加

此課程為 Skill Tree 專屬課程，想要參加者，請隨時注意 Skill Tree 的最新公告

參考網址 : [https://skilltree.my/Events/2022/1/9/Parallel-programming-in-dotnet-for-beginners-Batch-2](https://skilltree.my/Events/2022/1/9/Parallel-programming-in-dotnet-for-beginners-Batch-2)

## 由 Parallel.For 來看多執行緒程式設計 (非同步程式設計系列 之 2 / 6)

在多執行緒程式設計領域中，有 TPL , ThreadPool , Parallel.For , PLINQ 等等技術，其目的在降低使用複雜度，提供高階程式設計模型，開發者可以很容易地使用這些功能。但坊間流竄許多種各式各樣的奇技淫巧，有些是聽從前輩的建議，有些是自身特定情境中的經驗，這些招式與看法不能說錯，但總是片片斷斷無法有系統的理解背後的原理與限制，所以 SkillTree 舉辦了本活動針對 Parallel.For 做深入的探討，藉由因循漸進的案例讓您充分了解這技術的奧妙。

本課程是平行程式設計的初階，不是程式學習的初階，您必須具備 C# 開發經驗、了解泛型與委派的使用方式，並且具備基本電腦架構運作知識。

本課程的緣由來自臉書討論版上的一連長串討論內容，有興趣的人可以查看 [討論串列](https://vulcanfiles.blob.core.windows.net/$web/Share/2022/Multi-Thread-Parallel-For.png) 螢幕截圖，若你對於這片串列上所提出的問題，或者有人提出的解答，存在著疑問或者更多的問題，那麼，你一定要來報名參加這個課程，因為，你所有的問題都可以從這個課程中獲得解答。

### 課程大綱

* Paraller.For 效能驗證
* Paraller.For 與 Thread 兩者的差異與極限
* Paraller.For 與 Task(TPL) 相互的優缺點評比
* Task 實踐與原理探討

### 等級 

入門

### 需求

* 具備 C# 程式語言開發經驗
* 具有 委派 Delegate 的知識與使用技術

### 想要參加

此課程為 Skill Tree 專屬課程，想要參加者，請隨時注意 Skill Tree 的最新公告

參考網址 : [https://skilltree.my/Events/2022/1/16/NET-CSharp-Parallel-Programming-Batch-2](https://skilltree.my/Events/2022/1/16/NET-CSharp-Parallel-Programming-Batch-2)

## 精準解析 .NET Thread 執行緒 (非同步程式設計系列 之 3 / 6)

在 .NET 要建立一個執行緒，將需要指定一個委派方法，而一個執行緒 Thread 代表一個正在同步執行程式碼，若想要同時執行多個委派方法，則需要建立多個執行緒，而一台電腦能夠同時處理執行緒的數量，將會取決於這台電腦上的 CPU 的能力。

身為一個 .NET / C# 程式設計師，想要提升自我能力，使其可以進行平行程式設計技能，就需要具備多執行緒開發技術。透過多執行緒設計出來的程式碼，將可以同時執行多個程式碼，並會有助於整體應用程式的執行效能提升，充分發揮這台電腦 CPU 的執行效能。

然而，如何進行多執行緒的程式設計，將會需要學習 .NET 中的 Thread 物件的使用與操作，當完成此課程之後，你將會具有多執行緒程式設計的能力，並且了解到多執行緒程式設計上會遇到的問題與瓶頸。

### 課程大綱

* 執行緒　　　　Thread - 基本認識
* 產生　　　　　Creation
* 啟動　　　　　Start            
* 傳入參數　　　Parameter
* 結束　　　　　Wait / Join
* 傳回值　　　　Return Value
* 優先權　　　　Priority
* 前景與背景　　Foreground / Background
* 取消　　　　　Cancellation
* 異常與除錯　　Exception

### 等級 

中階

### 需求

* 具備 C# 程式語言開發經驗
* 具有 委派 Delegate 的知識與使用技術
* 了解 平行、並行、同步、非同步等知識

### 想要參加

此課程為 Skill Tree 專屬課程，想要參加者，請隨時注意 Skill Tree 的最新公告

參考網址 : [https://skilltree.my/Events/2022/7/17/analyzing-dot-net-thread](https://skilltree.my/Events/2022/7/17/analyzing-dot-net-thread)

## 精準解析 .NET Task 工作 (非同步程式設計系列 之 4 / 6)

以往想要進行平行或非同步程式設計(平行計算是一種非同步計算，前者屬於透過 CPU 來做到同時執行的需求，後者大多表示要進行 I/O 或者網路呼叫的時候，所要進行的處理作業)，往往需要透過多執行緒來完成，可是要能夠充分駕馭執行緒來完成上述設計需求，對於絕大多數的程式設計師而言，將不是一件簡單的工作；有鑑於此，微軟在 .NET Framework 4.0 之後，推出了 工作平行類別庫 Task Parallel Library (TPL)，而在 .NET BCL 中的許多 API，也都改寫成為使用了 TAP 以工作為基礎的非同步模式 Task-based Asynchronous Pattern 的 API，取代以往 APM 與 EAP 的程式設計做法；這樣的改變將會讓 C# 程式設計師可以享受到許多 TPL 類別庫所帶來好處。

使用 TPL 中來執行的工作 Task 物件，通常是從執行緒集區上取得執行緒來以非同步方式進行執行，透過將複雜的執行緒操作封裝到工作物件內，讓程式設計師可以更加輕鬆與容易地來進平行與非同步的需求設計。

當完成此活動之後，你將會具備使用 Task 物件來進行非同步程式設計能力，讓設計出來的應用程式專案不在執行時候發生卡卡現象，充分享受到非同步程式設計所帶來的好處，當然，對於日後要精通 C# 5.0 所提供的 async 與 await 技術，將是不可或缺與必備的知識。

### 課程大綱

* 什麼是「工作」(Task)
* Thread 與 Task 的異同說明
* Task 的使用情境
  * 產生
  * 啟動或傳入參數
  * 等候結束或傳回值
  * 繼續
  * 狀態
  * 取消
  * 進度
  * 異常與除錯

### 等級 

中階

### 需求

* 具備 C# 程式語言開發經驗
* 具有 委派 Delegate 的知識與使用技術
* 了解 平行、並行、同步、非同步等知識

### 想要參加

此課程為 Skill Tree 專屬課程，想要參加者，請隨時注意 Skill Tree 的最新公告

參考網址 : [https://skilltree.my/Events/2022/7/10/analyzing-dot-net-task](https://skilltree.my/Events/2022/7/10/analyzing-dot-net-task)

## 精通與使用 async 與 await (非同步程式設計系列 之 5 / 6)

正在開發中

### 課程大綱

正在開發中

### 等級 

高階

### 需求

* 具備 C# 程式語言開發經驗
* 具有 委派 Delegate 的知識與使用技術
* 了解 平行、並行、同步、非同步等知識
* 具有 Thread 執行緒類別使用與程式設計經驗
* 具有使用 TPL Task 工作類別的使用與程式設計經驗

### 想要參加

此課程為 Skill Tree 專屬課程，想要參加者，請隨時注意 Skill Tree 的最新公告

正在開發中

## Blazor 全端開發，新手村一日脫逃術

在 Visual Studio 2022 正式版即將推出的時候你有沒有發現 Blazor 經常出現在文件內？Blazor 是微軟全新的全端解決方案，它可以讓 C# 開發者只需要會 C# 就可以達到非常棒的網頁開發能力，它與 Web forms, Silverlight的理念類似，讓開發人員只需要會 C# 就可以完成 Web APP，現在競爭激烈，如何讓開發的經驗可以共通 Blazor 就是一個絕佳的選擇，畢竟僅需要會 .NET / C# 就可以開發 Web App 是件相當誘人因素，尤其讓許多桌面應用程式的開發者頭痛的 JavaScript 在這樣的開發方式下，就變成不是必須的考量了。

不相信 Blazor 的能力，或擔心使用 Blazor 造成技術門檻過高？我們準備了一個講者的真實經驗，你會發現使用 Blazor 是可以降低技術門檻的，也讓團隊補人變的更容易

對於身為 .NET C# 開發者而言，想要成為一個全端網站工程師，將不再是夢想，因為透過 Blazor 框架，不需要會 JavaScript ，便可以輕鬆、容易、快速地完成網站專案開發；Blazor 相較於其他前端網頁開發技術，其學習曲線不會十分陡峭，對於 .NET C# 不太有經驗的人，也是可以輕鬆上手的，現在就讓 SkillTree 一起帶你逃離新手村！

### 課程大綱

* 網站開發基本知識回顧
* 了解網頁之 HTML & CSS 轉譯過程、介紹三種網頁開發架構的比較與分析
* Blazor 雙開發框架解析
* Blazor Server
  * 運作原理
  * 專案結構
* Blazor WebAssembly 
  * 運作原理
  * 專案結構
  * 如何挑選 Blazor 架構來進行開發
* Blazor 預設範本專案解析
  * Counter 元件
  * FetchData 元件
  * Razor 語法
  * Dependency Injection 觀念與用法
* Blazor 新手必學實戰技能
  * 第一個 Blazor
  * C# 程式碼設計方法
  * 單向資料綁定與重新轉譯
  * 互動與事件設計
  * 元件生命週期事件
  * 元件參數傳遞與回應事件
  * C#/JavaScript 互相呼叫
  * 表單欄位輸入與驗證檢查
  * 頁面間的的導航切換'

### 等級 

入門

### 需求

* 具備 C# 程式語言開發經驗
* 具有 委派 Delegate 與 事件 Event 的知識與使用技術
* 使用過 LINQ, 非同步開發方式
* 孰悉 HTML, CSS 語言用法
* 原則上不需要了解與使用到 JavaScript

### 想要參加

此課程為 Skill Tree 專屬課程，想要參加者，請隨時注意 Skill Tree 的最新公告

參考網址 : [https://skilltree.my/Events/2021/12/11/Blazor-for-Beginners](https://skilltree.my/Events/2021/12/11/Blazor-for-Beginners)

## 軟體工程師應知道的作業系統概念

等級 : 入門

## 精通 async 與 awiat

等級 : 中階

## .NET 執行緒集區

等級 : 中階

## Xamarin.Forms 跨平台 App 開發與設計

等級 : 入門

## MAUI 跨平台 App 開發與設計

等級 : 入門

## .NET 多執行緒與執行緒同步程式設計

等級 : 高階

## Entity Framework Core 動手實作

等級 : 入門

## Blazor Server 動手實作

等級 : 入門

## Blazor Web API 認證、授權 動手練習

等級 : 入門

## Blazor Xamarin 超全端程式設計

等級 : 入門

## C# 委派與事件

等級 : 入門

## SOLID 原則

等級 : 入門

## 編譯器對 C# 語法糖 做了什麼事情？

等級 : 入門

## C#  與 .NET 程式設計：核心概念篇

等級 : 入門

## 相依性注入的觀念與開發技巧

等級 : 入門

