---
title: CORS在做些什麼
date: 2020-11-07 16:05:37
tags:
  - .NET
  - CORS
  - WEB API
---

## 什麼是 CORS(Cross-Origin Resource Sharing)

中文譯作「跨來源資源共用」，指請求的資源並**非相同來源**，比方引用外站的圖片、Javascript 等資源。

### 何謂同源(Same Orign Policy)

在釐清 CORS 在做什麼之前，首先我們要知道什麼樣的資源稱為同站資源？同站資源亦作為同源，當存取的資源**相同網路協定(Protocol)**、**相同網域(Domain)**、**相同連接阜(Port)**，就會視為同源

> 舉例
>
> > 相同網路協定(Protocol)
> > https://www.Sample.com.tw 與 http://www.Sample.com.tw
> > https 與 http 協定不同，會被視為不同源
>
> > 相同網域(Domain)
> > https://www.SampleA.com.tw 與 https://www.SampleB.com.tw
> > 兩個不同網域，會被視為不同源
> > https://aaa.Sample.com.tw 與 https://bbb.Sample.com.tw
> > 兩個子 subdomain 不同，會被視為不同源
>
> > 相同連接阜(Port)
> > https://www.Sample.com.tw:1234 與 https://www.Sample.com.tw:5678
> > 兩個連接阜不同，會被視為不同源

<br>
### 不同源會怎麼樣?

當網域 A 取用網域 B 的資源時，就會無法順利資源，錯誤訊息會像是下方

> Access to XMLHttpRequest at 'https://DomainA/api/v1/Test' from origin 'https://DomainB' has been blocked by CORS policy: No 'Access-Control-Allow-Origin' header is present on the requested resource.

但為什麼我可以用其它網域的 Image、Javascript、CSS 等資源？有些資源卻無法使用？決定這個來源存取的標準，是請求為「簡單請求」，當只要是簡單請求，就可以順利獲得資源
<br><br>

### 簡單請求的標準

1. 只能是 HTTP GET, POST or HEAD 方法
2. 自訂的 request header 只能是 Accept、Accept-Language、Content-Language 或 Content-Type（值只能是 application/x-www-form-urlencoded、multipart/form-data 或 text/plain）

僅要符合上述標準，即便是不同源，仍可以正常請求資源

_**符合簡單請求範例**_

```html{.line-numbers}
Post /data/ Host: outsideSite.com Origin: https://home.com Content-Type:
application/x-www-form-urlencoded
```

_**不符合簡單請求範例**_

```html{.line-numbers}
Post /data/ Host: outsideSite.com Origin: https://home.com Content-Type:
application/json
```

> 因為 Content-Type 不符合

<br><br>

### Preflight Requet (預檢)

當不符合簡單請求時，瀏覽器會正常 Request 前先對請求的資源發一個名為 **Preflight Request** ，目的要檢查這個資源 (Server) 是否允許目前要進行的真實請求，這個 Request 會對 Server 進行 Http Options 的動作， Server 必須對這個 Preflight Request 進行確認，判斷這個來源是否可以提供存取、使用什麼樣的資料等，接著依照真實的需求回覆，Response Header 可能包含**允許請求的來源(Access-Control-Allow-Origin)**、**允許的請求方式(Access-Control-Allow-Method)**、**允許請求發出的 Header 內容 (Access-Control-Allow-Headers)** 。

_**瀏覽器對 Server 進行 Preflight Request 範例**_

```html{.line-numbers}
OPTIONS /data/ Host: outsideSite.com Origin: https://home.com
Access-Control-Request-Method: POST Access-Control-Request-Headers: X-MY-HEADER,
Content-Type
```

_**Server 對瀏覽器 Preflight Response 範例**_

```html{.line-numbers}
Access-Control-Allow-Origin: * Access-Control-Allow-Method: POST
Access-Control-Allow-Headers: X-MY-HEADER, Content-Type
```

> `Access-Control-Allow-Origin: *` 表示允許所有來源
> 這部分允許的內容必須符合 Preflight Request 的 Origin

一旦 Preflight Response 與 Request 內的項目不符，就會被瀏覽器阻擋下來的動作。

### 參考資料

https://medium.com/@des75421/cors-%E8%B7%A8%E4%BE%86%E6%BA%90%E8%B3%87%E6%BA%90%E5%85%B1%E7%94%A8cors-191d4bfc4735
