# TechLens DB 정의서 (Enterprise Spec Ver.)

본 문서는 **KICOM x KNU TechLens 프로젝트**의 MariaDB 기반 데이터베이스 정의서를 기술합니다.  
더존비즈온 DB 표준을 준수하여 작성되었으며,  
KIPRIS Open API 기반의 특허 검색 및 관심특허 관리 서비스를 위한 구조를 정의합니다.

---

## 1️⃣ 테이블 목록 및 개요

본 시스템은 아래 5개의 주요 테이블로 구성됩니다.  
이후 각 테이블에 대한 상세 구조를 정의합니다.

| No | 테이블명 | 설명 |
|----|-----------|------|
| 1 | USERS | 사용자 계정 정보 |
| 2 | PRESETS | 검색 프리셋 관리 |
| 3 | FAVORITE_PATENTS | 관심특허 저장 |
| 4 | PATENT_IPC_MAP | 특허–IPC 매핑 관리 |
| 5 | IPC_MAP | IPC 코드 사전 (한글명 매핑용) |

---

## 2️⃣ 테이블 상세 정의

### 2.1 USERS (사용자 계정)

| 컬럼명 | 데이터 타입 | 제약조건 | 설명 |
|---------|-------------|-----------|------|
| USER_TBLKEY | INT | PK, AUTO_INCREMENT | 사용자 고유 식별자 |
| EMAIL | VARCHAR(255) | UNIQUE, NOT NULL | 로그인용 이메일 |
| PASSWORD | VARCHAR(255) | NOT NULL | 암호화된 비밀번호 |
| ADDDATE | VARCHAR(17) | NOT NULL | 생성일시 (yyyyMMddHHmmssfff) |
| MODIFYDATE | VARCHAR(17) | NULL | 수정일시 (yyyyMMddHHmmssfff) |

---

### 2.2 PRESETS (검색 프리셋)

| 컬럼명 | 데이터 타입 | 제약조건 | 설명 |
|---------|-------------|-----------|------|
| PRESET_TBLKEY | INT | PK, AUTO_INCREMENT | 프리셋 고유 식별자 |
| USER_TBLKEY | INT | FK → USERS.USER_TBLKEY, NOT NULL | 프리셋 소유 사용자 |
| PRESET_NAME | VARCHAR(255) | NOT NULL | 프리셋 이름 |
| APPLICANT | VARCHAR(255) | NOT NULL | 검색 대상 회사명 |
| START_DATE | VARCHAR(8) | NOT NULL | 검색 시작일 (YYYYMMDD) |
| END_DATE | VARCHAR(8) | NOT NULL | 검색 종료일 (YYYYMMDD) |
| DESCRIPTION | VARCHAR(255) | NULL | 프리셋 설명 |
| ADDDATE | VARCHAR(17) | NOT NULL | 생성일시 |
| MODIFYDATE | VARCHAR(17) | NULL | 수정일시 |

---

### 2.3 FAVORITE_PATENTS (관심특허)

| 컬럼명 | 데이터 타입 | 제약조건 | 설명 |
|---------|-------------|-----------|------|
| PATENT_TBLKEY | INT | PK, AUTO_INCREMENT | 관심특허 고유 식별자 |
| USER_TBLKEY | INT | FK → USERS.USER_TBLKEY, NOT NULL | 소유 사용자 |
| APPLICATION_NUMBER | VARCHAR(30) | NOT NULL | 특허 출원번호 |
| INVENTION_TITLE | VARCHAR(500) | NOT NULL | 발명의 명칭 |
| APPLICANT_NAME | VARCHAR(255) | NOT NULL | 출원인명 |
| ABSTRACT | TEXT | NULL | 초록 요약 |
| APPLICATION_DATE | VARCHAR(8) | NOT NULL | 출원일 |
| PUBLICATION_DATE | VARCHAR(8) | NULL | 공개일 |
| REGISTER_DATE | VARCHAR(8) | NULL | 등록일 |
| REGISTER_STATUS | VARCHAR(50) | NULL | 상태 (등록, 거절, 소멸 등) |
| DRAWING_URL | VARCHAR(500) | NULL | 대표 도면 이미지 URL |
| ADDDATE | VARCHAR(17) | NOT NULL | 생성일시 |
| MODIFYDATE | VARCHAR(17) | NULL | 수정일시 |

**제약조건 (Constraints)**  
- PRIMARY KEY (`PATENT_TBLKEY`)  
- FOREIGN KEY (`USER_TBLKEY`) REFERENCES `USERS`(`USER_TBLKEY`)  
- UNIQUE (`USER_TBLKEY`, `APPLICATION_NUMBER`)  
  → 동일 사용자는 동일 특허 중복 불가  
  → 다른 사용자는 동일 특허 등록 가능  

---

### 2.4 PATENT_IPC_MAP (특허–IPC 매핑)

| 컬럼명 | 데이터 타입 | 제약조건 | 설명 |
|---------|-------------|-----------|------|
| MAPPING_TBLKEY | INT | PK, AUTO_INCREMENT | 매핑 고유 식별자 |
| PATENT_TBLKEY | INT | FK → FAVORITE_PATENTS.PATENT_TBLKEY, NOT NULL | 특허 참조 |
| IPC_CODE | VARCHAR(20) | FK → IPC_MAP.IPC_CODE, NOT NULL | 세부 IPC 코드 (예: G06Q 50/06) |
| IPC_MAIN_GROUP | VARCHAR(10) | NOT NULL | 메인 그룹 코드 (예: G06Q 50) |
| ADDDATE | VARCHAR(17) | NOT NULL | 생성일시 |

**제약조건**  
- UNIQUE (`PATENT_TBLKEY`, `IPC_CODE`)  
  → 동일 특허에 동일 IPC 중복 등록 방지  

---

### 2.5 IPC_MAP (IPC 코드 사전)

| 컬럼명 | 데이터 타입 | 제약조건 | 설명 |
|---------|-------------|-----------|------|
| IPC_CODE | VARCHAR(20) | PK, NOT NULL | 세부 IPC 코드 (예: G06Q 50/06) |
| IPC_MAIN_GROUP | VARCHAR(10) | NOT NULL | 메인 그룹 코드 (예: G06Q 50) |
| KOR_NAME | VARCHAR(255) | NOT NULL | 한글 기술명 |
| ENG_NAME | VARCHAR(255) | NULL | 영문 기술명 |
| ADDDATE | VARCHAR(17) | NOT NULL | 생성일시 |

---

## 3️⃣ 설계 요약

| 구분 | 내용 |
|------|------|
| **데이터베이스** | MariaDB 10.11 이상 (UTF8MB4) |
| **지원 표준** | 더존비즈온 DB 정의서 규격 100% 준수 |
| **정규화 수준** | 3NF 완전 정규화 구조 |
| **핵심 관계** | USERS → PRESETS / FAVORITE_PATENTS, FAVORITE_PATENTS → PATENT_IPC_MAP |
| **날짜 타입** | VARCHAR(17) (yyyyMMddHHmmssfff) |
| **인덱스 권장** | USER_TBLKEY, APPLICATION_NUMBER, IPC_MAIN_GROUP |
| **데이터 무결성** | FK 및 복합 UNIQUE 제약으로 완전 보장 |
| **확장성** | IPC 통계, 기간별 관심분석, 기술 집중도 분석까지 확장 가능 |

---

📄 **Version:** 2.0 (Enterprise Spec)  
📅 **Date:** 2025-11-03  
👤 **Author:** SIM WOO-HYUN (KNU / KICOM Internship)
