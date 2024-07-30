### 바이트 코드 패턴 정의

- 가상 머신 명령어를 인코딩한 데이터로 행동을 표현할 수 있는 유연함을 제공하는 패턴

### 언제 쓸 것인가?

- 언어가 너무 저수준이라 만드는 데 손이 많이 가고 오류가 쉽게 발생
- 컴파일 시간으로 인해 반복 개발이 오래 걸림
- 보안에 취약. 정의하려는 행동이 나머지 코드로부터 격리 필요

한마디로 **정의할 행동은 많은데 게임 구현에 사용한 언어로는 구현하기 어려울 때** 사용한다.

 

**명령어로 표현할 열거형 정의** 

```cpp
enum Instruction {
	INST_SET_HEALTH = 0x00,
	INST_SET_WISDOM = 0x01,
	INST_SET_AGILITY = 0x02,
	INST_PLAY_SOUND = 0x03,
	INST_SPAWN_PARTICLE = 0x04
};
```

- 데이터로 인코딩하려면 이러한 열거형 값을 배열에 저장하면 된다.
- 이렇게 마법을 만들기 위한 코드가 실제로는 바이트들의 목록이라서 **바이트 코드** 라고 불린다.

**가상 기계어**

게임이 실행될 때 플레이어의 컴퓨터는 C++ 트리구조를 순회하는 것이 아니라 미리 컴파일해놓은 기계어를 실행한다.

### 기계어 장점

- 밀도가 높다: 바이너리 데이터가 꽉 차있다
- 선형적이다: 명령어가 같이 모여 있고, 순서대로 실행된다 (메모리를 넘나들지 않는다)
- 저수준이다: 각 명령어는 비교적 최소한의 작업만 한다
- 속도가 빠르다

그러나 우리가 실제로 게임을 기계어로 구현하여 유저에게 제공하는 것은 보안에 취약하다.

기계어의 성능과 인터프리터 패턴의 안정성 사이에서 절충해야 한다.

실제 기계어를 실행하지 말고, **가상 기계어**를 정의하고 이를 실행하는 것을 **에뮬레이터**라고 한다.

- **가상 기계어(바이트 코드)**: 실제 기계어처럼 밀도가 높고, 선형적이고, 저수준이지만, 게임에서 완전히 제어하여 안전하다.

에뮬레이터는 **가상 머신(VM)**, VM이 실행하는 가상 바이너리 기계어는 **바이트코드**라 한다.

### 스택 머신 ( 가상 머신 VM)

복잡한 중첩식을 빠른 속도로 실행하기 위해, 스택을 이용하여 명령어 실행 순서를 제어한다.

```cpp
class VM {
public:
	VM:stackSize_(0) {}
    
private:
	void push(int value);
    int pop();
    
private:
	static const int MAX_STACK = 128;
    int stackSize_;
    int stack_[MAX_STACK];
};
```

명령어가 매개변수를 받게 하기 위해 다음과 같이 스택에서 꺼내온다.

```cpp
switch (instruction) {
	case INST_SET_HEALTH:
    	int amount = pop();
        int wizard = pop();
    	setHealth(wizard, amount);
        break;
    case INST_SET_WISDOM:
    	// 같은 방식...
};
```

이렇게 스택에서 값을 얻어오기 위해서는 리터럴 명령어가 필요하다.

```cpp
case INST_SET_HEALTH:
    int value = bytecode[++i];
    push(value);
    break;
};
```

![스크린샷 2024-07-30 124651](https://github.com/user-attachments/assets/7b66bc6d-8590-4641-94b1-fb4993f8c14f)

- 리터럴 명령어를 만나면 옆에 있는 바이트를 숫자로 읽는다.

![스크린샷 2024-07-30 125631](https://github.com/user-attachments/assets/63378586-0119-4072-8b1d-24ea56042617)


- 먼저 INST_LITERAL부터 실행한다. 이 명령은 자신의 바이트코드 바로 옆 바이트 값(0) 을 읽어서 스택에 넣는다.

![스크린샷 2024-07-30 125712](https://github.com/user-attachments/assets/62286b41-4d0d-407f-a09e-f569047a2411)

- 두 번째 INST_LITERAL을 실행한다. 10을 읽어서 스택에 넣는다.

![스크린샷 2024-07-30 125759](https://github.com/user-attachments/assets/009ed81c-a19f-4d10-9f6b-e18a6dd98d21)

- 마지막으로 INSTSETHEALTH를 실행한다. 스택에서 10을 꺼내와 amount 매개변수에 넣고, 두 번째로 0을 스택에서 꺼내 wizard 매개변수에 넣어 setHealth 함수를 호출한다.

![스크린샷 2024-07-30 125554](https://github.com/user-attachments/assets/4a1e61f2-a043-4d1e-97a4-80b6f3cd1681)
