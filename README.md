# 🏫 KNU x Douzone-Kicom Internship 2025

## 프로젝트 개요

데이터 분석, 기획, 개발, 시각화까지 전 과정을 경험하고  
실제 서비스 형태로 발전시키는 것을 목표로 합니다.  
본 프로젝트는 **더존ICT그룹 Kicom 개발본부**와 **강원대학교 SW중심대학사업단**이 협력하여 진행한  
인턴십 프로젝트입니다.

---

## 리포지토리 구조 (Organization)

본 프로젝트는 하나의 Organization 내 **3개의 리포지토리**로 구성되어 있습니다.

    public-data-project/
    ├── Docs/       ← 기획서, 설계서, 분석 보고서 등 문서 저장소
    ├── Frontend/   ← React + TypeScript 기반 SPA
    └── Backend/    ← Node.js + Express 기반 API 서버

> 🧑‍💻 담당: **심우현 (Fullstack)** — 기획 · 설계 · 프론트엔드 · 백엔드 전담

---

## ⚙️ 기술 스택 (Tech Stack)

| 구분 | 기술 스택 |
|------|------------|
| **Frontend** | React, TypeScript, TailwindCSS, Zustand, Chart.js |
| **Backend** | Node.js (Express), KIPRIS Open API |
| **Database** | MariaDB |
| **Deployment** | Vercel (Frontend), Render (Backend) |
| **Data Source** | KIPRIS API (특허청 Open API) |

---

## 일정

자세한 타임테이블은 [docs/timetable.md](docs/timetable.md) 참조

---

## 팀원 및 역할

| 이름              | 역할       | 담당                            |
| ----------------- | ---------- | ------------------------------- |
| **심우현**        | Fullstack  | 전체 개발 (React UI / API / DB / 서버) |
| **박효민 선임연구원** | Mentor     | 프로젝트, 백엔드 멘토링          |
| **양태인 주임연구원** | Mentor     | 프론트엔드 멘토링               |

---

# 🧭 TechLens (가칭)

> **특허청 KIPRIS API 기반 경쟁사 R&D 동향 분석 대시보드**  
> 특허 빅데이터를 활용하여 기업의 기술집중도, R&D 활동 추이, 등록 현황을 시각화하는 웹 기반 분석 서비스입니다.  
> 프로젝트 기간: 2025.10 – Douzone-Kicom Internship  
> 개발자: **심우현 (WooHyun Sim)**, 강원대학교 컴퓨터공학과

---

## 📘 개요 (Overview)

**TechLens**는 특허청의 **KIPRIS Open API** 데이터를 기반으로  
경쟁사의 R&D 기술 동향을 실시간으로 분석·시각화하는 **Single Page Application (SPA)** 형태의 서비스입니다.  
데이터 기반의 의사결정을 돕고, 특히 **중소·중견기업의 R&D 전략 수립 지원**을 목표로 합니다.

---

## 🎯 프로젝트 배경

- **기술 경쟁력의 시대:** 기술이 곧 기업의 미래를 결정하는 핵심 자산  
- **특허 빅데이터의 가치:** 산업 트렌드와 연구개발 전략을 담은 데이터의 보고  
- **체계적 분석의 필요성:** 방대한 특허 데이터를 정량화해 기업의 기술 포트폴리오를 객관적으로 파악  

> 💡 매일 수천 건의 특허가 출원되는 현재,  
> TechLens는 “데이터를 읽는 R&D 전략”을 제공합니다.

---

## ⚠️ 문제 정의

| 문제 상황 | 설명 |
|------------|------|
| 방대한 데이터 | 매일 수천 건의 특허 데이터가 쏟아지지만 분석 역량이 부족 |
| 전통적 분석 방식 | 수작업 기반 분석으로 시간·인력 소모 과다 |
| 의사결정 리스크 | 불완전한 정보로 인한 잘못된 R&D 투자 및 기술 판단 |

➡ **해결 목표:**  
모든 기업이 데이터 기반으로 R&D 전략을 세울 수 있도록,  
특허 분석의 장벽을 낮춘 서비스를 구축합니다.

---

## 🚀 서비스 개요

| 항목 | 내용 |
|------|------|
| **서비스명** | TechLens (가칭) |
| **형태** | 웹 기반 SPA (Single Page Application) |
| **핵심 역할** | KIPRIS API를 통해 수집한 특허 데이터를 분석하고 시각화하여 R&D 인사이트 제공 |
| **대상 사용자** | 중소·중견기업, 연구기획자, 기술분석가 |

---

## 🧩 주요 기능 (Core Features)

### 1️⃣ IPC 기술 집중도 분석
- 경쟁사의 **핵심 기술 분야**를 IPC 코드 기반으로 파악  
- 상위 N개 기술 분야를 파이차트로 시각화  
- 📊 R&D 포트폴리오의 강점과 약점을 직관적으로 인식 가능

### 2️⃣ 시계열 R&D 활동 추이
- **기간별 출원 건수 변화**를 선그래프로 표현  
- R&D 활동량의 확장·축소 시점 파악  
- 📈 경쟁사의 연구 모멘텀과 투자 지속성을 예측

### 3️⃣ 특허 등록 현황 분석
- **등록/거절/보류 상태** 비율을 도넛차트로 시각화  
- R&D의 결과 효율성(‘성공률’)을 직관적으로 파악

---

## 🧠 활용 데이터 (KIPRIS Open API)

| 구분 | 설명 |
|------|------|
| **제공기관** | 특허청(KIPO) |
| **플랫폼** | KIPRIS (Korea Intellectual Property Rights Information Service) |
| **데이터 형태** | JSON (실시간 업데이트) |
| **주요 항목** | 출원번호, 출원인, IPC 코드, 출원일자, 등록상태, 요약/청구항 |
| **인증 방식** | Open API Key 기반 (무료 발급) |

---

## 🏗 시스템 아키텍처

SPA 기반으로 Frontend와 Backend가 REST API를 통해 통신합니다.

    [User] 
       ↓
    [Frontend: React] → 조건 입력
       ↓
    [Backend: Node.js] → KIPRIS API 호출 및 데이터 처리
       ↓
    [MariaDB] → 결과 캐싱 및 분석
       ↓
    [Frontend: Chart.js] → 시각화 렌더링

---

## 🧭 주요 페이지 구조

| 구분 | 페이지명 | 설명 |
|------|-----------|------|
| 1 | 홈(Home) | 초기 진입 화면 |
| 2 | 요약분석(Summary) | 핵심 분석 지표 대시보드 (특허 건수, 등록률, 월별 출원 등) |
| 3 | 특허검색(Search) | 조건 기반 특허 조회 및 상세 보기 |
| 4 | 관심특허(Favorites) | 즐겨찾기 등록/삭제 및 정렬 기능 |
| 5 | 프리셋(Presets) | 검색 조건 저장 및 재활용 |
| 6 | 도움말(Help) | KIPRIS 및 특허 용어 설명 + 하이퍼링크 제공 |

---

## 📈 추진 계획 및 향후 방향

### 🎯 최종 목표
- 3대 핵심 기능(IPC 집중도, 시계열 추이, R&D 현황)을 갖춘 동작 가능한 **웹 프로토타입** 완성  
- 특허 빅데이터의 **실무 적용 가능성** 입증 및 시연

### 🔍 향후 계획
- **특허 인용/피인용 문헌 API** 연동으로 기술 영향력 지표 확장  
- **Douzone-Kicom ERP/BI 시스템과의 연계**를 통한 R&D-경영 통합 분석 서비스로 발전

---

## 🧾 프로젝트 구조 예시

    TechLens/
    ├── frontend/
    │   ├── src/
    │   │   ├── components/
    │   │   ├── pages/
    │   │   ├── hooks/
    │   │   ├── stores/
    │   │   └── App.tsx
    │   └── package.json
    ├── backend/
    │   ├── src/
    │   │   ├── routes/
    │   │   ├── services/
    │   │   ├── utils/
    │   │   └── server.js
    │   └── package.json
    ├── docs/
    │   └── TechLens기획설계서V1.0.pdf
    └── README.md

---

## 🪪 License

This project is intended for **educational and demonstration purposes**  
as part of the *Douzone-Kicom Internship (2025)*.  
All rights for the KIPRIS API belong to the **Korean Intellectual Property Office (KIPO)**.

---

> 📎 참고 문서: [`/docs/TechLens기획설계서V1.0.pdf`](./docs/TechLens기획설계서V1.0.pdf)
