![logo](http://www.pwnable.kr/img/collision.png)

# collision
The second challenge from pwnable.kr

## Description
Daddy told me about cool MD5 hash collision today.
I wanna do something like that too!

`ssh col@pwnable.kr -p2222` (pw:guest)

## Solution

Let's start off by peaking around.

```sh
col@ubuntu:~$ ls -lh
total 16K
-r-sr-x--- 1 col_pwn col     7.2K Jun 11  2014 col
-rw-r--r-- 1 root    root     555 Jun 12  2014 col.c
-r--r----- 1 col_pwn col_pwn   52 Jun 11  2014 flag
```

Similar layout to the previous challenge, fd. Can't just cat flag obviously, so let's dive into the provided source code again.

```c
#include <stdio.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC;
unsigned long check_password(const char* p){
	int* ip = (int*)p;
	int i;
	int res=0;
	for(i=0; i<5; i++){
		res += ip[i];
	}
	return res;
}

int main(int argc, char* argv[]){
	if(argc<2){
		printf("usage : %s [passcode]\n", argv[0]);
		return 0;
	}
	if(strlen(argv[1]) != 20){
		printf("passcode length should be 20 bytes\n");
		return 0;
	}

	if(hashcode == check_password( argv[1] )){
		system("/bin/cat flag");
		return 0;
	}
	else
		printf("wrong passcode.\n");
	return 0;
}
```

So, cat is being run on flag is the output of check_password, which takes in argv[1], is equal to hashcode, 0x21DD09EC.

Let's attempt to pass in raw bytes using python. Accounting for endianness, a potential entry would be:

```sh
./col "`python -c "print '\xEC\x09\xDD\x21' + '\x00'*16"`"
```

But, we can't pass in null bytes since C will read that as the end of the string, so we need modify our input to account for padding.

If we pad using 16 hex bytes equal to 1, then we need to subtract 0x04040404 from our target 0x21DD09EC, which comes out to 0x1DD905E8.


### Capturing the Flag

```sh
col@ubuntu:~$ ./col "`python -c "print '\xE8\x05\xD9\x1D' + '\x01'*16"`"
daddy! I just managed to create a hash collision :)
```
