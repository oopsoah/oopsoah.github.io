---
title: web component 輕鬆上手
date: 2020-05-17 21:28:48
tags: [javascript, web-component]
categories: 
- web component
---

![](/images/web-component.jpg)

# 前言
Web-Component可以用來封裝你的客製化元件，目前環境尚不支援IE11。

<!-- more -->

# 關鍵部分
Web Component由三項技術組成。


* Custom Elements
`javascript api`，允許自訂tag，tag便可在html中使用。
* Shadow DOM
`javascript api`，用以與原始DOM隔離，讓自訂element具有私有的scope(style、script)。
* HTML Templates
透過`<template>, <scope>`等等元素，可以重複重用使用模板。


## Custom Elements

* 建立自訂元素類別
  ### javascript
  ```javascript=
  class MyCard extends HTMLElement {
    ...
  }
  window.customElements.define('my-card', MyCard);
  ```

  ### HTML
  ```html=
  <my-card></my-card>
  ```

* 元素類別的生命週期方法
  * constructor()
    元件建立時會執行
  * connectedCallback()
    當元件被插入至DOM時會呼叫
  * disconnectedCallback()
    當元件從DOM被移除時會呼叫
  * attributeChangedCallback(attributeName, oldValue, newValue)
    當元件屬性被新增、刪除、修改時會呼叫
  
### 範例
```javascript=
  class MyCard extends HTMLElement {
    constructor() {
      super();
    }

    connectedCallback() {}

    disconnectedCallback() {}

    attributeChangedCallback() {}
  }

  window.customElements.define('my-card', MyCard);
```

## Shadow DOM


## HTML Template

