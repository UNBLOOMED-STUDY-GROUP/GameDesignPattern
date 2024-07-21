### 옵저버 패턴을 써야하는 이유?

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/2fd6a694-632c-42a7-af17-936f9d0e1834/0c2f2fc9-e2a1-4505-88aa-1675d346f95e/Untitled.png)

- 날씨 데이터를 이용해서 디스플레이에 띄운다고 생각해보자.

```cpp
class WeatherData {
// 인스턴스 변수 선언
	public void measurementsChanged() {
		float temp = getTemperature();
		float humidity = getHumidity();
		float pressure = getPressure();
    
		currentWeatherDisplay.update(temp, humidity, pressure);
		forecastWeatherDisplay.update(temp, humidity, pressure);
		staticsticDisplay.update(temp, humidity, pressure);
	}
}
```

- 화면을 바꿀 때 이렇게 각 화면 별로 업데이트 하도록 객체가 나눠져 있다면, 새로운 화면을 추가할 때 또 다른 새로운 객체를 생성해야 한다. 그러면 WeatherData 함수를 고칠 수 밖에 없다.
- 이것은 캡슐화가 되었다고 보기 어렵다.

이러한 구조를 옵저버 패턴으로 해결 할 수 있다.

옵저버 패턴을 쓰기 전에 옵저버 패턴을 알아보자.

### 옵저버 패턴(Observer Pattern)

- 한 객체의 상태가 바뀌면 그 객체에 의존하는 다른 객체들한테 연락이 가고, 자동으로 내용이 갱신되는 방식으로 일대다(one-to-many) 의존성을 정의한다.

옵저버 패턴을 구현하는 방법에는 여러가지가 있지만, 대부분 주제(Subject)인터페이스와 옵저버(Observer) 인터페이스가 들어있는 클래스 디자인한다. 

### **주제 인터페이스와 구현체**

```cpp
interface Subject {
	registerObserver() // 옵저버 등록
	removeObserver() // 옵저버 삭제
	notifyObserver() // 옵저버에게 업데이트 알림
}

class SubjectImpl implements Subject {
	registerObserver() { ... }
	removeObserver() { ... }
	notifyObserver() { ... }

	getState() // 주제 객체는 상태를 설정하고 알기위한 겟터,셋터가 있을 수 있다.
	setState()
}
```

- 정보를 알리는 객체의 인터페이스를 보통 Subject라고 한다. (주제)
- 등록, 삭제, 업데이트 알림 함수로 구현되어있다.
- 옵저버 객체들을 모두 갖고 있는 배열을

### **옵저버 인터페이스와 구현체**

```cpp
interface Observer{ // 옵저버가 될 객체에서는 반드시 Observer 인터페이스를 구현해야함.
	update() // 주제의 상태가 바뀌었을때 호출됨
}

class ObserverImpl implements Observer {
	update() { 
		// 주제가 업데이트 될 때 해야하는 일
	}
}
```

- 옵저버에서 `update()`메소드를 통해 주제의 `state`를 전달 받는다.
- 여러 객체에서 동일한 데이터를 제어하도록 하는 것에 비해 더 깔끔한 객체지향 디자인을 만들 수 있다.

### **느슨한 결합(Loose Coupling)의 위력**

1. 옵저버를 언제든 새로 추가, 제거할 수 있다.
2. 새로운 형식의 옵저버라 할 지라도 주제를 전혀 변경할 필요가 없다.
3. 주제와 옵저버는 서로 독립적으로 재사용 할 수 있다.
4. 주제나 옵저버가 바뀌더라도 서로에게 영향을 미치지 않는다.

Loose Coupling 디자인을 활용하면 변경 사항이 생겨도 무난히 처리할 수 있는 유연한 객체지향 시스템을 구축할 수 있습니다. 객체 사이의 상호 의존성을 최소화 할 수 있기 때문이다. 

다시 날씨 앱 코드를 수정해보자!

```cpp
interface Subject {
	registerObserver()
	removeObserver()
	notifyObserver()
}

class WeatherData implements Subject {
	registerObserver()
	removeObserver()
	notifyObserver()

	getTemperature()
	getHumidity()
	getPressure()
	measurementsChanged()
}
```

```cpp
interface Observer {
	update() // 새로 갱신된 주제 데이터를 전달하는 인터페이스
}

interface DisplayElement {
	display() // 화면에 표현시키는 인터페이스
}
```

**위 인터페이스를 구현한 `현재날씨, 날씨예보, 기상통계` 클래스를 만듭니다.**

```cpp
class CurrentWeather implements Observer, DisplayElement{
	update()
	display() { // 현재 측정 값을 화면에 표시 }
}

class ForcastWeather implements Observer, DisplayElement{
	update()
	display() { // 날씨 예보 표시 }
}

class StatisticsDisplay implements Observer, DisplayElement{
	update()
	display() { // 평균 기온, 평균 습도 등 표시 }
}
```

**만약 새로운 화면 `불쾌지수` 화면을 만든다고 한다면 쉽게 추가 가능**

```cpp
class DiscomfortDisplay implements Observer, DisplayElement{
	update()
	display() { // 불쾌지수를 화면에 표시 }
}
```

### 전체 흐름

```cpp
public class WeatherData implements Subject {
	private List<Observer> observers;
	private float temperature;
	private float humidity;
	private float pressure;

	public WeatherData() {
		observers = new ArrayList<Observer>(); //알림을 알려줄 옵저버 배열 
	}

	public void registerObserver(Observer o) {
		observers.add(o);
	}

	public void removeObserver(Observer o) {
		observers.remove(o);
	}

	public void notifyObservers() {
		for (Observer observer : observers) {
			observer.update(temperature, humidity, pressure);
		}
	}
}

//...
```

```cpp
public class CurrentConditionsDisplay implements Observer, DisplayElement {
	private float temperature;
	private float humidity;
	private WeatherData weatherData;

	public CurrentConditionsDisplay(WeatherData weatherData) {
		this.weatherData = weatherData;
		weatherData.registerObserver(this);
	}

	public void update(float temperature, float humidity, float pressure) {
		this.temperature = temperature;
		this.humidity = humidity;
		display();
	}

	public void display() {
		System.out.println("Current conditions: " + temperature
			+ "F degrees and " + humidity + "% humidity");
	}
}
```

- 옵저버들은 public CurrentConditionsDisplay(WeatherData weatherData) 이 처럼 주제 객체를 전달 받아서 등록해야 한다. (옵저버들은 주제에게 의존성을 갖는다.)
- update함수는 주제 쪽에서 호출한다.


[참고 문헌]

https://velog.io/@hanna2100/디자인패턴-2.-옵저버-패턴-개념과-예제-observer-pattern
