# Command Pattern
### GoF
> 요청 자체를 캡슐화하는 것.</br>
> 요청이 서로 다른 사용자를 매개변수로 만들고, 요청을 대기시키거나 로깅하며 되돌릴 수 있는 연산을 지원한다.

**명령 패턴**은 메서드 호출을 실체화한 것이다

### 실체화 한다
- 실제하는 것으로 만든다.
- 일급으로 만든다
- 어떤 개념을 변수에 저장하거나 함수에 전달할 수 있도록 데이터, 즉 객체로 바꿀 수 있다는 걸 의미한다.

## 예시1 : 입력 키 변경
- 버튼이나 키보드, 마우스를 누르 등의 유저 입력을 읽는 코드

### **⬇️ 기본 예제** 
---
```cpp
void InputHandler::HandleInput() 
{
    if (IsPressed(BUTTON_X)) 
    {
        Jump();
    }
    else if (IsPressed(BUTTON_Y)) 
    {
        FireGun();
    }
    else if (IsPressed(BUTTON_A)) 
    {
        SwapWeapon();
    }
    else if (IsPressed(BUTTON_B)) 
    {
        LurchIneffectively();
    }
}
```

- 매 프레임마다 호출한다.

### **⬇️  명령 패턴으로 바꾼다면**
---
![image](https://github.com/user-attachments/assets/6b48ef01-e788-4128-9293-9e67578609cb)
- `Jump`, `FireGun` 함수를 직접 호출하지 않고, **교체 가능한 무엇인가**로 바꿔야 한다.
    - 인터페이스에 반환 값이 없는 메서드 하나밖에 없다면 명령 패턴일 가능성이 높다.</br>

#### 1. 게임에서 할 수 있는 행동을 실행할 수 있는 **공통 상위 클래스부터 정의**한다.

```cpp
class Command
{
public:
	Command();
	virtual ~Command();
	virtual void execute() = 0;
};
```

#### 2. 각 행동별로 **하위 클래스**를 만든다. → `JumpCommand`  클래스, `FireCommand` ****클래스

```cpp
class JumpCommand : public Command
{ 
public:
	virtual void execute() { Jump(); }
};

class FireCommand : public Command
{
public:
	virtual void execute() { FireGun(); }
};
```

#### 3. 입력 핸들러 코드에는 **각 버튼별로 Command 클래스 포인터를 저장**한다.

```cpp
class InputHandler 
{
public:
	void HandleInput();

private:
	Command* ButtonX;
	Command* ButtonY;
	Command* ButtonA;
	Command* ButtonB;
};
```

####  4. 입력처리는 다음 코드로 **위임**된다.

```cpp
void InputHandler::HandleInput()
{
	if (IsPressed(BUTTON_X))
	{
		ButtonX->execute();
	}
	else if (IsPressed(BUTTON_Y))
	{
		ButtonY->execute();
	}
	else if (IsPressed(BUTTON_A))
	{
		ButtonA->execute();
	}
	else if (IsPressed(BUTTON_B))
	{
		ButtonB->execute();
	}
}
```

####  5. **제어하려는 객체를 함수에서 직접 찾게 하지 말고 밖에서 전달하자 (액터에게 지시하기)**
- 명령에서 변경하려는 액터와 명령 사이를 추상화로 격리시킨다.

```cpp
class Command
{
public:
	Command();
	virtual ~Command();
	virtual void execute(GameActor& actor) = 0;
};
```

#### 6. Command가를 상속받은 클래스는 `execute`가 호출될 때, 원하는 GameActor 객체를 인수로 받아서 액터의 메서드를 호출할 수 있다.

```cpp
class JumpCommand : public Command
{ 
public:
	virtual void execute(GameActor& actor) { actor.Jump(); }
};
```

#### 7. 입력 핸들러에서 입력을 받아 적당한 객체의 메서드를 호출하는 명령 객체를 연결하자.
- `HandleInput` 에서 명령 객체를 반환하도록 변경한다.

```cpp
Command* InputHandler::HandleInput()
{
	if (IsPressed(BUTTON_X))
	{
		return ButtonX;
	}
	else if (IsPressed(BUTTON_Y))
	{
		return ButtonY;
	}
	else if (IsPressed(BUTTON_A))
	{
		return ButtonA;
	}
	else if (IsPressed(BUTTON_B))
	{
		return ButtonB;
	}
	return nullptr;
}
```

- 어떤 액터를 매개 변수로 넘겨줘야 할지 모르게 때문에 `HandleInput` 함수에서는 명령을 실행할 수 없다.
- 명령이 실체화된 함수 호출이라는 점을 활용해서, 함수 호출 시점을 지연한다.
    a. 명령이 실체화된 함수 호출
        - `JumpCommand` 클래스에서 `Jump` 함수 호출한다.
    b. 함수 호출 시점을 지연
        - 입력이 발생했을 때 즉시 동작을 실행하지 않고, 명령 객체를 반환하여 필요할 때 호출한다.

#### 8. 명령 객체를 받아서 플레이어를 대표하는 GameActor 객체에 적용하자.

```cpp
	Command* command = inputHandler.HandleInput();
	if (command) 
	{
		command->execute(actor);
	}
```

- 명령을 실행할 때, 액터만 바꾸면 플레이어가 게임에 있는 어떤 액터라도 제어할 수 있게 되었다.
- 같은 명령 패턴으로 AI도 제어 가능하다. </br>

### **⬇️  AI 기능에 적용한다면**
---
![image](https://github.com/user-attachments/assets/452a73b3-0d43-4c5c-b8f9-bb6f96210655)
- AI 코드에서 원하는  Command 객체를 이용하도록 한다.
- Command 객체를 선택하는 AI와 이를 실행하는 액터를 디커플링함으로써 코드가 훨씬 유연해졌다.
- 입력 핸들러나 AI 같은 코드에서는 명령 객체를 만들어 스트림에 밀어 넣는다.
- 디스패처나 액터에서는 명령 객체를 받아서 호출한다. 큐를 둘 사이에 끼워넣음으로써, 생산자와 소비자를 디커플링할 수 있게 되었다.

## 예시2 : 실행취소와 재실행
- 어떤 유닛을 옮기는 명령을 생각해보자.

```cpp
class MoveUnitCommand : public Command
{
public: 
	MoveUnitCommand(Unit* NewUnit, int NewX, int NewY)
		:unit(NewUnit), x(NewX), y(NewY) 
	{
	
	}
	
	virtual void execute(GameActor& actor) { unit->Move(x, y); }

private:
	Unit* unit;
	int x;
	int y;
};
```

- 이동하려는 유닛과 위치 값을 생성자에서 받아서 명령과 명시적으로 바인드 한다.
- `MoveUnitCommand`  객체는 게임에서의 구체적인 실제 이동을 담고 있다.

- 입력 핸들러 코드는 플레이어가 이동을 선택할 때 마다 명령 인스턴스를 생성해야 한다.

```cpp
Command* InputHandler::HandleInput()
{
	...
	else if (IsPressed(BUTTON_UP))
	{
		// 유닛을 한 칸 위로 이동한다.
		if (unit)
		{
			int destY = unit->GetY() - 1;
			return new MoveUnitCommand(unit, unit->GetX(), destY);
		}
	}
	else if (IsPressed(BUTTON_DOWN))
	{
		if (unit)
		{
			int destX = unit->GetX() - 1;
			return new MoveUnitCommand(unit, unit->GetX(), destX);
		}
	}
...
}
```

- 명령을 취소할 수 있도록 Command 클래스에 순수 가상 함수 `undo` 를 만든다.

```cpp
class Command
{
public:
	Command() {  }
	virtual ~Command() {  }
	virtual void execute(GameActor& actor) = 0;
	virtual void undo() = 0;
};
```

- `undo` 함수에서는 `excute` 함수에서 변경하는 게임 상태를 반대로 바꿔주면 된다.

- `MoveUnitCommand` 클래스에 **실행 취소 기능**을 넣어보자.

```cpp
class MoveUnitCommand : public Command
{
public:
	MoveUnitCommand(Unit* NewUnit, int NewX, int NewY)
		:unit(NewUnit), x(NewX), y(NewY), xBefore(0), yBefore(0)
	{  }

	virtual void execute(GameActor& actor) 
	{
		// 나중에 이동을 취소할 수 있도록 원래 유닛 위치를 저장한다.
		xBefore = x;
		yBefore = y;
		unit->Move(x, y); 
	}
	
	virtual void undo() 
	{  
		unit->Move(xBefore, yBefore);
	}

private:
	Unit* unit;

	int x;
	int y;
	int xBefore;
	int yBefore;
};
```

### **⬇️  실행 취소 스택 따라가기**
![image](https://github.com/user-attachments/assets/f2ef7283-f268-4a29-9b96-f93abe45ac23)

- 여러 단계의 실행 취소를 지원
- 가장 최근 명령만 기억하는 대신, 명령 목록을 유지하고 현재 명령이 무엇인지만 알고 있으면 된다.
- 유저가 명령을 실행하면, 새로 생성된 명령을 목록 맨 뒤에 추가하고 이를 현재 명령으로 기억하면 된다.

- 유저가 실행취소를 선택하면 **현재 명령**을 `실행취소`하고 **현재 명령을 가리키는 포인터**를 `뒤로 이동`한다.
- `재실행`을 선택하면, 포인터를 다음으로 이동시킨 후에 해당 포인터를 실행한다.
- 유저가 몇번 `실행취소` 한 뒤에 새로운 명령을 실행한다면, 현재 명령 뒤에 새로운 명령을 추가하고 그 다음에 붙어 있는 명령들은 버린다.
