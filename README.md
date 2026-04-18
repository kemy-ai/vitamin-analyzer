# vitamin-analyzer — Claude Skill

> **"지금 먹는 비타민, 정말 나한테 맞는 걸까?"**
> 부족한 건 없는지, 뭘 중복으로 먹고 있지는 않은지, 상한선을 넘긴 성분은 없는지 —
> **내 비타민 포트폴리오를 근거 기반으로 점검하고 최적의 조합을 찾아가는 탐색 여정**을 돕는 Claude Skill입니다.
>
> 성분표 사진 한 장, 또는 복용 중인 제품 이름만 알려주세요. 성별·연령·증상 같은 개인 맥락(선택)을 보태면, 아래 공신력 있는 출처의 근거와 함께 UL(상한섭취량) 초과·RDA(권장섭취량) 미달·중복·약물 상호작용을 점검하고, **국내·해외에서 구할 수 있는 대체 제품**을 가격·리뷰 요약까지 함께 제안합니다.
>
> - **NIH ODS** — 미국 국립보건원(National Institutes of Health) 산하 영양보충제 사무국(Office of Dietary Supplements). 영양소별 팩트시트의 국제 표준
> - **PubMed** — 미국 국립의학도서관(NLM)의 의학 논문 데이터베이스 (약 3,700만 건)
> - **Cochrane Library** — 전 세계 의학 근거를 비평적으로 종합하는 체계적 문헌고찰(Systematic Review) 데이터베이스
> - **Examine.com** — 영양·보충제 분야 근거 요약 플랫폼 (보조 근거)
> - **식약처 KDRI** — 식품의약품안전처·보건복지부 「한국인 영양소 섭취기준 2020」 (한국인 기준 병기)
> - **식약처 건강기능식품 Open API** — 식품의약품안전처 공공데이터 포털. 국내 품목제조신고된 건기식의 **정부 공식 성분·함량·섭취량 기준** (제품명만 알려주면 정확히 식별, 선택적 연동)
>
> 모든 의학적 주장에 **Tier 1~2 근거 URL**이 붙습니다. 처방약 복용량 조언·영유아(<12세)·반려동물 분석은 범위 밖입니다.

---

## 1. 핵심 기능

| 기능 | 설명 |
|------|------|
| 프로필 PRD | 6개 질문으로 프로필 수집 (선택 가능, 의무 아님) |
| 3종 입력 경로 | 텍스트 입력 / 성분표 사진 / 제품명만 |
| 복수 제품 스택 | 여러 제품을 한 번에 올리면 중복·합산·UL 평가 |
| RDA/UL 평가 | NIH ODS 기본 + 식약처 KDRI 2020 병기 |
| 상호작용 검출 | 20종 상호작용 (처방약·보충제·음식) |
| 대체 제품 | 국내·해외에서 구할 수 있는 대체 제품 제안 (국내는 쿠팡·네이버 가격만, 해외는 iHerb 가격 + 리뷰 요약) |
| 근거 투명성 | 모든 주장에 `[출처: Tier N / URL]` 표기 |
| 면책 자동 | 매 리포트 말미 자동 삽입 (축약·생략 금지) |
| 이중 언어 | 한국어 기본, `--en` 영어, `--both` 병기 |

---

## 2. 설치

### 2-1. Claude Code (플러그인 — 권장)

Claude Code에서 두 줄로 설치합니다.

```
/plugin marketplace add kemy-ai/vitamin-analyzer
/plugin install vitamin-analyzer@kemy-ai
```

설치 후 재시작하면 `/vitamin` 슬래시 커맨드가 활성화됩니다.

**사용 예:**
```
/vitamin
/vitamin 비맥스제트 먹고 있는데 괜찮은지 봐줘
/vitamin 35세 여성, 종합비타민 + 마그네슘 + D3 스택 분석
```

- 인자 없음 → 프로필 질문부터 풀 분석 시작 (Phase 1~6)
- 인자 있음 → 인텐트 라우터가 `quick_check` / `quick_replace` / `full_analysis` 중 선택

**업데이트:**
```
/plugin update vitamin-analyzer@kemy-ai
```

**제거:**
```
/plugin uninstall vitamin-analyzer@kemy-ai
/plugin marketplace remove kemy-ai
```

### 2-2. claude.ai (Skill 단독 업로드)

플러그인 기능을 쓰지 않는 claude.ai 사용자는 Skill 디렉토리만 업로드합니다.

1. <https://claude.ai> → Settings → Capabilities → **Skills → Upload**
2. 레포지토리의 `skills/vitamin-analyzer/` 폴더를 ZIP으로 압축해 업로드
3. 새 대화에서 "보충제 분석해줘" 또는 제품명을 입력 → 스킬이 자동 트리거

> 참고: Skill 업로드 UI는 Claude 계정 등급에 따라 제공 여부가 다를 수 있습니다.

### 2-3. 식약처 Open API 연동 (선택 — 한국 제품 분석 품질 향상)

한국 건강기능식품을 **제품명만으로 정확히 식별**하고 싶다면 식약처 Open API 키를 설정하세요. 미설정 시에도 스킬은 정상 동작하며, Perplexity·공식몰·iHerb 체인으로 폴백합니다.

**1) 키 발급 (무료)**
- <https://data.go.kr> 회원가입 → "건강기능식품 정보" 검색 → 활용 신청 → `KFDA_HTFS_KEY` (64자 hex) 수령
- <https://openapi.foodsafetykorea.go.kr> 별도 회원가입 → `C003` / `I0030` / `I2710` 각각 활용 신청 → `KFDA_FOODSAFETY_KEY` (20자) 수령
- 승인 후 1~2시간 대기 (전파 지연)

**2) 환경변수 설정**
```bash
# ~/.zshrc 또는 .env
export KFDA_HTFS_KEY="..."          # data.go.kr 64자
export KFDA_FOODSAFETY_KEY="..."    # foodsafetykorea 20자
```

**3) 스킬 파일에 키 하드코딩 금지**
로컬 테스트용 별도 파일(`.kfda-test-keys`)에만 보관. `.gitignore` + `chmod 600` 필수.

**제공 기능**:
- C003 API: 국내 건강기능식품 6만+ 제품 성분·함량·용법 (정부 공식 DB)
- I2710 API: 원료별 1일 섭취량 하한/상한 522종 (한국 공식 UL)
- 5건 이하 후보 → 리스트 제시 / 6건+ → 제조사·제형·함량 좁히기 질문
- 라벨 사진 업로드 시 교차검증 (**라벨 값이 ground truth**, API 차이는 리뉴얼 가능성 정보로 병기)

상세 규칙: [`references/kfda-htfs-api.md`](skills/vitamin-analyzer/references/kfda-htfs-api.md)

---

## 3. 빠른 사용 예

### 3-1. 텍스트 입력 (단일 제품)

```
제가 35세 여성이고 스트레스·수면 저하가 있어요.
NutriCalm Labs LiverRelax Complex 분석해줘 —
1정당 실리마린 130mg, L-테아닌 200mg, 비타민 B1/B2/B6 각 15mg,
나이아신(니코틴아미드) 15mg NE. 하루 1정 아침.
```

기대 동작:
1. Phase 1 — 프로필 확인 질문 (생략 응답 허용)
2. Phase 2 — 성분 정규화 + 단위 표준화
3. Phase 3 — RDA/UL 평가 + CYP3A4 관련 정보성 안내
4. Phase 4 — 대체 제품 (국내·해외) 검색
5. Phase 5 — 마크다운 리포트 출력 + 면책

실제 기대 출력: [`examples/liver-relax-sample.md`](skills/vitamin-analyzer/examples/liver-relax-sample.md)

### 3-2. 성분표 사진 (단일 제품)

사진과 함께:
```
이 라벨 분석해줘. 45세 여성, 처방약 없음.
```

`prompts/parse-image.md`가 OCR + 구조화 → 나머지 Phase 진행.

실제 기대 출력: [`examples/multi-vitamin-sample.md`](skills/vitamin-analyzer/examples/multi-vitamin-sample.md)

### 3-3. 복수 제품 스택

```
35세 여성, 현재 3개를 같이 먹어.
1) DailyCore Women's Complete (종합비타민)
2) VitalB Labs Stress B-Complex
3) SunD3 High Potency D3 5000
중복되거나 과한 게 있는지 봐줘.
```

실제 기대 출력: [`examples/stack-optimize-sample.md`](skills/vitamin-analyzer/examples/stack-optimize-sample.md)

### 3-4. 제품명만

```
노스토어 밀크씨슬 플러스 분석해줘. 30대 남성.
```

Claude가 WebSearch로 제품 성분 조회 → 저신뢰 시 사용자에게 재확인 요청 → 이후 동일 플로우.

---

## 4. 폴더 구조

```
vitamin-analyzer/                    # Claude Code 플러그인 루트
├── .claude-plugin/
│   ├── plugin.json                  # 플러그인 매니페스트
│   └── marketplace.json             # 셀프 마켓플레이스 등록
├── commands/
│   └── vitamin.md                   # /vitamin 슬래시 커맨드
├── skills/
│   └── vitamin-analyzer/            # Skill 본체
│       ├── SKILL.md                 # 메인 스킬 (Phase 1~6 조율)
│       ├── references/              # Lookup 데이터 (Claude 내부 참조)
│       ├── prompts/                 # 단계별 프롬프트
│       └── examples/                # 테스트 케이스
├── README.md                        # 이 파일
├── CHANGELOG.md
└── LICENSE                          # MIT
```

<details>
<summary>skills/vitamin-analyzer/ 상세 구조</summary>

```
skills/vitamin-analyzer/
├── SKILL.md
├── references/                      # Lookup 데이터
│   ├── ingredient-aliases.md        # 100종 한/영/라틴 동의어 + 단위 변환식
│   ├── rda-table.md                 # 28영양소 RDA/UL (NIH + KDRI)
│   ├── bioavailability-table.md     # 성분 형태별 흡수율 (산화·글리시네이트·메틸폴레이트 등)
│   ├── deficiency-excess-effects.md # 28영양소 부족·초과 건강 영향 카탈로그
│   ├── search-templates.md          # Tier 1~3 WebSearch 쿼리 템플릿
│   ├── interaction-patterns.md      # 20종 상호작용 (약물 12 + 보충제 5 + 음식 3)
│   ├── user-data-schema.md          # profiles/products/stacks/reports JSON 스키마
│   ├── kfda-htfs-api.md             # 식약처 Open API 4종 엔드포인트·필드·교차검증 규칙
│   ├── korean-intake-limits.md      # I2710 큐레이션 (28영양소 + 기능성 12종 한국 공식 섭취 기준)
│   └── disclaimer.md                # 한/영 면책 + 4종 조건부 배너
├── prompts/                         # 단계별 프롬프트
│   ├── detect-intent.md             # Phase 0 인텐트 라우터 (quick_check / quick_replace / full_analysis)
│   ├── detect-profile.md            # Phase 0 프로필 감지 (관계 키워드 → 기존 프로필 매칭)
│   ├── parse-image.md               # Phase 2 이미지 → JSON
│   ├── parse-kfda-response.md       # C003 응답 → 구조화 JSON (STDR_STND/RAWMTRL_NM/NTK_MTHD)
│   ├── analyze-ingredient.md        # Phase 3 정규화·합산·RDA/UL 평가
│   └── report-template.md           # Phase 5 JSON → 마크다운 리포트
└── examples/                        # 테스트 케이스 (가상 브랜드)
    ├── liver-relax-sample.md        # 단일 제품 (간 복합제)
    ├── multi-vitamin-sample.md      # 단일 제품 (25성분 종합)
    └── stack-optimize-sample.md     # 3제품 스택 최적화
```

</details>

사용자 데이터(`user-data/`)는 플러그인 본체 밖에 자동 생성됩니다 (Claude가 실행되는 작업 디렉토리 기준):
```
user-data/
├── profiles/{profile-id}.json       # 사용자 프로필 (본인/가족/친구)
├── products/{brand}_{product}.json  # 성분 DB (프로필 간 공유, 1년 캐시)
├── stacks/{profile-id}_latest.json  # 복용 스택 + 합산 영양소 매트릭스
└── reports/{profile-id}/            # 분석 리포트 .md
```

---

## 5. 데이터 소스 정책 (3-Tier)

| Tier | 소스 | 용도 |
|------|------|------|
| **Tier 1** | NIH ODS / PubMed / Cochrane | 1차 근거. RDA·UL·임상 연구 |
| **Tier 2** | Examine.com / **식약처 C003·I2710 Open API** / 식약처 건기식 원료DB / Drugs.com / NCCIH | 보조 근거 + **국내 제품 1차 식별**. KFDA API는 `KFDA_FOODSAFETY_KEY` env 설정 시 활성화 |
| **Tier 3** | iHerb / 쿠팡 / 네이버쇼핑 | **가격 조회만**. 리뷰는 iHerb만 요약 인용 |

**절대 준수**:
- 쿠팡·네이버 리뷰 수집·인용 **금지** (약관 리스크)
- iHerb 리뷰는 별점+리뷰 개수+요약 2-3줄만, 직접 인용 금지
- 근거 없으면 "근거 부족 (검색 0건)"으로 명시, 추측 금지

---

## 6. 범위 밖 (분석 거부 또는 제한)

| 대상 | 처리 |
|------|------|
| 영유아 (<12세) / 아동 (12-18세) | **분석 거부**. 소아청소년과 상담 권고 |
| 반려동물 (개·고양이 등) | **분석 거부**. 수의사 처방 권고 |
| 처방약 복용량·중단 권고 | **절대 금지**. 상호작용 경고까지만 |
| 질병 진단 | **절대 금지**. 증상 → 영양소 일반 관계만 |
| 임신·수유 | 분석 진행하되 **최상위 경고 배너 + 산부인과 상담 권고** |

---

## 7. 제한사항

- 성분 aliases **상위 100종** (희귀 성분은 미매칭으로 표시됨, 사용자가 직접 입력 보완 가능)
- RDA 테이블 **19세 이상 성인**만 (영유아·아동 미지원)
- 가격 정보는 **WebSearch 조회 시점** 기준 — 실시간 동기화 없음
- 리포트 형식 **마크다운만** (v1) — PDF 변환은 외부 도구 사용
- 언어 **한국어 기본 + 영어**(`--en`) — 다른 언어 미지원
- Claude의 WebSearch·WebFetch 접근 가능 환경에서만 대체 제품 검색 동작

---

## 8. 보안·프라이버시

- 스킬 파일에 **실제 개인정보 0건**. 예시는 **가상 브랜드·가상 페르소나**만 사용 (상표권 리스크 회피)
- 사용자가 대화에서 입력한 프로필·제품 정보는 **Claude 세션 범위 내**에서만 처리. 본 스킬은 외부 저장소로 전송하지 않음

---

## 9. 기여·이슈

- 본 Skill은 범용 배포용. Issue·PR 시 다음 준수:
  - 의학적 주장 변경 → Tier 1 근거 URL 필수
  - RDA·UL 수치 변경 → NIH ODS 또는 KDRI 공식 문서 링크
  - 상표권 리스크가 있는 실제 제품명 추가 **금지**
- 참조 문서 기여: `CONTRIBUTING.md`는 향후 추가 예정

---

## 10. 라이선스

MIT License. 단, 본 Skill은 **의학적 조언을 대체하지 않음**. 모든 리포트 말미 면책 조항을 그대로 유지해야 합니다.

자세한 면책 전문: [`references/disclaimer.md`](skills/vitamin-analyzer/references/disclaimer.md)

---

## 11. 버전

- **v1.3.0 (2026-04-18)**: Claude Code 플러그인 구조로 전환. `/vitamin` 슬래시 커맨드 + `/plugin install` 지원
- **v1.2.1 (2026-04-18)**: nedrug 상세 페이지 HTML 파싱 추가 — 의약품 함량 확보 경로 신설
- **v1.2.0 (2026-04-17)**: 범위 확장 — 건기식 + OTC 일반의약품 + 전문의약품 통합 분석
- **v1.1.3 (2026-04-17)**: 최초 공개 릴리즈

변경 이력: [`CHANGELOG.md`](CHANGELOG.md)
