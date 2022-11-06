# ASLR bypass
Techniques: https://www.abatchy.com/2017/06/exploit-dev-101-bypassing-aslr-on.html

Some Win32 APIs useful -> DebugHelp o SymGetSymFromName


Basically:
1. Info Leak
2. Modules without ASLR enabled

  ```py
  !py mona modules -cm aslr=false,rebase=false 
  ```
