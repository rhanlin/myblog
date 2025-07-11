+++
title = "元件開發原則: 元件設計五步驟"
description = ""
date = 2023-11-10 21:00:00
author = "Rhan0"
tags=["conventions"]
weight= 2
+++

## 重點

- 單向資料流 (one-way data flow)
- 資料由上往下傳遞
- 狀態權責 (父或子層)


## 元件設計五步驟

#### 1. 將 UI 拆分成 Component

- 👉 Programming (遵守單一職責原則 (SRP))

    > [單一職責原則 (Single Responsibility Principle)]({{< ref "draft/principles-single-responsibility" >}})
    
- 👉 CSS (依照 selectors 區分)
- 👉 Design (依照設計圖分層區分)

*(若 JSON 資料的結構良好，通常能直接映射到相對應的 Component)*


#### 2. 建立一個靜態版本的 Component (沒有任何互動功能)


#### 3. 設計異動最小但最完整的 state

*(有些 state 是不必要的，靠 props 傳遞即可，不需要再宣告一次 useState 來裝載 props)*


#### 4. state 要放哪裡？

*(當有多個子 component 需要共用 state，則 state 必須往上提升到父 component 管理 (Lifting State Up))*


#### 5. 反向數據流 (子層改變父層狀態)


## 參考規範
- [基礎規範]({{< ref "draft/principles-react-component" >}})
- [JS Sweetness]({{< ref "draft/principles-js-sweetness" >}})