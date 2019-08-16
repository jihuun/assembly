
# 20. Indirect calls
----
  
<!-- toc -->


Indirect calls 은 이름(label)을 가지고 함수를 호출하는 것이 아니라, 주소를 이용해 함수를 호출하는 것을 의미한다. 이방법을 사용하면 주소를 직접 계산해서 호출 할수 있고,  배열이나 구조체 클래스 처럼 offset이 있는 함수들의 주소를 쉽게 계산해서 호출할 수 있다.  


## Labels

label은 assembler 가 머신코드로 변환할때 주소로 바꿈.

```asm
fun: /* label 'fun' */
  push {r4, r5}
  ...
  pop {r4, r5}
  bx lr

 ...
  bl fun
```

## Our first indirect call


```asm
.data     /* data section */
...
.align 4  /* ensure the next label is 4-byte aligned */
ptr_of_fun: .word 0   /* we set its initial value zero */
```

```asm
.align 4
make_indirect_call:
    push {r4, lr}            /* keep lr because we call printf, 
                                we keep r4 to keep the stack 8-byte
                                aligned, as per AAPCS requirements */
    ldr r0, addr_ptr_of_fun  /* r0 ←  &ptr_of_fun */
    ldr r0, [r0]             /* r0 ←  *r0 */
    blx r0                   /* indirect call to r0 */
    pop {r4, lr}             /* restore r4 and lr */
    bx lr                    /* return to the caller */
 
addr_ptr_of_fun: .word ptr_of_fun
```
> `blx r0`: `bl label` 대신에 register에 있는 address 로 indirect call을 했다.  
> 이전장에 언급했드시 `bx`는 `lr`값을 자동으로 save하지 않기 때문에  indirect call로 사용할 수 없다.  


```asm
main:
    ...
    ldr r1, addr_say_hello   /* r1 ←  &say_hello */
    ldr r0, addr_ptr_of_fun  /* r0 ←  &addr_ptr_of_fun */
    str r1, [r0]             /* *r0 ←  r1
                                this is
                                ptr_of_fun ←  &say_hello */
    ...
    bl make_indirect_call
```
> 이런식으로 주소를 변경 가능

## 장점?

Being able to call a function indirectly is a very powerful thing. It allows us to keep the address of a function somewhere and call it. It allows us to pass the address of a function to another function.  

아래 코드를 보자. indirect call의 장점을 확인할 수 있다.   



```asm
.data     /* data section */
 
...
.text     /* text section (= code) */
 
say_hello:
    ...중략 print hello 하는 함수
 
say_bonjour:
    ...중략 print bonjour 하는 함수
 
 
.align 4
greeter:
    push {r4, lr}            /* keep lr because we call printf, 
                                we keep r4 to keep the stack 8-byte
                                aligned, as per AAPCS requirements */
    blx r0                   /* indirect call to r0 */
    pop {r4, lr}             /* restore r4 and lr */
    bx lr                    /* return to the caller */
 
.globl main /* state that 'main' label is global */

main:
    push {r4, lr}            /* keep lr because we call printf, 
                                we keep r4 to keep the stack 8-byte
                                aligned, as per AAPCS requirements */
 
    ldr r0, addr_say_hello   /* r0 ←  &say_hello */
    bl greeter               /* call greeter */
 
    ldr r0, addr_say_bonjour /* r0 ←  &say_bonjour */
    bl greeter               /* call greeter */
 
    mov r0, #0               /* return from the program, set error code */
    pop {r4, lr}             /* restore r4 and lr */
    bx lr                    /* return to the caller (the system) */
 
addr_say_hello : .word say_hello
addr_say_bonjour : .word say_bonjour
```
> 길지만 별거 없고, greeter 함수에서 인자로 전달받은 함수 주소를 `blx r0` 로 indirectly 하게 call 할 수 있다는 예제이다.  


## 다음 일련의 예제들을 보면 장점이 더욱 명확해 진다.  

- printf format 문자열

```asm
.data     /* data section */
 
.align 4  /* ensure the next label is 4-byte aligned */
message_hello: .asciz "Hello %s\n"
.align 4  /* ensure the next label is 4-byte aligned */
message_bonjour: .asciz "Bonjour %s\n"
```

- 

```asm
/* tags of kind of people */
.align 4  /* ensure the next label is 4-byte aligned */
person_english : .word say_hello /* tag for people
                                     that will be greeted 
                                     in English */
.align 4  /* ensure the next label is 4-byte aligned */
person_french : .word say_bonjour /* tag for people
                                     that will be greeted 
                                     in French */
```

```asm
/* several names to be used in the people definition */
.align 4
name_pierre: .asciz "Pierre"
.align 4
name_john: .asciz "John"
.align 4
name_sally: .asciz "Sally"
.align 4
name_bernadette: .asciz "Bernadette"
```

```asm
.align 4
person_john: .word name_john, person_english
.align 4
person_pierre: .word name_pierre, person_french
.align 4
person_sally: .word name_sally, person_english
.align 4
person_bernadette: .word name_bernadette, person_french

/* array of people */
people : .word person_john, person_pierre, person_sally, person_bernadette
```

- hello 출력 함수 정의

```asm
.text     /* text section (= code) */
 
.align 4  /* ensure the next label is 4-byte aligned */
say_hello:
    push {r4, lr}            /* keep lr because we call printf, 
                                we keep r4 to keep the stack 8-byte
                                aligned, as per AAPCS requirements */
    /* Prepare the call to printf */
    mov r1, r0               /* r1 ←  r0 */
    ldr r0, addr_of_message_hello
                             /* r0 ←  &message_hello */
    bl printf                /* call printf */
    pop {r4, lr}             /* restore r4 and lr */
    bx lr                    /* return to the caller */
 
.align 4  /* ensure the next label is 4-byte aligned */
addr_of_message_hello: .word message_hello
 
.align 4  /* ensure the next label is 4-byte aligned */
say_bonjour:
    push {r4, lr}            /* keep lr because we call printf, 
                                we keep r4 to keep the stack 8-byte
                                aligned, as per AAPCS requirements */
    /* Prepare the call to printf */
    mov r1, r0               /* r1 ←  r0 */
    ldr r0, addr_of_message_bonjour
                             /* r0 ←  &message_bonjour */
    bl printf                /* call printf */
    pop {r4, lr}             /* restore r4 and lr */
    bx lr                    /* return to the caller */
 
.align 4  /* ensure the next label is 4-byte aligned */
addr_of_message_bonjour: .word message_bonjour
```

- main 함수 

```asm
.globl main /* state that 'main' label is global */
.align 4  /* ensure the next label is 4-byte aligned */
main:
    push {r4, r5, r6, lr}    /* keep callee saved registers that we will modify */
 
    ldr r4, addr_of_people   /* r4 ←  &people */
    /* recall that people is an array of addresses (pointers) to people */
 
    /* now we loop from 0 to 4 */
    mov r5, #0               /* r5 ←  0 */
    b check_loop             /* branch to the loop check */
 
    loop:
      /* prepare the call to greet_person */
      ldr r0, [r4, r5, LSL #2]  /* r0 ←  *(r4 + r5 << 2)   this is
                                   r0 ←  *(r4 + r5 * 4)
                                   recall, people is an array of addresses,
                                   so this is
                                   r0 ←  people[r5]
                                */
      bl greet_person           /* call greet_person */
      add r5, r5, #1            /* r5 ←  r5 + 1 */
    check_loop:
      cmp r5, #4                /* compute r5 - 4 and update cpsr */
      bne loop                  /* if r5 != 4 branch to loop */
 
    mov r0, #0               /* return from the program, set error code */
    pop {r4, r5, r6, lr}     /* callee saved registers */
    bx lr                    /* return to the caller (the system) */
 
addr_of_people : .word people
```
> `ldr r0, [r4, r5, LSL #2]` : 호출할 함수의 이름을 알 필요없이 주소값을 계산해서 call할 수 있다는 장점이있다.  
> greet_person 함수 호출 (parameter with 계산된 함수 주소)


- greet_person 함수 정의

이 함수에서 매개변수로 전달 받은 주소값에서 실제 함수 주소를 추출해서 branch 한다. 

```asm
/* This function receives an address to a person */
.align 4
greet_person:
    push {r4, lr}            /* keep lr because we call printf, 
                                we keep r4 to keep the stack 8-byte
                                aligned, as per AAPCS requirements */
 
    /* prepare indirect function call */
    mov r4, r0               /* r0 ←  r4, keep the first parameter in r4 */
    ldr r0, [r4]             /* r0 ←  *r4, this is the address to the name
                                of the person and the first parameter
                                of the indirect called function*/
 
    ldr r1, [r4, #4]         /* r1 ←  *(r4 + 4) this is the address
                                to the person tag */
    ldr r1, [r1]             /* r1 ←  *r1, the address of the
                                specific greeting function */
 
    blx r1                   /* indirect call to r1, this is
                                the specific greeting function */
 
    pop {r4, lr}             /* restore r4 and lr */
    bx lr                    /* return to the caller */
```
> 이름 tag를 찾으면 그에 해당하는 함수 호출이 가능함. 마치 구조체처럼.   
> Indirect call 을 사용하면 OOP처럼 함수 호출이 가능해짐.  


## Late binding and object orientation

위의 예제는 아래와 같은 객체지향 C++ 코드와 동일하다.   
A feature of the object-oriented programming (OOP) called late binding, which means that one does not know which function is called for a given object

```c++
struct Person {
  const char* name;
  virtual void greet() = 0;
};
 
struct EnglishPerson : Person {
  virtual void greet() {
    printf("Hello %s\n", this->name);
  }
};
 
struct FrenchPerson : Person {
  virtual void greet() {
    printf("Bonjour %s\n", this->name);
  }
};
```



