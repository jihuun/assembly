
# 6. Control Structures
----


## 조건문 if, then, else

```c
if (E) then
	S1
else
	S2
```

> 이런 high level 프로그래밍 구조가 Assembly 에선 아래와 같은 구조가 됨.  

```asm
if_eval:	/* Assembler that evaluates E and updates the cpsr accordingly */

bXX else	/* Here XX is the appropiate condition */
then_part:	/* assembler for S1, the "then" part */
    b end_of_if
else:		/* assembler for S2, the "else" part */
end_of_if:
```

## 반복문 Loops

```c
while (E)
	s
```

```asm
while_condition:	/* assembler to evaluate E and update cpsr */
    bXX end_of_loop	/* If E is false, then leave the loop right now */

    			/* assembler of S */
    b while_condition	/* Unconditional branch to the beginning */
end_of_loop:
```

## Asm code for 1+2+3+4+...+22

- 내가 짜본 코드: 틀렸음
	```asm
	mov r1 #0
	mov r0 #1
	while_condition:
		bvs end_of_loop	/* overflow condition for cpsr */
		mov r1 r0	/* r1 1 */
		add r0 r0 #1 	/* r0 1+1 */
		add r1 r1 r0	/* r1 1 + 2 */
		b while_condition
	end_of_loop:
	```
	> my code: 22까지 더하면 253인데 overflow 발생한다고 생각했음.


- loop01.s
Reference code

	```asm
	/* -- loop01.s */
	.text
	.global main
	main:
	    mov r1, #0       /* r1 <- 0 (sum val) */
	    mov r2, #1       /* r2 <- 1 (counter val) */
	loop: 
	    cmp r2, #22      /* compare r2 and 22 */
	    bgt end          /* branch if r2 > 22 to end */
	    add r1, r1, r2   /* r1 <- r1 + r2 */
	    add r2, r2, #1   /* r2 <- r2 + 1 */
	    b loop
	end:
	    mov r0, r1       /* r0 <- r1 */
	    bx lr
	```

	```asm
	    cmp r2, #22 
	    bgt end    
	```
	> cpsr will be updated by cmp operation, then branch out by "gt" flag 

- 참고: loop02.s
loop01 보다 개선된 버전이다.  

	```asm
	.text
	.global main
	main:
	    mov r1, #0       /* r1 ← 0 */
	    mov r2, #1       /* r2 ← 1 */
	    b check_loop     /* unconditionally jump at the end of the loop */
	loop: 
	    add r1, r1, r2   /* r1 ← r1 + r2 */
	    add r2, r2, #1   /* r2 ← r2 + 1 */
	check_loop:
	    cmp r2, #22      /* compare r2 and 22 */
	    ble loop         /* branch if r2 <= 22 to the beginning of the loop */
	end:
	    mov r0, r1       /* r0 ← r1 */
	    bx lr
	```
	> 01과 명령어 갯수는 동일하다. 하지만 01과 다르게 마지막 조건에서 branching 하는 횟수가 하나 적다. 
	> ble loop 다음 end: 로 branching없이 진행되기 때문이다.  


## Asm code for 3n + 1 problems

다른 말로 [콜라츠 추측](https://ko.wikipedia.org/wiki/%EC%BD%9C%EB%9D%BC%EC%B8%A0_%EC%B6%94%EC%B8%A1) 이라고 부른다  
> 임의의 자연수가 다음 조작을 거쳐 항상 1이 된다는 추측  
	1. 짝수라면 2로 나눈다.  
	2. 홀수라면 3을 곱하고 1을 더한다.  
	3. 1이면 조작을 멈추고, 1이 아니면 첫 번째 단계로 돌아간다.  
	예를 들어, 6 에서 시작한다면, 차례로 6, 3, 10, 5, 16, 8, 4, 2, 1 이 된다.

```c
n = ...;
while (n != 1)
{
  if (n % 2 == 0)
     n = n / 2;
  else
     n = 3*n + 1;
}
```

```asm
/* -- collatz.s */
.text
.global main
main:
    mov r1, #123           /* r1 <- 123 */
    mov r2, #0             /* r2 <- 0 */
loop:
    cmp r1, #1             /* compare r1 and 1 */
    beq end                /* branch to end if r1 == 1 */
 
    and r3, r1, #1         /* r3 <- r1 & 1 */
    cmp r3, #0             /* compare r3 and 0 */
    bne odd                /* branch to odd if r3 != 0 */
even:
    mov r1, r1, ASR #1     /* r1 <- (r1 >> 1) */
    b end_loop
odd:
    add r1, r1, r1, LSL #1 /* r1 <- r1 + (r1 << 1) */
    add r1, r1, #1         /* r1 <- r1 + 1 */
 
end_loop:
    add r2, r2, #1         /* r2 <- r2 + 1 */
    b loop                 /* branch to loop */
 
end:
    mov r0, r2
    bx lr
```


```asm
    and r3, r1, #1         /* r3 <- r1 & 1 */
```
> bitwise operator (AND)

```asm
    mov r1, r1, ASR #1     /* r1 <- (r1 >> 1) */
```
> ASR (arithmetic shift right) 로 추가 reg사용없이 쉬프트 가능 for r1 <- r1/2

```asm
    add r1, r1, r1, LSL #1
```
> LSL (logical shift left) r1 x 2 한 뒤 다시 r1을 더해서 r1 x 3이 됨.




