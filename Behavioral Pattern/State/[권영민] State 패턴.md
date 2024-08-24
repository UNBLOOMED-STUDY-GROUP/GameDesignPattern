## 개요

<aside>
💡 GoF의 디자인 패턴

- 객체의 내부 상태에 따라 스스로 행동을 변경할 수 있게 허가하는 패턴, 이렇게 하면 객체는 마치 자신의 클래스를 바꾸는 것처럼 보인다.
</aside>

## 예시 - 상태 패턴을 사용하지 않을 때 (추억의 게임 만들기)

```cpp
void Hero::HandleInput(Input input)
{
	if(input == PRESS_B)
	{
		// 점프중이 아니라면 점프한다.
	}
	else if(input == PRESS_DOWN)
	{
		if(false == IsJumping)
		{
			SetGraphics(IMAGE_DUCK);
		}
	}
	else if(input == RELEASE_DOWN)
	{
			SetGraphics(IMAGE_STAND);
	}
	.....
}

```

- 캐릭터가 어떤 상태인지 검사하는 조건이 계속 추가되고, 버그가 발생할 확률이 높다.

## FSM
![image](https://github.com/user-attachments/assets/ee6ee690-9360-4027-972b-9322a550c77d)

- 유한 상태 기계
    - 오토마타 이론
    - 가질 수 있는 상태가 한정된다.
    - 한번에 한 가지 상태만 될 수 있다.
    - 입력이나 이벤트가 기계에 전달된다.
    - 각 상태에는 입력에 따라 다음 상태로 바뀌는 전이가 있다.
- `상태`, `입력` , `전이`

## 열겨형과 다중 선택문

- 여러 플래그 변수 중에서 하나만 참일때가 많다면 `열거형` 이 필요하다.

```cpp
enum State
{
	STATE_STANDING,
	****STATE_JUMPING,
	STATE_DUCKING,
	STATE_DIVING,
**};**
```

- 플래그 변수 여러 개 대신 state 필드 하나만 사용하도록 하자.

- 상태 패턴을 사용하지 않을 때에는 입력에 따라 먼저 분귀한 뒤에 상태에 따라 분기했다.
    - 따라서 하나의 버튼 입력에 대한 코드는 모아둘 수 있었으나, 하나의 상태에 대한 코드는 흩어져 있었다.

⇒ 상태에 따라 분기하게 하자.

### **`상태` 에 따라 분기하게 하자**

```cpp
void Hero::HandleInput(Input input)
{
	switch(state)
	{
		// STANDING
		case STATE_STANDING:
			if(input == PRESS_B)
			{
				state = STATE_JUMPING;
				yVelocity = JUMP_VELOCITY;
				SetGraphics(IMAGE_JUMP);
			} 
			else if(input == PRESS_DOWN)
			{
				state = STATE_DUCKING;
				SetGraphics(IMAGE_DUCK);
			}
		break;
		// JUMPING
		case STATE_JUMPING:
			if(input == PRESS_DOWN)
			{
				state = STATE_DIVING;
				SetGraphics(IMAGE_DIVE);
			} 
		break;
		// DUCKING
		case STATE_DUCKING:
			if(input == RELEASE_DOWN)
			{
				state = STATE_STANDING;
				SetGraphics(IMAGE_STAND);
			} 
		break;		
	}
}
```

- 업데이트해야 할 상태 변수를 하나로 줄였고, 하나의 상태를 관리하는 코드는 깔끔하게 한곳에 모은다.

- 열겨형만으로는 부족 + 시간을 기록해야 할때, `ChargeTime` 필드를 추가해서 사용하면 다음과 같다.

```cpp
void Hero::Update()
{
	if(state == STATE_DUCKING)
	{
		ChargeTime++;
		if(ChargeTIme > MAX_CHARGE)
		{
			SuperBomb();
		}
	}
}

void Hero::HandleInput(Input input)
{
	switch(state)
	{
		switch STATE_STANDING:
			if(input == PRESS_DOWN)
			{
				state = STATE_DUCKING;
				ChargeTime = 0;
				SetGraphics(IMAGE_DUCK);
				... 
			}
		break;
		....
	}
}
```

## 상태 패턴

### 상태 인터페이스

- 상태에 의존하는 모든 코드, 즉 다중 선택문에 있던 동작을 인터페이스의 가상 메서드로 만든다.

```cpp
class IHeroStateInterface
{
	virtual ~IHeroStateInterface() { }
	virtual void HandleInput(Hero& Hero, Input input) = 0;
	virtual void Update(Hero& hero) = 0;
};
```

### 상태별 클래스 만들기
![image](https://github.com/user-attachments/assets/98347397-3502-46cf-8ff7-e77cb8317f6c)

- 상태별로 인터페이스를 구현하는 클래스를 정의한다.
- 다중 선택문에 있던 case 별로 클래스를 만들어 코드를 옮기면 된다.
- ChargeTime는 특정 상태(DuckState)에서만 의미 있는 변수이기 때문에 해당 상태의 멤버 변수로 사용하면 된다.

```cpp
class DuckState : public HeroState, IHeroStateInterface
{
	public:
		DuckState() : ChargeTime(0) {}
		virtual ~DuckState() {}
		
		virtual void HandleInput(Hero& hero, Input input)
		{
			if(input == RELEASE_DOWN)
			{
					// 일어선 성태로 바꾼다.
				ChargeTime = 0;
				hero.SetGraphics(IMAGE_DUCK);
				... 
			}
		}
		
		virtual void Update(Hero& hero)
		{
			ChargeTime++;
			if(ChargeTime> MAX_CHARGE)
			{
				hero.SuperBomb();
			}
		}
	private:
		int ChargeTime;
}
```

### 동작을 상태에 위임하기

- Hero 클래스에 자신의 현재 상태 객체 포인터를 추가해, 거대한 다중 선택문은 제거하고 대신 상태 객체에 위임한다.

```cpp
class Hero
{
	public:
		virtual void HandleInput(Input input)
		{
			 state->HanleInput(*this, input);
			 ...
		}
		virtual void Update()
		{
			state->Update(*this);
			...
		}
	private:
		HeroState* state;
}
```

- 상태를 바꾸려면, state 포인터에 HeroState를 상속받는 다른 객체를 할당하기만 하면 된다.

## 상태 객체는 어디에 둬야 할까

- 상태를 바꾸려면 state에 새로운 상태 객체를 할당해야 한다.
- 새로운 상태 객체는 어디에서 온 것일까 ?
    - 상태 패턴 클래스를 쓰기 때문에 포인터에 저장할 실제 인스턴스가 필요하다.

### 1️⃣ 정적 객체

<aside>
💡 **사용 경우**

- **상태 객체에 필드가 따로 없다면** ( ex: ChargeTIme), 가상 메서드 호출에 필요한 vTable 포인터만 존재한다.
- 이럴 경우 모든 인스턴스가 같기 때문에 인스턴스는 하나만 있으면 된ㄷ.
</aside>

- 여러 FSM이 동시에 돌더라도 상태 기계는 다 같으므로 인스턴스 하나를 같이 사용하면 된다.
- 정적 인스턴스는 원하는 곳에 두면 된다.
- 특별히 다른 곳이 없다면 상위 상태 클래스에 두자.

```cpp
class HeroState
{
	public : 
		static StandingState Standing;
		static DuckingState Ducking;
		static JumpingState Jumping;
		static DivingState Diving;
		// 다른 코드들
}
```

- 각각의 정적 변수가 게임에서 사용하는 상태 인스턴스다.

```cpp
if(input == PRESS_B)
{
	hero.state = heroState::Jumping;
	hero.SetGraphics(IMAGE_JUMP);
}
```

### 2️⃣ 상태 객체 만들기

<aside>
💡 **사용 경우**

- 특정 필드가 있는데, 이 값이 Hero를 상속받는 클래스마다 다르다 보니 정적 객채로 만들 수 없다.
</aside>

- 전이할 때마다 상태 객체를 만들어야 한다.

→ FSM이 상태별로 인스턴스를 갖게 된다.

- 새로운 상태를 할당했기 때문에 이전 상태를 해제해야 한다.

- `HandleInput` 함수에서 상태가 바뀔때에만 새로운 상태를 반환하고, 밖에서는 반환 값에 따라 예전 상태를 삭제하고 새로운 상태를 저장하도록 바꿔보자

```cpp
void Hero::HandleInput(Input input)
{
	if(HeroState* newState = state->HandleInput(*this,input))
	{
		delete state;
		state = NewState;
	}
}
```

```cpp
void HeroState::HandleInput(Hero& hero, Input input)
{
	if(input == PRESS_DOWN)
	{
		// 다른 코드들
		delete state;
		return new DuckingState();
	}
		// 지금 상태를 유지한다.
}
```

## 입장과 퇴장

![image](https://github.com/user-attachments/assets/6ce57453-759a-46c8-899c-84b2eef04df4)

- 상태 패턴의 목표
    - 같은 상태에 대한 모든 동작과 데이터를 클래스 하나에 캡슐화하는 것
    
- 수정 방향
    - 해당 State에서 그래픽까지 제어하는 게 바람직하다
    
    ⇒ `입장 기능` 추가
    

**StandingState 클래스**

```cpp
class StandingState : public HeroState
{
public:
	virtual void Enter(class Hero& hero)
	{
		hero.SetGraphics(IMAGE_STAND);
	} 
};
```

**Hero 클래스**

```cpp
void Hero::HandleInput(Input input)
{
	HeroState* NewState = State->handleInput(*this,input);
	if(nullptr != NewState)
	{
		delete State;
		State = NewState;
		// 새로운 상태의 입장 함수를 호출한다.
		State->Enter();
	}
}
```

**DuckingState 클래스**

```cpp
HeroState* DuckingState::HandleInput(Hero& hero, Input input)
{
	if(input == RELEASE_DOWN)
	{
		return new StandingState();
	}
	// 다른 코드들
}
```

- 실제 게임 상태 그래프라면 점프 후 착지 혹은 내려찍기 후 착지하는 식으로 같은 상태에 여러 전이가 들어올 수 있다.
- 그냥 두면 전이가 일어나는 모든 곳에 중복 코드를 넣었겠지만
    - 이렇게 입장을 구현하면 입장 기능 한 곳에 코드를 모아둘 수 있다.
- 상태가 새로운 상태로 교체되기 직전에 호출되는 퇴장 코드도 이런 식으로 활용할 수 있다.

## 단점

- 상태 기계는 엄격하게 제한된 구조를 강제함으로써 복잡하게 얽힌 코드를 정리할 수 있게 해준다.
    - FSM에는 미리 정해놓은 여러 상태와 현재 상태 하나, 하드 코딩되어 있는 전이만이 존재한다.
- 상태 기계를 인공지능같이 더 복잡한 곳에 적용하다 보면 한계에 부딪히게 된다.

## 단점 보완

### 병행 상태 기계
![image](https://github.com/user-attachments/assets/cff83d8b-8fac-4fe1-a68d-82db19d8b2bd)

➡️ 상황

- 주인공이 총을 들 수 있다.
- 총을 장착한 후에도 이전에 할 수있었던 달리기, 점프, 엎드리기 같은 동작을 모두 할 수 있어야 한다.
- 그러면서 동시에 총도 쏠 수 있어야 한다.

➡️ FSM 방식을 고수한다면

- 모든 상태를 무장, 비무장에 맞춰 두 개씩 만들어야 한다.
    - 서기, 무장한 채로 서기, 점프, 무장한 채로 점프
    - 모든 가능한 조합에 대해 모델링하려다 보니 모든 쌍에 대해 상태를 만들어야 한다.

➡️ 해결법 ⇒ 상태 기계를 두개로 나누면 된다.

**Hero 클래스**

```cpp
class Hero
{
	// 다른 코드들
private:
	class HeroState* State;
	class HeroState* Equipment;
};

void Hero::HandleInput(Input input)
{
	State->HandleInput(*this, input);
	Equipment->HandleInput(*this, input);
}
```

- 각각의 상태 기계는 입력에 따라 동작을 실행하고 독립적으로 상태를 변경할 수 있다.
- **두 상태 기계가 서로 전혀 연관이 없다면** 이 방법이 잘 맞는다.

### 계층형 상태 기계
![image](https://github.com/user-attachments/assets/67e01a97-44ce-421a-9e31-7fb76e08683c)

- 상속으로 여러 상태가 코드를 공유할 수 있다.
- 점프와 엎드리기는 “땅 위에 있는” 상태 클래스를 정의해 처리한다.
- 서기, 걷기, 달리기, 미끄러지기는 “땅 위에 있는” 상태 클래스를 상속 받아 고유 동작을 추가하면 된다.
- 계층형 상태 기계에서 어떤 상태는 상위 상태를 가질 수 있고, 그 경우 그 상태 자신은 하위 상태가 된다.

**상위 상태용 OnGroundState 클래스**

```cpp
class OnGroundState
{
public:
	virtual void HandleInput(Hero& hero, Input input)
	{
		if(PRESS_B == input)
		{
			// 점프 구현
		}else if(PRESS_DOWN == input)
		{
			// 엎드리기
		}
	}
};
```

**하위 상태용 DuckingState 클래스**

```cpp
class DuckingState 
{
public:
	virtual void HandleInput(Hero& hero, Input input)
	{
		if(RELEASE_DOWN == input)
		{
			// 서기
		}else 
		{
			// 따로 입력을 처리하지 않고, 상위 상태로 보낸다.
			OnGroundState::HandleInput(hero, input);
		}
	}
};
```

**`Stack` 으로 구현하기**

- 이 방법 외에도, 계층형상태기계를 `Stack` 을 만들어 명시적으로 현재 상태의 `상위 상태 연쇄`를 모델링할 수도 있다.
- 현재 상태가 `Stack` 최상위에 있고 밑에는 바로 위 상위 상태가 있으며, 그 상위 상태 밑에는 그 상위 상태의 상위 상태가 있는 식이다.
- 상태 관련 동작이 들어오면 어느 상태든 동작을 처리할 때까지 스택 위에서부터 밑으로 전달한다.

### 푸시다운 오토마타

![image](https://github.com/user-attachments/assets/91fb0e39-5533-4e11-8964-9697e0383430)

- 상태 스택을 활용하여 FSM을 확장하는 방법

➡️ 상황

- 주인공이 총을 쏘면 발사 애니메이션 재생과 함께 총알과 시각 이펙트를 생성하는 새로운 상태가 필요하다
- 총을 쏠 수 있는 모든 상태에서 발사 버튼을 눌렀을 때 전이할 FiringState라는 상태를 만들어보자
- 이때 어려운 부분은 총을 쏜 뒤에 어느 상태로 돌아가야 하는가 하는 점이다.
- 서기, 달리기, 점프, 덮드리기 상태에서 총을 쏠 수 있는데 총을 쏘는 동작이 끝난 후에는 다시 이전 상태로 돌아가야 한다.

➡️ 해결법

- 상태를 스택으로 관리한다.
- 총 쏘기 애니메이션이 끝날 때 총쏘기 상태를 스택에서 빼면, 푸시다운 오토마타가 알아서 이전 상태로 보내준다.

## 얼마나 유용한가

- FSM 만으로는 한계가 있다
- 요즘 게임 AI는 `행동 트리`, `계획 시스템` 을 더 많이 쓰는 추세이다
- FSM은 다음의 경우에 사용하면 좋다.
    - 내부 상태에 따라 객체 동작이 바뀔 때
    - 이런 상태가 그다지 많지 않은 선택지로 분명하게 구분될 수 있을 때
    - 객체가 입력이나 이벤트에 따라 반응할 때
- 게임 사용 예시
    - 입력 처리
    - 메뉴 화면 전환
    - 문자 해석
    - 네트워크 프로토콜
    - 비동기 동작 구현
