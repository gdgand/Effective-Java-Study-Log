# Effective Java 3E

## 1장
**명료성(clarity), 단순성(simplicity)**

> 1. 컴포넌트는 사용자를 놀라게 하는 동작을 해서는 절대 안된다(정해진 동작이나 예측할 수 있는 동작만 수행해야한다)
> 2. 컴포넌트는 가능한 한 작되, 그렇다고 너무 작아서는 안된다.(이 책에서 컴포넌트란 개별 메서드부터 여러 패키지로 이뤄진 복잡한 프레임 워크까지 재사용 가능한 모든 소프트웨어 요소를 뜻한다.)
> 3. 코드는 복사되는게 아니라 재사용되여야 한다.
> 4. 컴포넌트 사이의 의존성은 최소로 유지해야 한다.
> 5. 오류는 만들어지자마자 가능한 한 빨리 (되도록 컴파일 타임에) 잡아야 한다.

 https://github.com/jbloch/effective-java-3e-source-code

___

## 2장 객체 생성과 파괴

### 아이템1. 생성자 대신 정적 팩토리 메소드를 고려하라

> public constructor 대신 **static factory method** 를 써라

//public constructor
```  Java
public class User {

    private final String name;
    private final String email;
    private final String country;

    public User(String name, String email, String country) {
        this.name = name;
        this.email = email;
        this.country = country;
    }

    // standard getters / toString
}
```

//Static Factory Methods
``` Java
public static User createWithDefaultCountry(String name, String email) {
    return new User(name, email, "Argentina");
}
User user = User.createWithDefaultCountry("John", "john@domain.com");
```

### static factory method의 장점
1. 이름을 가질 수 있다.

> BigInteger(int, int, Random) 보다 BigInteger.probablePrime(int, Random)이 훨씬 '값이 소수인 BigInteger를 반환한다'를 잘 설명한다.

2. 호출될 때마다 인스턴스를 새로 생성하지는 않아도 된다.

> 인스턴스를 미리 만들어 놓거나 새로 생성한 인스턴스를 캐싱하여 재활용 하는 식으로 불필요한 객체 생성을 피할수 있다. 객체

3. 반환 타입의 하위 타입 객체를 반환할 수 잇는 능력이 있다.
4. 입력 매개변수에 따라 매번 다른 클래스의 객체를 반환할 수 있다.

``` Java
class KoreanUser extends User {...}
class AmericanUser extends User {...}
// 3.
KoreanUser student = User.getKoreanUser("john");
AmericanUser teacher = User.getUSAUser("john");
// 4.
KoreanUser student = User.getUser("john", "Korea");
AmericanUser teacher = User.getUser("john", "USA");
```
5. static factory method를 작성하는 시점에는 반환할 객체의 클래스가 존재하지 않아도 된다.
> 이런 유연함은 service provider framework를 만드는 근간이 된다.



### static factory method의 단점
1. 상속을 하려면 public이나 protected 생성자가 필요하니 static factory method만 제공하면 하위 클래스를 만들 수 없다.
> 이 제약은 상속보다 컴포지션을 사용하도록 유도하고 불변 타입으로 만들려면 이 제약을 지켜야 한다는 점에서 오히려 장점으로 받아들일 수도 있다.

2. static factory method는 프로그래머가 찾기 어렵다.
> 생성자처럼 API 설명에 명확히 들어나지 않으므로 개발자가 해당 클래스를 인스턴스화 할 방법을 알아내야한다. 따라서 API 문서를 잘 써놓고 메서드 이름도 널리 알려진 규약을 따라 짓는 식으로 완화해야한다.

static factory method에서 흔히 사용하는 명명 방식
``` java
// from : 매개변수 하나를 받는 형변환 메서드
Date d = Date.from(instance);
// of : 매개변수를 여러개 받아 적절한 타입으로 인스턴스 반환
Set<Rank> faceCards = EnumSet.of(JACK, QUEEN, KING);
// valueOf : from과 of의 더 자세한 버전
BigInteger prime = BigInteger.valueOf(Integer.MAX_VALUE);
// instance, getInstance
StackWalker luke = StackWalker.getInstance(options);
// create, newInstance
Object newArray = Array.newInstance(classObject, arrayLen);
//get'Type' : getInstance와 같으나, 생성할 클래스가 아닌 다른 클래스에 팩토리 메소드를 정의할 때 쓴다.
FileStore fs = Files.getFileStore(path);
// new'Type'
BufferedReader br = Files.newBufferedReader(path);
// type : getType과 newType의 간결한 버전
List<Complaint> litany = Collections.list(legacyLitany);
```



### 아이템2. 생성자에 매개변수가 많다면 빌더를 고려하라

// Telescoping Constructor pattern
``` Java
public class Pizza {
  public Pizza(int size) { ... }
  public Pizza(int size, boolean cheese) { ... }
  public Pizza(int size, boolean cheese, boolean pepperoni) { ... }
  public Pizza(int size, boolean cheese, boolean pepperoni, boolean bacon) { ... }
  ...
}
```
> Telescoping Constructor pattern도 쓸수는 있지만, 매개변수 개수가 많아지면 코드를 작성하거나 읽기 어렵다.
> 매개변수 순서 바꿔 건네주는 오류(!!!)를 경험하기 쉽다.

// JavaBeans pattern
``` Java
public class CafeMenu {
	private int coffee = 1;		//필수
	private int beverage = 1;	//필수
	private int dessert = 0;	//선택
	private int bakery = 0;		//선택
	private int drinks = 0;		//선택

	public CafeMenu(){}

	//setter
  //필수
	public void setCoffee(int coffee) {...}
	public void setBeverage(int beverage) {...}
  //선택
	public void setDessert(int dessert) {...}
	public void setBakery(int bakery) {...}
	public void setDrinks(int drinks) {...}
}
```
``` Java
CafeMenu starBucks = new CafeMenu();
starBucks.setCoffee(10);
starBucks.setBeverage(30);
starBucks.setDessert(10);
starBucks.setBakery(20);
starBucks.setDrinks(5);
```
> 객체 하나를 만들려면 메소드 여러 개 호출해야 하고, 객체가 완전히 생성되기 전까지는 일관성(consistency)이 무너진 상태에 놓이게 된다.

// Builder pattern
``` Java
public class CafeMenu {
	private final int coffee;		//필수
	private final int beverage;		//필수
	private final int dessert;		//선택
	private final int bakery;		//선택
	private final int drinks;		//선택

	public static class Builder {
		private final int coffee;		//필수
		private final int beverage;		//필수
		private int dessert = 0 ;		//선택
		private int bakery = 0;			//선택
		private int drinks = 0;			//선택

		//필수 인자 생성자로
		public Builder(int coffee, int beverage) {...}
		//선택적 인자는 Builder 타입의 함수로
		public Builder dessert(int num) {...}
		public Builder bakery(int num) {...}
		public Builder drinks(int num) {...}

		//CafeMenu 타입으로 만들기 함수
		public CafeMenu build(){
      // 유효성 검사!!
			return new CafeMenu(this);
		}
	}

	private CafeMenu(Builder builder){
		coffee = builder.coffee;
		beverage = builder.beverage;
		dessert = builder.dessert;
		bakery = builder.bakery;
		drinks = builder.drinks;
	}
}
```
``` Java
CafeMenu starBucks = new CafeMenu.Builder(30, 20).bakery(10).dessert(5).drinks(10).build();
CafeMenu coffeeBean = new CafeMenu.Builder(20, 10).drinks(5).build();
```
> 빌더 패턴은 (파이썬과 스칼라에 있는) 명명된 선택적 매개변수(named optional parameters)를 흉내 낸 것이다.
> 빌더 패턴은 계층적으로 설계된 클래스와 함께 쓰기에 좋다.
> 매개변수 중 다수가 필수가 아니거나, 같은 타입인 경우 빌더 패틴이 더 낫다.

빌더 패턴 단점
> 빌더 생성 비용이 크지는 않지만 성능에 민감한 상황에서는 문제가 될 수 있다.
> 점층적 생성자 패턴보다는 코드가 장황해서 매개변수가 4개 이상은 되어야 값어치를 한다.



### 아이템3. private 생성자나 열거 타입으로 싱글턴임을 보증하라

// public static final 필드 방식의 싱글턴
``` Java
public class Elvis {
  public static final Elvis INSTANCE = new Elvis();
  private Elvis() {...}
}
```

> private 생성자는 public static final 필드의 Elvis.INSTANCE를 초기화할 때 딱 한 번만 호출된다.
> 장점. 해당 클래스가 싱글턴임이 API에 명백히 들어나고 간결하다.

// 정적 팩토리 방식의 싱글턴
``` Java
public class Elvis {
  private static final Elvis INSTANCE = new Elvis();
  private Elvis() {...}
  public static Elvis getInstance() { return INSTANCE; }

  //싱글턴임을 보장해주는 readResolve 메소드
  private Object readResolve(){
    // '진짜' Elvis를 반환하고, '가짜' Elvis는 가비지 컬렉터에 맡긴다.
    return INSTANCE;
  }
}
```

> 장점. 1) (마음이 바뀌면) API를 바꾸지 않고도 싱글턴이 아니게 변경할 수 있다. 2) 정적 팩토리를 제네릭 싱글턴 팩터리로 만들수있다. 3) 정적 팩토리의 매서드 참조를 공급자로 사용할 수 있다.
> 이러한 장점들이 굳이 필요하지않다면 public static final 필드 방식이 좋다.

// 정적 팩토리 방식의 싱글턴
``` Java
public enum Elvis {
  INSTANCE;
}
```
> 간결하고 추가적인 노력없이 직렬화할 수 있고, 심지어 아주 복잡한 직렬화 상황이나 리플렉션 공격에서도 제2의 인스턴스가 생기는 일을 완벽히 막아준다.


### 아이템4. 인스턴스화를 막으려거든 private 생성자를 사용하라

단순히 정적 메서드와 정적 필드만을 담은 클래스를 만들고 싶을 때가 있다면, private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.

``` Java
public class UtilityClass {
  private UtilityClass(){
    //기본 생성자가 만들어지는 것을 막는다.(인스턴스화 방지용)
    throw new AssertionError();
  }
}
```





## Reference
- https://www.baeldung.com/java-constructors-vs-static-factory-methods
- https://medium.com/@joongwon/multi-thread-%ED%99%98%EA%B2%BD%EC%97%90%EC%84%9C%EC%9D%98-%EC%98%AC%EB%B0%94%EB%A5%B8-singleton-578d9511fd42
