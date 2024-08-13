# 캐시는 모든 곳에 있다.

- CPU 의 L1, L2, L3 캐시
- 운영체제의 페이지 / 디스크 캐시
- DB 를 위한 MemCached, Redis 와 같은 캐시
- API 캐시
- 7계층 (애플리케이션 계층) HTTP 캐시 (CDN)
- DNS 캐싱
- 브라우저 안에 들어있는 캐시
- 복잡하고 시간 집약적인 작업을 위해 존재하는 마이크로서비스 내부에 캐싱
- 등등

# 데이터 지역성이 필요한 이유
>[게임 프로그래밍 패턴 17장](https://gameprogrammingpatterns.com/data-locality.html) p.326
- 캐시 최적화는 굉장히 큰 주제다. 
- 캐시 때문에, 데이터를 어떻게 두느냐가 성능에 직접적인 영향을 미친다. 
- 자료구조를 잘 만들어서 처리하려는 값이 메모리 내에서 서로 가까이 붙어 있도록 하는 것이 목표다.

즉, CPU 캐시를 최대한 활용할 수 있도록 데이터를 배치해 메모리 접근 속도를 높이기 위해 사용한다.

아래 표 출처 : [모두의 코드](https://modoocode.com/315)
![](https://velog.velcdn.com/images/strurao/post/4391a4ff-bd37-460d-a972-ef0e97e672bb/image.png)

아래 글 출처 : https://kimtaehyun98.tistory.com/48
![](https://velog.velcdn.com/images/strurao/post/2d40a743-0ba4-4416-adae-e76c8c13de29/image.png)

>그리고 이 패턴이 필요한 또다른 이유는 
- 캐시는 어디에나 있지만, 프로그래머는 캐시를 조작할 수 있는 명령어가 없어서라고 생각한다.
- HW 캐시는 제한되어 있다. 이 제한되어 있는 캐시를 효율적으로 사용하기 위해 프로그래머의 역량이 필요하다?

## 캐시 라인 (cache line) 이란?

![](https://velog.velcdn.com/images/strurao/post/bd9dacd3-c2ea-4b26-906a-6cfc4382ffe8/image.png)
**RAM은 (보통 64~128바이트 정도의) 연속된 메모리를 선택해 캐시에 복하한다. 이런 메모리 덩어리를 캐시 라인이라고 한다.** 

다음에 필요한 데이터가 캐시 라인 안에 들어 있다면 CPU는 RAM보다 훨씬 빠른 캐시로부터 데이터를 바로 가져온다.
이 때 캐시에서 원하는 데이터가 있다면 캐시 히트 (cache hit), 데이터를 찾지 못해 주메모리에서 데이터를 가져오는 것을 캐시 미스 (cache miss) 라고 한다.

캐시 미스가 발생하면, CPU는 멈춘다.

실제 캐시에는 더 많은 캐시 라인을 넣을 수 있다. 캐시 집합 (cache associativity)를 검색해보자.


칩이 어떤 데이터를 읽을 때마다 캐시 라인을 같이 읽어온다. 캐시 라인에 있는 값을 더 많이 사용할수록 더 빠르게 만들 수 있다. 즉, 자료구조를 잘 만들어서 처리하려는 값이 메모리 내에서 서로 가까이 붙어 있도록 하는 것이 목표다.

_(참고: 이 책에서는 싱글 스레드를 기준으로 설명하고 있다. 만약 멀티스레드 애플리케이션에서 근처 데이터를 고쳐야 한다면, 코어마다 다른 캐시 라인에 데이터를 두는 게 빠르다. 두 스레드에서 같은 캐시 라인에 들어 있는 데이터를 고치려 든다면 두 코어 모두 비싼 캐시 동기화 작업을 해야 한다. 관련 키워드: 캐시 일관성 프로토콜)_

### 관련 용어들
> 캐시 집합 (cache associativity)

> 캐시 무효화
- 캐시에서 데이터를 제거하여 캐시를 무효화하는 프로세스
- 캐시를 사용할때는 언제, 그리고 어떻게 데이터를 무효화하고 적절한 무효화 절차를 만드는 것이 중요하다.
- 객체가 캐시된 후 일반적으로 객체가 만료되거나 새 콘텐츠를 위한 공간 확보를 위해 삭제될 때까지 캐시에 남아 있다. 
캐시 무효화를 요청하기 전에 원본 서버가 올바른 콘텐츠를 반환하는지 확인하는 것이 중요하다. 그렇지 않으면 콘텐츠를 다시 요청할 때 잘못된 콘텐츠를 캐시할 수 있다.

> 캐시 뒤엎기 (cache thrash)
- thrash의 뜻: 매질, 몽둥이질 ㄷㄷ
- 캐시 무효화가 계속 반복되는 현상

> 명령어 캐시 (Instruction Cache)

> 캐시 일관성
- [병렬성과 메모리 계층구조 : 캐시 일관성](https://velog.io/@boram_in/5.10-%EB%B3%91%EB%A0%AC%EC%84%B1%EA%B3%BC-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EA%B3%84%EC%B8%B5%EA%B5%AC%EC%A1%B0-%EC%BA%90%EC%8B%9C-%EC%9D%BC%EA%B4%80%EC%84%B1)
- 캐시 메모리를 사용하면 같은 data가 여러 곳의 메모리에 존재하는 경우가 생기는데, 이 여러 곳에 존재하는 data들이 같은 값 또는 유효한 값을 가지도록 하는 것이 중요하다. 이것을 캐시 일관성(cache coherence) 문제라고 한다.

# 언제 쓸 것인가?
최적화 기법인 만큼, 성능 문제가 있을 때 써야 한다. 
특히 데이터 지역성 패턴은 성능 문제가 캐시 미스 때문인지를 확인해야 한다. 다른 이유 때문이라면 데이터 지역성 패턴이 전혀 도움이 안 된다.

보통 간단하게 프로파일링할 때에는 코드 두 지점 사이에 얼마나 시간이 지났는지를 직접 타이머 코드를 넣어서 확인한다. 하지만 캐시 사용량을 확인하려면 좀 더 복잡한 방법이 필요하다. 제대로 하려면 캐시 미스가 얼마나 많이 발생했는지, 어디에서 발생했는지를 알 수 있어야 한다.

다행히 캐시 미스를 분석할 수 있는 프로파일러들이 이미 나와 있다. 대대적으로 자료구조를 뜯어고치기 전에 먼저 프로파일러를 하나 선택해 실행해보고 프로파일러가 뱉어내는 (굉장히 복잡한) 숫자가 무엇을 의미하는지 먼저 이해하는 게 좋다.
안타깝게도 캐시 미스를 측정할 수 있는 프로파일러들은 대부분 비싸다. 콘솔 게임 개발팀에 있다면 팀에 하나쯤은 라이센스가 있을 것이다. 그렇지 않다면 [Cachegrind](https://valgrind.org/docs/manual/cg-manual.html#:~:text=Cachegrind%20is%20a%20high%2Dprecision,diff%20data%20from%20different%20runs.) 라는 끝내주는 무료 툴을 써보자. 이 툴은 우리 프로그램을 가상의 CPU와 캐시 계층 위에서 실행한 뒤에 모든 캐시 상호작용 결과를 알려준다.

분명히 캐시 미스는 게임 성능에 영향을 미친다! 미리부터 캐시 최적화를 하겠다고 많은 시간을 낭비해서는 안 되겠지만, 개발하는 내내 자료구조를 캐시하기 좋게 만들려고 노력할 필요는 있다.

# 주의사항
SW 아키텍처의 전형적인 특징 하나가 **추상화**다. 이 책에서도 코드를 쉽게 바꿀 수 있도록 디커플링해주는 패턴에 많은 분량을 할애했다. 객체지향 언어에서는 거의 언제나 **인터페이스**를 사용해 디커플링한다.

C++에서 인터페이스를 사용하려면 포인터나 레퍼런스를 통해 객체에 접근해야 한다. 하지만 포인터를 쓰게 되면 메모리를 여기저기 찾아가야 하므로 데이터 지역성 패턴을 통해서 피하고자 했던 캐시 미스가 발생한다 !!
(인터페이스를 쓰게 되면 **가상 함수 호출**도 불가피하다! 가상 함수를 호출하려면 CPU가 객체의 vtable에서 실제로 호출할 함수 포인터를 찾아야 한다. 이 과정에서 또다시 포인터 추적이 일어나 캐시 미스가 생길 수 있다)

**따라서 데이터 지역성과 추상화 사이의 절충이 필요하다.**

# 방법, 예시 코드

## 1. 연속 배열 (Contiguous arrays)
게임 개체는 각 컴포넌트(AI,물리,렌더링 등)에 몇몇 상태와 이를 업데이트하기 위한 메서드가 들어 있다.
```cpp
class GameEntity
{
public:
  GameEntity(AIComponent* ai,
             PhysicsComponent* physics,
             RenderComponent* render)
  : ai_(ai), physics_(physics), render_(render)
  {}

  AIComponent* ai() { return ai_; }
  PhysicsComponent* physics() { return physics_; }
  RenderComponent* render() { return render_; }

private:
  AIComponent* ai_;
  PhysicsComponent* physics_;
  RenderComponent* render_;
};
```
```cpp
class AIComponent
{
public:
  void update() { /* Work with and modify state... */ }

private:
  // Goals, mood, etc. ...
};

class PhysicsComponent
{
public:
  void update() { /* Work with and modify state... */ }

private:
  // Rigid body, velocity, mass, etc. ...
};

class RenderComponent
{
public:
  void render() { /* Work with and modify state... */ }

private:
  // Mesh, textures, shaders, etc. ...
};
```
월드에 있는 모든 개체는 거대한 포인터 배열 하나로 관리하는데, 매번 게임 루프를 돌 때마다 모든 개체의 컴포넌트(AI, 물리, 렝더링)를 업데이트한다.

```cpp
while (!gameOver)
{
  // Process AI.
  for (int i = 0; i < numEntities; i++)
  {
    entities[i]->ai()->update();
  }

  // Update physics.
  for (int i = 0; i < numEntities; i++)
  {
    entities[i]->physics()->update();
  }

  // Draw to screen.
  for (int i = 0; i < numEntities; i++)
  {
    entities[i]->render()->render();
  }

  // Other game loop machinery for timing...
}

```

CPU 캐시를 알기 전에는 이 코드가 전혀 문제가 없어 보이지만, 이런 문제들이 있다.

문제
1. 게임 개체가 배열에 포인터로 저장되어 있어서, 배열 값에 접근할 때마다 포인터를 따라가면서 캐시 미스가 발생한다.
2. 게임 개체는 컴포넌트를 포인터로 들고 있어서 다시 한 번 캐시 미스가 발생한다.
3. 컴포넌트를 업데이트한다.
4. 모든 개체의 모든 컴포넌트에 대해서 같은 작업을 반복한다.

결과 : 결국 컴포넌트 하나를 업데이트할 때마다 캐시미스가 발생할 가능성이 높다.

```cpp
while (!gameOver) {
    for (int i = 0; i < numEntities; ++i)
        entities[i]->ai()->update();

    for (int i = 0; i < numEntities; ++i)
        entities[i]->physics()->update();

    for (int i = 0; i < numEntities; ++i)
        entities[i]->render()->update();
}
```
![](https://velog.velcdn.com/images/strurao/post/67982b25-5d7d-4aea-88f2-5cee20a67199/image.png)

이를 개선해보자.
배열에 컴포넌트 포인터가 아니라 컴포넌트 객체를 넣는 방법으로 개선해보자.

배열에 컴포넌트 객체(포인터X)가 들어갔기 때문에, 바로 다음 객체에 접근할 수 있다.
캡슐화도 유지된다.
포인터 추적을 전부 제거하여 연속된 배열 세 개를 쭉 따라갈 수 있다.

결과 : 앞선 예제에 비해 업데이트 루프가 50배 빨라진다고 한다.

```cpp
AIComponent* aiComponents = new AIComponent[MAX_ENTITIES];
PhysicsComponent* physicsComponents = new PhysicsComponent[MAX_ENTITIES];
RenderComponent* renderComponents = new RenderComponent[MAX_ENTITIES];

while (!gameOver) {
    for (int i = 0; i < numEntities; ++i)
        aiComponents[i].update();

    for (int i = 0; i < numEntities; ++i)
        physicsComponents[i].update();

    for (int i = 0; i < numEntities; ++i)
        renderComponents[i].update();
}
```
![](https://velog.velcdn.com/images/strurao/post/c572731c-49f4-49b5-8ec8-a477ff9c0422/image.png)







## 2. 정렬된 데이터 
모든 파티클을 연속적인 하나의 배열에 두고, 이를 ParticleSystem이라는 관리 클래스로 래핑한다.
```cpp
class Particle
{
public:
  void update() { /* Gravity, etc. ... */ }
  // Position, velocity, etc. ...
};

class ParticleSystem
{
public:
  ParticleSystem()
  : numParticles_(0)
  {}

  void update();
private:
  static const int MAX_PARTICLES = 100000;

  int numParticles_;
  Particle particles_[MAX_PARTICLES];
};
```
아래 업데이트 방식은 매번 전부 처리하는 방법이라서
```cpp
void ParticleSystem::update()
{
  for (int i = 0; i < numParticles_; i++)
  {
    particles_[i].update();
  }
}
```

아래처럼 모든 파티클을 업데이트하지 않고, 활성화된 파티클만 업데이트한다.


```cpp
void particlsSystem::update() {
	 for (int i = 0; i < numParticles_; ++i)
     	if (particles_[i].isActive()) {
            particles_[i].update();
        }
    }
}
```

이번에는 활성/비활성화에 따라 같은 녀석들만 모아두도록 정렬한다.

포인터 추적을 생각하면 swap 과정이 복사 방식이더라도 더 싼 비용일 수 있다.

플래그 값을 캐시에 로딩하면서 나머지 파티클 데이터도 같이 캐시에 올린다.
비활성 파티클이 많이 있을수록 캐시 미스가 많이 발생한다.
즉, 활성 여부를 플래그로 검사하지 말고, 활성 파티클만 맨 앞으로 모아두자! 

```cpp
// 파티클 활성화
void ParticleSystem::activeParticle(int index) {
    assert(index >= numActive_);
	// 비활성 파티클 중 맨 앞 요소와 바꿔서 활성 파티클 맨 뒤로 옮긴다.
    Particle temp = particles_[numActive_];
    particles_[numActive_] = particles_[index];
    particles_[index] = temp;

    numActive_++;
}

// 파티클 비활성화
void ParticleSystem::deactiveParticle(int index) {
    assert(index < numActive_);

    numActive_--;
	// 마지막 활성 파티클과 맞바꿔서, 비활성 파티클 중 맨 앞으로 가게 한다.
    article temp = particles_[numActive_];
    particles_[numActive_] = particles_[index];
    particles_[index] = temp;
}
```

객체 복사가 필요하단 단점이 있으나, 캐시 미스가 발생하지 않을 것이다.
플래그가 필요 없어 객체 크기가 줄어들고, 캐시 라인에 객체를 더 많이 넣을 수 있다.
메모리 복사 비용이 포인터 추적 비용보다 낮을 수 있다. (프로파일링 필요)

단, `Particle` 클래스가 자신의 활성 상태를 스스로 제어할 수 없고, 직접 `activate`과 같은 메서드를 호출할 수가 없다.

반드시 `ParticleSystem`을 거쳐야만 한다.

## 3. hot/cold splitting (빈번한 코드와 한산한 코드 나누기)
어떤 게임 개체의 AI 컴포넌트가 프레임마다 업데이트되는데, 죽었을 때만 드랍되는 데이터는 개체가 죽었을 때에만 딱 한 번 사용된다.

컴포넌트 크기가 커져서 캐시 라인에 들어갈 컴포넌트의 개수가 줄어드므로 아래 코드는 캐시 미스가 더 자주 발생할 가능성이 있다.
```cpp
class AIComponent
{
public:
  void update() { /* ... */ }

private:
  Animation* animation_;
  double energy_;
  Vector goalPos_;
  
   LootType drop_;
  int minDrops_;
  int maxDrops_;
  double chanceOfDrop_;
};
```
그래서 이 때 빈번한 코드와 한산한 코드를 나누는 방법을 사용할 수 있다. 데이터를 두 개로 분리해 한 곳에는 매 프레임마다 필요로 하는 빈번한(hot) 데이터와 상태를 두고, 다른 곳에는 자주 사용하지 않는 (cold) 데이터를 모아두는 방법이다.

```cpp
class AIComponent
{
public:
  // Methods...
private:
  Animation* animation_;
  double energy_;
  Vector goalPos_;

  LootDrop* loot_;
};

class LootDrop
{
  friend class AIComponent;
  LootType drop_;
  int minDrops_;
  int maxDrops_;
  double chanceOfDrop_;
};
```

하지만 데이터를 이렇게 나누기는 굉장히 애매하다. 예제에서는 빈번한 데이터와 한산한 데이터가 분명하게 나뉘지만, 실제 게임 코드에서는 분간하기가 쉽지 않다. 어떤 필드는 개체가 특정 모드일 때에만 사용된다. 특정 레벨에서만 사용되는 데이터도 있을 수 있다.


# 관련자료
이번 장은 많은 부분에서 **컴포넌트** 패턴(14장)과 관련이 있다. 컴포넌트 패턴은 분명 캐시 사용 최적화를 위해 가장 많이 사용되는 자료구조 중 하나다. 개체를 한 번에 한 '분야'(AI, 물리 등등)씩 업데이트하기 때문에, 이를 컴포넌트로 분리하면 큰 덩어리의 개체를 캐시에 적합한 크기로 나눌 수 있어서다.

그렇다고 데이터 지역성 패턴을 꼭 컴포넌트 패턴과 함께 써야 한다는 건 아니다! 성능이 민감한 코드에서 많은 데이터를 작업해야 한다면 데이터 지역성을 고려하는 게 중요하다.

Pitfalls of Object-Oriented Programming 라는 캐시 최적화에 적합한 자료구조 설계를 소개하는 글
- https://harmful.cat-v.org/software/OO_programming/_pdf/Pitfalls_of_Object_Oriented_Programming_GCAP_09.pdf
- https://news.ycombinator.com/item?id=38781277

노엘 로피스의 굉장히 영향력 있는 블로그 글이라고 한다. 많은 사람이 캐시를 활용할 수 있도록 게임 코드를 디자인하게 만든 '데이터 중심 디자인(혹은 OOP의 위험성)'에 관한 글이다.
- [Data-Oriented Design (or why you might be shooting yourself in the foot with oop)](https://gamesfromwithin.com/data-oriented-design)

데이터 지역성 패턴에서는 거의 언제나 단일 자료형을 하나의 연속된 배열에 나열하는 방식을 활용하고 있다. 객체는 시간이 지남에 따라 배열에 추가되거나 제거된다. 이런 면에서는 객체 풀 패턴이라고도 볼 수 있다. (라고 한다.)

# 찾아본 실례

데이터를 최적화해 메모리에 효율적으로 저장되도록 만들면 성능이 향상하는 효과를 누릴 수 있다. 클래스를 구조체로 대체하면 데이터를 캐시하기가 더 쉬워진다. Unity의 ECS 및 DOTS 아키텍처는 이 패턴을 구현합니다.  - [출처](https://unity.com/kr/resources/level-up-your-code-with-game-programming-patterns)


# 같이 보면 좋을 것 같은 자료
[모두의 코드 C++ memory order와 atomic 객체](https://modoocode.com/271)

[컴퓨터 아키텍처(우종정, 한빛 아카데미)](https://kimtaehyun98.tistory.com/48)

[캐싱과 캐시 무효화에 대한 간단한 글](https://velog.io/@zenon8485/%EC%BA%90%EC%8B%B1%EA%B3%BC-%EC%BA%90%EC%8B%9C-%EB%AC%B4%ED%9A%A8%ED%99%94%EC%97%90-%EB%8C%80%ED%95%9C-%EA%B0%84%EB%8B%A8%ED%95%9C-%EA%B8%80)

[(번역) 캐시 시스템 설계할 때 기억해야 할 6가지 캐싱 전략
](https://soobing.github.io/cs/6-caching-strategies/)