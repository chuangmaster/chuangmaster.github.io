---
title: VueComponent
tags: Vue
categories: Front-End
date: 2021-04-01 00:47:24
---

#Vue Component Learning

2021.04.01
紀錄學習 Vue Component 的筆記，Component 是目的要更方便操作與管理 Vue 的 Template

## Vue.Template

這是一種最簡單使用 Template，在建構式中帶入 template 的內容，特別注意 template 第一層僅能有一個root，也就是剩下的dom要往內包，假設如果是 h1 與 button 並排就沒辦法被 render
```javascript{.line-numbers}
new Vue({
  el: "#app",
  template: "<div><h1>Server Status: {{ status }} </h1> <button @click='changeStatus'> clickMe</button></div>",
  data: {
    status:'Normal'
  },
  methods:{
    changeStatus: function () {
      if (this.status == "Normal") {
        this.status = "Closed";
      } else {
        this.status = "Normal";
      }
    }
  }
});
```

## Vue Component

使用 Component，先註冊 template，並給予這個 template 一個 Name，在綁定的區塊中就可以使用這個名稱來將 template 給印出，而且不僅可以使用一次，彈性比上述的 Template 還更好，特別需要注意的是使用 component來註冊的時候，data 必須是一個 function return。

```html{.line-numbers}
<div id="app2">
  <hello></hello>
  <hello></hello>
</div>
```

```javascript{.line-numbers}
Vue.component("hello", {
  template:
    '<h1>Server Status: {{ status }} <button @click="changeStatus">change</button></h1>',
  data: function () {
    return {
      status: "Normal"
    };
  },
  methods: {
    changeStatus: function () {
      if (this.status == "Normal") {
        this.status = "Closed";
      } else {
        this.status = "Normal";
      }
    }
  }
});

var vm = new Vue({
  el: "#app2",
});
```

而 Component 這個 Object 也可以獨立出來，並且在註冊回 Vue 內的 Component

```javascript{.line-numbers}
var component = {
  template:
    '<h1>Server Status: {{ status }} <button @click="changeStatus">change</button></h1>',
  data: function () {
    return {
      status: "Normal"
    };
  },
  methods: {
    changeStatus: function () {
      if (this.status == "Normal") {
        this.status = "Closed";
      } else {
        this.status = "Normal";
      }
    }
  }
};

new Vue({
  el: "#app3",
  components: {
    cmp: component
  }
});
```

```html{.line-numbers}
<div id="app3">
  <cmp></cmp>
</div>
```

- 執行結果
https://codepen.io/chuangmaster/pen/yLedXzL?editors=1010