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

