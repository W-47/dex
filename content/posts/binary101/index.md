---
title: "pwn101"
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

### Analysis

There are many ways to analyse a binary

We are now going to fire up ghidra which would help us in dissasembly. We are then going to disassemble the main function to get an idea of how the program runs. 

Here we can notice three things, first is that we have a dangerous function ***gets()***. This reads a line from stdin into the buffer pointed to by s until either a terminating newline or EOF, which it replaces with a null byte ('\0'), otherwise the function will continue to store characters past the end of the buffer allowing us to over write the adjacent memory.

Next we can also see that we have a funtion which calls to system. This is interesting.

![2](https://i.ibb.co/h2SyLr5/ghidra.png)

### Exploit 
To exploit this is very simple all we need to do is provide the program with 60 bytes and boom we spawn a shell. This can be easily done using the following ```cyclic 60```, which creates a cyclic pattern. Then we can feed that to the program and boom we have a shell.

![3](https://i.ibb.co/XbNcmQQ/shell.png)

