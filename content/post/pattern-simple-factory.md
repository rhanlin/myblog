+++
title = "簡單工廠模式 (Simple Factory Pattern)"
description = "深入探討簡單工廠模式如何統一管理物件建立，並提供實際的程式碼範例"
date = 2025-07-10 21:00:00
author = "Rhan0"
tags=["pattern", "design-pattern", "typescript", "factory"]
weight= 3
+++

## 什麼是簡單工廠模式？

簡單工廠模式是最基本的工廠模式，它提供一個統一的介面來建立物件，而不需要直接使用 `new` 關鍵字。這種模式將物件的建立邏輯集中管理，提高了程式碼的可維護性。

## 核心概念

- **統一介面**：提供一個靜態方法來建立物件
- **集中管理**：將建立邏輯集中在一個地方
- **降低耦合**：客戶端不需要知道具體的實作細節

## 實際範例

```typescript
// 產品介面
interface Vehicle {
    drive(): void;
}

// 具體產品
class Car implements Vehicle {
    drive(): void {
        console.log('Driving a car');
    }
}

class Bike implements Vehicle {
    drive(): void {
        console.log('Riding a bike');
    }
}

// 簡單工廠
class VehicleFactory {
    static create(type: 'car' | 'bike'): Vehicle {
        switch (type) {
            case 'car': 
                return new Car();
            case 'bike': 
                return new Bike();
            default: 
                throw new Error('Unknown vehicle type');
        }
    }
}

// 使用方式
const car = VehicleFactory.create('car');
const bike = VehicleFactory.create('bike');

car.drive(); // 輸出: Driving a car
bike.drive(); // 輸出: Riding a bike
```