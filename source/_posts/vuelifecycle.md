---
title: VueLifeCycle
tags: Vue
categories:
  - Tech
  - Front-End
date: 2019-04-03 12:00:00
---

# Vue 生命週期 (Vue Life Cycle)
這是一份記錄自己每次使用 Vue 的生命週期的筆記
![new Vue後的生命週期流程圖](https://i.imgur.com/QdgqeRA.png)

## beforeCreate 與 created

beforeCreate 是綁定 DOM 之前，並在 Vue instance Constuctor 建立之前
Created 是綁定 DOM 之前，並在 Vue instance Constructor 建立之後

## beforeMount 與 mounted

beforeMount template 或是 el 上的 HTML 已經被 compile，但是是綁定 DOM 之前
mounted 則是綁定 DOM 之後

## beforeUpdae 與 updated

beforeUpdae 當有資料更新時，但未更新 DOM，但是是準備要進行更新
updated 則是資料更新之後，也更新完 DOM
**特別注意，如果同個畫面上並沒有資料被異動，即便觸發 click 事件，也不會進行 beforeUpdate 與 updated**

## beforeDestroy 與 Destroy

當 Vue Instance 走到生命週期末端，執行 vm.\$distroy()或在 methods 內執行 this.distroy()時
beforeDestroy 在釋放這個物件之前被執行
Destroy 釋放這個物件之後被執行

---

HTML Code

```html{.line-numbers}
<div id="app">
  <h1>{{title}}</h1>
  <button @click="update">update title</button>
  <button @click="destroy">Click me to distory</button>
</div>
```

Javascript Code

```javascript{.line-numbers}
new Vue({
  el: "#app",
  data: {
    title: "This is Title",
  },
  methods: {
    update: function () {
      this.title = "After updating";
    },
    destroy: function () {
      this.$destroy();
    },
  },
  beforeCreate: function () {
    console.log("beforeCreate");
  },
  created: function () {
    console.log("created");
  },
  beforeMount: function () {
    console.log("beforeMounted");
  },
  mounted: function () {
    console.log("mounted");
  },
  beforeUpdate: function () {
    console.log("beforeUpdate");
  },
  updated: function () {
    console.log("updated");
  },
  beforeDestroy: function () {
    console.log("beforedDestory");
  },
  destroyed: function () {
    console.log("destroy");
  },
});
```

[CodePen 執行結果](https://codepen.io/chuangmaster/pen/poyopBg?editors=1011)
