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


```bash
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

i used gdb to debug the binary and typed info func to list all the functions and their addresses 













