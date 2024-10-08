# 프로토타입

- 프로토타입은 코드를 그들의 클래스들에 의존시키지 않고 기존 객체들을 복사할 수 있도록 패턴

### 문제

- 객체를 복제할 때 객체 필드 중 일부가 비공개라서 모든 것을 복제할 수 없음
- 객체의 복제본을 생성하려면 객체의 클래스를 알아야 하므로 코드가 해당 클래스에 의존

### 해결책

- 복제를 지원하는 모든 객체에 대한 공통 인터페이스를 선언
- 인터페이스를 사용하면 코드를 객체의 클래스에 결합하지 않고도 해당 객체를 복제
- 복제를 지원하는 객체를 프로토타입이라고 함
- Clone 메서드에서 현재 필드 값을 새 객체로 모두 전달. 멤버 변수기에 비공개 필드들을 복사하는 것도 가능하다.

### 구조

![image](https://github.com/user-attachments/assets/4a7fb9b0-5c6c-494f-96ea-60ca0321e5a3)


1. **프로토타입** 인터페이스는 복제 메서드들을 선언하며, 이 메서드들의 대부분은 단일 `clone` 메서드입니다.
2. **구상 프로토타입** 클래스는 복제 메서드를 구현합니다. 원본 객체의 데이터를 복제본에 복사하는 것 외에도 이 메서드는 복제 프로세스와 관련된 일부 예외적인 경우들도 처리할 수도 있습니다. (예: 연결된 객체 복제, 재귀 종속성 풀기).
3. **클라이언트**는 프로토타입 인터페이스를 따르는 모든 객체의 복사본을 생성할 수 있습니다.
