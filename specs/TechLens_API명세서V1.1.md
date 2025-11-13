# TechLens API 명세서

## 개요

TechLens는 특허 정보 검색 및 분석 서비스를 제공하는 REST API입니다.

### 기본 정보

| 항목 | 값 |
|------|-----|
| Base URL | https://api.techlens.net/api/v1 |
| 인증 방식 | JWT Bearer Token |
| API 버전 | 2.1.0 |
| 총 엔드포인트 | 16개 |
| 응답 형식 | JSON |

---

## 인증

모든 API 요청에 JWT 토큰을 포함하여 전송합니다.

### Authorization Header

```http
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
```

### 토큰 획득 방법

1. 회원가입: POST /users/signup  
2. 로그인: POST /users/login  
3. **응답의 `data.token` 값** 추출  
4. 모든 요청의 Authorization 헤더에 포함

### JWT & 로그아웃 정책

- 서버는 로그인 시 JWT를 발급하며 `exp`(만료시각)를 포함합니다.  
- **로그아웃(POST /users/logout)** 호출 시, `Authorization` 헤더의 토큰을 **블랙리스트에 등록**하여 **만료 전이라도 즉시 무효화**합니다.  
- 블랙리스트에 등록된 토큰으로 접근 시 **401 Unauthorized**가 반환됩니다.  
- 만료된 토큰의 블랙리스트 항목은 주기적으로 정리(purge)됩니다.

#### 블랙리스트된 토큰 접근 시 오류 예시

```json
{
  "status": "fail",
  "message": "로그아웃된 토큰입니다."
}
```

---

## API 엔드포인트

### 1. Users (사용자 인증)

#### 1.1 회원가입

| 항목 | 내용 |
|------|------|
| 엔드포인트 | POST /users/signup |
| 인증 | 불필요 |
| 상태 코드 | 201 Created |

**Request Body**
```json
{
  "email": "user@techlens.net",
  "password": "abcd1234!"
}
```

**Response Body**
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
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

---

#### 1.2 로그인

| 항목 | 내용 |
|------|------|
| 엔드포인트 | POST /users/login |
| 인증 | 불필요 |
| 상태 코드 | 200 OK |

**Request Body**
```json
{
  "email": "user@techlens.net",
  "password": "abcd1234!"
}
```

**Response Body**
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
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
  }
}
```

---

#### 1.3 로그아웃

| 항목 | 내용 |
|------|------|
| 엔드포인트 | POST /users/logout |
| 인증 | JWT Bearer Token 필수 |
| 상태 코드 | 200 OK |

**Request Parameters**  
없음

**Request Body**  
없음

**Response Body**
```json
{
  "status": "success",
  "message": "로그아웃 성공"
}
```

---

### 2. Presets (프리셋 관리)

프리셋은 검색 조건을 저장하는 템플릿입니다. 저장된 프리셋은 특허검색 및 요약분석 탭에서 재사용됩니다.

#### 2.1 프리셋 생성

| 항목 | 내용 |
|------|------|
| 엔드포인트 | POST /presets |
| 인증 | JWT Bearer Token 필수 |
| 상태 코드 | 201 Created |

**Request Parameters**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| presetName | String | Yes | 프리셋 이름 (최대 255자) |
| applicant | String | Yes | 회사명 (최대 255자) |
| startDate | String | Yes | 검색 시작일 (YYYYMMDD 형식) |
| endDate | String | Yes | 검색 종료일 (YYYYMMDD 형식) |
| description | String | No | 프리셋 설명 (최대 500자) |

**Request Body**
```json
{
  "presetName": "삼성 2024년 분석",
  "applicant": "삼성전자",
  "startDate": "20240101",
  "endDate": "20241231",
  "description": "삼성의 2024년 특허 분석"
}
```

**Response Body**
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

| 항목 | 내용 |
|------|------|
| 엔드포인트 | GET /presets |
| 인증 | JWT Bearer Token 필수 |
| 상태 코드 | 200 OK |
| 페이징 | skip/limit 기반 |

**Request Parameters**

| 필드 | 타입 | 필수 | 기본값 | 설명 |
|------|------|------|--------|------|
| skip | Integer | No | 0 | 건너뛸 항목 수 |
| limit | Integer | No | 10 | 조회할 항목 개수 |

**Query String**
```
GET /presets?skip=0&limit=10
Authorization: Bearer <token>
```

**Response Body**  

```json
{
  "status": "success",
  "message": "프리셋 목록 조회 성공",
  "data": {
    "total": 5,
    "page": 1,
    "pageSize": 10,
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

| 항목 | 내용 |
|------|------|
| 엔드포인트 | GET /presets/{presetId} |
| 인증 | JWT Bearer Token 필수 |
| 상태 코드 | 200 OK |

**Request Parameters**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| presetId | Integer | Yes | 프리셋 ID (Path) |

**Query String**
```
GET /presets/1
Authorization: Bearer <token>
```

**Response Body**
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

| 항목 | 내용 |
|------|------|
| 엔드포인트 | PUT /presets/{presetId} |
| 인증 | JWT Bearer Token 필수 |
| 상태 코드 | 200 OK |

**Request Parameters**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| presetId | Integer | Yes | 프리셋 ID (Path) |
| presetName | String | No | 프리셋 이름 |
| applicant | String | No | 회사명 |
| startDate | String | No | 검색 시작일 (YYYYMMDD) |
| endDate | String | No | 검색 종료일 (YYYYMMDD) |
| description | String | No | 프리셋 설명 |

**Request Body**
```json
{
  "presetName": "삼성 2024년 최종",
  "applicant": "삼성전자",
  "startDate": "20240101",
  "endDate": "20241231",
  "description": "수정된 설명"
}
```

**Response Body**
```json
{
  "status": "success",
  "message": "프리셋이 수정되었습니다."
}
```

---

#### 2.5 프리셋 삭제

| 항목 | 내용 |
|------|------|
| 엔드포인트 | DELETE /presets/{presetId} |
| 인증 | JWT Bearer Token 필수 |
| 상태 코드 | 204 No Content |

**Request Parameters**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| presetId | Integer | Yes | 프리셋 ID (Path) |

**Query String**
```
DELETE /presets/1
Authorization: Bearer <token>
```

**Response Body**  

없음 (204 상태 코드만 반환)

---

### 3. Patents (특허 검색)

특허 검색은 프리셋에서 추출한 회사명과 기간으로 수행됩니다.

#### 3.1 기본 검색

| 항목 | 내용 |
|------|------|
| 엔드포인트 | POST /patents/search/basic |
| 인증 | JWT Bearer Token 필수 |
| 상태 코드 | 200 OK |
| 페이징 | page 기반 (20개씩) |

**Request Parameters**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| applicant | String | Yes | 회사명 |
| startDate | String | Yes | 검색 시작일 (YYYYMMDD) |
| endDate | String | Yes | 검색 종료일 (YYYYMMDD) |
| page | Integer | No | 페이지 번호 (기본값: 1) |

**Request Body**
```json
{
  "applicant": "삼성전자",
  "startDate": "20240101",
  "endDate": "20241231",
  "page": 1
}
```

**Response Body**
```json
{
  "status": "success",
  "message": "특허 검색 성공",
  "data": {
    "total": 635,
    "page": 1,
    "totalPages": 32,
    "patents": [
      {
        "applicationNumber": "1020250146315",
        "inventionTitle": "개선된 SSD 신뢰성",
        "applicantName": "삼성전자주식회사",
        "applicationDate": "20251010",
        "ipcCode": "G06F",
        "registerStatus": "공개",
        "isFavorite": true
      },
      {
        "applicationNumber": "1020250145497",
        "inventionTitle": "통신 시스템 방법",
        "applicantName": "삼성전자주식회사",
        "applicationDate": "20251002",
        "ipcCode": "H04L",
        "registerStatus": "공개",
        "isFavorite": false
      }
    ]
  }
}
```

---

#### 3.2 상세 검색

| 항목 | 내용 |
|------|------|
| 엔드포인트 | POST /patents/search/advanced |
| 인증 | JWT Bearer Token 필수 |
| 상태 코드 | 200 OK |
| 페이징 | page 기반 (20개씩) |

**Request Parameters**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| inventionTitle | String | No | 특허명 |
| applicant | String | Yes | 회사명 |
| startDate | String | Yes | 검색 시작일 (YYYYMMDD) |
| endDate | String | Yes | 검색 종료일 (YYYYMMDD) |
| registerStatus | String | No | 등록상태 (공개, 등록, 거절, 심사중, 출원) |
| page | Integer | No | 페이지 번호 (기본값: 1) |

**Request Body**
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

**Response Body**
```json
{
  "status": "success",
  "message": "특허 검색 성공",
  "data": {
    "total": 145,
    "page": 1,
    "totalPages": 8,
    "patents": [
      {
        "applicationNumber": "1020250146315",
        "inventionTitle": "개선된 SSD 신뢰성",
        "applicantName": "삼성전자주식회사",
        "applicationDate": "20251010",
        "ipcCode": "G06F",
        "registerStatus": "공개",
        "isFavorite": true
      }
    ]
  }
}
```

---

#### 3.3 특허 상세 조회

| 항목 | 내용 |
|------|------|
| 엔드포인트 | GET /patents/{applicationNumber} |
| 인증 | JWT Bearer Token 필수 |
| 상태 코드 | 200 OK |

**Request Parameters**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| applicationNumber | String | Yes | 출원번호 (Path) |

**Query String**
```
GET /patents/1020250146315
Authorization: Bearer <token>
```

**Response Body**
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

### 4. Analysis (요약 분석)

요약 분석은 검색 조건으로 통계 및 차트 데이터를 제공합니다.

#### 4.1 요약 분석

| 항목 | 내용 |
|------|------|
| 엔드포인트 | POST /analysis/summary |
| 인증 | JWT Bearer Token 필수 |
| 상태 코드 | 200 OK |

**Request Parameters**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| applicant | String | Yes | 회사명 |
| startDate | String | Yes | 분석 시작일 (YYYYMMDD) |
| endDate | String | Yes | 분석 종료일 (YYYYMMDD) |

**Request Body**
```json
{
  "applicant": "삼성전자",
  "startDate": "20240101",
  "endDate": "20241231"
}
```

**Response Body**
```json
{
  "status": "success",
  "message": "요약 분석 성공",
  "data": {
    "statistics": {
      "totalPatents": 635,
      "registrationRate": 45.5,
      "monthlyAverage": 52.9,
      "searchPeriod": {
        "startDate": "2024-01-01",
        "endDate": "2024-12-31"
      }
    },
    "ipcDistribution": [
      {
        "ipcCode": "G06F",
        "ipcName": "정보 처리",
        "count": 145,
        "percentage": 22.8
      },
      {
        "ipcCode": "H04L",
        "ipcName": "통신",
        "count": 120,
        "percentage": 18.9
      },
      {
        "ipcCode": "H01L",
        "ipcName": "반도체",
        "count": 98,
        "percentage": 15.4
      },
      {
        "ipcCode": "G06Q",
        "ipcName": "시스템",
        "count": 87,
        "percentage": 13.7
      },
      {
        "ipcCode": "H10B",
        "ipcName": "메모리",
        "count": 78,
        "percentage": 12.3
      }
    ],
    "monthlyTrend": [
      {
        "month": "2024-01",
        "count": 48,
        "cumulativeCount": 48
      },
      {
        "month": "2024-02",
        "count": 55,
        "cumulativeCount": 103
      },
      {
        "month": "2024-03",
        "count": 62,
        "cumulativeCount": 165
      }
    ],
    "statusDistribution": [
      {
        "status": "공개",
        "count": 289,
        "percentage": 45.5
      },
      {
        "status": "심사중",
        "count": 180,
        "percentage": 28.3
      },
      {
        "status": "등록",
        "count": 120,
        "percentage": 18.9
      },
      {
        "status": "출원",
        "count": 35,
        "percentage": 5.5
      },
      {
        "status": "거절",
        "count": 11,
        "percentage": 1.7
      }
    ],
    "recentPatents": [
      {
        "applicationNumber": "1020250146315",
        "inventionTitle": "개선된 SSD 신뢰성",
        "applicantName": "삼성전자",
        "applicationDate": "2025-10-10",
        "ipcCode": "G06F",
        "registerStatus": "공개",
        "isFavorite": true
      },
      {
        "applicationNumber": "1020250145497",
        "inventionTitle": "통신 시스템 방법",
        "applicantName": "삼성전자",
        "applicationDate": "2025-10-02",
        "ipcCode": "H04L",
        "registerStatus": "공개",
        "isFavorite": false
      },
      {
        "applicationNumber": "1020250145081",
        "inventionTitle": "반도체 패키지 구조",
        "applicantName": "삼성전자",
        "applicationDate": "2025-10-02",
        "ipcCode": "H01L",
        "registerStatus": "등록",
        "isFavorite": true
      }
    ]
  }
}
```

---

### 5. Favorites (관심특허 관리)

#### 5.1 관심특허 목록 조회

| 항목 | 내용 |
|------|------|
| 엔드포인트 | GET /favorites/list |
| 인증 | JWT Bearer Token 필수 |
| 상태 코드 | 200 OK |
| 페이징 | page/pageSize 기반 (20개씩) |

**Request Parameters**

| 필드 | 타입 | 필수 | 기본값 | 설명 |
|------|------|------|--------|------|
| page | Integer | No | 1 | 페이지 번호 |
| pageSize | Integer | No | 20 | 페이지당 항목 수 |

**Query String**
```
GET /favorites/list?page=1&pageSize=20
Authorization: Bearer <token>
```

**Response Body**
```json
{
  "status": "success",
  "message": "관심특허 목록 조회 성공",
  "data": {
    "total": 45,
    "page": 1,
    "pageSize": 20,
    "totalPages": 3,
    "favorites": [
      {
        "id": 1,
        "applicationNumber": "1020250146315",
        "inventionTitle": "개선된 SSD 신뢰성",
        "applicantName": "삼성전자주식회사",
        "applicationDate": "20251010",
        "ipcCode": "G06F",
        "registerStatus": "공개",
        "addedAt": "2025-11-03T15:30:00Z"
      },
      {
        "id": 2,
        "applicationNumber": "1020250145497",
        "inventionTitle": "통신 시스템 방법",
        "applicantName": "삼성전자주식회사",
        "applicationDate": "20251002",
        "ipcCode": "H04L",
        "registerStatus": "공개",
        "addedAt": "2025-11-02T10:15:00Z"
      }
    ]
  }
}
```

---

#### 5.2 관심특허 추가

| 항목 | 내용 |
|------|------|
| 엔드포인트 | POST /favorites |
| 인증 | JWT Bearer Token 필수 |
| 상태 코드 | 201 Created |

**Request Body**
```json
{
  "applicationNumber": "1020250146315"
}
```

**Response Body**
```json
{
  "status": "success",
  "message": "관심특허가 추가되었습니다."
}
```

---

#### 5.3 관심특허 상세 조회

| 항목 | 내용 |
|------|------|
| 엔드포인트 | GET /favorites/{applicationNumber} |
| 인증 | JWT Bearer Token 필수 |
| 상태 코드 | 200 OK |

**Request Parameters**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| applicationNumber | String | Yes | 출원번호 (Path) |

**Query String**
```
GET /favorites/1020250146315
Authorization: Bearer <token>
```

**Response Body**  
특허 상세 조회와 동일 (3.3 참조)

---

#### 5.4 관심특허 제거

| 항목 | 내용 |
|------|------|
| 엔드포인트 | DELETE /favorites/{applicationNumber} |
| 인증 | JWT Bearer Token 필수 |
| 상태 코드 | 204 No Content |

**Request Parameters**

| 필드 | 타입 | 필수 | 설명 |
|------|------|------|------|
| applicationNumber | String | Yes | 출원번호 (Path) |

**Query String**
```
DELETE /favorites/1020250146315
Authorization: Bearer <token>
```

**Response Body**  
없음 (204 상태 코드만 반환)

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

특허 분류 코드로, 파이프(|)로 구분하여 다중 코드 표현.  
예: `G06F 3/06|G06F 11/10|G06F 9/451`

---

## HTTP 상태 코드

| 코드 | 설명 |
|------|------|
| 200 | 요청 성공 |
| 201 | 리소스 생성 성공 |
| 204 | 리소스 삭제 성공 |
| 400 | 잘못된 요청 (필수 파라미터 누락 등) |
| 401 | 인증 실패 (토큰 없음 또는 만료) |
| 404 | 리소스를 찾을 수 없음 |
| 500 | 서버 오류 |

---

## 설계 원칙

### 프리셋 관리
프리셋은 검색 조건을 저장하는 템플릿으로 작동합니다. 특허검색 및 요약분석 탭에서 프리셋을 선택하면 저장된 회사명과 기간이 검색창에 자동으로 채워집니다. **목록 조회에서는 description을 제외**하여 응답 크기를 줄이고, 상세 조회 시 전체 정보를 제공합니다.

### 특허 검색
특허 검색은 프리셋에서 추출한 회사명과 기간으로 수행되며, 검색 결과는 20개씩 페이징됩니다. 기본 검색과 상세 검색으로 나뉘어 필터링 옵션을 제공합니다.

### 요약 분석
요약 분석은 파이차트(IPC 분포), 라인차트(월별 동향), 원형차트(등록상태 분포), 통계 정보, 최근 3개 특허를 제공합니다.

### 관심특허 관리
관심특허는 출원번호만으로 추가할 수 있으며, 백엔드에서 특허 정보를 자동으로 조회합니다. 관심특허는 20개씩 페이징되어 조회됩니다.

---

Version: 1.1
Date: 2025-11-13
작성자: 심우현 (KNU / Kicom Internship)
변경사항: PostgreSQL 환경 대응, USERS.password_hash 반영, JWT_BLACKLIST 테이블 추가

© 2025 TechLens Project. All rights reserved.
본 문서의 내용은 Kicom × KNU 인턴십 프로그램의 일부로 작성되었으며,
무단 복제·배포를 금합니다.
