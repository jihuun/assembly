
# 26. A primer about linking
----

`NOTE`: #26, #27장에 걸쳐 소개되는 Linking 은 온전히 이해하지 못한부분이 많다. 앞으로도 관심갖고 지속적으로 수정 보완 할 예정.  

## GCC 와 Keil assembler 비교

이번장 Linker에 대해 소개하기 전에 먼저 짚고 넘어갈 사항이다. 사실 지금까지 배운 어셈블리 코드 구조는 gcc 기반이다. 반면 ARM에는 keil 이라는 어셈블러도 존재하는데 코드 골격이 약간 달라서 소개한다. 아래 두 코드는 syntax만 다를뿐 동일한 동작이다.

- Keil MDK-ARM syntax

	```asm
	; ARM Assembly Language Program To Add Some Data and Store the SUM in R3.

	       AREA PROG_2_1, CODE, READONLY
	       ENTRY
	       MOV R1, #0x25   ; R1 = 0x25 
	       MOV R2, #0x34   ; R2 = 0x34 
	       ADD R3, R2, R1  ; R3 = R2 + R1 
	HERE   B HERE          ; stay here forever 
	       END
	```
- GCC v2.24 syntax

	```asm
	@P2_1.s ARM Assembly Language Program To Add Some Data and Store the SUM in R3.

	         .global _start
	_start:  MOV R1, #0x25      @ R1 = 0x25
	         MOV R2, #0x34      @ R2 = 0x34
	         ADD R3, R2, R1     @ R3 = R2 + R1
	HERE:    B HERE             @ stay here forever
	```
그리고 GCC 와 Keil의 지시자 syntax가 어떻게 다른지는 아래와 같다.  

- 표: Assembler 지시자 의미: GCC 와 Keil assembler 비교

	| GCC | Directive | Keil MDK-ARM Directive Explanation |
	|-----|-----------|------------------------------------|
	| .text |TEXT| Signifies the beginning of code or constant|
	| .data |DATA |Signifies the beginning of read/write data |
	| .global label |GLOBAL or EXPORT |Makes the label visible to linker |
	| .extern |label EXTERN |label is declared outside of this file |
	| .byte byte [,byte, byte, ..] |DCB |Declares byte (8-bit) data |
	| .hword hw [,hw, hw, ..] |DCW |Declares halfword (16-bit) data |
	| .word word [,word, word, ..] |DCD |Declares word (32-bit) data |
	| .float float [,float, float, ..] |DCFS |Declares single precision floating point (32-bit) data |
	| .double double [,double, double, ..] |DCFD| Declares double precision floating point (64-bit) data |
	| .space #bytes [,fill] |SPACE or FILL |Declares memory (in bytes) with optional fill |
	| .align n |ALIGN | Aligns address to <2 to the power of n> byte |
	| .ascii "ASCII string" | DCB "ASCII string"  |Declares an ASCII string |
	| .asciz "ASCII string" | DCB "ASCII string", 0| Declares an ASCII string with null termination |
	| .equ symbol, value |EQU |Sets the symbol with a constant value |
	| .set variable, value |SETA |Sets the variable with a new value |
	| .end |END| Signifies the end of the program |


## Linkers, the magic between symbols and addresses

링커는 주목받지 못하는 tool이지만 프로그램을 만드는데에 핵심이다.   
링커의 주된 동작은 모든 코드 조각들을 합치는 것이다. 컴퓨터가 실제로 실행 가능하도록 말이다. 보다 정확히 말하면, Symbolic Name을 Address와 Binding 한다. 프로그램은 여러 파일로 작성할 수 있고 각각 object파일로 컴파일 된다. 그 여러개의 Object파일을 하나의 실행파일로 합치는것이 Linker가 하는 일이다.  

## ELF


여러 실행파일을 공통의 포맷으로 관리하면 좋을 것이다. 이런 포멧은 COFF, MACH-O, ELF등이 있는데 UNIX에서는 ELF를 주로 사용한다. ELF포맷은 Object 파일(relocatable), Shared Object, Executable 로 사용된다.   

Linker에게 ELF relocatable 파일은 섹션들의 모음집 이다. 섹션은 연속된 데이터들을 말한다. 이것은 instruction의 연속된 모음이 될 수도 있고, 초기화된 전역변수, 디버깅 정보등 어떤것도 될 수 있다. 각각의 섹션은 이름과 속성이 있다. 가령 속성은 다음과같은 것들이 될 수 있다. 메모리에 할당될 것인지, 이미지로부터 로딩될 것인지, 실행될 것인지, writable한지, 사이즈와, 메모리에 어떻게 Alignment 할건지 등이다.  


## Labels as symbolic names

만약 전역변수를 사용하다면 다음과 같은 구조를 사용한다. 

```asm
.data:
var: .word 42
.text
func:
    /* ... */
    ldr r0, addr_of_var  /* r0 ←  &var */
    ldr r0, [r0]         /* r0 ←  *r0 */
    /* ... */
addr_of_var : .word var
```

Assembler는 label을 다음과 같은 방식으로 PC + offset 으로 변경한다. 
이 Addressing mode는 이 offset을 12bit로 encode 할수 있다. 결국 4096byte의 offset을 가질 수 있다. 

```asm
ldr r0, [pc #offset]
```

질문, 어셈블러는 addr_of_var 라벨 위치에 어떤명령을 번역해서 놓는가? 우리는 .word var를 적었다. 그러나 이게 도대체 무슨 의미인가? 이시점에서는 var의 주소를 알수가 없다. 그래서 어셈블러는 pc + offset 이라는 일부의 정보(partial information)만 기록할 수 있는 것이다. 



## An example

두개의 전역변수가 있고, 그것을 더할 것이다. 그 뒤에 다른 파일에 작성된 함수를 호출 할 것이다. 그 함수는 결과값+1 을할것이다. 


```asm
/* main.s */
.data
 
one_var : .word 42
another_var : .word 66
 
.globl result_var             /* mark result_var as global */
^^^^^^^^^^^^^^^^^               ^^^^^^^^^^^^^^^^^^^^^^^^^^^
result_var : .word 0
 
.text
 
.globl main
^^^^^^
main:
    ldr r0, addr_one_var      /* r0 ←  &one_var */
    ldr r0, [r0]              /* r0 ←  *r0 */
    ldr r1, addr_another_var  /* r1 ←  &another_var */
    ldr r1, [r1]              /* r1 ←  *r1 */
    add r0, r0, r1            /* r0 ←  r0 + r1 */
    ldr r1, addr_result       /* r1 ←  &result */
    str r0, [r1]              /* *r1 ←  r0 */
    bl inc_result             /* call to inc_result */
    ^^^^^^^^^^^^^             /* 다른파일에 있는 함수 */
    mov r0, #0                /* r0 ←  0 */
    bx lr                     /* return */
 
addr_one_var  : .word one_var
addr_another_var  : .word another_var
addr_result  : .word result_var
```
> .globl: make the lebel visible to Linker

이 코드를 object파일로 만든뒤, objdump -d 로 어떻게 만들어졌는지 살펴보자. 

```sh
 $ as -march=armv6 -o main.o main.s      # creates object file main.o
```

## Relocations

위에서 언급했던 것 처럼 어셈블러는 final value가 무엇인지 모른다. 그 대신에 일부의 정보만을 남겨놓을수 있을 뿐이다. 이를테면 .data 기준으로 offset 이 어느정도 되는지 이런식이다. 결국 어떤 수정이 필요함을 의미한다. 그것이 바로 Relocation이다. 

```sh
$ objdump -dr main.o
```
> `-dr`: a option for reading relocations  

```txt
main.o:     file format elf32-littlearm
 
Disassembly of section .text:
 
00000000 <main>:
   0:	e59f0020 	ldr	r0, [pc, #32]	; 28 <addr_one_var>
   4:	e5900000 	ldr	r0, [r0]
   8:	e59f101c 	ldr	r1, [pc, #28]	; 2c <addr_another_var>
   c:	e5911000 	ldr	r1, [r1]
  10:	e0800001 	add	r0, r0, r1
  14:	e59f1014 	ldr	r1, [pc, #20]	; 30 <addr_result>
  18:	e5810000 	str	r0, [r1]
  1c:	ebfffffe 	bl	0 <inc_result>
			1c: R_ARM_CALL	inc_result
			^^^^^^^^^^^^^^^^^^^^^^^^^^
  20:	e3a00000 	mov	r0, #0
  24:	e12fff1e 	bx	lr
 
00000028 <addr_one_var>:
  28:	00000000 	.word	0x00000000
                        28: R_ARM_ABS32	.data
                        ^^^^^^^^^^^^^^^^^^^^^
 
0000002c <addr_another_var>:
  2c:	00000004 	.word	0x00000004
                        2c: R_ARM_ABS32	.data
                        ^^^^^^^^^^^^^^^^^^^^^
 
00000030 <addr_result>:
  30:	00000000 	.word	0x00000000
                        30: R_ARM_ABS32	result_var
                        ^^^^^^^^^^^^^^^^^^^^^^^^^^
```

`30: R_ARM_ABS32	result_var` 이런 부분이 relocation 정보이다.   
relocation 정보는 다음과 같은 구성을 따른다.  
```
OFFSET: TYPE   VALUE
```

OFFSET : 섹션으로부터 offset을 의미  
TYPE : relocation의 종류이다.  
VALUE : 주소의 symbolic entry 이다.  

위의 예제에는 다음과 같은 relocation 정보가 있다.  

.text+1c so we can call the actual inc_result.
.text+28, .text+2c are the relocations required to access .data.
.text+30 refers to the global symbol result_var.

모든 relocation은 몇 파라미터가 있다. 
S: is the address of the symbol referred by the relocation (the VALUE above)
P: is the address of the place (the OFFSET plus the address of the section itself),
A: (부가정보) is the value that the assembler has left in place.

예를 들어,
`R_ARM_ABS32` it is the value of the .word  
	R_ARM_ABS32의 relocation은 `S + A` 동작을 수행한다.   
`R_ARM_CALL` it is a set of bits in the bl instruction itself.  
	R_ARM_CALL의 relocation은 `(S + A) - P` 동작을 수행한다.   

Linker가 이것들을 계산하기 전에 inc_result 함수를 정의하지 않으면 linking은 실패 할 것이다. 

```asm
/* inc_result.s */ 다른 파일주의
.text
 
.globl inc_result
^^^^^^^^^^^^^^^^^
inc_result:
    ldr r1, addr_result  /* r1 ←  &result */
    ldr r0, [r1]         /* r0 ←  *r1 */
    add r0, r0, #1       /* r0 ←  r0 + 1 */
    str r0, [r1]         /* *r1 ←  r0 */
    bx lr                /* return */
 
addr_result  : .word result_var
                     ^^^^^^^^^^
```
> .globl inc_result 했기 때문에 링커가 이 symbol을 알수 있다.  
> result_var는 다른파일에서 .globl로 선언되었기 때문에 이 파일에서 링커가 그 symbol을 알수 있다.  


```bash
 $ as -march=armv6 -o inc_result.o inc_result.s
 $ objdump -dr inc_result.o
```
> as를 통해 obj파일 생성

```bash
inc_result.o:     file format elf32-littlearm
 
Disassembly of section .text:
 
00000000 <inc_result>:
   0:	e59f100c 	ldr	r1, [pc, #12]	; 14 <addr_result>
   4:	e5910000 	ldr	r0, [r1]
   8:	e2800001 	add	r0, r0, #1
   c:	e5810000 	str	r0, [r1]
  10:	e12fff1e 	bx	lr
 
00000014 <addr_result>:
  14:	00000000 	.word	0x00000000
			14: R_ARM_ABS32	result_var
```
> result_val 에 relocation 정보가 생성되었다.  

이제 두 파일을 합쳐보자.  

```bash
 $ gcc -o test.exe print_float.o reloc.o
```
몇가지 C 라이브러리를 포함할것이지만 여기서는 무시할 예정이다.   

```bash
 $ objdump -d test.exe
```

```bash
...
00008390 <main>:
    8390:       e59f0020        ldr     r0, [pc, #32]   ; 83b8 <addr_one_var>
    8394:       e5900000        ldr     r0, [r0]
    8398:       e59f101c        ldr     r1, [pc, #28]   ; 83bc <addr_another_var>
    839c:       e5911000        ldr     r1, [r1]
    83a0:       e0800001        add     r0, r0, r1
    83a4:       e59f1014        ldr     r1, [pc, #20]   ; 83c0 <addr_result>
    83a8:       e5810000        str     r0, [r1]
    83ac:       eb000004        bl      83c4            ; <inc_result>
                                        ^^^^^^^^^^^^^^^^
    83b0:       e3a00000        mov     r0, #0
    83b4:       e12fff1e        bx      lr
 
000083b8 <addr_one_var>:
    83b8:       00010578        .word   0x00010578
    ^^^^^       
    주소
 
000083bc <addr_another_var>:
    83bc:       0001057c        .word   0x0001057c
 
000083c0 <addr_result>:
    83c0:       00010580        .word   0x00010580
 
000083c4 <inc_result>:
    83c4:       e59f100c        ldr     r1, [pc, #12]   ; 83d8 <addr_result>
    83c8:       e5910000        ldr     r0, [r1]
    83cc:       e2800001        add     r0, r0, #1
    83d0:       e5810000        str     r0, [r1]
    83d4:       e12fff1e        bx      lr
 
000083d8 <addr_result>:
    83d8:       00010580        .word   0x00010580
 
...
```
> addr_one_var이 가지고 있는 값 00010578 은 말그대로 one_var의 주소값이다.  
> `ldr     r0, [pc, #32]`: 이 동작이 [00010578] 즉 주소가 가리키는 값을 얻어온다.  


자 그럼 `addr_one_var`, `addr_another_var`, `addr_result` 의 실제 relocate된 주소를 계산해보자.  
이 세가지의 relocation 정보는  `R_ARM_ABS32` 이었다.   
위에서 설명한대로 R_ARM_ABS32는 `S + A` 라는 동작을 수행한다. 여기서 S 는 .data 섹션이다. A는 생략가능하다.  
.data 섹션의 주소는 아래와 같이 `objdump -h` (-w: more readable) 로 얻어올 수 있다.  


```bash
 $ objdump -hw test.exe

test.exe:     file format elf32-littlearm
 
Sections:
Idx Name          Size      VMA       LMA       File off  Algn  Flags
...
 13 .text         0000015c  000082e4  000082e4  000002e4  2**2  CONTENTS, ALLOC, LOAD, READONLY, CODE
...
 23 .data         00000014  00010570  00010570  00000570  2**2  CONTENTS, ALLOC, LOAD, DATA
    ^^^^^                   ^^^^^^^^
...
```
> VMA 이 섹션의 주소를 의미한다. 따라서 .data섹션의 주소는 00010570이다.  
그리고 우리가 계산할 세 데이터의 주소는 0x00010578, 0x0001057c, 0x00010580 이었다. 따라서 .data로부터 각각의 offset은 0x8, 0xC, 0x10이다.  
Linker가 바로 이위치에 one_var, another_var, result_var 의 값을 저장하게 되는것이다!  

하지만 linker는 우리가 값을 쓰기전에 우선 어떠한 값들을 해당 위치에 놓는다. 다음의 명령으로 그 값들을 확인할 수 있다.  


```bash
 $ gcc -o test.exe main.o inc_result.o -Wl,--print-map > map.txt
 $ cat map.txt

.data           0x00010570       0x14
                0x00010570                PROVIDE (__data_start, .)
 *(.data .data.* .gnu.linkonce.d.*)
 .data          0x00010570        0x4 /usr/lib/gcc/arm-linux-gnueabihf/4.6/../../../arm-linux-gnueabihf/crt1.o
                0x00010570                data_start
                0x00010570                __data_start
 .data          0x00010574        0x0 /usr/lib/gcc/arm-linux-gnueabihf/4.6/../../../arm-linux-gnueabihf/crti.o
 .data          0x00010574        0x4 /usr/lib/gcc/arm-linux-gnueabihf/4.6/crtbegin.o
                0x00010574                __dso_handle
 .data          0x00010578        0xc main.o
                0x00010580                result_var
 .data          0x00010584        0x0 inc_result.o
 .data          0x00010584        0x0 /usr/lib/arm-linux-gnueabihf/libc_nonshared.a(elf-init.oS)
 .data          0x00010584        0x0 /usr/lib/gcc/arm-linux-gnueabihf/4.6/crtend.o
 .data          0x00010584        0x0 /usr/lib/gcc/arm-linux-gnueabihf/4.6/../../../arm-linux-gnueabihf/cr
```

> result_var 만 보이는 이유는 .globl 심볼 때문이다. 다른 변수는 global 심볼로 선언되어있지 않아보이지 않는다. 어쨋든. one_var 변수는  0x00010570 주소값에 , another_var 은 0x00010574,  result_var 은 0x00010578에 위치해있다.  




