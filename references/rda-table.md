# RDA Table — 성별·연령별 권장·상한 섭취량

> **용도**: 성분별 RDA/AI/UL 조회 (Claude 내부 참조용 lookup 테이블).
> **기관 병기**: NIH DRI (미국 국립보건원) + 식약처 KDRI 2020 (한국).
> **대상**: 19세 이상 성인. **영유아(<12세)·아동(12-18세)은 분석 범위 외**.
> **단위**: `references/ingredient-aliases.md` §0 변환식 참조.

---

## ⚠️ 데이터 검증 필수 경고

본 테이블의 수치는 **Claude 훈련 데이터 기반 초안**입니다. 배포 전 다음 공식 출처와 **교차 검증 필수**:
- NIH ODS Nutrient Recommendations: <https://ods.od.nih.gov/HealthInformation/Dietary_Reference_Intakes.aspx>
- 식약처 한국인 영양소 섭취기준 KDRI 2020 PDF: 보건복지부·한국영양학회 공동 발간 (<https://www.mohw.go.kr>)

검증되지 않은 행은 `⚠️검증` 플래그 표시. 임상 적용 전 원본 재확인.

---

## 1. 용어 정의

| 약어 | 정식명 | 의미 |
|------|--------|------|
| EAR | Estimated Average Requirement / 평균필요량 | 인구 50%가 충족 |
| **RDA** | Recommended Dietary Allowance / **권장섭취량** | 인구 97-98%가 충족 (목표) |
| **AI** | Adequate Intake / **충분섭취량** | RDA 데이터 부족 시 대체 기준 |
| **UL** | Tolerable Upper Intake Level / **상한섭취량** | **이 이상 섭취 시 부작용 위험** |
| ND | Not Determined | 근거 부족으로 UL 미설정 (과다 리스크 낮음, 단 체질 주의) |
| +값 | 부가량 | 식약처 기준. 임신·수유는 일반 성인 RDA에 가산 |
| CDRR | Chronic Disease Risk Reduction | NIH 만성질환 감소 목표 (나트륨) |

### 1-1. 연령 구간 (기관별 원본 구간 병기)

- **NIH**: 19-30 / 31-50 / 51-70 / 71+ (4구간)
- **식약처 KDRI 2020**: 19-29 / 30-49 / 50-64 / 65-74 / 75+ (5구간)
- 연속 구간에서 수치가 동일한 경우 "19-70" 등으로 **병합 기재**. 원본 구간은 비고에 명시.

### 1-2. 테이블 스키마

```
영양소 | 기관 | 대상 | 연령 | RDA/AI | UL/상한 | 단위 | 비고 | [출처]
```

- `RDA/AI`: 기관이 RDA를 설정했으면 RDA, 아니면 AI (비고 명시)
- `UL/상한`: 없으면 `ND`
- `출처`: `NIH` 또는 `KDRI` 태그. 전체 URL은 §10 출처 매핑 참조

---

## 2. 비타민 A (Retinol, mcg RAE)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| vitamin_a | NIH | 남 | 19+ | 900 | 3000 | mcg RAE | 19-71+ 동일. UL은 preformed retinol 기준 | NIH |
| vitamin_a | NIH | 여 | 19+ | 700 | 3000 | mcg RAE | 19-71+ 동일 | NIH |
| vitamin_a | NIH | 임신 | 19+ | 770 | 3000 | mcg RAE | 고용량 retinol 기형 유발 주의 🔴 | NIH |
| vitamin_a | NIH | 수유 | 19+ | 1300 | 3000 | mcg RAE | - | NIH |
| vitamin_a | KDRI | 남 | 19-49 | 800 | 3000 | mcg RAE | 식약처 구간: 19-29·30-49 동일 | KDRI |
| vitamin_a | KDRI | 남 | 50-64 | 750 | 3000 | mcg RAE | - | KDRI |
| vitamin_a | KDRI | 남 | 65+ | 700 | 3000 | mcg RAE | 식약처 구간: 65-74·75+ 동일 | KDRI |
| vitamin_a | KDRI | 여 | 19-49 | 650 | 3000 | mcg RAE | 식약처 구간: 19-29·30-49 동일 | KDRI |
| vitamin_a | KDRI | 여 | 50-64 | 600 | 3000 | mcg RAE | - | KDRI |
| vitamin_a | KDRI | 여 | 65+ | 550 | 3000 | mcg RAE | - | KDRI |
| vitamin_a | KDRI | 임신 | 19+ | +70 | 3000 | mcg RAE | 부가량 | KDRI |
| vitamin_a | KDRI | 수유 | 19+ | +490 | 3000 | mcg RAE | 부가량 | KDRI |

---

## 3. 비타민 D (Cholecalciferol, mcg)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| vitamin_d | NIH | 남 | 19-70 | 15 | 100 | mcg | = 600 IU / UL = 4000 IU | NIH |
| vitamin_d | NIH | 남 | 71+ | 20 | 100 | mcg | = 800 IU | NIH |
| vitamin_d | NIH | 여 | 19-70 | 15 | 100 | mcg | - | NIH |
| vitamin_d | NIH | 여 | 71+ | 20 | 100 | mcg | - | NIH |
| vitamin_d | NIH | 임신 | 19+ | 15 | 100 | mcg | - | NIH |
| vitamin_d | NIH | 수유 | 19+ | 15 | 100 | mcg | - | NIH |
| vitamin_d | KDRI | 남 | 19-64 | 10 | 100 | mcg | AI. 식약처 2020 | KDRI |
| vitamin_d | KDRI | 남 | 65+ | 15 | 100 | mcg | AI | KDRI |
| vitamin_d | KDRI | 여 | 19-64 | 10 | 100 | mcg | AI | KDRI |
| vitamin_d | KDRI | 여 | 65+ | 15 | 100 | mcg | AI | KDRI |
| vitamin_d | KDRI | 임신 | 19+ | +0 | 100 | mcg | 부가량 없음 | KDRI |
| vitamin_d | KDRI | 수유 | 19+ | +0 | 100 | mcg | 부가량 없음 | KDRI |

> **운영 규칙**: 비타민 D 보충제는 IU 표기가 많음. **계산 시 반드시 mcg로 환산** (1 mcg = 40 IU). 5000 IU = 125 mcg → UL 25% 초과 🔴.

---

## 4. 비타민 E (alpha-Tocopherol, mg)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| vitamin_e | NIH | 남 | 19+ | 15 | 1000 | mg alpha-TE | UL은 합성형(dl-) 기준 | NIH |
| vitamin_e | NIH | 여 | 19+ | 15 | 1000 | mg alpha-TE | - | NIH |
| vitamin_e | NIH | 임신 | 19+ | 15 | 1000 | mg alpha-TE | - | NIH |
| vitamin_e | NIH | 수유 | 19+ | 19 | 1000 | mg alpha-TE | - | NIH |
| vitamin_e | KDRI | 남 | 19+ | 12 | 540 | mg alpha-TE | AI. 식약처 UL 상이 | KDRI |
| vitamin_e | KDRI | 여 | 19+ | 12 | 540 | mg alpha-TE | AI | KDRI |
| vitamin_e | KDRI | 임신 | 19+ | +0 | 540 | mg alpha-TE | 부가량 없음 | KDRI |
| vitamin_e | KDRI | 수유 | 19+ | +3 | 540 | mg alpha-TE | 부가량 | KDRI |

> **경고**: NIH UL 1000 mg vs 식약처 540 mg 차이 큼. 안전 측면에서 **더 낮은 540 mg 기준 경고** 권장.

---

## 5. 비타민 K (Phylloquinone, mcg)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| vitamin_k | NIH | 남 | 19+ | 120 | ND | mcg | AI | NIH |
| vitamin_k | NIH | 여 | 19+ | 90 | ND | mcg | AI | NIH |
| vitamin_k | NIH | 임신·수유 | 19+ | 90 | ND | mcg | - | NIH |
| vitamin_k | KDRI | 남 | 19+ | 75 | ND | mcg | AI | KDRI |
| vitamin_k | KDRI | 여 | 19+ | 65 | ND | mcg | AI | KDRI |
| vitamin_k | KDRI | 임신 | 19+ | +0 | ND | mcg | - | KDRI |
| vitamin_k | KDRI | 수유 | 19+ | +0 | ND | mcg | - | KDRI |

> **상호작용 경고**: 와파린(warfarin) 복용자는 비타민 K 변동성 주의 — §interaction-patterns.md 참조.

---

## 6. 티아민 (Vitamin B1, mg)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| thiamine | NIH | 남 | 19+ | 1.2 | ND | mg | - | NIH |
| thiamine | NIH | 여 | 19+ | 1.1 | ND | mg | - | NIH |
| thiamine | NIH | 임신·수유 | 19+ | 1.4 | ND | mg | - | NIH |
| thiamine | KDRI | 남 | 19+ | 1.2 | ND | mg | - | KDRI |
| thiamine | KDRI | 여 | 19+ | 1.1 | ND | mg | - | KDRI |
| thiamine | KDRI | 임신 | 19+ | +0.4 | ND | mg | 부가량 | KDRI |
| thiamine | KDRI | 수유 | 19+ | +0.4 | ND | mg | 부가량 | KDRI |

---

## 7. 리보플라빈 (Vitamin B2, mg)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| riboflavin | NIH | 남 | 19+ | 1.3 | ND | mg | - | NIH |
| riboflavin | NIH | 여 | 19+ | 1.1 | ND | mg | - | NIH |
| riboflavin | NIH | 임신 | 19+ | 1.4 | ND | mg | - | NIH |
| riboflavin | NIH | 수유 | 19+ | 1.6 | ND | mg | - | NIH |
| riboflavin | KDRI | 남 | 19-49 | 1.5 | ND | mg | - | KDRI |
| riboflavin | KDRI | 남 | 50+ | 1.4 | ND | mg | - | KDRI |
| riboflavin | KDRI | 여 | 19+ | 1.2 | ND | mg | - | KDRI |
| riboflavin | KDRI | 임신 | 19+ | +0.4 | ND | mg | 부가량 | KDRI |
| riboflavin | KDRI | 수유 | 19+ | +0.5 | ND | mg | 부가량 | KDRI |

---

## 8. 나이아신 (Vitamin B3, mg NE)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| niacin | NIH | 남 | 19+ | 16 | 35 | mg NE | UL은 nicotinic acid 기준 (홍조/간독성) | NIH |
| niacin | NIH | 여 | 19+ | 14 | 35 | mg NE | - | NIH |
| niacin | NIH | 임신 | 19+ | 18 | 35 | mg NE | - | NIH |
| niacin | NIH | 수유 | 19+ | 17 | 35 | mg NE | - | NIH |
| niacin | KDRI | 남 | 19+ | 16 | 35/1000 | mg NE | 니코틴산 35 / 니코틴아미드 1000 | KDRI |
| niacin | KDRI | 여 | 19+ | 14 | 35/1000 | mg NE | - | KDRI |
| niacin | KDRI | 임신 | 19+ | +4 | 35/1000 | mg NE | 부가량 | KDRI |
| niacin | KDRI | 수유 | 19+ | +3 | 35/1000 | mg NE | 부가량 | KDRI |

> **형태별 UL 차이**: 니코틴산(nicotinic acid) UL 35 mg 초과 시 홍조·간독성 / 니코틴아미드(nicotinamide) UL 1000 mg (식약처). 라벨의 형태 반드시 확인.

---

## 9. 판토텐산 (Vitamin B5, mg)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| pantothenic_acid | NIH | 남·여 | 19+ | 5 | ND | mg | AI | NIH |
| pantothenic_acid | NIH | 임신 | 19+ | 6 | ND | mg | - | NIH |
| pantothenic_acid | NIH | 수유 | 19+ | 7 | ND | mg | - | NIH |
| pantothenic_acid | KDRI | 남·여 | 19+ | 5 | ND | mg | AI | KDRI |
| pantothenic_acid | KDRI | 임신 | 19+ | +1.0 | ND | mg | 부가량 | KDRI |
| pantothenic_acid | KDRI | 수유 | 19+ | +2.0 | ND | mg | 부가량 | KDRI |

---

## 10. 피리독신 (Vitamin B6, mg)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| pyridoxine | NIH | 남 | 19-50 | 1.3 | 100 | mg | UL 초과 시 말초신경병증 🔴 | NIH |
| pyridoxine | NIH | 남 | 51+ | 1.7 | 100 | mg | - | NIH |
| pyridoxine | NIH | 여 | 19-50 | 1.3 | 100 | mg | - | NIH |
| pyridoxine | NIH | 여 | 51+ | 1.5 | 100 | mg | - | NIH |
| pyridoxine | NIH | 임신 | 19+ | 1.9 | 100 | mg | - | NIH |
| pyridoxine | NIH | 수유 | 19+ | 2.0 | 100 | mg | - | NIH |
| pyridoxine | KDRI | 남 | 19+ | 1.5 | 100 | mg | - | KDRI |
| pyridoxine | KDRI | 여 | 19+ | 1.4 | 100 | mg | - | KDRI |
| pyridoxine | KDRI | 임신 | 19+ | +0.8 | 100 | mg | 부가량 | KDRI |
| pyridoxine | KDRI | 수유 | 19+ | +0.8 | 100 | mg | 부가량 | KDRI |

---

## 11. 비오틴 (Vitamin B7, mcg)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| biotin | NIH | 남·여 | 19+ | 30 | ND | mcg | AI | NIH |
| biotin | NIH | 임신 | 19+ | 30 | ND | mcg | - | NIH |
| biotin | NIH | 수유 | 19+ | 35 | ND | mcg | - | NIH |
| biotin | KDRI | 남·여 | 19+ | 30 | ND | mcg | AI | KDRI |
| biotin | KDRI | 임신 | 19+ | +0 | ND | mcg | - | KDRI |
| biotin | KDRI | 수유 | 19+ | +5 | ND | mcg | 부가량 | KDRI |

> **검사 간섭 주의**: 고용량 비오틴(≥5000 mcg) 복용 시 갑상선·심장 효소 검사 결과 왜곡 가능. FDA 경고 (2017).

---

## 12. 엽산 (Vitamin B9, mcg DFE)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| folate | NIH | 남·여 | 19+ | 400 | 1000 | mcg DFE | UL은 보충제(folic acid) 기준 | NIH |
| folate | NIH | 임신 | 19+ | 600 | 1000 | mcg DFE | 신경관 결손 예방 | NIH |
| folate | NIH | 수유 | 19+ | 500 | 1000 | mcg DFE | - | NIH |
| folate | KDRI | 남·여 | 19+ | 400 | 1000 | mcg DFE | - | KDRI |
| folate | KDRI | 임신 | 19+ | +220 | 1000 | mcg DFE | 부가량 | KDRI |
| folate | KDRI | 수유 | 19+ | +150 | 1000 | mcg DFE | 부가량 | KDRI |

> **가임기 여성 권장**: 임신 계획 3개월 전부터 folic acid 400-800 mcg/day (CDC 권고). 고용량 folic acid는 B12 결핍 마스킹 주의.

---

## 13. 코발라민 (Vitamin B12, mcg)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| cobalamin | NIH | 남·여 | 19+ | 2.4 | ND | mcg | 51+는 강화식품/보충제 권장 (흡수 저하) | NIH |
| cobalamin | NIH | 임신 | 19+ | 2.6 | ND | mcg | - | NIH |
| cobalamin | NIH | 수유 | 19+ | 2.8 | ND | mcg | - | NIH |
| cobalamin | KDRI | 남·여 | 19+ | 2.4 | ND | mcg | - | KDRI |
| cobalamin | KDRI | 임신 | 19+ | +0.2 | ND | mcg | 부가량 | KDRI |
| cobalamin | KDRI | 수유 | 19+ | +0.4 | ND | mcg | 부가량 | KDRI |

---

## 14. 비타민 C (Ascorbic acid, mg)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| vitamin_c | NIH | 남 | 19+ | 90 | 2000 | mg | 흡연자 +35 mg | NIH |
| vitamin_c | NIH | 여 | 19+ | 75 | 2000 | mg | 흡연자 +35 mg | NIH |
| vitamin_c | NIH | 임신 | 19+ | 85 | 2000 | mg | - | NIH |
| vitamin_c | NIH | 수유 | 19+ | 120 | 2000 | mg | - | NIH |
| vitamin_c | KDRI | 남·여 | 19+ | 100 | 2000 | mg | - | KDRI |
| vitamin_c | KDRI | 임신 | 19+ | +10 | 2000 | mg | 부가량 | KDRI |
| vitamin_c | KDRI | 수유 | 19+ | +40 | 2000 | mg | 부가량 | KDRI |

> **UL 초과 증상**: >2000 mg 설사·복통. 만성 고용량은 신결석 위험.

---

## 15. 칼슘 (Ca, mg 원소)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| calcium | NIH | 남 | 19-70 | 1000 | 2500 | mg | UL 19-50 | NIH |
| calcium | NIH | 남 | 71+ | 1200 | 2000 | mg | UL ↓ | NIH |
| calcium | NIH | 여 | 19-50 | 1000 | 2500 | mg | - | NIH |
| calcium | NIH | 여 | 51+ | 1200 | 2000 | mg | 폐경 후 | NIH |
| calcium | NIH | 임신·수유 | 19-50 | 1000 | 2500 | mg | - | NIH |
| calcium | KDRI | 남 | 19-49 | 800 | 2500 | mg | - | KDRI |
| calcium | KDRI | 남 | 50-64 | 750 | 2500 | mg | - | KDRI |
| calcium | KDRI | 남 | 65+ | 700 | 2000 | mg | - | KDRI |
| calcium | KDRI | 여 | 19-49 | 700 | 2500 | mg | - | KDRI |
| calcium | KDRI | 여 | 50-64 | 800 | 2500 | mg | 폐경 | KDRI |
| calcium | KDRI | 여 | 65+ | 800 | 2000 | mg | - | KDRI |
| calcium | KDRI | 임신 | 19+ | +0 | 2500 | mg | - | KDRI |
| calcium | KDRI | 수유 | 19+ | +0 | 2500 | mg | - | KDRI |

> **흡수 상한**: 1회 500 mg 이하로 분할 복용 권장 (Ca 흡수 포화).

---

## 16. 마그네슘 (Mg, mg 원소)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| magnesium | NIH | 남 | 19-30 | 400 | 350 | mg | UL은 보충제 기준 (식이 제외) | NIH |
| magnesium | NIH | 남 | 31+ | 420 | 350 | mg | - | NIH |
| magnesium | NIH | 여 | 19-30 | 310 | 350 | mg | - | NIH |
| magnesium | NIH | 여 | 31+ | 320 | 350 | mg | - | NIH |
| magnesium | NIH | 임신 | 19-30 | 350 | 350 | mg | - | NIH |
| magnesium | NIH | 임신 | 31+ | 360 | 350 | mg | - | NIH |
| magnesium | NIH | 수유 | 19-30 | 310 | 350 | mg | - | NIH |
| magnesium | NIH | 수유 | 31+ | 320 | 350 | mg | - | NIH |
| magnesium | KDRI | 남 | 19-29 | 360 | 350 | mg | UL은 보충제 기준 | KDRI |
| magnesium | KDRI | 남 | 30-64 | 370 | 350 | mg | - | KDRI |
| magnesium | KDRI | 남 | 65-74 | 350 | 350 | mg | - | KDRI |
| magnesium | KDRI | 남 | 75+ | 310 | 350 | mg | - | KDRI |
| magnesium | KDRI | 여 | 19-29 | 280 | 350 | mg | - | KDRI |
| magnesium | KDRI | 여 | 30-64 | 280 | 350 | mg | - | KDRI |
| magnesium | KDRI | 여 | 65-74 | 280 | 350 | mg | - | KDRI |
| magnesium | KDRI | 여 | 75+ | 260 | 350 | mg | - | KDRI |
| magnesium | KDRI | 임신 | 19+ | +40 | 350 | mg | 부가량 | KDRI |
| magnesium | KDRI | 수유 | 19+ | +0 | 350 | mg | - | KDRI |

> **UL 해석**: 보충제/약물 기준 350 mg. 식사 유래 Mg는 UL에 포함되지 않음. UL 초과 시 설사.

---

## 17. 철 (Fe, mg 원소)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| iron | NIH | 남 | 19+ | 8 | 45 | mg | - | NIH |
| iron | NIH | 여 | 19-50 | 18 | 45 | mg | 월경 | NIH |
| iron | NIH | 여 | 51+ | 8 | 45 | mg | 폐경 | NIH |
| iron | NIH | 임신 | 19+ | 27 | 45 | mg | 임신 전 필수 | NIH |
| iron | NIH | 수유 | 19+ | 9 | 45 | mg | - | NIH |
| iron | KDRI | 남 | 19-49 | 10 | 45 | mg | - | KDRI |
| iron | KDRI | 남 | 50+ | 9 | 45 | mg | - | KDRI |
| iron | KDRI | 여 | 19-49 | 14 | 45 | mg | - | KDRI |
| iron | KDRI | 여 | 50-64 | 8 | 45 | mg | 폐경 | KDRI |
| iron | KDRI | 여 | 65+ | 7 | 45 | mg | - | KDRI |
| iron | KDRI | 임신 | 19+ | +10 | 45 | mg | 부가량 | KDRI |
| iron | KDRI | 수유 | 19+ | +0 | 45 | mg | - | KDRI |

> **흡수**: 비타민 C 동반 시 흡수↑, 칼슘·탄닌(차·커피) 동반 시 흡수↓ (§interaction-patterns.md).

---

## 18. 아연 (Zn, mg 원소)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| zinc | NIH | 남 | 19+ | 11 | 40 | mg | - | NIH |
| zinc | NIH | 여 | 19+ | 8 | 40 | mg | - | NIH |
| zinc | NIH | 임신 | 19+ | 11 | 40 | mg | - | NIH |
| zinc | NIH | 수유 | 19+ | 12 | 40 | mg | - | NIH |
| zinc | KDRI | 남 | 19-49 | 10 | 35 | mg | 식약처 UL 더 낮음 | KDRI |
| zinc | KDRI | 남 | 50+ | 9 | 35 | mg | - | KDRI |
| zinc | KDRI | 여 | 19-49 | 8 | 35 | mg | - | KDRI |
| zinc | KDRI | 여 | 50+ | 7 | 35 | mg | - | KDRI |
| zinc | KDRI | 임신 | 19+ | +2.5 | 35 | mg | 부가량 | KDRI |
| zinc | KDRI | 수유 | 19+ | +5.0 | 35 | mg | 부가량 | KDRI |

> **경고**: 아연 고용량 장기 복용 시 구리 결핍 유발. 50 mg/day 이상 장기 복용 주의.

---

## 19. 셀레늄 (Se, mcg)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| selenium | NIH | 남·여 | 19+ | 55 | 400 | mcg | - | NIH |
| selenium | NIH | 임신 | 19+ | 60 | 400 | mcg | - | NIH |
| selenium | NIH | 수유 | 19+ | 70 | 400 | mcg | - | NIH |
| selenium | KDRI | 남·여 | 19+ | 60 | 400 | mcg | - | KDRI |
| selenium | KDRI | 임신 | 19+ | +4 | 400 | mcg | 부가량 | KDRI |
| selenium | KDRI | 수유 | 19+ | +20 | 400 | mcg | 부가량 | KDRI |

> **셀레니움 중독(selenosis)**: >400 mcg 장기 복용 시 탈모·손톱 변형.

---

## 20. 구리 (Cu, mcg)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| copper | NIH | 남·여 | 19+ | 900 | 10000 | mcg | = 0.9 / 10 mg | NIH |
| copper | NIH | 임신 | 19+ | 1000 | 10000 | mcg | - | NIH |
| copper | NIH | 수유 | 19+ | 1300 | 10000 | mcg | - | NIH |
| copper | KDRI | 남 | 19+ | 850 | 10000 | mcg | - | KDRI |
| copper | KDRI | 여 | 19+ | 650 | 10000 | mcg | - | KDRI |
| copper | KDRI | 임신 | 19+ | +130 | 10000 | mcg | 부가량 | KDRI |
| copper | KDRI | 수유 | 19+ | +480 | 10000 | mcg | 부가량 | KDRI |

---

## 21. 망간 (Mn, mg)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| manganese | NIH | 남 | 19+ | 2.3 | 11 | mg | AI | NIH |
| manganese | NIH | 여 | 19+ | 1.8 | 11 | mg | AI | NIH |
| manganese | NIH | 임신 | 19+ | 2.0 | 11 | mg | - | NIH |
| manganese | NIH | 수유 | 19+ | 2.6 | 11 | mg | - | NIH |
| manganese | KDRI | 남 | 19+ | 4.0 | 11 | mg | AI | KDRI |
| manganese | KDRI | 여 | 19+ | 3.5 | 11 | mg | AI | KDRI |
| manganese | KDRI | 임신 | 19+ | +0 | 11 | mg | - | KDRI |
| manganese | KDRI | 수유 | 19+ | +0 | 11 | mg | - | KDRI |

---

## 22. 요오드 (I, mcg)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| iodine | NIH | 남·여 | 19+ | 150 | 1100 | mcg | - | NIH |
| iodine | NIH | 임신 | 19+ | 220 | 1100 | mcg | - | NIH |
| iodine | NIH | 수유 | 19+ | 290 | 1100 | mcg | - | NIH |
| iodine | KDRI | 남·여 | 19+ | 150 | 2400 | mcg | 식약처 UL 더 높음 (해조류 섭취 반영) | KDRI |
| iodine | KDRI | 임신 | 19+ | +90 | - | mcg | 임신 UL 별도 (식약처 참조) | KDRI |
| iodine | KDRI | 수유 | 19+ | +190 | - | mcg | - | KDRI |

> **한국 특수성**: 해조류 섭취 많음. 다시마·미역 과다 + 보충제 병용 시 갑상선 기능 이상 주의.

---

## 23. 크롬 (Cr, mcg)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| chromium | NIH | 남 | 19-50 | 35 | ND | mcg | AI | NIH |
| chromium | NIH | 남 | 51+ | 30 | ND | mcg | AI | NIH |
| chromium | NIH | 여 | 19-50 | 25 | ND | mcg | AI | NIH |
| chromium | NIH | 여 | 51+ | 20 | ND | mcg | AI | NIH |
| chromium | NIH | 임신 | 19+ | 30 | ND | mcg | - | NIH |
| chromium | NIH | 수유 | 19+ | 45 | ND | mcg | - | NIH |
| chromium | KDRI | 남 | 19+ | 30 | ND | mcg | AI | KDRI |
| chromium | KDRI | 여 | 19+ | 20 | ND | mcg | AI | KDRI |
| chromium | KDRI | 임신·수유 | 19+ | +0 | ND | mcg | - | KDRI |

---

## 24. 몰리브덴 (Mo, mcg)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| molybdenum | NIH | 남·여 | 19+ | 45 | 2000 | mcg | - | NIH |
| molybdenum | NIH | 임신·수유 | 19+ | 50 | 2000 | mcg | - | NIH |
| molybdenum | KDRI | 남 | 19+ | 30 | 600 | mcg | 식약처 UL 더 낮음 | KDRI |
| molybdenum | KDRI | 여 | 19+ | 25 | 500 | mcg | - | KDRI |

---

## 25. 칼륨 (K, mg 원소)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| potassium | NIH | 남 | 19+ | 3400 | ND | mg | AI | NIH |
| potassium | NIH | 여 | 19+ | 2600 | ND | mg | AI | NIH |
| potassium | NIH | 임신 | 19+ | 2900 | ND | mg | - | NIH |
| potassium | NIH | 수유 | 19+ | 2800 | ND | mg | - | NIH |
| potassium | KDRI | 남·여 | 19+ | 3500 | ND | mg | AI | KDRI |
| potassium | KDRI | 임신·수유 | 19+ | +0/+400 | ND | mg | 임신 0 / 수유 +400 | KDRI |

> **신장질환자 주의**: 고칼륨혈증 위험. ACE 억제제·ARB·칼륨보존 이뇨제 병용 시 UL 없어도 부가 섭취 주의 🔴.

---

## 26. 나트륨 (Na, mg 원소)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL/CDRR | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| sodium | NIH | 남·여 | 19+ | 1500 | 2300 (CDRR) | mg | AI 1500 / 만성질환 감소 2300 | NIH |
| sodium | KDRI | 남·여 | 19-64 | 1500 | 2000 (목표) | mg | AI. 목표섭취량 2000 미만 | KDRI |
| sodium | KDRI | 남·여 | 65-74 | 1300 | 2000 | mg | - | KDRI |
| sodium | KDRI | 남·여 | 75+ | 1100 | 2000 | mg | - | KDRI |

> **보충제 고려**: 일반적으로 보충제에는 포함 안 됨. 식이 섭취만 평가.

---

## 27. 인 (P, mg 원소)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| phosphorus | NIH | 남·여 | 19-70 | 700 | 4000 | mg | - | NIH |
| phosphorus | NIH | 남·여 | 71+ | 700 | 3000 | mg | UL ↓ | NIH |
| phosphorus | NIH | 임신·수유 | 19+ | 700 | 3500 | mg | 임신 UL | NIH |
| phosphorus | KDRI | 남·여 | 19+ | 700 | 3500 | mg | - | KDRI |
| phosphorus | KDRI | 임신·수유 | 19+ | +0 | 3500 | mg | - | KDRI |

---

## 28. 붕소 (B, mg)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| boron | NIH | 남·여 | 19+ | **미설정** | 20 | mg | RDA/AI 없음. UL만 설정 | NIH |
| boron | KDRI | — | — | **미설정** | — | mg | 식약처 기준 없음 | — |

---

## 29. 바나듐 (V, mcg)

| 영양소 | 기관 | 대상 | 연령 | RDA/AI | UL | 단위 | 비고 | 출처 |
|---|---|---|---|---|---|---|---|---|
| vanadium | NIH | 남·여 | 19+ | **미설정** | 1800 | mcg | UL만 설정 (= 1.8 mg) | NIH |
| vanadium | KDRI | — | — | **미설정** | — | — | 식약처 기준 없음 | — |

---

## 30. RDA/UL 기준이 없는 성분 (안내)

아래 성분들은 NIH·식약처 DRI가 설정되지 않았습니다. **"일반 안전 범위"는 임상 연구 기반 권장치**이며, 기관 공식 기준이 아닙니다. 분석 시 반드시 [출처: 임상연구 URL]로 인용.

### 30-1. 아미노산 (§3 ingredient-aliases.md)
- **L-테아닌**: 연구 용량 100-400 mg/day (Examine 등급)
- **L-아르기닌**: 3-6 g/day (혈관 건강 연구)
- **L-카르니틴**: 500-2000 mg/day
- **타우린**: 500-2000 mg/day
- **BCAA**: 5-20 g/day (운동)
- **L-트립토판**: 500-2000 mg/day (수면)

### 30-2. 오메가-3
- **EPA+DHA**: AHA 권고 일반 성인 250-500 mg/day, 심혈관 고위험 1000 mg/day, 고중성지방 2-4 g/day (의사 감독)
- **EFSA UL**: EPA+DHA 보충제 5 g/day

### 30-3. 식물추출물
- **실리마린**: 140-420 mg/day (간기능)
- **커큐민**: 500-2000 mg/day (피페린/포뮬레이션 영향)
- **아슈와간다 KSM-66**: 300-600 mg/day
- **홍경천**: 200-600 mg/day
- **은행잎 (EGb 761)**: 120-240 mg/day

### 30-4. 프로바이오틱스
- 일반 유지: 10억-100억 CFU/day (1-100 × 10^9)
- 치료 목적: 100억-1000억 CFU/day (임상 연구별 상이)

### 30-5. 기타 기능성
- **CoQ10**: 100-200 mg/day (심부전 300-600 mg 의사 감독)
- **루테인**: 6-20 mg/day (AREDS2 10 mg)
- **지아잔틴**: 2 mg/day (AREDS2)
- **아스타잔틴**: 4-12 mg/day
- **멜라토닌**: 0.5-5 mg 취침 30분 전 (단기)
- **5-HTP**: 50-300 mg/day (우울·수면)
- **NAC**: 600-1800 mg/day
- **알파리포산**: 300-600 mg/day
- **크레아틴**: 3-5 g/day (유지기)

> 위 수치는 **일반 임상 연구 범위**로 UL이 아님. 고용량 사용 시 반드시 개별 근거 조회 (Tier 1~2).

---

## 31. 조회 알고리즘 (Claude 참조용)

RDA 테이블 조회 순서:

1. **canonical_name 매칭** → `ingredient-aliases.md` 결과로 정규화된 이름 사용
2. **프로필 파싱** → 성별(M/F/임신/수유), 연령
3. **기관 선택**: 양쪽 모두 조회 (사용자 한국인 기본값: 식약처 우선 + NIH 병기)
4. **행 선택**: 연령 범위에 매칭되는 행
5. **계산**:
 - `RDA% = (실제_용량 / RDA) × 100`
 - `UL% = (실제_용량 / UL) × 100`
6. **플래그 규칙**:
 - UL% ≥ 100 → 🔴 **UL 초과 경고** (리포트 최상단)
 - UL% 70-99 → 🟠 **UL 근접 경고**
 - RDA% < 50 → 🟡 **부족 가능성**
 - RDA% 70-150 → ✅ **적정**
 - RDA% > 300 (수용성) / > 200 (지용성) → 🟡 **고용량 검토**
7. **임신·수유 해당 시**: 반드시 해당 행 사용 + 리포트 최상단 경고 배너
8. **UL = ND인 영양소**: "UL 미설정 (일반 식이 안전 범위). 보충제 고용량은 개별 근거 조회 권장"으로 표시

---

## 32. 영양소별 교차 검증 체크리스트 (배포 전)

⚠️ 다음 영양소는 **치명적 오류 위험도 상**. 배포 전 공식 출처 재검증 필수.

- [ ] 비타민 D UL (100 mcg = 4000 IU) — 5000 IU 제품 경고 정확성
- [ ] 비타민 B6 UL (100 mg) — 말초신경병증 경고
- [ ] 니아신 UL — 니코틴산 35 mg vs 니코틴아미드 1000 mg 구분
- [ ] 철 UL (45 mg) — 임신부 고용량
- [ ] 엽산 UL (1000 mcg DFE) — B12 마스킹
- [ ] 셀레늄 UL (400 mcg) — selenosis
- [ ] 마그네슘 UL (350 mg, 보충제 기준) — 식이 제외 해석
- [ ] 비타민 A UL (3000 mcg RAE) — 임신부 기형

---

## 33. 출처 매핑 (§출처 태그 → 전체 URL)

### NIH (미국 국립보건원 ODS Fact Sheets)

기본 URL 패턴: `https://ods.od.nih.gov/factsheets/{Nutrient}-HealthProfessional/`

- `NIH` 일반 DRI 요약: <https://ods.od.nih.gov/HealthInformation/Dietary_Reference_Intakes.aspx>
- Vitamin A: <https://ods.od.nih.gov/factsheets/VitaminA-HealthProfessional/>
- Vitamin D: <https://ods.od.nih.gov/factsheets/VitaminD-HealthProfessional/>
- Vitamin E: <https://ods.od.nih.gov/factsheets/VitaminE-HealthProfessional/>
- Vitamin K: <https://ods.od.nih.gov/factsheets/VitaminK-HealthProfessional/>
- Thiamin (B1): <https://ods.od.nih.gov/factsheets/Thiamin-HealthProfessional/>
- Riboflavin (B2): <https://ods.od.nih.gov/factsheets/Riboflavin-HealthProfessional/>
- Niacin (B3): <https://ods.od.nih.gov/factsheets/Niacin-HealthProfessional/>
- Pantothenic Acid (B5): <https://ods.od.nih.gov/factsheets/PantothenicAcid-HealthProfessional/>
- Vitamin B6: <https://ods.od.nih.gov/factsheets/VitaminB6-HealthProfessional/>
- Biotin (B7): <https://ods.od.nih.gov/factsheets/Biotin-HealthProfessional/>
- Folate (B9): <https://ods.od.nih.gov/factsheets/Folate-HealthProfessional/>
- Vitamin B12: <https://ods.od.nih.gov/factsheets/VitaminB12-HealthProfessional/>
- Vitamin C: <https://ods.od.nih.gov/factsheets/VitaminC-HealthProfessional/>
- Calcium: <https://ods.od.nih.gov/factsheets/Calcium-HealthProfessional/>
- Magnesium: <https://ods.od.nih.gov/factsheets/Magnesium-HealthProfessional/>
- Iron: <https://ods.od.nih.gov/factsheets/Iron-HealthProfessional/>
- Zinc: <https://ods.od.nih.gov/factsheets/Zinc-HealthProfessional/>
- Selenium: <https://ods.od.nih.gov/factsheets/Selenium-HealthProfessional/>
- Copper: <https://ods.od.nih.gov/factsheets/Copper-HealthProfessional/>
- Manganese: <https://ods.od.nih.gov/factsheets/Manganese-HealthProfessional/>
- Iodine: <https://ods.od.nih.gov/factsheets/Iodine-HealthProfessional/>
- Chromium: <https://ods.od.nih.gov/factsheets/Chromium-HealthProfessional/>
- Molybdenum: <https://ods.od.nih.gov/factsheets/Molybdenum-HealthProfessional/>
- Potassium: <https://ods.od.nih.gov/factsheets/Potassium-HealthProfessional/>
- Sodium: <https://ods.od.nih.gov/Sodium-HealthProfessional/> ⚠️검증
- Phosphorus: <https://ods.od.nih.gov/factsheets/Phosphorus-HealthProfessional/>
- Boron: <https://ods.od.nih.gov/factsheets/Boron-HealthProfessional/>
- Vanadium: NIH ODS 별도 factsheet 없음. DRI 요약 문서 참조

### KDRI (식약처 한국인 영양소 섭취기준 2020)

- 공식 발간처: 보건복지부·한국영양학회
- 다운로드: 한국영양학회 (<https://www.kns.or.kr>) 또는 보건복지부 (<https://www.mohw.go.kr>) ⚠️ 정확한 PDF URL 세션 3 테스트 시 확인
- 식약처 건강기능식품 원료DB: <https://www.foodsafetykorea.go.kr/portal/healthyfoodlife/searchHomeHFMaterial.do>

> **⚠️검증 마크**: URL 패턴이 불확실하거나 원본 접근이 필요한 항목. 세션 3 테스트에서 WebFetch로 실제 접근 확인 후 URL 확정.

