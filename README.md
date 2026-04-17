# vitamin-analyzer — Claude Skill

> **복용 중인 보충제를 근거 기반으로 분석하는 Claude Skill.**
> 사용자 프로필(성별/연령/증상/복약/임신) + 제품 정보(텍스트·라벨 사진·제품명)를 받아 성분 균형·UL 초과·중복·상호작용을 평가하고, 국내 5 + 해외 3 대체 제품을 제안합니다.
> 모든 의학적 주장에 **Tier 1~2 근거 URL**을 첨부합니다.

---

## 1. 핵심 기능

| 기능 | 설명 |
|------|------|
| 프로필 PRD | 6개 질문으로 프로필 수집 (선택 가능, 의무 아님) |
| 3종 입력 경로 | 텍스트 입력 / 성분표 사진 / 제품명만 |
| 복수 제품 스택 | 여러 제품을 한 번에 올리면 중복·합산·UL 평가 |
| RDA/UL 평가 | NIH ODS 기본 + 식약처 KDRI 2020 병기 |
| 상호작용 검출 | 20종 상호작용 (처방약·보충제·음식) |
| 대체 제품 | 국내 5 (쿠팡·네이버 가격만) + 해외 3 (iHerb 리뷰 요약 포함) |
| 근거 투명성 | 모든 주장에 `[출처: Tier N / URL]` 표기 |
| 면책 자동 | 매 리포트 말미 자동 삽입 (축약·생략 금지) |
| 이중 언어 | 한국어 기본, `--en` 영어, `--both` 병기 |

---

## 2. 설치

### 2-1. Claude Code

```bash
# 프로젝트 루트에 클론 또는 복사
cp -R vitamin-analyzer ~/.claude/skills/
# 또는 현재 프로젝트의 .claude/skills/ 하위에 둠
```

Claude Code가 `SKILL.md` 프론트매터의 `name: vitamin-analyzer`를 자동 로드합니다.

### 2-2. claude.ai

1. <https://claude.ai> → Settings → Capabilities → **Skills → Upload**
2. `vitamin-analyzer/` 폴더 전체를 업로드 (ZIP 지원)
3. 새 대화에서 "보충제 분석해줘" 또는 "/vitamin-analyzer"로 호출

> 참고: Skill 업로드 UI는 Claude 계정 등급에 따라 제공 여부가 다를 수 있습니다.

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
4. Phase 4 — 대체 제품 국내 5 + 해외 3 검색
5. Phase 5 — 마크다운 리포트 출력 + 면책

실제 기대 출력: [`examples/liver-relax-sample.md`](examples/liver-relax-sample.md)

### 3-2. 성분표 사진 (단일 제품)

사진과 함께:
```
이 라벨 분석해줘. 45세 여성, 처방약 없음.
```

`prompts/parse-image.md`가 OCR + 구조화 → 나머지 Phase 진행.

실제 기대 출력: [`examples/multi-vitamin-sample.md`](examples/multi-vitamin-sample.md)

### 3-3. 복수 제품 스택

```
35세 여성, 현재 3개를 같이 먹어.
1) DailyCore Women's Complete (종합비타민)
2) VitalB Labs Stress B-Complex
3) SunD3 High Potency D3 5000
중복되거나 과한 게 있는지 봐줘.
```

실제 기대 출력: [`examples/stack-optimize-sample.md`](examples/stack-optimize-sample.md)

### 3-4. 제품명만

```
노스토어 밀크씨슬 플러스 분석해줘. 30대 남성.
```

Claude가 WebSearch로 제품 성분 조회 → 저신뢰 시 사용자에게 재확인 요청 → 이후 동일 플로우.

---

## 4. 폴더 구조

```
vitamin-analyzer/
├── SKILL.md                         # 메인 스킬 (Phase 1~6 조율)
├── README.md                        # 이 파일
├── CHANGELOG.md
├── LICENSE                          # MIT
├── references/                      # Lookup 데이터 (Claude 내부 참조)
│   ├── ingredient-aliases.md        # 100종 한/영/라틴 동의어 + 단위 변환식
│   ├── rda-table.md                 # 28영양소 RDA/UL (NIH + KDRI)
│   ├── bioavailability-table.md     # 성분 형태별 흡수율 (산화·글리시네이트·메틸폴레이트 등)
│   ├── deficiency-excess-effects.md # 28영양소 부족·초과 건강 영향 카탈로그
│   ├── search-templates.md          # Tier 1~3 WebSearch 쿼리 템플릿
│   ├── interaction-patterns.md      # 20종 상호작용 (약물 12 + 보충제 5 + 음식 3)
│   ├── user-data-schema.md          # profiles/products/stacks/reports JSON 스키마
│   └── disclaimer.md                # 한/영 면책 + 4종 조건부 배너
├── prompts/                         # 단계별 프롬프트
│   ├── detect-intent.md             # Phase 0 인텐트 라우터 (quick_check / quick_replace / full_analysis)
│   ├── detect-profile.md            # Phase 0 프로필 감지 (관계 키워드 → 기존 프로필 매칭)
│   ├── parse-image.md               # Phase 2 이미지 → JSON
│   ├── analyze-ingredient.md        # Phase 3 정규화·합산·RDA/UL 평가
│   └── report-template.md           # Phase 5 JSON → 마크다운 리포트
└── examples/                        # 테스트 케이스 (가상 브랜드)
    ├── liver-relax-sample.md        # 단일 제품 (간 복합제)
    ├── multi-vitamin-sample.md      # 단일 제품 (25성분 종합)
    └── stack-optimize-sample.md     # 3제품 스택 최적화
```

사용자 데이터(`user-data/`)는 스킬 본체 밖에 자동 생성됩니다:
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
| **Tier 2** | Examine.com / 식약처 건기식 원료DB / Drugs.com / NCCIH | 보조 근거. Tier 1 부족 시 |
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

## 7. 제한사항 (v1.0.0)

- 성분 aliases **상위 100종** (희귀 성분은 미매칭으로 표시됨, 사용자가 직접 입력 보완 가능)
- RDA 테이블 **19세 이상 성인**만 (영유아·아동 미지원)
- 가격 정보는 **WebSearch 조회 시점** 기준 — 실시간 동기화 없음
- 리포트 형식 **마크다운만** (v1) — PDF 변환은 외부 도구 사용
- 언어 **한국어 기본 + 영어**(`--en`) — 다른 언어 미지원
- Claude의 WebSearch·WebFetch 접근 가능 환경에서만 대체 제품 검색 동작

---

## 8. 보안·프라이버시

- 스킬 파일에 **실제 개인정보 0건** (릴리즈 전 `grep` 검증). 릴리즈 검증 명령 예시 (개발자 본인 이름·이메일·전화번호 키워드로 치환):
  ```bash
  grep -rniE "<YOUR_NAME>|<YOUR_HANDLE>|<YOUR_EMAIL_DOMAIN>|<YOUR_PHONE>" vitamin-analyzer/
  ```
- 예시는 **가상 브랜드·가상 페르소나**만 사용 (상표권 리스크 회피)
- 사용자가 대화에서 입력한 프로필·제품 정보는 **Claude 세션 범위 내**에서만 처리. 본 스킬은 외부 저장소로 전송하지 않음

---

## 9. 기여·이슈

- 본 Skill은 범용 배포용. Issue·PR 시 다음 준수:
  - 의학적 주장 변경 → Tier 1 근거 URL 필수
  - RDA·UL 수치 변경 → NIH ODS 또는 KDRI 공식 문서 링크
  - 상표권 리스크가 있는 실제 제품명 추가 **금지**
- 참조 문서 기여: `CONTRIBUTING.md`가 아직 없음 — v1.1에서 추가 예정

---

## 10. 라이선스

MIT License. 단, 본 Skill은 **의학적 조언을 대체하지 않음**. 모든 리포트 말미 면책 조항을 그대로 유지해야 합니다.

자세한 면책 전문: [`references/disclaimer.md`](references/disclaimer.md)

---

## 11. 버전

- **v1.0.0 (2026-04-17)**: 최초 공개 릴리즈. 100종 aliases / 28영양소 RDA / 20종 상호작용 / 3-Tier 데이터 소스 / 복수 제품 스택 최적화
- **v1.0.1 (2026-04-17)**: 실사용 테스트 피드백 반영 — 제품 완전성 체크리스트, 5단계 Fallback Chain, RDA 미달(deficiency_gap) 분석, `./reports/` 자동 저장
- **v1.1.0 (2026-04-17)**: 인텐트 라우터(3모드), 다중 프로필 지원, 제품 DB 캐싱, 생체이용률 테이블
- **v1.1.3 (2026-04-17)**: 사용자 가독성 개선 — 원소 기호 금지(Mg→마그네슘), 액션은 제품명 중심, 요약은 서술형 500자. §8 구조 재편(§8-A 의학·법 / §8-B 기술 / §8-C UX)

상세 변경 이력: [`CHANGELOG.md`](CHANGELOG.md)
