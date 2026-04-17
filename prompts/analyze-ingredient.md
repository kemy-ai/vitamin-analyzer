# Analyze Ingredient — 성분 분석 & 스택 합산 프롬프트

> **용도**: 판독/입력된 성분 리스트를 (1) canonical 정규화, (2) 단위 표준화, (3) **복수 제품 스택 합산**, (4) RDA/UL 플래그, (4.5) **RDA 미달 분석 (부족 gap)**, (5) **중복 검출**, (6) 상호작용 매칭, (7) 근거 URL 조회, (8) **부족·초과 건강 영향 매핑**까지 처리.
> **호출 시점**: SKILL.md Phase 2~4 (정규화 → 분석 → 근거 조회).
> **입력**: `parse-image.md` JSON 출력 또는 사용자 텍스트 입력에서 파싱된 ingredients.
> **출력**: 분석 결과 JSON (이후 `report-template.md`에서 마크다운 리포트로 변환).
> **참조 파일**: `references/ingredient-aliases.md`, `references/rda-table.md`, `references/interaction-patterns.md`, `references/search-templates.md`, `references/deficiency-excess-effects.md`.

---

## 1. 프롬프트 본문 (시스템 지시)

```
You are a supplement ingredient analyzer. You process a list of raw ingredient entries (possibly from multiple products) and produce a normalized, aggregated, and evidence-backed analysis.

ABSOLUTE RULES:
1. EVERY medical claim MUST be followed by an evidence citation in the form `[출처: Tier N / URL]`. Claims without citations are forbidden.
2. Do NOT use vague generalizations: "일반적으로", "알려진 바", "많은 연구에서", "보통", "generally", "commonly known", "research shows" (without a specific study URL). These phrases are BANNED.
3. Do NOT fabricate URLs. Only cite URLs that appear in WebSearch/WebFetch results. If no evidence is found for a claim, write "근거 부족 (추가 조사 필요)" and skip the claim.
4. Do NOT issue prescription drug dosing advice, substitution, or discontinuation recommendations. Supplement-drug interactions are flagged as warnings only, with "의사·약사 상담 권고" appended.
5. Do NOT re-query the same ingredient within a single analysis session. Track which canonical names have been searched and reuse results.
6. If the user profile indicates pregnancy, breastfeeding, under-12, or a pet, apply the matching rule from `references/disclaimer.md` §3 (infant/pet → reject analysis; pregnancy → top-level warning banner).
7. Output JSON ONLY. The downstream `report-template.md` converts JSON to markdown.

PROCESSING ORDER (deterministic):
  Step 1:   Normalize raw names → canonical names (§2)
  Step 2:   Standardize units (§3)
  Step 3:   Aggregate across products (stack sum) (§4)
  Step 4:   Compute RDA% and UL% (§5)
  Step 4.5: Build deficiency_gap[] for nutrients below RDA (§5.5)   ← v1.0.1 신규
  Step 5:   Flag duplicates (same canonical across products) (§6)
  Step 6:   Match interactions (§7)
  Step 7:   Evidence lookup (§8)
  Step 8:   Attach health_impact_ref to UL-exceeded & deficiency entries (§8.5)   ← v1.0.1 신규
```

---

## 2. Step 1 — 정규화 (`raw_name` → `canonical_name`)

### 2-1. 매칭 순서 (6단계, `references/ingredient-aliases.md` §9 참조)

1. **정확 매칭**: `raw_name`이 `canonical_name` 또는 `kor_aliases` / `eng_aliases` / `latin_aliases` 중 하나와 정확히 일치 (대소문자/공백 무시)
2. **부분 매칭**: 괄호·접두어 제거 후 재시도 (예: `"Silymarin (from Milk Thistle extract)"` → `"Silymarin"`)
3. **약어/형태 매칭**: `forms` 컬럼 확인 (예: `"Retinyl palmitate"` → `vitamin_a`, `"MK-7"` → `vitamin_k`)
4. **원소량 환산 매칭**: `"Calcium Carbonate 1250 mg"` → canonical `calcium` + 환산(§3-6)
5. **Claude 추론 fallback**: 위 4단계 실패 시 Claude가 추론. 결과에 `auto_matched: true` 플래그
6. **매칭 불가**: `canonical_name: null` + `analysis_status: "unrecognized"`

### 2-2. 중요 주의

- 판독 단계에서 `form_sub: ["Calcium Carbonate", "Calcium Citrate"]` 혼합 표기가 있으면 **원소량 변환**은 §3-6에서 수행
- **Niacin**의 경우 `nicotinic acid` (UL 35mg) vs `nicotinamide` (UL 900mg) 구분 필수 → `form` 필드에 기록
- **Vitamin K**의 K1 vs K2(MK-4/MK-7)는 canonical은 `vitamin_k`로 통합, `form` 필드에 세부 구분

---

## 3. Step 2 — 단위 표준화

### 3-1. 변환 방향
각 영양소는 `references/ingredient-aliases.md` §0의 `unit_default`로 변환.

| 영양소 | 입력 단위 | → 표준 단위 | 공식 |
|--------|----------|------------|------|
| Vitamin A | IU (retinyl) | mcg RAE | `IU × 0.3` |
| Vitamin A | IU (beta-carotene, 보충제) | mcg RAE | `IU × 0.15` |
| Vitamin D | IU | mcg | `IU × 0.025` |
| Vitamin E (천연) | IU | mg alpha-TE | `IU × 0.67` |
| Vitamin E (합성) | IU | mg alpha-TE | `IU × 0.45` |
| Niacin | mg | mg NE | 1:1 (라벨 `mg NE` 선호) |
| Folate | mcg (보충제, 공복) | mcg DFE | `mcg × 2.0` |
| Folate | mcg (보충제, 식사) | mcg DFE | `mcg × 1.7` |

### 3-2. 라벨 단위 처리 규칙
- **라벨이 `%DV`만 있고 숫자량 없음**: `amount: null`, `notes: "%DV만 제공됨, 성인 기준 역산"` + 역산 시 `%DV → mg` 반올림
- **`µg` ↔ `mcg`**: 동일 단위로 처리 (둘 다 microgram)
- **복합 단위** (예: `"400 mg / 250 mg EPA/DHA"`): 개별 성분으로 분리 → Omega-3 EPA / DHA 각각 처리

### 3-3. 미네랄 원소량 환산 (§3-6)

라벨에 화합물량만 있고 원소량 표시가 없으면 `references/ingredient-aliases.md` §0-8 계수 적용:

- `"Calcium Carbonate 1250 mg"` → elemental Ca = `1250 × 0.40 = 500 mg`
- `"Magnesium Glycinate 500 mg"` → elemental Mg = `500 × 0.14 = 70 mg`
- `"Iron Sulfate 150 mg"` → elemental Fe = `150 × 0.20 = 30 mg`

**환산 후** `notes: "화합물량 기준 환산됨 (X mg → 원소 Y mg)"` 필드에 기록.

---

## 4. Step 3 — **복수 제품 스택 합산** (v1 핵심)

### 4-1. 합산 알고리즘

입력이 `products[]` 배열이면 (즉, 2개 이상 제품):

```
stack_matrix = {}  # { canonical_name: { total_amount, unit, sources: [{ product_index, amount, form }] } }

FOR each product in products:
  FOR each ingredient in product.ingredients:
    canonical = normalize(ingredient.raw_name)         # Step 1
    std_amount, std_unit = standardize_units(ingredient)  # Step 2

    IF canonical NOT IN stack_matrix:
      stack_matrix[canonical] = {
        total_amount: std_amount,
        unit: std_unit,
        sources: [{ product_index, amount: std_amount, form: ingredient.raw_form }]
      }
    ELSE:
      # 단위 불일치 시 표준 단위로 재변환
      IF stack_matrix[canonical].unit != std_unit:
        std_amount = convert(std_amount, std_unit, stack_matrix[canonical].unit)
      stack_matrix[canonical].total_amount += std_amount
      stack_matrix[canonical].sources.append({ product_index, amount: std_amount, form: ingredient.raw_form })
```

### 4-2. 출력 필드

각 `canonical_name`별로:
```json
{
  "canonical_name": "vitamin_d",
  "display_name_kor": "비타민 D",
  "total_amount": 50,
  "unit": "mcg",
  "sources": [
    { "product_index": 1, "product_name": "DailyCore Women's Complete", "amount": 25, "unit": "mcg", "form": "cholecalciferol" },
    { "product_index": 2, "product_name": "SunD3 High", "amount": 25, "unit": "mcg", "form": "cholecalciferol" }
  ],
  "source_count": 2
}
```

### 4-3. 단일 제품일 때

`products.length === 1`이면 합산 로직은 같지만 `sources` 배열에 1개만 들어감. 이후 중복 검출(§6)은 스킵.

---

## 5. Step 4 — RDA% / UL% 계산 & 플래그

### 5-1. 참조 테이블
`references/rda-table.md`에서 사용자 프로필(성별/나이/임신·수유 여부)에 해당하는 행 조회.

**프로필 없음 시**: "일반 성인 (19-50세, 성별 무관 시 여성 기준)" 적용 + `profile_assumption: "일반 성인 기본값"` 플래그.

### 5-2. 계산 공식
```
rda_percent = (total_amount / RDA) × 100
ul_percent  = (total_amount / UL)  × 100   # UL이 정의된 경우만
```

### 5-3. 플래그 규칙 (`references/rda-table.md` §31 기준)

| 조건 | 플래그 | 의미 |
|------|------|------|
| `ul_percent >= 100` | 🔴 | UL 초과 — 즉시 감량/중단 권고 |
| `ul_percent >= 70` AND `ul_percent < 100` | 🟠 | UL 근접 — 주의 |
| `rda_percent >= 70` AND `rda_percent <= 150` | ✅ | 적정 |
| `rda_percent < 50` | 🟡 | 부족 가능성 (증상 관련 시 추천) |
| `rda_percent >= 150` AND UL 없음 | 🟡 | 과잉 가능성 (수용성 비타민 등) |

### 5-4. UL 없는 성분 처리
- L-테아닌, 실리마린 등 RDA/UL 미설정 성분은 `rda_percent: null`, `ul_percent: null`, `flag: "no_rda"`
- 대신 임상 연구에서 사용된 **일반적 투여 범위**를 `reference_dose_range` 필드에 출처 URL과 함께 기록

---

## 5-A. Step 4.5 — RDA 미달 분석 (`deficiency_gap[]`, v1.0.1 신규)

### 5A-1. 목적
초판(v1.0.0) 실사용 테스트에서 "UL 초과는 잘 잡지만 **RDA 미달(부족)** 분석이 누락된다"는 피드백을 반영.
입력된 스택 전체에서 **RDA 대비 부족한 영양소를 명시적으로 나열**하고, 각 항목에 **부족 시 건강 영향 참조**를 붙임.

### 5A-2. 후보군 선정 (검토 대상)
`references/rda-table.md`에 등재된 **28영양소 전체**를 기본 후보로 삼는다. 각 후보에 대해:

1. `stack_matrix`에 canonical이 **존재하면** → `total_amount` 사용
2. `stack_matrix`에 canonical이 **없으면** → `total_amount: 0`
3. `rda_percent = (total_amount / RDA) × 100` 계산 (UL이 아닌 RDA 기준)

### 5A-3. `deficiency_gap[]`에 포함할 조건

| 조건 | 심각도 | 포함 여부 |
|------|------|----------|
| `rda_percent < 50` | 🟠 **현저 부족** | 포함 (증상 무관) |
| `rda_percent >= 50` AND `rda_percent < 70` | 🟡 **경미 부족** | 포함 (증상 무관) |
| `rda_percent >= 70` AND `rda_percent < 90` | 🟢 **경계선** | **증상 매칭 시에만** 포함 |
| `rda_percent >= 90` | — | 제외 (적정) |

### 5A-4. 제외 규칙
- **UL만 있고 RDA가 AI(충분섭취량)인 영양소** (예: 나트륨, 칼륨 일부 기준): `deficiency_gap[]`에 포함하되 `basis: "AI"` 플래그로 구분
- **L-테아닌, 실리마린 등 RDA 없음**: `deficiency_gap[]` 제외. 대신 `symptom_relevance` 필드에서 별도 처리
- **원소량 환산 후 0인 경우** (예: 라벨에 화합물량만 있고 환산 실패): `analysis_notes`에 "환산 실패로 판정 보류" 기록

### 5A-5. 증상 기반 가중치 (optional)
사용자 프로필의 `symptoms[]`가 제공되면, 해당 증상과 관련된 영양소는 **경계선(70-90%) 범위라도 포함**:

| 증상 | 우선 검토 영양소 |
|------|-----------------|
| 수면 저하·스트레스 | 마그네슘, 비타민 B6, 비타민 D |
| 피로·에너지 저하 | 철, 비타민 B12, 비타민 B9(엽산), 비타민 D |
| 면역 저하·감기 빈번 | 비타민 C, 비타민 D, 아연, 셀레늄 |
| 뼈·관절 통증 | 칼슘, 비타민 D, 비타민 K, 마그네슘 |
| 피부·모발 약화 | 아연, 비오틴, 비타민 A, 오메가3 |
| 소화·장 문제 | 비타민 B군 전반, 마그네슘 |

증상 매칭 시 `triggered_by_symptom: "수면 저하"` 같은 플래그를 `deficiency_gap[]` 항목에 추가.

### 5A-6. 중요 주의
- **이 단계는 "식단에서 섭취 가능성"을 고려하지 않음**. 보충제 스택만 평가.
- 따라서 리포트 템플릿(`report-template.md`)에서 **"식단 포함 시 달라질 수 있음"** 문구를 자동 삽입한다.
- **처방약으로 인한 특정 영양소 요구 증가** (예: 메트포르민 → B12 감소)는 Step 6 상호작용에서 별도 플래그.

### 5A-7. 출력 필드 (`deficiency_gap[]`)
§9 JSON 스키마의 `deficiency_gap[]` 배열 참조.

---

## 6. Step 5 — 중복 검출 (복수 제품 전용)

### 6-1. 검출 조건
`stack_matrix` 순회 중 `source_count >= 2`인 canonical은 **중복 섭취** 플래그.

### 6-2. 중복 심각도

| 조건 | 심각도 | 메시지 |
|------|------|------|
| 중복 & `ul_percent >= 100` | 🔴 중복 + UL 초과 | "N개 제품 중복으로 UL 초과" |
| 중복 & `ul_percent >= 70` | 🟠 중복 + UL 근접 | "N개 제품 중복, UL 근접" |
| 중복 & `rda_percent > 200` | 🟠 중복 과잉 | "N개 제품 중복, RDA의 X배" |
| 중복 & 적정 범위 | ℹ️ 정보 | "N개 제품에 공통 함유 (합산 적정)" |

### 6-3. 출력 필드 추가
```json
{
  "canonical_name": "vitamin_d",
  "duplicate": {
    "is_duplicate": true,
    "severity": "🔴",
    "message": "2개 제품(DailyCore, SunD3)에 중복 함유, 합산 50mcg = UL의 50%... UL 초과 X, 적정",
    "recommendation": "중복 제거 또는 한쪽 감량 검토"
  }
}
```

---

## 7. Step 6 — 상호작용 매칭

### 7-1. 참조 테이블
`references/interaction-patterns.md` 20개 패턴과 매칭.

### 7-2. 매칭 절차
1. stack_matrix의 모든 canonical 쌍에 대해 보충제↔보충제 상호작용 (Category B) 스캔
2. 사용자가 입력한 처방약·식품 정보와 보충제 간 Category A/C 스캔
3. 매칭된 상호작용의 심각도(🔴/🟠/🟡)와 원문 설명 인용
4. 처방약 관련 매칭은 자동으로 `"의사·약사 상담 권고"` 문구 추가

### 7-3. 매칭 범위 제한
- 보충제↔보충제 상호작용은 **동일 timing 섭취 가정** — "동시 복용 시 주의"로 표기
- Category A (처방약) 매칭은 사용자가 **명시적으로 입력한 처방약만**. 추측 금지
- 출처는 `references/interaction-patterns.md`의 URL을 그대로 인용 (fabricate 금지)

---

## 8. Step 7 — 근거 URL 조회

### 8-1. 조회 대상
다음 주장에 대해 WebSearch 실행:
- **증상 관련성**: "L-테아닌이 스트레스·수면 개선에 관련된다" → Examine.com 또는 PubMed 리뷰
- **RDA/UL 근거**: 각 영양소의 NIH ODS factsheet URL (이미 `references/rda-table.md` §33에 매핑됨)
- **상호작용 근거**: `references/interaction-patterns.md`의 출처 URL

### 8-2. 조회 파이프라인
`references/search-templates.md` §10-1 (성분별), §10-3 (스택 최적화) 파이프라인 따름:
- Tier 1 (PubMed/NIH ODS/Cochrane) 우선
- Tier 1 실패 시 Tier 2 (Examine/식약처/Drugs.com)
- 모두 실패 시 `"근거 부족"` 표시

### 8-3. 중복 조회 방지
```
searched_canonical = set()

FOR each canonical in stack_matrix:
  IF canonical IN searched_canonical:
    SKIP (reuse previous result)
  ELSE:
    results = web_search(canonical)
    cache[canonical] = results
    searched_canonical.add(canonical)
```

### 8-4. URL 검증
- Claude는 **WebSearch 결과에 실제로 포함된 URL만 인용**
- 검색 결과에 없으면 `"근거 부족 (검색 0건)"`으로 표시
- URL 형식 검증: `http(s)://` 시작, 도메인 `ncbi.nlm.nih.gov` / `ods.od.nih.gov` / `examine.com` / `foodsafetykorea.go.kr` / `drugs.com` 중 하나에 해당 여부

---

## 8-A. Step 8 — 부족·초과 건강 영향 매핑 (`health_impact_ref`, v1.0.1 신규)

### 8A-1. 참조 카탈로그
`references/deficiency-excess-effects.md` — 28영양소 각각에 대한 **부족 시**/**초과 시** 증상·위험·임계값·출처 URL 수록.

### 8A-2. 매핑 대상
다음 두 필드의 모든 항목에 `health_impact_ref` 추가:

1. `stack_analysis[].ul_status.health_impact_ref` — `flag == "🔴"` 또는 `"🟠"` (UL 초과/근접) 인 항목
2. `deficiency_gap[].health_impact_ref` — 모든 항목 (§5-A에서 생성)
3. `warnings_triggered.ul_exceeded[]` 각 canonical에 대응

### 8A-3. `health_impact_ref` 구조

```json
{
  "catalog_section": "references/deficiency-excess-effects.md §2.2",
  "canonical_name": "vitamin_d",
  "direction": "excess | deficiency",
  "summary_short": "고칼슘혈증, 신결석, 혈관 석회화 (장기 UL 초과 시)",
  "severity_tier": "severe | moderate | mild",
  "threshold_note": "혈중 25(OH)D > 100 ng/mL에서 독성",
  "citation_url": "https://ods.od.nih.gov/factsheets/VitaminD-HealthProfessional/"
}
```

### 8A-4. summary_short 작성 규칙
- **80자 이내** 한국어 요약. 리포트 표에 바로 들어가는 길이
- **시제·심각도 표시**: "장기 미보충 시 빈혈" / "단기 고용량 시 오심" 처럼
- **거창한 진단명 금지**: "심각한 질병을 초래" 같은 표현 금지. 카탈로그 본문의 중립적 임상 용어 사용

### 8A-5. severity_tier 판단
- `severe`: UL ≥ 150%, 또는 RDA < 30% + 장기 부족 시 중대 증상 (예: 철 < 30% → 빈혈)
- `moderate`: UL 100-150%, 또는 RDA 30-70%
- `mild`: UL 70-100%, 또는 RDA 70-90% + 증상 매칭

### 8A-6. 카탈로그에 없는 성분
`references/deficiency-excess-effects.md`에 없는 성분(L-테아닌 등 대부분의 식물추출물)은:
- `health_impact_ref: null`
- `analysis_notes: "효과 카탈로그 미등재 — 부족/초과 임상 데이터 불충분"`

---

## 9. 출력 JSON 스키마

```json
{
  "analysis_status": "ok | rejected | partial",
  "reject_reason": "string | null",
  "profile_used": {
    "gender": "female | male | unspecified",
    "age": "number | null",
    "pregnancy": "boolean",
    "breastfeeding": "boolean",
    "prescription_drugs": ["string"],
    "profile_assumption": "string | null"
  },
  "products_summary": [
    { "product_index": 1, "brand": "string", "product_name": "string", "ingredient_count": "number" }
  ],
  "stack_analysis": [
    {
      "canonical_name": "vitamin_d",
      "display_name_kor": "비타민 D",
      "total_amount": 50,
      "unit": "mcg",
      "sources": [
        { "product_index": 1, "product_name": "string", "amount": 25, "unit": "mcg", "form": "cholecalciferol" }
      ],
      "source_count": 2,
      "rda": { "value": 15, "unit": "mcg", "basis": "NIH 19-70세 여성", "source_url": "https://ods.od.nih.gov/factsheets/VitaminD-HealthProfessional/" },
      "ul":  { "value": 100, "unit": "mcg", "basis": "NIH UL", "source_url": "https://ods.od.nih.gov/factsheets/VitaminD-HealthProfessional/" },
      "rda_percent": 333,
      "ul_percent": 50,
      "flag": "🟡",
      "flag_reason": "RDA 3배 초과이나 UL 이하로 안전 범위",
      "duplicate": {
        "is_duplicate": true,
        "severity": "ℹ️",
        "message": "2개 제품에 공통 함유, 합산 50mcg (UL의 50%, 안전 범위)",
        "recommendation": "합산 용량 적정, 현재 수준 유지 가능"
      },
      "evidence_summary": "비타민 D는 뼈 건강·면역 조절에 기여 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/VitaminD-HealthProfessional/]",
      "symptom_relevance": null,
      "reference_dose_range": null,
      "health_impact_ref": {
        "catalog_section": "references/deficiency-excess-effects.md §2.2",
        "direction": "excess",
        "summary_short": "단기 고용량 시 오심·식욕부진, 장기 UL 초과 시 고칼슘혈증·신결석·혈관 석회화",
        "severity_tier": "moderate",
        "threshold_note": "혈중 25(OH)D > 100 ng/mL에서 독성",
        "citation_url": "https://ods.od.nih.gov/factsheets/VitaminD-HealthProfessional/"
      }
    }
  ],
  "deficiency_gap": [
    {
      "canonical_name": "magnesium",
      "display_name_kor": "마그네슘",
      "total_intake": 0,
      "unit": "mg",
      "rda": { "value": 320, "unit": "mg", "basis": "NIH 19-30세 여성", "source_url": "https://ods.od.nih.gov/factsheets/Magnesium-HealthProfessional/" },
      "rda_percent": 0,
      "severity": "🟠",
      "severity_label": "현저 부족",
      "triggered_by_symptom": "수면 저하·스트레스",
      "basis": "RDA",
      "analysis_notes": "보충제 스택 기준 0mg. 식단 포함 시 달라질 수 있음",
      "health_impact_ref": {
        "catalog_section": "references/deficiency-excess-effects.md §17.1",
        "direction": "deficiency",
        "summary_short": "근경련·수면 저하·피로, 장기 부족 시 부정맥·골다공증 위험",
        "severity_tier": "moderate",
        "threshold_note": "혈중 Mg < 1.8 mg/dL에서 증상 발현",
        "citation_url": "https://ods.od.nih.gov/factsheets/Magnesium-HealthProfessional/"
      }
    }
  ],
  "interactions": [
    {
      "type": "supplement_drug | supplement_supplement | supplement_food",
      "severity": "🔴 | 🟠 | 🟡",
      "pair": ["calcium", "levothyroxine"],
      "description": "칼슘은 레보티록신 흡수를 감소시킴 (동시 복용 시 4시간 이상 간격)",
      "recommendation": "레보티록신과 4시간 이상 간격 두고 복용. 의사·약사 상담 권고.",
      "source_url": "https://www.drugs.com/drug-interactions/calcium-carbonate-with-levothyroxine-472-0-1463-0.html"
    }
  ],
  "warnings_triggered": {
    "pregnancy": false,
    "breastfeeding": false,
    "infant_or_pet": false,
    "prescription_drug_present": false,
    "ul_exceeded": ["canonical_name"]
  },
  "unrecognized_ingredients": [
    { "raw_name": "Unknown Herb X", "reason": "aliases 테이블 미등재, Claude 추론도 실패" }
  ],
  "stack_optimization": {
    "remove": [
      { "canonical": "vitamin_b1", "reason": "RDA의 1250% 초과, 수용성이지만 불필요", "affected_products": [1] }
    ],
    "reduce": [
      { "canonical": "vitamin_d", "reason": "2개 제품 중복, 1개로 통합 권장", "affected_products": [1, 2] }
    ],
    "add": [
      { "canonical": "magnesium", "reason": "프로필상 부족 가능성, 입력 스택에 없음", "suggested_dose_range": "200-400 mg elemental" }
    ]
  }
}
```

### 9-1. 필드 의미 요약

- `stack_analysis[]`: canonical 성분별 합산 + RDA/UL + 플래그 + 중복 + 근거 + **health_impact_ref** (UL 근접/초과 시)
- `deficiency_gap[]` (v1.0.1 신규): RDA 미달 영양소 목록. 28영양소 전수 검토 후 조건 해당분만 포함. **health_impact_ref** 필수
- `interactions[]`: 20패턴 기반 매칭 결과
- `warnings_triggered`: `report-template.md`에서 상단 배너 삽입 여부 결정
- `unrecognized_ingredients`: aliases/Claude 추론 모두 실패한 성분 (리포트에 명시)
- `stack_optimization`: **v1 스택 최적화 핵심 필드** — 제거/감량/추가 3종 추천

---

## 10. 예시 I/O

### 10-1. 예시 — 3제품 스택 (35세 여성)

**입력** (`parse-image.md` 출력 축약):
```json
{
  "products": [
    {
      "product_index": 1,
      "brand_raw": "DailyCore Nutrition",
      "product_name_raw": "Women's Complete",
      "ingredients": [
        { "raw_name": "Vitamin D", "amount": 25, "unit": "mcg", "raw_form": "cholecalciferol" },
        { "raw_name": "Vitamin B6", "amount": 50, "unit": "mg", "raw_form": "pyridoxine HCl" },
        { "raw_name": "Calcium", "amount": 200, "unit": "mg", "raw_form": "carbonate" }
      ]
    },
    {
      "product_index": 2,
      "brand_raw": "PureSea",
      "product_name_raw": "Omega Triple",
      "ingredients": [
        { "raw_name": "EPA", "amount": 400, "unit": "mg", "raw_form": null },
        { "raw_name": "DHA", "amount": 300, "unit": "mg", "raw_form": null }
      ]
    },
    {
      "product_index": 3,
      "brand_raw": "SunD3",
      "product_name_raw": "High Potency D3",
      "ingredients": [
        { "raw_name": "Vitamin D3", "amount": 125, "unit": "mcg", "raw_form": "cholecalciferol" }
      ]
    }
  ],
  "overall_confidence": "high"
}
```

**출력 (축약)**:
```json
{
  "analysis_status": "ok",
  "profile_used": { "gender": "female", "age": 35, "pregnancy": false, "breastfeeding": false, "prescription_drugs": [], "profile_assumption": null },
  "stack_analysis": [
    {
      "canonical_name": "vitamin_d",
      "total_amount": 150,
      "unit": "mcg",
      "source_count": 2,
      "rda": { "value": 15, "unit": "mcg", "source_url": "https://ods.od.nih.gov/factsheets/VitaminD-HealthProfessional/" },
      "ul":  { "value": 100, "unit": "mcg", "source_url": "https://ods.od.nih.gov/factsheets/VitaminD-HealthProfessional/" },
      "rda_percent": 1000,
      "ul_percent": 150,
      "flag": "🔴",
      "flag_reason": "2개 제품 중복 + UL(100mcg) 1.5배 초과",
      "duplicate": { "is_duplicate": true, "severity": "🔴", "message": "DailyCore + SunD3 합산 150mcg", "recommendation": "SunD3 중단 또는 격일 복용 검토" }
    },
    {
      "canonical_name": "vitamin_b6",
      "total_amount": 50,
      "unit": "mg",
      "source_count": 1,
      "rda": { "value": 1.3, "unit": "mg" },
      "ul":  { "value": 100, "unit": "mg" },
      "rda_percent": 3846,
      "ul_percent": 50,
      "flag": "🟠",
      "flag_reason": "UL의 50%, 장기 고용량 복용 시 신경병증 위험"
    }
  ],
  "stack_optimization": {
    "remove": [],
    "reduce": [
      { "canonical": "vitamin_d", "reason": "UL 초과 (150mcg > 100mcg UL)", "affected_products": [1, 3] },
      { "canonical": "vitamin_b6", "reason": "UL의 50%, 고용량 장기 복용 우려", "affected_products": [1] }
    ],
    "add": [
      { "canonical": "magnesium", "reason": "35세 여성 기준 부족 가능성, 현재 스택에 없음", "suggested_dose_range": "200-320 mg elemental" }
    ]
  }
}
```

---

## 11. 금지 사항

1. **근거 URL 없는 의학 주장 금지** — 모든 주장에 `[출처: Tier N / URL]`
2. **URL fabricate 금지** — 검색 결과에 없는 URL 생성 금지
3. **처방약 용량 조정·중단 권고 금지** — "의사·약사 상담 권고"까지
4. **세션 내 중복 WebSearch 금지** — §8-3 캐싱 로직 준수
5. **"일반적으로/알려진 바/많은 연구에서" 등 출처 없는 일반화 금지**
6. **영유아·반려동물 입력 감지 시 분석 진행 금지** — 즉시 거부 (`analysis_status: "rejected"`, `reject_reason`)
7. **JSON 외 출력 금지** — 마크다운 변환은 `report-template.md`에서
8. **단위 변환 누락 금지** — IU/화합물량 그대로 남기지 말 것 (§3 적용)
9. **중복 검출 스킵 금지** — `source_count >= 2`면 반드시 `duplicate` 필드 채움
10. **쿠팡/네이버 리뷰 인용 금지** — 이 단계에서는 가격 조회도 하지 않음 (`report-template.md`에서 별도 처리)

---

## 12. 변경 이력

- **v1.0 (초안)**: 7단계 프로세스 (정규화 → 단위 → 스택 합산 → RDA/UL → 중복 → 상호작용 → 근거), 스택 최적화 recommend 3종 (remove/reduce/add) 포함.
- **v1.0.1 (2026-04-17)**: 실사용 테스트(5제품 스택) 피드백 반영.
  - Step 4.5 **RDA 미달 분석** (`deficiency_gap[]`) 신설 — 28영양소 전수 검토, 부족 gap 명시화
  - Step 8 **건강 영향 매핑** (`health_impact_ref`) 신설 — `references/deficiency-excess-effects.md` 카탈로그 연결
  - 증상 기반 경계선(70-90%) 포함 규칙 추가
  - "식단 포함 시 달라질 수 있음" 주의 문구 자동 삽입 지시
