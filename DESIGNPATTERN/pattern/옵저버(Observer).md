# 옵저버(Observer)

- 디자인 원칙
  - 서로 상호작용을 하는 객체 사이에서는 가능하면 느슨하게 결합하는 디자인을 사용해야 한다.

> 옵저버 패턴에서는 한 객체의 상태가 바뀌면 그 객체에 의존하는 다른 객체들한테 연락이 가고 자동으로 내용이 갱신되는 방식으로 `일대다(one-to-many)`
의존성을 정의한다. one 이 주제(subject) 이며, many 는 옵저버(observer) 이다.

어떤 이벤트가 발생했을 때 한 객체(__주제(subject)__ 라 불리는)가 다른 객체 리스트(__옵저버(observer)__ 라 불리는)에 자동으로 알림을 보내야 하는 상황에서 옵저버 디자인 패턴을 사용한다. GUI 애플리케이션에서 옵저버 패턴이 자주 등장한다. 버튼 GUI 컴포넌트에 옵저버를 설정할 수 있다. 
그리고 사용자가 버튼을 클릭하면 옵저버에 알림이 전달되고 정해진 동작이 수행된다.

```
옵저버 패턴은 어떤 객체에 이벤트가 발생했을 때, 이 객체와 관련된 객체들( 옵저버들 )에게 통지하도록 하는 디자인 패턴을 말합니다.
즉, 객체의 상태가 변경되었을 때, 특정 객체에 의존하지 않으면서 상태의 변경을 관련된 객체들에게 통지하는 것이 가능해집니다.
이 패턴은 Pub/Sub( 발행/구독 ) 모델으로 불리기도 합니다.
```

## 느슨한 결합(Loose coupling)

두 객체가 느슨하게 결합되어 있다는 것은, 그 둘이 상호작용을 하긴 하지만 서로에 대해 잘 모른다는 것을 의미한다. 옵저버 패턴은 느슨한 결합을 제공한다. 이유는 아래와 같다.

- 주제가 옵저버에 대해서 아는 것은 옵저버가 특정 인터페이스(Observer 인터페이스)를 구현 한다는 것 뿐이다. 
  - 옵저버의 구상 클래스가 무엇인지, 옵저버가 무엇을 하는지 등에 대해서 알 필요가 없다.
- 옵저버는 언제든 새로 추가할 수 있다.
  - 실행 중 옵저버를 변경할 수도 있고, 제거할 수도 있다.
- 새로운 형식의 옵저버를 추가하려고 할 때도 주제를 전혀 변경할 필요가 없다.
  - 새로운 클래스에서 Observer 인터페이스를 구현하고 옵저버로 등록하면 된다.
- 주제나 옵저버가 바뀌더라도 서로한테 영향을 미치지는 않는다.

> 느슨한 결합하는 디자인을 사용하면 변경 사항이 생겨도 무난히 처리할 수 있는 유연한 객체지향 시스템을 구축할 수 있다. 객체 사이의 상호의존성을 최소화 할 수 있다.

## Example 1

옵저버 패턴으로 트위터 같은 커스터마이즈된 알림 시스템을 설계하고 구현할 수 있다. 다양한 신문 매체(뉴욕타임스, 가디언 등)가 뉴스 트윗을
구독하고 있으며 큭정 키워드를 포함하는 트윗이 등록되면 알림을 받고 싶어한다.

- 옵저버 인터페이스는 새로운 트윗이 있을 때 주제(Feed)가 호출할 수 있도록 notify라고 하는 하나의 메서드를 제공한다.

- 옵저버 인터페이스

```java
interface Observer {
  void notify(String tweet);
}
```

- 옵저버 구현

```java
class NYTimes implements Observer {
  public void notify(String tweet) {
    if(tweet != null & tweet.contains("money")) {
      System.out.println("Breaking news in NY! " + tweet);
    }
  }
}

class Guardian implements Observer {
  public void notify(String tweet) {
    if(tweet != null & tweet.contains("queen")) {
      System.out.println("Yet more news from London! " + tweet);
    }
  }
}
```

- 주제 인터페이스

```java
interface Subject {
  void registerObserver(Observer o);
  void notifyObservers(String tweet);
  void removeObserver(Observer o);
}
```

주제는 registerObserver 메서드로 새로운 옵저버를 등록한 다음에 notifyObservers 메서드로 트윗의 옵저버에 이를 알린다.

- 주제 구현

```java
class Feed implements Subject {
  private final List<Observer> observers = new ArrayList<>();
  // 옵저버 등록
  public void registerObserver(Observer o) {
    this.observers.add(o);
  }
  // 알림
  public void notifyObservers(String tweet) {
    observers.forEach(o -> o.notify(tweet));
  }
  // 옵저버 제거
  public void removeObserver(Observer o) {
    int i = this.observers.indexOf(o);
    if(i >= 0) {
      observers.remove(i);
    }
  }
}
```

- 주제와 옵저버를 연결하는 데모 애플리케이션

```java
Feed f = new Feed();
f.registerObserver(new NYTimes());
f.registerObserver(new Guardian());
f.notifyObservers("The queen said her favourite book is Modern Java in Action!");
```

### 람다로 리팩토링하기 


```java
f.registerObserver(String(tweet) -> {
  if(tweet != null && tweet.contains("money")) {
    System.out.println("Breaking news in NY! " + tweet);
  }
});
f.registerObserver(String(tweet) -> {
  if(tweet != null && tweet.contains("money")) {
    System.out.println("Yet more news from London! " + tweet);
  }
});
```

## Example 2

> 기상 스테이션

- 주제 인터페이스

```java
public inteface Subject {
  // 옵저버 등록
  public void registerObserver(Observer o);
  // 옵저버 제거
  public void removeObserver(Observer o);
  // 주제 객체의 상태가 변경되었을 때 모든 옵저버들한테 알리기 위해 호출되는 메서드
  pulbic void notifyObservers();
}
```

- 옵저버 인터페이스

```java
public interface Observer {
  // 기상정보가 변경되었을 때 옵저버한테 전달되는 상태값들
  public void update(float temp, float humidity, float pressure);
}
```

- 디스플레이 항목에서 구현해야 하는 인터페이스

```java
public interface DisplayElement {
  public void display();
}
```

- 주제 인터페이스 구현체

```java
public class WeatherData implements Subject {
  private ArrayList observers;
  private float temperature;
  private float humidity;
  private float pressure;
  
  /**
   * 생성자에서 Observer 객체를 저장하기 위한 observers 초기화
   */
  public WeatherData() {
    observers = new ArrayList();
  }
  
  /**
   * 옵저버가 등록을 하면 목록 맨 뒤에 추가
   */
  public void registerObserver(Observer o) {
    observers.add(o);
  }
  
  
  /**
   * 옵저버가 탈퇴를 신청하면 목록에서 제거
   */
  pulbic void removeObserver(Observer o) {
    int i = observers.indexOf(o);
    if (i >= 0) {
      observers.remove(i);
    }
  }
  
  
  /**
   * 상태가 변경되었을때, 모든 옵저버들한테 알리는 역할
   * ex) 유튜브 구독을 생각했을때, 보겸 이라는 유튜버가 영상을 올리면 해당 구독자들 전부한테 알리는 역할 등을 구현할 수 있다.
   */
  pulbic void notifyObservers() {
    for(int i=0; i<observers.size(); i++) {
      Observer observer = (Observer) observers.get(i);
      observer.update(temperature, humidity, pressure);
    }
  }
  
  
  /**
   * 기상 스테이션으로부터 갱신된 측정치를 받으면 옵저버한테 알린다.
   */
  public void measurementsChanged() {
    notifyObservers();
  }
  
  public void serMeasurements(float temperature, float humidity, float pressure) {
    this.temperature = temperature;
    this.humidity = humidity;
    this.pressure = pressure;
    measurementsChanged();
  }
  
  // 기타 메서드
}
```

- 디스플레이 항목

```java
public class CurrentConditionsDisplay implements Observer, DisplayElement {
  private float temperature;
  private float humidity;
  private Subject weatherData;
  
  
  /**
   * 생성자에 weatherData 라는 주제 객체가 전달되며, 그 객체를 써서 디스프레이를 옵저버로 등록한다.
   */
  public CurrentConditionsDisplay(Subject weatherData) {
    this.weatherData = weatherData;
    weatherData.registerObservers(this);
  }
  
  
  /**
   * update() 가 호출되면 기온과 습도를 저장하고 display() 호출
   */
  public void update(float temperature, float humidity, float pressure) {
    this.temperature = temperature;
    this.humidity = humidity;
    diplay();
  }
  
  
  /**
   * 가장 최근에 얻은 기온과 습도 출력
   */
  public void display() {
    System.out.println("Current conditions : " + temperature + "F degrees and " + humidity + "% humidity");
  }
}
```

- 테스트용 클래스

```java
public class WeatherStation {
  public static void main(String[] args) {
    WeatherData weatherData = new WeatherData();
    
    CurrentConditionsDisplay currentDisplay = new CurrentConditionsDisplay(weatherData);
    StatisticsDisplay statisticsDisplay = new StatisticsDisplay(weatherData);
    ForecastDisplay forecastDisplay = new ForecastDisplay(weatherData);
    
    weatherData.setMeasurements(80, 65, 30.4f);
    weatherData.setMeasurements(82, 70, 29.2f);
    weatherData.setMeasurements(78, 90, 29.4f);
  }
}
```

## Example 3

예를 들어, 유튜브를 생각해보겠습니다.

Pub/Sub 모델에 따르면, 유튜브 채널은 발행자가 되고 구독자들은 구독자(옵저버)가 되는 구조입니다.

즉, 유튜버가 영상을 올리면 구독자들은 영상이 올라왔다는 알림을 받을 수 있습니다.

이렇게 각각의 유저들을 유튜브 채널을 구독하고 있는 옵저버가 됩니다.

## 자바 내장 옵저버 

java.util.Observable 클래스와 java.util.Observer 인터페이스가 있다. 이 두개는 직접 구현하는것 보다 훨씬 많은 기능을 지원한다. `푸시 방식` 으로
갱신할 수도 있고, `풀 방식` 으로 갱신할 수도 있다.

- java.util.Observable 

```java
public class Observable {
    private boolean changed = false;
    private Vector<Observer> obs;

    /** Construct an Observable with zero Observers. */

    public Observable() {
        obs = new Vector<>();
    }

    /**
     * Adds an observer to the set of observers for this object, provided
     * that it is not the same as some observer already in the set.
     * The order in which notifications will be delivered to multiple
     * observers is not specified. See the class comment.
     *
     * @param   o   an observer to be added.
     * @throws NullPointerException   if the parameter o is null.
     */
    public synchronized void addObserver(Observer o) {
        if (o == null)
            throw new NullPointerException();
        if (!obs.contains(o)) {
            obs.addElement(o);
        }
    }

    /**
     * Deletes an observer from the set of observers of this object.
     * Passing <CODE>null</CODE> to this method will have no effect.
     * @param   o   the observer to be deleted.
     */
    public synchronized void deleteObserver(Observer o) {
        obs.removeElement(o);
    }

    /**
     * If this object has changed, as indicated by the
     * <code>hasChanged</code> method, then notify all of its observers
     * and then call the <code>clearChanged</code> method to
     * indicate that this object has no longer changed.
     * <p>
     * Each observer has its <code>update</code> method called with two
     * arguments: this observable object and <code>null</code>. In other
     * words, this method is equivalent to:
     * <blockquote><tt>
     * notifyObservers(null)</tt></blockquote>
     *
     * @see     java.util.Observable#clearChanged()
     * @see     java.util.Observable#hasChanged()
     * @see     java.util.Observer#update(java.util.Observable, java.lang.Object)
     */
    public void notifyObservers() {
        notifyObservers(null);
    }

    /**
     * If this object has changed, as indicated by the
     * <code>hasChanged</code> method, then notify all of its observers
     * and then call the <code>clearChanged</code> method to indicate
     * that this object has no longer changed.
     * <p>
     * Each observer has its <code>update</code> method called with two
     * arguments: this observable object and the <code>arg</code> argument.
     *
     * @param   arg   any object.
     * @see     java.util.Observable#clearChanged()
     * @see     java.util.Observable#hasChanged()
     * @see     java.util.Observer#update(java.util.Observable, java.lang.Object)
     */
    public void notifyObservers(Object arg) {
        /*
         * a temporary array buffer, used as a snapshot of the state of
         * current Observers.
         */
        Object[] arrLocal;

        synchronized (this) {
            /* We don't want the Observer doing callbacks into
             * arbitrary code while holding its own Monitor.
             * The code where we extract each Observable from
             * the Vector and store the state of the Observer
             * needs synchronization, but notifying observers
             * does not (should not).  The worst result of any
             * potential race-condition here is that:
             * 1) a newly-added Observer will miss a
             *   notification in progress
             * 2) a recently unregistered Observer will be
             *   wrongly notified when it doesn't care
             */
            if (!changed)
                return;
            arrLocal = obs.toArray();
            clearChanged();
        }

        for (int i = arrLocal.length-1; i>=0; i--)
            ((Observer)arrLocal[i]).update(this, arg);
    }

    /**
     * Clears the observer list so that this object no longer has any observers.
     */
    public synchronized void deleteObservers() {
        obs.removeAllElements();
    }

    /**
     * Marks this <tt>Observable</tt> object as having been changed; the
     * <tt>hasChanged</tt> method will now return <tt>true</tt>.
     */
    protected synchronized void setChanged() {
        changed = true;
    }

    /**
     * Indicates that this object has no longer changed, or that it has
     * already notified all of its observers of its most recent change,
     * so that the <tt>hasChanged</tt> method will now return <tt>false</tt>.
     * This method is called automatically by the
     * <code>notifyObservers</code> methods.
     *
     * @see     java.util.Observable#notifyObservers()
     * @see     java.util.Observable#notifyObservers(java.lang.Object)
     */
    protected synchronized void clearChanged() {
        changed = false;
    }

    /**
     * Tests if this object has changed.
     *
     * @return  <code>true</code> if and only if the <code>setChanged</code>
     *          method has been called more recently than the
     *          <code>clearChanged</code> method on this object;
     *          <code>false</code> otherwise.
     * @see     java.util.Observable#clearChanged()
     * @see     java.util.Observable#setChanged()
     */
    public synchronized boolean hasChanged() {
        return changed;
    }

    /**
     * Returns the number of observers of this <tt>Observable</tt> object.
     *
     * @return  the number of observers of this object.
     */
    public synchronized int countObservers() {
        return obs.size();
    }
}
```

- java.util.Observer

```java
public interface Observer {
    /**
     * This method is called whenever the observed object is changed. An
     * application calls an <tt>Observable</tt> object's
     * <code>notifyObservers</code> method to have all the object's
     * observers notified of the change.
     *
     * @param   o     the observable object.
     * @param   arg   an argument passed to the <code>notifyObservers</code>
     *                 method.
     */
    void update(Observable o, Object arg);
}
```

직접 생성하는것과 update 메서드가 다른데, arg 파라미터를 추가로 받는것을 볼 수 있다. 

java 에서 지원하는 Observable 과 Observer 를 사용하는 경우 아래와 같은 방식으로 진행된다.

- `객체의 옵저버가 되는 방법`
  - java.util.Observer 인터페이스를 구현하고 Observable 객체의 addObserver() 메서드를 호출하면된다.
  - 탈퇴하고 싶은 경우는 deleteObserver() 를 호출하면 된다.
- `Observable 에서 연락을 돌리는 방법`
  - 첫 번째로 setChanged() 메서드를 호출해서 객체의 상태가 바뀌었다는 것을 알린다.
  - 그 다음으로 다음 중 하나의 메서드를 호출해야 한다.
    - notifyObservers() 
    - notifyObservers(Object arg) 
      - 이 버전에서는 연락할 때 각 옵저버 객체한테 전달할 임의의 데이터 객체를 받아들인다.

- `푸시 방식`
  - 데이터를 옵저버들한테 보낼 때는 데이터를 notifyObservers(arg) 메서드의 인자로 전달하는 데이터 객체 형태로 전달해야 한다.
- `풀 방식`
  - 옵저버 쪽에서 전달받은 Observable 객체로부터 원하는 데이터를 가져가는 방식
  
java.util.Observable 클래스를 보면 `setChanged()` 메서드의 역할은 연락을 최적화할 수 있게 해준다. 예를들어 온도 센서가 민감해서 0.1 도 단위로 바뀌는 경우 매번 연락을 돌리는 것보다, 0.5 단위로 돌리고 싶은 경우도 있을텐데 이때 setChanged() 가 그 역할을 해준다. `clearChanged()` 는 다시 상태를 false 로 바꾼다. `hasChanged()` 는 현재 상태를 확인한다.

- 단점
  - Observable 은 클래스여서 재사용성이 떨어지게 된다.(옵저버 패턴의 가장 큰 장점이 재사용성이다.)
  - Observable 인터페이스가 없기 때문에 java.util 구현을 다른 구현으로 바꾸는 것이 불가능하다.
  - Observable 내부를 보면 메서드들이 synchronized 로 되어있어서 멀티스레드로 구현하는것은 아예 불가능하다.
  - Observable 클래스의 핵심 메서드를 외부에서 호출할 수 없다. setChanged() 메서드가 protected 로 되어있어서 서브 클래스에서만 호출할 수 있다.

## References.

> https://luckygg.tistory.com/181
