
# 9. Functions-1  
----
  
  
<!-- toc -->

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
  
  
  
  
  
  
