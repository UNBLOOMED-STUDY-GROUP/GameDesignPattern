# 전략 패턴
전략 패턴(Strategy Pattern)은 정책 패턴(Policy Pattern)이라고도 하며, 

객체의 행위를 바꾸고 싶은 경우 '직접' 수정하지 않고 전략이라고 부르는 '캡슐화한 알고리즘'을 컨텍스트 안에서 바꿔주면서 상호 교체가 가능하게 만드는 패턴이다.

예를 들어서 우리가 물건을 살 때 카카오페이, 네이버페이, 삼성페이 등 다양한 방법으로 결제하듯이, 어떤 아이템을 살 때 KAKAOCard와 SAMSUNGCard로 사는 걸 구현한다고 할 수 있다. 즉, 결제 방식의 '전략'만 바꿔서 여러 가지 방식으로 결제하는 것을 구현할 수 있다.

## 어울리는 상황
전략 패턴은 객체 내에서 한 알고리즘의 다양한 변형들을 사용하고 싶을 때, 그리고 런타임 중에 한 알고리즘에서 다른 알고리즘으로 전환하고 싶을 때 사용한다.

알고리즘 코드가 노출되어서는 안 되는 데이터에 액세스 하거나 데이터를 활용할 때 (캡슐화)

## 전략 패턴의 장점
전략 패턴을 이용하면 상위 클래스를 수정하지 않고 새로운 행동을 추가할 수 있다.

전략 패턴은 다양한 행동들을 별도의 클래스 계층구조로 추출하고, 추출된 클래스들을 하나의 인터페이스로 결합하여 중복 코드를 줄인다.

모든 알고리즘을 같은 인터페이스를 구현하는 별도의 클래스들로 추출하여 조건문을 제거할 수 있다.

## 전략 패턴의 예시
주인공이 무기 전략을 바꿔가며 대응하는 상황

### 클린하지 않은 코드
적이 오면 상수를 메소드에 넘겨 조건문으로 일일히 필터링하여 적절한 전략을 실행하였다.

하지만 상태 변수를 통해 행위를 분기문으로 나누는 행위는 좋지 않은 코드이다. 

자칫 잘못하면 if else 지옥에 빠질 수 있기 때문이다.

```cpp
#include <iostream>

// 무기 상수 정의
class TakeWeapon {
public:
    static const int SWORD = 0;
    static const int SHIELD = 1;
    static const int CROSSBOW = 2;

private:
    int state;

public:
    void setWeapon(int state) {
        this->state = state;
    }

    void attack() const {
        if (state == SWORD) {
            std::cout << "칼을 휘두르다" << std::endl;
        } else if (state == SHIELD) {
            std::cout << "방패로 밀친다" << std::endl;
        } else if (state == CROSSBOW) {
            std::cout << "석궁을 발사하다" << std::endl;
        }
    }
};

// 클라이언트 - 전략 제공/설정
int main() {
    // 플레이어 손에 무기 착용 전략을 설정
    TakeWeapon hand;

    // 플레이어가 검을 들도록 전략 설정
    hand.setWeapon(TakeWeapon::SWORD);
    hand.attack(); // "칼을 휘두르다"

    // 플레이어가 방패를 들도록 전략 설정
    hand.setWeapon(TakeWeapon::SHIELD);
    hand.attack(); // "방패로 밀친다"

    // 플레이어가 석궁을 들도록 전략 설정
    hand.setWeapon(TakeWeapon::CROSSBOW);
    hand.attack(); // "석궁을 발사하다"

    return 0;
}
```


### 전략 패턴을 사용하여 클린해진 코드
![image](https://github.com/user-attachments/assets/c78054a6-90d6-4cff-98d9-2897de7bb654)

위의 클린하지 않은 코드를 해결하는 가장 좋은 방법은 변경시키고자 하는 행위(전략)를 직접 넘겨주는 것이다.

우선 여러 무기들을 객체 구현체로 정의하고 이들을 Weapon이라는 인터페이스로 묶어 주었다. 

그리고 인터페이스를 컨텍스트 클래스에 합성(composition) 시키고, setWeapon() 메소드를 통해 전략 인터페이스 객체의 상태를 바로바로 변경할 수 있도록 구성 하였다.

클린하지 않는 코드에서는 메서드에 상수값을 넘겨주었지만, 전략 패턴에선 인스턴스를 넣어 알고리즘을 수행하도록 한 것이다.

이런식으로 구성하면 좋은 점은 나중에 칼이나 방패외에 도끼나 창과 같은 전략 무기들을 추가로 등록할때, 코드의 수정없이 빠르게 기능을 확장할 수 있다는 장점이 있다. 
(클래스를 추가하고 implements 해주면 끝)

요약하자면 이 공격 메소드를 각 무기 객체에 국한되게 하는게 아니라, 따로 행위 구현체로 빼서 정의하고 관리하는 것이다. 그리고 이 행위 객체들을 모아 인터페이스로 묶어 하나의 전략 묶움을 구성하고, 이것을 컨텍스트에 합성시켜 다형성을 통해 유기적으로 여러 전략 행위들을 사용할 수 잇도록 하는 것이다.

결국 객체 지향 프로그래밍의 핵심인 유지보수를 용이하게 하기위해, 약간 복잡하더라도 이러한 패턴을 적용하여 프로그램을 구성해 나가는 것이다.

```cpp
#include <iostream>

// 전략 - 추상화된 알고리즘
class Weapon {
public:
    virtual void offensive() const = 0; // 순수 가상 함수
    virtual ~Weapon() = default; // 가상 소멸자
};

class Sword : public Weapon {
public:
    void offensive() const override {
        std::cout << "칼을 휘두르다" << std::endl;
    }
};

class Shield : public Weapon {
public:
    void offensive() const override {
        std::cout << "방패로 밀친다" << std::endl;
    }
};

class CrossBow : public Weapon {
public:
    void offensive() const override {
        std::cout << "석궁을 발사하다" << std::endl;
    }
};

// 컨텍스트 - 전략을 등록하고 실행
class TakeWeaponStrategy {
    std::shared_ptr<Weapon> wp;
public:
    void setWeapon(std::shared_ptr<Weapon> wp) {
        this->wp = wp;
    }

    void attack() const {
        if (wp) {
            wp->offensive();
        } else {
            std::cout << "무기가 설정되지 않았습니다" << std::endl;
        }
    }
};

// 클라이언트 - 전략 제공/설정
int main() {
    // 플레이어 손에 무기 착용 전략을 설정
    TakeWeaponStrategy hand;

    // 플레이어가 검을 들도록 전략 설정
    hand.setWeapon(std::make_shared<Sword>());
    hand.attack(); // "칼을 휘두르다"

    // 플레이어가 방패를 들도록 전략 변경
    hand.setWeapon(std::make_shared<Shield>());
    hand.attack(); // "방패로 밀친다"

    // 플레이어가 석궁을 들도록 전략 변경
    hand.setWeapon(std::make_shared<CrossBow>());
    hand.attack(); // "석궁을 발사하다"

    return 0;
}
```
