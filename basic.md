#### 여러 개의 개념을 한번에 묶지 말고 개념을 쪼개서 하나씩 나눠서 이해하자
# intellij 단축키
|기능|단축키|
|:---|:---|
|줄 복사|ctrl + D|
|전체 파일에서 찾기|ctrl + shift + F|
|현재 파일에서 바꾸기|ctrl + R|
|전체 파일에서 바꾸기|ctrl + shift + R|
|빌드 도구(maven, gradle) 새로고침|ctrl + shift + o|
|자동으로 테스트 클래스 생성|클래스 이름에 커서 위치 > alt + Enter > Create Test|
|`System.out.println();`|`sout` + ctrl + space|
|`System.out.printf();`|`so` + ctrl + space|
# eclipse 프로젝트를 intellij로 가져오기
1. 일단 intellij에서 eclipse 프로젝트를 불러온다.
2. File > Project structure > Modules > +버튼 클릭 > Import Module 클릭 후 import할 프로젝트 선택
3. Import module from external model > Eclipse선택 후 계속 Next 클릭 후 Finish
# QA(quality assurance)란?
= 품질 보증
품질을 보증하기 위해 수행하는 여러 테스트, 성능 진단, 검증 등의 프로세스
[참고](https://support.kmong.com/hc/ko/articles/14656446751129-QA%EC%9D%98-%EB%AA%A8%EB%93%A0-%EA%B2%83-%EC%9C%A0%ED%98%95-%ED%8A%B9%EC%A7%95-%EC%97%AD%ED%95%A0-%EC%95%8C%EC%95%84%EB%B3%B4%EA%B8%B0)

# CI/CD
Continuous Integration(지속적인 통합)
* 새로운 코드 변경 사항이 정기적으로 빌드 및 테스트되어 저장소에 통합
Continuous Delivery(지속적인 배포)
* 변경된 코드가 실제 프로덕션 환경까지 반영되는 것
# 디렉터리 계층도 작성
1. CMD창을 연다
2. 파일 트리를 만들 위치로 이동한다. (예: C:\repository\TEST_PROJECT\src\views)
3. `tree /F | clip` 명령어를 입력한다.
4. 필요한 곳에 붙여넣기

# intellij에서 github 연동하기
1. intellij에서 프로젝트 생성
2. 상단 메뉴 > Git > GitHub > Share project on github
3. 레포지토리 이름 입력하고 commit, push까지 진행
4. github에 들어가면 레포지토리 생성되있음

# java streams
[streams 총정리](https://futurecreator.github.io/2018/08/26/java-8-streams/)
# 인증(authentication)과 인가(authorization)
authentication: 사용자가 사이트에 로그인할때 누구인지 확인하는 과정
authorization: 사이트의 특정 부분에 접근할 수 있는지에 권한을 확인하는 과정
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