+++
title = "命名原則-JavaScript"
description = ""
date = 2023-09-28 08:00:00
author = "Rhan0"
tags=["conventions"]
weight= 2
+++

## 變數的命名 (for Variables)

1. 使用 camelCase 為變數命名

```javascript
// bad
var dogname = 'Droopy'; 
// bad
var dog_name = 'Droopy'; 
// bad
var DOGNAME = 'Droopy'; 
// bad
var DOG_NAME = 'Droopy'; 

// good
var dogName = 'Droopy';
```

2. 明確表明意義的命名
(self-explanatory and describe the stored value)

```javascript
// bad
var d = 'Scooby-Doo';
// bad
var name = 'Scooby-Doo';

// good
var dogName = 'Scooby-Doo';
```

## 布林值的命名 (for Booleans)

使用開頭為 is/has 的命名
(Use is or has as prefixes)

```javascript
// bad
var bark = false;
// good
var isBark = false;

// bad
var ideal = true;
// good
var areIdeal = true;

// bad
var owner = true;
// good
var hasOwner = true;
```

## 函式的命名(for Functions)

1. 使用 camelCase 為函式命名
2. 使用動詞作為前綴
3. 如果有特定意義或範圍，使用描述性的名詞作為前綴

Use descriptive nouns and verbs as prefixes.

```javascript
// bad
function dogName(dogName, ownerName) { 
  return `${dogName} ${ownerName}`;
}

// good
function getDogName(dogName, ownerName) { 
  return `${dogName} ${ownerName}`;
}
```

## 常數的命名(for Constants)

1. 使用 UPPERCASE 為常數命名(表示它們是不變的變數)

```javascript
var SPEED = 1;
```

2. 如果常數宣告名稱包含多個單字，則應使用 UPPER_SNAKE_CASE。

```javascript
var DAYS_UNTIL_TOMORROW = 1;
```

## 類別的命名(for Classes)

使用 PascalCase 為類別命名

```javascript
class Dog {
  constructor() {}
}
```

## 方法的命名(for Methods)

使用 camelCase 為元件命名

```javascript
class Dog {
  constructor() {}
  pickUpBall() {}
}
```

## 私有函式/私有變數的命名(for Private Functions/Variables in Classes)

Use a hash # prefix.

```javascript
class ClassWithPrivate {
  #privateField;
  #privateFieldWithInitializer = 42;

  #privateMethod() {
    // …
  }

  static #privateStaticField;
  static #privateStaticFieldWithInitializer = 42;

  static #privateStaticMethod() {
    // …
  }
}
```

## 元件的命名(for Components)

使用 PascalCase 為元件命名

```javascript
function Dog(){
  return <div></div>
}
```

## 全域變數的命名 (for Global Variables)

如果變數在所有地方都可以存取它，則它被定義為全域變數
全域 JavaScript 變數沒有特殊的命名約定，但仍可以嘗試遵守下列原則：

1. 全域 JavaScript 變數在專案/文件的頂部聲明
2. 如果全域 JavaScript 變數是可變的，則以 camelCase 命名
3. 如果全域 JavaScript 變數是不可變的，則用 UPPERCASE 命名

## 檔案的命名 (for File Names)
// TODO: