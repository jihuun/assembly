
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








