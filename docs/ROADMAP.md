# ROADMAP - NexusPoint 구현 로드맵

## 전체 개요

| 순서 | 기능 | 예상 일수 | 상태 |
|------|------|----------|------|
| 1 | 프로젝트 셋업 + 인증 | 1일 | ⏳ |
| 2 | 프로필 CRUD + 슬러그 | 1일 | ⏳ |
| 3 | 경력/링크 관리 | 1일 | ⏳ |
| 4 | AI 스토리텔링 (Claude 연동) | 2일 | ⏳ |
| 5 | 공개 개인 페이지 (/u/{slug}) | 2일 | ⏳ |
| 6 | 명함 디자인 에디터 | 3일 | ⏳ |
| 7 | QR 코드 생성 | 0.5일 | ⏳ |
| 8 | 결제 연동 (구독 + 단건) | 2일 | ⏳ |
| 9 | 명함 인쇄 주문 | 2일 | ⏳ |
| 10 | 조회 통계 | 1일 | ⏳ |
| 11 | 랜딩 페이지 | 1일 | ⏳ |
| **합계** | | **~16일** | |

---

## 상세 구현 계획

### Step 1: 프로젝트 셋업 + 인증 (Day 1)

**목표**: 개발 환경 구축 및 인증 플로우 완성

**작업 목록**:
- [ ] Next.js 16 프로젝트 생성 (`pnpm create next-app`)
- [ ] TypeScript, Tailwind CSS 4, shadcn/ui 설정
- [ ] Supabase 프로젝트 생성 + 환경 변수 설정
- [ ] Prisma 설치 + schema.prisma 작성 (User, Profile)
- [ ] Supabase Auth 연동 (Google, Kakao OAuth)
- [ ] Next.js Middleware 작성 (보호된 라우트)
- [ ] 로그인/회원가입 페이지 UI
- [ ] 콜백 핸들러 (`/auth/callback`)

**완료 기준**:
- Google/Kakao OAuth 로그인 성공
- 로그인 후 `/dashboard`로 리다이렉트
- 로그아웃 기능 동작

---

### Step 2: 프로필 CRUD + 슬러그 (Day 2)

**목표**: 기본 프로필 생성/수정 + 공개 URL 설정

**작업 목록**:
- [ ] Profile 테이블 마이그레이션
- [ ] `GET/PATCH /api/profile` 구현
- [ ] `GET/PATCH /api/profile/slug` + 중복 체크
- [ ] 프로필 편집 페이지 UI (`/dashboard/profile`)
- [ ] 프로필 사진 업로드 (Supabase Storage)
- [ ] 슬러그 자동 생성 (가입 시 이름 기반)

**완료 기준**:
- 프로필 정보 저장/수정 동작
- 슬러그 중복 체크 + 변경 동작
- 프로필 사진 업로드 동작

---

### Step 3: 경력/링크 관리 (Day 3)

**목표**: 경력 사항 및 SNS 링크 CRUD

**작업 목록**:
- [ ] CareerEntry, ProfileLink 마이그레이션
- [ ] 경력 CRUD API + 순서 변경
- [ ] 링크 CRUD API + 순서 변경
- [ ] 경력 관리 UI (드래그앤드롭 정렬)
- [ ] 링크 관리 UI (링크 타입별 아이콘)

**완료 기준**:
- 경력 추가/수정/삭제/정렬 동작
- 링크 추가/삭제/정렬 동작

---

### Step 4: AI 스토리텔링 (Day 4-5)

**목표**: Claude AI 온보딩 인터뷰 + 스토리 생성

**작업 목록**:
- [ ] AIStory 마이그레이션
- [ ] Claude API 클라이언트 구현 (`src/lib/ai/client.ts`)
- [ ] 온보딩 3단계 UI (멀티스텝 폼)
- [ ] `POST /api/ai/start-interview` (스트리밍)
- [ ] 스토리 결과 표시 UI (섹션별)
- [ ] `POST /api/ai/regenerate-section`
- [ ] 인라인 편집 기능
- [ ] 버전 히스토리 UI
- [ ] Rate Limiting (Free: 월 3회)
- [ ] 경력 AI 향상 `POST /api/ai/enhance-career/{id}`

**완료 기준**:
- 온보딩 입력 → AI 스토리 생성 완료
- 섹션별 재생성 동작
- Free 플랜 3회 제한 동작

---

### Step 5: 공개 개인 페이지 (Day 6-7)

**목표**: `/u/{slug}` 공개 프로필 페이지

**작업 목록**:
- [ ] `/u/[slug]/page.tsx` ISR 구현 (revalidate: 60)
- [ ] 프로필 섹션 컴포넌트 (헤더, 경력, 링크, AI 스토리)
- [ ] 테마 시스템 구현 (modern, minimal, creative)
- [ ] OpenGraph 이미지 동적 생성 (`opengraph-image.tsx`)
- [ ] vCard 다운로드 API (`GET /api/public/vcard/{slug}`)
- [ ] `POST /api/public/view` (조회 기록)
- [ ] QR 리다이렉트 핸들러 (`/q/[code]`)
- [ ] "Powered by NexusPoint" 배지 (Free)
- [ ] 비밀번호 보호 (Premium+)

**완료 기준**:
- 공개 URL에서 프로필 페이지 표시
- ISR로 60초 캐시 동작
- OG 이미지 SNS 공유 시 표시
- QR 스캔 → 프로필 리다이렉트

---

### Step 6: 명함 디자인 에디터 (Day 8-10)

**목표**: Fabric.js 기반 명함 에디터

**작업 목록**:
- [ ] CardDesign 마이그레이션
- [ ] Fabric.js 에디터 컴포넌트 (`/dashboard/cards/[id]/editor`)
- [ ] 템플릿 시스템 (기본 3종 JSON 작성)
- [ ] 텍스트/이미지/QR 레이어 편집
- [ ] 색상/폰트 커스터마이징 패널
- [ ] 앞/뒤 전환 (양면 디자인)
- [ ] 실시간 저장 (자동 저장 + 수동 저장)
- [ ] 썸네일 이미지 생성 + Supabase Storage 저장
- [ ] 인쇄용 PDF 생성 (300 DPI, PDFKit)
- [ ] `GET /api/cards/templates` (템플릿 목록)

**완료 기준**:
- 템플릿 선택 → 에디터 열기
- 텍스트/이미지 수정 저장
- PNG 미리보기 + 인쇄용 PDF 생성

---

### Step 7: QR 코드 생성 (Day 10.5)

**목표**: 프로필 URL QR 코드 생성

**작업 목록**:
- [ ] `GET /api/qr/{profileSlug}` 구현
- [ ] QR 커스터마이징 (색상, 크기, 로고 - Pro+)
- [ ] 에디터에 QR 자동 삽입
- [ ] QR 다운로드 (PNG/SVG)

**완료 기준**:
- QR 생성 → 스캔 → 프로필 이동

---

### Step 8: 결제 연동 (Day 11-12)

**목표**: 구독 결제 + 단건 결제 (PortOne)

**작업 목록**:
- [ ] Subscription 마이그레이션
- [ ] PortOne SDK 연동
- [ ] 구독 결제 플로우 (월간/연간)
- [ ] 구독 상태 미들웨어 (플랜별 기능 제한)
- [ ] 결제 웹훅 처리 (`POST /api/payment/webhook`)
- [ ] 구독 관리 UI (`/dashboard/settings`)
- [ ] 업그레이드/다운그레이드/해지 플로우

**완료 기준**:
- Pro 구독 결제 → 기능 해제
- 구독 해지 → 다음 결제일에 Free 전환

---

### Step 9: 명함 인쇄 주문 (Day 13-14)

**목표**: 명함 인쇄 주문 플로우

**작업 목록**:
- [ ] PrintOrder, PrintOrderItem 마이그레이션
- [ ] 주문 생성 API + 가격 계산
- [ ] 인쇄 주문 UI (재질, 수량, 후가공 선택)
- [ ] 배송지 입력 폼
- [ ] 결제 연동 (PortOne 단건)
- [ ] 주문 확인 이메일 (Resend)
- [ ] 주문 내역 페이지 (`/dashboard/orders`)
- [ ] 인쇄 파트너 API 연동 (또는 수동 처리 임시)

**완료 기준**:
- 명함 선택 → 주문 → 결제 완료
- 주문 확인 이메일 수신
- 주문 내역 조회

---

### Step 10: 조회 통계 (Day 15)

**목표**: 프로필 방문 통계 대시보드

**작업 목록**:
- [ ] PageView 마이그레이션
- [ ] Redis를 이용한 실시간 조회수 캐싱
- [ ] 통계 집계 API (overview, timeline, sources)
- [ ] 통계 대시보드 UI (`/dashboard/analytics`)
- [ ] 차트 컴포넌트 (Recharts)

**완료 기준**:
- QR 스캔/직접 방문 시 조회 기록
- 대시보드에서 소스별/기간별 통계 조회

---

### Step 11: 랜딩 페이지 (Day 16)

**목표**: 마케팅 랜딩 페이지

**작업 목록**:
- [ ] 히어로 섹션 (핵심 가치 제안 + CTA)
- [ ] 기능 소개 섹션 (AI, 명함, 프로필)
- [ ] Before/After AI 데모 섹션
- [ ] 가격 비교 섹션
- [ ] 사용자 사례/후기 섹션 (목업)
- [ ] FAQ 섹션
- [ ] 가격 페이지 (`/pricing`)
- [ ] SEO 최적화 (메타 태그, 구조화 데이터)

**완료 기준**:
- 랜딩 페이지 완성 + CTA 동작
- Core Web Vitals LCP < 2.5초

---

## Post-MVP 로드맵 (v1.1 ~)

### 기능 확장
- [ ] 팀 협업 (Business 플랜 완전 구현)
- [ ] 커스텀 도메인 자동 연결 (Vercel Domains API)
- [ ] 다국어 지원 (영문, 일본어)
- [ ] 명함 NFC 태그 연동
- [ ] LinkedIn 임포트 (경력 자동 가져오기)
- [ ] AI 인터뷰 봇 (채팅 형식 온보딩)
- [ ] PDF 이력서 내보내기
- [ ] 팀 인트로 페이지 (팀 멤버 프로필 모음)

### 마케팅/성장
- [ ] "Powered by NexusPoint" 배지 클릭 통계
- [ ] 레퍼럴 프로그램 구현
- [ ] Before/After AI 변환 공유 기능
- [ ] 명함 디자인 갤러리/커뮤니티

### B2B
- [ ] 기업용 API
- [ ] 화이트라벨 솔루션
- [ ] SAML SSO 지원
- [ ] 대용량 명함 일괄 생성 (CSV 임포트)

---

## 기술 부채 관리

| 항목 | 우선순위 | 설명 |
|------|---------|------|
| 테스트 커버리지 | 높음 | MVP 후 핵심 로직 단위 테스트 추가 |
| 에러 모니터링 | 중간 | Sentry 연동 |
| 성능 모니터링 | 중간 | Vercel Analytics, Core Web Vitals |
| 접근성 (a11y) | 중간 | WCAG 2.1 AA 준수 |
| i18n | 낮음 | v1.1에서 영문 지원 |
| 문서화 | 낮음 | Storybook, API 문서 자동화 |
