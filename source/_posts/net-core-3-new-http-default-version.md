---
title: .Net Core 3 HttpRequestMessage version 預設變更
date: 2019-12-03 22:56:38
tags: dotnet, core-3
categories: dotnet-core-3
---

.Net Core 3.0 這次將 HttpRequestMessage 預設版本降回 1.1，但是在 .Net Core 2.1, .Net Core  2.2 時預設版本被調升為 2.0，而 2.1 之前則是 1.1。

| .net core version| Http version|
|:-----------------|:------------|
| .net core 1+  | 1.1 |
| .net core 2.1 | 2.0 |
| .net core 2.2 | 2.0 |
| .net core 3.0 | 1.1 |

這次會使用不同 .net core 版本 HttpClient 打 `https://http2.pro/` 驗證 response 的版號，並測試在 .net core 2.2 使用 http2 的方式。

`https://http2.pro/` 這個網站可以幫忙測試目標 api 的 http2 狀態，也可以自行發 request 過去驗證。

## 環境

* Windows 10 Pro version 2004 build 19033.1
* VisualStudio 2019
* dotnet sdk 2.2.402
* dotnet sdk 3.0.101

## .Net Core 2.2 測試

準備一個 .Net Core 2.2 版的 Console proj

準備三段程式測試，分別為
Test_Default : 驗證預設行為 Get 的版本。
Test_Default_HttpRequestMessage: 驗證使用 HttpRequestMessage 的版本。
Test_Set_HttpRequestMessage_HttpVersion20: 驗證使用 HttpRequestMessage 並使用 `System.Net.HttpVersion.Version20` 的版本。

測試完的結果會發現，response 怎麼跑都是 1.1 版。

```C#
static void Main(string[] args)
{
    Console.WriteLine($"{nameof(http_client_core22)}");
    Test_Default().Wait();
    Test_Default_HttpRequestMessage().Wait();
    Test_Set_HttpRequestMessage_HttpVersion20().Wait();
}

private static async Task Test_Default()
{
    Console.WriteLine($"{nameof(Test_Default)}");
    using (var client = new HttpClient())
    {
        var result = await client.GetAsync("https://http2.pro/api/v1");
        Console.WriteLine($"receive response with http protocol {result.Version}");
    }
    Console.WriteLine();
}

private static async Task Test_Default_HttpRequestMessage()
{
    Console.WriteLine($"{nameof(Test_Default_HttpRequestMessage)}");
    using (var client = new HttpClient())
    {
        var httpRequestMessage = new HttpRequestMessage(HttpMethod.Get, "https://http2.pro/api/v1");
        Console.WriteLine($"{nameof(httpRequestMessage)} version {httpRequestMessage.Version}");
        var result = await client.SendAsync(httpRequestMessage);
        Console.WriteLine($"receive response with http protocol {result.Version}");
    }
    Console.WriteLine();
}

private static async Task Test_Set_HttpRequestMessage_HttpVersion20()
{
    Console.WriteLine($"{nameof(Test_Set_HttpRequestMessage_HttpVersion20)}");
    using (var client = new HttpClient())
    {
        var httpRequestMessage = new HttpRequestMessage(HttpMethod.Get, "https://http2.pro/api/v1") { Version = System.Net.HttpVersion.Version20 };
        Console.WriteLine($"{nameof(httpRequestMessage)} version {httpRequestMessage.Version}");
        var result = await client.SendAsync(httpRequestMessage);
        Console.WriteLine($"receive response with http protocol {result.Version}");
    }
    Console.WriteLine();
}
```

結果

{% asset_img net-core-22-httprequestmessage-result.png net-core-22-httprequestmessage-result %}

## .Net Core 3.0 測試

準備三段程式測試，分別為
Test_Default : 驗證預設行為 Get 的版本。
Test_Default_HttpRequestMessage: 驗證使用 HttpRequestMessage 的版本，。
Test_Set_HttpRequestMessage_Version20: 驗證使用 HttpRequestMessage 並使用 HTTP/2 的版本。

我們在每段裡面都使用 .net core 3.0 新支援 `DefaultRequestVersion` 的 api 查看預設版本。

測試完的結果會發現，有設定 HTTP/2 的 response 就會是 2.0 版了。

```C#
static void Main(string[] args)
{
    Test_Default().Wait();
    Test_Default_HttpRequestMessage().Wait();
    Test_Set_HttpRequestMessage_Version20().Wait();
}

private static async Task Test_Default()
{
    Console.WriteLine($"{nameof(Test_Default)}");
    var client = new HttpClient();
    var result = await client.GetAsync("https://http2.pro/api/v1");
    Console.WriteLine($"receive response with http protocol {result.Version}");
    Console.WriteLine();
}

private static async Task Test_Default_HttpRequestMessage()
{
    Console.WriteLine($"{nameof(Test_Default_HttpRequestMessage)}");
    var client = new HttpClient();
    Console.WriteLine($"{nameof(client)} version {client.DefaultRequestVersion}");
    var httpRequestMessage = new HttpRequestMessage(HttpMethod.Get, "https://http2.pro/api/v1");
    Console.WriteLine($"{nameof(httpRequestMessage)} version {httpRequestMessage.Version}");
    var result = await client.SendAsync(httpRequestMessage);
    Console.WriteLine($"receive response with http protocol {result.Version}");
    Console.WriteLine();
}

private static async Task Test_Set_HttpRequestMessage_Version20()
{
    Console.WriteLine($"{nameof(Test_Set_HttpRequestMessage_Version20)}");
    var client = new HttpClient();
    Console.WriteLine($"{nameof(client)} version {client.DefaultRequestVersion}");
    var httpRequestMessage = new HttpRequestMessage(HttpMethod.Get, "https://http2.pro/api/v1") { Version = new Version(2, 0) };
    Console.WriteLine($"{nameof(httpRequestMessage)} version {httpRequestMessage.Version}");
    var result = await client.SendAsync(httpRequestMessage);
    Console.WriteLine($"receive response with http protocol {result.Version}");
    Console.WriteLine();
}
```

結果

{% asset_img net-core-3-httprequestmessage-result.png net-core-3-httprequestmessage-result %}

## .Net Core 2.2 開啟 HTTP/2 測試

使用 .Net Core 2.2 相同的程式碼，在建立 client 前加入 `AppContext.SetSwitch("System.Net.Http.UseSocketsHttpHandler", false);` 就可以讓 .NetCore 2.2 支援 HTTP/2。

這段是將系統環境變數 `DOTNET_SYSTEM_NET_HTTP_USESOCKETSHTTPHANDLE` 設為 0 或 false，讓 .Net Core 用回舊的 `HttpClientHandler`，也可以使用設定檔設定這項。

.Net Core 2.1 Preview 2 有說明他們使用新的 [SocketsHttpHandler](https://github.com/dotnet/corefx/blob/master/src/System.Net.Http/src/System/Net/Http/SocketsHttpHandler/SocketsHttpHandler.cs) 作為 HttpClient 的預設，最大的差異是在效能上、與平台相依脫勾、跨平台的一致性行為。

```C#
static void Main(string[] args)
{
    Console.WriteLine($"{nameof(http_client_core22)}");
    AppContext.SetSwitch("System.Net.Http.UseSocketsHttpHandler", false); // turn http2 on !
    Test_Default().Wait();
    Test_Default_HttpRequestMessage().Wait();
    Test_Set_HttpRequestMessage_HttpVersion20().Wait();
}
```

結果

{% asset_img net-core-22-support-http2-httprequestmessage-result.png net-core-22-support-http2-httprequestmessage-result %}

## References

[本次程式碼](https://github.com/GhostTW/demos/tree/master/httpclient-http2-core-2-3)

[compatibility/2.2-3.0#networking](https://docs.microsoft.com/en-us/dotnet/core/compatibility/2.2-3.0#networking)

[use-http-2-with-httpclient-in-net](https://stackoverflow.com/questions/53764083/use-http-2-with-httpclient-in-net)

[announcing-net-core-2-1-preview-2](https://devblogs.microsoft.com/dotnet/announcing-net-core-2-1-preview-2/)