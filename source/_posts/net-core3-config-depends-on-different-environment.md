---
title: 在 .Net Core Console 依環境使用不同設定檔
date: 2019-12-26 22:40:15
tags: ['dotnet', 'core-3', 'config']
categories:
---

上次有提到如何在 Console 使用 .Net Core 的 Configuration 自動對應到 class。

今天遇到需要依環境使用不同的環境設定檔，筆記一下。

## 環境

Visual Studio 2019
.Net Core 3.1

## 流程

1. 設定 Debug 執行的環境變數
    1. 在 Solution Explorer 要執行的專案上按右鍵選 Properties
    1. 選 Debug 分頁
    1. 在 Environment variables 加入 Name: `HOSTING_ENVIRONMENT` Value: `Development`
2. 在 `appsettings.json` 同個位置的地方加入新的設定檔 `appsettings.Development.json` ，這份設定檔可以加入該環境需要的不同參數。
3. 程式碼的部分加入如下

```csharp
var environmentName = Environment.GetEnvironmentVariable("HOSTING_ENVIRONMENT");
var builder = new ConfigurationBuilder()
  .SetBasePath(Directory.GetCurrentDirectory())
  .AddJsonFile("appsettings.json")
  .AddJsonFile($"appsettings.{environmentName}.json", true); // 多加這行
```

在準備要拿來對應的 class 加入該參數 Property ，在自動對應時就會依環境把值給賦與進去了，環境不對時就是 default 值了。

至於怎麼對應，class 怎麼寫就參考上一篇吧! {% post_link net-core3-config-to-class %}

## References

[SO](https://stackoverflow.com/questions/36943484/using-asp-net-cores-configurationbuilder-in-a-test-project)