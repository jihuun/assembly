
# 그외 명령어들

## TST 및 TEQ
`TST x y` y에 설정된 비트가 모두 x에 설정되어 있는지 검사
`TEQ x y` x와 y의 값이 동일 한지 검사 

`TST` 명령어는 Rn의 값과 Operand2의 값에 대해 비트 단위 AND 연산을 수행합니다. 이 명령어는 결과가 버려진다는 점을 제외하고 ANDS 명령어와 동일합니다.

`TEQ` 명령어는 Rn의 값과 Operand2의 값에 대해 비트 배타적 OR 연산을 수행합니다. 이 명령어는 결과가 버려진다는 점을 제외하고 EORS 명령어와 동일합니다.
`cmp` 대신 사용할 수 있음. carry flag가 변하지 않음

TEQ 명령어를 사용하면 CMP와 마찬가지로 V 또는 C 플래그에 영향을 주지 않고 두 값이 동일한지 여부를 테스트할 수 있습니다.
TEQ는 값의 부호를 테스트하는 데도 유용합니다. 비교 후, N 플래그는 두 피연산자 부호 비트의 논리 배타적 OR이 됩니다.
http://infocenter.arm.com/help/index.jsp?topic=/com.arm.doc.dui0204ik/Cihcdehh.html

```asm
	mov	r0, #1
	teq	r0, #1
	beq	teq_test
```
> 두 비트가 동일하면 teq_test 로 branch

http://forum.falinux.com/zbxe/index.php?document_srl=762683&mid=lecture_tip

----
To be updated..
