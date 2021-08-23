![logo](http://www.pwnable.kr/img/coin1.png)

# coin1
The eleventh challenge from pwnable.kr

## Description
Mommy, I wanna play a game!
(if your network response time is too slow, try nc 0 9007 inside pwnable.kr server)

Running at : nc pwnable.kr 9007

## Solution

```sh
ctftools ❯ nc pwnable.kr 9007

	---------------------------------------------------
	-              Shall we play a game?              -
	---------------------------------------------------

	You have given some gold coins .in your hand
	however, there is one counterfeit coin among them
	counterfeit coin looks exactly same as real coin
	however, its weight is different from real one
	real coin weighs 10, counterfeit coin weighes 9
	.help me to find the counterfeit coin with a scale
	.if you find 100 counterfeit coins, you will get reward :)
	FYI, you have 60 seconds.

	- How to play -
	1. you get a number of coins (N) and number of chances (C)
	2. .then you specify a .set of index numbers of coins to be weighed
	3. you get the weight information
	4. 2~3 repeats C time, .then you give the answer

	- Example -
	[Server] N=4 C=2 	# find counterfeit among 4 coins with 2 trial
	[Client] 0 1 		# weigh first and second coin
	[Server] 20			# scale result : 20
	[Client] 3			# weigh fourth coin
	[Server] 10			# scale result : 10
	[Client] 2 			# counterfeit coin is third!
	[Server] Correct!

	- Ready? starting .in 3 sec... -

```

We essentially need to split the sets of coins in half and follow the the binary search tree along the sets that are not divisible by 10, as those are the ones that contain a fake coin.

We'll use `pwntools` again for `exploit.py`.

```python
import re
from pwn import *

conn = remote('pwnable.kr', 9007)

conn.recvuntil('Ready?')

for _ in range(100):
    line = conn.recvline_regex("""N=(\d+) C=(\d+)""")
    n = int(re.search("""N=(\d+) C=(\d+)""", line).group(1))
    c = int(re.search("""N=(\d+) C=(\d+)""", line).group(2))
    left, right = 0, n

    for _ in range(c):
        set = ' '.join(str(left) for left in range(left, int((left+right)/2)))
        conn.sendline(set)
        weight = int(conn.recv(1024).strip())
        if (weight % 10 == 0):
            left = int((left+right)/2)
        else:
            right = int((left+right)/2)
    conn.sendline(str(left))

    print(conn.recv(1024).strip())
print(conn.recv(1024).strip())
```

### Capturing the Flag

```sh
ctftools ❯ python exploit.py
[+] Opening connection to pwnable.kr on port 9007: Done
...
Correct! (95)
Correct! (96)
Correct! (97)
Correct! (98)
Correct! (99)
Congrats! get your flag
b1NaRy_S34rch1nG_1s_3asy_p3asy
[*] Closed connection to pwnable.kr port 9007
```
