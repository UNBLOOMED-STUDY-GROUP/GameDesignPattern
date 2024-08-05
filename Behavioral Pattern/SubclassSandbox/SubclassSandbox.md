# 하위 클래스 샌드박스

**상위 클래스가 제공하는 기능들을 통해서 하위 클래스에서 행동을 정의한다.**

## 상황

슈퍼히어로 게임에서는 수십 개가 넘는 다양한 초능력을 선택할 수 있어야 한다.

먼저 Superpower라는 상위 클래스를 만든 후에 초능력별로 이를 상속받는 클래스를 정의하려고 한다. 기획서를 나눠서 받은 프로그래머들이 구현을 마치고 나면 수십 개가 넘는 초능력 클래스가 만들어져 있을 것이다. 

유저가 어릴 때 꿈꿔왔던 어떤 초능력이라도 모두 쓸 수 있는 풍부한 게임 월드를 제공하고 싶다. 이를 위해서는 Superpower를 상속받은 초능력 클래스에서 사운드, 이펙트, AI와의 상호작용, 다른 게임 객체의 생성과 파괴, 물리 작용 등과 같은 모든 일을 할 수 있어야 한다. 그렇게 되면 온갖 코드를 건드려야 한다.

### 이런 식으로 개발하게 된다면!?

- 중복 코드가 많아진다.
    - 많은 초능력이 이펙트와 사운드를 같은 방식으로 출력한다 → 서로 비슷하게 된다.
- 거의 모든 게임 코드가 초능력 클래스와 커플링 된다.
    - 초능력 클래스들을 다 구현해 놓고 보면 모든 렌더러 계층에 중구난방으로 접근하게 해놨을 것이다.
- 외부 시스템이 변경되면 초능력 클래스가 깨질 가능성이 높다.
    - 여러 초능력 클래스가 게임 내 다양한 코드와 커플링 되다 보니 이런 코드가 변경될 때 초능력 클래스에도 영향을 미친다.
- 모든 초능력 클래스가 지켜야 할 불변식을 정의하기 어렵다.
    - 사운드를 항상 큐를 통해 적절하게 우선순위를 맞추고 싶지만 많은 클래스가 사운드 엔진에 직접 접근하면 강제하기 쉽지 않다.

### 개선 방법

초능력 클래스를 구현하는 프로그래머가 사용할 원시명령 집합을 제공하자

이를 위해 원시 명령을 Superpower의 protected 메서드로 만들어 모든 하위 초능력 클래스에서 쉽게 접근할 수 있게 하자 (하위 클래스 용이라고 알려주기 위해)

하위 클래스가 구현해야하는 샌드박스 메서드를 순수 가상 메서드로 만들어 protected로 두자

1. **Superpower를 상속받는 새로운 클래스를 만든다.**
2. **샌드박스 메서드인 activate()를 오버라이드 한다.**
3. **Superpower 클래스가 제공하는 protected 메서드를 호출하여 activate()를 구현한다.**

## 언제 사용하면 좋을까?

클래스에 protected인 비-가상 함수가 있다면 이 패턴을 쓰고 있을 가능성이 높다.

- 클래스 하나에 하위 클래스가 많이 있다.
- 상위 클래스는 하위 클래스가 필요로 하는 기능을 전부 제공할 수 있다.
- 하위 클래스 행동 중에 겹치는 게 많아, 이를 하위 클래스끼리 쉽게 공유하고 싶다.
- 하위 클래스들 사이의 커플링 및 하위 클래스와 나머지 코드와의 커플링을 최소화하고 싶다.

## 예제 코드

```cpp
class Superpower
{
public:
    virtual ~Superpower() {}

protected:
		double getHeroX() { // 코드... }
    double getHeroY() { // 코드... }
    double getHeroZ() { // 코드... }
    
    virtual void activate() = 0;
    void move(double x, double y, double z)
    {
        // 코드...
    }
    void playSound(SoundId sound, double volume)
    {
        // 코드...
    }
    void spawnParticles(ParticleType type, int count) 
    {
        // 코드...
    }
};
```

```cpp
class SkyLaunch : public Superpower
{
protected:
    virtual void activate()
    {
        if(getHeroZ() == 0) 
        {
            // 땅이라면 공중으로 뛴다.
            playSound(SOUND_SPROING, 1.f);
            spawnParticles(PARTICLE_DUST, 10);
            move(0, 0, 20);
        }
        else if(getHeroZ() < 10.f)
        {
            // 거의 땅에 도착했다면 이중 점프를 한다.
            playSound(SOUND_SWOOP, 1.f);
            move(0, 0, getHeroZ() - 20);
        }
        else
        {
            // 공중에 높이 떠 있다면 내려찍기 공격을 한다.
            playSound(SOUND_DIVE, 0.7f);
            spwanParticles(PARTICLE_SPARKLES, 1);
            move(0, 0, -getHeroZ());
        }
    }
};
```

SkyLaunch 클래스에서 상위 클래스 메서드를 사용할 수 있다.

## 어떤 기능을 제공해야 하는가?

하위 클래스 샌드박스 패턴은 상당히 부드러운 패턴이다. 

→ 기본 아이디어는 제공하되 구체적인 부분은 많이 언급하지 않는다. 

→ 패턴을 적용할 때 선택 거리가 있다. 

- **제공 기능을 몇 안 되는 하위 클래스에서만 사용한다면 별 이득이 없다.**
    
    모든 하위 클래스가 영향을 받는 상위 클래스의 복잡도는 증가하는 반면, 혜택을 받는 클래스는 몇 안되기 때문이다.
    
- **다른 시스템의 함수를 호출할 때에도 그 함수가 상태를 변경하지 않는다면 크게 문제가 되지 않는다.**
    
    커플링은 생기겠지만 게임 내에서 다른 걸 망가뜨리지 않는다는 점에서 안전한 커플링이다.
    
- **제공 기능이 단순히 외부 시스템으로 호출을 넘겨주는 일밖에 하지 않는다면** 있어봐야 좋을 게 없다. 그럴 때는 **하위 클래스에서 외부 메서드를 직접 호출**하는 게 더 깔끔할 수도 있다.

### 메서드를 직접 제공할 것인가, 이를 담고 있는 객체를 통해서 제공할 것인가

하위 클래스 샌드박스 패턴의 골칫거리 중 하나는 상위 클래스 메서드 수가 끔찍하게 늘어난다는 점이다. 이들 메서드 일부를 다른 클래스로 옮기면 이런 문제를 완화할 수 있다. 상위 클래스의 제공 기능에서는 이들 객체를 반환하기만 하면 된다.

```cpp
class Superpower
{
protected:
    void playSound(SoundId sound, double volume) { // 코드... }
    void stopSound(SoundId sound) { // 코드... }
    void setVolume(SoundId sound) { // 코드... }
    
    // 샌드박스 메서드와 그 외 다른 기능들....
};
```

```cpp
// 보조 클래스
class SoundPlayer 
{
protected:
    void playSound(SoundId sound, double volume) { // 코드... }
    void stopSound(SoundId sound) { // 코드... }
    void setVolume(SoundId sound) { // 코드... }
    
    // 샌드박스 메서드와 그 외 다른 기능들....
};
```

```cpp
class Superpower
{
protected:
    SoundPlayer& getSoundPlayer() { return soundPlayer_; }
    
    // 샌드박스 메서드와 그 외 다른 기능들...
    
private:
    SoundPlayer soundPlayer_;
};
```

**장점**

- 상위 클래스의 메서드 개수를 줄일 수 있다.
- 보조 클래스에 있는 코드가 유지 보수하기 더 쉬운 편이다.
- 상위 클래스와 다른 시스템과의 커플링을 낮출 수 있다.

### 상위 클래스는 필요한 객체를 어떻게 얻는가?

**상위 클래스를 생성자로 받기**

```cpp
class Superpower
{
public:
    Superpower(ParticlesSystem* particles) : particles_(particles) {}
    // 그 외 기능들...
    
private:
    ParticleSystem* particles_;
};
```

모든 하위 클래스 생성자는 파티클 시스템을 인수로 받아서 상위 클래스 생성자에 전달해야 한다. **원치 않게 모든 하위 클래스에 상위 클래스의 상태가 노출**된다. 

상위 클래스에 다른 **상태를 추가**하면 하위 클래스 생성자도 해당 상태를 전달하도록 **전부 바꿔야 하기 때문에 유지 보수하기에도 좋지 않다.**

**2단계 초기화**

초기화를 2단계로 나누면 생성자로 모든 상태를 전달하는 번거로움을 피할 수 있다.

```cpp
Superpower* power = new SkyLaunch();
power->init(particles);
```

SkyLaunch 클래스 생성자에는 인수가 없기 때문에 Superpower 클래스가 private으로 숨겨놓은 멤버 변수와 전혀 커플링 되지 않는다. 단, 까먹지 말고 init()를 호출해야 한다.

```cpp
Superpower* createSkyLaunch(ParticlesSystem* particles) 
{
    Superpower* power = new SkyLaunch();
    power->init(particles);
    return power;
}
```

객체 생성 과정 전체를 한 함수로 캡슐화하면 해결할 수 있다.

**정적 객체로 만들기**

파티클 시스템이 싱글턴이라면 어차피 모든 초능력 인스턴스가 같은 상태를 공유할 것이다.

이럴 때는 상태를 **상위 클래스의 private 정적 멤버 변수**로 만들 수 있다. 여전히 초기화는 필요하지만 인스턴스마다 하지 않고 **초능력 클래스에서 한 번만 초기화**하면 된다.

**서비스 중개자를 이용하기**

앞에서는 상위 클래스가 필요로 하는 객체를 먼저 넣어주는 작업을 밖에서 잊지 말고 해줘야 했다. 즉, **초기화 부담을 외부 코드에 넘기고 있다.** 만약 **상위 클래스가 원하는 객체를 직접 가져올 수 있다면 스스로 초기화**할 수 있다. 이런 방법 중 하나가 **서비스 중개자 패턴**이다.

```cpp
class Superpower
{
protected:
    void spawnParticles(ParticleType type, int count)
    {
        ParticleSystem& particles = Locator::getParticles();
        particles.spawn(type, count);
    }
    // 그 외...
};
```
