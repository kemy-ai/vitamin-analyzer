# Example 4 — 보충제 + OTC 종합비타민 혼합 분석

> **용도**: v1.2.0 의약품 API 통합 플로우 검증. 건강기능식품(보충제) + 일반의약품(OTC)이 한 스택에 섞여 있을 때 `SKILL.md §4-1-3` Fallback Chain v3의 Step C → Step C2 자동 전환, `parse-drug-response.md` 병합 파싱, `report-template.md §8-A #11~#13` 금지 규약 준수를 확인.
> **페르소나**: 43세 남성, 가상 인물. 만성 피로 호소로 종합비타민 OTC 구매 후 기존 보충제와 중복 우려.
> **대상 제품**: 가상 보충제 **NutriCore Daily-B** (건기식) + 가상 OTC **Mega-Z Multi-Tab** (일반의약품, 비맥스제트를 모델로 한 가상 종합비타민제).
> **테스트 포커스**: (a) C003 0건 후 Step C2 자동 폴백 (b) 라틴명 `ITEM_INGR_NAME` 정규화 (c) 의약품↔보충제 성분 중복 표 생성 (d) 의약품 복용량 조정 금지 규약 준수 (§8-A #11).

---

## 1. 테스트 입력

### 1-1. 프로필

```yaml
gender: male
age: 43
pregnancy: N/A
breastfeeding: N/A
prescriptions: none
conditions:
 - 만성 피로 호소
symptoms:
 - 업무 과중·수면 부족
 - 어깨 근육통
goals:
 - 체력 회복
 - 피로 완화
country: KR
language: ko
```

### 1-2. 제품 입력 (Phase 2)

사용자 메시지:
```
"NutriCore Daily-B"는 원래 먹던 거야 (1정 중 비타민 B1 5mg, B2 5mg, B6 5mg, 엽산 200µg, B12 5µg, 하루 1정). 여기에 약국에서 "Mega-Z Multi-Tab" 샀는데 종합비타민 OTC래. 성분은 라벨에 적혀 있긴 한데 너무 많아. 같이 먹어도 돼?
```

---

## 2. Phase 2 — 제품 조회 (Fallback Chain v3 로그)

### 2-1. NutriCore Daily-B (건기식 경로)

```
Step B (DB): miss (첫 조회)
Step C (C003 API):
 GET /api/{KEY}/C003/json/1/10/PRDLST_NM=NutriCore Daily-B
 → 가상 응답: 1건 매칭
 → PRDLST_REPORT_NO: "202312345678"
 → STDR_STND: "비타민B1 : 표시량(5mg/1정)의 80~150%, 비타민B2 : 표시량(5mg/1정)의 80~150%..."
 → parse-kfda-response.md 통해 구조화
 → confirmation: "NutriCore건강의 Daily-B 정제, 맞나요?" → 사용자 확정
Step X: user-data/products/nutricore_daily-b.json 저장
```

### 2-2. Mega-Z Multi-Tab (의약품 경로 — v1.2.0 핵심)

```
Step B (DB): miss
Step C (C003 API):
 GET /api/{KEY}/C003/json/1/10/PRDLST_NM=Mega-Z Multi-Tab
 → 가상 응답: 0건
 → ★ Step C2 자동 전환 (v1.2.0 신규)
Step C2 (의약품 허가정보):
 GET /1471000/DrugPrdtPrmsnInfoService07/getDrugPrdtPrmsnInq07?item_name=Mega-Z Multi-Tab
 → 가상 응답: 1건 매칭
 {
 "ITEM_SEQ": "202400777",
 "ITEM_NAME": "메가제트멀티탭정",
 "ITEM_ENG_NAME": "Mega-Z Multi-Tab",
 "ENTP_NAME": "(주)제드파마",
 "SPCLTY_PBLC": "일반의약품",
 "PRDUCT_TYPE": "[03190]기타의 비타민제",
 "ITEM_INGR_NAME": "Ascorbic Acid/Benfotiamine/Riboflavin/Pyridoxine HCl/Cyanocobalamin/Nicotinamide/Calcium Pantothenate/Biotin/Folic Acid/Cholecalciferol/Tocopherol Acetate/Zinc Oxide/Magnesium Oxide/Selenium In Dried Yeast/Taurine/Ursodeoxycholic Acid",
 "ITEM_INGR_CNT": "16",
 "CANCEL_NAME": "정상"
 }
Step C2 후속 (e약은요):
 GET /1471000/DrbEasyDrugInfoService/getDrbEasyDrugList?itemName=Mega-Z Multi-Tab&itemSeq=202400777
 → 가상 응답:
 {
 "efcyQesitm": "이 약은 육체피로, 병중∙병후의 체력저하시 비타민 B군·C·D·E의 보급, 신경통·근육통·관절통의 완화, 구내염·구각염의 완화 및 간 기능 보조에 사용합니다.",
 "useMethodQesitm": "만 19세 이상 및 성인은 1일 1회, 1회 1정 식후 복용합니다.",
 "atpnWarnQesitm": "임부, 수유부, 신장 질환자는 복용 전 의사와 상담하십시오.",
 "intrcQesitm": "레보도파 제제와 병용 시 효과가 감소할 수 있으므로 주의가 필요합니다."
 }
Step X: user-data/drugs/zedpharm_mega-z_202400777.json 저장
```

### 2-3. 분류 판정

```
classification: "일반의약품"
→ §8-A #13 전문의약품 규약 적용 안 함 (배너 없음, 대체 섹션 생략 안 함)
→ §8-A #11·#12 적용 (복용량 조정 금지 / 대체 제안 금지)
```

---

## 3. Phase 3 — parse-drug-response.md 출력 (축약)

```json
{
 "drug": {
 "kfda_item_seq": "202400777",
 "product_name": "메가제트멀티탭정",
 "product_name_en": "Mega-Z Multi-Tab",
 "manufacturer": "(주)제드파마",
 "permit_date": "2024-xx-xx",
 "classification": "일반의약품",
 "efficacy_category": "기타의 비타민제",
 "status": "정상"
 },
 "ingredients": [
 {"name_raw": "Ascorbic Acid", "name_ko": "비타민 C", "alias_id": "vitamin_c", "amount_per_dose": null, "unit": null, "confidence": "medium"},
 {"name_raw": "Benfotiamine", "name_ko": "티아민 (지용성 유도체)", "alias_id": "vitamin_b1", "amount_per_dose": null, "unit": null, "confidence": "medium"},
 {"name_raw": "Riboflavin", "name_ko": "리보플라빈", "alias_id": "vitamin_b2", "amount_per_dose": null, "unit": null, "confidence": "medium"},
 {"name_raw": "Pyridoxine HCl", "name_ko": "피리독신 (B6)", "alias_id": "vitamin_b6", "amount_per_dose": null, "unit": null, "confidence": "medium"},
 {"name_raw": "Cyanocobalamin", "name_ko": "시아노코발라민 (B12)", "alias_id": "vitamin_b12", "amount_per_dose": null, "unit": null, "confidence": "medium"},
 {"name_raw": "Folic Acid", "name_ko": "엽산", "alias_id": "folate", "amount_per_dose": null, "unit": null, "confidence": "medium"},
 {"name_raw": "Zinc Oxide", "name_ko": "아연 (산화형)", "alias_id": "zinc", "amount_per_dose": null, "unit": null, "confidence": "medium"},
 {"name_raw": "Magnesium Oxide", "name_ko": "마그네슘 (산화형)", "alias_id": "magnesium", "amount_per_dose": null, "unit": null, "confidence": "medium"},
 {"name_raw": "Ursodeoxycholic Acid", "name_ko": "우르소데옥시콜산 (UDCA)", "alias_id": "ursodeoxycholic_acid", "amount_per_dose": null, "unit": null, "confidence": "medium"},
 {"name_raw": "Taurine", "name_ko": "타우린", "alias_id": "taurine"}
 /* 나머지 6종 생략 */
 ],
 "intake": {
 "age_group": "만 19세 이상",
 "times_per_day": 1,
 "amount_per_time": 1,
 "unit": "정",
 "per_day_total": 1,
 "with_meal": true
 },
 "cross_check_hints": {
 "overlaps_with_supplements": ["vitamin_b1", "vitamin_b2", "vitamin_b6", "vitamin_b12", "folate"],
 "unique_components": ["ursodeoxycholic_acid", "taurine"],
 "requires_doctor_consult": true
 }
}
```

---

## 4. 예상 리포트 (축약)

````markdown
# 보충제 분석 리포트 — 사용자

**분석 대상**: NutriCore Daily-B, Mega-Z Multi-Tab
**프로필**: 43세 남성 / 임신 없음 / 처방약 없음
**분석 일자**: 2026-XX-XX

(전문의약품 배너 없음 — 일반의약품만 포함)

> ⚠️ 본 분석은 의학적 조언이 아닙니다. (이하 일반 경고 배너)

## 분석 요약 결론

43세 남성이 기존 보충제 **NutriCore Daily-B**(종합 비타민 B군)에 일반의약품 **Mega-Z Multi-Tab**(식약처 공시 효능: 육체피로·비타민 B·C·D·E 보급·간 기능 보조)을 추가로 복용하려는 상황입니다. 두 제품은 비타민 B1·B2·B6·B12·엽산이 **중복 포함**되어 있으며, 정확한 의약품 함량은 식약처 API에서 제공되지 않아 라벨을 직접 확인해야 정량 판정이 가능합니다. Mega-Z에는 보충제에는 없는 **우루소데옥시콜산(간담즙 기능)**·**타우린** 성분이 포함되어 "단순 교체"가 아니라 "기전이 다른 추가"에 가깝습니다. 의약품 복용량 조정은 본 스킬이 판단할 수 없으며, 보충제와 의약품 동시 복용이 필요한지, 아니면 둘 중 하나만으로 충분한지는 **약사 또는 의사와 상담**해 결정하세요.

## 성분별 분석

(보충제 NutriCore Daily-B 수치만 RDA% 계산 가능. 의약품은 함량 없음 — "표기 없음"으로 표시)

| 영양소 | 보충제 합산 (mg/µg) | 의약품 포함 여부 | RDA% | UL% | 플래그 |
|-------|-----|-----|-----|-----|-------|
| 비타민 B1 | 5 mg | O (함량 미상) | 417% | — | ⚠️ 의약품 함량 확인 필요 |
| 비타민 B2 | 5 mg | O (함량 미상) | 385% | — | ⚠️ 의약품 함량 확인 필요 |
| 비타민 B6 | 5 mg | O (함량 미상) | 384% | 7.5% | ⚠️ 의약품 함량에 따라 UL 근접 가능 |
| 엽산 | 200 µg | O (함량 미상) | 50% | 20% | ⚠️ 의약품 함량 확인 필요 |

**플래그 범례**: 🔴 UL 초과/고위험 · 🟠 UL 근접/주의 · ⚠️ 의약품 함량 미상 — 라벨 확인 필요

## 💊 의약품 식별 및 성분 확인

#### 💊 메가제트멀티탭정 (Mega-Z Multi-Tab)
- **제조**: (주)제드파마 / **허가일**: 2024-xx-xx / **품목코드**: 202400777
- **분류**: 일반의약품 · 기타의 비타민제
- **식약처 공식 효능** (e약은요 인용):
 > "이 약은 육체피로, 병중∙병후의 체력저하시 비타민 B군·C·D·E의 보급, 신경통·근육통·관절통의 완화, 구내염·구각염의 완화 및 간 기능 보조에 사용합니다."
- **용법**: 만 19세 이상 및 성인은 1일 1회, 1회 1정 식후 복용 (식약처 공식)
- **성분 16종**: 비타민 C·B1(벤포티아민)·B2·B6·B12·나이아신아미드·판토텐산·비오틴·엽산·D·E + 아연·마그네슘·셀레늄 + 우르소데옥시콜산·타우린 (함량은 라벨 참조)

## 의약품↔보충제 성분 중복 점검

| 성분 | 보충제 (mg/µg) | 의약품 | 누적 판단 | 비고 |
|------|-----|-----|-----|-----|
| 비타민 B1 (티아민) | 5 mg | 함량 표기 없음 — 라벨 필요 | ⚠️ 중복 가능 | 벤포티아민 형태 (지용성, 흡수율 높음). 약사·의사와 상담 권고 |
| 비타민 B2 (리보플라빈) | 5 mg | 함량 표기 없음 | ⚠️ 중복 가능 | 위와 동일 |
| 비타민 B6 (피리독신) | 5 mg | 함량 표기 없음 | ⚠️ 중복 가능 | B6는 UL 67mg(KDRI)·100mg(NIH) — 고용량 장기 복용 시 말초신경증 주의. 상담 권고 |
| 비타민 B12 | 5 µg | 함량 표기 없음 | ⚠️ 중복 가능 | 수용성, UL 미설정 |
| 엽산 | 200 µg | 함량 표기 없음 | ⚠️ 중복 가능 | UL 1,000 µg — 의약품 함량 확인 필요 |

※ 의약품 함량 정보는 식약처 허가정보 API에서 제공되지 않습니다. 정확한 판단을 위해 제품 라벨(또는 제품설명서)을 확인하세요.

## 의약품 상호작용·주의사항

- 🟡 **Mega-Z Multi-Tab ↔ 레보도파 제제**: 식약처 공시 — "레보도파 제제와 병용 시 효과가 감소할 수 있으므로 주의." 본 스택에는 해당 없음 (참고용). [출처: Tier 2 / 식약처 e약은요 / itemSeq=202400777]
- ℹ️ **임부·수유부·신장 질환자**: 식약처 공시 — 복용 전 의사 상담 권고. 본 프로필 (43세 남성, 처방약 없음)은 해당되지 않음

## 스택 최적화 추천

### ✅ 유지
- **NutriCore Daily-B** — 보충제 기준 B군 RDA 100~417% 범위로 유지 가능

### ⚠️ 판단 필요 (의사·약사 상담 영역)
- **Mega-Z Multi-Tab**: 보충제와 비타민 B군이 중복됩니다. 본 스킬은 의약품 복용량 조정·중단을 권고하지 않습니다. 약국 상담 시 다음을 확인해보세요:
 1. Mega-Z의 비타민 B6 함량이 몇 mg인지 (NutriCore 5mg과 합산해 KDRI UL 67mg에 근접하는지)
 2. "만성 피로" 목적이라면 Mega-Z 단독으로 충분한지, 아니면 NutriCore와 병용이 필요한지
 3. 우르소데옥시콜산·타우린이 간 기능 보조 목적이라면, 본인 간 기능 상태 기준 실질 이득이 있는지

### 🆕 추가 검토
- 스트레스·수면 이슈 → 마그네슘 글리시네이트·L-테아닌 등 본 스택 밖 성분 추가 검토 가능

## 대체 제품 비교

**국내·해외에서 구할 수 있는 대체 제품** (보충제 NutriCore Daily-B 기준. Mega-Z는 일반의약품 — §8-A #12에 따라 대체 제안 제외)

| 카테고리 | 제품 | 성분 요약 | 가격 | 출처 |
|---|---|---|---|---|
| 국내 대체 1 | (가상) SmartB Complex | 메틸B12 + P5P 활성형 | (예시) 25,000원/월 | 가상 링크 |
| ... | ... | ... | ... | ... |

## 📁 리포트 저장

이 리포트는 `user-data/reports/{profile-id}/2026-XX-XX_otc-comparison.md`에 자동 저장되었습니다.

최신 리포트: `user-data/reports/{profile-id}/latest.md`

---

## ⚠️ 면책 조항

본 리포트는 정보 제공 목적이며 의학적 조언이 아닙니다. 의약품 복용 변경·추가·중단은 반드시 의사 또는 약사와 상담하세요. (이하 references/disclaimer.md §1 원문)
````

---

## 5. 검증 체크리스트 (QA)

- [ ] Fallback Chain v3 Step C → C2 자동 전환 (C003 0건 후 의약품 허가정보 조회)
- [ ] `ITEM_INGR_NAME` 정규화: Benfotiamine → `vitamin_b1`, Ursodeoxycholic Acid → `ursodeoxycholic_acid`
- [ ] `efcyQesitm`·`useMethodQesitm` 식약처 공식 문구 인용부호로 삽입
- [ ] 의약품 함량이 없으므로 "중복 가능" 표시 (정량 판정 불가)
- [ ] §8-A #11 준수: 의약품 복용량 조정·중단 권고 없음 → "의사·약사 상담" 수렴
- [ ] §8-A #12 준수: 의약품 대체 제안 섹션 생략 (보충제만 대체 후보)
- [ ] §8-A #13 분류 분기: "일반의약품"이므로 전문의약품 배너 없음
- [ ] `unique_components` 표시: 우르소데옥시콜산·타우린
- [ ] 출처 Tier 표기: `[출처: Tier 2 / 식약처 e약은요 / itemSeq=…]`
- [ ] 면책 조항 원문 삽입

---

## 6. 범용 스킬 검증 (Level 2 규약)

- [x] 실사용자 개인정보 없음 — 가상 페르소나·가상 브랜드
- [x] 비맥스제트·아로나민 등 실제 상표명 미사용 (유사 케이스이되 상표권 회피)
- [x] CLAUDE.md §0 원칙: 테스트 특이점을 규칙으로 맹목 확장 안 함
