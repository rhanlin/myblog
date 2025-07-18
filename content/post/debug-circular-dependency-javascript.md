+++
title = "踩坑筆記：JavaScript 模組的循環依賴與 Cannot access before initialization 錯誤"
description = "深入探討 JavaScript 模組循環依賴問題，解析 Cannot access before initialization 錯誤的成因與解決方案"
date = 2025-01-27 20:00:00
author = "Rhan0"
tags=["javascript", "module", "circular-dependency", "debug"]
weight= 2
+++

# JavaScript 模組的循環依賴與 `Cannot access ... before initialization` 錯誤

在日常的程式碼開發中，我們經常會遇到各種奇奇怪怪的錯誤。其中有一種錯誤，它的訊息看似簡單，卻可能隱藏著深層的架構問題，那就是 `Uncaught ReferenceError: Cannot access 'X' before initialization`。這篇文章將深入探討這個錯誤背後的「循環依賴」問題，以及一個看似無害的改動如何引爆這個潛在的炸彈。

## 什麼是循環依賴？

循環依賴是指兩個或多個模組互相依賴對方的情況：

- 模組 A 需要使用模組 B 的功能
- 同時，模組 B 也需要使用模組 A 的功能

這就像兩個人互相需要對方才能完成任務，形成了一個「你中有我，我中有你」的循環。

### 循環依賴的程式碼範例

```javascript
// moduleA.js
import { someFunctionB } from './moduleB';

export function someFunctionA() {
  // ... 使用 someFunctionB
}
```

```javascript
// moduleB.js
import { someFunctionA } from './moduleA';

export function someFunctionB() {
  // ... 使用 someFunctionA
}
```

## 為什麼循環依賴會導致 `Cannot access ... before initialization`？

JavaScript 的 ES Module 有一套嚴格的加載和初始化機制。當模組被加載時，它會經歷幾個階段：

1. **解析 (Parsing)**：讀取檔案，找出所有的 `import` 和 `export`
2. **加載 (Loading)**：遞歸地加載所有依賴的模組
3. **實例化 (Instantiation)**：為模組創建一個「空殼」，並將 `export` 的變數綁定到這個空殼上。此時，這些變數處於「暫時性死區 (Temporal Dead Zone, TDZ)」，尚未被賦值
4. **執行 (Evaluation)**：執行模組的頂層程式碼，為 `export` 的變數賦值

當循環依賴發生時，問題就來了：

1. 引擎開始加載 `moduleA.js`
2. 發現 `import { someFunctionB } from './moduleB';`，於是暫停 `moduleA.js`，轉去加載 `moduleB.js`
3. 引擎開始加載 `moduleB.js`
4. 發現 `import { someFunctionA } from './moduleA';`。此時，引擎發現 `moduleA.js` 正在加載中，它不會重新加載，而是給 `moduleB.js` 一個指向 `moduleA.js` 導出變數的「綁定」
5. 現在，如果 `moduleB.js` 的**頂層程式碼**嘗試使用 `someFunctionA`，而 `moduleA.js` 還沒有執行到為 `someFunctionA` 賦值的階段，那麼 `someFunctionA` 就還在 TDZ 中，導致 `Cannot access 'someFunctionA' before initialization` 錯誤

### 實際範例：會出錯的循環依賴

```javascript
// utils/constants.js
import { calculateValue } from '../core/calculator';

export const SOME_CONSTANT = calculateValue(10);
```

```javascript
// core/calculator.js
import { SOME_CONSTANT } from '../utils/constants';

export function calculateValue(input) {
  // 假設這裡會用到 SOME_CONSTANT，但為了簡化，我們只做個簡單計算
  return input * 2 + (SOME_CONSTANT || 0); // SOME_CONSTANT 在這裡可能還是 undefined
}

// 模組加載時就調用，觸發問題
console.log('Calculator initialized with:', calculateValue(5));
```

當 `core/calculator.js` 被加載時，它會嘗試導入 `SOME_CONSTANT`。但 `SOME_CONSTANT` 的值又依賴於 `calculateValue`，而 `calculateValue` 又在 `core/calculator.js` 中定義。這就形成了一個循環。在 `core/calculator.js` 執行到 `calculateValue(5)` 時，`SOME_CONSTANT` 可能還沒有被賦值，導致錯誤。

## 潛在的炸彈：從 async 到 sync 的轉變

有時候，循環依賴可能長期存在於你的程式碼中，卻從未引發問題。這通常是因為觸發循環的程式碼是在**非同步**的上下文中執行的，例如在一個 `async` 函式內部，或者在一個事件監聽器中。

```javascript
// 假設這是舊的程式碼，雖然有循環，但因為是 async，所以沒問題
// utils/api.js
import { processData } from '../data/processor';

export async function fetchDataAndProcess() {
  const rawData = await fetch('/api/data');
  return processData(rawData); // 這裡執行時，所有模組都已初始化
}
```

```javascript
// data/processor.js
import { fetchDataAndProcess } from '../utils/api';

export function processData(data) {
  // ... 處理數據
  return data;
}
```

然而，如果某個 commit 將 `fetchDataAndProcess` 從 `async` 改為 `sync`，並且在模組的頂層作用域調用它，那麼這個潛在的循環依賴就會立刻被引爆！

```javascript
// utils/api.js (改動後)
import { processData } from '../data/processor';

export function fetchDataAndProcess() { // 不再是 async
  const rawData = getCachedData(); // 假設這裡同步獲取數據
  return processData(rawData);
}

// 模組加載時就調用，觸發問題
const initialProcessedData = fetchDataAndProcess();
```

現在，當 `utils/api.js` 加載時，它會立即調用 `fetchDataAndProcess`，而 `fetchDataAndProcess` 又需要 `processData`，`processData` 又需要 `fetchDataAndProcess`... 由於現在是同步執行，模組加載器會發現 `processData` 在被使用時還沒有完全初始化，於是 `Cannot access ... before initialization` 錯誤就出現了。

### 立即執行模組 (IIFE) 與環境差異

除了 `async` 到 `sync` 的轉變，另一種常見的觸發循環依賴的方式是模組本身就是一個**立即執行函式表達式 (IIFE)**，或者在模組的頂層作用域就直接調用了依賴鏈上的函式。

此外，你可能會發現這類錯誤在開發環境 (例如 Vite dev server) 中頻繁出現，但在生產環境 (production build) 中卻「消失」了。這並非表示生產環境沒有循環依賴，而是因為：

- **開發環境**：通常會利用瀏覽器原生的 ES Modules 加載機制，嚴格遵循模組的加載和初始化規範
- **生產環境**：打包工具會將多個模組整合成單一或少數檔案，可能會重新排列執行順序，掩蓋了問題

## 如何解決與預防循環依賴？

解決循環依賴的根本方法就是**打破循環**。以下是一些常見且有效的方法：

### 1. 提取共同依賴 (Extract Common Dependencies)

這是最常見也最推薦的方法。將共同的依賴提取到一個全新的、獨立的模組中：

```javascript
// common/config.js (新的、無依賴的模組)
export const BASE_URL = 'https://api.example.com';
export const DEFAULT_TIMEOUT = 5000;
```

```javascript
// moduleA.js (現在只依賴 common/config)
import { BASE_URL } from './common/config';
// ...
```

```javascript
// moduleB.js (現在只依賴 common/config)
import { DEFAULT_TIMEOUT } from './common/config';
// ...
```

### 2. 重構模組職責 (Refactor Module Responsibilities)

重新審視模組的職責，確保每個模組都有清晰的單一職責：

```javascript
// 重構前：兩個模組互相依賴
// userService.js
import { validateUser } from './validationService';
export function createUser(userData) {
  return validateUser(userData);
}

// validationService.js
import { createUser } from './userService';
export function validateUser(userData) {
  // 驗證邏輯
}
```

```javascript
// 重構後：分離職責
// userService.js
import { validateUser } from './validationService';
export function createUser(userData) {
  const isValid = validateUser(userData);
  if (isValid) {
    // 創建用戶邏輯
  }
}

// validationService.js
export function validateUser(userData) {
  // 純驗證邏輯，不依賴其他服務
}
```

### 3. 延遲加載 (Lazy Loading)

如果某些依賴只在特定條件下才需要，可以考慮使用動態 `import()` 來延遲加載：

```javascript
// moduleA.js
export async function doSomethingConditional() {
  if (condition) {
    const { moduleBFunction } = await import('./moduleB');
    moduleBFunction();
  }
}
```

### 4. 依賴注入 (Dependency Injection)

通過參數傳遞依賴，而不是直接導入：

```javascript
// 重構前
// serviceA.js
import { serviceB } from './serviceB';
export function processData(data) {
  return serviceB.transform(data);
}

// 重構後
// serviceA.js
export function processData(data, transformFunction) {
  return transformFunction(data);
}

// 使用時
import { processData } from './serviceA';
import { transform } from './serviceB';
const result = processData(data, transform);
```

## 檢測循環依賴的工具

### 1. ESLint 插件

```bash
npm install eslint-plugin-import
```

```javascript
// .eslintrc.js
module.exports = {
  plugins: ['import'],
  rules: {
    'import/no-cycle': 'error'
  }
};
```

### 2. Webpack Bundle Analyzer

```bash
npm install webpack-bundle-analyzer
```

### 3. Madge

```bash
npm install -g madge
madge --circular src/
```

## 總結

`Cannot access ... before initialization` 錯誤是 JavaScript 模組化開發中一個常見的陷阱。它通常是循環依賴的表現，而一個看似無害的同步化改動，就可能將這個潛在的架構問題暴露出來。

解決之道在於理解模組的加載機制，並積極地重構程式碼，打破循環依賴，特別是將基礎的、無依賴的常數和工具函式抽離到獨立的模組中。這不僅能解決當前問題，更能提升程式碼的健壯性和可維護性。

### 關鍵要點

- 循環依賴會導致 `Cannot access ... before initialization` 錯誤
- 從 async 到 sync 的轉變可能引爆潛在的循環依賴
- 提取共同依賴是解決循環依賴最有效的方法
- 使用工具檢測和預防循環依賴
- 清晰的模組職責劃分有助於避免循環依賴

### 參考資料

- [ES Modules: A cartoon deep-dive](https://hacks.mozilla.org/2018/03/es-modules-a-cartoon-deep-dive/)
- [JavaScript modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
- [Circular dependency](https://en.wikipedia.org/wiki/Circular_dependency)
