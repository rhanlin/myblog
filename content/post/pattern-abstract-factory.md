+++
title = "抽象工廠模式 (Abstract Factory Pattern)"
description = "深入探討抽象工廠模式如何建立相關產品族，確保產品族內的一致性"
date = 2025-07-10 21:00:00
author = "Rhan0"
tags=["pattern", "design-pattern", "typescript", "factory"]
weight= 3
+++

## 什麼是抽象工廠模式？

抽象工廠模式提供一個建立一系列相關或相互依賴物件的介面，而不需要指定它們的具體類別。這種模式確保產品族內的一致性，常用於 UI 框架的實作。

## 核心概念

- **產品族**：一系列相關的產品
- **抽象工廠介面**：定義建立產品族的方法
- **具體工廠**：實作特定產品族的建立邏輯
- **產品一致性**：確保同一產品族內的產品能正常配合

## 實際範例

```typescript
// 產品介面
interface Button {
    render(): void;
}

interface Input {
    render(): void;
}

// Material UI 產品族
class MaterialButton implements Button {
    render(): void {
        console.log('Rendering Material UI button');
    }
}

class MaterialInput implements Input {
    render(): void {
        console.log('Rendering Material UI input');
    }
}

// Bootstrap 產品族
class BootstrapButton implements Button {
    render(): void {
        console.log('Rendering Bootstrap button');
    }
}

class BootstrapInput implements Input {
    render(): void {
        console.log('Rendering Bootstrap input');
    }
}

// 抽象工廠介面
interface UIFactory {
    createButton(): Button;
    createInput(): Input;
}

// 具體工廠類別
class MaterialUIFactory implements UIFactory {
    createButton(): Button { 
        return new MaterialButton(); 
    }
    
    createInput(): Input { 
        return new MaterialInput(); 
    }
}

class BootstrapUIFactory implements UIFactory {
    createButton(): Button { 
        return new BootstrapButton(); 
    }
    
    createInput(): Input { 
        return new BootstrapInput(); 
    }
}

// 使用方式
function createUI(factory: UIFactory) {
    const button = factory.createButton();
    const input = factory.createInput();
    
    button.render();
    input.render();
}

// 使用 Material UI
createUI(new MaterialUIFactory());
// 輸出:
// Rendering Material UI button
// Rendering Material UI input

// 使用 Bootstrap
createUI(new BootstrapUIFactory());
// 輸出:
// Rendering Bootstrap button
// Rendering Bootstrap input
```
