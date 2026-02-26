# DATA-MODEL - NexusPoint 데이터 모델

## Prisma 스키마

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider  = "postgresql"
  url       = env("DATABASE_URL")
  directUrl = env("DIRECT_URL")
}

// ─────────────────────────────────────────────
// ENUMS
// ─────────────────────────────────────────────

enum SubscriptionTier {
  FREE
  PRO
  PREMIUM
  BUSINESS
}

enum SubscriptionStatus {
  ACTIVE
  CANCELLED
  PAST_DUE
  EXPIRED
}

enum CareerType {
  JOB          // 직장 경력
  FREELANCE    // 프리랜서
  EDUCATION    // 학력
  PROJECT      // 프로젝트
  AWARD        // 수상/인증
  VOLUNTEER    // 봉사 활동
}

enum LinkType {
  LINKEDIN
  GITHUB
  TWITTER
  INSTAGRAM
  FACEBOOK
  YOUTUBE
  TIKTOK
  BLOG
  PORTFOLIO
  WEBSITE
  EMAIL
  PHONE
  KAKAO
  OTHER
}

enum PrintOrderStatus {
  PENDING       // 결제 대기
  PAID          // 결제 완료
  CONFIRMED     // 주문 확정 (인쇄소 전송)
  PRINTING      // 인쇄 중
  SHIPPED       // 발송 완료
  DELIVERED     // 배송 완료
  CANCELLED     // 취소
  REFUNDED      // 환불
}

enum ViewSource {
  QR_SCAN       // QR 코드 스캔
  DIRECT_LINK   // 직접 URL 입력
  SOCIAL_SHARE  // SNS 공유
  EMAIL         // 이메일 링크
  UNKNOWN
}

// ─────────────────────────────────────────────
// CORE ENTITIES
// ─────────────────────────────────────────────

model User {
  id            String    @id @default(cuid())
  supabaseId    String    @unique  // Supabase Auth UID
  email         String    @unique
  name          String?
  avatarUrl     String?
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt

  // Relations
  profile       Profile?
  subscription  Subscription?
  orders        PrintOrder[]

  @@map("users")
}

model Profile {
  id              String   @id @default(cuid())
  userId          String   @unique
  user            User     @relation(fields: [userId], references: [id], onDelete: Cascade)

  // 공개 URL
  slug            String   @unique  // nexuspoint.me/u/{slug}
  customDomain    String?  @unique  // 커스텀 도메인 (Premium+)

  // 기본 정보
  displayName     String
  headline        String?           // AI 생성 헤드라인 (50자 이내)
  tagline         String?           // AI 생성 태그라인 (100자 이내)
  bioShort        String?           // 짧은 소개 (150자 이내)
  bioLong         String?           // 긴 소개 (500자 이내)
  location        String?           // 위치 (시/도)
  email           String?           // 공개 이메일
  phone           String?           // 공개 전화번호
  avatarUrl       String?           // Supabase Storage URL

  // 테마/레이아웃
  theme           String   @default("default")  // 테마 이름
  primaryColor    String   @default("#000000")
  accentColor     String   @default("#6366f1")
  fontFamily      String   @default("inter")
  layoutType      String   @default("modern")   // modern | minimal | creative

  // 가시성
  isPublic        Boolean  @default(true)
  passwordHash    String?  // 비밀번호 보호 (Premium+)
  showPoweredBy   Boolean  @default(true)  // 워터마크

  createdAt       DateTime @default(now())
  updatedAt       DateTime @updatedAt

  // Relations
  careers         CareerEntry[]
  links           ProfileLink[]
  aiStories       AIStory[]
  cardDesigns     CardDesign[]
  pageViews       PageView[]

  @@index([slug])
  @@index([customDomain])
  @@map("profiles")
}

model CareerEntry {
  id                  String     @id @default(cuid())
  profileId           String
  profile             Profile    @relation(fields: [profileId], references: [id], onDelete: Cascade)

  type                CareerType
  order               Int        @default(0)    // 정렬 순서

  // 원본 정보
  originalTitle       String                    // 원본 직함/이름
  originalDescription String?    @db.Text       // 원본 설명

  // AI 향상 정보
  enhancedTitle       String?                   // AI가 향상한 직함
  enhancedDescription String?    @db.Text       // AI가 향상한 설명

  // 공통 필드
  organization        String?                   // 회사/학교/기관
  location            String?
  startDate           DateTime?
  endDate             DateTime?
  isCurrent           Boolean    @default(false)

  // 타입별 추가 정보
  url                 String?                   // 관련 URL
  tags                String[]                  // 기술 스택, 키워드

  createdAt           DateTime   @default(now())
  updatedAt           DateTime   @updatedAt

  @@index([profileId, order])
  @@map("career_entries")
}

model ProfileLink {
  id          String    @id @default(cuid())
  profileId   String
  profile     Profile   @relation(fields: [profileId], references: [id], onDelete: Cascade)

  type        LinkType
  label       String                // 표시명 (예: "GitHub", "개인 블로그")
  url         String                // 실제 URL
  order       Int       @default(0) // 정렬 순서
  isVisible   Boolean   @default(true)

  createdAt   DateTime  @default(now())
  updatedAt   DateTime  @updatedAt

  @@index([profileId, order])
  @@map("profile_links")
}

// ─────────────────────────────────────────────
// AI STORY
// ─────────────────────────────────────────────

model AIStory {
  id              String   @id @default(cuid())
  profileId       String
  profile         Profile  @relation(fields: [profileId], references: [id], onDelete: Cascade)

  version         Int      @default(1)   // 버전 번호
  isActive        Boolean  @default(false) // 현재 적용 중인 버전

  // 생성 설정
  tone            String   // professional | creative | warm | bold | global
  inputData       Json     // 온보딩 입력 원본 데이터

  // 생성 결과
  headline        String?               // 헤드라인 (15자 이내)
  tagline         String?               // 태그라인 (30자 이내)
  bioShort        String?               // 짧은 소개 (50자)
  bioLong         String?  @db.Text     // 긴 소개 (3-4문장)
  careerSummary   String[] // 불릿 포인트 리스트 (3-5개)
  valueProposition String? @db.Text    // 가치 제안문

  // 메타
  tokensUsed      Int?     // API 토큰 사용량
  generatedAt     DateTime @default(now())

  @@index([profileId, isActive])
  @@map("ai_stories")
}

// ─────────────────────────────────────────────
// CARD DESIGN
// ─────────────────────────────────────────────

model CardDesign {
  id            String   @id @default(cuid())
  profileId     String
  profile       Profile  @relation(fields: [profileId], references: [id], onDelete: Cascade)

  name          String   @default("내 명함")
  isActive      Boolean  @default(false) // QR에 연결된 기본 명함

  // 템플릿
  templateId    String                  // 사용한 템플릿 ID
  templateName  String

  // 디자인 데이터 (Fabric.js JSON)
  frontDesignJson Json                  // 앞면 디자인 데이터
  backDesignJson  Json?                 // 뒷면 디자인 데이터

  // QR 설정
  qrEnabled     Boolean  @default(true)
  qrPosition    Json?    // {x, y, width, height}
  qrColor       String   @default("#000000")
  qrBgColor     String   @default("#ffffff")
  qrLogoUrl     String?  // QR 중앙 로고 (Pro+)

  // 출력 이미지 (생성된 후 저장)
  frontImageUrl String?  // Supabase Storage URL (썸네일)
  backImageUrl  String?
  printPdfUrl   String?  // 인쇄용 300DPI PDF

  // 명함 크기 (mm)
  width         Float    @default(90)   // 90mm (표준)
  height        Float    @default(50)   // 50mm (표준)

  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt

  // Relations
  orderItems    PrintOrderItem[]

  @@index([profileId])
  @@map("card_designs")
}

// ─────────────────────────────────────────────
// PRINT ORDER
// ─────────────────────────────────────────────

model PrintOrder {
  id              String           @id @default(cuid())
  userId          String
  user            User             @relation(fields: [userId], references: [id])

  orderNumber     String           @unique  // ORD-20250101-XXXX
  status          PrintOrderStatus @default(PENDING)

  // 금액
  subtotal        Int              // 원 단위
  shippingFee     Int              @default(3000)
  discountAmount  Int              @default(0)
  totalAmount     Int

  // 결제
  paymentId       String?          // PortOne 거래 ID
  paidAt          DateTime?

  // 배송지
  recipientName   String
  recipientPhone  String
  zipCode         String
  address1        String
  address2        String?
  deliveryMemo    String?

  // 트래킹
  trackingCompany String?          // 택배사
  trackingNumber  String?          // 운송장 번호
  shippedAt       DateTime?
  deliveredAt     DateTime?

  createdAt       DateTime         @default(now())
  updatedAt       DateTime         @updatedAt

  // Relations
  items           PrintOrderItem[]

  @@index([userId])
  @@index([orderNumber])
  @@map("print_orders")
}

model PrintOrderItem {
  id            String     @id @default(cuid())
  orderId       String
  order         PrintOrder @relation(fields: [orderId], references: [id])
  cardDesignId  String
  cardDesign    CardDesign @relation(fields: [cardDesignId], references: [id])

  quantity      Int        // 수량 (50, 100, 200, 500)
  material      String     // standard | premium | linen | ultra
  finishing     String[]   // ["glossy_coating", "emboss", "hologram"]
  unitPrice     Int        // 단가 (원)
  totalPrice    Int        // 소계

  // 스냅샷 (주문 시점의 디자인 저장)
  frontImageUrl String
  backImageUrl  String?

  @@map("print_order_items")
}

// ─────────────────────────────────────────────
// SUBSCRIPTION
// ─────────────────────────────────────────────

model Subscription {
  id                String             @id @default(cuid())
  userId            String             @unique
  user              User               @relation(fields: [userId], references: [id])

  tier              SubscriptionTier   @default(FREE)
  status            SubscriptionStatus @default(ACTIVE)

  // 결제 정보
  billingCycle      String?   // monthly | yearly
  paymentMethodId   String?   // PortOne 빌링키
  currentPeriodStart DateTime?
  currentPeriodEnd   DateTime?
  cancelledAt        DateTime?

  // 사용량 추적 (Free 플랜 제한)
  aiGenerationsThisMonth Int   @default(0)
  aiGenerationsResetAt   DateTime?

  createdAt         DateTime  @default(now())
  updatedAt         DateTime  @updatedAt

  @@map("subscriptions")
}

// ─────────────────────────────────────────────
// ANALYTICS
// ─────────────────────────────────────────────

model PageView {
  id          String     @id @default(cuid())
  profileId   String
  profile     Profile    @relation(fields: [profileId], references: [id], onDelete: Cascade)

  source      ViewSource @default(UNKNOWN)
  referrer    String?    // 리퍼러 URL
  userAgent   String?    // 디바이스/브라우저 정보
  country     String?    // 국가 코드 (ISO 3166-1 alpha-2)
  city        String?    // 도시

  // 개인정보 비식별화
  ipHash      String?    // IP 해시 (원본 IP 저장 안함)

  viewedAt    DateTime   @default(now())

  @@index([profileId, viewedAt])
  @@index([profileId, source])
  @@map("page_views")
}
```

---

## 엔티티 관계 다이어그램 (ERD 텍스트)

```
User (1) ──────────── (1) Profile
User (1) ──────────── (1) Subscription
User (1) ──────────── (N) PrintOrder

Profile (1) ──────── (N) CareerEntry
Profile (1) ──────── (N) ProfileLink
Profile (1) ──────── (N) AIStory
Profile (1) ──────── (N) CardDesign
Profile (1) ──────── (N) PageView

CardDesign (1) ────── (N) PrintOrderItem
PrintOrder (1) ────── (N) PrintOrderItem
```

---

## 주요 쿼리 패턴

### 공개 프로필 페이지 (ISR)
```typescript
// /u/[slug] 페이지 - 프로필 + 경력 + 링크 + 활성 AI 스토리
const profile = await prisma.profile.findUnique({
  where: { slug, isPublic: true },
  include: {
    careers: { orderBy: { order: 'asc' } },
    links: { where: { isVisible: true }, orderBy: { order: 'asc' } },
    aiStories: { where: { isActive: true }, take: 1 },
  },
});
```

### AI 스토리 버전 관리
```typescript
// 새 AI 스토리 저장 (이전 버전 비활성화)
await prisma.$transaction([
  prisma.aIStory.updateMany({
    where: { profileId, isActive: true },
    data: { isActive: false },
  }),
  prisma.aIStory.create({
    data: { ...newStory, profileId, isActive: true, version: latestVersion + 1 },
  }),
]);
```

### 조회 통계 집계
```typescript
// 최근 30일 소스별 조회수
const stats = await prisma.pageView.groupBy({
  by: ['source'],
  where: { profileId, viewedAt: { gte: thirtyDaysAgo } },
  _count: { id: true },
});
```

---

## 인덱스 전략

| 테이블 | 인덱스 | 이유 |
|--------|--------|------|
| profiles | slug | 공개 URL 조회 (가장 빈번) |
| profiles | customDomain | 커스텀 도메인 조회 |
| career_entries | (profileId, order) | 프로필별 경력 정렬 조회 |
| ai_stories | (profileId, isActive) | 활성 스토리 조회 |
| page_views | (profileId, viewedAt) | 기간별 통계 조회 |
| page_views | (profileId, source) | 소스별 통계 조회 |
