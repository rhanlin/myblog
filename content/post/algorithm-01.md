+++
title = "資料結構 01"
description = "交大資訊工程學系 彭文志老師 資料結構課程筆記"
date = 2026-02-23 12:00:00
author = "Rhan0"
tags=["algorithm"]
weight= 1
+++

## Algorithm Specification

演算法是什麼？簡單來說就是**解決問題的方法**。一個合格的演算法必須滿足以下五個條件：

- **Input** — 輸入：你的問題
- **Output** — 輸出：你的答案
- **Definiteness** — 明確性：每一步都必須是明確的
- **Finiteness** — 有限性：必須在有限的步驟內結束
- **Effectiveness** — 有效性：每一步都必須是可執行的

---

## Selection Sort（選擇排序）

**Input：** 一堆未排序的數字

**Output：** 由小到大（或由大到小）排好的數字

> **核心概念：** 每次從剩餘的數字中找到最小的，放到當前位置，再從剩餘的數字中找下一個最小的，直到排完。

```javascript
// Time Complexity: O(n^2)
// Space Complexity: O(1)
function selectionSort(arr) {
  const N = arr.length;
  for (let i = 0; i < N - 1; i++) {
    let minIndex = i;
    for (let j = i + 1; j < N; j++) {
      if (arr[j] < arr[minIndex]) {
        minIndex = j;
      }
    }
    [arr[i], arr[minIndex]] = [arr[minIndex], arr[i]];
  }
  return arr;
}
```

---

## 多種 Sort

每一種排序演算法的設計理念不同，常見的排序演算法有：

| 排序演算法 | 英文 | 平均時間複雜度 |
|-----------|------|--------------|
| 選擇排序 | Selection Sort | O(n²) |
| 冒泡排序 | Bubble Sort | O(n²) |
| 插入排序 | Insertion Sort | O(n²) |
| 快速排序 | Quick Sort | O(n log n) |
| 歸併排序 | Merge Sort | O(n log n) |
| 基數排序 | Radix Sort | O(d × n) |

### Comparison-Based Sort 的理論下界

上面大部分的排序演算法（Selection, Bubble, Quick, Merge, Insertion）都是 **Comparison-Based**，也就是透過「比較兩個元素的大小」來決定順序。

任何 comparison-based 的排序演算法，最壞情況下至少需要 **O(n log n)** 次比較。因此 Quick Sort 的平均 O(n log n) 已經是這個框架下的極限。

### Radix Sort 如何突破 O(n log n)？

Radix Sort **不比較元素大小**，而是用 **Bucket（桶）** 按位數分類，本質上是一種 hashing / counting。因為不是 comparison-based，所以不受 O(n log n) 下界限制，時間複雜度為 **O(d × n)**（d = 最大位數）。當 d 是常數時，Radix Sort 可達到 **O(n)**。

> ⚠️ **限制：** Radix Sort 只適用於整數或可拆位的資料，且需要額外的桶空間。

---

## Binary Search（二分搜尋）

在一堆數字中找到某一個目標數字。利用**二分法**，每次將搜尋範圍砍半，因此前提是**資料必須是有序的**。

```javascript
// Time Complexity: O(log n)
// Space Complexity: O(1)
function binarySearch(arr, target) {
  let left = 0;
  let right = arr.length - 1;
  while (left <= right) {
    const mid = Math.floor((left + right) / 2);
    if (arr[mid] === target) {
      return mid;
    } else if (arr[mid] < target) {
      left = mid + 1;
    } else {
      right = mid - 1;
    }
  }
  return -1;
}
```

---

## Binary Search Tree（二元搜尋樹）

BST 是一種樹狀資料結構，每個節點最多有兩個子節點，且滿足：**左子節點 < 父節點 < 右子節點**。

```javascript
// Time Complexity: O(log n) average, O(n) worst case
// Space Complexity: O(n) for the tree, O(1) per operation
class Node {
  constructor(value) {
    this.value = value;
    this.left = null;
    this.right = null;
  }
}

class BinarySearchTree {
  constructor() {
    this.root = null;
  }

  insert(value) {
    const newNode = new Node(value);
    if (!this.root) {
      this.root = newNode;
      return this;
    }
    let current = this.root;
    while (true) {
      if (value === current.value) return undefined;
      if (value < current.value) {
        if (current.left === null) {
          current.left = newNode;
          return this;
        }
        current = current.left;
      } else {
        if (current.right === null) {
          current.right = newNode;
          return this;
        }
        current = current.right;
      }
    }
  }

  find(value) {
    if (!this.root) return false;
    let current = this.root;
    let found = false;
    while (current && !found) {
      if (value < current.value) {
        current = current.left;
      } else if (value > current.value) {
        current = current.right;
      } else {
        found = true;
      }
    }
    return current;
  }
}
```

**使用範例：**

```javascript
const bst = new BinarySearchTree();
bst.insert(10);
bst.insert(6);
bst.insert(15);
bst.insert(3);
bst.insert(8);
bst.insert(20);

// 產生的樹長這樣：
//        10
//       /  \
//      6    15
//     / \     \
//    3   8    20

bst.find(8);   // 回傳 Node { value: 8, left: null, right: null }
bst.find(99);  // 回傳 false（找不到）

// insert 回傳 this（整棵樹），所以可以 chain
const bst2 = new BinarySearchTree();
bst2.insert(5).insert(3).insert(7).insert(1);
```

---

## Selection Problem（選擇問題）

**問題：** 找到一堆數字中**第 K 大**的數字。

### Approach 1：全部排序

1. 把所有數字放進一個 array
2. 把 array 由大到小排序
3. 回傳第 K 個位置的數字

```javascript
// Time Complexity: O(n log n)
// Space Complexity: O(n)
function selectionProblem(arr, k) {
  arr = arr.sort((a, b) => b - a); // JS sort 內部使用額外空間
  return arr[k - 1];
}
```

### Approach 2：維護 K 個候選人

只維護一個大小為 K 的候選陣列，不需要排序整個 array：

1. 從數字中取前 K 個元素，放進候選陣列
2. 把候選陣列由大到小排序
3. 逐一檢查剩餘元素，跟候選陣列中最小的比較
4. 如果比候選中最小的**還小** → 跳過
5. 如果比候選中最小的**還大** → 踢掉最小的，放入新元素，重新排序

```javascript
// Time Complexity: O((n - K) × K log K)
//   - 最多 (n-K) 次比較，每次可能重新排序 K 個元素
// Space Complexity: O(K)
function selectionProblem2(arr, k) {
  // Step 1: 取前 K 個
  let candidates = arr.slice(0, k);
  // Step 2: 由大到小排序
  candidates.sort((a, b) => b - a);
  // Step 3: 逐一比較剩餘元素
  for (let i = k; i < arr.length; i++) {
    if (arr[i] > candidates[k - 1]) {
      candidates[k - 1] = arr[i];       // 踢掉最小的
      candidates.sort((a, b) => b - a); // 重新排序
    }
  }
  // Step 4: 回傳第 K 大
  return candidates[k - 1];
}
```

> 💡 **何時選 Approach 2？** 當 K 遠小於 n 時，不需要把整個陣列都排好，只關心最大的 K 個就夠了。

---

## Recursive Algorithm（遞迴演算法）

遞迴的核心概念：**把大問題拆成相同結構的小問題**，一定要有 **Boundary Condition（終止條件）**，否則會無限遞迴。

### Recursive Factorial（遞迴階乘）

計算 n! 可遞迴為 `n × (n-1)!`，直到 `0! = 1` 為止。

```javascript
// Time Complexity: O(n)
// Space Complexity: O(n)
function factorial(n) {
  if (n === 0) {
    return 1;
  }
  return n * factorial(n - 1);
}
```

### Recursive Multiplication（遞迴乘法）

計算 `a × b` 可遞迴為 `a + (a × (b-1))`，直到 `b = 0` 為止。

**運作方式：** 以 `5 × 3` 為例：

```
5 × 3
= 5 + (5 × 2)
= 5 + 5 + (5 × 1)
= 5 + 5 + 5 + (5 × 0)  ← 終止條件，回傳 0
= 15
```

```javascript
// Time Complexity: O(b)
// Space Complexity: O(b)
function multiply(a, b) {
  if (b === 0) {
    return 0;
  }
  return a + multiply(a, b - 1);
}
```

---

## 參考資料

- [Lec01 資料結構 第一週課程](https://www.youtube.com/watch?v=3503j2L6qNA)