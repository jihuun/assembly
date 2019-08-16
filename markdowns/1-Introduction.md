# 1. Introduction
----

<!-- toc -->

## The basic code structure

ARM assembly 소스코드의 기본적인 형태는 다음과 같다. 아래와 같은 식으로 모든 예제코드에는 주석이 있는데, 라인별 주석을 반드시 읽는것을 추천한다. 다음 예제 코드는 return 2 를 하는 단순한 C main 함수의 Assembly 버전이다.  

- first.s  

```asm
.global main		/* 'main' is our entry point and must be global */
 
main:			/* This is main */
	mov r0, #2	/* Put a 2 inside the register r0 */
	bx lr		/* Return from main */
```

라인별 설명  

```asm
.global main		/* 'main' is our entry point and must be global */
```
> main이 전역 name임을 선언함. global이어야만 C runtime이 main을 호출 가능.  
> .global label을 통해서 Linker가 label을 볼 수 있게 된다.  
> global이 아니라면 C runtime이 호출 하지 못할 뿐 더러, linking 단계에서 실패하게됨.   
 
```asm
 main:			/* This is main */
```


```asm
 	mov r0, #2	/* Put a 2 inside the register r0 */
```
> 값 2를 r0 레지스터에 이동
> 왼쪽 파라미터가 항상 destination임


```asm
	bx lr		/* Return from main */
```
> `bx`(branch and exchange)는 `lr` 레지스터에 담긴 위치로 branch (분기)하는 명령어이다.
> `lr` 레지스터는 서브루틴 함수에서 되돌아갈 주소를 지정하거나 예외처리 후 되돌아갈 주소가 저장되어 있다.  
> 만약 함수를 직접호출 하는 `bl` 명령을 사용한다면 복귀주소를 자동으로 `lr`에 저장한다. [9장 참고](# 9. Functions-1)  


```bash
 2
```
> 결과 값 2:
> `r0` 레지스터에는 항상 return value가 담긴다.  


## Compile with as

Assembly 파일(.s)을 작성하고, 그로부터 Object(.o)파일로 컴파일 하는 방법.   

```sh
 $ arm-linux-gnueabi-as -o first.o first.s
 $ arm-linux-gnueabi-gcc -o first first.s
```
> 위 내용은 우분투 환경이고 cross compiler를 사용한 것이다.   
> Target machine(튜토리얼은 Raspberry PI) 에서는 shell에서 그냥 `as -o` `gcc -o` 하면된다.   

결과는 다음과 같다.  

```sh
 $ ./first ; echo $?
 2
```
> 2가 출력됨  


참고: C파일에서 -> Asm파일 생성  

```sh
 $ arm-linux-gnueabi-gcc -S hello.c
```




