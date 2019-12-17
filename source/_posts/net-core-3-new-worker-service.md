---
title: .Net Core 3 新功能 Worker Service
date: 2019-12-02 22:38:57
tags: ['dotnet', 'core-3']
categories: dotnet-core-3
---

## .Net Core Worker Service

這個 .Net Core 3 的新功能將 windows service 功能包裝起來，方便開發 windows service ，更將 linux service 包裝起來，可以維護同個 source code 佈署不同環境，但本文目前先實作 windows 的部分。

### 環境

* VS 2019
* Windows 10
* .net sdk 3.0.101

### 建立專案

1. 開新資料夾並在下面執行 `dotnet new worker`，資料夾下自動建立一個與資料夾相同名稱的專案。
2. 安裝套件，Windows 可以只安裝 `WindowsServices`，linux 可以只安裝 `Systemd`

```powershell
dotnet add package Microsoft.Extensions.Hosting --version 3.0.1
dotnet add package Microsoft.Extensions.Hosting.Systemd --version 3.0.1
dotnet add package Microsoft.Extensions.Hosting.WindowsServices --version 3.0.1
```

3. 在 `program.cs` 下加入以下兩行，在不同 OS 會啟用但不互相影響。

```csharp
public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .UseSystemd() // for linux
        .UseWindowsService() // for windows
        .ConfigureServices((hostContext, services) =>
        {
            services.AddHostedService<Worker>();
        });
```

### 建立 Windows Service

1. Build `dotnet publish -o output`
1. 註冊服務 `sc.exe create servicetest binPath=C:\Users\ghost\Projects\Github\demos\net-core-3-services\output\NetCore3Service.exe`
1. 啟動服務 `sc.exe start servicetest`
1. 在 Services 可以觀察到服務已啟動

{% asset_img bg-service-cmd.png bg-service-cmd %}

{% asset_img bg-service.png bg-service %}

### 參考

[project](https://github.com/GhostTW/demos/tree/master/net-core-3-services)
