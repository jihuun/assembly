
# 10. Functions-2 The stack  
----
  
<!-- toc -->
  
  
  
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
  
  
  
  
