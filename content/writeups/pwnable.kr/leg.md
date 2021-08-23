![logo](http://www.pwnable.kr/img/leg.png)

# leg
The eighth challenge from pwnable.kr

## Description
Daddy told me I should study arm.
But I prefer to study my leg!

Download : http://pwnable.kr/bin/leg.c
Download : http://pwnable.kr/bin/leg.asm

ssh leg@pwnable.kr -p2222 (pw:guest)

## Solution

Let's try to ssh into the server and see what we can find.

```sh
ctftools ❯ ssh leg@pwnable.kr -p2222
```

We're immediately greeted with a bunch of error logs dumped to the screen, but let's look around.

```sh
/ $ ls -la
total 628
drwxr-xr-x   11 root     0                0 Nov 10  2014 .
drwxr-xr-x   11 root     0                0 Nov 10  2014 ..
drwxrwxr-x    2 root     0                0 Nov 10  2014 bin
drwxrwxr-x    2 root     0                0 Nov 10  2014 boot
drwxrwxr-x    2 root     0                0 Nov 10  2014 dev
drwxrwxr-x    3 root     0                0 Nov 10  2014 etc
-r--------    1 1001     0               38 Nov 10  2014 flag
---s--x---    1 1001     1000        636419 Nov 10  2014 leg
lrwxrwxrwx    1 root     0               11 Nov 10  2014 linuxrc -> bin/busybox
dr-xr-xr-x   33 root     0                0 Jan  1 00:00 proc
drwxrwxr-x    2 root     0                0 Nov 10  2014 root
drwxrwxr-x    2 root     0                0 Nov 10  2014 sbin
drwxrwxr-x    2 root     0                0 Nov 10  2014 sys
drwxrwxr-x    4 root     0                0 Nov 10  2014 usr
```

We'll need `leg` to read `flag` for us.

Let's examine the C and ASM source code of the two provided files.

```c
#include <stdio.h>
#include <fcntl.h>
int key1(){
	asm("mov r3, pc\n");
}
int key2(){
	asm(
	"push	{r6}\n"
	"add	r6, pc, $1\n"
	"bx	r6\n"
	".code   16\n"
	"mov	r3, pc\n"
	"add	r3, $0x4\n"
	"push	{r3}\n"
	"pop	{pc}\n"
	".code	32\n"
	"pop	{r6}\n"
	);
}
int key3(){
	asm("mov r3, lr\n");
}
int main(){
	int key=0;
	printf("Daddy has very strong arm! : ");
	scanf("%d", &key);
	if( (key1()+key2()+key3()) == key ){
		printf("Congratz!\n");
		int fd = open("flag", O_RDONLY);
		char buf[100];
		int r = read(fd, buf, 100);
		write(0, buf, r);
	}
	else{
		printf("I have strong leg :P\n");
	}
	return 0;
}
```

```asm
(gdb) disass main
Dump of assembler code for function main:
   0x00008d3c <+0>:	push	{r4, r11, lr}
   0x00008d40 <+4>:	add	r11, sp, #8
   0x00008d44 <+8>:	sub	sp, sp, #12
   0x00008d48 <+12>:	mov	r3, #0
   0x00008d4c <+16>:	str	r3, [r11, #-16]
   0x00008d50 <+20>:	ldr	r0, [pc, #104]	; 0x8dc0 <main+132>
   0x00008d54 <+24>:	bl	0xfb6c <printf>
   0x00008d58 <+28>:	sub	r3, r11, #16
   0x00008d5c <+32>:	ldr	r0, [pc, #96]	; 0x8dc4 <main+136>
   0x00008d60 <+36>:	mov	r1, r3
   0x00008d64 <+40>:	bl	0xfbd8 <__isoc99_scanf>
   0x00008d68 <+44>:	bl	0x8cd4 <key1>
   0x00008d6c <+48>:	mov	r4, r0
   0x00008d70 <+52>:	bl	0x8cf0 <key2>
   0x00008d74 <+56>:	mov	r3, r0
   0x00008d78 <+60>:	add	r4, r4, r3
   0x00008d7c <+64>:	bl	0x8d20 <key3>
   0x00008d80 <+68>:	mov	r3, r0
   0x00008d84 <+72>:	add	r2, r4, r3
   0x00008d88 <+76>:	ldr	r3, [r11, #-16]
   0x00008d8c <+80>:	cmp	r2, r3
   0x00008d90 <+84>:	bne	0x8da8 <main+108>
   0x00008d94 <+88>:	ldr	r0, [pc, #44]	; 0x8dc8 <main+140>
   0x00008d98 <+92>:	bl	0x1050c <puts>
   0x00008d9c <+96>:	ldr	r0, [pc, #40]	; 0x8dcc <main+144>
   0x00008da0 <+100>:	bl	0xf89c <system>
   0x00008da4 <+104>:	b	0x8db0 <main+116>
   0x00008da8 <+108>:	ldr	r0, [pc, #32]	; 0x8dd0 <main+148>
   0x00008dac <+112>:	bl	0x1050c <puts>
   0x00008db0 <+116>:	mov	r3, #0
   0x00008db4 <+120>:	mov	r0, r3
   0x00008db8 <+124>:	sub	sp, r11, #8
   0x00008dbc <+128>:	pop	{r4, r11, pc}
   0x00008dc0 <+132>:	andeq	r10, r6, r12, lsl #9
   0x00008dc4 <+136>:	andeq	r10, r6, r12, lsr #9
   0x00008dc8 <+140>:			; <UNDEFINED> instruction: 0x0006a4b0
   0x00008dcc <+144>:			; <UNDEFINED> instruction: 0x0006a4bc
   0x00008dd0 <+148>:	andeq	r10, r6, r4, asr #9
End of assembler dump.
(gdb) disass key1
Dump of assembler code for function key1:
   0x00008cd4 <+0>:	push	{r11}		; (str r11, [sp, #-4]!)
   0x00008cd8 <+4>:	add	r11, sp, #0
   0x00008cdc <+8>:	mov	r3, pc
   0x00008ce0 <+12>:	mov	r0, r3
   0x00008ce4 <+16>:	sub	sp, r11, #0
   0x00008ce8 <+20>:	pop	{r11}		; (ldr r11, [sp], #4)
   0x00008cec <+24>:	bx	lr
End of assembler dump.
(gdb) disass key2
Dump of assembler code for function key2:
   0x00008cf0 <+0>:	push	{r11}		; (str r11, [sp, #-4]!)
   0x00008cf4 <+4>:	add	r11, sp, #0
   0x00008cf8 <+8>:	push	{r6}		; (str r6, [sp, #-4]!)
   0x00008cfc <+12>:	add	r6, pc, #1
   0x00008d00 <+16>:	bx	r6
   0x00008d04 <+20>:	mov	r3, pc
   0x00008d06 <+22>:	adds	r3, #4
   0x00008d08 <+24>:	push	{r3}
   0x00008d0a <+26>:	pop	{pc}
   0x00008d0c <+28>:	pop	{r6}		; (ldr r6, [sp], #4)
   0x00008d10 <+32>:	mov	r0, r3
   0x00008d14 <+36>:	sub	sp, r11, #0
   0x00008d18 <+40>:	pop	{r11}		; (ldr r11, [sp], #4)
   0x00008d1c <+44>:	bx	lr
End of assembler dump.
(gdb) disass key3
Dump of assembler code for function key3:
   0x00008d20 <+0>:	push	{r11}		; (str r11, [sp, #-4]!)
   0x00008d24 <+4>:	add	r11, sp, #0
   0x00008d28 <+8>:	mov	r3, lr
   0x00008d2c <+12>:	mov	r0, r3
   0x00008d30 <+16>:	sub	sp, r11, #0
   0x00008d34 <+20>:	pop	{r11}		; (ldr r11, [sp], #4)
   0x00008d38 <+24>:	bx	lr
End of assembler dump.
(gdb)
```

Let's try to compile the C code.

```sh
ctftools ❯ gcc -Wall leg.c
leg.c: In function ‘main’:
leg.c:31:3: warning: implicit declaration of function ‘read’ [-Wimplicit-function-declaration]
   int r = read(fd, buf, 100);
   ^
leg.c:32:3: warning: implicit declaration of function ‘write’ [-Wimplicit-function-declaration]
   write(0, buf, r);
   ^
leg.c: In function ‘key1’:
leg.c:5:1: warning: control reaches end of non-void function [-Wreturn-type]
 }
 ^
leg.c: In function ‘key2’:
leg.c:19:1: warning: control reaches end of non-void function [-Wreturn-type]
 }
 ^
leg.c: In function ‘key3’:
leg.c:22:1: warning: control reaches end of non-void function [-Wreturn-type]
 }
 ^
leg.c: Assembler messages:
leg.c:4: Error: too many memory references for `mov'
leg.c:7: Error: invalid char '{' beginning operand 1 `{r6}'
leg.c:8: Error: too many memory references for `add'
leg.c:9: Error: no such instruction: `bx r6'
leg.c:10: Error: unknown pseudo-op: `.code'
leg.c:11: Error: too many memory references for `mov'
leg.c:12: Error: operand size mismatch for `add'
leg.c:13: Error: invalid char '{' beginning operand 1 `{r3}'
leg.c:14: Error: invalid char '{' beginning operand 1 `{pc}'
leg.c:15: Error: unknown pseudo-op: `.code'
leg.c:16: Error: invalid char '{' beginning operand 1 `{r6}'
leg.c:21: Error: too many memory references for `mov'
```

This is ARM code, so we're unable to compile on an x86 machine.

According to the C program, we can print `flag` if we make it inside the following condition:

```c
if( (key1()+key2()+key3()) == key )
```

Let's start reversing each `key` function.

#### Key 1

```c
int key1(){
	asm("mov r3, pc\n");
}
```

`pc` in ARM is the value of the current instruction plus 8 bytes, so `key1()` should return the address of the second next instruction at that moment.

```asm
0x00008cdc <+8>:	mov	r3, pc
0x00008ce0 <+12>:	mov	r0, r3
0x00008ce4 <+16>:	sub	sp, r11, #0
```

`key1()` is equal to `0x00008ce4`.

#### Key 2

```c
int key2(){
	asm(
	"push	{r6}\n"
	"add	r6, pc, $1\n"
	"bx	r6\n"
	".code   16\n"
	"mov	r3, pc\n"
	"add	r3, $0x4\n"
	"push	{r3}\n"
	"pop	{pc}\n"
	".code	32\n"
	"pop	{r6}\n"
	);
}
```

The first section of this function puts the CPU into Thumb State with the `bx r6` instruction, with the single bit set from the previous command indicating this state.

The CPU is now in 16-bit mode, which means values like `pc` are +4, not +8.

Next the program adds `0x4` to `pc`, then returns that value. So let's take a look at the assembly to get the right addresses.

```asm
0x00008cf0 <+0>:	push	{r11}		; (str r11, [sp, #-4]!)
0x00008cf4 <+4>:	add	r11, sp, #0
0x00008cf8 <+8>:	push	{r6}		; (str r6, [sp, #-4]!)
0x00008cfc <+12>:	add	r6, pc, #1 ; r6 = 0x00008d04 + 1 =
0x00008d00 <+16>:	bx	r6
0x00008d04 <+20>:	mov	r3, pc
0x00008d06 <+22>:	adds	r3, #4 ; r3 = 0x00008d08 + 4 = 0x00008d0c
0x00008d08 <+24>:	push	{r3}
0x00008d0a <+26>:	pop	{pc}
0x00008d0c <+28>:	pop	{r6}		; (ldr r6, [sp], #4)
0x00008d10 <+32>:	mov	r0, r3
0x00008d14 <+36>:	sub	sp, r11, #0
0x00008d18 <+40>:	pop	{r11}		; (ldr r11, [sp], #4)
0x00008d1c <+44>:	bx	lr
```

At instruction `0x00008d04`, the `pc` is `0x00008d08`, and adding `4` gives us `0x00008d0c`.

After that, the program pushes this value stored in `r3` onto the stack, then returns from Thumb State after popping `r6`, and moves `r3` into `r0`, the return register.

`key2()` is equal to `0x00008d0c`

#### Key 3

```c
int key3(){
	asm("mov r3, lr\n");
}
```

`lr` in ARM is Link Register, which stores the return address of the function.

```asm
0x00008d7c <+64>:	bl	0x8d20 <key3>
0x00008d80 <+68>:	mov	r3, r0
```

`key3()` is equal to `0x00008d80`.


`key1()+key2()+key3()` is `0x00008ce4 + 0x00008d0c + 0x00008d80 = 0x1a770` which is `108400` in decimal.


### Capturing the Flag

```sh
/ $ ./leg
Daddy has very strong arm! : 108400
Congratz!
My daddy has a lot of ARMv5te muscle!
```
