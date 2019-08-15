
# 23. Nested functions
----

Nested function 은 함수 내부에 정의된 함수이다. 함수 내부에서만 사용가능하다. 마치 로컬변수 처럼 lifetime이 함수와 동일하다.  

```c
float E(float x)
{
    float F(float y)
    {
        return x + y;
    }
    return F(3) + F(4);
}
```
> F()는 함수E() 안에 있는 함수인데, 그 scope가 함수E()와 동일하다. 


## Nested function for assembler level

기본적으로 Assembler 레벨에서는 함수는 Nested될 수 없다. (~~사실 따지고 보면 함수라는것도 존재하지 않는다~~) 함수는 Calling Convention(AAPCS) 과 함께 논리적으로 존재할 뿐이다.  Assembler 레벨에서는 모든것은 데이터나 명령어 아니면 주소이다. 

Assembler 레벨에서 Nested Function의 의미는 Dynamic Activate 범위 내 에서만 존재(life tiem을 같이하는)하는 함수를 의미한다. 

## Dynamic link

Dynamic link 란?  
The dynamic link is set at the beginning of the function and we use the fp register (an alias for r11) to keep an address in the stack usually called the frame pointer.  

frame pointer를 이용하면 dynamic activation 내의 로컬변수를 일관되게 엑세스할 수 있다. 로컬변수는 `fp`로부터 (-)offset, 전달받은 parameter는 (+)offset 으로 얻을 수 있다. 아래 그림에서 activation record 가 스택에 어떻게 쌓이는지 보여준다.  
(리눅스 ARM에서는 낮은 주소 방향으로 스택 사이즈가 커진다는것을 항상 유의)

![activation record](https://thinkingeek.com/wp-content/uploads/2015/01/activation_record.png)


## Static link

가정해보자, 위의 그림에서 함수 g가 함수 f의 Nested function이라면? 함수 g는 f의 로컬변수를 Access할 수 있어야한다. 그것은 previous fp를 얻음으로써 가능하다.    
결국, 이 장에서 우리의 최종 관심사는 어떻게 Caller함수의 로컬 변수를 Nedted함수에서 Access할수 있는가이다.  

다음과 같은 C nested 함수가 있다고 하자. 그림처럼 함수 g()가 함수 f()의 Nested function 이다.  

#### 예제 1

```c
void f() // non-nested (normal) function
{
    int x;
    void g()     // nested function
    {
      x = x + 1; // x ←  x + 1
    }
  
    x = 1;       // x ←  1
    g();         // call g
    x = x + 1;   // x ←  x + 1

}
```
> 최종 x 는 3이 될것이다.  
> Nested function g()에서 f()의 로컬변수 x를 접근해서 사용했다.  

어셈블리 레벨에서 g()가 f()의 로컬변수 x를 어떻게 Access할 수 있을까? g()함수에서 이전의 frame pointer 즉, 함수 f()의 `fp`를 얻어야한다.  


```asm
.text
 
f:
    push {r4, r5, fp, lr} /* keep registers */
    mov fp, sp /* keep dynamic link */
 
    sub sp, sp, #8      /* make room for x (4 bytes)
                           plus 4 bytes to keep stack
                           aligned */
    /* x is in address "fp - 4" */
 
    mov r4, #0          /* r4 ←  0 */
    str r4, [fp, #-4]   /* x ←  r4 */
 
    bl g                /* call (nested function) g
                           (the code of 'g' is given below, after 'f') */
 
    ldr r4, [fp, #-4]   /* r4 ←  x */
    add r4, r4, #1      /* r4 ←  r4 + 1 */
    str r4, [fp, #-4]   /* x ←  r4 */
 
    mov sp, fp /* restore dynamic link */
    pop {r4, r5, fp, lr} /* restore registers */
    bx lr /* return */
 
    /* nested function g */
    g:
        push {r4, r5, fp, lr} /* keep registers */
        mov fp, sp /* keep dynamic link */
 
        /* At this point our stack looks like this
 
          Data | Address | Notes
         ------+---------+--------------------------
           r4  | fp      |  
           r5  | fp + 4  |
           fp  | fp + 8  | This is the previous fp
                           ^^^^^^^^^^^^^^^^^^^^^^^
           lr  | fp + 16 |
        */
 
        ldr r4, [fp, #+8] /* get the frame pointer of my caller (since only f can call me) */
        ^^^^^^^^^^^^^^^^^
        /* now r4 acts like the fp we had inside 'f' */
        ldr r5, [r4, #-4] /* r5 ←  x */
        add r5, r5, #1    /* r5 ←  r5 + 1 */
        str r5, [r4, #-4] /* x ←  r5 */
 
        mov sp, fp /* restore dynamic link */
        pop {r4, r5, fp, lr} /* restore registers */
        bx lr /* return */
 
.globl main
 
main :
    push {r4, lr} /* keep registers */
 
    bl f          /* call f */
 
    mov r0, #0
    pop {r4, lr}
    bx lr
```
> - 함수 g 의 Activation Record 내에있는 보존된 레지스터들 중 하나가 caller함수 f의 `fp` 를 가지고 있다. 따라서 fp에서 (+)offset을 통해 previous fp를 얻을 수 있다. (이것은 마치 caller가 전달한 parameter를 얻는 것과 동일하다) 따로 stack operation(push/pop)을 통해 이전 fp를 얻어 낼 필요가 없다는 뜻이다.   
> - `ldr r4, [fp, #+8]` : [fp, #+8] 이 바로 이전 fp가 보존된 스택의 주소(가 가리키는 값)이다.  
> - 따라서 f()의 `fp`를 얻었으므로 f()의 로컬변수는 `fp`의 (-)offset을 통해 얻어낼수 있다.  
> - `ldr r5, [r4, #-4]` : [r4, #-4] 가 바로 local변수 x 의 주소(가 가리키는 값)이다.  

위의 예제에서는 함수 g()에서 현재`fp`에서 직접 offset 을 계산해 caller의 `fp`를 얻어왔다. 하지만 이 방식에는 명확한 한계가 있다. Nested function call을 한번밖에 못한다는 것이다!! 또다른 nested 함수를 호출하지 못할 뿐더러, 함수를 재귀적으로 호출 할 수도 없다. 따라서 호출된 함수 내에서 Frame pointer를 직접 이용하는 방법은 한계가 있다.  

따라서 항상 nested함수를 호출한 이전함수의 Activation 을 얻어와야 한다. 이 방식을 __Static Link__라고 부른다.
즉 Static Link는 실제 nested함수가 구현된, enclosing된 함수의 `fp` 를 의미한다. 단순이 caller의 `fp`가 아니다.  
Static Link의 개념은 간단하다. 서로 연결된 frame pointer의 체인이다. Static Link는 어떤 함수가 호출되느냐에 따라 서로 다르게 설정이 될 수 있다. 그리고 그 설정은 Caller가 한다.  다음 일련의 Assembly 예제들을 보면 무슨말인지 더 명확하게 이해할 수 있다.  

다음과 같은 중첩된 nested function 을 보자.   

#### 예제 2

```c
void f(void) // non nested (nesting depth = 0)
{
   int x;
 
   void g() // nested (nesting depth = 1)
   {
      x = x + 1; // x ←  x + 1
   }
   void h() // nested (nesting depth = 1)
   {
      void m() // nested (nesting depth = 2)
      {
         x = x + 2; // x ←  x + 2
         g(); // call g
      }
 
      g(); // call g
      m(); // call m
      x = x + 3; // x ←  x + 3
   }
 
   x = 1; // x ←  1
   h();  // call h
}
```
> 결과는 x 는 8이 될것이다.  
> h 와 g는 f에의해 둘러쌓이게(enclosing)된다.  
> m은 h에의해 둘러쌓이게(enclosing)된다.   
> m이 g를 호출하더라도 g의 static link는 m이 아니다. g가 어디서 호출되던 간에 g의 static link는 f임을 주의.   
> g는 f에 의해 enclosing되었기 때문이다.   

더욱 간단히 표현 하면 아래와 같다.  

```asm
f {
        g { }           // f 를 가리킴
        h {             // f 를 가리킴
                m { }   // h 를 가리킴
       	}
}
```

## Setting up a static link

AAPCS는 어떤레지스터를 Static link로 사용할지 강제하지는 않는다. 우리는 이 튜토리얼에서 `r10`을 사용할 것이다.  

Static Link를 setting할때에는 다음의 두가지 케이스를 고려해야 한다. 

* Nesting depth 1  
	예제 1
* Nesting depth 2 이상  
	예제 2

자, 그럼 이제 예제2 는 어떻게 assembly로 구현하는지 살펴 보자.  
핵심은 caller가 호출할 callee의 Static Link를 찾아서 `r10`에 넣어주고 호출한다는 것이다. 그 Static Link는 단순히 caller의 `fp`가 아닐수 있음을 위에서 설명하였다.   

__지금부터 나올 assembly 예제들의 주석을 꼼꼼히 살펴보자.__  

#### 함수 f()

```asm
f:
    push {r4, r10, fp, lr} /* keep registers */
    mov fp, sp             /* setup dynamic link */
 
    sub sp, sp, #8      /* make room for x (4 + 4 bytes) */
    /* x will be in address "fp - 4" */
 
    /* At this point our stack looks like this
 
     Data | Address | Notes
    ------+---------+---------------------------
          | fp - 8  | alignment (per AAPCS)
      x   | fp - 4  | 
      r4  | fp      |  
      r10 | fp + 8  | previous value of r10
      fp  | fp + 12 | previous value of fp
      lr  | fp + 16 |
   */
 
    mov r4, #1          /* r4 ←  1 */
    str r4, [fp, #-4]   /* x ←  r4 */
 
    /* prepare the call to h */
    mov r10, fp /* setup the static link,
                   since we are calling an immediately nested function
                   it is just the current frame */
    bl h        /* call h */
 
    mov sp, fp             /* restore stack */
    pop {r4, r10, fp, lr}  /* restore registers */
    bx lr /* return */
```
> 함수 f는 nested되지 않았기 때문에  `mov r10, fp` 여기서 이동작은 큰 의미가 없다.  

#### 함수 h()

```asm
h :
    push {r4, r5, r10, fp, lr} /* keep registers */
    mov fp, sp /* setup dynamic link */
 
    sub sp, sp, #4 /* align stack */
 
    /* At this point our stack looks like this
 
      Data | Address | Notes
     ------+---------+---------------------------
           | fp - 4  | alignment (per AAPCS)
       r4  | fp      |  
       r5  | fp + 4  | 
       r10 | fp + 8  | frame pointer of 'f'
       ^^^             ^^^^^^^^^^^^^^^^^^^^
       fp  | fp + 12 | frame pointer of caller
       lr  | fp + 16 |
    */
 
    /* g는 h와 형제이다. 따라서 h의 static link와 동일하다. */
    ldr r10, [fp, #8]
    ^^^^^^^^^^^^^^^^^
    bl g
 
    /* m은 h함수 내에서 nested되어있다. 따라서 m의 static link는 현재 fp와 동일하다 */
    mov r10, fp
    ^^^^^^^^^^^
    bl m
 
    ldr r4, [fp, #8]  /* load frame pointer of 'f' */
    ^^^^^^^^^^^^^^^^
    ldr r5, [r4, #-4]  /* r5 ←  x */
                         ^^^^^^^^^
    add r5, r5, #3     /* r5 ←  r5 + 3 */
    str r5, [r4, #-4]  /* x ←  r5 */
 
    mov sp, fp            /* restore stack */
    pop {r4, r5, r10, fp, lr} /* restore registers */
    bx lr
```
> * g 의 static link r10에 저장 `ldr r10, [fp, #8]`  
> * m 의 static link r10에 저장 `mov r10, fp`  
> * h 에서 f의 local 변수 얻기:  
>	`ldr r4, [fp, #8]`  /* load frame pointer of 'f' */  
>	`ldr r5, [r4, #-4]` : 결국 f의 로컬변수를 얻어옴
>	여기서 형제(Sibling)의 의미는 Nesting depth가 동일하다는것을 말한다.  

중첩된 nested 함수의 경우, 단순히 [예제 1](#예제 1) 처럼 `fp`를 이용하는 것이 아니다. `r10` 을 이용해서 static link를 미리 계산해서 전달하는 이유는 call하는 함수마다 static link를 다르게 계산 해 주어야 하기 때문이다. 위 예제에서 m()과 g()는 static link가 다르다. 함수 내에서 fp를 통해 계산할수 있지만 그러면 복잡해질 것이다. 미리 `r10` 에 계산하여 전달해주는것과 같다.  

#### 함수 m()

```asm
m:
    push {r4, r5, r10, fp, lr} /* keep registers */
    mov fp, sp /* setup dynamic link */
 
    sub sp, sp, #4 /* align stack */
    /* At this point our stack looks like this
 
      Data | Address | Notes
     ------+---------+---------------------------
           | fp - 4  | alignment (per AAPCS)
       r4  | fp      |  
       r5  | fp + 4  |
       r10 | fp + 8  | frame pointer of 'h'
       ^^^             ^^^^^^^^^^^^^^^^^^^^
       fp  | fp + 12 | frame pointer of caller
       lr  | fp + 16 |
    */
 
    ldr r4, [fp, #8]  /* r4 ←  frame pointer of 'h' */
    ^^^^^^^^^^^^^^^^
    ldr r4, [r4, #8]  /* r4 ←  frame pointer of 'f' */
    ^^^^^^^^^^^^^^^^
    ldr r5, [r4, #-4] /* r5 ←  x */
    add r5, r5, #2    /* r5 ←  r5 + 2 */
    str r5, [r4, #-4] /* x ←  r5 */
 
    /* setup call to g */
    ldr r10, [fp, #8]   /* r10 ←  frame pointer of 'h' */
    ^^^^^^^^^^^^^^^^^
    ldr r10, [r10, #8]  /* r10 ←  frame pointer of 'f' */
    ^^^^^^^^^^^^^^^^^^
    bl g
 
    mov sp, fp                /* restore stack */
    pop {r4, r5, r10, fp, lr} /* restore registers */
    bx lr
```
> `ldr r4, [fp, #8]` : h의 프레임포인터   
> `ldr r4, [r4, #8]` : 를 이용해 f 프레임포인터를 얻어옴  
> 함수 m()내에서 호출되는 __g() 의 static link는 f의 fp__이라는점이 중요하다. 그 때문에 호출하기 전에 위와 같이 계산함.  
> g()를 호출하는 caller가 m()이라고 하더라도 g()의 static link는 f()임에 주의  

#### 함수 g()


```asm
g:
    push {r4, r5, r10, fp, lr} /* keep registers */
    mov fp, sp /* setup dynamic link */
 
    sub sp, sp, #4 /* align stack */
 
    /* At this point our stack looks like this
 
      Data | Address | Notes
     ------+---------+---------------------------
           | fp - 4  | alignment (per AAPCS)
       r4  | fp      |  
       r5  | fp + 4  |  
       r10 | fp + 8  | frame pointer of 'f'
       ^^^             ^^^^^^^^^^^^^^^^^^^^
       fp  | fp + 12 | frame pointer of caller
       lr  | fp + 16 |
    */
 
    ldr r4, [fp, #8]  /* r4 ←  frame pointer of 'f' */
    ^^^^^^^^^^^^^^^^
    ldr r5, [r4, #-4] /* r5 ←  x */
    ^^^^^^^^^^^^^^^^^
    add r5, r5, #1    /* r5 ←  r5 + 1 */
    str r5, [r4, #-4] /* x ←  r5 */
 
    mov sp, fp /* restore dynamic link */
    pop {r4, r5, r10, fp, lr} /* restore registers */
    bx lr
```
> h와 비슷하나, 따로 함수를 호출하지는 않는다. 
> g와 h는 nesting depth가 동일함.  

지금까지 살펴본 예제 코드는 모두 fp + 8 로, static link를 얻어왔지만, 항상 그런것은 아니다. Caller가 r10을 push 하기전에  얼마나 많은 register를 push했는지에 따라 달라질 수 있기 때문이다.  
 
마지막으로 f()를 호출하는 main 함수는 아래와 같다.  

```asm
.globl main
 
main :
    push {r4, lr} /* keep registers */
 
    bl f          /* call f */
 
    mov r0, #0
    pop {r4, lr}
    bx lr
```
> nested 되지 않은 함수가 또다른 non-nested 함수를 호출할때는 r10에 어떤것도 할 필요가 없다.  


## 한계

실제로 r10은 함수내에서 사용되거나 변경되지 않았다. 그런데 왜 stack에 push하는가?  
동일한 함수 scope에 위치해야하기 때문이다? 이것을 Lexical scope라고 부른다. static link가 서로 chaining되어있기 때문에 각각의 lexical scope로 이동할 수 있다.  

`참고: Lexical scope`
C/C++, Java, 그리고 JavaScript 같이 우리가 접하는 대부분의 언어들은 Lexical Scope를 사용한다. Lexical Scope는 Static Scope라고도 불린다. Scope가 함수가 호출될때가 아니라 정의될때 생긴다는 뜻이다. Lexical Scope는 변수나 함수가 정의 된 곳의 context를 사용하며, Dynamic Scope는 변수나 함수가 불려진 곳의 context를 사용한다.   
__Lexical scope__: use environment where function [and variable] is __defined__  
__Dynamic scope__: use environment where function [and variable] is __called__  
 
출처: https://bestalign.github.io/2015/07/12/Lexical-Scope-and-Dynamic-Scope/  

callee가 r10을 stack에 push 하는대신에 caller가 stack에 push하는 방식으로 parameter로 전달할 수도 있다. 또한 물론 r0 ~ r3레지스터로 인자를 전달 할 수도 있다. 대신에 그러면 내부에서 stack에 다시 push해줘야한다.   

결국 static link는 항상 stack에 저장해야한다는 의미다.  

그런데 만약 static link를 이용해 접근해야할 로컬 변수가  pointer 라면 어떻게 해야하나?
지금까지 예제에서 우리는 어떤 위치 어떤 함수를 호출해야할지 미리 다 알고있었다. 만약 Indirect call로 함수를 호출할때는 어떻게 되는가? 우리는 어떤 nested함수가 호출될지 미리 알지 못하며 lexical socope를 어떻게 설정할지도 모른다. 결국 lexical sopce를 어딘가에 유지하지 않는이상 못한다.    

This means that just the address of the function will not do. We will need to keep, along with the address to the function, the lexical scope.
So a pointer to a nested function happens to be different to a pointer to a non-nested function, given that the latter does not need the lexical scope to be set.

다음 챕터에서는 non-nested함수와 포인터가 다른 서로다른 함수들의 포인터를 갖는것을 피하는 방법에 대해 알아보자 (?)

Having incompatible pointers for nested an non nested functions is not desirable. This may be a reason why C (and C++) do not directly support nested functions (albeit this limitation can be worked around using other approaches). In the next chapter, we will see a clever approach to avoid, to some extent, having different pointers to nested functions that are different from pointers to non nested functions. (~~?무슨말인지 모르겠음~~)






