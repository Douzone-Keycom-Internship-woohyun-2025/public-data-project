# 📘 TechLens 데이터베이스 정의서 (V1.0)

본 문서는 **Kicom × KNU TechLens 프로젝트**의 데이터베이스 구조를 정의한 문서입니다.  
본 정의서는 MariaDB 10.11 이상 환경에서 작성되었으며,  
KIPRIS Open API 기반의 특허 검색 및 분석 서비스를 위한 핵심 테이블 구조를 포함합니다.

---

## 1. 개요

| 항목 | 내용 |
|------|------|
| 데이터베이스명 | TECHLENS_DB |
| DBMS | MariaDB 10.11 이상 |
| 문자 인코딩 | UTF8MB4 |
| 정규화 수준 | 제3정규형 (3NF) |
| 주요 기능 | 사용자 관리, 프리셋 저장, 관심특허 관리, IPC 코드 매핑 |
| 테이블 수 | 5개 |

---

## 2. 테이블 목록

| No | 테이블명 | 설명 |
|----|-----------|------|
| 1 | USERS | 사용자 계정 정보 |
| 2 | PRESETS | 사용자 프리셋 정보 |
| 3 | FAVORITE_PATENTS | 관심특허 정보 |
| 4 | PATENT_IPC_SUBCLASS_MAP | 특허–Subclass 매핑 정보 |
| 5 | IPC_SUBCLASS_MAP | IPC Subclass 사전 정보 |

---

## 3. 테이블 정의서

### 3.1 USERS (사용자 계정 정보)

| 컬럼명 | 데이터 타입 | 제약조건 | 설명 |
|--------|--------------|-----------|------|
| USER_ID | INT | PK, AUTO_INCREMENT | 사용자 고유 식별자 |
| EMAIL | VARCHAR(255) | UNIQUE, NOT NULL | 로그인용 이메일 |
| PASSWORD | VARCHAR(255) | NOT NULL | 해시된 비밀번호 |
| CREATED_AT | VARCHAR(17) | NOT NULL | 계정 생성일시 (YYYYMMDDHHmmssfff) |

---

### 3.2 PRESETS (프리셋 정보)

| 컬럼명 | 데이터 타입 | 제약조건 | 설명 |
|--------|--------------|-----------|------|
| PRESET_ID | INT | PK, AUTO_INCREMENT | 프리셋 고유 식별자 |
| USER_ID | INT | FK → USERS.USER_ID, NOT NULL | 프리셋 소유 사용자 |
| PRESET_NAME | VARCHAR(255) | NOT NULL | 프리셋 이름 |
| APPLICANT | VARCHAR(255) | NOT NULL | 검색 대상 회사명 |
| START_DATE | VARCHAR(8) | NOT NULL | 검색 시작일 (YYYYMMDD) |
| END_DATE | VARCHAR(8) | NOT NULL | 검색 종료일 (YYYYMMDD) |
| DESCRIPTION | VARCHAR(500) | NULL | 프리셋 설명 |
| ADDDATE | VARCHAR(17) | NOT NULL | 등록일시 (YYYYMMDDHHmmssfff) |

---

### 3.3 FAVORITE_PATENTS (관심특허 정보)

| 컬럼명 | 데이터 타입 | 제약조건 | 설명 |
|--------|--------------|-----------|------|
| PATENT_ID | INT | PK, AUTO_INCREMENT | 관심특허 고유 식별자 |
| USER_ID | INT | FK → USERS.USER_ID, NOT NULL | 사용자 식별자 |
| APPLICATION_NUMBER | VARCHAR(30) | NOT NULL | 특허 출원번호 |
| INVENTION_TITLE | VARCHAR(500) | NOT NULL | 발명의 명칭 |
| APPLICANT_NAME | VARCHAR(255) | NOT NULL | 출원인명 |
| ABSTRACT | TEXT | NULL | 초록 요약 |
| APPLICATION_DATE | VARCHAR(8) | NOT NULL | 출원일 |
| PUBLICATION_DATE | VARCHAR(8) | NULL | 공개일 |
| REGISTER_DATE | VARCHAR(8) | NULL | 등록일 |
| REGISTER_STATUS | VARCHAR(50) | NULL | 등록 상태 (등록, 거절, 소멸 등) |
| DRAWING_URL | VARCHAR(500) | NULL | 대표 도면 이미지 URL |
| ADDDATE | VARCHAR(17) | NOT NULL | 등록일시 (YYYYMMDDHHmmssfff) |

**제약조건**
- PRIMARY KEY (`PATENT_ID`)
- FOREIGN KEY (`USER_ID`) REFERENCES `USERS`(`USER_ID`)
- UNIQUE (`USER_ID`, `APPLICATION_NUMBER`)

---

### 3.4 PATENT_IPC_SUBCLASS_MAP (특허–Subclass 매핑 정보)

| 컬럼명 | 데이터 타입 | 제약조건 | 설명 |
|--------|--------------|-----------|------|
| MAPPING_ID | INT | PK, AUTO_INCREMENT | 매핑 고유 식별자 |
| PATENT_ID | INT | FK → FAVORITE_PATENTS.PATENT_ID, NOT NULL | 특허 참조 |
| IPC_SUBCLASS | VARCHAR(10) | FK → IPC_SUBCLASS_MAP.IPC_SUBCLASS, NOT NULL | IPC Subclass 코드 (예: G06Q) |

**제약조건**
- UNIQUE (`PATENT_ID`, `IPC_SUBCLASS`)

---

### 3.5 IPC_SUBCLASS_MAP (IPC Subclass 사전)

| 컬럼명 | 데이터 타입 | 제약조건 | 설명 |
|--------|--------------|-----------|------|
| IPC_SUBCLASS | VARCHAR(10) | PK, NOT NULL | Subclass 코드 (예: G06Q) |
| KOR_NAME | VARCHAR(255) | NOT NULL | 한글 기술명 |

---

## 4. 설계 요약

| 항목 | 내용 |
|------|------|
| 외래키 관계 | USERS → PRESETS, FAVORITE_PATENTS → PATENT_IPC_SUBCLASS_MAP → IPC_SUBCLASS_MAP |
| 인덱스 권장 | USER_ID, APPLICATION_NUMBER, IPC_SUBCLASS |
| 정규화 수준 | 제3정규형 (3NF) |
| 무결성 확보 | FK 및 복합 UNIQUE 제약조건으로 데이터 중복 방지 |
| 확장성 | IPC Subclass 기반 분석, 통계, 관심 패턴 확장 가능 |
| 기본값 정책 | 지정하지 않음 (비즈니스 로직에서 처리) |
| NULL 정책 | PK 및 UNIQUE, FK 제외한 모든 컬럼 NULL 허용 |
| 날짜 형식 | VARCHAR(17) 사용 (YYYYMMDDHHmmssfff) |

---

📄 **Version:** 1.0  
📅 **Date:** 2025-11-03  
👤 **작성자:** 심우현 (KNU / Kicom Internship)  
