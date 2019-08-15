
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


