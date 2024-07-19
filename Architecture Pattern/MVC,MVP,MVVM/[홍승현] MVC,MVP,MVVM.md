# 소개

![image](https://github.com/user-attachments/assets/27ba2670-9602-4956-8226-6c8ee22da1cd)

# MVC

![image](https://github.com/user-attachments/assets/27b03540-76f5-41bb-abe8-0e67b7af79e6)
![image](https://github.com/user-attachments/assets/46c0deaf-be94-4db2-aced-0cccd5a5df69)
### MVC 패턴을 사용할 때 지켜야 할 점
1. Model은 Controller와 View에 의존해서는 안된다.
- Model이 Controller, View의 책임을 가지면 안 된다는 것이다. 각 레이어에서 담당하는 책임이라는 것이 존재한다.
- 오로지 데이터에 대한 순수 로직만 존재하는 것이 바람직하다.

2. View는 model에만 의존한다.
- 여기서의 의존은 코드의 의존이 아니라 순수 도메인 데이터에 대한 의존이다.
- view에서 사용되는 값은 Model에서 전달된 값이어야 한다. 컨트롤러가 Model에 대한 값을 View로 넘겨준다.

3. Controller는 Model과 View에 의존해야 한다.

4. View가 Model로부터 데이터를 받을 때는 Controller에서 받아야 한다,
- Controller는 요청에 맞는 서비스를 호출하고 그에 로직이 처리된 Model의 결과를 View로 전달해야 한다.

### 장점
- 비즈니스 로직(Model)과 UI로직(View)을 분리하여 유지보수를 독립적으로 수행가능하다. 따라서 디자이너와 개발자의 분업이 가능하며 유지보수에 유리하다 (의존성을 완전히 분리시킬 수 없다)

- Model과 View가 다른 컴포넌트들에 종속되지 않아 애플리케이션의 확장성, 유연성에 유리하다.

### 단점
- View가 업데이트 될 때마다 Model도 업데이트되므로 의존성이 강하다.
  
- 화면에 복잡한 화면과 데이터의 구성 필요한 구성이라면, Controller에 다수의 Model과 View가 복잡하게 연결되어 있는 상황이 생길 수 있다. MVC에서 View는 Controller에 연결되어 화면을 구성하는 단위요소이므로 다수의 View들을 가질 수 있다. 그리고 Model은 Controller를 통해서 View와 연결되어지지만, 이렇게 Controller를 통해서 하나의 View에 연결될 수 있는 Model도 여러개가 될 수 있다. 즉, 서비스가 커질수록 컨트롤러의 코드가 증가할 수 밖에 없다.

- 또한 Model과 View사이에는 Controller를 통해 소통을 이루기에 의존성이 완전히 분리될 수 없다. 그래서 복잡한 대규모 프로그램의 경우 다수의 View와 Model이 Controller를 통해 연결되기 때문에 컨트롤러가 불필요하게 커지는 현상이 발생한다. 이러한 현상을 Massive-View-Controller현상이라고 하며 이를 보완하기 위해 MVP, MVVM, Flux, Redux등의 다양한 패턴들이 생겨났다.

# MVP
![image](https://github.com/user-attachments/assets/15fdf107-e197-438f-b5dd-e1ec2d20f39a)
- MVC의 한계점인 View-Model 간 의존성을 해결하고자 등장했다.

- Presenter는 View에서 요청한 정보로 Model에서 받은 데이터를 가공하여 View에게 전달해주는 역할을 한다.

- 이렇게만 말해서는 MVC의 Controller와의 차이점을 알기가 모호할 수 있다. MVP는 MVC와 다르게 사용자의 입력이 View를 통해 들어온다.

- MVP 패턴에서 Presenter는 Model을 조작할 뿐만 아니라 View를 업데이트한다. Presenter와 View는 서로 완전히 분리되어 인터페이스를 통해 통신하므로 이 방식은 MVC보다 단위테스트가 훨씬 쉬워진다.

- 장점 : MVP패턴은 인터페이스를 통해 통신하기 때문에 MVC의 단점으로 지적되었던 View와 Model사이의 의존성이 없다.

- 단점
    - View와 Presenter사이의 의존성이 높고, 앱이 커질수록 이 의존성은 더 강해진다.
    - View : Presenter = 1 : 1 관계 형성 (각 View마다 Presenter가 필요하다)
    - 프로젝트가 확장될 수록 코드가 기하급수적으로 증가한다.

# MVVM
![image](https://github.com/user-attachments/assets/ace94bdf-ac52-4f6a-9f33-4e949e431dd9)

- MVP 패턴을 통해, MVC 패턴의 한계점인 View와 Model 사이의 의존성은 해결되었지만, View와 Presenter 사이에 강한 의존성을 지니게 된다는 단점이 발생했다. 이러한 단점을 보완하기 위해 View Model을 도입한 MVVM 패턴이 등장하게 되었다.

- View Model은 View를 표현해주는 View를 위한 Model이다. 오직 View를 나타내기 위한 데이터 처리를 하며, 오직 View 에게 맞춰진 Model을 제공해주는 역할을 한다.

-  MVVM도 MVP처럼 사용자의 입력 Action이 View를 통해 들어온다.

-  Command 패턴으로 ViewModel에 Action이 전달된다.
    - Command 패턴: 객체의 행위, 즉 메서드를 클래스로 만들어 캡슐화함으로써 주어진 기능을 실행 가능한 재사용성이 높은 클래스를 설계하는 패턴이다. 이벤트가 발생했을 때 실행될 기능이 다양하면서 변경이 필요할 경우, 이벤트를 발생시키는 클래스를 변경 없이 재사용할 때 유용하다.
 
- View는 ViewModel과 데이터 바인딩하여 화면에 나타낸다.

- MVVM 패턴은 양방향 데이터 바인딩을 지원한다. 이에 따라 뷰 모델 안에 있는 데이터의 변화를 View에 Broadcast한다. 일반적으로 옵저버 패턴을 활용하여 View Model의 변경사항을 Model에게 알린다.

- 언뜻 보기에는 MVP와 비슷한 부분이 많으나 MVP는 View와 Presenter 사이의 의존관계가 1:1로 형성되어있다면, MVVM은 View와 ViewModel사이의 관계가 1:n으로 되어있다. 또한 데이터 바인딩을 이용한다면 View와 ViewModel 사이의 의존성을 없앨 수 있다.

- 예를 들어 HP 가 변경된 이벤트가 있으면, View Model 이 이 이벤트를 구독하고 해당 이벤트가 발생하면 Model 은 내부 데이터를 업데이트한 다음 바인딩 메커니즘으로 인해 데이터가 인터페이스에 표시된다.

![image](https://github.com/user-attachments/assets/ed010dfb-f9bc-4fe1-959f-19bafa1e008f)

# 코드 예시 (UE5 C++)
### MVC
물약을 마셔서 체력을 증가시키는 상호작용. 

모델에서 체력이 변경되면 델리게이트를 통해 컨트롤러에 알리고, 컨트롤러는 이를 뷰에 반영하여 UI(HUD)를 업데이트.

즉, 델리게이트를 사용하여 모델에서 변경된 데이터를 컨트롤러가 감지하고, 이를 뷰에 반영하여 UI를 업데이트하는 방식으로 MVC 패턴을 구현


#### Model

```cpp
// PlayerStats.h
#pragma once

#include "CoreMinimal.h"
#include "PlayerStats.generated.h"

DECLARE_DYNAMIC_MULTICAST_DELEGATE(FOnHealthChanged);

UCLASS(Blueprintable)
class MYPROJECT_API UPlayerStats : public UObject
{
    GENERATED_BODY()

public:
    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Stats")
    int32 Health;

    UPROPERTY(EditAnywhere, BlueprintReadWrite, Category="Stats")
    int32 MaxHealth;

    // 체력 변경을 알리기 위한 델리게이트
    FOnHealthChanged OnHealthChanged;

    // 체력 설정 함수
    void SetHealth(int32 NewHealth)
    {
        Health = FMath::Clamp(NewHealth, 0, MaxHealth);
        OnHealthChanged.Broadcast();
    }

    // 체력 증가 함수
    void IncreaseHealth(int32 Amount)
    {
        SetHealth(Health + Amount);
    }
};
```

#### View

```cpp
// PlayerHUD.h
#pragma once

#include "CoreMinimal.h"
#include "Blueprint/UserWidget.h"
#include "PlayerHUD.generated.h"

UCLASS()
class MYPROJECT_API UPlayerHUD : public UUserWidget
{
    GENERATED_BODY()

public:
    UFUNCTION(BlueprintImplementableEvent, Category="HUD")
    void UpdateHealthUI(int32 CurrentHealth, int32 MaxHealth);

    UFUNCTION(BlueprintCallable, Category="HUD")
    void OnDrinkPotionClicked();
};
```

#### Controller

```cpp
// PlayerController.h
#pragma once

#include "CoreMinimal.h"
#include "GameFramework/PlayerController.h"
#include "PlayerStats.h"
#include "PlayerHUD.h"
#include "PlayerController.generated.h"

UCLASS()
class MYPROJECT_API APlayerController : public APlayerController
{
    GENERATED_BODY()

public:
    virtual void BeginPlay() override;

private:
    // 플레이어의 상태를 저장하는 포인터
    TObjectPtr<UPlayerStats> PlayerStats;

    // HUD를 가리키는 포인터
    TObjectPtr<UPlayerHUD> PlayerHUD;

    // 초기화 함수
    void Initialize();

    // 체력 변경 시 호출되는 함수
    void OnHealthChanged();

    // 물약을 마시는 함수
    void OnDrinkPotion();
};
```

```cpp
// PlayerController.cpp
#include "PlayerController.h"
#include "Kismet/GameplayStatics.h"
#include "Blueprint/UserWidget.h"

void APlayerController::BeginPlay()
{
    Super::BeginPlay();
    Initialize();
}

void APlayerController::Initialize()
{
    // 블루프린트에서 설정된 PlayerHUD를 생성
    PlayerHUD = CreateWidget<UPlayerHUD>(this, LoadClass<UPlayerHUD>(nullptr, TEXT("/Game/UI/PlayerHUD.PlayerHUD_C")));
    if (PlayerHUD)
    {
        PlayerHUD->AddToViewport();

        // 플레이어 상태 초기화
        PlayerStats = NewObject<UPlayerStats>();
        PlayerStats->MaxHealth = 100;
        PlayerStats->SetHealth(50);

        // 체력 변경 시 HUD 업데이트 설정
        PlayerStats->OnHealthChanged.AddDynamic(this, &APlayerController::OnHealthChanged);

        // 초기 UI 업데이트
        OnHealthChanged();
    }
}

void APlayerController::OnHealthChanged()
{
    if (PlayerStats && PlayerHUD)
    {
        PlayerHUD->UpdateHealthUI(PlayerStats->Health, PlayerStats->MaxHealth);
    }
}

void APlayerController::OnDrinkPotion()
{
    if (PlayerStats)
    {
        PlayerStats->IncreaseHealth(20);
    }
}
```
