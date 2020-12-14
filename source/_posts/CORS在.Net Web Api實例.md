---
title: CORS在.Net Web API實例
date: 2020-12-13 14:53:12
tags:
  - .NET
  - WEB API
  - CORS
categories:
  - techblog
---

## 前言

經過[上一篇](https://chuangmaster.github.io/uncategorized/CORS%E5%9C%A8%E5%81%9A%E4%BA%9B%E4%BB%80%E9%BA%BC)了解到了 CORS 運作原理，為了感受上一篇運作在真實環境的情形，因此有了這一篇文章的出現，這一篇文章紀錄我目前在 .NET 專案環境在遇到 CORS 的情境與我使用的解決方案。

---

## 情境

假設外部資源是 API ，我的網站直接透過 jquery 對 API 進行請求並取得資源。

## 環境設置

為了實驗 CORS 的情境，我使用了 Visual Stuido Express IIS Server，透過內建網頁伺服器建立兩個專案，一個為提供服務的 Web API 專案，另外一個則是呼叫 Web API 的 MVC 專案。
如果不願意自己建立，可以參考我建立的[沙盒](https://github.com/chuangmaster/CORS_SandBox)，包含了兩個專案，編譯執行後，確認 port 就可以展示情境。

## 簡單請求 ( Simple Request )

依照需求給予回應，這邊僅允許不同源來存取這個端口，但不允許 ContentType: application/json 所以瀏覽器在預檢請求會顯示錯誤。

## 失敗的不同源請求

為了看一下瀏覽器阻擋 CORS 的錯誤，我製造了一個在不同源的狀況下，進行請求

_**前端請求**_

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

_**資源正常 Request 接口**_

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

接下來我們提供該組服務

_**新增處理 OPTIONS 接口**_

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

_**補上相關回應**_

```C#{.line-numbers}
[HttpOptions]
[Route("~/api/v1/normalRequest/error")]
public HttpResponseMessage NormalRequest_error_option()
{
    var resp = Request.CreateResponse(HttpStatusCode.OK, "OK");
    resp.Content.Headers.Add("Access-Control-Allow-Origin", "*");
    resp.Content.Headers.Add("Access-Control-Allow-Headers", "*");
    return resp;
}
```

## 預檢請求 ( Preflight Request )

## CORS 套件
