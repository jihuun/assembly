
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


