# CHANGELOG — vitamin-analyzer

모든 의미 있는 변경은 이 파일에 기록합니다. 형식은 [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)을 따릅니다.

---

## [1.1.3] — 2026-04-17

최초 공개 릴리즈.

### 핵심 기능

- **성분 정규화**: 100종 한/영/라틴 동의어 + 단위 변환식 (IU↔mcg, mcg NE/DFE/RAE 등)
- **RDA/UL 평가**: 28영양소 기준 (NIH ODS 기본 + 식약처 KDRI 병기). `ul_basis: "total" | "supplement_only"` 구분
- **생체이용률 테이블**: 성분 형태별 흡수율(산화·글리시네이트·메틸폴레이트 등)로 "총량은 초과이나 실제 흡수는 부족" 케이스 감지
- **복수 제품 스택**: 중복·합산·UL 초과 평가 + `stack_optimization` (remove/reduce/add 3그룹)
- **상호작용 검출**: 20종 (약물 12 + 보충제 5 + 음식 3)
- **3-Tier 데이터 소스**: Tier 1 (NIH/PubMed/Cochrane) · Tier 2 (Examine/KFDA/Drugs.com) · Tier 3 (iHerb/쿠팡 — 가격만)
- **5단계 Fallback Chain**: 라벨 사진 → 제품 DB → Perplexity → WebSearch → 공식몰/쇼핑몰
- **인텐트 라우터 (3모드)**: `quick_check` / `quick_replace` / `full_analysis` — 질문 깊이와 답변 깊이 매칭
- **다중 프로필**: 사용자가 프로필 이름 지정 (본인·가족·친구). 제품 DB는 프로필 간 공유
- **RDA 미달 분석**: 28영양소 전수 검토, 현저 부족 / 경미 부족 / 경계선 3단계 (`deficiency_gap[]`)
- **대체 제품 제안**: 국내 5 + 해외 3 (iHerb 리뷰 요약 포함)
- **근거 투명성**: 모든 의학적 주장에 `[출처: Tier N / URL]`
- **이중 언어**: 한국어 기본, `--en` 영어, `--both` 병기

### 리포트 형식

- 서술형 요약(약 500자) + UL 초과·RDA 미달·스택 최적화·근거 표·대체 제품·면책
- **원소 기호 금지** (Mg·Ca·Zn 등) — 한국어 정식 명칭 필수
- **액션은 제품명 중심** — 사용자가 실물을 특정할 수 있는 형태
- 면책 조항 자동 삽입 (한/영)

### 안전장치

- 영유아(<12세)·반려동물 분석 거부
- 처방약 복용량 조정 권고 금지 (상호작용 경고까지만)
- 임신·수유 시 최상위 경고 배너 + 산부인과 상담 권고
- 쿠팡·네이버 리뷰 수집 금지 (약관 리스크)

### 지속성 

- `user-data/profiles/`, `user-data/products/`, `user-data/stacks/`, `user-data/reports/`에 JSON·마크다운 자동 저장
- 제품 DB는 1년 캐시 (리뉴얼 의심 시 강제 재조회)

### 스킬 아키텍처

- `SKILL.md` (Phase 1~6 조율)
- `references/` 8종: ingredient-aliases · rda-table · bioavailability-table · deficiency-excess-effects · search-templates · interaction-patterns · user-data-schema · disclaimer
- `prompts/` 5종: detect-intent · detect-profile · parse-image · analyze-ingredient · report-template
- `examples/` 3종 (전부 가상 브랜드·가상 페르소나)
