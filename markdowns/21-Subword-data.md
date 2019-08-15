
# 21. Subword data  
----

sub-word 즉, word interger 32bit size보다 작은 데이터를 사용하려면 어떻게 해야하나?  


## Subword data  

byte(2byte), half word(4byte) 를 사용하는 방법.   

```asm
.align 4
one_byte: .byte 205
/* This number in binary is 11001101 */
 
.align 4
one_halfword: .hword 42445
/* This number in binary is 1010010111001101 */
```
> `.byte` 1byte 만 값 저장  
> `.hword`2byte 만 값 저장  

## load and store 

ARM은 1Byte 2Byte 데이터를 load하기 위해서 다음과 같은 명령어를 제공한다.  
`ldrb` `ldrh` load되지 않는 나머지 비트들은 0으로 채워진다.  

- __Load__

	```asm
	.text
	 
	.globl main
	main:
	    push {r4, lr}
	 
	    ldr r0, addr_of_one_byte     /* r0 ←  &one_byte */
	    ldrb r0, [r0]                /* r0 ←  *{byte}r0 */
	 
	    ldr r1, addr_of_one_halfword /* r1 ←  &one_halfword */
	    ldrh r1, [r1]                /* r1 ←  *{half}r1 */
	 
	    pop {r4, lr}
	    mov r0, #0
	    bx lr
	 
	addr_of_one_byte: .word one_byte
	addr_of_one_halfword: .word one_halfword
	```
	> `ldr`이 아니라, `ldrb` `ldrh` 사용됨 유의.  

- __메모리에 로딩되는 형태__

	만약 다음과 같은 값이 주소 [addr] 부터 1바이트씩 저장 되어있다면  

	|주소	| Data		|
	|-------|---------------|
	|addr 	| 11001101	|
	|addr+1 | 10100101	|

	`ldrb addr` 그리고 `ldrh addr` 했을때, register에 로딩되는 값의 모습은 다음과 같다. 

	| 명령어	| 32-bit register r0에 로딩된 값		|
	|---------------|---------------------------------------|
	|ldr**b**  r0 [addr] |00000000 00000000 00000000 11001101|
	|ldr**h**  r0 [addr] |00000000 00000000 10100101 11001101|

	참고로 Little Endian 아키텍쳐이다. 위 표처럼  Byte는 낮은 주소에 LSB 높은 주소에 MSB 순으로 놓인다. 

- __2's complement 로드하기__

	만약 음수값을 사용한다면 `ldrsb` `ldrsh`를 사용해야 한다.  
	(아래 표) 만약 1바이트 크기의 음수 값 -51 (0x11001101)을 `ldr`로 로드하면 r0에 205로 잘못 저장된다. 그럴때는 `ldrsb`로 로드해야 정상적으로 -51로 저장된다.   

	| 명령어	| 32-bit register r0에 로딩된 값		| 10진수 |
	|---------------|---------------------------------------|---------|
	|ldr**b**  r0 [addr]  |00000000 00000000 00000000 11001101| 205 |
	|ldr**sb**  r0 [addr] | 11111111 11111111 11111111 11001101| -51 |


- __Store__

	Store연산은 위의 2's complement 를 고려할 필요없이 `strb` `strh` 만 사용하면 된다. 

	```asm
	ldr r1, addr_of_one_byte     /* r0 ←  &one_byte */
	ldrsb r0, [r1]               /* r0 ←  *{signed byte}r1 */
	strb r0, [r1]                /* *{byte}r1 ←  r0 */

	ldr r0, addr_of_one_halfword /* r0 ←  &one_halfword */
	ldrsh r1, [r0]               /* r1 ←  *{signed half}r0 */
	strh r1, [r0]                /* *{half}r0 ←  r1 */
	```

### 참고 ldrd, strd

d 는 doubleword  

```
LDR{type}{cond}{.W} Rt, label
LDRD{cond} Rt, Rt2, label        ; Doubleword
```
> type:
`B`  
unsigned Byte (Zero extend to 32 bits on loads.)  
`SB`  
signed Byte (LDR only. Sign extend to 32 bits.)  
`H`  
unsigned Halfword (Zero extend to 32 bits on loads.)  
`SH`  
signed Halfword (LDR only. Sign extend to 32 bits.)  
-  
omitted, for Word.  

