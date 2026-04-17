# 식약처 의약품 Open API — 통합 참조

> 일반의약품(OTC)·전문의약품(ETC) 제품 식별·성분 확인의 **1차 근거**. `kfda-htfs-api.md`(건기식)의 자매 문서. C003(건기식) 조회가 0건일 때 자동 폴백 대상.

---

## 0. 범위 경계 (★최우선)

### 0-1. 이 API가 하는 일
- ✅ 사용자가 복용하는 의약품 **식별** (제품명·각인·모양 기반)
- ✅ 의약품에 포함된 **성분·함량 추출** (보충제 중복 체크용)
- ✅ **보충제↔의약품 상호작용** 경고 (일반 원칙 수준)
- ✅ 공식 **적응증·주의사항** 안내 (식약처 승인 문구 그대로)

### 0-2. 이 API가 하지 않는 일
- ❌ **의약품 복용량 증감/중단 권고** — 의사 판단 영역. 절대 금지
- ❌ **의약품 간 대체 제안** — OTC끼리도 금지. 건기식 교체 제안(국내 5+해외 3)과 다른 영역
- ❌ **증상 진단** — "당신의 통증은 X 때문"류 추론 금지
- ❌ **처방약 임의 변경 유도** — 전문의약품은 식별·성분 확인까지만

### 0-3. 리포트 삽입 원칙
- 의약품 데이터는 "**식별 결과 + 보충제 스택과의 교차검증**"까지만 리포트에 반영
- 모든 의약품 관련 권고는 **"의사/약사 상담"**으로 수렴
- 성분 중복이 감지되어도 "중단하세요"가 아니라 "중복 가능성 — 의사 확인 권고"

상세 금지 규칙: `prompts/report-template.md` §8-A 참조.

---

## 1. 전제

### 1-1. 왜 이 API가 필요한가
- **식약처 공식 DB** — 품목허가된 국내 의약품 전수
- **건기식 API(C003)의 공백 보완** — OTC 종합비타민(예: 비맥스제트·아로나민·센트룸)은 의약품 분류로, C003에서 0건 반환
- **성분 authoritativeness** — 제품 허가정보의 `ITEM_INGR_NAME`은 식약처 승인 시점의 전 성분 리스트 (라틴/영문명 표준 표기)
- **환자 친화적 톤** — e약은요(`DrbEasyDrugInfoService`)는 일반인용 한국어 문장으로 반환 → 리포트에 그대로 인용 가능

### 1-2. 한계
- **해외 의약품 미등록** — 미국/유럽 OTC는 기존 Fallback Chain 후순위
- **건강기능식품은 여기 없음** — C003이 담당. 두 API를 혼용하지 말 것
- **전파 지연** — data.go.kr 승인 직후 최대 2시간 `Forbidden` 반환 가능
- **용법·용량 자유텍스트** — 연령대별 다른 용법이 한 문장에 섞임. 자동 파싱 난이도 높음

---

## 2. API 3종 개요

| # | 서비스명 | ServiceID | Endpoint | 용도 |
|---|----------|-----------|----------|------|
| 1 | 의약품 낱알식별 정보 | `MdcinGrnIdntfcInfoService03` | `apis.data.go.kr` | 각인·색상·모양·크기 → 제품 역추적 (사진 기반 식별) |
| 2 | 의약품 제품 허가정보 | `DrugPrdtPrmsnInfoService07` | `apis.data.go.kr` | **★ 핵심 — 공식 전 성분 리스트 + 허가 상세** |
| 3 | 의약품개요정보(e약은요) | `DrbEasyDrugInfoService` | `apis.data.go.kr` | **★ 환자용 한국어 요약** (효능·용법·주의·상호작용) |

### 2-1. 키 관리 (절대 준수)
- **기존 `KFDA_HTFS_KEY` 재사용 가능** — 3개 API 모두 data.go.kr 1471000 네임스페이스, 동일 사용자 키로 호출 확인됨 (2026-04-17 테스트)
- 별도 환경변수 분리 불필요. 단, 인증 오류 추적 편의성을 원할 경우 `KFDA_DRUG_KEY` 별도 설정도 허용
- 하드코딩 금지, `.gitignore` 필수 — `kfda-htfs-api.md` §2-1 동일 규칙 적용

### 2-2. 호출 공통 규칙
- HTTP GET, 응답 `type=json` 권장
- 파라미터 명칭 **API별로 snake_case/camelCase 혼재 — 주의** (§3/§4/§5 각각 명시)
- 타임아웃 30초, 지수 백오프 3회

---

## 3. Service 1 — 의약품 낱알식별 정보

### 3-1. 엔드포인트
```
GET https://apis.data.go.kr/1471000/MdcinGrnIdntfcInfoService03/getMdcinGrnIdntfcInfoList03
  ?serviceKey={KEY}
  &type=json
  &pageNo=1
  &numOfRows=10
  &item_name={제품명}       # 부분 일치
  &entp_name={제조사명}     # 선택
  &print_front={앞각인}     # 선택 (예: "B-Z")
  &drug_shape={모양}        # 선택 (예: "타원형")
  &color_class1={색상}      # 선택 (예: "빨강")
```

### 3-2. 주요 응답 필드
| 필드 | 의미 | 활용 |
|------|------|------|
| `ITEM_SEQ` | 품목기준코드 (9자리) | 다른 API 조인 키 |
| `ITEM_NAME` | 제품명 (한글) | 표시 |
| `ENTP_NAME` | 제조사명 | 표시·식별 |
| `CHART` | 성상 (자유텍스트) | 예: "적갈색의 타원형 필름코팅정" |
| `ITEM_IMAGE` | 제품 이미지 URL | 시각 확인 |
| `PRINT_FRONT` / `PRINT_BACK` | 앞/뒤 각인 | 식별 key (예: "B-Z") |
| `DRUG_SHAPE` | 모양 | 타원형·원형 등 |
| `COLOR_CLASS1` / `COLOR_CLASS2` | 색상 | 식별 보조 |
| `LINE_FRONT` / `LINE_BACK` | 분할선 유무 | 식별 보조 |
| `LENG_LONG` / `LENG_SHORT` / `THICK` | 장축·단축·두께 (mm) | 식별 보조 |
| `CLASS_NO` / `CLASS_NAME` | 분류코드·명 | 예: "[03190]기타의 비타민제" |
| `ETC_OTC_NAME` | **전문/일반 구분** | "일반의약품" / "전문의약품" (§8 처리 분기) |
| `FORM_CODE_NAME` | 제형 | "필름코팅정" 등 |
| `ITEM_PERMIT_DATE` | 허가일 (YYYYMMDD) | 리뉴얼 추적 |

### 3-3. 용도 시나리오
- **사진 기반 역추적**: 사용자가 알약 사진만 제시 → OCR/시각 파싱으로 각인 추출 → `print_front` 쿼리로 후보 압축
- **라벨 정보 불완전 시**: 제품명만 알고 정확한 버전 모를 때, 모양·색상으로 좁히기
- **2개 이상 제형 존재 시**: 같은 성분의 정제/캡슐/시럽 구분

---

## 4. Service 2 — 의약품 제품 허가정보 ★핵심

### 4-1. 엔드포인트
```
GET https://apis.data.go.kr/1471000/DrugPrdtPrmsnInfoService07/getDrugPrdtPrmsnInq07
  ?serviceKey={KEY}
  &type=json
  &pageNo=1
  &numOfRows=10
  &item_name={제품명}         # 부분 일치
  &entp_name={제조사명}       # 선택
  &item_seq={품목기준코드}    # 정확 일치
  &edi_code={EDI코드}         # 선택 (보험약가코드)
```

> ⚠️ **operation 이름 주의**: `getDrugPrdtPrmsnInq07` (Dtl**Inq** 아님. 2026-04-17 테스트에서 `getDrugPrdtPrmsnDtlInq05`는 404).

### 4-2. 주요 응답 필드
| 필드 | 의미 | 활용 |
|------|------|------|
| `ITEM_SEQ` | 품목기준코드 | 낱알식별/e약은요 조인 키 |
| `ITEM_NAME` / `ITEM_ENG_NAME` | 제품명 한/영 | 표시·해외 보고서 대응 |
| `ENTP_NAME` / `ENTP_ENG_NAME` | 제조사 한/영 | 표시 |
| `ITEM_PERMIT_DATE` | 허가일 | 리뉴얼 추적 |
| `INDUTY` | 업종 | 대부분 "의약품" |
| `PRDLST_STDR_CODE` | 품목코드 | 식약처 내부 참조 |
| `SPCLTY_PBLC` | **전문/일반 구분** | "일반의약품" / "전문의약품" |
| `PRDUCT_TYPE` | 약효분류 | 예: "[03190]기타의 비타민제" |
| `PRDUCT_PRMISN_NO` | 허가번호 | 공식 참조 |
| `ITEM_INGR_NAME` | **★ 전 성분 리스트 (영문·라틴명, `/` 구분)** | `parse-drug-response.md` 정규화 |
| `ITEM_INGR_CNT` | 성분 개수 | 검증 |
| `BIG_PRDT_IMG_URL` | 대표 이미지 URL | 시각 확인 |
| `PERMIT_KIND_CODE` | 허가 종류 | "신고" / "허가" 등 |
| `CANCEL_DATE` / `CANCEL_NAME` | 취소일/상태 | "정상" 확인 — 취소된 품목 분석 금지 |
| `EDI_CODE` | 보험약가코드 | 전문약 연동 시 |
| `BIZRNO` | 사업자번호 | 참고 |

### 4-3. `ITEM_INGR_NAME` 파싱 규칙
원본 예시 (비맥스제트정 ITEM_SEQ=202300436):
```
Ascorbic Acid Coated/Benfotiamine/Biotin/Bisbentiamine/Calcium Pantothenate/
Cholecalciferol Concentrate Powder/Choline Tartrate/Chromium in Dried Yeast/
Folic Acid/Gamma-Oryzanol/Inositol/Magnesium Oxide/Mecobalamin/Nicotinamide/
Pyridoxal Phosphate Hydrate/Riboflavin Butyrate/Selenium In Dried Yeast/Taurine/
Tocopherol Acetate Powder 50%/Ursodeoxycholic Acid/Zinc Oxide
```
- 구분자: `/`
- **함량 정보 없음** — 허가정보는 성분명만 제공. 함량은 `e약은요` 또는 라벨/제품설명서 필요
- 정규화: `prompts/parse-drug-response.md` 라틴→한국어 매핑 적용

### 4-4. 검색 결과 개수별 처리
| 결과 수 | 동작 |
|---------|------|
| 0건 | 해외 의약품 가능성 — 기존 Fallback Chain(Perplexity) 이동 |
| 1건 | 즉시 확정 |
| 2~5건 | 리스트 제시 + 선택 요청 |
| 6건 이상 | **좁히기 질문** — `entp_name` → 제형 → 허가일 순 |

---

## 5. Service 3 — 의약품개요정보 (e약은요) ★환자용 요약

### 5-1. 엔드포인트
```
GET https://apis.data.go.kr/1471000/DrbEasyDrugInfoService/getDrbEasyDrugList
  ?serviceKey={KEY}
  &type=json
  &pageNo=1
  &numOfRows=10
  &itemName={제품명}          # ⚠️ 카멜케이스
  &entpName={제조사명}        # 선택, 카멜케이스
  &itemSeq={품목기준코드}     # 정확 일치
```

> ⚠️ **파라미터 naming 주의**: 본 API는 `itemName`/`entpName` (**camelCase**). 낱알식별(§3)과 허가정보(§4)의 `item_name`/`entp_name` (**snake_case**)과 다름. 2026-04-17 테스트에서 `item_name`으로 호출 시 전체 목록(4711건) 반환 — 필터 무시됨.

### 5-2. 주요 응답 필드 (전부 한국어 평이체)
| 필드 | 의미 | 활용 |
|------|------|------|
| `itemSeq` | 품목기준코드 | 조인 키 |
| `itemName` / `entpName` | 제품명 / 제조사 | 표시 |
| `efcyQesitm` | **이 약의 효능은 무엇입니까?** (적응증) | 리포트 "복용 목적" 섹션 |
| `useMethodQesitm` | **이 약은 어떻게 사용합니까?** (용법·용량) | 정확한 복용량 확정 |
| `atpnWarnQesitm` | **이 약을 사용하기 전에 반드시 알아야 할 내용은?** | 금기·경고 |
| `atpnQesitm` | **이 약의 사용상의 주의사항은?** | 주의 |
| `intrcQesitm` | **이 약을 사용하는 동안 주의해야 할 약 또는 음식은?** (상호작용) | ★ 보충제 교차체크 |
| `seQesitm` | **이 약은 어떤 이상반응이 나타날 수 있습니까?** | 참고 |
| `depositMethodQesitm` | **이 약은 어떻게 보관해야 합니까?** | 참고 |
| `openDe` | 공개일자 | 참고 |
| `updateDe` | 갱신일자 | 신뢰도 판정 |

### 5-3. 예시 응답 (비맥스제트정)
```json
{
  "entpName": "(주)한풍제약",
  "itemName": "비맥스제트정",
  "itemSeq": "202300436",
  "efcyQesitm": "이 약은 육체피로, 임신∙수유기, 병중∙병후의 체력저하시... 비타민 D, E, B1, B2, B6, C의 보급과 뼈, 이의 발육불량, 구루병의 예방 및 신경통, 근육통, 관절통(요통, 어깨결림 등), 구각염, 구순염, 구내염, 설염, 습진, 피부염의 완화, 각기, 눈의피로, 아연의 보급에 사용합니다.",
  "useMethodQesitm": "만 19세 이상 및 성인은 1일 1회, 1회 1정 식후 복용합니다."
}
```

### 5-4. 리포트 인용 규칙
- `efcyQesitm`·`useMethodQesitm`은 식약처 공식 문구 → **가공 없이 인용 가능** (인용 부호 + "식약처 공식 문구" 명시)
- `intrcQesitm`에 명시된 상호작용은 `interaction-patterns.md`와 병기. 중복되면 식약처 문구 우선

---

## 6. 3개 API 조합 활용 패턴

### 6-1. 제품명만 아는 경우 (가장 일반)
```
1. DrugPrdtPrmsnInfoService07 조회 → ITEM_INGR_NAME 전 성분 + SPCLTY_PBLC 확인
2. ITEM_SEQ 확보 → DrbEasyDrugInfoService 조회 → 효능·용법·주의·상호작용 획득
3. 보충제 스택과 교차검증 (parse-drug-response.md의 RDA 매칭)
```

### 6-2. 사진만 있는 경우
```
1. 사진 OCR → 각인·색상·모양 추출
2. MdcinGrnIdntfcInfoService03 조회 (print_front·drug_shape·color_class1 조합) → ITEM_SEQ 역추적
3. 이후 6-1과 동일
```

### 6-3. 전문약으로 판정된 경우 (`SPCLTY_PBLC == "전문의약품"`)
- 상호작용·금기 정보만 리포트에 반영
- 복용량·대체 제안 섹션 **자동 생략**
- 최상단에 "전문의약품 — 의사/약사 상담 필수" 배너 의무

---

## 7. 캐싱 전략

### 7-1. 제품 DB 파일
```
user-data/drugs/{drug_key}.json
```
- 건기식 `products/`와 **별도 디렉터리** (성분 구조·사용 규칙이 다름)
- `drug_key` 생성: `{entp_slug}_{item_slug}_{item_seq}` (중복 방지)

### 7-2. 저장 시점
- `DrugPrdtPrmsnInfoService07` 조회 성공 + 사용자 확정 후
- `DrbEasyDrugInfoService` 필드는 같은 파일에 병합 저장

### 7-3. 재조회 조건
- `verified_at` 365일 초과 → 재조회
- 사용자가 "리뉴얼된 것 같다" 언급 → 재조회
- `CANCEL_NAME != "정상"` → 재조회 + 경고

---

## 8. 의약품 분류별 처리 분기 (★확정 규칙)

| `SPCLTY_PBLC` | 리포트 처리 |
|---------------|-------------|
| "일반의약품" | 성분·효능·상호작용 요약. 보충제 스택과 중복 체크. **"의사/약사 상담" 디스클레이머 필수** |
| "전문의약품" | 식별·성분·상호작용까지만. **복용량·대체 제안 섹션 생략**. 상단 배너 의무 |
| "한약(생약)제제" | 일반의약품 기준 적용. 추가로 "한약재-보충제 상호작용 근거 부족 가능" 고지 |
| `null`/공란 | 낱알식별 `ETC_OTC_NAME`로 보강. 그래도 불명이면 전문약 기준으로 보수적 처리 |

### 8-1. 리포트 생략 섹션 (전문의약품 시)
- {{ALTERNATIVE_PRODUCTS_SECTION}} — 대체 제안 금지
- {{STACK_OPTIMIZATION_SECTION}}의 의약품 관련 줄 — 증감 지시 금지
- {{DOSING_SECTION}} — 의사 영역

---

## 9. 에러 처리

| 응답 | 원인 | 대응 |
|------|------|------|
| `Forbidden` | 키 승인 대기 (1~2h) / 미승인 API | 재시도 1시간 후 / 포털에서 활용승인 확인 |
| `API not found` | ServiceID 오타 / 버전 불일치 | §3/§4/§5 endpoint 정확히 확인 |
| `resultCode: "00"` + `totalCount: 0` | 정상 응답 0건 | 해외 약 가능성 → Fallback |
| `resultCode: "99"` | 시스템 오류 | 지수 백오프 3회 |
| `totalCount` 이상 큰 값 (수천 건) | **필터 파라미터 무시** — 파라미터명 오타 | camelCase/snake_case 확인 (§5-1 경고) |

---

## 10. 보안·프라이버시

- 응답 원문은 로컬 캐시(`drugs/`)에만. 외부 전송 금지
- 의약품 데이터는 건강 민감정보 → 사용자가 분석 요청한 본인/가족 것만 저장
- 키 노출 점검: `grep -rE "[a-f0-9]{20,}" vitamin-analyzer/ --exclude-dir=.git`

---

## 11. 설치 (사용자용)

### 11-1. 키 발급
1. <https://data.go.kr> 로그인
2. 각각 활용 신청 (3개):
   - [의약품 낱알식별 정보](https://www.data.go.kr/data/15057639/openapi.do)
   - [의약품 제품 허가정보](https://www.data.go.kr/data/15095677/openapi.do)
   - [의약품개요정보(e약은요)](https://www.data.go.kr/data/15075057/openapi.do)
3. 이미 `KFDA_HTFS_KEY`가 있으면 **같은 키 재사용 가능**. 별도 설정 불필요

### 11-2. 환경변수
```bash
# 기존 키 재사용
export KFDA_HTFS_KEY="..."

# 별도 관리 희망 시 (선택)
export KFDA_DRUG_KEY="..."
```

### 11-3. 미설정 시
- 의약품 분석 기능만 자동 비활성
- 리포트에 "의약품 API 미연동 — 건기식 분석만 수행" 안내 1회

---

## 12. 참고

- `kfda-htfs-api.md` — 건기식 API (C003/I2710 등) 자매 문서
- `prompts/parse-drug-response.md` — 응답 필드 파싱·정규화 상세
- `prompts/report-template.md` §8-A — 의약품 리포트 금지사항
- data.go.kr 공공데이터포털 약관 — 2차 가공 허용, 출처 표시 필수
