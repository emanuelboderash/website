![logo](http://www.pwnable.kr/img/random.png)

# random
The sixth challenge from pwnable.kr

## Description
Daddy, teach me how to use random value in programming!

`ssh random@pwnable.kr -p2222` (pw:guest)

## Solution
```sh
random@ubuntu:~$ ls -lh
total 20K
-r--r----- 1 random_pwn root     49 Jun 30  2014 flag
-r-sr-x--- 1 random_pwn random 8.4K Jun 30  2014 random
-rw-r--r-- 1 root       root    301 Jun 30  2014 random.c
```

This setup seems similar. Same suid bit that should allow us to read `flag` using `random` with `random_pwn` user.

We get source code again so let's take a look at it.

```c
#include <stdio.h>

int main(){
	unsigned int random;
	random = rand();	// random value!

	unsigned int key=0;
	scanf("%d", &key);

	if( (key ^ random) == 0xdeadbeef ){
		printf("Good!\n");
		system("/bin/cat flag");
		return 0;
	}

	printf("Wrong, maybe you should try 2^32 cases.\n");
	return 0;
}
```

`system("/bin/cat flag")` is called if `(key ^ random) == 0xdeadbeef`, where key is `stdin` and random is `rand()`.

`rand()` is not truly random, so instead of wasting our brute force efforts, let's narrow it down based on how the randomization works.

`man rand`

```
...
If no seed value is provided, the functions are automatically seeded with
a value of 1.
...
```

If we print the value of `random`, we get `1804289383`.

`0xdeadbeef ^ 0x6B8B4567` is `0xB526FB88` or `3039230856`

### Capturing the Flag

```sh
random@ubuntu:~$ ./random
3039230856
Good!
Mommy, I thought libc random is unpredictable...
```
