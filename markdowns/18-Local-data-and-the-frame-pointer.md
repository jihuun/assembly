
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


