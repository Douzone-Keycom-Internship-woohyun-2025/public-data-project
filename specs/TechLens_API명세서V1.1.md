# TechLens API 명세서 · v1.1

Base URL: `https://techlens-backend-develop.onrender.com` · Auth: JWT Bearer · 응답: JSON

---

## 목차
- [개요](#개요)
- [인증](#인증)
  - [Authorization 헤더](#authorization-헤더)
  - [토큰 획득](#토큰-획득)
  - [AccessToken / RefreshToken 정책](#AccessToken/RefreshToken정책)
- [API 엔드포인트](#api-엔드포인트)
  - [1. Users (사용자 인증)](#1-users-사용자-인증)
    - [1.1 회원가입](#11-회원가입)
    - [1.2 로그인](#12-로그인)
    - [1.3 로그아웃](#13-로그아웃)
    - [1.4 토큰재발급](#14-토큰재발급)
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
  - [4. Summary (요약 분석)](#4-summary-요약-분석)
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
| 총 엔드포인트 | 17개 |
| 응답 형식 | JSON |

> 참고: 현재 배포는 URL 버전 프리픽스(`/api/v1`) 없이 **루트 경로**에 라우팅됩니다. 예) `/users`, `/presets`.

---

## 인증

### Authorization 헤더
```http
Authorization: Bearer <accessToken>
```

### 토큰 획득
1) `POST /users/signup`
2) `POST /users/login`
3) 응답의 data.accessToken + data.refreshToken 추출
4) API 요청 시 Authorization에 accessToken 포함

### AccessToken / RefreshToken 정책
- 로그인 시 accessToken(1시간) + refreshToken(7일) 발급
- refreshToken은 서버 DB에 저장됨
- accessToken이 만료되면 /users/refresh 호출해 재발급

### 로그아웃 정책
- /users/logout 호출 시 해당 사용자의 refreshToken을 DB에서 삭제
- refreshToken 삭제 후:
  - 더 이상 accessToken 재발급 불가
  - 기존 accessToken은 자체 만료 시까지 유효

오류 예시:
```json
{ "status": "fail", "message": "RefreshToken이 유효하지 않습니다." }
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
    "user": {
      "user_tblkey": 2,
      "email": "user@techlens.net",
      "adddate": "2025-11-12T21:36:16.430Z"
    },
    "accessToken": "xxx.yyy.zzz",
    "refreshToken": "rrr.sss.ttt"
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
    "user": {
      "user_tblkey": 1,
      "email": "user@techlens.net",
      "adddate": "2025-11-11T23:08:29.608Z"
    },
    "accessToken": "xxx.yyy.zzz",
    "refreshToken": "rrr.sss.ttt"
  }
}
```

---

#### 1.3 로그아웃
- 메서드/경로: **POST** `/users/logout`
- 인증: accessToken 필수

Request:
```json
{ "refreshToken": "rrr.sss.ttt" }
```

Response:
```json
{ "status": "success", "message": "로그아웃 완료" }
```

> 로그아웃 시 DB에 저장된 refreshToken이 삭제되며, 기존 accessToken은 자연 만료됩니다.

---

#### 1.4 토큰재발급
- 메서드/경로: **POST** `/users/refresh`
- 인증: 불필요 (refreshToken으로 인증)

Request:
```json
{ "refreshToken": "rrr.sss.ttt" }
```

Response:
```json
{
  "status": "success",
  "message": "토큰 재발급 성공",
  "data": {
    "accessToken": "새로운AccessToken"
  }
}
```

---

### 2) Presets (프리셋 관리)

프리셋은 검색 조건 템플릿입니다. 목록 조회는 description을 제외하고, 상세 조회에서만 전체 정보를 제공합니다.

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
    "totalPages": 1,
    "presets": [
      {
        "id": 1,
        "presetName": "삼성 2024년 분석",
        "applicant": "삼성전자",
        "startDate": "20240101",
        "endDate": "20241231",
        "createdAt": "2025-11-03T10:30:00Z"
      },
      {
        "id": 2,
        "presetName": "LG 2023년 분석",
        "applicant": "LG전자",
        "startDate": "20230101",
        "endDate": "20231231",
        "createdAt": "2025-11-02T14:15:00Z"
      }
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
- 메서드/경로: **PATCH** `/presets/{presetId}`
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

프리셋에서 추출된 회사명과 기간으로 특허를 검색합니다. 페이지당 20건입니다.

#### 3.1 기본 검색
- 메서드/경로: **POST** `/patents/search/basic`
- 인증: 필수
- 응답: **200 OK**

Request:
```json
{
  "applicant": "삼성전자",
  "startDate": "20240101",
  "endDate": "20241231",
  "page": 1
}
```

Response (요약):
```json
{
  "status": "success",
  "message": "특허 검색 성공",
  "data": {
    "total": 635,
    "page": 1,
    "totalPages": 32,
    "patents": [ /* ... */ ]
  }
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
  "data": {
    "total": 145,
    "page": 1,
    "totalPages": 8,
    "patents": [ /* ... */ ]
  }
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

### 4) Summary (요약 분석)

#### 4.1 요약 분석
- 메서드/경로: **GET** `/summary`
- 인증: 필수
- 응답: **200 OK**
- 설명: 요약 분석은 두 가지 방식으로 조회 가능합니다:
  - **프리셋 기반 조회**: `presetId` 쿼리 파라미터 사용
  - **직접 검색**: `applicant`, `startDate`, `endDate` 쿼리 조합 사용

Request 예시 (presetId 사용):
```
GET /summary?presetId=1
Authorization: Bearer <accessToken>
```

Request 예시 (applicant + startDate + endDate 사용):
```
GET /summary?applicant=카카오&startDate=20250501&endDate=20251114
Authorization: Bearer <accessToken>
```

Response:
```json
{
  "status": "success",
  "message": "요약 분석 완료",
  "data": {
    "applicant": "카카오",
    "period": {
      "startDate": "20250501",
      "endDate": "20251114"
    },
    "totalCount": 123,
    "statusCount": {
      "공개": 50,
      "등록": 30
    },
    "statusPercent": {
      "공개": 40.65,
      "등록": 24.39
    },
    "monthlyTrend": [
      { "month": "2025-05", "count": 10 },
      { "month": "2025-06", "count": 15 }
    ],
    "topIPC": [
      { "code": "G06F", "count": 15 },
      { "code": "H04L", "count": 12 }
    ],
    "recentPatents": [
      {
        "applicantName": "카카오",
        "inventionTitle": "AI 기반 검색 시스템",
        "applicationDate": "20251110",
        "registerStatus": "공개",
        "ipcMain": "G06F"
      }
    ],
    "avgMonthlyCount": 15.5
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
  "data": {
    "total": 45,
    "page": 1,
    "pageSize": 20,
    "totalPages": 3,
    "favorites": [ /* ... */ ]
  }
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

응답은 [3.3 특허 상세 조회](#33-특허-상세-조회)와 동일합니다.

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

- **프리셋 관리**: 목록 조회는 최소한의 필드(minimal fields)로, 상세 조회에서 전체 정보 제공하여 성능과 가독성 개선
- **특허 검색**: 프리셋 기반 파라미터로 통일, 페이지당 20건
- **요약 분석**: presetId 또는 applicant + startDate + endDate 쿼리 조합으로 유연한 검색 지원
- **관심특허**: 출원번호(applicationNumber)만으로 관리
- **인증**: 모든 엔드포인트는 JWT Bearer Token 기반 인증 (Users 회원가입, 로그인, 토큰 재발급 제외)

---

## 메타

- **Version**: 1.1
- **Date**: 2025-11-14
- **작성자**: 심우현 (KNU / Kicom Internship)
- **변경사항**: 구현 코드 반영, Summary 경로/메서드 통일, Summary 프리셋 및 직접 검색 옵션 추가, Preset 수정 메서드 PATCH로 변경, 로그아웃 메시지 수정
- © 2025 TechLens Project. All rights reserved. (Kicom × KNU 인턴십 문서 / 무단 복제·배포 금지)
