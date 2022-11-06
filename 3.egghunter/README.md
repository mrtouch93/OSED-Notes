# EGGHUNTER (no DEP, no ASLR)

Use if you have no space. The ultimate goal is to have the egg somewhere in memory.

## Step
1. Search the tag with mona 
```py
s -a 0x0 L?80000000 w00tw00t
```

2. Create the egghunter with https://github.com/epi052/osed-scripts/blob/main/egghunter.py

## Resources
1. http://www.hick.org/code/skape/papers/egghunt-shellcode.pdf
2. https://www.corelan.be/index.php/2010/01/09/exploit-writing-tutorial-part-8-win32-egg-hunting/
3. https://fuzzysecurity.com/tutorials/expDev/4.html
4. https://www.secpod.com/blog/hunting-the-egg-egg-hunter/
5. https://sec4us.com.br/cheatsheet/bufferoverflow-egghunting
