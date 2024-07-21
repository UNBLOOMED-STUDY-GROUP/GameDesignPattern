# 👨‍💻 Singleton Pattern
주요 목적 : 객체의 최대 생성갯수를 1개로 제한하기 위한 패턴이다.
부수적인 목적 : 전역적인 데이터 관리를 위해 사용하기도 한다.

다음의 세 가지 구성으로 만든다.

1. 해당 클래스 타입의 포인터 변수 mpInstance를 static으로 만든다.
2. 생성자는 public이 아니다.
3. GetInastance()함수 정의에는 최대 생성갯수를 1개로 제한하는 판단 제어구조(if문)가 위치한다.

## 구성방식

### 사고 흐름

- 인스턴스가 하나만 = 외부에서 생성 X = 생성자를 private으로 하자.

```cpp
class EngineCore
{
private:
	EngineCore();
	~EngineCore();
};
```

문제는 생성자를 private에 넣는 바람에 객체 생성이 아예 안된다.

```cpp
EngineCore c; // 에러. 생성자에 접근 불가
```

따라서 객체 선언을 안하고 객체를 한번만 만들 수 있는 유일한 함수를 만들어야 한다. 여기서 우리는 static을 생각해볼 수 있다. 

- 인스턴스가 하나만 = 단 하나만 있는 변수 ⇒ static 변수

따라서 아래와 같이 써볼 수 있다. 

```cpp
class EngineCore
{
public:
    static EngineCore instance;
    void Test();
private:
    EngineCore() {}
    ~EngineCore() {}
};

// 클래스 외부에서 정적 멤버 변수 초기화
EngineCore EngineCore::instance;

int main()
{
    EngineCore::instance.Test();
}
```

하지만 위와 같이 하면 정적 멤버 변수는 외부에서 초기화해야 한다는 점에서 `EngineCore::instance`를 클래스 외부에서 초기화해야 한다. 이는 초기화 순서를 정확히 관리해야 하는 부담이 있다.

그래서 이를 EngineCore클래스 내부에서 함수하나만 딱 호출하면 초기화된 정적 변수 EngineCore를 반환하도록 구성했다. 

```cpp
#include <iostream>
using namespace std;

class EngineCore
{
public:
    static EngineCore& GetInstance()
    {
        static EngineCore instance; // 정적 지역 변수로 초기화
        return instance;
    }
private:
    EngineCore();
    ~EngineCore();
};
```

- getInstance()함수가 처음 호출될 때 정적 지역 변수가 초기화되며, 이후에는 동일한 인스턴스를 반환
- 함수 내부에서 선언한 정적 지역 변수의 reference을 통해 접근이 가능하게 한다.
- 초기화 순서 문제가 해결되고, 더 간결하게 싱글톤 패턴을 구현할 수 있다.

### 복사, 이동 방지

```cpp
#include <iostream>
using namespace std;

// 싱글톤 클래스
class EngineCore {

public:
	// static 함수로 선언
	static EngineCore& GetInstance() {
		// staitc 변수로 선언함으로서, instance 뱐수는 한번만 초기화되고, 프로그램 수명 내내 지속됨.
		// 특히 C++11부터 thread-safe 변수 초기화가 보장됨.
		static EngineCore instance;
		return instance;
	}

private:
	// Default 생성자 사용 (필요시 생성자를 원하는데로 수정해서 사용해도 됨)
	EngineCore() = default;

	// 객체는 유일하게 하나만 생성되어야 하기에 복사(대입), 이동(대입) 생성자 비활성화
	// 복사, 이동 생성자를 delete로 선언함으로서, 
	// 프로그래머 실수에 의한 복사, 이동 생성자 호출을 원천에 방지할 수 있음.
	EngineCore(const EngineCore&) = delete;
	EngineCore& operator=(const EngineCore&) = delete;
	EngineCore(EngineCore&&) = delete;
	EngineCore& operator=(EngineCore&&) = delete;
};

int main()
{
    EngineCore::GetInstance();
}
```

### 다른 버전

```cpp
#pragma once
class EngineCore
{	
public:
	static EngineCore* GetInstance()
	{
		static EngineCore engineCore;
		return &engineCore;	// 주소 반환 
	}

private:
	EngineCore();
	~EngineCore();
};
```

차이점은?

ChatGPT

```cpp
- 접근 방식
첫 번째 코드는 참조를 반환하여 객체에 직접 접근합니다. 이는 간결하고 사용하기 편리합니다.
두 번째 코드는 포인터를 반환하여 간접적으로 객체에 접근합니다. 포인터를 사용하면 메모리 관리나 포인터 연산에 신경 써야 합니다.

- 안전성
참조 반환 방식은 nullptr을 반환할 가능성이 없기 때문에 더 안전한 접근 방식을 제공합니다.
포인터 반환 방식은 nullptr 검사를 추가로 해야 할 수 있습니다.

- 사용 편의성
참조를 반환하는 방식은 객체의 멤버에 접근할 때 점 연산자(.)를 사용하여 직관적이고 편리합니다.
포인터를 반환하는 방식은 화살표 연산자(->)를 사용하여 접근해야 하므로 조금 더 번거로울 수 있습니다.
```

따라서, 첫 번째 코드는 참조를 반환하여 더 간단하고 안전한 사용법을 제공하며, 두 번째 코드는 포인터를 반환하여 유연하지만 추가적인 포인터 연산과 관리가 필요합니다. 일반적으로 싱글톤 패턴 구현 시 첫 번째 방식이 더 선호됩니다.

# Unreal Engine 속 Singleton
언리얼 엔진은 Game Singleton Class를 제공한다. GEnigine이라는 전역변수에서 가져올  GameSingleton이라는 걸 가져올 수 있다. 

```cpp
/** A UObject spawned at initialization time to handle game-specific data */
	UPROPERTY()
	TObjectPtr<UObject> GameSingleton;
```

```cpp
void UEngine::InitializeObjectReferences()
{
	if (GameSingleton == nullptr && GameSingletonClassName.ToString().Len() > 0)
	{
		UClass *SingletonClass = LoadClass<UObject>(nullptr, *GameSingletonClassName.ToString());

		if (SingletonClass)
		{
			GameSingleton = NewObject<UObject>(this, SingletonClass);
		}
		else
		{
			UE_LOG(LogEngine, Error, TEXT("Engine config value GameSingletonClassName '%s' is not a valid class name."), *GameSingletonClassName.ToString());
		}
	}
}
```

언리얼 엔진에서 제공하는 싱글톤 클래스는 다음과같다.

- **GameInstance**
- **AssetManager**
- **GameMode**
- **GameState**

## 언리얼에서 싱글톤을 만드는 방법

```cpp
#include <iostream>
using namespace std;

class EngineCore
{
public:
    static EngineCore& GetInstance()
    {
        static EngineCore instance; // 정적 지역 변수로 초기화
        return instance;
    }
private:
    EngineCore();
    ~EngineCore();
};
```

```cpp
UCLASS()
class ALSTEST_API USOGameSingleton : public UObject
{
	GENERATED_BODY()

public:
	USOGameSingleton();
	static USOGameSingleton& Get();        // static 함수
};

// ===================== cpp =========================
USOGameSingleton& USOGameSingleton::Get()
{
	USOGameSingleton* Singleton = CastChecked<USOGameSingleton>(GEngine->GameSingleton);
	
	// static 지역변수처럼 한번만 생성되고 그 뒤로는 같은거 반환하게 만들었다. 
	if(Singleton) return *Singleton;

	return *NewObject<USOGameSingleton>();
}
```

생성자가 public이어도 어차피 Get을 호출해도 Singleton은 단 한번만 생성되기에 문제가 없다. 

USOGameSingleton이 여러 개여도 거기서 생성하는 Singleton은 로직상 단 한번만 생성되게 해서 문제가 없다.
추가로 언리얼 에디터 상에서 싱글톤 클래스를 지정해줘야 한다. 

![image](https://github.com/user-attachments/assets/47c522a5-27fb-4163-b726-944736e6f226)

참고 문헌

https://computing-jhson.tistory.com/135
https://openmynotepad.tistory.com/125

