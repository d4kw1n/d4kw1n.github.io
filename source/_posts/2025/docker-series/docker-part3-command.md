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
## Docker Container

Sau khi đã nắm được cách thao tác với Image, bước tiếp theo là sử dụng Image để khởi tạo và vận hành các Container – đơn vị chạy thực tế của Docker. Container là một instance của image, giống như một máy ảo nhẹ, chứa mọi thứ cần thiết để chạy ứng dụng.

### Khởi tạo và chạy Container

Để **chạy một container từ image**, bạn sử dụng câu lệnh cơ bản:

```bash
$ docker run [OPTIONS] IMAGE[:TAG] [COMMAND]
```

Ví dụ:

```bash
$ docker run -it ubuntu:20.04 /bin/bash
```

Giải thích:

- `-it`: kết hợp giữa `-i` (interactive) và `-t` (pseudo-TTY) để bạn có thể tương tác với container qua terminal.
- `ubuntu:20.04`: tên image và tag.
- `/bin/bash`: lệnh được thực thi khi container khởi động.

> Khi container chạy với `-it`, bạn sẽ “chui” vào bên trong nó và có thể thao tác như đang ở một máy Linux thật.

---

### Liệt kê và Kiểm tra Container

Để quản lý các container hiệu quả, bạn cần nắm vững các lệnh xem trạng thái:

```bash
# Xem các container đang chạy
docker ps

# Xem tất cả container (bao gồm cả đã dừng)
docker ps -a

# Chỉ xem ID của các container
docker ps -q

# Xem thông tin chi tiết về container
docker inspect <CONTAINER_ID hoặc NAME>

# Xem logs của container
docker logs <CONTAINER_ID hoặc NAME>

# Theo dõi logs realtime
docker logs -f <CONTAINER_ID hoặc NAME>
```

Thông tin hiển thị bao gồm:

```
- CONTAINER ID
- IMAGE
- COMMAND
- CREATED
- STATUS
- PORTS
- NAMES
```

---

### Quản lý Vòng đời Container

Docker cung cấp một bộ lệnh đầy đủ để quản lý toàn bộ vòng đời của container:

```bash
# Dừng container nhẹ nhàng (gửi SIGTERM)
docker stop <CONTAINER_ID hoặc NAME>

# Dừng container ngay lập tức (gửi SIGKILL)
docker kill <CONTAINER_ID hoặc NAME>

# Khởi động lại container đã dừng
docker start <CONTAINER_ID hoặc NAME>

# Khởi động lại container
docker restart <CONTAINER_ID hoặc NAME>

# Tạm dừng container (pause)
docker pause <CONTAINER_ID hoặc NAME>

# Bỏ tạm dừng container
docker unpause <CONTAINER_ID hoặc NAME>

# Xóa container (phải dừng trước)
docker rm <CONTAINER_ID hoặc NAME>

# Xóa container đang chạy (force remove)
docker rm -f <CONTAINER_ID hoặc NAME>

# Xóa tất cả container đã dừng
docker container prune

# Xem tài nguyên container sử dụng
docker stats <CONTAINER_ID hoặc NAME>
```

---

### Vào lại Container

- Nếu bạn có một container đang chạy và muốn vào lại bên trong, hãy dùng:

```bash
$ docker exec -it <CONTAINER_ID hoặc NAME> /bin/bash
```

> Đây là cách phổ biến để “SSH” vào một container.

---

## Build Image từ Dockerfile

Ngoài việc dùng image có sẵn từ Docker Hub, bạn có thể **tự tạo một Docker image** bằng cách sử dụng file `Dockerfile`.

Một ví dụ đơn giản:

```dockerfile
# syntax=docker/dockerfile:1
FROM node:20

WORKDIR /app
COPY . .
RUN npm install

CMD ["npm", "start"]
```

Build image từ Dockerfile:

```bash
$ docker build -t ten_image:tag .
```

- `-t`: đặt tên cho image.
- `.`: đường dẫn đến thư mục chứa Dockerfile.

Sau khi build xong, bạn có thể kiểm tra bằng `docker images` và dùng như một image bình thường.

---

_Phần tiếp theo mình sẽ nói về Docker Volume, Network và cách quản lý tài nguyên trong Docker. Hẹn gặp lại trong Part 4 nhé!_
