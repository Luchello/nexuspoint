# AI-ENGINE - NexusPoint AI 스토리텔링 엔진 설계

## 개요

NexusPoint의 핵심 가치 제안은 **"평범한 커리어 정보를 전문적으로 포장"**하는 것입니다.
Claude AI를 활용하여 사용자의 실제 경험을 기반으로, 진실되면서도 매력적인 개인 브랜딩 스토리를 생성합니다.

---

## 설계 원칙

1. **진실 기반 포장**: 사실을 과장하거나 왜곡하지 않고, 가치를 극대화하는 방향으로 표현
2. **업계 전문 언어**: 해당 분야에서 실제로 사용하는 전문 용어와 표현 방식 활용
3. **구체적 가치 강조**: 숫자, 결과, 임팩트를 중심으로 서술
4. **청중 지향**: 타겟 독자(투자자, 채용 담당자, 파트너 등)에 맞는 어조
5. **일관성 유지**: 프로필 전체에 걸쳐 동일한 브랜딩 톤 유지

---

## 온보딩 인터뷰 (3단계)

### 1단계: 기본 정보 (30초)
```
질문:
- 이름과 현재 직함/역할은 무엇인가요?
- 어느 업종/분야에서 일하고 계신가요?
- 총 경력 기간은 얼마나 되나요?
- 현재 소속 (회사, 프리랜서, 창업 등)?
```

### 2단계: 커리어 스토리 (2분)
```
질문:
- 지금까지 가장 자랑스러운 성과나 프로젝트를 3가지 말씀해주세요.
- 당신만의 핵심 강점이나 전문성은 무엇인가요?
- 어떤 문제를 잘 해결하시나요?
- 기억에 남는 힘든 도전을 극복한 경험이 있나요?
- 앞으로 어떤 방향으로 나아가고 싶으신가요?
```

### 3단계: 브랜딩 취향 (1분)
```
질문:
- 주로 어떤 사람들에게 자신을 소개할 건가요? (채용 담당자, 클라이언트, 투자자 등)
- 어떤 인상을 남기고 싶으신가요? (전문적, 창의적, 따뜻한, 대담한)
- 피하고 싶은 표현이나 느낌이 있나요?
- 글로벌 시장을 염두에 두고 계신가요?
```

---

## 변환 패턴 예시 (Before → After)

| 직업/역할 | Before (원본) | After (AI 향상) |
|-----------|--------------|-----------------|
| 배관공 | "배관 설치 및 수리" | "산업 인프라 설비 전문 엔지니어 | 주거·상업 공간의 유체 시스템을 설계하고 최적화" |
| 바리스타 | "카페에서 커피 만들기" | "스페셜티 커피 큐레이터 | 원두 산지부터 추출까지, 감각적 경험을 설계합니다" |
| 중학교 수학 교사 | "수학 가르치는 교사" | "수리적 사고력 개발 전문가 | 15년간 2,000+ 학생의 수학적 잠재력을 발굴" |
| 영업사원 | "B2B 영업, 고객 관리" | "기업 성장 파트너 | 솔루션 기반 접근으로 클라이언트의 비즈니스 문제를 해결하는 B2B 전략가" |
| 주부 | "가정 관리, 육아" | "가족 운영 전문가 | 복잡한 일정 조율, 예산 관리, 인적 자원 개발의 실전 전문가" |
| React 개발자 | "React로 웹 개발" | "사용자 경험 중심 프론트엔드 아키텍트 | React 생태계로 1초 이하 인터랙션 경험을 설계" |

---

## 톤 (5종)

### Professional (전문적)
- 격식체, 명확한 전문 용어
- 성과와 수치 중심
- 예: "5년간 B2B SaaS 제품 개발로 ARR $2M 달성에 기여"

### Creative (창의적)
- 은유와 이미지를 활용한 생동감 있는 표현
- 개성과 독창성 강조
- 예: "코드는 캔버스, 아이디어는 물감 — 디지털 경험을 그립니다"

### Warm (따뜻한)
- 사람 중심, 관계 강조
- 부드러운 문체, 진정성 있는 톤
- 예: "사람과 기술 사이의 다리를 놓는 것, 그게 제가 가장 잘하는 일입니다"

### Bold (대담한)
- 직설적, 자신감 넘치는 어조
- 강한 동사와 적극적 표현
- 예: "결과를 만들어냅니다. 방법은 나중에 설명하겠습니다."

### Global (글로벌)
- 영어권/글로벌 시장을 위한 간결하고 보편적 표현
- 문화적 맥락 없이도 이해 가능
- 예: "Full-stack engineer crafting seamless digital experiences across 3 continents"

---

## 생성 항목

| 항목 | 길이 | 설명 |
|------|------|------|
| `headline` | 최대 50자 | 프로필 제목 (직함 + 핵심 가치) |
| `tagline` | 최대 80자 | 부제목 (인상적인 한 줄) |
| `bioShort` | 최대 150자 | 짧은 자기소개 (명함, SNS용) |
| `bioLong` | 3-4문장 | 긴 자기소개 (홈페이지용) |
| `careerSummary` | 3-5 bullet | 핵심 성과 및 역량 리스트 |
| `valueProposition` | 2-3문장 | 내가 제공하는 가치 선언문 |

---

## 프롬프트 설계

### 시스템 프롬프트

```
당신은 개인 브랜딩 전문가이자 카피라이터입니다.
사용자의 실제 경력과 성과를 바탕으로, 진실에 기반하면서도
전문적이고 매력적인 프로필 텍스트를 작성합니다.

핵심 원칙:
1. 사실만 사용 - 과장이나 허위 정보 금지
2. 구체적 표현 - 추상적 형용사 대신 구체적 결과와 숫자
3. 능동형 동사 - "담당했습니다" 대신 "이끌었습니다", "달성했습니다"
4. 업계 언어 - 해당 분야 전문가가 공감할 수 있는 표현
5. 청중 의식 - 독자가 누구인지를 항상 염두에 두고 작성

금지 표현:
- "열심히", "성실하게" (진부함)
- "다양한", "여러" (불명확함)
- "~에 관심이 많습니다" (수동적)
- "~하려고 노력합니다" (불확실함)
```

### 사용자 프롬프트 예시

```
다음 정보를 바탕으로 {tone} 톤의 프로필 텍스트를 생성해주세요.

[기본 정보]
이름: {name}
직함: {title}
업종: {industry}
경력: {years}년

[성과 및 강점]
{achievements_list}

[강점]
{core_powers}

[타겟 독자]
{target_audience}

[요청 형식]
반드시 다음 JSON 형식으로 응답하세요:
{
  "headline": "최대 50자",
  "tagline": "최대 80자",
  "bioShort": "최대 150자",
  "bioLong": "3-4문장의 자기소개",
  "careerSummary": ["성과1", "성과2", "성과3"],
  "valueProposition": "2-3문장의 가치 제안"
}
```

---

## 편집 UX 워크플로우

```
AI 스토리 생성 완료
        ↓
┌─────────────────────────────┐
│     생성 결과 미리보기        │
│  ┌───────────────────┐      │
│  │ [headline] ✏️ 재생성│      │
│  │ [tagline]  ✏️ 재생성│      │
│  │ [bioShort] ✏️ 재생성│      │
│  └───────────────────┘      │
│  톤 변경: [Pro][Warm][Bold]   │
│  전체 재생성 [버튼]           │
└─────────────────────────────┘
        ↓
┌─────────────────────────────┐
│     인라인 편집 모드          │
│  클릭하면 직접 수정 가능       │
│  "이렇게 바꿔줘" 자유 지시     │
└─────────────────────────────┘
        ↓
┌─────────────────────────────┐
│     버전 히스토리             │
│  v3 (현재) | v2 | v1        │
│  이전 버전으로 되돌리기        │
└─────────────────────────────┘
        ↓
     프로필에 적용
```

### 지원 편집 기능

1. **섹션별 재생성**: 특정 섹션만 Claude에 재요청
2. **톤 변경 재생성**: 전체를 다른 톤으로 재생성
3. **자유 지시**: "더 간결하게", "기술적인 표현 더 추가해줘" 등 자연어 지시
4. **인라인 편집**: 생성된 텍스트를 직접 수정
5. **버전 히스토리**: 최대 10개 버전 저장, 언제든 롤백

---

## AI Provider 추상화

```typescript
// src/lib/ai/client.ts

interface AIProvider {
  generateStory(input: StoryInput): Promise<StoryOutput>;
  regenerateSection(input: RegenerateInput): Promise<string>;
}

class ClaudeProvider implements AIProvider {
  private client: Anthropic;

  constructor() {
    this.client = new Anthropic({
      apiKey: process.env.ANTHROPIC_API_KEY,
    });
  }

  async generateStory(input: StoryInput): Promise<StoryOutput> {
    const response = await this.client.messages.create({
      model: "claude-sonnet-4-20250514",
      max_tokens: 1024,
      messages: [
        { role: "user", content: buildPrompt(input) }
      ],
      system: SYSTEM_PROMPT,
    });

    return parseStoryOutput(response);
  }
}

// OpenAI 폴백 (선택적)
class OpenAIProvider implements AIProvider { ... }

// 사용
const aiProvider: AIProvider = process.env.AI_PROVIDER === 'openai'
  ? new OpenAIProvider()
  : new ClaudeProvider();
```

---

## 비용 분석

| 시나리오 | 입력 토큰 | 출력 토큰 | 비용 (Sonnet) |
|---------|---------|---------|-------------|
| 전체 스토리 생성 | ~500 | ~600 | ~35원 |
| 섹션 재생성 | ~300 | ~200 | ~10원 |
| 경력 1개 향상 | ~200 | ~150 | ~7원 |
| **1회 온보딩 평균** | | | **~50-80원** |

**플랜별 비용 커버 분석**:
- Free: 월 3회 × 80원 = 240원/월 → 구독료 없음, 마케팅 비용으로 간주
- Pro (9,900원/월): 무제한 생성 ≈ 월 50회 사용 시 = 4,000원 AI 비용 → 5,900원 마진 (60%)
- Premium/Business: 더 높은 마진

---

## 에러 처리 및 폴백

```typescript
async function generateWithFallback(input: StoryInput): Promise<StoryOutput> {
  try {
    return await claudeProvider.generateStory(input);
  } catch (error) {
    if (error.status === 529) {
      // Claude 서버 과부하 → OpenAI 폴백
      return await openaiProvider.generateStory(input);
    }
    if (error.status === 429) {
      // Rate limit → 대기 후 재시도
      await sleep(2000);
      return await claudeProvider.generateStory(input);
    }
    throw error;
  }
}
```

---

## 콘텐츠 모더레이션

생성 전후로 부적절한 콘텐츠 필터링:

1. **입력 필터링**: 명백한 욕설, 불법 활동, 혐오 표현 차단
2. **출력 검증**: 생성된 텍스트의 길이, 형식 검증
3. **Claude 내장 안전 장치**: Claude 자체의 안전 필터 활용

```typescript
function sanitizeInput(text: string): string {
  // 프롬프트 인젝션 방지
  const dangerous = ['\n\nHuman:', '\n\nAssistant:', 'Ignore previous'];
  for (const pattern of dangerous) {
    text = text.replace(pattern, '');
  }
  return text.slice(0, 1000); // 최대 1000자 제한
}
```
