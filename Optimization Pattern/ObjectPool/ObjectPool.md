# 객체 풀

**객체를 매번 할당, 해제하지 않고 고정 크기 풀에 들어있는 객체를 재사용함으로써 메모리 사용 성능을 개선한다.**

## 메모리 단편화

힙에 사용 가능한 메모리 공간이 크게 뭉쳐 있지 않고 작게 조각나 있는 상태

단편화가 빈번하지 않더라도 단편화를 그냥 놔두면, 지속적으로 힙에 구멍과 틈을 내서 힙을 사용 못 하게 하고 결국에는 게임을 망가뜨린다.

### 해결 방법

게임이 실행될 때 메모리를 크게 잡고 끝날때까지 들고 있지 

→ 게임 실행 중에 객체를 생성, 제거해야하는 경우 어렵다

## 패턴

재사용 가능한 객체들을 모아놓은 객체 **풀 클래스를 정의**한다. 

여기에 들어가는 객체는 현재 자신이 **사용 중인지 여부**를 알 수 있는 방법을 제공해야 한다. 

풀은 초기화될 때 사용할 객체들을 미리 생성하고, 이들 객체를 사용 안함 상태로 초기화한다

새로운 객체가 필요하면 **풀에 요청**한다. 

풀은 사용 가능한 객체를 찾아서 **사용 중으로 초기화한 뒤 반환**한다. 

객체를 더 이상 사용하지 않는다면 사용 안 함 상태로 되돌린다.

이런 식으로 메모리나 다른 자원 할당을 신경 쓰지 않고 마음껏 객체를 생성, 삭제할 수 있다.

## 언제 쓸 것인가?

보통 오브젝트나 파티클같이 시각적으로 보이는 것에 많이 사용된다.

- 객체를 빈번하게 생성/삭제해야 한다.
- 객체들의 크기가 비슷하다.
- 객체를 힙에 생성하기가 느리거나 메모리 단편화가 우려된다.
- DB 연결이나 네트워크 연결같이 접근 비용이 비싸면서 재사용 가능한 자원을 객체가 캡슐화하고 있다.

## 주의 사항

가비지 컬렉터나 new, delete로 메모리를 관리한다. 객체 풀을 사용하는 것은 **메모리를 어떻게 처리할지를 내가 더 잘 안다**라고 선언하는 것인다 

**→ 객체 풀의 한계를 우리가 직접 해결해야한다.**

### 객체 풀에서 사용되지 않는 객체는 메모리 낭비와 다를 바 없다.

- 객체 풀은 필요에 따라 크기를 조절해야한다
    - 너무 작으면 크래시 너무 크면 메모리 낭비이다.

### 한 번에 사용 가능한 객체 개수가 정해져 있다

- 객체 풀의 모든 객체가 사용 중이어서 재사용할 객체를 반환받지 못할 때를 대비해야한다.
    - 이런 일이 아예 생기지 않게 한다
        - 사용자가 풀이 절대 부족하지 않도록 풀의 크기를 조절한다.
    - 그냥 객체를 생성하지 않는다.
        - 모든 파티클이 사용중이라는 것은 화면에 많은 파티클이 있다는 것
        - 새로운 폭발 이펙트는 눈에 띄지 않는다.
    - 기존 객체를 강제로 제거한다.
        - 이미 재생 중인 사운드 중에서 가장 소리가 작은 것을 새로운 사운드로 교체하자
        - 기존 객체가 사라지는 것이 새로 나와야 할 객체가 안보이는 것보다 눈에 덜 띌때 사용한다.
    - 풀의 크기를 늘린다
        - 런타임에 풀의 크기를 늘리거나 보조 풀을 만든다.

### 객체를 위한 메모리 크기는 고정되어 있다.

- 대부분 객체 풀은 배열로 관리하도록 구현한다.
- 풀에 들어가는 객체가 전부 같은 타입이면 상관없지만 크기가 다양한 서로 다른 타입이면 메모리를 낭비하게 된다
- 객체의 크기별로 풀을 나누는 것이 좋다

### 재사용 되는 객체는 저절로 초기화되지 않는다.

- 풀에서 새로운 객체를 초기화할 때는 주의해서 객체를 완전히 초기화해야한다.

### 사용 중이지 않는 객체도 메모리에 남아있다.

- 가비지 컬렉션을 지원하는 시스템에서는 GC가 메모리 단편화를 알아서 처리하기 때문에 객체 풀을 덜 쓰는 편이다.
- 모바일 같이 CPU가 느리고 단순한 GC만 지원하는 곳에서는 객체 풀로 메모리 할당, 해제 비용을 줄이는 게 의미가 있다.
- GC와 객체 풀을 같이 사용한다면 충돌에 주의해야한다.
    
    → 객체가 다른 객체를 참조하고 있으면 GC에서 객체를 회수할 수 없기 때문이다.
    
- 풀에 있는 객체를 더 이상 사용하지 않을 때 객체에서 다른 객체를 참조하는 부분을 전부 정리해야한다.

## 예시 코드

```cpp
class Particle {
public:
    Particle() : frameLeft_(0) {} //사용 안함으로 초기화
    void init(double x, double y, double xVel, double yVel, int lifetime);
    void animate();
    bool inUse() const { return frameLeft_ > 0; }
    
private:
    int frameLeft_;
    double x_, y_;
    double xVel_, yVel_;
};

void Particle::init(double x, double y, double xVel, double yVel, int lifetime) {
    x_ = x;
    y_ = y;
    xVel_ = xVel;
    yVel_ = yVel;
    frameLeft_ = lifetime; //사용중 상태로 변경
}
```

```cpp
void Particle::animate() { // 업데이트 메서드 패턴 사용
    if (!inUse()) return;
    
    frameLeft_--;
    x_ += xVel_;
    y_ += yVel_;
}
```

- 기본 생성자에서 파티클 사용 안함으로 초기화하고 init()함수에서 파티클 사용중 상태로 바꾼다.
- frameLeft 변수로 현재 파티클이 사용중인지 확인한다.

```cpp
class ParticlePool {
public:
    void create(double x, double y, double xVel, double yVel, int lifetime);
    void animate();
    
private:
    static const int POOL_SIZE = 100;
    Particle particles_[POOL_SIZE];
};

void ParticlePool::animate() {
    for (int i = 0; i < POOL_SIZE; ++i)
        particles_[i].animate();
}

void ParticlePool::create(double x, double y, double xVel, double yVel, int lifetime) {
    for (int i = 0; i < POOL_SIZE; ++i) {
        if (!particles_[i].inUse()) {
            particles_[i].init(x, y, xVel, yVel, lifetime);
            return;
        }
    }
}
```

- 사용 가능한 파티클을 찾을 때까지 풀을 순회하고 찾으면 해당 파티클을 초기화한다.
- 파티클은 생명주기가 끝날때 스스로 비활성화된다.
- 사용 가능한 객체를 찾기 위해 매번 순회해야한다는 문제가 있다

### 빈칸 리스트 기법

- 사용 가능한 파티클을 찾느라 전체 컬렉션을 순회하며 낭비하고 싶지 않으면 사용 가능한 파티클을 계속 추적해야 한다.
- 사용 가능한 파티클 포인터를 별도의 리스트에 저장하는 것도 한 방법이지만 풀에 들어 있는 객체 개수만큼의 포인터가 들어 있는 리스트를 따로 관리해야 한다.

리스트를 따로 관리하는 대신 사용하지 않는 파티클 객체의 데이터 일부를 활용할 수 있다.

```cpp
class Particle {
public:
    Particle* getNext() const { return state_.next; }
    void setNext(Particle* next) {
        state_.next = next;
    }
    
private:
    int frameLeft_;
    
    union {
        struct {
            double x, y;
            double xVel, yVel;
        } live;
        
        Particle* next;
    } state_;
};
```

파티클이 활성화 된 동안에는 **구조체의 데이터를 사용**하고 비활성화 되었을 때는 구조체의 데이터가 필요 없기 때문에 **포인터로 사용 가능한 다음 파티클을 가리키면** 추가 메모리 낭비 없이 리스트를 만들 수 있다.

```cpp
class ParticlePool {
    // ...
private:
    Particle* firstAvailable_; // head
};

ParticlePool::ParticlePool() {
    firstAvailable_ = &particles_[0];
    
    for (int i = 0; i < POOL_SIZE - 1; ++i)
        particles_[i].setNext(&particles_[i+1]);
    
    particles_[POOL_SIZE-1].setNext(nullptr);
}
```

- 풀을 최초 생성시에는 모든 파티클이 사용 가능하기 때문에 배열 전체를 관통하여 리스트를 만든다.

```cpp
void ParticlePool::create(double x, double y, double xVel, double yVel, int lifetime) {
    // 풀이 비어 있는지 확인
    assert(firstAvailable_ != nullptr);
   
    Particle* newParticle = firstAvailable_;
    firstAvailable_ = newParticle->getNext();
    newParticle->init(x, y, xVel, yVel, lifetime);
}
```

- 새로운 파티클 생성시에는 첫 번째로 사용 가능한 파티클의 포인터를 바로 얻어온다.

```cpp
bool Particle::animate() {
    if (!inUse()) return false;
    
    frameLeft_--;
    x_ += xVel_;
    y_ += yVel_;
    
    return frameLeft_ == 0;
}
```

- 파티클 생명 주기 알 수 있도록 반환값을 bool로 변경

```cpp
void ParticlePool::animate() {
    for (int i = 0; i < POOL_SIZE; ++i) {
        if (particles_[i].animate()) {
            particles_[i].setNext(firstAvailable_);
            firstAvailable_ = &particles_[i];
        }
    }
}
```

- 파티클이 죽으면 빈칸 리스트에 다시 넣는다.

## 디자인 결정

### 풀이 객체와 커플링 되는가?

**객체가 풀과 커플링 된다면**

- 더 간단하게 구현 가능
- 객체가 풀을 통해서만 생성할 수 있도록 강제할 수 있다.

**객체가 풀과 커플링 되지 않는다면**

- 어떤 객체라도 풀에 넣을 수 있다.
- ‘사용중’ 상태를 객체 외부에서 관리해야한다 → 비트 필드를 따로 두는 것

### 재사용되는 객체를 초기화하는 장소

**객체를 풀 안에서 초기화한다면**

- 풀은 객체를 완전히 캡슐화할 수 있다.
- 풀 클래스는 객체가 초기화되는 방법과 결합된다.

**객체를 밖에서 초기화한다면**

- 풀의 인터페이스는 단순해진다.
- 객체 생성이 실패할 때의 예외처리를 구현해야한다.
