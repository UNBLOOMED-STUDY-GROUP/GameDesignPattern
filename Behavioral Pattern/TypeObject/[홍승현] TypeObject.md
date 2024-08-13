# 추천하는 분

🤔 비슷하지만 조금씩 다르긴 한 타입의 여러 객체들을 표현하고 싶다면 ?

🤔 그런 객체의 수가 많다면 ?

🤔 앞으로도 추가될 가능성이 많다면 ?

→ 타입 객체 패턴을 활용하는 방법도 있습니다

---

# 타입 객체 패턴

## 소개

`Type Object` 패턴은 게임 개발에서 가장 유용한 패턴 중 하나로, 

**데이터 주도형 디자인 (Data Driven Design)** 에 속한다. 

데이터 주도형 디자인은 게임의 기능을 최대한 코드 밖으로 빼내서 데이터쪽으로 옮기는 것을 의미한다.

프로그래머가 새로운 기능을 추가하지 않고도 디자이너 및 기획자가 쉽게 클래스의 변형을 정의할 수 있게 한다. 

(=**협업에 용이**하다)

## 플라이웨이트 패턴과의 차이점

타입 객체 패턴은 플라이웨이트 패턴 (경량 패턴) 이랑 많이 유사한데,

타입 객체 패턴은 **플라이웨이트 패턴**의 `암시적 데이터/명시적 데이터`의 개념을 가져와 **게임 플레이의 세계로 확장** 한 것이기 때문이다.

원리는 동일하다. 특정 타입의 모든 인스턴스에서 공통적인 모든 데이터를 분리하지만, 

이를 모든 곳에서 참조하는 대신, 이 데이터를 혼합하여 많은 변형 **(=타입)** 을 만들어낸다. 

그 결과 동일한 기능을 가지지만 사용하는 암시적 데이터의 집합이 다른 객체들로 연결된 구조이다.

이 내용을 아래 그림으로 정리해보았다.


![](https://velog.velcdn.com/images/strurao/post/2df7b68c-5cc8-4e61-83a4-0a1b01f2520d/image.png)

## 클래스로 만들지 않은 이유?

위 그림에는 클래스라는 개념이 없다. 

(클래스를 사용하는 타입 객체 패턴이 불가능한 건 아니다, 클래스를 만드는 방법으로도 효율적인 성능을 만들어낼 수 있기 때문에 케바케이다)

그런데 위 그림과 다르게, 플라이웨이트 패턴을 유지한 채로 각 타입에 대해 새 **클래스**를 만들 수도 있다. 

위 그림에서 클래스를 사용하지 않는다면 그 이유가 뭘까?

우선, 클래스가 많아지면 많아질 수록 유지보수가 어렵다는 단점이 있다.

그리고 지나친 OOP (Object-Oriented Programming; 객체 지향 프로그래밍) 보다는,

DoD (Data-Oriented Design; 데이터 지향 설계) 가 낫다. 

DoD는 *게임 최적화 : 데이터 지역성 패턴* 과 연관이 있다. 

객체지향 프로그래밍이 나온지 얼마 안되었던 옛날에는, 클래스 수를 많이 만들 수록 개발자의 연봉이 높아졌다고 한다. 그만큼 '클래스'에 집중했는데,

사실 객체를 지향하면 가독성도 좋고 프로그래머도 편하지만, HW 에게 마냥 친화적인 것만은 아니다.

최근에는 하드웨어가 곧 플랫폼이다. 모바일, PC, 콘솔.. 하드웨어 사양이 다양하다. 

게임 개발 같이 다양한 하드웨어에서 각각 **고성능을 내야 하는 경우**에는 

캐시 히트율을 높이기 위해 지나친 OOP는 피한다고 한다. (자세한 내용은 아래 링크에서 볼 수 있다)

[참고 : 개발자가 알아야 할 데이터 지향 설계란?](https://yozm.wishket.com/magazine/detail/2157/)

[참고 : OOP Is Dead, Long Live Data-oriented Design](https://youtu.be/yy8jQgmhbAU?si=au4whukADYQhdUIw)

[참고 : Data-Oriented Design (Or Why You Might Be Shooting Yourself in The Foot With OOP)](http://gamesfromwithin.com/data-oriented-design)

[참고 : Data oriented design (데이터 중심 디자인)](https://starmethod.tistory.com/3)

이 내용을 정리해보자면 아래와 같다.

> 데이터 주도형 디자인 (Data Driven Design) 에 속하며 객체 지향 프로그래밍의 연장선상에 있는 타입 객체 패턴을 사용한다고 해도, 캐시 히트율을 높이기 위해서 타입 객체 패턴과 DoD를 혼용해서 자유롭게 설계할 수 있다.
> 

> 그렇지만 DoD는 딱히 데이터 주도형 디자인 (Data driven Design) 과는 관계가 없다. 
데이터 주도형 디자인은 게임의 기능을 최대한 코드 밖으로 빼내서 데이터쪽으로 옮기는 것을 의미하고, 
DoD은 데이터 주도형 디자인과 독립된 개념으로서, 어떤 방식의 프로그래밍과도 함께 쓰일 수 있다.
> 

## UE5 타입 객체 패턴 구현

언리얼에서 타입 객체 패턴을 어떻게 활용할 수 있을지 알아보자!

타입 객체 패턴이 Unreal에서 작동하게 하려면, 

런타임에 RAM에 로드될 수 있는 데이터를 수집할 수 있어야 한다. 

JSON 또는 XML과 같은 다른 구조화된 파일 형식을 사용할 수 있다. 이 때 문제는 디자이너가 변경 사항을 적용하기 위해 별도의 텍스트 편집기에서 데이터를 열어야 하고, 많은 작은 변경 사항이 반복적으로 이루어지는 경우 편집기 미리 보기가 번거로워질 수 있다는 점이다.

다행히도 Unreal Engine은 사용할 수 있는 몇 가지 내장 구조를 제공하는데, Data Asset, Data Table을 사용할 수 있다.

아래 사진은 구글링해서 주워온... 무기 밸런싱 데이터가 포함된 **Data Table** 애셋의 예시 스크린샷이다.

![](https://velog.velcdn.com/images/strurao/post/535c5f27-92eb-4115-b676-139eab31a5e1/image.png)

Data Table로는 저번 프로젝트에서 했었는데,

Data Asset은 제대로 사용해본 적이 없어서,,, 이번 예시는 Data Asset으로 들어보았다.

**Data Asset**은 데이터 테이블의 개념을 받아들여 이를 개별 행으로 분해한다. 

*+) Data Asset이 구조체와 닮아보이지만, 차이점이 있다!*

*만약 Enemy 라는 구조체가 있다면 이를 사용해서 '적' 객체를 만들 수 있다.* 

*구조체 자체는 메모리에 로드되지 않지만, 그 구조체를 사용하는 특정 인스턴스만 메모리에 로드된다.*

*반면 Data Asset은 그 자체로 World에 새로운 객체를 추가하지 않지만,* 

*코드에서 참조될 때 (=이 데이터를 실제로 사용할 때) Data Asset의 내용이 메모리에 로드된다.*

예를 들어서 EnemyType이라는 Data Asset을 만들면 게임에서 여러 가지의 EnemyType을 정의할 때 사용할 수 있다.

```cpp
UCLASS(BlueprintType)
class TEST_API UEnemyType : public UDataAsset
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    float _Health;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    TObjectPtr<UMaterialInstance> _Material1;

    UPROPERTY(EditAnywhere, BlueprintReadWrite)
    TObjectPtr<UMaterialInstance> _Material2;
};
```

적의 체력과 매터리얼만 변경할 것이라서 위와 같이 작성했다.

핵심은 적 클래스 내에서만 다를 수 있는 다른 클래스에 대한 데이터를 저장하지 않도록 하는 것이다.

예를 들어, `무기 발사 속도`는 `무기 유형`에 따라 다르기 때문에, 이는 EnemyType 내에서 정의되는 데이터가 아니다. 무기 데이터가 EnemyType 내에서 `무기 타입`으로 정의되지 않도록 따로 처리해줘야 한다.

![](https://velog.velcdn.com/images/strurao/post/a0d7af49-b3cc-4af9-8d7f-f45d854f7c17/image.png)


그리고 에디터에서 EnemyType DataAsset 블루프린트를 만들어주면 

`TS_EnemyUnit_01` , ... , `TS_EnemyUnit_100` 등등 

자유롭게 만들고 에디터에서 기획자가 설정할 수 있게 해준다.

이후 코드에서는 설정된 이 DataAsset 의 정보대로 데이터가 초기화되도록 해줄 수 있다.

[참고: 언리얼 엔진의 데이터 기반 게임플레이 엘리먼트](https://dev.epicgames.com/documentation/ko-kr/unreal-engine/data-driven-gameplay-elements-in-unreal-engine)

