---
title: 在 .Net Core Console 自動將設定檔對應到類別上
date: 2019-12-05 21:38:19
tags: ['dotnet', 'core-3', 'config']
categories:
---

今天在工作上需要在 .Net Core 3 Console app 上將 appsettings.json 的設定檔轉到類別物件上，但是找到的資訊都是在 Asp.net Core 3 上的範例，後來就自行嘗試在 Console 上用一樣的方式試看看，好在該套件並不一定要跟 Asp.net core 3 一起使用。

順便嘗試不同的設定檔格式轉換，官方文件上也有提供各種不同的 provider 供解析轉換。

|Provider|Provides configuration from…|
|:-------|----------------------------|
|Azure Key Vault Configuration Provider (Security topics) | Azure Key Vault|
|Azure App Configuration Provider (Azure documentation) | Azure App Configuration|
|Command-line Configuration Provider |Command-line parameters|
|Custom configuration provider | Custom source|
|Environment Variables Configuration Provider | Environment variables|
|File Configuration Provider | Files (INI, JSON, XML)|
|Key-per-file Configuration Provider | Directory files|
|Memory Configuration Provider | In-memory collections|
|User secrets (Secret Manager) (Security topics) | File in the user profile directory|

## 環境

* Windows 10 Pro version 2004 build 19033.1
* VisualStudio 2019
* dotnet sdk 2.2.402
* dotnet sdk 3.0.101

## Json

### 標準 Json 物件

#### 套件

```csharp
<PackageReference Include="Microsoft.Extensions.Configuration" Version="3.1.0" />
<PackageReference Include="Microsoft.Extensions.Configuration.Json" Version="3.1.0" />
<PackageReference Include="Microsoft.Extensions.Options.ConfigurationExtensions" Version="3.1.0" />
```

#### 設定檔

```json
{
  "section0": {
    "key0": "section0-key0-value",
    "key1": "section0-key1-value"
  },
  "section1": {
    "key0": "section1-key0-value",
    "key1": "section1-key1-value"
  }
}
```

#### 類別

```csharp
public class RootConfig
{
    public SectionConfig Section0 { get; set; }
    public SectionConfig Section1 { get; set; }
}

public class SectionConfig
{
    public string Key0 { get; set; }
    public string Key1 { get; set; }
}
```

#### 程式

```csharp
private static void Json_AppSettings()
{
    //Keys are case-insensitive
    var config = new ConfigurationBuilder().AddJsonFile("rootconfig.json").Build();
    var rootConfig = config.Get<RootConfig>();

    Console.WriteLine(nameof(Json_AppSettings));
    Console.WriteLine(rootConfig.Section0.Key0);
    Console.WriteLine(rootConfig.Section0.Key1);
    Console.WriteLine(rootConfig.Section1.Key0);
    Console.WriteLine(rootConfig.Section1.Key1);
    Console.WriteLine();
}
```

#### 結果

{% asset_img json-config-result.png json-config-result %}

### Json 物件使用冒號分隔

#### 設定檔

```json
{
  "ReportSheet:OutPutPath": "/Export/",
  "ReportSheet:ProjectsPackagesList": "ProjectsPkgsList",
  "ReportSheet:PackageInfoList": "PackageInfoList"
}
```

#### 類別

```csharp
public class ReportConfig
{
    public ReportSectionConfig ReportSheet { get; set; }
}
public class ReportSectionConfig
{
    public string OutPutPath { get; set; }
    public string ProjectsPackagesList { get; set; }
    public string PackageInfoList { get; set; }
}
```

#### 程式

```csharp
private static void Json_AppSettings_Colon()
{
    var config = new ConfigurationBuilder().AddJsonFile("appsettings.json").Build();
    var reportConfig = config.Get<ReportConfig>();

    Console.WriteLine(nameof(Json_AppSettings_Colon));
    Console.WriteLine(reportConfig.ReportSheet.OutPutPath);
    Console.WriteLine(reportConfig.ReportSheet.PackageInfoList);
    Console.WriteLine(reportConfig.ReportSheet.ProjectsPackagesList);
    Console.WriteLine();
}
```

#### 結果

{% asset_img json-colon-config-result.png json-colon-config-result %}

### 使用參數式

這邊需要多安裝一個 `Microsoft.Extensions.Configuration.CommandLine` 套件

#### 套件

```csharp
<PackageReference Include="Microsoft.Extensions.Configuration.CommandLine" Version="3.1.0" />
```

#### 類別

```csharp
public class CommandlineArgumentsConfig
{
    public string User { get; set; }
    public string Password { get; set; }
    public string Address { get; set; }
    public string Mail { get; set; }
    public string Comment { get; set; }
}
```

#### 程式

```csharp
private static void Commandline_arguments()
{
    var args = new string[] { "--user", "ghost", "--password=123456", "address=tw", "/mail", "ghost@everwhere.com", "/comment=ok" };

    //Keys are case-insensitive
    var config = new ConfigurationBuilder().AddCommandLine(args).Build();
    var rootConfig = config.Get<CommandlineArgumentsConfig>();

    Console.WriteLine(nameof(Commandline_arguments));
    Console.WriteLine(rootConfig.User);
    Console.WriteLine(rootConfig.Password);
    Console.WriteLine(rootConfig.Address);
    Console.WriteLine(rootConfig.Mail);
    Console.WriteLine(rootConfig.Comment);
    Console.WriteLine();
}
```

#### 結果

{% asset_img commandline-config.png commandline-config %}

## References

[此次程式碼](https://github.com/GhostTW/demos/tree/master/net-core3-config-to-class/ConfigurationMapping)

[docs.microsoft](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/configuration/?view=aspnetcore-3.1)