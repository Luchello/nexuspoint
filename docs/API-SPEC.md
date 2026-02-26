# API-SPEC - NexusPoint API 설계

## 개요

- **Base URL**: `https://nexuspoint.me/api`
- **인증**: Supabase JWT (Bearer token 또는 httpOnly 쿠키)
- **Content-Type**: `application/json`
- **응답 형식**: `{ data: T, error: string | null }`
- **Rate Limiting**: Upstash Redis 기반

---

## 인증 헤더

```http
Authorization: Bearer <supabase_jwt>
```

인증 필요 엔드포인트는 `[Auth]` 표시.

---

## 1. Profile API

### `GET /api/profile` [Auth]
현재 로그인 사용자의 프로필 조회

**Response**
```json
{
  "data": {
    "id": "cuid",
    "slug": "john-doe",
    "displayName": "John Doe",
    "headline": "풀스택 엔지니어",
    "tagline": "사용자 경험을 코드로 설계합니다",
    "bioShort": "...",
    "bioLong": "...",
    "theme": "modern",
    "isPublic": true,
    "careers": [...],
    "links": [...]
  }
}
```

### `PATCH /api/profile` [Auth]
프로필 기본 정보 수정

**Request Body**
```json
{
  "displayName": "string",
  "headline": "string (max 50)",
  "tagline": "string (max 100)",
  "bioShort": "string (max 150)",
  "bioLong": "string (max 500)",
  "location": "string",
  "email": "string | null",
  "phone": "string | null",
  "theme": "string",
  "primaryColor": "#RRGGBB",
  "accentColor": "#RRGGBB",
  "fontFamily": "string",
  "layoutType": "modern | minimal | creative",
  "isPublic": "boolean"
}
```

### `PATCH /api/profile/slug` [Auth]
슬러그 변경 (가용 여부 확인 후)

**Request Body**
```json
{ "slug": "new-slug" }
```

**Response (conflict)**
```json
{ "error": "이미 사용 중인 슬러그입니다." }
```

### `GET /api/profile/slug-check?slug={slug}`
슬러그 사용 가능 여부 확인 (공개 API)

**Response**
```json
{ "data": { "available": true } }
```

### `POST /api/profile/avatar` [Auth]
프로필 사진 업로드 (multipart/form-data)

**Response**
```json
{ "data": { "avatarUrl": "https://storage.supabase.co/..." } }
```

---

## 2. Career API

### `GET /api/career` [Auth]
경력 목록 조회

### `POST /api/career` [Auth]
경력 항목 추가

**Request Body**
```json
{
  "type": "JOB | FREELANCE | EDUCATION | PROJECT | AWARD | VOLUNTEER",
  "originalTitle": "백엔드 개발자",
  "originalDescription": "Node.js와 PostgreSQL로 API 개발",
  "organization": "스타트업 A",
  "location": "서울",
  "startDate": "2022-03-01",
  "endDate": null,
  "isCurrent": true,
  "url": "https://example.com",
  "tags": ["Node.js", "PostgreSQL", "AWS"]
}
```

### `PATCH /api/career/{id}` [Auth]
경력 수정

### `DELETE /api/career/{id}` [Auth]
경력 삭제

### `PATCH /api/career/reorder` [Auth]
경력 순서 변경

**Request Body**
```json
{ "orders": [{ "id": "cuid", "order": 0 }, ...] }
```

---

## 3. Links API

### `GET /api/links` [Auth]
링크 목록 조회

### `POST /api/links` [Auth]
링크 추가

**Request Body**
```json
{
  "type": "GITHUB | LINKEDIN | ... | OTHER",
  "label": "GitHub",
  "url": "https://github.com/username",
  "isVisible": true
}
```

### `PATCH /api/links/{id}` [Auth]
링크 수정

### `DELETE /api/links/{id}` [Auth]
링크 삭제

### `PATCH /api/links/reorder` [Auth]
링크 순서 변경

---

## 4. AI API

### `POST /api/ai/start-interview` [Auth]
AI 스토리텔링 온보딩 시작 (Rate: 5회/분)

**Request Body**
```json
{
  "basicInfo": {
    "name": "홍길동",
    "currentTitle": "개발자",
    "company": "스타트업",
    "industry": "IT/소프트웨어",
    "yearsOfExperience": 5
  },
  "careerStory": {
    "keyAchievements": ["앱 사용자 10만명 달성", "API 응답속도 50% 개선"],
    "corePowers": ["문제 해결", "팀 리더십"],
    "proudMoments": "3인 스타트업에서 MVP를 3개월 만에 출시"
  },
  "brandingPrefs": {
    "tone": "professional | creative | warm | bold | global",
    "targetAudience": "투자자, 동료 개발자",
    "avoidWords": ["평범한", "일반적인"]
  }
}
```

**Response** (streaming 또는 complete)
```json
{
  "data": {
    "storyId": "cuid",
    "status": "generating | complete"
  }
}
```

### `GET /api/ai/story/{id}` [Auth]
생성된 AI 스토리 조회

**Response**
```json
{
  "data": {
    "id": "cuid",
    "version": 1,
    "isActive": true,
    "tone": "professional",
    "headline": "사용자 경험을 코드로 설계하는 5년 경력 풀스택 엔지니어",
    "tagline": "3인 스타트업에서 MVP를 3개월 만에",
    "bioShort": "기술과 비즈니스의 교차점에서 제품을 만듭니다.",
    "bioLong": "...",
    "careerSummary": ["성과1", "성과2", "성과3"],
    "valueProposition": "..."
  }
}
```

### `GET /api/ai/stories` [Auth]
AI 스토리 버전 목록 조회

### `POST /api/ai/regenerate-section` [Auth]
특정 섹션만 재생성

**Request Body**
```json
{
  "storyId": "cuid",
  "section": "headline | tagline | bioShort | bioLong | careerSummary",
  "tone": "bold",
  "customInstruction": "더 대담하게 표현해줘"
}
```

### `POST /api/ai/enhance-career/{careerId}` [Auth]
특정 경력 항목 AI 향상

**Response**
```json
{
  "data": {
    "enhancedTitle": "스케일업 전문 풀스택 엔지니어",
    "enhancedDescription": "..."
  }
}
```

### `PATCH /api/ai/story/{id}/activate` [Auth]
특정 버전의 AI 스토리를 프로필에 적용

### `PATCH /api/ai/story/{id}` [Auth]
AI 스토리 수동 편집

---

## 5. Cards API

### `GET /api/cards` [Auth]
명함 디자인 목록 조회

### `POST /api/cards` [Auth]
명함 생성 (빈 디자인 또는 템플릿 기반)

**Request Body**
```json
{
  "name": "메인 명함",
  "templateId": "template_modern_01"
}
```

### `GET /api/cards/{id}` [Auth]
명함 상세 조회 (Fabric.js JSON 포함)

### `PATCH /api/cards/{id}` [Auth]
명함 저장 (에디터 자동 저장)

**Request Body**
```json
{
  "name": "string",
  "frontDesignJson": { ...fabricjs_json },
  "backDesignJson": { ...fabricjs_json },
  "qrEnabled": true,
  "qrPosition": { "x": 200, "y": 30, "width": 40, "height": 40 },
  "qrColor": "#000000"
}
```

### `DELETE /api/cards/{id}` [Auth]
명함 삭제

### `POST /api/cards/{id}/generate-images` [Auth]
명함 이미지 생성 (미리보기용 PNG + 인쇄용 PDF)

**Response**
```json
{
  "data": {
    "frontImageUrl": "https://storage...",
    "backImageUrl": "https://storage...",
    "printPdfUrl": "https://storage..."
  }
}
```

### `GET /api/cards/templates`
명함 템플릿 목록 조회 (공개 API)

**Query Params**: `?tier=free|all`

**Response**
```json
{
  "data": [
    {
      "id": "template_modern_01",
      "name": "모던 미니멀",
      "category": "business",
      "tier": "free",
      "thumbnailUrl": "...",
      "frontPreviewUrl": "..."
    }
  ]
}
```

---

## 6. QR API

### `GET /api/qr/{profileSlug}`
프로필용 QR 코드 생성

**Query Params**:
- `format`: `png | svg` (default: png)
- `size`: `200 | 300 | 400` (픽셀, default: 300)
- `color`: `#RRGGBB` (default: #000000)
- `bgColor`: `#RRGGBB` (default: #ffffff)

**Response**: 이미지 파일 (Content-Type: image/png)

### `POST /api/qr/generate` [Auth]
QR 생성 + Supabase Storage 저장

---

## 7. Orders API

### `POST /api/orders` [Auth]
인쇄 주문 생성

**Request Body**
```json
{
  "items": [
    {
      "cardDesignId": "cuid",
      "quantity": 100,
      "material": "standard | premium | linen | ultra",
      "finishing": ["glossy_coating"]
    }
  ],
  "shippingAddress": {
    "recipientName": "홍길동",
    "recipientPhone": "010-1234-5678",
    "zipCode": "06236",
    "address1": "서울특별시 강남구 테헤란로 123",
    "address2": "101호",
    "deliveryMemo": "문 앞에 놓아주세요"
  },
  "couponCode": "NEXUS10"
}
```

**Response**
```json
{
  "data": {
    "orderId": "cuid",
    "orderNumber": "ORD-20250101-A1B2",
    "totalAmount": 15000,
    "checkoutUrl": "https://portone..."
  }
}
```

### `GET /api/orders` [Auth]
주문 내역 조회

### `GET /api/orders/{id}` [Auth]
주문 상세 조회

### `POST /api/orders/{id}/cancel` [Auth]
주문 취소 (PENDING 상태만 가능)

### `GET /api/orders/price-calc`
가격 계산 (주문 전 금액 조회)

**Query Params**: `material`, `quantity`, `finishing[]`

---

## 8. Payment API

### `POST /api/payment/checkout` [Auth]
결제 세션 생성

**Request Body**
```json
{
  "type": "subscription | print_order",
  "orderId": "cuid (print_order 시)",
  "planId": "pro_monthly | pro_yearly | premium_monthly (subscription 시)"
}
```

### `POST /api/payment/confirm` [Auth]
결제 완료 확인 (PortOne)

**Request Body**
```json
{
  "paymentId": "portone_payment_id",
  "orderId": "cuid"
}
```

### `POST /api/payment/webhook`
PortOne 웹훅 수신 (서명 검증 필수)

### `GET /api/payment/subscription` [Auth]
현재 구독 상태 조회

### `POST /api/payment/subscription/cancel` [Auth]
구독 해지

---

## 9. Public API

### `GET /api/public/profile/{slug}`
공개 프로필 데이터 조회 (ISR용 내부 API)

### `POST /api/public/view`
페이지 조회 기록 (비인증, Rate Limited)

**Request Body**
```json
{
  "profileSlug": "john-doe",
  "source": "QR_SCAN | DIRECT_LINK | SOCIAL_SHARE | EMAIL",
  "referrer": "https://..."
}
```

### `GET /api/public/vcard/{slug}`
vCard 파일 다운로드 (.vcf)

**Response**: text/vcard

---

## 10. Analytics API

### `GET /api/analytics/overview` [Auth]
통계 요약 (총 조회수, 오늘 조회수, 상위 소스)

**Query Params**: `?days=7|30|90`

**Response**
```json
{
  "data": {
    "totalViews": 1234,
    "todayViews": 42,
    "weekViews": 289,
    "topSource": "QR_SCAN",
    "sources": [
      { "source": "QR_SCAN", "count": 700, "percentage": 56.7 },
      { "source": "DIRECT_LINK", "count": 400, "percentage": 32.4 }
    ]
  }
}
```

### `GET /api/analytics/timeline` [Auth]
기간별 조회수 타임라인

**Query Params**: `?days=30&groupBy=day|week`

### `GET /api/analytics/devices` [Auth]
디바이스/브라우저별 통계 (Pro+)

### `GET /api/analytics/geography` [Auth]
국가/도시별 통계 (Pro+)

---

## 에러 코드

| HTTP 코드 | 에러 코드 | 설명 |
|-----------|-----------|------|
| 400 | INVALID_INPUT | 입력값 유효성 검사 실패 |
| 401 | UNAUTHORIZED | 인증 토큰 없음/만료 |
| 403 | FORBIDDEN | 권한 없음 (다른 사용자 리소스) |
| 403 | PLAN_LIMIT | 플랜 제한 초과 |
| 404 | NOT_FOUND | 리소스 없음 |
| 409 | CONFLICT | 슬러그 중복 등 충돌 |
| 429 | RATE_LIMIT | Rate Limit 초과 |
| 500 | SERVER_ERROR | 서버 내부 오류 |

---

## Rate Limit 정책

| 엔드포인트 | 제한 | 기준 |
|-----------|------|------|
| `POST /api/ai/*` | 5회/분 | IP + User ID |
| `POST /api/public/view` | 30회/분 | IP |
| `GET /api/qr/*` | 100회/분 | IP |
| `POST /api/payment/*` | 10회/분 | User ID |
| 기타 인증 API | 60회/분 | User ID |
