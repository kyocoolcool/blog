---
title: "Use TypeScript Better"
date: 2019-08-14T12:00:00+08:00
hero: /images/posts/writing-posts/git.svg
author:
  name: Chris Chen
#   image: /assets/images/profile-image.jpg
categories:
- front-end
---

## 1、About TypeScript

TypeScript is a typed superset of JavaScript that compiles to plain JacaScript。

由微軟開發開源的項目，JavaScript的擴充版，可使用ES6開發，運行在不支援ES6的瀏覽器，會自動編譯成JavaScript。

## 2、Why use TypeScript

1. 強型別:能夠在編譯時檢查型別的正確性。

2. 減少重複的 code:在 TypeScript 中可以利用class、constructor、interface、inheritance、moudle來編寫程式，像寫Java一樣方便(支持Lambda 😎)。

3. 限制存取範圍：可以方法利用 public / private / protected 來限制存取權限，變數可以利用 let 來使用區域變數。

   

## 3、TypeScript編譯成JacaScript

> TypeScript

```typescript
function sum5(a:number,b:number,...result:number[]):number {
    let sum4:number=a+b;
    result.forEach((x) => sum4+=x);
    return sum4;
}
alert(sum5(1, 2, 3, 4, 5,6,7));
```

> JavaScript

```javascript
function sum5(a, b) {
    var result = [];
    for (var _i = 2; _i < arguments.length; _i++) {
        result[_i - 2] = arguments[_i];
    }
    var sum4 = a + b;
    result.forEach(function (x) { return sum4 += x; });
    return sum4;
}
alert(sum5(1, 2, 3, 4, 5, 6, 7));
```

Note:比較一下上下區塊，是否上面更平易近人呢?

## 4、總結

以前在學習JavaScript時總覺得很痛苦，因為已經習慣Java強型態的語言，JavaScript無法在編譯時期就發現錯誤，在語法的要求上也沒有很嚴謹，使用TypeScript後，發現在寫前端網頁會快樂許多，更易讀更親近。

實作功能代碼：[typescript](https://github.com/kyocoolcool/typescript-fundamentals)

