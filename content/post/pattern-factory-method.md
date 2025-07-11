+++
title = "工廠方法模式 (Factory Method Pattern)"
description = "深入探討工廠方法模式如何讓每個產品都有自己的工廠，符合開放封閉原則"
date = 2025-07-10 21:00:00
author = "Rhan0"
tags=["pattern", "design-pattern", "typescript", "factory"]
weight= 3
+++

## 什麼是工廠方法模式？

工廠方法模式定義了一個建立物件的介面，但讓子類別決定要實例化哪個類別。這種模式將物件的建立延遲到子類別，符合開放封閉原則。

## 核心概念

- **抽象工廠介面**：定義建立物件的抽象方法
- **具體工廠類別**：每個產品都有自己的工廠
- **開放封閉原則**：新增產品時不需要修改現有程式碼

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

// 抽象工廠介面
interface VehicleFactory {
    create(): Vehicle;
}

// 具體工廠類別
class CarFactory implements VehicleFactory {
    create(): Vehicle { 
        return new Car(); 
    }
}

class BikeFactory implements VehicleFactory {
    create(): Vehicle { 
        return new Bike(); 
    }
}

// 使用方式
const carFactory = new CarFactory();
const bikeFactory = new BikeFactory();

const car = carFactory.create();
const bike = bikeFactory.create();

car.drive(); // 輸出: Driving a car
bike.drive(); // 輸出: Riding a bike
```