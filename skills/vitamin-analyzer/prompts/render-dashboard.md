# Render Dashboard - 대시보드 HTML 생성 절차

> **용도**: 분석 결과를 로컬 브라우저에서 여는 HTML 대시보드로도 보여 준다. 마크다운 리포트를 **대체하지 않는 보조 산출물**이다.
> **템플릿**: `assets/dashboard-template.html` (스킬 디렉터리 기준). 템플릿은 그대로 두고, 데이터 자리 하나에 JSON만 끼워 넣는다.
> **판정 기준 숫자의 출처**: `prompts/analyze-ingredient.md` §5-3·§5A-3 하나. 템플릿에는 판정 숫자가 없고, 이 절차가 그 값을 데이터 JSON의 `status_rules`로 옮긴다.

---

## 1. 언제 만드는가

| 조건 | 동작 |
|------|------|
| `--dashboard` 플래그 | 만든다 |
| 자연어 요청: "대시보드로 보여줘", "그래프로", "차트로", "시각화해줘", "한눈에 보게" 등 | 만든다 |
| 요청 없음 | **만들지 않는다** (기본값은 마크다운만) |
| 영유아·반려동물 거부 (`infant_or_pet: true`) | **만들지 않는다**. 거부 리포트만 |
| 성분 수치가 하나도 확보되지 않음 | 만들지 않고 "대시보드에 넣을 수치가 없어 만들지 않았어요"라고 한 줄 안내 |

모드별 범위 (`prompts/detect-intent.md`):

- `full_analysis`: Phase 6의 마크다운 리포트를 **먼저** 저장한 뒤 대시보드를 만든다. 대시보드에는 스택 전체를 담는다
- `quick_check` · `quick_replace`: 대시보드를 **직접 요청했을 때만** HTML 하나를 저장한다. 마크다운 리포트는 그 모드 규칙대로 만들지 않는다. 대시보드에는 **질문에 나온 제품만** 담고, 성분도 그 제품에 든 것만 넣는다 (다른 제품으로 채워지는 성분이 "부족"으로 잘못 뜨지 않게). `cautions[]` 첫 줄에 "질문한 제품만 계산했습니다. 다른 제품까지 합친 판정은 전체 분석에서 볼 수 있어요"를 넣는다

---

## 2. 데이터 JSON 형식 (schema_version 1)

템플릿은 아래 형식만 읽는다. 필드가 빠지거나 형식이 틀리면 템플릿이 오류 안내를 띄우고 차트를 그리지 않는다.

```json
{
  "schema_version": 1,
  "lang": "ko",
  "mode": "full_analysis",
  "title": "영양제 스택 점검",
  "meta_line": "40대 남성 · 기준: 한국인 영양소 섭취기준 (남 30-49세) · 보충제에서 들어오는 양만 계산 (식사 제외)",
  "generated_at": "2026-10-23 15:33",

  "banners": [
    { "kind": "prescription", "text": "🔴 처방약 상호작용 주의 - 복용 중인 처방약이 입력되었습니다. ..." }
  ],
  "ul_banner_text": "🔴 상한섭취량(UL) 초과 경고 - 분석 결과 [성분명]이(가) 상한섭취량을 초과했습니다. ...",

  "status_rules": [
    { "key": "over",   "label": "상한 초과",   "tone": "bad",     "metric": "ul_pct",  "min": 100 },
    { "key": "near",   "label": "상한 근접",   "tone": "warn",    "metric": "ul_pct",  "min": 70 },
    { "key": "lack",   "label": "현저 부족",   "tone": "warn",    "metric": "rda_pct", "below": 50,  "low": true },
    { "key": "short",  "label": "경미 부족",   "tone": "warn",    "metric": "rda_pct", "below": 70,  "low": true },
    { "key": "excess", "label": "과잉 가능성", "tone": "warn",    "metric": "rda_pct", "above": 150, "only_if_no_ul": true },
    { "key": "high",   "label": "권장량 이상", "tone": "neutral", "metric": "rda_pct", "above": 150 },
    { "key": "ok",     "label": "적정",        "tone": "ok" }
  ],

  "products": [
    {
      "id": "p1",
      "name": "가상 종합비타민 A",
      "kind": "종합비타민",
      "category": "health_food",
      "unit": "정",
      "label_dose": 2,
      "now_dose": 1,
      "plan_dose": 2,
      "plan_note": "라벨 용량대로 늘림"
    }
  ],
  "plan_summary": "새로 살 제품 없이 개수만 바꿉니다.",

  "nutrients": [
    {
      "id": "magnesium",
      "name": "마그네슘",
      "group": "mineral",
      "unit": "mg",
      "rda": 370,
      "rda_basis": "KDRI 남 30-49세",
      "ul": 350,
      "ul_basis": "보충제만",
      "ref_range": null,
      "source_url": "https://ods.od.nih.gov/factsheets/Magnesium-HealthProfessional/",
      "source_label": "NIH ODS"
    }
  ],

  "per_unit": [
    { "product": "p1", "nutrient": "magnesium", "amount": 50 }
  ],

  "cautions": [
    { "text": "칼슘이 든 제품과 철이 든 제품은 시간을 나눠 드시는 것이 좋습니다.", "source_url": "https://ods.od.nih.gov/factsheets/Iron-HealthProfessional/" }
  ],

  "disclaimer_md": "## ⚠️ 면책 조항\n\n**본 분석은 의학적 조언이 아닙니다.**\n\n- ...\n\n> 분석 결과에 대한 ..."
}
```

### 2-1. 필드 규칙

| 필드 | 규칙 |
|------|------|
| `lang` | **필수**, `"ko"` 또는 `"en"`만. 기본 `"ko"`, `--en`이면 `"en"`, `--both`면 `"ko"`. 템플릿 고정 문구(버튼·축·제목)가 이 값으로 바뀐다. 나머지 글은 데이터에 쓴 언어 그대로 |
| `mode` | `full_analysis` · `quick_check` · `quick_replace` 중 하나 (표시용) |
| `meta_line` | 프로필 요약 + 기준표 이름 + "보충제에서 들어오는 양만 계산 (식사 제외)". 이름·증상·복용 약 이름은 넣지 않는다 (나이대·성별까지만) |
| `banners[]` | 고정 배너. 문구는 아래 2-2의 원문을 **그대로** 옮긴다. 해당 없으면 `[]` |
| `ul_banner_text` | `references/disclaimer.md` §3-4 원문. 자리 표시 `[성분명]`(영어판은 `[ingredient]`)은 그대로 둔다 (템플릿이 개수에 따라 바뀌는 성분명으로 채운다) |
| `status_rules[]` | 위 예시 7줄을 그대로 쓴다. 숫자는 `analyze-ingredient.md` §5-3·§5A-3에서 왔다. 그 절이 바뀌면 이 예시도 같은 커밋에서 바꾼다. 위에서부터 처음 맞는 규칙 하나가 판정이다 (`min` = 이상, `below` = 미만, `above` = 초과) |
| `products[].id` | `p1`, `p2` ... 순서대로. 화면 색도 이 순서로 정해진다 |
| `products[].category` | `health_food` (건강기능식품·일반 보충제) / `otc_drug` (일반의약품) / `rx_drug` (전문의약품). 의약품 두 종류는 개수 조절 버튼이 잠긴다 |
| `products[].label_dose` | 라벨의 **하루** 섭취 개수 (`parse-kfda-response.md` §1-6 판정 결과). 모르면 `null` |
| `products[].now_dose` | 지금 먹는 하루 개수 |
| `products[].plan_dose` | 권고안의 하루 개수. 권고가 없으면 `null`. 건강기능식품은 `label_dose`를 넘길 수 없다. **의약품은 항상 `null`** (`report-template.md` §8-A #11, 복용량 조정 안내 금지) |
| `products[].plan_note` | 바꾸는 이유 메모 (선택). 화면에는 표시하지 않는다 |
| `plan_summary` | 권고안 한 줄 덧붙임. 없으면 `null` |
| `nutrients[].group` | `vitamin` · `mineral` · `functional` (분류 기록용, 화면 배치에는 쓰지 않음). 배치는 `rda`로 정해진다: `rda`가 `null`인 성분은 막대가 아니라 "자세히 보기"의 기능성 성분 표로 간다 |
| `nutrients[].rda` · `ul` | 마크다운 리포트 §3-4 표와 **같은 값** (같은 분석 JSON의 `stack_analysis[].rda.value` · `ul.value`). 없으면 `null` |
| `nutrients[].ul_basis` | "보충제만" 또는 "식사 포함" (마그네슘처럼 보충제만 기준인 성분 구분, 교훈 L7) |
| `nutrients[].source_url` | 실제 조회한 Tier 1-2 URL만, `https://`로 시작하는 것 (그 밖의 주소는 템플릿이 링크로 만들지 않는다). 없으면 `null` (화면에 "근거 링크 미확인"이 뜬다). **URL을 지어내지 않는다** |
| `nutrients[]` 범위 | `full_analysis`: `stack_analysis[]`의 성분 + `deficiency_gap[]`의 성분(섭취 0 포함). 막대에 "0%"로 보이는 성분이 부족 판정을 받는다. quick 모드: 담은 제품에 든 성분만 (1절) |
| `per_unit[].amount` | **1개(1정·1캡슐)당** 양, `nutrients[].unit` 단위. = 라벨 하루 표시량 ÷ `label_dose`. `label_dose`가 `null`이면 = 지금 하루 양 ÷ `now_dose`. 둘 다 없거나 0이면 1개당 양을 알 수 없으니 그 제품은 대시보드에서 빼고 `cautions[]`에 이유를 적는다 |
| `cautions[]` | "이 결과를 읽을 때" 목록: 데이터 출처, 가정, 식사 미포함, 상호작용, 복용 간격. 의학적 주장이 든 줄은 `source_url` 필수, 가정·출처 설명 줄은 `null` 가능 |
| `disclaimer_md` | `references/disclaimer.md` §1 (영어면 §2) 코드블록 안의 글을 **한 글자도 바꾸지 않고** 옮긴다. 맨 앞 `---` 줄만 뺀다 |

### 2-1a. 템플릿이 통째로 거부하는 데이터 (오류 화면만 뜸)

하나라도 해당하면 대시보드가 그려지지 않는다. 1단계 작성 때 피한다.

- `schema_version`이 1이 아님, `lang`이 `ko`·`en` 외
- 문자열 필수 필드 누락: `mode` · `title` · `meta_line` · `ul_banner_text` · `disclaimer_md` (`ul_banner_text`·`disclaimer_md`는 빈 글자도 거부)
- 배열 필수 필드 누락: `banners` · `per_unit` · `cautions` (빈 배열은 허용) / `status_rules` · `products` · `nutrients` (1개 이상)
- id 중복, `per_unit`이 없는 제품·성분 id를 가리킴, `category`가 3종 외
- `rda`·`ul`이 0 이하, 양·개수가 음수이거나 숫자가 아님
- 건강기능식품의 `plan_dose`가 상한(`label_dose`, 이미 더 먹고 있으면 `now_dose`, `label_dose`가 `null`이면 `now_dose`)을 넘음

조용히 처리하는 것 (오류는 아님): 의약품의 `plan_dose`는 값이 있어도 무시, `https://`가 아닌 근거 주소는 "근거 링크 미확인"으로 표시.

### 2-2. 고정 배너 원문

| `kind` | 조건 (`warnings_triggered`) | 원문 |
|------|------|------|
| `rx_product` | 전문의약품이 제품 목록에 있음 | `report-template.md` §8-A #13 "전문의약품 - 담당 의사·약사 상담 필수" |
| `pregnancy` | `pregnancy` 또는 `breastfeeding` | `disclaimer.md` §3-1 |
| `prescription` | `prescription_drug_present` | `disclaimer.md` §3-3 |

순서는 위 표 순서. `kind`는 기록용이라 화면에 나오지 않는다. 마크다운 배너의 `>`와 줄바꿈은 빼고 한 문단으로 잇되, 굵은 제목과 본문 사이에는 ` - `를 넣는다 (예: "🔴 상한섭취량(UL) 초과 경고 - 분석 결과 ..."). 굵게 표시(`**`)는 빼도 된다. 문장 자체는 바꾸지 않는다. `ul_banner_text`도 같은 방식으로 잇는다.

### 2-3. 넣지 않는 것

- 사용자 이름, 증상, 복용 중인 처방약 이름, 기저질환 (`meta_line` 포함 어디에도)
- 대체 제품·가격 (마크다운 리포트에만)
- 의약품의 권고 개수 (`plan_dose`는 `null`)

---

## 3. 생성 절차

1. **데이터 JSON 작성**: 같은 분석의 출력 JSON(`analyze-ingredient.md` §9)과 `user-data/stacks/{profile_id}_latest.json`에서 옮긴다. 새로 계산하지 않는다
2. **자체 점검** (하나라도 어긋나면 고친 뒤 진행):
   - `now_dose` 기준으로 손으로 계산한 성분 3개의 하루 양이 마크다운 리포트 §3-4 표와 같다
   - 모든 `per_unit[].product`가 `products[].id`에, 모든 `per_unit[].nutrient`가 `nutrients[].id`에 있다
   - 건강기능식품의 `plan_dose`가 `label_dose` 이하, 의약품의 `plan_dose`가 `null`
   - `disclaimer_md`가 `disclaimer.md` 원문과 같다
3. **끼워 넣기**: 템플릿의 `__VITAMIN_DATA__` 한 곳을 JSON으로 바꾼다. 이때 JSON 안의 `<`를 모두 여섯 글자 `\u003c`(역슬래시 u 0 0 3 c)로 바꿔 `</script>`가 스크립트를 끊지 못하게 한다
4. **저장**: `user-data/reports/{profile_id}/YYYY-MM-DD_HHMM_dashboard.html` (마크다운 리포트와 같은 폴더, 같은 시각 접두어). 이름이 겹치면 `_2`, `_3`. 배포 폴더·스킬 디렉터리 안에는 저장하지 않는다
5. **띄우기**: 아래 4절

Claude Code에서 3·4단계 (Bash 사용 가능할 때):

```bash
python3 - "$TEMPLATE" "$DATA_JSON" "$OUT_HTML" <<'PY'
import json, sys
tpl, data, out = sys.argv[1:4]
html = open(tpl, encoding="utf-8").read()
assert html.count("__VITAMIN_DATA__") == 1, "template placeholder missing"
payload = json.dumps(json.load(open(data, encoding="utf-8")), ensure_ascii=False).replace("<", "\\u003c")
assert "<" not in payload, "unescaped < in data"
open(out, "w", encoding="utf-8").write(html.replace("__VITAMIN_DATA__", payload))
PY
```

- `$TEMPLATE` = 스킬 디렉터리의 `assets/dashboard-template.html`
- `$DATA_JSON` = 1단계 JSON을 임시 파일로 쓴 것 (작업 후 지워도 된다)
- Bash가 없으면 템플릿 전문을 읽어 같은 치환을 한 결과를 Write로 저장한다. 저장 뒤 `<script id="vitamin-data"` 태그 안쪽(데이터 구간)에 `<` 글자가 하나도 없는지 다시 읽어 확인한다. 있으면 그 자리를 3단계 규칙대로 바꿔 다시 저장한다 (제품명 등은 외부에서 온 값이라 `</script>`가 들어 있을 수 있다)

---

## 4. 사용자에게 보여 주기

| 환경 | 방법 |
|------|------|
| Claude Code (macOS) | `open "<저장 경로>"` |
| Claude Code (Linux / Windows) | `xdg-open` / `start ""` . 실패하면 경로만 알린다 |
| claude.ai | 로컬 저장·`open`이 없다. 같은 HTML을 아티팩트로 보여 주고, 파일로 받으려면 아티팩트에서 내려받도록 안내 |

응답 끝에 한 줄: "대시보드: `<저장 경로>` (브라우저에서 열림. 개수를 바꿔 보는 기능은 참고용입니다)". 마크다운 리포트 저장 안내(`report-template.md` §3-10a)와 나란히 둔다.

---

## 5. 안전 장치 (템플릿이 하는 일, 절차에서 깨지 않을 것)

- 면책 조항 전문은 "자세히 보기" 안에 있고, "참고용 분석이며 의학적 조언이 아닙니다" 한 줄은 접히지 않고 늘 보인다
- 개수 조절 상한은 라벨 하루 용량. 이미 라벨보다 많이 먹는 제품은 지금 개수까지만, "라벨 용량보다 많아요" 표시. 라벨 용량을 모르면 지금 개수까지만
- 고른 개수에서 상한을 넘는 성분이 있으면 `ul_banner_text`가 맨 위 빨간 배너로 뜬다
- 의약품은 버튼이 잠기고 "의약품은 복용량 조정 안내를 하지 않습니다"가 뜬다. 라벨보다 많이 먹어도 의약품에는 "라벨 용량보다 많아요"를 띄우지 않는다
- 판정은 반올림 전 값으로 한다 (마크다운 리포트와 같게). 막대 옆 %는 정수 반올림, 마우스 설명 창의 %는 소수 1자리 내림이라 경계 근처에서 두 숫자가 조금 다를 수 있다
- 모든 글은 텍스트로만 넣는다 (HTML로 해석하지 않음). 링크는 `https://`만 연다
- 외부 네트워크 요청이 없다 (오프라인에서 더블클릭으로 열림)
