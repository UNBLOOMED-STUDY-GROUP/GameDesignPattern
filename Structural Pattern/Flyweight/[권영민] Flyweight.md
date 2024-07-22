# Flyweight Pattern
### GoF
> 공유(sharing을 통해 많은 수의 소립(fine-grained)객체들을 효과적으로 지원합니다.
### 숲에 들어갈 나무들
- 숲을 그리기 위해서는 전체 데이터를 CPU를 GPU로 버스를 통해 전달해야 한다.
- 나무마다 필요한 데이터
    - 줄기, 가지, 잎의 형태를 나타내는 폴리곤 메시
    - 나무 껍질과 잎사귀 텍스처
    - 숲에서의 위치와 방향
    - 각각의 나무가 다르게 보이도록 크게와 음영 같은 값을 조절할 수 있는 매개변수<br>

## 예시1 : 숲, 나무
![image](https://github.com/user-attachments/assets/f055633a-d7b3-4a6d-851b-f455ca770476)
### **⬇️ 기본 설계**
---
```cpp
class Tree
{
private:
	Mesh mesh;
	Texture bark;
	Texture leaves;
	Vector position;
	double height;
	double thickness;
	Color barkTint;
	Color leafTint;
};
```

**단점**

- 메시와 텍스처는 크기가 크기때문에, 데이터가 많다
- 많은 객체로 이루어진 숲 전체는 1프레임에 GPU로 모두 전달하기에는 양이 너무 많다.

### **⬇️ 플라이웨이트 패턴으로 바꾼다면**
---
![image](https://github.com/user-attachments/assets/efbe7bde-8e85-4bd8-9165-41e33f747db8)
- 나무 객체에 들어 있는 데이터 대부분이 인스턴스별로 다르지 않다
- 모든 나무가 다 같이 사용하는 데이터를 뽑아내 새로운 클래스( `TreeModel` )에 모아보자.
- 나무 인스턴스 4개가 모델 하나를 공유한다.

```cpp
class TreeModel
{
private:
	Mesh mesh;
	Texture bark;
	Texture leaves;
};
```

```cpp
class Tree
{
private:
	TreeModel* model;
	
	Vector position;
	double height;
	double thickness;
	Color barkTint;
	Color leafTint;
}
```
- `TreeModel` 객체는 하나만 존재한다.
- 각 나무 인스턴스는 공유 객체인 `TreeModel` 을 참조하기만 한다.
- `Tree` 클래스에는 인스턴스별로 다른 상태 값만 남겨둔다.<br>

## 수천 개의 인스턴스
- GPU로 보내는 데이터 양을 최소화하기 위해서는
    - **공유 데이터인 TreeModel를 딱 한번만 보낼 수 있어야 한다.**
    - 그런 후에 나무마다 값이 다른 위치, 색, 크기를 전달한다.
    - 마지막으로 GPU에 전체 나무 인스턴스를 그릴 때 공유 데이터를 사용해라고 말하면 된다.
### 인스턴스 렌더링
- 인스턴스 렌더링이란 동일한 지오메트리 데이터를 GPU에 여러 번 전송하는 대신 인스턴싱은 데이터를 한번 전송한 다음 GPU에 다른 변환 또는 속성을 사용하여 여러 번 렌더링하도록 지시합니다.
- 두 개의 데이터 스트림이 필요하다.
- 첫번째 스트림
    - 여러 번 렌더링되어야 하는 공유 데이터
- 두번째 스트림
    - 인스턴스 목록과 이들 인스턴스를 첫 번째 스트림 데이터를 이용해 그릴 때 각기 다르게 보이기 위해 필요한 매개변수
- `Direct3D`, `OpenGL` 모두 `인스턴스 렌더링`을 지원한다.
- `하드웨어`가 지원하는 패턴이다.<br>

## 경량 패턴

- 어떤 객체의 개수가 너무 많아서 좀 더 가볍게 만들고 싶을 때 사용한다.
- `고유 상태` , `자유문맥 상태` : 공유할 수 있는 데이터
    - 두 종류의 객체 데이터
    - 나무 형태(geometry), 텍스처
- `외부상태` : 인스턴스 별로 값이 다른 데이터
    - 나무의 위치, 크기, 색 등<br>

## 예시2 : 지형 정보
- 땅 : 작은 타일들이 모여 있는 거대한 격자
    - 풀, 흙, 언덕, 호수, 강
- 타일 기반
- 지형 종류
- 게임 플레이에 영향을 주는 여러 속성
  - 플레이어 이동 비용 값
  - 강이나 바다처럼 보트로 건너갈 수 있는 곳인지 여부
  - 텍스처<br><br>

### **⬇️  열거형을 사용**
---
```cpp
enum Terrain
{
	TERRAIN_GRASS,
	TERRAIN_HILL,
	TERRAIN_RIVER,
	// 그 외 다른 지형
};
```

```cpp
class World
{
private:
	Terrain Tiles[WIDTH][HEIGHT];
};
```

```cpp
int World::GetMovementCost(int x, int y)
{
	switch(Tiles[x][y])
	{
		case TERRAIN_GRASS: return 1;
		case TERRAIN_HILL: return 2;
		case TERRAIN_RIVER: return 3;
		// 그 외 다른 지형들..
	}
}
int World::IsWater(int x, int y) const
{
	switch(Tiles[x][y])
	{
		case TERRAIN_GRASS: return false;
		case TERRAIN_HILL: return false;
		case TERRAIN_RIVER: return true;
		// 그 외 다른 지형들..
	}
}
```

### **⬇️  플라이웨이트(인스턴스 렌더링) 패턴을 사용하면**
---
![image](https://github.com/user-attachments/assets/dc02d20d-b237-4452-b44d-e4133aeac170)
- 이런 데이터는 하나로 합쳐서 지형 클래스로 만들어서 캡슐화하는 게 좋다.

```cpp
class Terrain
{
public:
	Terrain(int movementCost, bool isWater, Texture NewTexture)
		: MovementCost(movementCost), bIsWater(isWater), texture(newTexture) {  }
	
	int GetMovementCost() const {return MovementCost;}
	bool IsWater() const {return bIsWater;}
	const Texture& GetTexture() const {return texture;}

private:
	int MovementCost;
	bool bIsWater;
	Texture texture;
};
```

- 모든 지형 상태는 `고유`하다 (= `자유 문맥`)
- 모든 메서드는 `const` 를 사용한다
    - Terrain 객체를 여러 곳에서 공유해서 쓰기 때문이다.

- 지형 종류 별로 Terrain 객체가 여러 개 있을 필요가 없다.
- World 클래스 격자 멤버 변수에 열거형이나 Terrain 객체 대신 Terrain 객체의 포인터를 넣을 수 있다.

```cpp
class World
{
private:
	Terrain* tiles[WIDTH][HEIGHT];
	// 그 외..
};
```

- Terrain 인스턴스가 여러 곳에서 사용되다 보니, 동적으로 할당하면 생명 주기를 관리하기가 좀 더 어렵기 때문에, World 클래스에 저장한다.

```cpp
class World
{
public:
	World();
	: GrassTerrain(1, false, GRASS_TEXTURE), HillTerrain(3, false, HILL_TEXTURE), RiverTerrain(2, false, RIVER_TEXTURE);

private:
	Terrain GrassTerrain;
	Terrain HillTerrain;
	Terrain RiverTerrain;
	// 그 외..
};
```

```cpp
void World::GenerateTerrain()
{
	// 땅에 풀을 채운다.
	for(int x =0; x <WIDTH; ++x)
	{
		for(int y=0; y< HEIGHT; ++y)
		{
			// 언덕을 몇 개 놓는다.
			if(random(10) == 0)
			{
				Tiles[x][y] = &HillTerrain;
			}
			else
			{
				Tiles[x][y] = &GrassTerrain;
			}
		}		
	}
	
	// 강을 하나 놓는다.
	int x = random(WIDTH);
	for(int y =0; y <HEIGHT; ++y)
	{
		Tiles[x][y] = &RiverTerrain;
	}
}

// 지형 속성 값을 World의 메서드 대신 Terrain 객체에서 바로 얻을 수 있다.
const Terrain& World::GetTile(int x, int y) const
{
	return *Tiles[x][y];
}
```

- World 클래스는 더 이상 지형의 세부 정보와 커플링되지 않는다.
- 타일 속성은 Terrain 객체에서 바로 얻을 수 있다.

```cpp
int cost = World.GetTile(2,3).GetMovementCost();
```

## 정리
- 경량 패턴 방식이 훨씬 빨랐다.
- 객체가 메모리에 어떤 식으로 배치되느냐에 따라 달라질 수 있다.
- 경량 패턴을 사용하면 객체를 마구 늘리지 않으면서도 객체 지향 방식의 장점을 취할 수 있다.
- 열거 형을 선언해 수많은 다중 선택문을 만들 생각이라면 경량 패턴을 먼저 고려해보자.
</br>

### 오브젝트 풀링과 플라이웨이트 패턴의 차이 
- 오브젝트 풀링과 플라이웨이트 패턴은 객체 생성을 최적화하는 장점이 있다.
- 플라이웨이트 패턴
  - 동일한 상태를 가진 객체를 공유하여 메모리 사용을 줄이는 것이 목표이다.
  - 변경 불가능한 객체를 사용한다. 
- 오브젝트 풀링
  - 객체의 생성과 파괴 비용을 절감하기 위해 객체를 미리 생성해 두고 재사용하는 것이 목표이다.
  - 변경 가능한 객체를 사용한다.
### static 으로 자원을 올려놓으면, 공유해서 자원을 사용하는 것인가 ?
- `static`으로 선언한 객체를 공유하기 때문에, Flyweight 패턴의 원리를 따른 것이다.
- `static`멤버를 사용하면 클래스 레벨에서 자원이 공유되므로, 자원 관리와 접근 제어를 신중하게 다루어야 한다.
