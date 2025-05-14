# Todo Management API

## 🔧 프로젝트 개요
간단한 TODO 관리 시스템으로, Spring Boot 기반 REST API와 JWT 인증을 포함한 백엔드 구현.

---

## ⚙️ 개발 환경

- **언어**: Java 17
- **프레임워크**: Spring Boot 3.3.4
- **데이터베이스**: SQLite3
- **ORM**: MyBatis
- **인증**: JWT
- **빌드 도구**: Gradle
- **테스트**: JUnit5 + Mockito
- **로깅**: Logback + SLF4J

---

## 1. 테스트 코드

### 1.1 TodoControllerTest
- **실행 방법** : `.\gradlew clean test --tests "com.todomanage.controller.TodoControllerTest" --info`

`TodoControllerTest` 클래스는 Todo API의 주요 기능이 정상적으로 동작하는지 검증하는 통합 테스트입니다.  
전체 생명주기(Lifecycle)를 따라 아래와 같은 흐름으로 테스트가 수행됩니다:

#### ✔️ 테스트 순서

1. **회원가입 및 로그인**  
    - 테스트용 사용자를 동적으로 생성하고 회원가입 요청을 보냅니다.  
    - 로그인 후 JWT 토큰을 발급받아 이후 요청에 사용합니다.
    - 사용된 API:
        - `POST /users/signup` (회원가입)
        - `POST /users/login` (로그인)

2. **Todo 생성**  
    - 새로운 Todo 항목을 3개 생성하고, 생성된 항목의 ID를 저장합니다.
        - Test Todo1 : 이후 단일 조회, 목록 조회, 삭제 TEST
        - Test Todo2 : 이후 목록 조회, 수정 TEST
        - Test Todo3 : 이후 목록 조회 TEST
    - 검증: 응답 코드 `201 Created`, JSON 필드 값 확인
    - 사용된 API:
        - `POST /todos` (Todo 생성)

3. **Todo 목록 조회**  
    - 생성한 Todo 항목들이 목록에 포함되어 있는지 확인합니다.  
    - 검증: 응답 코드 `200 OK`, JSON 배열 내에 생성된 Todo ID 포함 여부
    - 사용된 API:
        - `GET /todos` (Todo 목록 조회)

4. **단일 Todo1 조회**  
    - 특정 ID로 단일 Todo 항목을 조회합니다.  
    - 검증: 응답 코드 `200 OK`, JSON 응답의 title, description 등 필드 값 확인
    - 사용된 API:
        - `GET /todos/{id}` (단일 Todo 조회)

5. **Todo2 수정**  
    - 생성된 Todo2 항목의 내용을 수정합니다.  
    - 검증: 응답 코드 `200 OK`, 수정된 필드(title, completed 등) 값이 반영되었는지 확인
    - 사용된 API:
        - `PUT /todos/{id}` (Todo 수정)

6. **Todo1 삭제**  
    - 생성된 Todo1 항목을 삭제 요청합니다.  
    - 검증: 응답 코드 `204 No Content`
    - 사용된 API:
        - `DELETE /todos/{id}` (Todo 삭제)

7. **Todo1 삭제 후 조회 확인**  
    - 삭제한 Todo를 다시 조회할 경우 `404 Not Found`가 반환되는지 확인합니다.  
    - 검증: 응답 코드 `404 Not Found`
    - 사용된 API:
        - `GET /todos/{id}` (삭제된 Todo 조회)

---

### 1.2 UserControllerTest
- **실행 방법**: `.\gradlew clean test --tests "com.todomanage.controller.UserControllerTest" --info`

`UserControllerTest` 클래스는 **회원가입** 및 **로그인** 흐름을 테스트하는 통합 테스트입니다.  
Spring Boot의 `@SpringBootTest`와 `MockMvc`를 사용하여 실제 API 호출 흐름을 시뮬레이션합니다.

#### ✔️ 테스트 순서

1. **회원가입 (Sign Up)**  
    - 새로운 `UserDto` 객체를 생성하여 회원가입 요청을 보냅니다.  
    - 검증: 응답 코드 `201 Created`, 정상적으로 사용자 생성 확인
    - 사용된 API:
        - `POST /users/signup` (회원가입)

2. **로그인 (Login)**  
    - 회원가입한 사용자의 정보를 사용하여 로그인 요청을 보냅니다.  
    - 검증: 응답 코드 `200 OK`, 응답 내용이 **JWT 패턴**과 일치하는지 확인
    - 사용된 API:
        - `POST /users/login` (로그인)

---

### 1.3 TodoSecurityTest
- **실행 방법**: `.\gradlew clean test --tests "com.todomanage.controller.TodoSecurityTest" --info`

`TodoSecurityTest` 클래스는 **Todo API**의 **보안 관련** 테스트를 다룹니다.  
이 테스트는 인증되지 않은 요청을 처리하고, JWT 토큰을 이용해 인증을 요구하는 보안 관련 API 동작을 검증합니다.

#### ✔️ 테스트 순서

1. **잘못된 Todo 조회 (Invalid Todo ID)**  
    - 로그인 후, 존재하지 않는 Todo ID로 조회 요청을 보냅니다.  
    - 검증: `404 Not Found` 상태 코드 확인
    - 사용된 API:
        - `GET /todos/{id}` (잘못된 Todo ID 조회)

2. **JWT 없이 Todo 목록 조회**  
    - JWT 토큰 없이 Todo 목록을 조회하려고 시도합니다.  
    - 검증: `401 Unauthorized` 상태 코드 확인
    - 사용된 API:
        - `GET /todos` (Todo 목록 조회)

---

## 2. 사용자 관련 API (User)

### 2.1 회원가입 (Sign Up)
- **URL**: `/api/users/signup`
- **Method**: `POST`
- **Request Body**:
    ```json
    {
        "userId": "string",
        "userName": "string",
        "password": "string",
        "email": "string"
    }
    ```
- **Response**:
    - **201 Created**: 사용자 생성 성공
    - **400 Bad Request**: 필수 정보가 부족할 경우
    - **409 Conflict**: 이미 존재하는 사용자 ID
    - **500 Internal Server Error**: 서버 오류

---

### 2.2 로그인 (Login)
- **URL**: `/api/users/login`
- **Method**: `POST`
- **Request Body**:
    ```json
    {
        "userId": "string",
        "password": "string"
    }
    ```
- **Response**:
    - **200 OK**: 로그인 성공, JWT 토큰 반환
    - **400 Bad Request**: 로그인 정보 부족
    - **401 Unauthorized**: 인증 실패

---

### 2.3 내 정보 조회 (Get Current User)
- **URL**: `/api/users/me`
- **Method**: `GET`
- **Headers**:
    - `Authorization`: Bearer `<JWT Token>`
- **Response**:
    - **200 OK**: 사용자 정보 반환
    - **401 Unauthorized**: 인증 실패

---

### 2.4 내 정보 수정 (Update User)
- **URL**: `/api/users/me`
- **Method**: `PUT`
- **Headers**:
    - `Authorization`: Bearer `<JWT Token>`
- **Request Body**:
    ```json
    {
        "userName": "string",
        "password": "string",
        "email": "string"
    }
    ```
- **Response**:
    - **200 OK**: 수정된 사용자 정보 반환
    - **401 Unauthorized**: 인증 실패

---

### 2.5 사용자 삭제 (Delete User)
- **URL**: `/api/users/me`
- **Method**: `DELETE`
- **Headers**:
    - `Authorization`: Bearer `<JWT Token>`
- **Response**:
    - **204 No Content**: 삭제 성공
    - **401 Unauthorized**: 인증 실패

---

## 3. Todo 관련 API (Todo)

### 3.1 Todo 목록 조회 (Get All Todos)
- **URL**: `/api/todos`
- **Method**: `GET`
- **Headers**:
    - `Authorization`: Bearer `<JWT Token>`
- **Response**:
    - **200 OK**: Todo 목록 반환
    - **401 Unauthorized**: 인증 실패

---

### 3.2 Todo 생성 (Create Todo)
- **URL**: `/api/todos`
- **Method**: `POST`
- **Headers**:
    - `Authorization`: Bearer `<JWT Token>`
- **Request Body**:
    ```json
    {
        "title": "string",
        "description": "string"
    }
    ```
- **Response**:
    - **201 Created**: 생성된 Todo 반환
    - **401 Unauthorized**: 인증 실패

---

### 3.3 Todo 수정 (Update Todo)
- **URL**: `/api/todos/{id}`
- **Method**: `PUT`
- **Path Parameters**:
    - `id`: 수정할 Todo의 ID
- **Headers**:
    - `Authorization`: Bearer `<JWT Token>`
- **Request Body**:
    ```json
    {
        "title": "string",
        "description": "string"
    }
    ```
- **Response**:
    - **200 OK**: 수정된 Todo 반환
    - **404 Not Found**: Todo가 존재하지 않는 경우

---

### 3.4 Todo 삭제 (Delete Todo)
- **URL**: `/api/todos/{id}`
- **Method**: `DELETE`
- **Path Parameters**:
    - `id`: 삭제할 Todo의 ID
- **Headers**:
    - `Authorization`: Bearer `<JWT Token>`
- **Response**:
    - **204 No Content**: 삭제 성공
    - **404 Not Found**: Todo가 존재하지 않는 경우

---

### 3.5 단일 Todo 조회 (Get Todo by ID)
- **URL**: `/api/todos/{id}`
- **Method**: `GET`
- **Path Parameters**:
    - `id`: 조회할 Todo의 ID
- **Headers**:
    - `Authorization`: Bearer `<JWT Token>`
- **Response**:
    - **200 OK**: 조회된 Todo 반환
    - **404 Not Found**: Todo가 존재하지 않는 경우

---

### 3.6 Todo 검색 (Search Todos)
- **URL**: `/api/todos/search`
- **Method**: `GET`
- **Query Parameters**:
    - `query`: 검색할 제목 또는 설명 내용
- **Headers**:
    - `Authorization`: Bearer `<JWT Token>`
- **Response**:
    - **200 OK**: 검색된 Todo 목록 반환
    - **401 Unauthorized**: 인증 실패

---

## 4. 인증 및 권한

모든 API는 `Authorization` 헤더에 **JWT 토큰**을 포함하여 요청을 보내야 합니다.

- **Authorization Header Format**: `Bearer <JWT Token>`
- **토큰 유효 기간**: 1시간

---

## 5. SQLite 초기화 코드

- **코드 위치** : /todoManage/src/main/resources/schema.sql

---
