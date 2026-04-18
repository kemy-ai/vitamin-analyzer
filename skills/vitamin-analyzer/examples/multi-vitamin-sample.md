# Example 2 — 단일 제품 (25성분 종합비타민)

> **용도**: `vitamin-analyzer` Skill 2차 테스트 케이스. 25성분을 한 제품에 담은 **광범위 종합비타민** 분석.
> **페르소나**: 45세 여성, 가상 인물. 갱년기 이행기(perimenopause) 증상 호소, 처방약 없음.
> **대상 제품**: 가상 브랜드 **DailyCore Nutrition**의 가상 제품 **Women's Complete** (라벨 이미지 입력 가정).
> **테스트 포커스**: (a) 대량 성분 정규화 (b) IU↔mcg·mg NE·mcg DFE·mcg RAE 단위 변환 (c) 폐경 이행기 RDA 해석 (d) 적정/고용량/부족 혼재 시 TL;DR 우선순위.

---

## 1. 테스트 입력

### 1-1. 사용자 프로필

```yaml
gender: female
age: 45
pregnancy: none
breastfeeding: no
prescriptions: none
conditions: none
symptoms:
 - 월경 주기 불규칙 (3-6개월 변동)
 - 수면 저하·야간 열감
 - 체력 저하
goals:
 - 갱년기 이행기 전반 건강 유지
 - 뼈·관절 건강
country: KR
language: ko
```

### 1-2. 제품 입력 (Phase 2 이미지 경로)

사용자가 성분표 사진을 업로드. `parse-image.md` 프롬프트가 수행된다고 가정. 라벨 원문은 영문 + 일부 한글 혼재, 단위는 `IU`, `mg`, `mcg`, `mcg DFE`, `mg NE`, `mcg RAE` 혼용.

라벨 상 1일 섭취량(Serving Size): 1 tablet, 1 serving per day.

| # | 라벨 표기 (raw_name) | raw amount | unit (라벨) |
|---|---------------------|------------|-------------|
| 1 | Vitamin A (as Retinyl Palmitate) | 2666 | IU |
| 2 | Vitamin D3 (Cholecalciferol) | 1000 | IU |
| 3 | Vitamin E (as d-alpha Tocopherol) | 22 | IU |
| 4 | Vitamin K1 | 80 | mcg |
| 5 | Vitamin C (as Ascorbic Acid) | 100 | mg |
| 6 | Thiamin (B1) | 3 | mg |
| 7 | Riboflavin (B2) | 3 | mg |
| 8 | Niacin (as Nicotinamide) | 20 | mg NE |
| 9 | Pantothenic Acid (B5) | 10 | mg |
| 10 | Vitamin B6 (as Pyridoxine HCl) | 3 | mg |
| 11 | Biotin (B7) | 50 | mcg |
| 12 | Folate (as Folic Acid) | 400 | mcg DFE |
| 13 | Vitamin B12 (as Cyanocobalamin) | 10 | mcg |
| 14 | Calcium (as Calcium Carbonate) | 200 | mg |
| 15 | Magnesium (as Magnesium Oxide) | 100 | mg |
| 16 | Iron (as Ferrous Fumarate) | 14 | mg |
| 17 | Zinc (as Zinc Citrate) | 10 | mg |
| 18 | Selenium (as Selenomethionine) | 55 | mcg |
| 19 | Copper (as Copper Gluconate) | 900 | mcg |
| 20 | Manganese (as Manganese Sulfate) | 2 | mg |
| 21 | Iodine (as Potassium Iodide) | 150 | mcg |
| 22 | Chromium (as Chromium Picolinate) | 30 | mcg |
| 23 | Molybdenum (as Sodium Molybdate) | 45 | mcg |
| 24 | Lutein (from Marigold Extract) | 6 | mg |
| 25 | Coenzyme Q10 (Ubiquinone) | 30 | mg |

---

## 2. 예상 단위 변환 결과 (Phase 2 → Phase 3)

`analyze-ingredient.md` §2 단위 표준화 단계에서 변환되어야 할 행:

| 성분 | 라벨 원값 | 표준화 값 | 변환식 출처 |
|------|----------|-----------|------------|
| Vitamin A | 2666 IU (Retinyl Palmitate) | **800 mcg RAE** | 1 IU (preformed retinol) = 0.3 mcg RAE → 2666 × 0.3 = 800 |
| Vitamin D3 | 1000 IU | **25 mcg** | 1 mcg = 40 IU → 1000 / 40 = 25 |
| Vitamin E | 22 IU (d-alpha) | **약 15 mg alpha-TE** | 1 IU (d-alpha natural) = 0.67 mg → 22 × 0.67 ≈ 14.74 |
| Niacin | 20 mg NE (nicotinamide) | **20 mg NE, form_sub=nicotinamide** | NE는 이미 표준. UL 기준 선택은 form_sub에 따라 nicotinamide→1000 mg |
| Folate | 400 mcg DFE (folic acid) | **400 mcg DFE** | folic acid가 보충제 형태로 라벨화되면 DFE 계산식: `mcg DFE = mcg folic acid × 1.7`이지만 라벨이 이미 DFE로 표기 → 원값 유지 |

---

## 3. 예상 Phase 3 per_ingredient_eval (요약표)

45세 여성, 한국(KDRI 우선 + NIH 병기). KDRI 구간 30-49, NIH 구간 31-50.

| # | canonical | 합산 | RDA (KDRI/NIH) | RDA% | UL | UL% | 플래그 | 핵심 코멘트 |
|---|-----------|------|----------------|------|------|------|--------|------------|
| 1 | vitamin_a | 800 mcg RAE | 650 / 700 | 123% / 114% | 3000 | 27% | ✅ | retinyl palmitate 형태. 임신 시 기형 경고 대상이나 본 프로필은 임신 없음 |
| 2 | vitamin_d | 25 mcg | 10 / 15 | 250% / 167% | 100 | 25% | ✅ | 25 mcg = 1000 IU. 한국 여성 평균 혈중 농도 낮아 고용량 권장 연구 다수 |
| 3 | vitamin_e | 15 mg | 12 / 15 | 125% / 100% | 540 / 1000 | 3% / 1.5% | ✅ | - |
| 4 | vitamin_k | 80 mcg | 65 / 90 | 123% / 89% | ND | — | ✅ | AI 기준. 와파린 병용 시 경고(본 프로필 해당 없음) |
| 5 | vitamin_c | 100 mg | 100 / 75 | 100% / 133% | 2000 | 5% | ✅ | - |
| 6 | thiamine | 3 mg | 1.1 / 1.1 | 273% | ND | — | 🟡 | UL 미설정 수용성. RDA 2.7배 — 실질 이득 근거 부족 |
| 7 | riboflavin | 3 mg | 1.2 / 1.1 | 250% | ND | — | 🟡 | 동일 사유 |
| 8 | niacin | 20 mg NE | 14 / 14 | 143% | 1000 (nicotinamide) | 2% | ✅ | 니코틴아미드 형태라 홍조 위험 낮음 |
| 9 | pantothenic_acid | 10 mg | 5 (AI) | 200% | ND | — | ✅ | AI 초과, 위험 낮음 |
| 10 | pyridoxine | 3 mg | 1.4 / 1.3 | 214% | 100 | 3% | ✅ | 단일 제품 기준 안전. 다른 B6 제품 병용 시 합산 모니터링 |
| 11 | biotin | 50 mcg | 30 (AI) | 167% | ND | — | ✅ | 갑상선·심장 검사 4일 전 중단 권고는 ≥5000 mcg에만 해당 |
| 12 | folate | 400 mcg DFE | 400 | 100% | 1000 | 40% | ✅ | B12 마스킹 우려는 고용량(>1000)에만 해당 |
| 13 | cobalamin | 10 mcg | 2.4 | 417% | ND | — | ℹ️ | UL 미설정. 수용성 |
| 14 | calcium | 200 mg | 700 / 1000 | 29% / 20% | 2500 | 8% | 🟡 | **부족 가능성**. 식이+보충제 합산으로 700 mg 이상 목표 |
| 15 | magnesium | 100 mg | 280 / 320 | 36% / 31% | 350 | 29% | 🟡 | **부족 가능성**. 수면·근육 호소 프로필에 증량 고려 |
| 16 | iron | 14 mg | 14 / 18 | 100% / 78% | 45 | 31% | ✅ | 월경 지속 중이라 적정. 폐경 이후에는 과잉 가능 |
| 17 | zinc | 10 mg | 8 / 8 | 125% | 35 / 40 | 29% / 25% | ✅ | 장기 >50 mg 시 구리 결핍 위험. 본 용량은 안전 |
| 18 | selenium | 55 mcg | 60 / 55 | 92% / 100% | 400 | 14% | ✅ | - |
| 19 | copper | 900 mcg | 650 / 900 | 138% / 100% | 10000 | 9% | ✅ | - |
| 20 | manganese | 2 mg | 3.5 (AI, KDRI) / 1.8 (NIH) | 57% / 111% | 11 | 18% | ✅ | 기관별 AI 차이 큼. NIH 기준 충분 |
| 21 | iodine | 150 mcg | 150 / 150 | 100% | 2400 / 1100 | 6% / 14% | ✅ | 한국 해조류 섭취 많음 — 식이 합산 시 UL 근접 주의 |
| 22 | chromium | 30 mcg | 20 (AI) / 25 (AI) | 150% / 120% | ND | — | ✅ | AI 초과 이슈 없음 |
| 23 | molybdenum | 45 mcg | 25 (KDRI) / 45 (NIH) | 180% / 100% | 500 / 2000 | 9% / 2% | ✅ | KDRI UL 낮음 |
| 24 | lutein | 6 mg | — | — | — | — | ℹ️ | AREDS2 권장 10 mg. 6 mg은 일반 섭취 연구 범위 |
| 25 | coenzyme_q10 | 30 mg | — | — | — | — | ℹ️ | 연구 용량 100-200 mg/day. 30 mg은 유지 목적 하단 |

**플래그 집계**: 🔴 0 · 🟠 0 · 🟡 4 (B1·B2·칼슘·마그네슘) · ✅ 17 · ℹ️ 4

---

## 4. 예상 Phase 3 JSON (요약)

```json
{
 "analysis_status": "ok",
 "profile_summary": "45세 여성 / 갱년기 이행기 / 임신·수유 없음 / 처방약 없음 / 한국",
 "warnings_triggered": {
 "infant_or_pet": false,
 "pregnancy": false,
 "breastfeeding": false,
 "prescription_drug_present": false,
 "ul_exceeded": []
 },
 "duplicates": [],
 "interactions": [
 {
 "severity": "🟡",
 "type": "food_or_timing",
 "pair": "철 ↔ 칼슘·차·커피 동반 흡수 저하",
 "summary": "철 14 mg + 칼슘 200 mg 동일 제품 — 흡수 간섭 가능. 제품이 분할 복용이 아닌 1일 1정이라 실질 영향은 제한적. 식이 차/커피는 제품 복용 전후 2시간 피하기 권장",
 "evidence": [{"tier": 1, "url": "https://ods.od.nih.gov/factsheets/Iron-HealthProfessional/"}]
 }
 ],
 "stack_optimization": {
 "remove": [],
 "reduce": [
 {"canonical": "thiamine", "reason": "RDA 273%, UL 미설정 수용성이나 실질 이득 근거 부족", "target": "1-2 mg 수준의 저용량 종합비타민 선택지 고려"},
 {"canonical": "riboflavin", "reason": "RDA 250%, 동일 사유", "target": "동일"}
 ],
 "add": [
 {"canonical": "calcium", "reason": "45세 여성 RDA 700 mg (KDRI). 본 제품 200 mg만으로는 부족. 식이 섭취 확인 후 보충제 추가 여부 판단",
 "target": "식이 포함 합산 700 mg 목표. 부족 시 200-500 mg (Citrate) 추가, 1회 500 mg 이하 분할",
 "evidence": [{"tier": 1, "url": "https://ods.od.nih.gov/factsheets/Calcium-HealthProfessional/"}]},
 {"canonical": "magnesium", "reason": "45세 여성 KDRI 280 mg / NIH 320 mg. 본 제품 100 mg으로 부족. 수면·근육 호소 프로필에 부합",
 "target": "200-250 mg 추가 (Glycinate/Citrate)",
 "evidence": [{"tier": 1, "url": "https://ods.od.nih.gov/factsheets/Magnesium-HealthProfessional/"}]}
 ]
 },
 "unrecognized_ingredients": []
}
```

---

## 5. 예상 최종 리포트 (축약)

````markdown
# 보충제 분석 리포트

**분석 대상**: DailyCore Nutrition Women's Complete
**프로필**: 45세 여성 / 갱년기 이행기 / 임신·수유 없음 / 처방약 없음 / 한국
**분석 일자**: 2026-04-17

## TL;DR
- UL 초과 0건 / UL 근접 0건 / 부족 가능성 2건 (칼슘 200 mg, 마그네슘 100 mg) / 수용성 B 고용량 2건 (B1·B2)
- ℹ️ 철 14 mg과 칼슘 200 mg이 같은 정제에 포함 — 흡수 간섭 가능하나 단일 정제라 실질 영향 제한적
- 우선 조정: 마그네슘 200-250 mg 추가 고려 (수면·근육 호소 프로필 + RDA 31-36% 수준)

## 성분별 분석

| 성분 | 합산량 | RDA% | UL% | 플래그 | 코멘트 [출처] |
|------|--------|------|------|--------|-----------|
| 비타민 A | 800 mcg RAE | 123% | 27% | ✅ | retinyl palmitate. 임신 시 기형 경고 대상이나 본 프로필 해당 없음 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/VitaminA-HealthProfessional/] |
| 비타민 D | 25 mcg (1000 IU) | 167-250% | 25% | ✅ | 한국 여성 혈중 농도 낮음, 고용량 권장 연구 다수 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/VitaminD-HealthProfessional/] |
| 비타민 E | 15 mg | 125% | 3% (KDRI 540) | ✅ | [출처: Tier 1 / https://ods.od.nih.gov/factsheets/VitaminE-HealthProfessional/] |
| 비타민 K | 80 mcg | 123% (KDRI) | — | ✅ | AI 기준. 와파린 복용자는 별도 경고 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/VitaminK-HealthProfessional/] |
| 비타민 C | 100 mg | 100% | 5% | ✅ | [출처: Tier 1 / https://ods.od.nih.gov/factsheets/VitaminC-HealthProfessional/] |
| 비타민 B1 | 3 mg | 273% | — | 🟡 | UL 미설정 수용성, 실질 이득 근거 부족 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Thiamin-HealthProfessional/] |
| 비타민 B2 | 3 mg | 250% | — | 🟡 | 동일 사유, 소변 황색 변화 정상 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Riboflavin-HealthProfessional/] |
| 나이아신 (니코틴아미드) | 20 mg NE | 143% | 2% | ✅ | 니코틴아미드라 홍조형 35 mg UL 대상 아님 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Niacin-HealthProfessional/] |
| 판토텐산 | 10 mg | 200% (AI) | — | ✅ | [출처: Tier 1 / https://ods.od.nih.gov/factsheets/PantothenicAcid-HealthProfessional/] |
| 비타민 B6 | 3 mg | 214% | 3% | ✅ | 장기 50 mg+ 복용 회피 — 본 용량 안전 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/VitaminB6-HealthProfessional/] |
| 비오틴 | 50 mcg | 167% (AI) | — | ✅ | 갑상선·심장 검사 간섭은 ≥5000 mcg에만 해당 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Biotin-HealthProfessional/] |
| 엽산 | 400 mcg DFE | 100% | 40% | ✅ | 가임 가능성 있는 연령 — 적정 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Folate-HealthProfessional/] |
| 비타민 B12 | 10 mcg | 417% | — | ℹ️ | UL 미설정 수용성 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/VitaminB12-HealthProfessional/] |
| 칼슘 | 200 mg | 20-29% | 8% | 🟡 | **부족 가능성**. 식이 포함 합산 700-1000 mg 목표 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Calcium-HealthProfessional/] |
| 마그네슘 | 100 mg | 31-36% | 29% | 🟡 | **부족 가능성**, 수면·근육 호소에 부합 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Magnesium-HealthProfessional/] |
| 철 | 14 mg | 78-100% | 31% | ✅ | 월경 지속 중이라 적정. 폐경 이후 14 mg은 과잉 가능 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Iron-HealthProfessional/] |
| 아연 | 10 mg | 125% | 25-29% | ✅ | 장기 >50 mg 시 구리 결핍 위험 — 본 용량 안전 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Zinc-HealthProfessional/] |
| 셀레늄 | 55 mcg | 92-100% | 14% | ✅ | [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Selenium-HealthProfessional/] |
| 구리 | 900 mcg | 100-138% | 9% | ✅ | [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Copper-HealthProfessional/] |
| 망간 | 2 mg | 57-111% | 18% | ✅ | 기관별 AI 차이 큼 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Manganese-HealthProfessional/] |
| 요오드 | 150 mcg | 100% | 6-14% | ✅ | 한국 해조류 식이 많음 — 보충제+식이 합산 주의 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Iodine-HealthProfessional/] |
| 크롬 | 30 mcg | 120-150% (AI) | — | ✅ | [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Chromium-HealthProfessional/] |
| 몰리브덴 | 45 mcg | 100-180% | 2-9% | ✅ | [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Molybdenum-HealthProfessional/] |
| 루테인 | 6 mg | — | — | ℹ️ | AREDS2 권장 10 mg. 6 mg은 일반 섭취 범위 [출처: Tier 2 / https://examine.com/supplements/lutein/] |
| CoQ10 | 30 mg | — | — | ℹ️ | 연구 용량 100-200 mg/day. 30 mg은 유지 목적 하단 [출처: Tier 2 / https://examine.com/supplements/coenzyme-q10/] |

**플래그 범례**: 🔴 UL 초과/고위험 · 🟠 UL 근접/주의 · 🟡 부족 또는 과잉 가능성 · ✅ 적정 · ℹ️ RDA 미설정

## 상호작용 경고

- 🟡 **철 ↔ 칼슘·차·커피 (흡수 간섭)**: 철 14 mg과 칼슘 200 mg이 같은 정제에 포함되어 있어 이론상 흡수 간섭이 가능하나, 단일 정제로 1일 1회 복용이라 실질 영향은 제한적. 식이 차·커피는 제품 복용 전후 2시간 피하기 권장 [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Iron-HealthProfessional/]

## 스택 최적화 추천

### 📉 감량/통합 권장
- **비타민 B1 (3 mg) / 비타민 B2 (3 mg)** — RDA 2.5-2.7배. 수용성이라 급성 위험은 낮지만 실질 이득 근거 부족. 저용량(1-2 mg) 종합비타민 선택지 고려.

### ➕ 추가 고려
- **칼슘** — 45세 여성 KDRI 700 mg 기준 본 제품 200 mg만으로 부족. 식이 섭취(유제품·뼈째 먹는 생선) 점검 후 부족 시 200-500 mg (Citrate) 추가, **1회 500 mg 이하 분할** 원칙.
 - [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Calcium-HealthProfessional/]
- **마그네슘** — KDRI 280 / NIH 320 mg 기준 본 제품 100 mg은 31-36% 수준. 수면 저하·야간 열감 프로필에 부합. 200-250 mg (Glycinate/Citrate) 추가 고려.
 - [출처: Tier 1 / https://ods.od.nih.gov/factsheets/Magnesium-HealthProfessional/]

## 대체 제품 비교

> ※ 아래 표 제품명·브랜드·가격은 **예시**이며, 실제 검색은 실행 시 `references/search-templates.md`의 쿼리 템플릿으로 Claude가 WebSearch 결과를 채웁니다. 쿠팡·네이버 리뷰 인용은 금지이며 **iHerb 리뷰만** 요약 인용합니다.

### 국내 대체 제품 (5개 예시)

| 제품 | 브랜드 | 주요 성분 | 국내가 | 출처 |
|------|--------|-----------|--------|------|
| 여성 종합비타민 40+ | (가상 A) | 25종 + 저용량 B군 | ₩28,000 / 30정 | [쿠팡](placeholder) |
| 멀티비타민 우먼 | (가상 B) | 25종 + 칼슘 400 mg + 마그 150 mg | ₩36,500 / 60정 | [네이버](placeholder) |
| 데일리 멀티 W | (가상 C) | 22종 + Mg 200 mg | ₩32,000 / 60정 | [쿠팡](placeholder) |
| 컴플리트 우먼 | (가상 D) | 25종 저용량 B + Fe 14 mg | ₩27,500 / 30정 | [네이버](placeholder) |
| 갱년기 멀티 | (가상 E) | 23종 + 이소플라본 40 mg | ₩42,000 / 60정 | [쿠팡](placeholder) |

### 해외 대체 제품 (iHerb, 3개 예시)

| 제품 | 브랜드 | 주요 성분 | 해외가 | 리뷰 요약 | 링크 |
|------|--------|-----------|--------|-----------|------|
| Women's Multi 40+ | (가상 X) | 24종 + Ca 400 + Mg 200 | $19.90 / 60정 (약 ₩26,800) | ⭐4.6 (22,000+): "정제 크기 큼" vs "성분 범위 만족" | [iHerb](placeholder) |
| Active Women Multi | (가상 Y) | 25종 + CoQ10 50 + 루테인 10 | $23.49 / 90정 (약 ₩31,700) | ⭐4.5 (8,000+): "에너지 체감" 후기 다수 | [iHerb](placeholder) |
| Daily One Women | (가상 Z) | 26종 1일 1정 | $14.99 / 30정 (약 ₩20,200) | ⭐4.4 (18,000+): "가성비" vs "철 없음" 의견 혼재 | [iHerb](placeholder) |

※ 가격은 2026-04-17 조회 기준, 변동 가능. 해외가는 1 USD ≈ 1,350 KRW 환율 기준 근사치.

---

## ⚠️ 면책 조항

[ `references/disclaimer.md` §1 한국어 본문 전체 자동 삽입 ]
````

---

## 6. 검증 체크리스트

### 6-1. 단위 변환
- [ ] Vit A: 2666 IU × 0.3 = **800 mcg RAE**
- [ ] Vit D3: 1000 IU ÷ 40 = **25 mcg**
- [ ] Vit E: 22 IU (d-alpha) × 0.67 ≈ **14.74 mg alpha-TE** (반올림 15)
- [ ] Niacin: `form_sub=["nicotinamide"]`로 UL 1000 기준 선택 (니코틴산 35 **금지**)
- [ ] Folate: 라벨이 이미 `mcg DFE`이므로 재계산 없이 400

### 6-2. 프로필별 RDA 매칭
- [ ] KDRI 30-49 여성 구간 사용: 칼슘 700 / 마그네슘 280 / 철 14 / 아연 8 등
- [ ] NIH 31-50 여성 구간 병기
- [ ] 연령·성별 구간 잘못 매칭 **0건**

### 6-3. 플래그 정확성
- [ ] 칼슘 29% → 🟡 (RDA 50% 미만)
- [ ] 마그네슘 36% → 🟡 (RDA 50% 미만)
- [ ] B1·B2 250-273% → 🟡 (RDA 대비 고용량 검토, UL 미설정이라 🔴 아님)
- [ ] 나머지 RDA 70-150% → ✅
- [ ] 루테인·CoQ10 → ℹ️

### 6-4. 상호작용
- [ ] 철↔칼슘 동일 정제 간섭 **있음** (🟡 정보성)
- [ ] 처방약 없어 CYP·와파린 관련 경고 **없음**

### 6-5. 리포트 필수 요소
- [ ] 25행의 성분표가 모두 렌더됨 (생략·누락 0건)
- [ ] 경고 배너 **빈 문자열** (UL 초과·임신·처방약·영유아 모두 해당 없음)
- [ ] TL;DR 3줄 고정
- [ ] 대체제품 국내 5 + 해외 3 = **8개**
- [ ] `{{DISCLAIMER}}` 자동 삽입

### 6-6. 개인정보 & 상표권
- [ ] 특정 사용자명·이메일 등 개인정보 키워드 **0건**
- [ ] 제품·브랜드·대체제품 모두 **가상** (DailyCore Nutrition / Women's Complete / 가상 A~E / 가상 X~Z)

---

## 7. 회귀 변주

| 변주 | 기대 차이 |
|------|----------|
| 같은 제품 + 55세 여성 | 철 RDA 7-8 mg으로 낮아짐 → 14 mg이 UL 45의 31%로 동일하나 **과잉 메시지** 추가. 칼슘 RDA 800 mg로 상향 |
| 같은 제품 + 임신 8주 | **🔴 최상단 임신 배너**. Vit A 800 mcg RAE는 임신 UL 3000의 27%로 안전 범위이나 retinyl palmitate 형태라 **"과량·누적 섭취 주의" 메모**. 엽산 400이 임신 RDA 600에 미달 → add에 엽산 200 mcg 권고 |
| 같은 제품 + 와파린 복용 | **🔴 최상단 처방약 배너**. 비타민 K 80 mcg는 소량이지만 와파린 INR 변동 우려 → "섭취량 일정 유지" 경고. 제품 교체·중단 결정은 의사 상담 |

