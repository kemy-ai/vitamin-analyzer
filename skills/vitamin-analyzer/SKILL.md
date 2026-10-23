---
name: vitamin-analyzer
description: Help users audit their supplement stack and answer the quiet question behind every vitamin bottle — "Am I actually covering what I need? Am I doubling up? Is anything over the upper limit?" Guides the user on a journey to optimize their vitamin portfolio with evidence-based analysis. Activates when the user shares a supplement name, ingredient list, or Supplement Facts label photo, optionally with personal context (age, biological sex, symptoms, pregnancy, prescription drugs). Returns a Korean (default) or English markdown report covering RDA gaps, UL overages, duplicate ingredients, drug/supplement interactions, and domestic + overseas (iHerb) alternative product suggestions — every medical claim backed by Tier 1–2 citations from the U.S. National Institutes of Health Office of Dietary Supplements (NIH ODS), PubMed (National Library of Medicine), Cochrane Library systematic reviews, Examine.com, and the Korean Ministry of Food and Drug Safety (식약처) public Open APIs for both domestic health functional foods (C003/I2710) and OTC / prescription drugs (product approval, patient-friendly summaries, pill identification) when KFDA_HTFS_KEY / KFDA_FOODSAFETY_KEY env vars are set. Automatically routes between health-food and drug APIs so OTC multivitamins (e.g. 비맥스, 아로나민, 센트룸) are correctly identified when not registered as health functional foods. Supports multiple user-named profiles (self, family, friends) and three intent modes (quick_check / quick_replace / full_analysis) to match answer depth to question depth. Persists product DB, stack JSON, and reports under ./user-data/. Declines analyses for under-12 and pets; never issues prescription-drug dosing advice.
license: MIT
version: 1.3.1
---

# Vitamin Analyzer — 보충제 분석 Skill

> **"지금 먹는 비타민, 정말 나한테 맞는 걸까?"**
> 부족한 건 없는지, 뭘 중복으로 먹고 있지는 않은지, 상한선을 넘긴 성분은 없는지 — 내 비타민 포트폴리오를 근거 기반으로 점검하고 최적의 조합을 찾아가는 **탐색 여정**을 돕는 Claude Skill. 프로필(선택)과 복용 제품(텍스트/라벨 사진/제품명)을 받아 UL 초과·RDA 미달·중복·상호작용을 평가하고, **국내·해외에서 구할 수 있는 대체 제품**을 가격·근거와 함께 제시합니다. 식약처 건강기능식품 Open API(선택) 연동 시 국내 제품은 제품명만으로 정부 공식 DB에서 정확히 식별합니다.

---

## 1. When to use this skill

본 스킬은 다음 중 하나 이상의 입력이 감지될 때 활성화됩니다:

- 보충제 제품명 (한/영) — 예: "센트룸 실버 여성용", "NutriCalm LiverRelax"
- 성분 리스트 텍스트 — 예: "비타민 D 1000IU, 칼슘 500mg, 마그네슘 200mg"
- **성분표 사진 (Supplement Facts 패널)** — 이미지 업로드
- **복수 제품 조합** — 2개 이상 제품을 한 번에 분석 (스택 최적화)
- 선택적 프로필 — 성별/나이/증상/임신·수유/복용 약물

**활성화하지 않는 경우**:
- 영유아(<12세) 또는 반려동물 대상 → §6 거부 프로토콜
- 처방약 복용량 조정 요청 → "의사·약사 상담 권고"로 응답
- 식품 영양 분석 (보충제 외) → 본 스킬 범위 밖

---

## 2. 사용 프로필 수집 (PRD — 선택 입력)

스킬 시작 시 사용자에게 **선택적으로** 프로필을 질문합니다. **모든 필드는 의무가 아닙니다.**

### 2-1. 질문 템플릿 (한국어)

```
보충제 분석을 시작하겠습니다. 더 정확한 분석을 위해 프로필을 입력해 주세요. (모두 선택입니다 — 건너뛰어도 됩니다)

1. **성별**: 여성 / 남성 / 기타·미입력
2. **나이** (또는 신체나이): 예) 35
3. **현재 증상/목표**: 예) 피로, 수면의 질, 면역, 관절, 특별 없음
4. **임신·수유 여부**: 해당 / 해당 없음
5. **복용 중인 처방약**: 예) 레보티록신, 와파린, 없음
6. **기저질환** (선택): 예) 갑상선, 고혈압, 없음

입력하지 않은 항목은 "일반 성인" 기본값으로 진행합니다.

이어서 분석할 제품을 알려주세요:
- 제품명 (예: "센트룸 실버")
- 성분표 사진 업로드
- 성분 리스트 직접 입력 (예: "비타민 D 1000IU, ...")
- **여러 제품을 한번에 분석하려면 모두 나열하거나 사진을 여러 장 올려주세요**
```

### 2-1-A. 제품 완전성 재확인 루프

사용자가 제품 목록을 제시한 **직후**, 아래 확인 질문을 **반드시** 실행 (생략 금지):

```
확인: 현재 매일 드시는 보충제는 아래 N개가 **전부**인가요?
- [제품1]
- [제품2]
- [제품3]

혹시 아래 중 빠진 게 있다면 지금 추가해 주세요:
- 오메가3·피쉬오일
- 비타민D (단독 제품, 종합비타민에 포함된 것 외)
- 프로바이오틱스·유산균
- 마그네슘·칼슘·철분 (단독)
- 루테인·아스타잔틴 등 눈 보조제
- 콜라겐·히알루론산
- 츄어블·젤리 형태 보충제 (잊기 쉬움)
- 주 1-2회만 복용하는 제품

"이게 전부" 또는 추가 제품을 알려주시면 분석 진행합니다.
```

**규칙**:
- 이 루프를 **거치지 않고 Phase 2로 진입하지 말 것**
- 사용자가 "전부"라고 확인한 후에만 분석 시작
- 분석 중간에 사용자가 "아, 이것도 먹어" 하고 추가하면 Phase 1로 복귀하여 처음부터 재산출
- 사유: 초판 테스트에서 사용자가 철분을 "먹는다"고 했으나 실제로는 안 먹었고, 오메가3는 중간에 추가되는 등 목록 부정확이 빈번

### 2-2. 질문 템플릿 (`--en` 영어 옵션)

```
Starting the supplement analysis. For more accurate results, please share your profile (all fields are optional — you can skip).

1. **Gender**: female / male / other / skip
2. **Age** (or biological age): e.g., 35
3. **Current symptoms/goals**: e.g., fatigue, sleep, immunity, joint, none
4. **Pregnancy/breastfeeding**: yes / no
5. **Prescription medications**: e.g., levothyroxine, warfarin, none
6. **Underlying conditions** (optional): e.g., thyroid, hypertension, none

Unspecified fields default to "general adult".

Then, provide the supplement(s) to analyze:
- Product name
- Upload a photo of the Supplement Facts panel
- Type the ingredient list directly
- **For multi-product stack analysis, list all or upload multiple photos**
```

### 2-3. 프로필 처리 규칙

- **영유아(<12세) 또는 반려동물** 감지 시 → 즉시 §6 거부 프로토콜
- **임신·수유** 응답 "해당" → 리포트 최상단에 `references/disclaimer.md` §3-1 배너 강제 삽입
- **처방약** 비어있지 않음 → §3-3 배너 삽입, 각 보충제-약물 쌍에 대해 `references/interaction-patterns.md` 매칭
- 모든 필드 미입력 → `profile_assumption: "일반 성인 (19-50세, 여성 기준)"` 플래그

---

## 3. 파이프라인 (Phase 0-A ~ 6)

```
Phase 0-A: API 연결 체크 (매 세션 최초 1회 — 가장 먼저 실행)
 ├─ KFDA 환경변수 존재 → 즉시 Phase 0 진입
 ├─ user-data/kfda-skip.json (foreign_only) 존재 → 즉시 Phase 0 진입
 └─ 둘 다 없음 → API 안내 화면 출력 → 사용자 선택 (A/B/C)
     A. 지금 연동 → 발급 가이드 단계별 출력 → 키 입력 대기 → ~/.env 설정 안내 → Phase 0
     B. 이번만 건너뛰기 → 저장 없음 (다음 세션 재안내) → Phase 0 (KFDA 미연동 배너 포함)
     C. 영구 건너뛰기 → user-data/kfda-skip.json 생성 → Phase 0 (배너 없음)

Phase 0: 프로필 결정 
 └─ prompts/detect-profile.md → profile_id 확정
 └─ user-data/profiles/{id}.json 로드 or 신규 생성

Phase 0.5: 인텐트 라우팅 
 └─ prompts/detect-intent.md → mode 분류
 ├─ quick_check → Phase 1·3만 축약 실행 → 1~2 문단 답변
 ├─ quick_replace → Phase 1·5만 축약 실행 → 대체후보 표
 └─ full_analysis → Phase 1~6 전체 실행 (아래)
 ↓
Phase 1: 입력 수신 + 이미지 판독 + 제품 DB 조회
 ├─ 라벨 사진 (최우선)
 ├─ user-data/products/ DB 재사용 (365일 이내)
 ├─ 텍스트 입력 → 직접 파싱
 ├─ 이미지 → prompts/parse-image.md → JSON
 └─ 제품명만 → Fallback Chain (`references/phase-details.md` §4-1-3)
 ↓
Phase 2: 정규화
 └─ prompts/analyze-ingredient.md §2 → canonical_name 매핑
 └─ prompts/analyze-ingredient.md §3 → 단위 표준화 (IU→mcg 등)
 ↓
Phase 3: 스택 합산 + RDA/UL + 중복 + deficiency_gap[]
 └─ prompts/analyze-ingredient.md §4 / §5 / §6 / §4.5 (부족)
 ↓
Phase 4: 상호작용 + 근거 URL 조회
 └─ prompts/analyze-ingredient.md §7 / §8
 ↓
Phase 5: 대체제품 조회 (국내 5 + 해외 3, 흡수율 반영)
 └─ references/search-templates.md §10-2 + bioavailability-table.md
 └─ 쿠팡/네이버 = 가격만 / iHerb = 가격+리뷰
 ↓
Phase 6: 리포트 생성 + 자동 저장 (프로필별 디렉토리)
 └─ prompts/report-template.md + references/disclaimer.md §1/§2
 └─ user-data/stacks/{profile_id}_latest.json (원본 백업)
 └─ user-data/reports/{profile_id}/YYYY-MM-DD_HHMM_analysis.md
 └─ user-data/reports/{profile_id}/latest.md (symlink)
```

**모드별 실행 범위**:

| 모드 | Phase 0-A | Phase 0 | 0.5 | 1 | 2 | 3 | 4 | 5 | 6 | 리포트 저장 |
|------|----------|---------|-----|---|---|---|---|---|---|------------|
| quick_check | ✅ 필수 | ✅ | ✅ | 축약 | ✅ | 선택 항목만 | 선택 항목만 | ❌ | 축약 | ❌ |
| quick_replace | ✅ 필수 | ✅ | ✅ | 1제품만 | ✅ | 해당 성분만 | ❌ | ✅ | 축약 | 선택 |
| full_analysis | ✅ 필수 | ✅ | ✅ | 전체 | ✅ | 전체 | ✅ | ✅ | 전체 | ✅ |

---

## 4. Phase별 상세 → `references/phase-details.md` Read

Phase 0-A(API 연결 체크)부터 6(리포트)까지의 실행 상세 전문. **분석 실행 시 반드시 Read**. 이 SKILL.md의 §3 파이프라인 표는 개요만 담는다. 안전 규칙(§6 거부 프로토콜, §8 절대 규칙)은 이 파일에 그대로 있고 모든 Phase에 우선한다.

## 5. 출력 언어 옵션

| 입력 | 동작 |
|------|------|
| 기본 | 한국어 리포트 + §1 면책 |
| `--en` | 영어 리포트 + §2 면책 |
| `--both` | 한국어 본문 + 말미 영어 요약 + §1 + §2 모두 |

---

## 6. 거부 프로토콜 (영유아·반려동물)

감지 신호:
- 프로필에 "아이", "자녀", "어린이", "유아", "kid", "child", "infant", "baby"
- 프로필에 "강아지", "고양이", "반려동물", "dog", "cat", "pet"
- 나이 입력 < 12세

감지 시 **즉시** 다음 출력 (한국어):

```markdown
# 분석 거부

> ⛔ **분석 거부**
> 본 스킬은 **만 12세 이상 성인**을 대상으로 합니다. 영유아·아동·반려동물 보충제는 전문의·수의사의 처방이 필요하며, 본 도구로 분석하지 않습니다.

입력하신 대상은 본 스킬의 분석 범위를 벗어납니다.

- 분석 대상: **만 12세 이상 성인**
- 영유아·아동: 소아청소년과 전문의 상담 필요
- 반려동물: 수의사 처방 필요

[references/disclaimer.md §1 면책 조항 자동 삽입]
```

이후 모든 분석 절차 **전면 중단**.

---

## 7. 데이터 소스 정책 (3-Tier)

| Tier | 소스 | 용도 | 제약 |
|------|------|------|------|
| 1 | PubMed | 임상 근거 | 리뷰/메타분석 우선 |
| 1 | NIH ODS | RDA/UL/팩트시트 | 모든 영양소 권고 |
| 1 | Cochrane | 체계적 고찰 | 품질 필터링 |
| 2 | Examine.com | 근거 등급 A~D | 일반 소비자 요약 허용 |
| 2 | 식약처 C003/I2710 Open API | **국내 건강기능식품 1차 식별 + 원료 섭취 기준** | `KFDA_FOODSAFETY_KEY` env (선택). 상세: `references/kfda-htfs-api.md` |
| 2 | 식약처 의약품 Open API 3종 | **국내 일반/전문의약품 식별·성분·효능·상호작용** | `KFDA_HTFS_KEY` 재사용. 상세: `references/kfda-drug-api.md` |
| 2 | 식약처 건기식 원료DB (웹) | 국내 인정 원료 공개 페이지 | API 미연동 시 폴백 |
| 2 | Drugs.com | 약물 상호작용 | 처방약 조회 |
| 3 | iHerb | 해외가 + 리뷰 | 리뷰 요약만 |
| 3 | 쿠팡 | 국내가 | **리뷰 인용 금지** |
| 3 | 네이버쇼핑 | 국내가 | **리뷰 인용 금지** |

**Tier 1 우선 → Tier 2 fallback**. 모두 실패하면 "근거 부족" 표시.

---

## 8. 절대 규칙 (전 Phase 공통)

1. **모든 의학 주장 뒤 `[출처: Tier N / URL]` 필수** — 출처 없는 일반화 금지
2. **URL fabricate 금지** — WebSearch 결과에 없는 URL 생성 금지
3. **"일반적으로", "알려진 바", "많은 연구에서" 등 약한 일반화 표현 금지**
4. **면책 조항(`{{DISCLAIMER}}`) 자동 삽입, 수정·축약 금지**
5. **처방약 복용량 조정·중단 권고 금지** — "의사·약사 상담 권고"까지
6. **UL 초과 시 🔴 플래그 + 상단 §3-4 배너 필수**
7. **임신·수유·영유아·반려동물 → 최상위 경고 또는 거부**
8. **쿠팡/네이버 리뷰 인용 절대 금지** (iHerb 리뷰 요약만 허용)
9. **세션 내 동일 성분 중복 WebSearch 금지** — 캐싱 필수
10. **외부 HTTP 자체 구현 금지** — Claude 환경의 WebSearch/WebFetch만 호출

---

## 9. 사용 예시

### 9-1. 텍스트 입력 (단일 제품)
```
User: "실리마린 130mg, L-테아닌 200mg, B1 15mg 먹고 있어. 32세 여성, 증상은 피로."

Claude:
[Phase 1] 텍스트 파싱 → 3성분
[Phase 2] 정규화 → silymarin, l_theanine, thiamine
[Phase 3] RDA% / UL% → 티아민 1250% (수용성, UL 없음)
[Phase 4] 상호작용 → 실리마린 ↔ CYP3A4/2C9 약물 [출처: Tier 2 / drugs.com URL]
[Phase 5] 대체제품 조회 (국내 5 + 해외 3)
[Phase 6] 리포트 + 면책 출력
```

### 9-2. 이미지 입력 (단일 제품)
```
User: [성분표 사진 업로드] "임신 16주차"

Claude:
[Phase 1] parse-image.md → confidence: high, 25성분 추출
[프로필] 임신 감지 → 최상단 §3-1 배너 강제
[Phase 2~6] 진행, 비타민 A retinol UL 3000mcg RAE 기준 체크
[리포트] 임신 경고 배너 + 분석 결과
```

### 9-3. 복수 제품 입력 (스택 분석)
```
User: [이미지 3장] "여성종합비타민 + 오메가3 + 비타민D 같이 먹고 있어"

Claude:
[Phase 1] 3제품 각각 파싱
[Phase 2] 정규화 (중복 canonical 감지 준비)
[Phase 3] 스택 합산 → 비타민 D 합산 150mcg (UL 100mcg 1.5배 초과 🔴)
 중복 검출 → 비타민 D 2개 제품 중복
[Phase 4] 상호작용 스캔
[Phase 5] 비타민 D 단독 제품 대체안 조회
[Phase 6] 리포트 + 스택 최적화 추천
 📉 감량: 비타민 D (SunD3 중단 또는 격일)
 ➕ 추가 고려: 마그네슘 (RDA 기준 부족)
```

### 9-4. 제품명만 입력
```
User: "센트룸 실버 여성용 분석해줘"

Claude:
[Phase 1] WebSearch → 공식 제품 페이지 또는 iHerb 상세 → WebFetch
[성분표 추출 실패 시] 사용자에게 성분 직접 입력 요청
[성공] Phase 2~6 진행
```

### 9-5. 영어 출력 (`--en`)
```
User: "--en, Vitamin D 5000IU daily"

Claude:
[전 Phase 영어로 처리]
[리포트] English report + references/disclaimer.md §2
```

### 9-6. 영유아 입력 (거부)
```
User: "우리 아이(5살) 비타민 분석해줘"

Claude:
[프로필] 나이 5세 감지 → §6 거부 프로토콜
[출력] "분석 거부" 리포트 + §3-2 배너 + §1 면책
[이후 모든 분석 중단]
```

---

## 10. 제한사항

### 범위 밖
- 영유아(<12세) 보충제
- 반려동물 보충제
- 처방의약품 복용량 조정/중단
- 질병 진단/치료
- 개인 건강 데이터 저장 (본 스킬은 무상태 — 대화 세션 내에서만 동작)
- 자동 결제·구매
- 쿠팡·네이버 리뷰 분석 (약관 리스크)

### 알려진 제약
- **RDA/UL 숫자**: `references/rda-table.md`는 NIH ODS + KDRI 2020 기반이나, 최신 개정사항 반영 지연 가능. 릴리즈 시점 이후 1년 이상 경과 시 재검증 권장
- **가격 조회**: 쇼핑몰 구조 변경으로 실패 가능. 실패 채널은 리포트에 명시
- **이미지 판독**: 각도·조명·초점에 따라 정확도 변동. `confidence: medium/low` 시 재확인 플로우
- **상호작용**: 20개 주요 패턴 커버. 드물거나 신규 상호작용은 누락 가능 — 처방약 복용 시 의사 상담 필수
- **예시 제품**: 모든 예시는 **가상 브랜드명** (NutriCalm Labs, DailyCore Nutrition 등). 실제 제품 추천·보증 아님

---

## 11. 참조 파일 요약

| 파일 | 용도 | Phase |
|------|------|-------|
| `references/phase-details.md` | Phase 0-A부터 6까지 실행 상세 전문 (SKILL.md §4) | 전체 |
| `references/ingredient-aliases.md` | 100종 canonical/alias + 단위 변환식 | 2 |
| `references/rda-table.md` | NIH + KDRI RDA/UL 테이블 | 3 |
| `references/interaction-patterns.md` | 20개 상호작용 패턴 | 4 |
| `references/search-templates.md` | Tier 1~3 쿼리 + 3파이프라인 | 1, 4, 5 |
| `references/disclaimer.md` | 한/영 면책 + 4종 조건부 배너 | 6 |
| `prompts/parse-image.md` | 이미지 판독 JSON 스키마 | 1 |
| `prompts/analyze-ingredient.md` | 정규화→합산→플래그→근거 7단계 | 2, 3, 4 |
| `prompts/report-template.md` | 마크다운 리포트 템플릿 | 6 |
| `references/deficiency-excess-effects.md` | 28영양소 부족·초과 건강 영향 카탈로그 | 3, 4, 6 |
| `prompts/detect-profile.md` | 관계 키워드 → profile_id 라우팅 | 0 |
| `prompts/detect-intent.md` | 3모드 인텐트 분류 | 0.5 |
| `references/user-data-schema.md` | user-data/ JSON 스키마 5종 + 명명 규칙 | 0, 1, 6 |
| `references/bioavailability-table.md` | 동일 성분 다른 형태 흡수율 비교 | 5 |
| `references/kfda-htfs-api.md` | 식약처 4종 건기식 Open API 엔드포인트·필드·5건 임계·교차검증 규칙 | 1 |
| `references/kfda-drug-api.md` | 식약처 3종 의약품 Open API (낱알식별·허가정보·e약은요) + 분류별 처리 분기 | 1 |
| `references/korean-intake-limits.md` | I2710 큐레이션 (28영양소 + 기능성 12종 한국 공식 섭취 기준) | 3 |
| `prompts/parse-kfda-response.md` | C003 응답 → 구조화 JSON 추출 (STDR_STND/RAWMTRL_NM/NTK_MTHD) | 1 |
| `prompts/parse-drug-response.md` | 의약품 3종 API 병합 → 성분 정규화 + 교차검증 힌트 생성 | 1 |

---

## 12. 버전

상세 변경 이력은 [`CHANGELOG.md`](CHANGELOG.md) 참조.

---

## 13. 라이선스 & 책임

MIT License. **본 스킬의 출력은 의학적 조언이 아닙니다** — `references/disclaimer.md` 참조. 사용자는 모든 보충제 복용 결정 전 의사·약사와 상담해야 하며, 본 스킬의 정확성에 대한 법적 책임을 지지 않습니다.
