# &#35; chall_16


#### &#42; purpose
1. input string("Queue epic guitar solo *syn starts shredding*")


#### &#42; execute
```bash
## 1 file
$ ls ./chall_16
./chall_16

## 64-bit LSB, dynamically linked and not stripped
$ file ./chall_16
./chall_16: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=aa84bb2d58ac39489fd3d317d118223492904e56, for GNU/Linux 3.2.0, not stripped

## Full RELRO, Canary found and PIE enabled
$ checksec --file=./chall_16
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Full RELRO      Canary found      NX enabled    PIE enabled     No RPATH   No RUNPATH   72) Symbols	  No	0		1		./chall_16

## just input once
$ ./chall_16
AAAAAAAA
$


```

#### &#42; disassemble using ghidra
```bash
## vulnerable point is fgets(local_4e,90,stdin); in vuln()

void main(void)

{
  size_t sVar1;
  size_t sVar2;
  long in_FS_OFFSET;
  int local_60;
  char local_58 [56];
  long local_20;
  
  local_20 = *(long *)(in_FS_OFFSET + 0x28);
  fgets(local_58,0x31,stdin);
  sVar1 = strlen(local_58);
  sVar2 = strlen(key);
  if (sVar1 == sVar2) {
    local_60 = 0;
    while( true ) {
      sVar1 = strlen(key);
      if (sVar1 <= (ulong)(long)local_60) break;
      // key = "Queue epic guitar solo *syn starts shredding*"
      if (local_58[local_60] != key[local_60]) {
                    /* WARNING: Subroutine does not return */
        exit(0);
      }
      local_60 = local_60 + 1;
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
1. if input = "Queue epic guitar solo *syn starts shredding*"
2. then execute system("/bin/sh")
3. else exit(0);
4. thus input string is very important

```




```

#### &#42; payload(python)
```python
payload = b"Queue epic guitar solo *syn starts shredding*"

```


#### &#42; make exploit code
```python
$ cat exp.py
from pwn import *

# Queue epic guitar solo *syn starts shredding*

filename = "./chall_16"
conn = process(filename)

payload = b"Queue epic guitar solo *syn starts shredding*"
conn.sendline(payload)

conn.interactive()

```


#### &#42; try to exploit
```bash
$ python2 exp.py
[+] Starting local process './chall_16': pid 2018
[*] Switching to interactive mode
$
$ cat flag.txt
sun{beast-and-the-harlot-73058b6d2812c771}
$

```
