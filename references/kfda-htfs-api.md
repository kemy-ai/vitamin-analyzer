# 식약처 건강기능식품 Open API — 통합 참조

> 한국 건강기능식품 제품 식별·성분 확인의 **1차 근거**. 사용자가 제품명만 제시한 경우 이 API가 최우선. 라벨 사진이 있으면 교차검증용.

---

## 1. 전제

### 1-1. 왜 이 API가 최우선인가
- **출처 신뢰도**: 식품의약품안전처 공식 DB (정부 인증 제품만 등록)
- **데이터 완전성**: C003 STDR_STND 필드에 성분명 + 함량 + 단위 구조화 텍스트
- **커버리지**: 국내 품목제조신고된 건강기능식품 전수 (약 6만 건+)
- **안정성**: 크롤링 차단·안티봇·JS 렌더링 이슈 없음

### 1-2. 한계
- **국내 제품만**: 해외 브랜드(iHerb·Now Foods·Thorne 등)는 등록 안 됨 → 해외 제품은 기존 Fallback Chain 후순위 유지
- **전파 지연**: data.go.kr 승인 직후 최대 2시간 API 호출 실패 가능
- **리뉴얼 반영 지연**: 제조사가 성분 변경 후 신고 갱신까지 시차 → 라벨 사진이 있으면 라벨 값 우선

---

## 2. API 4종 개요

| # | 서비스명 | 서비스 ID | 호스트 | 용도 |
|---|----------|-----------|--------|------|
| 1 | 건강기능식품 정보 | `HtfsInfoService03` | `apis.data.go.kr` (REST) | 제품 기본 정보 (제조사·제품명·신고번호) |
| 2 | 품목제조 신고사항 현황 | `I0030` | `openapi.foodsafetykorea.go.kr` (LINK) | 신고 상세 정보 |
| 3 | 품목제조 신고(원재료) | `C003` | `openapi.foodsafetykorea.go.kr` (LINK) | **★ 핵심 — 성분명·함량·용법** |
| 4 | 품목분류 정보 | `I2710` | `openapi.foodsafetykorea.go.kr` (LINK) | 원료별 1일 섭취량 기준 (522종) |

### 2-1. 키 관리 (절대 준수)
- 환경변수명: `KFDA_HTFS_KEY` (HtfsInfoService03용, 64자 hex), `KFDA_FOODSAFETY_KEY` (C003/I0030/I2710용, 20자)
- 스킬 파일·문서·예시 어디에도 **실제 키 하드코딩 금지**
- 호출 전 env 존재 확인 → 없으면 "KFDA 연동 미설정" 배너 + 다른 Fallback 단계로 진행
- 로컬 테스트용 `.kfda-test-keys` 파일은 `.gitignore` + `chmod 600`
- 외부 공유·커밋·스킬 업로드 전 `grep -r "[0-9a-f]\{20,\}" .` 로 누출 점검

### 2-2. 호출 공통 규칙
- **전부 GET 요청**. 응답 포맷 `json` 권장 (`xml` 지원하지만 파싱 번거로움)
- 쿼리 파라미터 `type=json` 명시 (HtfsInfoService03)
- 1일 호출 한도: 1,000건/개발, 10,000건/운영 (data.go.kr 기본값 — 포털에서 변경)
- 타임아웃: **30초 필수**. Rate limit 감지 시 지수 백오프(2^n 초, 최대 3회)

---

## 3. Service 1 — HtfsInfoService03 (건강기능식품 정보)

### 3-1. 엔드포인트
```
GET https://apis.data.go.kr/1471000/HtfsInfoService03/getHtfsItem01
  ?serviceKey={KFDA_HTFS_KEY}
  &type=json
  &pageNo=1
  &numOfRows=10
  &prduct={제품명}       # 부분 일치
  &entrps={제조사명}     # 부분 일치, 선택
```

### 3-2. serviceKey 포맷
- data.go.kr 발급 키는 **Encoding 탭의 URL-encoded 값** 사용 (`%2F`, `%2B` 포함)
- 코드에서 `requests.get(params={...})` 사용 시 자동 재인코딩 방지 필요 → URL에 직접 삽입 또는 `urllib.parse.quote` 회피

### 3-3. 주요 응답 필드
| 필드 | 의미 |
|------|------|
| `PRDUCT` | 제품명 |
| `ENTRPS` | 제조사명 |
| `STTEMNT_NO` | 품목제조신고번호 (C003 조회 키) |
| `REGIST_DT` | 신고일자 |
| `DISTB_PD` | 유통기한 표시 방법 |

### 3-4. 용도
- 1차 제품 후보 검색
- 결과의 `STTEMNT_NO`를 Service 3 (C003)에 재질의하여 성분 확정

---

## 4. Service 3 — C003 (품목제조 신고 원재료) ★핵심

### 4-1. 엔드포인트
```
GET http://openapi.foodsafetykorea.go.kr/api/{KFDA_FOODSAFETY_KEY}/C003/json/{START_IDX}/{END_IDX}[/{FIELD}={VALUE}]
```

- `START_IDX/END_IDX`: 페이징 (1~1000 등)
- `/{FIELD}={VALUE}`: URL 경로 뒤에 필터 추가 (쿼리스트링 아님, **경로 세그먼트**)
  - 예: `/C003/json/1/10/PRDLST_NM=종근당`

### 4-2. 필터 가능 필드
| 필드 | 의미 | 매칭 |
|------|------|------|
| `PRDLST_NM` | 품목제조 신고 제품명 | 부분 일치 |
| `BSSH_NM` | 영업소(제조사)명 | 부분 일치 |
| `PRDLST_REPORT_NO` | 품목제조신고번호 | 정확 일치 |
| `LCNS_NO` | 영업허가번호 | 정확 일치 |

### 4-3. 주요 응답 필드
| 필드 | 의미 | 파싱 대상 |
|------|------|----------|
| `PRDLST_REPORT_NO` | 품목제조신고번호 (고유 ID) | DB 키로 사용 |
| `PRDLST_NM` | 제품명 | 표시용 |
| `BSSH_NM` | 제조사명 | 표시용 |
| `PRDT_SHAP_CD_NM` | 제형 (정제·캡슐·액상 등) | 복용단위 판정 |
| `STDR_STND` | **★ 기준·규격 (성분·함량 자유텍스트)** | `parse-kfda-response.md` 정규식 적용 |
| `RAWMTRL_NM` | 원재료명 (콤마 구분 리스트) | 성분 목록 교차검증 |
| `NTK_MTHD` | 섭취방법 (용법·용량 자유텍스트) | 1일 복용량 추출 |
| `PRIMARY_FNCLTY` | 주된 기능성 (자유텍스트) | 표시용·참고 |
| `IFTKN_ATNT_MATR_CN` | 섭취 시 주의사항 | 리포트에 경고 병기 |
| `POG_DAYCNT` | 유통기간 | 참고 |

### 4-4. 검색 결과 개수별 처리 (★확정 규칙)

| 결과 수 | 동작 |
|---------|------|
| 0건 | "식약처 DB에 미등록 — 해외 제품이거나 신고번호 확인 필요". 다음 Fallback 단계로 |
| 1건 | 즉시 확정. 사용자에게 "이 제품 맞나요?" 확인 1회 |
| **2~5건** | **리스트로 보여주고 선택 요청** |
| **6건 이상** | **먼저 좁히기 질문** — 제조사명(`BSSH_NM`), 제형(`PRDT_SHAP_CD_NM`), 1정당 함량, 구매 시기 순. 5건 이하로 줄어들면 리스트 제시 |

### 4-5. 좁히기 질문 순서 (우선순위)
1. **제조사명** — 가장 효과적. `BSSH_NM=종근당` → 240+건 → `종근당 + 칼맥디` 교차 10건대
2. **제형** — 정제·캡슐·액상·분말·젤리 등. `PRDT_SHAP_CD_NM` 필터
3. **1정당 주요 성분 함량** — 예: "비타민 D 1000IU인지 2000IU인지"
4. **구매 시기** — 리뉴얼 판단용. `REGIST_DT` 연도 비교

---

## 5. Service 2 — I0030 (품목제조 신고사항 현황)

### 5-1. 엔드포인트
```
GET http://openapi.foodsafetykorea.go.kr/api/{KFDA_FOODSAFETY_KEY}/I0030/json/{START}/{END}[/{FIELD}={VALUE}]
```

### 5-2. 용도
- C003와 상당 부분 중복. **기본적으로 C003 우선** 사용
- 신고 이력·상태(정상·취소·변경) 확인이 필요한 경우에만 보조

---

## 6. Service 4 — I2710 (품목분류 정보) ★ 한국 섭취 기준

### 6-1. 엔드포인트
```
GET http://openapi.foodsafetykorea.go.kr/api/{KFDA_FOODSAFETY_KEY}/I2710/json/{START}/{END}[/{FIELD}={VALUE}]
```

### 6-2. 주요 응답 필드
| 필드 | 의미 |
|------|------|
| `PRDCT_NM` | 원료명 (한글) |
| `DAY_INTK_LOWLIMIT` | 1일 섭취량 하한 |
| `DAY_INTK_HIGHLIMIT` | 1일 섭취량 상한 |
| `INTK_UNIT` | 단위 (mg·µg·g 등) |
| `FNCLTY_CN` | 기능성 내용 |

### 6-3. 용도
- **한국 공식 섭취 기준** — NIH RDA와 다른 경우 병기
- 식약처 UL 해당 기준 (NIH UL과 수치 다를 수 있음)
- 선별 결과는 `references/korean-intake-limits.md`에 28영양소 큐레이션

---

## 7. 캐싱 전략

### 7-1. 제품 DB 파일
```
user-data/products/{product_key}.json
```
- `product_key` 생성 규칙: `{brand_slug}_{product_slug}` (영문 소문자 + 숫자 + `_`만)
- KFDA 원본에는 `PRDLST_REPORT_NO`(신고번호)도 필드로 저장 → 재조회 키

### 7-2. 재조회 조건
| 조건 | 재조회? |
|------|---------|
| `verified_at` 365일 이내 | 사용 (캐시 hit) |
| `verified_at` 365일 초과 | 강제 재조회 |
| `source.confidence: "low"` | 항상 재조회 |
| 사용자가 "리뉴얼된 것 같다" 언급 | 강제 재조회 |
| 사용자가 새 라벨 사진 제공 | 강제 재조회 (라벨이 우선) |

### 7-3. 저장 시점
- C003 조회 성공 직후 저장. 사용자가 "이 제품 맞아"로 확정한 후에만
- 여러 후보 중 선택된 것만 저장 (탈락 후보는 저장 안 함)

---

## 8. 라벨 사진 vs API 교차검증 (★확정 규칙)

### 8-1. 2-Track 절차
1. **Track A (OCR)**: 라벨 사진 → `parse-image.md` → 성분·함량 JSON 추출
2. **Track B (API)**: 제품명·제조사명으로 C003 조회 → STDR_STND·RAWMTRL_NM 파싱
3. 두 결과 비교

### 8-2. 충돌 처리 (라벨 우선)
| 차이 유형 | 처리 |
|-----------|------|
| 성분 함량 동일 | ✅ 확정. `confidence: "high"` |
| **성분 함량 10% 이상 차이** | **라벨 값 채택**. 리포트에 "식약처 등록 스펙과 차이 — 리뉴얼 또는 다른 버전 가능성" 정보성 병기 |
| 라벨에 있는 성분이 API에 없음 | 라벨 기준. 신고 누락 가능성 병기 |
| API에 있는 성분이 라벨에 없음 | 라벨 기준 (소비자가 실제 복용하는 건 라벨). API 차이 병기 |
| 제형·용법 불일치 | 라벨 우선 |

### 8-3. 원칙
> 라벨 = 소비자가 실제 복용하는 물리적 제품. API = 정부 등록부 스냅샷. 리뉴얼·OEM·회차 차이 시 라벨이 ground truth.

---

## 9. 에러 처리

| HTTP/응답 | 원인 | 대응 |
|-----------|------|------|
| 403 Forbidden | 키 승인 대기(신청 직후 1~2h) / 키 오기입 | 재시도 1시간 후 / env 값 grep 확인 |
| 404 Not Found | 엔드포인트 오탈자 | 본 문서 §3/§4/§5/§6 정확히 확인 |
| 응답 `RESULT.CODE: "INFO-200"` | 검색 결과 0건 | 후속 Fallback 단계로 이동 |
| 응답 `RESULT.CODE: "ERROR-300"` | 키 유효하지 않음 | env 교체 후 재시도 |
| 타임아웃 | 네트워크 / 서버 일시 장애 | 지수 백오프 3회. 전부 실패 시 후속 Fallback |
| Rate limit | 한도 초과 | 재시도 금지. 사용자에게 "내일 다시" or 해외 fallback |

---

## 10. 보안·프라이버시

- API 응답 원문은 로컬 캐시(`products/`)에만. 외부 전송 금지
- 키 노출 점검 명령:
  ```
  grep -rE "[a-f0-9]{20,}" vitamin-analyzer/ --exclude-dir=.git
  grep -r "KFDA_HTFS_KEY=" vitamin-analyzer/ --exclude-dir=.git  # 변수명만 나와야 함
  ```
- 키 재발급: 통합 테스트 완료 후 data.go.kr + foodsafetykorea.go.kr 양쪽에서 전부 재발급 권장

---

## 11. 설치·설정 (사용자용)

### 11-1. 키 발급 (선택 — 한국 제품 분석 품질 향상)
1. <https://data.go.kr> 회원가입 → "건강기능식품 정보" 검색 → 활용 신청
2. <https://openapi.foodsafetykorea.go.kr> 별도 회원가입 → C003/I0030/I2710 각각 신청
3. 승인 후 1~2시간 대기

### 11-2. 환경변수 설정
```bash
# ~/.zshrc 또는 .env
export KFDA_HTFS_KEY="..."          # data.go.kr 64자 hex
export KFDA_FOODSAFETY_KEY="..."    # foodsafetykorea 20자
```

### 11-3. 미설정 시
- 키 없어도 스킬 동작. 한국 제품은 기존 Fallback Chain(공식몰→쇼핑몰) 우선
- 리포트 상단에 "KFDA 미연동 — 설치 가이드 참조" 배너 1회 표시
