---
title: CORS在.Net Web API實例
date: 2020-12-13 14:53:12
tags:
  - .NET
  - WEB API
  - CORS
  - GG
categories:
  - tech
  - dot-net
---

## 前言

經過[上一篇](https://chuangmaster.github.io/tech/CORS%E5%9C%A8%E5%81%9A%E4%BA%9B%E4%BB%80%E9%BA%BC/)了解到了 CORS 運作原理，為了感受上一篇運作在真實環境的情形，因此有了這一篇文章的出現，這一篇文章紀錄我目前在 .NET 專案環境在遇到 CORS 的情境與我使用的解決方案。

---

## 情境

假設外部資源是 API ，我的網站直接透過 jquery 對 API 進行請求並取得資源。

## 環境設置

為了實驗 CORS 的情境，我使用了 Visual Stuido Express IIS Server，透過內建網頁伺服器建立兩個專案，一個為提供服務的 Web API 專案，另外一個則是呼叫 Web API 的 MVC 專案。
如果不願意自己建立，可以參考我建立的[沙盒](https://github.com/chuangmaster/CORS_SandBox)，包含了兩個專案，編譯執行後，確認 port 就可以展示情境。

## 簡單請求 ( Simple Request )

依照需求給予回應，這邊僅允許不同源來存取這個端口，但不允許 ContentType: application/json 所以瀏覽器在預檢請求會顯示錯誤。

## 預檢請求 ( Preflight Request )

為了看一下瀏覽器阻擋 CORS 的錯誤，我製造了一個在不同源的狀況下，進行請求

- 前端執行的 ajax

```javascript{.line-numbers}
var data = {
  value: 123,
};
$.ajax({
  url: 'https://localhost:44351/api/v1/simpleRequest',
  method: 'post',
  data: data,
  contentType: 'application/json',
  success: function (result) {
    //alert(result);
  },
  error: function () {},
});
```

- 資源 Request 接口

```C#{.line-numbers}
[HttpPost]
[Route("~/api/v1/normalRequest/error")]
public HttpResponseMessage NormalRequest_error() {
  var resp = Request.CreateResponse(HttpStatusCode.OK, "OK");
    return resp;
}
```

從上面的程式碼，可以發現我的請求 **ContentType** 為 **application/json**，因此不符合簡單請求( Simple Request )，所以就會在真實請求( Actual Request ) 前進行預檢請求( Preflight Request )，而我們目前的 API 僅提供 Http POST 的服務，因此預檢得到的錯誤狀態是 405 error

![405錯誤](https://i.imgur.com/j6vVr2A.jpg)

接下來我們提供遺漏的該組 API

- 新增處理 OPTIONS 接口

```C#{.line-numbers}
[HttpOptions]
[Route("~/api/v1/normalRequest/error")]
public HttpResponseMessage NormalRequest_error_option()
{
    var resp = Request.CreateResponse(HttpStatusCode.OK, "OK");
    return resp;
}
```

再來就會看到 Http Options 雖然成功了，但是在 Actual Request 卻失敗了，原因就是此資源不提供不同源的存取
![error in actual request 1 ](https://i.imgur.com/XVctzcO.jpg)

- 加上伺服器對 CORS 回應

為了讓預檢能夠對不同來源進行存取，所以在 Header 加入了 `Access-Control-Allow-Origin` 並設定允許任何不同源位置 (`*`)

```C#{.line-numbers}
[HttpOptions]
[Route("~/api/v1/normalRequest/error")]
public HttpResponseMessage NormalRequest_error_option()
{
    var resp = Request.CreateResponse(HttpStatusCode.OK, "OK");
    resp.Content.Headers.Add("Access-Control-Allow-Origin", "*");
    return resp;
}
```

其次因為發送的 Request Content-Type 為 **application/json**，與簡單請求所使用的 Content-Type 不同，所以這邊我也一併將 `Content-Type`也給加上，如果不加上的話，雖然 OPTIONS 通過了，但 Actual Request 則會出現下方圖的錯誤
![Access-Control-Allow-Origin error](https://i.imgur.com/5wCXrYE.jpg)

```C#{.line-numbers}
[HttpOptions]
[Route("~/api/v1/normalRequest/error")]
public HttpResponseMessage NormalRequest_error_option()
{
    var resp = Request.CreateResponse(HttpStatusCode.OK, "OK");
    resp.Content.Headers.Add("Access-Control-Allow-Origin", "*");
    resp.Content.Headers.Add("Access-Control-Allow-Headers", "Content-Type");
    return resp;
}
```

最後就可以順利完成 CORS 的流程

## 其它作法

從上一節可以知道在 .NET Web API 處理 CORS 的流程，但可以發現一個問題，如果我今天有 10 個 API 接口，我必須同時有 10 個處理預檢的 Action methods，這樣顯得相當的麻煩，有沒有其它作法呢？

### Global.asax Event

我們在 Global.asax 加上事件 `Application_BeginRequest` 來取得當下的 Request Method ，並在 Response Header 加上對應的內容就可以。

```C#{.line-numbers}
protected void Application_BeginRequest()
{
    if (Request.HttpMethod == "OPTIONS")
    {
        Response.StatusCode = (int)HttpStatusCode.OK;
        Response.AppendHeader("Access-Control-Allow-Origin", Request.Headers.GetValues("Origin")[0]);
        Response.AddHeader("Access-Control-Allow-Headers", "Content-Type, Accept");
        Response.AddHeader("Access-Control-Allow-Methods", "GET, POST, PUT, DELETE");
        Response.End();
    }
}
```

### 使用 Microsoft.AspNet.WebApi.Cors 套件

雖然上述加入了事件可以完全解決整個站的 CORS 問題，但是又涵蓋太廣，可能有些資源我們想允許非同源；有些資源不允許非同源，但上述的做法就要額外加入很多判斷，才能做到這件事情。

因此我們可以使用官方提供的套件來解決這個問題

1. 在 [Nuget](https://www.nuget.org/packages/Microsoft.AspNet.WebApi.Cors) 進行安裝套件
2. 在 _App_Start/WebApiConfig_ 啟用 CORS 支援

```C#{.line-numbers}
public static void Register(HttpConfiguration config)
{
    // Web API 設定和服務
    config.EnableCors();
    ....
}
```

### 使用 EnableCors Attribute

做完上述步驟，我們就已經做完第一步的設定，接下來就針對我們要做 CORS 服務的 Action methods 做屬性設定，三個參數分別為**Origin**、**Headers**、**Methods**，可以使用 **\*** 表示全部，也允許多個項目，項目與項目之間要使用 **,** 來做區隔，接下來就可以正常運作了。

```C#{.line-numbers}
[HttpPost]
[EnableCors("https://localhost:44354", "Content-Type", "POST")]
[Route("~/api/v1/normalRequest/success")]
public HttpResponseMessage NormalRequest_success(NormalRequestParameter parameter)
{
    var resp = Request.CreateResponse(HttpStatusCode.OK, "OK");
    return resp;
}
```

### 使用 自訂 Attribute

如果設定的內容複雜時，有時候在 Attribute 上設定，會讓整個長度太長，而且不好管理，因此可以繼承 Attribute 並且實作 ICorsPolicyProvider ，最後掛上 Action Method 上頭。 GetCorsPolicyAsync 會在啟用該服務的時候被叫用，取得就是 CORS 的設定，也就是 `CorsPolicy` ，這樣就可以完成第一節複雜的內容了。

```C#{.line-numbers}
public class CorsHandleAttribute : Attribute, ICorsPolicyProvider
{
    private CorsPolicy _Policy;
    public CorsHandleAttribute()
    {
        _Policy = new CorsPolicy()
        {
            Methods = { "POST" },
            AllowAnyHeader = true
        };
        _Policy.Origins.Add("https://localhost:44354");

    }
    public Task<CorsPolicy> GetCorsPolicyAsync(HttpRequestMessage request, CancellationToken cancellationToken)
    {
        return Task.FromResult(_Policy);
    }
}

[HttpPost]
[CorsHandle]
[Route("~/api/v1/normalRequest/success")]
public HttpResponseMessage NormalRequest_success(NormalRequestParameter parameter)
{
    var resp = Request.CreateResponse(HttpStatusCode.OK, "OK");
    return resp;
}
```

---

## 參考

- [Working with the ASP.NET Global.asax file](https://www.techrepublic.com/article/working-with-the-aspnet-globalasax-file/)
- [Handling CORS Preflight requests to ASP.NET MVC actions](https://stackoverflow.com/questions/13624386/handling-cors-preflight-requests-to-asp-net-mvc-actions)
- [Enable cross-origin requests in ASP.NET Web API 2](https://docs.microsoft.com/en-us/aspnet/web-api/overview/security/enabling-cross-origin-requests-in-web-api)
