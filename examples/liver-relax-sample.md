# Example 1 — 단일 제품 (간 기능 복합제)

> **용도**: `vitamin-analyzer` Skill 1차 테스트 케이스. 단일 제품 입력 → 성분 정규화 → RDA/UL 평가 → 상호작용 검출 → 리포트 생성 플로우 검증.
> **페르소나**: 32세 여성, 가상 인물 (실제 사용자 개인정보 배제). 간헐적 음주 + 스트레스 호소.
> **대상 제품**: 가상 브랜드 **NutriCalm Labs**의 가상 제품 **LiverRelax Complex** (텍스트 입력).
> **테스트 포커스**: (a) 한/영 성분명 정규화 (b) B군 고용량 vs UL 플래그 (c) 실리마린↔CYP 상호작용 일반 안내 (d) 대체제품 국내 5 + 해외 3.

---

## 1. 테스트 입력

### 1-1. 사용자 프로필 (Phase 1 응답)

```yaml
gender: female
age: 32
pregnancy: none
breastfeeding: no
prescriptions: none       # 처방약 복용 없음
conditions: none
symptoms:
  - 간헐적 음주 (주 1-2회)
  - 업무 스트레스 호소
  - 수면 질 저하
goals:
  - 간 기능 지원
  - 스트레스·수면 완화
country: KR
language: ko
```

### 1-2. 제품 입력 (Phase 2 텍스트 경로)

사용자 입력:
```
NutriCalm Labs LiverRelax Complex 분석해줘. 1정당 실리마린(밀크씨슬 추출물) 130mg, L-테아닌 200mg, 비타민 B1 15mg, 비타민 B2 15mg, 비타민 B6 15mg, 나이아신(니코틴아미드) 15mg NE. 하루 1정 아침에.
```

---

## 2. 예상 Phase 2 JSON (parse 결과 — 텍스트 경로 축약)

텍스트 입력이므로 `parse-image.md` 스키마를 그대로 채움. confidence는 `high`.

```json
{
  "parse_status": "success",
  "products": [
    {
      "brand_raw": "NutriCalm Labs",
      "product_name_raw": "LiverRelax Complex",
      "serving_size_raw": "1 tablet",
      "ingredients": [
        {"raw_name": "실리마린(밀크씨슬 추출물)", "raw_form": null, "amount": 130, "unit": "mg", "percent_dv": null, "form_sub": [], "confidence": "high", "notes": null},
        {"raw_name": "L-테아닌", "raw_form": null, "amount": 200, "unit": "mg", "percent_dv": null, "form_sub": [], "confidence": "high", "notes": null},
        {"raw_name": "비타민 B1", "raw_form": null, "amount": 15, "unit": "mg", "percent_dv": null, "form_sub": [], "confidence": "high", "notes": null},
        {"raw_name": "비타민 B2", "raw_form": null, "amount": 15, "unit": "mg", "percent_dv": null, "form_sub": [], "confidence": "high", "notes": null},
        {"raw_name": "비타민 B6", "raw_form": null, "amount": 15, "unit": "mg", "percent_dv": null, "form_sub": [], "confidence": "high", "notes": null},
        {"raw_name": "나이아신(니코틴아미드)", "raw_form": "니코틴아미드", "amount": 15, "unit": "mg NE", "percent_dv": null, "form_sub": ["nicotinamide"], "confidence": "high", "notes": null}
      ]
    }
  ],
  "overall_confidence": "high",
  "requires_user_confirmation": false,
  "unclear_fields": []
}
```

---

## 3. 예상 Phase 3 JSON (analyze 결과 — 축약)

```json
{
  "analysis_status": "ok",
  "profile_summary": "32세 여성 / 임신·수유 없음 / 처방약 없음 / 한국",
  "warnings_triggered": {
    "infant_or_pet": false,
    "pregnancy": false,
    "breastfeeding": false,
    "prescription_drug_present": false,
    "ul_exceeded": []
  },
  "stack_matrix": {
    "silymarin":   {"total_amount": 130, "unit": "mg", "sources": [{"product": "LiverRelax Complex", "amount": 130, "unit": "mg"}]},
    "l_theanine":  {"total_amount": 200, "unit": "mg", "sources": [{"product": "LiverRelax Complex", "amount": 200, "unit": "mg"}]},
    "thiamine":    {"total_amount": 15,  "unit": "mg", "sources": [{"product": "LiverRelax Complex", "amount": 15, "unit": "mg"}]},
    "riboflavin":  {"total_amount": 15,  "unit": "mg", "sources": [{"product": "LiverRelax Complex", "amount": 15, "unit": "mg"}]},
    "pyridoxine":  {"total_amount": 15,  "unit": "mg", "sources": [{"product": "LiverRelax Complex", "amount": 15, "unit": "mg"}]},
    "niacin":      {"total_amount": 15,  "unit": "mg NE", "form_sub": ["nicotinamide"], "sources": [{"product": "LiverRelax Complex", "amount": 15, "unit": "mg NE"}]}
  },
  "per_ingredient_eval": [
    {"canonical": "silymarin",  "total": 130, "unit": "mg", "rda_pct": null, "ul_pct": null, "flag": "ℹ️", "ref_range": "140-420 mg/day", "evidence": [{"tier": 2, "url": "https://www.nccih.nih.gov/health/milk-thistle"}]},
    {"canonical": "l_theanine", "total": 200, "unit": "mg", "rda_pct": null, "ul_pct": null, "flag": "ℹ️", "ref_range": "100-400 mg/day",  "evidence": [{"tier": 2, "url": "https://examine.com/supplements/theanine/"}]},
    {"canonical": "thiamine",   "total": 15,  "unit": "mg", "rda_pct": 1364, "ul_pct": null, "flag": "🟡", "note": "수용성 고용량, UL 미설정이나 RDA 대비 13배", "evidence": [{"tier": 1, "url": "https://ods.od.nih.gov/factsheets/Thiamin-HealthProfessional/"}]},
    {"canonical": "riboflavin", "total": 15,  "unit": "mg", "rda_pct": 1250, "ul_pct": null, "flag": "🟡", "note": "UL 미설정, 소변 황색 변화 가능",        "evidence": [{"tier": 1, "url": "https://ods.od.nih.gov/factsheets/Riboflavin-HealthProfessional/"}]},
    {"canonical": "pyridoxine", "total": 15,  "unit": "mg", "rda_pct": 1071, "ul_pct": 15,   "flag": "✅", "note": "UL 100 mg의 15%, 단일 제품 복용 기준 안전. 장기 50 mg+ 복용 회피",
     "evidence": [{"tier": 1, "url": "https://ods.od.nih.gov/factsheets/VitaminB6-HealthProfessional/"}]},
    {"canonical": "niacin",     "total": 15,  "unit": "mg NE", "rda_pct": 107, "ul_pct": null, "flag": "✅", "note": "니코틴아미드 형태이므로 KDRI UL 1000 mg 기준 적용. 홍조형(nicotinic acid) 아님",
     "evidence": [{"tier": 1, "url": "https://ods.od.nih.gov/factsheets/Niacin-HealthProfessional/"}]}
  ],
  "duplicates": [],
  "interactions": [
    {
      "severity": "ℹ️",
      "type": "supplement_drug_reference_only",
      "pair": "Silymarin ↔ CYP3A4/2C9 기질 약물",
      "summary": "현재 프로필에 해당 처방약 없음 — 정보성 안내. 향후 스타틴(심바스타틴·아토르바스타틴), 벤조디아제핀, 와파린, 일부 면역억제제를 추가 복용하게 되면 실리마린 유지 여부를 의사·약사와 상담 권고",
      "evidence": [
        {"tier": 2, "url": "https://www.nccih.nih.gov/health/milk-thistle"},
        {"tier": 1, "url": "https://pubmed.ncbi.nlm.nih.gov/?term=silymarin+CYP3A4+pharmacokinetic"}
      ]
    }
  ],
  "stack_optimization": {
    "remove": [],
    "reduce": [
      {"canonical": "thiamine",   "reason": "RDA의 1364%. 수용성이라 급성 위험은 낮지만 실질적 이득 근거 부족", "target": "1-2 mg 수준의 B컴플렉스로 교체 검토"},
      {"canonical": "riboflavin", "reason": "RDA의 1250%. 동일 사유", "target": "1-2 mg 수준으로 교체 검토"}
    ],
    "add": [
      {"canonical": "magnesium", "reason": "32세 여성 KDRI RDA 280 mg / NIH RDA 320 mg. 스트레스·수면 질 저하 호소 프로필에 부합", "target": "200-320 mg (Glycinate 또는 Citrate)",
       "evidence": [{"tier": 1, "url": "https://ods.od.nih.gov/factsheets/Magnesium-HealthProfessional/"}]}
    ]
  },
  "unrecognized_ingredients": []
}
```

---

## 4. 예상 최종 리포트 (마크다운)

````markdown
# 보충제 분석 리포트

**분석 대상**: NutriCalm Labs LiverRelax Complex
**프로필**: 32세 여성 / 임신·수유 없음 / 처방약 없음 / 한국
**분석 일자**: 2026-04-17

## TL;DR
- UL 초과 0건, UL 근접 0건, 수용성 B군 고용량 2건 (B1·B2 RDA의 12-14배). 전반 스택 안전 범위
- ℹ️ 실리마린↔CYP3A4/2C9 상호작용은 현재 처방약 없어 즉시 위험 없음 — 향후 스타틴·항응고제 복용 시 의사 상담 권고
- 우선 개선: 수면·스트레스 호소에 대응해 마그네슘 200-320 mg 추가 고려

## 성분별 분석

| 성분 | 합산량 | RDA% | UL% | 플래그 | 코멘트 [출처] |
|------|--------|------|------|--------|-----------|
| 실리마린 | 130 mg | — | — | ℹ️ | 임상 연구 권장 범위 140-420 mg/day의 하단 근처. 간기능 지원 용량 [출처: Tier 2 / https://www.nccih.nih.gov/health/milk-thistle] |
| L-테아닌 | 200 mg | — | — | ℹ️ | 연구 용량 100-400 mg/day 범위 내, 이완·수면 질 연구 다수 [출처: Tier 2 / https://examine.com/supplements/theanine/] |
| 비타민 B1 | 15 mg | 1364% | — | 🟡 | UL 미설정 수용성. RDA 대비 14배, 실질적 추가 이득 근거 부족 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Thiamin-HealthProfessional/] |
| 비타민 B2 | 15 mg | 1250% | — | 🟡 | UL 미설정 수용성. 소변 황색 변화는 정상 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Riboflavin-HealthProfessional/] |
| 비타민 B6 | 15 mg | 1071% | 15% | ✅ | UL 100 mg의 15%, 단일 제품 복용 기준 안전. 다른 B6 함유 제품 병용 시 합산 모니터링 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/VitaminB6-HealthProfessional/] |
| 나이아신 (니코틴아미드) | 15 mg NE | 107% | — | ✅ | 니코틴아미드 형태이므로 KDRI UL 1000 mg 기준 적용. 홍조형(nicotinic acid)이 아니어서 홍조 위험 낮음 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Niacin-HealthProfessional/] |

**플래그 범례**: 🔴 UL 초과/고위험 · 🟠 UL 근접/주의 · 🟡 부족 또는 과잉 가능성 · ✅ 적정 · ℹ️ RDA 미설정

## 상호작용 경고

- ℹ️ **실리마린 ↔ CYP3A4/2C9 기질 약물 (정보성)**: 현재 프로필에 해당 처방약 없음. 실리마린은 시험관·동물실험에서 CYP3A4/2C9/P-gp 억제가 보고되어 스타틴(심바스타틴·아토르바스타틴), 벤조디아제핀, 와파린, 일부 면역억제제 등의 농도에 영향 가능성이 있음. 향후 이러한 약물이 추가되면 **의사·약사 상담 권고** [출처: Tier 2 / https://www.nccih.nih.gov/health/milk-thistle] [출처: Tier 1 / https://pubmed.ncbi.nlm.nih.gov/?term=silymarin+CYP3A4+pharmacokinetic]

## 스택 최적화 추천

### 📉 감량/통합 권장
- **비타민 B1 (15 mg)** — RDA 1364%. 수용성이라 급성 독성 위험은 낮지만 과잉 섭취의 뚜렷한 이득 근거 부족. 저용량(1-2 mg) 또는 B컴플렉스 교체 검토.
- **비타민 B2 (15 mg)** — 동일 사유. 1-2 mg 수준 제품으로 교체 검토.
  - [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Thiamin-HealthProfessional/]
  - [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Riboflavin-HealthProfessional/]

### ➕ 추가 고려
- **마그네슘** — 32세 여성 RDA 280 mg (KDRI) / 320 mg (NIH) 기준 현재 스택에 없음. 수면 질 저하·스트레스 호소 프로필에 부합. 200-320 mg (Glycinate 또는 Citrate 형태) 추가 고려.
  - [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Magnesium-HealthProfessional/]

## 대체 제품 비교

> ※ 아래 표의 제품명·브랜드·가격은 **예시**이며, 실제 검색은 실행 시 `references/search-templates.md`의 iHerb/쿠팡/네이버 쿼리로 Claude가 WebSearch하여 최신 결과로 채웁니다. 쿠팡·네이버 리뷰 인용은 금지이며 **iHerb 리뷰만** 요약 인용합니다.

### 국내 대체 제품 (5개 예시)

| 제품 | 브랜드 | 주요 성분 | 국내가 | 출처 |
|------|--------|-----------|--------|------|
| Milk Thistle 150 | (가상 A) | 실리마린 130 mg | ₩18,000 / 30정 | [쿠팡](placeholder) |
| 밀크씨슬 플러스 | (가상 B) | 실리마린 175 mg + 셀레늄 | ₩24,500 / 60정 | [네이버](placeholder) |
| 간 솔루션 | (가상 C) | 실리마린 140 mg + L-시스테인 | ₩27,000 / 60정 | [쿠팡](placeholder) |
| 리버케어 | (가상 D) | 실리마린 150 mg | ₩15,900 / 30정 | [네이버](placeholder) |
| 간 서포트 프리미엄 | (가상 E) | 실리마린 175 mg + 아티초크 | ₩32,000 / 60정 | [쿠팡](placeholder) |

### 해외 대체 제품 (iHerb, 3개 예시)

| 제품 | 브랜드 | 주요 성분 | 해외가 | 리뷰 요약 | 링크 |
|------|--------|-----------|--------|-----------|------|
| Silymarin Milk Thistle | (가상 X) | 실리마린 150 mg | $11.99 / 100정 (약 ₩16,200) | ⭐4.6 (15,000+): "가성비·흡수력 만족" 취지의 다수 리뷰 요약 | [iHerb](placeholder) |
| Milk Thistle Extract | (가상 Y) | 실리마린 175 mg | $16.49 / 120정 (약 ₩22,200) | ⭐4.7 (9,000+): "간수치 개선 체감" 취지 리뷰 다수 | [iHerb](placeholder) |
| Liver Complex | (가상 Z) | 실리마린 140 mg + 아티초크 + NAC | $19.90 / 90정 (약 ₩26,800) | ⭐4.5 (4,500+): "복합제 편의성" vs "캡슐 큼" 의견 혼재 | [iHerb](placeholder) |

※ 가격은 2026-04-17 조회 기준, 변동 가능. 해외가는 1 USD ≈ 1,350 KRW 예상 환율 기준 근사치.

---

## ⚠️ 면책 조항

[ `references/disclaimer.md` §1 한국어 본문 전체 자동 삽입 ]
````

---

## 5. 검증 체크리스트 (이 예시로 확인할 항목)

### 5-1. 정규화 정확성
- [ ] `실리마린(밀크씨슬 추출물)` → `silymarin` 매칭 (한글 + 영문 동의어 우선순위)
- [ ] `L-테아닌` → `l_theanine`
- [ ] `비타민 B1`~`B6` → `thiamine` / `riboflavin` / `pyridoxine`
- [ ] `나이아신(니코틴아미드)` → `niacin` + `form_sub: ["nicotinamide"]`

### 5-2. RDA/UL 계산
- [ ] B1 RDA 1.1 (여성) 기준 15/1.1 = **1363.6% → 1364% 표기**
- [ ] B2 RDA 1.2 (여성 KDRI) 기준 15/1.2 = **1250%**
- [ ] B6 RDA 1.4 (여성 KDRI) 기준 15/1.4 = **1071%**, UL 100의 **15%** ✅
- [ ] 나이아신 15 mg NE / RDA 14 = **107%** ✅, form_sub nicotinamide이므로 UL 1000 기준 (니코틴산 35 아님)
- [ ] B6가 UL 근접(70%)에 도달하지 않아 🟠 플래그 **발생 안 함** (단일 제품 기준)

### 5-3. 상호작용 규칙
- [ ] 프로필에 처방약 없으므로 실리마린↔CYP 경고는 🔴가 아니라 ℹ️ **정보성**으로 강등되어야 함
- [ ] 상호작용 코멘트에 "의사·약사 상담 권고" 문구 존재
- [ ] 처방약 복용량 조정 권고 **금지** 규칙 위반 없음 (코멘트에 중단·감량 등 표현 없음)

### 5-4. 대체 제품 표
- [ ] 국내 5 + 해외 3 **총 8개** 필수
- [ ] 해외 리뷰는 iHerb만, 요약(별점+개수+2-3줄) — 쿠팡/네이버 리뷰 인용 **0건**
- [ ] 해외가 원화 환산 병기
- [ ] 조회 일자 `2026-04-17` 명시

### 5-5. 면책/배너
- [ ] 임신·처방약 없어 `{{WARNING_BANNERS}}` 치환 **빈 문자열**
- [ ] 말미에 `{{DISCLAIMER}}` 자동 삽입 (`references/disclaimer.md` §1 전체)

### 5-6. 개인정보 & 범용성
- [ ] 본 문서 내 특정 사용자명·이메일 등 개인정보 키워드 **0건**
- [ ] 제품명·브랜드는 모두 가상(NutriCalm Labs / LiverRelax Complex / 가상 A~E / 가상 X~Z)
- [ ] 32세 여성은 가상 페르소나 — 실제 개인 병력·이름 없음

---

## 6. 다른 프로필 변주 (회귀 테스트용 메모)

같은 제품을 아래 프로필로 다시 돌려 예상 결과 확인:

| 프로필 변주 | 기대 차이 |
|------------|----------|
| 임신 15주 여성 | 🔴 임신 배너 최상단. 비타민 A 없어 기형 경고는 해당 안 되나 "임신 중 보충제 모두 의사 상담" 배너. 실리마린 임신 안전성 근거 부족 명시 |
| 와파린 복용 중 | 🔴 실리마린↔와파린 경고 최상단. INR 모니터링·의사 상담 필수 문구. 스택 최적화 remove에 **실리마린 검토** 추가 |
| 60세 남성 | B1/B2/B6 RDA 변경 (51+ 여성 B6 1.5, 남성 1.7 등). 나머지 거의 동일. 나이대별 수치만 달라지는지 확인 |

---

## 7. 변경 이력

- **v1.0 (2026-04-17)**: 세션 3 착수 — 단일 제품 1차 테스트 케이스 초안.
