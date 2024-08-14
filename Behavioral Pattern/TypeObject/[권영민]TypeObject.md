## 의도

- 클래스 하나를 인스턴스별로 다른 객체형으로 표현할 수 있게 만들어, 새로운 `클래스`들을 유연하게 만들 수 있게 한다.

## 동기

- 몬스터 특징을 나타내는 종족을 다양하게 만들고 싶다.
- 종족은 몬스터의 최대 체력, 공격 문구, 공격하는 방식을 결정한다.

### 전형적인 OOP 방식

상위 클래스 `Monster`  

- 생성자는 protected이고 몬스터의 최대 체력을 인수로 받는다.

```cpp
class Monster
{
public:
	virtual ~Monster() {}
	virtual const char* GetAttack() = 0;
protected:
	Monster(int StartingHealth) : Health(StartingHealth) {}
private:
	int Health;
};
```

종족을 표현한 하위 클래스들

- 각각의 종족을 정의한 하위 클래스에서는 public 생성자를 정의해 상위 클래스의 생성자를 호출하면서 종족에 맞는 최대 체력을 인수로 전달한다.

```cpp
class Dragon : public Monster
{
public:
	Dragon() : Monster(230) {}
	virtual const char* GetAttack() { return "용이 불을 뿜습니다!"; }
};
```

```cpp
class Troll: public Monster
{
public:
	Dragon() : Monster(48) {}
	virtual const char* GetAttack() { return "트롤이 공격합니다!"; }
};
```

- Monster의 하위 클래스는 몬스터의 최대 체력을 전달하고, GetAttack을 오버라이드해서 종족에 맞는 문구를 반환한다.
- 이런 식으로 몬스터 클래스를 만들다보면, Monster 하위 클래스가 굉장히 많아져 있을 것이다.
- Monster하위 클래스를 계속 작성한 후 컴파일 해야 한다.

## 클래스를 위한 클래스

- 몇몇 속성은 여러 몬스터가 공유하게 만들면 된다.
- 몬스터를 종족이라고 정의하고, 종족이 같으면 공격 문구를 같게 만든다.

### 문제 상황 : 클래스 상속으로 구현
![image](https://github.com/user-attachments/assets/38c3b6f2-7251-4dc4-ae03-c2aee1972c2f)

- 종족을 늘릴때마다 코드를 추가하고 컴파일해야하는 문제가 있다.

### 해결 방법 1. 몬스터마다 종족에 대한 정보를 둔다.
![image](https://github.com/user-attachments/assets/8df534db-808e-4c55-9249-4a248e481d0c)

- 종족마다 Monster 클래스를 상속받게 하지 않고, Mosnter 클래스 하나와 Breed 클래스 하나만 만든다.
- 상속 없이 클래스 두개만으로 해결한다.
- Breed클래스에는 종족이 같은 몬스터가 공유하는 정보인 최대 체력과 공격 문구가 들어 있다.
- 몬스터와 종족을 결합하기 위해 모든 monster 인스턴스는 종족 정보를 담고 있는 Breed 객체를 참조한다.
- Breed 클래스는 본질적으로 몬스터 타입을 정의한다.
- 각각의 종족 객체는 개념적으로 다른 타입을 의미한다.

## 타입 객체 패턴

- 타입 객체 패턴은 코드 수정 없이 새로운 타입을 정의할 수 있다.
- 코드에서 클래스 상속으로 만들던 타입 시스템 일부를 런타임에 정의할 수 있는 데이터로 옮긴 것이다.

### 구현 과정

- 타입 객체 클래스와 타입 사용 객체 클래스를 정의한다.
- 모든 타입 객체 인스턴스는 논리적으로 다른 타입을 의미한다
- 타입 사용 객체는 자신의 타입을 나타내는 타입 객체를 참조한다.

### 언제

- 다양한 종류를 정의해야 하는데 개발 언어의 타입 시스템이 유연하지 않아 코드로 표현하기 어려울 때 적합하다
    - 나중에 어떤 타입이 필요할지 알 수 없다
    - 컴파일이나 코드 변경 없이 새로운 타입을 추가하거나 타입을 변경하고 싶다.

### 주의 사항

**타입 객체를 직접 관리해야 한다**

- 몬스터 인스턴스뿐만 아니라 타입 객체도 직접 관리해야 한다. 타입 객체를 생성하고, 이를 필요로 하는 몬스터가 있는 한 메모리에 유지해야 한다.
- 몬스터 인스턴스를 생성할 때 말았던 종족 객체 레퍼런스로 초기화하는 것도 우리의 몫이다.
- 컴파일러의 한계를 일부 뛰어넘는 대가로 컴파일러가 해주던 일을 우리가 직접 구현해야 한다.

**타입별로 동작을 표현하기가 더 어렵다**

- 타입 객체로 타입 종속적인 데이터를 정의하기는 쉽지만 타입 종속적인 동작을 정의하기는 어렵다.
- 한계 극복 방법
    - 미리 동작 코드를 여러 개 정의해놓은 뒤에 타입 객체 데이터에서 이 중 하나를 선택하는 것
    
    > 몬스터 AI 상태가 ‘날카로운 이빨’일지, ‘무서워서 벌벌 떨기’(예를, 몬스터가 전혀 쓸 데 없이 겁에 질린다) 중에서 하나라고 해보자. 먼저 이들 동작을 각각 구현한 함수를 정의한다. 타입 객체가 적당한 함수 포인터를 저장하게 하면 타입 객체로 AI 알고리즘과 연계하는 것
    > 
    - 데이터만으로 동작을 정의
    
    > 바이트코드 패턴(11장)과 GoF의 인터프리터 패턴을 이용하면 동작을 표현하는 객체를 만들 수 있다. 파일에서 데이터를 읽어 이들 패턴으로 자료구조를 만들면 동작 정의를 코드에서 데이터로 완전히 옮길 수 있다.
    > 

## 예제 1 - 타입 객체 기본 구현

```cpp
class Breed {
public:
    Breed(int health, const char* attack)
        : health_(health),
          attack_(attack) {
    }
    int getHealth() { return health_; }
    const char* getAttack() { return attack_; }

private:
    int health_;  // 최대(초기) 체력
    const char* attack_;
};
```

```cpp
class Monster {
public:
    Monster(Breed& breed)
        : health_(breed.getHealth()),
          breed_(breed) {
    }
    const char* getAttack() { return breed_.getAttack(); }

private:
    int health_;  // 현재 체력
    Breed& breed_;
};

```

- Monster 클래스 생성자는 Breed 객체를 레퍼런스로 받는다.
    - 이를 통해 상속 없이 몬스터 종족을 정의한다.
- 최대 체력은 생성자에서 breed 인수를 통해 얻는다.
- 공격 문구는 breed_에 포함해서 얻는다.

## 예제 2 - 생성자 함수를 통해 타입 객체를 좀 더 타입같이 만들기

- 이제까지는 몬스터를 만들고 그 몬스터에 맞는 종족 객체도 직접 전달했다.
- 이런 방식은 메모리를 먼저 할당한 후에 그 메모리 영역에 클래스를 할당하는 것과 다를 바 없다.
- 클래스의 생성자 함수를 호출해 클래스가 알아서 새로운 인스턴스를 생성하게 한다.

```cpp
class Breed {
public:
    Monster* newMonster() {
        return new Monster(*this);
    }
    // 나머지는 동일
};
```

```cpp
class Monster {
    friend class Breed;

public:
    const char* getAttack() { return breed_.getAttack(); }

private:
    Monster(Breed& breed)
        : health_(breed.getHealth()),
          breed_(breed) {
    }
    int health_;  // 현재 체력
    Breed& breed_;
};

```

- 가장 큰 차이점은 Breed 클래스의 `newMonster` 함수다.
    - 이게 팩토리 메서드 패턴의 ‘생성자’다.

**이전 코드**

```cpp
Monster* monster = new Monster(someBreed);
```

**수정된 부분**

```cpp
Monster* monster = someBreed.newMonster();
```

- 객체는 메모리 할당과 초기화 2단계로 생성된다.
- Monster 클래스 생성자 함수에는 필요한 모든 초기화 작업을 다 할 수 있다.
- 예제에서는 breed 객체를 전달하는 게 초기화의 전부지만, 실제 프로젝트에서는 그래픽을 로딩하고, 몬스터 AI를 설정하는 등 다른 초기화 작업이 많이 있을 수 있다.
- 하지만 이런 초기화 작업은 메모리를 할당한 다음에 진행된다.
- 아직 제대로 초기화되지 않은 몬스터가 메모리에 먼저 올라가 있는 것이다.
- 게임에서는 객체 생성 과정을 제어하고 싶을 때가 중요하다.
- 그럴 때는 보통 **커스텀 할당자나 오브젝트 풀링을 이용해 객체가 메모리 어디에 생성될지를 제어**한다.
- Breed 클래스에 ‘생성자’ 함수를 정의하면 이런 로직을 둘 곳이 생긴다.
- 그냥 new를 호출하는 게 아니라 newMonster 함수를 호출하면 Monster 클래스에 초기화 권한을 넘겨주기 전에 메모리 풀이나 커스텀 할당자로 메모리를 가져올 수 있다.
- 몬스터를 생성할 수 있는 유일한 곳이 Breed 클래스 안에 이런 로직을 둘모로서, 모든 몬스터가 정해놓은 메모리 루틴을 따라 생성되도록 강제할 수 있다.

## 예제 3 - 상속으로 데이터 공유하기

단일 상속만 지원

- 클래스가 상위 클래스가 있는 것처럼 종족 객체도 상위 `parent` 종족 객체를 가질 수 있게 만든다.

```cpp
class Breed {
public:
    Breed(Breed* parent, int health, const char* attack)
        : parent_(parent),
          health_(health),
          attack_(attack) {
    }
    int getHealth();
    const char* getAttack();

private:
    Breed* parent_;
    int health_; // 최대 체력
    const char* attack_;
};

```

- Breed 객체를 만들 때 상속 받을 종족 객체를 넘겨준다.
- 상위 종족이 없는 최상위 종족은 `parent`에 NULL을 전달한다.
- 하위 객체는 어떤 속성을 상위 객체로부터 받을지, 자기 값으로 오버라이드할지를 제어할 수 있어야 한다.

### 속성 값을 요청 받을 때마다 동적으로 위임하는 방식

```cpp
int Breed::getHealth() {
// 오버라이딩
if (health_ != 0 || parent_ == NULL) {
return health_;
}
// 상속
return parent_->getHealth();
}

const char* Breed::getAttack() {
// 오버라이딩
if (attack_ != NULL || parent_ == NULL) {
return attack_;
}
// 상속
return parent_->getAttack();
}
```

- 장점
    - 종족이 특정 속성 값을 더 이상 오버라이드하지 않거나 상속받지 않도록 런타임에 바뀔 때 좋다
- 단점
    - 메모리를 더 차지
    - 속성을 반환할 때마다 상위 객체들을 줄줄이 확인해보느라 더 느림

### 카피다운(copy-down) 위임하는 방식

- 종족 속성 값이 바뀌지 않는다면 생성 시점에 바로 상속을 적용
- 객체가 생성될 때 상속 받는 속성 값을 하위 타입으로 복사해서 넣는다.

```cpp
Breed(Breed* parent, int health, const char* attack)
    : health_(health),
      attack_(attack) {
    // 오버라이드하지 않는 속성만 상속받는다.
    if (parent != NULL) {
        if (health_ == 0)
            health_ = parent->getHealth();
        if (attack_ == NULL)
            attack_ = parent->getAttack();
    }
}
```

```cpp
int getHealth() { return health_; }
const char* getAttack() { return attack_; }
```

- 더 이상 상위 종족 객체를 포인터로 들고 있지 않아도 된다.
- 생성자에서 상위 속성을 전부 복사했기 때문에 더 이상 신경 쓰지 않아도 된다.
- 종족 속성 값을 반환할 때는 필드 값을 그대로 쓰면 된다
- Json 같은 파일로 기획자가 더 변경하기 쉽다.
