---
title: [Docker] Docker vs vagrant, tôi đã quản lý container như thế nào?
author: Hieu Nguyen
authorURL: https://twitter.com/hieunv8
authorFBID: 100006410080637
---

![container](https://images.unsplash.com/photo-1493946740644-2d8a1f1a6aff?ixlib=rb-1.2.1&auto=format&fit=crop&w=2736&q=80)

Bạn có gặp khó khăn trong việc xây dựng môi trường phát triển ứng dụng không? Mỗi khi bắt đầu một dự án mới, việc cài đặt môi trường phát triển thường tốn khá nhiều thời gian. Do đó việc xây dựng và chia sẻ môi trường phát triển giữa các thành viên trong dự án là thực sự cần thiết. Trong bài viết này tôi sẽ chia sẻ với các bạn cách mà tôi đã làm với các dự án của mình.

![docker-compose up -d](https://s3-ap-southeast-1.amazonaws.com/techover.storage/wp-content/uploads/2020/03/12232802/render1584030034324.gif)

## Tại sao lại sử dụng `docker`?

Chia sẻ qua một chút về quá trình trước khi tôi sử dụng docker trong các dự án của mình. Năm 2014, tôi bắt đầu xây dựng môi trường phát triển cho dự án sử dụng [vargrant](https://www.vagrantup.com/). Với vargrant tôi đã có thể đóng gói các dịch vụ được sử dụng trong dự án và chia sẻ với các thành viên trong dự án. Nhưng bạn biết không dự án có sử dụng PostgreSQL và Couchbase, khi đó tôi đã tạo 2 máy ảo vagrant cho các dịch vụ này. Bạn biết đấy, vargrant sẽ tạo ra một máy ảo hoàn chỉnh và sau đó tôi cài các dịch vụ tôi cần sử dụng lên đó. Nó thực sự là vấn đề với chiếc PC của tôi :). Ngoài ra bạn sẽ gặp khó khăn trong việc kết nối các máy ảo này với nhau nữa, bạn cần setting sao cho chúng ở cùng một mạng riêng.

Khi sử dụng `docker` thì sao? Với mỗi dịch vụ bạn tạo ra một container riêng giống như một máy ảo `vargrant` tôi đã tạo ở trên. Nhưng điểm khác biệt là gì?

- Với `docker` bạn không cần lo lắng về vấn đề cài dịch vụ nữa. Nó đã tự động làm việc đó rồi.

- `docker` không tạo ra một máy ảo hoàn chỉnh với các dịch vụ thừa trong đó. Nó đơn giản tạo ra một môi trường đủ để bạn chạy dịch vụ của mình.

- Việc cấu hình mạng riêng và chuyển tiếp cổng vào container cũng được được thiết lập dễ dàng hơn.

Với chừng đó lý do là đủ để tôi chuyến sang sử dụng docker rồi :)

## Tôi đã quản lý container bằng docker như thế nào?

Để quản lý container tôi sử dụng [Docker Compose](https://docs.docker.com/compose/), công cụ này có sẵn khi bạn cài [Docker Desktop]()

- [Docker Desktop for MacOS](https://docs.docker.com/docker-for-mac/)
- [Docker Desktop for Windows](https://docs.docker.com/docker-for-windows/)

### Cấu hình container

- Sử dụng `docker-compose.yml` để cấu hình các dịch vụ bạn muốn sử dụng trong ứng dụng

Trong phạm vi bài viết này tôi sẽ cấu hình để tạo ra hai container cho các dịch vụ MySQL và DynamoDB.

`docker-compose.yml`

```yml
version: '3'
services:
  mysql:
    image: mysql:5.7.26
    # set hostname để bạn có thể access vào container bằng tên này
    container_name: mysql
    ports:
      # Cấu hình forward port từ host vào docker container
      - '3306:3306'
    volumes:
      # Cấu hình thư mục chưa schema bạn muốn import vào MySQL
      - ./mysql/initdb.d:/docker-entrypoint-initdb.d
      # mount thư mục MySQL data để có thể backup dữ liệu nếu cần
      - ./mysql/data:/var/lib/mysql
      # Các cấu hình bạn cần thay đổi cho dịch vụ MySQL
      - ./mysql/conf.d/my.cnf:/etc/mysql/my.cnf
      # mount thư mục log để trace lỗi nếu cần
      - ./mysql/log:/var/log/mysql
    # các biến môi trường sử dụng qua tham số -e khi run container hoặc cấu hình trong .env như bên dưới
    environment:
      # set mật khẩu cho tài khoản root
      - MYSQL_ROOT_PASSWORD=$DB_ROOT_PASSWORD
      # tên database bạn muốn tạo sau khi container được khởi động
      - MYSQL_DATABASE=$DB_DATABASE
      # tạo thêm một user mới với tên được cấu hình trong $MYSQL_USER
      - MYSQL_USER=$DB_USER
      # set mật khẩu cho user được tạo ở trên
      - MYSQL_PASSWORD=$DB_PASSWORD

  dynamodb:
    image: amazon/dynamodb-local
    # set hostname để bạn có thể access vào container bằng tên này
    container_name: dynamodb
    ports:
      # Cấu hình forward port từ host vào docker container
      - '8000:8000'
    volumes:
      # mount thử mục data của DynamoDB để có thể backup
      - ./dynamodb/data:/home/dynamodblocal/data
    entrypoint: java
    command: '-jar DynamoDBLocal.jar -sharedDb -dbPath /home/dynamodblocal/data'
```

- Set các biến môi trường bằng `.env`

Tại thư mục chưa `docker-compose.yml` bạn tạo `.env` như sau:

`.env`

```env
DB_ROOT_PASSWORD=hieunv@123456
DB_DATABASE=test
DB_USER=hieunv
DB_PASSWORD=hieunv@123456
```

### Khởi động các dịch vụ

Bạn sử dụng `docker-compose` để khởi động các dịch vụ như đã cấu hình ở trên

```bash
docker-compose up -d
```

### Xoá các container

Khi bạn không muốn sử dụng container nữa hoặc khi có lỗi container mà bạn muốn tạo lại thì có thể xoá nhanh các container đã tạo bằng lệnh sau:

```bash
docker-compose down -v
```

Cám ơn các bạn đã theo dõi bài viết. Hy vọng sau bài viết này các bạn sẽ sử dụng Docker trong dự án của mình.
