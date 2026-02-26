# NexusPoint

> QR 코드가 새겨진 프리미엄 명함 + AI가 만들어주는 개인 브랜딩 홈페이지

명함의 QR 코드를 찍으면 당신만의 프리미엄 프로필 페이지로 연결됩니다.
AI가 평범한 커리어를 전문적으로 포장해주고, 명함 실물 제작/배송까지 원스톱으로 제공합니다.

## 핵심 기능

- **AI 커리어 스토리텔링**: Claude AI가 평범한 직함과 경력을 전문적이고 매력적인 이야기로 변환
- **개인 프로필 홈페이지**: `nexuspoint.me/u/{slug}` 형태의 세련된 개인 랜딩 페이지
- **QR 명함 에디터**: 템플릿 기반 명함 디자인 + QR 코드 자동 삽입
- **명함 인쇄 주문**: 디자인 완성 후 실물 명함 제작/배송까지 원스톱 처리

## 기술 스택

| 영역 | 기술 |
|------|------|
| Frontend | Next.js 16 (App Router) + TypeScript + Tailwind CSS 4 + shadcn/ui |
| Backend | Next.js API Routes |
| Database | Supabase PostgreSQL + Prisma 7 |
| Auth | Supabase Auth (Google, Kakao, Naver) |
| AI | Claude API (claude-sonnet-4-20250514) |
| Storage | Supabase Storage |
| Cache | Upstash Redis |
| Payment | PortOne / TossPayments |
| QR | qrcode.react + node-qrcode |
| Card Editor | Fabric.js |
| Email | Resend |
| Deploy | Vercel + Supabase Cloud |

## 디렉토리 구조

```
nexuspoint/
├── prisma/
│   └── schema.prisma          # DB 스키마
├── src/
│   ├── app/
│   │   ├── (auth)/            # 인증 (로그인/회원가입)
│   │   ├── (dashboard)/       # 대시보드 (로그인 필요)
│   │   │   ├── profile/       # 프로필 편집
│   │   │   ├── ai-story/      # AI 스토리텔링
│   │   │   ├── cards/         # 명함 관리
│   │   │   ├── orders/        # 주문 내역
│   │   │   ├── analytics/     # 방문 통계
│   │   │   └── settings/      # 설정/구독
│   │   ├── u/[slug]/          # 공개 개인 페이지 (ISR)
│   │   ├── q/[code]/          # QR 리다이렉트 + 분석
│   │   ├── (marketing)/       # 랜딩/가격/사례
│   │   └── api/               # API Routes
│   ├── components/            # UI 컴포넌트
│   ├── lib/
│   │   ├── ai/                # Claude API + 프롬프트
│   │   ├── qr/                # QR 생성
│   │   ├── card/              # 명함 렌더링
│   │   └── payment/           # 결제 연동
│   ├── hooks/                 # React 훅
│   ├── stores/                # Zustand 스토어
│   └── types/                 # TypeScript 타입
├── tests/                     # 테스트
├── docs/
│   ├── PRD.md                 # 제품 요구사항
│   ├── TECH-SPEC.md           # 기술 아키텍처
│   ├── DATA-MODEL.md          # 데이터 모델
│   ├── API-SPEC.md            # API 설계
│   ├── AI-ENGINE.md           # AI 엔진 설계
│   ├── BUSINESS-MODEL.md      # 비즈니스 모델
│   └── ROADMAP.md             # 구현 로드맵
└── README.md
```

## 로컬 개발 환경 셋업

### 필수 조건
- Node.js 22+
- pnpm 9+
- Supabase CLI

### 1. 저장소 클론

```bash
git clone https://github.com/YOUR_USERNAME/nexuspoint.git
cd nexuspoint
```

### 2. 의존성 설치

```bash
pnpm install
```

### 3. 환경 변수 설정

```bash
cp .env.example .env.local
```

`.env.local` 파일을 편집하여 필요한 환경 변수를 설정하세요:

```env
# Supabase
NEXT_PUBLIC_SUPABASE_URL=your_supabase_url
NEXT_PUBLIC_SUPABASE_ANON_KEY=your_supabase_anon_key
SUPABASE_SERVICE_ROLE_KEY=your_service_role_key

# Anthropic Claude API
ANTHROPIC_API_KEY=your_anthropic_api_key

# Upstash Redis
UPSTASH_REDIS_REST_URL=your_redis_url
UPSTASH_REDIS_REST_TOKEN=your_redis_token

# Payment
PORTONE_API_KEY=your_portone_key
PORTONE_SECRET_KEY=your_portone_secret

# Resend Email
RESEND_API_KEY=your_resend_key

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

### 4. 데이터베이스 마이그레이션

```bash
pnpm prisma migrate dev
pnpm prisma generate
```

### 5. 개발 서버 실행

```bash
pnpm dev
```

브라우저에서 [http://localhost:3000](http://localhost:3000) 을 열어 확인하세요.

## 문서

- [PRD (제품 요구사항)](./docs/PRD.md)
- [기술 아키텍처](./docs/TECH-SPEC.md)
- [데이터 모델](./docs/DATA-MODEL.md)
- [API 설계](./docs/API-SPEC.md)
- [AI 엔진 설계](./docs/AI-ENGINE.md)
- [비즈니스 모델](./docs/BUSINESS-MODEL.md)
- [구현 로드맵](./docs/ROADMAP.md)

## 라이선스

MIT License - 자세한 내용은 [LICENSE](./LICENSE) 파일을 참조하세요.

---

Built with ❤️ by the NexusPoint Team
