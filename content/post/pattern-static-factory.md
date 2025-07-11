+++
title = "靜態工廠模式 (Static Factory Pattern) 的應用與優勢"
description = "深入探討靜態工廠模式如何解決複雜物件建立問題，並以實際的 ImageLoader 重構案例展示其優勢"
date = 2025-07-10 21:00:00
author = "Rhan0"
tags=["pattern", "design-pattern", "typescript", "refactoring"]
weight= 2
+++

## 引言

在軟體開發中，物件的建立是最常見的操作之一。我們通常直接使用 `new` 關鍵字來實例化物件，但當建立過程變得複雜、需要非同步操作，或者我們想對建立過程有更多控制時，直接使用 `new` 可能就不夠了。

本文將介紹**靜態工廠模式**如何解決這些問題，並以 `ImageLoader` 重構為例，展示實際應用場景

## 什麼是工廠模式？

工廠模式的核心概念是將物件的建立邏輯從直接的 `new` 呼叫中抽離出來，委託給一個專門的「工廠」來負責。這樣做的主要目的是降低耦合、提高彈性。

工廠模式有多種變體：
- **簡單工廠**：一個工廠類別負責建立多種產品
- **工廠方法**：每個產品都有自己的工廠
- **抽象工廠**：建立相關產品族
- **靜態工廠方法**：本文重點討論的模式

## 靜態工廠方法詳解

靜態工廠方法是類別內部的一個 `static` 方法，負責建立並返回該類別的實例（或其子類別的實例）。

### 命名慣例

常見的靜態工廠方法名稱：

#### `create()` - 建立新實例
最常見的命名，表示建立一個全新的實例：

```typescript
// 組件建立
const button = Button.create({ text: 'Click me', onClick: handleClick });

// API 客戶端建立
const apiClient = ApiClient.create({ baseURL: 'https://api.example.com' });

// 資料庫連接建立
const dbConnection = await DatabaseConnection.create(config);
```

#### `of()` - 從參數建立實例
表示從某些參數或值建立實例：

```typescript
// 從顏色名稱建立顏色物件
const red = Color.of('red');
const blue = Color.of('blue');

// 從數字建立點座標
const point = Point.of(10, 20);

// 從字串建立使用者
const user = User.of('john', 'doe');
```

#### `from()` - 從來源轉換建立實例
表示從某種來源或格式轉換建立實例：

```typescript
// 從 JSON 建立物件
const user = User.from(jsonData);

// 從 URL 建立圖片載入器
const imageLoader = await ImageLoader.from('https://example.com/image.jpg');

// 從檔案建立配置
const config = Config.fromFile('config.json');
```

#### `getInstance()` - 單例或快取實例
通常用於單例模式或快取實例：

```typescript
// 全域的設定管理器
const configManager = ConfigManager.getInstance();

// 應用程式狀態管理
const appState = AppState.getInstance();

// 快取管理器
const cacheManager = CacheManager.getInstance();
```

### 基本範例

```typescript
class MyClass {
    private constructor(public id: string) {} // 私有建構子

    public static create(id: string): MyClass {
        // 可以在這裡加入額外邏輯
        return new MyClass(id);
    }
}

// 使用方式: MyClass.create('abc') 而不是 new MyClass('abc')
```

## 靜態工廠模式解決的問題

### 1. 複雜或非同步的初始化流程

**問題**：constructor 必須是同步的，因為 constructor 是在類別實例化時自動同步呼叫的函數，無法回傳 Promise
。如果物件建立需要讀取檔案、網路請求、大量計算等非同步或耗時操作，建構子無法直接處理。

```typescript
class MyClass {
  async constructor() { // ❌ 語法錯誤
    await someAsyncTask();
  }
}
```

我們很直覺的會想到，如果建構子必須是同步的，那就分開建構與初始化就好，不過總覺得不夠優雅。

```typescript
class MyClass {
  data: string = '';

  async init() {
    this.data = await fetchData();
  }
}

// 使用方式
const instance = new MyClass();
await instance.init();
```

**解決方案**：使用 static async create() 靜態工廠方法，此作法可以是 `async` 的，允許在物件完全準備好後才返回實例。

```typescript
class MyClass {
  private constructor(private data: string) {
    // 同步初始化邏輯
  }

  static async create() {
    const result = await fetchData(); // 非同步處理
    return new MyClass(result);
  }
}

// 使用方式
const instance = await MyClass.create();
```

### 2. 強制控制實例的建立

**問題**：直接使用 `new` 允許任何人隨意建立物件，難以控制實例的生命週期或狀態。

**解決方案**：將建構子設為 `private`，強制所有實例都必須透過靜態工廠方法建立。

```typescript
class ImageLoader {
    private constructor(private url: string, private element: HTMLImageElement) {} // 私有建構子
    
    public static async create(url: string, element: HTMLImageElement): Promise<ImageLoader> {
        // 工廠方法作為「守門員」，確保實例符合特定規則
        const instance = new ImageLoader(url, element);
        await instance.load();
        return instance;
    }
    
    private async load(): Promise<void> {
        // 載入圖片的邏輯
        return new Promise((resolve, reject) => {
            this.element.onload = () => resolve();
            this.element.onerror = () => reject(new Error('Failed to load image'));
            this.element.src = this.url;
        });
    }
}
```

### 3. 返回子類型或不同實作 (多型)

**問題**：建構子永遠只能返回其自身的實例。

**解決方案**：靜態工廠方法可以根據輸入參數，返回不同子類別或不同實作的實例。

```typescript
class Vehicle {
    static create(type: 'car' | 'bike', config: any): Vehicle {
        switch (type) {
            case 'car': return new Car(config);
            case 'bike': return new Bike(config);
            default: throw new Error('Unknown vehicle type');
        }
    }
}
```

### 4. 實例的快取與重用 (單例、物件池)

**問題**：每次 `new` 都會建立新實例，可能造成資源浪費。

**解決方案**：工廠方法可以在內部維護一個快取，如果實例已存在，就直接返回舊實例。

```typescript
class ConfigManager {
    private static instance: ConfigManager;
    
    public static getInstance(): ConfigManager {
        if (!ConfigManager.instance) {
            ConfigManager.instance = new ConfigManager();
        }
        return ConfigManager.instance;
    }
}
```

## 靜態工廠模式的優勢

1. **更清晰的意圖**：方法名稱比建構子名稱更能表達建立的意圖
2. **更好的控制**：可以控制實例的建立過程、數量和類型
3. **更高的彈性**：易於修改建立邏輯，而無需修改客戶端程式碼
4. **支援非同步建立**：解決了建構子無法處理非同步操作的限制
5. **保證實例狀態**：確保返回的實例是完全初始化且可用的

## 實際案例：圖片載入器重構

### 重構前的問題

```typescript
// 舊的 ImageLoader 實作
class ImageLoader implements ILoader {
    private element: HTMLImageElement | null = null;
    private isLoaded: boolean = false;
    
    constructor() {
        // 只能同步初始化
    }
    
    async load(url: string, element: HTMLImageElement): Promise<ImageLoader> {
        // 雙重職責：載入圖片和返回已存在實例
        // 物件在一段時間內處於「半初始化」狀態
        this.element = element;
        this.isLoaded = false;
        
        return new Promise((resolve, reject) => {
            element.onload = () => {
                this.isLoaded = true;
                resolve(this);
            };
            element.onerror = () => reject(new Error('Failed to load image'));
            element.src = url;
        });
    }
    
    getElement(): HTMLImageElement | null {
        return this.element;
    }
}

// 使用方式
const loader = new ImageLoader();
await loader.load('https://example.com/image.jpg', imgElement); // 流程不夠直觀
```

**主要問題**：
- `constructor` 只能同步初始化，但實際載入圖片是透過 `async load()` 方法
- `load()` 方法有雙重職責（載入和返回已存在實例）
- `ILoader` 介面與實際的建立流程不符
- 客戶端必須先 `new ImageLoader()` 再 `await loader.load()`，流程不夠直觀

### 重構後的解決方案

```typescript
// 新的 ImageLoader 實作
class ImageLoader implements ILoader {
    private constructor(
        private element: HTMLImageElement,
        private url: string
    ) {
        // 私有建構子，強制透過工廠方法建立
    }
    
    public static async create(url: string, element: HTMLImageElement): Promise<ImageLoader> {
        const instance = new ImageLoader(element, url);
        await instance._loadImage();
        return instance; // 保證返回完全載入的實例
    }
    
    private async _loadImage(): Promise<void> {
        return new Promise((resolve, reject) => {
            this.element.onload = () => resolve();
            this.element.onerror = () => reject(new Error('Failed to load image'));
            this.element.src = this.url;
        });
    }
    
    public getElement(): HTMLImageElement {
        return this.element;
    }
    
    public getUrl(): string {
        return this.url;
    }
}

// 使用方式
const loader = await ImageLoader.create('https://example.com/image.jpg', imgElement); // 程式碼更簡潔、意圖更明確
```

**重構優勢**：
- **單一職責**：`create` 方法專注於建立和初始化
- **狀態保證**：返回的實例是完全載入且可用的
- **更清晰的介面**：`ILoader` 移除了 `load` 方法，使其更符合「實例契約」的職責
- **更好的客戶端體驗**：客戶端現在可以直接使用 `await ImageLoader.create(...)`

## 總結

靜態工廠模式特別適合處理複雜的物件建立邏輯，它不僅解決了建構子的限制，還提供了更好的封裝和控制能力。

在我們的 `ImageLoader` 重構案例中，靜態工廠模式幫助我們：
- 解決了非同步初始化的問題
- 提供了更清晰的 API 設計
- 確保了物件狀態的一致性
- 改善了程式碼的可讀性和可維護性

當你遇到複雜的物件建立需求時，不妨考慮使用靜態工廠模式來改善你的程式碼架構。
