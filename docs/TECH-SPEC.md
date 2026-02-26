# TECH-SPEC - NexusPoint 기술 아키텍처 명세

## 기술 스택

### Frontend
| 기술 | 버전 | 용도 |
|------|------|------|
| Next.js | 16 (App Router) | 프레임워크, SSG/ISR/SSR |
| TypeScript | 5+ | 정적 타입 |
| Tailwind CSS | 4 | 스타일링 |
| shadcn/ui | latest | UI 컴포넌트 라이브러리 |
| Zustand | 5 | 클라이언트 상태 관리 |
| React Query | 5 | 서버 상태 관리 + 캐싱 |
| Fabric.js | 6 | 명함 에디터 캔버스 |
| qrcode.react | 4 | 클라이언트 QR 렌더링 |
| Framer Motion | 11 | 애니메이션 |

### Backend
| 기술 | 버전 | 용도 |
|------|------|------|
| Next.js API Routes | 16 | REST API |
| Prisma | 7 | ORM |
| Zod | 3 | 입력 유효성 검사 |
| node-qrcode | 1.5 | 서버 QR 생성 |
| sharp | 0.33 | 이미지 처리 |
| PDFKit | 0.15 | 인쇄용 PDF 생성 |

### 인프라 / 서드파티
| 서비스 | 용도 |
|--------|------|
| Supabase PostgreSQL | 메인 데이터베이스 |
| Supabase Auth | 인증 (Google, Kakao, Naver) |
| Supabase Storage | 파일 저장 (이미지, PDF, QR) |
| Upstash Redis | Rate Limiting, 조회수 캐싱 |
| Anthropic Claude | AI 스토리텔링 (claude-sonnet-4-20250514) |
| PortOne / TossPayments | 결제 처리 |
| Resend | 트랜잭션 이메일 |
| Vercel | 배포 (Edge Network) |

---

## 시스템 아키텍처

```
┌─────────────────────────────────────────────────────────────┐
│                         CLIENTS                              │
│   Browser (Next.js)  │  Mobile Browser  │  QR Scanner       │
└─────────────┬───────────────────────────────────────────────┘
              │ HTTPS
┌─────────────▼───────────────────────────────────────────────┐
│                    VERCEL EDGE NETWORK                        │
│   CDN + Edge Middleware (Auth, Rate Limit, A/B)              │
└─────────────┬───────────────────────────────────────────────┘
              │
┌─────────────▼───────────────────────────────────────────────┐
│                    NEXT.JS APPLICATION                        │
│                                                              │
│  ┌─────────────────┐  ┌────────────────┐  ┌──────────────┐ │
│  │   App Router     │  │   API Routes   │  │  Middleware  │ │
│  │   (RSC + ISR)    │  │   (Edge/Node)  │  │  (Auth+RL)  │ │
│  └─────────────────┘  └────────┬───────┘  └──────────────┘ │
└────────────────────────────────┼────────────────────────────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                        │                        │
┌───────▼──────┐  ┌─────────────▼──────┐  ┌─────────────▼─────┐
│   Supabase   │  │   Upstash Redis    │  │  Anthropic Claude  │
│  PostgreSQL  │  │  (Cache / RL)      │  │  (AI API)          │
│  Auth        │  └────────────────────┘  └───────────────────┘
│  Storage     │
└──────────────┘
        │
┌───────▼──────┐  ┌─────────────────────┐  ┌──────────────────┐
│   PortOne /  │  │   Resend            │  │  Print Partner   │
│  TossPayments│  │   (Email)           │  │  API             │
└──────────────┘  └─────────────────────┘  └──────────────────┘
```

---

## 프로젝트 디렉토리 구조

```
nexuspoint/
├── prisma/
│   ├── schema.prisma           # DB 스키마 (Prisma)
│   └── migrations/             # 마이그레이션 파일들
├── public/
│   ├── templates/              # 명함 템플릿 이미지
│   ├── fonts/                  # 커스텀 폰트
│   └── icons/                  # SVG 아이콘
├── src/
│   ├── app/
│   │   ├── (auth)/             # 인증 라우트 그룹
│   │   │   ├── login/          # 로그인 페이지
│   │   │   ├── signup/         # 회원가입 페이지
│   │   │   └── callback/       # OAuth 콜백
│   │   ├── (dashboard)/        # 대시보드 라우트 그룹
│   │   │   ├── layout.tsx      # 대시보드 레이아웃 (사이드바)
│   │   │   ├── page.tsx        # 대시보드 홈
│   │   │   ├── profile/        # 프로필 편집
│   │   │   │   ├── page.tsx
│   │   │   │   └── edit/
│   │   │   ├── ai-story/       # AI 스토리텔링
│   │   │   │   ├── page.tsx    # 스토리 목록
│   │   │   │   ├── onboard/    # 3단계 온보딩
│   │   │   │   └── [id]/       # 스토리 상세/편집
│   │   │   ├── cards/          # 명함 관리
│   │   │   │   ├── page.tsx    # 명함 목록
│   │   │   │   ├── new/        # 명함 생성
│   │   │   │   └── [id]/
│   │   │   │       ├── page.tsx    # 명함 상세
│   │   │   │       └── editor/     # 명함 에디터
│   │   │   ├── orders/         # 주문 내역
│   │   │   ├── analytics/      # 방문 통계
│   │   │   └── settings/       # 설정/구독
│   │   ├── u/[slug]/           # 공개 개인 페이지 (ISR)
│   │   │   ├── page.tsx
│   │   │   └── opengraph-image.tsx  # OG 이미지 동적 생성
│   │   ├── q/[code]/           # QR 리다이렉트 + 분석
│   │   │   └── route.ts        # 리다이렉트 핸들러
│   │   ├── (marketing)/        # 마케팅 페이지 그룹
│   │   │   ├── page.tsx        # 랜딩 페이지
│   │   │   ├── pricing/        # 가격 정책
│   │   │   ├── examples/       # 사례 갤러리
│   │   │   └── about/          # 회사 소개
│   │   ├── api/
│   │   │   ├── auth/           # 인증 엔드포인트
│   │   │   ├── profile/        # 프로필 CRUD
│   │   │   ├── career/         # 경력 CRUD
│   │   │   ├── links/          # 링크 CRUD
│   │   │   ├── ai/             # AI 생성 엔드포인트
│   │   │   ├── cards/          # 명함 CRUD
│   │   │   ├── qr/             # QR 생성
│   │   │   ├── orders/         # 주문 관리
│   │   │   ├── payment/        # 결제 처리
│   │   │   ├── public/         # 공개 API (조회 기록)
│   │   │   └── analytics/      # 통계 API
│   │   ├── layout.tsx          # 루트 레이아웃
│   │   └── globals.css
│   ├── components/
│   │   ├── ui/                 # shadcn/ui 컴포넌트
│   │   ├── layout/             # 레이아웃 컴포넌트
│   │   ├── profile/            # 프로필 관련 컴포넌트
│   │   ├── ai/                 # AI 인터뷰/결과 컴포넌트
│   │   ├── card-editor/        # 명함 에디터 컴포넌트
│   │   ├── qr/                 # QR 코드 컴포넌트
│   │   └── public/             # 공개 페이지 컴포넌트
│   ├── lib/
│   │   ├── supabase/           # Supabase 클라이언트
│   │   ├── prisma.ts           # Prisma 클라이언트 싱글톤
│   │   ├── ai/
│   │   │   ├── client.ts       # Claude API 클라이언트
│   │   │   ├── prompts.ts      # 프롬프트 템플릿
│   │   │   └── story-gen.ts    # 스토리 생성 로직
│   │   ├── qr/
│   │   │   └── generator.ts    # QR 생성 유틸
│   │   ├── card/
│   │   │   ├── renderer.ts     # 명함 캔버스 렌더링
│   │   │   └── pdf-export.ts   # PDF 출력
│   │   ├── payment/
│   │   │   ├── portone.ts      # PortOne 연동
│   │   │   └── webhook.ts      # 결제 웹훅 처리
│   │   ├── rate-limit.ts       # Upstash Rate Limiting
│   │   ├── email.ts            # Resend 이메일
│   │   └── utils.ts            # 공통 유틸
│   ├── hooks/
│   │   ├── useProfile.ts
│   │   ├── useAIStory.ts
│   │   ├── useCardEditor.ts
│   │   └── useAnalytics.ts
│   ├── stores/
│   │   ├── onboard.store.ts    # 온보딩 상태
│   │   └── editor.store.ts     # 에디터 상태
│   └── types/
│       ├── profile.ts
│       ├── ai.ts
│       ├── card.ts
│       └── order.ts
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/
├── .env.example
├── .gitignore
├── next.config.ts
├── tailwind.config.ts
├── tsconfig.json
├── prisma/schema.prisma
└── package.json
```

---

## 라우팅 전략

### SSG (Static Site Generation)
- 마케팅 페이지 (랜딩, 가격, 사례) → `generateStaticParams` 불필요

### ISR (Incremental Static Regeneration)
- 공개 프로필 페이지 `u/[slug]` → `revalidate: 60` (60초)
- 자주 바뀌지 않는 정적 데이터 + 빠른 첫 로딩 중요

### SSR (Server-Side Rendering)
- 대시보드 페이지 → 인증 확인 후 개인화 데이터

### CSR (Client-Side Rendering)
- 명함 에디터 (Fabric.js - 순수 클라이언트)
- 실시간 인터랙션 높은 컴포넌트

---

## 인증 흐름

```
사용자 → Supabase Auth (OAuth)
              ↓
        JWT 토큰 발급
              ↓
        Supabase 세션 쿠키 저장
              ↓
        Next.js Middleware에서 검증
              ↓
        보호된 라우트 접근 허용
```

### 소셜 로그인
- **Google OAuth**: 글로벌 사용자
- **Kakao OAuth**: 국내 사용자 (가장 보편적)
- **Naver OAuth**: 국내 직장인/중장년층

---

## 성능 최적화

### 이미지 최적화
- Next.js Image 컴포넌트 사용
- 프로필 사진: WebP 변환, 여러 사이즈 생성
- 명함 썸네일: 캐시 우선 정책

### 캐싱 전략
- Redis: 공개 프로필 페이지 조회수 (배치 업데이트)
- Redis: AI 생성 결과 임시 캐시 (중복 요청 방지)
- Next.js: ISR로 프로필 페이지 60초 캐시

### Rate Limiting
- AI 엔드포인트: IP당 분당 5회 (Upstash Redis)
- QR 리다이렉트: IP당 초당 30회
- 결제 엔드포인트: 사용자당 분당 10회

---

## 보안 고려사항

### 입력 검증
- 모든 API 엔드포인트: Zod 스키마 유효성 검사
- 파일 업로드: 확장자, MIME 타입, 크기 제한 (10MB)
- AI 프롬프트: 주입 공격 방지 필터링

### 인증/인가
- API Routes: `getServerSession()` + 사용자 소유권 확인
- 관리자 API: 별도 서비스 키 인증

### 개인정보 보호
- 조회 통계: 개인 식별 불가 (IP 해시 저장)
- 민감 정보: 환경 변수만 사용, 코드에 하드코딩 금지
- GDPR 준수: 사용자 데이터 삭제 기능 필수

---

## 배포 설정

### Vercel 환경
```
Production:  main 브랜치 → nexuspoint.me
Preview:     PR별 자동 배포
Development: .env.local
```

### 환경 변수 체크리스트
```env
# DB
DATABASE_URL
DIRECT_URL

# Supabase
NEXT_PUBLIC_SUPABASE_URL
NEXT_PUBLIC_SUPABASE_ANON_KEY
SUPABASE_SERVICE_ROLE_KEY

# AI
ANTHROPIC_API_KEY

# Redis
UPSTASH_REDIS_REST_URL
UPSTASH_REDIS_REST_TOKEN

# Payment
PORTONE_API_KEY
PORTONE_SECRET_KEY
PORTONE_WEBHOOK_SECRET

# Email
RESEND_API_KEY
RESEND_FROM_EMAIL

# App
NEXT_PUBLIC_APP_URL
NEXTAUTH_SECRET
```

---

## MVP 예상 인프라 비용

| 서비스 | 플랜 | 월 비용 |
|--------|------|--------|
| Vercel | Pro | $20 |
| Supabase | Pro | $25 |
| Upstash Redis | Pay-as-you-go | $1-5 |
| Resend | Free (100/day) | $0 |
| Anthropic API | 사용량 기반 | $10-50 |
| **합계** | | **~$56-100/월** |

> AI API 비용은 사용자 수와 생성 횟수에 따라 크게 달라집니다.
> Pro 사용자의 AI 비용은 구독료로 충분히 커버됩니다.
