# &#35; chall_09


#### &#42; purpose
1. understand xor encoding
2. twice xor is original string


#### &#42; execute
```bash
## 1 file
$ ls ./chall_09
./chall_09

## 64-bit LSB, dynamically linked and not stripped
$ file ./chall_09
./chall_09: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=26ab587c3ae46a3cfc3d266d0d7f38ee981d2245, for GNU/Linux 3.2.0, not stripped

## Full RELRO and PIE enabled
$ checksec --file=./chall_09
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Full RELRO      Canary found      NX enabled    PIE enabled     No RPATH   No RUNPATH   72) Symbols	  No	0		1		./chall_09

## just do echo
$ ./chall_09
AAAAAAAA
$

```

#### &#42; decompile using ghidra
```bash
## vulerable point is if ((local_58[local_5c] ^ 0x30) != key[local_5c])
## need to passthru upper if statement and go system("/bin/sh")
void main(void)

{
  size_t sVar1;
  size_t sVar2;
  long in_FS_OFFSET;
  int local_5c;
  byte local_58 [56];
  long local_20;
  
  local_20 = *(long *)(in_FS_OFFSET + 0x28);
  fgets((char *)local_58,0x31,stdin);
  sVar1 = strlen((char *)local_58);
  sVar2 = strlen(key);
  if (sVar1 == sVar2) {
    local_5c = 0;
    while( true ) {
      # key = [0x79,0x17,0x46,0x55,0x10,0x53,0x5f,0x5d,0x55,0x10,0x58,0x55,0x42,0x55,0x10,0x44,0x5f,0x3a]
      sVar1 = strlen(key);
      if (sVar1 <= (ulong)(long)local_5c) break;
      if ((local_58[local_5c] ^ 0x30) != key[local_5c]) {
                    /* WARNING: Subroutine does not return */
        exit(0);
      }
      local_5c = local_5c + 1;
    }
    system("/bin/sh");
  }
  if (local_20 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
}


void win(void)

{
  system("/bin/sh");
  return;
}

```


#### &#42; exploit step
```bash
1. key is number
2. xor with key number and 0x30
3. change key number to character because of curious about key number
  - continueous twice xor is itself

```


#### &#42; make exploit code
```python
$ cat exp.py
from pwn import *

filename = './chall_09'
conn = process(filename)

'''
>> keys = [0x79,0x17,0x46,0x55,0x10,0x53,0x5f,0x5d,0x55,0x10,0x58,0x55,0x42,0x55,0x10,0x44,0x5f,0x3a,0x00]

>> for key in keys:
...     print(chr(key^0x30), end='')
...
I've come here to
'''

payload  = b"I've come here to"
#payload += b"\x00"*(50-len(payload))

conn.sendline(payload)


with open("payload", "ab") as fd:
    fd.write(payload)

conn.interactive()

```


#### &#42; try to exploit
```bash
$ python2 exp.py
[+] Starting local process './chall_09': pid 123
[*] Switching to interactive mode
$
$ cat flag.txt
sun{coming-home-4202dcd54b230a00}
$

```


#### &#42; tips
```python
$ cat hex2str.py
from __future__ import print_function

nums = [0x79,0x17,0x46,0x55,0x10,0x53,0x5f,0x5d,0x55,0x10,0x58,0x55,0x42,0x55,0x10,0x44,0x5f,0x3a]

for num in nums:
    print( chr(int(hex(num),16)), end='')

print()


$ cat str2hex.py
import sys


#strings="/bin/sh"

if len(sys.argv) != 2:
    print("Usage : " + sys.argv[0] + " string" )
    sys.exit(-1)

strings = sys.argv[1]

print(strings)
for string in strings:
    print("\\x"+hex(ord(string))[2:], end='')

print()

print("0x",end='')
for string in strings:
    print(hex(ord(string))[2:], end='')

print()

```
