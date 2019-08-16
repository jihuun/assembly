
# 16. Switch control structure  
----

  
<!-- toc -->

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





