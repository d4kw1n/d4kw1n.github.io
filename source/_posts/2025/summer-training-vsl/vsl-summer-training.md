---
layout: post
title: '[Writeup] VSL Summer Training CTF 2025'
date: 2025-06-24 10:15:45
tags: ['Writeup']
category: ['Writeup']
top_img: /img/2025/summer-training-vsl/banner.png
cover: /img/2025/summer-training-vsl/banner.png
author:
    - d4kw1n
---

# Lời nói đầu

- Đây là Writeup cho những challenge 0 solve trong CTF VSL Training tháng 6, 7, 8 (Sẽ update vào cuối mỗi tháng)

# Writeup

## [RE] Crackme

### Description

![alt text](/img/2025/summer-training-vsl/crackme-des.png)

### Overview

Khi tải challenge về và mở lên trong IDA, các đoạn code phức tạp hiện ra, chương trình cũng không có các hàm cụ thể (Chỉ gồm các hàm của thư viện chuẩn).

![alt text](/img/2025/summer-training-vsl/overview-crack.png)

![alt text](/img/2025/summer-training-vsl/overview2-crack.png)

Thử chạy chương trình -> Chương trình đợi nhập vào -> Trả về kết quả

Khi nhập thử một giá trị bất kỳ vào thì sẽ nhận được kết quả trả về là `WROOONG!`

![alt text](/img/2025/summer-training-vsl/test-crack.png)

### Solution

Lúc này, để xem chương trình hoạt động như nào -> Dùng công cụ `ltrace`

Chạy lệnh `ltrace ./crackme`

```bash
┌──(kali㉿DESKTOP-44RH0FS)-[/mnt/c/Users/tripm/Desktop/crack]
└─$ ltrace ./crackme
--- SIGSEGV (Segmentation fault) ---
memset(0x8625ae8, '\0', 10000)                                            = 0x8625ae8
--- SIGSEGV (Segmentation fault) ---
fgets(
```

Lúc này, chương trình sẽ pause lại tại hàm `fgets` để chờ người dùng nhập vào

```bash
┌──(kali㉿DESKTOP-44RH0FS)-[/mnt/c/Users/tripm/Desktop/crack]
└─$ ltrace ./crackme
--- SIGSEGV (Segmentation fault) ---
memset(0x8625ae8, '\0', 10000)                                            = 0x8625ae8
--- SIGSEGV (Segmentation fault) ---
fgets(aaaaaaaaaaaaaaaa
"\303aaaaaaaaaaaaaaaa\n", 10000, 0xf7eb5700)                        = 0x8625ae8
--- SIGSEGV (Segmentation fault) ---
strlen("\303aaaaaaaaaaaaaaaa\n")                                          = 18
--- SIGSEGV (Segmentation fault) ---
puts("WROOONG!"WROOONG!
)                                                          = 9
--- SIGILL (Illegal instruction) ---
--- SIGSEGV (Segmentation fault) ---
exit(1 <no return ...>
+++ exited (status 1) +++
```

Nhập vào một dữ liệu, sau đó chúng ta thấy chương trình sẽ gọi đến hàm `strlen` với chuỗi vừa nhập vào -> Chương trình sẽ kiểm tra độ dài của chuỗi nhập vào, nếu sai thì sẽ nhảy đến in ra `"WROONGG!"`

Thử từng độ dài thì nhận thấy chương trình sẽ nhận chuỗi có độ dài là 70

```python
from pwn import *
context.log_level = "error"

for i in range(100):
    r = process(["ltrace", "./crackme"])
    r.recvuntil(b"fgets(")
    r.sendline(b"A"*i)
    r.recvuntil(b"strlen(")
    r.recvline()    
    r.recvline()    
    res = r.recvline()
    if b"WROOONG!" not in res:
        print(i)
        break
    r.close()
```

Ngay sau đó, tiếp tục dùng `ltrace` thì dễ dàng nhận thấy chương trình sẽ tiếp tục so sánh chuỗi nhập vào với các chuỗi con.

```bash
┌──(kali㉿DESKTOP-44RH0FS)-[/mnt/c/Users/tripm/Desktop/crack]
└─$ cyclic 70
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaara

┌──(kali㉿DESKTOP-44RH0FS)-[/mnt/c/Users/tripm/Desktop/crack]
└─$ ltrace ./crackme
--- SIGSEGV (Segmentation fault) ---
memset(0x8625ae8, '\0', 10000)                                            = 0x8625ae8
--- SIGSEGV (Segmentation fault) ---
fgets(aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaara
"aaaabaaacaaadaaaeaaafaaagaaahaaa"..., 10000, 0xf7ebd700)           = 0x8625ae8
--- SIGSEGV (Segmentation fault) ---
strlen("aaaabaaacaaadaaaeaaafaaagaaahaaa"...)                             = 71
--- SIGILL (Illegal instruction) ---
--- SIGSEGV (Segmentation fault) ---
strstr("aaaabaaacaaadaaaeaaafaaagaaahaaa"..., "zihldazjcn")               = nil
--- SIGSEGV (Segmentation fault) ---
puts("WROOONG!"WROOONG!
)                                                          = 9
--- SIGILL (Illegal instruction) ---
--- SIGSEGV (Segmentation fault) ---
exit(1 <no return ...>
+++ exited (status 1) +++
```

Tiếp tục viết script để có thể đoán được chuỗi đúng.

```python
from pwn import *
context.log_level = "error"

inpt = b"A"*70
final = b""
arr_final = []
while True:
    r = process(["ltrace", "./crackme"])
    r.recvuntil(b"fgets(")
    r.sendline(inpt)
    res = r.recvuntil(b"= nil")
    print(f"Correct: {res[-18:-8]}")
    final = final + res[-18:-8]
    inpt =  final + b"A"* (70 - len(final))
    arr_final.append(res[-18:-8])
    if len(final) == 70:
        break
    r.close()

print(final)
```

- Result:

```bash
┌──(kali㉿DESKTOP-44RH0FS)-[/mnt/c/Users/tripm/Desktop/crack]
└─$ python3 solve.py 
Correct: b'zihldazjcn'
Correct: b'vlrgmhasbw'
Correct: b'jqvanafylz'
Correct: b'hhqtjylumf'
Correct: b'yemlopqosj'
Correct: b'mdcdyamgec'
Correct: b'nhnewfhetk'
b'zihldazjcnvlrgmhasbwjqvanafylzhhqtjylumfyemlopqosjmdcdyamgecnhnewfhetk'
```

Tuy nhiên, sau khi lấy được chuỗi rồi thì chương trình vẫn chưa in ra flag đúng.

```bash
┌──(kali㉿DESKTOP-44RH0FS)-[/mnt/c/Users/tripm/Desktop/crack]
└─$ ./crackme 
zihldazjcnvlrgmhasbwjqvanafylzhhqtjylumfyemlopqosjmdcdyamgecnhnewfhetk
Congrats. If that is the correct input you will now get a flag
If all you see is garbage, try a different one
�\Q\�jo
       ���3�?�½8a3:���.�t���>:��
```

Hàm `strstr` chỉ kiểm tra chuỗi trong chuỗi -> Thứ tự các chuỗi con chưa đúng -> Viết script để dò thứ tự đúng của chuỗi con

```python
from pwn import *
context.log_level = "error"

inpt = b"A"*70
final = b""
arr_final = []
while True:
    r = process(["ltrace", "./crackme"])
    r.recvuntil(b"fgets(")
    r.sendline(inpt)
    res = r.recvuntil(b"= nil")
    print(f"Correct: {res[-18:-8]}")
    final = final + res[-18:-8]
    inpt =  final + b"A"* (70 - len(final))
    arr_final.append(res[-18:-8])
    if len(final) == 70:
        break
    r.close()

from itertools import permutations
import codecs

perms = permutations(arr_final)

for i in perms:
    inp_str = b""
    for word in i:
        inp_str = inp_str + word
    try:
        p = process(["./crackme"])
        p.sendline(inp_str)
        res = p.recvline()
        res = p.recvline()
        res = p.recvline()
        if b"timctf{" in codecs.decode(res, 'ascii'):
            print(f"Input: {inp_str}")
            print(f"Found: {codecs.decode(res, 'ascii')}")
            break
        p.kill()
    except:
        p.kill()
        continue
```

- Result:

```bash

```

### Flag

`timctf{7dfadd1ee67a9c516c9efbf8f0cf43f4}`

## [PWN] Casino bro

### Description

