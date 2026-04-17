# Disclaimer — 면책 조항 (한/영)

> **용도**: 모든 분석 리포트 **말미에 자동 삽입**. 수정·축약·생략 금지.
> **삽입 마커**: `prompts/report-template.md`에서 `{{DISCLAIMER}}` 치환 방식.
> **출력 언어**: 사용자 요청 언어에 따라 한국어(기본) 또는 영어(`--en`). 둘 다 함께 출력해도 무방.

---

## 1. 한국어 (기본)

```markdown
---

## ⚠️ 면책 조항

**본 분석은 의학적 조언이 아닙니다.**

- 본 리포트는 공개된 연구 자료와 영양 정보를 기반으로 한 **참고용 분석**이며, 진단·치료·처방을 대체하지 않습니다.
- **기저질환, 임신·수유 중이거나 복용 중인 약물이 있는 경우** 반드시 의사·약사와 상담 후 보충제 복용을 결정하세요.
- 본 분석은 대체제품·용량에 대한 일반 정보를 제공하며, **개인별 체질·건강 상태·유전적 차이를 모두 반영하지 못합니다.**
- 제품명·브랜드는 **비교 목적의 예시**이며, 특정 제품을 추천·보증하지 않습니다.
- 가격·유통·재고 정보는 조회 시점 기준이며 변동될 수 있습니다.
- **본 분석의 정확성, 완전성, 최신성에 대한 어떠한 법적 책임도 지지 않습니다.**
- 응급 상황 또는 심각한 증상이 있을 경우 즉시 의료기관에 연락하십시오.

> 분석 결과에 대한 문제 제기 또는 정정 요청은 이 스킬을 제공한 운영자/저장소에 문의하세요.
```

---

## 2. English (`--en` 옵션)

```markdown
---

## ⚠️ Disclaimer

**This analysis is not medical advice.**

- This report is a **reference-only analysis** based on publicly available research and nutrition information. It does not substitute for diagnosis, treatment, or prescription.
- **If you have any underlying condition, are pregnant or breastfeeding, or are taking any medication**, consult a physician or pharmacist before taking supplements.
- This analysis provides general information about alternative products and dosages and **cannot account for individual variations in physiology, health status, or genetics.**
- Product names and brands mentioned are **examples for comparison** and do not constitute endorsement or recommendation of any specific product.
- Price, availability, and stock information reflect the time of query and may change.
- **No legal liability is accepted for the accuracy, completeness, or currency of this analysis.**
- In case of emergency or serious symptoms, contact a medical professional immediately.

> For concerns about or corrections to the analysis, contact the operator/repository providing this skill.
```

---

## 3. 추가 경고 배너 (조건부 삽입)

아래 조건 해당 시 **리포트 최상단에 추가 배너**. 면책 조항 말미가 아니라 **상단 TL;DR 위**.

### 3-1. 임신·수유

```markdown
> 🔴 **임신/수유 중 경고**
> 프로필에 임신·수유 상태가 확인되었습니다. 비타민 A(레티놀 ≥3000 mcg RAE) 등 일부 성분은 기형 유발 위험이 있습니다. **모든 보충제는 산부인과 의사와 상담 후 결정**하세요.

> 🔴 **Pregnancy/Breastfeeding Alert**
> Your profile indicates pregnancy or breastfeeding. Some supplements (e.g., Vitamin A as retinol ≥3000 mcg RAE) carry teratogenic risk. **Consult your OB-GYN before any supplement use.**
```

### 3-2. 영유아·반려동물 입력 감지 시 (분석 거부)

```markdown
> ⛔ **분석 거부**
> 본 스킬은 **만 12세 이상 성인**을 대상으로 합니다. 영유아·아동·반려동물 보충제는 전문의·수의사의 처방이 필요하며, 본 도구로 분석하지 않습니다.

> ⛔ **Analysis Declined**
> This skill is intended for **persons aged 12 and above**. Supplements for infants, children, or pets require professional (pediatrician/veterinarian) guidance and are outside the scope of this tool.
```

### 3-3. 처방약 복용 입력 시

```markdown
> 🔴 **처방약 상호작용 주의**
> 복용 중인 처방약이 입력되었습니다. 본 리포트는 보충제↔약물 상호작용을 **경고 수준**으로만 제공하며, **처방약의 용량 조정·중단은 반드시 담당 의사와 상담**하세요.

> 🔴 **Prescription Drug Interaction Notice**
> A prescription medication was entered. This report provides supplement-drug interaction alerts at a **caution level only**. **Any adjustment or discontinuation of prescription medication must be discussed with your prescribing physician.**
```

### 3-4. UL 초과 성분 감지 시

```markdown
> 🔴 **상한섭취량(UL) 초과 경고**
> 분석 결과 [성분명]이(가) 상한섭취량을 초과했습니다. 장기 복용 시 부작용 위험이 있습니다. **제품 감량·교체·중단을 검토**하고 담당 의료진과 상담하세요.

> 🔴 **Upper Intake Level (UL) Exceeded**
> Analysis shows [ingredient] exceeds the Tolerable Upper Intake Level. Long-term use carries risk of adverse effects. **Consider reducing, replacing, or discontinuing** the product and consult your healthcare provider.
```

---

## 4. 삽입 규칙 (report-template.md 구현 시)

1. 리포트 말미 `{{DISCLAIMER}}` 마커는 **한국어 섹션 전체(§1) 삽입**. `--en` 옵션 시 영어 섹션(§2) 삽입.
2. 리포트 상단 `{{WARNING_BANNERS}}` 마커는 조건부 배너(§3-1 ~ §3-4)를 순서대로 삽입:
   - 우선순위: 영유아·반려동물 감지(3-2) > 임신·수유(3-1) > 처방약(3-3) > UL 초과(3-4)
   - 영유아·반려동물 감지 시 **분석 자체 거부** — §3-2 배너만 출력하고 나머지 리포트 생략
3. `--en` 요청 시 한국어·영어 **둘 다 출력 가능**. 기본은 한국어만.

---

## 5. 금지 사항

1. 면책 조항 **축약·생략 금지** — 문구 전체 유지
2. 면책 조항에 프로모션·연락처·외부 링크 **삽입 금지** (운영자 정보는 README·스킬 메타데이터로)
3. 경고 배너를 **리포트 상단이 아닌 곳에 숨기지 말 것** — TL;DR 위 최상단 필수
4. "본 리포트를 참고만 하세요" 식의 **약한 문구로 대체 금지** — 원문 그대로

---

## 6. 변경 이력

- **v1.0 (초안)**: 한/영 면책 + 4종 조건부 경고 배너.
