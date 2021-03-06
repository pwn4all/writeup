# &#35; chall_17


#### &#42; purpose
1. find random number with srand() and rand in c
2. find random number with srand() and rand using ctypes in python


#### &#42; execute
```bash
## 1 file
$ ls ./chall_17
./chall_17

## 64-bit LSB, dynamically linked and not stripped
$ file ./chall_17
./chall_17: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=35bbdad2936fe6daba78adf8f578d88cbcf8453d, for GNU/Linux 3.2.0, not stripped

## Full RELRO, Canary found and PIE enabled
$ checksec --file=./chall_17
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Full RELRO      Canary found      NX enabled    PIE enabled     No RPATH   No RUNPATH   76) Symbols	  No	0		1		./chall_17

## just input once and 2 output
$ ./chall_17
AAAAAAAA
Got: 2109361344
Expected: 479487375
$


```

#### &#42; disassemble using ghidra
```bash
## vulnerable point is fgets(local_4e,90,stdin); in vuln()

void main(void)

{
  time_t tVar1;
  long in_FS_OFFSET;
  uint local_18;
  uint local_14;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  tVar1 = time((time_t *)0x0);
  srand((uint)tVar1);
  local_14 = rand();
  
  //scanf("%d",&local_18);
  __isoc99_scanf(&DAT_0010202a,&local_18);
  if (local_14 == local_18) {
    win();
  }
  else {
    printf("Got: %d\nExpected: %d\n",(ulong)local_18,(ulong)local_14);
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
}

void win(void)

{
  int iVar1;
  FILE *__stream;
  
  __stream = fopen("flag.txt","rb");
  if (__stream == (FILE *)0x0) {
    puts("Could not open flag file.");
                    /* WARNING: Subroutine does not return */
    exit(0);
  }
  do {
    iVar1 = fgetc(__stream);
    putchar((int)(char)iVar1);
  } while ((char)iVar1 != -1);
  return;
}

```


#### &#42; exploit step
```bash
1. input(local_18) and local_14(srand()) must be the same.
2. rand() of srand() in c is different rand() of srand() in python
3. thus need to use ctypes in python

```



#### &#42; payload(python)
```python
libc.srand(libc.time(0))
key = libc.rand()

```


#### &#42; make exploit code
```python
$ cat exp.py                                                                                                         15,1          All
from pwn import *
from ctypes import *

fileanme = "./chall_17"
conn = process(fileanme)

libc = CDLL("/lib/x86_64-linux-gnu/libc.so.6")
libc.srand(libc.time(0))
key = libc.rand()

print(key)

conn.sendline(str(key))

conn.interactive()

```


#### &#42; try to exploit
```bash
$ python2 exp.py
[+] Starting local process './chall_17': pid 2111
175236379
[*] Switching to interactive mode
[*] Process './chall_17' stopped with exit code 0 (pid 2111)
sun{unholy-confessions-b74c1ed1f1d486fe}
\xff[*] Got EOF while reading in interactive
$
[*] Got EOF while sending in interactive

```
