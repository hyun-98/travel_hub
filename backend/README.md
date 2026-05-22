# Travel Hub — Backend

> 한국관광공사 TourAPI를 연동한 여행 정보 플랫폼의 Spring Boot 백엔드 서버

---

## 목차

1. [프로젝트 소개](#1-프로젝트-소개)
2. [사용 기술](#2-사용-기술)
3. [실행 방법](#3-실행-방법)
4. [주요 기능](#4-주요-기능)
5. [아키텍처](#5-아키텍처)
6. [배운 점](#6-배운-점)
7. [트러블슈팅](#7-트러블슈팅)

---

## 1. 프로젝트 소개

Travel Hub Backend는 한국 여행 정보 플랫폼의 서버 사이드를 담당합니다.  
한국관광공사 TourAPI(KorService2)를 실시간으로 호출해 관광지·축제 데이터를 제공하고, 자체 DB에는 커뮤니티(게시판), 여행 일정, 리뷰 등 사용자 생성 데이터를 저장합니다.

- 로컬 이메일/비밀번호 인증과 Google·GitHub·Kakao 소셜 로그인을 모두 지원합니다.
- JWT 기반 Stateless 인증으로 프론트엔드(React)와 분리된 API 서버로 동작합니다.
- 게시글 다중 이미지 업로드, 댓글/대댓글, 좋아요·북마크, 여행 일정 CRUD 등 다양한 기능을 제공합니다.

---

## 2. 사용 기술

| 분류 | 기술 |
|---|---|
| Language | Java 17 |
| Framework | Spring Boot 3.5.7 |
| ORM | Spring Data JPA (Hibernate) |
| Database | PostgreSQL |
| Security | Spring Security, JWT (jjwt 0.11.5) |
| OAuth2 | Spring OAuth2 Client (Google, GitHub, Kakao) |
| HTTP Client | Spring WebFlux WebClient |
| Build Tool | Gradle |
| Utility | Lombok, Jackson, Spring Validation |

---

## 3. 실행 방법

### 사전 요구사항

- Java 17 이상
- PostgreSQL 실행 중
- 한국관광공사 TourAPI 서비스 키 발급 ([공공데이터포털](https://www.data.go.kr))
- Kakao Map API 키 (선택)

### 환경 변수 설정

`src/main/resources/application.properties` 또는 환경 변수로 아래 값을 설정합니다.

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/travelhub
spring.datasource.username=your_db_user
spring.datasource.password=your_db_password

secret_key=YOUR_TOUR_API_SERVICE_KEY
tourism.serviceKey=YOUR_TOUR_API_SERVICE_KEY

spring.security.oauth2.client.registration.google.client-id=YOUR_GOOGLE_CLIENT_ID
spring.security.oauth2.client.registration.google.client-secret=YOUR_GOOGLE_CLIENT_SECRET
```

### 실행

```bash
# 프로젝트 루트(backend/)에서 실행
./gradlew bootRun

# 또는 JAR 빌드 후 실행
./gradlew build
java -jar build/libs/backend-0.0.1-SNAPSHOT.jar
```

서버는 기본적으로 `http://localhost:8080`에서 실행됩니다.

---

## 4. 주요 기능

### 인증 / 회원
- 이메일·비밀번호 회원가입 및 로그인 (BCrypt 암호화)
- JWT 발급 및 검증 (만료 1시간)
- Google, GitHub, Kakao OAuth2 소셜 로그인
- 프로필 이미지 업로드, 닉네임·이름 수정, 비밀번호 변경

### 관광지 / 축제
- 한국관광공사 TourAPI 실시간 연동 (관광지 상세, 키워드 검색, 지역 기반 목록)
- 인기 관광지 Top 10 조회 (DB 인기도 기준)
- 진행 중인 축제 목록 필터링
- 관광지 혼잡도 통계 조회 (`tatsCnctrRatedList`)
- 지역명 → areaCode 자동 매핑 (17개 광역시·도)

### 커뮤니티 (게시판)
- 게시글 CRUD (카테고리: 잡담 / 질문 / 꿀팁)
- 다중 이미지 업로드, 썸네일 지정, 이미지 순서 관리
- 소프트 삭제 (`deletedAt` 필드)
- 5가지 정렬: 최신순 / 오래된순 / 조회수 / 댓글수 / 좋아요수
- 4가지 검색: 제목 / 내용 / 닉네임 / 제목+내용
- 좋아요·북마크 토글
- 조회수 별도 엔드포인트로 분리 (중복 증가 방지)

### 댓글
- 댓글 및 대댓글 (자기참조 계층 구조)
- 댓글 좋아요 토글
- 소프트 삭제

### 여행 일정
- 일정 CRUD (관광지 ID, 날짜, 시간, 순서 포함)
- 날짜 범위 필터 조회
- TourAPI에서 관광지 제목 실시간 조회

### 리뷰
- 관광지별 별점 리뷰 작성·조회
- 평균 별점 계산
- 내 리뷰 목록 조회

### 지도
- Kakao Map API 좌표 변환 및 주소 검색 프록시

---

## 5. 아키텍처

```
┌─────────────────────────────────────────────────────┐
│                   Frontend (React)                   │
│                  localhost:5173                       │
└──────────────────────┬──────────────────────────────┘
                       │ HTTP / REST API
                       ▼
┌─────────────────────────────────────────────────────┐
│              Spring Boot Backend                     │
│                 localhost:8080                        │
│                                                      │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐  │
│  │Controller│→ │ Service  │→ │   Repository     │  │
│  └──────────┘  └──────────┘  └────────┬─────────┘  │
│                     │                  │             │
│              ┌──────┴──────┐           ▼             │
│              │  WebClient  │     PostgreSQL DB        │
│              └──────┬──────┘                         │
└─────────────────────┼───────────────────────────────┘
                       │ HTTPS
                       ▼
          ┌────────────────────────┐
          │  한국관광공사 TourAPI   │
          │  apis.data.go.kr       │
          └────────────────────────┘
```

### 패키지 구조

```
com.example.backend
├── config/          # SecurityConfig, CorsConfig, WebClientConfig
├── controller/      # REST 컨트롤러 (15개)
├── dto/             # 요청/응답 DTO
├── entity/          # JPA 엔티티
├── exception/       # 커스텀 예외 클래스
├── repository/      # Spring Data JPA 레포지토리
├── security/        # JwtFilter, JwtUtil
└── service/         # 비즈니스 로직 (15개)
```

### 인증 흐름

```
[로컬 로그인]
Client → POST /auth/login → AuthService → BCrypt 검증 → JWT 발급 → Client

[소셜 로그인]
Client → /oauth2/authorization/{provider} → Spring OAuth2 → OAuthService
→ 신규 유저 자동 가입 → JWT 발급 → /oauth/success → 프론트 /oauth/callback?token=...
```

---

## 6. 배운 점

- **WebClient 비동기 호출**: RestTemplate 대신 WebFlux WebClient를 사용해 외부 API(TourAPI)를 호출하면서 리액티브 프로그래밍의 기초를 익혔습니다. `block()`으로 동기 처리하는 방식과 그 한계도 이해하게 되었습니다.

- **Spring Security FilterChain 분리**: `/api/**`와 OAuth2 경로를 별도 FilterChain으로 분리하여 JWT 인증과 세션 기반 OAuth2 흐름을 독립적으로 관리하는 방법을 배웠습니다.

- **소프트 삭제 패턴**: `deletedAt` 필드를 활용한 논리 삭제를 구현하면서 데이터 복구 가능성과 쿼리 복잡도 사이의 트레이드오프를 경험했습니다.

- **다중 이미지 관리**: 게시글에 여러 이미지를 첨부하고 순서와 썸네일을 관리하는 로직을 구현하면서 연관 엔티티 설계와 `@OneToMany` 관계 처리를 깊이 이해했습니다.

- **외부 API 데이터 정규화**: TourAPI 응답의 HTML 태그 포함 문자열, 날짜 포맷(`yyyyMMdd`) 등 비정형 데이터를 파싱하고 정규화하는 과정에서 방어적 코딩의 중요성을 배웠습니다.

---

## 7. 트러블슈팅

### TourAPI 응답의 `homepage` 필드에 HTML 태그 포함

**문제**: TourAPI의 `homepage` 필드가 `<a href="...">링크</a>` 형태의 HTML 문자열로 반환되어 프론트에서 그대로 표시되는 문제가 발생했습니다.

**해결**: `cleanHomepage()` 유틸 메서드를 작성해 정규식으로 `href` 속성값만 추출하고, HTML 엔티티(`&lt;`, `&gt;`)를 디코딩하도록 처리했습니다.

---

### OAuth2 로그인 후 JWT를 프론트엔드로 전달하는 방법

**문제**: Spring OAuth2의 성공 핸들러는 서버 사이드 리다이렉트를 사용하는데, Stateless JWT 방식에서는 세션 없이 토큰을 전달해야 했습니다.

**해결**: OAuth 성공 후 `/oauth/success` 엔드포인트에서 JWT를 생성하고, 쿼리 파라미터(`?token=...`)로 프론트엔드 콜백 URL에 리다이렉트하는 방식으로 해결했습니다.

---

### 게시글 조회수 중복 증가 문제

**문제**: 게시글 상세 조회 API(`GET /api/posts/{id}`)를 호출할 때마다 조회수가 증가해, 새로고침이나 데이터 재조회 시 조회수가 부정확하게 집계되었습니다.

**해결**: 조회수 증가 로직을 별도 엔드포인트(`POST /api/posts/{id}/view`)로 분리하고, 데이터 조회용 엔드포인트(`GET /api/posts/{id}/data`)는 조회수를 변경하지 않도록 분리했습니다.

---

### SecurityConfig에서 두 FilterChain 충돌

**문제**: 하나의 SecurityFilterChain에서 JWT 인증과 OAuth2 로그인을 함께 설정하면 경로 매칭이 충돌하거나 OAuth2 흐름이 JWT 필터에 의해 차단되는 문제가 발생했습니다.

**해결**: `@Order(1)` API 전용 FilterChain(`/api/**`)과 `@Order(2)` OAuth2 FilterChain을 분리하여 각각 독립적으로 동작하도록 구성했습니다.
