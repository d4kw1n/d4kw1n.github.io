---
title: '[Docker Series] Hướng dẫn cài đặt (P2)'
description: "Hướng dẫn cài đặt Docker"
tag: ['Docker', 'Knowledge']
top_img: /img/2025/docker-series/banner.png
cover: /img/2025/docker-series/banner.png
category: ['Knowledge', 'Docker']
date: 2025-07-08 01:42:00
author:
    - d4kw1n
---

# Nối tiếp

- Ở phần trước, mình đã giới thiệu khái quát về Docker.
- Phần này sẽ tập trung vào phần cài đặt Docker trên các hệ điều hành.

# Cài đặt Docker

## Yêu cầu cấu hình

- Docker CLI khá nhẹ nên không cần yêu cầu quá cao. Còn đối với Docker Desktop thì có phần nặng hơn. Ở dưới là yêu cầu tối thiểu mà mình tìm hiểu được.

| Thành phần    | Yêu cầu tối thiểu                                         |
| ------------- | --------------------------------------------------------- |
| **CPU**       | 64-bit, hỗ trợ **virtualization** (VT-x, AMD-V)           |
| **RAM**       | Tối thiểu **4 GB** (khuyến nghị >= 8 GB)                  |
| **Ổ đĩa**     | Trống ít nhất **10 GB**                                   |
| **Kiến trúc** | 64-bit (x86\_64/amd64), ARM64 (đối với Raspberry Pi v.v.) |

## Window

### Yêu cầu

- Phải là Windows 10/11 64-bit
    - Home: Yêu cầu WSL 2
    - Pro/Enterprise/Education: Có thể dùng Hyper-V hoặc WSL 2
    - CPU hỗ trợ ảo hóa (VT-x / AMD-V)
    - RAM ≥ 4 GB
    - Đã bật Virtualization trong BIOS

### Cài đặt

1. Bạn cần cài đặt và kích hoạt WSL2

- Mở terminal powershell với quyền admin và gõ lệnh `wsl --version` để kiểm tra wsl của bạn.
- Nếu chưa có hãy làm theo các bước

```ps
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
wsl --install
wsl --set-default-version 2
```

(Bạn có thể làm theo các bước chi tiết hơn tại đây: [Install WSL on Window](https://learn.microsoft.com/en-us/windows/wsl/install-manual))

2. Tải Docker về từ trang chủ

- Truy cập vào [Docker.com](https://docker.com) và tải về Docker phù hợp với hệ điều hành của bạn.
- Khởi chạy và cài đặt theo từng bước.
- Khởi động lại máy tính.
- Khởi chạy Docker để kiểm tra.
- Nếu thành công sẽ có giao diện tương tự bên dưới

![Docker Example](/img/2025/docker-series/docker-example.png)

## Linux 

### Cài đặt

- Bạn có thể làm theo các lệnh bên dưới để cài đặt `Docker` và `docker-compose` với người dùng `root`

```bash
sudo apt-get update -y && sudo apt-get upgrade -y
sudo apt-get install docker.io
sudo systemctl start docker && sudo systemctl enable docker
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```
- Cài đặt xong, các bạn có thể kiểm tra với các lệnh `docker --version` và `docker-compose --version`

- Ngoài ra, các bạn có thể cài đặt theo các cách cầu hình khác với đường dẫn sau: [Install Docker for Linux with Different Configuration](https://docs.docker.com/engine/install/linux-postinstall/)

## MacOS

- Mình chưa có cơ hội cài Docker trên Mac, nên bó tay @.@
- Hy vọng có đạo hữu nào có kinh nghiệm thì share để mình bổ sung vào bài viết nhé hehe ^^

# Tổng kết

- Bài viết này tập trung về việc cài đặt Docker trên các hệ điều hành.
- Bài viết tiếp theo có thể sẽ là về cách sử dụng cơ bản của Docker.
- Ace đọc và thấy thiếu sót, sai sót chỗ nào xin cứ ib cho mình để mình kịp thời sửa chữa. Xin chân thành cảm ơn orz

