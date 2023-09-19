# 유효성 체크를 위한 Validation API 사용

- 강의대로 @Size 어노테이션 따라서 치는데 안 되어서 직접 validation 라이브러리 적용 시켰음 강의에도 적용 시키는게 안나왔음

```xml
<!-- https://mvnrepository.com/artifact/org.springframework.boot/spring-boot-starter-validation -->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-validation</artifactId>
			<version>3.1.2</version>
		</dependency>
```

```java
	 	@Size(min = 2)   //사용자 이름에는 @Size 어노테이션으로 최솟값 2로 설정
    private String name;
    @Past     //@Past 어노테이션으로 미래 날짜 사용 불가 하고 과거 날짜 사용만 가능하게 
    private Date joinDate;
```

```java
//createUser 메서드는 PostMapping 어노테이션에 있는 uri값과 일치가 되면 실행
    /* json이나 xml같이 오브젝트 형태의
    데이터를 받기위해서는
    매개변수 타입의 어노테이션 선언 - @RequestBody*/
    @PostMapping("/users")
    public ResponseEntity<User>  createUser(@Valid @RequestBody User user) {
            User savedUser = service.save(user);

        URI location = ServletUriComponentsBuilder.fromCurrentRequest()
                .path("/{id}")
                .buildAndExpand(savedUser.getId())
                .toUri();

        return ResponseEntity.created(location).build();
    }
```

- 이렇게 설정하고 createUser메서드에 @Valid어노테이션 넣어주고나서 위에서 제약조건에 반하도록 사용자 이름을 2글자 미만으로 입력해서 추가하면 400 Bad Request가 뜸

  ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/fbcec718-3e18-4aaa-a574-a95e5adc8c17)
    
- `ResponseEntityExceptionHandler` 클래스에서  `handleMethodArgumentNotValid` 메서드를 긁어와서

```java
protected ResponseEntity<Object> handleMethodArgumentNotValid(MethodArgumentNotValidException ex, HttpHeaders headers, HttpStatusCode status, WebRequest request) {
        return this.handleExceptionInternal(ex, (Object)null, headers, status, request);
    }
```

- 추가하고 오버라이딩 해서 커스터마이징 `ustomizedResponseEntityExceptionHandler.class`

```java

    //유효성 검사 예외처리
    @Override // 오버라이드 하는 이유는 부모가 갖고 있는 정의를 재정의 후 사용하겠다는 것
    protected ResponseEntity<Object> handleMethodArgumentNotValid(MethodArgumentNotValidException ex, // 발생한 exception 객체
                                                                  HttpHeaders headers, // request 헤더값
                                                                  HttpStatusCode status, // status 값
                                                                  WebRequest request) { // 요청 request 값
        ExceptionResponse exceptionResponse = new ExceptionResponse(new Date(),
                "Validation Failed", ex.getBindingResult().toString());

        return new ResponseEntity(exceptionResponse, HttpStatus.BAD_REQUEST);
    }

```

@Size 어노테이션 인자에 message 추가하면 이렇게 나옴

```java
	@Size(min = 2, message = "Name은 2글자 이상 입력해주세요.")   //사용자 이름에는 @Size 어노테이션으로 최솟값 2로 설정
    private String name;
    @Past     //@Past 어노테이션으로 미래 날짜 사용 불가 하고 과거 날짜 사용만 가능하게 
    private Date joinDate;
```

![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/cc736cd7-20ac-48eb-af6b-3b253defcc30)

# 다국어 처리를 위한 Internationalization 구현 방법

- 하나의 출력 값을 여러가지 언어로 출력해주는 것
- 먼저 main 에서 LocaleResolver 메서드 생성

```java
package com.example.restfulwebservice;

import org.apache.tomcat.util.descriptor.LocalResolver;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.annotation.Bean;
import org.springframework.web.servlet.LocaleResolver;
import org.springframework.web.servlet.i18n.SessionLocaleResolver;

import java.util.Locale;

@SpringBootApplication
public class RestfulWebServiceApplication {

    public static void main(String[] args) {
		SpringApplication.run(RestfulWebServiceApplication.class, args);
    }

여기
    @Bean
    public LocaleResolver localeResolver(){
        SessionLocaleResolver localeResolver = new SessionLocaleResolver();
        localeResolver.setDefaultLocale(Locale.KOREA);
        return localeResolver;
    }
}
```

- resources 폴더 안에 messages.properties(한국어), messages_fr.properties(불어), messages_en.properties(영어) 생성 후 이 코드들 입력
    - 주의 할 점은 일단 설정에서 File Encoding에서 Default encoding for properties files를 UTF-8로 해주자

```java
greeting.message=hello
```

```java
greeting.message=Bonjour
```

```java
greeting.message=hello
```

- application.yml 파일 코드 추가

```yaml
server:
  port: 8088

logging:
  level:
    org.springframework: DEBUG

---여기---
spring:
  messages:
    basename: messages
```

- 그리고 HelloWorldController에서 테스트 하기 위해 코드 작성

```java
package com.example.restfulwebservice.helloworld;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.MessageSource;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestHeader;
import org.springframework.web.bind.annotation.RestController;

import java.util.Locale;

@RestController
public class HelloWorldController {

-----여기------
    @Autowired // 어노테이션을 통한 주입 현재 스프링 프레임 워크에 등록되어있는 빈들 중에서 같은 타입을 갖고있는 빈을 자동으로 주입
    private MessageSource messageSource;

    //  GET
    // /hello-world (endpoint)
    //  @RequestMapping(method=ReqestMethod.GET, path="/hello-world")보다는 @GetMapping()을 더 많이 사용함
    @GetMapping(path = "/hello-world")
    public String helloWorld(){
        return "Hello World";
    }

    //클래스 바로 만드는 단축키 : alt + enter
    @GetMapping(path = "/hello-world-bean")
    public HelloWorldBean helloWorldBean(){
        return new HelloWorldBean("Hello World");
    }

    @GetMapping(path = "/hello-world-bean/path-variable/{name}")
    public HelloWorldBean helloWorldBean(@PathVariable String name){
        return new HelloWorldBean(String.format("Hello World, %s", name));
    }

-----여기-----
    @GetMapping(path = "/hello-world-internationalized")
    public String helloWorldInternationalized(
            @RequestHeader(name="Accept-Language", required = false) Locale locale){
        return messageSource.getMessage("greeting.message", null, locale);
    }

}
```

- Postman에서 테스트 해보았을때 성공

  ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/9f943bcd-5271-492b-bd34-de9720a511e2)
    

# Response 데이터 형식 변환 - XML format

- 모든 결과값을 JSON 포맷이 아닌 XML 포맷으로 전달하는 방법으로 하기
- xml 포맷으로 전달 해달라고 했을 때 406에러가 뜸

  ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/acbcb9b0-4633-4cd8-b858-6804ff3e579a)
    
- pom.xml에서 디펜던시 추가
    
    ```xml
     		<dependency>
    			<groupId>com.fasterxml.jackson.dataformat</groupId>
    			<artifactId>jackson-dataformat-xml</artifactId>
    			<version>2.14.2</version>
    		</dependency>
    ```
    
    그러면 postman에서 이렇게 잘 뜸
    
    ```xml
    <List>
        <item>
            <id>1</id>
            <name>Kenneth</name>
            <joinDate>2023-09-19T07:28:58.921+00:00</joinDate>
        </item>
        <item>
            <id>2</id>
            <name>Alice</name>
            <joinDate>2023-09-19T07:28:58.921+00:00</joinDate>
        </item>
        <item>
            <id>3</id>
            <name>Elena</name>
            <joinDate>2023-09-19T07:28:58.921+00:00</joinDate>
        </item>
    </List>
    ```
    

# Response 데이터 제어를 위한 Filtering

- User  정보에서 매개변수 password와 ssn을 추가하고나면 이렇게 그대로 개인정보가 나오는데 filtering 처리를 해야함
    
    ```json
    [
        {
            "id": 1,
            "name": "Kenneth",
            "joinDate": "2023-09-19T07:42:08.531+00:00",
            "password": "test1",
            "ssn": "701010-1111111"
        },
        {
            "id": 2,
            "name": "Alice",
            "joinDate": "2023-09-19T07:42:08.531+00:00",
            "password": "test2",
            "ssn": "324632-2222222"
        },
        {
            "id": 3,
            "name": "Elena",
            "joinDate": "2023-09-19T07:42:08.531+00:00",
            "password": "test3",
            "ssn": "932166-3333333"
        }
    ]
    ```
    
- 특수한 문자로 바꿔주기 & NULL값으로 만들어버리기
- @JsonIgnore라는 어노테이션 사용 - 우리가 전달하고자 하는 데이터 값 제어 가능
    
    ```java
    package com.example.restfulwebservice.user;
    
    import com.fasterxml.jackson.annotation.JsonIgnore;
    import jakarta.validation.constraints.Past;
    import jakarta.validation.constraints.Size;
    import lombok.AllArgsConstructor;
    import lombok.Data;
    
    import java.util.Date;
    
    @Data
    @AllArgsConstructor
    public class User {
        private Integer id;
        @Size(min = 2, message = "Name은 2글자 이상 입력해 주세요.")
        private String name;
        @Past
        private Date joinDate;
    
        @JsonIgnore
        private String password;
        @JsonIgnore
        private String ssn;
    }
    ```
    
    결과
    
    ```json
    [
        {
            "id": 1,
            "name": "Kenneth",
            "joinDate": "2023-09-19T07:59:03.217+00:00"
        },
        {
            "id": 2,
            "name": "Alice",
            "joinDate": "2023-09-19T07:59:03.217+00:00"
        },
        {
            "id": 3,
            "name": "Elena",
            "joinDate": "2023-09-19T07:59:03.217+00:00"
        }
    ]
    ```
    
- @JsonIgnoreProperties라는 어노테이션으로 일괄적으로 처리도 가능
    
    ```java
    package com.example.restfulwebservice.user;
    
    import com.fasterxml.jackson.annotation.JsonIgnore;
    import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
    import jakarta.validation.constraints.Past;
    import jakarta.validation.constraints.Size;
    import lombok.AllArgsConstructor;
    import lombok.Data;
    
    import java.util.Date;
    
    @Data
    @AllArgsConstructor
    @JsonIgnoreProperties(value={"password"}) //일괄적으로 보여주고 싶지 않은 정보 선택하기
    public class User {
        private Integer id;
        @Size(min = 2, message = "Name은 2글자 이상 입력해 주세요.")
        private String name;
        @Past
        private Date joinDate;
    
        private String password;
        private String ssn;
    }
    ```
    
    결과 - 개별 사용자 조회를 하더라도 똑같이 적용
    
    ```json
    [
        {
            "id": 1,
            "name": "Kenneth",
            "joinDate": "2023-09-19T08:03:59.574+00:00",
            "ssn": "701010-1111111"
        },
        {
            "id": 2,
            "name": "Alice",
            "joinDate": "2023-09-19T08:03:59.574+00:00",
            "ssn": "324632-2222222"
        },
        {
            "id": 3,
            "name": "Elena",
            "joinDate": "2023-09-19T08:03:59.574+00:00",
            "ssn": "932166-3333333"
        }
    ]
    ```
    

# 프로그래밍으로 제어하는 Filtering방법 - 개별 사용자 조회

- @JsonFilter 어노테이션에 이름 지어주고
    
    ```java
    package com.example.restfulwebservice.user;
    
    import com.fasterxml.jackson.annotation.JsonFilter;
    import com.fasterxml.jackson.annotation.JsonIgnore;
    import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
    import jakarta.validation.constraints.Past;
    import jakarta.validation.constraints.Size;
    import lombok.AllArgsConstructor;
    import lombok.Data;
    
    import java.util.Date;
    
    @Data
    @AllArgsConstructor
    //@JsonIgnoreProperties(value={"password"}) //일괄적으로 보여주고 싶지 않은 정보 선택하기
    @JsonFilter("UserInfo")//이름 지어줬음
    public class User {
        private Integer id;
        @Size(min = 2, message = "Name은 2글자 이상 입력해 주세요.")
        private String name;
        @Past
        private Date joinDate;
    
        private String password;
        private String ssn;
    }
    ```
    
- UserController 클래스 복사후 이름만 AdminUserController클래스로 바꾸고, Create, Delete 메서드 없애준 다음 MappingJacksonValue 사용
    
    ```java
    package com.example.restfulwebservice.user;
    
    import com.fasterxml.jackson.databind.ser.FilterProvider;
    import com.fasterxml.jackson.databind.ser.impl.SimpleBeanPropertyFilter;
    import com.fasterxml.jackson.databind.ser.impl.SimpleFilterProvider;
    import jakarta.validation.Valid;
    import org.springframework.http.ResponseEntity;
    import org.springframework.http.converter.json.MappingJacksonValue;
    import org.springframework.web.bind.annotation.*;
    import org.springframework.web.servlet.support.ServletUriComponentsBuilder;
    
    import java.net.URI;
    import java.util.List;
    
    @RestController
    @RequestMapping("/admin")   //공통적으로 앞에 붙는 접두사에 해당하는 프리픽스를 적용하고자 한다면 이 어노테이션 사용
    public class AdminUserController {
        private UserDaoService service; //의존성 주입?
    
        public AdminUserController(UserDaoService service) {
            this.service = service;
        }
    
        @GetMapping("/users")
        public List<User> retrieveAllUsers() {
            return service.findAll();
        }
    
        // GET /users/1 or /users/10 -> String
        @GetMapping("/users/{id}")
        public MappingJacksonValue retrieveUsers(@PathVariable int id) { //매개변수에서 정보 반환을 받고 싶을때 어노테이션 : @PathVariable
            User user = service.findOne(id);
    
            if (user == null) {
                throw new UserNotFoundException(String.format("ID[%s] not found", id));
            }
    
            SimpleBeanPropertyFilter filter = SimpleBeanPropertyFilter
                    .filterOutAllExcept("id", "name", "password", "ssn");
    
            FilterProvider filters = new SimpleFilterProvider().addFilter("UserInfo", filter);
    
            MappingJacksonValue mapping = new MappingJacksonValue(user);
            mapping.setFilters(filters);
    
            return mapping;
        }
    
    }
    ```
    

# 프로그래밍으로 제어하는 Filtering방법 - 전 사용자 조회

- 위 방식이랑 비슷하게 하면 됨
    
    ```java
    package com.example.restfulwebservice.user;
    
    import com.fasterxml.jackson.databind.ser.FilterProvider;
    import com.fasterxml.jackson.databind.ser.impl.SimpleBeanPropertyFilter;
    import com.fasterxml.jackson.databind.ser.impl.SimpleFilterProvider;
    import jakarta.validation.Valid;
    import org.springframework.http.ResponseEntity;
    import org.springframework.http.converter.json.MappingJacksonValue;
    import org.springframework.web.bind.annotation.*;
    import org.springframework.web.servlet.support.ServletUriComponentsBuilder;
    
    import java.net.URI;
    import java.util.List;
    
    @RestController
    @RequestMapping("/admin")   //공통적으로 앞에 붙는 접두사에 해당하는 프리픽스를 적용하고자 한다면 이 어노테이션 사용
    public class AdminUserController {
        private UserDaoService service; //의존성 주입?
    
        public AdminUserController(UserDaoService service) {
            this.service = service;
        }
    
        @GetMapping("/users")
        public MappingJacksonValue retrieveAllUsers() {
            List<User> users = service.findAll();
    
            SimpleBeanPropertyFilter filter = SimpleBeanPropertyFilter
                    .filterOutAllExcept("id", "name", "joinDate", "password");
    
            FilterProvider filters = new SimpleFilterProvider().addFilter("UserInfo", filter);
    
            MappingJacksonValue mapping = new MappingJacksonValue(users);
            mapping.setFilters(filters);
    
            return mapping;
        }
    
        // GET /users/1 or /users/10 -> String
        @GetMapping("/users/{id}")
        public MappingJacksonValue retrieveUsers(@PathVariable int id) { //매개변수에서 정보 반환을 받고 싶을때 어노테이션 : @PathVariable
            User user = service.findOne(id);
    
            if (user == null) {
                throw new UserNotFoundException(String.format("ID[%s] not found", id));
            }
    
            SimpleBeanPropertyFilter filter = SimpleBeanPropertyFilter
                    .filterOutAllExcept("id", "name", "password", "ssn");
    
            FilterProvider filters = new SimpleFilterProvider().addFilter("UserInfo", filter);
    
            MappingJacksonValue mapping = new MappingJacksonValue(user);
            mapping.setFilters(filters);
    
            return mapping;
        }
    
    }
    ```
    

# URI를 이용한 REST API Version 관리

- 기존에 User클래스를 상속받는 UserV2클래스 생성
    
    ```java
    package com.example.restfulwebservice.user;
    
    import com.fasterxml.jackson.annotation.JsonFilter;
    import lombok.AllArgsConstructor;
    import lombok.Data;
    import lombok.NoArgsConstructor;
    
    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    @JsonFilter("UserInfoV2")//이름 지어줬음
    public class UserV2 extends User {
        private String grade;  //새롭게 grade 생성
    }
    ```
    
    이 과정에서 두 클래스에 default 생성자 만들어주는 @NoArgsConstructor 어노테이션 입력
    
- AdminUserController클래스에서 retrieveUser메서드를 복사하고 새로운 메서드 retrieveUserV2생성
    
    ```java
    package com.example.restfulwebservice.user;
    
    import com.fasterxml.jackson.databind.ser.FilterProvider;
    import com.fasterxml.jackson.databind.ser.impl.SimpleBeanPropertyFilter;
    import com.fasterxml.jackson.databind.ser.impl.SimpleFilterProvider;
    import jakarta.validation.Valid;
    import org.springframework.beans.BeanUtils;
    import org.springframework.http.ResponseEntity;
    import org.springframework.http.converter.json.MappingJacksonValue;
    import org.springframework.web.bind.annotation.*;
    import org.springframework.web.servlet.support.ServletUriComponentsBuilder;
    
    import java.net.URI;
    import java.util.List;
    
    @RestController
    @RequestMapping("/admin")   //공통적으로 앞에 붙는 접두사에 해당하는 프리패스를 적용하고자 한다면 이 어노테이션 사용
    public class AdminUserController {
        private UserDaoService service; //의존성 주입?
    
        public AdminUserController(UserDaoService service) {
            this.service = service;
        }
    
        @GetMapping("/users")
        public MappingJacksonValue retrieveAllUsers() {
            List<User> users = service.findAll();
    
            SimpleBeanPropertyFilter filter = SimpleBeanPropertyFilter
                    .filterOutAllExcept("id", "name", "joinDate", "password");
    
            FilterProvider filters = new SimpleFilterProvider().addFilter("UserInfo", filter);
    
            MappingJacksonValue mapping = new MappingJacksonValue(users);
            mapping.setFilters(filters);
    
            return mapping;
        }
    
        // GET /admin/users/1  -> /admin/v1/users/1
        @GetMapping("/v1/users/{id}")
        public MappingJacksonValue retrieveUsersV1(@PathVariable int id) { //매개변수에서 정보 반환을 받고 싶을때 어노테이션 : @PathVariable
            User user = service.findOne(id);
    
            if (user == null) {
                throw new UserNotFoundException(String.format("ID[%s] not found", id));
            }
    
            SimpleBeanPropertyFilter filter = SimpleBeanPropertyFilter
                    .filterOutAllExcept("id", "name", "password", "ssn");
    
            FilterProvider filters = new SimpleFilterProvider().addFilter("UserInfo", filter);
    
            MappingJacksonValue mapping = new MappingJacksonValue(user);
            mapping.setFilters(filters);
    
            return mapping;
        }
    
        @GetMapping("/v2/users/{id}")
        public MappingJacksonValue retrieveUsersV2(@PathVariable int id) { //매개변수에서 정보 반환을 받고 싶을때 어노테이션 : @PathVariable
            User user = service.findOne(id);
    
            if (user == null) {
                throw new UserNotFoundException(String.format("ID[%s] not found", id));
            }
    
            // User -> User2
            UserV2 userV2 = new UserV2();
            //BeanUtils : Bean들 간의 관련 작업들 도와주는 클래스
            BeanUtils.copyProperties(user, userV2); // id, name, joinDate, password, ssn
            userV2.setGrade("VIP");
    
            SimpleBeanPropertyFilter filter = SimpleBeanPropertyFilter
                    .filterOutAllExcept("id", "name", "joinDate", "grade");
    
            FilterProvider filters = new SimpleFilterProvider().addFilter("UserInfoV2", filter);
    
            MappingJacksonValue mapping = new MappingJacksonValue(userV2);
            mapping.setFilters(filters);
    
            return mapping;
        }
    }
    ```
    
- V2 API 조회했을 때 결과
    
    ```json
    {
        "id": 1,
        "name": "Kenneth",
        "joinDate": "2023-09-19T11:21:39.623+00:00",
        "grade": "VIP"
    }
    ```
    

# Request Parameter와 Header를 이용한 API Version관리

- Request Parameter 적용하기 - 위 코드에서 어노테이션 값 다르게 하기
    
    ```java
    // GET /admin/users/1  -> /admin/v1/users/1
    //    @GetMapping("/v1/users/{id}")
        @GetMapping(value = "/users/{id}/", params = "version=1")
        public MappingJacksonValue retrieveUsersV1(@PathVariable int id) { //매개변수에서 정보 반환을 받고 싶을때 어노테이션 : @PathVariable
            User user = service.findOne(id);
    
            if (user == null) {
                throw new UserNotFoundException(String.format("ID[%s] not found", id));
            }
    
            SimpleBeanPropertyFilter filter = SimpleBeanPropertyFilter
                    .filterOutAllExcept("id", "name", "password", "ssn");
    
            FilterProvider filters = new SimpleFilterProvider().addFilter("UserInfo", filter);
    
            MappingJacksonValue mapping = new MappingJacksonValue(user);
            mapping.setFilters(filters);
    
            return mapping;
        }
    
    //    @GetMapping("/v2/users/{id}")
        @GetMapping(value = "/users/{id}/", params = "version=2")
    
        public MappingJacksonValue retrieveUsersV2(@PathVariable int id) { //매개변수에서 정보 반환을 받고 싶을때 어노테이션 : @PathVariable
            User user = service.findOne(id);
    
            if (user == null) {
                throw new UserNotFoundException(String.format("ID[%s] not found", id));
            }
    
            // User -> User2
            UserV2 userV2 = new UserV2();
            //BeanUtils : Bean들 간의 관련 작업들 도와주는 클래스
            BeanUtils.copyProperties(user, userV2); // id, name, joinDate, password, ssn
            userV2.setGrade("VIP");
    
            SimpleBeanPropertyFilter filter = SimpleBeanPropertyFilter
                    .filterOutAllExcept("id", "name", "joinDate", "grade");
    
            FilterProvider filters = new SimpleFilterProvider().addFilter("UserInfoV2", filter);
    
            MappingJacksonValue mapping = new MappingJacksonValue(userV2);
            mapping.setFilters(filters);
    
            return mapping;
        }
    }
    ```

  ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/610172cc-8a81-4d92-a07f-d76e64adae2f)
    
- Header값으로 적용
    
    ```java
    // GET /admin/users/1  -> /admin/v1/users/1
    //    @GetMapping("/v1/users/{id}")
    //    @GetMapping(value = "/users/{id}/", params = "version=1")
        @GetMapping(value = "/users/{id}",headers="X-API-VERSION=1")
        public MappingJacksonValue retrieveUsersV1(@PathVariable int id) { //매개변수에서 정보 반환을 받고 싶을때 어노테이션 : @PathVariable
            User user = service.findOne(id);
    
            if (user == null) {
                throw new UserNotFoundException(String.format("ID[%s] not found", id));
            }
    
            SimpleBeanPropertyFilter filter = SimpleBeanPropertyFilter
                    .filterOutAllExcept("id", "name", "password", "ssn");
    
            FilterProvider filters = new SimpleFilterProvider().addFilter("UserInfo", filter);
    
            MappingJacksonValue mapping = new MappingJacksonValue(user);
            mapping.setFilters(filters);
    
            return mapping;
        }
    
    //    @GetMapping("/v2/users/{id}")
    //    @GetMapping(value = "/users/{id}/", params = "version=2")
    @GetMapping(value = "/users/{id}",headers="X-API-VERSION=2")
    public MappingJacksonValue retrieveUsersV2(@PathVariable int id) { //매개변수에서 정보 반환을 받고 싶을때 어노테이션 : @PathVariable
            User user = service.findOne(id);
    
            if (user == null) {
                throw new UserNotFoundException(String.format("ID[%s] not found", id));
            }
    
            // User -> User2
            UserV2 userV2 = new UserV2();
            //BeanUtils : Bean들 간의 관련 작업들 도와주는 클래스
            BeanUtils.copyProperties(user, userV2); // id, name, joinDate, password, ssn
            userV2.setGrade("VIP");
    
            SimpleBeanPropertyFilter filter = SimpleBeanPropertyFilter
                    .filterOutAllExcept("id", "name", "joinDate", "grade");
    
            FilterProvider filters = new SimpleFilterProvider().addFilter("UserInfoV2", filter);
    
            MappingJacksonValue mapping = new MappingJacksonValue(userV2);
            mapping.setFilters(filters);
    
            return mapping;
        }
    }
    ```

  ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/96c562ea-caa1-4ee1-a670-caf3de039b73)
    
- mime-type 적용
    
    ```java
    // GET /admin/users/1  -> /admin/v1/users/1
    //    @GetMapping("/v1/users/{id}")
    //    @GetMapping(value = "/users/{id}/", params = "version=1")
    //    @GetMapping(value = "/users/{id}",headers="X-API-VERSION=1")
        @GetMapping(value = "/v1/users/{id}", produces = "application/vnd.company.appv1+json")
        public MappingJacksonValue retrieveUsersV1(@PathVariable int id) { //매개변수에서 정보 반환을 받고 싶을때 어노테이션 : @PathVariable
            User user = service.findOne(id);
    
            if (user == null) {
                throw new UserNotFoundException(String.format("ID[%s] not found", id));
            }
    
            SimpleBeanPropertyFilter filter = SimpleBeanPropertyFilter
                    .filterOutAllExcept("id", "name", "password", "ssn");
    
            FilterProvider filters = new SimpleFilterProvider().addFilter("UserInfo", filter);
    
            MappingJacksonValue mapping = new MappingJacksonValue(user);
            mapping.setFilters(filters);
    
            return mapping;
        }
    
    //    @GetMapping("/v2/users/{id}")
    //    @GetMapping(value = "/users/{id}/", params = "version=2")
    //@GetMapping(value = "/users/{id}",headers="X-API-VERSION=2")
    @GetMapping(value = "/v1/users/{id}", produces = "application/vnd.company.appv2+json")
    public MappingJacksonValue retrieveUsersV2(@PathVariable int id) { //매개변수에서 정보 반환을 받고 싶을때 어노테이션 : @PathVariable
            User user = service.findOne(id);
    
            if (user == null) {
                throw new UserNotFoundException(String.format("ID[%s] not found", id));
            }
    
            // User -> User2
            UserV2 userV2 = new UserV2();
            //BeanUtils : Bean들 간의 관련 작업들 도와주는 클래스
            BeanUtils.copyProperties(user, userV2); // id, name, joinDate, password, ssn
            userV2.setGrade("VIP");
    
            SimpleBeanPropertyFilter filter = SimpleBeanPropertyFilter
                    .filterOutAllExcept("id", "name", "joinDate", "grade");
    
            FilterProvider filters = new SimpleFilterProvider().addFilter("UserInfoV2", filter);
    
            MappingJacksonValue mapping = new MappingJacksonValue(userV2);
            mapping.setFilters(filters);
    
            return mapping;
        }
    }
    ```

  ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/b3378d08-5d25-4a82-960e-4608ccd6637f)
    
- 버전 관리
    - 단순하게 사용자에게 보여주는 항목 제한 용도가 아닌 restAPI 설계가 바뀌거 어플리케이션 구조가 바뀔 때도 버전을 관리해야함
- URI versioning, Request Parameter versioning → 일반 브라우저에서 실행 가능
- Mime type versioning, headers versioning → 일반 브라우저에 실행 불가
