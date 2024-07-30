# 업데이트 메서드 패턴이란
- 9장 게임 루프와 연결되는 내용
- 14장 컴포넌트 패턴까지 더해진 구조가 유니티 엔진

## 의도

> 컬렉션에 들어있는 객체별로 한 프레임 단위의 작업을 진행하라고 알려줘서 전체를 시뮬레이션한다.
> 

## 동기

게임 메인 루프가 있다 해보자. 여기서 플레이어, 몬스터 A, 몬스터 B를 구현한다고  해보자.

```cpp
void EngineCore::Update()
{
	// 플레이어 조작 코드
	float move = 0;

	bool moveLeft = GetKeyHold(LEFT);
	bool moveRight = GetKeyHold(RIGHT);

	if (moveLeft || moveRight) 
	{
		if (moveLeft) { m_iDir = -1; }
		else { m_iDir = 1; }

		move = m_iDir * m_speed;
		m_curpos.x += move * fDT;
	}
	
	// 몬스터 A 순찰 로직
	// 몬스터 B 순찰 로직
	// ...
}
```

보기만 해도 더러움이 느껴진다. 그리고 객체가 추가되면 추가될수록 코드가 점점 유지보수하기 어려워진다. 

해결책은 너무 쉽기 때문에 다들 머릿속에 그리고 있을 것이다.

> 모든 객체가 자신의 동작을 각자 update하도록 캡슐화하면 된다.  - p.181
> 

이러면 메인 게임루프를 어지럽히지 않고도 쉽게 객체를 추가, 삭제할 수 있다.

```cpp
void EngineCore::Update()
{
	CTimeMgr::GetInstance()->Update();
	CKeyMgr::GetInstance()->Update();
	CCamera::GetInstance()->Update();
	CSceneMgr::GetInstance()->Update();	
	CCollisionMgr::GetInstance()->Update();
}

void CSceneMgr::Update()
{
	if (m_pCurScene == nullptr) return;
	m_pCurScene->Update();
	m_pCurScene->FinalUpdate();
}
```

```cpp
class CScene
{
protected:
    vector<CGameObject*> m_arrObj[(UINT)GROUP_TYPE::END];
public:
    CScene();
    virtual ~CScene();
    virtual void Update();
    // ...
}

void CScene::Update()
{
	for (int i = 0; i < (UINT)GROUP_TYPE::END; i++)
	{
		for (int j = 0; j < m_arrObj[i].size(); j++)
		{
			if (!m_arrObj[i][j]->IsDead()) 
			{
				m_arrObj[i][j]->Update();	// i그룹 j 객체
			}
		}
	}
}
```

```cpp
class CPlayer : public CGameObject {}

void CPlayer::Update()
{
	// 플레이어 조작 update
	float move = 0;

	bool moveLeft = GetKeyHold(LEFT);
	bool moveRight = GetKeyHold(RIGHT);

	if (moveLeft || moveRight) 
	{
		if (moveLeft) { m_iDir = -1; }
		else { m_iDir = 1; }

		move = m_iDir * m_speed;
		m_curpos.x += move * fDT;
	}
}
```

게임 루프는 매 프레임마다 객체 컬렉션을 쭉 돌면서 모든 update를 호출한다. 이때 각 객체는 한 프레임만큼 동작을 진행한다. 덕분에 모든 게임의 객체가 동시에 동작한다.

## 써야 하는 상황

- 동시에 동작해야 하는 객체나 시스템이 게임에 많다.
- 각 객체의 동작은 다른 객체와 거의 독립적이다.
- 객체는 시간의 흐름에 따라 시뮬레이션되어야 한다. 

## 주의사항

- 코드를 한프레임 단위로 끊어서 실행하는 게 더 복잡하다.
- 다음 프레임에서 다시 시작할 수 있도록 현재 상태를 저장해야 한다.
- 모든 객체는 매 프레임마다 시뮬레이션되지만 진짜로 동시에 되는 건 아니다.
    - 모든 객체를 접근할 때, 순차 처리로 이루어지기 때문에 동시에 되는 건 아니다.
    - 이를 위해서 update종류가 다양해지고 update순서도 고려해야 한다. 
    - 예를 들어 충돌처리 이벤트는 순서상 앞에 모든 update가 다 끝나고 다음 프레임의 첫 번째에 배치되어 있다.
- 업데이트 도중에 객체 목록을 바꾸는 건 조심해야 한다.
    - 이또한 Destroy할 객체를 바로 삭제하지 않고 미리 표시해둔 후 다음 프레임까지 기다렸다가 Destroy가 처리된다.

---

## 구현

### 객체 상속 구조

언리얼에서 Actor가 있고, 유니티에는 MonoBehaviour가 있듯이 객체를 상속받아서 Tick, Update를 구현한다. 이렇게 객체마다 Update를 가지고 있어서 게임 월드에 추가만 하면 독자적으로 동작하게 할 수 있다. 

하지만 책에서는 이렇게 묘사되어 있다. 

> 바벨탑처럼 쌓아올린 클래스 상속 구조는 너무나 높고 거대해서 해를 가릴 지경이었다. 시간이 지나면서 거대한 상속구조가 형편없다는 것을 알게 되었다. 이를 쪼개서 쓸 수 없어서 유지보수가 불가능했다.
>


※ 책에서는 상속구조가 엄청 나쁜 것처럼 묘사되었는데 오해의 소지가 있는 거 같다. 책에서는 “거대한 상속구조가 형편없다” 라는 표현이 있는데 이 “거대한” 은 정말 하나의 Update문에서 모든 걸 처리해서 일어나는 현상을 말하는 거 같다. 

언리얼로 예를 들면 입력함수, 이동함수, 충돌처리, 랜더링, 모든 것들이 하나의 Playere클래스의 Tick함수에서 일어나고 있는 코드를 보고 “거대한 상속구조가 형편없다”라고 쓴게 아닐까 싶다. 


### 컴포넌트 패턴

유니티가 대표적으로 컴포넌트 패턴이 들어간 게임 엔진이다.

