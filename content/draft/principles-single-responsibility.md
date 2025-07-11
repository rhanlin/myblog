+++
title = "元件開發原則: 單一職責原則(Single Responsibility Principle)"
description = ""
date = 2023-11-09 21:00:00
author = "Rhan0"
tags=["conventions"]
weight= 2
+++

### 一個 類/模塊/元件 應該有且只有一個改變的原因。

*(如果使用到多於一個的動機去改變一個 類/模塊/元件，那麼這個 類/模塊/元件 就具有多於一個的職責，不符合 SRP 原則)*


### Component

```jsx
const getTitle = ({ type, title }) => {
  switch (type) {
    case 'a':
      return 'A';
    case 'b':
      // ❌ 控制項變成兩個
      if (title) return `B-${title}`;
      else return 'B';
    case 'c':
      return 'C';
    default:
      return null;
  }
};

const getContent = (type) => {
  switch (type) {
    case 'a':
      return '是否刪除該筆資料？';
    case 'b':
      return '是否更新該筆資料？';
    case 'c':
      return '請確認內容無誤。';
    default:
      return null;
  }
};

const getActions = ({ type, actions, onConfirm, onCancel }) => {
  switch (type) {
    case 'a':
    case 'b':
      return (
        <div>
          {actions.map(({ label, onClick }) => (
            <Button onClick={onClick}>{label}</Button>
          ))}
        </div>
      );
    case 'c':
      return (
        <div>
          <Button onConfirm={onConfirm}>確認</Button>
          <Button onCancel={onCancel}>取消</Button>
        </div>
      );
    default:
      return null;
  }
};

// ❌ 把有不同的改變原因的事物耦合在一起的設計是糟糕的。
// 太多關注點需要留意，getTitle, getContent, getActions
const Dialog = (props) => (
  <div>
    <DialogTitle>{getTitle(props)}</DialogTitle>
    <DialogContent>{getContent(props.type)}</DialogContent>
    <DialogFooter>{getActions(props)}</DialogFooter>
  </div>
);

// ✅ 應該把模組切開來，而非把所有判斷都寫在同一包裡面
const Dialog = (props) => {
  const { type } = props;

  // 可使用 switch case 區分模組
  switch (type) {
    case 'a':
      return <ADialog {...props} />;
    case 'b':
      return <BDialog {...props} />;
    case 'c':
      return <CDialog {...props} />;
    default:
      return null;
  }
};

// ✅ 也可透過物件方式區分
const dialogs = {
  a: (props) => <ADialog {...props} />,
  b: (props) => <BDialog {...props} />,
  c: (props) => <CDialog {...props} />,
};

const Dialog = (props) => {
  const { type } = props;

  return dialogs[type](props) || null;
};
```


### Function

```jsx
// ❌ 一個 function 只能有一個控制項
const getRoutes = (type, name, subRoute) => {
  switch (type) {
    case 'a':
      // ❌ 此時控制項就變成 type, name, subRoute
      if (subRoute === 'project') {
        return '/detail';
      }

      if (subRoute === 'project' && name === 'Gary') {
        return `/${subRoute}/detail/${name}`;
      }

      return '/a/project/detail';
    case 'b':
      return '/b/home';
    case 'c':
      return '/c/settings';
    default:
      return '';
  }
};

// ✅ 特殊規格獨立模組化，依照複雜度決定做成 function/class
const getTheARoutes = (name, subRoute) => {
  if (subRoute === 'project') {
    return name === 'Gary' ? `/${subRoute}/detail/${name}` : '/detail';
  }

  return '/a/project/detail';
};

const getRoutes = (type, ...rest) => {
  switch (type) {
    case 'a':
      // ✅ 把該邏輯抽出來獨立成一個 function/module/class
      return getTheARoutes(...rest);
    case 'b':
      return '/b/home';
    case 'c':
      return '/c/settings';
    default:
      return '';
  }
};
```