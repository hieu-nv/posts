---
title: [React] Tôi đã làm toast component như thế nào?
author: Hieu Nguyen
authorURL: https://twitter.com/hieunv8
authorFBID: 100006410080637
---

![react](https://images.unsplash.com/photo-1581276879432-15e50529f34b)

Với bất kỳ ứng dụng nào việc hiển thị thông báo lỗi là không thể thiếu được. Các bạn có thể có rất nhiều cách làm để hiển thị được các thông báo lỗi này. Trong bài viết này tôi xin chia sẻ cách làm của tôi để các bạn cùng tham khảo nhé.

Mục đích của tôi khi viết component này phải đảm bảo nó dễ dùng như hàm `window.alert()` vậy. Trong bài viết này tôi sẽ sử dụng [Material UI](https://material-ui.com/) để hiển thị thông báo lỗi.

## Toast Context

Vì chúng ta cần hàm hiển thị thông báo lỗi có thể sử dụng như `window.alert()` nên chúng ta cần khai báo một nơi lưu trữ để từ đó chúng ta có thể gọi được ở bất kỳ đâu trong ứng dụng. Có nhiều bạn sẽ nghĩ tới Redux nhưng trong bài viết này tôi sẽ sử dụng React Context API để làm việc đó. Chúng ta bắt đầu bằng việc khai báo `ToastContext` nào.

`src/contexts/toast.js`

```js
import React, { useContext } from 'react';

export const ToastContext = React.createContext();

export const useToast = () => useContext(ToastContext);
```

## Toast Provider

Trong React Context API có định nghĩa Provider để khi có sử thay đổi trong component thì các component bên trong có thể nhận biết sử thay đổi đó và phản anh kết quả thay đổi lên màn hình. Ở đây chúng ta cần hiển thị message lỗi mỗi khi có component nào đó thông báo về việc hiển thị message lỗi.

```jsx
import React, { useState } from 'react';
import { Snackbar } from '@material-ui/core';
import { Alert } from '@material-ui/lab';
import { ToastContext } from '../contexts/toast';

export const ToastProvider = (props) => {
  const { children } = props;
  const [state, setState] = useState({ isOpen: false });

  const show = (message) => {
    setState({ isOpen: true, message });
  };

  const hide = () => setState({ isOpen: false });

  const error = (message) => {
    show({ type: 'error', text: message });
  };

  const warn = (message) => {
    show({ type: 'warning', text: message });
  };

  const info = (message) => {
    show({ type: 'info', text: message });
  };

  const success = (message) => {
    show({ type: 'success', text: message });
  };
  const { isOpen, message } = state;
  return (
    <ToastContext.Provider
      value={{
        error: error,
        warn: warn,
        info: info,
        success: success,
        hide: hide
      }}>
      {children}
      {message && (
        <Snackbar open={isOpen} autoHideDuration={6000} onClose={hide}>
          <Alert elevation={6} variant="filled" onClose={hide} severity={message.type}>
            {message.text}
          </Alert>
        </Snackbar>
      )}
    </ToastContext.Provider>
  );
};
```

## Sử dụng các hàm `error`, `warn`, `info`, `success` để hiển thị thông báo

Để sử dụng được các hàm này trong ứng dụng bạn cần khai báo nó trong component chính của ứng dụng

```jsx
import React from 'react';
import { ToastProvider } from './providers/ToastProvider';
import { useToast } from './contexts/toast';
import './App.css';

function ButtonList() {
  const { error, warn, info, success } = useToast();
  return (
    <>
      <button onClick={() => error('error message!')}>error</button>
      <button onClick={() => warn('warn message!')}>warn</button>
      <button onClick={() => info('info message!')}>info</button>
      <button onClick={() => success('success message!')}>success</button>
    </>
  );
}

function App() {
  return (
    <ToastProvider>
      <ButtonList />
    </ToastProvider>
  );
}
export default App;
```

Cám ơn các bạn đã theo dõi bài viết. Hy vọng bài viết đã cung cấp cho các bạn thêm một các để hiển thị thông báo tốt hơn cho ứng dụng của bạn.
