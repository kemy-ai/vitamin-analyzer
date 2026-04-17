# Example 3 — 복수 제품 스택 최적화

> **용도**: `vitamin-analyzer` Skill 3차 테스트 케이스. **3개 제품을 동시 복용 중**인 사용자의 스택을 합산·중복·UL 초과 평가.
> **페르소나**: 35세 여성, 가상 인물. 직장·육아 복합 스트레스 호소. 처방약 없음.
> **대상 제품**:
> - 제품 1 — 가상 **DailyCore Nutrition / Women's Complete** (25성분 종합비타민)
> - 제품 2 — 가상 **VitalB Labs / Stress B-Complex** (고용량 B군)
> - 제품 3 — 가상 **SunD3 / High Potency D3 5000** (비타민 D 단일)
> **테스트 포커스**: (a) 중복 성분 검출 (b) Vit D UL 1.5배 초과 🔴 + B6 UL 근접 🟠 + Mg 부족 🟡 혼재 (c) stack_optimization 3그룹(remove/reduce/add) 전부 활성화 (d) 기여도 표(`CONTRIBUTION_TABLE`) 렌더.

---

## 1. 테스트 입력

### 1-1. 사용자 프로필

```yaml
gender: female
age: 35
pregnancy: none
breastfeeding: no
prescriptions: none
conditions: none
symptoms:
  - 직장·육아 복합 스트레스
  - 수면 질 저하
  - 근육 뭉침·종아리 경련 간헐적
goals:
  - 종합 영양 + 스트레스 완화
  - 피로·에너지 개선
country: KR
language: ko
```

### 1-2. 제품 입력

사용자 입력 (텍스트 경로):
```
현재 3가지를 같이 먹고 있어. 분석 부탁해.
1) DailyCore Women's Complete — 하루 1정 (종합비타민)
2) VitalB Labs Stress B-Complex — 하루 1캡슐
3) SunD3 High Potency D3 5000 — 하루 1캡슐
```

각 제품 라벨 요약 (분석 대상 성분만):

| 제품 | 성분 | 용량 (라벨) | 표준화 |
|------|------|------------|--------|
| 1 DailyCore | Vitamin D3 | 1000 IU | 25 mcg |
| 1 DailyCore | Vitamin B6 (Pyridoxine HCl) | 3 mg | 3 mg |
| 1 DailyCore | Magnesium (Oxide) | 100 mg | 100 mg |
| 1 DailyCore | Calcium | 200 mg | 200 mg |
| 1 DailyCore | Iron | 14 mg | 14 mg |
| 1 DailyCore | (기타 B군·미네랄 다수) | 적정 범위 | — |
| 2 Stress B-Complex | Vitamin B6 (P-5-P) | 75 mg | 75 mg |
| 2 Stress B-Complex | B1 / B2 / B5 / B12 | 50 / 50 / 100 / 500 mcg | 각 기재 |
| 2 Stress B-Complex | Folate (5-MTHF) | 400 mcg DFE | 400 mcg DFE |
| 3 SunD3 5000 | Vitamin D3 | 5000 IU | **125 mcg** |
| 3 SunD3 5000 | (오일 base만) | — | — |

> **본 예시에서 분석 초점**: Vit D, B6, Mg, Ca, Fe. 2번 제품의 B1/B2/B5/B12는 RDA 고배수이지만 UL 미설정 수용성이라 경고 대상 아님(🟡).

---

## 2. 예상 Phase 3 stack_matrix (요약)

```json
{
  "vitamin_d":  {"total_amount": 150, "unit": "mcg",
                 "sources": [{"product": "Women's Complete", "amount": 25}, {"product": "SunD3 5000", "amount": 125}]},
  "pyridoxine": {"total_amount": 78,  "unit": "mg",
                 "sources": [{"product": "Women's Complete", "amount": 3},  {"product": "Stress B-Complex", "amount": 75}]},
  "magnesium":  {"total_amount": 100, "unit": "mg",
                 "sources": [{"product": "Women's Complete", "amount": 100}]},
  "calcium":    {"total_amount": 200, "unit": "mg",
                 "sources": [{"product": "Women's Complete", "amount": 200}]},
  "iron":       {"total_amount": 14,  "unit": "mg",
                 "sources": [{"product": "Women's Complete", "amount": 14}]},
  "folate":     {"total_amount": 400, "unit": "mcg DFE",
                 "sources": [{"product": "Stress B-Complex", "amount": 400}]}
}
```

---

## 3. per_ingredient_eval (핵심 5개만)

35세 여성, KDRI 30-49 구간 우선 + NIH 31-50 병기.

| 성분 | 합산 | RDA (KDRI/NIH) | RDA% | UL | UL% | 플래그 | 코멘트 |
|------|------|----------------|------|------|------|--------|-------|
| 비타민 D | 150 mcg | 10 / 15 | 1000% / 1000% | 100 | **150%** | 🔴 | **UL 1.5배 초과**. 장기 복용 시 고칼슘혈증·신결석 위험 |
| 비타민 B6 | 78 mg | 1.4 / 1.3 | 5571% / 6000% | 100 | **78%** | 🟠 | **UL 근접**. 장기 고용량 시 말초신경병증 보고 |
| 마그네슘 | 100 mg | 280 / 320 | 36% / 31% | 350 | 29% | 🟡 | RDA 50% 미만, 부족 가능성 |
| 칼슘 | 200 mg | 700 / 1000 | 29% / 20% | 2500 | 8% | 🟡 | RDA 50% 미만 |
| 철 | 14 mg | 14 / 18 | 100% / 78% | 45 | 31% | ✅ | 월경 지속 시 적정 |

---

## 4. 중복 검출 (duplicates)

```json
{
  "duplicates": [
    {
      "canonical": "vitamin_d",
      "severity": "🔴",
      "total": 150,
      "ul": 100,
      "ul_ratio": 1.5,
      "products": ["Women's Complete", "SunD3 5000"],
      "recommendation": "SunD3 5000 중단 또는 격일 복용 전환. 종합비타민의 25 mcg만 유지 시 UL 안전 범위(25% UL)"
    },
    {
      "canonical": "pyridoxine",
      "severity": "🟠",
      "total": 78,
      "ul": 100,
      "ul_ratio": 0.78,
      "products": ["Women's Complete", "Stress B-Complex"],
      "recommendation": "Stress B-Complex의 B6를 저용량(10-25 mg, P-5-P) 제품으로 교체"
    }
  ]
}
```

---

## 5. stack_optimization

```json
{
  "remove": [],
  "reduce": [
    {"canonical": "vitamin_d",  "reason": "UL 1.5배 초과 (150 mcg > UL 100 mcg), 2개 제품 중복",
     "action": "SunD3 5000 중단 또는 격일 복용. 종합비타민의 25 mcg(1000 IU) 단독 유지 시 UL 25%로 안전",
     "evidence": [{"tier": 1, "url": "https://ods.od.nih.gov/factsheets/VitaminD-HealthProfessional/"}]},
    {"canonical": "pyridoxine", "reason": "UL 78% — 장기 복용 시 말초신경병증 위험. 2개 제품 중복",
     "action": "Stress B-Complex의 B6 75 mg을 10-25 mg (P-5-P 유지) 제품으로 교체",
     "evidence": [{"tier": 1, "url": "https://ods.od.nih.gov/factsheets/VitaminB6-HealthProfessional/"}]}
  ],
  "add": [
    {"canonical": "magnesium", "reason": "RDA 31-36% — 부족 가능성. 근육 뭉침·수면 저하 호소 프로필에 부합",
     "action": "200-250 mg 추가 (Glycinate 또는 Citrate). 취침 전 복용 권장",
     "evidence": [{"tier": 1, "url": "https://ods.od.nih.gov/factsheets/Magnesium-HealthProfessional/"}]},
    {"canonical": "calcium", "reason": "RDA 20-29% — 식이 미흡 시 부족 가능. 35세 여성 골량 유지 중요",
     "action": "식이(유제품·뼈째 먹는 생선) 섭취 확인 후 부족 시 200-500 mg (Citrate) 추가. 1회 500 mg 이하 분할",
     "evidence": [{"tier": 1, "url": "https://ods.od.nih.gov/factsheets/Calcium-HealthProfessional/"}]}
  ]
}
```

---

## 6. 예상 최종 리포트 (축약)

````markdown
# 보충제 분석 리포트

**분석 대상**: DailyCore Women's Complete, VitalB Labs Stress B-Complex, SunD3 High Potency D3 5000
**프로필**: 35세 여성 / 임신·수유 없음 / 처방약 없음 / 한국
**분석 일자**: 2026-04-17

> 🔴 **상한섭취량(UL) 초과 경고**
> 분석 결과 **비타민 D**가 상한섭취량을 초과했습니다(150 mcg > UL 100 mcg, 1.5배). 장기 복용 시 고칼슘혈증·신결석 위험이 있습니다. **제품 감량·교체·중단을 검토**하고 담당 의료진과 상담하세요.

## TL;DR
- UL 초과 1건 (비타민 D 150%), UL 근접 1건 (비타민 B6 78%), 부족 가능성 2건 (마그네슘·칼슘)
- 🔴 **비타민 D가 2개 제품(Women's Complete + SunD3)에 중복 함유, UL 1.5배 초과** — 즉시 조정 권고
- 우선 조정: **SunD3 5000 중단 또는 격일 복용 전환**. 종합비타민의 25 mcg만으로 UL 25% 안전 범위

## 입력 제품 요약
| # | 브랜드 | 제품명 | 분석 성분 수 |
|---|--------|--------|-------------|
| 1 | DailyCore Nutrition | Women's Complete | 25 |
| 2 | VitalB Labs | Stress B-Complex | 6 |
| 3 | SunD3 | High Potency D3 5000 | 1 |

## 성분별 분석

| 성분 | 합산량 | RDA% | UL% | 플래그 | 코멘트 [출처] |
|------|--------|------|------|--------|-----------|
| 비타민 D | 150 mcg | 1000% | 150% | 🔴 | 2개 제품 중복 + UL 초과, 장기 복용 시 고칼슘혈증·신결석 위험 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/VitaminD-HealthProfessional/] |
| 비타민 B6 | 78 mg | 5571% | 78% | 🟠 | 2개 제품 중복 + UL 78%, 장기 고용량 시 말초신경병증 보고 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/VitaminB6-HealthProfessional/] |
| 마그네슘 | 100 mg | 36% | 29% | 🟡 | RDA 50% 미만 — 근육 뭉침·수면 호소 프로필에 증량 권장 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Magnesium-HealthProfessional/] |
| 칼슘 | 200 mg | 29% | 8% | 🟡 | RDA 50% 미만 — 식이 확인 후 보충 여부 판단 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Calcium-HealthProfessional/] |
| 철 | 14 mg | 100% | 31% | ✅ | 월경 지속 시 적정 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Iron-HealthProfessional/] |
| 엽산 | 400 mcg DFE | 100% | 40% | ✅ | 가임기 적정 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Folate-HealthProfessional/] |
| B1/B2/B5/B12 | (고용량) | 100-500배 | — | 🟡 | UL 미설정 수용성. 실질 이득 근거 부족 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/ (각 B군 factsheet)] |

**플래그 범례**: 🔴 UL 초과/고위험 · 🟠 UL 근접/주의 · 🟡 부족 또는 과잉 가능성 · ✅ 적정 · ℹ️ RDA 미설정

## 제품별 성분 기여도

| 성분 | 합산 | 제품 1 (DailyCore) | 제품 2 (Stress B) | 제품 3 (SunD3) |
|------|------|--------------------|-------------------|----------------|
| 비타민 D | 150 mcg | 25 mcg | — | **125 mcg** |
| 비타민 B6 | 78 mg | 3 mg | **75 mg** | — |
| 엽산 | 400 mcg DFE | — | 400 mcg DFE | — |

### 🔴 중복 섭취 경고

- **비타민 D**: 2개 제품(Women's Complete + SunD3 5000)에 중복 함유, 합산 150 mcg (UL 100 mcg의 **1.5배 초과**)
  - 권고: **SunD3 5000 중단 또는 격일 복용 전환**. 종합비타민 25 mcg 단독 유지 시 UL 25%
  - [출처: Tier 1 / https://ods.od.nih.gov/factsheets/VitaminD-HealthProfessional/]
- **비타민 B6**: 2개 제품(Women's Complete + Stress B-Complex)에 중복 함유, 합산 78 mg (UL 100 mg의 **78%**)
  - 권고: Stress B-Complex의 B6 75 mg을 저용량(10-25 mg, P-5-P) 제품으로 교체
  - [출처: Tier 1 / https://ods.od.nih.gov/factsheets/VitaminB6-HealthProfessional/]

## 스택 최적화 추천

### 📉 감량/통합 권장
- **비타민 D** — UL 1.5배 초과. **SunD3 5000 중단 또는 격일 복용 전환**. 종합비타민의 25 mcg(1000 IU) 단독 유지 시 UL 25%로 안전 범위.
  - [출처: Tier 1 / https://ods.od.nih.gov/factsheets/VitaminD-HealthProfessional/]
- **비타민 B6** — UL 78% — 장기 복용 시 말초신경병증 위험. **Stress B-Complex의 B6 75 mg을 10-25 mg (P-5-P 형태 유지) 제품으로 교체**.
  - [출처: Tier 1 / https://ods.od.nih.gov/factsheets/VitaminB6-HealthProfessional/]

### ➕ 추가 고려
- **마그네슘** — 35세 여성 KDRI 280 / NIH 320 mg 기준 현재 100 mg(31-36%)로 부족. 근육 뭉침·수면 저하 호소 프로필에 부합. **200-250 mg (Glycinate 또는 Citrate) 추가, 취침 전 복용** 권장.
  - [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Magnesium-HealthProfessional/]
- **칼슘** — KDRI 700 mg 기준 200 mg(29%). 식이 섭취(유제품·뼈째 먹는 생선) 확인 후 부족 시 200-500 mg (Citrate) 추가, **1회 500 mg 이하 분할** 원칙.
  - [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Calcium-HealthProfessional/]

## 대체 제품 비교

> ※ 아래 표 제품명·브랜드·가격은 **예시**이며, 실제 검색은 실행 시 `references/search-templates.md`의 쿼리 템플릿으로 Claude가 WebSearch 결과를 채웁니다. 쿠팡·네이버 리뷰 인용 금지, **iHerb 리뷰만** 요약 인용합니다.

### 스택 조정 후 대체 구성 (예시)

**조정 시나리오**: SunD3 중단 + B-Complex 저용량 교체 + Mg 추가 ⇒ 총 3제품 유지.

#### 국내 대체 제품 (5개 예시)

| 제품 | 브랜드 | 주요 성분 | 국내가 | 출처 |
|------|--------|-----------|--------|------|
| 마그네슘 글리시네이트 200 | (가상 A) | Mg Glycinate 200 mg | ₩19,800 / 60캡슐 | [쿠팡](placeholder) |
| 저용량 B컴플렉스 | (가상 B) | B6 10 mg (P-5-P) + B군 적정 | ₩22,500 / 60캡슐 | [네이버](placeholder) |
| 시트레이트 마그네슘 250 | (가상 C) | Mg Citrate 250 mg | ₩16,900 / 90정 | [쿠팡](placeholder) |
| 액티브 B-Complex | (가상 D) | B6 15 mg + 엽산 + B12 | ₩26,000 / 60캡슐 | [네이버](placeholder) |
| 칼슘 시트레이트 400 | (가상 E) | Ca Citrate 400 mg + Vit D 10 mcg | ₩18,500 / 120정 | [쿠팡](placeholder) |

#### 해외 대체 제품 (iHerb, 3개 예시)

| 제품 | 브랜드 | 주요 성분 | 해외가 | 리뷰 요약 | 링크 |
|------|--------|-----------|--------|-----------|------|
| Magnesium Glycinate 200 | (가상 X) | Mg Glycinate 200 mg | $13.99 / 100캡슐 (약 ₩18,900) | ⭐4.7 (25,000+): "수면·근육 체감" 취지 리뷰 다수 | [iHerb](placeholder) |
| Active B-Complex | (가상 Y) | B6 10 mg (P-5-P) + 활성형 B군 | $15.49 / 60캡슐 (약 ₩20,900) | ⭐4.6 (12,000+): "소변색 변화" "피로 개선" 혼재 | [iHerb](placeholder) |
| Calcium Citrate + D3 | (가상 Z) | Ca Citrate 500 mg + Vit D3 10 mcg | $14.99 / 120정 (약 ₩20,200) | ⭐4.5 (8,500+): "흡수 부담 적음" 요약 | [iHerb](placeholder) |

※ 가격은 2026-04-17 조회 기준, 변동 가능. 해외가는 1 USD ≈ 1,350 KRW 환율 기준 근사치.

> **주의**: 새 Vit D 제품을 추가하지 마세요. 현재 Women's Complete의 25 mcg로 충분합니다. 위 대체 제품 표에서 **Ca Citrate + D3**를 선택할 경우 Women's Complete와 합산 시 Vit D가 35 mcg이 되어 UL의 35%로 여전히 안전하나, 기존 SunD3 중단이 선행되어야 합니다.

---

## ⚠️ 면책 조항

[ `references/disclaimer.md` §1 한국어 본문 전체 자동 삽입 ]
````

---

## 7. 검증 체크리스트

### 7-1. stack_matrix 합산
- [ ] Vit D: 25 + 125 = **150 mcg** (2개 제품 합산 정확)
- [ ] B6: 3 + 75 = **78 mg**
- [ ] Mg: 100 mg (1개 제품만)
- [ ] Ca: 200 mg (1개 제품만)

### 7-2. 플래그 규칙
- [ ] Vit D UL 150% → **🔴 리포트 최상단 배너** + `{{WARNING_BANNERS}}` 의 UL 초과 섹션 치환
- [ ] B6 UL 78% (70-99%) → **🟠 UL 근접**
- [ ] Mg RDA 36% (<50%) → **🟡 부족 가능성**
- [ ] Ca RDA 29% (<50%) → **🟡 부족 가능성**
- [ ] Fe RDA 78-100% → **✅ 적정**

### 7-3. 중복 검출
- [ ] Vit D 중복 2제품 검출, severity 🔴
- [ ] B6 중복 2제품 검출, severity 🟠
- [ ] 오탐 **0건**: 제품 1에만 있는 Mg/Ca/Fe는 중복으로 분류 **안 됨**

### 7-4. 기여도 표 (`CONTRIBUTION_TABLE`)
- [ ] **2개 이상 제품에 등장하는 성분만** 표시 (Vit D, B6, 엽산 3행)
- [ ] 단일 제품 성분(Mg, Ca, Fe)은 기여도 표에 **미포함**
- [ ] `—` 표기가 "해당 제품에 미포함"을 명확히 전달

### 7-5. stack_optimization 3그룹
- [ ] remove 그룹은 현 시나리오에 해당 없음 → **서브헤더 생략** (빈 그룹 "해당 없음" 금지)
- [ ] reduce 그룹에 Vit D + B6 (2건)
- [ ] add 그룹에 Mg + Ca (2건)
- [ ] 순서 고정: reduce → add (remove가 빠졌어도 남은 그룹 순서 유지)

### 7-6. 경고 배너 / 면책
- [ ] UL 초과 있어 `{{WARNING_BANNERS}}` = **Vit D UL 초과 배너** (최상단 빨강)
- [ ] 임신·처방약 배너는 해당 없음
- [ ] 말미에 `{{DISCLAIMER}}` 자동 삽입

### 7-7. 대체 제품
- [ ] 국내 5 + 해외 3 = **8개**
- [ ] **Vit D 추가 제품을 새로 추천하지 않음** — 주의 박스에 명시
- [ ] iHerb 리뷰는 별점+개수+요약만, 쿠팡·네이버 리뷰 인용 **0건**

### 7-8. 개인정보 & 상표권
- [ ] 특정 사용자명·이메일 등 개인정보 키워드 **0건**
- [ ] 3개 브랜드 모두 가상 (DailyCore Nutrition / VitalB Labs / SunD3)

---

## 8. 회귀 변주

| 변주 | 기대 차이 |
|------|----------|
| 같은 스택 + 와파린 복용 | 🔴 최상단에 처방약 배너 추가. Vit D ↔ 와파린 직접 상호작용은 약하나 **전반 보충제 추가·변경 시 의사 상담** 강조. B6·Mg도 "변동 시 INR 모니터링" 권고 |
| 같은 스택 + 임신 8주 | 🔴 최상단 임신 배너. Vit D UL 초과가 임신 중 더 민감 → SunD3 즉시 중단 권고 격상. 엽산 400 DFE는 임신 RDA 600 미달 → add에 엽산 200 mcg 권고 |
| SunD3를 2000 IU(50 mcg) 제품으로 교체 | Vit D 합산 75 mcg = UL 75% → **🟠 UL 근접**으로 강등. remove/reduce의 Vit D 추천이 "격일 복용" 수준으로 약화 |

---

## 9. 변경 이력

- **v1.0 (2026-04-17)**: 세션 3 — 3제품 스택 최적화 테스트 케이스 초안. Vit D UL 1.5배 + B6 UL 근접 + Mg 부족 혼재 시나리오.
