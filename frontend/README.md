# Travel Hub — Frontend

> 한국 여행 정보 플랫폼의 React 프론트엔드 클라이언트

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

Travel Hub Frontend는 한국 여행 정보 플랫폼의 사용자 인터페이스를 담당합니다.  
관광지·축제 탐색, 여행 일정 관리, 커뮤니티 게시판 등의 기능을 React 기반 SPA로 구현했습니다.

- 한국관광공사 TourAPI 데이터를 기반으로 관광지 검색, 지역별 탐색, 축제 정보를 제공합니다.
- 로컬 로그인과 Google·GitHub·Kakao 소셜 로그인을 지원합니다.
- Zustand로 전역 인증 상태를 관리하고, Tailwind CSS로 반응형 UI를 구성했습니다.

---

## 2. 사용 기술

| 분류 | 기술 |
|---|---|
| Language | JavaScript (ES2022+) |
| Framework | React 19 |
| Build Tool | Vite 7 |
| Routing | React Router DOM 7 |
| 상태 관리 | Zustand 5 |
| HTTP Client | Axios 1.13 |
| Styling | Tailwind CSS 4 |
| 아이콘 | Lucide React, React Icons |
| 슬라이더 | Swiper 12 |
| WebSocket | @stomp/stompjs, SockJS |

---

## 3. 실행 방법

### 사전 요구사항

- Node.js 18 이상
- 백엔드 서버 실행 중 (`http://localhost:8080`)

### 설치 및 실행

```bash
# 의존성 설치
npm install

# 개발 서버 실행
npm run dev
```

개발 서버는 기본적으로 `http://localhost:5173`에서 실행됩니다.

### 빌드

```bash
# 프로덕션 빌드
npm run build

# 빌드 결과물 미리보기
npm run preview
```

### 환경 변수 (선택)

프로젝트 루트에 `.env` 파일을 생성해 API 베이스 URL 등을 설정할 수 있습니다.

```env
VITE_API_BASE_URL=http://localhost:8080
```

---

## 4. 주요 기능

### 홈 화면
- 인기 관광지 Top 10 카드 슬라이더
- 현재 진행 중인 축제 목록
- 6개 주요 지역 카드 (클릭 시 지역별 관광지 목록으로 이동)
- 통합 검색창 (키워드 입력 후 검색 결과 페이지로 이동)

### 관광지 상세
- 관광지 기본 정보 (이름, 주소, 전화번호, 홈페이지, 운영시간)
- 카카오맵 지도 표시
- 별점 리뷰 작성 및 목록 조회, 평균 별점 표시
- 주변 숙박업소 정보 (Kakao Local API 연동)

### 검색 / 탐색
- 키워드 검색 결과 그리드 (관광지명, 지역명 모두 지원)
- 지역 기반 관광지 목록 (지역 코드 + 콘텐츠 타입 필터, 페이지네이션)
- 축제 목록 페이지 (진행 중인 축제 카드 그리드)

### 커뮤니티 (게시판)
- 게시글 목록: 카테고리 필터(잡담/질문/꿀팁), 정렬(최신/조회수/좋아요/댓글수), 키워드 검색
- 게시글 작성: 다중 이미지 첨부, 썸네일 지정
- 게시글 상세: 좋아요·북마크 토글, 댓글/대댓글 작성
- 페이지네이션

### 인증
- 이메일·비밀번호 회원가입 및 로그인
- Google, GitHub, Kakao 소셜 로그인 (OAuth2)
- 로그인 상태에 따른 라우트 보호 (미인증 시 리다이렉트)

### 마이페이지
- 프로필 이미지 업로드 및 닉네임·이름 수정
- 내가 쓴 게시글, 댓글, 리뷰 탭별 조회
- 비밀번호 변경

---

## 5. 아키텍처

### 디렉토리 구조

```
src/
├── components/          # 재사용 가능한 UI 컴포넌트
│   ├── community/       # 게시판 관련 컴포넌트
│   ├── festival/        # 축제 카드 컴포넌트
│   ├── layout/          # 헤더, 푸터, 레이아웃
│   ├── review/          # 리뷰 작성/목록 컴포넌트
│   ├── spot/            # 관광지 카드 컴포넌트
│   └── ui/              # 공통 UI (버튼, 모달 등)
├── pages/               # 라우트별 페이지 컴포넌트
│   ├── TravelHubHome.jsx
│   ├── spotDetail.jsx
│   ├── community.jsx
│   ├── SearchResults.jsx
│   ├── FestivalListPage.jsx
│   ├── area_based_list_page.jsx
│   ├── profile.jsx
│   ├── login.jsx
│   ├── signup.jsx
│   ├── OAuthCallback.jsx
│   └── ChangePassword.jsx
├── services/            # API 호출 함수 모음
├── store/               # Zustand 전역 상태
│   ├── authStore.jsx    # 인증 상태 (user, isAuthenticated)
│   ├── spotStore.jsx
│   ├── festivalStore.jsx
│   ├── postStore.jsx
│   ├── reviewStore.jsx
│   └── userStore.jsx
├── App.jsx              # 라우팅 설정
└── main.jsx             # 앱 진입점
```

### 라우팅 구조

| 경로 | 페이지 | 인증 필요 |
|---|---|---|
| `/` | 홈 | ❌ |
| `/spotDetail` | 관광지 상세 | ❌ |
| `/search` | 검색 결과 | ❌ |
| `/explore` | 지역별 관광지 | ❌ |
| `/festivals` | 축제 목록 | ❌ |
| `/community` | 커뮤니티 | ❌ |
| `/login` | 로그인 | ❌ (인증 시 `/`로 이동) |
| `/signup` | 회원가입 | ❌ (인증 시 `/`로 이동) |
| `/oauth/callback` | OAuth 콜백 | ❌ |
| `/profile` | 마이페이지 | ✅ |
| `/change-password` | 비밀번호 변경 | ❌ |

### 데이터 흐름

```
Page Component
    │
    ├── Zustand Store (전역 상태 읽기/쓰기)
    │
    └── services/ (API 호출)
            │
            └── axios instance (Bearer 토큰 자동 주입)
                    │
                    └── Spring Boot Backend (localhost:8080)
```

### 인증 흐름

```
[소셜 로그인]
Login 페이지 → /oauth2/authorization/{provider} (백엔드)
→ OAuth2 인증 완료 → 백엔드 /oauth/success
→ 프론트 /oauth/callback?token=JWT
→ OAuthCallback 컴포넌트에서 토큰 추출
→ authStore.oauthLogin(token) → localStorage 저장 → 홈으로 이동
```

---

## 6. 배운 점

- **Zustand로 전역 상태 관리**: Redux 대비 보일러플레이트가 적고 직관적인 Zustand를 사용하면서, 인증 상태처럼 앱 전역에서 필요한 데이터를 효율적으로 관리하는 방법을 익혔습니다.

- **Axios 인터셉터 활용**: 모든 API 요청에 JWT를 자동으로 주입하고, 401 응답 시 로그인 페이지로 리다이렉트하는 인터셉터를 구현하면서 공통 처리 로직을 중앙화하는 패턴을 배웠습니다.

- **React Router의 보호 라우트 패턴**: `isAuthenticated` 상태에 따라 `<Navigate>`로 리다이렉트하는 방식으로 인증이 필요한 페이지를 보호하는 패턴을 구현했습니다.

- **Tailwind CSS 유틸리티 클래스**: 컴포넌트 단위로 스타일을 관리하면서 반응형 레이아웃을 빠르게 구성하는 방법을 익혔습니다. 클래스가 길어지는 단점도 경험했습니다.

- **외부 지도 API 연동**: 카카오맵 SDK를 React 컴포넌트 라이프사이클에 맞게 동적으로 로드하고, 좌표 기반으로 마커를 표시하는 방법을 배웠습니다.

---

## 7. 트러블슈팅

### OAuth 콜백에서 JWT 토큰 추출 실패

**문제**: 소셜 로그인 성공 후 백엔드가 `?token=...` 쿼리 파라미터로 리다이렉트하는데, React Router가 URL을 파싱하기 전에 컴포넌트가 렌더링되어 토큰이 `null`로 읽히는 경우가 발생했습니다.

**해결**: `OAuthCallback` 컴포넌트에서 `useEffect` 내부에서 `URLSearchParams`로 토큰을 추출하고, 추출 성공 시에만 `authStore`에 저장 후 홈으로 이동하도록 처리했습니다.

---

### 게시글 이미지 업로드 시 Content-Type 충돌

**문제**: `multipart/form-data`로 이미지를 업로드할 때 axios 인터셉터에서 `Content-Type: application/json`을 강제로 설정하면 boundary 값이 사라져 서버에서 파싱 오류가 발생했습니다.

**해결**: axios 인스턴스의 요청 인터셉터에서 `FormData` 인스턴스인 경우 `Content-Type` 헤더를 삭제하도록 처리했습니다. 브라우저가 자동으로 올바른 `multipart/form-data; boundary=...` 헤더를 설정합니다.

---

### 인증 상태가 새로고침 후 초기화되는 문제

**문제**: Zustand 스토어는 메모리 상태이므로 페이지를 새로고침하면 `isAuthenticated`가 `false`로 초기화되어 로그인 상태가 유지되지 않았습니다.

**해결**: `authStore`에서 초기화 시 `localStorage`에 저장된 JWT 토큰을 읽어 유효한 토큰이 있으면 인증 상태를 복원하도록 구현했습니다.

---

### 카카오맵 SDK 중복 로드 문제

**문제**: 관광지 상세 페이지를 여러 번 방문하면 카카오맵 SDK 스크립트가 중복으로 `<head>`에 추가되어 콘솔 경고와 함께 지도가 정상적으로 초기화되지 않는 문제가 발생했습니다.

**해결**: SDK 스크립트를 추가하기 전에 `document.querySelector`로 이미 로드된 스크립트가 있는지 확인하고, 없을 때만 동적으로 추가하도록 처리했습니다.
