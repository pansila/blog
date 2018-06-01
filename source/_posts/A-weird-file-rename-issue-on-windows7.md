---
title: A Weird File Rename Issue On Windows7
abbrlink: b4ce772d
date: 2018-06-01 23:36:19
tags:
---

Recently, I came upon this issue which bothered me badly as many tools fail to work with that. The symptom is that I can't rename any file in the file Explorer or command line, both give me an error message saying that `access denied` or `invalid device`, I can create a new file though.

To debug what happened under the hood, I wrote a python script to call rename api periodically, and hopes some tool could reveal something.

```python
import os
import time

if __name__ == "__main__":
    while True:
        try:
            os.rename('test.txt', 'test1.txt')
        except:
            pass
        time.sleep(1)
```

First I use Windbg to attach to the python process, run the program and pause it later, it stops at somewhere that there are lot of int 3 instructions that I don't familiar with so I gave up. I need a breakpoint! So I look up the CPython source code to see what Win API is used for os.rename, here we got [it](https://github.com/python/cpython/blob/master/Modules/posixmodule.c#L4105:14), it's `MoveFileExW`.

Type in the breakpoint in the windbg

```
bp MoveFileExW
```

Then more assembly came out, in fact I'm not sure what they are doing, just following the execution flow to see where it exits, and hope will get some useful information about the early abort error.

```arm
...
00000000`770d2dd5 ff158d9e0900    call    qword ptr [kernel32!UnhandledExceptionFilter+0x11a8 (00000000`7716cc68)] ds:00000000`7716cc68={ntdll!ZwSetInformationFile (00000000`77359b10)}
0:000> p
kernel32!MoveFileExW+0x27b:
00000000`770d2ddb 89842480000000  mov     dword ptr [rsp+80h],eax ss:00000000`0026f190=0281af48
0:000> p
kernel32!MoveFileExW+0x282:
00000000`770d2de2 65488b042530000000 mov   rax,qword ptr gs:[30h] gs:00000000`00000030=????????????????
0:000> p
kernel32!MoveFileExW+0x28b:
00000000`770d2deb 488b4860        mov     rcx,qword ptr [rax+60h] ds:000007ff`fffde060=000007fffffd6000
0:000> p
kernel32!MoveFileExW+0x28f:
00000000`770d2def 4c8b8424b8000000 mov     r8,qword ptr [rsp+0B8h] ss:00000000`0026f1c8=00000000004508a0
0:000> p
kernel32!MoveFileExW+0x297:
00000000`770d2df7 33d2            xor     edx,edx
0:000> p
kernel32!MoveFileExW+0x299:
00000000`770d2df9 488b4930        mov     rcx,qword ptr [rcx+30h] ds:000007ff`fffd6030=00000000003f0000
0:000> p
kernel32!MoveFileExW+0x29d:
00000000`770d2dfd ff15c59e0900    call    qword ptr [kernel32!UnhandledExceptionFilter+0x1208 (00000000`7716ccc8)] ds:00000000`7716ccc8={ntdll!RtlFreeHeap (00000000`7731a020)}
0:000> p
kernel32!MoveFileExW+0x2a3:
00000000`770d2e03 8b8c2480000000  mov     ecx,dword ptr [rsp+80h] ss:00000000`0026f190=c00000ba
0:000> p
kernel32!MoveFileExW+0x2aa:
00000000`770d2e0a 85c9            test    ecx,ecx
0:000> p
kernel32!MoveFileExW+0x2ac:
00000000`770d2e0c 0f88a2060200    js      kernel32!GetModuleHandleW+0x13f4 (00000000`770f34b4) [br=1]
0:000> p
kernel32!GetModuleHandleW+0x13f4:
00000000`770f34b4 81f9d40000c0    cmp     ecx,0C00000D4h
0:000> p
kernel32!GetModuleHandleW+0x13fa:
00000000`770f34ba 7408            je      kernel32!GetModuleHandleW+0x1404 (00000000`770f34c4) [br=0]
0:000> p
kernel32!GetModuleHandleW+0x13fc:
00000000`770f34bc 81f9380019c0    cmp     ecx,0C0190038h
0:000> p
kernel32!GetModuleHandleW+0x1402:
00000000`770f34c2 754c            jne     kernel32!GetModuleHandleW+0x1450 (00000000`770f3510) [br=1]
0:000> p
kernel32!GetModuleHandleW+0x1450:
00000000`770f3510 e80be5ffff      call    kernel32!BaseSetLastNTError (00000000`770f1a20)
0:000> p
kernel32!GetModuleHandleW+0x1455:
00000000`770f3515 e9fff8fdff      jmp     kernel32!MoveFileExW+0x2b9 (00000000`770d2e19)
...
``` 

Here we got `ZwSetInformationFile`, I didn't notice it in the first place since I don't have any idea about windows driver development.

I went for the [Process Monitor](https://docs.microsoft.com/zh-cn/sysinternals/downloads/procmon) (procmon), a tool provided by Microsoft to track all kinds of system events.

![](/img/procmon-rename-issue.png)

It's Chinese, the messages say that renaming failed because API thinks the target file name is actually a directory. Well it's not of course!

After googling a lot, I realized that it's a minifilter issue that some malfunctioned minifilter dirver intercepts the rename request and drops it accidentally or intentionally. We have fltmc.exe to list them all.

![](/img/fltmc.jpg)

`FileInfo` and `luafv` are legal drivers form Microsoft, while `eamonm` is a driver file form ESET, an anti virus software, you'd think it should be innocent? No, it's the culprit, after uninstalling it, the issue gone. What the heck! An anti-virus software doing a virus thing!
