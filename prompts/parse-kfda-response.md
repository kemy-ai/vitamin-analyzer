# Parse KFDA C003 Response — 구조화 추출 프롬프트

> 식약처 C003 API 응답 원문(JSON)에서 성분·함량·용법을 구조화 JSON으로 추출.
> `references/kfda-htfs-api.md` §4-3 응답 필드 참조.

---

## 입력

C003 응답의 단일 `row` 객체:

```json
{
  "PRDLST_REPORT_NO": "20200123456",
  "PRDLST_NM": "칼맥디 1000",
  "BSSH_NM": "종근당건강",
  "PRDT_SHAP_CD_NM": "정제",
  "STDR_STND": "칼슘 : 표시량(285.9mg/1500mg)의 80~150%, 마그네슘 : 표시량(100mg/1500mg)의 80~150%, 비타민D : 표시량(10㎍/1500mg)의 80~150%",
  "RAWMTRL_NM": "탄산칼슘, 산화마그네슘, 비타민D3, 결정셀룰로스, 스테아린산마그네슘",
  "NTK_MTHD": "1일 2회, 1회 1정을 물과 함께 섭취하십시오.",
  "PRIMARY_FNCLTY": "뼈와 치아 형성에 필요 / 신경과 근육 기능 유지에 필요",
  "IFTKN_ATNT_MATR_CN": "특정 질환자, 임산부, 수유부 섭취 주의"
}
```

---

## 출력 스키마

```json
{
  "product": {
    "kfda_report_no": "20200123456",
    "product_name": "칼맥디 1000",
    "manufacturer": "종근당건강",
    "dosage_form": "정제"
  },
  "ingredients": [
    {
      "name_ko": "칼슘",
      "amount_per_serving": 285.9,
      "unit": "mg",
      "basis_serving_total_mg": 1500,
      "tolerance_percent_range": [80, 150],
      "form_inferred": "탄산칼슘",
      "source": "STDR_STND"
    },
    {
      "name_ko": "마그네슘",
      "amount_per_serving": 100,
      "unit": "mg",
      "basis_serving_total_mg": 1500,
      "tolerance_percent_range": [80, 150],
      "form_inferred": "산화마그네슘",
      "source": "STDR_STND"
    },
    {
      "name_ko": "비타민D",
      "amount_per_serving": 10,
      "unit": "µg",
      "basis_serving_total_mg": 1500,
      "tolerance_percent_range": [80, 150],
      "form_inferred": "비타민D3",
      "source": "STDR_STND"
    }
  ],
  "other_raw_materials": [
    "결정셀룰로스",
    "스테아린산마그네슘"
  ],
  "intake": {
    "times_per_day": 2,
    "amount_per_time": 1,
    "unit": "정",
    "per_day_total": 2,
    "notes": "물과 함께 섭취"
  },
  "functionality": [
    "뼈와 치아 형성에 필요",
    "신경과 근육 기능 유지에 필요"
  ],
  "warnings": [
    "특정 질환자 섭취 주의",
    "임산부 섭취 주의",
    "수유부 섭취 주의"
  ],
  "extraction_confidence": "high"
}
```

---

## 1. STDR_STND 파싱 규칙

### 1-1. 기본 패턴
```
{성분명} : 표시량({함량}{단위}/{1회총량}{단위})의 {하한}~{상한}%
```

### 1-2. 정규식 (참고용 — 실제 추출은 LLM이 수행)
```
([가-힣A-Za-z0-9\-]+)\s*:\s*표시량\(([\d.]+)([a-zA-Zµ㎍]+)\s*/\s*([\d.]+)([a-zA-Zµ㎍]+)\)의\s*(\d+)~(\d+)%
```

### 1-3. 단위 정규화
- `㎍` → `µg` (KFDA 원본은 한자계 단위 `㎍` 사용, 내부 스키마는 `µg` 통일)
- `I.U.` / `IU` → `IU` (공백 제거)
- `mg`, `g`, `µg`, `IU`, `NE`, `DFE`, `RAE` 외 단위는 `raw_unit` 필드에 원문 보존 + 경고

### 1-4. 구분자
- 성분 간 구분자: `, ` (콤마 + 공백) / `,` / ` / ` (슬래시) 모두 허용
- 1개 row에 수십 개 성분이 붙을 수 있음 — 모두 추출

### 1-5. 예외 패턴
- `표시량(X단위)의 80% 이상`: 상한 없음 → `tolerance_percent_range: [80, null]`
- `X단위 이상` (하한만): `amount_per_serving: X, tolerance_percent_range: null`
- `표시량의 80~150%` (절대값 없음): `amount_per_serving: null, source_note: "라벨 참조 필요"`

---

## 2. RAWMTRL_NM 파싱 규칙

### 2-1. 기본
- 콤마(`,`)로 구분된 자유 리스트
- 각 항목을 공백 제거 후 배열화

### 2-2. 기능성 성분 vs 부형제 구분
- STDR_STND에 이미 등장한 원료 → `ingredients[].form_inferred` 로 매핑
- STDR_STND에 없는 원료 → `other_raw_materials[]` (부형제·착색제·향료 등)

### 2-3. 매핑 힌트
| RAWMTRL_NM 항목 | ingredients[].name_ko 매핑 | form_inferred |
|------------------|----------------------------|---------------|
| 탄산칼슘 | 칼슘 | oxide/carbonate |
| 구연산칼슘 | 칼슘 | citrate |
| 산화마그네슘 | 마그네슘 | oxide |
| 글리신산마그네슘 | 마그네슘 | glycinate |
| 비타민D3 | 비타민D | cholecalciferol |
| 메틸코발라민 | 비타민B12 | methylcobalamin |
| 메틸엽산 | 엽산 | methylfolate |

전체 매핑 → `references/bioavailability-table.md` 참조

---

## 3. NTK_MTHD 파싱 규칙

### 3-1. 기본 패턴
```
1일 {N}회, 1회 {M}{단위}(을/를) ...
```

### 3-2. 단위
- `정` · `캡슐` · `포` · `ml` · `스푼` — 원문 그대로 `unit`에 저장
- `per_day_total = times_per_day × amount_per_time`

### 3-3. 예외
- `1일 1회 1~2정`: `amount_per_time: [1, 2]` 배열. 분석 시 상한(2)으로 보수적 계산
- `공복에` / `식후 30분` / `물과 함께`: `notes` 필드에 원문 보존
- 빈 문자열·"상품상세 참조": `times_per_day: null, notes: "라벨 확인 필요"`

---

## 4. IFTKN_ATNT_MATR_CN 파싱 규칙

### 4-1. 분리
- `,` / `.` / `·` / 줄바꿈 / ` 및 ` 로 분리
- 각 항목 앞뒤 공백 제거

### 4-2. 키워드 매핑 (경고 우선순위)
| 원문 키워드 | 플래그 |
|-------------|--------|
| 임산부·임신 | `pregnancy_warning: true` |
| 수유부 | `lactation_warning: true` |
| 어린이·소아 | `child_warning: true` |
| 의약품 복용 | `medication_interaction_warning: true` |
| 알레르기 | `allergy_warning: true` |

---

## 5. 추출 신뢰도 판정

| 조건 | confidence |
|------|-----------|
| STDR_STND 전성분 파싱 + RAWMTRL_NM 매핑 + NTK_MTHD 용량 추출 모두 성공 | `"high"` |
| STDR_STND 일부 실패 / RAWMTRL_NM 매핑 일부 실패 | `"medium"` + 실패 항목 `parsing_warnings[]`에 명시 |
| STDR_STND가 "상품상세 참조" 등 비구조화 / NTK_MTHD 추출 불가 | `"low"` + 사용자에게 라벨 재요청 |

---

## 6. 금지 사항

1. **추정 금지** — 원문에 없는 성분·함량·단위를 LLM이 생성하면 안 됨. 애매하면 `null` + `parsing_warnings[]`
2. **단위 임의 환산 금지** — `µg`을 `mg`으로 자동 변환 X. 분석 단계(analyze-ingredient.md)에서 RDA 비교용으로만 환산
3. **영양소명 번역 금지** — `name_ko`는 KFDA 원문 그대로. 영문·라틴명은 별도 필드(`name_en`, `alias_id`)로 `ingredient-aliases.md` 매핑 통해 붙임
4. **부정확한 form 추론 금지** — RAWMTRL_NM에 `비타민D3`라고 써 있으면 `cholecalciferol`, `비타민D`만 있으면 `form_inferred: null`
5. **경고사항 축약·의역 금지** — IFTKN_ATNT_MATR_CN은 원문 병기 필수 (법적 고지 성격)

---

## 7. 통합 호출 흐름

```
사용자: "종근당 칼맥디 먹고 있어"
  ↓
[KFDA API Service 3 (C003) 호출]
  URL: http://openapi.foodsafetykorea.go.kr/api/{KEY}/C003/json/1/10/PRDLST_NM=칼맥디
  ↓
결과 3건 반환 (1000/2000/3000 시리즈)
  ↓
후보 5건 이하 → 사용자에게 리스트 제시
"'칼맥디 1000', '칼맥디 2000', '칼맥디 3000' 중 어느 것인가요?"
  ↓
사용자 선택: "칼맥디 1000"
  ↓
선택된 row를 본 프롬프트(parse-kfda-response.md)로 파싱
  ↓
products/jongkundang_calmacd_1000.json 저장
  ↓
analyze-ingredient.md로 전달 → RDA/UL 평가
```
