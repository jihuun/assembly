# Learning ARM32 Assembly Language
----

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; __작성: 김지훈, System 개발 2그룹(ji_hun.kim@samsung.com), 2018-07__



----

## Table Of Contents

[TOC]

<div style="page-break-after: always;"></div>


# 1. Introduction
----
> 작성: 김지훈, 2018-07, ji_hun.kim@samsung.com 

이 문서는 ARM Assembly 명령어를 쉽게 배울 수 있도록 설명한 튜토리얼이다. Assembly언어를 전혀 몰라도 충분히 읽을 수 있는 Beginner용으로, 32bit ARMv6 Architecture 기반의 Assembly 언어를 매우 익힐 수있다. 또한 Architecture를 떠나서 일반적인 Assembly Language 의 개념을 익힐 수 있다. 시중에는 많은 ARM assembly 언어 참고자료가 있지만 이 튜토리얼 만큼 초보자가 체계적으로 쉽게 익힐 수 있는 컨텐츠는 본적이 없었던것 같다. 참고로, 튜토리얼은 Raspberry PI 2를 Target machine으로 두고 작성되었지만 내용과는 크게 상관이 없어서 해당 machine이 없어도 무방하다.   

이 문서는 주로 Roger Ferrer Ibanez라는 ARM 엔지니어가 작성한 튜토리얼에서 내가 배운것들을 정리한 문서이다. 이 문서는 거의 요약정리이기 때문에 설명이 부족하다면 아래 튜토리얼에서 각 챕터의 내용을 참고하기 바란다. This documentation is what I've learned from a tutorial below writen by Roger Ferrer Ibanez, an ARM engineer.  

[튜토리얼 링크: http://thinkingeek.com/arm-assembler-raspberry-pi/](http://thinkingeek.com/arm-assembler-raspberry-pi)  

다음은 ARM 64bit assembly 언어에 대한 post로 추후에 해당 내용도 업데이트 할 예정이다.  
https://thinkingeek.com/2016/10/08/exploring-aarch64-assembler-chapter1/  
https://thinkingeek.com/2016/10/08/exploring-aarch64-assembler-chapter-2/  
https://thinkingeek.com/2016/10/23/exploring-aarch64-assembler-chapter-3/  
https://thinkingeek.com/2016/10/23/exploring-aarch64-assembler-chapter-4/  
https://thinkingeek.com/2016/11/13/exploring-aarch64-assembler-chapter-5/  
https://thinkingeek.com/2016/11/27/exploring-aarch64-assembler-chapter-6/  
https://thinkingeek.com/2017/03/19/exploring-aarch64-assembler-chapter-7/  
https://thinkingeek.com/2017/05/29/exploring-aarch64-assembler-chapter-8/  
https://thinkingeek.com/2017/11/05/exploring-aarch64-assembler-chapter-9/  

---

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





<div style="page-break-after: always;"></div>


# 2. Registers and Basic arithmetic
----

두 `r#` 레지스터를 더하는 방법

- sum01.s

	```asm
	.global main
	 
	main:
	    mov r1, #3      /* r1 <- 3 */
	    mov r2, #4      /* r2 <- 4 */
	    add r0, r1, r2  /* r0 <- r1 + r2 */
	    bx lr
	```
	> r1 r2를 더해 r0 에 저장  


- sum02.s

	```asm
	.global main
	 
	main:
	    mov r0, #3      /* r0 <- 3 */
	    mov r1, #4      /* r1 <- 4 */
	    add r0, r0, r1  /* r0 <- r0 + r1 */
	    bx lr
	```
	> r0 를 재활용함.  

- Run on the target RPI after compile

	```bash
	 $ ./sum01 ; echo $?
	 7

	 $ ./sum02 ; echo $?
	 7
	```

<div style="page-break-after: always;"></div>

# 3. Memory, addreses, Load and store
----

2장에서는 `mov` 명령어를 이용해 레지스터간 데이터를 이동하는 방법을 살펴보았다. 만약 프로세서가 이 기능만을 지원한다면 매우 제약적일것이다. 3장에서는 메모리에서 레지스터로 데이터를 주고 받는 방법에 대해 살펴본다.  

## Momory

Load and Store operations are the ways CPU could operate or access the memory  

`ldr` (load):	register <----data---- memory   
`str` (store):	register -----data---> memory  

## Addreses
메모리에 접근하기 위해서는 해당하는 이름이 필요하다. 하지만 모든 메모리 조각에 이름을 붙일 수 없다. 그래서 주소를 사용한다.  

Data를 loading & store하기 위해서는 주소를 계산해야한다. 다양한 방식으로 주소가 계산 될 수 있는데, 이것을 Addressing Mode라고 한다. 
괄호 []를 씌우면 []내에 있는 주소값에 저장된 값을 의미하게 된다. [r1] -> *r1 과 동일 

![https://thinkingeek.com/wp-content/uploads/2013/01/load-282x300.png](https://thinkingeek.com/wp-content/uploads/2013/01/load-282x300.png)
> byte 순서가 little-endian 임 유의  

## Data
label: is symbolic name to addresses in your program. 주소를 나타낸다.

```asm
.balign 4
myval1:
	.word 3
```
> .balign 4  
다음 주소가 4바이트 단위로 시작할 것이라는것을 ensure하는 명령  
이 지시자 다음 나오는 심볼은 한줄 한줄이 4byte offset을 갖는 다는 의미이다.  
> myval1: 
이 변수의 주소를 정의  
> .word 3  
4byte integer 데이터 타입임을 지시, 그리고 3으로 초기화  


## Sections
Data는 메모리에 상주 한다. .data라고 그 영역을 지정할 수 있다. 
.text는 코드 영역

## load and store

```asm
/* -- store01.s */
 
.data				/* -- Data section */
 
.balign 4			/* Ensure variable is 4-byte aligned */
myvar1:				/* Define storage for myvar1 */
    .word 0			/* Contents of myvar1 is just '3' */
 
.balign 4			/* Ensure variable is 4-byte aligned */
myvar2:				/* Define storage for myvar2 */
    .word 0			/* Contents of myvar2 is just '3' */
 
.text				/* -- Code section */
 
.balign 4			/* Ensure variable is 4-byte aligned */
.global main
main:
    ldr r1, addr_of_myvar1	/* r1 <- &myvar1 */
    mov r3, #3            	/* r3 <- 3 */
    str r3, [r1]          	/* *r1 <- r3 */
    ldr r2, addr_of_myvar2 	/* r2 <- &myvar2 */
    mov r3, #4             	/* r3 <- 4 */
    str r3, [r2]           	/* *r2 <- r3 */
 
    ldr r1, addr_of_myvar1 	/* r1 <- &myvar1 */
    ldr r1, [r1]           	/* r1 <- *r1 */
    ldr r2, addr_of_myvar2 	/* r2 <- &myvar2 */
    ldr r2, [r2]           	/* r2 <- *r2 */
    add r0, r1, r2
    bx lr
 
/* Labels needed to access data */

addr_of_myvar1 : .word myvar1
addr_of_myvar2 : .word myvar2
```

라인별 설명:  

```asm
addr_of_myvar1 : .word myvar1
addr_of_myvar2 : .word myvar2
```
> 다른 section에 있는 심볼을 바로 access할 수 없다. 따라서 .data에 있는 myval 주소를 .text영역으로 가져온 것.
> ld가 relrocation 함.  



```asm
    ldr r1, addr_of_myval1 	/* r1 <- &myvar1 */
    ldr r1, [r1]           	/* r1 <- *r1 */
```
relocation된 myval1주소를를 r1에 넣은뒤,  
myval1의 주소가 담겨있던 r1에 다시 myval1주소가 가지고 있던 값을 넣음.  
[r1] <- []는 addressing mode임. 이것은 r1이 저장된 주소값을 의미한다.   

```asm
    ldr r1, addr_of_myvar1	/* r1 <- &myvar1 */
    mov r3, #3            	/* r3 <- 3 */
    str r3, [r1]          	/* *r1 <- r3 */
```
relocation된 myval1주소를를 r1에 넣은뒤,  
r3에 값3을 초기화.  
myval1의 주소가 담겨있던 r1이 가리키는 값(주소)에 r3에 담겨있는 3을 store함.  




<div style="page-break-after: always;"></div>

# 4. GDB
----

[생략한다.](https://thinkingeek.com/2013/01/12/arm-assembler-raspberry-pi-chapter-4/)

<div style="page-break-after: always;"></div>

# 5. Branches
----


## r15, a PC register

`pc` has address which is next instruction going to be executed.
__The branching is same with changing `pc` register value.__
ARM has own branch instruction for modifying `pc` register.

`pc` (Program Counter) 는 다음에 실행될 명령어 주소가 담겨있다.   

## Unconditional branches

Instruction `b` with label is used for unconditional branching.

```asm
.text
.global main
main:
    mov r0, #2 /* r0 ←  2 */
    b end      /* branch to 'end' */
    mov r0, #3 /* r0 ←  3 */
end:
    bx lr
```

### 참고: branch instruction 종류

- `b` : branch <immediate>  
명령어 뒤에 지정된 상수값(혹은 Label)에 해당하는 주소로 분기  

- `bl` : branch with link <immediate>  
명령어 뒤에 지정된 상수값(혹은 Label)에 해당하는 주소로 분기 하되, 현재의 `pc` + 0x4 번지의 복귀 주소값을 `lr` 레지스터에 저장   

- `bx` : branch indirect <register>  
명령어 뒤에 지정된 레지스터로 분기  

- `blx` : branch indirect with link <register>  
명령어 뒤에 지정된 레지스터로 분기 하되,  현재의 `pc` + 0x4 번지의 복귀 주소값을 `lr` 레지스터에 저장    

	출처 [http://trace32.com/wiki/index.php/B,_BL,_BX_and_BLX](http://trace32.com/wiki/index.php/B,_BL,_BX_and_BLX)  

## Conditional branches

### CPSR (Current Program Status Register)

It has flags called N (negative), Z (zero), C (carry) and V (overflow).  
These condition flags are read by branch instructions. And,  
Arithmetic instruction/ special testing/ comparison instruction이 이 상태 레지스터 값을 update 할 수 있다.

	* N : Negative : 연산결과가 마이너스인 경우에 set됨
	* Z : Zero : 연산결과가 0인 경우에 set됨
	* C : Carry : 연산결과에 자리 올림이 발생한 경우에 set됨.
	* V : oVer flow : 연산의 결과가 overflow 났을 경우에 set됨.
		(by any instructions)
```
cmp r1, r2
```
> will update cpsr doing "r1 - r2", (r1, r2값은 바뀌지 않음)  
cmp 명령어만 cpsr 을 update할 수 있다? -> [#12장 참고](#12. Loops and the status register)  
>
> r1 < r2 인경우:	N updated   
r1 == r2 인경우:	Z updated  
r1 == 1, r2 == 0 인경우: C enabled(1) (no carries)  
r1 == 0, r2 == 1 인경우: C disabled(0)  
r1 == 2147483647(+ integer 최대 값), r2 == -1 인 경우: Overflow, V updated  

```txt
EQ	(equal) When Z is enabled (Z is 1)
NE 	(not equal). When Z is disabled. (Z is 0)
GE 	(greater or equal than, in two’s complement).
	When both V and N are enabled or disabled (V is N)
LT 	(lower than, in two’s complement).
	This is the opposite of GE, so when V and N are not both enabled or disabled
	(V is not N)
GT 	(greather than, in two’s complement).
	When Z is disabled and N and V are both enabled or disabled
	(Z is 0, N is V)
LE 	(lower or equal than, in two’s complement).
	When Z is enabled or if not that, N and V are both enabled or disabled
	(Z is 1. If Z is not 1 then N is V)
MI 	(minus/negative) When N is enabled (N is 1)
PL 	(plus/positive or zero) When N is disabled (N is 0)
VS 	(overflow set) When V is enabled (V is 1)
VC 	(overflow clear) When V is disabled (V is 0)
HI 	(higher) When C is enabled and Z is disabled (C is 1 and Z is 0)
LS 	(lower or same) When C is disabled or Z is enabled (C is 0 or Z is 1)
CS/HS 	(carry set/higher or same) When C is enabled (C is 1)
CC/LO 	(carry clear/lower) When C is disabled (C is 0)
```
> 위 condition들이 b 와 합쳐져서 instruction 이름이 됨.

가령 beq는 Z가 1일 때만 branch함.


```asm
/* -- compare01.s */
.text
.global main
main:
    mov r1, #2       /* r1 ←  2 */
    mov r2, #2       /* r2 ←  2 */
    cmp r1, r2       /* update cpsr condition codes with the value of r1-r2 */
    beq case_equal   /* branch to case_equal only if Z = 1 */
case_different :
    mov r0, #2       /* r0 ←  2 */
    b end            /* branch to end */
case_equal:
    mov r0, #1       /* r0 ←  1 */
end:
    bx lr
```




<div style="page-break-after: always;"></div>

# 6. Control Structures
----


## 조건문 if, then, else

```c
if (E) then
	S1
else
	S2
```

> 이런 high level 프로그래밍 구조가 Assembly 에선 아래와 같은 구조가 됨.  

```asm
if_eval:	/* Assembler that evaluates E and updates the cpsr accordingly */

bXX else	/* Here XX is the appropiate condition */
then_part:	/* assembler for S1, the "then" part */
    b end_of_if
else:		/* assembler for S2, the "else" part */
end_of_if:
```

## 반복문 Loops

```c
while (E)
	s
```

```asm
while_condition:	/* assembler to evaluate E and update cpsr */
    bXX end_of_loop	/* If E is false, then leave the loop right now */

    			/* assembler of S */
    b while_condition	/* Unconditional branch to the beginning */
end_of_loop:
```

## Asm code for 1+2+3+4+...+22

- 내가 짜본 코드: 틀렸음
	```asm
	mov r1 #0
	mov r0 #1
	while_condition:
		bvs end_of_loop	/* overflow condition for cpsr */
		mov r1 r0	/* r1 1 */
		add r0 r0 #1 	/* r0 1+1 */
		add r1 r1 r0	/* r1 1 + 2 */
		b while_condition
	end_of_loop:
	```
	> my code: 22까지 더하면 253인데 overflow 발생한다고 생각했음.


- loop01.s
Reference code

	```asm
	/* -- loop01.s */
	.text
	.global main
	main:
	    mov r1, #0       /* r1 <- 0 (sum val) */
	    mov r2, #1       /* r2 <- 1 (counter val) */
	loop: 
	    cmp r2, #22      /* compare r2 and 22 */
	    bgt end          /* branch if r2 > 22 to end */
	    add r1, r1, r2   /* r1 <- r1 + r2 */
	    add r2, r2, #1   /* r2 <- r2 + 1 */
	    b loop
	end:
	    mov r0, r1       /* r0 <- r1 */
	    bx lr
	```

	```asm
	    cmp r2, #22 
	    bgt end    
	```
	> cpsr will be updated by cmp operation, then branch out by "gt" flag 

- 참고: loop02.s
loop01 보다 개선된 버전이다.  

	```asm
	.text
	.global main
	main:
	    mov r1, #0       /* r1 ← 0 */
	    mov r2, #1       /* r2 ← 1 */
	    b check_loop     /* unconditionally jump at the end of the loop */
	loop: 
	    add r1, r1, r2   /* r1 ← r1 + r2 */
	    add r2, r2, #1   /* r2 ← r2 + 1 */
	check_loop:
	    cmp r2, #22      /* compare r2 and 22 */
	    ble loop         /* branch if r2 <= 22 to the beginning of the loop */
	end:
	    mov r0, r1       /* r0 ← r1 */
	    bx lr
	```
	> 01과 명령어 갯수는 동일하다. 하지만 01과 다르게 마지막 조건에서 branching 하는 횟수가 하나 적다. 
	> ble loop 다음 end: 로 branching없이 진행되기 때문이다.  


## Asm code for 3n + 1 problems

다른 말로 [콜라츠 추측](https://ko.wikipedia.org/wiki/%EC%BD%9C%EB%9D%BC%EC%B8%A0_%EC%B6%94%EC%B8%A1) 이라고 부른다  
> 임의의 자연수가 다음 조작을 거쳐 항상 1이 된다는 추측  
	1. 짝수라면 2로 나눈다.  
	2. 홀수라면 3을 곱하고 1을 더한다.  
	3. 1이면 조작을 멈추고, 1이 아니면 첫 번째 단계로 돌아간다.  
	예를 들어, 6 에서 시작한다면, 차례로 6, 3, 10, 5, 16, 8, 4, 2, 1 이 된다.

```c
n = ...;
while (n != 1)
{
  if (n % 2 == 0)
     n = n / 2;
  else
     n = 3*n + 1;
}
```

```asm
/* -- collatz.s */
.text
.global main
main:
    mov r1, #123           /* r1 <- 123 */
    mov r2, #0             /* r2 <- 0 */
loop:
    cmp r1, #1             /* compare r1 and 1 */
    beq end                /* branch to end if r1 == 1 */
 
    and r3, r1, #1         /* r3 <- r1 & 1 */
    cmp r3, #0             /* compare r3 and 0 */
    bne odd                /* branch to odd if r3 != 0 */
even:
    mov r1, r1, ASR #1     /* r1 <- (r1 >> 1) */
    b end_loop
odd:
    add r1, r1, r1, LSL #1 /* r1 <- r1 + (r1 << 1) */
    add r1, r1, #1         /* r1 <- r1 + 1 */
 
end_loop:
    add r2, r2, #1         /* r2 <- r2 + 1 */
    b loop                 /* branch to loop */
 
end:
    mov r0, r2
    bx lr
```


```asm
    and r3, r1, #1         /* r3 <- r1 & 1 */
```
> bitwise operator (AND)

```asm
    mov r1, r1, ASR #1     /* r1 <- (r1 >> 1) */
```
> ASR (arithmetic shift right) 로 추가 reg사용없이 쉬프트 가능 for r1 <- r1/2

```asm
    add r1, r1, r1, LSL #1
```
> LSL (logical shift left) r1 x 2 한 뒤 다시 r1을 더해서 r1 x 3이 됨.




<div style="page-break-after: always;"></div>

# 7. Indexing modes
----


## Indexing modes란?

ARM 명령어는 operand로 register또는 immediate value 를 사용한다.(load, store, branch 제외)

```
명령어 Rdest, Rsource1, source2
```
Operand는 레지스터 이거나 혹은 값이 될 수 있다.  
첫번째 Operand 즉, Rdest위치에 항상 destination이 온다. (STR 예외)   
Rsource1 에는 항상 register가 온다.  
source2에는 register가 올 수도 있고, immediate value가 올 수도 있다.  

이렇게 명령어의 Operand 집합을 총칭하여 인덱싱 모드라고 한다.   

## Shifted operand

ARM은 Shift (좌/우) operation 에 관련 명령어가 없다.  
Operand는 Shift operation과 결합될 수 있다.  

```
LSL #n 		Logical Shift Left. 
LSL Rsource3
LSR #n		Logical Shift Right.
LSR Rsource3
ASR #n		Arithmetic Shift Right.
ASR Rsource3
ROR #n	 	Rotate Right.
ROR Rsource3
```
> source2 

```asm
mov r1, r2, LSL #1
```

## Usage of Shifted operand 

```
mov r1, r2, LSL #1      /* r1 ← (r2*2) */
mov r1, r2, LSL #2      /* r1 ← (r2*4) */
mov r1, r3, ASR #3      /* r1 ← (r3/8) */
mov r3, #4
mov r1, r2, LSL r3      /* r1 ← (r2*16) */
```
> shift 하여 2의 배수 곱셈 가능

```
add r1, r2, r2, LSL #1   /* r1 ← r2 + (r2*2) equivalent to r1 ← r2*3 */
add r1, r2, r2, LSL #2   /* r1 ← r2 + (r2*4) equivalent to r1 ← r2*5 */
```
> 이렇게 하면 홀수배도 곱셈 가능

```
sub r1, r2, r2, LSL #3  /* r1 ← r2 - (r2*8) equivalent to r1 ← r2*(-7)
```
> 음수배 곱셈 가능

```
rsb r1, r2, r2, LSL #3      /* r1 ← (r2*8) - r2 equivalent to r1 ← r2*7 */
```
> rsb 로 곱셈 가능
> rsb (Reverse Substract)

```
sub Rd A B
```
> A - B

```
rsb Rd A B
```
> B - A
> Shift 연산에서 어떤 값을 뺄때 유용



- 왜 더 간단한 곱셈연산을 사용하지 않나?

일반 곱셈연산은 연산량이 더 많다. 계산하는데  오버헤드가 더 크다. 





<div style="page-break-after: always;"></div>

# 8. Arrays and structures
----

이번 장에서는 Indexing mode for load and store 에 대해 알아본다.  
괄호 [r#] 는 r# 이 저장된 주소값을 의미한다.

## Array, structure

- 배열의 asm 코드

```c
int a[100];
```
> in C

```asm
.data
 
.balign 4
a: .skip 400
```
> in assembly
> 100 x 4 의 공간을 만든다.
> .skip 지시자: 어셈블러에게 미리 메모리 공간을 할당한다고 지시


- 구조체의 asm코드

```c
struct my_struct
{
    char f0;
    int f1;
} b;
```
> in C

```asm
.balign 4
b: .skip 8
```
> in assembly
> 왜 char 멤버가 있는데도 8바이트를 할당? C코드에서 Padding 기법 으로 alignment를 하지 않았기 때문이다.


## 배열 초기화: Indexing modes 사용 안함

```c
for (i = 0; i < 100; i++)
    a[i] = i;
```

```asm
/********** data **********/
.data
 
.balign 4
a: .skip 400

.balign 4
b: .skip 8

/********** text **********/
.text
 
.global main
main:
    ldr r1, addr_of_a       /* r1 <- &a , r1은 a의 base 주소 */
    mov r2, #0              /* r2 <- 0 */
loop:
    cmp r2, #100            /* Have we reached 100 yet<- */
    beq end                 /* If so, leave the loop, otherwise continue */
    add r3, r1, r2, LSL #2  /* r3 <- r1 + (r2*4): 배열의 주소 계산, index계산 */
    str r2, [r3]            /* *r3 <- r2 , Addressing modes */
    add r2, r2, #1          /* r2 <- r2 + 1, r2: count 변수 i */
    b loop                  /* Go to the beginning of the loop */
end:
    bx lr
addr_of_a: .word a
```
> addr_of_a: .data영역의 심볼을 .text영역으로 가져온것임. (#3장 참고)



```asm
    add r3, r1, r2, LSL #2  /* r3 <- r1 + (r2*4) */
```
> r3는 배열 a의 base주소인 r1에서부터 4byte씩 증가



## 배열 초기화: Indexing mode를 사용한 

ARM instruction set은 위의 asm코드 보다 compact한 방법을 제공한다!  
진정한 의미의 indexing mode임  

## ARM이 제공하는 9개의 인덱싱 모드 형태

```
1) [Rsource1, #+immediate]
2) [Rsource1, +Rsource2]
3) [Rsource1, +Rsource2, shift_operation #immediate]

4) [Rsource1], #+immediate 
5) [Rsource1], +Rsource2
6) [Rsource1], +Rsource2, shift_operation #immediate

7) [Rsource1, #+immediate]!
8) [Rsource1, +Rsource2]!
9) [Rsource1, +Rsource2, shift_operation #immediate]!
```



## 1. Non updating indexing modes

#### (1) [Rsource1, #+immediate] or [Rsource1, #-immediate]

```asm
mov r2, #3          /* r2 ← 3 */
str r2, [r1, #+12]  /* *(r1 + 12) ← r2 */
```
> 배열 a의 base 주소 r1으로부터 인덱스[3]에 12byte 직접 접근   
> immediate 크기는 12비트를 초과할 수 없다.    
  
#### (2) [Rsource1, +Rsource2] or [Rsource1, -Rsource2]  
  
```asm  
mov r2, #3          /* r2 ← 3 */  
mov r3, #12         /* r3 ← 12 */  
str r2, [r1, +r3]   /* *(r1 + r3) ← r2 */  
```  
> offset을 값대신 reg에 있는 값 사용.  
> offset 값이 큰 경우 유용  
  
#### (3) [Rsource1, +Rsource2, shift_operation #immediate]  
			또는 [Rsource1, -Rsource2, shift_operation #immediate]  
  
```asm  
add r3, r1, r2, LSL #2  /* r3 ← r1 + r2*4 */  
str r2, [r3]            /* *r3 ← r2 */  
```  
> 이 방법이 아래와 같은 indexing mode로 compact 해짐  
```asm  
str r2, [r1, +r2, LSL #2]  /* *(r1 + r2*4) ← r2 */  
```  
> r1 base주소에 r2 x 4 에 해당하는 index에 r2 값 store  
> 주소값을 고정된 상수로 곱할 때 유용  
  
  
## 2. Updating indexing modes  
  
순차적으로 접근한다면 매번 base주소에 offset을 계산할 필요가 없다.  
no need to compute everytime from the beginning the address of the next item   
  
  
### a) Post-indexing modes  
  
#### (4) [Rsource1], #+immediate 또는 [Rsource1], #-immediate  
  
```asm  
loop:  
    cmp r2, #100            /* Have we reached 100 yet? */  
    beq end                 /* If so, leave the loop, otherwise continue */  
    str r2, [r1], #4        /* *r1 ← r2 then r1 ← r1 + 4 */  
    add r2, r2, #1          /* r2 ← r2 + 1 */  
    b loop                  /* Go to the beginning of the loop */  
end:  
```  
> base 주소 r1자체가 4씩 증가되고 값이 저장됨  
> 먼저 r2를 a[r1]에 저장한 뒤, r1+=4 함.  
> `#`뒤에 +는 생략가능?  
  
#### (5) [Rsource1], +Rsource2 또는 [Rsource1], -Rsource2  
(4) 와 동일 #imediate 대신 reg사용   
  
#### (6) [Rsource1], +Rsource2, shift_operation #immediate  
			또는 [Rsource1], -Rsource2, shift_operation #immediate   
(4) 와 동일 #imediate 대신 reg에 shift 연산 사용  
		  
  
### b) Pre-indexing modes  
`!` 사용  
  
#### (7) [Rsource1, #+immediate]! 또는 [Rsource1, #-immediate]!  
  
```asm  
ldr r2, [r1, #+12]!  /* r1 ← r1 + 12 then r2 ← *r1 */  
add r2, r2, r2       /* r2 ← r2 + r2 */  
str r2, [r1]         /* *r1 ← r2 *  
```  
> a[3] = a[3] + a[3]   
> (1) 과 유사해 보임. r1이 저장된다는점이 다름.  
> (4) 와 다른점은 미리 인덱싱을 한뒤 값을 저장한다는 점.  
> []`!` 를 붙이면 Rsrc1이 저장됨 Updated됨.  
  
  
#### (8) [Rsource1, +Rsource2]! 또는 [Rsource1, +Rsource2]!  
(7) 과 동일 #imediate 대신 reg사용   
  
#### (9) [Rsource1, +Rsource2, shift_operation #immediate]!  
			또는 [Rsource1, -Rsource2, shift_operation #immediate]!  
  
(7) 과 동일 #imediate 대신 reg에 shift 연산 사용  
  
  
  
## pre-indexing 과 post-indexing 차이 확인 (feat. 'sp' operation)  
  
```asm  
str lr, [sp, #-8]!  /* Pre-index: sp ←  sp - 8; *sp ←  lr */  
... // Code of the function  
ldr lr, [sp], #+8   /* Post-index; lr ←  *sp; sp ←  sp + 8 */  
bx lr  
```  
> `sp`를 8바이트 확장 시켜놓고 ->  `sp`에 `lr`값을 저장한다. (pre-index; 미리 인덱싱)  
> `lr`을 `*sp` 로부터 복구 시켜놓은 뒤 ->  `sp`를 다시 8byte 줄인다. (post-index)  
  
  
## 구조체 초기화 with 인덱싱 모드  
  
  
  
  
```asm  
b.f1 = b.f1 + 7;  
```  
> in C code  
  
```asm  
ldr r2, [r1, #+4]!  /* r1 ← r1 + 4 then r2 ← *r1 */  
add r2, r2, #7     /* r2 ← r2 + 7 */  
str r2, [r1]       /* *r1 ← r2 */  
```  
> in asm code  
> r1 은 +4byte만큼 증가되어 저장됨.  
> 따라서 r2를 r1에 str 할때 index를 다시 계산 할 필요없음.  
  
  
  
  
  
  
  
  
  
  
<div style="page-break-after: always;"></div>

# 9. Functions-1  
----
  
## AAPCS 요약 (ARM Procedure Call Standard)  
  
- SP register  
Stack is an area of memory owned only by the current function.  
  
- Parameters  
4개의 파라미터 까지만 r0 ~ r3에 저장되고 나머지는 stack에 push 됨  
  
- AAPCS 규칙  
	* 함수 진입시에는 cpsr 상태를 모른다. 예측/가정 하면안됨.  
	* 함수에서 r0 ~ r3 를 자유자재로 변경 가능  
	* r4 ~ r11 은 수정가능 하지만 함수가 종료되었을 때 복구 되어야 한다.   
- 함수 호출  
직접 호출: 함수 위치를 알 경우 `bl label` 사용   
간접호출: 함수의 주소를 register에 저장하고 `blx Rsource1` 사용  
  
- 함수 Exit  
`lr` 의 초기값을 간직하고 있다가 함수 종료전에 다시 불러들여야함. `lr`에 저장가능  
그리고 `bx Rsource1` 함  

- Returning data  
`r0` 이용  

- (참고) `fp`(r11) 이란?
"Stack Frame" This register is used to keep the stack frame (also called the stack base or base pointer).  
What is the stack frame?  
The stack frame is just a register that keeps the value that sp got at the beginning.  
PC에서 arm-linux-*-gcc로 크로스 컴파일하면 많이 보임. AAPCS에는 없고 GCC가 사용하는 규약.  
  
## Hello world  
  
```asm  
/* -- hello01.s */  
.data  
   
greeting:  
.asciz "Hello world"		/* .asciz 또는 .ascii: GNU에서 제공하는 지시자.  
				 * 문자열 용. string + \0  
				 */  
.balign 4  
return: .word 0  
   
.text  
   
.global main  
main:  
    ldr r1, address_of_return     /*   r1 ←  &address_of_return */  
    str lr, [r1]                  /*   *r1 ← lr */  
   
    ldr r0, address_of_greeting   /* r0 ← &address_of_greeting */  
                                  /* First parameter of puts */  
   
    bl puts                       /* Call to puts */  
                                  /* lr ← address of next instruction */  
   
    ldr r1, address_of_return     /* r1 ← &address_of_return */  
    ldr lr, [r1]                  /* lr ← *r1 */  
    bx lr                         /* return from main */  
address_of_greeting: .word greeting  
address_of_return: .word return  
   
/* External */  
.global puts  
```  
  
  
```  
    ldr r1, address_of_return     /*   r1 ←  &address_of_return */  
    str lr, [r1]                  /*   *r1 ← lr */  
```  
> main 함수 진입전의 `lr` 값을 저장(보존)해야함. main 함수가 종료될 때 되돌아갈 주소.  
> 함수 진입 직후에는 반드시 현 함수에서 사용할 registor를 다른곳에 보관해 놓아야 한다. 그리고 함수 종료 직전 다시 복원한다.  
> 이 main 함수에서 또다른 함수 call이 있기 때문에 lr이 변경될 것이다. 따라서 lr을 미리 preserve 해 놓는것이다.  
  
  
```asm  
    ldr r0, address_of_greeting   /* r0 ← &address_of_greeting */  
```  
> puts 의 파라미터를 정의함. r0 가 첫번째 파라미터로 사용됨.  
  
  
```asm  
    bl puts                       /* Call to puts */  
```  
> `bl` 명령으로 인해 `lr` 레지스터가 puts가 복귀할 주소로 overwrite 될 것이다. 그래서 미리 `[r1]`에 main함수의 복귀 주소를 저장한것.  
> `bl` 명령 사용시 복귀주소를 자동으로 `lr`에 저장 한다.  
  
  
```asm  
    ldr r1, address_of_return     /* r1 ← &address_of_return */  
    ldr lr, [r1]                  /* lr ← *r1 */  
```  
> main 함수의 복귀주소를 복구.  
  
  
## printf() scanf()  
  
  
```asm  
/* -- printf01.s */  
.data  
   
/* First message */  
.balign 4  
message1: .asciz "Hey, type a number: "  
   
/* Second message */  
.balign 4  
message2: .asciz "I read the number %d\n"  
   
/* Format pattern for scanf */  
.balign 4  
scan_pattern : .asciz "%d"  
   
/* Where scanf will store the number read */  
.balign 4  
number_read: .word 0  
   
.balign 4  
return: .word 0  
   
.text  
   
.global main  
main:  
    ldr r1, address_of_return        /* r1 ←  &address_of_return */  
    str lr, [r1]                     /* *r1 ←  lr */  
   
    ldr r0, address_of_message1      /* r0 ←  &message1 */  
    bl printf                        /* call to printf */  
   
    ldr r0, address_of_scan_pattern  /* r0 ←  &scan_pattern */  
    ldr r1, address_of_number_read   /* r1 ←  &number_read */  
    bl scanf                         /* call to scanf */  
   
    ldr r0, address_of_message2      /* r0 ←  &message2 */  
    ldr r1, address_of_number_read   /* r1 ←  &number_read */  
    ldr r1, [r1]                     /* r1 ←  *r1 */  
    bl printf                        /* call to printf */  
   
    ldr r0, address_of_number_read   /* r0 ←  &number_read */  
    ldr r0, [r0]                     /* r0 ←  *r0 */  
   
    ldr lr, address_of_return        /* lr ←  &address_of_return */  
    ldr lr, [lr]                     /* lr ←  *lr */  
    bx lr                            /* return from main using lr */  
address_of_message1 : .word message1  
address_of_message2 : .word message2  
address_of_scan_pattern : .word scan_pattern  
address_of_number_read : .word number_read  
address_of_return : .word return  
   
/* External */  
.global printf  
.global scanf  
```  
  
```asm  
    ldr r1, address_of_return        /* r1 ←  &address_of_return */  
    str lr, [r1]                     /* *r1 ←  lr */  
```  
> 복귀주소 lr 따로 저장  
> 함수 내에 다른 subrutine call이 있다면, 함수 시작시 반드시 필요.  
> 이것은 주소를 r1에 저장하는 것이 아님, r1에 "저장된" 주소인 [r1] 즉, address_of_return 에 저장하는 것 유의.  
  
  
  
```asm  
/* -- printf02.s */  
.data  
   
/* First message */  
.balign 4  
message1: .asciz "Hey, type a number: "  
   
/* Second message */  
.balign 4  
message2: .asciz "%d times 5 is %d\n"  
   
/* Format pattern for scanf */  
.balign 4  
scan_pattern : .asciz "%d"  
   
/* Where scanf will store the number read */  
.balign 4  
number_read: .word 0  
   
.balign 4  
return: .word 0  
   
.balign 4  
return2: .word 0  
   
.text  
   
/*  
mult_by_5 function  
*/  
mult_by_5:   
    ldr r1, address_of_return2       /* r1 ←  &address_of_return */  
    str lr, [r1]                     /* *r1 ←  lr */  
   
    add r0, r0, r0, LSL #2           /* r0 ←  r0 + 4*r0 */  
   
    ldr lr, address_of_return2       /* lr ←  &address_of_return */  
    ldr lr, [lr]                     /* lr ←  *lr */  
    bx lr                            /* return from main using lr */  
address_of_return2 : .word return2  
   
.global main  
main:  
    ldr r1, address_of_return        /* r1 ←  &address_of_return */  
    str lr, [r1]                     /* *r1 ←  lr */  
   
    ldr r0, address_of_message1      /* r0 ←  &message1 */  
    bl printf                        /* call to printf */  
   
    ldr r0, address_of_scan_pattern  /* r0 ←  &scan_pattern */  
    ldr r1, address_of_number_read   /* r1 ←  &number_read */  
    bl scanf                         /* call to scanf */  
   
    ldr r0, address_of_number_read   /* r0 ←  &number_read */  
    ldr r0, [r0]                     /* r0 ←  *r0 */  
    bl mult_by_5  
   
    mov r2, r0                       /* r2 ←  r0 */  
    ldr r1, address_of_number_read   /* r1 ←  &number_read */  
    ldr r1, [r1]                     /* r1 ←  *r1 */  
    ldr r0, address_of_message2      /* r0 ←  &message2 */  
    bl printf                        /* call to printf */  
   
    ldr lr, address_of_return        /* lr ←  &address_of_return */  
    ldr lr, [lr]                     /* lr ←  *lr */  
    bx lr                            /* return from main using lr */  
address_of_message1 : .word message1  
address_of_message2 : .word message2  
address_of_scan_pattern : .word scan_pattern  
address_of_number_read : .word number_read  
address_of_return : .word return  
   
/* External */  
.global printf  
.global scanf  
```  
```asm  
    bl mult_by_5  
  
    mov r2, r0                       /* r2 ←  r0 */  
    ldr r1, address_of_number_read   /* r1 ←  &number_read */  
    ldr r1, [r1]                     /* r1 ←  *r1 */  
    ldr r0, address_of_message2      /* r0 ←  &message2 */  
    bl printf                        /* call to printf */  
```  
> printf 가 3개의 파라미터를 갖는 경우임  
> mult_by_5 함수가 return한 값 r0를 r2에 저장함.  
  
  
`bx lr`    
`bl funcrions_label`    
> bx와 bl 차이 주의, bx는 보통 함수 종료할때 사용하는듯. bl는 함수 분기용 (직접호출)  
  
  
  
  
  
  
<div style="page-break-after: always;"></div>

# 10. Functions-2 The stack  
----
  
  
  
  
## Dynamic activation  
  
(또는 Activation Record, 또는 stack frame)  
함수가 호출된 시점 부터 리턴 될때까지 존재하는 런타임 Stack 메모리 영역.  
The span of a dynamic activation goes from the point where the function is called until it returns.   
  
```c  
int factorial(int n)  
{  
   if (n == 0)  
      return 1;  
   else  
      return n * factorial(n-1);  
}  
```  
> 재귀 호출이다. 각 재귀 함수가 호출될때 마다. `lr`은 동일한 주소이다. 따라서 각 함수, Dynamic activation 마다 `lr`을 따로 저장해야 한다.   
> `lr` 뿐만 아니라, r4 ~ r11`까지도 각 Dynamic activation 마다 보존 되어야 한다.  
  
## 스택  
  
스택이란? The stack is a region of memory owned solely by the current Dynamic activation.  
각 함수의 Dynamic activation 마다 유지 되어야할 값들이 저장되어 있다.  
현재 Stack Pointer 위치부터 이전 Stack Pointer 위치까지, 그 영역을 Local memory라고 부르기로 한다.   
(The local memory is then defined by the range of memory from the current `sp` value to the original value that `sp` had at the beginning of the function.)  
  
SP는 함수가 종료될때 그 함수가 호출되기 이전값으로 복구 되어야 한다.  
SP 는 항상 4byte로 aligned 되어있음  

__리눅스 ARM에서는 stack은 주소값이 낮아지는 방향으로 증가함__  
In Linux ARM, the stack grows downwards, towards zero.  
If it grows downwards, the value of the sp register must be subtracted as many bytes as the size of the local storage  
  

## 함수에서 `sp` 수정하는 방법  
  
- Naive 한 방법
```asm  
sub sp, sp, #8  	/* sp ←  sp - 8. This enlarges the stack by 8 bytes */  
str lr, [sp]    	/* *sp ←  lr */  
... // Code of the function  
ldr lr, [sp]    	/* lr ←  *sp */  
add sp, sp, #8  	/* sp ←  sp + 8. /* This reduces the stack by 8 bytes  
                                effectively restoring the stack   
                                pointer to its original value */  
bx lr  
```  
  
- Indexing mode 사용  
  
```asm  
str lr, [sp, #-8]!  /* Pre-index: sp ←  sp - 8; *sp ←  lr */  
... // Code of the function  
ldr lr, [sp], #+8   /* Post-index; lr ←  *sp; sp ←  sp + 8 */  
bx lr  
```  
> sp를 8바이트 확장 시켜놓고 ->  sp에 lr값을 저장한다. (pre-index; 미리 인덱싱)  
> lr을 *sp 로부터 복구 시켜놓은 뒤 ->  sp를 다시 8byte 줄인다. (post-index)  
  
  
## Factorial 함수에 SP 조작 implementation 하기  
  
```asm  
/* -- factorial01.s */  
.data  
   
message1: .asciz "Type a number: "  
format:   .asciz "%d"  
message2: .asciz "The factorial of %d is %d\n"  
   
.text  
   
factorial:  
    str lr, [sp,#-4]!  /* Push lr onto the top of the stack */  
    str r0, [sp,#-4]!  /* Push r0 onto the top of the stack */  
                       /* Note that after that, sp is 8 byte aligned */  
    cmp r0, #0         /* compare r0 and 0 */  
    bne is_nonzero     /* if r0 != 0 then branch */  
    mov r0, #1         /* r0 ←  1. This is the return */  
    b end  
is_nonzero:  
                       /* Prepare the call to factorial(n-1) */  
    sub r0, r0, #1     /* r0 ←  r0 - 1 */  
    bl factorial  
                       /* After the call r0 contains factorial(n-1) */  
                       /* Load r0 (that we kept in th stack) into r1 */  
    ldr r1, [sp]       /* r1 ←  *sp */  
    mul r0, r0, r1     /* r0 ←  r0 * r1 */  
   
end:  
    add sp, sp, #+4    /* Discard the r0 we kept in the stack */  
    ldr lr, [sp], #+4  /* Pop the top of the stack and put it in lr */  
    bx lr              /* Leave factorial */  
   
.global main  
main:  
    str lr, [sp,#-4]!            /* Push lr onto the top of the stack */  
    sub sp, sp, #4               /* Make room for one 4 byte integer in the stack */  
                                 /* In these 4 bytes we will keep the number */  
                                 /* entered by the user */  
                                 /* Note that after that the stack is 8-byte aligned */  
    ldr r0, address_of_message1  /* Set &message1 as the first parameter of printf */  
    bl printf                    /* Call printf */  
   
    ldr r0, address_of_format    /* Set &format as the first parameter of scanf */  
    mov r1, sp                   /* Set the top of the stack as the second parameter */  
                                 /* of scanf */  
    bl scanf                     /* Call scanf */  
   
    ldr r0, [sp]                 /* Load the integer read by scanf into r0 */  
                                 /* So we set it as the first parameter of factorial */  
    bl factorial                 /* Call factorial */  
   
    mov r2, r0                   /* Get the result of factorial and move it to r2 */  
                                 /* So we set it as the third parameter of printf */  
    ldr r1, [sp]                 /* Load the integer read by scanf into r1 */  
                                 /* So we set it as the second parameter of printf */  
    ldr r0, address_of_message2  /* Set &message2 as the first parameter of printf */  
    bl printf                    /* Call printf */  
   
   
    add sp, sp, #+4              /* Discard the integer read by scanf */  
    ldr lr, [sp], #+4            /* Pop the top of the stack and put it in lr */  
    bx lr                        /* Leave main */  
   
address_of_message1: .word message1  
address_of_message2: .word message2  
address_of_format: .word format  
```  
  
## LDM and STM  
  
LDR/STR이 메모리와 레지스터간에 데이터를 한번만 이동하는것이라면,  
LDM/STM은 여러개의 데이터를 한번에 이동시키는 것.  
주로 LDM은 IA와, STM은 DB와 쓰이는 것이 일반적.  
LDM과 LDMIA는 동일함, Access한 뒤, 주소 증가  
  
참고 http://trace32.com/wiki/index.php/LDM_and_STM  
  
```  
ldm addressing-mode Rbase{!}, register-set  
stm addressing-mode Rbase{!}, register-set  
```  
> Rbase is the base address used to load to or store from the register-set.  
> addressing-mode: IA 또는 DB  
	IA: **I**ncrement address **A**fter each access  
	DB: **D**ecrement address **B**efore each access  
  
LDM	주소값이 증가  
STM	주소값이 ?  
LDMIA	주소값이 "증가"  
STMDB	주소값이 "감소"  
  
```asm  
LDM	baseAddr! {rX, rY}  
```  
> bassAddr 에 rX, rY 를 순서대로 저장한다.   
> baseAddr이 담고있는 주소값은 ! 로 인해서 자동으로 +4bype 커지고 저장된다. update  
  
```asm  
LDMIA	baseAddr! {rX, rY}  
```  
  
```asm  
STM	baseAddr! {rX, rY}  
```  
  
```asm  
STMDB	baseAddr! {rX, rY}  
```  
> bassAddr 에있는 값을  rY, rX 로 순서대로 저장한다.  (순서 반대 주의)  
> DB이기때문에  baseAddr이 담고있는 주소값은 ! 로 인해서 자동으로 -4bype 줄어들어 저장된다. update  
  
  
## LDM/STM을 활용한 stack 연산   
  
```asm  
str lr, [sp,#-4]!  /* Push lr onto the top of the stack */  
str r4, [sp,#-4]!  /* Push r4 onto the top of the stack */  
```  
> push와  

```asm  
ldr r4, [sp], #+4  /* Pop the top of the stack and put it in r4 */  
ldr lr, [sp], #+4  /* Pop the top of the stack and put it in lr */  
```  
> pop을  
  
아래와 같이 변경 가능  
  
```asm  
stmdb sp!, {r4, lr}    /* Push r4 and lr onto the stack */  
```  
> lr과 r4순서로 base address즉 sp 의 주소에 저장.  
> 먼저 sp에 저장을 하고 주소값 감소시킴. (db)  
  
```asm  
ldmia sp!, {r4, lr}    /* Pop lr and r4 from the stack */  
```  
> sp에 있는 값을 먼저 r4에 저장(pop)하고, lr에 pop함.  
> 주소값을 먼저 증가시키고(ia) sp 에서 pop함  
  
  
stm ~= push  
ldm ~= pop  
라고 이해하면 됨.  
  
- GNU assembler는 위의 명령어를 더 쉽게 사용하기 위해 아래 명령을 제공함  
  
```asm  
push {r4, lr}  
pop {r4, lr}  
```  
  
  
  
  
<div style="page-break-after: always;"></div>

# 11. Predication  
----

`NOTE` Prediction 아님 Predic__a__tion임 주의 (Not to be confused with branch prediction.)

가용 자원이 부족한 임베디드 시스템에서는 코드 사이즈를 줄일 필요가 있는데 이때 Predication을 사용하면 좋다.  
ARM 명령어에는 Predication 기능이 있다. 아얘 분기를 없애는 방법이다.  Predication을 이용하면 분기를 줄여 성능향상에 도움이 된다.  
ARM에서 Predication을 사용하는 방법은 매우 간단하다. 명령어에 eq, neq, le,lt,ge,gt등의 suffix를 붙여서 사용하면 된다. 이를 조건부실행 명령이라고도 부른다.  아래 예제를 보면 바로 이해가 된다.  
  
### (참고) 다른 주제이지만 잠시 Branch Prediction 에대해 살펴보자  

문제: 파이프 라인에서 분기 조건에따라, 느려지는 문제가 발생한다.  
  
(어떤 문제가 발생하는지는 다음 내용파악.)  
https://everylwn.blogspot.kr/2016/12/branch-prediction-logic-in-arm.html  
branch 명령어를 fetch 한후, processor는 다음 명령어를 가져와야함.  
그런데 문제는 분기문 이기 때문에 if(참)일때 명령 if(거짓)일때 명령,  2개의 브랜치가 다음 명령어가 될 수 있다는 점이다. 이런 branching 명령어가 파이프라인의 끝에 갈때까지 어떤 명령어가 next명령어인지 확신 할 수 없다.  
Branching 명령어가 완전히 수행될 때까지 파이프라이닝을 지연시키는 대신에 예측을 한다. 그런다음 그 예상되는 명령어를 fetch 할수 있고, 예측이 잘못되면 폐기한다.   
likely()와 unlikely() 매크로가 바로 branch prediction을 위해 사용하는 리눅스 커널 매크로인 것이다.   
  
### (참고) ARM의 파이프라인  
  
ARM7 core 에서는 파이프라인이 3가지다. Fetch/Decode/Execute  
`mov r0 r1` `add r0 r1 #3` 같이 레지스터간 데이터 이동명령은 ARM core내에서 동작하기 때문에 실행 속도가 매우 빠르다. 반면 `ldr r3, 0x4CC` `str r3, 0x455` 같이 외부 BUS를 통해 접근해야하는 명령은 상대적으로 속도가 느리기 때문에 파이프라인 단계도 추가된다.  
ARM7 core에는 ldr/str을 위해 메모리에서 데이터를 읽는Memory stage와  
레지스터에 쓰기를 하는 Write stage라는 단계가 존재한다.   
하지만 ARM7 core자체는 3단계의 파이프 라인만 있기 때문에 이 M과W가 수행되는동안에 다른 파이프라인은 stall된다. 그림참고  
  
그래서 ARM9 core는 애초에 파이프라인이 5단계다. F/D/E/M/W   
중간에 ldr/str명령이 있어도 stall되지 않음  
  
ARM11 core부터는 8단계이다.  
Fetch stage 2단, Decode Stage, Issue Stage, Execute Stage 4단  
ARM11에서는 pipeline 효율을 높이기 위한 Program Flow Prediction 기능이 있다. 명령의 흐름을 예측하여 pipeline의 stall 현상을 줄여 효율성을 높여 주는 역할  
  
http://recipes.egloos.com/5643663  
  
## Predication 예제  

다시 Predication으로 돌아와서 예제를 통해 바로 이해하자.  
  
- 이전 콜라츠 추측 collatz.s 예제
  
	소스 코드의 일부분 (전체코드는 #5장 참고)  
	  
	```asm  
	    and r2, r1, #1             /* r2 ←  r1 & 1 */  
	    cmp r2, #0                 /* compare r2 and 0 */  
	    bne collatz_odd            /* if r2 != 0 (this is r1 % 2 != 0) branch to collatz_odd */  
	  collatz_even:  
	    mov r1, r1, ASR #1         /* r1 ←  r1 >> 1. This is r1 ←  r1/2 */  
	    b collatz_end_loop         /* branch to collatz_end_loop */  
	  collatz_odd:  
	    add r1, r1, r1, LSL #1     /* r1 ←  r1 + (r1 << 1). This is r1 ←  3*r1 */  
	    add r1, r1, #1             /* r1 ←  r1 + 1. */  
	  collatz_end_loop:  
	```  
	> Original code: has many branching instructions.  
	> if/else label이동하는 분기가 많다.  
	  
	```asm  
	    cmp r2, #0                 /* compare r2 and 0 */  
	    moveq r1, r1, ASR #1       /* if r2 == 0, r1 ←  r1 >> 1. This is r1 ←  r1/2 */  
	    addne r1, r1, r1, LSL #1   /* if r2 != 0, r1 ←  r1 + (r1 << 1). This is r1 ←  3*r1 */  
	    addne r1, r1, #1           /* if r2 != 0, r1 ←  r1 + 1. */  
	```  
	> Predication 이 포함된 코드  
	> moveq, addne 로 분기를 없앴다.  
  
  
- 위 두 차이를 perf로 비교했을때, 아래와 같이 유의미한 성능 향상이 있다.  
  
	```  
	       3359,953200 cpu-clock                  ( +-  0,01% )  
	       3,365263737 seconds time elapsed      
	```  
	> 분기 사용 코드  
	  
	```  
	       2318,217200 cpu-clock                  ( +-  0,01% )  
	       2,322732232 seconds time elapsed           
	```  
	> Predication 사용 코드  
  
  
  
  
<div style="page-break-after: always;"></div>

# 12. Loops and the status register  
----
  
## 접미사 `-s` -> CPSR update  
  
지금까지 나온 예제에선 `cmp` 만 `CPSR`을 update하는 명령어였다. 기본 명령어는 `CPSR`을 update하지 않는다. 반면 접미사 `-s`를 붙이면 `CPSR`을 update한다.  
  
  
`for (int i = 100 ; i >= 0; i--)` 에 대한 다음 asm명령어를 보자.  
  
- cmp 사용 cpsr update  
```asm  
mov r1, #100  
loop:  
  /* do something */  
  sub r1, r1, #1      /* r1 ←  r1 - 1 */  
  cmp r1, #0          /* update cpsr with r1 - 0 */  
  bge loop            /* branch if r1 >= 100 */  
```  
  
- -s suffix 를 이용한 cpsr update  
```asm  
mov r1, #100  
loop:  
  /* do something */  
  subs r1, r1, #1      /* r1 ←  r1 - 1  and update cpsr with the final r1 */  
  bpl loop             /* branch if the previous sub computed a positive number (PL: N flag in cpsr is 0) */  
```  
> `subs` 뺄셈 연산한 뒤 `cpsr` update
  
  
## Operating 64-bit numbers in 32-bit ARM  
  
  
- 더하기  
  
첫번째 값 {r2,r3}에 저장되어 있음. the second in {r4,r5} 결과값은  {r0,r1} 에 저장한다고 할때:  
  
```asm  
add r1, r3, r5      /* First we add the higher part */  
                    /* r1 ←  r3 + r5 */  
adds r0, r2, r4     /* Now we add the lower part and we update cpsr */  
                    /* r0 ←  r2 + r4 */  
addcs r1, r1, #1    /* If adding the lower part caused carry, add 1 to the higher part */  
                    /* if C = 1 then r1 ←  r1 + 1 */  
                    /* Note that here the suffix -s is not applied, -cs means carry set */  
```  
> `adds`: -s 접미사로 `cpsr` carry 가 업데이트됨  
> `addcs`: `cpsr` C가 업데이트 되어있으면 더함.  
-cs (carry set) 으로 carry가 있으면 1을 더함  
  
  
```asm  
adds r0, r2, r4     /* First add the lower part and update cpsr */  
                    /* r0 ←  r2 + r4 */  
adc r1, r3, r5      /* Now add the higher part plus the carry from the lower one */  
                    /* r1 ←  r3 + r5 + C */  
```  
> `adc`: carry flag가 있으면 자동으로 1까지 더하는 명령어  
  
  
  
- 빼기  
  
  
```asm  
subs r0, r2, r4     /* First subtract the lower part and update cpsr */  
                    /* r0 ←  r2 - r4 */  
sbc r1, r3, r5      /* Now subtract the higher part plus the NOT of the carry from the lower one */  
                    /* r1 ←  r3 - r5 - ~C */  
```  
  
  
- 곱하기  
  
잘 이해못함, 공부를 다시 할것  
  
```asm  
umull r0, r6, r2, r4  
```  
>   
  
  
```asm  
/* Note: This is not the most efficient way to doa 64-bit multiplication.  
   This is for illustration purposes */  
mult64:  
   /* The argument will be passed in r0, r1 and r2, r3 and returned in r0, r1 */  
   /* Keep the registers that we are going to write */  
   push {r4, r5, r6, r7, r8, lr}  
   /* For covenience, move {r0,r1} into {r4,r5} */  
   mov r4, r0   /* r0 ←  r4 */  
   mov r5, r1   /* r5 ←  r1 */  
   
   umull r0, r6, r2, r4    /* {r0,r6} ←  r2 * r4 */  
   umull r7, r8, r3, r4    /* {r7,r8} ←  r3 * r4 */  
   umull r4, r5, r2, r5    /* {r4,r5} ←  r2 * r5 */  
   adds r2, r7, r4         /* r2 ←  r7 + r4 and update cpsr */  
   adc r1, r2, r6          /* r1 ←  r2 + r6 + C */  
   
   /* Restore registers */  
   pop {r4, r5, r6, r7, r8, lr}  
   bx lr                   /* Leave mult64 */  
   
```  
  
  
<div style="page-break-after: always;"></div>

# 13. Floating point numbers  
----
  
from the IEEE754   
```
sign(부호) | exponent(지수) | mentissa(가수)
```
(가수)x2^(지수)  
  
 single-precision floating point: 단정밀도, 32bit  
 double-precision floating point: 배정밀도, 64bit  
연산속도는 배정도가 1.5~2배 느리다.  
  
## VFPv2 Registers: `s0` ~ `s31`, `d0` ~ `d15`

ARM은 부동소수점 연산을 위한 Special register를 가지고 있다. 이름은   
single precision : s0 ~ s31  
double precision : d0 ~ d15  
  
3종류의 컨트롤 register를 가지고 있다  
`fpscr` : `cpsr`과 유사 N,Z,C,V flags를 가지고 있음  
  
![vfp](http://thinkingeek.com/wp-content/uploads/2013/04/vfp-registers.png)  
	그림: 순서대로 bank 0, 1, 2, 3 인듯  
  
## Arithmetic operations  
  
두 종류의 산술연산 형식이 있다.  
	* `vname Rdest, Rsource1, Rsource2`  
		name은 명령어로 대체  
	* `fname Rdest, Rsource1`  
  
그리고 세 종류의 연산 모드가 있다.  
	* Scalar : bank0 사용  
	* Vectorial : bank 1/2/3 사용  
	* Scalar expanded : 혼합해서 사용?  
  
  
  
```asm  
// For this example assume that len = 4, stride = 2  
vadd.f32 s1, s2, s3	/* s1 ←  s2 + s3. Scalar operation because Rdest = s1 in the bank 0 */  
vadd.f32 s1, s8, s15	/* s1 ←  s8 + s15. ditto */  
vadd.f32 s8, s16, s24	/* s8  ←  s16 + s24  
                      	   s10 ←  s18 + s26  
                      	   s12 ←  s20 + s28  
                      	   s14 ←  s22 + s30  
                      	   or more compactly {s8,s10,s12,s14} ←  {s16,s18,s20,s22} + {s24,s26,s28,s30}  
                      	   Vectorial, since Rdest and Rsource2 are not in bank 0 */  
  
vadd.f32 s10, s16, s24	/* {s10,s12,s14,s8} ←  {s16,s18,s20,s22} + {s24,s26,s28,s30}.  
                           Vectorial, but note the wraparound inside the bank after s14.  */  
vadd.f32 s8, s16, s3	/* {s8,s10,s12,s14} ←  {s16,s18,s20,s22} + {s3,s3,s3,s3}  
                     	   Scalar expanded since Rsource2 is in the bank 0 */  
```  
> `vadd.f32` : single-precision 의 add 연산  
  
> `vaddne.f32` 이런식으로 predication도 가능 (not eq 일때 add 하라)  
  
  
## Load and store  
  
`vldr` `vstr` 을 사용해서 부동소수점 값을 load/store할 수 있다.  
  
```asm  
vldr s1, [r3]         /* s1 ←  *r3 */  
vldr s2, [r3, #4]     /* s2 ←  *(r3 + 4) */  
vldr s3, [r3, #8]     /* s3 ←  *(r3 + 8) */  
vldr s4, [r3, #12]    /* s3 ←  *(r3 + 12) */  
   
vstr s10, [r4]        /* *r4 ←  s10 */  
vstr s11, [r4, #4]    /* *(r4 + 4) ←  s11 */  
vstr s12, [r4, #8]    /* *(r4 + 8) ←  s12 */  
vstr s13, [r4, #12]   /* *(r4 + 12) ←  s13 */  
```  
> 이런 연산이 가능  
  
```  
vldm indexing-mode precision Rbase{!}, floating-point-register-set  
vstm indexing-mode precision Rbase{!}, floating-point-register-set  
```  
> 다수레지스터 연산인 ldm/stm도 가능  
  
```asm  
vldmias r4, {s3-s8} /* s3 ←  *r4  
                       s4 ←  *(r4 + 4)  
                       s5 ←  *(r4 + 8)  
                       s6 ←  *(r4 + 12)  
                       s7 ←  *(r4 + 16)  
                       s8 ←  *(r4 + 20)  
                     */  
vldmias r4!, {s3-s8} /* Like the previous instruction  
                        but at the end r4 ←  r4 + 24   
                      */  
vstmdbs r5!, {s12-s13} /*  *(r5 - 4 * 2) ←  s12  
                           *(r5 - 4 * 1) ←  s13  
                           r5 ←  r5 - 4*2  
                       */  
  
```  
> Addressing mode, as/bs  
  
  
```asm  
vpush {s0-s5} /* Equivalent to vstmdb sp!, {s0-s5} */  
vpop {s0-s5}  /* Equivalent to vldmia sp!, {s0-s5} */  
  
```  
> Stack Operation  
  
  
  
## Movements between registers  
  
register간 데이터 이동시: mov 명령어.  
  
```asm  
vmov s2, s3  /* s2 ← s3 */  
```  
  
```  
vmov s2, r3  /* s2 ← r3 */  
```  
> `r#` 과 `s#` 를 혼용해서 사용한 경우 비트만 복사될 뿐, data가 복사되는것이 아님을 주의  
  
## Conversions  
  
  
```asm  
vcvt.f64.f32 d0, s0  /* Converts s0 single-precision value   
                        to a double-precision value and stores it in d0 */  
   
vcvt.f32.f64 s0, d0  /* Converts d0 double-precision value   
                        to a single-precision value  and stores it in s0 */  
   
vmov s0, r0          /* Bit copy from integer register r0 to s0 */  
vcvt.f32.s32 s0, s0  /* Converts s0 signed integer value   
                        to a single-precision value and stores it in s0 */  
   
vmov s0, r0          /* Bit copy from integer register r0 to s0 */  
vcvt.f32.u32 s0, s0  /* Converts s0 unsigned integer value   
                        to a single-precision value and stores in s0 */  
   
vmov s0, r0          /* Bit copy from integer register r0 to s0 */  
vcvt.f64.s32 d0, s0  /* Converts r0 signed integer value   
                        to a double-precision value and stores in d0 */  
   
vmov s0, r0          /* Bit copy from integer register r0 to s0 */  
vcvt.f64.u32 d0, s0  /* Converts s0 unsigned integer value   
                        to a double-precision value and stores in d0 */  
```  
  
  
## fpscr 변경  
  
```asm  
/* Set the len field of fpscr to be 8 (bits: 111) */  
mov r5, #7                            /* r5 ←  7. 7 is 111 in binary */  
mov r5, r5, LSL #16                   /* r5 ←  r5 << 16 */  
vmrs r4, fpscr                        /* r4 ←  fpscr */  
orr r4, r4, r5                        /* r4 ←  r4 | r5. Bitwise OR */  
vmsr fpscr, r4                        /* fpscr ←  r4 */  
```  
> `vmrs` : fpscr을 general purpose register에 load하는것 대신에 mrs 명령어 사용가능  
> `vmsr` : register 에서 다시 fpscr에 전달  
  
## 참고: `mrs` `msr` 명령어 

`cpsr` 또는 `spsr` control하는 명령어 (PSR 명령)
CPSR과 SPSR 두개와 일반 Register 사이를에 값을 서로 복사할 수 있는 명령어들을 말한다. MRS, MSR 두개로 모든걸 처리한다.  

  
```asm  
mrs r0, cpsr  
```  
> `r0`에 `cpsr` 값을 읽어온다.  
  
```asm  
msr cpsr, r0  
```  
> `r0`의 내용을 `cpsr`에 복사 한다.  
  
  
## Function call convention and floating-point registers  
http://thinkingeek.com/2013/05/12/arm-assembler-raspberry-pi-chapter-13/ 읽기  
  
  
## Assembler  
  
Make sure you pass the flag `-mfpu=vfpv2` to as, otherwise it will not recognize the VFPv2 instructions.  
  
## 참고  
  
[quick reference card of VFP](http://infocenter.arm.com/help/topic/com.arm.doc.qrc0007e/QRC0007_VFP.pdf)  
  





<div style="page-break-after: always;"></div>

# 14. Matrix multiply
----

추후 필요시 study.




<div style="page-break-after: always;"></div>

# 15. Integer division  
----

ARMv6 는 integer 나누기 연산을 하는 명령어가 없다. 반면 floating point 연산을 위한 명령어는 있는데도 말이다. 

https://thinkingeek.com/2013/08/11/arm-assembler-raspberry-pi-chapter-15/
추후 필요시  study




<div style="page-break-after: always;"></div>

# 16. Switch control structure  
----

6장, 12장에서 if/else/for/while/.. 등의 몇 control structure를 살펴봤다. 이번장에서는 switch/case 가 어떻게 assembly 코드로 구현될 수 있는지 알아본다. 

먼저 C 언어의 switch 문.   

```c
switch (E) {
   case V1: S1;
   case V2: S2;
   default: Sdefault;
}
```
> case: or default: 가 없으면 switch {} 내에 어떠한 코드가 있어도 실행되지 않음에 유의.  

```c
switch (x)
{
  case 5: code_for_case5; break;
  case 10: code_for_case10; break;
  default: code_for_default; break; 
  // break would not be required here as this is the last case
}
```
> break;는 Unconditional Branch 와 같다.


```asm
  /* Here we evaluate x and keep it in r0 */
  case_5:             /* case 5 */
    cmp r0, #5        /* Compute r0 - 5 and update cpsr */
    bne case_10       /* if r0 != 5 branch to case_10 */
    code_for_case5
    b after_switch    /* break */
 
  case_10:            /* case 10 */
    cmp r0, #10       /* Compute r0 - 10 and update cpsr */
    bne case_default  /* If r0 != 10 branch to case_default */
    code_for_case10
    b after_switch    /* break */
 
  case_default:
    code_for_default 
    /* Note that if default is not the last case
       we need a branch to after_switch here */
 
  after_switch:
```
> `b` after_switch : Unconditional Branching  

C언어에서는 case 번호를 한번에 찾는것 같지만 Assembly단에서는 아니다. case 번호를 비교해서 찾는데 탐색 비용이 들어간다. 이런 비용을 줄이는 방법론으로 __Jump Tables__와 __Binary search__ 가 있다.

## Jump tables

jump table 은 case의 주소를 배열처럼 모아놓은 영역이다.  indexing 해서 쉽게 jump 할수 있도록 한다.


```asm
/* jumptable.s */
 
main:
... 중략

  sub r0, r0, #1              /* r0 ←  r0 - 1. Required to index the table */
  ldr r1, addr_of_jump_table  /* r1 ←  &jump_table */
  ldr r1, [r1, +r0, LSL #2]   /* r1 ←  *(r1 + r0*4).
                                 This is r1 ←  jump_table[r0] */
 
  mov pc, r1                  /* pc ←  r1
                                 This will cause a branch to the
                                 computed address */
 
  case_1:
   mov r0, #1                 /* r0 ←  1 */ 
   b after_switch             /* break */
 
  case_2: ...중략
 
  case_3:
   mov r0, #3                 /* r0 ←  3 */
   b after_switch             /* break */
 
  case_default:
   mov r0, #42                /* r0 ←  42 */
   b after_switch             /* break (unnecessary) */
 
  after_switch:
 
  bx lr                       /* Return from main */
 
.align 4
jump_table:
   .word case_1
   .word case_2
   .word case_3
 
.align 4
addr_of_jump_table: .word jump_table
```
> Jump table 에 각 case주소가 모여있어서 따로 탐색,비교없이 배열 index로 바로 jump가 가능하다.  
> 탐색 시간에 대한 비용은 constant하다.  
  
  
그러나 두가지 단점이 있다. 그래서 좋은 방법이 아님.  
  
1. case 범위가 많아지면 code size가 커진다. (Trade time by space 임.)  
	case 당 4byte 씩 증가  
2. jump table에 구멍이 존재할수 밖에 없음  
	case 가 1, 2, 100 이라고 했을 때, 3 ~ 99 까지 각 4byte씩 빈 공간이 있어야함 (그래야 배열처럼 indexing이 가능하기 때문)  
	1,2,100 3개의 case나 1부터 100까지 100개의 case나 jump table 사이즈는 동일해야함.  
  

## Compute the case address

jump table 방식보다 약간 복잡하기 때문에 덜 일반적이다.  

만약 모든 case가 순서대로 정렬되어있고 동일한 길이의 instruction으로 구성 되어있다면, 굳이 jump table을 사용할 필요가 없이 주소를 계산할 수 있다. (그러나 risky 한 방법)  

만약 case 마다 명령어들의 크기가 다르다면 `nop` 명령어를 사용해 채워 맞출 수 있다.  

```asm
main:
  cmp r0, #1                  /* r0 - 1 and update cpsr */
  blt case_default            /* branch to case_default if r0 < 1 */
  cmp r0, #3                  /* r0 - 3 and update cpsr */
  bgt case_default            /* branch to case_default if r0 > 3 */
 
  sub r0, r0, #1              /* r0 ←  r0 - 1. Required to index the table */
  ldr r1, addr_of_case_1      /* r1 ←  &case_1 */
  add r1, r1, r0, LSL #3      /* r1 ←  r1 + r0 * 8
                                 Each instruction is 4 bytes
                                 Each case takes 2 instructions
                                 Thus, each case is 8 bytes (4 * 2)
                                 */
 
  mov pc, r1                  /* pc ←  r1
                                 This will cause a branch to the
                                 computed address */
 
  case_1:
  ... 중략
```
> 모든 case에 2개의 instruction 만 있다고하면, jump table 없이 바로 case_N: 주소값 계산   


## Binary search

A binary search will discard half of the case set each time.   
아래 예제는 jump table 예제와 동일한 코드에 case만 1 ~ 10.  

```asm
/* binsearch.s */
.data
 
.text
 
.globl main
 
main:
 
  cmp r0, #1              /* r0 - 1 and update cpsr */
  blt case_default        /* if r0 < 1 then branch to case_default */
  cmp r0, #10             /* r0 - 10 and update cpsr */
  bgt case_default        /* if r0 > 10 then branch to case default */
 
  case_1_to_10:
    cmp r0, #5            /* r0 - 5 and update cpsr */
    beq case_5            /* if r0 == 5 branch to case_5 */
    blt case_1_to_4       /* if r0 < 5 branch to case_1_to_4 */
    bgt case_6_to_10      /* if r0 > 5 branch to case_6_to_4 */
 
  case_1_to_4:
    cmp r0, #2            /* r0 - 2 and update cpsr */
    beq case_2            /* if r0 == 2 branch to case_2 */
    blt case_1            /* if r0 < 2 branch to case_1 
                             (case_1_to_1 does not make sense) */
    bgt case_3_to_4       /* if r0 > 2 branch to case_3_to_4 */
 
  case_3_to_4:
    cmp r0, #3            /* r0 - 3 and update cpsr */
    beq case_3            /* if r0 == 3 branch to case_3 */
    b case_4              /* otherwise it must be r0 == 4,
                             branch to case_4 */
 
  case_6_to_10:
    cmp r0, #8            /* r0 - 8 and update cpsr */
    beq case_8            /* if r0 == 8 branch to case_8 */
    blt case_6_to_7       /* if r0 < 8 then branch to case_6_to_7 */
    bgt case_9_to_10      /* if r0 > 8 then branch to case_9_to_10 */
 
  case_6_to_7:
    cmp r0, #6            /* r0 - 6 and update cpsr */
    beq case_6            /* if r0 == 6 branch to case_6 */
    b case_7              /* otherwise it must be r0 == 7,
                             branch to case 7 */
 
  case_9_to_10:
    cmp r0, #9            /* r0 - 9 and update cpsr */
    beq case_9            /* if r0 == 9 branch to case_9 */
    b case_10             /* otherwise it must be r0 == 10,
                             branch to case 10 */
 
  case_1:
     mov r0, #1
     b after_switch
  case_2:
     mov r0, #2
     b after_switch

  ...  /* Cases from 3 to 9 omitted */

  case_10:
     mov r0, #10
     b after_switch
 
  case_default:
   mov r0, #42                /* r0 ←  42 */
   b after_switch             /* break (unnecessary) */
 
  after_switch:
 
  bx lr                       /* Return from main */
```

이 방법은 다음의 장점이 있다.   
- 비교 횟수가 3회 밖에 되지 않는다. O(logN)   
- case 에 hole 이 있어도 됨. case들이 continuous 하지 않고 scattered 해도 됨.  
- case 순서가 순서대로 정렬 되어있을 필요 없음.  
  
그러나 단점:  
- Code size 측면에서는 jump table 방법보다 우위에 있다고 볼수 없다.  


## Hybrid approach

따라서 이런 문제들을 해결한 방법이 있다. case 값 테이블과 case address 테이블 을 두개 사용하는것이고 그 두 테이블은 오름차순으로 정렬되어있다.  


```asm
/* prepare the binary search. */
     r0: switch() 로 입력된 case 번호, 찾을 값
     r1: 이진탐색시 lower 인덱스
     r2: 이진탐색시 upper 인덱스
     r3: case_value_table 의 base address
     r4: r1 r2의 중앙값 인덱스
     r5: case값[r4] 또는 case주소[r4], destination 결과값.
```
> 각 reg 가 사용되는 목적  

```asm
main:
....중략

/* -- 이진 탐색 시작 -- */ 
  mov r1, #0 /* lower인덱스: case value는 총 10개이기 때문에 인덱스는 0 ~ 9*/
  mov r2, #9 /* upper 인덱스 */
  ldr r3, addr_case_value_table /* r3 ←  &case_value_table */
 
  b check_binary_search
  binary_search:
    add r4, r1, r2              /* r4 ←  r1 + r2 */
    mov r4, r4, ASR #1          /* r4 ←  r4 / 2 */
                                /* r4는 r1과 r2의 중앙 인덱스 */
    ldr r5, [r3, +r4, LSL #2]   /* r5 ←  *(r3 + r4 * 4). 
                                   This is r5 ←  case_value_table[r4] */
                                /* 중앙 인덱스의 case value값은? r5에 저장 */
    cmp r0, r5                  /* r0 - r5 and update cpsr */
                                /* input 케이스 값 r0 과 찾을 값 r5 비교 */
    sublt r2, r4, #1            /* if r0 < r5 then r2 ←  r4 - 1 : upper값을 중앙 인덱스로 */
                                /* r0 가 더 작으면 범위 줄여서 다시 이진 탐색: Recursive Call */
    addgt r1, r4, #1            /* if r0 > r5 then r1 ←  r4 + 1 */
    bne check_binary_search     /* if r0 != r5 branch to binary_search */
 
    /* if we reach here it means that r0 == r5 */
    /* 이진 탐색으로 결국 값 찾았을때 */
    ldr r5, addr_case_addresses_table   /* r5 ←  &addr_case_value_table */
    ldr r5, [r5, +r4, LSL #2]           /* r5 ←  *(r5 + r4*4) 
                                           This is r5 ←  case_addresses_table[r4] */
                                        /* 동일한 인덱스 r4로 case_N 주소값 저장 -> r5 */
    mov pc, r5                          /* 해당 주소로 jump */
 
  check_binary_search:
    cmp r1, r2              /* r1 - r2 and update cpsr */
    ble binary_search       /* if r1 <= r2 branch to binary_search */

/* -- 여기 까지 이진 탐색 -- */ 
 
  /* if we reach here it means the case value
     was not found. branch to default case */
  b case_default
 
    case_1:
     mov r0, #1
     b after_switch
  case_2:
     mov r0, #2
     b after_switch
  case_3:
     mov r0, #3
     b after_switch
  case_24:
     mov r0, #24
     b after_switch
  case_25:
     mov r0, #95
     b after_switch
  case_26:
     mov r0, #96
     b after_switch
  case_97:
     mov r0, #97
     b after_switch
  case_98:
     mov r0, #98
     b after_switch
  case_99:
     mov r0, #99
     b after_switch
  case_300:
     mov r0, #300    /* The error code will be 44 */
     b after_switch
 
  case_default:
   mov r0, #42       /* r0 ←  42 */
   b after_switch    /* break (unnecessary) */
 
  after_switch:
 
  pop {r4,r5,r6,lr}
  bx lr              /* Return from main */
 
case_value_table: .word 1, 2, 3, 24, 25, 26, 97, 98, 99, 300
addr_case_value_table: .word case_value_table
 
case_addresses_table:
    .word case_1
    .word case_2
    .word case_3
    .word case_24
    .word case_25
    .word case_26
    .word case_97
    .word case_98
    .word case_99
    .word case_300
addr_case_addresses_table: .word case_addresses_table
```
> 주석 확인. case_value_table: 10 개의 값  

결론:  
- 이진탐색으로 비교 횟수 줄였음 O(logN)  
- 구멍 없는 jump table 로 code size 줄임   
  case 값과 주소가 1:1 매칭되는 두개의 테이블 필요  





<div style="page-break-after: always;"></div>

# 17. Passing data to functions  
----

함수에 데이터 전달하기  
다음 두가지 항목에 대해 더 알아본다  
	* 많은 양의 디이터 함수에 인자로 전달하기  
	* 하나 이상의 값 return 하기  
 
## So what is a pointer?


```asm
.data
 
.align 4
number_1  : .word 3
number_2  : .word 4
 
.text
.globl main
 
main:
    ldr r1, address_of_number_2  /* r1 ←  &number_2 */
    str r1, pointer_to_number    /* pointer_to_number ←  r1, this is pointer_to_number ←  &number_2 */
 
    bx lr
 
pointer_to_number: .word number_1
address_of_number_2: .word number_2
```
> address_of_number_2변수에 담긴 주소를  pointer_to_number 변수에 저장  
> -> Segmentation fault 발생함.  
> .text 섹션에 있는 변수는 값을 변경하는것이 불가능. 
> 포인터 값을 변경하기 위해서는 .data 섹션에 있어야 함.   

```asm
.data
 
.align 4
number_1  : .word 3
number_2  : .word 4
pointer_to_number: .word 0
 
.text
.globl main
 
 
main:
    ldr r0, addr_of_pointer_to_number
                             /* r0 ←  &pointer_to_number 주소 */
 
    ldr r1, addr_of_number_2 /* r1 ←  &number_2 주소*/
 
    str r1, [r0]             /* *r0 ←  r1.
                                This is actually pointer_to_number ←  &number_2 */
			     /* r0(주소)가 가 가리키는 값에 r1( number_2의 주소, 포인터) 저장 */
			     r0: addr_of_number2: 저장

 
    ldr r1, [r0]             /* r1 ←  *r0.
                                This is actually r1 ←  pointer_to_number
                                Since pointer_to_number has the value &number_2
                                then this is like r1 ←  &number_2 */
			     /* number_2 주소를 r1에 저장 */
			     /* r1에는 주소값이 있음 */
			     r1: addr_of_number2: 주소값 저장
 
 
    ldr r0, [r1]             /* r0 ←  *r1
                                Since r1 had as value &number_2
                                then this is like r0 ←  number_2 */
			     /* r1이 가리키는 값을 r0에 저장  */
			     /* 결국 r0 에는 addr_of_number_2 주소값 저장. */
 
    bx lr
 
addr_of_number_1: .word number_1
addr_of_number_2: .word number_2
addr_of_pointer_to_number: .word pointer_to_number
```
> label 이 가진 주소자체를 바꿀 수는 없음. 변수에 저장해서 포인터로 사용해야함.



## 함수에 배열 전달하기 "Call by Value"

sum_array_value 함수는 인자로 전달 된 배열을 modify하지않고 단지 read만 한다.  
r0는 배열의 크기이고 r1 ~ r3은 실제 배열의 값이 전달되어 온다. 배열 크기가 2라면 r2까지만 배열 값이 전달되고 크기가 4이상이라면 배열 4번째 값부터는 다른방식으로 다루어야 한다.   

- sum_array_value 함수

```asm
sum_array_value : 
    push {r4, r5, r6, lr}  /* 함수 시작전 이 함수에서 사용할 register변수의 기존값은 stack에 따로 보존, 함수 종료 직전 복원 */
 
    /* We have passed all the data by value */
 
    /* r4 는 최종 합계가 저장될것 */
    mov r4, #0      /* r4 ←  0 */
    /* In r0 we have the number of items of the array */
 
    cmp r0, #1            /* r0 - #1 and update cpsr */
    blt .Lend_of_sum_array  /* if r0 < 1 branch to end_of_sum_array */
    add r4, r4, r1        /* add the first item */
 
    cmp r0, #2            /* r0 - #2 and update cpsr */
    blt .Lend_of_sum_array  /* if r0 < 2 branch to end_of_sum_array */
    add r4, r4, r2        /* add the second item */
 
    cmp r0, #3            /* r0 - #3 and update cpsr */
                          /* 배열크기가 2였다면 r4에 r1+r2 까지한뒤 .Lend_* 로 이동해서 return */
    blt .Lend_of_sum_array  /* if r0 < 3 branch to end_of_sum_array */
    add r4, r4, r3        /* add the third item */
 
    /* 
     The stack at this point looks like this
       |                | (lower addresses)
       |                |
       | lr             |  <- sp points here
       | r6             |  <- this is sp + 4
       | r5             |  <- this is sp + 8
       | r4             |  <- this is sp + 12
       | big_array[3]   |  <- this is sp + 16 (we want r5 to point here)
       | big_array[4]   |
       |     ...        |
       | big_array[255] |
       |                | 
       |                | (higher addresses)
 
    keep in r5 the address where the stack-passed portion of the array starts */
    /* 배열의 a[0~2]는 r#에 나머지는 stack 에 push 해놓음 */

    add r5, sp, #16 /* r5 ←  sp + 16 */ /* r5는 다음 배열값 시작 지점 */
 
    /* in register r3 we will count how many items we have read
       from the stack. */
    mov r3, #0
 
    /* in the stack there will always be 3 less items because
       the first 3 were already passed in registers
       (recall that r0 had how many items were in the array) */
    sub r0, r0, #3
 
    b .Lcheck_loop_sum_array
    .Lloop_sum_array:
      ldr r6, [r5, r3, LSL #2]       /* r6 ←  *(r5 + r3 * 4) load 
                                        the array item r3 from the stack */
			             /* r5는 배열 a[3] 시작 지점 */
      add r4, r4, r6                 /* r4 ←  r4 + r6
                                        accumulate in r4 */
      add r3, r3, #1                 /* r3 ←  r3 + 1 
                                        move to the next item */
    .Lcheck_loop_sum_array:
      cmp r3, r0           /* r0 - r3 and update cpsr */ /* r0: 남은 총 배열 갯수, r3: 배열 카운트 0부터 */
      blt .Lloop_sum_array   /* if r3 < r0  branch to loop_sum_array */
 
  .Lend_of_sum_array:
    mov r0, r4  /* r0 ←  r4, to return the value of the sum */
    pop {r4, r5, r6, lr}
 
    bx lr
```
> r1, r2, r3 까지는 register에서 더하고 그 이후 값은 loop를 돌려서 더했음.  


- sum_array_value 함수 호출 하기

위 sum_array_value : 함수는 아래와 같은 방식으로 call 한다.   
일부는 r0~r3 로 전달하고 나머지는 stack에 push하는 과정  

```asm
.data
 
.align 4
 
big_array :
.word 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12, 13, 14, 15, 16, 17, 18, 19, 20, 21
.word 22, 23, 24, 25, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40, 41
.word 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, 52, 53, 54, 55, 56, 57, 58, 59, 60, 61
.word 62, 63, 64, 65, 66, 67, 68, 69, 70, 71, 72, 73, 74, 75, 76, 77, 78, 79, 80, 81
.word 82, 83, 84, 85, 86, 87, 88, 89, 90, 91, 92, 93, 94, 95, 96, 97, 98, 99, 100
.word 101, 102, 103, 104, 105, 106, 107, 108, 109, 110, 111, 112, 113, 114, 115, 116
.word 117, 118, 119, 120, 121, 122, 123, 124, 125, 126, 127, 128, 129, 130, 131, 132
.word 133, 134, 135, 136, 137, 138, 139, 140, 141, 142, 143, 144, 145, 146, 147, 148
.word 149, 150, 151, 152, 153, 154, 155, 156, 157, 158, 159, 160, 161, 162, 163, 164
.word 165, 166, 167, 168, 169, 170, 171, 172, 173, 174, 175, 176, 177, 178, 179, 180
.word 181, 182, 183, 184, 185, 186, 187, 188, 189, 190, 191, 192, 193, 194, 195, 196
.word 197, 198, 199, 200, 201, 202, 203, 204, 205, 206, 207, 208, 209, 210, 211, 212
.word 213, 214, 215, 216, 217, 218, 219, 220, 221, 222, 223, 224, 225, 226, 227, 228
.word 229, 230, 231, 232, 233, 234, 235, 236, 237, 238, 239, 240, 241, 242, 243, 244
.word 245, 246, 247, 248, 249, 250, 251, 252, 253, 254, 255
 
 
.text
.globl main
 
sum_array_value :
   /* code shown above */
 
main:
    push {r4, r5, r6, r7, r8, lr}
    /* we will not use r8 but we need to keep the function 8-byte aligned */
 
    ldr r4, address_of_big_array
 
    /* Prepare call */
 
    mov r0, #256   /* Load in the first parameter the number of items r0 ←  256 */
 
    ldr r1, [r4]     /* load in the second parameter the first item of the array */
    ldr r2, [r4, #4] /* load in the third parameter the second item of the array */
    ldr r3, [r4, #8] /* load in the fourth parameter the third item of the array */
 
    /* before pushing anything in the stack keep its position */
    mov r7, sp     /* stack에 배열 값들을 push 하기 전에 기존 stack pointer는 save시켜놓음 */
 
    /* We cannot use more registers, now we have to push them onto the stack
       (in reverse order) */
    mov r5, #255   /* r5 ←  255
                      This is the last item position
                      (note that the first would be in position 0) */
		   /* stack에는 맨마지막 값 255부터 push */
 
    /* 255부터 3까지 순서대로 stack에 push */

    b .Lcheck_pass_parameter_loop
    .Lpass_parameter_loop:
 
      ldr r6, [r4, r5, LSL #2]  /* r6 ←  *(r4 + r5 * 4).
                                   loads the item in position r5 into r6. Note that
                                   we have to multiply by 4 because this is the size
                                   of each item in the array */
      push {r6}                 /* push the loaded value to the stack */
      sub r5, r5, #1            /* we are done with the current item,
                                   go to the previous index of the array */
    .Lcheck_pass_parameter_loop:
      cmp r5, #2                /* compute r5 - #2 and update cpsr */
      bne .Lpass_parameter_loop   /* if r5 != #2 branch to pass_parameter_loop */
 
    /* We are done, we have passed all the values of the array,
       now call the function */
    bl sum_array_value
 
    /* restore the stack position */
    /* 함수 종료후 기존 stack pointer로 복구해줘야함.  */
    mov sp, r7 

    /* prepare the call to printf */
    ... 중략
 
    pop {r4, r5, r6, r7, r8, lr}
    bx lr
 
address_of_big_array : .word big_array
```
> `mov sp, r7`  
> 중요: **Keeping the stack synched** is essential when **calling functions**.  



## 함수에 배열 전달하기 "Call by Reference"

모든 배열값을 stack에 push하는 방법은 wasteful 하다. 배열값 자체를 함수로 전달하기보다 배열의 주소값을 함수의 인자로 전달하는 방법이 필요하다.  


- 함수인자로 배열 주소값 전달하는 code  

```asm
.data
 
.align 4
 
big_array :
  /* Same as above */
 
.align 4
 
message: .asciz "The sum of 0 to 255 is %d\n"
 
.text
.globl main
 
sum_array_ref :
    /* Parameters: 
           r0  Number of items
           r1  Address of the array
    */
    push {r4, r5, r6, lr}
 
    /* We have passed all the data by reference */
 
    /* r4 will hold the sum so far */
    mov r4, #0      /* r4 ←  0 */
    mov r5, #0      /* r5 ←  0 */
 
    b .Lcheck_loop_array_sum
    .Lloop_array_sum:
      ldr r6, [r1, r5, LSL #2]   /* r6 ←  *(r1 + r5 * 4) */
      add r4, r4, r6             /* r4 ←  r4 + r6 */
      add r5, r5, #1             /* r5 ←  r5 + 1 */
    .Lcheck_loop_array_sum:
      cmp r5, r0                 /* r5 - r0 and update cpsr */
      bne .Lloop_array_sum       /* if r5 != r0 go to .Lloop_array_sum */
 
    mov r0, r4  /* r0 ←  r4, to return the value of the sum */
    pop {r4, r5, r6, lr}
 
    bx lr
 
 
main:
    push {r4, lr}
    /* we will not use r4 but we need to keep the function 8-byte aligned */
 
    mov r0, #256
    ldr r1, address_of_big_array
 
    /* 함수 호출*/
    bl sum_array_ref
 
    /* prepare the call to printf */ ...(생략)
 
    pop {r4, lr}
    bx lr
 
address_of_big_array : .word big_array
address_of_message : .word message
```
> 상대적으로 대단히 심플함.  


## 포인터를 이용해서 data 변경하기 

- call by value
```asm
increment:
    add r0, r0, #1  /* r0 ←  r0 + 1 */
```
> r0 로 인자(값)를 전달받고 r0를 return.  
> 함수 종료후 r0 는 변동없음  


- call by reference

```asm
increment_ptr:
  ldr r1, [r0]      /* r1 ←  *r0 */
  add r1, r1, #1    /* r1 ←  r1 + 1 */
  str r1, [r0]      /* *r0 ←  r1 */
```
> r0 로 인자(포인터)를 전달받고 r0를 return.  
> 함수 종료후 r0+=1 되어있음.  


- big_array[] 각 값에 x2하는 코드 (by reference)

```asm
/* double_array.s */
 
.data
 
.align 4
big_array :
 /* Same as above */
 
.align 4
message: .asciz "Item at position %d has value %d\n"
 
.text
.globl main
 
double_array : 
    /* Parameters: 
           r0  Number of items
           r1  Address of the array
    */
    push {r4, r5, r6, lr} /* 사용할 register변수의 기존값은 stack에 따로 보존 */
 
    mov r4, #0      /* r4 ←  0 */
 
    b .Lcheck_loop_array_double
    .Lloop_array_double:
      ldr r5, [r1, r4, LSL #2]   /* r5 ←  *(r1 + r4 * 4) */
      mov r5, r5, LSL #1         /* r5 ←  r5 * 2 */
      str r5, [r1, r4, LSL #2]   /* *(r1 + r4 * 4) ←  r5 */
      add r4, r4, #1             /* r4 ←  r4 + 1 */
    .Lcheck_loop_array_double:
      cmp r4, r0                 /* r4 - r0 and update cpsr */
      bne .Lloop_array_double    /* if r4 != r0 go to .Lloop_array_double */
 
    pop {r4, r5, r6, lr}
 
    bx lr
 
print_each_item:
    push {r4, r5, r6, r7, r8, lr} /* r8 is unused */
 
    mov r4, #0      /* r4 ←  0 */
    mov r6, r0      /* r6 ←  r0. Keep r0 because we will overwrite it */
    mov r7, r1      /* r7 ←  r1. Keep r1 because we will overwrite it */
 
 
    b .Lcheck_loop_print_items
    .Lloop_print_items:
      ldr r5, [r7, r4, LSL #2]   /* r5 ←  *(r7 + r4 * 4) */
 
      /* Prepare the call to printf */
      ldr r0, address_of_message /* first parameter of the call to printf below */
      mov r1, r4      /* second parameter: item position */
      mov r2, r5      /* third parameter: item value */
      bl printf       /* call printf */
 
      add r4, r4, #1             /* r4 ←  r4 + 1 */
    .Lcheck_loop_print_items:
      cmp r4, r6                 /* r4 - r6 and update cpsr */
      bne .Lloop_print_items       /* if r4 != r6 goto .Lloop_print_items */
 
    pop {r4, r5, r6, r7, r8, lr}
    bx lr
 
main:
    push {r4, lr}
    /* we will not use r4 but we need to keep the function 8-byte aligned */
 
    /* first call print_each_item */
    mov r0, #256                   /* first_parameter: number of items */
    ldr r1, address_of_big_array   /* second parameter: address of the array */
    bl print_each_item             /* call to print_each_item */
 
    /* call to double_array */
    mov r0, #256                   /* first_parameter: number of items */
    ldr r1, address_of_big_array   /* second parameter: address of the array */
    bl double_array               /* call to double_array */
 
    /* second call print_each_item */
    mov r0, #256                   /* first_parameter: number of items */
    ldr r1, address_of_big_array   /* second parameter: address of the array */
    bl print_each_item             /* call to print_each_item */
 
    pop {r4, lr}
    bx lr
 
address_of_big_array : .word big_array
address_of_message : .word message
```


<div style="page-break-after: always;"></div>

# 18. Local data and the frame pointer  
----

## Local Data

지금까지는 program existance 에 종속된 global 변수만 사용했다. 이번 장 에서는 memory existence 에 종속된 local 변수를 사용할 수 있는 방법에 대해 알아본다. Stack이 그 역할을 한다. Stack은 함수와 lifetime을 같이하는 [Dynamic Activation](10. Functions-2 The stack) 메모리 영역을 이용할 수 있게 한다. 그래서 함수 내에서만 사용하는 local data 를 사용할 수 있다.   

17장에서 stack을 사용해 배열의 데이터를 함수의 인자로 전달했는데, 이것이 일종의 local 변수 사용 방식과 유사하다.  

## Frame Pointer

17장에서 살펴봤듯이 함수 인자로 전달되는 많은 수의 parameter는 stack을 이용한다. 그리고 local 변수 또한 stack을 이용한다. 함수에서 동일한 stack 영역에 parameter와 local variable 이 두가지 데이터가 있어 혼란이 발생할 수 있다. 이럴 경우 어떻게 해결해야하나?  

이를 해결하기 위해 frame pointer 개념이 생겼다. Frame pointer는 스택내에 있는 일종의 포지션 마커이다. 이 마커는 한 함수의 Activation Record(Stack Frame)에서 parameter와 local veriable을 구분하는데 사용 한다. (A frame pointer is a sort of marker in the stack that we will use to tell apart local variables from parameters. frame pointer gives us a consistent solution to access local data and parameters in the stack.)  

ARM gcc에서는 Frame Pointer를 `fp` 로 사용한다. 이것은 보통 r11 이다. 참고로 AAPCS(ARMv6)에는 fp가 정의되어있지 않다.   


## Dynamic link of the activation record

Activation Record (stack frame) 은 호출된 하나의 함수가 사용하는 Stack내의 영역 및 context이다.   

```asm
function:
  /* Keep callee-saved registers */
  push {r4, r5, fp, lr} /* Keep the callee saved registers.
                           We added r5 to keep the stack 8-byte aligned
                           but the important thing here is fp */
  mov fp, sp            /* fp ←  sp. Keep dynamic link in fp */
  sub sp, sp, #8        /* Enlarge the stack by 8 bytes */
  ... /* code of the function */
  mov sp, fp            /* sp ←  fp. Restore dynamic link in fp */
  pop {r4, r5, fp, lr}  /* Restore the callee saved registers.
                           This will restore fp as well */
  bx lr         /* Return from the function */
```
> 코드 수행후 stack은 아래와 같은 모습이된다. sp는 top을 가리키지만 fp는 아니다. fp ~ sp 사이가 local data가 위치할 영역이다.   
> `sub sp, sp, #8` : stack에 local val로 사용할 메모리 영역을 alloc 하는것과 같다.  
> `mov fp, sp` : fp를 top of stack으로 update하는것을 keeping dynamic link 라고 한다.   
> dynamic link는 새롭게 늘리기 이전 스택 포인터를 의미하는듯  
> `mov sp, fp` : 명령은 alloc했던 local data영역을 free하는 역할을 한다.  

![fpsp](https://thinkingeek.com/wp-content/uploads/2014/05/path48641.png)


## What about parameters passed in the stack?

만약 parameter도 stack으로 전달되었다면?  아래 그림과 같이 된다.  
현재 함수의 `fp`는 local variable 의 base 주소를 가리키고 있으며, old 'fp'는 caller 함수의 local variable base 주소를 가리킨다.  
그림에서 Local data of the caller 는 caller함수에서 alloc된 local data 이고 추후에 caller함수가 종료될때 free될 영역이다. 마치 현재 함수도 8byte 만큼 sp 를 증가시킨것과 동일함.  

![fpsp2](https://thinkingeek.com/wp-content/uploads/2014/05/path48643.png)


## Indexing through the frame pointer

local data는 항상 fp보다 낮은 주소에 위치해야한다. (Stack이 low 주소로 grow하는 경우). `

아래와 같은 C 함수는 어떻게 Asm으로 변역될 것인가?  

```asm
int sq_sum5(int a, int b, int c, int d, int e)
{
  sq(&a);
  sq(&b);
  sq(&c);
  sq(&d);
  sq(&e);
  return a + b + c + d + e;
}
```
> sq() 함수로 전달된 인자가 5개이기 때문에 a ~ d 까지는 `r0` ~ `r3`까지 저장되겠지만 e는 stack에 저장된다. 

```asm
sq: 
  ldr r1, [r0]   /* r1 ←  (*r0) */
  mul r1, r1, r1 /* r1 ←  r1 * r1 */
  str r1, [r0]   /* (*r0) ←  r1 */
  bx lr          /* Return from the function */
```
> 참고로 sq는 제곱하는 함수(by reference).  

아래 코드와 같이 reg로 전달된 r0 ~ r3 까지를 local data stack 영역에 push하면 계산이 편해진다.  

```asm

sq_sum5:
  push {fp, lr}         /* Keep fp and all callee-saved registers. */
  mov fp, sp            /* Set the dynamic link 현재 fp를 stack의 top으로 변경 */ 
 
  sub sp, sp, #16       /* sp ←  sp - 16. Allocate space for 4 integers in the stack */
                        /* 인자로 전달된 r0 ~ r3를 저장할 local data영역 allocate */

  /* Keep parameters in the stack */
  str r0, [fp, #-16]    /* *(fp - 16) ←  r0 */
  str r1, [fp, #-12]    /* *(fp - 12) ←  r1 */
  str r2, [fp, #-8]     /* *(fp - 8) ←  r2 */
  str r3, [fp, #-4]     /* *(fp - 4) ←  r3 */
 
  /* At this point the stack looks like this
     | Value  |  Address(es)
     +--------+-----------------------
     |   r0   |  [fp, #-16], [sp]
     |   r1   |  [fp, #-12], [sp, #4]
     |   r2   |  [fp, #-8],  [sp, #8]
     |   r3   |  [fp, #-4],  [sp, #12]
     |   fp   |  [fp],       [sp, #16]    fp: 기준 위치
     |   lr   |  [fp, #4],   [sp, #20]
     |   e    |  [fp, #8],   [sp, #24]    함수 호출시 stack에 push된 인자 e 
     v
   Higher addresses */
 
  sub r0, fp, #16    /* r0 ←  fp - 16 */
  bl sq              /* call sq(&a); */
  sub r0, fp, #12    /* r0 ←  fp - 12 */
  bl sq              /* call sq(&b); */
  sub r0, fp, #8     /* r0 ←  fp - 8 */
  bl sq              /* call sq(&c); */
  sub r0, fp, #4     /* r0 ←  fp - 4 */
  bl sq              /* call sq(&d) */
  add r0, fp, #8     /* r0 ←  fp + 8 */
  bl sq              /* call sq(&e) */
 
  ldr r0, [fp, #-16] /* r0 ←  *(fp - 16). Loads a into r0 */
  ldr r1, [fp, #-12] /* r1 ←  *(fp - 12). Loads b into r1 */
  add r0, r0, r1     /* r0 ←  r0 + r1 */
  ldr r1, [fp, #-8]  /* r1 ←  *(fp - 8). Loads c into r1 */
  add r0, r0, r1     /* r0 ←  r0 + r1 */
  ldr r1, [fp, #-4]  /* r1 ←  *(fp - 4). Loads d into r1 */
  add r0, r0, r1     /* r0 ←  r0 + r1 */
  ldr r1, [fp, #8]   /* r1 ←  *(fp + 8). Loads e into r1 */
  add r0, r0, r1     /* r0 ←  r0 + r1 */
 
  mov sp, fp         /* Undo the dynamic link */
  pop {fp, lr}       /* Restore fp and callee-saved registers */
  bx lr              /* Return from the function */
```


위의 __sq_sum5:__ 함수를 호출하는 Caller는 아래와 같다.   
bl로 sq_sum5: 를 호출하기 전에 r0 ~ r3까지 인자를 직접 저장해야하고 그 이상의 인자는 직접 stack에 push 해야한다.  

```asm
/* squares.s */
.data
 
.text
 
sq:
  <<defined above>>
 
sq_sum5:
  <<defined above>>
 
.globl main
 
main:
    push {r4, lr}          /* Keep callee-saved registers */
 
    /* Prepare the call to sq_sum5 */
    mov r0, #1             /* Parameter a ←  1 */
    mov r1, #2             /* Parameter b ←  2 */
    mov r2, #3             /* Parameter c ←  3 */
    mov r3, #4             /* Parameter d ←  4 */
 
    /* Parameter e goes through the stack,
       so it requires enlarging the stack */
    mov r4, #5             /* r4 ←  5 */
    sub sp, sp, #8         /* Enlarge the stack 8 bytes,
                              we will use only the
                              topmost 4 bytes */
    str r4, [sp]           /* Parameter e ←  5 */
    bl sq_sum5             /* call sq_sum5(1, 2, 3, 4, 5) */
    add sp, sp, #8         /* Shrink back the stack */
 
    /* Prepare the call to printf */ .. 중략
 
    pop {r4, lr}           /* Restore callee-saved registers */
    bx lr
 
 
```
> `str r4, [sp]` : 로 5번째 Parameter "e"를 미리 stack에 push했음에 유의


<div style="page-break-after: always;"></div>

# 19. System calls
----

유저 프로그램의 관점에서 보면, 운영체제는 수 많은 서비스를 제공하는 거대한 Servant와 같다. 유저 프로그램이 요청할때, 항상 응답할 수 있도록 기본적인 프로그램들이 운영체제에 존재하는데, 이것은 항상 Running(적어도 Ready) 상태이다. 운영체제는 리소스를 할당하기 위해서 Process라는 독립적인 요소가 필요하다. Process는 Running상태의 프로그램을 의미한다. 하나의 프로그램은 여러번 실행 될 수 있는데, 하나의 프로그램은 서로다른 Process의 형태로 실행된다. 

이번장에서는 이러한 유저 프로그램과 운영체제의 상호작용에 대해 알아본다.  

## System calls

유저 프로세스는 운영체제와 System Call로 상호작용한다. 시스템 콜은 개념적으로는 함수를 호출하는것과 유사하지만 그보다는 정교하고 복잡하다.  

ARM Linux 에서는 시스템콜을 사용하기 위해 `swi` 명령어를 사용한다. `swi` 는 software interrupt를 의미한다. 이 명령어는 24 bit의 Operand가 있지만 리눅스에서는 항상 0이다. 즉 시스템 콜을 사용하기 위해 항상 `swi #0` 으로 사용된다.

리눅스 시스템콜 호출 규약: 
	* register는 r7 사용
	* 7개의 parameter를 전달 받을 수 있다. r0 ~ r6
	* r0 로 값을 return 한다.
	* AAPCS와 호환되지 않음에 유의

## Hello world, the system call way

System call `write` 를 사용한다. write는 세가지 Parameter를 전달 받는다. 1. File descriptor 2. Data의 Pointer 3.   
File Descriptor 는 파일을 구분하는 숫자이며 프로세스에게 할당한다. 프로세스는 다음의 세가지중 하나로 미리할당 하며 시작한다. standard inpu1 (0), standard output (1), standard error(2)  

### The “ea-C” way
 
```asm
ssize_t write(int fd, const void *buf, size_t count);
```
> Linux man page에 정의된 write system call의 함수 원형  

write 시스템 콜 호출하는 코드: 기존 함수 호출방식과 동일  

```asm
/* write_c.s */
 
.data
 
greeting: .asciz "Hello world\n"
after_greeting:
 
/* This is an assembler constant: the assembler will compute it. Needless to say
   that this must evaluate to a constant value. In this case we are computing the
   difference of addresses between the address after_greeting and greeting. In this
   case it will be 13 */
.set size_of_greeting, after_greeting - greeting
 
.text
 
.globl main
 
main:
    push {r4, lr}
 
    /* Prepare the call to write */  
    mov r0, #1                /* First argument: 1 */
    ldr r1, addr_of_greeting  /* Second argument: &greeting */
    mov r2, #size_of_greeting /* Third argument: sizeof(greeting) */
    bl write                  /* write(1, greeting, sizeof(greeting));
 
    mov r0, #0
    pop {r4, lr}
    bx lr
 
addr_of_greeting : .word greeting
```
> `bl write`: 기존 함수 호출 방식과 동일. 직접 write함수를 호출했다.       

### The system call way

실제로 시스템콜은 어떻게 호출 하는가?    
먼저 시스템콜 번호를 `r7`에 저장한뒤 parameter를 전할하여 `swi`를 호출한다.    
write의 시스템콜 번호는 4인데 /usr/include/arm-linux-gnueabihf/asm/unistd.h 에 정의되어 있다.    

```asm
/* write_sys.s */
 
.data
 
 
greeting: .asciz "Hello world\n"
after_greeting:
 
.set size_of_greeting, after_greeting - greeting
 
.text
 
.globl main
 
main:
    push {r7, lr}
 
    /* Prepare the system call */
    mov r0, #1                  /* r0 ←  1 */
    ldr r1, addr_of_greeting    /* r1 ←  &greeting */
    mov r2, #size_of_greeting   /* r2 ←  sizeof(greeting) */
 
    mov r7, #4                  /* select system call 'write' */
    swi #0                      /* perform the system call */
 
    mov r0, #0
    pop {r7, lr}
    bx lr
 
addr_of_greeting : .word greeting
```
> `mov r7, #4`  
> `swi #0`   
> branching 명령으로 함수를 call하지않고 `swi` 명령 사용해서 호출함.  

참고로 최근 ARM 문서에`swi`는 `svc`바뀌었다고 함. 그러나 여전히 swi사용 가능.  
[http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0204ik/Cihidabi.html](http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0204ik/Cihidabi.html)


<div style="page-break-after: always;"></div>

# 20. Indirect calls
----

Indirect calls 은 이름(label)을 가지고 함수를 호출하는 것이 아니라, 주소를 이용해 함수를 호출하는 것을 의미한다. 이방법을 사용하면 주소를 직접 계산해서 호출 할수 있고,  배열이나 구조체 클래스 처럼 offset이 있는 함수들의 주소를 쉽게 계산해서 호출할 수 있다.  


## Labels

label은 assembler 가 머신코드로 변환할때 주소로 바꿈.

```asm
fun: /* label 'fun' */
  push {r4, r5}
  ...
  pop {r4, r5}
  bx lr

 ...
  bl fun
```

## Our first indirect call


```asm
.data     /* data section */
...
.align 4  /* ensure the next label is 4-byte aligned */
ptr_of_fun: .word 0   /* we set its initial value zero */
```

```asm
.align 4
make_indirect_call:
    push {r4, lr}            /* keep lr because we call printf, 
                                we keep r4 to keep the stack 8-byte
                                aligned, as per AAPCS requirements */
    ldr r0, addr_ptr_of_fun  /* r0 ←  &ptr_of_fun */
    ldr r0, [r0]             /* r0 ←  *r0 */
    blx r0                   /* indirect call to r0 */
    pop {r4, lr}             /* restore r4 and lr */
    bx lr                    /* return to the caller */
 
addr_ptr_of_fun: .word ptr_of_fun
```
> `blx r0`: `bl label` 대신에 register에 있는 address 로 indirect call을 했다.  
> 이전장에 언급했드시 `bx`는 `lr`값을 자동으로 save하지 않기 때문에  indirect call로 사용할 수 없다.  


```asm
main:
    ...
    ldr r1, addr_say_hello   /* r1 ←  &say_hello */
    ldr r0, addr_ptr_of_fun  /* r0 ←  &addr_ptr_of_fun */
    str r1, [r0]             /* *r0 ←  r1
                                this is
                                ptr_of_fun ←  &say_hello */
    ...
    bl make_indirect_call
```
> 이런식으로 주소를 변경 가능

## 장점?

Being able to call a function indirectly is a very powerful thing. It allows us to keep the address of a function somewhere and call it. It allows us to pass the address of a function to another function.  

아래 코드를 보자. indirect call의 장점을 확인할 수 있다.   



```asm
.data     /* data section */
 
...
.text     /* text section (= code) */
 
say_hello:
    ...중략 print hello 하는 함수
 
say_bonjour:
    ...중략 print bonjour 하는 함수
 
 
.align 4
greeter:
    push {r4, lr}            /* keep lr because we call printf, 
                                we keep r4 to keep the stack 8-byte
                                aligned, as per AAPCS requirements */
    blx r0                   /* indirect call to r0 */
    pop {r4, lr}             /* restore r4 and lr */
    bx lr                    /* return to the caller */
 
.globl main /* state that 'main' label is global */

main:
    push {r4, lr}            /* keep lr because we call printf, 
                                we keep r4 to keep the stack 8-byte
                                aligned, as per AAPCS requirements */
 
    ldr r0, addr_say_hello   /* r0 ←  &say_hello */
    bl greeter               /* call greeter */
 
    ldr r0, addr_say_bonjour /* r0 ←  &say_bonjour */
    bl greeter               /* call greeter */
 
    mov r0, #0               /* return from the program, set error code */
    pop {r4, lr}             /* restore r4 and lr */
    bx lr                    /* return to the caller (the system) */
 
addr_say_hello : .word say_hello
addr_say_bonjour : .word say_bonjour
```
> 길지만 별거 없고, greeter 함수에서 인자로 전달받은 함수 주소를 `blx r0` 로 indirectly 하게 call 할 수 있다는 예제이다.  


## 다음 일련의 예제들을 보면 장점이 더욱 명확해 진다.  

- printf format 문자열

```asm
.data     /* data section */
 
.align 4  /* ensure the next label is 4-byte aligned */
message_hello: .asciz "Hello %s\n"
.align 4  /* ensure the next label is 4-byte aligned */
message_bonjour: .asciz "Bonjour %s\n"
```

- 

```asm
/* tags of kind of people */
.align 4  /* ensure the next label is 4-byte aligned */
person_english : .word say_hello /* tag for people
                                     that will be greeted 
                                     in English */
.align 4  /* ensure the next label is 4-byte aligned */
person_french : .word say_bonjour /* tag for people
                                     that will be greeted 
                                     in French */
```

```asm
/* several names to be used in the people definition */
.align 4
name_pierre: .asciz "Pierre"
.align 4
name_john: .asciz "John"
.align 4
name_sally: .asciz "Sally"
.align 4
name_bernadette: .asciz "Bernadette"
```

```asm
.align 4
person_john: .word name_john, person_english
.align 4
person_pierre: .word name_pierre, person_french
.align 4
person_sally: .word name_sally, person_english
.align 4
person_bernadette: .word name_bernadette, person_french

/* array of people */
people : .word person_john, person_pierre, person_sally, person_bernadette
```

- hello 출력 함수 정의

```asm
.text     /* text section (= code) */
 
.align 4  /* ensure the next label is 4-byte aligned */
say_hello:
    push {r4, lr}            /* keep lr because we call printf, 
                                we keep r4 to keep the stack 8-byte
                                aligned, as per AAPCS requirements */
    /* Prepare the call to printf */
    mov r1, r0               /* r1 ←  r0 */
    ldr r0, addr_of_message_hello
                             /* r0 ←  &message_hello */
    bl printf                /* call printf */
    pop {r4, lr}             /* restore r4 and lr */
    bx lr                    /* return to the caller */
 
.align 4  /* ensure the next label is 4-byte aligned */
addr_of_message_hello: .word message_hello
 
.align 4  /* ensure the next label is 4-byte aligned */
say_bonjour:
    push {r4, lr}            /* keep lr because we call printf, 
                                we keep r4 to keep the stack 8-byte
                                aligned, as per AAPCS requirements */
    /* Prepare the call to printf */
    mov r1, r0               /* r1 ←  r0 */
    ldr r0, addr_of_message_bonjour
                             /* r0 ←  &message_bonjour */
    bl printf                /* call printf */
    pop {r4, lr}             /* restore r4 and lr */
    bx lr                    /* return to the caller */
 
.align 4  /* ensure the next label is 4-byte aligned */
addr_of_message_bonjour: .word message_bonjour
```

- main 함수 

```asm
.globl main /* state that 'main' label is global */
.align 4  /* ensure the next label is 4-byte aligned */
main:
    push {r4, r5, r6, lr}    /* keep callee saved registers that we will modify */
 
    ldr r4, addr_of_people   /* r4 ←  &people */
    /* recall that people is an array of addresses (pointers) to people */
 
    /* now we loop from 0 to 4 */
    mov r5, #0               /* r5 ←  0 */
    b check_loop             /* branch to the loop check */
 
    loop:
      /* prepare the call to greet_person */
      ldr r0, [r4, r5, LSL #2]  /* r0 ←  *(r4 + r5 << 2)   this is
                                   r0 ←  *(r4 + r5 * 4)
                                   recall, people is an array of addresses,
                                   so this is
                                   r0 ←  people[r5]
                                */
      bl greet_person           /* call greet_person */
      add r5, r5, #1            /* r5 ←  r5 + 1 */
    check_loop:
      cmp r5, #4                /* compute r5 - 4 and update cpsr */
      bne loop                  /* if r5 != 4 branch to loop */
 
    mov r0, #0               /* return from the program, set error code */
    pop {r4, r5, r6, lr}     /* callee saved registers */
    bx lr                    /* return to the caller (the system) */
 
addr_of_people : .word people
```
> `ldr r0, [r4, r5, LSL #2]` : 호출할 함수의 이름을 알 필요없이 주소값을 계산해서 call할 수 있다는 장점이있다.  
> greet_person 함수 호출 (parameter with 계산된 함수 주소)


- greet_person 함수 정의

이 함수에서 매개변수로 전달 받은 주소값에서 실제 함수 주소를 추출해서 branch 한다. 

```asm
/* This function receives an address to a person */
.align 4
greet_person:
    push {r4, lr}            /* keep lr because we call printf, 
                                we keep r4 to keep the stack 8-byte
                                aligned, as per AAPCS requirements */
 
    /* prepare indirect function call */
    mov r4, r0               /* r0 ←  r4, keep the first parameter in r4 */
    ldr r0, [r4]             /* r0 ←  *r4, this is the address to the name
                                of the person and the first parameter
                                of the indirect called function*/
 
    ldr r1, [r4, #4]         /* r1 ←  *(r4 + 4) this is the address
                                to the person tag */
    ldr r1, [r1]             /* r1 ←  *r1, the address of the
                                specific greeting function */
 
    blx r1                   /* indirect call to r1, this is
                                the specific greeting function */
 
    pop {r4, lr}             /* restore r4 and lr */
    bx lr                    /* return to the caller */
```
> 이름 tag를 찾으면 그에 해당하는 함수 호출이 가능함. 마치 구조체처럼.   
> Indirect call 을 사용하면 OOP처럼 함수 호출이 가능해짐.  


## Late binding and object orientation

위의 예제는 아래와 같은 객체지향 C++ 코드와 동일하다.   
A feature of the object-oriented programming (OOP) called late binding, which means that one does not know which function is called for a given object

```c++
struct Person {
  const char* name;
  virtual void greet() = 0;
};
 
struct EnglishPerson : Person {
  virtual void greet() {
    printf("Hello %s\n", this->name);
  }
};
 
struct FrenchPerson : Person {
  virtual void greet() {
    printf("Bonjour %s\n", this->name);
  }
};
```



<div style="page-break-after: always;"></div>

# 21. Subword data  
----

sub-word 즉, word interger 32bit size보다 작은 데이터를 사용하려면 어떻게 해야하나?  


## Subword data  

byte(2byte), half word(4byte) 를 사용하는 방법.   

```asm
.align 4
one_byte: .byte 205
/* This number in binary is 11001101 */
 
.align 4
one_halfword: .hword 42445
/* This number in binary is 1010010111001101 */
```
> `.byte` 1byte 만 값 저장  
> `.hword`2byte 만 값 저장  

## load and store 

ARM은 1Byte 2Byte 데이터를 load하기 위해서 다음과 같은 명령어를 제공한다.  
`ldrb` `ldrh` load되지 않는 나머지 비트들은 0으로 채워진다.  

- __Load__

	```asm
	.text
	 
	.globl main
	main:
	    push {r4, lr}
	 
	    ldr r0, addr_of_one_byte     /* r0 ←  &one_byte */
	    ldrb r0, [r0]                /* r0 ←  *{byte}r0 */
	 
	    ldr r1, addr_of_one_halfword /* r1 ←  &one_halfword */
	    ldrh r1, [r1]                /* r1 ←  *{half}r1 */
	 
	    pop {r4, lr}
	    mov r0, #0
	    bx lr
	 
	addr_of_one_byte: .word one_byte
	addr_of_one_halfword: .word one_halfword
	```
	> `ldr`이 아니라, `ldrb` `ldrh` 사용됨 유의.  

- __메모리에 로딩되는 형태__

	만약 다음과 같은 값이 주소 [addr] 부터 1바이트씩 저장 되어있다면  

	|주소	| Data		|
	|-------|---------------|
	|addr 	| 11001101	|
	|addr+1 | 10100101	|

	`ldrb addr` 그리고 `ldrh addr` 했을때, register에 로딩되는 값의 모습은 다음과 같다. 

	| 명령어	| 32-bit register r0에 로딩된 값		|
	|---------------|---------------------------------------|
	|ldr**b**  r0 [addr] |00000000 00000000 00000000 11001101|
	|ldr**h**  r0 [addr] |00000000 00000000 10100101 11001101|

	참고로 Little Endian 아키텍쳐이다. 위 표처럼  Byte는 낮은 주소에 LSB 높은 주소에 MSB 순으로 놓인다. 

- __2's complement 로드하기__

	만약 음수값을 사용한다면 `ldrsb` `ldrsh`를 사용해야 한다.  
	(아래 표) 만약 1바이트 크기의 음수 값 -51 (0x11001101)을 `ldr`로 로드하면 r0에 205로 잘못 저장된다. 그럴때는 `ldrsb`로 로드해야 정상적으로 -51로 저장된다.   

	| 명령어	| 32-bit register r0에 로딩된 값		| 10진수 |
	|---------------|---------------------------------------|---------|
	|ldr**b**  r0 [addr]  |00000000 00000000 00000000 11001101| 205 |
	|ldr**sb**  r0 [addr] | 11111111 11111111 11111111 11001101| -51 |


- __Store__

	Store연산은 위의 2's complement 를 고려할 필요없이 `strb` `strh` 만 사용하면 된다. 

	```asm
	ldr r1, addr_of_one_byte     /* r0 ←  &one_byte */
	ldrsb r0, [r1]               /* r0 ←  *{signed byte}r1 */
	strb r0, [r1]                /* *{byte}r1 ←  r0 */

	ldr r0, addr_of_one_halfword /* r0 ←  &one_halfword */
	ldrsh r1, [r0]               /* r1 ←  *{signed half}r0 */
	strh r1, [r0]                /* *{half}r0 ←  r1 */
	```

### 참고 ldrd, strd

d 는 doubleword  

```
LDR{type}{cond}{.W} Rt, label
LDRD{cond} Rt, Rt2, label        ; Doubleword
```
> type:
`B`  
unsigned Byte (Zero extend to 32 bits on loads.)  
`SB`  
signed Byte (LDR only. Sign extend to 32 bits.)  
`H`  
unsigned Halfword (Zero extend to 32 bits on loads.)  
`SH`  
signed Halfword (LDR only. Sign extend to 32 bits.)  
-  
omitted, for Word.  

<div style="page-break-after: always;"></div>

# 22. The Thumb instruction set  
----

ARM사는 16 bit bus line을 가진 Memory에서도 효율적으로 사용할 수 있도록 ARM 명령어들을 16 bit로 압축한 명령어 set을 발표했    다.  
Memory자원이 부족한 임베디드 시스템에서 유용한 명령어. 

## The Thumb instruction set

지금까지 명령어 셋은 모두 32bit 크기를 가지고 있었다. (RISC processor의 일반적인 디자인이 그렇다.) 
Thumb instruction set 은 16bit의 크기를 갖는 명령어 셋이기 때문에 더 적은 메모리공간을 사용할 수 있다.  

## Instructions

Thumb은 45개의 명령어셋을 제공한다. (ARMv6에서는 115개 이다.)  16bit이라는 것은 명령어 하나로 할 수 있는 일에 한계가 있다는 뜻이다. 다음과 같은 한계가 있다.   
	* Register가 두 세트로 나뉘어 있다. low register: r0 ~ r7, high reigster: r8 ~ r15.  
	* 조건부 실행 명령 Predication을 사용할 수 없다. 가령, sub**eq** [11장 Predication](#11. Predication)    
	* 하나의 함수에서는 ARM 명령어와 Thumb 명령어의 둘중 하나만 사용할 수 있다.  

## ARM 명령어에서 Thumb 함수 호출하기

ARM명령어 에서 Thumb 함수 호출을 하거나 그 반대의 경우 무조건 `blx`명령어를 사용해서 호출해야한다. 

```asm
/* thumb-first.s */
.text
 
.code 16     /* Thumb 명령어 사용한다는 표기 */
.align 2     /* Make sure instructions are aligned at 2-byte boundary */
             /* Thumb 명령어 2byte align해야함 */
 
thumb_function:
    mov r0, #2   /* r0 ←  2 */
    bx lr        /* return */
 
.code 32     /* ARM 명령어 사용한다는 표기 */
.align 4     /* Make sure instructions are aligned at 4-byte boundary */
 
.globl main
main:
    push {r4, lr}
 
    blx thumb_function /* From ARM to Thumb we use blx */
 
    pop {r4, lr}
    bx lr
```
> 보이는 것처럼 Thumb 명령어 형태는 ARM과 다르지 않은 것들이 많다.   



- 실제로 Obj파일에 Thumb 명령어가 있는지 확인하는 방법? 
Thumb 명령어는 2바이트 단위로 있는것을 확인할 수 있다.  

```bash
 $ objdump  -d thumb-first.o 
 
thumb-first.o:     file format elf32-littlearm
 
 
Disassembly of section .text:
 
00000000 <thumb_function>:
   0:	2002      	movs	r0, #2
   2:	4770      	bx	lr
   4:	e1a00000 	nop			; (mov r0, r0)
   8:	e1a00000 	nop			; (mov r0, r0)
   c:	e1a00000 	nop			; (mov r0, r0)
 
00000010 <main>:
  10:	e92d4010 	push	{r4, lr}
  14:	fafffff9 	blx	0 <thumb_function>
  18:	e8bd4010 	pop	{r4, lr}
  1c:	e12fff1e 	bx	lr
```
>  0:	2002      	movs	r0, #2
>  2:	4770      	bx	lr


## Calling functions in Thumb

Thumb 에서는 아래와 같이 AAPCS처럼 push pop을 할 수 없다.  
`push`는 low reg와 `lr` `pop`은 low reg와 `pc` 밖에 사용할 수 없다.  

```asm
.code 16     /* Here we say we will use Thumb */
.align 2     /* Make sure instructions are aligned at 2-byte boundary */

thumb_function_2:
    /* Do something here */
    bx lr

thumb_function_1:
    push {r4, lr}
    bl thumb_function_2
    pop {r4, lr}    /* 여기서 ERROR: lr을 pop에서 사용할 수 없다. */
    bx lr
```

그래서 아래와 같은 방식의 trick을 사용한다.  

```asm
.code 16     /* Here we say we will use Thumb */
.align 2     /* Make sure instructions are aligned at 2-byte boundary */
 
thumb_function_2:
    mov r0, #2
    bx lr   /* A leaf Thumb function (i.e. a function that does not call
               any other function so it did not have to keep lr in the stack)
               returns using "bx lr" */
 
thumb_function_1:
    push {r4, lr}
    bl thumb_function_2 /* From Thumb to Thumb we use bl */
    pop {r4, pc}  /* This is how we return from a non-leaf Thumb function */
 
```
> pop {r4, pc} : lr대신 pc  

Thumb모드에서는 `pop {pc}` 는 스택의 top에서 값 val0 을 하나 pop한 뒤 `bx val0` 하는것과  동일하다. 


## Thumb 명령어에서 ARM 함수 호출하는 방법 

마찬가지로 `blx` 로 호출해야한다.  


```asm
/* thumb-first.s */
 
.text
 
.data
message: .asciz "Hello world %d\n"
 
.code 16     /* Here we say we will use Thumb */
.align 2     /* Make sure instructions are aligned at 2-byte boundary */
thumb_function:
    push {r4, lr}         /* keep r4 and lr in the stack */
    mov r4, #0            /* r4 ←  0 */
    b check_loop          /* unconditional branch to check_loop */
    loop:        
       /* prepare the call to printf */
       ldr r0, addr_of_message  /* r0 ←  &message */
       mov r1, r4               /* r1 ←  r4 */
       blx printf               /* From Thumb to ARM we use blx.
                                   printf is a function
                                   in the C library that is implemented
                                   using ARM instructions */
       add r4, r4, #1           /* r4 ←  r4 + 1 */
    check_loop:
       cmp r4, #4               /* compute r4 - 4 and update the cpsr */
       blt loop                 /* if the cpsr means that r4 is lower than 4 
                                   then branch to loop */
 
    pop {r4, pc}          /* restore registers and return from Thumb function */
.align 4
addr_of_message: .word message
 
.code 32     /* Here we say we will use ARM */
.align 4     /* Make sure instructions are aligned at 4-byte boundary */
.globl main
main:  
    push {r4, lr}      /* keep r4 and lr in the stack */
    blx thumb_function /* from ARM to Thumb we use blx  */       
    pop {r4, lr}       /* restore registers */
    bx lr              /* return */
```

아래 링크에서 Thumb 모드에대해 더 자세히 확인할 수 있다.  
http://infocenter.arm.com/help/topic/com.arm.doc.qrc0006e/QRC0006_UAL16.pdf



<div style="page-break-after: always;"></div>

# 23. Nested functions
----

Nested function 은 함수 내부에 정의된 함수이다. 함수 내부에서만 사용가능하다. 마치 로컬변수 처럼 lifetime이 함수와 동일하다.  

```c
float E(float x)
{
    float F(float y)
    {
        return x + y;
    }
    return F(3) + F(4);
}
```
> F()는 함수E() 안에 있는 함수인데, 그 scope가 함수E()와 동일하다. 


## Nested function for assembler level

기본적으로 Assembler 레벨에서는 함수는 Nested될 수 없다. (~~사실 따지고 보면 함수라는것도 존재하지 않는다~~) 함수는 Calling Convention(AAPCS) 과 함께 논리적으로 존재할 뿐이다.  Assembler 레벨에서는 모든것은 데이터나 명령어 아니면 주소이다. 

Assembler 레벨에서 Nested Function의 의미는 Dynamic Activate 범위 내 에서만 존재(life tiem을 같이하는)하는 함수를 의미한다. 

## Dynamic link

Dynamic link 란?  
The dynamic link is set at the beginning of the function and we use the fp register (an alias for r11) to keep an address in the stack usually called the frame pointer.  

frame pointer를 이용하면 dynamic activation 내의 로컬변수를 일관되게 엑세스할 수 있다. 로컬변수는 `fp`로부터 (-)offset, 전달받은 parameter는 (+)offset 으로 얻을 수 있다. 아래 그림에서 activation record 가 스택에 어떻게 쌓이는지 보여준다.  
(리눅스 ARM에서는 낮은 주소 방향으로 스택 사이즈가 커진다는것을 항상 유의)

![activation record](https://thinkingeek.com/wp-content/uploads/2015/01/activation_record.png)


## Static link

가정해보자, 위의 그림에서 함수 g가 함수 f의 Nested function이라면? 함수 g는 f의 로컬변수를 Access할 수 있어야한다. 그것은 previous fp를 얻음으로써 가능하다.    
결국, 이 장에서 우리의 최종 관심사는 어떻게 Caller함수의 로컬 변수를 Nedted함수에서 Access할수 있는가이다.  

다음과 같은 C nested 함수가 있다고 하자. 그림처럼 함수 g()가 함수 f()의 Nested function 이다.  

#### 예제 1

```c
void f() // non-nested (normal) function
{
    int x;
    void g()     // nested function
    {
      x = x + 1; // x ←  x + 1
    }
  
    x = 1;       // x ←  1
    g();         // call g
    x = x + 1;   // x ←  x + 1

}
```
> 최종 x 는 3이 될것이다.  
> Nested function g()에서 f()의 로컬변수 x를 접근해서 사용했다.  

어셈블리 레벨에서 g()가 f()의 로컬변수 x를 어떻게 Access할 수 있을까? g()함수에서 이전의 frame pointer 즉, 함수 f()의 `fp`를 얻어야한다.  


```asm
.text
 
f:
    push {r4, r5, fp, lr} /* keep registers */
    mov fp, sp /* keep dynamic link */
 
    sub sp, sp, #8      /* make room for x (4 bytes)
                           plus 4 bytes to keep stack
                           aligned */
    /* x is in address "fp - 4" */
 
    mov r4, #0          /* r4 ←  0 */
    str r4, [fp, #-4]   /* x ←  r4 */
 
    bl g                /* call (nested function) g
                           (the code of 'g' is given below, after 'f') */
 
    ldr r4, [fp, #-4]   /* r4 ←  x */
    add r4, r4, #1      /* r4 ←  r4 + 1 */
    str r4, [fp, #-4]   /* x ←  r4 */
 
    mov sp, fp /* restore dynamic link */
    pop {r4, r5, fp, lr} /* restore registers */
    bx lr /* return */
 
    /* nested function g */
    g:
        push {r4, r5, fp, lr} /* keep registers */
        mov fp, sp /* keep dynamic link */
 
        /* At this point our stack looks like this
 
          Data | Address | Notes
         ------+---------+--------------------------
           r4  | fp      |  
           r5  | fp + 4  |
           fp  | fp + 8  | This is the previous fp
                           ^^^^^^^^^^^^^^^^^^^^^^^
           lr  | fp + 16 |
        */
 
        ldr r4, [fp, #+8] /* get the frame pointer of my caller (since only f can call me) */
        ^^^^^^^^^^^^^^^^^
        /* now r4 acts like the fp we had inside 'f' */
        ldr r5, [r4, #-4] /* r5 ←  x */
        add r5, r5, #1    /* r5 ←  r5 + 1 */
        str r5, [r4, #-4] /* x ←  r5 */
 
        mov sp, fp /* restore dynamic link */
        pop {r4, r5, fp, lr} /* restore registers */
        bx lr /* return */
 
.globl main
 
main :
    push {r4, lr} /* keep registers */
 
    bl f          /* call f */
 
    mov r0, #0
    pop {r4, lr}
    bx lr
```
> - 함수 g 의 Activation Record 내에있는 보존된 레지스터들 중 하나가 caller함수 f의 `fp` 를 가지고 있다. 따라서 fp에서 (+)offset을 통해 previous fp를 얻을 수 있다. (이것은 마치 caller가 전달한 parameter를 얻는 것과 동일하다) 따로 stack operation(push/pop)을 통해 이전 fp를 얻어 낼 필요가 없다는 뜻이다.   
> - `ldr r4, [fp, #+8]` : [fp, #+8] 이 바로 이전 fp가 보존된 스택의 주소(가 가리키는 값)이다.  
> - 따라서 f()의 `fp`를 얻었으므로 f()의 로컬변수는 `fp`의 (-)offset을 통해 얻어낼수 있다.  
> - `ldr r5, [r4, #-4]` : [r4, #-4] 가 바로 local변수 x 의 주소(가 가리키는 값)이다.  

위의 예제에서는 함수 g()에서 현재`fp`에서 직접 offset 을 계산해 caller의 `fp`를 얻어왔다. 하지만 이 방식에는 명확한 한계가 있다. Nested function call을 한번밖에 못한다는 것이다!! 또다른 nested 함수를 호출하지 못할 뿐더러, 함수를 재귀적으로 호출 할 수도 없다. 따라서 호출된 함수 내에서 Frame pointer를 직접 이용하는 방법은 한계가 있다.  

따라서 항상 nested함수를 호출한 이전함수의 Activation 을 얻어와야 한다. 이 방식을 __Static Link__라고 부른다.
즉 Static Link는 실제 nested함수가 구현된, enclosing된 함수의 `fp` 를 의미한다. 단순이 caller의 `fp`가 아니다.  
Static Link의 개념은 간단하다. 서로 연결된 frame pointer의 체인이다. Static Link는 어떤 함수가 호출되느냐에 따라 서로 다르게 설정이 될 수 있다. 그리고 그 설정은 Caller가 한다.  다음 일련의 Assembly 예제들을 보면 무슨말인지 더 명확하게 이해할 수 있다.  

다음과 같은 중첩된 nested function 을 보자.   

#### 예제 2

```c
void f(void) // non nested (nesting depth = 0)
{
   int x;
 
   void g() // nested (nesting depth = 1)
   {
      x = x + 1; // x ←  x + 1
   }
   void h() // nested (nesting depth = 1)
   {
      void m() // nested (nesting depth = 2)
      {
         x = x + 2; // x ←  x + 2
         g(); // call g
      }
 
      g(); // call g
      m(); // call m
      x = x + 3; // x ←  x + 3
   }
 
   x = 1; // x ←  1
   h();  // call h
}
```
> 결과는 x 는 8이 될것이다.  
> h 와 g는 f에의해 둘러쌓이게(enclosing)된다.  
> m은 h에의해 둘러쌓이게(enclosing)된다.   
> m이 g를 호출하더라도 g의 static link는 m이 아니다. g가 어디서 호출되던 간에 g의 static link는 f임을 주의.   
> g는 f에 의해 enclosing되었기 때문이다.   

더욱 간단히 표현 하면 아래와 같다.  

```asm
f {
        g { }           // f 를 가리킴
        h {             // f 를 가리킴
                m { }   // h 를 가리킴
       	}
}
```

## Setting up a static link

AAPCS는 어떤레지스터를 Static link로 사용할지 강제하지는 않는다. 우리는 이 튜토리얼에서 `r10`을 사용할 것이다.  

Static Link를 setting할때에는 다음의 두가지 케이스를 고려해야 한다. 

* Nesting depth 1  
	예제 1
* Nesting depth 2 이상  
	예제 2

자, 그럼 이제 예제2 는 어떻게 assembly로 구현하는지 살펴 보자.  
핵심은 caller가 호출할 callee의 Static Link를 찾아서 `r10`에 넣어주고 호출한다는 것이다. 그 Static Link는 단순히 caller의 `fp`가 아닐수 있음을 위에서 설명하였다.   

__지금부터 나올 assembly 예제들의 주석을 꼼꼼히 살펴보자.__  

#### 함수 f()

```asm
f:
    push {r4, r10, fp, lr} /* keep registers */
    mov fp, sp             /* setup dynamic link */
 
    sub sp, sp, #8      /* make room for x (4 + 4 bytes) */
    /* x will be in address "fp - 4" */
 
    /* At this point our stack looks like this
 
     Data | Address | Notes
    ------+---------+---------------------------
          | fp - 8  | alignment (per AAPCS)
      x   | fp - 4  | 
      r4  | fp      |  
      r10 | fp + 8  | previous value of r10
      fp  | fp + 12 | previous value of fp
      lr  | fp + 16 |
   */
 
    mov r4, #1          /* r4 ←  1 */
    str r4, [fp, #-4]   /* x ←  r4 */
 
    /* prepare the call to h */
    mov r10, fp /* setup the static link,
                   since we are calling an immediately nested function
                   it is just the current frame */
    bl h        /* call h */
 
    mov sp, fp             /* restore stack */
    pop {r4, r10, fp, lr}  /* restore registers */
    bx lr /* return */
```
> 함수 f는 nested되지 않았기 때문에  `mov r10, fp` 여기서 이동작은 큰 의미가 없다.  

#### 함수 h()

```asm
h :
    push {r4, r5, r10, fp, lr} /* keep registers */
    mov fp, sp /* setup dynamic link */
 
    sub sp, sp, #4 /* align stack */
 
    /* At this point our stack looks like this
 
      Data | Address | Notes
     ------+---------+---------------------------
           | fp - 4  | alignment (per AAPCS)
       r4  | fp      |  
       r5  | fp + 4  | 
       r10 | fp + 8  | frame pointer of 'f'
       ^^^             ^^^^^^^^^^^^^^^^^^^^
       fp  | fp + 12 | frame pointer of caller
       lr  | fp + 16 |
    */
 
    /* g는 h와 형제이다. 따라서 h의 static link와 동일하다. */
    ldr r10, [fp, #8]
    ^^^^^^^^^^^^^^^^^
    bl g
 
    /* m은 h함수 내에서 nested되어있다. 따라서 m의 static link는 현재 fp와 동일하다 */
    mov r10, fp
    ^^^^^^^^^^^
    bl m
 
    ldr r4, [fp, #8]  /* load frame pointer of 'f' */
    ^^^^^^^^^^^^^^^^
    ldr r5, [r4, #-4]  /* r5 ←  x */
                         ^^^^^^^^^
    add r5, r5, #3     /* r5 ←  r5 + 3 */
    str r5, [r4, #-4]  /* x ←  r5 */
 
    mov sp, fp            /* restore stack */
    pop {r4, r5, r10, fp, lr} /* restore registers */
    bx lr
```
> * g 의 static link r10에 저장 `ldr r10, [fp, #8]`  
> * m 의 static link r10에 저장 `mov r10, fp`  
> * h 에서 f의 local 변수 얻기:  
>	`ldr r4, [fp, #8]`  /* load frame pointer of 'f' */  
>	`ldr r5, [r4, #-4]` : 결국 f의 로컬변수를 얻어옴
>	여기서 형제(Sibling)의 의미는 Nesting depth가 동일하다는것을 말한다.  

중첩된 nested 함수의 경우, 단순히 [예제 1](#예제 1) 처럼 `fp`를 이용하는 것이 아니다. `r10` 을 이용해서 static link를 미리 계산해서 전달하는 이유는 call하는 함수마다 static link를 다르게 계산 해 주어야 하기 때문이다. 위 예제에서 m()과 g()는 static link가 다르다. 함수 내에서 fp를 통해 계산할수 있지만 그러면 복잡해질 것이다. 미리 `r10` 에 계산하여 전달해주는것과 같다.  

#### 함수 m()

```asm
m:
    push {r4, r5, r10, fp, lr} /* keep registers */
    mov fp, sp /* setup dynamic link */
 
    sub sp, sp, #4 /* align stack */
    /* At this point our stack looks like this
 
      Data | Address | Notes
     ------+---------+---------------------------
           | fp - 4  | alignment (per AAPCS)
       r4  | fp      |  
       r5  | fp + 4  |
       r10 | fp + 8  | frame pointer of 'h'
       ^^^             ^^^^^^^^^^^^^^^^^^^^
       fp  | fp + 12 | frame pointer of caller
       lr  | fp + 16 |
    */
 
    ldr r4, [fp, #8]  /* r4 ←  frame pointer of 'h' */
    ^^^^^^^^^^^^^^^^
    ldr r4, [r4, #8]  /* r4 ←  frame pointer of 'f' */
    ^^^^^^^^^^^^^^^^
    ldr r5, [r4, #-4] /* r5 ←  x */
    add r5, r5, #2    /* r5 ←  r5 + 2 */
    str r5, [r4, #-4] /* x ←  r5 */
 
    /* setup call to g */
    ldr r10, [fp, #8]   /* r10 ←  frame pointer of 'h' */
    ^^^^^^^^^^^^^^^^^
    ldr r10, [r10, #8]  /* r10 ←  frame pointer of 'f' */
    ^^^^^^^^^^^^^^^^^^
    bl g
 
    mov sp, fp                /* restore stack */
    pop {r4, r5, r10, fp, lr} /* restore registers */
    bx lr
```
> `ldr r4, [fp, #8]` : h의 프레임포인터   
> `ldr r4, [r4, #8]` : 를 이용해 f 프레임포인터를 얻어옴  
> 함수 m()내에서 호출되는 __g() 의 static link는 f의 fp__이라는점이 중요하다. 그 때문에 호출하기 전에 위와 같이 계산함.  
> g()를 호출하는 caller가 m()이라고 하더라도 g()의 static link는 f()임에 주의  

#### 함수 g()


```asm
g:
    push {r4, r5, r10, fp, lr} /* keep registers */
    mov fp, sp /* setup dynamic link */
 
    sub sp, sp, #4 /* align stack */
 
    /* At this point our stack looks like this
 
      Data | Address | Notes
     ------+---------+---------------------------
           | fp - 4  | alignment (per AAPCS)
       r4  | fp      |  
       r5  | fp + 4  |  
       r10 | fp + 8  | frame pointer of 'f'
       ^^^             ^^^^^^^^^^^^^^^^^^^^
       fp  | fp + 12 | frame pointer of caller
       lr  | fp + 16 |
    */
 
    ldr r4, [fp, #8]  /* r4 ←  frame pointer of 'f' */
    ^^^^^^^^^^^^^^^^
    ldr r5, [r4, #-4] /* r5 ←  x */
    ^^^^^^^^^^^^^^^^^
    add r5, r5, #1    /* r5 ←  r5 + 1 */
    str r5, [r4, #-4] /* x ←  r5 */
 
    mov sp, fp /* restore dynamic link */
    pop {r4, r5, r10, fp, lr} /* restore registers */
    bx lr
```
> h와 비슷하나, 따로 함수를 호출하지는 않는다. 
> g와 h는 nesting depth가 동일함.  

지금까지 살펴본 예제 코드는 모두 fp + 8 로, static link를 얻어왔지만, 항상 그런것은 아니다. Caller가 r10을 push 하기전에  얼마나 많은 register를 push했는지에 따라 달라질 수 있기 때문이다.  
 
마지막으로 f()를 호출하는 main 함수는 아래와 같다.  

```asm
.globl main
 
main :
    push {r4, lr} /* keep registers */
 
    bl f          /* call f */
 
    mov r0, #0
    pop {r4, lr}
    bx lr
```
> nested 되지 않은 함수가 또다른 non-nested 함수를 호출할때는 r10에 어떤것도 할 필요가 없다.  


## 한계

실제로 r10은 함수내에서 사용되거나 변경되지 않았다. 그런데 왜 stack에 push하는가?  
동일한 함수 scope에 위치해야하기 때문이다? 이것을 Lexical scope라고 부른다. static link가 서로 chaining되어있기 때문에 각각의 lexical scope로 이동할 수 있다.  

`참고: Lexical scope`
C/C++, Java, 그리고 JavaScript 같이 우리가 접하는 대부분의 언어들은 Lexical Scope를 사용한다. Lexical Scope는 Static Scope라고도 불린다. Scope가 함수가 호출될때가 아니라 정의될때 생긴다는 뜻이다. Lexical Scope는 변수나 함수가 정의 된 곳의 context를 사용하며, Dynamic Scope는 변수나 함수가 불려진 곳의 context를 사용한다.   
__Lexical scope__: use environment where function [and variable] is __defined__  
__Dynamic scope__: use environment where function [and variable] is __called__  
 
출처: https://bestalign.github.io/2015/07/12/Lexical-Scope-and-Dynamic-Scope/  

callee가 r10을 stack에 push 하는대신에 caller가 stack에 push하는 방식으로 parameter로 전달할 수도 있다. 또한 물론 r0 ~ r3레지스터로 인자를 전달 할 수도 있다. 대신에 그러면 내부에서 stack에 다시 push해줘야한다.   

결국 static link는 항상 stack에 저장해야한다는 의미다.  

그런데 만약 static link를 이용해 접근해야할 로컬 변수가  pointer 라면 어떻게 해야하나?
지금까지 예제에서 우리는 어떤 위치 어떤 함수를 호출해야할지 미리 다 알고있었다. 만약 Indirect call로 함수를 호출할때는 어떻게 되는가? 우리는 어떤 nested함수가 호출될지 미리 알지 못하며 lexical socope를 어떻게 설정할지도 모른다. 결국 lexical sopce를 어딘가에 유지하지 않는이상 못한다.    

This means that just the address of the function will not do. We will need to keep, along with the address to the function, the lexical scope.
So a pointer to a nested function happens to be different to a pointer to a non-nested function, given that the latter does not need the lexical scope to be set.

다음 챕터에서는 non-nested함수와 포인터가 다른 서로다른 함수들의 포인터를 갖는것을 피하는 방법에 대해 알아보자 (?)

Having incompatible pointers for nested an non nested functions is not desirable. This may be a reason why C (and C++) do not directly support nested functions (albeit this limitation can be worked around using other approaches). In the next chapter, we will see a clever approach to avoid, to some extent, having different pointers to nested functions that are different from pointers to non nested functions. (~~?무슨말인지 모르겠음~~)






<div style="page-break-after: always;"></div>

# 24. Trampolines
----


이 장에서도 계속해서 Nested Function에 대해 설명한다.  

## Sorting with Nested Function

이번 장에서는 다음의 qsort 를 구현하면서 pointer를 사용하는 nested function 을 구현해 볼 것이다. C로된 함수 원형은 다음과 같다.    

```c
void qsort(void *base, size_t nmemb, size_t size, int (*compar)(const void *, const void *));
```

먼저 array를 print하는 골격코드를 보자.  

```asm
/* print-array.s */
 
.data
 
/* declare an array of 10 integers called my_array */
.align 4
my_array: .word 82, 70, 93, 77, 91, 30, 42, 6, 92, 64
 
/* format strings for printf */
/* format string that prints an integer plus a space */
.align 4
integer_printf: .asciz "%d "
/* format string that simply prints a newline */
.align 4
newline_printf: .asciz "\n"
 
.text
 
print_array:
    /* r0 will be the address of the integer array */
    /* r1 will be the number of items in the array */
    push {r4, r5, r6, lr}  /* keep r4, r5, r6 and lr in the stack */
 
    mov r4, r0             /* r4 ←  r0. keep the address of the array */
    mov r5, r1             /* r5 ←  r1. keep the number of items */
    mov r6, #0             /* r6 ←  0.  current item to print */
 
    b .Lprint_array_check_loop /* go to the condition check of the loop */
 
    .Lprint_array_loop:
      /* prepare the call to printf */
      ldr r0, addr_of_integer_printf  /* r0 ←  &integer_printf */
      /* load the item r6 in the array in address r4.
         elements are of size 4 bytes so we need to multiply r6 by 4 */
      ldr r1, [r4, +r6, LSL #2]       /* r1 ←  *(r4 + r6 << 2)
                                         this is the same as
                                         r1 ←  *(r4 + r6 * 4) */
      bl printf                       /* call printf */
 
      add r6, r6, #1                  /* r6 ←  r6 + 1 */
    .Lprint_array_check_loop: 
      cmp r6, r5               /* perform r6 - r5 and update cpsr */
      bne .Lprint_array_loop   /* if cpsr states that r6 is not equal to r5
                                  branch to the body of the loop */
 
    /* prepare call to printf */
    ldr r0, addr_of_newline_printf /* r0 ←  &newline_printf */
    bl printf
 
    pop {r4, r5, r6, lr}   /* restore r4, r5, r6 and lr from the stack */
    bx lr                  /* return */
 
addr_of_integer_printf: .word integer_printf
addr_of_newline_printf: .word newline_printf
 
.globl main
main:
    push {r4, lr}             /* keep r4 and lr in the stack */
 
    /* prepare call to print_array */
    ldr r0, addr_of_my_array  /* r0 ←  &my_array */
    mov r1, #10               /* r1 ←  10
                                 our array is of length 10 */
    bl print_array            /* call print_array */
 
    mov r0, #0                /* r0 ←  0 set errorcode to 0 prior returning from main */
    pop {r4, lr}              /* restore r4 and lr in the stack */
    bx lr                     /* return */
 
addr_of_my_array: .word my_array
```

결과는 다음과 같이 단순히 배열 contents 출력이다.  

```sh
 $ ./print-array 
 82 70 93 77 91 30 42 6 92 64
```

### Comparison

qsort의 마지막 인자인 compr 함수
비교하는 함수는 아래와 같이 구현한다. 


```asm
integer_comparison:
    /* r0 will be the address to the first integer */
    /* r1 will be the address to the second integer */
    ldr r0, [r0]    /* r0 ←  *r0
                       load the integer pointed by r0 in r0 */
    ldr r1, [r1]    /* r1 ←  *r1
                       load the integer pointed by r1 in r1 */
 
    cmp r0, r1      /* compute r0 - r1 and update cpsr */
    moveq r0, #0    /* if cpsr means that r0 == r1 then r0 ←   0 */
    movlt r0, #-1   /* if cpsr means that r0 <  r1 then r0 ←  -1 */
    movgt r0, #1    /* if cpsr means that r0 >  r1 then r0 ←   1 */
    bx lr           /* return */
```
> 11장 predication을 이용해 따로 branch하지 않음.  

qsort를 적용하기 전과후 배열 출력하는 최종 골격 코드는 아래와 같다. 

```asm

.globl main
main:
    push {r4, lr}             /* keep r4 and lr in the stack */
 
    /* prepare call to print_array */
    ldr r0, addr_of_my_array  /* r0 ←  &my_array */
    mov r1, #10               /* r1 ←  10
                                 our array is of length 10 */
    bl print_array            /* call print_array */
 
    /* prepare call to qsort */
    /*
    void qsort(void *base,
         size_t nmemb,
         size_t size,
         int (*compar)(const void *, const void *));
    */
    ldr r0, addr_of_my_array  /* r0 ←  &my_array
                                 base */
    mov r1, #10               /* r1 ←  10
                                 nmemb = number of members
                                 our array is 10 elements long */
    mov r2, #4                /* r2 ←  4
                                 size of each member is 4 bytes */
    ldr r3, addr_of_integer_comparison
                              /* r3 ←  &integer_comparison
                                 compar */
    bl qsort                  /* call qsort */
 
    /* now print again to see if elements were sorted */
    /* prepare call to print_array */
    ldr r0, addr_of_my_array  /* r0 ←  &my_array */
    mov r1, #10               /* r1 ←  10
                                 our array is of length 10 */
    bl print_array            /* call print_array */
 
    mov r0, #0                /* r0 ←  0 set errorcode to 0 prior returning from main */
    pop {r4, lr}              /* restore r4 and lr in the stack */
    bx lr                     /* return */
 
addr_of_my_array: .word my_array
addr_of_integer_comparison : .word integer_comparison
```

결과는 다음과 같음

```sh
 $ ./sort-array 
 82 70 93 77 91 30 42 6 92 64 
 6 30 42 64 70 77 82 91 92 93
```




## Count how many comparisons happen




```asm
.data
global_counter: .word 0
 
.text
integer_comparison_count_global:
    /* r0 will be the address to the first integer */
    /* r1 will be the address to the second integer */
    push {r4, r5}   /* keep callee-saved registers */
    ldr r0, [r0]    /* r0 ←  *r0
                       load the integer pointed by r0 in r0 */
    ldr r1, [r1]    /* r1 ←  *r1
                       load the integer pointed by r1 in r1 */
 
    cmp r0, r1      /* compute r0 - r1 and update cpsr */
    moveq r0, #0    /* if cpsr means that r0 == r1 then r0 ←   0 */
    movlt r0, #-1   /* if cpsr means that r0 <  r1 then r0 ←  -1 */
    movgt r0, #1    /* if cpsr means that r0 >  r1 then r0 ←   1 */
 
    ldr r4, addr_of_global_counter /* r4 ←  &global_counter */
    ldr r5, [r4]    /* r5 ←  *r4 */ 
    add r5, r5, #1  /* r5 ←  r5 + 1 */
    str r5, [r4]    /* *r4 ←  r5 */
 
    pop {r4, r5}    /* restore callee-saved registers */
    bx lr           /* return */
addr_of_global_counter: .word global_counter
```
> 이 함수는 다음 예제를 통해 nested function을 이용해 구현할 것이다.  

지난 챕터에서 nested functions의 한계에 대해 말했다. nested functions의 포인터는 두 가지가 필요하다. 함수의 주소와 Lexical scope다. 일단 다음 에서 그 한계를 어떻게 극복하는지 확인해보자.     

## Trampoline

이 장에서는 Nested function을 함수포인터로 indirect call 하여 static link를 통해 caller함수(enclosing function)의 로컬변수를 어떻게 Access하는지 보여줄 것이다. 거기서 Stack에 저장되고 실행되는 명령어 집합(template)인 Trampoline을 사용할 것이다. 일반적인 함수포인터를 indirect call 할 때, static link를 알아서 설정해주지 않기 때문에 그것을 대신해주는 trampoline코드를 사용하는 것이다.   

`참고: trampoline 이란`
> [gcc doc: Trampolines](https://gcc.gnu.org/onlinedocs/gccint/Trampolines.html)
> A trampoline is a small piece of code that is created at run time when the address of a nested function is taken. It normally resides on the stack, in the stack frame of the containing function. These macros tell GCC how to generate code to allocate and initialize a trampoline.  
The instructions in the trampoline must do two things: load a constant address into the static chain register, and jump to the real address of the nested function.   

지난 챕터에서 본 예제처럼 `r10`는 함수 진입시 Lexical scope(Static link)를 갖게 된다. 함수로 branch를 할때 r10값을 미리 셋팅해 주어야 한다. 그러나 qsort를 호출할때는 그렇지 못하다. qsort함수 내부에서 함수포인터 (*compar) 를 호출할 때, r10 (static link)를 저장한뒤 함수로 jump하는 동작을 스스로 하지 않는다. 따라서 무언가가 이동작을 대신 해줘야하는데 이 코드를 Trampoline 이라고 한다. 

Trampoline은 단순한 목적을 가지고 있다. 항상 다음의 동작만 한다.
1) r10을 설정  
2) Nested 함수를 Indirect Call   

```asm
.Laddr_trampoline_template : .word .Ltrampoline_template /* we will use this below */
.Ltrampoline_template:
    .Lfunction_called: .word 0x0     /* 후에 이 코드를 호출하기 전에 적절하게 set될 라인 */
    .Llexical_scope: .word 0x0       /* 후에 이 코드를 호출하기 전에 적절하게 set될 라인 */
    push {r4, r5, r10, lr}           /* keep callee-saved registers */
    ldr r4, .Lfunction_called        /* r4 ←  function called */
    ldr r10, .Llexical_scope         /* r10 ←  lexical scope */
    blx r4                           /* indirect call to r4 */
    pop {r4, r5, r10, lr}            /* restore callee-saved registers */
    bx lr                            /* return */
```
> 총 8개의 명령어로 이루어짐. 4x8 == 32bytes


다음부터 나오는 코드 조각(snippet)은 동일한 파일을 나눠서 설명한 것이다 


```asm
.globl main
main:
    push {r4, r5, r6, fp, lr} /* keep callee saved registers */
    mov fp, sp                /* setup dynamic link */
 
    sub sp, sp, #4            /* counter will be in fp - 4 */
    /* note that now the stack is 8-byte aligned */
 
    /* set counter to zero */
    mov r4, #0        /* r4 ←  0 */
    str r4, [fp, #-4] /* counter ←  r4 */
```
> main 함수 시작 부분


```asm
    /* Make room for the trampoline */
    sub sp, sp, #32 /* sp ←  sp - 32 */
    ^^^^^^^^^^^^^^^
    /* note that 32 is a multiple of 8, so the stack
       is still 8-byte aligned */
```
> 그 뒤 Tempoline 코드를 스택에 넣기 위해 공간 할당한다. Tempoline 은 스택에서 동작하는 code이다.  


```asm
    /* copy the trampoline into the stack */
    mov r4, #32                        /* r4 ←  32 */
    ldr r5, .Laddr_trampoline_template /* r4 ←  &trampoline_template */
    mov r6, sp                         /* r6 ←  sp */
    b .Lcopy_trampoline_loop_check     /* branch to copy_trampoline_loop_check */
 
    .Lcopy_trampoline_loop:
        ldr r7, [r5]     /* r7 ←  *r5 */ // trampoline 코드를 4byte씩 가져옴
        ^^^^^^^^^^^^
        str r7, [r6]     /* *r6 ←  r7 */ // Stack에 4byte씩 저장 
        ^^^^^^^^^^^^
        add r5, r5, #4   /* r5 ←  r5 + 4 */
        add r6, r6, #4   /* r6 ←  r6 + 4 */
        sub r4, r4, #4   /* r4 ←  r4 - 4 */
    .Lcopy_trampoline_loop_check:
        cmp r4, #0                  /* compute r4 - 0 and update cpsr */
        bgt .Lcopy_trampoline_loop  /* if cpsr means that r4 > 0
                                       then branch to copy_trampoline_loop */
```
> 그 뒤 trampoline 을 stack으로 copy  

Setup the trampoline:  

```asm
    /* setup the trampoline */
    ldr r4, addr_of_integer_comparison_count
                                  /* r4 ←  &integer_comparison_count */
    str r4, [fp, #-36]            /* *(fp - 36) ←  r4 */ // 함수포인터
                                  /* set the function_called in the trampoline
                                     to be &integer_comparison_count */
    str fp, [fp, #-32]            /* *(fp - 32) ←  fp */ // Static Link
                                  /* set the lexical_scope in the trampoline to be fp */
```
> .Lfunction_called: .word 0x0 -> `str r4, [fp, #-36]` 
> .Llexical_scope: .word 0x0 -> `str fp, [fp, #-32]`  
> trampoline 코드의 상위 두 라인이다.  
> 위의 예제에서 8개의 trampoline 명령어에서 상위 두라인까지는 어떠한 값으로 설정해야 했다. 다음 코드는 첫번째, 두번째 내용을 설정하는 코드이다.  
> 첫번째 4바이트는 호출할 integer_comparision_count의 함수 포인터이다.  
> 두번째 4바이트는 Static Link이다.  
> 이 코드 수행후 cache flush를 한다. (이 장(24) 맨아래챕터 참고)  


```asm
   /* prepare call to print_array */
    ldr r0, addr_of_my_array /* r0 ←  &my_array */
    mov r1, #10              /* r1 ←  10
                                our array is of length 10 */
    bl print_array           /* call print_array */
 
    /* prepare call to qsort */
    /*
    void qsort(void *base,
         size_t nmemb,
         size_t size,
         int (*compar)(const void *, const void *));
    */
    ldr r0, addr_of_my_array /* r0 ←  &my_array base */
    mov r1, #10              /* r1 ←  10
                                nmemb = number of members
                                our array is 10 elements long */
    mov r2, #4               /* r2 ←  4
                                size of each member is 4 bytes */
    sub r3, fp, #28          /* r3 ←  fp - 28 */
    ^^^^^^^^^^^^^^^          /* 참고로 tempoline의 3번째 바이트: 코드의 시작위치이다. */
    bl qsort                 /* call qsort */
 
    /* prepare call to printf */
    ldr r1, [fp, #-4]                    /* r1 ←  counter */
    ldr r0, addr_of_comparison_message   /* r0 ←  &comparison_message */
    bl printf                            /* call printf */
 
    /* now print again the array to see if elements were sorted */
    /* prepare call to print_array */
    ldr r0, addr_of_my_array  /* r0 ←  &my_array */
    mov r1, #10               /* r1 ←  10
                                 our array is of length 10 */
    bl print_array            /* call print_array */
```
> qsort의 매개변수를 r0 ~ r3 에 저장한 뒤 호출한다.  
> `sub r3, fp, #28` : 즉 함수포인터 integer_comparision_count 를 직접호출 하는것이 아니라, 그 사이에 다른 보조함수 tempoline이 대신 Indirect Call 시켜주는 것이다.  
> fp - 28은 tempoline의 세번째 4바이트 즉,  시작 코드가 위치한 주소이다.  


```asm
    /* nested function integer comparison */
    addr_of_integer_comparison_count : .word integer_comparison_count
    integer_comparison_count:
        /* r0 will be the address to the first integer */
        /* r1 will be the address to the second integer */
        push {r4, r5, r10, fp, lr} /* keep callee-saved registers */
        mov fp, sp                 /* setup dynamic link */
 
        ldr r0, [r0]    /* r0 ←  *r0
                           load the integer pointed by r0 in r0 */
        ldr r1, [r1]    /* r1 ←  *r1
                           load the integer pointed by r1 in r1 */
 
        cmp r0, r1      /* compute r0 - r1 and update cpsr */
        moveq r0, #0    /* if cpsr means that r0 == r1 then r0 ←   0 */
        movlt r0, #-1   /* if cpsr means that r0 <  r1 then r0 ←  -1 */
        movgt r0, #1    /* if cpsr means that r0 >  r1 then r0 ←   1 */
 
        ldr r4, [fp, #8]  /* r4 ←  *(fp + 8)
        ^^^^^^^^^^^^^^^^^  get static link in the stack */
        ldr r5, [r4, #-4] /* r5 ←  counter
        ^^^^^^^^^^^^^^^^^  main 함수의 로컬변수를 얻어옴!
                             get value of counter */
        add r5, r5, #1    /* r5 ←  r5 + 1 */
        str r5, [r4, #-4] /* counter ←  r5
                             update counter */
 
        mov sp, fp        /* restore stack */
        pop {r4, r5, r10, fp, lr} /* restore callee-saved registers */
        bx lr           /* return */
```
> 그 Nested function은 위와 같다.
>  *compar의 함수포인터는 이 함수를 가리킨다. tempoline은 이함수를 indirect call해준다. 핵심은 이 함수가 함수포인터를 통해 indirect call 된 nested function임에도 불구하고 enclosing function인 main: 의 로컬변수를 Access 할 수 있다는 점이다. 이것이 이 장의 핵심이다.   

이렇게 Indirect call된 nested function이 static link를 정확하게 가져고 있음을 확인할수 있다. 이것이 trampoline의 하는 일이다. 

## Self Modifying code 와 Cache

사실 위의 예제는 일종의 Self Modifying code 이다. 

Raspberry PI의 프로세서는 modified harvard architecture인데, 이것은 instruction memory(.text) 와 data memory(.data)를 구분해서 동작한다. 이것은 서로다른 cache가 필요함을 의미한다. Instruction Cache 와 Data Cache 이다.   

명령어 캐시는 data캐시의 값을 얻어올수 없다.  명령어 캐시는 데이터를 main memory에서밖에 얻어올수 없기 때문에 data 캐시에 기록한 모든 수정사항을 main memory에 update  해야한다. 이것을 Flushing the Cache 라고 한다.  
(Conversely, since the instruction cache will only get data from the main memory (and not from the data cache), we need to write back all the changes we did in the data cache to the main memory (this is called flushing the cache))  

> Write Back: Data를 쓸 때 Memory에는 안 쓰고, Cache만 update하는 방법

```
Data --------------> Data cache --------------> memory
        write back                    flush
```

다시말하면, 캐시 write back을하면 data cache에만 쓰고 실제 메모리에는 write하지 않는데, 이때 memory와 cache에 inconsistancy가 발생한다. 그래서 메모리에 write해줘야 하는데 이것을 여기선 Flushing the cache라고 부른다. (어떤 자료는 이것을 cache clean이라고 부르던데 뭐가 맞는지 확인필요 : http://recipes.egloos.com/5170809)

ARM 명령어는 이 Flushing the Cache 동작은 privileged operation이다. 이것은 coprocessor 명령어 coprocessor 15 를 이용한다. 이 뜻은 운영체제가 이 명령어를 사용할 수 있다는 뜻이다. 리눅스 커널은 `cacheflush` 라는 시스템콜을 제공한다.  

19장에서 살펴봤던것 처럼 시스템콜은 `swi`명령을 사용한다. r0는 flush해야할 영역의 시작주소가 있어야 하고, r1은  must contain the address of the first byte that will not be invalidated. r2는 0이어야 하고, 시스템콜의 구분 코드 r7은 0xF0002 이다.   

```asm
    push {r7}          /* keep r7 because we are going to modify it */
    mov r7, #0xf0000   /* r7 ←  0xf0000 */
    add r7, r7, #2     /* r7 ←  r7 + 2. So r7 ←  0xf0002
                          We do this in two steps because
                          we cannot encode 0xf0002 in
                          the instruction */
    mov r0, sp         /* r0 ←  sp */
    add r1, sp, #32    /* r1 ←  sp + 32 */
    mov r2, #0         /* r2 ←  0 */
    swi 0              /* system call */
    ^^^^^
    pop  {r7}          /* restore r7 */
```

libgcc에는 __clear_cache 함수가 있기도 하다.  

```asm
    /* prepare call to __clear_cache */
    mov r0, sp       /* r0 ←  sp */
    add r1, sp, #32  /* r1 ←  sp + 32 */
    bl __clear_cache /* call __clear_cache */
```

이 code를 tempoline 을 설정한뒤에 바로 수행해야한다. 

전체 코드는 [Full source](https://github.com/rofirrim/raspberry-pi-assembler/blob/master/chapter24/trampoline-sort-array.s) 에서 확인 가능하다.  

## 한계

대부분의 modern 운영체제는 메모리 영역을 readable/writable/executable로 구분하는데 stack 이나 .data 는 보통 executable 하지않다. 오직 .text 영역만 executable 하다. 우리는 executable stack을 사용했기 때문에 이번장의 예제가 가능했던 것이다. 그러나 executable stack 은 buffer overflows같은 해킹등에 취약하다.

23장 24장에서 살펴본것 처럼 Nested function은 여러 단점이 존재한다. 그래서 많은 프로그램 언어가 이 기능을 지원하지 않는 배경일것이다.  








<div style="page-break-after: always;"></div>

# 25. Integer SIMD
----

13장에서 VFPv2 와 floating point number를 vector operation 할 수 있음을 확인했다. integer number도 벡터연산이 가능한가? 이장에서는 그것을 확인한다.  

## SIMD

SIMD는 문자 그대로 __S__ingle __I__nstruction 으로 한번에 __M__ultible __D__ata를 처리한다는 의미이다.  
#13 #14장에서 

`참고: NEON`

Arm NEON technology is an __advanced SIMD__ (single instruction multiple data) architecture extension for the Arm Cortex-A series and Cortex-R52 processors.  
NEON technology was introduced to the __Armv7-A__ and __Armv7-R__ profiles. It is also now an extension to the __Armv8-A__ and __Armv8-R__ profiles.  
NEON technology is intended to improve the __multimedia__ user experience by accelerating __audio__ and __video encoding/decoding__, user interface, __2D/3D graphics or gaming__. NEON can also accelerate __signal processing algorithms__ and functions to speed up applications such as audio and video processing, voice and facial __recognition__, computer __vision__ and __deep learning__.  


## Motivating example

이러한 기능이 왜 필요한지 다음의 예제로 살펴보자. 먼저 16bit PCM audio signal 이 샘플링 되어있다고 하자. 그리고 다음의 코드처럼 그 중 두개의 signal의 평균을 구한다고 해보자.  

```c
short int channel1[num_samples]; // in our environment a 'short int' is a half-word
short int channel2[num_samples];
 
short int channel_out[num_samples];
for (i = 0; i < num_samples; i++)
{
   channel_out[i] = (channel1[i] + channel2[i]) / 2;
}
```
> short int 즉 2byte 사용 예제임을 주의  

그리고 assembly 명령어로는 다음과 같이 번역될 수 있다.  

```asm
naive_channel_mixing:
    /* r0 contains the base address of channel1 */
    /* r1 contains the base address of channel2 */
    /* r2 contains the base address of channel_out */
    /* r3 is the number of samples */
    /* r4 is the number of the current sample
          so it holds that 0 ←  r4 < r3 */
 
    mov r4, #0              /* r4 ←  0 */
    b .Lcheck_loop          /* branch to check_loop */
    .Lloop:
      mov r5, r4, LSL #1    /* r5 ←  r4 << 1 (this is r5 ←  r4 * 2) */
                            /* a halfword takes two bytes, so multiply
                               the index by two. We do this here because
                               ldrsh does not allow an addressing mode
                               like [r0, r5, LSL #1] */
      ldrsh r6, [r0, r5]    /* r6 ←  *{signed half}(r0 + r5) */
      ldrsh r7, [r1, r5]    /* r7 ←  *{signed half}(r1 + r5) */
      add r8, r6, r7        /* r8 ←  r6 + r7 */
      mov r8, r8, ASR #1    /* r8 ←  r8 >> 1 (this is r8 ←  r8 / 2)*/
      strh r8, [r2, r5]     /* *{half}(r2 + r5) ←  r8 */
      add r4, r4, #1        /* r4 ←  r4 + 1 */
    .Lcheck_loop: 
      cmp r4, r3            /* compute r4 - r3 and update cpsr */
      blt .Lloop            /* if r4 < r3 jump to the
                               beginning of the loop */
```
> 이 코드는 우리가 익숙해서 좋지만, 다음 장 처럼 병렬연산으로 개선 해야한다.    
> 참고, `sh`? signed half, 2byte를 사용한다.  [21장 Subword data참고](#21. Subword data)    


## 병렬 연산 (더하기, 빼기)

통째로 더하거나 빼기 위해서는 아래와 같은 명령을 사용한다.  

- Halfwords
	* Signed: sadd16, ssub16
	* Unsigned: uadd16, usub16
- Bytes
	* Signed: sadd8, ssub8
	* Unsigned: uadd8, usub8

다음의 C 코드가 있다고 하자.  

```c
// unsigned char is an unsigned byte in our environment
// a, b and c are arrays of N unsigned chars
unsigned char a[N], b[N], c[N];
 
int i;
for (i = 0; i < N; i++)
{
    c[i] = a[i] + b[i];
}
```

이것을 순수하게 loop를 이용해 더하면 아래와 같다.  

```asm
naive_byte_array_addition:
    /* r0 contains the base address of a */
    /* r1 contains the base address of b */
    /* r2 contains the base address of c */
    /* r3 is N */
    /* r4 is the number of the current item
          so it holds that 0 ←  r4 < r3 */
 
    mov r4, #0             /* r4 ←  0 */
    b .Lcheck_loop0        /* branch to check_loop0 */
 
    .Lloop0:
      ldrb r5, [r0, r4]    /* r5 ←  *{unsigned byte}(r0 + r4) */
      ldrb r6, [r1, r4]    /* r6 ←  *{unsigned byte}(r1 + r4) */
      add r7, r5, r6       /* r7 ←  r5 + r6 */
      strb r7, [r2, r4]    /* *{unsigned byte}(r2 + r4) ←  r7 */
      add r4, r4, #1       /* r4 ←  r4 + 1 */
                 ^^^^
    .Lcheck_loop0:
       cmp r4, r3          /* perform r4 - r3 and update cpsr */
       blt .Lloop0         /* if cpsr means that r4 < r3 jump to loop0 */
```

sadd8 을 이용하기

```asm
simd_byte_array_addition_0:
    /* r0 contains the base address of a */
    /* r1 contains the base address of b */
    /* r2 contains the base address of c */
    /* r3 is N */
    /* r4 is the number of the current item
          so it holds that 0 ←  r4 < r3 */
 
    mov r4, #0             /* r4 ←  0 */
    b .Lcheck_loop1        /* branch to check_loop1 */
 
    .Lloop1:
      ldr r5, [r0, r4]     /* r5 ←  *(r0 + r4) */
      ldr r6, [r1, r4]     /* r6 ←  *(r1 + r4) */
      sadd8 r7, r5, r6     /* r7[7:0] ←  r5[7:0] + r6[7:0] */
      ^^^^^^^^^^^^^^^^     /* r7[15:8] ←  r5[15:8] + r6[15:8] */
                           /* r7[23:16] ←  r5[23:16] + r6[23:16] */
                           /* r7[31:24] ←  r5[31:24] + r6[31:24] */
                           /* rA[x:y] means bits x to y of the register rA */
      str r7, [r2, r4]     /* *(r2 + r4) ←  r7 */
      add r4, r4, #4       /* r4 ←  r4 + 4 */
                 ^^^^
    .Lcheck_loop1:
       cmp r4, r3          /* perform r4 - r3 and update cpsr */
       blt .Lloop1         /* if cpsr means that r4 < r3 jump to loop1 */
```
> sadd8 을 이용하면 4byte씩 한번에 더한다.  

하지만 이 예제는 N이 4의 배수일때만 사용된다. N이 4의 배수가 아닌경우는 다음과 같이한다. 

```asm
simd_byte_array_addition_2:
    /* r0 contains the base address of a */
    /* r1 contains the base address of b */
    /* r2 contains the base address of c */
    /* r3 is N */
    /* r4 is the number of the current item
          so it holds that 0 ←  r4 < r3 */
 
    mov r4, #0             /* r4 ←  0 */
    sub r8, r3, #3         /* r8 ←  r3 - 3
    ^^^^^^^^^^^^^^            this is r8 ←  N - 3 */
    b .Lcheck_loop2        /* branch to check_loop2 */
 
    .Lloop2:
      ldr r5, [r0, r4]     /* r5 ←  *(r0 + r4) */
      ldr r6, [r1, r4]     /* r6 ←  *(r1 + r4) */
      sadd8 r7, r5, r6     /* r7[7:0] ←  r5[7:0] + r6[7:0] */
                           /* r7[15:8] ←  r5[15:8] + r6[15:8] */
                           /* r7[23:16] ←  r5[23:16] + r6[23:16] */
                           /* r7[31:24] ←  r5[31:24] + r6[31:24] */
      str r7, [r2, r4]     /* *(r2 + r4) ←  r7 */
      add r4, r4, #4       /* r4 ←  r4 + 4 */
    .Lcheck_loop2:
       cmp r4, r8          /* perform r4 - r8 and update cpsr */
       blt .Lloop2         /* if cpsr means that r4 < r8 jump to loop2 */
                           /* i.e. if r4 < N - 3 jump to loop2 */
```

N - 3으로 비교한뒤 나머지 값들은 다음 코드에서 byte단위로 더해서 마무리한다.  

```asm
     /* epilog loop */
     b .Lcheck_loop3       /* branch to check_loop3 */
 
     .Lloop3:
        ldrb r5, [r0, r4]  /* r5 ←  *{unsigned byte}(r0 + r4) */
        ldrb r6, [r1, r4]  /* r6 ←  *{unsigned byte}(r1 + r4) */
        add r7, r5, r6     /* r7 ←  r5 + r6 */
        strb r7, [r2, r4]  /* *{unsigned byte}(r2 + r4) ←  r7 */
 
        add r4, r4, #1     /* r4 ←  r4 + 1 */
     .Lcheck_loop3:
        cmp r4, r3         /* perform r4 - r3 and update cpsr */
        blt .Lloop3        /* if cpsr means that r4 < r3 jump to loop 3 */
```

## Halving instructions

half word 에 대해서도 다음의 연산이 존재한다.  

- Halfwords
	* Signed: shadd16, shsub16
	* Unsigned: uhadd16, uhsub16
- Bytes
	* Signed: shadd8, shsub8
	* Unsigned: uhadd8, uhsub8

위의 motivation example은 다음과 같이 번역될 수 있다. 이 방식은 loop종료 이후 추가 연산이 필요없다.   

```asm
better_channel_mixing:
    /* r0 contains the base address of channel1 */
    /* r1 contains the base address of channel2 */
    /* r2 contains the base address of channel_out */
    /* r3 is the number of samples */
    /* r4 is the number of the current sample
          so it holds that 0 ←  r4 < r3 */
 
    mov r4, #0              /* r4 ←  0 */
    b .Lcheck_loop          /* branch to check_loop */
    .Lloop:
      ldr r6, [r0, r4]      /* r6 ←  *(r0 + r4) */
      ldr r7, [r1, r4]      /* r7 ←  *(r1 + r4) */
      shadd16 r8, r6, r7    /* r8[15:0] ←  (r6[15:0] + r7[15:0]) >> 1*/
                            /* r8[31:16] ←  (r6[31:16] + r7[31:16]) >> 1*/
      str r8, [r2, r4]      /* *(r2 + r4) ←  r8 */
      add r4, r4, #2        /* r4 ←  r4 + 2 */
    .Lcheck_loop:
      cmp r4, r3            /* compute r4 - r3 and update cpsr */
      blt .Lloop            /* if r4 < r3 jump to the
                               beginning of the loop */
```


## Saturating arithmetic

만약 2byte에서 더한값이 overflow된다면? 그것을 확인하는 동작이 필요하다. 아래와 같은 함수를 이용할 수 있다.  


```asm
.data
max16bit: .word 32767
 
.text
 
clipped_add16bit:
    /* first operand is in r0 */
    /* second operand is in r1 */
    /* result is left in r0 */
    push {r4, lr}             /* keep registers */
 
    ldr r4, addr_of_max16bit  /* r4 ←  &max16bit */
    ldr r4, [r4]              /* r4 ←  *r4 */
                              /* now r4 == 32767 (i.e. 2^15 - 1) */
 
    add r0, r0, r1            /* r0 ←  r0 + r1 */
    cmp r0, r4                /* perform r0 - r4 and update cpsr */
    movgt r0, r4              /* if r0 > r4 then r0 ←  r4 */
    bgt end                   /* if r0 > r4 then branch to end */
 
    mvn r4, r4                /* r4 ←  ~r4
                                 now r4 == -32768 (i.e. -2^15) */
    cmp r0, r4                /* perform r0 - r4 and update cpsr */
    movlt r0, r4              /* if r0 < r4 then r0 ←  r4 */
 
    end:
 
    pop {r4, lr}              /* restore registers */
    bx lr                     /* return */
addr_of_max16bit: .word max16bit
```
> 기존의 명령어로 처리하기 위해서는 상당히 많은 라인이 필요하지만 다행이도 Saturated arithmetics 명령어가 이를 해결해준다.  

- Halfwords
	* Signed: qadd16, qsub16
	* Unsigned: uqadd16, uqsub16
- Bytes
	* Signed: qadd8, qsub8
	* Unsigned: uqadd8, uqsub8

```asm
more_realistic_channel_mixing:
    /* r0 contains the base address of channel1 */
    /* r1 contains the base address of channel2 */
    /* r2 contains the base address of channel_out */
    /* r3 is the number of samples */
    /* r4 is the number of the current sample
          so it holds that 0 ←  r4 < r3 */
 
    mov r4, #0              /* r4 ←  0 */
    b .Lcheck_loop          /* branch to check_loop */
    .Lloop:
      ldr r6, [r0, r4]      /* r6 ←  *(r0 + r4) */
      ldr r7, [r1, r4]      /* r7 ←  *(r1 + r4) */
      qadd16 r8, r6, r7     /* r8[15:0] ←  saturated_sum_16(r6[15:0], r7[15:0]) */
                            /* r8[31:16] ←  saturated_sum_16(r6[31:16], r7[31:16]) */
      str r8, [r2, r4]      /* *(r2 + r4) ←  r8 */
      add r4, r4, #2        /* r4 ←  r4 + 2 */
    .Lcheck_loop:
      cmp r4, r3            /* compute r4 - r3 and update cpsr */
      blt .Lloop            /* if r4 < r3 jump to the
                               beginning of the loop */
```



<div style="page-break-after: always;"></div>

# 26. A primer about linking
----

`NOTE`: #26, #27장에 걸쳐 소개되는 Linking 은 온전히 이해하지 못한부분이 많다. 앞으로도 관심갖고 지속적으로 수정 보완 할 예정.  

## GCC 와 Keil assembler 비교

이번장 Linker에 대해 소개하기 전에 먼저 짚고 넘어갈 사항이다. 사실 지금까지 배운 어셈블리 코드 구조는 gcc 기반이다. 반면 ARM에는 keil 이라는 어셈블러도 존재하는데 코드 골격이 약간 달라서 소개한다. 아래 두 코드는 syntax만 다를뿐 동일한 동작이다.

- Keil MDK-ARM syntax

	```asm
	; ARM Assembly Language Program To Add Some Data and Store the SUM in R3.

	       AREA PROG_2_1, CODE, READONLY
	       ENTRY
	       MOV R1, #0x25   ; R1 = 0x25 
	       MOV R2, #0x34   ; R2 = 0x34 
	       ADD R3, R2, R1  ; R3 = R2 + R1 
	HERE   B HERE          ; stay here forever 
	       END
	```
- GCC v2.24 syntax

	```asm
	@P2_1.s ARM Assembly Language Program To Add Some Data and Store the SUM in R3.

	         .global _start
	_start:  MOV R1, #0x25      @ R1 = 0x25
	         MOV R2, #0x34      @ R2 = 0x34
	         ADD R3, R2, R1     @ R3 = R2 + R1
	HERE:    B HERE             @ stay here forever
	```
그리고 GCC 와 Keil의 지시자 syntax가 어떻게 다른지는 아래와 같다.  

- 표: Assembler 지시자 의미: GCC 와 Keil assembler 비교

	| GCC | Directive | Keil MDK-ARM Directive Explanation |
	|-----|-----------|------------------------------------|
	| .text |TEXT| Signifies the beginning of code or constant|
	| .data |DATA |Signifies the beginning of read/write data |
	| .global label |GLOBAL or EXPORT |Makes the label visible to linker |
	| .extern |label EXTERN |label is declared outside of this file |
	| .byte byte [,byte, byte, ..] |DCB |Declares byte (8-bit) data |
	| .hword hw [,hw, hw, ..] |DCW |Declares halfword (16-bit) data |
	| .word word [,word, word, ..] |DCD |Declares word (32-bit) data |
	| .float float [,float, float, ..] |DCFS |Declares single precision floating point (32-bit) data |
	| .double double [,double, double, ..] |DCFD| Declares double precision floating point (64-bit) data |
	| .space #bytes [,fill] |SPACE or FILL |Declares memory (in bytes) with optional fill |
	| .align n |ALIGN | Aligns address to <2 to the power of n> byte |
	| .ascii "ASCII string" | DCB "ASCII string"  |Declares an ASCII string |
	| .asciz "ASCII string" | DCB "ASCII string", 0| Declares an ASCII string with null termination |
	| .equ symbol, value |EQU |Sets the symbol with a constant value |
	| .set variable, value |SETA |Sets the variable with a new value |
	| .end |END| Signifies the end of the program |


## Linkers, the magic between symbols and addresses

링커는 주목받지 못하는 tool이지만 프로그램을 만드는데에 핵심이다.   
링커의 주된 동작은 모든 코드 조각들을 합치는 것이다. 컴퓨터가 실제로 실행 가능하도록 말이다. 보다 정확히 말하면, Symbolic Name을 Address와 Binding 한다. 프로그램은 여러 파일로 작성할 수 있고 각각 object파일로 컴파일 된다. 그 여러개의 Object파일을 하나의 실행파일로 합치는것이 Linker가 하는 일이다.  

## ELF


여러 실행파일을 공통의 포맷으로 관리하면 좋을 것이다. 이런 포멧은 COFF, MACH-O, ELF등이 있는데 UNIX에서는 ELF를 주로 사용한다. ELF포맷은 Object 파일(relocatable), Shared Object, Executable 로 사용된다.   

Linker에게 ELF relocatable 파일은 섹션들의 모음집 이다. 섹션은 연속된 데이터들을 말한다. 이것은 instruction의 연속된 모음이 될 수도 있고, 초기화된 전역변수, 디버깅 정보등 어떤것도 될 수 있다. 각각의 섹션은 이름과 속성이 있다. 가령 속성은 다음과같은 것들이 될 수 있다. 메모리에 할당될 것인지, 이미지로부터 로딩될 것인지, 실행될 것인지, writable한지, 사이즈와, 메모리에 어떻게 Alignment 할건지 등이다.  


## Labels as symbolic names

만약 전역변수를 사용하다면 다음과 같은 구조를 사용한다. 

```asm
.data:
var: .word 42
.text
func:
    /* ... */
    ldr r0, addr_of_var  /* r0 ←  &var */
    ldr r0, [r0]         /* r0 ←  *r0 */
    /* ... */
addr_of_var : .word var
```

Assembler는 label을 다음과 같은 방식으로 PC + offset 으로 변경한다. 
이 Addressing mode는 이 offset을 12bit로 encode 할수 있다. 결국 4096byte의 offset을 가질 수 있다. 

```asm
ldr r0, [pc #offset]
```

질문, 어셈블러는 addr_of_var 라벨 위치에 어떤명령을 번역해서 놓는가? 우리는 .word var를 적었다. 그러나 이게 도대체 무슨 의미인가? 이시점에서는 var의 주소를 알수가 없다. 그래서 어셈블러는 pc + offset 이라는 일부의 정보(partial information)만 기록할 수 있는 것이다. 



## An example

두개의 전역변수가 있고, 그것을 더할 것이다. 그 뒤에 다른 파일에 작성된 함수를 호출 할 것이다. 그 함수는 결과값+1 을할것이다. 


```asm
/* main.s */
.data
 
one_var : .word 42
another_var : .word 66
 
.globl result_var             /* mark result_var as global */
^^^^^^^^^^^^^^^^^               ^^^^^^^^^^^^^^^^^^^^^^^^^^^
result_var : .word 0
 
.text
 
.globl main
^^^^^^
main:
    ldr r0, addr_one_var      /* r0 ←  &one_var */
    ldr r0, [r0]              /* r0 ←  *r0 */
    ldr r1, addr_another_var  /* r1 ←  &another_var */
    ldr r1, [r1]              /* r1 ←  *r1 */
    add r0, r0, r1            /* r0 ←  r0 + r1 */
    ldr r1, addr_result       /* r1 ←  &result */
    str r0, [r1]              /* *r1 ←  r0 */
    bl inc_result             /* call to inc_result */
    ^^^^^^^^^^^^^             /* 다른파일에 있는 함수 */
    mov r0, #0                /* r0 ←  0 */
    bx lr                     /* return */
 
addr_one_var  : .word one_var
addr_another_var  : .word another_var
addr_result  : .word result_var
```
> .globl: make the lebel visible to Linker

이 코드를 object파일로 만든뒤, objdump -d 로 어떻게 만들어졌는지 살펴보자. 

```sh
 $ as -march=armv6 -o main.o main.s      # creates object file main.o
```

## Relocations

위에서 언급했던 것 처럼 어셈블러는 final value가 무엇인지 모른다. 그 대신에 일부의 정보만을 남겨놓을수 있을 뿐이다. 이를테면 .data 기준으로 offset 이 어느정도 되는지 이런식이다. 결국 어떤 수정이 필요함을 의미한다. 그것이 바로 Relocation이다. 

```sh
$ objdump -dr main.o
```
> `-dr`: a option for reading relocations  

```txt
main.o:     file format elf32-littlearm
 
Disassembly of section .text:
 
00000000 <main>:
   0:	e59f0020 	ldr	r0, [pc, #32]	; 28 <addr_one_var>
   4:	e5900000 	ldr	r0, [r0]
   8:	e59f101c 	ldr	r1, [pc, #28]	; 2c <addr_another_var>
   c:	e5911000 	ldr	r1, [r1]
  10:	e0800001 	add	r0, r0, r1
  14:	e59f1014 	ldr	r1, [pc, #20]	; 30 <addr_result>
  18:	e5810000 	str	r0, [r1]
  1c:	ebfffffe 	bl	0 <inc_result>
			1c: R_ARM_CALL	inc_result
			^^^^^^^^^^^^^^^^^^^^^^^^^^
  20:	e3a00000 	mov	r0, #0
  24:	e12fff1e 	bx	lr
 
00000028 <addr_one_var>:
  28:	00000000 	.word	0x00000000
                        28: R_ARM_ABS32	.data
                        ^^^^^^^^^^^^^^^^^^^^^
 
0000002c <addr_another_var>:
  2c:	00000004 	.word	0x00000004
                        2c: R_ARM_ABS32	.data
                        ^^^^^^^^^^^^^^^^^^^^^
 
00000030 <addr_result>:
  30:	00000000 	.word	0x00000000
                        30: R_ARM_ABS32	result_var
                        ^^^^^^^^^^^^^^^^^^^^^^^^^^
```

`30: R_ARM_ABS32	result_var` 이런 부분이 relocation 정보이다.   
relocation 정보는 다음과 같은 구성을 따른다.  
```
OFFSET: TYPE   VALUE
```

OFFSET : 섹션으로부터 offset을 의미  
TYPE : relocation의 종류이다.  
VALUE : 주소의 symbolic entry 이다.  

위의 예제에는 다음과 같은 relocation 정보가 있다.  

.text+1c so we can call the actual inc_result.
.text+28, .text+2c are the relocations required to access .data.
.text+30 refers to the global symbol result_var.

모든 relocation은 몇 파라미터가 있다. 
S: is the address of the symbol referred by the relocation (the VALUE above)
P: is the address of the place (the OFFSET plus the address of the section itself),
A: (부가정보) is the value that the assembler has left in place.

예를 들어,
`R_ARM_ABS32` it is the value of the .word  
	R_ARM_ABS32의 relocation은 `S + A` 동작을 수행한다.   
`R_ARM_CALL` it is a set of bits in the bl instruction itself.  
	R_ARM_CALL의 relocation은 `(S + A) - P` 동작을 수행한다.   

Linker가 이것들을 계산하기 전에 inc_result 함수를 정의하지 않으면 linking은 실패 할 것이다. 

```asm
/* inc_result.s */ 다른 파일주의
.text
 
.globl inc_result
^^^^^^^^^^^^^^^^^
inc_result:
    ldr r1, addr_result  /* r1 ←  &result */
    ldr r0, [r1]         /* r0 ←  *r1 */
    add r0, r0, #1       /* r0 ←  r0 + 1 */
    str r0, [r1]         /* *r1 ←  r0 */
    bx lr                /* return */
 
addr_result  : .word result_var
                     ^^^^^^^^^^
```
> .globl inc_result 했기 때문에 링커가 이 symbol을 알수 있다.  
> result_var는 다른파일에서 .globl로 선언되었기 때문에 이 파일에서 링커가 그 symbol을 알수 있다.  


```bash
 $ as -march=armv6 -o inc_result.o inc_result.s
 $ objdump -dr inc_result.o
```
> as를 통해 obj파일 생성

```bash
inc_result.o:     file format elf32-littlearm
 
Disassembly of section .text:
 
00000000 <inc_result>:
   0:	e59f100c 	ldr	r1, [pc, #12]	; 14 <addr_result>
   4:	e5910000 	ldr	r0, [r1]
   8:	e2800001 	add	r0, r0, #1
   c:	e5810000 	str	r0, [r1]
  10:	e12fff1e 	bx	lr
 
00000014 <addr_result>:
  14:	00000000 	.word	0x00000000
			14: R_ARM_ABS32	result_var
```
> result_val 에 relocation 정보가 생성되었다.  

이제 두 파일을 합쳐보자.  

```bash
 $ gcc -o test.exe print_float.o reloc.o
```
몇가지 C 라이브러리를 포함할것이지만 여기서는 무시할 예정이다.   

```bash
 $ objdump -d test.exe
```

```bash
...
00008390 <main>:
    8390:       e59f0020        ldr     r0, [pc, #32]   ; 83b8 <addr_one_var>
    8394:       e5900000        ldr     r0, [r0]
    8398:       e59f101c        ldr     r1, [pc, #28]   ; 83bc <addr_another_var>
    839c:       e5911000        ldr     r1, [r1]
    83a0:       e0800001        add     r0, r0, r1
    83a4:       e59f1014        ldr     r1, [pc, #20]   ; 83c0 <addr_result>
    83a8:       e5810000        str     r0, [r1]
    83ac:       eb000004        bl      83c4            ; <inc_result>
                                        ^^^^^^^^^^^^^^^^
    83b0:       e3a00000        mov     r0, #0
    83b4:       e12fff1e        bx      lr
 
000083b8 <addr_one_var>:
    83b8:       00010578        .word   0x00010578
    ^^^^^       
    주소
 
000083bc <addr_another_var>:
    83bc:       0001057c        .word   0x0001057c
 
000083c0 <addr_result>:
    83c0:       00010580        .word   0x00010580
 
000083c4 <inc_result>:
    83c4:       e59f100c        ldr     r1, [pc, #12]   ; 83d8 <addr_result>
    83c8:       e5910000        ldr     r0, [r1]
    83cc:       e2800001        add     r0, r0, #1
    83d0:       e5810000        str     r0, [r1]
    83d4:       e12fff1e        bx      lr
 
000083d8 <addr_result>:
    83d8:       00010580        .word   0x00010580
 
...
```
> addr_one_var이 가지고 있는 값 00010578 은 말그대로 one_var의 주소값이다.  
> `ldr     r0, [pc, #32]`: 이 동작이 [00010578] 즉 주소가 가리키는 값을 얻어온다.  


자 그럼 `addr_one_var`, `addr_another_var`, `addr_result` 의 실제 relocate된 주소를 계산해보자.  
이 세가지의 relocation 정보는  `R_ARM_ABS32` 이었다.   
위에서 설명한대로 R_ARM_ABS32는 `S + A` 라는 동작을 수행한다. 여기서 S 는 .data 섹션이다. A는 생략가능하다.  
.data 섹션의 주소는 아래와 같이 `objdump -h` (-w: more readable) 로 얻어올 수 있다.  


```bash
 $ objdump -hw test.exe

test.exe:     file format elf32-littlearm
 
Sections:
Idx Name          Size      VMA       LMA       File off  Algn  Flags
...
 13 .text         0000015c  000082e4  000082e4  000002e4  2**2  CONTENTS, ALLOC, LOAD, READONLY, CODE
...
 23 .data         00000014  00010570  00010570  00000570  2**2  CONTENTS, ALLOC, LOAD, DATA
    ^^^^^                   ^^^^^^^^
...
```
> VMA 이 섹션의 주소를 의미한다. 따라서 .data섹션의 주소는 00010570이다.  
그리고 우리가 계산할 세 데이터의 주소는 0x00010578, 0x0001057c, 0x00010580 이었다. 따라서 .data로부터 각각의 offset은 0x8, 0xC, 0x10이다.  
Linker가 바로 이위치에 one_var, another_var, result_var 의 값을 저장하게 되는것이다!  

하지만 linker는 우리가 값을 쓰기전에 우선 어떠한 값들을 해당 위치에 놓는다. 다음의 명령으로 그 값들을 확인할 수 있다.  


```bash
 $ gcc -o test.exe main.o inc_result.o -Wl,--print-map > map.txt
 $ cat map.txt

.data           0x00010570       0x14
                0x00010570                PROVIDE (__data_start, .)
 *(.data .data.* .gnu.linkonce.d.*)
 .data          0x00010570        0x4 /usr/lib/gcc/arm-linux-gnueabihf/4.6/../../../arm-linux-gnueabihf/crt1.o
                0x00010570                data_start
                0x00010570                __data_start
 .data          0x00010574        0x0 /usr/lib/gcc/arm-linux-gnueabihf/4.6/../../../arm-linux-gnueabihf/crti.o
 .data          0x00010574        0x4 /usr/lib/gcc/arm-linux-gnueabihf/4.6/crtbegin.o
                0x00010574                __dso_handle
 .data          0x00010578        0xc main.o
                0x00010580                result_var
 .data          0x00010584        0x0 inc_result.o
 .data          0x00010584        0x0 /usr/lib/arm-linux-gnueabihf/libc_nonshared.a(elf-init.oS)
 .data          0x00010584        0x0 /usr/lib/gcc/arm-linux-gnueabihf/4.6/crtend.o
 .data          0x00010584        0x0 /usr/lib/gcc/arm-linux-gnueabihf/4.6/../../../arm-linux-gnueabihf/cr
```

> result_var 만 보이는 이유는 .globl 심볼 때문이다. 다른 변수는 global 심볼로 선언되어있지 않아보이지 않는다. 어쨋든. one_var 변수는  0x00010570 주소값에 , another_var 은 0x00010574,  result_var 은 0x00010578에 위치해있다.  




<div style="page-break-after: always;"></div>

# 27. dynamic-linking  
----

dynamic-linking이란?  프로그램이 수행중(run)인 상태에서 obj Linking 하는것.  


26장에서 살펴봤던것 처럼 linking process 단계에서 몇 obj를 합쳐서 최종 하나의 binary파일을 만드는 것이다. 그러나 대개에 많은 프로그램들은 재 사용가능한 작은 단위로 존재한다. 이런것들의 집합을 Library라고 부른다.  UNIX는 전통적으로 이것을 Archives 라고 하고 일반적으로는  __Static Library__ 라고 부른다.  

## Dynamic linking

linking time에 프로그램 조각들을 합치는 것이아니라, running time에 합친다면 어떨까? __Dynamic Library__를 사용하면 동적링킹(Dynamic linking)이 가능하다. 

Dynamic linking을 사용하려면 기존의 linking보다 약간의 노력이 더 필요하다. 먼저 loading 에 대해 알아보자.  


## Loading a program

Loading이란? 실행가능한(executable) 프로그램을 메모리에 위치시키는것을 말한다. 운영체제의 목적 중 하나가 이렇게 프로그램 loading 시키는 것이다.    

앞장에서 살펴본 another_var result_var그리고 함수 inc_result는 Linking 프로세스를 거친뒤에 hard-coded 된 주소값을 갖게 된다. loader의 동작은 간단하다. 이 executable binary를 메모리에 copy 하는 것이다. Linking 단계에서 이미 주소값이 수정되었기 때문에, 정확한 메모리 주소에 copy된다. 

현대의 운영체제는 virtual memory 주소 체계를 사용한다. 그래서 각 프로세스는 일관성있는 메모리 주소값을 접근하면 된다. 그래서 로딩되는 프로그램은 동일한 virtual address에 로딩된다. 운영체제는 각각의 running program(process) 가 가지는 virtual memory를 알아서 실제 물리주소에 mapping한다.  

Dynamic Library를 run time에 Linking하기 위해서는 또다른 프로그램 __Dynamic Linker__ 가 필요하다. Dynamic Linker는 Dynamic Library의 코드 데이터를 로딩하고 relocation 작업을 수행한다. relocation작업은 position이 독립적인지 종속적인지에 따라 차이가 있다.


#### Position independent code (PIC)

Code는 position dependent 또는 position independent 할 수 있다. 전자는 absolute address를 직접 사용한다. 후자는 상대 주소를 사용한다. 실행중에 이 상대 주소가 convert 된다. 이것은 절대주소에서 추가 computation이 필요하다는 의미다. ELF에서 이 메커니즘은 Global Offset Table(GOT)을 사용한다. 
모든 프로그램과 Dynamic 라이브러리는 각자가 고유한 GOT를 가지고 있다. 이것을 다른곳과 공유하지도 않는다. GOT는 절대주소를 사용하지 않고도 접근할 수 있는 메모리에 위치해 있다. 이를 위해서 PC + 상대offset 방식의 주소체계가 반드시 사용되어야 한다. 따라서 GOT는 어떤 고정된 공간에 위치해 있고 정적 link time에 계산될 수 있다. 

position independent 는 코드가 로딩타임에 relocation 될 필요가 없다는 장점이 있다. 
오직 GOT만 relocation 되면된다. 코드가 동적으로 로딩될때 말이다. 이것은 로딩시간을 극적으로 개선시킬 수 있다. 

GOT?  
 The mechanism used in ELF uses a table called the __Global Offset Table__. This table contains entries, one entry per global entity we want to access. Each entry, at run time, will hold the absolute address of the object. Every program and dynamic library has its own GOT, not shared with anyone else.

이런 일련의 과정은 운영체제가 virtual memory를 사용함으로써 가능하다. dynamic lib는 각각의 프로세스가 공유할 수 있다. (추가적인 물리 메모리할당이 필요없다는 뜻) 

단점은 GOT때문에 발생한다. 전역변수를 접근하는게 복잡해진다는 것이다. 


## Accessing a global variable

전역변수를 access하는게 얼마나 복잡한지 예제를 살펴보자.  
간단한 예제를 소개한다. 전역변수 값을 증가시킬 것인데, 이 전역변수는 Library에서 제공하는 것이다.  
Static library 방식과 Dynamic library 방식 두가지를 확인해보자.  

#### Static library

```asm
/* mylib.s */
.data
 
.balign 4
.globl myvar
myvar : .word 42     /* 42 as initial value */
.size myvar, .-myvar
```
> mayvar 라는 전역변수를 선언햇다.  
> .size 는 dynamic library를 사용할때 필요한 지시자이다.  
> `.-myvar` : current addreaa(.) 

그리고 이 라이브러리를 사용하는 메인함수는 아래와 같다.  
특별할것 없는 예 였다.


```asm
/* main.s */
.data
.globl myvar
 
.text
.globl main
 
.balign 4
main:
    ldr r0, addr_myvar  /* r0 ←  &myvar */
    ldr r1, [r0]        /* r1 ←  *r0 */
    add r1, r1, #1      /* r1 ←  r1 + 1 */
    str r1, [r0]        /* *r0 ←  r1 */
 
    mov r0, #0          /* end as usual */
    bx lr
 
addr_myvar: .word myvar
```

어셈블러로 컴파일, static lib (.a) 을 생성한뒤  한뒤 link를 해보자.  

```asm
# (static) library
as -o mylib.o mylib.s
ar cru mylib.a mylib.o
# program
as -o main.o main.s
gcc -o main main.o -L. -l:mylib.a
```
> `ar`, the archiver: is the tool that creates a static library, an .a file

Static library 방식은 특별할것이 없었다.  

#### Dynamic library

이제 Dynamic library로 만들어본다. Dynamic library를 생성하기 위해서는 Linker에게 dynamic lib을 사용할것이라고 명시해야한다. ELF에서  Dynamic library 는 __Shared Object (.so)__ 라고 불린다. 

```asm
# dynamic library
as -o mylib.o mylib.s
gcc -shared -o mylib.so mylib.o
```
> mylib.so 생성

이제 우리 프로그램이 position independent 한 방법으로 변수 myvar을 access하게 만들어보자. 
사실,  PIC는 오직 dynamic lib을 위해서만 존재한다. 

PIC 접근방식은 pc-relative 하게 주소를 사용한다. 그러기 위해서는 GOT필요.
아래는 static linker로부터 GOT의 정확확한 offset을 얻어올수 있다.

```asm
...
.word _GLOBAL_OFFSET_TABLE_
...
```

이상적이라면 아래와 같이 할수 있어야하나, 불가능하다. 

```asm
add r0, pc, .word _GLOBAL_OFFSET_TABLE_  /* r0 ←  pc + "offset-to-GOT" */
```

그래서 아래와 같은 기존의 방식대로 한다. 

```asm
ldr r0, offset_of_GOT    /* r0 ←  "offset-to-GOT" */
add r0, pc, r0           /* r0 ←  pc + r0 */ 
...
offset_of_GOT: .word _GLOBAL_OFFSET_TABLE_
```

But that "offset-to-got" must be the offset to the GOT at the point where we are actually adding the pc, this is, in the second instruction. This means that we need to ask the linker to adjust it so the offset make sense for the instruction that adds that offset to the pc. We can do this using an additional label.

```asm
ldr r0, offset_of_GOT           /* r0 ←  "offset-to-GOT" */
got_address: add r0, pc, r0     /* r0 ←  pc + r0 */ 
...
offset_of_GOT: .word _GLOBAL_OFFSET_TABLE_  - got_address
```

ARM의 pc reading 특성을 고려해서 최종적으로 아래와 같이 하면 된다. (사실 이해 잘 못함)

```asm
ldr r0, offset_of_GOT           /* r0 ←  "offset-to-GOT" */
got_address: add r0, pc, r0     /* r0 ←  pc + r0 */ 
...
offset_of_GOT: .word _GLOBAL_OFFSET_TABLE_  - (got_address + 8)
```

이제 r0에는 GOT의 절대 주소가 들어있다. 그러나 우리는 myvar 변수를 access 해야한다. 어떻게해야할까? static linker에게 해당심볼을 위해 GOT내의 offset을 사용하라고 아래와같은 방식으로 알려줘야한다. 

```asm
.word myvar(GOT)
```

이제 position independent한 방법으로 myval에 access 해보자 

![](https://thinkingeek.com/wp-content/uploads/2017/04/got.svg_.png)


```asm
/* main.s */
.data
 
.text
.globl main
 
.balign 4
 
main:
  ldr r0, offset_of_GOT        /* r0 ←  offset-to-GOT
                                  (respect to got_address)*/
  got_address: add r0, pc, r0  /* r0 ←  pc + r0 
                                  this is
                                    r0 ←  &GOT */
  ldr r1, myvar_in_GOT         /* r1 ←  offset-of-myvar-inside-GOT */
  add r0, r0, r1               /* r0 ←  r0 + r1 
  ^^^^^^^^^^^^^^                  this is r0 ←  &GOT + offset-of-myvar-inside-GOT */
  ldr r0, [r0]                 /* r0 ←  *r0
  ^^^^^^^^^^^^                    this is r0 ←  &myvar
                                */
  ldr r1, [r0]                  /* r0 ←  *r1 */
  add r1, r1, #1                /* r1 ←  r1 + 1 */
  str r1, [r0]                  /* *r0 ←  r1 */
 
  mov r0, #0          /* end as usual */
  bx lr
 
offset_of_GOT: .word _GLOBAL_OFFSET_TABLE_  - (got_address + 8)
myvar_in_GOT : .word myvar(GOT)
```
> 밑줄처럼 r0에 GOT내부 오프셋으로 얻어낸 myval 주소가 담겨있다. 

참고로 위 두 밑줄을 아래와 같이 사용할 수 도있다. 

```asm
  ldr r0, [r0, r1]     /* r0 ←  *(r0 + r1)
                          this is r0 ←  *(&GOT + offset-of-my-var-inside-GOT) */
```

이제 build 해보자 

```asm
# program
as -o main.o main.s
gcc -o main main.o -L. -l:mylib.so -Wl,-rpath,$(pwd)
```
> `-Wl`, `-rpath,$(pwd)` option tells the dynamic linker to use the current directory, $(pwd), to find the library. 


## Calling a function

Dynamic lib에서 함수를 호출 하는것은 GOT에서 access하는것 보다 약간 더 복잡하다(more involved). ELF 에서 Lazy Binding 이라고 불리는 기능 때문이다. 함수는 게으른 방식으로 로드 될 수 있다. 그 이유는 라이브러리는 정말 많은 함수를 제공하지만 실행중인 프로그램은 그 중 매우 적은 함수만 요구하기 때문이다. 만약 static linking을 사용한다면 거의 문제가 없다.  왜냐하면 링커는 프로그램에서 잠재적으로 사용하는 심볼을 정의하는 객체 파일 만 사용하기 때문이다. 그러나 동적 라이브러리는 전체적으로 처리해야하므로이 작업을 수행 할 수 없다.  

따라서 Lazy loading에서는 미리 하는것이 아니라, 함수를 처음 호출할 때 로드 해야한다. 이 방식은 효율적이지만 약간의 수고가 필요하다. ELF에서는 Procedure Linkage Table(PLT)을 사용하여 이 작업을 수행한다. 테이블의 항목에는 프로그램에 의해  잠재적으로 사용 될 함수들이 있다. 이 항목들은 GOT에도 복사 된다.  GOT와 달리 PLT는 코드이므로 PLT를 수정하지는 않는다. PLT의 항목은 GOT의 항목으로 분기하는 작은 일련의 명령어이다.  

함수에 대한 GOT 엔트리는 동적 링커에 의해 함수의 주소를 검색하는 동적 링커의 내부 함수에 대한 주소로 초기화되고, GOT를 해당 주소로 업데이트하고 분기한다. 동적 링커가 GOT 테이블을 업데이트했기 때문에, 다음번 호출이 PLT를 통해 일어났을때, 다음 호출 (단순히 GOT으로 분기 함)이 함수로 직접 이동한다.   

그렇다면 왜 직접 GOT주소에 접근하지 않고 PLT를 거치는가? 이유는 dynamic linker는 반드시 처음으로 로드되어야 할 함수가 어떤 함수인지 알아야 하기 때문이다.  GOT에서 직접 주소를 호출하면 어떤 함수를로드해야 하는지를 알 수있는 메커니즘을 따로 고안해야한다.  동적 로더가 로드해야하는 정확한 함수를 알고 있도록 모든 것을 준비하는 테이블에 함수에 대한 GOT 항목을 초기화하는 방법이있을 수 있다. 그러나 이것은 실제로 PLT와 동일하다!  

복잡해보이지만 다행이도, PLT 항목을 만드는것은 우리가 아니라 링커이며, 그 PLT항목은 일반 함수 호출로 사용될 수 있다는것이다.  GOT 항목에 대한 주소를 얻어올 필요가 없으며, 변수에 대한 모든 작업이 필요가 없다.  (함수의 주소를 사용하려면이 작업을 수행해야한다!).  우리는 항상 그렇게 할 수는 있지만 모든 함수 호출이 GOT 테이블에서 복잡한 색인을 요구하기 때문에 코드가 늘어나게 될 것이다.  이 메커니즘은 PIC와 non-PIC 모두에서 작동하며, 어떤 동적 라이브러리를 사용할지 여부를 염려 할 필요없이, 우리가 printf와 같은 C 라이브러리 함수를 호출 할 수 있었던 이유이다.  즉, PLT를 통해 함수를 호출하고자 함을 명시하기 위해 접미사 __@PLT__를 명시 적으로 사용할 수 있다. 이것은 라이브러리 내에서 수행되는 호출에 대해 필수이다.  

- __결론__: PIC code를 이용하는 동적라이브러리내의 
변수: __GOT__ 통해 access  
함수: __PLT__ 통해 access  

### Complete example

우리가 위에서 만든 라이브러리에 myvar 값을 출력하는 함수도 추가해보자. 이 코드는 라이브러리내에 있으므로, 변수를 GOT를 통해 접근하고 PLT를 통해 함수를 접근하는 PIC code이어야 한다. 
변수를 increment하는것 말고는 이전예제와 동일 


```asm
/* mylib.s */
.data
 
.balign 4
.globl myvar
myvar : .word 42 /* global variable "myvar" */
.size myvar, .-myvar
 
message: .asciz "Value of 'myvar' is %d\n"
 
.text
 
.balign 4
.globl myfun
myfun:
  push {r4, lr}                /* we are going to do a
                                  call so keep lr, and also r4
                                  for a 8-byte aligned stack */
  ldr r0, offset_of_GOT        /* r0 ←  offset-to-GOT
                                  (respect to got_address)*/
  got_address: add r0, pc, r0  /* r0 ←  pc + r0 
                                  this is r0 ←  &GOT */
  ldr r1, myvar_in_GOT         /* r1 ←  offset-of-myvar-inside-GOT */
  ldr r0, [r0, r1]             /* r0 ←  *(r0 + r1)
                                  this is r0 ←  *(&GOT + offset-of-myvar-inside-GOT) */
  ldr r1, [r0]                 /* r0 ←  *r1 */
 
  ldr r0, addr_of_message      /* r0 ←  &message */
  /* r1 already contains the value we want */
  bl printf@PLT                /* call to printf via the PLT */
  ^^^^^^^^^^^^^
  pop {r4, lr}                 /* restore registers */
  bx lr

offset_of_GOT: .word _GLOBAL_OFFSET_TABLE_  - (got_address + 8)
myvar_in_GOT : .word myvar(GOT)
addr_of_message: .word message
```
> myvar변수를 접근하는것은 GOT를 이용하는것은 이전 main함수의 방식과 동일하다.  

그리고 라이브러리내의 myval변수를 ++하는 myfun 함수를 호출할 수 있도록 main함수를 수정해보자.   

```asm
/* main.s */
.data
 
.text
.globl main
 
.balign 4
 
main:
  push {r4, lr}                /* we are going to do a
                                  call so keep lr, and also r4
                                  for a 8-byte aligned stack */
  bl myfun@PLT                 /* call function in library */
 
  ldr r0, offset_of_GOT        /* r0 ←  offset-to-GOT
                                  (respect to got_address)*/
  got_address: add r0, pc, r0  /* r0 ←  pc + r0 
                                  this is
                                    r0 ←  &GOT */
  ldr r1, myvar_in_GOT         /* r1 ←  offset-of-myvar-inside-GOT */
  ldr r0, [r0, r1]             /* r0 ←  *(r0 + r1)
                                  this is
                                    r0 ←  *(&GOT + offset-of-myvar-inside-GOT) */
  ldr r1, [r0]                 /* r0 ←  *r1 */
  add r1, r1, #1               /* r1 ←  r1 + 1 */
  str r1, [r0]                 /* *r0 ←  r1 */
 
  bl myfun@PLT                 /* call function in library a second time */
 
  pop {r4, lr}                 /* restore registers */
  mov r0, #0                   /* end as usual */
  bx lr
 
offset_of_GOT: .word _GLOBAL_OFFSET_TABLE_  - (got_address + 8)
myvar_in_GOT : .word myvar(GOT)
```

그리고 dynamic lib과 함께 빌드!  

```bash
# Dynamic library
 $ as -o mylib.o mylib.s
 $ gcc -shared -o mylib.so mylib.o
# Program
 $ as -o main.o main.s
 $ gcc -o main main.o -L. -l:mylib.so -Wl,-rpath,$(pwd)
```
ldd 명령을 사용해서 어떤 동적라이브러리가 사용되었는지 확인이 가능하다. 
ldd - print shared library dependencies

```asm
$ ldd main
	/usr/lib/arm-linux-gnueabihf/libcofi_rpi.so (0xb6f3b000)
	mylib.so => $(pwd)/mylib.so (0xb6f32000)
	libc.so.6 => /lib/arm-linux-gnueabihf/libc.so.6 (0xb6df0000)
	/lib/ld-linux-armhf.so.3 (0x7f5cb000)
```

그리고 실행해보면 잘 동작한다! 


```asm
$ ./main 
Value of 'myvar' is 42
Value of 'myvar' is 43
```


<div style="page-break-after: always;"></div>


# 그외 명령어들

## TST 및 TEQ
`TST x y` y에 설정된 비트가 모두 x에 설정되어 있는지 검사
`TEQ x y` x와 y의 값이 동일 한지 검사 

`TST` 명령어는 Rn의 값과 Operand2의 값에 대해 비트 단위 AND 연산을 수행합니다. 이 명령어는 결과가 버려진다는 점을 제외하고 ANDS 명령어와 동일합니다.

`TEQ` 명령어는 Rn의 값과 Operand2의 값에 대해 비트 배타적 OR 연산을 수행합니다. 이 명령어는 결과가 버려진다는 점을 제외하고 EORS 명령어와 동일합니다.
`cmp` 대신 사용할 수 있음. carry flag가 변하지 않음

TEQ 명령어를 사용하면 CMP와 마찬가지로 V 또는 C 플래그에 영향을 주지 않고 두 값이 동일한지 여부를 테스트할 수 있습니다.
TEQ는 값의 부호를 테스트하는 데도 유용합니다. 비교 후, N 플래그는 두 피연산자 부호 비트의 논리 배타적 OR이 됩니다.
http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0204ik/Cihcdehh.html

```asm
	mov	r0, #1
	teq	r0, #1
	beq	teq_test
```
> 두 비트가 동일하면 teq_test 로 branch

http://forum.falinux.com/zbxe/index.php?document_srl=762683&mid=lecture_tip

----
To be updated..
