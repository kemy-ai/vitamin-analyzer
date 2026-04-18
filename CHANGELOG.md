# CHANGELOG — vitamin-analyzer

모든 의미 있는 변경은 이 파일에 기록합니다. 형식은 [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)을 따릅니다.

---

## [1.3.0] — 2026-04-18

**Claude Code 플러그인 전환**: 스킬 단독 배포 → 플러그인 + 마켓플레이스 구조. 사용자가 `/plugin install` 두 줄로 설치하고 `/vitamin` 슬래시 커맨드로 호출 가능.

### 배경

v1.2.1까지는 `SKILL.md`를 사용자가 `~/.claude/skills/`로 수동 복사해야 했음. Claude Code 표준 플러그인 배포 체계(docs-guide·vercel 등과 동일 방식)로 전환하여 설치·업데이트·제거를 `/plugin` 명령으로 표준화.

### 설치 방법 변경

**Before (v1.2.x)**:
```bash
cp -R vitamin-analyzer ~/.claude/skills/
```

**After (v1.3.0)**:
```
/plugin marketplace add kemy-ai/vitamin-analyzer
/plugin install vitamin-analyzer@kemy-ai
```

설치 후 `/vitamin [인자]` 슬래시 커맨드 활성화.

### 디렉토리 구조 재편

```
vitamin-analyzer/                    # 플러그인 루트
├── .claude-plugin/
│   ├── plugin.json                  # 매니페스트 (name·version·description·author)
│   └── marketplace.json             # 셀프 마켓플레이스 등록
├── commands/
│   └── vitamin.md                   # /vitamin 슬래시 커맨드
├── skills/
│   └── vitamin-analyzer/            # Skill 본체 (기존 평면 구조를 이 하위로 이동)
│       ├── SKILL.md
│       ├── references/
│       ├── prompts/
│       └── examples/
├── README.md, CHANGELOG.md, LICENSE
```

### 신규 파일

- `.claude-plugin/plugin.json` — 플러그인 매니페스트 (name: vitamin-analyzer, version: 1.3.0, author: kemy-ai)
- `.claude-plugin/marketplace.json` — 셀프 마켓플레이스 (name: kemy-ai, 단일 플러그인 등록)
- `commands/vitamin.md` — `/vitamin` 슬래시 커맨드. `$ARGUMENTS` 분기 (빈 인자 → Phase 1 / 인자 있음 → detect-intent 라우터)

### 수정 파일

- `SKILL.md` — version 1.2.1 → 1.3.0. 위치 이동: 루트 → `skills/vitamin-analyzer/SKILL.md`
- `README.md` — §2 설치 섹션 전면 개편(플러그인 방식 + claude.ai ZIP 업로드 병기). §4 폴더 구조 플러그인 레이아웃 반영. 내부 링크 `skills/vitamin-analyzer/...` 경로로 갱신
- 기존 `references/` · `prompts/` · `examples/` → `skills/vitamin-analyzer/` 하위로 이동 (내용 변경 없음)

### 슬래시 커맨드 동작

| 입력 | 동작 |
|------|------|
| `/vitamin` | Phase 1 프로필 질문부터 풀 분석 시작 (§2-1-A 8개 카테고리 체크리스트 포함) |
| `/vitamin [자연어]` | `detect-intent.md` 실행 → `quick_check` / `quick_replace` / `full_analysis` 3모드 중 선택 |

### 호환성

- v1.2.x 사용자는 기존 `~/.claude/skills/vitamin-analyzer/` 디렉토리 제거 후 재설치 권장
- claude.ai 사용자는 `skills/vitamin-analyzer/` 폴더만 ZIP으로 업로드하면 기존 방식과 동일 동작
- 스킬 내부 로직·프롬프트·참조 파일 **변경 없음** (경로만 이동). 리포트 출력·분석 규칙은 v1.2.1과 동일

---

## [1.2.1] — 2026-04-18

**의약품 함량 확보 경로 신설**: v1.2.0에서 "식약처 허가정보 API는 성분명만 제공, 함량 미제공" 한계를 보완. nedrug.mfds.go.kr 상세 페이지 "원료약품 및 분량" 테이블 HTML 파싱을 Fallback Chain에 정식 편입.

### 배경

v1.2.0 파이프라인으로 OTC 의약품(비맥스제트정 등) 21개 성분명은 확보 가능하나, 정량 평가(UL 초과·RDA 미달·보충제 중복 합산)는 여전히 불가했음. 3개 Open API(허가정보·e약은요·낱알식별) 모두 함량 필드 미제공을 XML·JSON 형식에서 각각 확인. 공개된 대안으로 nedrug 상세 페이지가 Thymeleaf 서버 렌더링으로 HTML에 함량·단위·환산값을 직접 포함하는 것을 발견.

### Fallback Chain v3 → v3.1

```
Step C2 (의약품 조회) 분할:
  Step C2-a: Open API 3종 (성분명·분류·제조사·효능·용법)
  Step C2-b: ★ nedrug 상세 페이지 파싱 (함량·단위·환산값) — Step C2-a 성공 시 필수
```

### 수정 파일

- `SKILL.md` — version 1.2.0 → 1.2.1, §4-1-3 헤더 "Fallback Chain v3 → v3.1", Step C2 → Step C2-a + Step C2-b 분할. 우선순위 표 갱신 ("한국 제품명 (의약품)" 행이 C2-a → C2-b → D 순서로 명시)
- `references/kfda-drug-api.md` — **§6-B 신규 섹션 추가** (§6-3과 §7 사이). 10개 하위 섹션:
  - 6-B-1 배경 / 6-B-2 엔드포인트 (User-Agent + Referer 필수 헤더) / 6-B-3 응답 구조 (Thymeleaf SSR) / 6-B-4 파싱 대상 테이블 ("원료약품 및 분량") / 6-B-5 단위 정규화 (밀리그램→mg 등) / 6-B-6 환산값 파싱 규칙 (정규식 포함) / 6-B-7 실패/부분 파싱 / 6-B-8 약관·레이트 제한 / 6-B-9 출력 스키마 / 6-B-10 검증 완료 사례
- `prompts/parse-drug-response.md` — §4 재구성 + **§4-B 신규 섹션 추가**. 8개 하위 섹션:
  - 4-B-1 전제 조건 / 4-B-2 호출 / 4-B-3 HTML 테이블 블록 추출 (Python regex 예시) / 4-B-4 행→JSON 매핑 / 4-B-5 elemental 파싱 (regex 포함) / 4-B-6 출력 예시 (벤포티아민·제피아스코르브산·산화마그네슘) / 4-B-7 실패 처리 / 4-B-8 중복 성분 처리 (벤포티아민 + 비스벤티아민 = B1 합산)

### 운영 규칙

- Step C2-a 성공 → Step C2-b 필수 (함량 없는 부분 분석 방지)
- 요청 헤더: `User-Agent: Mozilla/5.0 ...` + `Referer: https://nedrug.mfds.go.kr/` — 없으면 0바이트 응답
- 최대 2회 재시도 → 실패 시 Step D(Perplexity) 폴백
- 캐시: `user-data/products/*.json`에 `material_source: "nedrug_scrape"` + `material_fetched_at` 태깅하여 중복 호출 방지
- 환산값 파싱: `(성분명)(으)로서 (숫자) (단위)` 패턴 → `elemental_amount_mg/µg/IU` 필드 분리 저장 (예: 산화아연 30mg → 아연 24.1mg)

### 검증 완료

- **비맥스제트정** (ITEM_SEQ 202300436, (주)한풍제약) — 21개 성분 전량 함량+단위 파싱 성공 (100%)
- 벤포티아민 + 비스벤티아민 중복 성분 합산 로직 적용 → 비타민 B1 환산 총량 산출
- 리버칸 릴렉스 ↔ 비맥스제트정 재분석 리포트(`user-data/reports/2026-04-17_2330_리버칸vs비맥스제트_재분석.md`)에 v1.2.1 파이프라인 최초 적용

### 호환성

- v1.2.0 리포트는 유효. v1.2.1부터는 의약품 리포트에 함량 기반 UL/RDA 평가가 포함 가능
- v1.1.3 이하 건기식 전용 경로는 변경 없음

---

## [1.2.0] — 2026-04-17

**범위 확장**: 건강기능식품(C003) 단독 → 건기식 + OTC 일반의약품 + 전문의약품 통합 분석.

### 배경

v1.1.3까지는 건기식 API(C003)만 사용 → OTC 멀티비타민(비맥스제트 등)이 "0건"으로 조회되는 한계. 식약처 2026-04-17 승인 API 3종 통합하여 OTC/전문의약품까지 커버.

### 신규 통합 API (식약처 Open API 3종)

| API | 용도 | 키 컨벤션 |
|-----|------|----------|
| `DrugPrdtPrmsnInfoService07/getDrugPrdtPrmsnInq07` | 의약품 제품 허가정보 (성분·분류·제조사) | snake_case (`item_name`) |
| `DrbEasyDrugInfoService/getDrbEasyDrugList` | e약은요 (효능·용법·주의사항) | **camelCase** (`itemName`) |
| `MdcinGrnIdntfcInfoService03/getMdcinGrnIdntfcInfoList03` | 낱알식별 (모양·색·각인) | snake_case |

### Fallback Chain v2 → v3

```
Step C (C003 건기식) 0건 → ★ Step C2 자동 전환 (신규)
Step C2: 허가정보 → e약은요 → (옵션)낱알식별
```

### 신규 파일

- `references/kfda-drug-api.md` — 3개 API 엔드포인트·파라미터·응답 필드·분류 분기 테이블
- `prompts/parse-drug-response.md` — 3개 API 응답을 통합 JSON 스키마로 머지. 라틴/영문 성분명 → alias_id 매핑 (활성형 포함)
- `examples/otc-comparison-sample.md` — 가상 페르소나 + 가상 OTC 제품으로 v1.2.0 파이프라인 검증

### 수정 파일

- `SKILL.md` — frontmatter description 확장, version 1.1.3 → 1.2.0, §4-1-3 Fallback Chain v3 재작성, §7 데이터 소스 표에 의약품 API 3종 추가, §11 참조 파일 갱신
- `prompts/report-template.md`:
  - §1 마커 4종 추가 (`{{DRUG_IDENTIFICATION_SECTION}}` / `{{DRUG_SUPPLEMENT_OVERLAP_SECTION}}` / `{{DRUG_INTERACTION_SECTION}}` / `{{PRESCRIPTION_BANNER}}`)
  - §2 전체 템플릿에 의약품 섹션 3종 + 전문의약품 배너 삽입
  - §3-7a/7b/7c/7d 렌더링 규칙 4종 신설
  - §8-A 필수 규칙 10개 → 13개:
    - #11 의약품 복용량 증감·중단 권고 금지
    - #12 의약품 간 대체 제안 금지
    - #13 전문의약품 필수 규약 (배너 + 자동 섹션 스킵)

### 분류 분기 (SPCLTY_PBLC 기반)

| 분류 | 배너 | 대체 제안 | 복용량 조정 |
|------|------|----------|-----------|
| 건기식 | — | 국내 5 + 해외 3 | 가능 |
| 일반의약품(OTC) | — | 금지 (§8-A #12) | 금지 (§8-A #11) |
| 전문의약품 | 🔴 최상위 배너 | 금지 | 금지 + 자동 섹션 스킵 |
| 한약(생약)제제 | 경고 | 금지 | 금지 |

### 제약 (v1.2.0 현재)

- 허가정보 API는 성분명만 제공, **함량 미제공** → UL/RDA 정량 평가는 건기식에만 적용. 의약품은 성분 존재 여부 + 보충제 중복 여부만 판별
- 함량 정보 필요 시 `user-data/products/*.json` 캐시 또는 라벨 사진 의존

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
