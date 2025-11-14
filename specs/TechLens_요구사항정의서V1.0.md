# TechLens 요구사항정의서 (V1.1 수정본)

## 1. 개요
| 항목 | 내용 |
| :--- | :--- |
| **프로젝트명** | TechLens (테크렌즈) |
| **버전** | 1.1 (수정) |
| **작성일자** | 2025-11-03 |
| **작성자** | 심우현 (KNU / Kicom Internship) |
| **DBMS** | PostgreSQL 13 이상 |
| **Frontend** | React + TypeScript + Zustand + TailwindCSS + Chart.js |
| **Backend** | Node.js (Express) + Raw Query(pg) + xml2js |
| **배포 환경** | Frontend(Vercel) / Backend(Render) |
| **API 연동** | KIPRIS Open API (getAdvancedSearch 활용) |
| **문서 목적** | 본 문서는 TechLens 시스템의 요구사항을 정의하고 설계 및 구현의 기준을 제공하기 위함입니다. |

---

## 2. 시스템 개요
TechLens는 KIPRIS Open API 특허 데이터를 활용하여 기업의 R&D 동향을 분석·시각화하는 웹 기반 대시보드 서비스입니다.  
복잡한 특허 데이터를 쉽게 해석하여 전략적 의사결정에 활용할 수 있도록 지원합니다.

> ⚙️ 백엔드는 XML 기반 API를 xml2js로 파싱하고, PostgreSQL에는 Raw SQL Query(pg)를 통해 데이터를 저장·집계합니다.

---

## 3. 주요 기능 요약
| 구분 | 기능명 | 설명 |
| :--- | :--- | :--- |
| **핵심 분석 기능 (3)** | IPC 기술 집중도 분석 | IPC 분포 기반 기술 집중 영역 시각화 |
| | R&D 추세 분석 | 출원일 기준 월/분기별 출원량 분석 |
| | R&D 결과 분석 | 등록·거절·공개 상태 통계 분석 |
| **관리 기능 (2)** | 프리셋 관리 (CRUD) | 자주 사용하는 검색 조건 저장/재사용 |
| | 관심특허 관리 (CRD) | 즐겨찾기 특허 저장·조회·삭제 |
| **보조 기능 (2)** | IPC 사전 | IPC 코드 → 기술 설명 데이터 제공 |
| | 로그인/인증 | JWT Access Token + Refresh Token 기반 인증 |

---

## 4. 시스템 구조
| 계층 | 구성요소 | 역할 |
| :--- | :--- | :--- |
| **Frontend** | React SPA, TailwindCSS, Chart.js | UI 및 그래프 시각화 |
| **Backend** | Express.js + Raw Query(pg) + xml2js | API 처리 / 인증 / 분석 로직 |
| **Database** | PostgreSQL 13 이상 | 사용자 · 프리셋 · 관심특허 · IPC 사전 저장 |
| **External API** | KIPRIS Open API | 특허 메타데이터(XML) 수집 |
| **Deployment** | Vercel / Render | FE·BE 분리 배포 |
| **Logging** | morgan + winston | 요청/응답 로그 관리 |

---

## 5. 주요 요구사항 정의

### 5.1 기능 요구사항 (Functional Requirements)

| ID | 항목 | 요구사항 내용 |
| :-- | :-- | :-- |
| **FR-01** | 사용자 인증 | 이메일/비밀번호 기반 회원가입 및 로그인 |
| **FR-02** | 세션 유지 | JWT Access Token + Refresh Token 구조로 인증 상태 유지 |
| **FR-03** | 프리셋 생성 | 회사명, 기간, 설명 입력 후 생성 |
| **FR-04** | 프리셋 조회 | 저장된 프리셋 목록 및 상세 조회 |
| **FR-05** | 프리셋 수정·삭제 | CRUD |
| **FR-06** | 관심특허 등록 | 검색 결과에서 즐겨찾기 등록 |
| **FR-07** | 관심특허 조회·삭제 | 등록된 관심특허 관리 |
| **FR-08** | IPC 분석 | IPC 코드 기반 기술 집중도 통계 산출 |
| **FR-09** | R&D 추세 분석 | 월별/분기별 출원량 집계 |
| **FR-10** | R&D 결과 분석 | 등록·거절·공개 상태 집계 |
| **FR-11** | 시각화 | 파이/도넛/라인 차트로 분석 결과 표시 |
| **FR-12** | IPC 사전 | IPC Subclass → 기술명 조회 |
| **FR-13** | 로그 기록 | 요청 URL, 응답 코드, 처리 시간 기록 |
| **FR-14** | 오류 처리 | XML 파싱 오류, DB 오류, 인증 오류 공통 처리 |

---

### 5.2 비기능 요구사항 (Non-Functional Requirements)

| ID | 항목 | 요구사항 내용 |
| :-- | :-- | :-- |
| **NFR-01** | 보안성 | 비밀번호 해시 저장, JWT Access/Refresh Token 만료 관리 |
| **NFR-02** | 성능 | API 평균 응답속도 3초 이내 |
| **NFR-03** | 가용성 | Render 인프라 기반 무중단 운영 |
| **NFR-04** | 확장성 | IPC 사전 및 분석 로직 구조적 확장성 보장 |
| **NFR-05** | 유지보수성 | ORM 미사용 → Raw SQL 기반 구조로 쿼리 변경에 직관적으로 대응 |
| **NFR-06** | 로그 관리 | winston log rotation 적용 |
| **NFR-07** | CORS | Vercel 도메인만 허용 |
| **NFR-08** | 데이터 무결성 | FK 및 UNIQUE 제약조건 유지 |
| **NFR-09** | 호환성 | Node.js 20.x 이상, PostgreSQL 13 이상 |

---

## 6. 데이터 흐름 요약

<img width="881" height="902" alt="image" src="https://github.com/user-attachments/assets/d2e8789a-7139-45bb-9336-44b5004c575c" />

---

## 7. 시스템 아키텍처

<img width="881" height="902" alt="제목 없는 다이어그램 drawio (2)" src="https://github.com/user-attachments/assets/efd15038-480f-4ecf-8b8e-b55db8131362" />

---

## 8. 제약사항 및 고려사항
- KIPRIS는 XML만 제공 → xml2js를 통한 JSON 변환 필요  
- KIPRIS 요청 제한(초당 5회) 고려  
- IPC Subclass 매핑 누락 가능  
- Redis 등 외부 캐싱 미사용 (메모리 캐싱만 허용)  
- PostgreSQL JSON/Array 기능 활용 가능  
- KIPRIS 데이터는 상업적 재배포 금지  

---

## 9. 향후 확장 계획
| 항목 | 설명 |
| :--- | :--- |
| 경쟁사 비교 | 두 기업 IPC 분포 비교 분석 |
| 국가별 분석 | 출원 국가별 기술 비중 분석 |
| AI IPC 자동 분류 | 신규 특허 IPC 자동 분류 모델 |
| 프리셋 공유 | 사용자 간 프리셋 공유 기능 |

---

**Version:** 1.1 (수정)  
**Date:** 2025-11-03  
**Author:** 심우현 (KNU / Kicom Internship)

