# 자바 스프링

# 섹션 1. Spring Boot로 개발하는 RESTful Service

## 한명의 사용자는 여러게 포스팅 가능 일대다 구조
![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/565454e6-5d33-4ea9-8ec4-cd712512aefb)
![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/62c8df38-9683-4da0-8baf-1745be388125)

1. 목록 조회: users - GET
2. 새로운 사용자 등록: users - POST 
3. 상세 조회: users/{id} - GET
4. 삭제: users/{id} - DELETE
5. 사용자가 작성한 포스트 조회: users/{id}/posts - GET
6. 포스트 생성: users/{id}/posts - POST

## 스프링 세팅

![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/fae8c442-f6e4-40b7-adc6-f7b476f5550a)


## 설정

- [**application.yml((application.properties)): 개발자가 필요한 스프링 부트의 설정을 하는 곳**
    - application.yml파일에서 서버 포트번호 변경 가능
        
![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/32e6c575-5309-4629-9283-6aa6918e1036)
        
- **prom.xml: 전체 프로젝트에 필요한 메이븐 프로젝트 설정 하는 곳**

## RestController 만들기

- 클래스 만들고 어노테이션 - **@RestController** 입력
    
![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/b44933c4-025c-489e-8eeb-86a791014cba)
    
- 메서드 만들고 그 위에 @GetMapping(path = “path”) 어노테이션으로 경로 지정해줌
    
![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/3f7de25f-caf1-4301-ae6f-4e0bba51d9ba)
    
- 실행 시키면
    - 크롬

      ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/9270c6ed-3004-4905-9da0-c16e5ddbb1e7)
        
    - postman에서 테스트

      ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/10332c2e-27ab-4bc0-a3fc-630a4f078bf3)
        
    

## HelloWorld Bean 추가

- 일단 위의 메서드와 다르게 경로를 따로 지정 해주고 → String으로 선언하는 것이 아닌 곧 만들 클래스 HelloWorldBean으로 선언
    
    ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/5f06b8ed-0ef1-457e-94cc-7676d10cef30)
    
- 클래스 만들 때 이렇게 어떤 형식의 클래스를 만들 것인지 설정 가능
    
    ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/7afd451a-be91-4d48-887b-26cddb448b9a)
    
    - 그래서 맨 위에 그냥 클래스 만들어주면
        
    ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/d889fa44-9505-46a0-a8b7-7736e1187bc8)
        
- lombok 라이브러리 사용하기
    
  ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/05c8ddb3-d6ca-44c3-b2e3-46b65813acb0)
    
    - 원래 이렇게 게터 세터 만들어서 쓰는데 lombok 쓰면 다른 방법을 쓴다 함
        
        ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/92328ad6-e922-4b54-812c-748714da7e04)

        
    - @AllArgsConstructor 어노테이션 사용하면 이렇게 할 필요도 없다고 함
        
        ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/acf80c0a-450e-4315-8415-a6e1c0b6d98b)

        
    - 지워주면 오류도 안 뜸
        
        ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/bf855232-3b89-4f36-b3ce-ed8b57c59baa)
        
    - 구조는 이렇게 생김
        
        ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/ae37fa18-7276-4085-8a68-6093c4d86926)

        

## Spring Boot 동작 원리

- application.properties → 설정이름=값
    - logging.level.org.springframework = debug
- application.yml → 설정이름:값
    - logging:
       level:
         org.springframework**:** debug
- DispatcherServlet → ‘/ ’
    
    ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/99437167-acaf-451f-b257-2c0c1b112026)
    
- RestController
    
    ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/99c11385-4eea-4134-b7da-d8dd24036214)
    
    - REST Controller → @Controller + @ResponseBody

## Path Variable 정의한 API  Url에 변수 지정해서 활용하기

![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/b00dcd58-c0e4-4c74-8109-82f402fc95a0)

- 위의 메서드와 이름이 같지만 인자를 넣어줘서 오버라이딩 됨
    
   ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/d9722ad3-3a1d-4bc5-8971-b3664f168220)

    
- 크롬 웹 스토어에서 JSON Viewer 확장 프로그램 깔아주면 이쁘게 나옴
  ![image](https://github.com/jinyongkim123/KimJinyong/assets/117449640/8b0c2838-687c-4fdf-82eb-0eae452f5cbc)
