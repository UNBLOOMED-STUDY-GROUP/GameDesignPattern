# 명령 패턴

명령(Command) 패턴은 요청을 요청에 대한 모든 정보가 포함된 독립실행형 객체로 변환하는 행동 디자인 패턴이다. 

→ 요청을 객체의 형태로 캡슐화 한다.

유저의 입력을 다룰 때 많이 사용하는 패턴으로 요청 코드와 수행 코드 사이의 의존성을 줄인다.

## 어울리는 상황

- 되돌릴 수 있는 작업을 구현하려고 할 때 (취소/다시실행)
- 플레이어가 액터를 선택하여 제어하려고 할 때

## 전략 패턴의 장점

- 단일 책임 원칙 (Single Responsibility Principle)
    
    → 각 요청을 객체의 형태로 캡슐화하여 명령을 실행하는 객체(Receiver)와 명령을 내리는 객체(Invoker)를 분리한다.
    
- 개방/폐쇄의 원칙 (Open/Closed Principle)
    
    → 새로운 명령을 추가할 때 기존 코드를 수정할 필요 없이 새로운 구상 클래스를 추가할 수 있다.
    

## 전략 패턴의 단점

- 각각의 개별 요청에 대해 Command 클래스를 만들어야 하기 때문에, 클래스가 많아질 수 있다.

## 전략 패턴 구현 예시

유저가 자신이 원하는 키를 설정할 수 있도록 제공할 때 사용

```cpp
void InputHandler::handleInput()
{
    if (isPressed(BUTTON_X)) jump();
    else if(isPressed(BUTTON_Y)) fireGun();
    else if(isPressed(BUTTON_A)) swapWeapon();
    else if(isPressed(BUTTON_B)) kneel();
} //명령 패턴 사용 X, 함수 직접 호출
```

키 변경을 지원하려면 함수를 직접 호출하지 말고 교체 가능한 무엇인가로 바꿔야한다.

게임에서 할 수 있는 행동을 실행할 수 있는 공통 상위 클래스부터 정의한다.

```cpp
class Command {
public:
    virtual ~Command() { }
    virtual void excute() = 0;
};
// 인터페이스에 반환 값이 없는 메서드 하나밖에 없다면 명령 패턴일 가능성이 높다.
```

행동별로 하위 클래스 만든다.

```cpp
class JumpCommand : public Command {
public:
    virtual void execute() { jump(); }
};

class FireCommand : public Command {
public:
    virtual void execute() { fireGun(); }
};
```

입력 핸들러 코드에는 각 버튼별로 Command 클래스 포인터를 저장한다.

```cpp
class InputHandler {
public:
    void handleInput();
    // 명령을 바인드(bind)할 메서드들

private:
    Command* buttonX_;
    Command* buttonY_;
    Command* buttonA_;
    Command* buttonB_;
};
```

```cpp
void InputHandler::handleInput()
{
    if (isPressed(BUTTON_X)) buttonX_->excute();
    else if(isPressed(BUTTON_Y)) buttonY_->excute();
    else if(isPressed(BUTTON_A)) buttonA_->excute();
    else if(isPressed(BUTTON_B)) buttonB_->excute();
}
```

직접 함수를 호출하는 대신 한 겹 우회하는 계층이 생긴다.
