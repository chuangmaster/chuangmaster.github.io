---
title: Cookie SameSite 讓 Session失效
tags:
  - SameSite
  - Cookie
  - Chrome
  - Session
categories:
  - Tech
date: 2020-12-16 10:59:46
---


## 前言
在工作的時候遇到一個物流專案，選擇物流會導頁到廠商的網站，在導頁回原本站上，但這個時候卻發現 Session 竟然失效了。花了大半天 Debug ，後來發現原來是 [Chrome 對 Cookie 政策修改](https://blog.chromium.org/2020/02/samesite-cookie-changes-in-february.html)了，因此 ASP.NET_SessionId 的 Cookie 消失了，造成 Session 失效。
此篇是將網路上的資源集合做下的筆記，希望在未來不要再踩到相同的雷。

## Chrome 做了什麼
Chrome 對為了安全性對 Cookie 的 SameSite 屬性改動了預設值。

>With the stable release of Chrome 80 this month, Chrome will begin enforcing a new secure-by-default cookie classification system, treating cookies that have no declared SameSite value as  SameSite=Lax cookies. Only cookies set as SameSite=None; Secure will be available in third-party contexts, provided they are being accessed from secure connections.

Chrome 會將沒有對 Cookie SameSite 屬性的 Cookie 預設加入 `SameSite=Lax` 的屬性，這會致使**不相同的網站**，無法再透過 Get 以外的請求取得到 Cookie ，因此對外站進行 POST 請求；
如果要避免這問題，就必須將 SameSite 屬性設為 `SameSite=None; secure`，並且透過 Https 進行資料傳輸，就能避免 Cookie 在不同網站能被存取且又能安全傳遞資料。如果沒加上 `secure` ，一樣僅能透過 Https 運作。

## .NET 專案該如何因應
根據上一節，我們必須告訴專案要做上述的設定，而在ASP.NET中，不同版本有不同的解決方法。
### ASP .NET 4.7.2 以上 / .NET Core 2.1 以上的版本
會預設將 Cookie 設為  `SameSite=None`，如果專案有做 FormsAuth 登入驗證 SessionState Cookie 會被設為 `SameSite=Lax`。

如果想要自訂也可以考慮以下的作法
```xml
<sessionState mode="StateServer" cookieSameSite="None" cookieless="false" timeout="20" />
```
### ASP .NET 4.7.2以前
對 IIS 進行 rewrite ， Web Config 加上以下內容

```xml
<rewrite>
  <outboundRules>
    <rule name="SessionCookieAddNoneHeader">
      <match serverVariable="RESPONSE_Set-Cookie" pattern="(.*ASP.NET_SessionId.*)" />
      <!-- Use this regex if your OS/framework/app adds SameSite=Lax automatically to the end of the cookie -->
      <!-- <match serverVariable="RESPONSE_Set-Cookie" pattern="((.*)(ASP.NET_SessionId)(=.*))(?=SameSite)" /> -->
      <action type="Rewrite" value="{R:1}; SameSite=None" />
    </rule>
  </outboundRules>
</rewrite>
```

### 遇不支援的瀏覽器
為了避免有瀏覽器不支援 `SameSite=None`，預設忽略或是更改成 **Strict** (只允許同站存取)，更完整的 rewrite 做法可以換成以下
```xml 
<rewrite>
  <outboundRules>
    <preConditions>
      <!-- Checks User Agent to identify browsers incompatible with SameSite=None -->
      <preCondition name="IncompatibleWithSameSiteNone" logicalGrouping="MatchAny">
        <add input="{HTTP_USER_AGENT}" pattern="(CPU iPhone OS 12)|(iPad; CPU OS 12)" />
        <add input="{HTTP_USER_AGENT}" pattern="(Chrome/5)|(Chrome/6)" />
        <add input="{HTTP_USER_AGENT}" pattern="( OS X 10_14).*(Version/).*((Safari)|(KHTML, like Gecko)$)" />
      </preCondition>
    </preConditions>

    <!-- Adds or changes SameSite to None for the session cookie -->
    <!-- Note that secure header is also required by Chrome and should not be added here -->
    <rule name="SessionCookieAddNoneHeader">
      <match serverVariable="RESPONSE_Set-Cookie" pattern="(.*ASP.NET_SessionId.*)" />
      <!-- Use this regex if your OS/framework/app adds SameSite=Lax automatically to the end of the cookie -->
      <!-- <match serverVariable="RESPONSE_Set-Cookie" pattern="((.*)(ASP.NET_SessionId)(=.*))(?=SameSite)" /> -->
      <action type="Rewrite" value="{R:1}; SameSite=None" />
    </rule>

    <!-- Removes SameSite=None header from all cookies, for most incompatible browsers -->
    <rule name="CookieRemoveSameSiteNone" preCondition="IncompatibleWithSameSiteNone">
      <match serverVariable="RESPONSE_Set-Cookie" pattern="(.*)(SameSite=None)" />
      <action type="Rewrite" value="{R:1}" />
    </rule>
  </outboundRules>
</rewrite>
```



## 參考
* [Chrome's SameSite Cookie Changes are Breaking Apps](https://www.coderfrontline.com/chromes-samesite-cookie-changes-are-breaking-apps/)

* [Chrome SameSite Attribute 簡介](https://medium.com/it-digital-%E4%BA%92%E8%81%AF%E7%B6%B2/%E4%BB%80%E9%BA%BC-samesite-cookies-policy-%E6%9B%B4%E6%96%B0%E4%BA%86-2b317e6cf6bb)