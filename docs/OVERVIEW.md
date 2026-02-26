# NexusPoint — 프로젝트 전체 개요

> **"모든 사람이 30초 만에 자신만의 프리미엄 명함과 홈페이지를 가진다"**

NexusPoint는 **디지털 프로필 + 물리적 명함 + AI 콘텐츠 생성**을 하나로 묶은 개인 브랜딩 플랫폼입니다.
이 문서는 7개 명세 문서의 핵심 내용을 한 곳에서 파악할 수 있도록 통합 정리한 온보딩 가이드입니다.

---

## 목차

1. [서비스 소개](#1-서비스-소개)
2. [사용자 여정](#2-사용자-여정)
3. [기술 아키텍처](#3-기술-아키텍처)
4. [데이터 모델](#4-데이터-모델)
5. [API 구조](#5-api-구조)
6. [AI 스토리텔링 엔진](#6-ai-스토리텔링-엔진)
7. [비즈니스 모델](#7-비즈니스-모델)
8. [구현 로드맵](#8-구현-로드맵)
9. [문서 가이드](#9-문서-가이드)

---

## 1. 서비스 소개

### 핵심 기능 4가지

| # | 기능 | 설명 |
|---|------|------|
| 1 | **AI 커리어 스토리텔링** | Claude AI가 평범한 경력을 전문적인 브랜딩 문구로 변환 |
| 2 | **개인 프로필 홈페이지** | `nexuspoint.me/u/{slug}` — QR 스캔 시 연결되는 개인 랜딩 페이지 |
| 3 | **명함 디자인 에디터** | Fabric.js 기반 템플릿 편집기 + QR 자동 삽입 + 인쇄용 PDF 출력 |
| 4 | **명함 인쇄 주문** | 재질·수량·후가공 선택 → 원클릭 주문 → 5~7일 배송 |

### 타겟 사용자
- 개인 브랜딩이 필요한 **직장인·프리랜서·1인 크리에이터**
- 통일된 명함이 필요한 **스타트업 팀 (Business 플랜)**
- 취업 준비를 위한 **대학생·취업 준비생**
- 명함이 핵심 영업 도구인 **부동산·보험·금융 업종**

### 경쟁사 대비 차별화

| 서비스 | 강점 | NexusPoint 차별화 |
|--------|------|-------------------|
| 리멤버 | 국내 명함 스캔 1위 | AI 생성 + 실물 인쇄 |
| LinkedIn | 전문가 네트워크 | 가벼운 명함 컨셉 |
| Linktree | SNS 링크 모음 | 명함 + 전문 프로필 통합 |
| Blinq / HiHello | 디지털 명함 | AI 스토리텔링 + 인쇄 |

---

## 2. 사용자 여정

```
1. 가입 (Google / Kakao / Naver OAuth)
        ↓
2. AI 온보딩 인터뷰 (3단계, 약 3분)
   ├─ 기본 정보 (이름, 직함, 회사, 경력)
   ├─ 커리어 스토리 (성과, 강점, 도전 경험)
   └─ 브랜딩 취향 (톤, 타겟 인상, 피할 표현)
        ↓
3. AI 스토리텔링 결과 확인 및 편집
   ├─ headline / tagline / bio 자동 생성
   ├─ 경력별 향상된 설명
   └─ 섹션별 재생성 / 인라인 편집 가능
        ↓
4. 프로필 홈페이지 커스터마이징
   ├─ 테마 선택 (modern / minimal / creative)
   ├─ 섹션 순서 조절
   └─ SNS / 포트폴리오 링크 + 프로필 사진
        ↓
5. 명함 디자인
   ├─ 템플릿 선택 (무료 3종, Pro 20종+)
   ├─ 세부 편집 (텍스트, 색상, 폰트)
   └─ QR 코드 자동 연결 (프로필 URL)
        ↓
6. QR 연결 확인 + 공유
   ├─ QR 스캔 → 개인 페이지 이동 확인
   └─ 링크 공유 / vCard 다운로드
        ↓
7. 인쇄 주문 (선택)
   └─ 재질 · 수량 · 후가공 → 결제 → 배송
```

---

## 3. 기술 아키텍처

### 기술 스택 요약

| 레이어 | 기술 |
|--------|------|
| **프레임워크** | Next.js 16 (App Router) + TypeScript 5 |
| **스타일링** | Tailwind CSS 4 + shadcn/ui |
| **상태 관리** | Zustand 5 (클라이언트) + React Query 5 (서버) |
| **명함 에디터** | Fabric.js 6 |
| **ORM / DB** | Prisma 7 + Supabase PostgreSQL |
| **인증** | Supabase Auth (Google · Kakao · Naver OAuth) |
| **파일 저장** | Supabase Storage |
| **캐싱 / Rate Limit** | Upstash Redis |
| **AI** | Anthropic Claude Model |
| **결제** | PortOne / TossPayments |
| **이메일** | Resend |
| **배포** | Vercel (Edge Network) |

### 시스템 다이어그램

```
클라이언트 (Browser / QR Scanner)
        │ HTTPS
Vercel Edge Network (CDN + Middleware: Auth, Rate Limit)
        │
Next.js App (App Router RSC + API Routes)
        │
   ┌────┼─────────────────┐
   │    │                 │
Supabase  Upstash Redis  Anthropic Claude
(DB+Auth  (캐싱/RL)      (AI API)
+Storage)
   │
PortOne / Resend / Print Partner API
```

### 라우팅 전략

| 전략 | 적용 페이지 | 이유 |
|------|-----------|------|
| **SSG** | 마케팅 페이지 (랜딩, 가격, 사례) | 정적 콘텐츠, 빠른 로딩 |
| **ISR** | `/u/[slug]` 공개 프로필 | revalidate 60초, 빠른 첫 로딩 |
| **SSR** | 대시보드 | 인증 후 개인화 데이터 |
| **CSR** | 명함 에디터 | Fabric.js 순수 클라이언트 |

---

## 4. 데이터 모델

### 핵심 9개 엔티티 관계도

```
User (1) ────────────── (1) Profile
User (1) ────────────── (1) Subscription
User (1) ────────────── (N) PrintOrder

Profile (1) ──────────── (N) CareerEntry      ← 경력 항목
Profile (1) ──────────── (N) ProfileLink      ← SNS/링크
Profile (1) ──────────── (N) AIStory          ← AI 스토리 버전
Profile (1) ──────────── (N) CardDesign       ← 명함 디자인
Profile (1) ──────────── (N) PageView         ← 방문 통계

CardDesign (1) ──────── (N) PrintOrderItem
PrintOrder (1) ──────── (N) PrintOrderItem
```

### 주요 엔티티 설명

| 엔티티 | 역할 | 핵심 필드 |
|--------|------|---------|
| `User` | 인증 사용자 | supabaseId, email |
| `Profile` | 공개 프로필 | slug, headline, tagline, theme |
| `CareerEntry` | 경력 사항 | originalTitle, enhancedTitle (AI), type |
| `ProfileLink` | 외부 링크 | type(GITHUB/LINKEDIN 등), url |
| `AIStory` | AI 생성 스토리 | version, tone, headline~valueProposition |
| `CardDesign` | 명함 디자인 | templateId, frontDesignJson, printPdfUrl |
| `PrintOrder` | 인쇄 주문 | orderNumber, status, totalAmount |
| `Subscription` | 구독 상태 | tier(FREE/PRO/PREMIUM/BUSINESS), status |
| `PageView` | 조회 분석 | source(QR_SCAN/DIRECT_LINK 등), ipHash |

---

## 5. API 구조

**Base URL**: `https://nexuspoint.me/api`
**인증**: Supabase JWT (`Authorization: Bearer <token>` 또는 httpOnly 쿠키)
**응답 형식**: `{ data: T, error: string | null }`

### 10개 도메인별 주요 엔드포인트

| 도메인 | 주요 엔드포인트 | 설명 |
|--------|--------------|------|
| **Profile** | `GET/PATCH /api/profile` | 프로필 조회/수정 |
| | `PATCH /api/profile/slug` | 슬러그 변경 |
| **Career** | `POST /api/career` | 경력 추가 |
| | `PATCH /api/career/reorder` | 순서 변경 |
| **Links** | `POST /api/links` | 링크 추가 |
| **AI** | `POST /api/ai/start-interview` | 스토리 생성 (Rate: 5회/분) |
| | `POST /api/ai/regenerate-section` | 섹션별 재생성 |
| | `POST /api/ai/enhance-career/{id}` | 경력 AI 향상 |
| **Cards** | `POST /api/cards` | 명함 생성 |
| | `POST /api/cards/{id}/generate-images` | PNG + PDF 생성 |
| | `GET /api/cards/templates` | 템플릿 목록 |
| **QR** | `GET /api/qr/{slug}` | QR 이미지 생성 |
| **Orders** | `POST /api/orders` | 인쇄 주문 생성 |
| | `GET /api/orders/price-calc` | 가격 계산 |
| **Payment** | `POST /api/payment/checkout` | 결제 세션 생성 |
| | `POST /api/payment/webhook` | PortOne 웹훅 수신 |
| **Public** | `GET /api/public/profile/{slug}` | 공개 프로필 조회 |
| | `POST /api/public/view` | 조회 기록 |
| | `GET /api/public/vcard/{slug}` | vCard 다운로드 |
| **Analytics** | `GET /api/analytics/overview` | 통계 요약 |
| | `GET /api/analytics/timeline` | 기간별 타임라인 |

### Rate Limit 요약

| 엔드포인트 | 제한 | 기준 |
|-----------|------|------|
| AI API | 5회/분 | IP + User ID |
| 공개 조회 기록 | 30회/분 | IP |
| 결제 | 10회/분 | User ID |

---

## 6. AI 스토리텔링 엔진

### 온보딩 3단계 흐름

```
1단계 (30초) — 기본 정보
  이름 · 직함 · 업종 · 경력 기간 · 소속

2단계 (2분) — 커리어 스토리
  자랑스러운 성과 3가지 · 핵심 강점 · 극복 경험

3단계 (1분) — 브랜딩 취향
  타겟 독자 · 원하는 인상 · 피할 표현
```

### Before → After 변환 예시

| 직업 | Before (원본) | After (AI 향상) |
|------|--------------|-----------------|
| 배관공 | "배관 설치 및 수리" | "산업 인프라 설비 전문 엔지니어 \| 유체 시스템을 설계하고 최적화" |
| 바리스타 | "카페에서 커피 만들기" | "스페셜티 커피 큐레이터 \| 원두 산지부터 추출까지 감각적 경험 설계" |
| 교사 | "수학 가르치는 교사" | "수리적 사고력 개발 전문가 \| 15년간 2,000+ 학생의 잠재력 발굴" |
| React 개발자 | "React로 웹 개발" | "사용자 경험 중심 프론트엔드 아키텍트 \| 1초 이하 인터랙션 경험 설계" |

### 톤 5종

| 톤 | 특징 | 예시 |
|----|------|------|
| **Professional** | 격식체, 수치 중심 | "ARR $2M 달성에 기여한 5년 경력 엔지니어" |
| **Creative** | 은유, 생동감 | "코드는 캔버스, 아이디어는 물감 — 디지털 경험을 그립니다" |
| **Warm** | 사람 중심, 진정성 | "사람과 기술 사이의 다리를 놓는 것이 제가 가장 잘하는 일입니다" |
| **Bold** | 직설적, 자신감 | "결과를 만들어냅니다. 방법은 나중에 설명하겠습니다." |
| **Global** | 간결, 글로벌 보편 | "Full-stack engineer crafting seamless digital experiences" |

### AI 생성 항목

| 항목 | 길이 | 용도 |
|------|------|------|
| `headline` | 최대 50자 | 프로필 제목 |
| `tagline` | 최대 80자 | 인상적인 부제목 |
| `bioShort` | 최대 150자 | 명함·SNS용 소개 |
| `bioLong` | 3-4문장 | 홈페이지용 소개 |
| `careerSummary` | 3-5 bullet | 핵심 성과 리스트 |
| `valueProposition` | 2-3문장 | 가치 선언문 |

---

## 7. 비즈니스 모델

### 요금제 비교표

| | Free | Pro | Premium | Business |
|--|------|-----|---------|----------|
| **가격** | 무료 | 9,900원/월 | 19,900원/월 | 49,900원/월 (5인) |
| AI 스토리텔링 | 월 3회 | 무제한 | 무제한 | 무제한 |
| 명함 템플릿 | 3종 | 20종+ | 프리미엄 포함 | 프리미엄 포함 |
| QR 커스터마이징 | ✗ | ✓ | ✓ | ✓ |
| 상세 통계 | ✗ | ✓ | ✓ | ✓ |
| 커스텀 도메인 | ✗ | ✗ | 최대 3개 | 최대 3개 |
| 인쇄 할인 | ✗ | ✗ | 10% | 10% |
| 팀 관리 | ✗ | ✗ | ✗ | ✓ |
| 화이트라벨 | ✗ | ✗ | ✗ | ✓ |

### Unit Economics (Pro 기준)

```
평균 구독 유지: 24개월
월 ARPU: 9,900원(구독) + 10,000원(명함 주문) = 19,900원
LTV: 19,900 × 24 = 477,600원
순 LTV (마진 60%): 286,560원
CAC (유료 채널 평균): ~10,000원

LTV / CAC = 28.6x  ✅ (목표: 3x 이상)
```

### 바이럴 루프

```
명함 제작 → QR 인쇄 → 명함 배포
     ↑                      ↓
 신규 가입              QR 스캔
     ↑                      ↓
 레퍼럴 클릭          프로필 페이지 방문
     ↑                      ↓
 가입 페이지    "Powered by NexusPoint" 배지 클릭
     └──────────────────────┘
```

### Year 1 수익 예측

| 분기 | 유료 구독자 | MRR | 월 인쇄 수익 |
|------|-----------|-----|------------|
| Q1 | 25명 | 250,000원 | 450,000원 |
| Q2 | 160명 | 1,600,000원 | 2,400,000원 |
| Q3 | 700명 | 7,000,000원 | 10,500,000원 |
| Q4 | 1,800명 | 18,000,000원 | 27,000,000원 |

> **Year 1 총 수익 예상**: ~3.2억원 | 손익분기점: Q3 말 예상

---

## 8. 구현 로드맵

### 11단계 16일 계획

| 순서 | 기능 | 일수 | 핵심 내용 |
|------|------|------|---------|
| **1** | 프로젝트 셋업 + 인증 | 1일 | Next.js 16, Supabase Auth, Google/Kakao OAuth |
| **2** | 프로필 CRUD + 슬러그 | 1일 | 프로필 생성/수정, 슬러그 중복 체크 |
| **3** | 경력/링크 관리 | 1일 | CareerEntry·ProfileLink CRUD + 순서 변경 |
| **4** | AI 스토리텔링 | 2일 | Claude 연동, 온보딩 3단계, Rate Limit |
| **5** | 공개 개인 페이지 | 2일 | ISR `/u/[slug]`, OG 이미지, vCard |
| **6** | 명함 디자인 에디터 | 3일 | Fabric.js, 템플릿, PDF 출력 |
| **7** | QR 코드 생성 | 0.5일 | QR 커스터마이징, 에디터 자동 삽입 |
| **8** | 결제 연동 | 2일 | PortOne 구독·단건 결제, 플랜별 기능 제한 |
| **9** | 명함 인쇄 주문 | 2일 | 주문 플로우, 배송지, 확인 이메일 |
| **10** | 조회 통계 | 1일 | Redis 캐싱, 소스별/기간별 차트 |
| **11** | 랜딩 페이지 | 1일 | 히어로, 기능 소개, 가격, SEO |
| **합계** | | **~16일** | |

### Post-MVP 주요 계획
- LinkedIn 임포트 (경력 자동 가져오기)
- 커스텀 도메인 자동 연결 (Vercel Domains API)
- 명함 NFC 태그 연동
- 팀 인트로 페이지 (Business 플랜 완전 구현)
- 다국어 지원 (영문, 일본어)

---

## 9. 문서 가이드

프로젝트 `docs/` 폴더의 각 명세 문서 역할과 참조 가이드입니다.

| 파일 | 역할 | 참조 시점 |
|------|------|---------|
| [PRD.md](./PRD.md) | 제품 요구사항: 비전, 핵심 기능, 요금제, KPI | 기획·기능 범위 확인 시 |
| [TECH-SPEC.md](./TECH-SPEC.md) | 기술 아키텍처: 스택, 디렉토리 구조, 배포 설정 | 개발 환경 셋업·구조 파악 시 |
| [DATA-MODEL.md](./DATA-MODEL.md) | 데이터 모델: Prisma 스키마 전체 + 주요 쿼리 패턴 | DB 작업·관계 파악 시 |
| [API-SPEC.md](./API-SPEC.md) | API 명세: 10개 도메인 엔드포인트 + 에러 코드 | API 개발·프론트 연동 시 |
| [AI-ENGINE.md](./AI-ENGINE.md) | AI 엔진 설계: 프롬프트, 편집 UX, 비용 분석 | AI 기능 개발 시 |
| [BUSINESS-MODEL.md](./BUSINESS-MODEL.md) | 비즈니스 모델: 요금제, Unit Economics, 성장 전략 | 기획·투자자 설명 시 |
| [ROADMAP.md](./ROADMAP.md) | 구현 로드맵: 단계별 작업 목록 + Post-MVP | 개발 진행 관리 시 |
| **OVERVIEW.md** (이 파일) | 전체 통합 요약: 처음 합류하는 팀원·파트너용 | 프로젝트 첫 파악 시 |

---

## 빠른 시작 체크리스트

프로젝트에 새로 합류했다면 이 순서로 읽는 것을 권장합니다.

- [ ] 이 문서 (`OVERVIEW.md`) — 전체 컨텍스트 파악
- [ ] `PRD.md` — 제품 비전과 기능 범위 확인
- [ ] `TECH-SPEC.md` — 기술 스택과 디렉토리 구조 파악
- [ ] `DATA-MODEL.md` — Prisma 스키마 숙지
- [ ] `ROADMAP.md` — 현재 진행 단계 확인 후 작업 시작

---

*최종 업데이트: 2026-02-26 | 상세 내용은 각 docs/*.md 파일을 참조하세요.*

