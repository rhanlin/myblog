+++
title = "元件開發原則: JS Sweetness"
description = ""
date = 2023-11-08 21:00:00
author = "Rhan0"
tags=["conventions"]
weight= 2
+++

## Switch-case to Object

```javascript
// ❌ 過多無謂的判斷及重新復賦值
const getDay = (type) => {
  let day = null;

  switch (type) {
    case 0:
      day = 'Sunday';
      break;
    case 1:
      day = 'Monday';
      break;
    case 2:
      day = 'Tuesday';
      break;
    case 3:
      day = 'Wednesday';
      break;
    case 4:
      day = 'Thursday';
      break;
    case 5:
      day = 'Friday';
      break;
    case 6:
      day = 'Saturday';
      break;
    default:
      break;
  }

  return day;
};

// ✅ 變得更一目了然、簡單明瞭
const days = {
  0: 'Sunday',
  1: 'Monday',
  2: 'Tuesday',
  3: 'Wednesday',
  4: 'Thursday',
  5: 'Friday',
  6: 'Saturday',
  7: 'Sunday',
};

const getDay = (type) => {
  return days[type] || null;
};
```


## Default parameters

```javascript
// ❌ 預設參數不為最後一個
function getPassword(account, name = '', id) {
  // do something
}
// ❌ 若預設參數不為最後一個，則可能會有這種程式碼出現 (bad)
getPassword(account, undefined, id);

// ✅ 保持預設參數為最後一個的習慣
function getPassword(account, id, name = '') {
  // do something
}

// ✅ 若為最後一個，則可以避免剛才那種程式碼出現 (good)
getPassword(account, id);
```


## Multiple parameters

```javascript
// ❌ 若過多個參數須控制，則會造成閱讀上的困難
function getPassword(account, id, name, birthday, sex, phone) {
  // do something
}

// ❌ 呼叫此 function 時，會需要數第幾個
getPassword(acc, id, roleName, roleBirthDay, roleSex, rowPhone);

// ✅ 建議超過 3 個就改為物件
function getPassword({ account, id, name, birthday, sex, phone }) {
  // do something
}

// ✅ 呼叫此 function 時，也不需要數第幾個，名稱直接對應
getPassword({ account, id, name, birthday, sex, phone });
```