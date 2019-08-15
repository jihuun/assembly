
# 2. Registers and Basic arithmetic
----

두 `r#` 레지스터를 더하는 방법

- sum01.s

	```asm
	.global main
	 
	main:
	    mov r1, #3      /* r1 <- 3 */
	    mov r2, #4      /* r2 <- 4 */
	    add r0, r1, r2  /* r0 <- r1 + r2 */
	    bx lr
	```
	> r1 r2를 더해 r0 에 저장  


- sum02.s

	```asm
	.global main
	 
	main:
	    mov r0, #3      /* r0 <- 3 */
	    mov r1, #4      /* r1 <- 4 */
	    add r0, r0, r1  /* r0 <- r0 + r1 */
	    bx lr
	```
	> r0 를 재활용함.  

- Run on the target RPI after compile

	```bash
	 $ ./sum01 ; echo $?
	 7

	 $ ./sum02 ; echo $?
	 7
	```

