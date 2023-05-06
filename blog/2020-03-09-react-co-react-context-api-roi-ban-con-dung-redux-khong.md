<!-- ---
title: [React] Có React Context API rồi, bạn còn dùng Redux không?
author: Hieu Nguyen
authorURL: https://twitter.com/hieunv8
authorFBID: 100006410080637
--- -->

![react](https://images.unsplash.com/photo-1581276879432-15e50529f34b)

State Management là một trong những vấn đề tồn tại ở bất kỳ ứng dụng nào viết bằng React. Redux là một trong những công cụ như thế. Nó từng một thời là xu hướng được tất cả các ứng dụng viết bằng React sử dụng. Và mỗi khi nhắc đến React thì người ta cũng nhắc đến Redux luôn. Không những thế các biến thể của nó cũng được implement cho Angular như ngrx/store hay vuex cho VueJS. Tuy nhiên với ứng dụng ngày càng cồng kềnh thì việc định nghĩa và quản lý state ngày càng trở nên vất vả hơn. Trải qua một thời gian khá dài với Redux thì mãi tới gần đây Facebook mới cho ra đời Context API để giải quyết các vấn đề còn tồn tại với Redux. Trong bài viết này tôi sẽ đưa ra một số so sánh để giúp các bạn có thể chuyển từ ứng dụng viết bằng Redux sang sử dụng Context API.

## Redux

Khi bạn muốn định nghĩa `state` với Redux thì mọi thứ có vẻ khá phức tạp. Bước đầu tiên bạn cần làm là phải khai báo đăng ký Redux Store với React

- Khai báo Redux Store:

`src/store/index.js`

```js
import { createStore, applyMiddleware, compose } from "redux";
import thunk from "redux-thunk";
import { createLogger } from "redux-logger";
// import reducer để thay đổi giá trị state
import createRootReducer from "../reducers";

// kiểm tra sự tồn tại của Redux Dev Tools và add nó vào phục vụ quá trình debug
const composeEnhancers =
  typeof window === "object" && window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__
    ? window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__({})
    : compose;

export default (initialState) => {
  const { NODE_ENV } = process.env;

  let store;
  if (NODE_ENV === "development") {
    // khởi tạo store sử dụng API của Redux, ở đây có sử dụng thêm redux-thunk để đơn giản hoá việc xử lý HTTP API
    store = createStore(
      createRootReducer(),
      initialState,
      composeEnhancers(applyMiddleware(thunk, createLogger()))
    );

    // cấu hình hot reload để mỗi khi bạn thay đổi *.js thì nó tự động được load lại
    if (module.hot) {
      module.hot.accept("../reducers", () => {
        const nextRootReducer = require("../reducers").default;
        store.replaceReducer(nextRootReducer);
      });
    }
  } else {
    // chỗ này để định nghĩa store trên môi trường production
    store = createStore(
      createRootReducer(),
      initialState,
      composeEnhancers(applyMiddleware(thunk))
    );
  }
  return store;
};
```

Có vẻ khá nhiều việc cần phải làm với Redux. Nhưng vẫn chưa hết đâu :)

- Đăng ký Redux Store vào ứng dụng React:

`src/index.js`

```js
// sử dụng thư viện này để đăng ký Redux với React nhé
import { Provider } from "react-redux";
// import định nghĩa store được khai báo bên trên
import createStore from "./store";
//khởi tạo Redux store
const store = createStore({});

// render ứng dụng React
render(
  // kết nối Redux store vào React
  <Provider store={store}>
    {
      // mã nguồn ứng dụng được khái báo ở đây. Trong này bạn có thể truy cập vào state của Redux thông qua hàm connect
    }
  </Provider>,
  document.getElementById("app-container")
);
```

- Định nghĩa `reducer` để thay đổi giá trị `state`

```js
const counterReducer = (state = { count: 0 }, action) => {
  switch (action.type) {
    case "INCREMENT":
      return { count: count + 1 };
    case "DECREMENT":
      return { count: count - 1 };
    default:
      return state;
  }
};
export default reducer;
```

- Kết nỗi Redux Store với React Component

```js
class Counter extends Component {
  render() {
    // Trạng thái và các hàm được đăng ký bởi hàm connect bên dưới
    const { count, increment, decrement } = this.props;

    return (
      <div>
        <button onClick={decrement}>-</button>
        <span>{count}</span>
        <button onClick={increment}>+</button>
      </div>
    );
  }
}

export default connect(
  // map giá trịg trong Redux Store vào props của component
  (state) => ({ count: state.counter.count }),
  //  đăng ký các hàm sau vào props
  (dispatch) => ({
    // gọi action INCREMENT để tăng gía trị count.
    increment: () => dispatch({ type: "INCREMENT" }),
    // gọi action DECREMENT để tăng gía trị count
    decrement: () => dispatch({ type: "DECREMENT" }),
  })
)(Counter);
```

Có vẻ phải viết code khá nhiều và nhiều đoạn không tường minh nếu các bạn không nắm được [Flux Architecture](https://scotch.io/tutorials/getting-to-know-flux-the-react-js-architecture). Redux là một biến thể của Flux nên các bạn có thể tham khảo thêm [Flux Architecture](https://scotch.io/tutorials/getting-to-know-flux-the-react-js-architecture) nhé. Bây giờ chúng ta thử với Context API thì sao nhé.

## Context API

Với Context API bạn cũng cần một nơi để lưu trũ trạng thái của ứng dụng. Tuy nhiên Context API không định nghĩa tất cả vào trong một như Redux. Các bạn có thể quản lý các `state` khác nhau bằng các `context` khác nhau. Do đó với mỗi `state`, các bạn cần định nghĩa một `context` riêng cho nó.

- Định nghĩa context

`src/contexts/counter.js`

```js
import React, { useContext } from "react";

export const CounterContext = React.createContext();

export const useCounter = () => useContext(LoadingContext);
```

- Định nghĩa provider để khai báo `state` và cung cấp các hàm thay đổi giá trị `state`

```js
import React, { useState } from 'react';
import PropTypes from 'prop-types';
import { CounterContext } from '../../contexts/counter';

export function CounterProvider(props) {
  const [count, setCount] = useState(0);

  return (
    <CounterContext.Provider
      value={{
        count: count,
        decrement: () => setCount(count - 1)
        increment: () => setCount(count + 1),
      }}>
      {props.children}
    </CounterContext.Provider>
  );
}

CounterContext.propTypes = {
  children: PropTypes.node
};
```

- Đăng ký `CounterProvider` trong ứng dụng React

```js
// render ứng dụng React
render(
  // kết nối Redux store vào React
  <CounterProvider>
    {
      // sử dụng useCounter để lấy giá trị count và các hàm increment, decrement được provider truyển xuống trong giá trị value khai báo ở trên
    }
  </CounterProvider>,
  document.getElementById("app-container")
);
```

- Sử dụng `useCounter` trong React Component

```js
export function Counter() {
  const [count, increment, decrement] = useCounter();

  return (
    <div>
      <button onClick={() => decrement()}>-</button>
      {count}
      <button onClick={() => increment()}>+</button>
    </div>
  );
}
```

Với React Context API mọi thứ trở nên đơn giản hơn bao giờ hết. Bạn vẫn còn dùng Redux chứ? Sau bài này bạn sẽ chuyển sang Context API chứ?
