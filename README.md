# Travel Hub

> 한국관광공사 TourAPI 기반 여행 정보 플랫폼

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

**Travel Hub**는 한국 여행 정보를 한 곳에서 탐색하고 공유할 수 있는 풀스택 웹 플랫폼입니다.

한국관광공사 TourAPI(KorService2)를 실시간으로 연동해 전국 관광지·축제 정보를 제공하고, 사용자가 직접 여행 일정을 관리하거나 커뮤니티에서 여행 경험을 나눌 수 있습니다.

- 관광지 키워드 검색 및 지역별 탐색
- 진행 중인 축제 목록 조회
- 관광지 별점 리뷰 작성
- 여행 일정 CRUD
- 커뮤니티 게시판 (게시글, 댓글, 좋아요, 북마크)
- 로컬 회원가입 및 Google·GitHub·Kakao 소셜 로그인

```
backend/   → Spring Boot REST API 서버
frontend/  → React + Vite SPA 클라이언트
```

---

## 2. 사용 기술

### Backend

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

### Frontend

| 분류 | 기술 |
|---|---|
| Language | JavaScript |
| Framework | React 19 |
| Build Tool | Vite 7 |
| Routing | React Router DOM 7 |

---

## 3. 실행 방법

### 사전 요구사항

- Java 17 이상
- Node.js 18 이상
- PostgreSQL 실행 중
- 한국관광공사 TourAPI 서비스 키 ([공공데이터포털](https://www.data.go.kr) 발급)

### Backend 실행

`backend/src/main/resources/application.properties`에 아래 값을 설정합니다.

```properties
spring.datasource.url=jdbc:postgresql://localhost:5432/travelhub
spring.datasource.username=your_db_user
spring.datasource.password=your_db_password

secret_key=YOUR_TOUR_API_SERVICE_KEY
tourism.serviceKey=YOUR_TOUR_API_SERVICE_KEY

spring.security.oauth2.client.registration.google.client-id=YOUR_GOOGLE_CLIENT_ID
spring.security.oauth2.client.registration.google.client-secret=YOUR_GOOGLE_CLIENT_SECRET
```

```bash
cd backend
./gradlew bootRun
# → http://localhost:8080
```

### Frontend 실행

```bash
cd frontend
npm install
npm run dev
# → http://localhost:5173
```

두 서버를 모두 실행한 뒤 브라우저에서 `http://localhost:5173`에 접속합니다.

---

## 4. 주요 기능

### 관광지 / 축제 탐색
- 키워드 검색 (관광지명, 지역명 모두 지원)
- 지역 기반 관광지 목록 (17개 광역시·도, 콘텐츠 타입 필터)
- 인기 관광지 Top 10
- 진행 중인 축제 목록
- 관광지 상세 정보 + 카카오맵 지도 표시
- 관광지 혼잡도 통계 조회

<img width="600" height="300" alt="image" src="https://github.com/user-attachments/assets/61d5b6d6-fc5b-48d6-8aac-76b932ef1835" />


### 리뷰
- 관광지별 별점 리뷰 작성 및 목록 조회
- 평균 별점 표시
- 내 리뷰 목록 조회

### 여행 일정
- 일정 생성·수정·삭제 (관광지 ID, 날짜, 시간, 순서 포함)
- 날짜 범위 필터 조회

### 커뮤니티 (게시판)
- 게시글 CRUD (카테고리: 잡담 / 질문 / 꿀팁)
- 다중 이미지 업로드, 썸네일 지정, 이미지 순서 관리
- 5가지 정렬: 최신순 / 오래된순 / 조회수 / 댓글수 / 좋아요수
- 4가지 검색: 제목 / 내용 / 닉네임 / 제목+내용
- 좋아요·북마크 토글
- 댓글 및 대댓글, 댓글 좋아요

### 인증 / 회원
- 이메일·비밀번호 회원가입 및 로그인 (BCrypt 암호화)
- Google, GitHub, Kakao OAuth2 소셜 로그인
- JWT 기반 Stateless 인증 (만료 1시간)
- 프로필 이미지 업로드, 닉네임·이름 수정, 비밀번호 변경

<img width="600" height="300" alt="image" src="https://github.com/user-attachments/assets/20a2f551-4e30-487a-8f0a-dc28b690575f" />
<img width="600" height="300" alt="image" src="https://github.com/user-attachments/assets/1f6c3e07-8adf-4617-9137-bcab13581e0a" />


---

## 5. 아키텍처

### 전체 구성

```
┌──────────────────────────────────────────────────────────┐
│                  Browser (localhost:5173)                  │
│                   React + Vite SPA                        │
│                                                           │
│  Pages → Components → Zustand Store                      │
│                  ↕ Axios (Bearer JWT)                     │
└──────────────────────┬───────────────────────────────────┘
                       │ REST API
                       ▼
┌──────────────────────────────────────────────────────────┐
│              Spring Boot (localhost:8080)                  │
│                                                           │
│  Controller → Service → Repository → PostgreSQL DB        │
│                  ↕                                        │
│             WebClient                                     │
└──────────────────────┬───────────────────────────────────┘
                       │ HTTPS
          ┌────────────┴────────────┐
          ▼                         ▼
 한국관광공사 TourAPI          Kakao Map API
 apis.data.go.kr
```

### 인증 흐름

```
[로컬 로그인]
POST /auth/login → BCrypt 검증 → JWT 발급 → localStorage 저장

[소셜 로그인]
/oauth2/authorization/{provider}
→ Spring OAuth2 처리 → 신규 유저 자동 가입
→ JWT 발급 → /oauth/callback?token=JWT (프론트)
→ authStore 저장 → 홈으로 이동
```

### 백엔드 패키지 구조

```
com.example.backend
├── config/       # Security, CORS, WebClient 설정
├── controller/   # REST 컨트롤러 (15개)
├── dto/          # 요청/응답 DTO
├── entity/       # JPA 엔티티 (User, Post, Comment, Spot 등)
├── exception/    # 커스텀 예외
├── repository/   # Spring Data JPA 레포지토리
├── security/     # JwtFilter, JwtUtil
└── service/      # 비즈니스 로직 (15개)
```

### 프론트엔드 디렉토리 구조

```
frontend/src
├── components/   # 재사용 UI 컴포넌트 (community, festival, layout, review, spot, ui)
├── pages/        # 라우트별 페이지 (11개)
├── services/     # API 호출 함수
├── store/        # Zustand 전역 상태 (auth, spot, festival, post, review, user)
├── App.jsx       # 라우팅 설정
└── main.jsx      # 앱 진입점
```

---

## 6. 배운 점

- **외부 API 실시간 연동**: TourAPI를 WebClient로 호출하면서 비동기 HTTP 클라이언트 사용법과 비정형 응답 데이터(HTML 태그 포함 문자열, 날짜 포맷 등)를 정규화하는 방어적 코딩의 중요성을 배웠습니다.

- **JWT + OAuth2 혼합 인증**: Stateless JWT 방식과 Spring OAuth2의 세션 기반 흐름을 함께 사용하면서 두 방식의 차이를 이해하고, FilterChain을 분리해 충돌 없이 공존시키는 방법을 익혔습니다.

- **풀스택 데이터 흐름 설계**: 백엔드 엔티티 설계부터 DTO 변환, API 응답, 프론트엔드 상태 관리까지 데이터가 흐르는 전체 경로를 직접 설계하면서 각 레이어의 역할과 책임을 명확히 이해했습니다.

- **소프트 삭제 패턴**: `deletedAt` 필드를 활용한 논리 삭제로 데이터 복구 가능성을 확보하면서, 모든 조회 쿼리에 `deletedAt IS NULL` 조건을 일관되게 적용하는 것의 중요성을 배웠습니다.

- **Zustand 전역 상태 관리**: Redux 대비 간결한 Zustand로 인증 상태를 관리하고, localStorage와 연동해 새로고침 후에도 상태를 복원하는 패턴을 구현했습니다.

- **다중 이미지 업로드 처리**: `multipart/form-data`로 여러 이미지를 업로드하고 순서·썸네일을 관리하는 로직을 프론트-백엔드 양쪽에서 구현하면서 파일 업로드의 전체 흐름을 이해했습니다.

---

## 7. 트러블슈팅

### OAuth2 로그인 후 JWT를 프론트엔드로 전달하는 방법

**문제**: Spring OAuth2의 성공 핸들러는 서버 사이드 리다이렉트를 사용하는데, Stateless JWT 방식에서는 세션 없이 토큰을 전달해야 했습니다.

**해결**: OAuth 성공 후 `/oauth/success` 엔드포인트에서 JWT를 생성하고, 쿼리 파라미터(`?token=...`)로 프론트엔드 콜백 URL에 리다이렉트했습니다. 프론트의 `OAuthCallback` 컴포넌트에서 `URLSearchParams`로 토큰을 추출해 저장합니다.

---

### SecurityConfig에서 두 FilterChain 충돌

**문제**: 하나의 SecurityFilterChain에서 JWT 인증과 OAuth2 로그인을 함께 설정하면 경로 매칭이 충돌하거나 OAuth2 흐름이 JWT 필터에 의해 차단되었습니다.

**해결**: `@Order(1)` API 전용 FilterChain(`/api/**`)과 `@Order(2)` OAuth2 FilterChain을 분리해 각각 독립적으로 동작하도록 구성했습니다.

---

### 게시글 조회수 중복 증가 문제

**문제**: 게시글 상세 조회 API를 호출할 때마다 조회수가 증가해 새로고침이나 데이터 재조회 시 조회수가 부정확하게 집계되었습니다.

**해결**: 조회수 증가 로직을 별도 엔드포인트(`POST /api/posts/{id}/view`)로 분리하고, 데이터 조회용 엔드포인트는 조회수를 변경하지 않도록 분리했습니다.

---

### 이미지 업로드 시 Content-Type 충돌

**문제**: `multipart/form-data`로 이미지를 업로드할 때 axios 인터셉터에서 `Content-Type: application/json`을 강제 설정하면 boundary 값이 사라져 서버에서 파싱 오류가 발생했습니다.

**해결**: axios 인터셉터에서 요청 데이터가 `FormData` 인스턴스인 경우 `Content-Type` 헤더를 삭제하도록 처리했습니다. 브라우저가 자동으로 올바른 boundary를 포함한 헤더를 설정합니다.

---

### TourAPI 응답의 `homepage` 필드에 HTML 태그 포함

**문제**: TourAPI의 `homepage` 필드가 `<a href="...">링크</a>` 형태의 HTML 문자열로 반환되어 프론트에서 그대로 표시되는 문제가 발생했습니다.

**해결**: `cleanHomepage()` 유틸 메서드를 작성해 정규식으로 `href` 속성값만 추출하고, HTML 엔티티(`&lt;`, `&gt;`)를 디코딩하도록 처리했습니다.

---

> 각 서버의 상세 내용은 [backend/README.md](./backend/README.md)와 [frontend/README.md](./frontend/README.md)를 참고하세요.
