# 동기

게임 안에서가까이 있는 물체들로부터만 소리가 들린다거나, 서로 붙어 있는 유저들끼리만 채팅이 가능하다던지, 주변의 객체들하고만 상호작용하기 위해서 주변에 어떤 객체들이 있는지를 알아야 할 필요가 있다.

모든 오브젝트들의 서로 간의 거리를 일일히 계산한다면 **O(n^2)의 확인을 매번마다 해줘야 한다.**

![스크린샷 2024-08-13 105144](https://github.com/user-attachments/assets/df338b49-c04e-4c40-8b3e-3d15ce710490)


**공간 분할 패턴**에서는 이러한 객체들의 위치에 따라 객체들을 **정렬하여 보관** 해놓는다.

가장 문제는 정렬이 되어 있지 않다는 것이다. 만약 거리순으로 정렬이 되어있다면 이진 탐색 등으로 매우 빠른 시간내에 주변 유닛을 탐색할 수 있다.
 
 전체 범위에 대해 1차원 배열 사용하여 여러 범위로 쪼개어 확장시키면 2차원 이상의 공간을 사용할 수 있고 이것이 바로 공간 분할 패턴이다.

# 공간 분할 패턴

객체들은 공간 위에서의 위치 값을 갖는다. 이들 객체를 객체 위치에 따라 구성되는 공간 자료구조에 저장한다. 공간 자료구조를 통해서 같은 위치 혹은 주변에 있는 객체를 빠르게 찾을 수 있다. 객체 위치가 바뀌면 공간 자료구조도 업데이트해 계속해서 객체를 찾을 수 있도록 한다.

# 언제 사용할 것인가 ?

- 살아 움직이는 게임 객체
- 정적인 Prop 지형을 저장하는데 흔히 사용
- 위치 값이 있는 객체가 많고 위치에 따라 객체를 찾는 질의가 성능에 영향을 줄 정도면 사용

# 주의 사항

공간 분할 패턴을 사용하면 O(n²)인 복잡도를 쓸만한 수준으로 낮출 수 있지만 이는 객체가 많을수록 의미가 생긴다.
만약 n이 충분히 작으면 굳이 신경 쓸 필요가 없다.

객체를 위치에 따라 정리하기 때문에 객체의 위치 변경을 처리하기가 매우 어렵다.
객체의 바뀐 위치에 맞춰서 자료구조를 재정리해야 하기 때문에 코드가 더 복잡해질 뿐더러 CPU도 더 많이 소모하므로 공간 분할 패턴을 적용할만한 값어치가 있는지를 먼저 확인해보아야 한다.

게다가 일반적인 최적화들처럼 속도를 위해 메모리를 희생하기 때문에 메모리가 부족한 환경이라면 오히려 손해일 수도 있다.

# 예제

```cpp
class Unit {
    friend class Grid;

public:
    Unit(Grid* grid, double x, double y) : grid_(grid), x_(x), y_(y) {}
    void move(double x, double y);

private:
    double x_, y_;
    Grid* grid_;
    Unit* prev_;
    Unit* next_;
};

```

- Unit에는 2차원 상의 자기 위치 값과 자기가 속해 있는 Grid 포인터가 존재한다.
- 유닛이 이전 포인터와 다음 포인터를 갖도록 확장하여 배열이 아니라 이중 연결 리스트로 유닛을 관리 할 수 있다.

![스크린샷 2024-08-13 111012](https://github.com/user-attachments/assets/63b56c8e-915c-4e8b-a9f2-a51eb550f678)


```cpp
class Grid {
public:
    Grid() {
        for (int x = 0; x < NUM_CELLS; ++x) {
            for (int y = 0; y < NUM_CELLS; ++y) {
                cells_[x][y] = nullptr;
            }
        }
    }
    
    static const int NUM_CELLS = 10;
    static const int CELL_SIZE = 20;
    
private:
    Unit* cells_[NUM_CELLS][NUM_CELLS];
};
```

- 유닛이 움직일 때 격자에 속해 있는 데이터도 제대로 위치해 있도록 정보를 알아야 하기 때문에 Gird 클래스가 friend로 정의
- 격자 칸은 그 칸에 들어있는 유닛 리스트의 첫 번째 유닛을 포인터로 가리킨다. 유닛은 자기 이전과 이후 유닛을 포인터로 가리킨다.

```cpp
Unit::Unit(Grid* grid, double x, double y)
    : grid_(grid), x_(x), y_(y), prev_(nullptr), next_(nullptr) {
    grid_->add(this);
}

void Grid::add(Unit* unit) {
    int cellX = (int)(unit->x_ / Grid::CELL_SIZE);
    int cellY = (int)(unit->y_ / Grid::CELL_SIZE);
    
    unit->prev_ = nullptr;
    unit->next_ = cells+[cellX][cellY];
    cells_[cellX][cellY] = unit;
    
    if (unit->next_ != nullptr)
        unit->next_->prev_ = unit;
}
```

- 새로 만든 유닛을 적당한 격자 칸에  넣어야 한다.
- Unit 클래스 생성자에서 한다.
- 유닛이 들어갈 칸을 찾은 뒤 그 칸에 들어 있는 리스트 맨 앞에 유닛을 추가한다.
- 칸에 이미 유닛 리스트가 들어 있다면 추가한 유닛 뒤에 유닛리스트를 붙인다.

### 격자를 이용해서 전투를 처리하는 메서드

```cpp
void Grid::handleMelee() {
    for (int x = ; x < NUM_CELLS; ++x) {
        for (int y = 0; y < NUM_CELLS; ++y) {
            handleCell(cells_[x][y]);
        }
    }
}

void Grid::handleCell(Unit* unit) {
    while (unit != nullptr) {
        Unit* other = unit->next_;
        while (other != nullptr) {
            if (unit->x_ == other->x_ && unit->y_ == other->y_)
                handleAttack(unit, other);
            other = other->next_;
        }
        unit = unit->next_;
    }
}
```

- 포인터로 연결리스트를 순회하기 위한 코드만 다를 뿐 이 코드에서도 모든 유닛 쌍에 대해서 같은 위치에 있는지를 검사한다.
- 차이점이 있다면 모든 전장의 유닛을 확인하지 않아도 된다. 칸에 들어 있을 정도로 가까운 유닛들만 검사한다.
- 칸이 너무 작아서 유닛 하나만 있을 정도라면 오버헤드가 커지는 결과를 가져올 수 있다.

### 추가 작업

```cpp
void Unit::move(double x, double y) {
    grid_->move(this, x, y);
}
```

- 유닛이 항상 칸에 묶여 있어야 하다보니 유닛이 다른 칸으로 이동할 때마다 현재 칸의 리스트에서 제거하고 이동하는 칸의 리스트에 추가해주는 작업 진행해야 한다.
- 많은 유닛들을 매 프레임마다 연결 리스트에서 넣었다 뺄 수 있기 때문에 추가와 삭제가 빠른 이중 연결 리스트를 이용할 수밖에 없다.

![스크린샷 2024-08-13 114221](https://github.com/user-attachments/assets/5d2b3b23-e4ea-4fdf-b453-d51fdc6df2b0)


- 위의 그림처럼 사거리 안에 들었음에도 다른 칸인 경우를 꼭 고려해야 한다.

```cpp
void Grid::handleUnit(Unit* unit, Unit* other) {
    while (other != nullptr) {
        if (distance(unit, other) < ATTACK_DISTANCE)
            handleAttack(unit, other);
        other = other->next_;
    }
}
```

- 연결 리스트를 순회하며 근처 거리에 있다면 공격 함수를 호출하는 함수이다.

```cpp
void Grid::handleCell(int x, int y) {
    Unit* unit = cells_[x][y];
    while (unit != nullptr) {
        handleUnit(unit, unit->next_);
        
        if (x > 0) handleUnit(unit, cells_[x-1][y]);
        if (y > 0) handleUnit(unit, cells_[x][y-1]);
        if (x > 0 && y > 0) handleUnit(unit, cells_[x-1][y-1]);
        if (x > 0 && y < NUM_CELLS-1) handleUnit(unit, cells_[x-1][y-1]);
        unit = unit->next_;
    }
```

- handleCell은 더 이상 유닛 리스트가 아닌 칸의 좌표 값을 받도록 변경한다.
- 주변 8칸의 유닛 리스트에 관해서도 충돌 검사를 진행한다.
