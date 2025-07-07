---
title: '[Docker Series] Tìm hiểu về Độc-kơ (P1)'
tag: ['Docker', 'Knowledge']
top_img: /img/2025/docker-series/banner.png
cover: /img/2025/docker-series/banner.png
category: ['Knowledge', 'Docker']
date: 2025-06-27 14:53:00
author:
    - d4kw1n
---

# Miếng trầu là đầu câu chuyện

> ♪ Một chiều mưa, trong lòng anh thấm bao nỗi sầu ~~ ♪

> Đang ngồi pen-tét trong cái nóng 35°C, cơn mưa to âp đến Đà Nẵng. Tâm trạng mình cũng quay ngoắt 180° như thời tiết =))))

Vui vẻ thôi hehe. Thật ra, series bài viết Docker này mình viết ra 70 ~ 80% là dành cho member của VSL. Với vai trò là bang chủ cái bang, mình nhận thấy kỹ năng về docker của các bạn khá yếu. 20 ~ 30% còn lại mình viết là để cô động lại kiến thức của bản thân hehe :>

Em cũng xin phép sử dụng lại một số kiến thức em thu gom ở trên in-tơ-nét (Khả năng viết của em không tốt lắm :<). Nếu cô/bác/anh/chị có đọc được bài này và thấy giống giống thì cho em xin phép xin lỗi và được dùng ké kiến thức xíu ạ. Em xin chân thành cảm ơn orz

![](https://www.focusonthefamily.com/wp-content/uploads/2021/05/Meme-Respect2-1024x1024.jpg)

## Docker là gì?

- Docker là container platform được dùng để triển khai, quản lí và phát triển ứng dụng một cách nhanh chóng và hiệu quả.
- Docker cho phép chúng ta triển khai ứng dụng một cách độc lập cho với cơ sở hạ tầng thực tế. Điều này khiến cho việc chúng ta có thể dễ dàng triển khai ứng dụng trên các thiết bị khác nhau mà không cần cài đặt quá nhiều, đồng thời cũng dễ dàng hơn trong việc quản lý chúng như quản lý các hệ thống máy ảo.
    - Ví dụ: Khi các bạn triển khai một ứng dụng web php. Lập trình viên phải cài đặt các phần mềm, ngôn ngữ, công cụ, ... hỗ trợ: php language, apache server, cơ sở dữ liệu (Mysql, MSSQL, Oracle, ...), ...
    -> Điều này khiến cho nếu như bạn muốn chia sẻ ứng dụng web này đến người khác để người khác triển khai hoặc phát triển, thì người đó phải cài đặt môi trường giống bạn như mình đã nêu trên.
    - Bằng cách dùng Docker, bạn có thể đóng gói lại mọi thứ và gửi cho người mà bạn muốn chia sẻ. Người kia chỉ cần dùng Docker để khai mở và sử dụng. Rất đơn giản và tiện lợi hơn đúng không nào.

## Tại sao lại dùng Docker?

### Nhanh, gọn, đồng bộ nhất quán

Như mình đã nói ở trên, Docker hỗ trợ rất tốt cho việc phát triển và triển khai ứng dụng. Docker hỗ trợ container để đóng gói ứng dụng và chạy trên môi trường độc lập mà không cần phải cài đặt quá nhiều thứ phức tạp khác. -> Điều này giúp cho việc đồng bộ giữa các thiết bị khác nhau, không gây ra xung đột phiên bản, ...

### Đa tác vụ

Vì đặc tính nhanh và nhẹ, dễ triển khai. Docker hoàn toàn có thể thay thế cho việc sử dụng máy ảo để phát triển, chúng ta có thể hoàn toàn sử dụng docker để làm được nhiều công việc hơn vì lượng tài nguyên được sử dụng ít hơn.

### Khả năng mở rộng

Vì việc Docker có thể chạy trên hầu hết mọi thiết bị và môi trường (Laptop cá nhân, server, VPS, ...), đồng thời đáp ứng cho khối lượng công việc có tính linh động cao nên Docker đang dần dần trở thành một phần không thể thiếu. Docker cũng hỗ trợ quản lý scale tài nguyên theo thời gian thực nên tính ứng dụng rất cao.

## Kiến trúc

Docker được thiết kế theo mô hình client-server, giúp đơn giản hóa việc đóng gói, phân phối và chạy các ứng dụng trong các container
- Docker client sẽ chịu trách nhiệm làm việc với Docker Daemon thông qua REST API (thường là socket Unix hoặc TCP). Đây cũng là công cụ dòng lệnh (CLI) mà người dùng sử dụng để giao tiếp với Docker. 

### Docker Daemon (dockerd)

- Là dịch vụ nền xử lý các yêu cầu từ Docker Client.
- Có nhiệm vụ quản lý:
    - Docker objects (containers, images, networks, volumes)
    - Tạo và chạy containers
- Giao tiếp với các daemon khác để quản lý Docker Swarm nếu cần.

### Docker Images

- Đây là các snapshot của hệ thống, chứa code, runtime, thư viện, và các dependencies.
- Là nền tảng để tạo containers.
- Để xây dựng riêng images, chúng ta tạo Dockfile với những cú pháp đơn giản định nghĩa các bước cần thiết để tạo image và chạy nó. Mỗi câu lệnh trong Dockerfile tạo một layer trong image. Khi bạn thay đổi Dockerfile và rebuild image, chỉ những layer bị thay đổi mới buil lại. Đây chính là lí do làm cho image trở nên nhẹ, nhỏ, nhanh khi đem so sánh với những công nghệ ảo hóa khác.
- Ví dụ: ubuntu, nginx, node:18, ...

### Docker Containers

- Đây chính là thành phần sẽ chịu trách nhiệm đóng gói phần mềm lại trong một môi trường cô lập, chứa tất cả những thành phần cần thiết để có thể chạy phần mềm. (Có thể bao gồm: library, tool, mã nguồn, ...)
- Là phiên bản đang chạy của một image.
- Isolated (cô lập), lightweight và portable.
- Được quản lý bởi Docker Daemon.
- Một container được định nghĩa bởi images của nó, cũng như bất kỳ tùy chọn cấu hình nào bạn cung cấp cho nó khi tạo hoặc khởi động nó. Khi một container bị xóa, những thay đổi về trạng thái của nó không được lưu mà sẽ bị xóa.

### Docker Registries

- Đây là nơi lưu trữ các `Docker Images`
- Docker Reistries public là `Docker Hub`
- Chúng ta cũng có thể host Private registry bằng Docker Registry

### Sơ đồ kiến trúc tổng quan

```txt
+-----------------+
| Docker Client   |
|  (CLI / API)    |
+--------+--------+
         |
         v
+--------+--------+
| Docker Daemon   | <===> Docker Registry (Docker Hub)
|  - Manages      |
|    Containers   |
|  - Builds       |
|    Images       |
+--------+--------+
         |
         v
+--------+--------+
| Containers      |
|  (Runtime)      |
+-----------------+
```

## Tổng kết

- Ở bài viết phần này, mình sẽ chỉ nêu khai quát khái niệm, kiến trúc, lợi ích của Docker.
- Ở phần tiếp theo, mình sẽ hướng dẫn cách cài đặt Docker trên các hệ điều hành khác nhau.
- Hy vọng bản thân mình sẽ lưu lại được những kiến thức này thật lâu 🧙 