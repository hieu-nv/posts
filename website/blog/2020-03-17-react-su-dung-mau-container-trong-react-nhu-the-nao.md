---
title: [React] Sử dụng mẫu container trong React như thế nào?
author: Hieu Nguyen
authorURL: https://twitter.com/hieunv8
authorFBID: 100006410080637
---

![react](https://images.unsplash.com/photo-1581276879432-15e50529f34b)

Chắc hẳn ai cũng biết tới nguyên lý đầu tiên của SOLID đó là trách nhiệm đơn nhất. Nghĩa là mỗi một đối tượng chỉ làm một nhiệm vụ duy nhất. Đơn giản vì với tính đơn nhất chúng ta sẽ dễ dàng kiểm soát mã nguồn hơn (cho cả người code lần người review). Chưa kể tới việc với tính đơn nhất thì mã nguồn của bạn cũng trở nên trong sáng hơn. Với anh em lập trình React cũng vậy, bài toán khá đau đầu là làm sao tách logic ra khỏi phần mã nguồn kết xuất HTML. Cách làm như vậy giúp chúng ta thay vì viết một React Component để là cả hai việc thì bây giờ chúng ta sẽ viết hai React Component để làm hai nhiệm vụ khác nhau. Trong bài viết này tôi sẽ hướng dẫn các bạn sử dụng mấu container để làm việc đó.

## Container component là gì?

Chúng ta không có một định nghĩa chính xác nào về container component cả, container component đơn giản là một React Component làm nhiệm vụ goi API để lấy dữ liệu và sau đó truyền nó xuống một React Component để kết xuất mã HTML.

## Định nghĩa container

Để các bạn dễ hình dung tôi sẽ viết thử một container, container này sẽ thực hiện lấy thông tin của octocat

`src/containers/OctocatContainer.jsx`

```jsx
import React, { useState, useEffect } from 'react';
import { Octocat } from '../components/Octocat';
export function OctocatContainer() {
  const [octocat, setOctocat] = useState(null);

  useEffect(() => {
    async function getOctocat() {
      // gọi hàm fetch với phướng thực mặc định là GET
      const response = await fetch('https://api.github.com/users/octocat');
      const body = await response.json();
      // sử dụng React Hook để thông tin octocat vào state
      setOctocat(body);
    }

    getOctocat();
  }, []); // tham số mảng rỗng thể hiện hàm effect không phụ thuộc vào đối tượng nào, nó tương đương với hàm componentDidMount

  return <Octocat {...octocat} />;
}
```

Trong ví dụ trên tôi có sử dụng thêm [JSX spread attributes](https://reactpatterns.com/#jsx-spread-attributes) để tránh việc phải khai báo `props` cho Octocat component. Mẫu này đặc biệt có ý nghĩa đối với các component mà bạn cần khai báo nhiều `props`.

## Định nghĩa component để kết xuất mã HTML

Sau khi có dữ liệu rồi thì chúng ta sẽ kết xuất nó ra HTML như sau:

`src/components/Octocat/Octocat.jsx`

```jsx
import React from 'react';

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
```

Như các bạn thấy, component trên chỉ làm nhiệm vụ bind dữ liệu lên màn hình mà thôi.

Cùng xem kết quả nhé :)

![octocat](https://s3-ap-southeast-1.amazonaws.com/techover.storage/wp-content/uploads/2020/03/17154053/Screen-Shot-2020-03-17-at-3.39.28-PM.png)

Cám ơn các bạn đã theo dõi bài viết. Đây là bài đầu tiên trong loạt bài React Patterns mình sẽ chia sẻ với các bạn. Các bạn cùng chờ các bài tiếp theo nhé.
