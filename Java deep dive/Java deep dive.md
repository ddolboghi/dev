---
_filters: []
_contexts: []
_links: []
_sort:
  field: rank
  asc: false
  group: false
_template: ""
_templateName: ""
Created: 2023-10-31
---
# Java 8의 3가지 프로그래밍 개념
## stream processing
- 스트림 : 한 번에 한 개씩 만들어지는 연속적인 데이터 항목들의 모임
- 스트림 패키지의 `Stream<T>`는 T형식으로 구성된 일련의 항목
- 스트림 API는 어떤 항목을 연속으로 제공하는 어떤 기능이다.
- 데이터베이스 쿼리처럼 고수준으로 추상화해서 일련의 스트림으로 만들어 처리할 수 있다.
- 스트림 파이프라인을 이용해서 입력 부분을 여러 CPU코어에 쉽게 할당할 수 있다.(병렬성)
- 스트림을 이용하면 멀티코어 CPU를 이용하는것보다 비용이 훨씬 비싼 키워드 synchronized를 사용하지 않아도 된다.

## 동작 파라미터화(behavior parameterization)
- 메서드(코드)를 다른 메서드의 인수로 넘겨줄 수 있다.
- 결과를 반환하고 다른 자료구조로 전달할 수도 있다.
- 예시) 약간만 다른 두 메서드가 있을때 인수를 이용해서 다른 동작을 하도록 하나의 메서드로 합친다.
- 스트림 API는 연산의 동작을 파라미터화할 수 있는 코드를 전달한다는 사상에 기초한다.

## 함수형 프로그래밍
- 공유되지 않은 가변 데이터, 메서드 코드를 다른 메서드로 전달하는 기능은 함수형 프로그래밍의 핵심
- 병렬성을 얻는 대신 스트림 메서드로 전달하는 코드의 동작 방식을 조금 바꿔야한다.
- 다른 코드와 동시에 실행해도 안전한 코드는 공유 가변 데이터에 접근하지 않아야한다.

# Java 8의 기능
## 메서드 참조 `::`
- `클래스::메서드`는 이 메서드를 값으로 사용하라는 의미다.
- 객체 참조처럼 메서드 참조를 할수있기때문에 메서드는 일급값이 되었다.
```java
//숨겨진 파일 필터링
File[] hiddenFiles = new File(".").listFiles(File::isHidden);
```
- isHidden이라는 함수는 준비되어있으니 `::`를 이용해서 listFiles에 직접 전달한다.

## lambda : 익명 함수
- 메서드뿐만 아니라 람다를 포함하여 함수도 일급값으로 취급할 수 있다.
- 이용할 수 있는 편리한 클래스나 메서드가 없을 때 람다 문법으로 간결하게 코드를 구현할 수 있다.
- 한두번만 사용할 메서드를 따로 정의하지 않고 람다를 사용할 수 있다.
- 람다가 복잡한 동작을 수행한다면 메서드를 따로 정의하고 메서드 참조를 활용하는 것이 좋다.
- 람다 문법 형식으로 구현된 프로그래밍을 함수형 프로그래밍(함수를 일급값으로 넘겨주는 것)이라고 한다.
## java streams
[streams 총정리](https://futurecreator.github.io/2018/08/26/java-8-streams/)

# predicate
- 인수로 값을 받아 true나 false를 반환하는 함수
```java
public interface Predicate<T> {
	boolean test(T t);
}
```

# java.util.Optional
[참고](https://mangkyu.tistory.com/70)
- Java8에서는 `Optional<T>` 클래스를 사용해 NullPointerException를 방지할 수 있음 
- `Optional<T>`는 null이 올 수 있는 값을 감싸는 Wrapper 클래스로, 참조하더라도 NPE가 발생하지 않도록 함 
- Optional 클래스는 아래와 같은 value에 값을 저장하기 때문에 값이 null이더라도 바로 NPE가 발생하지 않으며, 클래스이기 때문에 각종 메소드를 제공함
- Optional은 값을 Wrapping하고 다시 풀고, null 일 경우에는 대체하는 함수를 호출하는 등의 오버헤드가 있으므로 잘못 사용하면 시스템 성능이 저하됨. 메소드의 반환 값이 절대 null이 아니라면 Optional을 사용하지 않는 것이 좋음
- ==Optional은 메소드의 결과가 null이 될 수 있으며, null에 의해 오류가 발생할 가능성이 매우 높을 때 반환값으로만 사용되어야 한다.== 
- Optional은 파라미터로 넘어가는 등이 아니라 반환 타입으로써 제한적으로 사용되도록 설계됨
```java
List<String> names = getNames(); names.sort(); // names가 null이라면 NPE 발생

List<String> names = getNames(); // NPE를 방지하기 위해 null 검사를 해야함 
if(names != null){ 
	names.sort(); 
}
```

```java
// Java8 이전 
List<String> names = getNames(); List<String> tempNames = list != null 
	? list 
	: new ArrayList<>(); 

// Java8 이후 
List<String> nameList = Optional.ofNullable(getNames()) 
	.orElseGet(() -> new ArrayList<>());
```
- `Optional.ofNullbale()` 
	- 값이 Null일수도, 아닐수도 있는 경우
	- orElse 또는 orElseGet 메소드를 이용해서 값이 없는 경우라도 안전하게 값을 가져올 수 있음
```java
// Optional의 value는 값이 있을 수도 있고 null 일 수도 있다. 
Optional<String> optional = Optional.ofNullable(getName()); 
String name = optional.orElse("anonymous"); // 값이 없다면 "anonymous" 를 리턴
```
- `orElse()`
	- 파라미터로 값을 받음
	- 값이 미리 존재하는 경우에 사용
	- orElse는 값을 생성하여 orElseGet보다 비용이 크므로 최대한 사용을 피해야 함
- `orElseGet()`
	- 파라미터로 함수형 인터페이스(함수)를 받음
	- 값이 미리 존재하지 않는 거의 대부분의 경우에 orElseGet을 사용하면 됨
