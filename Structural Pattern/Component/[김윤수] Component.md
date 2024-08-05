### **컴포넌트 패턴, Component Pattern**

- 한 개체가 여러 분야를 서로 커플링 없이 다룰 수 있게 한다.
- 여러 분야를 다루는 하나의 개체가 있다. 분야별로 격리하기 위해, 각각의 코드를 별도의 컴포넌트 클래스에 둔다. 이제 개체 클래스는 단순히 이들 컴포넌트들의 컨테이너 역할만 한다.

---

### **컴포넌트 패턴을 사용하는 이유?**

- AI, 물리, 렌더링, 사운드처럼 분야가 다른 코드끼리는 최대한 서로 모르는 게 좋다.
- 클래스가 크다는 것은 정말 사소한 걸 바꾸려고 해도 엄청난 작업이 필요할 수 있음을 의미한다. 이런 클래스는 기능보다 버그가 더 빨리 늘어난다.

---

### 언제 쓸 것인가?

- 한 클래스에서 여러 분야를 건드리고 있어서, 이들을 서로 디커플링하고 싶다.
- 클래스가 거대해져서 작업하기가 어렵다.
- 여러 다른 기능을 공유하는 다양한 객체를 정의하고 싶다.

상속으로는 딱 원하는 부분만 골라서 재사용할 수가 없지만 컴포넌트 패턴에선 가능하다.

---

### 컴포넌트 패턴을 사용하지 않았을 때 문제점

### **고르디우스의 매듭**

```cpp
if( collidingWithFloor() && (getRenderState() != INVISIBLE) ) {
    playSound(HIT_FLOOR);
}
```

- 이 코드를 문제없이 고치려면 물리(collidingWithFloor), 그래픽(getRenderState), 사운드(playSound)를 전부 알아야 한다.
- 커플링과 코드 길이 문제는 서로 약영향을 미친다.  클래스가 크면 프로그래머가 작업하기 매우 어렵다.

## 컴포넌트 패턴의 장점

### **1. 매듭 끊기**

- 한 덩어리였던 Character 클래스를 분야에 따라 여러 부분으로 나누면 된다.
- 예를 들어 사용자 입력에 관련된 코드는 InputComponent 클래스에 옮겨둔 뒤에, Character 클래스가 InputComponent 인스턴스를 갖게 한다.
- Character 클래스가 다루는 나머지 분야에 대해서도 이런 작업을 반복한다. 컴포넌트 외에는  클래스에 남는 게 거의 없게 된다.
- 즉, 캐릭터 클래스 크기를 컴포넌트 방식으로 줄일 수 있다.

### **2. 느슨한 구조**

- 컴포넌트 클래스들은 디커플링되어 있다. PhysicsComponet와 GraphicsComponent는 Character 클래스 안에 들어 있지만 서로에 대해 알지 못한다.  즉 물리 프로그래머는 그래픽 처리는 신경 쓰지 않고 자기 코드를 수정할 수 있다.
- 모든 코드를 한곳에 섞어놓지 않았기 때문에 서로 통신이 필요한 컴포넌트만으로 결합을 제한할 수 있다.

### 3. 다시 합치기

- 만든 컴포넌트는 재사용할 수 있다

### 컴포넌트를 사용하지 않는다면?

- 데코레이션(decoration)은 덤불이나 먼지같이 볼 수는 있으나 상호작용은 할 수 없는 객체다.
- 프랍(prop)은 상자, 바위, 나무같이 볼 수 있으면서 상호작용 할 수 있는 객체다.
- 존(zone)은 데코레이션과는 반대로 보이지는 않지만 상호작용은 할 수 있는 객체다.

![image](https://github.com/user-attachments/assets/fb37e5f6-0eab-4704-81d3-c8fa434bc106)


- GameObject 클래스에는 위치나 방향 같은 기본 데이터를 둔다.
- Zone은 GameObject을 상속받은 뒤에 충돌 검사를 추가한다.
- Decoration도 GameObject를 상속받은 뒤 렌더링 기능을 추가한다.
- Prop은 충돌 검사 기능을 재사용하기 위해 Zone을 상속받는다.

⇒  Prop이 렌더링 코드를 재사용하기 위해 Decoration 클래스를 상속하려는 순간 '죽음의 다이아몬드'라고 불리는 다중 상속 문제가 생긴다.

### 컴포넌트로 만든다면 상속은 전혀 필요가 없다.

GameObject 클래스 하나와 PhysicsComponent, GraphicsComponent 클래스 두 개만 있으면 된다.

- Decoration : GameObject - GraphicsComponent
- Zone  : GameObject - PhysicsComponent
- Prop : GameObject - PhysicsComponent - GraphicsComponent

⇒ 코드 중복도, 다중 상속이 없는 구조가 완성된다.

⇒ 클래스 개수도 네 개에서 세 개로 줄었다. 

### 예시 코드

입력 인터페이스 

```cpp
class InputComponent {
public:
    virtual ~InputComponent() {}
    virtual void update(Character& character = 0);
};
```

입력 인터페이스를 상속 받아 사용자 입력 클래스 구현

```cpp
class PlayerInputComponent : public InputComponent {
public:
    virtual void update(Character& character) {
        switch(Controller::getJoystickDirection()) {
            case DIR_LEFT:
                character.velocity -= WALK_ACCELERATION;
                break;
            case DIR_RIGHT:
                character.velocity += WALK_ACCELERATION;
                break;
        }
    }

private:
    static const int WALK_ACCELERATION = 1;
};
```

Character 클래스는 InputComponent 구체 클래스의 인스턴스가 아닌 인터페이스의 포인터를 들고 있도록 구현

```cpp
class Character {
public:
    int velocity;
    int x, y;

    Character(InputComponent* input) : input_(input) {}

    void update(World& world, Graphics& graphics) {
        input_->update(*this);
        physics_.update(*this, world);
        graphics_.update(*this, graphics);
    }

private:
    InputComponent* input_;
    PhysicsComponent physics_;
    GraphicsComponent graphics_;
};
```

```cpp
Character* character = new Character(new PlayerInputComponent());
```

- 어떤 클래스라도 InputComponent 추상 인터페이스만 구현하면 입력 컴포넌트가 될 수 있다.
- '데모 모드'에서 플레이어가 아무것도 안하고 가만히 앉아 있을 때, 대신 컴퓨터가 자동으로 게임을 플레이 하는등의 기능을 만들 수 있다.
