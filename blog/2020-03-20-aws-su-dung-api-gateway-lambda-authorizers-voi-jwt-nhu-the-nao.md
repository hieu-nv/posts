<!-- ---
title: [AWS] Sử dụng API Gateway Lambda Authorizers với JWT như thế nào?
author: Hieu Nguyen
authorURL: https://twitter.com/hieunv8
authorFBID: 100006410080637
--- -->

Một trọng những vấn đề quan trọng trong các dự án đó là điều khiển quyền truy cập. Với các ứng dụng xây dựng trên nền tảng AWS việc điều khiển truy cập cũng phức tạp hơn. Trong bài viết này tôi sẽ hướng dẫn các bạn cách tôi đã làm để điểu khiển truy cập với các API sử dụng API Gateway Lambda Authorizers.

## Luồng xác thực Lambda Authorizer

Luồng xác thực của Lambda Authorizer được minh hoạ trong hình sau:

![auhorizer](https://s3-ap-southeast-1.amazonaws.com/techover.storage/wp-content/uploads/2020/03/20165337/custom-auth-workflow.png)

Các bược xác thực như sau:

1. Máy khách gửi yêu cầu lên API Gateway API có kèm theo Bearer Token.
2. API Gateway kiểm tra cấu hình authorizer đã được cấu hình tương ứng với hàm Lambda. Nếu nó tồn tại thì Lambda Authoirizer sẽ được gọi.
3. Lambda Authorizer sẽ thực hiện xác thực bằng Bearer Token đã được gửi lên.
4. Nếu việc gọi Lambda Authrorizer thực hiện thành công, hàm Lambda sẽ trả về thông tin chứa chính sách IAM và thông tin người dùng.
5. API Gateway sử dụng thông tin trả về từ Lambda Authorizer để kiểm tra quyền truy cập:

- Trường hợp nhận được thông tin từ chối truy cập thì API Gateway sẽ trả về mã 403 và từ chối truy cập tới API từ máy khách.
- Trường hợp nhận được thôn tin cho phép truy cập thì phương thức sẽ được thực thi.

## Định nghĩa Lambda Authorizer

- Khai báo authorizer trong `serverless.yml`:

```yml
functions:
  authorizer:
    handler: src.api.authorizer.lambda_handler
    cors: true
```

- Định nghĩa hàm Lambda Authorizer:

```py
import jwt


def lambda_handler(event, context):
    try:
        token = event.get("authorizationToken").split(" ")[1] # lấy thông tin token trong Authorization header
        claims = jwt.decode(token, "secret", algorithms=["HS256"]) # decode xem token có hợp lệ không
        return {
            "principalId": claims["uid"], # lấy thông tin user đề gán vào IAM
            "policyDocument": {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Action": "execute-api:Invoke",
                        "Effect": "Allow", # cho phép nếu token hợp lệ
                        "Resource": event["methodArn"],
                    }
                ],
            },
        }
    except:
        return {
            "principalId": "denied",
            "policyDocument": {
                "Version": "2012-10-17",
                "Statement": [
                    {
                        "Action": "execute-api:Invoke",
                        "Effect": "Deny", # từ chối nếu token không hợp lệ
                        "Resource": event["methodArn"],
                    }
                ],
            },
        }
```

## Định nghĩa hàm Lambda cần điều khiển quyề truy cập

- Khai báo hàm Lambda trong `serverless.yml`

```yml
functions:
  test:
    handler: src.api.test.lambda_handler
    events:
      - http:
          method: get
          path: api/test
          cors: true
          authorizer: authorizer # khai báo Lambda Authorizer
```

- Định nghĩa hàm Lambda:

```py
import json


def lambda_handler(event, context):
    headers = {"Access-Control-Allow-Origin": "*", "Accept": "application/json"}
    body = {"status": "success"}
    return {
        "statusCode": 200,
        "headers": headers,
        "body": json.dumps(body),
    }
```

## Test thử Lambda với Authorizer

- Trường hợp không truyền token trên Authorizer Header, API Gateway sẽ trả về 403

![No Auth](https://s3-ap-southeast-1.amazonaws.com/techover.storage/wp-content/uploads/2020/03/20164022/Screen-Shot-2020-03-20-at-4.39.16-PM.png)

- Trường hợp có truyền token trên Authorization Header, API Gateway sẽ cho phép phương thức được thực thi

Token được tạo như sau

```bash
(zpn) hieunv@HieuNV lambda % python
Python 3.7.7 (default, Mar 10 2020, 15:43:33)
[Clang 11.0.0 (clang-1100.0.33.17)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import jwt
>>> jwt.encode({'uid': 'hieunv'}, "secret", algorithm='HS256')
b'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1aWQiOiJoaWV1bnYifQ.xuZSlS_3lw6NvGvw_fQ2qXGBWiv2HpXTFtYtO85lQac'
```

Truyền token lên Authorizer Header

![Bearer Auth](https://s3-ap-southeast-1.amazonaws.com/techover.storage/wp-content/uploads/2020/03/20164614/Screen-Shot-2020-03-20-at-4.45.53-PM.png)

Tài liệu tham khảo:

- [Use API Gateway Lambda Authorizers](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-use-lambda-authorizer.html)

Cám ơn các bạn đã theo dõi bài viết. Hy vọng bài viết có thể giúp các bạn cài đặt việc điều khiển truy cập dễ dàng hơn với các ứng dụng trên nền tảng AWS.
