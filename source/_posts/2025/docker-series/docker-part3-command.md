---
title: '[Docker Series] Các câu lệnh của Docker (P3)'
description: "Hướng dẫn sử dụng Docker"
tag: ['Docker', 'Knowledge']
top_img: /img/2025/docker-series/banner.png
cover: /img/2025/docker-series/banner.png
category: ['Knowledge', 'Docker']
date: 2025-08-06 01:43:00
author:
    - d4kw1n
---

# Nối tiếp

- Cũng được 1 tháng từ Phần 2, hôm nay mình mới quyết định hoàn thành và public phần 3 của series này. Phần này sẽ tập trung vào việc sử dụng các câu lệnh phổ biến của Docker.

# Note for myself (Please skip it)

- Bây giờ là đầu tháng 8, cũng đã 2h sáng rồi, deadline thì chưa xong mà còn buồn vì đời và vì tình nên tôi suy quá (╥﹏╥)
- Thời điểm này, bản thân mình cũng quá nhiều lựa chọn làm thay đổi lớn đến cuộc sống. Nhưng mình hy vọng những quyết định này sẽ không để lại những nuối tiếc cho bản thân và những người xung quanh.

> _"❀❀❀❀ I hope everything is always perfect for those I love and for myself ❀❀❀❀"_

# Câu lệnh với Docker

## Docker

- Để bắt đầu với việc sử dụng môi trường dòng lệnh với Docker, bạn có thể gõ `docker` hoặc `docker --version` và xem kết quả trả về.

![Docker command](/img/2025/docker-series/image.png)

![docker --version](/img/2025/docker-series/image-1.png)

## Docker Image

- Bạn có thể xem các danh sách lệnh liên quan đến Docker image và cách gõ lệnh

```bash
$ docker image
```

![docker image](/img/2025/docker-series/image-4.png)


### Liệt kê Image

Để liệt kê các Images đang có, bạn hãy sử dụng lệnh

```bash
$ docker images
```

![docker images](/img/2025/docker-series/image-3.png)

```bash
$ docker image ls
```

![docker image ls](/img/2025/docker-series/image-2.png)

Các trường thông tin sẽ bao gồm

```txt
- REPOSITORY: Tên image
- TAG: Phiên bản image
- IMAGE ID: Mã định danh của image
- CREATED: Thời gian image được khởi tạo
- SIZE: Kích thước của image
```

### Tìm Image

Bạn có thể sử dụng câu lệnh bên dưới để tìm kiếm các image liên quan đến "PHP"

```bash
$ docker search <keyword>
```

![Tìm các image liên quan đến PHP](/img/2025/docker-series/image-5.png)

### Pull & Push Image

- Để kéo một image từ kho lưu trữ về, bạn có thể sử dụng câu lệnh

```bash
$ docker pull image:tag
```

Ví dụ:

![Pull image Ubuntu:20.04](/img/2025/docker-series/image-6.png)

- Tiếp theo, bạn cũng có thể đẩy một Image lên kho lưu trữ. Tuy nhiên, bạn cần phải đăng nhập mới có thể push được

```bash
$ docker push [OPTIONS] NAME[:TAG]
```

Ví dụ:

```bash
$ docker push lab1_fetchimage-ftp:latest
```

![Push image](/img/2025/docker-series/image-7.png)

### Xóa Image

Để xóa một Image, bạn có thể sử dụng câu lệnh

```bash
$ docker image rm <ten_image_muon_xoa:tag>
```

hoặc

```bash
$ docker rmi <ten_image_muon_xoa:tag>
```

hoặc 

```bash
$ docker rmi <IMAGE ID>
```

hoặc

```bash
$ docker image rm <IMAGE ID>
```
