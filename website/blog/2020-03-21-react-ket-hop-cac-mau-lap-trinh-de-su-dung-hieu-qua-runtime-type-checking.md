---
title: [React] Kết hợp các mẫu lập trình để sử dụng hiệu quả Runtime Type Checking
author: Hieu Nguyen
authorURL: https://twitter.com/hieunv8
authorFBID: 100006410080637
---

![introduction](https://s3-ap-southeast-1.amazonaws.com/techover.storage/wp-content/uploads/2020/03/21154244/photo-1484154218962-a197022b5858.jpeg)

Như các bạn đã biết, Static Type Checking giúp chúng ta kiểm soát kiểu dữ liệu tốt hơn khi viết mã nguồn. Các bạn có thể sử dụng Flow hoặc TypeScript đều được. Tuy nhiên khi dữ liệu được liên kết với API chẳng hạn, khi đó bạn không thể kiểm soát được giá trị được truyền vào cho biến nào đó. Do đó chúng ta vẫn cần thêm một bước nữa để hạn chế các lỗi tiềm ẩn bằng Runtime Type Checking. Trong bài viết này tôi sẽ hướng dẫn các bạn sử dụng Runtime Type Checking như thế nào cho hiệu quả.

## Runtime Type Checking là gì?

Runtime Type Checking là quá trình xác minh lại kiểu dữ liệu của giá trị truyền vào cho biến có đúng với kiểu dữ liệu mong muốn hay không. Trong ứng dụng React khi bạn khai báo `propTypes` cho component nào đó chính là lúc bạn đang định nghĩa kiểu dữ liệu mong muốn. Tại thời điểm thực thi các định nghĩa này sẽ được kiểm tra giúp bạn kiểm soát việc binding dữ liệu từ API vào component có tương thích với nhau hay không.

## Định nghĩa `propTypes`

React cung cấp gói `prop-types` giúp bạn định nghĩa các thuộc tính cần cho Runtime Type Checking. Trước đây thì gói này nằm trong React, hiện tại thì nó đã tách ra thành một gói riêng rồi. Các bạn khai báo `propTypes` như sau:

```js
import React from 'react';
import PropTypes from 'prop-types';

export function Octocat(props) {
  const { login, avatar_url, name } = props;
  return (
    <ul>
      <li>{login}</li>
      <li>{name}</li>
      <li>
        <img src={avatar_url} alt={login} />
      </li>
    </ul>
  );
}

Octocat.propTypes = {
  login: PropTypes.string,
  avatar_url: PropTypes.string,
  name: PropTypes.string
};
```

Trong quá trình thực thi nếu dữ liệu được truyền vào component không đúng với định nghĩa `propTypes` bạn sẽ nhận được một thông báo tương tự như sau:

![prop-types](https://s3-ap-southeast-1.amazonaws.com/techover.storage/wp-content/uploads/2020/03/21153050/Screen-Shot-2020-03-21-at-3.30.17-PM.png)

## Kết hợp với Spread Attributes

Bạn có để ý rằng khi chúng ta khai báo giá trị truyền vào trong props thì khi sử dụng component này bạn phải khai báo như sau:

```js
<Octocat login="abc" avatar_url="def" name="xyz" />
```

Nếu số lượng thuộc tính trong component này tăng lên thì việc viết mã nguồn sẽ khá là vất vả. Rất may là trong JSX cung cấp cho bạn tính năng Spead Attributes và bạn có thể viết code như sau:

```js
const data = {login: "abc" avatar_url: "def" name: "xyz"};
<Octocat {...data} />
```

Toán tử `...` được gọi là toán tử `spead`.

## Kết hợp với Destructuring Props

Các bạn có thấy rằng khi khai báo `Octocat` component tôi đã truyền `props` như làm tham số của một hàm và sau đó mới gán lại vào các biến. Với Destructuring Props bạn có thể viết mã nguồn đơn giản hơn như sau:

```js
import React from 'react';
import PropTypes from 'prop-types';

export function Octocat({ login, avatar_url, name }) {
  return (
    <ul>
      <li>{login}</li>
      <li>{name}</li>
      <li>
        <img src={avatar_url} alt={login} />
      </li>
    </ul>
  );
}

Octocat.propTypes = {
  login: PropTypes.string,
  avatar_url: PropTypes.string,
  name: PropTypes.string
};
```

Tài liệu tham khảo:

- [JSX in Depth](https://reactjs.org/docs/jsx-in-depth.html)
- [React Patterns](https://reactpatterns.com/)

Cám ơn các bạn đã theo dõi bài viết. Hy vọng bài viết sẽ giúp các bạn viết mã nguồn tốt hơn.
