
# 27. dynamic-linking  
----

dynamic-linking이란?  프로그램이 수행중(run)인 상태에서 obj Linking 하는것.  


26장에서 살펴봤던것 처럼 linking process 단계에서 몇 obj를 합쳐서 최종 하나의 binary파일을 만드는 것이다. 그러나 대개에 많은 프로그램들은 재 사용가능한 작은 단위로 존재한다. 이런것들의 집합을 Library라고 부른다.  UNIX는 전통적으로 이것을 Archives 라고 하고 일반적으로는  __Static Library__ 라고 부른다.  

## Dynamic linking

linking time에 프로그램 조각들을 합치는 것이아니라, running time에 합친다면 어떨까? __Dynamic Library__를 사용하면 동적링킹(Dynamic linking)이 가능하다. 

Dynamic linking을 사용하려면 기존의 linking보다 약간의 노력이 더 필요하다. 먼저 loading 에 대해 알아보자.  


## Loading a program

Loading이란? 실행가능한(executable) 프로그램을 메모리에 위치시키는것을 말한다. 운영체제의 목적 중 하나가 이렇게 프로그램 loading 시키는 것이다.    

앞장에서 살펴본 another_var result_var그리고 함수 inc_result는 Linking 프로세스를 거친뒤에 hard-coded 된 주소값을 갖게 된다. loader의 동작은 간단하다. 이 executable binary를 메모리에 copy 하는 것이다. Linking 단계에서 이미 주소값이 수정되었기 때문에, 정확한 메모리 주소에 copy된다. 

현대의 운영체제는 virtual memory 주소 체계를 사용한다. 그래서 각 프로세스는 일관성있는 메모리 주소값을 접근하면 된다. 그래서 로딩되는 프로그램은 동일한 virtual address에 로딩된다. 운영체제는 각각의 running program(process) 가 가지는 virtual memory를 알아서 실제 물리주소에 mapping한다.  

Dynamic Library를 run time에 Linking하기 위해서는 또다른 프로그램 __Dynamic Linker__ 가 필요하다. Dynamic Linker는 Dynamic Library의 코드 데이터를 로딩하고 relocation 작업을 수행한다. relocation작업은 position이 독립적인지 종속적인지에 따라 차이가 있다.


#### Position independent code (PIC)

Code는 position dependent 또는 position independent 할 수 있다. 전자는 absolute address를 직접 사용한다. 후자는 상대 주소를 사용한다. 실행중에 이 상대 주소가 convert 된다. 이것은 절대주소에서 추가 computation이 필요하다는 의미다. ELF에서 이 메커니즘은 Global Offset Table(GOT)을 사용한다. 
모든 프로그램과 Dynamic 라이브러리는 각자가 고유한 GOT를 가지고 있다. 이것을 다른곳과 공유하지도 않는다. GOT는 절대주소를 사용하지 않고도 접근할 수 있는 메모리에 위치해 있다. 이를 위해서 PC + 상대offset 방식의 주소체계가 반드시 사용되어야 한다. 따라서 GOT는 어떤 고정된 공간에 위치해 있고 정적 link time에 계산될 수 있다. 

position independent 는 코드가 로딩타임에 relocation 될 필요가 없다는 장점이 있다. 
오직 GOT만 relocation 되면된다. 코드가 동적으로 로딩될때 말이다. 이것은 로딩시간을 극적으로 개선시킬 수 있다. 

GOT?  
 The mechanism used in ELF uses a table called the __Global Offset Table__. This table contains entries, one entry per global entity we want to access. Each entry, at run time, will hold the absolute address of the object. Every program and dynamic library has its own GOT, not shared with anyone else.

이런 일련의 과정은 운영체제가 virtual memory를 사용함으로써 가능하다. dynamic lib는 각각의 프로세스가 공유할 수 있다. (추가적인 물리 메모리할당이 필요없다는 뜻) 

단점은 GOT때문에 발생한다. 전역변수를 접근하는게 복잡해진다는 것이다. 


## Accessing a global variable

전역변수를 access하는게 얼마나 복잡한지 예제를 살펴보자.  
간단한 예제를 소개한다. 전역변수 값을 증가시킬 것인데, 이 전역변수는 Library에서 제공하는 것이다.  
Static library 방식과 Dynamic library 방식 두가지를 확인해보자.  

#### Static library

```asm
/* mylib.s */
.data
 
.balign 4
.globl myvar
myvar : .word 42     /* 42 as initial value */
.size myvar, .-myvar
```
> mayvar 라는 전역변수를 선언햇다.  
> .size 는 dynamic library를 사용할때 필요한 지시자이다.  
> `.-myvar` : current addreaa(.) 

그리고 이 라이브러리를 사용하는 메인함수는 아래와 같다.  
특별할것 없는 예 였다.


```asm
/* main.s */
.data
.globl myvar
 
.text
.globl main
 
.balign 4
main:
    ldr r0, addr_myvar  /* r0 ←  &myvar */
    ldr r1, [r0]        /* r1 ←  *r0 */
    add r1, r1, #1      /* r1 ←  r1 + 1 */
    str r1, [r0]        /* *r0 ←  r1 */
 
    mov r0, #0          /* end as usual */
    bx lr
 
addr_myvar: .word myvar
```

어셈블러로 컴파일, static lib (.a) 을 생성한뒤  한뒤 link를 해보자.  

```asm
# (static) library
as -o mylib.o mylib.s
ar cru mylib.a mylib.o
# program
as -o main.o main.s
gcc -o main main.o -L. -l:mylib.a
```
> `ar`, the archiver: is the tool that creates a static library, an .a file

Static library 방식은 특별할것이 없었다.  

#### Dynamic library

이제 Dynamic library로 만들어본다. Dynamic library를 생성하기 위해서는 Linker에게 dynamic lib을 사용할것이라고 명시해야한다. ELF에서  Dynamic library 는 __Shared Object (.so)__ 라고 불린다. 

```asm
# dynamic library
as -o mylib.o mylib.s
gcc -shared -o mylib.so mylib.o
```
> mylib.so 생성

이제 우리 프로그램이 position independent 한 방법으로 변수 myvar을 access하게 만들어보자. 
사실,  PIC는 오직 dynamic lib을 위해서만 존재한다. 

PIC 접근방식은 pc-relative 하게 주소를 사용한다. 그러기 위해서는 GOT필요.
아래는 static linker로부터 GOT의 정확확한 offset을 얻어올수 있다.

```asm
...
.word _GLOBAL_OFFSET_TABLE_
...
```

이상적이라면 아래와 같이 할수 있어야하나, 불가능하다. 

```asm
add r0, pc, .word _GLOBAL_OFFSET_TABLE_  /* r0 ←  pc + "offset-to-GOT" */
```

그래서 아래와 같은 기존의 방식대로 한다. 

```asm
ldr r0, offset_of_GOT    /* r0 ←  "offset-to-GOT" */
add r0, pc, r0           /* r0 ←  pc + r0 */ 
...
offset_of_GOT: .word _GLOBAL_OFFSET_TABLE_
```

But that "offset-to-got" must be the offset to the GOT at the point where we are actually adding the pc, this is, in the second instruction. This means that we need to ask the linker to adjust it so the offset make sense for the instruction that adds that offset to the pc. We can do this using an additional label.

```asm
ldr r0, offset_of_GOT           /* r0 ←  "offset-to-GOT" */
got_address: add r0, pc, r0     /* r0 ←  pc + r0 */ 
...
offset_of_GOT: .word _GLOBAL_OFFSET_TABLE_  - got_address
```

ARM의 pc reading 특성을 고려해서 최종적으로 아래와 같이 하면 된다. (사실 이해 잘 못함)

```asm
ldr r0, offset_of_GOT           /* r0 ←  "offset-to-GOT" */
got_address: add r0, pc, r0     /* r0 ←  pc + r0 */ 
...
offset_of_GOT: .word _GLOBAL_OFFSET_TABLE_  - (got_address + 8)
```

이제 r0에는 GOT의 절대 주소가 들어있다. 그러나 우리는 myvar 변수를 access 해야한다. 어떻게해야할까? static linker에게 해당심볼을 위해 GOT내의 offset을 사용하라고 아래와같은 방식으로 알려줘야한다. 

```asm
.word myvar(GOT)
```

이제 position independent한 방법으로 myval에 access 해보자 

![](https://thinkingeek.com/wp-content/uploads/2017/04/got.svg_.png)


```asm
/* main.s */
.data
 
.text
.globl main
 
.balign 4
 
main:
  ldr r0, offset_of_GOT        /* r0 ←  offset-to-GOT
                                  (respect to got_address)*/
  got_address: add r0, pc, r0  /* r0 ←  pc + r0 
                                  this is
                                    r0 ←  &GOT */
  ldr r1, myvar_in_GOT         /* r1 ←  offset-of-myvar-inside-GOT */
  add r0, r0, r1               /* r0 ←  r0 + r1 
  ^^^^^^^^^^^^^^                  this is r0 ←  &GOT + offset-of-myvar-inside-GOT */
  ldr r0, [r0]                 /* r0 ←  *r0
  ^^^^^^^^^^^^                    this is r0 ←  &myvar
                                */
  ldr r1, [r0]                  /* r0 ←  *r1 */
  add r1, r1, #1                /* r1 ←  r1 + 1 */
  str r1, [r0]                  /* *r0 ←  r1 */
 
  mov r0, #0          /* end as usual */
  bx lr
 
offset_of_GOT: .word _GLOBAL_OFFSET_TABLE_  - (got_address + 8)
myvar_in_GOT : .word myvar(GOT)
```
> 밑줄처럼 r0에 GOT내부 오프셋으로 얻어낸 myval 주소가 담겨있다. 

참고로 위 두 밑줄을 아래와 같이 사용할 수 도있다. 

```asm
  ldr r0, [r0, r1]     /* r0 ←  *(r0 + r1)
                          this is r0 ←  *(&GOT + offset-of-my-var-inside-GOT) */
```

이제 build 해보자 

```asm
# program
as -o main.o main.s
gcc -o main main.o -L. -l:mylib.so -Wl,-rpath,$(pwd)
```
> `-Wl`, `-rpath,$(pwd)` option tells the dynamic linker to use the current directory, $(pwd), to find the library. 


## Calling a function

Dynamic lib에서 함수를 호출 하는것은 GOT에서 access하는것 보다 약간 더 복잡하다(more involved). ELF 에서 Lazy Binding 이라고 불리는 기능 때문이다. 함수는 게으른 방식으로 로드 될 수 있다. 그 이유는 라이브러리는 정말 많은 함수를 제공하지만 실행중인 프로그램은 그 중 매우 적은 함수만 요구하기 때문이다. 만약 static linking을 사용한다면 거의 문제가 없다.  왜냐하면 링커는 프로그램에서 잠재적으로 사용하는 심볼을 정의하는 객체 파일 만 사용하기 때문이다. 그러나 동적 라이브러리는 전체적으로 처리해야하므로이 작업을 수행 할 수 없다.  

따라서 Lazy loading에서는 미리 하는것이 아니라, 함수를 처음 호출할 때 로드 해야한다. 이 방식은 효율적이지만 약간의 수고가 필요하다. ELF에서는 Procedure Linkage Table(PLT)을 사용하여 이 작업을 수행한다. 테이블의 항목에는 프로그램에 의해  잠재적으로 사용 될 함수들이 있다. 이 항목들은 GOT에도 복사 된다.  GOT와 달리 PLT는 코드이므로 PLT를 수정하지는 않는다. PLT의 항목은 GOT의 항목으로 분기하는 작은 일련의 명령어이다.  

함수에 대한 GOT 엔트리는 동적 링커에 의해 함수의 주소를 검색하는 동적 링커의 내부 함수에 대한 주소로 초기화되고, GOT를 해당 주소로 업데이트하고 분기한다. 동적 링커가 GOT 테이블을 업데이트했기 때문에, 다음번 호출이 PLT를 통해 일어났을때, 다음 호출 (단순히 GOT으로 분기 함)이 함수로 직접 이동한다.   

그렇다면 왜 직접 GOT주소에 접근하지 않고 PLT를 거치는가? 이유는 dynamic linker는 반드시 처음으로 로드되어야 할 함수가 어떤 함수인지 알아야 하기 때문이다.  GOT에서 직접 주소를 호출하면 어떤 함수를로드해야 하는지를 알 수있는 메커니즘을 따로 고안해야한다.  동적 로더가 로드해야하는 정확한 함수를 알고 있도록 모든 것을 준비하는 테이블에 함수에 대한 GOT 항목을 초기화하는 방법이있을 수 있다. 그러나 이것은 실제로 PLT와 동일하다!  

복잡해보이지만 다행이도, PLT 항목을 만드는것은 우리가 아니라 링커이며, 그 PLT항목은 일반 함수 호출로 사용될 수 있다는것이다.  GOT 항목에 대한 주소를 얻어올 필요가 없으며, 변수에 대한 모든 작업이 필요가 없다.  (함수의 주소를 사용하려면이 작업을 수행해야한다!).  우리는 항상 그렇게 할 수는 있지만 모든 함수 호출이 GOT 테이블에서 복잡한 색인을 요구하기 때문에 코드가 늘어나게 될 것이다.  이 메커니즘은 PIC와 non-PIC 모두에서 작동하며, 어떤 동적 라이브러리를 사용할지 여부를 염려 할 필요없이, 우리가 printf와 같은 C 라이브러리 함수를 호출 할 수 있었던 이유이다.  즉, PLT를 통해 함수를 호출하고자 함을 명시하기 위해 접미사 __@PLT__를 명시 적으로 사용할 수 있다. 이것은 라이브러리 내에서 수행되는 호출에 대해 필수이다.  

- __결론__: PIC code를 이용하는 동적라이브러리내의 
변수: __GOT__ 통해 access  
함수: __PLT__ 통해 access  

### Complete example

우리가 위에서 만든 라이브러리에 myvar 값을 출력하는 함수도 추가해보자. 이 코드는 라이브러리내에 있으므로, 변수를 GOT를 통해 접근하고 PLT를 통해 함수를 접근하는 PIC code이어야 한다. 
변수를 increment하는것 말고는 이전예제와 동일 


```asm
/* mylib.s */
.data
 
.balign 4
.globl myvar
myvar : .word 42 /* global variable "myvar" */
.size myvar, .-myvar
 
message: .asciz "Value of 'myvar' is %d\n"
 
.text
 
.balign 4
.globl myfun
myfun:
  push {r4, lr}                /* we are going to do a
                                  call so keep lr, and also r4
                                  for a 8-byte aligned stack */
  ldr r0, offset_of_GOT        /* r0 ←  offset-to-GOT
                                  (respect to got_address)*/
  got_address: add r0, pc, r0  /* r0 ←  pc + r0 
                                  this is r0 ←  &GOT */
  ldr r1, myvar_in_GOT         /* r1 ←  offset-of-myvar-inside-GOT */
  ldr r0, [r0, r1]             /* r0 ←  *(r0 + r1)
                                  this is r0 ←  *(&GOT + offset-of-myvar-inside-GOT) */
  ldr r1, [r0]                 /* r0 ←  *r1 */
 
  ldr r0, addr_of_message      /* r0 ←  &message */
  /* r1 already contains the value we want */
  bl printf@PLT                /* call to printf via the PLT */
  ^^^^^^^^^^^^^
  pop {r4, lr}                 /* restore registers */
  bx lr

offset_of_GOT: .word _GLOBAL_OFFSET_TABLE_  - (got_address + 8)
myvar_in_GOT : .word myvar(GOT)
addr_of_message: .word message
```
> myvar변수를 접근하는것은 GOT를 이용하는것은 이전 main함수의 방식과 동일하다.  

그리고 라이브러리내의 myval변수를 ++하는 myfun 함수를 호출할 수 있도록 main함수를 수정해보자.   

```asm
/* main.s */
.data
 
.text
.globl main
 
.balign 4
 
main:
  push {r4, lr}                /* we are going to do a
                                  call so keep lr, and also r4
                                  for a 8-byte aligned stack */
  bl myfun@PLT                 /* call function in library */
 
  ldr r0, offset_of_GOT        /* r0 ←  offset-to-GOT
                                  (respect to got_address)*/
  got_address: add r0, pc, r0  /* r0 ←  pc + r0 
                                  this is
                                    r0 ←  &GOT */
  ldr r1, myvar_in_GOT         /* r1 ←  offset-of-myvar-inside-GOT */
  ldr r0, [r0, r1]             /* r0 ←  *(r0 + r1)
                                  this is
                                    r0 ←  *(&GOT + offset-of-myvar-inside-GOT) */
  ldr r1, [r0]                 /* r0 ←  *r1 */
  add r1, r1, #1               /* r1 ←  r1 + 1 */
  str r1, [r0]                 /* *r0 ←  r1 */
 
  bl myfun@PLT                 /* call function in library a second time */
 
  pop {r4, lr}                 /* restore registers */
  mov r0, #0                   /* end as usual */
  bx lr
 
offset_of_GOT: .word _GLOBAL_OFFSET_TABLE_  - (got_address + 8)
myvar_in_GOT : .word myvar(GOT)
```

그리고 dynamic lib과 함께 빌드!  

```bash
# Dynamic library
 $ as -o mylib.o mylib.s
 $ gcc -shared -o mylib.so mylib.o
# Program
 $ as -o main.o main.s
 $ gcc -o main main.o -L. -l:mylib.so -Wl,-rpath,$(pwd)
```
ldd 명령을 사용해서 어떤 동적라이브러리가 사용되었는지 확인이 가능하다. 
ldd - print shared library dependencies

```asm
$ ldd main
	/usr/lib/arm-linux-gnueabihf/libcofi_rpi.so (0xb6f3b000)
	mylib.so => $(pwd)/mylib.so (0xb6f32000)
	libc.so.6 => /lib/arm-linux-gnueabihf/libc.so.6 (0xb6df0000)
	/lib/ld-linux-armhf.so.3 (0x7f5cb000)
```

그리고 실행해보면 잘 동작한다! 


```asm
$ ./main 
Value of 'myvar' is 42
Value of 'myvar' is 43
```

