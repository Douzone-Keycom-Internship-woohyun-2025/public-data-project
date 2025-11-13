# TechLens 데이터베이스 정의서 (V1.1)

본 문서는 **Kicom × KNU TechLens 프로젝트**의 데이터베이스 구조를 정의한 문서입니다.  
본 정의서는 **PostgreSQL 환경**에서 작성되었으며,  
KIPRIS Open API 기반의 특허 검색 및 분석 서비스를 위한 핵심 테이블 구조를 포함합니다.

---

## 목차
- [ERD 다이어그램](#erd-다이어그램)
- [1. 개요](#1-개요)
- [2. 테이블 목록](#2-테이블-목록)
- [3. 테이블 정의서](#3-테이블-정의서)
  - [3.1 USERS (사용자 계정 정보)](#31-users-사용자-계정-정보)
  - [3.2 REFRESH_TOKENS (리프레시 토큰 저장)](#32-refresh_tokens-리프레시-토큰-저장)
  - [3.3 PRESETS (프리셋 정보)](#33-presets-프리셋-정보)
  - [3.4 FAVORITE_PATENTS (관심특허 정보)](#34-favorite_patents-관심특허-정보)
  - [3.5 PATENT_IPC_SUBCLASS_MAP (특허–Subclass 매핑 정보)](#35-patent_ipc_subclass_map-특허subclass-매핑-정보)
  - [3.6 IPC_SUBCLASS_MAP (IPC Subclass 사전)](#36-ipc_subclass_map-ipc-subclass-사전)
- [4. 설계 요약](#4-설계-요약)
- [5. 주요 변경사항 (V1.1 → V1.2)](#5-주요-변경사항-v11--v12)
- [6. PostgreSQL 생성 스크립트](#6-postgresql-생성-스크립트)
- [메타](#메타)

---

## ERD 다이어그램

<img width="1221" height="866" alt="BookStore (3)" src="https://github.com/user-attachments/assets/873a6b53-ee1c-452c-b23f-a6c386c2d294" />


---

## 1. 개요

| 항목 | 내용 |
|------|------|
| 데이터베이스명 | TECHLENS_DB |
| DBMS | **PostgreSQL 14 이상** |
| 문자 인코딩 | UTF8 |
| 정규화 수준 | 제3정규형 (3NF) |
| 주요 기능 | 사용자 관리, 프리셋 저장, 관심특허 관리, IPC 코드 매핑, RefreshToken 관리 |
| 테이블 수 | **6개** |

---

## 2. 테이블 목록

| No | 테이블명 | 설명 |
|----|-----------|------|
| 1 | USERS | 사용자 계정 정보 |
| 2 | **REFRESH_TOKENS** | RefreshToken 저장 |
| 3 | PRESETS | 사용자 프리셋 정보 |
| 4 | FAVORITE_PATENTS | 관심특허 정보 |
| 5 | PATENT_IPC_SUBCLASS_MAP | 특허–Subclass 매핑 정보 |
| 6 | IPC_SUBCLASS_MAP | IPC Subclass 사전 정보 |

---

## 3. 테이블 정의서

---

### 3.1 USERS (사용자 계정 정보)

| 컬럼명 | 데이터 타입 | 제약조건 | 설명 |
|--------|--------------|-----------|------|
| USER_TBLKEY | **SERIAL** | PK | 사용자 고유 식별자 |
| EMAIL | VARCHAR(255) | UNIQUE, NOT NULL | 로그인용 이메일 |
| PASSWORD_HASH | VARCHAR(255) | NOT NULL | 해시된 비밀번호 (bcrypt) |
| ADDDATE | **TIMESTAMP** | **DEFAULT NOW()** | 계정 생성일시 (자동) |

---

### 3.2 REFRESH_TOKENS (리프레시 토큰 저장)

| 컬럼명 | 데이터 타입 | 제약조건 | 설명 |
|--------|--------------|-----------|------|
| USER_TBLKEY | INTEGER | **PK**, FK → USERS.USER_TBLKEY | 사용자 고유 식별자 |
| REFRESH_TOKEN | TEXT | NOT NULL | RefreshToken 원문 |
| EXPIRES_AT | TIMESTAMPTZ | NOT NULL | RefreshToken 만료 일시 |

**운영 규칙**
- 사용자 1명당 RefreshToken **1개만 저장**
- 로그인 시 기존 토큰 갱신(UPSERT)
- 로그아웃 시 해당 row 삭제
- AccessToken 재발급 시 RefreshToken 유효성 검사 후 발급

---

### 3.3 PRESETS (프리셋 정보)

| 컬럼명 | 데이터 타입 | 제약조건 | 설명 |
|--------|--------------|-----------|------|
| PRESET_TBLKEY | **SERIAL** | PK | 프리셋 고유 식별자 |
| USER_TBLKEY | INTEGER | FK → USERS.USER_TBLKEY, NOT NULL | 프리셋 소유 사용자 |
| PRESET_NAME | VARCHAR(255) | NOT NULL | 프리셋 이름 |
| APPLICANT | VARCHAR(255) | NOT NULL | 검색 대상 회사명 |
| START_DATE | VARCHAR(8) | NOT NULL | 검색 시작일 (YYYYMMDD) |
| END_DATE | VARCHAR(8) | NOT NULL | 검색 종료일 (YYYYMMDD) |
| DESCRIPTION | VARCHAR(500) | NULL | 프리셋 설명 |
| ADDDATE | **TIMESTAMP** | DEFAULT NOW() | 등록일시 (자동) |

**제약조건**
- FOREIGN KEY (`USER_TBLKEY`) REFERENCES `USERS`(`USER_TBLKEY`) ON DELETE CASCADE

---

### 3.4 FAVORITE_PATENTS (관심특허 정보)

| 컬럼명 | 데이터 타입 | 제약조건 | 설명 |
|--------|--------------|-----------|------|
| PATENT_TBLKEY | **SERIAL** | PK | 관심특허 고유 식별자 |
| USER_TBLKEY | INTEGER | FK → USERS.USER_TBLKEY, NOT NULL | 사용자 식별자 |
| APPLICATION_NUMBER | VARCHAR(30) | NOT NULL | 특허 출원번호 |
| INVENTION_TITLE | VARCHAR(500) | NOT NULL | 발명의 명칭 |
| APPLICANT_NAME | VARCHAR(255) | NOT NULL | 출원인명 |
| ABSTRACT | TEXT | NULL | 초록 요약 |
| APPLICATION_DATE | VARCHAR(8) | NOT NULL | 출원일 (YYYYMMDD) |
| PUBLICATION_DATE | VARCHAR(8) | NULL | 공개일 |
| REGISTER_DATE | VARCHAR(8) | NULL | 등록일 |
| REGISTER_STATUS | VARCHAR(50) | NULL | 등록 상태 |
| DRAWING_URL | VARCHAR(500) | NULL | 대표 도면 이미지 |
| ADDDATE | **TIMESTAMP** | DEFAULT NOW() | 등록일시 |

**제약조건**
- UNIQUE (`USER_TBLKEY`, `APPLICATION_NUMBER`)
- FOREIGN KEY (`USER_TBLKEY`) REFERENCES USERS(USER_TBLKEY) ON DELETE CASCADE

---

### 3.5 PATENT_IPC_SUBCLASS_MAP (특허–Subclass 매핑 정보)

| 컬럼명 | 데이터 타입 | 제약조건 | 설명 |
|--------|--------------|-----------|------|
| MAPPING_TBLKEY | **SERIAL** | PK | 매핑 고유 식별자 |
| PATENT_TBLKEY | INTEGER | FK → FAVORITE_PATENTS.PATENT_TBLKEY, NOT NULL | 특허 참조 |
| IPC_SUBCLASS | VARCHAR(10) | FK → IPC_SUBCLASS_MAP.IPC_SUBCLASS, NOT NULL | Subclass 코드 |

**제약조건**
- UNIQUE (`PATENT_TBLKEY`, `IPC_SUBCLASS`)

---

### 3.6 IPC_SUBCLASS_MAP (IPC Subclass 사전)

| 컬럼명 | 데이터 타입 | 제약조건 | 설명 |
|--------|--------------|-----------|------|
| IPC_SUBCLASS | VARCHAR(10) | PK, NOT NULL | Subclass 코드 |
| KOR_NAME | VARCHAR(255) | NOT NULL | 한글 기술명 |

---

## 4. 설계 요약

| 항목 | 내용 |
|------|------|
| JWT 정책 | RefreshToken DB 저장 방식 |
| 로그아웃 | RefreshToken 삭제 방식 |
| 유효성 검사 | AccessToken: 서명 + exp / RefreshToken: DB 조회 |
| 확장성 | 다중 디바이스 로그인 시 USER_TBLKEY → 복합 PK로 확장 가능 |
| 무결성 | FK + UNIQUE 제약 활용 |
| 정규화 | 제3정규형 유지 |

---

## 5. 주요 변경사항 (V1.1 → V1.2)

| 항목 | V1.1 | V1.2 |
|------|------|------|
| JWT_BLACKLIST | **사용** | **삭제됨** |
| RefreshToken | 없음 | **REFRESH_TOKENS 테이블 추가** |
| 로그아웃 방식 | Blacklist 삽입 | **RefreshToken 삭제 방식** |
| 테이블 수 | 6개 | 6개 |

---

## 6. PostgreSQL 생성 스크립트

```sql
-- ============================================
-- TechLens Database - PostgreSQL V1.2 (RefreshToken 적용)
-- ============================================

DROP TABLE IF EXISTS patent_ipc_subclass_map CASCADE;
DROP TABLE IF EXISTS ipc_subclass_map CASCADE;
DROP TABLE IF EXISTS favorite_patents CASCADE;
DROP TABLE IF EXISTS presets CASCADE;
DROP TABLE IF EXISTS refresh_tokens CASCADE;
DROP TABLE IF EXISTS users CASCADE;

-- 1) USERS
CREATE TABLE users (
  user_tblkey    SERIAL PRIMARY KEY,
  email          VARCHAR(255) UNIQUE NOT NULL,
  password_hash  VARCHAR(255) NOT NULL,
  adddate        TIMESTAMP DEFAULT NOW()
);

-- 2) REFRESH_TOKENS
CREATE TABLE refresh_tokens (
  user_tblkey     INTEGER PRIMARY KEY,
  refresh_token   TEXT NOT NULL,
  expires_at      TIMESTAMPTZ NOT NULL,
  FOREIGN KEY (user_tblkey) REFERENCES users(user_tblkey) ON DELETE CASCADE
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

-- 7) 인덱스
CREATE INDEX idx_users_email           ON users(email);
CREATE INDEX idx_presets_user          ON presets(user_tblkey);
CREATE INDEX idx_favorites_user        ON favorite_patents(user_tblkey);
CREATE INDEX idx_favorites_appnum      ON favorite_patents(application_number);
CREATE INDEX idx_patent_ipc_patent     ON patent_ipc_subclass_map(patent_tblkey);
CREATE INDEX idx_patent_ipc_subclass   ON patent_ipc_subclass_map(ipc_subclass);
```

```
메타

Version: 1.1

Date: 2025-11-14

작성자: 심우현 (KNU / Kicom Internship)

변경사항: RefreshToken 기반 인증 시스템 반영, JWT_BLACKLIST 제거

© 2025 TechLens Project. All rights reserved.
본 문서는 Kicom × KNU 인턴십 프로그램의 일부이며 무단 복제·배포를 금합니다.
```
