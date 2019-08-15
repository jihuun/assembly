
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




