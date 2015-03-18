# rootkits101

###Windows architecture
#####Rings of security
-4 rings: ring 0 is the kernel, handles hardware and important things.  Can do -anything- it wants.
-Rings 1-2 are typically device drivers.  Most OS's don't implement these as separate rings.
-Ring 3 is userland.  Userland programs can crash without endangering the rest of the OS.  Typically will use a hardware interrupt (remember int 80h in assembly?) to do things they need to do, like I/O.

#####Windows impelmentation
-Windows is more abstract than linux, usually.  Reading directory metadata, for example.
-Most programs run through the Windows API, a collection of dlls which provides the C functions required to do Windows things.
-Slightly below that is the Native API, the original one used in Windows NT.  The Windows API is instantiated through the Native API, which starts at boot.  Relevant for hooking on earlier than others.
-Sample stack for listing a directory through GUI:
explorer.exe
Windows API -> kernel32.dll
Native API -> corresponding function in Windows API called in ntdll.dll
kernel -> ntoskrnl.exe
hardware abstraction layer -> hal.dll
disk driver -> disk.sys (translates I/O request for next level)
hard drive driver -> atapi.sys and others (communicates with disk hardware; TDSS likes to exploit this)

###Rootkits
-Rootkits try to disguise the prescence of themselves and other malware they may be bundled with.  Looking at the stack, there's a number of places to hook or replace.
1. Hook the Windows or Native API: relatively easy.  However, antiviruses may run using the Native API and can detect any higher hooks; at the same level, there is a race condition.  In addition, injecting code in general is very suspicious.
2. Hook the kernel dispatch table.  This is a simple hash of function names and their pointers in memory.  If you can get kernel access, you can replace the pointer of a function with your own to do whatever you want.
3. Hook/replace a system driver.  These will also run at ring 0, so you can do whatever you want with code in here.  More relevant is for viruses that infect the MBR, like TDSS.  In this case, low level disk access is required to intercept any calls attempting to check the MBR and hide the infection.
