# CHANGELOG — vitamin-analyzer

모든 의미 있는 변경은 이 파일에 기록합니다. 형식은 [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)을 따릅니다.

---

## [1.0.1] — 2026-04-17

실사용 테스트(5제품 스택: LiverRelax + Calci-Mag-D + 비타민D3 + Multi + 오메가3) 피드백 반영 patch 릴리즈.

### Added

- **`references/deficiency-excess-effects.md`** (신규, ~700줄) — 28영양소 각각에 대한 부족·초과 건강 영향 카탈로그
  - 각 영양소 섹션: N.1 부족 시 (초기/중기/장기 증상·고위험군·출처) + N.2 초과 시 (단기/장기 영향·위험 임계값·약물 상호작용·출처)
  - §29 리포트 인용 가이드 + §30 버전 관리
- **`SKILL.md` Phase 1 — 제품 완전성 재확인 루프** (§2-1-A): 빠지기 쉬운 8개 카테고리 체크리스트로 사용자 입력 보완
- **`SKILL.md` Phase 1 — 제품명 5단계 Fallback Chain** (§4-1-3): 공식 브랜드몰 → 쿠팡/11번가 → 식약처 건기식 DB → iHerb → 사용자 재질문
- **`SKILL.md` Phase 6 — 자동 .md 저장**: `./reports/YYYY-MM-DD_HHMM_analysis.md` (PDF 생략)
- **`prompts/analyze-ingredient.md` Step 4.5 — RDA 미달 분석** (§5-A): `deficiency_gap[]` 배열 신설
  - 28영양소 전수 검토, <50% 현저 부족 / 50-70% 경미 부족 / 70-90% 경계선(증상 매칭 시만) 3단계
  - `triggered_by_symptom` 플래그로 수면·피로·면역·뼈·피부·소화 증상 기반 가중
  - "식단 포함 시 달라질 수 있음" 주의 자동 삽입 지시
- **`prompts/analyze-ingredient.md` Step 8 — 건강 영향 매핑** (§8-A): `health_impact_ref` 객체 신설
  - `catalog_section` / `direction` / `summary_short` (80자 이내) / `severity_tier` / `threshold_note` / `citation_url`
  - UL 초과·근접 + RDA 미달 항목에 카탈로그 연결
- **`prompts/report-template.md` 마커 3종 신설**:
  - `{{UL_EXCEEDED_SECTION}}` — UL 초과/근접 상세 (단기/장기/위험 임계값/고위험군 불릿)
  - `{{DEFICIENCY_SECTION}}` — RDA 미달 상세 (증상 연관 + 건강 영향 + 식단 보완 + 보충제 형태 옵션)
  - `{{FILE_SAVED_PATH}}` — 자동 저장 경로
- **`references/search-templates.md` §9-A — 네이버 브랜드몰 차단 대응 박스**: WebFetch/Chrome MCP/markitdown 차단 시 대체 경로
- **`references/search-templates.md` §10-2A — 제품명 5단계 Fallback 파이프라인**

### Changed

- **리포트 섹션 순서 재정렬**: 경고배너 → TL;DR → 제품 요약 → 성분 분석 → 기여도/중복 → **UL 상세** → **부족 상세** → 상호작용 → 스택 최적화 → 대체 제품 → 미매칭 → 저장 경로 → 면책
- **`SKILL.md`** frontmatter version `1.0.0` → `1.0.1`
- **Phase 2 정규화 단계 입력**이 Phase 1 재확인 루프를 **반드시 거친** 후여야 함 (루프 생략 금지 규칙 추가)
- **"PDF 제공 여부"** 질문 삭제 (사용자 요청 반영) → .md 자동 저장만 남김

### Fixed

- 초판에서 UL 초과는 잘 잡지만 **RDA 미달(부족) 분석이 누락**되던 문제 → deficiency_gap[] + DEFICIENCY_SECTION으로 해결
- **제품명만 주어졌을 때** 공식몰 "상품상세페이지 참조" 메시지에 막혀 분석이 중단되던 문제 → 5단계 Fallback으로 해결
- **네이버 brand.naver.com** WebFetch 차단 시 무한 재시도하던 문제 → §9-A 차단 인지 + 즉시 대체 경로 이동

### Known Limitations (carried over + new)

- **`deficiency-excess-effects.md`의 citation URL**: v1.0.1에서는 NIH ODS factsheet URL 기반. PubMed 원저 PMID는 v1.1에서 보강 예정
- **식약처 건기식 DB 파싱**: 검색 페이지는 JS 기반이라 WebFetch 직접 파싱 불가 가능 — 검색 쿼리만 유도하고 실제 결과는 Claude 응답에 의존
- **자동 .md 저장**: Skill 실행 환경에 Write 도구 권한이 있어야 함. 권한 없는 환경(일부 claude.ai 배포)에서는 저장 실패 → 본문만 노출

---

## [1.0.0] — 2026-04-17

최초 공개 릴리즈.

### Added

#### 스킬 구조
- `SKILL.md` (428줄) — Phase 1~6 파이프라인 조율, PRD 프로필 질문 6종, 거부 프로토콜, 사용 예시 6종
- `README.md` — 설치·사용법·데이터 소스 정책·범위·제한사항
- YAML 프론트매터 (`name: vitamin-analyzer`, `version: 1.0.0`, MIT)

#### References (1,688줄)
- `references/ingredient-aliases.md` (239줄) — 상위 **100종** 성분 한/영/라틴명 동의어 + 카테고리 7분류 (비타민 13 + 미네랄 15 + 아미노산 10 + 오메가3 4 + 식물추출물 20 + 프로바이오틱스 10 + 기타 28) + IU↔mcg 변환식 9종 + 6단계 매칭 알고리즘
- `references/rda-table.md` (620줄) — **28영양소 × (NIH + KDRI 2020)** × (성별·연령·임신·수유) RDA/AI/UL, UL 치명 8항목 체크리스트, 출처 URL 매핑
- `references/search-templates.md` (423줄) — 9개 소스 WebSearch 쿼리 템플릿 + Fallback 매트릭스 + 3종 파이프라인 (성분별·대체제품·스택 최적화)
- `references/interaction-patterns.md` (286줄) — 20종 상호작용 (보충제↔처방약 12, 보충제↔보충제 5, 보충제↔음식 3), 심각도 🔴/🟠/🟡 3단계
- `references/disclaimer.md` (120줄) — 한/영 면책 본문 + 4종 조건부 배너 (임신·영유아·처방약·UL 초과)

#### Prompts (1,148줄)
- `prompts/parse-image.md` (317줄) — 이미지 → JSON 구조화, confidence 3단계, 복수 제품 `products[]`, 프롬프트 인젝션 방어 5패턴
- `prompts/analyze-ingredient.md` (442줄) — **7단계 파이프라인**: 정규화 → 단위 표준화 → 스택 합산 → RDA/UL → 중복 검출 → 상호작용 → 근거 URL. `stack_optimization` 3그룹(remove/reduce/add) 생성
- `prompts/report-template.md` (389줄) — 12종 템플릿 마커, 복수 제품 기여도 표, 중복 검출 섹션, 스택 최적화 3그룹, 국내 5 + 해외 3 대체제품 표, 거부 리포트 별도 경로

#### Examples (887줄, 전부 가상 브랜드)
- `examples/liver-relax-sample.md` — 단일 제품, NutriCalm Labs LiverRelax Complex (실리마린 130 + L-테아닌 200 + B군), 32세 여성 페르소나, CYP3A4 정보성 안내 테스트
- `examples/multi-vitamin-sample.md` — 단일 제품, DailyCore Women's Complete 25성분 종합비타민, 45세 갱년기 이행기 여성, IU/mcg/mg NE/mcg DFE 단위 혼용 변환 테스트
- `examples/stack-optimize-sample.md` — 3제품 스택, Vit D UL 1.5배 초과 🔴 + B6 UL 78% 🟠 + Mg 부족 🟡 혼재, stack_optimization 3그룹 전부 활성화 시나리오

### 핵심 동작 규칙

- **3-Tier 데이터 소스**: Tier 1 (NIH ODS/PubMed/Cochrane) → Tier 2 (Examine/식약처/Drugs.com/NCCIH) → Tier 3 (iHerb/쿠팡/네이버)
- **모든 의학 주장에 `[출처: Tier N / URL]` 첨부** 강제
- **쿠팡·네이버 리뷰 수집·인용 금지** — 가격만 허용. iHerb 리뷰만 별점+개수+요약 허용
- **면책 조항 자동 삽입** — 매 리포트 말미 `references/disclaimer.md` §1/§2 원문 그대로
- **처방약 복용량·중단 권고 금지** — 상호작용 경고 수준에서 정지, "의사·약사 상담 권고"로 마무리
- **영유아·반려동물 분석 거부** — 감지 시 거부 리포트만 출력
- **임신·수유** — 분석 진행하되 최상단 경고 배너 + 산부인과 상담 권고
- **UL 초과 시 리포트 최상단 🔴 경고 배너** 필수
- **Claude WebSearch 결과에 없는 URL을 fabricate 금지** — 프롬프트에서 강제

### 언어 옵션

- 한국어 기본 / `--en` 영어 / `--both` 병기
- 면책 조항은 언어별로 `references/disclaimer.md` §1(한) / §2(영) 교체

### 검증

- 개인정보 grep 0건 (배포 전 자동 검증)
- SKILL.md 800줄 이하 (428줄)
- RDA/UL 8개 치명 항목 체크리스트 (Vit D/B6, 니아신, 철, 엽산, 셀레늄, 마그네슘, 비타민 A)
- 3가지 examples로 regression 테스트 (단일 / 단일 25성분 / 3제품 스택)

---

## Known Limitations (v1.0.0)

### Data

- **RDA 표 숫자는 Claude 훈련 데이터 기반 초안**. 일부 URL은 `⚠️검증` 플래그 (예: KDRI 공식 PDF URL, NIH Sodium factsheet URL 패턴). 배포 환경에서 WebFetch로 실제 접근 확인 후 v1.1에서 확정 예정.
- **상호작용 20종의 PubMed PMID 미포함**. 현재 검색 엔드포인트·NCCIH·Drugs.com 요약 URL 수준. v1.1에서 각 항목별 원저 PMID 보강 예정.
- **희귀 성분 (100종 외) 미매칭**. 사용자가 직접 이름 입력해도 `unrecognized_ingredients[]`로 분리되어 분석 제외.

### Scope

- **19세 이상 성인만 지원**. 영유아·아동 DRI는 본 버전에 미포함 (추후 확장 고려).
- **국가별 가격·배송은 한국·iHerb 해외 직구 기준**. 다른 국가 거주자는 가격 참고만.
- **리얼타임 가격 동기화 없음** — WebSearch 결과 시점 기준.
- **PDF·이미지 리포트 출력 미지원** — 마크다운만.

### Known Edge Cases

- **라벨 해상도 낮은 이미지** — `parse-image.md`의 confidence `medium/low` 판정 시 사용자 재확인 요청. 자동 OCR 보정은 없음.
- **복합 원료 (혼합물 브랜드명)** — 실제 성분 함량이 표기되지 않은 "프로프라이어터리 블렌드"는 분석 제외, `unrecognized` 처리.
- **같은 성분의 다른 형태 (예: Mg Oxide vs Glycinate)** — 흡수율 차이는 보고서에 코멘트로 표기하나 UL/RDA 계산은 elemental mg 기준만 (형태별 가중치 계산은 v1.1+).
- **IU 표기의 형태 의존성** — Vit E의 d-alpha vs dl-alpha, Vit A의 retinol vs beta-carotene은 변환 계수가 다름. 현재 라벨에 명시된 형태를 `parse-image.md`가 추출하면 `analyze-ingredient.md`가 해당 변환식 적용. 형태 미기재 시 보수적(preformed retinol/natural d-alpha)으로 계산 + 불확실성 코멘트.

---

## 향후 계획 (로드맵 참고)

### v1.1 (예정)
- KDRI 공식 PDF URL 확정 (현재 `⚠️검증`)
- Sodium NIH factsheet URL 패턴 수정
- 상호작용 20종 각 항목에 PubMed PMID 추가
- `deficiency-excess-effects.md` 각 항목에 PubMed 원저 PMID 보강 (현재 NIH ODS factsheet 기반)
- 성분 aliases 추가 30종 (총 130종 목표)
- 영유아·아동 DRI 별도 모듈 (거부 대신 전문의 상담 링크 강화)

### v1.2+ (검토)
- 형태별 흡수율 가중치 (Mg Oxide vs Glycinate 등)
- 사용자 기록 세션 간 유지 (Claude Memory 도구 연동)
- PDF 리포트 출력 옵션
- 다국어 (일본어·영어 확장·중국어)

---

## 문의·이슈

- 의학 주장 오류 / URL 깨짐 / RDA 수치 검증 요청 → Issue로 제보 (Tier 1 출처 링크 동봉 권장)
- 본 Skill은 **의학 조언이 아님**. 개인 건강 결정은 담당 의료진 상담 필수.
