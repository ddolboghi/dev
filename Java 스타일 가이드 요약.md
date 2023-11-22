## Formatting

### enum

- enum 상수의 각 `,` 다음의 개행 허용, 추가로 하나까지 더 허용

```java
private enum Answer {
  YES {
    @Override public String toString() {
      return "yes";
    }
  },

  NO,
  MAYBE
}
```

```java
private enum Suit { CLUBS, HEARTS, SPADES, DIAMONDS }
```

### 변수

- 매 초기화 마다 하나의 변수만 초기화한다. (`int a, b;` 금지) (for문 헤더 제외)
- 지역변수는 굳이 블럭이 시작될때 선언할 필요는 없다. 처음 사용되는 위치와 가깝게 선언한다.
- `String args[]` 대신에 `String[] args` 사용.

### 배열

"block like construct" 처럼 포매팅 가능하다. 모두 허용

```java
new int[] {           new int[] {
  0, 1, 2, 3            0,
}                       1,
                        2,
new int[] {             3,
  0, 1,               }
  2, 3
}                     new int[]
                          {0, 1, 2, 3}
```

### 애노테이션

- 애노테이션은 클래스, 함수, 생성자 바로 위에 작성하지만, 한줄에 써도 된다.

```java
@Override
@Nullable
public String getNameIfPresent() { ... }

@Override public int hashCode() { ... }
```

- @Override: 생략하지 않는다.  
    (부모 메서드가 @Deprecated 인 경우 생략 가능)  
      
      
      
    

## 네이밍

- 모든 식별자는 ASCII 숫자와 문자와 `_`를 쓴다. 고로 정규식 `\w+` 와 매칭된다.

### 패키지

- 전부 소문자로 연속된 단어로 이룬다. (com.example.deepspace)

### 클래스

- **클래스 이름은 UpperCamelCase로 작성 (LottoNumber)**
- **클래스 이름은 명사 또는 명사구 (Character, ImmutableList)**
- 인터페이스 이름은 추가로 형용사 또는 형용사 구 (Readable, Testable)
- 테스트 중인 클래스 이름은 마지막에 Test 붙여주기. (HashTest)

### 메소드 (함수)

- **메소드 이름은 lowerCamelCase로 작성 (validateInput)**
- **메소드 이름은 동사 또는 동사구 (sendMessage, stop)**

### Constant (상수)

- **상수 이름은 CONSTANT_CASE를 사용 (MAX_INDEX)**
- **상수 이름은 일반적으로 명사 또는 명사구**
- final 이라도 상태가 변경될수 있는 인스턴스라면 상수가 아니다.

### 상수가 아닌 필드 변수, 매개변수, 지역변수 이름

- **상수가 아닌 필드 이름은 lowerCamelCase로 작성한다.**
- **일반적으로 명사 또는 명사구 (addedValues, index)**
- final, 불변인 경우에도 지역 변수는 상수로 간주되지 않는다.

### Type 변수 이름

- 각 유형 변수의 이름은 단일 대문자 혹은 단일 숫자가 따라올 수 있다. (E, T, X, T2)
- 클래스에 사용되는 형식의 이름 뒤에 대문자 T를 붙힌다. (RequestT, FooT)

### 정의된 단어들에 이름을 붙힐 때 (IPv6, ASCII, iOS)

- 단어로 나누고 UpperCamelCase 또는 lowerCamelCase 적용
- (XML HTTP request -> XmlHttpRequest)
- (new customer ID -> newCustomerId)
- (supports IPv6 on iOS? -> supportsIpv6OnIos)

### Javadoc 형식

```java
/**
 * Multiple lines of Javadoc text are written here,
 * wrapped normally...
 */
public int method(String p1) { ... }
```

```java
/** 짧은 Javadoc 입니다. */
```

### Javadoc 을 어디에 써야하는가

- 모든 public class에 쓰이고, 그 클래스의 public, protected 멤버에 쓰인다. (너무 간단한 경우 제외)