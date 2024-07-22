# Flyweight Pattern
### GoF
> 공유(sharing을 통해 많은 수의 소립(fine-grained)객체들을 효과적으로 지원합니다.
### 숲에 들어갈 나무들
- 숲을 그리기 위해서는 전체 데이터를 CPU를 GPU로 버스를 통해 전달해야 한다.
- 나무마다 필요한 데이터
    - 줄기, 가지, 잎의 형태를 나타내는 폴리곤 메시
    - 나무 껍질과 잎사귀 텍스처
    - 숲에서의 위치와 방향
    - 각각의 나무가 다르게 보이도록 크게와 음영 같은 값을 조절할 수 있는 매개변수
## 예시
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
- `Tree` 클래스에는 인스턴스별로 다른 상태 값만 남겨둔다.

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
