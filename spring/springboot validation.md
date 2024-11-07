# Form
- HTTP요청을 통해 전달 받은 값 검증하는 클래스
- spring boot validation 라이브러리 사용
# spring boot validation
의존성 추가
```gradle
implementation 'org.springframework.boot:spring-boot-starter-validation'
```

|항목|설명|
|---|---|
|@Size|문자 길이를 제한한다.|
|@NotNull|Null을 허용하지 않는다.|
|@NotEmpty|Null 또는 빈 문자열(`""`)을 허용하지 않는다.|
|@Past|과거 날짜만 가능|
|@Future|미래 날짜만 가능|
|@FutureOrPresent|미래 또는 오늘날짜만 가능|
|@Max|최대값|
|@Min|최소값|
|@Pattern|정규식으로 검증|
[more](https://beanvalidation.org/](https://beanvalidation.org/)

사용 예시
```java
package com.mysite.sbb.question;

import jakarta.validation.constraints.NotEmpty;
import jakarta.validation.constraints.Size;

import lombok.Getter;
import lombok.Setter;

@Getter
@Setter
public class QuestionForm {
    @NotEmpty(message="제목은 필수항목입니다.")
    @Size(max=200)
    private String subject;

    @NotEmpty(message="내용은 필수항목입니다.")
    private String content;
}
```

```java
//controller
@PostMapping("/create")  
public String questionCreate(@Valid QuestionForm questionForm, BindingResult bindingResult) {  
    if (bindingResult.hasErrors()) {  
        return "question_form";  
    }  
    this.questionService.create(questionForm.getSubject(), questionForm.getContent());  
    return "redirect:/question/list";  
}
```
- 스프링의 바인딩 기능: Form클래스에 있는 속성을 가진 폼이 전송되면 Form객체의 속성이 자동으로 바인딩됨
- Form 매개변수 앞에 @Valid를 적용해야 Form클래스에 설정한 검증 기능 동작함
- BindingResult 매개변수: 검증 결과를 가진 객체. ==항상 @Valid 매개변수 바로 뒤에 위치해야함. 안그러면 검증 실패시 400오류==