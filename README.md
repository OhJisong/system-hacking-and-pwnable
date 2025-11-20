# system-hacking-and-pwnable
A comprehensive study repository for System Hacking, Binary Exploitation, Reversing, and Pwnable challenges. Includes memory vulnerability analysis, assembly notes, exploit development, and CTF-style write-ups.

### 목표

- 기초부터 차근차근 시스템 해킹 시작
- 너무 어려운 이론보다는, **실습 위주로 감각 익히기**

---

### 전체 커리큘럼

1. 시스템 메모리 구조 & 함수 호출 원리**
   - 프로그램을 실행할 때, 메모리에서 어떤 일이 일어나는지
   - 스택/힙/데이터/코드 4영역과 함수 호출 시 스택 변화

2. 어셈블리, `gdb`, 쉘**
   - CPU가 소스코드를 어떻게 이해하는지 (어셈블리)
   - `gdb`로 레지스터, 스택, 메모리 추적
   - 리눅스 쉘 환경에서 실습

3. BOF(Buffer Overflow) & `pwntools`**
   - 버퍼 오버플로우 개념과 원리
   - `pwntools`로 익스플로잇 스크립트 작성
