+++
title = "開發慣例: 通用命名慣例"
description = ""
date = 2023-09-27 20:00:00
author = "Rhan0"
tags=["conventions"]
weight= 2
+++

## 淺而易懂的命名 (Use Intention-Revealing Names)
如果一個命名，還需要註解來解釋它，就代表還不夠清楚

```javascript
// Bad
var dog; // the dog is barking
// Good
var barkingDog;
```

## 有意義的命名 (Make Meaningful Distinctions)
使用帶有目的性或是意義明確的命名，避免使用縮寫、或是用數字編號作為命名

```javascript
// Bad
var e = 'event'
// Good
var event = 'event'
```

## 避免誤導 (Avoid Disinformation)
  1. 避免用不同字眼代表同一件事，例如：info/data
  2. 避免贅字
  3. 避免兩個命名太相像

## 可以被搜尋 (Use Searchable Names)
  1. 避免用數字與單一字母命名，可改採用 Constant 或 Enum 賦予意義
  2. longer names比shorter names好，至少搜尋容易定位

## 可以被發音 (Use Pronounceable Names)
  使用可以發音的命名可以幫助溝通