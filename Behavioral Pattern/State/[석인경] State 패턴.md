# State Pattern

**유한 상태 기계(FSM, Finite state machine)**, **계층형 상태 기계(Hierarchical state machine)**, 
**푸시다운 오토마타(Pushdown automate)** 에 대해서 다룬다.

## 1. FSM 나온 배경

횡스크롤 게임을 만든다고 할 때 주인공이 사용자 입력에 따라 반응하도록 구현해야한다.

```cpp
void Heroine::handleInput(Input input) {
    if (input == PRESS_B) {
        // 점프 중이 아니라면 점프한다.
    } else if (input == PRESS_DOWN) {
        if (!isJumping_) {
            setGraphics(IMAGE_DUCK);
        }
    } else if (input == RELEASE_DOWN) {
        setGraphics(IMAGE_STAND);
    }
}
```

- 버그를 해결 하기 위해 플래그 변수를 많이 넣어야 한다.
    - 엎드리기 위해 아래 버튼 누른 후
    - B 버튼을 눌러 엎드린 상태에서 점프하고 나서
    - 공중에서 아래 버튼을 떼면
    
    → 점프 중인데도 땅에 서 있는 모습으로 보인다.
    
- 코드가 얼마 없는데 조금만 건드리면 망가진다.

## 2. FSM(**Finite state machine)**

![Untitled (4)](https://github.com/user-attachments/assets/89d39708-3d3a-40fd-928d-6c0f49840568)


### FSM의 요점

- 가질 수 있는 ‘상태’가 한정된다. (서기, 점프, 엎드리기 등등)
- 한 번에 ‘한 가지’ 상태만 될 수 있다. (점프와 동시에 서 있을 수 없다.)
- ‘입력’이나 ‘이벤트’가 기계에 전달된다. (버튼을 누르기, 버튼 떼기)
- 각 상태에는 입력에 따라 다음 상태로 바뀌는 ‘전이’가 있다. (입력이 들어오고 해당하는 전이가 있으면 다음 상태로 변경된다.)

**상태, 입력, 전이가 FSM의 전부이다.**

## 3. FSM을 구현하는 방법 (간단한 버전)

### 열겨형과 다중 선택문

```cpp
enum State {
    STATE_STANDING,
    STATE_JUMPING,
    STATE_DUCKING,
    STATE_DIVING
};
```

이전에는 입력에 따라 분기한 후 상태에 따라 분기했다

→ 하나의 버튼 입력에 대한 코드는 모아둘 수 있었으나 상태에 대한 코드는 흩어져 있었다.

상태 관련 코드를 한곳에 모아두기 위해 상태에 따라 분기하자

```cpp
void Heroine::handleInput(Input input) {
    switch (state_) {
        case STATE_STANDING:
            if (input == PRESS_B) {
                state_ = STATE_JUMPING;
                yVelocity_ = JUMP_VELOCITY;
                setGraphics(IMAGE_JUMP);
            } else if (input == PRESS_DOWN) {
                state_ = STATE_DUCKING;
                setGraphics(IMAGE_DUCK);
            }
            break;

        case STATE_JUMPING:
            if (input == PRESS_DOWN) {
                state_ = STATE_DIVING;
                setGraphics(IMAGE_DIVE);
            }
            break;

        case STATE_DUCKING:
            if (input == RELEASE_DOWN) {
                state_ = STATE_STANDING;
                setGraphics(IMAGE_STAND);
            }
            break;
    }
}
```

열거형 만으로는 부족할 수있다.

- 이동을 구현하는데 엎드려 있으면 기가 모여서 놓는 순간 특수 공격을 쏜다.
    
    → 기를 모으는 시간을 기록해야한다.
    

```cpp
void Heroine::update() {
    if (state_ == STATE_DUCKING) {
        chargeTime_++;
        if (chargeTime_ > MAX_CHARGE) {
            superBomb();
        }
    }
}
```

```cpp
void Heroine::handleInput(Input input) {
    switch (state_) {
        case STATE_STANDING:
            if (input == PRESS_DOWN) {
                state_ = STATE_DUCKING;
                chargeTime_ = 0;
                setGraphics(IMAGE_DUCK);
            }
            // 다른 입력 처리
            break;
        // 다른 상태 처리...
    }
}
```

→ 이것보다는 모든 코드와 데이터를 한곳에 모아둘 수 있는 것이 좋다

## 4. 상태 패턴

### 상태 인터페이스

상태에 의존하는 모든 코드, 다중 선택문에 있던 동작을 인터페이스의 가상 메서드로 만든다.

(예시에서는 handleInput()과 update()가 해당)

```cpp
class HeroineState {
public:
    virtual ~HeroineState() {}
    virtual void handleInput(Heroine& heroine, Input input) {}
    virtual void update(Heroine& heroine) {}
};
```

### 상태별 클래스 만들기

```cpp
class DuckingState : public HeroineState {
public:
    DuckingState() : chargeTime_(0) {}

    virtual void handleInput(Heroine& heroine, Input input) {
        if (input == RELEASE_DOWN) {
            // 일어선 상태로 바꾼다..
            heroine.setGraphics(IMAGE_STAND);
        }
    }

    virtual void update(Heroine& heroine) {
        chargeTime_++;
        if (chargeTime_ > MAX_CHARGE) {
            heroine.superBomb();
        }
    }

private:
    int chargeTime_; 
};
```

chargeTime_ : 엎드리기 상태에서만 의미가 있다는 것을 객체 모델링을 통해서 분명하게 보여준다.

### 동작을 상태에 위임하기

Heroin 클래스에 자신의 현재 상태 객체 포인터를 추가해 거대한 다중 선택문을 제거하고 상태 객체에 위임한다.

```cpp
class Heroine {
public:
    virtual void handleInput(Input input) {
        state_->handleInput(*this, input);
    }
    virtual void update() {
        state_->update(*this);
    }
    // 다른 메서드들...

private:
    HeroineState* state_;
};
```

상태를 바꾸려면 state_ 포인터에 다른 객체를 할당하면 된다. 


### 다른 패턴과의 차이


상태패턴은 전략패턴, 타입 객체 패턴과 비슷해 보인다.

공통점과 차이점을 비교해보자

공통점 : 여러 하위 객체에 동작을 위임한다.

차이점 : 의도

- 전략패턴 : 주요 클래스를 일부 동작으로부터 **디커플링**하는 게 목표
- 타입 객체 패턴 : 같은 타입 객체의 레퍼런스를 **공유**함으로써 **여러** 객체를 비슷하게 동작시키는 게 목표
- 상태 패턴 : 동작을 위임하는 객체를 **변경**함으로써 주요 클래스의 동작을 **변경**하는 게 목표

## 5. 상태 객체는 어디에 둬야 할까?

### 정적 객체

필드가 따로 없다면 모든 인스턴스가 같기 때문에 인스턴스는 하나만 있으면 된다.

정적 인스턴스로 만들고 원하는 곳에 두면 된다.

특별히 다른 곳이 없다면 상위 상태 클래스에 두자.

### 상태 객체 만들기

changeTime_ 처럼 필드가 있는 경우 이 값이 주인공마다 다르기 때문에 정적 객체로 만들 수 없다.

handleInput()에서 상태가 바뀔 때 새로운 상태를 반환하고 밖에서는 반환 값에 따라 예전 상태를 삭제하고 새로운 상태를 저장하자

```cpp
void Heroine::handleInput(Input input) {
    HeroineState* state = state_->handleInput(*this, input);
    if (state != NULL) {
        delete state_;
        state_ = state;
    }
}
```

```cpp
HeroineState* StandingState::handleInput(Heroine& heroine, Input input) {
    if (input == PRESS_DOWN) {
        // 다른 코드들...
        return new DuckingState();
    }
    // 지금 상태를 유지한다.
    return NULL;
}
```

## 6. 입장과 퇴장 (조금 더 상태스럽게 만드는 방법)

주인공의 상태가 변경되면서 주인공의 스프라이트도 같이 변경된다.

지금까지는 이전 상태에서 스프라이트를 변경했었다. 

(엎드리기 상태에서 서기로 넘어갈 때 엎드리기 상태에서 주인공의 이미지가 변경됨)

### 입장 기능을 추가하자

```cpp
class StandingState : public HeroineState {
public:
    virtual void enter(Heroine& heroine) {
        heroine.setGraphics(IMAGE_STAND); //서기 들어올 때 그래픽 변경
    }
    // 다른 코드들...
};
```

```cpp
void Heroine::handleInput(Input input) {
    HeroineState* state = state_->handleInput(*this, input);
    if (state != NULL) {
        delete state_;
        state_ = state;
        // 새로운 상태의 입장 함수를 호출한다.
        state_->enter(*this);
    }
}
```

## 7. FSM의 단점

상태 기계는 엄격하게 제한된 구조를 강제한다.

미리 정해놓은 여러 상태와 현재 상태, 하드코딩 되어있는 전이만 존재한다

인공 지능같이 더 복잡한 곳에 적용하면 한계에 부딪힌다.

## 8. 병행 상태 기계

주인공이 총을 들 수 있는 경우 → 상태 기계를 둘로 나누자

무엇을 하는 가에 대한 상태 기계를 그대로 두고, 무엇을 들고 있는가에 대한 상태 기계를 따로 정의하고 Heroin 클래스는 상태를 각각 참조한다.

이 방식은 두 상태 기계가 서로 연관이 없으면 좋다. 

하지만 복수의 상태 기계가 상호작용해야하는 경우가 많은 경우 이 방법은 좋지 못하다.

## 9. 계층형 상태 기계

### 한번 구현하고 다른 상태에서는 재사용하자.

이벤트가 들어올 때 하위 상태에서 처리하지 않으면 상위 상태로 넘어간다.

```cpp
class OnGroundState : public HeroineState {
public:
    virtual void handleInput(Heroine& heroine, Input input) {
        if (input == PRESS_B) {
            // 점프...
        } else if (input == PRESS_DOWN) {
            // 엎드리기...
        }
    }
};
```

```cpp
class DuckingState : public OnGroundState {
public:
    virtual void handleInput(Heroine& heroine, Input input) {
        if (input == RELEASE_DOWN) {
            // 서기...
        } else {
            // 따로 입력을 처리하지 않고, 상위 상태로 보낸다.
            **OnGroundState::handleInput(heroine, input);**
        }
    }
};
```

### 상태 스택

클래스에 상태 하나만 두지 않고 스택을 만들어서 명시적으로 현재 상태의 상위 상태 연쇄를 모델링 할 수 있다. 

상태 관련 동작 들어오면 어느 상태든 동작을 처리할 때까지 스택을 위에서 밑으로 전달한다.

## 10. 푸시다운 오토마타

### 상태 스택을 활용하여 FSM 확장하는 방법 (9에서 나온 상태 스택과는 다르다!)

FSM은 이력 개념이 없다. 이전 상태로 쉽게 돌아갈 수 없다.

FSM : 한 개의 상태를 포인터로 관리

푸시다운 오토마타 : 상태를 스택으로 관리

![Untitled (5)](https://github.com/user-attachments/assets/00d8e1f7-cfab-4df3-b6eb-798535970b96)

## 11. 마무리

요즘 AI는 행동 트리나 계획 시스템을 더 많이 사용한다

### FSM은 언제 사용하면 좋을까?

- 내부 상태에 따라 객체 동작이 바뀔 때
- 이런 상태가 그다지 많지 않은 선택지로 분명하게 구분될 수 있을 때
- 객체가 입력이나 이벤트에 따라 반응할 때
