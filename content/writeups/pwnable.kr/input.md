![logo](http://www.pwnable.kr/img/input.png)

# input
The seventh challenge from pwnable.kr

## Description
Mom? how can I pass my input to a computer program?

`ssh input2@pwnable.kr -p2222` (pw:guest)

## Solution

```sh
input2@ubuntu:~$ ls -lh
total 24K
-r--r----- 1 input2_pwn root     55 Jun 30  2014 flag
-r-sr-x--- 1 input2_pwn input2  13K Jun 30  2014 input
-rw-r--r-- 1 root       root   1.8K Jun 30  2014 input.c
```

Nothing new here in terms of setup. Let's look at the source code.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <arpa/inet.h>

int main(int argc, char* argv[], char* envp[]){
	printf("Welcome to pwnable.kr\n");
	printf("Let's see if you know how to give input to program\n");
	printf("Just give me correct inputs then you will get the flag :)\n");

	// argv
	if(argc != 100) return 0;
	if(strcmp(argv['A'],"\x00")) return 0;
	if(strcmp(argv['B'],"\x20\x0a\x0d")) return 0;
	printf("Stage 1 clear!\n");

	// stdio
	char buf[4];
	read(0, buf, 4);
	if(memcmp(buf, "\x00\x0a\x00\xff", 4)) return 0;
	read(2, buf, 4);
        if(memcmp(buf, "\x00\x0a\x02\xff", 4)) return 0;
	printf("Stage 2 clear!\n");

	// env
	if(strcmp("\xca\xfe\xba\xbe", getenv("\xde\xad\xbe\xef"))) return 0;
	printf("Stage 3 clear!\n");

	// file
	FILE* fp = fopen("\x0a", "r");
	if(!fp) return 0;
	if( fread(buf, 4, 1, fp)!=1 ) return 0;
	if( memcmp(buf, "\x00\x00\x00\x00", 4) ) return 0;
	fclose(fp);
	printf("Stage 4 clear!\n");

	// network
	int sd, cd;
	struct sockaddr_in saddr, caddr;
	sd = socket(AF_INET, SOCK_STREAM, 0);
	if(sd == -1){
		printf("socket error, tell admin\n");
		return 0;
	}
	saddr.sin_family = AF_INET;
	saddr.sin_addr.s_addr = INADDR_ANY;
	saddr.sin_port = htons( atoi(argv['C']) );
	if(bind(sd, (struct sockaddr*)&saddr, sizeof(saddr)) < 0){
		printf("bind error, use another port\n");
    		return 1;
	}
	listen(sd, 1);
	int c = sizeof(struct sockaddr_in);
	cd = accept(sd, (struct sockaddr *)&caddr, (socklen_t*)&c);
	if(cd < 0){
		printf("accept error, tell admin\n");
		return 0;
	}
	if( recv(cd, buf, 4, 0) != 4 ) return 0;
	if(memcmp(buf, "\xde\xad\xbe\xef", 4)) return 0;
	printf("Stage 5 clear!\n");

	// here's your flag
	system("/bin/cat flag");
	return 0;
}
```

We can write a wrapper C program to pass the various stages.

Our script will call execve on `input` with modified argv and envp parameters.

#### Stage 1 : argv
```c
char* argv2[101];

for (int i = 0; i < 100; i++) {
	argv2[i] = "";
}
argv2[100] = NULL;
argv2['A'] = "\x00";
argv2['B'] = "\x20\x0a\x0d";
```

#### Stage 2 : stdio
```c
int stdin2 = open("./stdin2", O_RDWR | O_CREAT, 00777);
write(stdin2, "\x00\x0a\x00\xff", 4);
int stderr2 = open("./stderr2", O_RDWR | O_CREAT, 00777);
write(stderr2, "\x00\x0a\x02\xff", 4);

lseek(stdin2, 0, SEEK_SET);
lseek(stderr2, 0, SEEK_SET);

dup2(stdin2, 0);
dup2(stderr2, 2);
```

#### Stage 3 : env
```c
char* envp2[2];

envp2[0] = "\xca\xfe\xba\xbe=\xde\xad\xbe\xef";
envp2[1] = NULL;
```

#### Stage 4 : file
```c
int fp2 = open("./\x0a", O_RDWR | O_CREAT, 00777);
write(fp2, "\x00\x00\x00\x00", 4);
close(fp2);
```

#### Stage 5 : network
```c
argv2['C'] = "8080";
```

```sh
python -c "print('\xde\xad\xbe\xef')" | nc localhost 8080
```

Execute the above nc command in a second ssh window while the program is running and listening on the pwnable.kr server.

### Capturing the Flag
```sh
input2@ubuntu:/tmp/eboderas-input2$ ./a.out
Welcome to pwnable.kr
Let's see if you know how to give input to program
Just give me correct inputs then you will get the flag :)
Stage 1 clear!
Stage 2 clear!
Stage 3 clear!
Stage 4 clear!
Stage 5 clear!
Mommy! I learned how to pass various input in Linux :)
```
