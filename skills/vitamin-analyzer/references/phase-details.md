<!-- SKILL.md §4 본문. 섹션 번호 체계는 SKILL.md를 따른다. -->

## 4. Phase별 상세

### Phase 0-A — API 연결 체크 (매 세션 최초 1회)

**모든 모드에서 가장 먼저 실행.** Phase 0보다 선행.

#### 판단 순서

```
1. KFDA_HTFS_KEY + KFDA_FOODSAFETY_KEY 환경변수 존재?
   → YES: 즉시 Phase 0 진입 (이 Phase 종료)

2. user-data/kfda-skip.json 존재 + reason == "foreign_only"?
   → YES: 즉시 Phase 0 진입 (이 Phase 종료)

3. 둘 다 아니면: 아래 안내 화면 출력 → 선택 대기
```

#### 안내 화면 (출력 템플릿)

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⚠️  식약처 API 미연동 — 한국 제품 정확도 제한
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

이 스킬은 식약처 Open API 연동을 기본으로 설계됐습니다.
API 없이도 동작하지만, 한국 제품(비맥스·아로나민·센트룸 등)의
성분·함량 정확도가 크게 낮아집니다.

  항목              | API 없음          | API 있음
  ─────────────────|──────────────────|─────────────────
  한국 제품 성분     | ★★☆☆☆ (웹 검색)  | ★★★★★ (식약처 공식 DB)
  성분 함량 정확도   | 낮음             | 높음
  의약품/건기식 구분  | 어려움           | 자동 구분

선택해 주세요:
  A. 지금 연동하기 (API 키 발급, 약 5분 — 승인까지 최대 2시간)
  B. 이번만 건너뛰기 (다음 세션에 다시 안내)
  C. 계속 건너뛰기 (외국 보충제만 분석)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

#### 선택별 처리

**A — 지금 연동:**
아래 가이드를 순서대로 출력하고, 키 입력을 기다린 후 Phase 0 진입.

```
[식약처 API 연동 가이드]

Step 1. 건강기능식품 API 키 발급 (KFDA_FOODSAFETY_KEY)
  1) https://openapi.foodsafetykorea.go.kr 회원가입
  2) C003 / I0030 / I2710 각각 활용신청
  3) 발급된 키 복사

Step 2. 의약품 API 키 발급 (KFDA_HTFS_KEY)
  1) https://data.go.kr 회원가입
  2) "식품의약품안전처_의약품개요정보" 검색 → 활용신청
  3) Encoding 탭의 URL-encoded 키 복사 (%2F·%2B 포함)
  ⚠️  승인까지 최대 2시간 소요. 승인 후 아래 입력.

Step 3. 환경변수 등록 (~/.env 또는 claude_desktop_config.json)
  KFDA_FOODSAFETY_KEY=발급받은_키
  KFDA_HTFS_KEY=발급받은_키

키를 받으셨으면 "등록 완료"라고 입력해 주세요.
(지금 당장 발급이 어려우면 'B'를 입력하면 이번만 건너뜁니다)
```

사용자가 "등록 완료" 입력 시 → Phase 0 진입 (환경변수는 재시작 후 적용됨을 안내).

**B — 이번만 건너뛰기:**
- `user-data/kfda-skip.json` **저장하지 않음** (다음 세션 재안내 보장)
- Phase 0 진입. 리포트 상단에 KFDA 미연동 배너 1회 삽입.

**C — 영구 건너뛰기 (외국 제품 전용):**
- `user-data/kfda-skip.json` 생성:
  ```json
  { "reason": "foreign_only", "skipped_at": "YYYY-MM-DD" }
  ```
- Phase 0 진입. 이후 세션에서 이 Phase 재실행 안 함.

#### 규칙

- 이 Phase는 **세션 내 1회만** 실행 (같은 세션에서 두 번째 쿼리부터는 스킵)
- 사용자가 먼저 보충제 정보를 입력해도 **이 Phase를 먼저 처리한 뒤** 분석 시작
- "나중에", "무시", "괜찮아" 같은 우회 표현 → B 선택으로 처리 (저장 없음)

---

### Phase 0 — 프로필 결정 

모든 사용자 발화에 대해 **가장 먼저** 실행. 분석 대상 프로필을 확정.

1. `user-data/profiles/_index.json` 읽기 (없으면 빈 인덱스 생성)
2. `prompts/detect-profile.md` 프롬프트로 발화 분류 → `decision` 수신
3. `decision` 별 처리:

| decision | 처리 |
|----------|------|
| `match` | `profiles/{id}.json` + `stacks/{id}_latest.json` 로드, "지난번 OO 드시던 '{display_name}' 맞으신가요?" 1회 확인 (세션 내 2분 이내 재참조는 생략) |
| `default` | `default_profile_id` 사용. 첫 방문이면 `display_name`을 사용자에게 질문 후 신규 생성 |
| `needs_new_profile` | 프로필명 질문 → slug 변환 → `_index.json` 추가 + `{id}.json` 신규 (demographics는 Phase 1에서) |
| `ambiguous` | 후보 목록 제시, 사용자 선택 후 재진입 |
| `reject_scope` | 영유아·반려동물 거부 문구 출력 후 종료 (SKILL.md §6 거부 프로토콜과 동일) |

**프라이버시 원칙**: profile JSON은 로컬에만 저장. WebSearch 쿼리에는 비식별 정보만 사용 (예: 연령대·성별·일반 증상 키워드).

**스키마**: `references/user-data-schema.md` 참조.

---

### Phase 0.5 — 인텐트 라우팅 

Phase 0 직후 실행. 사용자 발화의 깊이에 맞는 모드를 선택.

1. `prompts/detect-intent.md` 프롬프트 실행 → `mode`, `target`, `suggestion_message` 수신
2. 모드별 분기:

| mode | 다음 단계 |
|------|----------|
| `quick_check` | Phase 1·3 축약 진입. `target.ingredient_ids`만 RDA/UL 조회. 답변 말미에 `suggestion_message` 1문장 |
| `quick_replace` | Phase 1·5 축약 진입. `target.product_keys[0]`의 핵심 성분 중심 대체후보 5+3. `suggestion_message`로 풀 분석 제안 |
| `full_analysis` | 기존 Phase 1~6 전체 실행. `suggestion_message` = null |

**핵심 원칙**:
- 풀 분석을 **강제 유도하지 않음**. 제안만.
- 모호할 때 라이트 모드 편향 (`quick_check` 우선)
- 풀 분석 직후 10분 이내 같은 프로필 질문은 `escalation_allowed: false` → 제안 생략

---

### Phase 1 — 입력 수신

#### 4-1-1. 텍스트 입력
사용자가 성분 리스트를 텍스트로 입력하면 그대로 `ingredients[]` 배열로 변환:
```
"비타민 D 1000IU, 칼슘 500mg, 마그네슘 200mg"
→ [{ raw_name: "비타민 D", amount: 1000, unit: "IU" }, ...]
```

#### 4-1-2. 이미지 입력
- `prompts/parse-image.md` 프롬프트로 Claude native vision 호출
- JSON 출력 수신 → `overall_confidence` 확인
- `medium`/`low` 또는 `partial` → §4-1-2-a 재확인 플로우

##### 4-1-2-a. 재확인 플로우
`requires_user_confirmation: true` 시:
```
성분표 이미지 일부를 명확하게 판독하지 못했습니다.

불명확한 항목: [unclear_fields 리스트]
전체 신뢰도: [overall_confidence]

다음 중 선택해 주세요:
1. 더 선명한 사진을 다시 업로드 (정면, 밝은 조명 권장)
2. 성분을 직접 텍스트로 입력
3. 현재 판독된 부분만으로 분석 진행 (일부 성분 누락 가능, 권장 X)
```

#### 4-1-3. 제품명만 입력 — Fallback Chain v3.1 (KFDA 건기식 + 의약품 통합 + 함량 스크래핑)

**핵심 원칙 (2026-04-18 v3.1 — nedrug 상세 스크래핑 통합)**:
- 한국 제품 → **식약처 Open API가 1차 근거** (건기식 C003 → 의약품 허가정보 순차 조회)
- 건기식 0건 ≠ "모른다". **자동으로 의약품 API 폴백** (OTC 종합비타민이 건기식이 아닌 의약품인 경우 많음 — 비맥스·아로나민·센트룸 등)
- **의약품은 Open API만으로 함량 확보 불가** (허가정보 API는 성분명만 제공). ITEM_SEQ 확보 후 **nedrug 상세 페이지 "원료약품 및 분량" 테이블을 필수 파싱** → 진짜 함량(mg/IU/µg)·규격(KP/USP/EP/별규)·환산값(으로서 X mg) 확보
- 라벨 사진 업로드 시 → OCR + 적절한 API 교차검증 (**라벨 값이 ground truth**)
- 해외 제품은 식약처 미등록 → Perplexity/공식몰/iHerb 체인으로 폴백
- 전체 규칙: `references/kfda-htfs-api.md` (건기식), `references/kfda-drug-api.md` (의약품 API + §9 nedrug 스크래핑)

```
Step A: 사용자 라벨 사진 요청 (항상 1순위 질문)
 - "라벨 사진 있으면 훨씬 정확해요. 있으시면 올려주세요."
 - 사진 제공 시 → prompts/parse-image.md로 JSON 추출 + 라벨의 "의약품/건강기능식품" 표기 감지 → Step B-KFDA로 교차검증 → Step X
 - 없으면 Step B로

Step B: user-data/ DB 재사용
 - 건기식: user-data/products/{key}.json
 - 의약품: user-data/drugs/{item_seq}.json
 - verified_at 365일 이내 + confidence ≥ medium이면 재사용
 - DB hit 시 → Step X로 (네트워크 호출 0건)

Step C: ★ 식약처 C003 API (건강기능식품 1차 근거)
 - 조건: env KFDA_FOODSAFETY_KEY 설정됨
 - 엔드포인트:
   GET http://openapi.foodsafetykorea.go.kr/api/{KEY}/C003/json/1/10/PRDLST_NM={제품명}
 - 결과 처리 (kfda-htfs-api.md §4-4):
   * 1건 → 확정 → prompts/parse-kfda-response.md → Step X
   * 2~5건 → 리스트 제시 + 사용자 선택
   * 6건+ → 좁히기 질문 (제조사 → 제형 → 함량 → 구매시기 순)
   * 0건 → ★ Step C2로 (건기식 미등록 = 의약품 가능성)

Step C2-a: ★ 식약처 의약품 Open API (일반의약품/전문의약품 폴백) [v3 신규, v3.1 재구성]
 - 조건: env KFDA_HTFS_KEY 설정됨 (또는 KFDA_DRUG_KEY)
 - 엔드포인트 순차:
   1. DrugPrdtPrmsnInfoService07/getDrugPrdtPrmsnInq07?item_name={제품명}  (허가정보 — 성분명·분류·제조사·ITEM_SEQ)
   2. 매칭 시 ITEM_SEQ 확보 → DrbEasyDrugInfoService/getDrbEasyDrugList?itemName={제품명}&itemSeq={seq}  (환자용 요약: 효능·용법·주의사항)
   3. (선택) 라벨 사진에 각인 있으면 → MdcinGrnIdntfcInfoService03/getMdcinGrnIdntfcInfoList03?print_front={각인}  (낱알식별)
 - 결과 처리 (kfda-drug-api.md §4-4):
   * 1건 → ITEM_SEQ 확보 → **반드시 Step C2-b 진행** (함량 확보 필수)
   * 2~5건 → 리스트 제시
   * 6건+ → entp_name(제조사)로 좁히기
   * 0건 → Step D로
 - ★ 분류별 분기 (SPCLTY_PBLC 값):
   * "일반의약품" → 성분·효능·상호작용 리포트. "의사/약사 상담" 디스클레이머 필수
   * "전문의약품" → 식별·상호작용까지만. 복용량/대체 섹션 자동 생략 + 상단 배너 의무 (kfda-drug-api.md §8)
   * "정상" 외(취소/변경) → 분석 중단 + "최신 정보 확인 필요" 에러

Step C2-b: ★ nedrug 상세 페이지 "원료약품 및 분량" 파싱 [v3.1 신규 — 함량 확보 필수 경로]
 - 조건: Step C2-a에서 ITEM_SEQ 확보됨
 - 엔드포인트: GET https://nedrug.mfds.go.kr/pbp/CCBBB01/getItemDetail?itemSeq={ITEM_SEQ}
 - 필수 헤더:
   User-Agent: Mozilla/5.0 (일반 브라우저 UA)
   Referer: https://nedrug.mfds.go.kr/
 - 파싱 대상: HTML 내 "원료약품 및 분량" 테이블 (<thead>성분명 | 분량 | 단위 | 규격 | 성분정보</thead> 블록) — prompts/parse-drug-response.md §5
 - 각 행 추출: 순번 / 성분명(한글) / 분량(숫자) / 단위(밀리그램·마이크로그램·아이.유 등) / 규격(KP·USP·EP·별규) / 성분정보(환산: "X(으)로서 Y 단위")
 - 환산 처리: "아연(으)로서 24.1 밀리그램" → elemental_amount_mg = 24.1. UL/RDA 평가는 elemental 기준이 있으면 우선
 - 결과 처리:
   * 성공(≥1행 파싱) → 통합 JSON에 ingredients[] 실함량 주입 → Step X (confidence: high)
   * 실패(테이블 비어있거나 5xx) → 최대 2회 재시도(10초 간격) → 그래도 실패면 Step D(Perplexity)로 폴백 + 리포트 상단 "nedrug 함량 미확보 — 공식 라벨 또는 PDF 첨부문서 참조 권고" 배너
 - 약관: 공공 서비스(식약처)이나 과도한 반복 호출 금지. 캐시(Step B) 우선. 동일 ITEM_SEQ 재조회 금지

Step B-KFDA: 라벨 사진 교차검증 (Step A에서 사진 받았을 때만)
 - 라벨에 "건강기능식품" 마크 → C003 조회
 - 라벨에 "의약품" 마크 → DrugPrdtPrmsnInfoService07 조회
 - 결과 비교 규칙 (kfda-htfs-api.md §8):
   * 함량 동일 → confidence: "high" 확정
   * 10% 이상 차이 → 라벨 값 채택 + "등록 스펙과 차이 — 리뉴얼/배치 가능성" 병기
   * API에만 있는 성분 → 라벨 기준 + 병기
   * API에 없는 성분 → 라벨 기준 + "DB 미등록" 병기

Step D: Perplexity MCP 조회
 - perplexity_research("{브랜드} {제품명} 전성분 함량 mg µg")
 - 해외 제품·식약처 미등록 국내 제품 대상
 - 출처 URL 명시 없으면 confidence: "low"

Step E: 공식 브랜드몰 / 제조사 홈페이지
 - WebSearch: "{브랜드} {제품명} 성분" site:공식도메인
 - 성공 조건: 성분명+함량+단위가 **텍스트로 노출** (이미지/"상품상세 참조"는 실패 — L4)

Step F: iHerb / Amazon 해외 동일 제품
 - WebSearch: "{제품명 영문} Supplement Facts"
 - 해외 직구 제품·글로벌 브랜드 대상

Step G: 사용자 재질문 (최후)
 - "공개된 성분 정보를 찾지 못했습니다."
 - 옵션: (1) 라벨 사진 직접 업로드 (2) 성분 직접 텍스트 입력 (3) 제외

Step X (공통): 성공 시 적절한 위치에 저장
 - 건기식: user-data/products/{key}.json (kfda_report_no 보존)
 - 의약품: user-data/drugs/{item_seq}.json (classification 필드 필수)
 - source.type: label_photo / db_reuse / kfda_htfs / kfda_drug / perplexity / official_site / iherb
 - verified_at = 오늘 날짜
```

**우선순위 요약**:
| 입력 유형 | 1순위 | 2순위 | 3순위 | 교차검증 |
|-----------|-------|-------|-------|----------|
| 라벨 사진 (건기식 마크) | Step A | — | — | Step B-KFDA → C003 |
| 라벨 사진 (의약품 마크) | Step A | — | — | Step B-KFDA → 허가정보 |
| 한국 제품명 (건기식) | Step C (C003) | Step D | — | — |
| 한국 제품명 (의약품) | Step C2-a (Open API) | **Step C2-b (nedrug 함량)** | Step D | — |
| 해외 제품명 | Step D | Step E/F | — | — |
| 제품명+라벨 | Step A + 자동라우팅 | — | — | Step B-KFDA |

**금지 도메인 / 주의사항**:
- ⚠️ **네이버 brand.naver.com 차단율 100%** (L3) — Step E에서 감지 시 **1회만** 시도 후 즉시 다음
- 공식몰 "상품상세페이지 참조" 텍스트만 있으면 **실패 판정** (L4)
- **추측/환각 금지**: 못 찾으면 지어내지 말 것. `{unit}` 미상이면 `null` + confidence: "low"
- **KFDA 키 미설정 시**: Step C/C2 건너뛰고 Step D로. Phase 0-A에서 B 선택한 경우에만 리포트 상단에 "KFDA 미연동 — 분석 시작 전 [A]로 연동하시면 정확도가 크게 향상됩니다" 배너 1회 삽입 (C 선택 시 배너 없음)
- **의약품↔건기식 혼동 금지**: Step C에서 0건이어도 Step C2-a를 반드시 시도해야 OTC 종합비타민(비맥스·아로나민 등)을 놓치지 않음
- **Step C2-a 성공 → Step C2-b 필수**: Open API만으로 끝내면 함량이 전부 "비공개"로 남아 UL/RDA 정량 평가 불가. Step C2-b 생략 금지 (파싱 실패 시에만 Step D로 폴백 허용)

**추가 규칙**:
- Step A~G 각 단계 **실제 시도**했음을 내부 로그로 추적 → 사용자에게 "Step C에서 0건 → Step C2-a에서 일반의약품 1건 확정 → Step C2-b에서 21성분 함량 확보" 등 투명하게 보고
- 동일 세션 내 동일 제품 재조회 금지 (Step B DB hit로 해결)
- KFDA 응답 원문은 로컬 캐시에만. 외부 전송 금지 (kfda-htfs-api.md §10 / kfda-drug-api.md §10)
- 의약품 데이터는 `references/kfda-drug-api.md §0` 범위 경계 엄격 준수 — 복용량 증감/대체 제안 금지

#### 4-1-4. 복수 제품 입력
- 2개 이상의 이미지/텍스트 입력을 `products[]` 배열로 관리
- 각 제품에 `product_index` 부여 (1부터)

---

### Phase 2 — 정규화

`prompts/analyze-ingredient.md` §2 (정규화) + §3 (단위 표준화) 실행.

참조 파일:
- `references/ingredient-aliases.md` — 100종 canonical/alias 테이블
- `references/ingredient-aliases.md` §0 — 단위 변환식 9종

출력: `{ canonical_name, standardized_amount, standardized_unit, notes }`

**미매칭 성분**: `unrecognized_ingredients[]` 배열에 기록 (리포트에 명시).

---

### Phase 3 — 스택 합산 + RDA/UL + 중복 검출

`prompts/analyze-ingredient.md` §4~§6 실행.

참조 파일:
- `references/rda-table.md` — NIH + KDRI RDA/AI/UL 테이블
- `references/rda-table.md` §31 — 플래그 규칙 (🔴/🟠/🟡/✅/ℹ️)
- `references/rda-table.md` §32 — UL 치명 8항목 체크리스트

프로필이 없으면 "일반 성인 (19-50세, 여성 기준)" 기본값. 프로필에 따라:
- 성별 미입력 → 여성 기준 (UL 기준 보수적)
- 나이 미입력 → 19-50세 평균
- 임신·수유 → 해당 열 조회

---

### Phase 4 — 상호작용 + 근거 URL 조회

`prompts/analyze-ingredient.md` §7 (interaction) + §8 (evidence).

참조 파일:
- `references/interaction-patterns.md` — 20개 상호작용 패턴
- `references/search-templates.md` — Tier 1~2 WebSearch 쿼리 템플릿

**절대 규칙**:
1. 모든 의학적 주장 뒤 `[출처: Tier N / URL]` 필수
2. URL fabricate 금지 — WebSearch 결과에 있는 URL만 인용
3. 세션 내 중복 WebSearch 금지 (성분별 캐싱)
4. 처방약 상호작용은 "상담 권고"만, 용량 조정 권고 금지

---

### Phase 5 — 대체제품 조회 (국내 5 + 해외 3)

`references/search-templates.md` §10-2 (대체제품 파이프라인).

| 채널 | 용도 | 제약 |
|------|------|------|
| 쿠팡 | 국내 가격 | **가격만, 리뷰 인용 절대 금지** |
| 네이버쇼핑 | 국내 가격 | **가격만, 리뷰 인용 절대 금지** |
| iHerb | 해외 가격 + 리뷰 | 리뷰 요약 허용 (별점+개수+요약) |

- 국내 5개: 쿠팡·네이버 혼합
- 해외 3개: iHerb 전용
- 가격 조회 실패한 채널 수를 리포트에 명시 ("N개 조회 실패")
- 해외가는 원화 환산 병기
- **가격 조회 일자 반드시 명시**

---

### Phase 6 — 리포트 생성 + 자동 저장 (프로필별 디렉토리 + 원본 백업)

`prompts/report-template.md` 호출.

1. `analyze-ingredient.md`의 출력 JSON을 템플릿에 적용
2. 조건부 배너 삽입 (`warnings_triggered` 평가):
 - `infant_or_pet: true` → §5 거부 리포트만 출력, 이후 섹션 스킵
 - `pregnancy: true` → §3-1 배너
 - `prescription_drug_present: true` → §3-3 배너
 - `ul_exceeded.length > 0` → §3-4 배너
3. `{{DISCLAIMER}}`는 `references/disclaimer.md` §1(한) / §2(영) 자동 삽입 — 수정·축약 금지
4. `{{PROFILE_NAME}}` 마커 치환 — "{display_name}" 형식
5. `{{UL_EXCEEDED_SECTION}}` — UL 초과/근접 상세 (`ref/deficiency-excess-effects.md` 카탈로그 연결)
6. `{{DEFICIENCY_SECTION}}` — RDA 미달 상세 (부족 건강 영향 + 식단 보완)
7. **원본 성분 JSON 백업** :
 - 파싱된 모든 영양소 mg/µg 수치를 `user-data/stacks/{profile_id}_latest.json`에 저장
 - `/compact` 후 재분석 시 추정치로 대체되지 않도록 보호
 - 실패 시 리포트 최상단에 `{{SAVE_FAILURE_BANNER}}` 치환 (리포트 본문은 정상 노출)
8. **자동 .md 파일 저장** (프로필별 경로):
 - 저장 경로: `./user-data/reports/{profile_id}/YYYY-MM-DD_HHMM_analysis.md`
 - 파일명 중복 시 `_2`, `_3` 서픽스
 - Write 도구로 저장 → `{{FILE_SAVED_PATH}}` 마커에 경로 치환
9. **`latest.md` 심볼릭 링크 갱신** :
 - `./user-data/reports/{profile_id}/latest.md` → 방금 저장한 파일로 `ln -sfn`
 - 실패 시 plain 파일 복사로 fallback
10. 사용자에게 리포트 전문 노출 + 저장 경로 알림 (`{{FILE_SAVED_PATH}}`)
11. **대시보드 (요청 시만)**: `--dashboard` 또는 자연어 요청이 있을 때만 `prompts/render-dashboard.md` 절차로 `./user-data/reports/{profile_id}/YYYY-MM-DD_HHMM_dashboard.html` 생성 후 브라우저로 연다. `full_analysis`에서는 반드시 8단계(마크다운 저장) **뒤**에 한다 (quick 모드는 8단계가 없으니 바로). 요청이 없으면 건너뛴다. `infant_or_pet: true`면 만들지 않는다

**모드별 축약 규칙**:
- `quick_check` — Phase 6는 **대화 내 응답만**, 파일 저장 없음 (Step 7/8/9 모두 스킵). `{{SUGGESTION}}` 마커만 사용. 단 대시보드를 직접 요청하면 Step 11로 질문에 나온 제품만 담은 HTML 하나를 저장한다
- `quick_replace` — 대체후보 표만 저장 선택 가능 (사용자가 "저장해줘" 할 때만). Step 7(stack 백업)은 건너뜀. 대시보드 직접 요청 시 Step 11 (quick_check와 같은 범위)
- `full_analysis` — 위 10단계 전체 실행 + 요청 시 Step 11

---
