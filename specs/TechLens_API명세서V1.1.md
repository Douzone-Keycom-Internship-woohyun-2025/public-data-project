# TechLens API 명세서 · v1.1
Base URL: `https://techlens-backend-develop.onrender.com` · Auth: JWT Bearer · 응답: JSON

---

## 목차
- [개요](#개요)
- [인증](#인증)
  - [Authorization 헤더](#authorization-헤더)
  - [토큰 획득](#토큰-획득)
  - [JWT & 로그아웃 정책](#jwt--로그아웃-정책)
- [API 엔드포인트](#api-엔드포인트)
  - [1. Users (사용자 인증)](#1-users-사용자-인증)
    - [1.1 회원가입](#11-회원가입)
    - [1.2 로그인](#12-로그인)
    - [1.3 로그아웃](#13-로그아웃)
  - [2. Presets (프리셋 관리)](#2-presets-프리셋-관리)
    - [2.1 프리셋 생성](#21-프리셋-생성)
    - [2.2 프리셋 목록 조회](#22-프리셋-목록-조회)
    - [2.3 프리셋 상세 조회](#23-프리셋-상세-조회)
    - [2.4 프리셋 수정](#24-프리셋-수정)
    - [2.5 프리셋 삭제](#25-프리셋-삭제)
  - [3. Patents (특허 검색)](#3-patents-특허-검색)
    - [3.1 기본 검색](#31-기본-검색)
    - [3.2 상세 검색](#32-상세-검색)
    - [3.3 특허 상세 조회](#33-특허-상세-조회)
  - [4. Analysis (요약 분석)](#4-analysis-요약-분석)
    - [4.1 요약 분석](#41-요약-분석)
  - [5. Favorites (관심특허 관리)](#5-favorites-관심특허-관리)
    - [5.1 관심특허 목록 조회](#51-관심특허-목록-조회)
    - [5.2 관심특허 추가](#52-관심특허-추가)
    - [5.3 관심특허 상세 조회](#53-관심특허-상세-조회)
    - [5.4 관심특허 제거](#54-관심특허-제거)
- [데이터 타입](#데이터-타입)
- [HTTP 상태 코드](#http-상태-코드)
- [설계 원칙](#설계-원칙)
- [메타](#메타)

---

## 개요
| 항목 | 값 |
|------|-----|
| Base URL | `https://techlens-backend-develop.onrender.com` |
| 인증 방식 | JWT Bearer Token |
| API 버전 | 2.1.0 |
| 총 엔드포인트 | 16개 |
| 응답 형식 | JSON |

> 참고: 현재 배포는 URL 버전 프리픽스(`/api/v1`) 없이 **루트 경로**에 라우팅됩니다. 예) `/users`, `/presets`.

---

## 인증

### Authorization 헤더
```http
Authorization: Bearer <JWT 토큰>
```

### 토큰 획득
1) `POST /users/signup`  
2) `POST /users/login`  
3) 응답의 `data.token` 추출  
4) 모든 요청의 `Authorization`에 포함

### JWT & 로그아웃 정책
- 로그인 시 `exp` 포함된 JWT 발급
- `POST /users/logout` 시 해당 토큰을 블랙리스트에 등록하여 만료 전에도 즉시 무효화
- 블랙리스트 토큰 접근 시 401
- 만료된 블랙리스트 항목은 주기적으로 purge

오류 예시:
```json
{ "status": "fail", "message": "로그아웃된 토큰입니다." }
```

---

## API 엔드포인트

### 1) Users (사용자 인증)

#### 1.1 회원가입
- 메서드/경로: **POST** `/users/signup`
- 인증: 불필요
- 응답: **201 Created**

Request:
```json
{ "email": "user@techlens.net", "password": "abcd1234!" }
```

Response:
```json
{
  "status": "success",
  "message": "회원가입 성공",
  "data": {
    "user": { "user_tblkey": 2, "email": "user@techlens.net", "adddate": "2025-11-12T21:36:16.430Z" },
    "token": "eyJhbGciOiJIUzI1NiIsInR..."
  }
}
```

---

#### 1.2 로그인
- 메서드/경로: **POST** `/users/login`
- 인증: 불필요
- 응답: **200 OK**

Request:
```json
{ "email": "user@techlens.net", "password": "abcd1234!" }
```

Response:
```json
{
  "status": "success",
  "message": "로그인 성공",
  "data": {
    "user": { "user_tblkey": 1, "email": "user@techlens.net", "adddate": "2025-11-11T23:08:29.608Z" },
    "token": "eyJhbGciOiJIUzI1NiIsInR..."
  }
}
```

---

#### 1.3 로그아웃
- 메서드/경로: **POST** `/users/logout`
- 인증: 필수
- 응답: **200 OK**

Response:
```json
{ "status": "success", "message": "로그아웃 성공" }
```

---

### 2) Presets (프리셋 관리)
프리셋은 검색 조건 템플릿. 목록 조회는 description 제외, 상세 조회에서 전체 정보 제공.

#### 2.1 프리셋 생성
- 메서드/경로: **POST** `/presets`
- 인증: 필수
- 응답: **201 Created**

Request:
```json
{
  "presetName": "삼성 2024년 분석",
  "applicant": "삼성전자",
  "startDate": "20240101",
  "endDate": "20241231",
  "description": "삼성의 2024년 특허 분석"
}
```

Response:
```json
{
  "status": "success",
  "message": "프리셋이 생성되었습니다.",
  "data": {
    "id": 1,
    "presetName": "삼성 2024년 분석",
    "applicant": "삼성전자",
    "startDate": "20240101",
    "endDate": "20241231",
    "description": "삼성의 2024년 특허 분석",
    "createdAt": "2025-11-03T10:30:00Z"
  }
}
```

---

#### 2.2 프리셋 목록 조회
- 메서드/경로: **GET** `/presets?skip=0&limit=10`
- 인증: 필수
- 응답: **200 OK**

Response (description 제외):
```json
{
  "status": "success",
  "message": "프리셋 목록 조회 성공",
  "data": {
    "total": 5,
    "page": 1,
    "pageSize": 10,
    "presets": [
      { "id": 1, "presetName": "삼성 2024년 분석", "applicant": "삼성전자", "startDate": "20240101", "endDate": "20241231", "createdAt": "2025-11-03T10:30:00Z" },
      { "id": 2, "presetName": "LG 2023년 분석", "applicant": "LG전자", "startDate": "20230101", "endDate": "20231231", "createdAt": "2025-11-02T14:15:00Z" }
    ]
  }
}
```

---

#### 2.3 프리셋 상세 조회
- 메서드/경로: **GET** `/presets/{presetId}`
- 인증: 필수
- 응답: **200 OK**

Response:
```json
{
  "status": "success",
  "data": {
    "id": 1,
    "presetName": "삼성 2024년 분석",
    "applicant": "삼성전자",
    "startDate": "20240101",
    "endDate": "20241231",
    "description": "삼성의 2024년 특허 분석",
    "createdAt": "2025-11-03T10:30:00Z"
  }
}
```

---

#### 2.4 프리셋 수정
- 메서드/경로: **PUT** `/presets/{presetId}`
- 인증: 필수
- 응답: **200 OK**

Request:
```json
{
  "presetName": "삼성 2024년 최종",
  "applicant": "삼성전자",
  "startDate": "20240101",
  "endDate": "20241231",
  "description": "수정된 설명"
}
```

Response:
```json
{ "status": "success", "message": "프리셋이 수정되었습니다." }
```

---

#### 2.5 프리셋 삭제
- 메서드/경로: **DELETE** `/presets/{presetId}`
- 인증: 필수
- 응답: **204 No Content**

---

### 3) Patents (특허 검색)
프리셋에서 추출된 회사명·기간으로 검색. 페이지당 20건.

#### 3.1 기본 검색
- 메서드/경로: **POST** `/patents/search/basic`
- 인증: 필수
- 응답: **200 OK**

Request:
```json
{ "applicant": "삼성전자", "startDate": "20240101", "endDate": "20241231", "page": 1 }
```

Response (요약):
```json
{
  "status": "success",
  "message": "특허 검색 성공",
  "data": { "total": 635, "page": 1, "totalPages": 32, "patents": [ /* ... */ ] }
}
```

---

#### 3.2 상세 검색
- 메서드/경로: **POST** `/patents/search/advanced`
- 인증: 필수
- 응답: **200 OK**

Request:
```json
{
  "inventionTitle": "AI 기반",
  "applicant": "삼성전자",
  "startDate": "20240101",
  "endDate": "20241231",
  "registerStatus": "공개",
  "page": 1
}
```

Response (요약):
```json
{
  "status": "success",
  "message": "특허 검색 성공",
  "data": { "total": 145, "page": 1, "totalPages": 8, "patents": [ /* ... */ ] }
}
```

---

#### 3.3 특허 상세 조회
- 메서드/경로: **GET** `/patents/{applicationNumber}`
- 인증: 필수
- 응답: **200 OK**

Response (요약):
```json
{
  "status": "success",
  "message": "특허 상세 조회 성공",
  "data": {
    "applicationNumber": "1020250146315",
    "inventionTitle": "개선된 SSD 신뢰성",
    "applicantName": "삼성전자주식회사",
    "applicationDate": "20251010",
    "openDate": "20251021",
    "openNumber": "1020250151330",
    "publicationDate": null,
    "publicationNumber": null,
    "registerDate": null,
    "registerNumber": null,
    "ipcNumber": "G06F 3/06|G06F 11/10|G06F 9/451",
    "registerStatus": "공개",
    "astrtCont": "솔리드 스테이트 드라이브(SSD)의 신뢰성을 향상시키는 방법...",
    "bigDrawing": "http://plus.kipris.or.kr/...",
    "drawing": "http://plus.kipris.or.kr/..."
  }
}
```

---

### 4) Analysis (요약 분석)

#### 4.1 요약 분석
- 메서드/경로: **POST** `/analysis/summary`
- 인증: 필수
- 응답: **200 OK**

Request:
```json
{ "applicant": "삼성전자", "startDate": "20240101", "endDate": "20241231" }
```

Response (요약):
```json
{
  "status": "success",
  "message": "요약 분석 성공",
  "data": {
    "statistics": { "totalPatents": 635, "registrationRate": 45.5, "monthlyAverage": 52.9, "searchPeriod": { "startDate": "2024-01-01", "endDate": "2024-12-31" } },
    "ipcDistribution": [ /* ... */ ],
    "monthlyTrend": [ /* ... */ ],
    "statusDistribution": [ /* ... */ ],
    "recentPatents": [ /* ... */ ]
  }
}
```

---

### 5) Favorites (관심특허 관리)

#### 5.1 관심특허 목록 조회
- 메서드/경로: **GET** `/favorites/list?page=1&pageSize=20`
- 인증: 필수
- 응답: **200 OK**

Response (요약):
```json
{
  "status": "success",
  "message": "관심특허 목록 조회 성공",
  "data": { "total": 45, "page": 1, "pageSize": 20, "totalPages": 3, "favorites": [ /* ... */ ] }
}
```

---

#### 5.2 관심특허 추가
- 메서드/경로: **POST** `/favorites`
- 인증: 필수
- 응답: **201 Created**

Request:
```json
{ "applicationNumber": "1020250146315" }
```

Response:
```json
{ "status": "success", "message": "관심특허가 추가되었습니다." }
```

---

#### 5.3 관심특허 상세 조회
- 메서드/경로: **GET** `/favorites/{applicationNumber}`
- 인증: 필수
- 응답: **200 OK**

응답은 [3.3 특허 상세 조회](#33-특허-상세-조회)와 동일

---

#### 5.4 관심특허 제거
- 메서드/경로: **DELETE** `/favorites/{applicationNumber}`
- 인증: 필수
- 응답: **204 No Content**

---

## 데이터 타입

### 날짜 형식
| 형식 | 예시 | 사용처 |
|------|------|--------|
| YYYYMMDD | 20240101 | API 요청 파라미터 |
| ISO 8601 | 2024-01-01 | API 응답 데이터 |

### 등록 상태
| 상태 | 설명 |
|------|------|
| 공개 | 공개 |
| 심사중 | 심사 중 |
| 등록 | 등록 완료 |
| 출원 | 출원됨 |
| 거절 | 거절됨 |

### IPC 코드
- 특허 분류 코드, 파이프(`|`)로 다중 표현  
- 예: `G06F 3/06|G06F 11/10|G06F 9/451`

---

## HTTP 상태 코드
| 코드 | 설명 |
|------|------|
| 200 | 요청 성공 |
| 201 | 리소스 생성 성공 |
| 204 | 리소스 삭제 성공 |
| 400 | 잘못된 요청 (필수 파라미터 누락 등) |
| 401 | 인증 실패 (토큰 없음/만료/블랙리스트) |
| 404 | 리소스를 찾을 수 없음 |
| 500 | 서버 오류 |

---

## 설계 원칙
- 프리셋 관리: 목록(minimal fields) vs 상세(full fields) 분리로 성능·가독성 개선  
- 특허 검색: 프리셋 기반 파라미터로 통일, page=20 기본  
- 요약 분석: 통계/IPC 분포/월별 추이/상태 분포/최근 건 제공  
- 관심특허: 출원번호만으로 추가, 상세는 특허 상세와 동일 형식 재사용

---

## 메타
- Version: 1.1  
- Date: 2025-11-13  
- 작성자: 심우현 (KNU / Kicom Internship)  
- 변경사항: **배포 URL 반영(`onrender.com`)**, PostgreSQL 대응, `USERS.password_hash` 반영, JWT 블랙리스트 정책 반영  
- © 2025 TechLens Project. All rights reserved. (Kicom × KNU 인턴십 문서 / 무단 복제·배포 금지)
