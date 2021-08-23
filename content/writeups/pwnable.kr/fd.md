![logo](http://www.pwnable.kr/img/fd.png)

# fd
The first challenge from pwnable.kr

## Description
Mommy! what is a file descriptor in Linux?

* try to play the wargame your self but if you are ABSOLUTE beginner, follow this tutorial link:
https://youtu.be/971eZhMHQQw

`ssh fd@pwnable.kr -p2222` (pw:guest)

## Solution

SSH into the server, and inspect the immediate directory's contents.

```sh
fd@ubuntu:~$ ls -lh
total 16K
-r-sr-x--- 1 fd_pwn fd   7.2K Jun 11  2014 fd
-rw-r--r-- 1 root   root  418 Jun 11  2014 fd.c
-r--r----- 1 fd_pwn root   50 Jun 11  2014 flag
```

We get permission denied if we try to cat flag, but fd has suid so it executes as fd_pwn, the same owner as flag.

Source code for fd is provided, so let's see what fd does by opening up fd.c:

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char buf[32];
int main(int argc, char* argv[], char* envp[]){
	if(argc<2){
		printf("pass argv[1] a number\n");
		return 0;
	}
	int fd = atoi( argv[1] ) - 0x1234;
	int len = 0;
	len = read(fd, buf, 32);
	if(!strcmp("LETMEWIN\n", buf)){
		printf("good job :)\n");
		system("/bin/cat flag");
		exit(0);
	}
	printf("learn about Linux file IO\n");
	return 0;

}
```

You can quickly spot that cat is being run on flag, and in order to get there, we need to pass in LETMEWIN during the read() call.

read() takes in STDIN if fd is valid (0, 1 or 2 for STDIN, STDOUT, or STDERR).

But fd is being modified by having 0x1234 subtracted from argv[1]. So supply int value between 4660-4662 to be able to get into the read() call.

### Capturing the Flag

```sh
fd@ubuntu:~$ ./fd 4660
LETMEWIN
good job :)
mommy! I think I know what a file descriptor is!!
```
