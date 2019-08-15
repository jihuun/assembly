
# 7. Indexing modes
----


## Indexing modes란?

ARM 명령어는 operand로 register또는 immediate value 를 사용한다.(load, store, branch 제외)

```
명령어 Rdest, Rsource1, source2
```
Operand는 레지스터 이거나 혹은 값이 될 수 있다.  
첫번째 Operand 즉, Rdest위치에 항상 destination이 온다. (STR 예외)   
Rsource1 에는 항상 register가 온다.  
source2에는 register가 올 수도 있고, immediate value가 올 수도 있다.  

이렇게 명령어의 Operand 집합을 총칭하여 인덱싱 모드라고 한다.   

## Shifted operand

ARM은 Shift (좌/우) operation 에 관련 명령어가 없다.  
Operand는 Shift operation과 결합될 수 있다.  

```
LSL #n 		Logical Shift Left. 
LSL Rsource3
LSR #n		Logical Shift Right.
LSR Rsource3
ASR #n		Arithmetic Shift Right.
ASR Rsource3
ROR #n	 	Rotate Right.
ROR Rsource3
```
> source2 

```asm
mov r1, r2, LSL #1
```

## Usage of Shifted operand 

```
mov r1, r2, LSL #1      /* r1 ← (r2*2) */
mov r1, r2, LSL #2      /* r1 ← (r2*4) */
mov r1, r3, ASR #3      /* r1 ← (r3/8) */
mov r3, #4
mov r1, r2, LSL r3      /* r1 ← (r2*16) */
```
> shift 하여 2의 배수 곱셈 가능

```
add r1, r2, r2, LSL #1   /* r1 ← r2 + (r2*2) equivalent to r1 ← r2*3 */
add r1, r2, r2, LSL #2   /* r1 ← r2 + (r2*4) equivalent to r1 ← r2*5 */
```
> 이렇게 하면 홀수배도 곱셈 가능

```
sub r1, r2, r2, LSL #3  /* r1 ← r2 - (r2*8) equivalent to r1 ← r2*(-7)
```
> 음수배 곱셈 가능

```
rsb r1, r2, r2, LSL #3      /* r1 ← (r2*8) - r2 equivalent to r1 ← r2*7 */
```
> rsb 로 곱셈 가능
> rsb (Reverse Substract)

```
sub Rd A B
```
> A - B

```
rsb Rd A B
```
> B - A
> Shift 연산에서 어떤 값을 뺄때 유용



- 왜 더 간단한 곱셈연산을 사용하지 않나?

일반 곱셈연산은 연산량이 더 많다. 계산하는데  오버헤드가 더 크다. 





