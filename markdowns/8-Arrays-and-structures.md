
# 8. Arrays and structures
----
  
<!-- toc -->


이번 장에서는 Indexing mode for load and store 에 대해 알아본다.  
괄호 [r#] 는 r# 이 저장된 주소값을 의미한다.

## Array, structure

- 배열의 asm 코드

```c
int a[100];
```
> in C

```asm
.data
 
.balign 4
a: .skip 400
```
> in assembly
> 100 x 4 의 공간을 만든다.
> .skip 지시자: 어셈블러에게 미리 메모리 공간을 할당한다고 지시


- 구조체의 asm코드

```c
struct my_struct
{
    char f0;
    int f1;
} b;
```
> in C

```asm
.balign 4
b: .skip 8
```
> in assembly
> 왜 char 멤버가 있는데도 8바이트를 할당? C코드에서 Padding 기법 으로 alignment를 하지 않았기 때문이다.


## 배열 초기화: Indexing modes 사용 안함

```c
for (i = 0; i < 100; i++)
    a[i] = i;
```

```asm
/********** data **********/
.data
 
.balign 4
a: .skip 400

.balign 4
b: .skip 8

/********** text **********/
.text
 
.global main
main:
    ldr r1, addr_of_a       /* r1 <- &a , r1은 a의 base 주소 */
    mov r2, #0              /* r2 <- 0 */
loop:
    cmp r2, #100            /* Have we reached 100 yet<- */
    beq end                 /* If so, leave the loop, otherwise continue */
    add r3, r1, r2, LSL #2  /* r3 <- r1 + (r2*4): 배열의 주소 계산, index계산 */
    str r2, [r3]            /* *r3 <- r2 , Addressing modes */
    add r2, r2, #1          /* r2 <- r2 + 1, r2: count 변수 i */
    b loop                  /* Go to the beginning of the loop */
end:
    bx lr
addr_of_a: .word a
```
> addr_of_a: .data영역의 심볼을 .text영역으로 가져온것임. (#3장 참고)



```asm
    add r3, r1, r2, LSL #2  /* r3 <- r1 + (r2*4) */
```
> r3는 배열 a의 base주소인 r1에서부터 4byte씩 증가



## 배열 초기화: Indexing mode를 사용한 

ARM instruction set은 위의 asm코드 보다 compact한 방법을 제공한다!  
진정한 의미의 indexing mode임  

## ARM이 제공하는 9개의 인덱싱 모드 형태

```
1) [Rsource1, #+immediate]
2) [Rsource1, +Rsource2]
3) [Rsource1, +Rsource2, shift_operation #immediate]

4) [Rsource1], #+immediate 
5) [Rsource1], +Rsource2
6) [Rsource1], +Rsource2, shift_operation #immediate

7) [Rsource1, #+immediate]!
8) [Rsource1, +Rsource2]!
9) [Rsource1, +Rsource2, shift_operation #immediate]!
```



## 1. Non updating indexing modes

#### (1) [Rsource1, #+immediate] or [Rsource1, #-immediate]

```asm
mov r2, #3          /* r2 ← 3 */
str r2, [r1, #+12]  /* *(r1 + 12) ← r2 */
```
> 배열 a의 base 주소 r1으로부터 인덱스[3]에 12byte 직접 접근   
> immediate 크기는 12비트를 초과할 수 없다.    
  
#### (2) [Rsource1, +Rsource2] or [Rsource1, -Rsource2]  
  
```asm  
mov r2, #3          /* r2 ← 3 */  
mov r3, #12         /* r3 ← 12 */  
str r2, [r1, +r3]   /* *(r1 + r3) ← r2 */  
```  
> offset을 값대신 reg에 있는 값 사용.  
> offset 값이 큰 경우 유용  
  
#### (3) [Rsource1, +Rsource2, shift_operation #immediate]  
			또는 [Rsource1, -Rsource2, shift_operation #immediate]  
  
```asm  
add r3, r1, r2, LSL #2  /* r3 ← r1 + r2*4 */  
str r2, [r3]            /* *r3 ← r2 */  
```  
> 이 방법이 아래와 같은 indexing mode로 compact 해짐  
```asm  
str r2, [r1, +r2, LSL #2]  /* *(r1 + r2*4) ← r2 */  
```  
> r1 base주소에 r2 x 4 에 해당하는 index에 r2 값 store  
> 주소값을 고정된 상수로 곱할 때 유용  
  
  
## 2. Updating indexing modes  
  
순차적으로 접근한다면 매번 base주소에 offset을 계산할 필요가 없다.  
no need to compute everytime from the beginning the address of the next item   
  
  
### a) Post-indexing modes  
  
#### (4) [Rsource1], #+immediate 또는 [Rsource1], #-immediate  
  
```asm  
loop:  
    cmp r2, #100            /* Have we reached 100 yet? */  
    beq end                 /* If so, leave the loop, otherwise continue */  
    str r2, [r1], #4        /* *r1 ← r2 then r1 ← r1 + 4 */  
    add r2, r2, #1          /* r2 ← r2 + 1 */  
    b loop                  /* Go to the beginning of the loop */  
end:  
```  
> base 주소 r1자체가 4씩 증가되고 값이 저장됨  
> 먼저 r2를 a[r1]에 저장한 뒤, r1+=4 함.  
> `#`뒤에 +는 생략가능?  
  
#### (5) [Rsource1], +Rsource2 또는 [Rsource1], -Rsource2  
(4) 와 동일 #imediate 대신 reg사용   
  
#### (6) [Rsource1], +Rsource2, shift_operation #immediate  
			또는 [Rsource1], -Rsource2, shift_operation #immediate   
(4) 와 동일 #imediate 대신 reg에 shift 연산 사용  
		  
  
### b) Pre-indexing modes  
`!` 사용  
  
#### (7) [Rsource1, #+immediate]! 또는 [Rsource1, #-immediate]!  
  
```asm  
ldr r2, [r1, #+12]!  /* r1 ← r1 + 12 then r2 ← *r1 */  
add r2, r2, r2       /* r2 ← r2 + r2 */  
str r2, [r1]         /* *r1 ← r2 *  
```  
> a[3] = a[3] + a[3]   
> (1) 과 유사해 보임. r1이 저장된다는점이 다름.  
> (4) 와 다른점은 미리 인덱싱을 한뒤 값을 저장한다는 점.  
> []`!` 를 붙이면 Rsrc1이 저장됨 Updated됨.  
  
  
#### (8) [Rsource1, +Rsource2]! 또는 [Rsource1, +Rsource2]!  
(7) 과 동일 #imediate 대신 reg사용   
  
#### (9) [Rsource1, +Rsource2, shift_operation #immediate]!  
			또는 [Rsource1, -Rsource2, shift_operation #immediate]!  
  
(7) 과 동일 #imediate 대신 reg에 shift 연산 사용  
  
  
  
## pre-indexing 과 post-indexing 차이 확인 (feat. 'sp' operation)  
  
```asm  
str lr, [sp, #-8]!  /* Pre-index: sp ←  sp - 8; *sp ←  lr */  
... // Code of the function  
ldr lr, [sp], #+8   /* Post-index; lr ←  *sp; sp ←  sp + 8 */  
bx lr  
```  
> `sp`를 8바이트 확장 시켜놓고 ->  `sp`에 `lr`값을 저장한다. (pre-index; 미리 인덱싱)  
> `lr`을 `*sp` 로부터 복구 시켜놓은 뒤 ->  `sp`를 다시 8byte 줄인다. (post-index)  
  
  
## 구조체 초기화 with 인덱싱 모드  
  
  
  
  
```asm  
b.f1 = b.f1 + 7;  
```  
> in C code  
  
```asm  
ldr r2, [r1, #+4]!  /* r1 ← r1 + 4 then r2 ← *r1 */  
add r2, r2, #7     /* r2 ← r2 + 7 */  
str r2, [r1]       /* *r1 ← r2 */  
```  
> in asm code  
> r1 은 +4byte만큼 증가되어 저장됨.  
> 따라서 r2를 r1에 str 할때 index를 다시 계산 할 필요없음.  
  
  
  
  
  
  
  
  
  
  
