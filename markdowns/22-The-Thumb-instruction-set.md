
# 22. The Thumb instruction set  
----
  
<!-- toc -->


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



