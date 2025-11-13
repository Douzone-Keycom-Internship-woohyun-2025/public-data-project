# TechLens 데이터베이스 정의서 (V1.1)

본 문서는 **Kicom × KNU TechLens 프로젝트**의 데이터베이스 구조를 정의한 문서입니다.  
본 정의서는 **PostgreSQL 환경**에서 작성되었으며,  
KIPRIS Open API 기반의 특허 검색 및 분석 서비스를 위한 핵심 테이블 구조를 포함합니다.

---

## ERD 다이어그램


<img width="1391" height="866" alt="BookStore (2)" src="https://github.com/user-attachments/assets/9e02caa1-0c94-4828-b495-8695189beb6f" />



---

## 1. 개요

| 항목 | 내용 |
|------|------|
| 데이터베이스명 | TECHLENS_DB |
| DBMS | **PostgreSQL 14 이상** |
| 문자 인코딩 | UTF8 |
| 정규화 수준 | 제3정규형 (3NF) |
| 주요 기능 | 사용자 관리, 프리셋 저장, 관심특허 관리, IPC 코드 매핑, JWT 블랙리스트 |
| 테이블 수 | 6개 |

---

## 2. 테이블 목록

| No | 테이블명 | 설명 |
|----|-----------|------|
| 1 | USERS | 사용자 계정 정보 |
| 2 | PRESETS | 사용자 프리셋 정보 |
| 3 | FAVORITE_PATENTS | 관심특허 정보 |
| 4 | PATENT_IPC_SUBCLASS_MAP | 특허–Subclass 매핑 정보 |
| 5 | IPC_SUBCLASS_MAP | IPC Subclass 사전 정보 |
| 6 | JWT_BLACKLIST | 로그아웃 처리용 JWT 보관 |

---

## 3. 테이블 정의서

### 3.1 USERS (사용자 계정 정보)

| 컬럼명 | 데이터 타입 | 제약조건 | 설명 |
|--------|--------------|-----------|------|
| USER_TBLKEY | **SERIAL** | PK | 사용자 고유 식별자 |
| EMAIL | VARCHAR(255) | UNIQUE, NOT NULL | 로그인용 이메일 |
| PASSWORD_HASH | VARCHAR(255) | NOT NULL | 해시된 비밀번호 (bcrypt) |
| ADDDATE | **TIMESTAMP** | **DEFAULT NOW()** | 계정 생성일시 (자동) |

---

### 3.2 PRESETS (프리셋 정보)

| 컬럼명 | 데이터 타입 | 제약조건 | 설명 |
|--------|--------------|-----------|------|
| PRESET_TBLKEY | **SERIAL** | PK | 프리셋 고유 식별자 |
| USER_TBLKEY | INTEGER | FK → USERS.USER_TBLKEY, NOT NULL | 프리셋 소유 사용자 |
| PRESET_NAME | VARCHAR(255) | NOT NULL | 프리셋 이름 |
| APPLICANT | VARCHAR(255) | NOT NULL | 검색 대상 회사명 |
| START_DATE | VARCHAR(8) | NOT NULL | 검색 시작일 (YYYYMMDD) |
| END_DATE | VARCHAR(8) | NOT NULL | 검색 종료일 (YYYYMMDD) |
| DESCRIPTION | VARCHAR(500) | NULL | 프리셋 설명 |
| ADDDATE | **TIMESTAMP** | **DEFAULT NOW()** | 등록일시 (자동) |

**제약조건**
- FOREIGN KEY (`USER_TBLKEY`) REFERENCES `USERS`(`USER_TBLKEY`) ON DELETE CASCADE

---

### 3.3 FAVORITE_PATENTS (관심특허 정보)

| 컬럼명 | 데이터 타입 | 제약조건 | 설명 |
|--------|--------------|-----------|------|
| PATENT_TBLKEY | **SERIAL** | PK | 관심특허 고유 식별자 |
| USER_TBLKEY | INTEGER | FK → USERS.USER_TBLKEY, NOT NULL | 사용자 식별자 |
| APPLICATION_NUMBER | VARCHAR(30) | NOT NULL | 특허 출원번호 |
| INVENTION_TITLE | VARCHAR(500) | NOT NULL | 발명의 명칭 |
| APPLICANT_NAME | VARCHAR(255) | NOT NULL | 출원인명 |
| ABSTRACT | TEXT | NULL | 초록 요약 |
| APPLICATION_DATE | VARCHAR(8) | NOT NULL | 출원일 (YYYYMMDD) |
| PUBLICATION_DATE | VARCHAR(8) | NULL | 공개일 (YYYYMMDD) |
| REGISTER_DATE | VARCHAR(8) | NULL | 등록일 (YYYYMMDD) |
| REGISTER_STATUS | VARCHAR(50) | NULL | 등록 상태 (등록, 거절, 소멸 등) |
| DRAWING_URL | VARCHAR(500) | NULL | 대표 도면 이미지 URL |
| ADDDATE | **TIMESTAMP** | **DEFAULT NOW()** | 등록일시 (자동) |

**제약조건**
- PRIMARY KEY (`PATENT_TBLKEY`)
- FOREIGN KEY (`USER_TBLKEY`) REFERENCES `USERS`(`USER_TBLKEY`) ON DELETE CASCADE
- UNIQUE (`USER_TBLKEY`, `APPLICATION_NUMBER`)

---

### 3.4 PATENT_IPC_SUBCLASS_MAP (특허–Subclass 매핑 정보)

| 컬럼명 | 데이터 타입 | 제약조건 | 설명 |
|--------|--------------|-----------|------|
| MAPPING_TBLKEY | **SERIAL** | PK | 매핑 고유 식별자 |
| PATENT_TBLKEY | INTEGER | FK → FAVORITE_PATENTS.PATENT_TBLKEY, NOT NULL | 특허 참조 |
| IPC_SUBCLASS | VARCHAR(10) | FK → IPC_SUBCLASS_MAP.IPC_SUBCLASS, NOT NULL | IPC Subclass 코드 (예: G06Q) |

**제약조건**
- FOREIGN KEY (`PATENT_TBLKEY`) REFERENCES `FAVORITE_PATENTS`(`PATENT_TBLKEY`) ON DELETE CASCADE
- FOREIGN KEY (`IPC_SUBCLASS`) REFERENCES `IPC_SUBCLASS_MAP`(`IPC_SUBCLASS`) ON DELETE CASCADE
- UNIQUE (`PATENT_TBLKEY`, `IPC_SUBCLASS`)

---

### 3.5 IPC_SUBCLASS_MAP (IPC Subclass 사전)

| 컬럼명 | 데이터 타입 | 제약조건 | 설명 |
|--------|--------------|-----------|------|
| IPC_SUBCLASS | VARCHAR(10) | PK, NOT NULL | Subclass 코드 (예: G06Q) |
| KOR_NAME | VARCHAR(255) | NOT NULL | 한글 기술명 |

---

### 3.6 JWT_BLACKLIST (JWT 로그아웃 토큰 보관)

| 컬럼명     | 데이터 타입 | 제약조건         | 설명                  |
|------------|-------------|------------------|-----------------------|
| TOKEN      | TEXT        | **PK, NOT NULL** | JWT 원문(서명 포함)\* |
| EXPIRES_AT | TIMESTAMPTZ | **NOT NULL**     | 토큰 만료 일시(UTC)   |


**운영 메모**
- 로그아웃 시: (token, exp) 저장
- 요청 인증 시: 블랙리스트 존재 여부 조회 → 존재하면 401
- 정리 작업: `DELETE FROM jwt_blacklist WHERE expires_at < NOW()`

---

## 4. 설계 요약

| 항목 | 내용 |
|------|------|
| 외래키 관계 | USERS → PRESETS, FAVORITE_PATENTS → PATENT_IPC_SUBCLASS_MAP → IPC_SUBCLASS_MAP |
| 인덱스 권장 | EMAIL, APPLICATION_NUMBER, IPC_SUBCLASS, (JWT_BLACKLIST.expires_at) |
| 정규화 수준 | 제3정규형 (3NF) |
| 무결성 확보 | FK 및 복합 UNIQUE 제약조건으로 데이터 중복 방지 |
| 확장성 | IPC Subclass 기반 분석, 통계, 관심 패턴 확장 가능 |
| 기본값 정책 | ADDDATE는 NOW() 기본값 사용 (JWT_BLACKLIST 제외) |
| 날짜 형식 | **TIMESTAMP 사용 (ISO 8601: 2025-11-12T15:27:30.123Z)** |
| CASCADE 정책 | 사용자 삭제 시 관련 데이터 자동 삭제 (ON DELETE CASCADE) |

---

## 5. 주요 변경사항 (V1.0 → V1.1)

| 항목 | V1.0 (MariaDB) | V1.1 (PostgreSQL) |
|------|----------------|-------------------|
| DBMS | MariaDB 10.11+ | **PostgreSQL 14+** |
| AUTO_INCREMENT | AUTO_INCREMENT | **SERIAL** |
| USERS 비밀번호 컬럼 | `password` | **`password_hash`** |
| ADDDATE 타입 | VARCHAR(17) | **TIMESTAMP** |
| ADDDATE 형식 | YYYYMMDDHHmmssfff | **ISO 8601 (자동)** |
| 기본값 정책 | 수동 입력 | **DEFAULT NOW()** |
| CASCADE 지원 | 수동 설정 | **ON DELETE CASCADE** |
| 신규 테이블 | 없음 | **JWT_BLACKLIST 추가** |

---

## 6. PostgreSQL 생성 스크립트

```sql
-- ============================================
-- TechLens Database - PostgreSQL V1.1 (최신 반영)
-- ============================================

-- 1) 기존 테이블 삭제 (의존성 고려해 역순)
DROP TABLE IF EXISTS patent_ipc_subclass_map CASCADE;
DROP TABLE IF EXISTS ipc_subclass_map CASCADE;
DROP TABLE IF EXISTS favorite_patents CASCADE;
DROP TABLE IF EXISTS presets CASCADE;
DROP TABLE IF EXISTS jwt_blacklist CASCADE;
DROP TABLE IF EXISTS users CASCADE;

-- 2) USERS
CREATE TABLE users (
  user_tblkey    SERIAL PRIMARY KEY,
  email          VARCHAR(255) UNIQUE NOT NULL,
  password_hash  VARCHAR(255) NOT NULL,
  adddate        TIMESTAMP DEFAULT NOW()
);

-- 3) PRESETS
CREATE TABLE presets (
  preset_tblkey  SERIAL PRIMARY KEY,
  user_tblkey    INTEGER NOT NULL,
  preset_name    VARCHAR(255) NOT NULL,
  applicant      VARCHAR(255) NOT NULL,
  start_date     VARCHAR(8) NOT NULL,
  end_date       VARCHAR(8) NOT NULL,
  description    VARCHAR(500),
  adddate        TIMESTAMP DEFAULT NOW(),
  FOREIGN KEY (user_tblkey) REFERENCES users(user_tblkey) ON DELETE CASCADE
);

-- 4) FAVORITE_PATENTS
CREATE TABLE favorite_patents (
  patent_tblkey       SERIAL PRIMARY KEY,
  user_tblkey         INTEGER NOT NULL,
  application_number  VARCHAR(30) NOT NULL,
  invention_title     VARCHAR(500) NOT NULL,
  applicant_name      VARCHAR(255) NOT NULL,
  abstract            TEXT,
  application_date    VARCHAR(8) NOT NULL,
  publication_date    VARCHAR(8),
  register_date       VARCHAR(8),
  register_status     VARCHAR(50),
  drawing_url         VARCHAR(500),
  adddate             TIMESTAMP DEFAULT NOW(),
  FOREIGN KEY (user_tblkey) REFERENCES users(user_tblkey) ON DELETE CASCADE,
  UNIQUE (user_tblkey, application_number)
);

-- 5) IPC_SUBCLASS_MAP
CREATE TABLE ipc_subclass_map (
  ipc_subclass  VARCHAR(10) PRIMARY KEY NOT NULL,
  kor_name      VARCHAR(255) NOT NULL
);

-- 6) PATENT_IPC_SUBCLASS_MAP
CREATE TABLE patent_ipc_subclass_map (
  mapping_tblkey  SERIAL PRIMARY KEY,
  patent_tblkey   INTEGER NOT NULL,
  ipc_subclass    VARCHAR(10) NOT NULL,
  FOREIGN KEY (patent_tblkey) REFERENCES favorite_patents(patent_tblkey) ON DELETE CASCADE,
  FOREIGN KEY (ipc_subclass) REFERENCES ipc_subclass_map(ipc_subclass) ON DELETE CASCADE,
  UNIQUE (patent_tblkey, ipc_subclass)
);

-- 7) JWT_BLACKLIST (adddate 없음)
CREATE TABLE jwt_blacklist (
  token       TEXT PRIMARY KEY,
  expires_at  TIMESTAMPTZ NOT NULL
);

-- 8) 인덱스
CREATE INDEX idx_users_email           ON users(email);
CREATE INDEX idx_presets_user          ON presets(user_tblkey);
CREATE INDEX idx_favorites_user        ON favorite_patents(user_tblkey);
CREATE INDEX idx_favorites_appnum      ON favorite_patents(application_number);
CREATE INDEX idx_patent_ipc_patent     ON patent_ipc_subclass_map(patent_tblkey);
CREATE INDEX idx_patent_ipc_subclass   ON patent_ipc_subclass_map(ipc_subclass);
CREATE INDEX idx_jwt_blacklist_expires ON jwt_blacklist(expires_at);
```

Version: 1.1
Date: 2025-11-13
작성자: 심우현 (KNU / Kicom Internship)
변경사항: PostgreSQL 환경 대응, USERS.password_hash 반영, JWT_BLACKLIST 테이블 추가

© 2025 TechLens Project. All rights reserved.
본 문서의 내용은 Kicom × KNU 인턴십 프로그램의 일부로 작성되었으며,
무단 복제·배포를 금합니다.
