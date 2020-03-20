---
title: [AWS] Serverless và SAM, bạn chọn dùng cái nào?
author: Hieu Nguyen
authorURL: https://twitter.com/hieunv8
authorFBID: 100006410080637
---

Mình đã viết khá nhiều bài sử dụng Serverless, tại sao mình lại viết bài này. Thực ra mình cũng mới bắt đầu làm AWS Lambda được một thời gian ngắn. Dự án đầu tiên mình làm Lambda thì đã các bạn đi được đã chọn Serverless để phát triển. Dự án thứ hai mình làm với AWS Lambda thì khách hàng đưa cho mình bộ mã nguồn đã được cấu hình sử dụng Serverless. Mọi thứ đều có vẻ ổn cho đến một ngày mình quyết định thử debug Lambda bằng Visual Studio Code. Mọi thứ trở nên phức tạp và mình tìm thấy SAM, dường như nó đã giải quyết vấn đề của mình nên mình quyết định viết bài này để cho các bạn nếu mới đến với thế giới AWS thì có thể dễ dàng lựa chọn thứ mình cần.

## Chọn [Serverless](https://github.com/dherault/serverless-offline) hay [SAM](https://docs.aws.amazon.com/serverless-application-model/index.html)?

Dự án đầu tiên mình dùng Serverless và viết bằng JavaScript, mọi thứ đều ổn vì mình chỉ dùng Serverless có kết hợp với Serverless Offline để chạy các hàm Lambda API trên máy tính cá nhân được. Việc debug cũng không gặp trở ngại gì do Serverless Offline có hỗ trợ debug. Thế nhưng đến dự án tiếp theo, ngôn ngữ được chọn làm môi trường thực thi là Python và mình thực sự gặp khó khăn. Mình vẫn có thể chạy được các hàm Lambda trên máy tính cá nhân nhưng không thể debug đươc. Và thế là mình bắt đầu tìm hiểu để giải quyết vấn đề này. Rồi mình tìm thấy SAM và mọi thứ dường như được giải quyết.

### Ngôn ngữ nào được hỗ trợ?

- Serverless Offline hỗ trợ những ngôn ngữ sau:
  - Python
  - Ruby
  - Node
- SAM hỗ trợ nhưng ngôn ngữ sau:
  - Python
  - Ruby
  - Node
  - Java
  - .NET Core
  - ...

### Được hỗ trợ như thế nào?

- Serverless Offline là plugin được cá nhân phát triển. Nó không phải gói được hỗ trợ chính thức từ AWS.

- SAM được hỗ trợ chính thức từ AWS.

### Hỗ trợ debug như thế nào?

- Serverless Offline chỉ hỗ trợ debug với Node.

- SAM thì hiện tại theo mình tìm hiểu tài liệu chính thức thì đang hỗ trợ debug cho:
  - Python
  - Ruby
  - Node

### Môi trường thực thi

- Serverless chạy trực triếp trên máy host.
- SAM thì sử dụng container trong docker để thực thi.

Các bạn có thể tham khảo hướng dẫn sử dụng Serverless ở các tài liệu sau nhé:

- [Mô phỏng AWS Lambda & API Gateway bằng Serverless Offline](https://magz.techover.io/2020/03/01/mo-phong-aws-lambda-api-gateway-bang-serverless-offline/)
- [Sử dụng API Gateway Lambda Authorizers với JWT như thế nào?](https://magz.techover.io/2020/03/20/aws-su-dung-api-gateway-lambda-authorizers-voi-jwt-nhu-the-nao/)
- [Lambda và DynamoDB Streams không còn khó nữa!](https://magz.techover.io/2020/03/15/aws-lambda-va-dynamodb-streams-khong-con-kho-nua/)

### Kết luận

SAM dường như có những lợi thế hơn hẳn so với Serverless. Nếu bạn quyết định phát triển bằng Node thì bạn sẽ không gặp nhiều khó khăn khi dùng Serverless hay SAM. Nếu bạn chọn một môi trường thực thi khác như Python hay Ruby hay bất kỳ môi trường nào khác thì lựa chọn SAM sẽ là quyết định sáng suốt hơn đấy. Mình sẽ hướng dẫn các bạn sử dụng SAM trong loạt bài viết về SAM sau nhé.
