
# 5. Branches
----

  
<!-- toc -->


## r15, a PC register

`pc` has address which is next instruction going to be executed.
__The branching is same with changing `pc` register value.__
ARM has own branch instruction for modifying `pc` register.

`pc` (Program Counter) 는 다음에 실행될 명령어 주소가 담겨있다.   

## Unconditional branches

Instruction `b` with label is used for unconditional branching.

```asm
.text
.global main
main:
    mov r0, #2 /* r0 ←  2 */
    b end      /* branch to 'end' */
    mov r0, #3 /* r0 ←  3 */
end:
    bx lr
```

### 참고: branch instruction 종류

- `b` : branch <immediate>  
명령어 뒤에 지정된 상수값(혹은 Label)에 해당하는 주소로 분기  

- `bl` : branch with link <immediate>  
명령어 뒤에 지정된 상수값(혹은 Label)에 해당하는 주소로 분기 하되, 현재의 `pc` + 0x4 번지의 복귀 주소값을 `lr` 레지스터에 저장   

- `bx` : branch indirect <register>  
명령어 뒤에 지정된 레지스터로 분기  

- `blx` : branch indirect with link <register>  
명령어 뒤에 지정된 레지스터로 분기 하되,  현재의 `pc` + 0x4 번지의 복귀 주소값을 `lr` 레지스터에 저장    

	출처 [http://trace32.com/wiki/index.php/B,_BL,_BX_and_BLX](http://trace32.com/wiki/index.php/B,_BL,_BX_and_BLX)  

## Conditional branches

### CPSR (Current Program Status Register)

It has flags called N (negative), Z (zero), C (carry) and V (overflow).  
These condition flags are read by branch instructions. And,  
Arithmetic instruction/ special testing/ comparison instruction이 이 상태 레지스터 값을 update 할 수 있다.

	* N : Negative : 연산결과가 마이너스인 경우에 set됨
	* Z : Zero : 연산결과가 0인 경우에 set됨
	* C : Carry : 연산결과에 자리 올림이 발생한 경우에 set됨.
	* V : oVer flow : 연산의 결과가 overflow 났을 경우에 set됨.
		(by any instructions)
```
cmp r1, r2
```
> will update cpsr doing "r1 - r2", (r1, r2값은 바뀌지 않음)  
cmp 명령어만 cpsr 을 update할 수 있다? -> [#12장 참고](#12. Loops and the status register)  
>
> r1 < r2 인경우:	N updated   
r1 == r2 인경우:	Z updated  
r1 == 1, r2 == 0 인경우: C enabled(1) (no carries)  
r1 == 0, r2 == 1 인경우: C disabled(0)  
r1 == 2147483647(+ integer 최대 값), r2 == -1 인 경우: Overflow, V updated  

```txt
EQ	(equal) When Z is enabled (Z is 1)
NE 	(not equal). When Z is disabled. (Z is 0)
GE 	(greater or equal than, in two’s complement).
	When both V and N are enabled or disabled (V is N)
LT 	(lower than, in two’s complement).
	This is the opposite of GE, so when V and N are not both enabled or disabled
	(V is not N)
GT 	(greather than, in two’s complement).
	When Z is disabled and N and V are both enabled or disabled
	(Z is 0, N is V)
LE 	(lower or equal than, in two’s complement).
	When Z is enabled or if not that, N and V are both enabled or disabled
	(Z is 1. If Z is not 1 then N is V)
MI 	(minus/negative) When N is enabled (N is 1)
PL 	(plus/positive or zero) When N is disabled (N is 0)
VS 	(overflow set) When V is enabled (V is 1)
VC 	(overflow clear) When V is disabled (V is 0)
HI 	(higher) When C is enabled and Z is disabled (C is 1 and Z is 0)
LS 	(lower or same) When C is disabled or Z is enabled (C is 0 or Z is 1)
CS/HS 	(carry set/higher or same) When C is enabled (C is 1)
CC/LO 	(carry clear/lower) When C is disabled (C is 0)
```
> 위 condition들이 b 와 합쳐져서 instruction 이름이 됨.

가령 beq는 Z가 1일 때만 branch함.


```asm
/* -- compare01.s */
.text
.global main
main:
    mov r1, #2       /* r1 ←  2 */
    mov r2, #2       /* r2 ←  2 */
    cmp r1, r2       /* update cpsr condition codes with the value of r1-r2 */
    beq case_equal   /* branch to case_equal only if Z = 1 */
case_different :
    mov r0, #2       /* r0 ←  2 */
    b end            /* branch to end */
case_equal:
    mov r0, #1       /* r0 ←  1 */
end:
    bx lr
```




