<!-- ---
title: [AWS] Remote Debugging ứng dụng Lambda viết bằng Java với Visual Studio Code
author: Hieu Nguyen
authorURL: https://twitter.com/hieunv8
authorFBID: 100006410080637
--- -->

Debug cũng quan trọng giống như lúc bạn code vậy. Với những bạn làm quen với Lambda thì không phải ai cũng biết làm sao để có thể debug được. Đa phần các bạn sẽ chọn cách in dữ liệu ra màn hình để debug. Hôm nay tôi sẽ hướng dẫn các bạn cách debug ứng dụng viết bằng Lamba nhé.

## Remote Debugging

Như các bạn đều biết thì để có thể debug được ứng dụng Java thì bạn cần phải Remote tới cổng Debug của trình thực thi Java. Quá trình này được gọi là Remote Debugging. Với ứng dụng Java bình thường các bạn có thể dễ dàng sử dụng các IDE có hỗ trợ Remote Debugging một cách dễ dàng. Với các ứng dụng Lambda bằng Java thì sao?

## Khởi động ứng dụng Lambda với chế độ Remote Debugging

Trong bài viết [Phát triển ứng dụng Lambda bằng Java](https://magz.techover.io/2020/03/22/aws-phat-trien-ung-dung-lambda-bang-java/), tôi đã hướng dẫn các bạn cách sử dụng SAM để chạy các ứng dụng Lambda viết bằng ngôn ngữ Java. Các bạn theo dõi bài viết trên sẽ thấy ứng dụng được chạy trên một máy ảo Java trông một docker container. Để khời động chế để Remote Debugging thì các bạn gõ lệnh sau(các bạn nhớ khởi động Docker trước khi khởi động SAM nhé):

```bash
hieunv@HieuNV sam-app % sam local start-api -d 5858
Mounting HelloWorldFunction at http://127.0.0.1:3000/hello [GET]
You can now browse to the above endpoints to invoke your functions. You do not need to restart/reload SAM CLI while working on your functions, changes will be reflected instantly/automatically. You only need to restart SAM CLI if you update your AWS SAM template
2020-03-30 20:10:33  * Running on http://127.0.0.1:3000/ (Press CTRL+C to quit)
```

Các bạn sẽ thấy SAM được khởi động và lắng nghe ở cổng `3000`. Còn cổng `5858` thì sao? Tại thời điểm này nó chưa được khởi động. Khi bạn access vào `http://127.0.0.1:3000/hello` thì cổng Remote Debugging `5858` mới được khởi động.

```bash
Invoking helloworld.App::handleRequest (java11)

Fetching lambci/lambda:java11 Docker container image......
Mounting /Users/hieunv/Projects/hieunv/sam-app/.aws-sam/build/HelloWorldFunction as /var/task:ro,delegated inside runtime container
Picked up _JAVA_OPTIONS: -agentlib:jdwp=transport=dt_socket,server=y,suspend=y,quiet=y,address=*:5858 -XX:MaxHeapSize=2834432k -XX:MaxMetaspaceSize=163840k -XX:ReservedCodeCacheSize=81920k -XX:+UseSerialGC -XX:-TieredCompilation -Djava.net.preferIPv4Stack=true
```

## Cấu hình Remote Debug trong Visual Studio Code

Các bạn quay lại Visual Studio Code, vào Tab Debug sau đó chọn create a launch.json file. Tại mục chọn kiểu Debug bạn chon `Add Configuration` và chọn

![add configuration](https://s3-ap-southeast-1.amazonaws.com/techover.storage/wp-content/uploads/2020/03/30202922/Screen-Shot-2020-03-30-at-8.28.42-PM.png)

Sau đó các bạn chon `Attach To Remote Program`

![Attach To Remote Program](https://s3-ap-southeast-1.amazonaws.com/techover.storage/wp-content/uploads/2020/03/30203144/Screen-Shot-2020-03-30-at-8.31.15-PM.png)

Tiếp đó các bạn sửa lại cấu hình `hostName` thành `localhost` và `port` thành `5858`(đấy là cổng Remote Debug của trình thực thi Java)

```json
{
  // Use IntelliSense to learn about possible attributes.
  // Hover to view descriptions of existing attributes.
  // For more information, visit: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "type": "java",
      "name": "Debug (Attach) - Remote",
      "request": "attach",
      "hostName": "localhost",
      "port": 5858
    }
  ]
}
```

## Đặt break point

Các bạn quay lại Visual Studio Code và mở mã nguồn muốn debug sau đó đặt break point

![break point](https://s3-ap-southeast-1.amazonaws.com/techover.storage/wp-content/uploads/2020/03/30203623/Screen-Shot-2020-03-30-at-8.35.58-PM.png)

## Khởi động Visual Studio Code Debug bằng các click vào nut Start

Xem output log bạn sẽ thấy thông báo sau:

```
START RequestId: 4f69214b-9a3a-19ef-0137-5081d7caccea Version: $LATEST
END RequestId: 4f69214b-9a3a-19ef-0137-5081d7caccea
REPORT RequestId: 4f69214b-9a3a-19ef-0137-5081d7caccea	Init Duration: 58932.30 ms	Duration: 10421.29 ms	Billed Duration: 10500 ms	Memory Size: 512 MB	Max Memory Used: 73 MB
2020-03-30 20:39:18 127.0.0.1 - - [30/Mar/2020 20:39:18] "GET /hello HTTP/1.1" 500 -
``
```

Sau đó bạn access `http://127.0.0.1:3000/hello` bằng Postman và quay lại Visual Studio Code

![debug mode](https://s3-ap-southeast-1.amazonaws.com/techover.storage/wp-content/uploads/2020/03/30204136/Screen-Shot-2020-03-30-at-8.40.51-PM.png)

Như vậy là chúng ta đã debug thành công vào hàm Lambda rồi.

Cám ơn các bạn đã theo dõi bài viết. Hy vọng bài viết sẽ giúp ích với dự án của các bạn. Chúc các bạn thành công.
