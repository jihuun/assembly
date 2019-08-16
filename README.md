# Learning Assembly Language with ARM

> 작성: 김지훈 (jihuun.k@gmail.com)
> 2018-07 작성

이 문서는 ARM Assembly 명령어를 쉽게 배울 수 있도록 설명한 튜토리얼이다. Assembly언어를 전혀 몰라도 충분히 읽을 수 있는 Beginner용으로, 32bit ARMv6 Architecture 기반의 Assembly 언어를 매우 익힐 수있다. 또한 Architecture를 떠나서 일반적인 Assembly Language 의 개념을 익힐 수 있다. 시중에는 많은 ARM assembly 언어 참고자료가 있지만 이 튜토리얼 만큼 초보자가 체계적으로 쉽게 익힐 수 있는 컨텐츠는 본적이 없었던것 같다. 참고로, 튜토리얼은 Raspberry PI 2를 Target machine으로 두고 작성되었지만 내용과는 크게 상관이 없어서 해당 machine이 없어도 무방하다.   

이 문서는 주로 Roger Ferrer Ibanez라는 ARM 엔지니어가 작성한 튜토리얼에서 내가 배운것들을 정리한 문서이다. 이 문서는 거의 요약정리이기 때문에 설명이 부족하다면 아래 튜토리얼에서 각 챕터의 내용을 참고하기 바란다. This documentation is what I've learned from a tutorial below writen by Roger Ferrer Ibanez, an ARM engineer.  

[튜토리얼 링크: http://thinkingeek.com/arm-assembler-raspberry-pi/](http://thinkingeek.com/arm-assembler-raspberry-pi)  

다음은 ARM 64bit assembly 언어에 대한 post로 추후에 해당 내용도 업데이트 할 예정이다.  
https://thinkingeek.com/2016/10/08/exploring-aarch64-assembler-chapter1/  
https://thinkingeek.com/2016/10/08/exploring-aarch64-assembler-chapter-2/  
https://thinkingeek.com/2016/10/23/exploring-aarch64-assembler-chapter-3/  
https://thinkingeek.com/2016/10/23/exploring-aarch64-assembler-chapter-4/  
https://thinkingeek.com/2016/11/13/exploring-aarch64-assembler-chapter-5/  
https://thinkingeek.com/2016/11/27/exploring-aarch64-assembler-chapter-6/  
https://thinkingeek.com/2017/03/19/exploring-aarch64-assembler-chapter-7/  
https://thinkingeek.com/2017/05/29/exploring-aarch64-assembler-chapter-8/  
https://thinkingeek.com/2017/11/05/exploring-aarch64-assembler-chapter-9/  

---
