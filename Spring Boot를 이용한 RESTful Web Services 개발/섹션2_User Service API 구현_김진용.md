# User 도메인 클래스 생성

- 도메인이란?
    - 인터넷을 사용할 때 각 회사들이 가지고 있는 도메인 주소가 아닌, 특정한 전문 분야에서 사용 되어지는 전문 지식
- User클래스 생성 후 lombok 임포트 시키고 User 도메인 생성
    
    ```java
    package com.example.restfulwebservice.user;
    
    import lombok.AllArgsConstructor;
    import lombok.Data;
    
    import java.util.Date;
    
    @Data
    @AllArgsConstructor
    public class User {
        private Integer id;
        private String name;
        private Date joinDate;
    }
    ```
    
- 도메인 정보를 이용한 사용자 정보 조회 (비즈니스 로직) 추가
    
    ```java
    package com.example.restfulwebservice.user;
    
    import java.util.ArrayList;
    import java.util.Date;
    import java.util.List;
    
    public class UserDaoService {
        private static List<User> users = new ArrayList<>();
    
        private static int usersCount = 3;
    
        static {
            users.add(new User(1, "Kenneth", new Date()));
            users.add(new User(2, "Alice", new Date()));
            users.add(new User(3, "Elena", new Date()));
        }
    
        public List<User> findAll(){
            return users;
        }
    
        public User save(User user){
            if(user.getId() == null){
                user.setId(++usersCount);
            }
    
            users.add(user);
            return user;
        }
    
        public User findOne(int id){
            for (User user : users){
                if(user.getId()== id){
                    return user;
                }
            }
            return null;
        }
    }
    ```
    

# 사용자 목록 조회를 위한 API 구현 - GET HTTP Method

- @Service
    
    ```java
    package com.example.restfulwebservice.user;
    
    import org.springframework.stereotype.Service;
    
    import java.util.ArrayList;
    import java.util.Date;
    import java.util.List;
    
    //용도에 맞게 어노테이션 지정
    @Service
    public class UserDaoService {
        private static List<User> users = new ArrayList<>();
    
        private static int usersCount = 3;
    
        static {
            users.add(new User(1, "Kenneth", new Date()));
            users.add(new User(2, "Alice", new Date()));
            users.add(new User(3, "Elena", new Date()));
        }
    
        public List<User> findAll(){
            return users;
        }
    
        public User save(User user){
            if(user.getId() == null){
                user.setId(++usersCount);
            }
    
            users.add(user);
            return user;
        }
    
        public User findOne(int id){
            for (User user : users){
                if(user.getId()== id){
                    return user;
                }
            }
            return null;
        }
    }
    ```
    
- UserController 클래스 생성후 API 구현하기
    
    ```java
    package com.example.restfulwebservice.user;
    
    import org.springframework.web.bind.annotation.GetMapping;
    import org.springframework.web.bind.annotation.PathVariable;
    import org.springframework.web.bind.annotation.RestController;
    
    import java.util.List;
    
    @RestController
    public class UserController {
        private UserDaoService service; //의존성 주입?
    
        public UserController(UserDaoService service){
            this.service = service;
        }
    
        @GetMapping("/users")
        public List<User> retrieveAllUsers(){
            return service.findAll();
        }
    
        // GET /users/1 or /users/10 -> String
        @GetMapping("/users/{id}")
        public User retrieveUsers(@PathVariable int id){ //매개변수에서 정보 반환을 받고 싶을때 어노테이션 : @PathVariable
            return service.findOne(id);
        }
    
    }
    ```
    

# 사용자 등록을 위한 API구현 - **POST** HTTP Method

- 사용자 등록할 때 정보 입력을 위한 POST 맵핑

```java
package com.example.restfulwebservice.user;

import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
public class UserController {
    private UserDaoService service; //의존성 주입?

    public UserController(UserDaoService service){
        this.service = service;
    }

    @GetMapping("/users")
    public List<User> retrieveAllUsers(){
        return service.findAll();
    }

    // GET /users/1 or /users/10 -> String
    @GetMapping("/users/{id}")
    public User retrieveUsers(@PathVariable int id){ //매개변수에서 정보 반환을 받고 싶을때 어노테이션 : @PathVariable
        return service.findOne(id);
    }

여기 추가
    //createUser 메서드는 PostMapping 어노테이션에 있는 uri값과 일치가 되면 실행
    @PostMapping("/users")
    public void createUser(@RequestBody User user) {/* json이나 xml같이 오브젝트 형태의 데이터를 받기위해서는
                                                      매개변수 타입의 어노테이션 선언*/
            User savedUser = service.save(user);
    }

}
```

확인해보면(JSON 코드는 raw 선택 → JSON)

![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/e5f4c7dc-789e-465b-854b-30b538acbf30)

진짜 됨

![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/9d90de3f-d92c-474b-b16e-7f88d28c4687)

# HTTP Status Code 제어

- 사용자 생성과 같이 추가가 되었을 때 HTTP 상태 코드를 제어 하는건데
    
    ```java
    //createUser 메서드는 PostMapping 어노테이션에 있는 uri값과 일치가 되면 실행
        /* json이나 xml같이 오브젝트 형태의
        데이터를 받기위해서는
        매개변수 타입의 어노테이션 선언 - @RequestBody*/
        @PostMapping("/users")
        public ResponseEntity<User>  createUser(@RequestBody User user) {
                User savedUser = service.save(user);
    
            URI location = ServletUriComponentsBuilder.fromCurrentRequest()
                    .path("/{id}")
                    .buildAndExpand(savedUser.getId())
                    .toUri();
    
            return ResponseEntity.created(location).build();
        }
    ```
    
    - 안 좋은 API 설정
        - 클라이언트의 용도에 맞춰서 GET, POST, PUT, DELETE 메서드로 구분하지 않고, 모든 코드를 POST메서드로 통일해서 200번(OK)코드로 처리하는 경우 → 예외 핸들링으로 조합

# HTTP Status Code 제어를 위한 Exception Handling

- 리스트에 없는 사용자 검색 했을 때 예외 처리 하기
    - 이렇게 수정 해주고
        
        ```java
        // GET /users/1 or /users/10 -> String
            @GetMapping("/users/{id}")
            public User retrieveUsers(@PathVariable int id){ //매개변수에서 정보 반환을 받고 싶을때 어노테이션 : @PathVariable
                User user = service.findOne(id);
        
                if (user == null){
                    throw new UserNotFoundException(String.format("ID[%s] not found", id));
                }
        
                return user;
            }
        ```
        
    - 예외 클래스 만들기
        
        ```java
        package com.example.restfulwebservice.user;
        
        public class UserNotFoundException extends RuntimeException {
            public UserNotFoundException(String message) {
                super(message);
            }
        }
        ```
        ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/0e90942a-2e96-48f6-b50d-5b49fecff9a3)
        
    - 근데 이 오류는 사용자 측에서 잘못했기 때문에 4XX번대 오류 코드가 나와야 함
    
    ```java
    package com.example.restfulwebservice.user;
    
    import org.springframework.http.HttpStatus;
    import org.springframework.web.bind.annotation.ResponseStatus;
    
    //HTTP Status code
    //2XX -> OK
    //4XX -> Client 측 오류
    //5XX -> Server 측 오류
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public class UserNotFoundException extends RuntimeException {
        public UserNotFoundException(String message) {
            super(message);
        }
    }
    ```
    ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/33c2d63f-ee17-43e4-81f6-1d212512ffbd)
    

# Spring의 AOP를 이용한 Exception Handling

- 시간, 메세지, 제공하고자 하는 정보를 나타낸 예외처리 그리고 일반화 / 없는 사용자일 경우 예외처리
    
    ```java
    package com.example.restfulwebservice.exception;
    
    import com.example.restfulwebservice.user.UserNotFoundException;
    import org.springframework.http.HttpStatus;
    import org.springframework.http.ResponseEntity;
    import org.springframework.web.bind.annotation.ControllerAdvice;
    import org.springframework.web.bind.annotation.ExceptionHandler;
    import org.springframework.web.bind.annotation.RestController;
    import org.springframework.web.context.request.WebRequest;
    import org.springframework.web.servlet.mvc.method.annotation.ResponseEntityExceptionHandler;
    
    import java.util.Date;
    
    @RestController
    @ControllerAdvice // 모든 컨트롤러가 실행이 될 때 이 어노테이션이 사전에 실행 됨
    public class CustomizedResponseEntityExceptionHandler extends ResponseEntityExceptionHandler {
    
        //일반화 한 예외처리
        @ExceptionHandler(Exception.class)
        public final ResponseEntity<Object> handleAllExceptions(Exception ex, WebRequest request){
            ExceptionResponse exceptionResponse =
                    new ExceptionResponse(new Date(), ex.getMessage(), request.getDescription(false));
    
            return new ResponseEntity(exceptionResponse, HttpStatus.INTERNAL_SERVER_ERROR);
        }
    
        //없는 사용자일 경우 예외처리
        @ExceptionHandler(UserNotFoundException.class)
        public final ResponseEntity<Object> handleUserNotFoundExceptions(Exception ex, WebRequest request){
            ExceptionResponse exceptionResponse =
                    new ExceptionResponse(new Date(), ex.getMessage(), request.getDescription(false));
    
            return new ResponseEntity(exceptionResponse, HttpStatus.NOT_FOUND);
        }
    }
    ```
    

# 사용자 삭제를 위한 API 구현 - DELETE HTTP Method

- UserController에 deleteUser 메서드 추가

```java
@DeleteMapping("/users/{id}")
    public void deleteUser(@PathVariable int id){
        User user = service.deleteById(id);

        if( user == null){
            throw new UserNotFoundException(String.format("ID[%s] not found", id));
        }
    }
```

1번 아이디 사용자 삭제

![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/5f90cf77-1d55-4998-955a-39b80638644f)

200OK 코드 뜨고 삭제 성공

![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/cc099923-2d2d-407e-9cd3-054e549ce144)
