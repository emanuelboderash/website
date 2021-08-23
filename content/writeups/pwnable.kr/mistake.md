![logo](http://www.pwnable.kr/img/mistake.png)

# mistake
The ninth challenge from pwnable.kr

## Description
We all make mistakes, let's move on.
(don't take this too seriously, no fancy hacking skill is required at all)

This task is based on real event
Thanks to dhmonkey

hint : operator priority

ssh mistake@pwnable.kr -p2222 (pw:guest)

## Solution

```sh
ctftools ❯ ssh mistake@pwnable.kr -p2222
```

```sh
mistake@ubuntu:~$ ls -la
total 44
drwxr-x---  5 root        mistake 4096 Oct 23  2016 .
drwxr-xr-x 93 root        root    4096 Oct 10 22:56 ..
d---------  2 root        root    4096 Jul 29  2014 .bash_history
-r--------  1 mistake_pwn root      51 Jul 29  2014 flag
dr-xr-xr-x  2 root        root    4096 Aug 20  2014 .irssi
-r-sr-x---  1 mistake_pwn mistake 8934 Aug  1  2014 mistake
-rw-r--r--  1 root        root     792 Aug  1  2014 mistake.c
-r--------  1 mistake_pwn root      10 Jul 29  2014 password
drwxr-xr-x  2 root        root    4096 Oct 23  2016 .pwntools-cache
```

Interestingly, it's not just `flag` that's of interest, but we also have a `password` file.

Let's open up the source code for `mistake`.

```c
#include <stdio.h>
#include <fcntl.h>

#define PW_LEN 10
#define XORKEY 1

void xor(char* s, int len){
	int i;
	for(i=0; i<len; i++){
		s[i] ^= XORKEY;
	}
}

int main(int argc, char* argv[]){

	int fd;
	if(fd=open("/home/mistake/password",O_RDONLY,0400) < 0){
		printf("can't open password %d\n", fd);
		return 0;
	}

	printf("do not bruteforce...\n");
	sleep(time(0)%20);

	char pw_buf[PW_LEN+1];
	int len;
	if(!(len=read(fd,pw_buf,PW_LEN) > 0)){
		printf("read error\n");
		close(fd);
		return 0;		
	}

	char pw_buf2[PW_LEN+1];
	printf("input password : ");
	scanf("%10s", pw_buf2);

	// xor your input
	xor(pw_buf2, 10);

	if(!strncmp(pw_buf, pw_buf2, PW_LEN)){
		printf("Password OK\n");
		system("/bin/cat flag\n");
	}
	else{
		printf("Wrong Password\n");
	}

	close(fd);
	return 0;
}
```

So it looks like the program reads in the `password` file, and compares with our input, which has `xor(pw_buf2, 10)` performed on it.

Let's try running it.

```sh
mistake@ubuntu:~$ ./mistake
do not bruteforce...

input password : wtf? ^^
Wrong Password
```

Strangely, the code takes in two newlines, even though there's only one call to `scanf()` or similar.

If we take a close look at where the `password` file is supposed to be opened, we notice `fd` is set to the result of `open()` < 0, which is always 0, which means fd is reading from `STDIN`.

That just means we enter our own input, then the `XOR` of that to get the flag.

### Capturing the Flag

```sh
mistake@ubuntu:~$ ./mistake
do not bruteforce...
11111111110000000000
input password : Password OK
Mommy, the operator priority always confuses me :(
```
