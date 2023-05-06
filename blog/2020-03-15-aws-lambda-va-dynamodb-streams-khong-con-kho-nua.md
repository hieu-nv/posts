<!-- ---
title: [AWS] Lambda và DynamoDB Streams không còn khó nữa!
author: Hieu Nguyen
authorURL: https://twitter.com/hieunv8
authorFBID: 100006410080637
--- -->

![dynamodb streams](https://images.unsplash.com/photo-1453503795393-c496eee08c98?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2704&q=80)

Với các ứng dụng hiện nay, việc giao tiếp giữa client và server phổ biến đang sử dụng Rest API. Trong một số trường hợp việc phải để client đợi xử lý là điều không thể chấp nhận được. Do đó để giải quyết vấn đề này thì phần lớn cách giải quyết là sử dụng batch process, nghĩa là tại thời điểm đó chúng ta sẽ trả lại dữ liệu cho client ở trạng thái đang xử lý(tránh xảy ra tình trạng timeout). Tuy nhiên ngay tại thời điểm đó một batch process sẽ được khởi động để thực thi tiếp các công việc còn lại. Trong bài viết này tôi sẽ hướng dẫn các bạn viết batch process bằng AWS Lambda bằng cách sử dụng DynamoDB Streams

## DynamoDB Streams

DynamoDB Streams là một tính năng trong DynamoDB cho phép bạn lắng nghe thay đổi trên một bảng dữ liệu nào đó và thực hiện các tác vụ đáp ứng yêu cầu nghiệp vụ trong ứng dụng của bạn. Mỗi khi có sự thay đổi DynamoDB sẽ ghi các bản ghi gần như ngay lập tức là dòng dữ liệu mà các ứng dụng đang lắng nghe.

Với DynamoDB Streams để giải quyết vấn đề timeout của API chúng ta chỉ gần ghi dữ liệu vào bảng trong DynamoDB sau đó dữ liệu được ghi lên dòng dữ liệu mà batch process của chúng ta đang lắng nghe rồi tiếp tục thực hiện nhiệm vụ còn lại.

Các bạn tham khảo link sau để cài đặt [DynamoDB](docker-docker-vs-vagrant-toi-da-quan-ly-container-nhu-the-nao) ở local nhé.

## Định nghĩa bảng trong DynamoDB

Các bạn có thẻ sử dụng [NoSQL Workbench for Amazon DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/workbench.settingup.html) để tạo bảng hoặc viết code để chia sẻ với các member khác như sau:

```py
# -*- coding: utf-8 -*-
import os
from datetime import datetime
import boto3

dynamodb = boto3.client(
    "dynamodb",
    endpoint_url="http://localhost:8000",
    region_name="us-east-1",
    aws_access_key_id="test",
    aws_secret_access_key="test",
)


def create_orders():
    try:
        dynamodb.delete_table(TableName="dev_orders")
    except Exception as exp:
        print(exp)

    response = dynamodb.create_table(
        TableName="dev_orders",
        AttributeDefinitions=[
            {"AttributeName": "id", "AttributeType": "S"},
            {"AttributeName": "status", "AttributeType": "S"},
        ],
        KeySchema=[{"AttributeName": "id", "KeyType": "HASH"}],
        ProvisionedThroughput={"ReadCapacityUnits": 1, "WriteCapacityUnits": 1},
        GlobalSecondaryIndexes=[
            {
                "IndexName": "statusGSIndex",
                "KeySchema": [{"AttributeName": "status", "KeyType": "HASH"}],
                "Projection": {"ProjectionType": "ALL"},
                "ProvisionedThroughput": {
                    "ReadCapacityUnits": 1,
                    "WriteCapacityUnits": 1,
                },
            },
        ],
        # bắt buộc phải có khai báo này để sử dụng DynamoDB Streams cho bảng này
        StreamSpecification={
            "StreamEnabled": True,
            "StreamViewType": "NEW_AND_OLD_IMAGES",
        },
    )
    print(response)


create_orders()
```

Kiểm tra bảng được tạo bằng NoSQL Workbench for Amazon DynamoDB

![dev_orders](https://s3-ap-southeast-1.amazonaws.com/techover.storage/wp-content/uploads/2020/03/15150835/Screen-Shot-2020-03-15-at-3.03.57-PM.png)

Các bạn chú ý giá trị bôi vàng nhé. Đây là dòng dữ liệu sẽ được DynamoDB ghi lên đó. Khi ứng dụng của bạn lắng nghe dòng dữ liệu này thì bất kỳ hành động nào xảy ra trên bảng sẽ được ghi lên dòng dữ liệu và ứng dụng của chúng ta sẽ phát hiện được điều đó.

## Định nghĩa Lambda lắng nghe dòng dữ liệu từ DynamoDB

Để lắng nghe dòng dữ liệu từ DynamoDB Streams bạn cần thêm `serverless-offline-dynamodb-streams` và cấu hìn `serverless.yml` như sau:

```yml
custom:
  # ...
  serverless-offline-dynamodb-streams:
    endpoint: http://dynamodb:8000
    accessKeyId: root
    secretAccessKey: root
# ...
plugins:
  - serverless-offline
  - serverless-python-requirements
  - serverless-offline-dynamodb-streams
```

Các bạn tham khảo bài viết [Mô phỏng AWS Lambda & API Gateway bằng Serverless Offline](https://magz.techover.io/2020/03/01/mo-phong-aws-lambda-api-gateway-bang-serverless-offline/) để biết các viết API bằng Lambda nhé.

Trong bài viết này, dể thực hiện lắng nghe dòng dữ liệu, bạn định nghĩa hàm Lambda trong Serverless như sau:

```yml
jobs_order:
  handler: src.jobs.order.lambda_handler
  events:
    - stream:
        enabled: true
        type: dynamodb
        # đây là giá trị màu vàng tôi có đề cập ở trên
        arn: arn:aws:dynamodb:ddblocal:000000000000:table/dev_orders/stream/2020-03-15T07:59:46.532
        batchSize: 1
```

## Thử viết Rest API ghi dữ liệu vào bảng và kiểm tra DynamoDB Streams

Các bạn định nghĩa một API như sau:

```yml
functions:
  post_orders:
    handler: src.api.post_orders.lambda_handler
    events:
      - http:
          method: post
          path: api/orders
          cors: true
```

`src/api/post_orders.py`

```py
import json
import logging
from datetime import datetime
from uuid import uuid4
import boto3

LOGGER = logging.getLogger()
LOGGER.setLevel(logging.INFO)


def lambda_handler(event, context):
    headers = {"Access-Control-Allow-Origin": "*", "Accept": "application/json"}
    body = json.loads(event["body"])
    dynamodb = boto3.client(
        "dynamodb",
        endpoint_url="http://localhost:8000",
        region_name="us-east-1",
        aws_access_key_id="test",
        aws_secret_access_key="test",
    )
    now = int(datetime.utcnow().timestamp())
    body = dynamodb.put_item(
        TableName="dev_orders",
        Item={
            "id": {"S": str(uuid4())},
            "name": {"S": body["name"]},
            "status": {"S": " "},
            "created_at": {"N": str(now)},
            "updated_at": {"N": str(now)},
        },
    )
    return {
        "statusCode": 200,
        "headers": headers,
        "body": json.dumps(body),
    }
```

Các bạn thử post dữ liệu bằng Postman nhé
![post_orders](https://s3-ap-southeast-1.amazonaws.com/techover.storage/wp-content/uploads/2020/03/15160205/Screen-Shot-2020-03-15-at-4.01.11-PM.png)

Các bạn để ý Terminal sau khi post dữ liệu nhé

![terminal](https://s3-ap-southeast-1.amazonaws.com/techover.storage/wp-content/uploads/2020/03/15160339/render1584262789150.gif)

Cám ơn các bạn đã theo dõi bài viết. Hy vọng bài viết đã giúp các bạn có thể sử dùng Lambda và DynamoDB tốt hơn.
