
# 12. Loops and the status register  
----
  
<!-- toc -->

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
  
  
