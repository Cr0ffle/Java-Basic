# #12 제네릭, 열거형, 애너테이션

## 제네릭스(Generics)

### 제네릭이란?

- 제네릭은 다양한 타입의 객체들을 다루는 메서드나 컬렉션 클래스에 컴파일 시의 타입 체크를 해주는 기능임
- 객체의 타입을 컴파일시에 체크하기 때문에 객체의 타입 안전성을 높이고 형변환의 번거로움이 줄어듦

### 제네릭 클래스의 선언

```java
class Box<T>
	T item;
	
	void setItem(T item) {
		this.item = item;
	}
	T getItem() {
		return item;
}
```

- 타입변수는 의미있게 사용하면됨 ex Key Value경우 <K, V>
- 기호의 종류만 다를 뿐 임의의 참조형 타입을 의미한다는 개념만 이해하면됨

> 제네릭을 사용하지 않았을 때의 List
>
> ```java
> List integers = Arrays.asList(1,2,3,4);
> for(Object number: numbers) {
> Integer numberAsInteger = (Integer) number; //컬렉션의 요소를 꺼내올때마다 이런식으로 형변환이 필요했음
> System.out.print(numberAsInteger.intValue() + “ “);
> }
> ```
>
> 제네릭이 도입된 이후 List
>
> ```java
> List<Integer> integers = Arrays.asList(1,2,3,4);
> for(Integer integer: integers) {
> 	System.out.print(integer + “ “); //형변환 없이 바로 사용
> }
> 
> ```

```java
Box<String> b = new Box<>();
b.setItem("ABS");

class Box<String> //제네릭 타입을 String으로 지정했을 때
	String item;
	
	void setItem(String item) {
		this.item = item;
	}
	String getItem() {
		return item;
}
```

- 제네릭이 도입되기 이전의 코드와 호환을 위해 제네릭 클래스이지만 예전 방식으로 객체를 생성하는 것이 허용됨
- 제네릭클래스에 타입을 지정하지 않을 수 있지만 컴파일러가 경고를 표시함
- 추가적으로 제네릭클래스에 타입을 지정하지 않으면 T 타입은 Object 타입으로 사용됨 따라서 실제 사용시에 형변환해줘야함

```java
Box b = new Box(); //T는 Object로 간주
b.setItem("ABS"); //경고 unchecked or unsafe operation
b.setItem(new Object()); //경고 unchecked or unsafe operation

Box<Object> b = new Box<>();
b.setItem("ABS"); //경고 발생 안함 
b.setItem(new Object()); //경고 발생 안함
```

- 타입 매개변수에 타입을 지정하는 것을 '제네릭 타입 호출'이라하고 지정된 타입을  '매개변수화된 타입' 이라함

```java
Box<T> //제네릭 클래스. T의 Box or T Box
T      //타입 변수 또는 타입 매개변수
Box    //원시 타입
```

- `Box<String>`과 `Box<Integer>` 는 서로 다른 타입 변수를 가진 같은 클래스임



- 모든 객체에 동일하게 동작해야하는 static 멤버 타입에는 T 변수를 사용할 수 없음 그 이유는 모든 클래스에게 동일하게 제공되어야하지만 이 T 변수는 런타임에 확정되기 때문에 불가능함

```java
class Box<T> {
	static T item; //에러
  static int compare(T t1, T t2) {...} //에러
}
```

- 제네릭 배열 타입의 참조변수를 선언하는 것은 가능하지만 new T[10]와 같은 형태로 배열을 생성하는 것은 안됨 new는 컴파일 시점에 타입T가 뭔지 정확히 알아야하기 때문.

```java
class Box<T> {
	T[] item; //가능
	T[] toArray() {
    T[] tmpArr = new T[itemArr.length]; //에러
  }	
}
```

- 반드시 제네릭 배열을 생성해야할 필요가 있을 경우 리플렉션의 newInstance()와 같이 동적으로 객체를 생성하는 메서드로 배열을 생성하거나 Object 배열을 생성해서 복사한 다음에 T[]로 형변환하는 방법을 사용

> 제네릭 배열 사용하기
>
> ```java
> private List<E> elements;
> 
> public MyStack(Class<E> clazz, int capacity) {
>  elements = (E[]) Array.newInstance(clazz, capacity);
> }
> ```
>
> 사실 배열을 사용하지 않고 Collection 사용하는게 맞음

- 위와 같은 맥락으로 instanceof 연산자도 T타입을 피연산자로 사용할 수 없음



### 제네릭 클래스의 객체 생성과 사용

- 참조변수와 생성자에 대입된 타입이 일치해야함

```java
Box<Apple> appleBox = new Box<Apple>();
Box<Apple> appleBox = new Box<Grape>(); // 에러

Box<Fruit> fruitBox = new Box<Grape>(); // Fruit의 자손이 Grape여도 불가능
```

- 아래와 같이 제네릭 클래스 타입이 상속 관계에 있고 대입된 타입이 같은 것은 괜찮음

```java
Box<Apple> appleBox = new FruitBox<Apple>();
```

- jdk 7부터 추정이 가능한 경우 타입을 생략 가능

```java
Box<Apple> appleBox = new Box<>();
```

- `Box<T>`에 item을 추가할때는 T타입과 다른 타입의 객체는 추가할 수 없음

```java
Box<Apple> appleBox = new Box<>();

appleBox.add(new Aaple()); // OK
appleBox.add(new Grape()); // 에러

```

- 그러나 선언된 타입T에 자손들은 추가될 수 있음 

```java
Box<Fruit> fruitBox = new Box<>();

fruitBox.add(new Aaple()); // Fruit의 자손 Apple
fruitBox.add(new Grape()); // Fruit의 자손 Grape
```



### 제한된 제네릭 클래스

- 다음과 같이 제네릭 타입에 `extends` 를 사용하면 특정 타입의 자손들만 대입할 수 있게 제한할 수 있음

```java  
class FruitBox<T extneds Fruit> { //Fruit의 자손만 타입으로 지정 가능
	ArrayList<T> list = new ArrayList<T>();
}
```

- 인터페이스더라도 `extends` 키워드를 사용함
- 다중 바운드타입은 아래와같이 `&` 로 묶을 수 있음

```java
class FruitBox<T extends Fruit & Eatable> { ... }
```

### 와일드카드

- 와일드카드가 필요한 경우

```java
class FruitBox<T extends Fruit> extends Box<T> {}


class Juicer {
	static Juice makeJuice(FruitBox<Grape> box) {
		for(Grape g : box.getList()) {
			//do something .. 
		}
	}
}

FruitBox<Grape> grapeBox = new FruitBox<Grape>();
FruitBox<Apple> appleBox = new FruitBox<Apple>();

Juicer.makeJuice(grapeBox);
Juicer.makeJuice(appleBox); //에러, makeJuice()의 파라미터 타입이 FruitBox<Grape>이라 불가능




class Juicer {
	static Juice makeJuice(FruitBox<Grape> box) {
		for(Grape f : box.getList()) {
			//do something .. 
		}
	}
  
  static Juice makeJuice(FruitBox<Apple> box) { //파라미터 타입이 FruitBox<Apple> 인 메서드를 생성해줘야함
		for(Apple f : box.getList()) {
			//do something .. 
		}
	}
}



```

- 제네릭 타입은 컴파일러가 컴파일할때만 사용하고 제거해버리기 때문에 제네릭타입이 다른것만으로 오버로딩이 성립되지 않음 따라서 메서드 중복으로 인한 컴파일 에러

- 이런 경우에 와일드카드 `?` 를 사용하면 됨
  - `<? extends T>` 와일드 카드의 상한 제한(upper bound). T와 그 자손들만 가능
  - `<? super T>` 와일드 카드의 하한 제한(lower bound). T와 그 조상들만 가능
  - `<?>` 제한 없음. 모든 타입이 가능함 `<? extends Object>` 와 동일함

```java
class Juicer {
	static Juice makeJuice(FruitBox<? extends Fruit> box) {
		for(Fruit f : box.getList()) {
			//do something .. 
		}
	}
}

FruitBox<Grape> grapeBox = new FruitBox<Fruit>();
FruitBox<Apple> appleBox = new FruitBox<Apple>();

Juicer.makeJuice(grapeBox);
Juicer.makeJuice(appleBox);
```

- Collections.sort() 메서드

```java
public static <T> void sort(List<T> list, Comparator<? super T> c) {  list.sort(c);}
```



### 제네릭 메서드

- 메서드의 선언부에 제네릭 타입이 선언된 메서드를 제네릭 메서드라함.

```java
public static <T> void sort(List<T> list, Comparator<? super T> c) {
  list.sort(c);
}
```

- 제네릭 클래스에 정의된 타입 매개변수와 제네릭 메서드에 정의된 타입 매개변수는 전혀 별개의 것임. 같은 타입 문자 T를 사용해도 같은 것이 아니라는것에 주의해야함

```java
class FruitBox<T> {
	static <T> void sort(List<T> list, Comparator<? super T> c) {
		
	}
}
```

- static 멤버에는 제네릭을 사용할 수 없지만 메서드에는 제네릭 타입을 선언하고 사용하는 것이 가능함 

> 어떻게 정적메서드에 제네릭 타입을 사용할 수 있을까?
>
> ...

- 메서드에 선언된 제네릭 타입은 지역 변수를 선언한 것과 같다고 생각하면 이해하기 쉬움

```java
class Juicer {
	static Juice makeJuice(FruitBox<? extends Fruit> box) {
		for(Fruit f : box.getList()) {
			//do something .. 
		}
	}
}

//제네릭 메서드
class Juicer {
	static <T extends Fruit> Juice makeJuice(FruitBox<T> box) {
		for(Fruit f : box.getList()) {
			//do something .. 
		}
	}
}


FruitBox<Fruit> fruitBox = new FruitBox<Fruit>();
FruitBox<Apple> appleBox = new FruitBox<Apple>();

Juicer.<Fruit>makeJuice(fruitBox);
Juicer.<Apple>makeJuice(appleBox);

//대부분의 경우 컴파일러가 타입을 추정해서 아래와 같이 생략된 형태로 사용 가능
Juicer.makeJuice(fruitBox);
Juicer.makeJuice(appleBox);



```

> - 제네릭 메서드를 반드시 사용해야 하는 경우
>
> ```java
> static void fromArrayToCollection(Object[] a, Collection<?> c) {
>     for (Object o : a) { 
>         c.add(o); // compile-time error
>     }
> }
> 
> static <T> void fromArrayToCollection(T[] a, Collection<T> c) {
>     for (T o : a) {
>         c.add(o); // Correct
>     }
> }
> ```



- 제네릭 메서드 호출시 대입된 타입 생략이 불가능할 경우 아래와 같이 참조변수나 클래스 이름을 생략할 수 없음

```java
<Fruit>makeJuice(fruitBox); // 에러. 클래스 이름 생략 불가
this.<Fruit>makeJuice(fruitBox); // 에러. 클래스 이름 생략 불가
Juicer.<Fruit>makeJuice(fruitBox); // 에러. 클래스 이름 생략 불가
```

> - 제네릭 메서드 호출시 대입된 타입이 생략이 불가능한 경우는 어떤때일까? 
>
> ...

- 제네릭 메서드는 매개변수의 타입이 복잡할 때도 유용하게 사용이 가능함 
- Collections.sort 메서드

```java
public static <T extends Comparable<? super T>> void sort(List<T> list) {
	list.sort(null);
}
```

- 타입 T를 요소로하는 List를 매개변수로 허용
- T는 Comparable을 구현한 클래스이어야 하고 T또는 그 조상의 타입을 비교하는  Comparable 이어야 한다는 것을 의미함
- 만일  T가 Student이고 Person과 조상관계라면 `<? super T>`는  Student, Person, Object가 모두 가능함



- 제네릭 타입과 원시 타입간의 형변환

```java
Box box = null;
Box<Object> objBox = null;

objBox = (Box) objBox; //제네릭타입 -> 원시타입 형변환 가능, 컴파일 경고 발생
objBox = (Box<Object>) box; //원시타입 -> 제네릭타입 형변환 가능, 컴파일 경고 발생

```

- 서로다른 제네릭타입간의 형변환

```java
Box<Object> objbox = null;
Box<String> strBox = null;

objBox = (Box<Object>) strBox; //컴파일 에러
strBox = (Box<String>) objbox; //컴파일 에러

```

- 와일드카드를 쓴 제네릭 타입간의 형변환

```java
Box<? extends Object> wBox = new Box<String>(); //가능
```

- Optional의 형변환 사례

```java
public final class Optional<T> {
    
  	//Optional<?>는 Optional<? extends Object>
    private static final Optional<?> EMPTY = new Optional<>(null);
  
    public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }
  
}
```

- 만약 `EMPTY`가 `Optional<Object>` 타입이었다면 `<T>` 타입으로 형변환이 불가능함

```java
Optional<?> wopt = new Optional<Object>();
Optional<Object> oopt = new Optional<Object>();

Optional<String> sopt = (Optional<String>) wopt //OK
Optional<String> sopt = (Optional<String>) oopt //에러
```

- `Optional<Object>` 을 `Optional<String>` 으로 직접 형변환하는 것은 불가능하지만 와일드카드가 포함된 제네릭 타입으로 형변환하면 가능함

```java
Optional<Object> -> Optional<T> //형변환 불가능
Optional<Object> -> Optional<?> -> Optional<T> //형변환 가능
```

### 제네릭 타입의 제거

- 컴파일러는 제네릭 타입을 이용해서 소스파일을 체크하고 필요한 곳에 형변환을 넣어줌 그리고 제네릭 타입을 제거함
- 컴파일된 파일에는 제네릭 타입에 대한 정보가 없음
- 이렇게 하는 주된 이유는 제네릭이 도입되기 이전의 소스 코드와의 호환성을 유지하기 위해서임
- 제네릭 타입 제거 과정
  1. 제네릭 타입의 bound 제거
     - 제네릭 타입이 `<T extends Fruit>` 라면 T는 Fruit으로 치환, `<T>` 인경우는 T는 Object로 치환
  2. 제네릭 타입을 제거한 후에 타입이 일치하지 않으면 형변환을 추가함



> - Invariance
>
> 기본적으로 자바의 generic은 Invariance, 불변임. 아래의 자바 코드는 문법상의 오류가 없지만
>
> ```java
> Double numDouble = 1.1;
> Number number = numDouble;
> System.out.println(number);
> ```
>
> 아래와 같은 코드는 컴파일 에러가 발생함
>
> ```java
> List<Double> doubles = Arrays.asList(1.1, 2.2, 3.3);
> List<Number> numbers = doubles; // compile error
> ```
>
> `List<Double>` 가 `List<Number>` 의 자손이 아니기 때문임.
>
> 클래스의 상속 관계가 제네릭에서는 상속관계로 유지되지 않는 것을 Invariance라고함 제네릭은 컴파일타임에 타입이 지워지기 때문에 컴파일 이후에는 제네릭의 타입을 알 수 없음
>
> - Covariance
>
> 클래스의 상속 관계가 Generics에서도 유지되는 것을 Covariance, 공변이라고함 Covariance 설정은  `<? extends ParentClass>` 처럼 입력하면됨
>
> ```java
> List<Double> doubles = Arrays.asList(1.1, 2.2, 3.3);
> List<? extends Number> numbers = doubles;
> ```
>
> Covariance를 이용하여 객체를 할당하면, 객체를 read할 수 있지만 write는 어려움
>
> ```java
> List<Double> doubles = Arrays.asList(1.1, 2.2, 3.3);
> List<? extends Number> numbers = doubles; // ok
> 
> Number number = numbers.get(0);
> System.out.println(number);
> numbers.add(1.1); // compile error
> ```
>
> 이 경우 List의 타입이 정해져 있지 않아 `add()` 를 사용할 수 없는 것
>
> ```java
> // compile error, List<>의 타입이 Double인지 알 수 없음
> numbers.add(1.1);
> // compile error, List<>의 타입이 Integer인지 알 수 없음
> numbers.add(1);
> // compile error, List<>의 타입이 Long인지 알 수 없음
> numbers.add(1L);
> ```
>
> - Contravariance
>
> Contravariance는 반대로 변하는 성질. 클래스의 상속 관계가 제네릭에서 반대인 것을 의미함
>
> Contravariance는 `<? super ParentClass>` 로 설정할 수 있음
>
> ```java
> List<Number> numbers = Arrays.asList(1.1, 2, 3L);
> List<? super Double> list = numbers;
> 
> Double number = list.get(0); // compile error
> list.add(new Double(4));
> ```
>
> get()은 `<? super Double>` 타입을 컴파일러가 추론할 수 없기 때문에 컴파일오류가 발생하고 add()의 경우는 `List<Number>` 으로 이미 타입이 결정되었기 때문에 불가능함 아래와 같이 사용하면 가능함
>
> ```java
> List<? super Number> list = new ArrayList<>();
> list.add(1);
> Object object = list.get(0);
> int i = (int) object;
> System.out.println(i);
> ```



## 열거형(enums)

### 열거형이란?

- 열거형은 서로 관련된 상수를 편리하게 선언하기 위한 것으로 여러 상수를 정의할 때 사용하면 유용함
- jdk1.5부터 추가
- 자바의 열거형은 c언어의 열거형보다 더 향상된 것으로 열거형이 갖는 값뿐만 아니라 타입도 관리하기 떄문에 보다 논리적인 오류를 줄일 수 있음
- C언어에서는 타입이 달라도 값이 같으면 조건식 결과가 true이었으나 자바의 열걸형은 타입에 안전한 열거형이라서 실제 값이 같아도 타입이 다르면 컴파일 에러가 발생함
- 상수의 값이 바뀌면 해당 상수를 참조하는 모든 소스를 다시 컴파일해야 하지만 열거형 상수를 사용하면 기존의 소스를 다시 컴파일하지 않아도됨

> 상수를 참조하는 클라이언트는 해당 상수의 참조값을 복사하기때문에 상수의 값이 바뀌면 해당 상수를 참조하는 모든 소스를 다시 컴파일 해야 함. https://stackoverflow.com/questions/17592584/enum-requires-no-recompilation-of-its-clients-how



### 열거형의 정의와 사용

- 다음과 같이 열거형을 정의함

```java
enum 열거형이름 {
	상수명1, 상수명2 ... 
}
```

- 사용방법은 열거형이름.상수명1, 열거형이름.상수명2 형태로 사용함
- 열거형 상수간의 비교에는 `==` 사용이 가능함 but 비교연산자는 불가능. 하지만 compareTo()는 비교연산자도 사용이 가능함
- switch문에도 사용할 수 있지만 case문에 열거형의 이름은 적지 않고 상수의 이름만 적어야 한다는 제약이 있음.

```java
switch (d) {
  case EAST :
  case WEST :
  case SOUTH :
  case NORTH :
}
```



- 열거형에 정의된 모든 상수를 출력하려면 다음과 같이 사용하면 됨

```java
Direction[] dArr = Direction.values();

for(Direction d : dArr) {
	...
}
```



- enum의 기타 메서드

| 메서드                                      | 설명                                                     |
| ------------------------------------------- | -------------------------------------------------------- |
| Class\<E\> getDeclaringClass()              | 얼거형의 Class 객체를 반환한다                           |
| String name()                               | 열거형 상수의 이름을 문자열로 반환한다                   |
| int ordinal()                               | 열거형 상수가 정의된 순서를 반환한다 (0부터 시작)        |
| T valueOf(Class\<T\> enumType, String name) | 지정된 열거형에서 name과 일치하는 열거형 상수를 반환한다 |

- 이외에도 컴파일러가 자동으로 추가해주는 메서드들

| 메서드                        | 설명                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| static E values()             | 열거형의 모든 상수를 배열에 담아 반환한다                    |
| static E valueOf(String name) | 열거형 상수의 이름으로 문자열 상수에 대한 참조를 얻을 수 있게 해준다. |



### 열거형에 멤버 추가하기

- Enum클래스에 정의된 ordinal()이 열거형 상수가 정의된 순서를 반환하지만, 이 값을 열거형 상수의 값으로 사용하지 않는 것이 좋음

> - ordinal()을 사용하지 말아야하는 이유
>
> ordinal()에 의존하는 클라이언트들은 enum의 상수의 순서가 변경됨에 따라 논리가 깨질수 있기 때문에 ordinal()을 사용하지 않는게 좋음.
>
> 대부분의 프로그래머는 이 메서드를 쓸 일이 없음 이 메서드는 EnumSet과 EnumMap 같이 열거 타입 기반의 범용 자료구조에 쓸 목적으로 설계되었음
>
> [https://johngrib.github.io/wiki/java-enum/#ordinal-%EB%A9%94%EC%84%9C%EB%93%9C%EC%9D%98-%EC%82%AC%EC%9A%A9](https://johngrib.github.io/wiki/java-enum/#ordinal-%EB%A9%94%EC%84%9C%EB%93%9C%EC%9D%98-%EC%82%AC%EC%9A%A9)

- 열거형 상수의 값을 열거형 상수 이름 옆에 괄호()안에 넣어주면됨

```java
enum Direction {
	EAST(1), SOUTH(5), WEST(-1), NORTH(10);
  
  private final int value;
  Direction(int value) {this.value = value;}
  
}
```

- 지정된 값을 저장할 수 있는 인스턴스 변수와 생성자를 새로 추가해주어야함
- 열거형의 객체를 생성할수는 없음. 기본적으로 열거형의 생성자는 제어자가 묵시적으로 private임

> - 열거형의 생성자가 private인 이유
>
> 생성자가 public이면 의도하지 않은 값 이외의 상수들 선언될 수 있기 때문. 그리고 enum에 선언된 모든 상수값들은 static하게 접근이 가능하기 때문에 인스턴스를 생성할 이유가 없음
>
> [https://www.geeksforgeeks.org/why-enum-class-can-have-a-private-constructor-only-in-java/](https://www.geeksforgeeks.org/why-enum-class-can-have-a-private-constructor-only-in-java/)

- 필요에따라 열거형 상수에 여러 값을 지정할 수 있지만 그에 맞게 인스턴스 변수와 생성자 등을 새로 추가해주어야함
- 열거형에 추상 메서드를 선언하면 각 열거형 상수가 이추상 메서드를 반드시 구현해야함

```java
public class Main {

    public static void main(String[] args) {

        int distance = 10;
        System.out.println("bus = " + Transportation.BUS.calcFare(distance));
        System.out.println("train = " + Transportation.TRAIN.calcFare(distance));
        System.out.println("ship = " + Transportation.SHIP.calcFare(distance));
        System.out.println("airPlane = " + Transportation.AIRPLANE.calcFare(distance));

    }
}

enum Transportation {
    BUS(100) {
        int calcFare(int distance) {
            return distance * fare;
        }
    },
    TRAIN(150) {
        int calcFare(int distance) {
            return distance * fare;
        }
    },
    SHIP(100) {
        int calcFare(int distance) {
            return distance * fare;
        }
    }, AIRPLANE(300) {
        int calcFare(int distance) {
            return distance * fare;
        }
    };

    protected final int fare;

    Transportation(final int fare) {
        this.fare = fare;
    }

    abstract int calcFare(final int distance);
}
```

```
bus = 1000
train = 1500
ship = 1000
airPlane = 3000
```



### 열거형의 이해

```java
enum Diriection {EAST, SOUTH, WEST, NORTH}
```

- 열거형 상수 하나하나가 Direction 객체임
- 위 문장을 클래스로 정의하면 다음과 같음

```java
class Direction {
	static final Direction EAST = new Direction("EAST");
	static final Direction SOUTH = new Direction("SOUTH");
	static final Direction WEST = new Direction("WEST");
	static final Direction NORTH = new Direction("NORTH");

  private String name;
  
  private Direction(String name) {
    this.name = name;
  }
}
```

- Direction 클래스의 static 상수의 값은 객체의 주소이고 이 값은 바뀌지 않는 값이기때문에 `==`로 비교가 가능한 것
- 모든 열거형은 추상 클래스 Enum의 자손이기때문에 Enum을 흉내내어 MyEnum을 작성하면 다음과 같음

```java
abstract class MyEnum<T extends MyEnum<T>> implements Comparable<T> {
  static int id = 0;
  
  int ordinal;
  String name = "";
  
  public int ordinal() {return ordinal};
  
  MyEnum(String name) {
    this.name = name;
    ordinal = id++;
  }
  public int compareTo(T t) {
    return ordinal - t.ordinal();
  }
  
}
```

- 모든 열거형은 암시적으로 Enum의 자손이고 Java는 다중상속이 불가능하기 때문에 열거형을 확장할 수 없음




## 애너테이션(annotation)

### 애너테이션이란?

- 애너테이션은 주석처럼 프로그래밍언어에 영향을 미치지 않으면서도 다른 프로그램에게 유용한 정보를 제공할 수 있다는 장점이 있음

```java
@Test
public void test() {
  ...
}
```

- `@Test`는 이 메서드를 테스트해야 한다는 것을 테스트 프로그램에게 알리는 역할만 할 뿐 메서드가 포함된 프로그램 자체에는 아무런 영향을 끼치지 않음

- jdk에서 제공하는 표준 애너테이션은 주로 컴파일러를 위한 것으로 컴파일러에게 유용한 정보를 제공함

### 표준 애너테이션

| 애너테이션           | 설명                                                    |
| -------------------- | ------------------------------------------------------- |
| @Override            | 컴팡일러에게 오버라이딩하는 메서드라는 것을 알림        |
| @Deprecated          | 앞으로 사용하지 않을 것을 권장하는 대상에 붙임          |
| @SuppressWarnings    | 컴파일러의 특정 경고 메시지가 나타나지 않게 해줌        |
| @SafeVarargs         | 제네릭스 타입의 가변 인자에 사용함 (jdk1.7)             |
| @FunctionalInterface | 함수형 인터페이스라는 것으 알림(jdk1.8)                 |
| @Native              | native메서드에서 참조되는 상수 앞에 붙임(jdk1.8)        |
| @Target*             | 애너테이션이 적용가능한 대상을 지정하는데 사용함        |
| @Documented*         | 애너테이션 정보가 javadoc으로 작성된 문서에 포함되게 함 |
| @Inherited*          | 애너테이션이 자손 클래스에 상속되도록함                 |
| @Retention*          | 애너테이션이 유지되는 범위를 지정하는데 사용함          |
| @Repeatable*         | 애너테이션을 반복해서 적용할 수 있게 함(jdk1.8)         |

- `*` 이 붙은 애너테이션들은 새로운 애너테이션을 정의하는데 사용하는 메타애너테이션

**@Override**

- 오버라이딩할때는 상위 클래스의 메서드의 이름을 잘못 적는 경우가 있는데 `@Override`를 사용하면 그런 실수를 미연에 방지할 수 있음
- Optional

**@Deprecated**

- 하위호한성때문에 이미 배포된 jdk의 함수들은 함부로 삭제될 수 없음
- 따라서 `@Deprecated` 애너테이션을 붙임으로써 이 메서드를 사용하는 클라이언트들에게 더이상 사용하지 않을 것을 권하는 의미

**@FunctionalInterface**

- 함수형 인터페이스를 선언할 때 이 애너테이션을 붙이면 컴팡일러가 함수형 인터페이스를 올바르게 선언했는지 확인하고 잘못된 경우 에러를 발생시킴
- 필수는 아니지만 실수를 방지할 수 있음
- 함수형 인터페이스는 추상 메서드가 하나뿐이어야하는 제약이 있음

**@SuppressWarnings**

- 컴파일러가 보여주는 경고 메시지가 나타나지 않게 억제해줌
- 주로사용되는 부분에는 deprecatioin, unchcked, rwatypes, varargs

> effective java item27 참고
>
> - `@SuppressWarnings` 은 가장 작은 단위로만 처리해야함
> - `@SuppressWarnings` 애너테이션을 사용할 때면 그 경고를 무시해도 안전한 이유를 항상 주석으로 남겨야 함

**@SafeVarargs**

- 어떤 타입들은 컴파일 이후 제거되는데 컴파일 후에도 제거되지 않는 타입을 reifiable타입이라고하고 제고되는 타입을 non-refiable타입이라고 함. 제네릭 타입들은 대부분 컴파일 시에 제거되기 때문에 non-refiable타입임
- 메서드에 선언된 가변인자타입이 non-reifiable타입일 경우 해당 메서드를 선언하는 부분과 호출하는 부분에서 unchcked경고가 발생함 
- 해당 코드에 문제가 없다면 경고를 억제하기 위해 `@SafeVarargs` 를 사용해야함
- 이 애너테이션은 static이나 final이 붙은 메서드와 생성자에만 붙일 수 있음. 오버라이드될 수 있는 메서드에는 사용 불가능

> - static 메서드는 오버라이드 될까?
>
> 재정의는 클래스의 인스턴스가 있는지에 따라 다름. 정적 메서드는 클래스의 인스턴스와 연결되지 않기 때문에 불가능
>
> [https://stackoverflow.com/questions/2223386/why-doesnt-java-allow-overriding-of-static-methods](https://stackoverflow.com/questions/2223386/why-doesnt-java-allow-overriding-of-static-methods)

- Arrays.asList 메서드 

```java
    @SafeVarargs
    @SuppressWarnings("varargs")
    public static <T> List<T> asList(T... a) {
        return new ArrayList<>(a);
    }
```

- 메서드가 가변인자인 동시에 제네릭타입임
- 메서드에 선언된 타입 T는 컴파일 과정에서 Object로 바뀜 Object[]
- Object[]에는 모든 타입이 들어올 수 있으므로 컴파일러가 경고를 내는것
- 그러나 실제로 asList()는 제네릭 메서드이기때문에 컴파일러가 T 타입을 체크해서 타입 T가 아닌 다른 타입이 들어올 수 있는 경우가 없음
- 이럴때 사용하는 것이 `@SafeVarargs` 
- `@SuppressWarnings("varargs")` 을 같이 붙이는 이유는 메서드 선언부외에도 메서드 호출되는 부분의 경고를 제거하기 위함임
- 따라서 일반적으로  `@SafeVarargs` 와 `@SuppressWarnings("varargs")` 을 같이 사용하는 것이 좋음



### 메타 애너테이션

- 메타에너테이션은 애너테이션에 붙이는 애너테이션
- 애너테이션을 새롭게 정의할 때 애너테이션의 적용 대상이나 유지 기간등을 지정하는 데 사용함

**@Target**

-  애너테이션이 적용 가능한 대상을 지정하는데 사용함

```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE, MODULE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    String[] value();
}

```

- java.laong.annotation.ElementType에 정의된 값들


| 대상 타입       | 의미                            |
| --------------- | ------------------------------- |
| ANNOTATION_TYPE | 애너테이션                      |
| CONSTRUCTOR     | 생성자                          |
| FIELD           | 필드(멤버변수, enum 상수)       |
| LOCAL_VARIABLE  | 지역변수                        |
| METHOD          | 메서드                          |
| PACKAGE         | 패키지                          |
| PARAMETER       | 매개변수                        |
| TYPE            | 타입(클래스, 인터페이스, enum)  |
| TYPE_PARAMETER  | 타입 매개변수(jdk1.8)           |
| TYPE_USE        | 타입이 사용되는 모든 곳(jdk1.8) |



**@Retention**

- 애너테이션이 유지되는 기간을 지정하는데 사용됨



| 유지 정책 | 의미                                                |
| --------- | --------------------------------------------------- |
| SOURCE    | 소스 파일에만 존재. 클래스파일에서는 존재하지 않음. |
| CLASS     | 클래스 파일에 존재. 실행시에 사용 불가. 기본값      |
| RUNTIME   | 클래스 파일에 존재. 실행시에 사용 가능              |

- 컴파일러가 사용하는 유지정책은 `SOURCE`. 컴파일러를 직접 작성하는 것이 아니라면 이 유지정책은 필요 없음
- 유지정책을 `RUNTIME` 으로 하면 실행시에 리플렉션을 통해 클래스파일에 저장된 애너테이션의 정보를 읽어서 처리할 수 있음
- 유지정책을 `CLASS` 로 하면 클래스 파일이 JVM에 로딩될 때 애너테이션의 정보가 무시되어 런타임에 애너테이션에 대한 정보를 얻을 수 없음 따라서 잘 사용되지 않음



**@Documented**

- 애너테이션에 대한 정보가 javadoc으로 작성한 문서에 포함되도록 함
- 자바에서 제공하는 기본 애너테이션 중에 `@Ovveride`, `@SuppressWarnings` 를 제외하고는 모두 이 메타 애너테이션이 붙어 있음



**@Inherited**

- 애너테이션이 자손 클래스에 상속되도록 함
- 이 애너테이션이 붙은 애너테이션을 조상 클래스에 붙이면 자손 클래스도 이 애너테이션이 붙은 것과 같이 인식됨



**@Repeatable**

- 보통은 하나의 대상에 한 종류의 애너테이션을 붙이지만 이 `@Repeatable` 이 붙은 애너테이션은 여러번 붙일 수 있음
- 일반적인 애너테이션과 달리 같은 이름의 애너테이션이 여러 개가 하나의 대상에 적용될 수 있기 때문에 이 애너테이션들을 하나로 묶어서 다룰 수 있는 애너테이션도 추가로 정의해야함

```java
@interface Todos { //<-- 컨테이너 애너테이션
	Todo[] value();
}

@Repeatable(Todos.class) //<-- 컨테이너 애너테이션
@interface Todo {
  String value();
}
```





**@Native**

- 네이티브 메서드에 참조되는 상수필드에 붙이는 애너테이션

```java
public final class Long extends Number implements Comparable<Long> {
    /**
     * A constant holding the minimum value a {@code long} can
     * have, -2<sup>63</sup>.
     */
    @Native public static final long MIN_VALUE = 0x8000000000000000L;

    /**
     * A constant holding the maximum value a {@code long} can
     * have, 2<sup>63</sup>-1.
     */
    @Native public static final long MAX_VALUE = 0x7fffffffffffffffL;
}
```

- 네이티브 메서드는 jvm이 설치된 OS의 메서드를 의미함
- 네이티브 메서드는 보통 c언어로 작성되어 있고 자바에서는 메서드의 선언부만 정의하고 구현하지 않음
- 자바에 정의된 네이티브 메서드와 OS의 메서드를 연결해주는 작업은 JNI(Java Native Interface)가 함

### 애너테이션 타입 정의하기

- 새로운 애너테이션을 정의하는 방법

```java
@interface 애너테이션이름 {
	타입 요소이름();
}
```

- 애너테이션 내에 선언된 메서드를 애너테이션의 요소라고함

```java
@interface TestInfo {
  int count();
  String testedBy();
  String[] testTools();
  TestType testType(); // enum TestType {FRIST, FINAL}
  DateTime testDate(); // 자신이 아닌 다른 애너테이션을 포함할 수 있음
}

@interface DateTime {
  String yymmdd();
  String hhmmss();
}
```



- 애너테이션의 요소는 반환값이 있고 매개변수는 없는 추상 메서드의 형태, 상속을 통해 구현하지 않아도됨
- 다만 애너테이션을 적용할 때 이 요소들의 값을빠짐 없이 지정해주어야함
- 애너테이션의 요소는 기본값을 가질 수 있음
- 애너테이션 요소가 오직 하나뿐이고 이름이 value인 경우 애너테이션을 적용할 때 요소의 이름을 생략하고 값만 적어도됨
- 요소 타입이 배열인 경우 괄호를 사용해서 여러 개의 값을 지정할 수 있음
- 모든 애너테이션의 조상은 java.lang.annotation.Annotation 임
- java.lang.annotation.Annotation은 상속이 불가능하고 아래와 같이 인터페이스 타입임

```java
public interface Annotation {
  boolean equals(Object obj);
  int hashCode();
  String toString();
  
  Class<? extends Annotation> annotationType(); // 애너테이션 타입 반환
}
```

- 모든 애너테이션의 조상이기때문에 java.lang.annotation.Annotation에 선언된 메서드들은 모든 애너테이션에서 사용 가능함
- 값을 정의하지 않은 경우 Maker Annotation이라고함. ex `@Test`, `@Override`
- 애너테이션의 요소를 선언할 때 반드시 지켜야 할 규칙
  - 요소 타입은 기본형, String, enum, 애너테이션, Class만 허용됨.
  - ()안에 매개변수를 선언할 수 없음
  - 예외를 선언할 수 없음
  - 요소를 타입 매개변수로 정의할 수 없음
- 클래스 객체가 가지고 있는 getAnnotations()라는 메서드로 모든 애너테이션을 배열로 받아 올 수 있음



