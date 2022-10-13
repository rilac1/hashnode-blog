# 함수형 인터페이스와 람다

# 함수형 인터페이스와 람다 표현식 소개 

## 함수형 인터페이스 
- 인터페이스에 추상 메소드가 하나만 있으면 함수형(FunctionalInterface) 인터페이스다.
- 두 개는 안 됨.
- `@FunctionalInterface` 애노테이션을 통해 강제시킬 수 있음.

```java
public interface FunctionalInterface {

    // 추상 메소드. 앞에 abstract 생략됨.
    int doIt(int number);

    // void doThat(); -> 추상 메소드가 두 개라서 에러 발생.

    // static 메소드
    static void printName() {
        System.out.println("Keesun");
    }

    // default: 인터페이스지만 함수 선언 가능
    default void printAge() {
        System.out.println("40");
    }

}
```
### 고차 함수
함수가 함수를 매개변수로 받을 수 있고 함수를 리턴할 수 있다.

## 람다 표현식
### Java 8 이전
익명 내부 클래스(anonymous inner class) 사용
```java
FunctionalInterface functionalInterfaceOrigin = new FunctionalInterface() {
    @Override
    public int doIt(int number) {
        return number + 10;
    }
};
```
### Java 8 이후
- 람다 형태의 표현법 사용
- 람다는 함수형 인터페이스를 인라인으로 구현할 수 있다.
```java
FunctionalInterface functionalInterfaceLambda = (number) ->  {
    return number + 10;
};
```

# 자바에서 제공하는 함수형 인터페이스
> java.util.function

- `Function<T,R>`
  - T 타입을 입력 받고, R 타입을 리턴
    - R apply(T t)
  - 함수 조합용 메소드
    - andThen: 뒤에 있는 함수 > 앞에 있는 함수에
    - compose: 앞에 있는 함수 > 뒤에 있는 함수에

- `BiFunction<T,U,R>`
  - 두 개의 타입 (T, U)를 입력 받고, R 타입을 리턴
    - R apply(T t, U u)

- `Consumer<T>`
  - T 타입을 입력 받고, 리턴 X
    - void Accept(T t)
  - 함수 조합용 메소드
    - andThen

- `Supplier<T>`
  - 아무 값도 입력 받지 않고, T 타입 리턴
    - T get()

- `Predicate<T>`
  - T 타입을 입력 받고, boolean 타입 리턴
    - boolean test(T t)
  - 함수 조합용 메소드
    - And
    - Or
    - Negate
  - `Predicate.not()`

- `UnaryOperator<T>`
  - T 타입을 입력 받고, T 타입을 리턴
    - T apply(T t)
    - Function<T, R> 의 특수한 형태.

- `BinaryOperator<T>`
  - T 타입 두 개를 입력 받고, T 타입을 리턴
    - T apply(T t1, T t2)
    - BiFunction<T, U, R> 의 특수한 형태.

# 람다 표현식
## 변수 캡쳐 (Variable Capture)

### Effective final
- Java 8 부터 지원하는 기능
- "사실상" final인 변수. (변경하면 안됨)
- final 키워드를 사용하지 않은 변수를 익명클래스나 람다에서 참조할 수 있다.

### 로컬 클래스
```java
int baseNumber = 10;

class LocalClass {
    void printBaseNumber() {
        int baseNumber = 11;
        System.out.println(baseNumber); // 11 출력
    }
}
```

### 익명 클래스
```java
int baseNumber = 10;

Consumer<Integer> integerConsumer = new Consumer<Integer>() {
    @Override
    public void accept(Integer baseNumber) {
        System.out.println(baseNumber);
    }
};

integerConsumer.accept(11); // 11 출력
```
> 로컬 클래스와 익명 클래스는 baseNumber 필드가 덮어써진다. (쉐도윙)

### 람다
```java
int baseNumber = 10;

IntConsumer printInt = (i) -> {
    int baseNumber = 11; // Error!!!
};
```
> 람다는 람다를 감싸고 있는 Scope를 공유한다. (쉐도윙하지 않는다.)

# 메소드 레퍼런스
함수형 인터페이스를 기존의 메소드나 생성자와 똑같이 구현할 경우에는 **메소드 레퍼런스**를 사용해라.

아래 Greeting 이라는 임의의 클래스가 있다.
```java 
public class Greeting {

    private String name;

    public Greeting() {
    }

    public Greeting(String name) {
        this.name = name;
    }

    public String hello(String name) {
        return "hello " + name;
    }

    public static String hi(String name) {
        return "hi " + name;
    }
}
```
이 Greeting 클래스에 사용된 `static method`, `생성자`, `instance method` 등을 함수형 인터페이스를 사용하여 구현할 때,
아래와 같이 메소드 레퍼런스를 사용할 수 있다.

### 1. static method 참조
```java
UnaryOperator<String> hi = Greeting::hi;
```

### 2. 생성자
```java
// Greeting()
Supplier<Greeting> newGreeting1 = Greeting::new;

// Greeting(String name)
Function<String, Greeting> newGreeting2 = Greeting::new;
```

### 3. 특정 객체의 instance method 참조
```java
// newGreeting을 통해 greeting 객체 생성
Greeting greeting1 = newGreeting1.get();
Greeting greeting2 = newGreeting2.apply("tom");

UnaryOperator<String> hello = greeting1::hello;
hello.apply("hello");
```

### 4. 임의의 객체의 instance method 참조
String 타입의 Arrays를 정렬할 때에는 일반적으로 아래와 같이 구현한다.
```java
String[] names = {"A", "C", "B"};
Arrays.sort(names, new Comparator<String>() {
    @Override
    public int compare(String o1, String o2) {
        ...
    }
});
```
하지만 String 타입의 메소드 레퍼런스를 사용하면 코드가 매우 간결해진다.
```java
String[] names = {"A", "C", "B"};
Arrays.sort(names, String::compareToIgnoreCase);
```
#### String.compareToIgnoreCase
```java
public int compareToIgnoreCase(String str) {
    return CASE_INSENSITIVE_ORDER.compare(this, str);
}
```