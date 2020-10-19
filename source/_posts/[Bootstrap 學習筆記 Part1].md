---
title: [Bootstrap 學習筆記 Part1]
date: 2019-04-03 12:00:00
categories:
- Boostrap Learning
tags:
- Boostrap4
- HTML
notebook: L14.Bootstrap
---

# Bootstrap 學習筆記 Part1

## BoxSizing

原本在 HTML 預設的 CSS，當設定一個物件的長寬度時，會不包含 padding 與 border 的大小
但在 boostrap 中，BoxSizing 被設為 **border-box**

```CSS {.line-numbers}
div{
    box-sizing: border-box;
}
```

舉例來說，下方為一個非 boostrap 環境下的 div

```CSS {.line-numbers}
div{
    width: 300px;
    height: 300px;
    padding: 50px;
    border: 10px solid green;
    margin: 50px;
}
```

真實的該 div 的寬度可能為
$RealWidth = width + padding + border = 300 + 50 * 2 + 10 *2 = 420 px$

而在 **Bootstrap** 的環境真實的 div 寬度就為您給的寬度(被包含 Padding、border)
假設今天設寬度為 300 px

```CSS {.line-numbers}
div{
    width: 300px;
}
```

$RealWidth = width + padding + border = x + 50 * 2 +  10 *2 = 300 px$

## 語言

在 Boostrap 文字設定上，Boostrap3 時，採用 Helvetica Neue、Helvetica 和 Arial，Boostrap4 的版本調整了預設。 **Mac 與 iOS 都是有走預設的字體**，英文會使用`San Francisco`，中文是`PingFang`，但在 Windows 與 Android 就沒有使用預設的字體，所以僅會使用所認得的字體，以 Windows 中文預設是`新細明體`，但這樣與原本英文採用的 Segoe UI，長得很不協調，因此建議可以將中文字改為讀取微軟正黑體(請使用英文 `Microsoft JhengHei`)，而 Android 本身沒有預設的中文字體所以無要緊。

| 系統    | 系統預設      | 英文字體                       | 中文字體                            |
| ------- | ------------- | ------------------------------ | ----------------------------------- |
| Windows |               | Segoe UI                       | Microsoft JhengHei                  |
| Mac OS  | -apple-system | San Francisco / Helvetica Neue | 蘋方(PingFang) / Heiti TC           |
| iOS     | -apple-system | San Francisco / Helvetica Neue | 蘋方(PingFang) / Heiti TC           |
| Android |               | Roboto                         | Droidsansfallback(有無選差異不太大) |

_註: -apple-system 已經包含預設的中文與英文字體_

### 載入的範例

```CSS {.line-numbers}
body{
    font-family:
    /* 1 */  -apple-system, BlinkMacSystemFont,
    /* 2 */ "Segoe UI", "Roboto", "Oxygen", "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans", "Microsoft JhengHei",
    /* 3 */ "Helvetica Neue", sans-serif;
}
```

1. 使用系統預設字體
2. 已知系統用字體
3. 備援用字體

[參考](https://csspod.com/using-the-system-font-in-web-content/)
