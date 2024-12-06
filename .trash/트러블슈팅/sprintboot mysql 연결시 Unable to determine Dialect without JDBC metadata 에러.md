---
tistoryBlogName: developing-mind
tistoryTitle: sprintboot mysql 연결시 Unable to determine Dialect without JDBC metadata 에러
tistoryTags: springboot, mysql
tistoryVisibility: "3"
tistoryCategory: "1059234"
tistorySkipModal: true
tistoryPostId: "66"
tistoryPostUrl: https://developing-mind.tistory.com/66
---
application.yml에 mysql 연결 설정 작성후 서버를 시작하니 다음과 같은 오류가 발생했습니다.
```
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'entityManagerFactory' defined in class path resource [org/springframework/boot/autoconfigure/orm/jpa/HibernateJpaConfiguration.class]: Unable to create requested service [org.hibernate.engine.jdbc.env.spi.JdbcEnvironment] due to: Unable to determine Dialect without JDBC metadata (please set 'javax.persistence.jdbc.url', 'hibernate.connection.url', or 'hibernate.dialect')
```

정확한 원인은 파악하지 못했지만 mysql의 DB 비밀번호를 초기화해주니 해결됐습니다.
```sql
alter user 'root'@'localhost' identified with mysql_native_password by '1234';
```

비밀번호를 4자리로 설정할때 여러값을 테스트해본 결과,
**첫째 자리가 0일때 나머지 자리 중 하나라도 8이상이어야 오류가 발생하지 않았습니다.**