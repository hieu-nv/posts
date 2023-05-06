<!-- ---
title: [AWS] Phát triển ứng dụng Lambda bằng Java
author: Hieu Nguyen
authorURL: https://twitter.com/hieunv8
authorFBID: 100006410080637
--- -->

Như các bạn đã biết hiện nay môi trường thực thi sử dụng trong Lambda phần lớn đang sử dụng Node hay Python. Tuy nhiên trên thực tế đôi khi bạn cần sử dụng một môi trường thực thi khác như Java chẳng hạn. Trên thực tế thì AWS cũng đang hỗ trợ khá nhiều môi trường thực thi khác nhau. Có nhiều lý do dẫn tới việc chúng ta phải sử dụng một môi trường thực thi nào đó tuỳ vào tình hình dự án. Trong bài viết này tôi sẽ hướng dẫn các bạn xây dựng ứng dụng Lamba sử dụng môi trường thực thi là Java.

## Các công cụ cần thiết

- [Docker](https://www.docker.com/)
- [SAM](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/what-is-sam.html)
- [Oracle JDK](https://www.oracle.com/java/technologies/javase-downloads.html)
- [Maven](https://maven.apache.org/)

### Docker

Chúng ta cần Docker bởi vì công cụ thực thi SAM CLI sẽ sử dụng docker container để thực thi ứng dụng. Bạn thao khảo đường dẫn sau để [cài đặt Docker](https://docs.docker.com/docker-for-mac/install/)

### SAM

Chúng ta sẽ sử dụng SAM vì chúng ta cần một môi trường thực thi có thể chạy trên môi trường cục bộ và có thể debug được. Để cài SAM bạn làm theo hướng dẫn sau:

```bash
brew tap aws/tap
brew install aws-sam-cli
```

Chúng ta sử dụng brew để cài SAM nên bạn cần cài brew trước. Nếu chưa cài brew thì bạn có thể thao khảo cách cài brew như sau:

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
hieunv@HieuNV ~ % brew --version
Homebrew 2.2.10
Homebrew/homebrew-core (git revision f0179; last commit 2020-03-22)
Homebrew/homebrew-cask (git revision 0a88ae; last commit 2020-03-22)
```

Để kiểm tra xem bạn đã cài đặt thành công chưa, bạn sử dụng lệnh sau:

```bash
hieunv@HieuNV ~ % sam --version
SAM CLI, version 0.45.0
```

Trên Windows thì bạn thao khảo [đường dẫn này](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install-windows.html)

### Oracle JDK

Chúng ta sẽ sử dụng môi trường thực thi Java nên việc cài đặt Oracle JDK là đương nhiên đúng không. Các bạn tham khảo cách cài đặt Oracle JDK tại đây nhé.

### Maven

SAM sẽ sử dụng maven để build nên chúng ta cần cài đặt thêm maven. Để cài đặt Maven các bạn sử dụng lệnh sau:

```bash
brew install --ignore-dependencies maven
```

Các bạn chú ý, chúng ta cần sử dụng `--ignore-dependencies` để bỏ qua việc cài đặt Open JDK nhé. Mặc định maven sẽ sử dụng Open JDK. Tuy nhiên chúng ta đã cài đặt Oracle JDK rồi nên không cần cài Open JDK nữa.

Tài liệu tham khảo:

- [Installing the AWS SAM CLI](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/serverless-sam-cli-install.html)

## Tạo project bằng SAM

- Tạo một project mới

```
hieunv@HieuNV hieunv % sam init -r java11
Which template source would you like to use?
	1 - AWS Quick Start Templates
	2 - Custom Template Location
Choice: 1

Which dependency manager would you like to use?
	1 - maven
	2 - gradle
Dependency manager: 1

Project name [sam-app]:

Cloning app templates from https://github.com/awslabs/aws-sam-cli-app-templates.git

AWS quick start application templates:
	1 - Hello World Example: Maven
	2 - EventBridge Hello World: Maven
	3 - EventBridge App from scratch (100+ Event Schemas): Maven
Template selection: 1

-----------------------
Generating application:
-----------------------
Name: sam-app
Runtime: java11
Dependency Manager: maven
Application Template: hello-world
Output Directory: .

Next steps can be found in the README file at ./sam-app/README.md
```

- Trước khi thực thi bạn cần build project trước

```bash
hieunv@HieuNV hieunv % cd sam-app
hieunv@HieuNV sam-app % sam build
Building resource 'HelloWorldFunction'
/usr/local/bin/mvn is using a JVM with major version 13 which is newer than 11 that is supported by AWS Lambda. The compiled function code may not run in AWS Lambda unless the project has been configured to be compatible with Java 11 using 'maven.compiler.target' in Maven.
Running JavaMavenWorkflow:CopySource
Running JavaMavenWorkflow:MavenBuild
Running JavaMavenWorkflow:MavenCopyDependency
Running JavaMavenWorkflow:MavenCopyArtifacts

Build Succeeded

Built Artifacts  : .aws-sam/build
Built Template   : .aws-sam/build/template.yaml

Commands you can use next
=========================
[*] Invoke Function: sam local invoke
[*] Deploy: sam deploy --guided
```

- Khởi động ứng dụng (trước khi khởi động bạn cần đảm bảo rằng Docker đang hoạt động)

```
hieunv@HieuNV sam-app % sam local start-api
Mounting HelloWorldFunction at http://127.0.0.1:3000/hello [GET]
You can now browse to the above endpoints to invoke your functions. You do not need to restart/reload SAM CLI while working on your functions, changes will be reflected instantly/automatically. You only need to restart SAM CLI if you update your AWS SAM template
2020-03-22 22:07:45  * Running on http://127.0.0.1:3000/ (Press CTRL+C to quit)
```

Chúng ta thử truy cập vào `http://127.0.0.1:3000/hello` bằng Postman. Nếu các bạn chưa chạy lần nào thì sẽ phải chờ hơi lâu một chút để Docker tải image cần thiết.

![start-api](https://s3-ap-southeast-1.amazonaws.com/techover.storage/wp-content/uploads/2020/03/22221132/Screen-Shot-2020-03-22-at-10.10.41-PM.png)

Trong bài viết này tôi đã hướng dẫn các bạn cách viết một API bằng Lambda sử dụng môi trường thực thi Java. Hy vọng bài viết sẽ giúp ích cho dự án của các bạn.
