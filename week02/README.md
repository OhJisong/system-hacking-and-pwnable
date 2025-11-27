# Pwnable Week2 — Function Call & Stack Frame 분석 정리

Week2 실습을 통해 확인한 32-bit 환경의 함수 호출 과정과 스택 프레임 구조를 정리한 것이다.
gdb 분석과 어셈블리 수준의 동작을 기반으로 내용을 재구성하였다.

---

## 1. 실습 목표

**1. C 프로그램이 함수를 호출할 때 스택이 어떻게 변하는지 확인**
**2. Prologue / Epilogue 를 통한 스택 프레임 생성 및 복원**
**3.EBP 기준으로 로컬 변수/함수 인자의 위치 이해**
**4. eax를 통한 리턴값 전달 구조 확인**
**5. gdb로 실제 스택 레이아웃을 관찰하여 함수 호출 흐름 확인**

---

## 2. 실습 코드

아래 프로그램을 실제로 디버깅하며 스택 구조를 분석한다.

```
#include <stdio.h>

int sum(int x, int y) {
    int result = x + y;
    return result;
}

int main() {
    int a = 2;
    int b = 3;
    int c = sum(a, b);
    return 0;
}

```

## 3. main() 함수 스택 프레임 분석

### 3.1 Prologue 이후 구조

main() 진입 직후 어셈블리:

```
push ebp
mov ebp, esp
sub esp, 0x10
```

명령어 의미는 다음과 같다:

| 명령어             | 의미                |
| --------------- | ----------------- |
| `push ebp`      | 이전 프레임 포인터 저장     |
| `mov ebp, esp`  | 새로운 스택 프레임 생성     |
| `sub esp, 0x10` | 로컬 변수 공간 16바이트 확보 |

로컬 변수는 다음 위치에 저장된다.

```
[ebp-4]   → a
[ebp-8]   → b
[ebp-12]  → c
```

### 3.2 sum(a, b) 호출 시 인자 전달

함수 인자는 모두 스택을 통해 전달된다.

```
push DWORD PTR [ebp-0x8]   ; push b
push DWORD PTR [ebp-0x4]   ; push a
call sum
```

이때의 스택 구조 : 
```
[esp]       → a
[esp+4]     → b
[esp+8]     → return address
[esp+12]    → saved ebp (main)
```

---

## 4. sum() 함수 스택 프레임 분석

### 4.1 Prologue

```
push ebp
mov ebp, esp
sub esp, 0x4
```

`result` 변수를 위한 공간이 `[ebp-4]`에 생성된다.

### 4.2 함수 인자 접근

x, y 인자는 다음 offset에서 찾을 수 있다:

| 항목             | 위치         |
| -------------- | ---------- |
| return address | `[ebp+4]`  |
| x              | `[ebp+8]`  |
| y              | `[ebp+12]` |
| result         | `[ebp-4]`  |


실제 어셈블리 : 
```
mov eax, DWORD PTR [ebp+0x8]   ; eax = x
add eax, DWORD PTR [ebp+0xc]   ; eax = x + y
mov DWORD PTR [ebp-0x4], eax   ; result 저장
```

### 4.3 Epilogue — 스택 복원
```
leave
ret
```

`leave`는 다음과 동일:
```
mov esp, ebp
pop ebp
```

---

## 5. gdb 분석 과정

### 5.1 Breakpoint 설정
```
b main
b sum
run
```

### 5.2 레지스터 확인
```
info registers
```

### 5.2 스택 메모리 확인
```
x/20x $esp
x/20x $ebp
```

### 5.4 함수 호출 직전 esp 변화 보기

`push`두 번 후 `esp`가 8바이트 감소했는지 확인.

### 5.5 sum 종료 직전 상태 확인
```
print $eax
```
`eax=5` 이면 정상 (2+3 = 5)

---

## 6. 함수 호출 전체 흐름 정리

1. caller가 인자 push
2. return address push
3. callee가 Prologue로 스택 프레임 생성
4. 계산 후 eax에 리턴값 저장
5. leave + ret으로 기존 스택 복원 후 caller 복귀

C의 함수 호출은 실제로는 스택 조작(PUSH/POP) + 레지스터 운영의 연속이다.

---

## 7. Week2 핵심 요약

* 함수 호출은 스택 기반 동작이다

* 로컬 변수 → [ebp - offset]

* 함수 인자 → [ebp + offset]

* 리턴값은 eax로 전달

* gdb는 스택 구조를 직접 확인하는 가장 확실한 방법

* 이 구조는 버퍼 오버플로우, ROP 등 스택 기반 취약점 분석의 기초가 된다
