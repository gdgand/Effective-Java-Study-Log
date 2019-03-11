[발표자료](https://ej-study1-sangcomz.now.sh/#0)

# Effective Java 발표
Item 5, 6, 7, 8, 9



 

## 아이템5 : 리소스를 엮을 때는 의존성 주입을 선호하라 :syringe:

 

- 정적 유틸리티로 사용하는 경우
```java
  public class SpellChecker{
    private final Lexicon dictionary = ...;

    private SpellChecker() {}
    
    public static boolean isValid(String word) { ... }
    public static List<String> suggestions (String typo) { ... }
  }
```

 
- 싱글턴을 잘못 사용하는 경우
```java
  public class SpellChecker{
    private final Lexicon dictionary = ...;

    private SpellChecker(...) {}
    public static SpellChecker INSTANCE = new SpellChecker(...);
    
    public boolean isValid(String word) { ... }
    public List<String> suggestions (String typo) { ... }
  }
```

 

## 유연하지 않고 테스트 하기 어렵다!

 

## 사용하는 자원에 따라 동작이 달라지는 클래스에는 
## ~~정적 유틸리티 클래스~~나 ~~싱글턴 방식~~이 *적합하지 않다*.

 

## 의존 객체 주입 패턴 (Dependency Injection)
- 인스턴스를 생성할 때 생성자에 필요한 자원을 넘겨주는 방식
  - Constructor Injection
  - Method(Setter) Injection
  - Field Injection

 

## Constructor Injection
```java
public class SpellChecker {
  private final Lexicon dictionary;

  public SpellChecker(Lexicon dictionary){
    this.dictionary = Objects.requireNonNull(dictionary)
  }

  public boolean isValid(String word) { ... }
  public List<String> suggestions(String typo) { ... }
}
```

 

## Method(Setter) Injection
```java
public class SpellChecker {
  private Lexicon dictionary;

  public SpellChecker(){ ... }

  public void setDictionary(Lexicon dictionary){
    this.dictionary = dictionary;
  }

  public boolean isValid(String word) { ... }
  public List<String> suggestions(String typo) { ... }
}
```
 

## Field Injection
```java
public class SpellChecker {
  @Inject private Lexicon dictionary;

  public SpellChecker(){ ... }

  public boolean isValid(String word) { ... }
  public List<String> suggestions(String typo) { ... }
}
```

 

# 장점
- ### 유연성
- ### 재사용성
- ### 테스트 용이성

 

## 팩터리 메서드 패턴 (Factory Method pattern)
- 상위 클래스에서 객체를 생성하는 인터페이스를 정의하고, 하위 클래스에서 인스턴스를 생성하도록 하는 방식

 

## Supplier interface
```java
@FunctionalInterface
public interface Supplier<T> {
    /**
     * Gets a result.
     * @return a result
     */
    T get();
}
```

 

## Supplier 팩토리
```java
Mosaic create(Supplier<? extends Tile> tileFactory) { 
  Tile tile = tileFactory.get();
  .
  .
  .
 }
```

 

## 의존성이 수 천 개나 되는 큰 프로젝트에서는 코드를 어지럽게 만들기도 한다.
## 그때 `Dagger`, `Guice`, `Spring`과 같은 의존 객체 프레임워크를 사용하면 이 문제를 해결 할 수 있다. 

 

## 아이템6 : 불필요한 객체를 만들지 말자 :no_entry_sign:

 

```java
String s = new String("bikini") // 따라 하지 말 것!
String s = "bikini"
```

-  new String()을 할 경우엔 새로운 인스턴스를 만든다.
-  "bikini"의 경우 하나의 인스턴스를 사용한다.

 

![](https://cdn.journaldev.com/wp-content/uploads/2012/11/String-Pool-Java1.png)

출처 : http://www.journaldev.com/797/what-is-java-string-pool

 

## 정적 팩터리 메서드를 사용해 불필요한 객체 생성 피하기

- #### 계속 새로운 인스턴스 생성
```java
  public Boolean(String s) {
        this(parseBoolean(s));
    }
```

- #### 선언된 TRUE, FALSE 사용
```java
    public static final Boolean TRUE = new Boolean(true);

    public static final Boolean FALSE = new Boolean(false);

    public static Boolean valueOf(String s) {
        return toBoolean(s) ? TRUE : FALSE;
    }
```

 

## Map에서 불필요하게 같은 인스턴스를 만드는 경우
```java
public class UsingKeySet {
    public static void main(String[] args) {
        Map<String, Integer> menu = new HashMap<>();
        menu.put("Burger", 8);
        menu.put("Pizza", 9);

        Set<String> names1 = menu.keySet();
        Set<String> names2 = menu.keySet();

        names1.remove("Burger");
        System.out.println(names2.size()); // 1
        System.out.println(menu.size()); // 1
    }
}
```
출처 : https://github.com/keesun/study/blob/master/effective-java-3rd/src/main/java/me/whiteship/effectivejava3rd/item06/map/UsingKeySet.java

 

## 오토박싱 (auto boxing)
```java
private static long sum(){
  Long sum = 0L;
  for (long i = 0; i <= Integer.MAX_VALUE; i++)
    sum += i; // 값을 더 할때 마다 Long 객체를 만든다.

  return sum;
}
```
### 불필요한 오토박싱을 피하려면 기본 타입을 사용해야한다.

 

## 무조건 객체 생성을 피해야하는 것이 아니고,
## 프로그램의 `명확성`, `간결성`, `기능`을 위해서 추가로 객체를 생성하는 것이라면 일반적으로 좋은 일이다.
 
## 생성 비용이 비싼 객체 ex)Pattern, DB
- #### 정적 초기화 후에 인스턴스 재사용
```java
public class RomanNumerals {
    private static final Pattern ROMAN = Pattern.compile(
            "^(?=.)M*(C[MD]|D?C{0,3})"
                    + "(X[CL]|L?X{0,3})(I[XV]|V?I{0,3})$");

    static boolean isRomanNumeralFast(String s) {
        return ROMAN.matcher(s).matches();
    }
}
```

 

## 아이템7 : 더이상 쓰지 않는 객체 레퍼런스는 없애자 :free:

 

## 자바는 GC가 있기때문에 메모리 관리에 신경쓰지 않아도 될까?

 

## Code 7-1 에서의 pop()
```java
public Object pop() {
  if (size == 0)
    throw new EmptyStackException();
  retrun elements[--size];
}
```

 

- ### 앞의 코드는 OutOfMemoryError를 발생 시킬 수 있다.
- ### 다쓴 참조를 들고 있어서 이 부분에서 GC가 발생하지 않는다.

 

## Code 7-1 에서의 pop() 을 수정해 제대로 구현한 pop()
```java
public Object pop() {
  if (size == 0)
    throw new EmptyStackException();
  Object result = elements[--size];
  elements[size] = null;
  retrun result;
}
```

 

- ### 객체 참조를 null 처리하는 일은 예외적인 경우여야 한다.
  - ##### 스택과 같이 자신의 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야 한다. 
- ### 일반적으로 변수의 범위를 최소한으로 정의했다면 자연스럽게 GC가 이뤄진다.

 

# 캐시, 리스너 혹은 콜백
## 메모리 누수를 일으키는 주범

 

# WeakHashMap 사용
## 캐시의 키에 대한 레퍼런스가 캐시 밖에서 필요 없어지면

## 해당 엔트리를 캐시에서 자동으로 비워준다.

### (:bangbang: 캐시 외부에서 키를 참조하는 동안에만 엔트리가 살아있을 필요가 있는 캐시만 해당)

 

# 백그라운드 스레드 사용 (ScheduledThreadPoolExecutor 같은)
## 특정 시간이 지나면 제거

 

# LinkedHashMap(removeEldestEntry)
## 캐시에 새 엔트리를 추가할때 가장 오래된 엔트리를 제거
```java
new LinkedHashMap<String, String>() {
  @Override
  protected boolean removeEldestEntry(Map.Entry<String,String> eldest) {
    retrun condition; //조건을 만족하면 가장 오래된 기록 제거
  }  
}

```

 


## 아이템8 : Finalizer와 Cleaner는 사용하지 말 것 :no_good:

 

# Finalizer와 Cleaner가 하는 일? 
## 특정 객체와 관련된 자원 회수

 

# 단점
- ### 예측 불가
- ### 메모리 문제 
- ### 성능적인 문제
- ### 보안의 문제

 

# 예측불가 (언제 실행될지 모름)
- System.gc(), System.Finalization() : 가능성 :up: 하지만 여전히 보장 :x:
- Runtime.runFinalizersOnExit() : 보장 :o: 하지만 결함이 존재

 

# 메모리 문제
- finalizer에서 동작해야 하는 작업을 수행하지 못해서 OutOfMemoryError를 발생시킬 수 있음


 

# 성능 문제
- try-with-resources : 12ns
- finalizer : 550ns (50배나 느린 성능)
- cleaner : 66ns (5배 느린 성능)

 

# 보안의 문제
* A클래스를 공격하려는 B 클래스가 A클래스를 상속받는다.
* B클래스는 인스턴스 생성 혹은 직렬화 과정에서 예외가 발생하면, finalizer 가 수행
* 자신을 정적 인스턴스에 할당하며 가비지 컬렉터가 수집 못하게 막음
* 이렇게 만들어진 객체로 허용되지 않았을 작업을 수행

## 예방
- final class가 아닐 경우엔 finalize를 final로 선언해서 방지

 

# 이런 친구들을 도대체 어디에 사용을..?
# :thinking_face:

 

- ## 안전망
- ## Native Peer 

 

## 안전망
자원의 소유자가 close 메서드를 호출하지 않는 것에 대비한 안전망 역할

ex) FileInputStream, FileOutputStream, ThreadPoolExecutor

 

## Native Peer
일반 자바 객체가 네이티브 메서드를 통해 기능을 위임한 네이티브 객체

네이티브 객체의 경우 일반적인 객체가 아니라 GC가 그 존재를 몰라서 발생

 

## try-with-resources를 쓰자!

 

## 아이템9 : Try-Finally 대신 Try-with-Resource 사용하라 :arrows_counterclockwise:

자원 닫기는 클라이언트가 놓치기 쉬워서 예측할 수 없는 성능 문제로 이어지기도 한다.

 

## 전통적인 방법 (try-finally)
```java
static String firstLineOfFile(String path) throws IOException {
  BufferedReader br = new BufferedReader(new FileReader(path));
  try {
    return br.readLine();
  } finally {
    br.close();
  }
}
```
 
## 전통적인 방법 (try-finally)
### 자원이 둘 이상 
```java
static void copy(String src, String dst) throws IOException {
  InputStream in = new FileInputStream(src);
  try {
    OutputStream out = new FileOutputStream(dst);
    try {
      byte[] buf = new byte[BUFFER_SIZE];
      int n;
      while ((n = in.read(buf)) >= 0)
        out.write(buf, 0, n);
    } finally {
      out.close()
    }
  } finally {
    in.close();
  }
}
```

 

# 자원이 둘 이상일 경우에 코드가 너무 지저분 하다.

 

# 하지만 자바7이 투척한 try-with-resource 덕에 모두 해결!

 

# 사용방법?
## AutoCloseable 인터페이스를 구현 하면 사용 가능

 

```java
/**
 * @author Josh Bloch
 * @since 1.7
 */
public interface AutoCloseable {
    void close() throws Exception;
}
```

 

## 투척한 방법 (try-with-resource)으로 변경
```java
static void copy(String src, String dst) throws IOException {
  try (InputStream in = new FileInputStream(src);
      OutputStream out = new FileOutputStream(dst)) {
        byte[] buf = new byte[BUFFER_SIZE];
        int n;
        while ((n = in.read(buf)) >= 0)
          out.write(buf, 0, n);
  }
}
```

 

## 전통적인 방법(try-finally)의 또 하나의 문제
# 디버깅을 어렵게 만든다.

 

```java
static String firstLineOfFile(String path) throws IOException {
  BufferedReader br = new BufferedReader(new FileReader(path));
  try {
    return br.readLine(); //첫 번째 오류 발생!
  } finally {
    br.close(); //두 번째 오류 발생!
  }
}
```
***두 번째로 발생한 오류만 보임***

 

# 하지만 투척한 방법을 사용한다면?

 
```java
public class MyResource implements AutoCloseable {

    public void doSomething() {
        System.out.println("Do something");
        throw new FirstError();
    }

    @Override
    public void close() {
        System.out.println("Close My Resource");
        throw new SecondError();
    }
}
```

 

```java
public class TryWithResourceError {
    public static void main(String[] args) {
        try (MyResource myResource = new MyResource()){
            myResource.doSomething();
        }
    }
}
```

### 발생된 오류 로그 
```java
Exception in thread "main" item09.error.FirstError
	at item09.error.MyResource.doSomething(MyResource.java:7)
	at item09.error.TryWithResourceError.main(TryWithResourceError.java:6)
	Suppressed: item09.error.SecondError
		at item09.error.MyResource.close(MyResource.java:13)
		at item09.error.TryWithResourceError.main(TryWithResourceError.java:5)
```

 

# 감사합니다 :pray:

 



