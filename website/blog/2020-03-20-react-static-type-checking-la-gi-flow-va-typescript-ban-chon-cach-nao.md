---
title: [React] Static Type Checking là gì? Flow và TypeScript, bạn chọn cách nào?
author: Hieu Nguyen
authorURL: https://twitter.com/hieunv8
authorFBID: 100006410080637
---

![react](https://images.unsplash.com/photo-1581276879432-15e50529f34b)

Static Type Checking và Runtime Type Checking là hai thuật ngữ ban đâu cảm giác có vẻ không quen thuộc cho nhưng nó lại lai thứ có lẽ các bạn đang dùng hàng ngày. Như các bạn biết thì JavaScript là ngôn ngữ định kiểu động, nghĩa là kiểu dữ liệu chỉ được xác định tại thời điểm thực thi. Do đó khi lập trình chúng ta không cần xác định kiểu dữ liệu cho biến. Điều này vô hình chung làm cho mã nguồn của chúng ta trở nên khó đọc hơn và khó debug hơn nữa. Chính vì vậy việc xác định kiểu dữ liệu giúp cho chúng ta kiểu soát và debug dễ dàng hơn. Trong bài viết này tôi sẽ chia sẻ với các bạn cách công đồng đang làm để hỗ trợ hai kiểu kiểm tra kiểu dữ liệu này.

## Static Type Checking là gì?

Hiện nay có hai xu hướng để kiểm tra kiểu dữ liệu tĩnh là [Flow](https://flow.org/) và [TypeScript](https://www.typescriptlang.org/). Static Type Checking sẽ phân tích mã nguồn tĩnh trước khi biên dịch mã nguồn sang mã JavsScript(ES5) là mã nguồn mà trình duyệt có thể đọc được. Vậy thì chúng khác nhau ở điểm nào?

### Flow

Flow là một phần mở rộng của [Babel](https://babeljs.io/) cung cấp cho bạn việc kiểm tra kiểu dữ liệu tĩnh bằng cách sử dụng các chỉ thị bằng comment mà bạn thêm vào mã nguồn.

Ví dụ cách sử dụng flow như bên dưới:

```js
// @flow
function square(n: number): number {
  return n * n;
}

square('2'); // Error!
```

### TypeScript

TypeScript là ngôn ngữ dựa trên JavaScript được Microsoft phát triển. TypeScript khác với Flow là nó không phân tích mã nguồn dự trên các chỉ thị bằng comment. Kiểu dữ liệu được hỗ trợ từ trong ngôn ngữ luôn. Dữ liệu được kiểm tra tại thời điểm phân tích cú pháp của chương trình dịch. Ngoài ra thì TypeScript hỗ trợ rất mạnh OOP nên nó rất phù hợp với các bạn quen thuộc với các ngôn ngữ định kiểu như Java và C#.

```js
const add = (x: number, y: number) => {
  return x + y;
};
```

## Sử dụng Static Type Checking có lợi ích gì?

- Khi sử dụng Static Type Checking bạn được hỗ trợ việc kiểm soát dữ liệu tại thời điểm viết mã nguồn, tất cả việc gán hoặc gọi hàm với kiểu dữ liệu không phù hợp sẽ được phát hiện tại thời điểm này.

- IDE hỗ trợ việc kiểm soát kiểu dữ liệu giúp bạn viết code nhanh hơn và sai sốt ít hơn.

- Mã nguồn dễ đọc hơn cho người mới.

## Bạn mất gì khi sử dụng Static Type Checking?

Static Type Checking mang những lợi ích nhất định trong việc kiểm soát kiểu dữ liệu, tuy nhiên nó cũng có những nhiểm điểm:

- Bạn phải viết code nhiều hơn thì mới kiểm soát được kiểu dữ liệu, việc này đương nhiên sẽ tốn công sức. Tuy nhiên so với việc mất thời gian vào debug thì có lẽ nó vẫn tốt hơn.

- Bạn vẫn không kiểm soát được kiểu dữ liệu tại thời điểm chạy chương trình, lý do là nếu chương trình của bạn có liên kết với API thì không có gì đảm bảo kiểu dữ liệu của bạn chạy đúng với backend cả. Đương nhiên cái này là vấn đề không thể giải quyết được rồi. Tôi sẽ hướng dẫn các bạn giải quyết vấn đề này trong bài viết Runtime Type Checking sau nhé.

## Kết luận

Đây chỉ là kết luận mang tính cá nhân của mình :). Các bạn tự suy ngẫm để đưa ra lựa chọn phù hợp nhé.

- Việc viết theo Flow có cảm giác không tự nhiên lắm, thường xuyên phải thêm comment khiến bạn khá mất thời gian. Thêm một vấn đề nữa là lúc viết code sử dụng Flow khá tốn tài nguyên của máy. Về vấn đề này thì mình thấy TypeScript có vẻ ổn hơn.

- TypeScript không đơn giản là một sự mở rộng của JavaScript, Microsoft đã thêm vào nhiều thứ hơn thế và nó cũng không dựa trên các tiêu chuẩn của ES6, ES7, ... (có support ES6, ES7, ... nhưng có những thêm vào nhiều thứ khác nữa). Về điểm này thì Flow có vẻ tốt hơn.

- Sau sự ra đời của React Hooks thì mọi thế mạnh của TypeScript gần như không còn nữa so với Flow, bản thân mình cũng không dùng TypeScript nữa.

- Một điểm quan trong nữa là cả TypeScript và Flow đều không thể xác mình kiểu dữ liệu lúc thực thi nên bạn vẫn cần Runtime Type Checking.

Các bạn có dùng Static Type Checking không? Mình thì đã không dùng nữa và chuyển sang xác mình toàn bộ bằng Runtime Type Checking rồi. Thời gian tốn cho debug cũng không phát sinh nhiều so với việc phải viết mã nguồn xác định kiểu dữ liệu.

Cảm ơn các bạn đã theo dõi bài viết. Các bạn cùng chờ bài viết mình hướng dẫn về Runtime Type Checking nhé.
