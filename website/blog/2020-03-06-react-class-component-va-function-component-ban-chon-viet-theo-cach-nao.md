---
title: [React] Class Component và Function Component, bạn chọn viết theo cách nào?
author: Hieu Nguyen
authorURL: https://twitter.com/hieunv8
authorFBID: 100006410080637
---

![react](https://images.unsplash.com/photo-1581276879432-15e50529f34b)

Trong thế giới React thì chắc hẳn ai cũng biết đến Class Component và Function Component. Tuy nhiên có thể có những hiểu nhầm về hai loại component này. Trong bài viết này tôi sẽ thử so sánh hai cách viết này để giúp bạn có thể lựa chọn viết theo cách nào. Chúng ta cùng bắt đầu nhé.

## Cú pháp

Khác nhau đầu tiên giữa `Class Component` và `Function Component` thể hiện ngay ở cách khai báo.

### Class Component

```js
import React, { Component } from 'react';

class TestComponent extends Components {
  // phương pháp này bắt buộc phải khai báo hàm để kết xuất mã HTML
  render() {
    return <div>TestComponent</div>;
  }
}
```

Cách khai báo này khá quen thuộc với các bạn có nền tảng lập trình hướng đối tượng (OOP). Với những bạn mới học React hoặc chuyển sang học React thì phương pháp tiếp cận này có vẻ phù hợp và dễ hiểu.

### Function Component

```js
import React from 'react';

export function TestComponent() {
  // phương pháp xem kết xuất mã HTML như là giá trị trả về của hàm
  return <div>TestComponent</div>;
}
```

Function component sử dụng cách tiếp cận khác đó là sử dụng pure function để khai báo component. Ban đầu function component được sử dụng để viết các component chỉ với mục đích kết xuất HTML mà thôi. Với các component với theo hướng tiếp cận này thì bạn sẽ không can thiệp được vào lifecycle của component. Do đó nó thướng được biết đến với tên gọi Stateless Component.

## Props

### Class Component

`props` trong Class Component được xem như giá trị truyển vào cho hàm khởi tạo class.

```js
import React, { Component } from 'react';

class TestComponent extends Components {
  constructor(props) {
    super(props); // bắt buộc phải có dòng này để gọi hàm khởi tạo của class cha nhé
  }

  render() {
    return <div>TestComponent</div>;
  }
}
```

### Function Component

`props` trong Function Component thì được xem như là giá trị truyền vào hàm pure function khi định nghĩa component.

```js
import React from 'react';

export function TestComponent(props) {
  return <div>TestComponent</div>;
}
```

Định nghĩa `defaultProps` và `propTypes` thì không có sự khác biệt giữa Class Component và Function Component.

```js
TestComponent.defaultProps = {};

TestComponent.propTypes = {};
```

## State

Trước khi React Hooks ra đời thì như đã nói ở trên Function Component con được biết đến với tên gọi Stateless Component. Nghĩa là nó không có state. Khi React Hooks ra đời thì Function Component cũng có `state` của riêng nó.

### Class Component

`state` trong Class Component dược định nghĩa như sau:

```js
import React, { Component } from 'react';

class TestComponent extends Components {
  constructor(props) {
    super(props);
    // khởi tạo giá trị state
    this.state = { isLoading: false };
  }

  render() {
    return <div>TestComponent</div>;
  }
}
```

Khi muốn thay đổi giá trị `state` bạn gọi phương thức `setState` của component:

```js
this.setState((state) => ({ isLoading: true }));
```

### Function Component

`state` trong Function Component được định nghĩa như sau:

```js
import React, { useState } from 'react';

export function TestComponent(props) {
  // giá trị khởi tạo state được truyền vào trong useState hook
  const [state, setState] = useState({ isLoading: false });

  return <div>TestComponent</div>;
}
```

Các bạn để ý hàm `useState` trả về giá trị của component `state` trong biến `state` và hàm `setState`. Khi muốn thay đổi giá trị của state thì bạn có thể gọi hàm `setState`.

```js
setState({ isLoading: true });
```

## Component Lifecycle

Với Class component các bạn sẽ thấy component lifecycle khá rõ ràng với các hàm như `componentDidMount`, `componentDidUpdate`. Function Component thì không như vậy, toàn bộ việc sử lý lifecycle đều thông qua `useEffect` hook.

```js
// componentDidMount
useEffect(() => {
  return () => {}; // componentWillUnmount
}, []);
```

```js
// componentDidUpdate
useEffect(() => {
  return () => {}; // componentWillUnmount
}, [state]);
```

Như các bạn thấy thì `componentDidMount` và `componentDidUpdate` không chỉ định rõ khi nào thì hàm được gọi. Việc gọi hàm thì tự chúng ta hiểu dựa theo lifecycle của React component thôi. Với Function Component và `useEffect` thì khác, bạn có thể thấy `[]` và `[state]` chị định rõ ràng đối tượng phụ thuộc mà khi chúng thay đổi thì hàm truyển vào `useEffect` sẽ được gọi. Có vẻ như nó lại tường mình hơn là các hàm lifecycle trong class Component.

Sau sự có mặt của TypeScript thì có lẽ Class Component và React Hooks thì có lẽ bạn Class Component chiếm ưu thế tuyệt đối. Thời điểm đó anh em thi nhau viết Class Component với TypeScript và tôi cũng thế :). Ngay đến cả facebook cũng support TypeScript với create-react-app nữa cơ mà. Thế nhưng khi React Hooks xuất hiện thì có vẻ gió đã đổi chiều, việc xử lý state và lifecycle với hook có vẻ đơn giản hơn rất nhiều. Các bạn thì thấy thế nào. Bạn sẽ chọn cách nào với dự án của mình?
