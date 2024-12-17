# Spring Boot 학습 내용 정리

## 1. Spring Boot와 다른 웹 프레임워크 비교

### 1.1 프로젝트 구조의 유사성

기존에 Django, Node.js 기반 웹 프레임워크를 사용해본 경험을 토대로 Spring Boot의 구조를 이해해보았다. 
특히 찾아보며 공부하니 다른 웹프레임워크와 유사한 점이 많아 학습에 도움이 되었다.

프레임워크별 구조 비교

- Django: MTV(Model-Template-View) 패턴
  - Model: 데이터 구조와 비즈니스 로직 (Spring Boot의 Model과 유사)
  - Template: 화면 표시 로직 (Spring Boot의 View와 대응)
  - View: HTTP 요청 처리와 응답 제어 (Spring Boot의 Controller와 대응)
- Spring Boot: MVC(Model-View-Controller) 패턴
- Node.js: 자유로운 구조 (레이어드 아키텍처 또는 MVC 패턴과 같이 다양한 패턴 적용 가능)

각 프레임워크의 디렉토리 역할 비교

```
[Spring Boot]
src/main/java/
    ├── controller/    # HTTP 요청 처리
    ├── service/       # 비즈니스 로직
    ├── repository/    # 데이터베이스 접근
    ├── entity/        # JPA 엔티티
    └── domain/        # 도메인 모델

[Django]
myapp/
    ├── views.py       # HTTP 요청 처리
    ├── models.py      # 데이터 모델 (ORM)
    ├── forms.py       # 폼 처리
    └── urls.py        # 라우팅

[Node.js (Express)]
src/
    ├── routes/        # 라우팅
    ├── controllers/   # 요청 처리
    ├── models/        # 데이터 모델
    └── services/      # 비즈니스 로직
```

### 1.2 프레임워크별 특징 비교

1. 환경 설정
    - Spring Boot: application.yml/properties
    - Django: settings.py
    - Node.js: package.json, .env
2. ORM
    - Spring Boot: JPA
    - Django: 내장 ORM
    - Node.js: Prisma, Sequelize 등 선택적 사용
3. 라우팅
    - Spring Boot: 어노테이션 기반(@RequestMapping)
    - Django: urls.py에서 정의
    - Node.js: express.Router() 등으로 정의

## 2. Spring Boot만의 특징

### 2.1 기본 프로젝트 구조 (Gradle)

Spring Initializr로 새 프로젝트를 생성하면 다음과 같은 구조가 생성됨

```
project-root/
├── gradle/
│   └── wrapper/                  # Gradle Wrapper 파일
│       ├── gradle-wrapper.jar
│       └── gradle-wrapper.properties
│
├── src/
│   ├── main/
│   │   ├── java/                # Java 소스 코드
│   │   │   └── com/example/demo/
│   │   │       └── DemoApplication.java # 진입점
│   │   └── resources/           # 리소스 파일
│   │       ├── static/          # 정적 파일 (CSS, JS, 이미지)
│   │       ├── templates/       # 템플릿 파일
│   │       └── application.yml  # 애플리케이션 설정 파일
│   │
│   └── test/                    # 테스트 코드
│       └── java/
│           └── com/example/demo/
│               └── DemoApplicationTests.java
│
├── .gitignore                   # Git 무시 파일 목록
├── build.gradle                 # Gradle 빌드 스크립트
├── gradlew                      # Gradle Wrapper 실행 스크립트 (Unix)
├── gradlew.bat                  # Gradle Wrapper 실행 스크립트 (Windows)
└── settings.gradle              # Gradle 설정 파일
```

주요 디렉토리와 파일 설명

1. gradle/wrapper/
    - Gradle Wrapper 관련 파일들이 위치
    - 프로젝트에서 사용할 Gradle 버전을 지정하고 자동으로 다운로드
2. src/main/java/
    - 애플리케이션의 주요 소스 코드가 위치
    - 패키지 구조에 따라 하위 디렉토리 구성
3. src/main/resources/
    - static/: 정적 리소스 파일 (JS, CSS, 이미지 등)
    - templates/: 템플릿 파일
    - application.yml: 애플리케이션 설정 파일
4. src/test/
    - 테스트 코드가 위치
    - 단위 테스트, 통합 테스트 등
5. build.gradle
    - 프로젝트의 의존성과 빌드 설정을 정의
    - 라이브러리 의존성, 빌드 스크립트, 플러그인 설정 등
    - 단순히 gradle 패키지 URL만 긁어서 추가하면 설치끝

### 2.2 도메인 기반 패키지 구조

Spring Boot는 역도메인 방식의 패키지 구조를 사용한다.
예) myhomepage123.kr 도메인의 경우

```
kr.myhomepage123.todo/
├── controller/
├── service/
└── repository/
```

이러한 역도메인 구조와 함께, Spring Boot에서는 크게 두 가지 방식의 패키지 구조가 사용된다

1. 계층형 구조 

```
com.example.project/
├── controller/      # 모든 컨트롤러
├── service/         # 모든 서비스
├── repository/      # 모든 리포지토리
├── domain/          # 모든 도메인 모델
└── dto/            # 모든 DTO
```

2. 도메인 주도 설계(DDD) 기반 구조

```
com.example.project/
├── domain/                 # 핵심 도메인
│   ├── user/              # 사용자 도메인
│   │   ├── controller/    # 사용자 관련 컨트롤러
│   │   ├── service/       # 사용자 관련 서비스
│   │   ├── repository/    # 사용자 관련 리포지토리
│   │   ├── dto/          # 사용자 관련 DTO
│   │   └── entity/       # 사용자 엔티티 등등 ... 
│   │
│   └── order/            # 주문 도메인 등등 ...
│       ├── controller/
│       ├── service/
│       ├── repository/
│       ├── dto/
│       └── entity/
│
├── global/               # 전역 설정
│   ├── config/          # 애플리케이션 설정
│   ├── error/           # 예외 처리
│   └── util/            # 유틸리티
│
└── infrastructure/      # 외부 시스템 통합
    ├── security/        # 보안 관련 설정
    ├── redis/          # Redis 연동
    ├── s3/             # AWS S3 연동
    ├── messaging/      # 메시징 시스템 연동
    └── external/       # 외부 API 클라이언트
```

DDD 기반 구조의 장점:
- 도메인별로 관련 코드가 모여있어 응집도가 높음
- 도메인 간 의존성 파악이 용이
- 특정 도메인의 코드 수정이 다른 도메인에 영향을 주지 않음
- 도메인별 독립적인 개발과 확장이 용이
- 마이크로서비스로의 전환이 상대적으로 쉬움

### 2.3 어노테이션 시스템

Python의 데코레이터와 비슷하지만 다른 특징을 가진다

- 파이썬 데코레이터: 함수를 감싸서 기능을 수정/추가
- 스프링 어노테이션: 메타데이터를 통한 설정과 기능 활성화

주요 어노테이션은 아래와같다. 근데 보통 컴포넌트 관련 어노테이션만 사용한다고 한다. 그리고 모르면 그냥 검색하면된다고한다.
자주쓰는것만 눈에 익혀두면 된다고한다.

1. 컴포넌트 관련
   - @SpringBootApplication: 진입점 설정
   - @Component: 일반적인 스프링 컴포넌트
   - @Controller/@RestController: 웹 요청 처리 컴포넌트
   - @Service: 비즈니스 로직 컴포넌트
   - @Repository: 데이터 접근 컴포넌트

2. 의존성 주입 관련
   - @Autowired: 자동 의존성 주입
   - @Qualifier: 특정 빈 지정
   - @Value: 프로퍼티 값 주입

3. 설정 관련
   - @Configuration: 설정 클래스 지정
   - @Bean: 수동으로 빈 등록
   - @PropertySource: 프로퍼티 파일 지정

4. JPA 관련
   - @Entity: JPA 엔티티 선언
   - @Table: 테이블 매핑
   - @Id: 기본키 지정

### 2.4 Gradle을 통한 빌드 관리

Node.js의 package.json이나 Python의 requirements.txt보다 더 강력한 빌드 시스템을 제공한다

Gradle의 주요 특징

- 의존성 관리
  - 트랜지티브 의존성 자동 관리
  - 버전 충돌 해결
- 빌드 자동화
  - 증분 빌드 지원
  - 빌드 캐시를 통한 성능 최적화
- 멀티 프로젝트 빌드
  - 대규모 프로젝트의 모듈화 지원
  - 서브 프로젝트 간 의존성 관리
- 테스트 통합
  - 다양한 테스트 프레임워크 지원
  - 테스트 리포트 생성
- 아티팩트 생성
  - JAR/WAR 파일 생성
  - 배포 설정 관리