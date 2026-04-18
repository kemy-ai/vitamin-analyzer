# User-Data Schema

> vitamin-analyzer 스킬이 사용자 워크스페이스에 생성·읽는 JSON 파일의 스키마 정의.
> 범용 배포 원칙: 사용자가 프로필명을 직접 결정. 특정 개인의 이름이나 식별자를 하드코딩 금지.

---

## 디렉토리 구조

```
./user-data/
├── profiles/
│   ├── _index.json               # 프로필 목록 + 별칭 매핑
│   └── {profile-id}.json         # 개별 프로필 (사용자 명명)
├── products/
│   └── {brand}_{product}.json    # 성분 DB (프로필 간 공유)
├── stacks/
│   └── {profile-id}_latest.json  # 프로필별 현재 스택 + 파싱 원본
└── reports/
    └── {profile-id}/
        ├── YYYY-MM-DD_HHMM_analysis.md
        └── latest.md              # 최신 리포트 심볼릭 링크
```

**`profile-id`** / **`product_key`** 생성 규칙:
- 영소문자 + 숫자 + 언더스코어만 허용 (`[a-z0-9_]+`)
- 한글 프로필명 → 로마자 slug 또는 사용자 지정 ID
  - 예: "엄마" → `mom` / "아내" → `wife` / "친구 A" → `friend_a`
- `product_key`는 `{brand_slug}_{product_slug}` 조합
  - 예: Example Brand ProbioDaily Gold → `example_probiodaily_gold`

---

## 1. `profiles/_index.json`

프로필 목록 + 관계 별칭 매핑. 새 프로필 추가·삭제 시 갱신.

```json
{
  "schema_version": "1.1.0",
  "default_profile_id": "default",
  "profiles": [
    {
      "profile_id": "default",
      "display_name": "기본",
      "aliases": ["본인", "나", "내"],
      "relationship": "self",
      "created_at": "2026-04-17",
      "last_analyzed_at": null
    }
  ]
}
```

**필드**:
- `schema_version` (string, required): 스키마 버전
- `default_profile_id` (string, required): 사용자 발화에 특정 지시어가 없을 때 적용할 기본 프로필
- `profiles[]` (array, required):
  - `profile_id` (string, required): 파일명 slug
  - `display_name` (string, required): 사용자가 부르는 이름 (한글 허용)
  - `aliases[]` (string array): 발화 감지용 관계 키워드 ("엄마", "어머니" 등)
  - `relationship` (enum): `"self" | "family" | "friend" | "pet_owner_proxy" | "other"` — 관계 유형
  - `created_at` / `last_analyzed_at` (ISO date)

---

## 2. `profiles/{profile-id}.json`

개별 프로필의 인구통계 + 건강 맥락 + 선호.

```json
{
  "profile_id": "default",
  "display_name": "기본",
  "relationship": "self",
  "created_at": "2026-04-17",
  "updated_at": "2026-04-17",

  "demographics": {
    "age": 42,
    "biological_sex": "male",
    "is_pregnant": false,
    "is_lactating": false,
    "weight_kg": null,
    "height_cm": null
  },

  "health_context": {
    "symptoms": [],
    "conditions": [],
    "medications": [],
    "diet_notes": null,
    "allergies": []
  },

  "preferences": {
    "budget_monthly_krw": null,
    "preferred_source": "auto",
    "language": "ko"
  },

  "consent": {
    "disclaimer_acknowledged": true,
    "acknowledged_at": "2026-04-17"
  },

  "notes": ""
}
```

**필드 상세**:
- `demographics.biological_sex`: `"male" | "female"` — RDA/UL 룩업용. 성별 정체성과 무관, 생물학적 기준
- `demographics.age`: 정수. **<12세는 분석 거부** (CLAUDE.md §2-1)
- `demographics.is_pregnant` / `is_lactating`: `true`면 경고 배너 최상위 삽입
- `health_context.conditions[]`: 기저질환 문자열 배열 (예: `["고혈압", "당뇨"]`) — 참고용, 진단 아님
- `health_context.medications[]`: 처방약 이름 배열 — 보충제↔약물 상호작용 체크용
- `preferences.preferred_source`: `"domestic" | "international" | "auto"` — 대체제품 제안 편향
- `preferences.language`: `"ko" | "en"` — 리포트 출력 언어
- `consent.disclaimer_acknowledged`: 면책 조항 동의 여부 (첫 분석 시 1회 확인)

**프라이버시 원칙**:
- 이 파일은 로컬에만 저장. 스킬이 외부 API에 **원문 전송 금지**
- WebSearch 쿼리에는 비식별화된 정보만 사용 (예: 연령대, 성별, 일반 증상 키워드)

---

## 3. `products/{brand}_{product}.json`

성분 DB. 프로필 간 공유 (성분은 프로필 무관).

```json
{
  "product_key": "example_probiodaily_gold",
  "brand": "Example Brand",
  "product_name": "ProbioDaily Gold",
  "aliases": ["ProbioDaily", "Example ProbioDaily Gold"],
  "category": "probiotics",

  "serving": {
    "unit": "포",
    "per_day_default": 1,
    "amount_per_serving_g": 2.0
  },

  "ingredients": [
    {
      "name_ko": "프로바이오틱스",
      "name_en": "Probiotics",
      "alias_id": "probiotics",
      "amount": 1000000000,
      "unit": "CFU",
      "form": null,
      "basis": "per_serving"
    }
  ],

  "source": {
    "type": "label_photo",
    "url": null,
    "verified_at": "2026-04-17",
    "confidence": "high"
  },

  "notes": ""
}
```

**필드 상세**:
- `product_key` (string, required): 파일명과 동일한 slug. 검색 키
- `category` (enum): `"multivitamin" | "single_vitamin" | "mineral" | "omega3" | "probiotics" | "herbal" | "amino_acid" | "other"`
- `serving.per_day_default`: 라벨에 표기된 권장 복용량. 사용자 실제 복용과 다를 수 있음 (스택에서 개별 지정)
- `ingredients[].alias_id`: `ingredient-aliases.md`의 표준 ID와 매핑 (정규화 키)
- `ingredients[].unit`: `"mg" | "µg" | "IU" | "g" | "CFU" | "NE" | "DFE" | "RAE"` 등. `rda-table.md` 단위 규칙 따름
- `ingredients[].form` (nullable): 흡수율 구분 (예: `"oxide"`, `"glycinate"`, `"methylfolate"`) — `bioavailability-table.md` 참조
- `ingredients[].basis`: `"per_serving" | "per_day_recommended"` — 기준점 명시
- `source.type`: `"label_photo" | "official_site" | "shopping_mall" | "kfda" | "iherb" | "perplexity" | "websearch" | "user_text"` — Fallback Chain 단계
- `source.confidence`: `"high"` (라벨 사진 / 공식몰 텍스트) / `"medium"` (쇼핑몰 / WebSearch) / `"low"` (추정)

**DB 갱신 규칙**:
- 동일 `product_key` 재조회 시 `verified_at`이 **365일(1년) 이내**면 재사용, 초과 시 재크롤링
- 기간과 무관하게 강제 재조회되는 조건:
  - `source.confidence: "low"` 데이터
  - 사용자가 "리뉴얼된 것 같다" / "성분 바뀐 것 같다" 등 변동 가능성 언급
  - 사용자가 새 라벨 사진을 제공한 경우
- `confidence: "low"`는 사용자에게 "재확인 권장" 배너 표시

---

## 4. `stacks/{profile-id}_latest.json`

프로필별 현재 복용 스택 + 파싱된 원본 수치. `/compact` 대비 백업 용도.

```json
{
  "profile_id": "default",
  "captured_at": "2026-04-17T17:33:00+09:00",
  "skill_version": "1.1.0",

  "products": [
    {
      "product_key": "example_probiodaily_gold",
      "display_name": "Example Brand ProbioDaily Gold",
      "intake": {
        "amount_per_day": 1,
        "unit": "포",
        "notes": null
      }
    }
  ],

  "aggregated_nutrients": [
    {
      "nutrient_id": "vitamin_d",
      "name_ko": "비타민 D",
      "total_amount": 125,
      "unit": "µg",
      "contributors": [
        {"product_key": "mobita_d3", "amount": 125, "unit": "µg"}
      ],
      "rda": 15,
      "ul": 100,
      "ul_basis": "total",
      "percent_rda": 833,
      "percent_ul": 125,
      "status": "ul_exceeded"
    }
  ],

  "report_path": "./user-data/reports/default/2026-04-17_1733_analysis.md",
  "last_report_symlink": "./user-data/reports/default/latest.md"
}
```

**필드 상세**:
- `products[].intake`: 사용자가 실제로 복용하는 양. 라벨 권장량과 다를 수 있음 (예: 종합비타민 라벨 "2정/회" → 사용자 실제 "1정/일")
- `aggregated_nutrients[].status` (enum):
  - `"severe_deficient"` (<50% RDA)
  - `"mild_deficient"` (50-70% RDA)
  - `"borderline"` (70-90% RDA)
  - `"adequate"` (90-150% RDA)
  - `"above_rda"` (150% RDA ~ UL)
  - `"ul_exceeded"` (>100% UL)
- `aggregated_nutrients[].ul_basis`: `"total" | "supplement_only"` — L7 교훈 반영
  - NIH가 "보충제에서만" UL을 지정한 경우 `"supplement_only"` (예: 마그네슘 350mg)
  - 식단 포함 UL은 `"total"`

---

## 5. `reports/{profile-id}/latest.md`

심볼릭 링크. 항상 최신 리포트를 가리킴. 재분석 시 Before/After 비교 용이.

```bash
# 생성 예 (Phase 6 자동 실행)
ln -sfn "./user-data/reports/default/2026-04-17_1733_analysis.md" \
         "./user-data/reports/default/latest.md"
```

**저장 실패 대응**:
- 쓰기 권한 없음 / 디스크 풀 등 실패 시 리포트 최상단에 배너:
  ```
  > ⚠️ 자동 저장 실패 — 본문을 직접 복사해주세요. (원인: <에러 요약>)
  ```
- `stacks/{profile-id}_latest.json` 저장이 우선 (원본 수치 백업이 가장 중요)

---

## 6. 파일 명명 규칙 요약

| 파일 | 패턴 | 예 |
|------|------|-----|
| 프로필 | `profiles/{profile_id}.json` | `profiles/default.json`, `profiles/mom.json` |
| 프로필 인덱스 | `profiles/_index.json` | (고정) |
| 제품 | `products/{brand_slug}_{product_slug}.json` | `products/example_probiodaily_gold.json` |
| 스택 | `stacks/{profile_id}_latest.json` | `stacks/default_latest.json` |
| 리포트 | `reports/{profile_id}/{YYYY-MM-DD_HHMM}_analysis.md` | `reports/default/2026-04-17_1733_analysis.md` |
| 최신 링크 | `reports/{profile_id}/latest.md` | `reports/default/latest.md` |

---

## 7. 최초 실행 시 동작

프로필 파일이 없으면 `default` 프로필을 자동 생성 (빈 demographics — 분석 시 사용자에게 질문).
