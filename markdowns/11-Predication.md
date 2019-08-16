
# 11. Predication  
----

<!-- toc -->

`NOTE` Prediction 아님 Predic__a__tion임 주의 (Not to be confused with branch prediction.)

가용 자원이 부족한 임베디드 시스템에서는 코드 사이즈를 줄일 필요가 있는데 이때 Predication을 사용하면 좋다.  
ARM 명령어에는 Predication 기능이 있다. 아얘 분기를 없애는 방법이다.  Predication을 이용하면 분기를 줄여 성능향상에 도움이 된다.  
ARM에서 Predication을 사용하는 방법은 매우 간단하다. 명령어에 eq, neq, le,lt,ge,gt등의 suffix를 붙여서 사용하면 된다. 이를 조건부실행 명령이라고도 부른다.  아래 예제를 보면 바로 이해가 된다.  
  
### (참고) 다른 주제이지만 잠시 Branch Prediction 에대해 살펴보자  

문제: 파이프 라인에서 분기 조건에따라, 느려지는 문제가 발생한다.  
  
(어떤 문제가 발생하는지는 다음 내용파악.)  
https://everylwn.blogspot.kr/2016/12/branch-prediction-logic-in-arm.html  
branch 명령어를 fetch 한후, processor는 다음 명령어를 가져와야함.  
그런데 문제는 분기문 이기 때문에 if(참)일때 명령 if(거짓)일때 명령,  2개의 브랜치가 다음 명령어가 될 수 있다는 점이다. 이런 branching 명령어가 파이프라인의 끝에 갈때까지 어떤 명령어가 next명령어인지 확신 할 수 없다.  
Branching 명령어가 완전히 수행될 때까지 파이프라이닝을 지연시키는 대신에 예측을 한다. 그런다음 그 예상되는 명령어를 fetch 할수 있고, 예측이 잘못되면 폐기한다.   
likely()와 unlikely() 매크로가 바로 branch prediction을 위해 사용하는 리눅스 커널 매크로인 것이다.   
  
### (참고) ARM의 파이프라인  
  
ARM7 core 에서는 파이프라인이 3가지다. Fetch/Decode/Execute  
`mov r0 r1` `add r0 r1 #3` 같이 레지스터간 데이터 이동명령은 ARM core내에서 동작하기 때문에 실행 속도가 매우 빠르다. 반면 `ldr r3, 0x4CC` `str r3, 0x455` 같이 외부 BUS를 통해 접근해야하는 명령은 상대적으로 속도가 느리기 때문에 파이프라인 단계도 추가된다.  
ARM7 core에는 ldr/str을 위해 메모리에서 데이터를 읽는Memory stage와  
레지스터에 쓰기를 하는 Write stage라는 단계가 존재한다.   
하지만 ARM7 core자체는 3단계의 파이프 라인만 있기 때문에 이 M과W가 수행되는동안에 다른 파이프라인은 stall된다. 그림참고  
  
그래서 ARM9 core는 애초에 파이프라인이 5단계다. F/D/E/M/W   
중간에 ldr/str명령이 있어도 stall되지 않음  
  
ARM11 core부터는 8단계이다.  
Fetch stage 2단, Decode Stage, Issue Stage, Execute Stage 4단  
ARM11에서는 pipeline 효율을 높이기 위한 Program Flow Prediction 기능이 있다. 명령의 흐름을 예측하여 pipeline의 stall 현상을 줄여 효율성을 높여 주는 역할  
  
http://recipes.egloos.com/5643663  
  
## Predication 예제  

다시 Predication으로 돌아와서 예제를 통해 바로 이해하자.  
  
- 이전 콜라츠 추측 collatz.s 예제
  
	소스 코드의 일부분 (전체코드는 #5장 참고)  
	  
	```asm  
	    and r2, r1, #1             /* r2 ←  r1 & 1 */  
	    cmp r2, #0                 /* compare r2 and 0 */  
	    bne collatz_odd            /* if r2 != 0 (this is r1 % 2 != 0) branch to collatz_odd */  
	  collatz_even:  
	    mov r1, r1, ASR #1         /* r1 ←  r1 >> 1. This is r1 ←  r1/2 */  
	    b collatz_end_loop         /* branch to collatz_end_loop */  
	  collatz_odd:  
	    add r1, r1, r1, LSL #1     /* r1 ←  r1 + (r1 << 1). This is r1 ←  3*r1 */  
	    add r1, r1, #1             /* r1 ←  r1 + 1. */  
	  collatz_end_loop:  
	```  
	> Original code: has many branching instructions.  
	> if/else label이동하는 분기가 많다.  
	  
	```asm  
	    cmp r2, #0                 /* compare r2 and 0 */  
	    moveq r1, r1, ASR #1       /* if r2 == 0, r1 ←  r1 >> 1. This is r1 ←  r1/2 */  
	    addne r1, r1, r1, LSL #1   /* if r2 != 0, r1 ←  r1 + (r1 << 1). This is r1 ←  3*r1 */  
	    addne r1, r1, #1           /* if r2 != 0, r1 ←  r1 + 1. */  
	```  
	> Predication 이 포함된 코드  
	> moveq, addne 로 분기를 없앴다.  
  
  
- 위 두 차이를 perf로 비교했을때, 아래와 같이 유의미한 성능 향상이 있다.  
  
	```  
	       3359,953200 cpu-clock                  ( +-  0,01% )  
	       3,365263737 seconds time elapsed      
	```  
	> 분기 사용 코드  
	  
	```  
	       2318,217200 cpu-clock                  ( +-  0,01% )  
	       2,322732232 seconds time elapsed           
	```  
	> Predication 사용 코드  
  
  
  
  
