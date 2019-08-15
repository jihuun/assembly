
# 25. Integer SIMD
----

13장에서 VFPv2 와 floating point number를 vector operation 할 수 있음을 확인했다. integer number도 벡터연산이 가능한가? 이장에서는 그것을 확인한다.  

## SIMD

SIMD는 문자 그대로 __S__ingle __I__nstruction 으로 한번에 __M__ultible __D__ata를 처리한다는 의미이다.  
#13 #14장에서 

`참고: NEON`

Arm NEON technology is an __advanced SIMD__ (single instruction multiple data) architecture extension for the Arm Cortex-A series and Cortex-R52 processors.  
NEON technology was introduced to the __Armv7-A__ and __Armv7-R__ profiles. It is also now an extension to the __Armv8-A__ and __Armv8-R__ profiles.  
NEON technology is intended to improve the __multimedia__ user experience by accelerating __audio__ and __video encoding/decoding__, user interface, __2D/3D graphics or gaming__. NEON can also accelerate __signal processing algorithms__ and functions to speed up applications such as audio and video processing, voice and facial __recognition__, computer __vision__ and __deep learning__.  


## Motivating example

이러한 기능이 왜 필요한지 다음의 예제로 살펴보자. 먼저 16bit PCM audio signal 이 샘플링 되어있다고 하자. 그리고 다음의 코드처럼 그 중 두개의 signal의 평균을 구한다고 해보자.  

```c
short int channel1[num_samples]; // in our environment a 'short int' is a half-word
short int channel2[num_samples];
 
short int channel_out[num_samples];
for (i = 0; i < num_samples; i++)
{
   channel_out[i] = (channel1[i] + channel2[i]) / 2;
}
```
> short int 즉 2byte 사용 예제임을 주의  

그리고 assembly 명령어로는 다음과 같이 번역될 수 있다.  

```asm
naive_channel_mixing:
    /* r0 contains the base address of channel1 */
    /* r1 contains the base address of channel2 */
    /* r2 contains the base address of channel_out */
    /* r3 is the number of samples */
    /* r4 is the number of the current sample
          so it holds that 0 ←  r4 < r3 */
 
    mov r4, #0              /* r4 ←  0 */
    b .Lcheck_loop          /* branch to check_loop */
    .Lloop:
      mov r5, r4, LSL #1    /* r5 ←  r4 << 1 (this is r5 ←  r4 * 2) */
                            /* a halfword takes two bytes, so multiply
                               the index by two. We do this here because
                               ldrsh does not allow an addressing mode
                               like [r0, r5, LSL #1] */
      ldrsh r6, [r0, r5]    /* r6 ←  *{signed half}(r0 + r5) */
      ldrsh r7, [r1, r5]    /* r7 ←  *{signed half}(r1 + r5) */
      add r8, r6, r7        /* r8 ←  r6 + r7 */
      mov r8, r8, ASR #1    /* r8 ←  r8 >> 1 (this is r8 ←  r8 / 2)*/
      strh r8, [r2, r5]     /* *{half}(r2 + r5) ←  r8 */
      add r4, r4, #1        /* r4 ←  r4 + 1 */
    .Lcheck_loop: 
      cmp r4, r3            /* compute r4 - r3 and update cpsr */
      blt .Lloop            /* if r4 < r3 jump to the
                               beginning of the loop */
```
> 이 코드는 우리가 익숙해서 좋지만, 다음 장 처럼 병렬연산으로 개선 해야한다.    
> 참고, `sh`? signed half, 2byte를 사용한다.  [21장 Subword data참고](#21. Subword data)    


## 병렬 연산 (더하기, 빼기)

통째로 더하거나 빼기 위해서는 아래와 같은 명령을 사용한다.  

- Halfwords
	* Signed: sadd16, ssub16
	* Unsigned: uadd16, usub16
- Bytes
	* Signed: sadd8, ssub8
	* Unsigned: uadd8, usub8

다음의 C 코드가 있다고 하자.  

```c
// unsigned char is an unsigned byte in our environment
// a, b and c are arrays of N unsigned chars
unsigned char a[N], b[N], c[N];
 
int i;
for (i = 0; i < N; i++)
{
    c[i] = a[i] + b[i];
}
```

이것을 순수하게 loop를 이용해 더하면 아래와 같다.  

```asm
naive_byte_array_addition:
    /* r0 contains the base address of a */
    /* r1 contains the base address of b */
    /* r2 contains the base address of c */
    /* r3 is N */
    /* r4 is the number of the current item
          so it holds that 0 ←  r4 < r3 */
 
    mov r4, #0             /* r4 ←  0 */
    b .Lcheck_loop0        /* branch to check_loop0 */
 
    .Lloop0:
      ldrb r5, [r0, r4]    /* r5 ←  *{unsigned byte}(r0 + r4) */
      ldrb r6, [r1, r4]    /* r6 ←  *{unsigned byte}(r1 + r4) */
      add r7, r5, r6       /* r7 ←  r5 + r6 */
      strb r7, [r2, r4]    /* *{unsigned byte}(r2 + r4) ←  r7 */
      add r4, r4, #1       /* r4 ←  r4 + 1 */
                 ^^^^
    .Lcheck_loop0:
       cmp r4, r3          /* perform r4 - r3 and update cpsr */
       blt .Lloop0         /* if cpsr means that r4 < r3 jump to loop0 */
```

sadd8 을 이용하기

```asm
simd_byte_array_addition_0:
    /* r0 contains the base address of a */
    /* r1 contains the base address of b */
    /* r2 contains the base address of c */
    /* r3 is N */
    /* r4 is the number of the current item
          so it holds that 0 ←  r4 < r3 */
 
    mov r4, #0             /* r4 ←  0 */
    b .Lcheck_loop1        /* branch to check_loop1 */
 
    .Lloop1:
      ldr r5, [r0, r4]     /* r5 ←  *(r0 + r4) */
      ldr r6, [r1, r4]     /* r6 ←  *(r1 + r4) */
      sadd8 r7, r5, r6     /* r7[7:0] ←  r5[7:0] + r6[7:0] */
      ^^^^^^^^^^^^^^^^     /* r7[15:8] ←  r5[15:8] + r6[15:8] */
                           /* r7[23:16] ←  r5[23:16] + r6[23:16] */
                           /* r7[31:24] ←  r5[31:24] + r6[31:24] */
                           /* rA[x:y] means bits x to y of the register rA */
      str r7, [r2, r4]     /* *(r2 + r4) ←  r7 */
      add r4, r4, #4       /* r4 ←  r4 + 4 */
                 ^^^^
    .Lcheck_loop1:
       cmp r4, r3          /* perform r4 - r3 and update cpsr */
       blt .Lloop1         /* if cpsr means that r4 < r3 jump to loop1 */
```
> sadd8 을 이용하면 4byte씩 한번에 더한다.  

하지만 이 예제는 N이 4의 배수일때만 사용된다. N이 4의 배수가 아닌경우는 다음과 같이한다. 

```asm
simd_byte_array_addition_2:
    /* r0 contains the base address of a */
    /* r1 contains the base address of b */
    /* r2 contains the base address of c */
    /* r3 is N */
    /* r4 is the number of the current item
          so it holds that 0 ←  r4 < r3 */
 
    mov r4, #0             /* r4 ←  0 */
    sub r8, r3, #3         /* r8 ←  r3 - 3
    ^^^^^^^^^^^^^^            this is r8 ←  N - 3 */
    b .Lcheck_loop2        /* branch to check_loop2 */
 
    .Lloop2:
      ldr r5, [r0, r4]     /* r5 ←  *(r0 + r4) */
      ldr r6, [r1, r4]     /* r6 ←  *(r1 + r4) */
      sadd8 r7, r5, r6     /* r7[7:0] ←  r5[7:0] + r6[7:0] */
                           /* r7[15:8] ←  r5[15:8] + r6[15:8] */
                           /* r7[23:16] ←  r5[23:16] + r6[23:16] */
                           /* r7[31:24] ←  r5[31:24] + r6[31:24] */
      str r7, [r2, r4]     /* *(r2 + r4) ←  r7 */
      add r4, r4, #4       /* r4 ←  r4 + 4 */
    .Lcheck_loop2:
       cmp r4, r8          /* perform r4 - r8 and update cpsr */
       blt .Lloop2         /* if cpsr means that r4 < r8 jump to loop2 */
                           /* i.e. if r4 < N - 3 jump to loop2 */
```

N - 3으로 비교한뒤 나머지 값들은 다음 코드에서 byte단위로 더해서 마무리한다.  

```asm
     /* epilog loop */
     b .Lcheck_loop3       /* branch to check_loop3 */
 
     .Lloop3:
        ldrb r5, [r0, r4]  /* r5 ←  *{unsigned byte}(r0 + r4) */
        ldrb r6, [r1, r4]  /* r6 ←  *{unsigned byte}(r1 + r4) */
        add r7, r5, r6     /* r7 ←  r5 + r6 */
        strb r7, [r2, r4]  /* *{unsigned byte}(r2 + r4) ←  r7 */
 
        add r4, r4, #1     /* r4 ←  r4 + 1 */
     .Lcheck_loop3:
        cmp r4, r3         /* perform r4 - r3 and update cpsr */
        blt .Lloop3        /* if cpsr means that r4 < r3 jump to loop 3 */
```

## Halving instructions

half word 에 대해서도 다음의 연산이 존재한다.  

- Halfwords
	* Signed: shadd16, shsub16
	* Unsigned: uhadd16, uhsub16
- Bytes
	* Signed: shadd8, shsub8
	* Unsigned: uhadd8, uhsub8

위의 motivation example은 다음과 같이 번역될 수 있다. 이 방식은 loop종료 이후 추가 연산이 필요없다.   

```asm
better_channel_mixing:
    /* r0 contains the base address of channel1 */
    /* r1 contains the base address of channel2 */
    /* r2 contains the base address of channel_out */
    /* r3 is the number of samples */
    /* r4 is the number of the current sample
          so it holds that 0 ←  r4 < r3 */
 
    mov r4, #0              /* r4 ←  0 */
    b .Lcheck_loop          /* branch to check_loop */
    .Lloop:
      ldr r6, [r0, r4]      /* r6 ←  *(r0 + r4) */
      ldr r7, [r1, r4]      /* r7 ←  *(r1 + r4) */
      shadd16 r8, r6, r7    /* r8[15:0] ←  (r6[15:0] + r7[15:0]) >> 1*/
                            /* r8[31:16] ←  (r6[31:16] + r7[31:16]) >> 1*/
      str r8, [r2, r4]      /* *(r2 + r4) ←  r8 */
      add r4, r4, #2        /* r4 ←  r4 + 2 */
    .Lcheck_loop:
      cmp r4, r3            /* compute r4 - r3 and update cpsr */
      blt .Lloop            /* if r4 < r3 jump to the
                               beginning of the loop */
```


## Saturating arithmetic

만약 2byte에서 더한값이 overflow된다면? 그것을 확인하는 동작이 필요하다. 아래와 같은 함수를 이용할 수 있다.  


```asm
.data
max16bit: .word 32767
 
.text
 
clipped_add16bit:
    /* first operand is in r0 */
    /* second operand is in r1 */
    /* result is left in r0 */
    push {r4, lr}             /* keep registers */
 
    ldr r4, addr_of_max16bit  /* r4 ←  &max16bit */
    ldr r4, [r4]              /* r4 ←  *r4 */
                              /* now r4 == 32767 (i.e. 2^15 - 1) */
 
    add r0, r0, r1            /* r0 ←  r0 + r1 */
    cmp r0, r4                /* perform r0 - r4 and update cpsr */
    movgt r0, r4              /* if r0 > r4 then r0 ←  r4 */
    bgt end                   /* if r0 > r4 then branch to end */
 
    mvn r4, r4                /* r4 ←  ~r4
                                 now r4 == -32768 (i.e. -2^15) */
    cmp r0, r4                /* perform r0 - r4 and update cpsr */
    movlt r0, r4              /* if r0 < r4 then r0 ←  r4 */
 
    end:
 
    pop {r4, lr}              /* restore registers */
    bx lr                     /* return */
addr_of_max16bit: .word max16bit
```
> 기존의 명령어로 처리하기 위해서는 상당히 많은 라인이 필요하지만 다행이도 Saturated arithmetics 명령어가 이를 해결해준다.  

- Halfwords
	* Signed: qadd16, qsub16
	* Unsigned: uqadd16, uqsub16
- Bytes
	* Signed: qadd8, qsub8
	* Unsigned: uqadd8, uqsub8

```asm
more_realistic_channel_mixing:
    /* r0 contains the base address of channel1 */
    /* r1 contains the base address of channel2 */
    /* r2 contains the base address of channel_out */
    /* r3 is the number of samples */
    /* r4 is the number of the current sample
          so it holds that 0 ←  r4 < r3 */
 
    mov r4, #0              /* r4 ←  0 */
    b .Lcheck_loop          /* branch to check_loop */
    .Lloop:
      ldr r6, [r0, r4]      /* r6 ←  *(r0 + r4) */
      ldr r7, [r1, r4]      /* r7 ←  *(r1 + r4) */
      qadd16 r8, r6, r7     /* r8[15:0] ←  saturated_sum_16(r6[15:0], r7[15:0]) */
                            /* r8[31:16] ←  saturated_sum_16(r6[31:16], r7[31:16]) */
      str r8, [r2, r4]      /* *(r2 + r4) ←  r8 */
      add r4, r4, #2        /* r4 ←  r4 + 2 */
    .Lcheck_loop:
      cmp r4, r3            /* compute r4 - r3 and update cpsr */
      blt .Lloop            /* if r4 < r3 jump to the
                               beginning of the loop */
```



