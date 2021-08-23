![logo](http://www.pwnable.kr/img/flag.png)

# flag
The fourth challenge from pwnable.kr

## Description
Papa brought me a packed present! let's open it.

Download : http://pwnable.kr/bin/flag

This is reversing task. all you need is binary

## Solution

`strings flag` is completely useless as we can see the only legible text we get is `$Info: This file is packed with the UPX executable packer http://upx.sf.net`

Let's see what system calls are being executing when running `flag`.

```sh
strace ./flag
```

```
execve("./flag", ["./flag"], [/* 16 vars */]) = 0
mmap(0x800000, 2959710, PROT_READ|PROT_WRITE|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, 0, 0) = 0x800000
readlink("/proc/self/exe", "/home/ctf/challenges/flag/flag", 4096) = 30
mmap(0x400000, 2912256, PROT_NONE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x400000
mmap(0x400000, 790878, PROT_READ|PROT_WRITE|PROT_EXEC, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x400000
mprotect(0x400000, 790878, PROT_READ|PROT_EXEC) = 0
mmap(0x6c1000, 9968, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0xc1000) = 0x6c1000
mprotect(0x6c1000, 9968, PROT_READ|PROT_WRITE) = 0
mmap(0x6c4000, 8920, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) = 0x6c4000
munmap(0x801000, 2955614)               = 0
uname({sys="Linux", node="b1b157128929", ...}) = 0
brk(0)                                  = 0x8f4000
brk(0x8f51c0)                           = 0x8f51c0
arch_prctl(ARCH_SET_FS, 0x8f4880)       = 0
brk(0x9161c0)                           = 0x9161c0
brk(0x917000)                           = 0x917000
fstat(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 0), ...}) = 0
mmap(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0x7f10666a8000
write(1, "I will malloc() and strcpy the f"..., 52I will malloc() and strcpy the flag there. take it.
) = 52
exit_group(0)                           = ?
+++ exited with 0 +++
```

The program tells us it'll write the flag to a location in memory after it calls malloc().

Let's run in gdb and break on the exit_group() syscall and dump the memory.

```
pwndbg> catch syscall exit_group
Catchpoint 1 (syscall 'exit_group' [231])
```

```
pwndbg> generate-core-file flag.core
Saved corefile flag.core
```

### Capturing the Flag

`strings flag.core`

```
...
UPX...? sounds like a delivery service :)
I will malloc() and strcpy the flag there. take it.
...
```
