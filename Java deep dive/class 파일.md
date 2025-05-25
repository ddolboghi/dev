.java 파일이 Javac 컴파일러에 의해 변환된 결과물입니다.
.class 파일은 JVM이 실행할 수 있는 플랫폼 독립적인 이진 형식의 파일입니다. JVM의 실행 엔진으로 들어가는 데이터의 입구 역할을 합니다.

# 기본 데이터 타입
.class 파일은 Java Virtual Machine Specification에 의해 엄격하게 정의된 순서와 구조를 가지는 이진 스트림으로 구성되어 있습니다.
데이터를 저장하기 위해 **무부호 수**(**unsigned number**)와 **표**(**table**)라는 두 가지 기본 데이터 타입을 사용합니다.
- 무부호 수는 1, 2, 4, 8 바이트의 정수(u1, u2, u4, u8)를 나타냅니다.
- 표는 여러 무부호 수나 다른 표로 구성된 복합 데이터 타입입니다.
- 여러 개의 동일한 타입 데이터가 연속될 때는 앞에 개수 카운터가 붙어 집합(collection) 형태를 이룹니다.
# 주요 구성 요소
## 1. 매직 넘버
- .class 파일의 시작을 알리는 4바이트 값입니다.
- 이 파일이 JVM이 받아들일 수 있는 유효한 .class 파일인지 식별합니다.
- 파일 확장명 대신 매직 넘버를 사용하여 보안을 높입니다.
## 2. 버전 정보
- 매직 넘버 바로 뒤에 오는 4바이트로, 앞의 2바이트는 minor version, 뒤의 2바이트는 major version을 나타냅니다.
- Java 버전은 45부터 시작하며, JDK 메이저 버전이 올라갈 때마다 major version 번호가 1씩 증가합니다.
- 상위 JDK 버전에서 컴파일된 .class 파일은 하위 JDK 버전의 JVM에서 실행될 수 없지만, 하위 버전에서 컴파일된 .class 파일은 상위 버전에 실행될 수 있습니다.
## 3. Constant pool
- major/minor version 정보 바로 뒤에 위치하며, .class 파일에서 가장 크고 다른 구성 요소들과 연관이 많은 부분 중 하나입니다.
- literal과 symbolic references 두 가지 주요 타입의 상수를 저장합니다.
	- literal: 텍스트 문자열, final 상수 값 등 Java 언어 레벨의 상수
	- symbolic references: 클래스와 인터페이스의 전체 한정 이름(fully qualified name), 필드와 메서드의 이름 및 descriptor, 메드 핸들, 동적 호출 지점 등 컴파일 원리 측면의 개념
- 상수 풀에는 JDK 13기준, 17가지 구조가 다른 table 타입 상수가 있으며, 각 상수는 u1 타입의 태그 값으로 타입을 식별합니다.
	- CONSTANT_Class_info는 클래스나 인터페이스의 심볼릭 참조를 나타내며, 그 안의 인덱스는 CONSTANT_Utf8_info를 가리켜 전체 한정 이름 문자열을 얻습니다.
- Java는 컴파일 시 linking 단계가 없으며, .class 파일에 저장된 심볼릭 참조는 JVM이 클래스 파일을 로할때 dynamic linking을 통해 실제 메모리 주소로 변환됩니다.
- 상수 풀의 크기는 가변적이며, 시작 부분에 u2 타입의 상수 풀 용량 카운터가 있습니다. 이 카운터는 1부터 시작하며, 인덱스 0은 특별한 용도로 비워둡니다.
## 4. Access flags
- 상수 풀 바로 뒤에 위치하는 2바이트의 값으로, 해당 클래스 또는 인터페이스의 접근 권한 및 속성 정보를 나타냅니다.
- public, abstract, final, interface, synthetic, annotation, enum, module 등의 속성을 나타내는 여러 비트 플래그의 조합으로 구성됩니다.
## 5. Class Index, Superclass Index, Interfaces Index Collection
- access flag 뒤에 순서대로 배치됩니다.
- 클래스 인덱스(this_class): 2바이트 값으로, 현재 클래스의 전체 한정 이름을 나타내는 상수 풀의 CONSTANT_Class_info 타입을 가리킵니다.
- 상위 클래스 인덱스(super_class): 2바이트 값으로, 상위 클래스의 fully qualified name을 나타내는 상수 풀의 CONSTANT_Class_info 타입을 가리킵니다. java.lang.Object를 제외한 모든 클래스는 유효한 상위 클래스 인덱스를 가집니다.
- 인터페이스 인덱스 컬렉션(interfaces): 구현된 인터페이스들의 심볼릭 참조를 나타내는 u2 타입 인덱스들의 집합입니다. 시작 부분에 u2 타입의 개수 카운터가 있으며, 이 값이 0이면 인터페이스가 없음을 의미합니다.

## 6. Fields Table Collection
- 인터페이스 인덱스 컬렉션 뒤에 위치합니다.
- 클래스에 정의된 필드들의 정보를 저장하는 테이블들의 집합입니다.
- 시작 부분에 u2 타입의 개수 카운터가 있습니다.
- 각 필드 정보 테이블은 엑세스 플래그(public, private, protected, static, final, volatile, transient 등), 이름 인덱스(필드 이름), 디스크립터 인덱스(필드 타입), 그리고 속성 테이블 컬렉션(예: ConstantValue)으로 구성됩니다.
## 7. Methods Table Collection
- 필드 테이블 컬렉션 뒤에 위치합니다.
- 클래스에 정의된 메서드들의 정보를 저장하는 테이블들의 집합입니다.
- 시작 부분에 u2 타입의 개수 카운터가 있습니다.
- 각 메서드 정보 테이블은 엑세스 플래그(public, private, protected, static, final, synchronized, native, abstract 등), 이름 인덱스(메서드 이름), 디스크립터 인덱스(메서드 파라미터 타입과 반환 타입), 그리고 속성 테이블 컬렉션으로 구성됩니다.
- 실제 메서드 코드(바이트코드 명령어)는 메서드 속성 테이블 중 "Code" 속성 내에 저장됩니다.
- 소스 코드에 정의된 메서드 외에도 컴파일러가 추가하는 인스턴스 생성자 `<init>()`나 클래스 생성자 `<clinit>()` 같은 메서드 정보가 포함될 수 있습니다.
## 8. Attributes Table Collection
- .class 파일 자체, 필드 테이블, 메서드 테이블 각각이 가질 수 있는 추가적인 정보의 집합입니다.
- 이 부분은 확장성이 뛰어나며, 표준에 정의된 속성 외에 사용자 정의 속성도 포함될 수 있습니다. 단, JVM은 인식하지 못하는 속성은 무시합니다.
- 중요한 속성으로는 Code (메서드 바이트코드), ConstantValue (상수 필드 값), SourceFile (소스 파일 이름), InnerClasses (내부 클래스 정보), StackMapTable (바이트코드 검증 정보), MethodParameters (메서드 파라미터 이름) 등이 있습니다.

.class 파일의 이러한 구조는 플랫폼 중립성, 간결성, 안정성 및 확장성을 특징으로 하며, 이는 Java 기술의 플랫폼 및 언어 독립성을 가능하게 하는 중요한 기반입니다.
.class 파일의 내용이 로드되면 JVM의 메서드 영역 등 런타임 데이터 영역에 적재되어 실행됩니다.