---
title: 簡易編譯器原理介紹
date: 2020-04-16 22:05:16
tags: [complier, javascript]
categories: 
- program
---

# 前言
編譯器一直都在你我每日的程式工作中，當然如果是資訊領域背景的，相信對編譯器並不會太陌生，早期大學時期寫過scheme這教育意義比較大的語言時，有了解編譯器實作，但時至今日沒有自己真的親手實過過。

# 編譯步驟
大多數的編譯器都有這三步驟
1. Parsing - 將原始資料轉換為可以抽象表示。
2. Transformation - 將抽象表示進行操作或轉換，轉換為你期待的結構。
3. Code Generation - 透過轉換過的抽換結構，產出新的程式。

## Parsing
Parsing可分為兩個步驟，詞法分析(Lexical Analysis)和句法分析(Syntactic Analysis)。
詞法分析其實就是針對單個字進行解析，將單詞轉化為token。句分析則是將token組合出AST(Abstract Syntax Tree)。

舉例以下詞句
```
(add 2(subtract 4 2))
```

Token如下
```
[
  { type: 'paren',  value: '('        },
  { type: 'name',   value: 'add'      },
  { type: 'number', value: '2'        },
  { type: 'paren',  value: '('        },
  { type: 'name',   value: 'subtract' },
  { type: 'number', value: '4'        },
  { type: 'number', value: '2'        },
  { type: 'paren',  value: ')'        },
  { type: 'paren',  value: ')'        },
]
```

而AST則是如下
```
{
  type: 'Program',
  body: [{
    type: 'CallExpression',
    name: 'add',
    params: [{
      type: 'NumberLiteral',
      value: '2',
    }, {
      type: 'CallExpression',
      name: 'subtract',
      params: [{
        type: 'NumberLiteral',
        value: '4',
      }, {
        type: 'NumberLiteral',
        value: '2',
      }]
    }]
  }]
}
```

## Transformation
轉換只是對上一步的AST進行更改或置換，這個步驟是將原始AST可能改成新的語言格式進行的轉換，或是增加一些新的節點，可能增加前後括號？可能更改順序，可能刪除不必要節點等等..，這時候產出的新AST，就會是我們目標編譯語言可能的結構。


### Traversal
為了能到各節點，我們得能尋訪各節點，這也是必要的。
```
{
  type: 'Program',
  body: [{
    type: 'CallExpression',
    name: 'add',
    params: [{
      type: 'NumberLiteral',
      value: '2'
    }, {
      type: 'CallExpression',
      name: 'subtract',
      params: [{
        type: 'NumberLiteral',
        value: '4'
      }, {
        type: 'NumberLiteral',
        value: '2'
      }]
    }]
  }]
}
```
以上的結構呢，將會是以下的步驟尋訪。
```
1. Program - 從最上面的節點開始
2. CallExpression (add) - 從第一個程式元素開始
3. NumberLiteral (2) - 移動到參數的第一個元素
4. CallExpression (subtract) - 移動到參數的第二個元素
5. NumberLiteral (4) - 移動到參數的第一個元素
6. NumberLiteral (2) - 移動到參數的第二個元素
```

### Visitors
這邊呢基本上是創建一個Visitor的物件，這物件可接受不同節點類型。
```
var visitor = {
  NumberLiteral() {},
  CallExpression() {},
};
```
當我們在尋訪AST時，當我們輸入對應的類型時，我們可以呼叫visitor上對應的方法。

為了更便於使用，再加上傳遞父節點的參考。
```
var visitor = {
  NumberLiteral(node, parent) {},
  CallExpression(node, parent) {},
};
```

當然我們一定有存在關閉的點。想像一下以下的樹狀結構。
```
* Program
  * CallExpression
    * NumberLiteral
    * CallExpression
        * NumberLiteral
        * NumberLiteral
```

其實也就是當我們走到那節點的終點時，我們會退回上一步，其實就是基本樹尋訪的基本知識而已。
```
→ Program (enter)
  → CallExpression (enter)
    → NumberLiteral (enter)
    ← NumberLiteral (exit)
    → CallExpression (enter)
      → NumberLiteral (enter)
      ← NumberLiteral (exit)
      → NumberLiteral (enter)
      ← NumberLiteral (exit)
    ← CallExpression (exit)
  ← CallExpression (exit)
← Program (exit)
```

為了實現退出，我們的visitor可能會長的這樣。
```
var visitor = {
  NumberLiteral: {
    enter(node, parent) {},
    exit(node, parent) {},
  }
};
```

## Code Generation
最後一階段就是產生程式碼，有時候會重疊到transformation的工作，但大部分Code Generation其實就是將AST產成代碼。

程式碼產生器有很多不同種方式產程式碼，但大多都是用我們剛剛建好的AST進行。程式碼產生器，透過遞迴、迭代等等方式，最終將產出可運行的程式碼。

# 結論
大體而言，其實就是語意轉為原始AST，再透過自訂規則轉為我們期待的AST，透過AST再輸出成可以執行的程式碼。