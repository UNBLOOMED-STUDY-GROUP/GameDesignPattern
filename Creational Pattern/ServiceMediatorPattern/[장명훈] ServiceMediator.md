# 16.1 의도

> 서비스를 구현한 구체 클래스는 숨김 채로 어디에서나 서비스에 접근할 수 있게 한다.
> 

# 16.2 동기

객체나 시스템 중에는 거의 모든 코드에서 사용되는 것들이 있다. 게임 코드 중에서 메모리 할당, 로그, 난수 생성을 쓰지 않는 곳은 찾아보기어렵다. 

이런 시스템은 게임 전체에서 사용 가능해야 하는 , 일종의 서비스라고 볼 수 있다. 예를 들어 오디오 시스템만 해도 ~~메모리 할당 같은 저수준 시스템만큼은 아니지만~~ 여러 게임 시스템과 연결되어 있다.

돌 굴러떨어지는 소리(물리), NPC 총소리(AI), 메뉴 선택 소리(UI) 등등 다양한 곳에서 소리가 쓰인다. 이런 코드에서는 다음과 같이 오디오 시스템을 호출할 수 있어야 한다.

```cpp
// 정적 클래스를 쓸 수도 있고
AudioSystem::playSound(VERY_LOUD_BANG);

// 싱글턴을 쏠 수도 있다.
AudioSystem::instance()->playSound(VERY_LOUD_BANG);
```

하지만 두 경우 모두 강한 커플링도 함께 생긴다. 오디오 시스템을 접근하는 모든 곳에서 AudioSystem이라는 구체 클래스뿐만 아니라 정적 클래스 또는 싱글턴으로 만든 접근 메커니즘까지 직접 참조하게 된다. 

단지 소리를 재생하고 싶은건데 수많은 시스템(물리, AI, UI)에서 AudioSystem주소를 알려주는 것과 같다. 이러지 말고 호출하려는 쪽()에서 전화번호부를 통해서 찾게 하여 **찾을 방법을 한 곳에서 편리하게 관리**해보자.는 것이 서비스 중개자 패턴의 핵심이다.

- p.304 

> 서비스 중개자 패턴은 서비스를 사용하는 코드로부터 서비스 ***제공자***가 누구인지(서비스를 구현한 구체 클래스 자료형이 무엇인지),어디에 있는지(클래스 인스턴스를 어떻게 얻을지)를 몰라도 되게 해준다.
> 

늘 그랬듯이 이해가 안된다. 예제를 보자. 

# 16.3 패턴

| 용어 | 정의 |
| --- | --- |
| 서비스(service) | 여러 기능을 추상 인터페이스로 정의 |
| 서비스 제공자(service provider) | 서비스 인터페이스를 상속받아 구현 |
| 서비스 중개자(service locator) | 서비스 제공자의 실제 자료형과 이를 등록하는 과정은 숨긴채 적절한 서비스 제공자를 찾아 서비스에 대한 접근을 제공 |

# 16.4 예제

15장의 오디오 API부터 시작. 

| 용어 | 클래스 |
| --- | --- |
| 서비스(service) | Audio |
| 서비스 제공자(service provider) | ConsoleAudio |
| 서비스 중개자(service locator) | Locator |

```cpp
// #include<bits/stdc++.h>
#include<vector>
#include<iostream>
using namespace std;

const int VERY_LOUD_BANG = 100;

class Audio 
{
public:
    virtual ~Audio() = default;
    virtual void PlaySound(int SoundID) = 0;
    virtual void StopSound(int SoundID) = 0;
    virtual void StopAllSounds() = 0;
};

class ConsoleAudio : public Audio
{
public:
    virtual void PlaySound(int SoundID) { cout << "Play " << SoundID << "\n"; }
    virtual void StopSound(int SoundID) {}
    virtual void StopAllSounds() {}
};

class Locator 
{
public:
    static Audio* GetAudio() { return Service; }
    static void Provide(Audio* Service_) { Service = Service_; }

private:
    static Audio* Service;
};

Audio* Locator::Service = nullptr;

int main()
{
    // 서비스 : 추상 인터페이스로 정의             Audio
    // 서비스 제공자 :                            ConsoleAudio        
    // 서비스 중개자 :                            Locator

    // 서비스 제공자(ConsoleAudio)를 Locator의 Service로 등록
    ConsoleAudio* consoleAudio = new ConsoleAudio();
    Locator::Provide(consoleAudio);     // Locator의 Service 등록

    // 중개역할 : GetAudio
    // Locator에 등록된 서비스 사용
    Audio* audio = Locator::GetAudio();
    if (audio != nullptr)
    {
        audio->PlaySound(VERY_LOUD_BANG);
    }
    // PlaySound를 호출하는 쪽에서는 Audio라는 추상 클래스만 있을 뿐, ConsoleAudio를 모른다는 게 핵심

    delete consoleAudio;

    return 0;
}
```

p.304에 있던 문장에 대입해보자. 

오디오를 사용하는 코드(main함수)는 오디오 구체 클래스(consoleAudio)가 뭔지, 어디서 얻는지(`Locator::Provide(consoleAudio);`)를 몰라도 되게 해준다.

## Null 객체 패턴

만약 서비스 제공자가 서비스를 등록하기 전에 서비스를 사용하려고 시도하면 NULL을 반환한다. 이때 호출하는 쪽에서 NULL 검사를 하지 않으면 크래시된다.

이럴 때 써먹을 수 있는 널 객체(Null Object) 디자인 패턴이 있따. 핵심은 객체를 찾지 못하거나 만들지 못해 NULL을 반환해야 할 때, 대신 같은 인터페이스를 구현한 특수한 객체를 반환하는 데있다. 이런 특수 객체에는 아무런 구현이 되어 있지 않지만, 객체를 반호나하는 쪽에서 '진짜'객체를 받은 것처럼 안전하게 작업을 진행할 수 있다.

널 객체 패턴을 사용하려면 다음과 같이 '널(null)' 서비스 제공자를 정의한다.

```cpp
class NullAudio :: public Audio 
{
public:
    virtual void playSound(int soundID) { /* 아무것도 하지 않는다. */ }
    virtual void stopSound(int soundID) { /* 아무것도 하지 않는다. */ }
    virtual void stopAllSounds(int soundID) { /* 아무것도 하지 않는다. */ }
};
```

NullAudio는 Audio 서비스 인터페이스를 상속받지만 아무 기능도 하지 않는다. 이제 Locator 클래스를 다음과 같이 바꾼다.

```cpp
class Locator 
{
public:
    static void Initialize() {
        Service = NullService;
    }
    static Audio* GetAudio() { return Service; }
    static void Provide(Audio* Service_) 
    {
        if (Service_ == nullptr)
        {
            Service = NullService;
        }
        else
        {
            Service = Service_;
        }
    }

private:
    static Audio* Service;
    static NullAudio* NullService;
};
```

호출하는 쪽에서는 '진짜' 서비스가 준비되어 있는지를 신경 쓰지 않아도 되고 NULL 반환 처리도 피료 없다. Locator는 항상 유효한 객체를 반환한다는 점을 보장한다.

- 전체 코드
    
    ```cpp
    // #include<bits/stdc++.h>
    #include<vector>
    #include<iostream>
    using namespace std;
    
    const int VERY_LOUD_BANG = 100;
    
    class Audio 
    {
    public:
        virtual ~Audio() = default;
        virtual void PlaySound(int SoundID) = 0;
        virtual void StopSound(int SoundID) = 0;
        virtual void StopAllSounds() = 0;
    };
    
    class ConsoleAudio : public Audio
    {
    public:
        virtual void PlaySound(int SoundID) { cout << "Play " << SoundID << "\n"; }
        virtual void StopSound(int SoundID) {}
        virtual void StopAllSounds() {}
    };
    
    class NullAudio : public Audio
    {
    public:
        virtual void PlaySound(int soundID) { /* 아무것도 하지 않는다. */ }
        virtual void StopSound(int soundID) { /* 아무것도 하지 않는다. */ }
        virtual void StopAllSounds() { /* 아무것도 하지 않는다. */ }
    };
    
    class Locator 
    {
    public:
        static void Initialize() { Service = &NullService; }
        static Audio* GetAudio() { return Service; }
        static void Provide(Audio* Service_) 
        {
            if (Service_ == nullptr)
            {
                Service = &NullService;
            }
            else
            {
                Service = Service_;
            }
        }
    
    private:
        static Audio* Service;
        static NullAudio NullService;
    };
    
    Audio* Locator::Service = nullptr;
    NullAudio Locator::NullService;
    
    int main()
    {
        // 서비스 중개자 초기화
        Locator::Initialize();
    
        // 서비스 제공자(ConsoleAudio)를 Locator의 Service로 등록
        ConsoleAudio* consoleAudio = new ConsoleAudio();
        Locator::Provide(consoleAudio);
    
        // Locator에 등록된 서비스 사용
        Audio* audio = Locator::GetAudio();
        audio->PlaySound(VERY_LOUD_BANG);
    
        // PlaySound를 호출하는 쪽에서는 Audio라는 추상 클래스만 있을 뿐, ConsoleAudio를 모른다는 게 핵심
    
        delete consoleAudio;
    
        return 0;
    }
    ```
    

## 로그 데커레이터

개발하다 보면 게임 코드 안에서 어떤 일이 벌어지는지 알기 위해 원하는 이벤트에 간단하게 로그를 설치해야 할 떄가 있다. 마치 우리가 UE_LOG하듯이 말이다.

이럴 대는 보통 log() 함수를 코드 여기저기에 집어넣는데. 이러다 보면 로그가 너무 많아지는 문제가 생긴다.

원하는 로그만 켰다 껏다 할 수 있고, 최종 빌드에는 로그를 전부 제거할 수 있다면 가장 이상적이다. 조건적으로 로그를 남기고 싶은 시스템이 서비스로 노출되어 있다면, GoF의 데코레이터(decorator) 패턴으로 이를 해결할 수 있다. 사운드 서비스 제공자를 다음과 같이 정의해보자.

```cpp
class LoggedAudio : public Audio
{
public:
    LoggedAudio(Audio& wrapped_) : wrapped(wrapped_) {}
    void PlaySound(int SoundID) override
    {
        log("사운드 출력");
        wrapped.PlaySound(SoundID);
    }
    void StopSound(int SoundID) override
    {
        log("사운드 중지");
        wrapped.StopSound(SoundID);
    }
    void StopAllSounds() override
    {
        log("모든 사운드 중지");
        wrapped.StopAllSounds();
    }

private:
    void log(const char* message) { cout << message << endl; }
    Audio& wrapped;
};
```

LoggedAudio 클래스는 다른 오디오 서비스 제공자를 래핑하는 동시에 같은 인터페이스를 상속받는다. 실제 기능 요청은 내부에 있는 서비스에 전달하고, 사운드가 호출될 때마다 로그를 남긴다. 

오디오 로그 기능을 켜고 싶다면 다음과 같이 하면 된다.

```cpp
void EnableAudioLogging()
{
    // 기존 서비스를 데코레이트한다.
    Audio* service = new LoggedAudio(*Locator::GetAudio());
    // 이 값으로 바꿔치기한다.
    Locator::Provide(service);
}
```

전체 코드를 보고 파악해보자.

```cpp
#include <vector>
#include <iostream>
using namespace std;

const int VERY_LOUD_BANG = 100;

class Audio
{
public:
    virtual ~Audio() = default;
    virtual void PlaySound(int SoundID) = 0;
    virtual void StopSound(int SoundID) = 0;
    virtual void StopAllSounds() = 0;
};

class ConsoleAudio : public Audio
{
public:
    void PlaySound(int SoundID) override { cout << "Play " << SoundID << "\n"; }
    void StopSound(int SoundID) override {}
    void StopAllSounds() override {}
};

class NullAudio : public Audio
{
public:
    void PlaySound(int SoundID) override { /* 아무것도 하지 않는다. */ }
    void StopSound(int SoundID) override { /* 아무것도 하지 않는다. */ }
    void StopAllSounds() override { /* 아무것도 하지 않는다. */ }
};

class Locator
{
public:
    static void Initialize() { Service = &NullService; }
    static Audio* GetAudio() { return Service; }
    static void Provide(Audio* Service_)
    {
        if (Service_ == nullptr)
        {
            Service = &NullService;
        }
        else
        {
            Service = Service_;
        }
    }

private:
    static Audio* Service;
    static NullAudio NullService;
};

class LoggedAudio : public Audio
{
public:
    LoggedAudio(Audio& wrapped_) : wrapped(wrapped_) {}
    void PlaySound(int SoundID) override
    {
        log("사운드 출력");
        wrapped.PlaySound(SoundID);
    }
    void StopSound(int SoundID) override
    {
        log("사운드 중지");
        wrapped.StopSound(SoundID);
    }
    void StopAllSounds() override
    {
        log("모든 사운드 중지");
        wrapped.StopAllSounds();
    }

private:
    void log(const char* message) { cout << message << endl; }
    Audio& wrapped;
};

void EnableAudioLogging()
{
    // 기존 서비스를 데코레이트한다.
    Audio* service = new LoggedAudio(*Locator::GetAudio());
    // 이 값으로 바꿔치기한다.
    Locator::Provide(service);
}

Audio* Locator::Service = nullptr;
NullAudio Locator::NullService;

int main()
{
    // 서비스 중개자 초기화
    Locator::Initialize();

    // 서비스 제공자(ConsoleAudio)를 Locator의 Service로 등록
    ConsoleAudio* consoleAudio = new ConsoleAudio();
    Locator::Provide(consoleAudio);

    // Locator에 등록된 서비스 사용
    Audio* audio = Locator::GetAudio();
    audio->PlaySound(VERY_LOUD_BANG);

    // PlaySound를 호출하는 쪽에서는 Audio라는 추상 클래스만 있을 뿐, ConsoleAudio를 모른다는 게 핵심

    EnableAudioLogging();
    audio = Locator::GetAudio();  // Locator에 등록된 LoggedAudio 객체로 갱신
    
    audio->PlaySound(VERY_LOUD_BANG);

    delete consoleAudio;

    return 0;
}
```

- 출력

```cpp
Play 100
사운드 출력
Play 100
```

과정

![image](https://github.com/user-attachments/assets/13de0fb3-491e-44fc-ba2f-388a1a87143f)


```cpp
void EnableAudioLogging()
{
    // 기존 서비스를 데코레이트한다.
    // 매개변수(ConsoleAudio)는 LoggedAudio의 생성자에 의해 wrapped로 저장된다.
    Audio* service = new LoggedAudio(*Locator::GetAudio());    
    // Locator의 서비스가 ConsoleAudio -> LoggedAudio 타입으로 바뀐다.     
    Locator::Provide(service);       
}
```

```cpp
class LoggedAudio : public Audio
{
public:
    LoggedAudio(Audio& wrapped_) : wrapped(wrapped_) {}
    // ... 
private:
    Audio& wrapped;
};
```

---

```cpp
audio = Locator::GetAudio();
```

- Locator에 등록된 LoggedAudio 객체 반환

![image](https://github.com/user-attachments/assets/8e27c9b9-7a39-4369-86c4-1c1c421d8a7f)


- audio는 이제 LoggedAudio 이다.

![image](https://github.com/user-attachments/assets/69f23ba9-f0db-4458-be30-99ccbfea106a)


---

```cpp
audio->PlaySound(VERY_LOUD_BANG);
```

- wrapped에 기존의 ConsoleAudio에 저장된다.

![image](https://github.com/user-attachments/assets/5ad94b6b-33c9-4b41-9788-38d55b5cedfd)


- audio->PlaySound(VERY_LOUD_BANG); 에서 PlaySound는 LoggedAudio의 것이 불린다.

![image](https://github.com/user-attachments/assets/0df58de3-ab88-4099-b463-7a0f87a27b40)


- Log를 찍고 wrapped의 PlaySound도 호출된다.

![image](https://github.com/user-attachments/assets/18651de0-137c-4618-9ff1-9764ebd9bd00)


---

# 16.5 언제?

- 게임 엔진의 프레임워크 급으로 방대한 패턴으로 콘텐츠 프로그래머에게 이 패턴을 활용해서 콘텐츠를 만들기는 쉽지 않을 것 같다. 오히려 엔진 프로그래머에게 추천되는 패턴?
- 서비스 중재자 패턴은 여러 면에서 싱글턴 패턴과 비슷하다. 어느 쪽이 더 필요에 맞는지 살펴본 뒤에 결정하자.
    - 게임 전체에서 많이 사용되는 시스템으로 전역 접근이 가능. ⇒ 싱글톤과 이 패턴을 고민하게 된다.
    - 여기서 디커플링까지 하고 싶다면 서비스 - 중개자 모델

# 16.6 주의 사항

## 서비스가 실제로 등록되어 있어야 한다.

싱글턴이나 정적 클래스는 인스턴스가 준비되어 있다는 것을 어느 정도 보장할 수 있으나 서비스 중개자 패턴은 직접 객체를 등록해야 하기 때문에 찾는 서비스가 없을 때를 대비해야 한다.

## 서비스는 누가 자기를 가져다가 놓는지 모른다.

특정 상황에서만 써야 하는 코드라면 패턴을 적용하지 않는 것이 안전.

커플링되는 의존성을 런타임 시점까지 미루는 부분이 가장 어려움

# 16.7 디자인 결정

## 서비스 어떻게 등록?

### ***외부 코드에서 등록***

위의 예제 코드와 같은 방법으로, 가장 일반적인 방법이다.

가장 일반적으로 사용되는 방법이라고 한다. 

### ***컴파일할 때 등록***

```cpp
class Locator
{
public:
	static Audio& getAudio() { return service_;}
	
private:
#if DEBUG
	static DebugAudio service_;
#else
	static ReleaseAudio service_;
#endif
};
```

- 빠르다. 컴파일할 때 이미 등록이 끝나 있기 때문.
- 서비스는 항상 사용 가능하다. 서비스 등록이 안 되어있는 경우를 고려하지 않아도 된다.
- 서비스를 변경하려면 컴파일을 다시 해야 한다.

### ***런타임에 값 읽기***

리플렉션 등을 이용해서 설정 파일을 통해 원하는 서비스 제공자 클래스를 생성한다.

## **서비스 없으면?**

### ***사용자가 알아서 처리***

가장 간단한 방법으로 서비스를 찾지 못하면 NULL을 반환하고 사용자가 알아서 하게 한다.

### ***게임을 멈춘다***

없는 서비스를 얻어 오려는 것을 명백한 버그로 간주해서 단언문(assert)을 추가한다.

### ***널 서비스 반환***

앞의 예제에서 처럼 서비스가 없는 경우 아무것도 하지 않는 널 객체를 반환한다.

## 서비스 범위는?

### ***전역 접근***

- 전체 코드에서 같은 서비스를 쓴다. 서비스가 단 한 개만 존재하게 된다.
- 전역 접근의 일반적인 단점을 가진다. 언제 어디에서 서비스가 사용되는지 제어할 수 없다.

### ***특정 클래스 제한***

- 커플링을 제어할 수 있다.
- 중복 작업이 생길 수 있다. 상관없는 클래스에서 같은 서비스를 사용해야 한다면 각자 그 서비스를 참조하고 등록하는 작업을 해줘야 한다.

보통은 어디에서나 중개자를 통해 서비스에 접근할 수 있도록 한다. 그러나 다음과 같이 특정 클래스를 상속한 그룹에서만 접근을 할 수 있도록 제한할 수 있다.

```cpp
class Base
{
public:
	//이런 식으로 적당히 서비스를 찾아 등록해준다.
	void init()	{service_ = new ConsoleAudio();	}

//하위 클래스에서만 접근이 가능하다.
protected:
	Audio& getAudio(){return *service_;}

private:
	static Audio* service_;
};
```

---

유니티 프레임워크에서는 GetComponent()에서 컴포넌트 패턴과 함께 서비스 중개자 패턴을 사용한다.

참고 자료

https://boycoding.tistory.com/120

https://tsyang.tistory.com/89
