# Interaction Patterns — 보충제 상호작용 20종

> **용도**: 보충제↔처방약, 보충제↔보충제, 보충제↔음식 상호작용 lookup.
> **적용**: 프로필에 약물·기저질환 입력되면 해당 패턴 자동 매칭, 리포트 상단에 경고.
> **원칙**: 처방약 용량 조정·중단 권고 **절대 금지**. 반드시 "의사·약사 상담 권고" 고정 문구.

---

## ⚠️ URL 검증 경고

본 파일의 출처 URL은 **알려진 도메인 패턴 기반**이며, 구체 PubMed PMID는 포함하지 않습니다 (환각 방지). **세션 3 테스트에서 WebFetch로 접근 확인 후 실제 원저 PMID를 추가**해야 합니다. 현재 URL은 공식 팩트시트·검색 엔드포인트 수준.

---

## 1. 임상 심각도 등급

| 등급 | 의미 | 리포트 색상 |
|------|------|------------|
| 🔴 **심각 (Major)** | 생명 위협 or 중대 부작용 위험. 병용 금기 또는 면밀한 모니터링 필수 | 빨강 |
| 🟠 **중등도 (Moderate)** | 임상적으로 의미 있는 변화. 용량 조정·간격 분리·주기 모니터링 | 주황 |
| 🟡 **경미 (Minor)** | 이론적/약한 영향. 일반인은 실사용 문제 드물지만 특정 프로필 주의 | 노랑 |

---

## 2. 카테고리 A — 보충제 ↔ 처방약 (12개)

### 1. 🔴 Vitamin K ↔ Warfarin (와파린)
- **메커니즘**: 와파린은 비타민 K 의존성 응고인자(II, VII, IX, X)를 억제. 비타민 K 섭취 변동은 항응고 효과를 직접 상쇄
- **임상 의미**: INR(국제표준화비율) 변동 → 출혈 또는 혈전 위험. K1 갑자기 증가 시 INR↓, 감소 시 INR↑
- **대응**:
 - 와파린 복용자는 **비타민 K 섭취량 일관성 유지**가 중요 (금기보다 "변동 금지")
 - 비타민 K 보충제 **추가·중단 시 의사 상담 필수**
 - 녹색채소 섭취도 일정하게 유지
- **출처**:
 - NIH ODS Vitamin K Factsheet: <https://ods.od.nih.gov/factsheets/VitaminK-HealthProfessional/>
 - Drugs.com: <https://www.drugs.com/drug-interactions/vitamin-k-phytonadione-with-warfarin-2338-2153-2311-0.html> ⚠️검증

### 2. 🔴 Silymarin (밀크씨슬) ↔ CYP3A4/2C9 기질 약물
- **메커니즘**: 실리마린(flavonolignans)은 시험관·동물실험에서 CYP3A4·2C9·P-gp 억제. 인간 임상 데이터는 혼재되어 있으나 일부 기질 약물의 혈중 농도 변동 가능
- **영향 약물**: 스타틴(심바스타틴, 아토르바스타틴), 벤조디아제핀(미다졸람), 일부 항암제, 와파린, 일부 면역억제제
- **임상 의미**: 약물 농도↑ 또는 ↓로 효과·부작용 변화. 임상 중요도는 연구마다 상이하나 고위험군(간기능 저하·다약제 복용)은 주의
- **대응**:
 - 관련 약물 복용자는 **실리마린 복용 전 의사·약사 상담**
 - 용량 모니터링이 필요한 약물(와파린, 일부 항경련제) 병용 시 혈액검사 권장
- **출처**:
 - NIH NCCIH Milk Thistle: <https://www.nccih.nih.gov/health/milk-thistle>
 - PubMed 검색: <https://pubmed.ncbi.nlm.nih.gov/?term=silymarin+CYP3A4+pharmacokinetic>

### 3. 🔴 St. John's Wort (세인트존스워트) → CYP3A4 강력 유도
- **메커니즘**: Hyperforin 성분이 pregnane X receptor 활성화 → CYP3A4·P-gp 강력 유도 → 약물 대사 촉진 → 혈중 농도 저하
- **영향 약물**: 경구피임약(실패 보고), SSRI·SNRI(세로토닌 증후군 위험 추가), 면역억제제(시클로스포린·타크로리무스), HIV 프로테아제 억제제, 일부 항암제, 와파린, 디곡신
- **임상 의미**: 병용 시 약물 효과 **큰 폭 감소 → 치료 실패**. 경구피임약 실패 → 원치 않은 임신 사례 보고됨
- **대응**:
 - **상기 약물 복용자 병용 금기** 수준
 - 복용자에게는 "처방약 복용 중이면 세인트존스워트 중단" 강력 권고
 - SSRI 병용 시 세로토닌 증후군 위험 별도 경고
- **출처**:
 - NIH NCCIH St. John's Wort: <https://www.nccih.nih.gov/health/st-johns-wort>
 - NIH ODS Drug Interactions: <https://ods.od.nih.gov/factsheets/HerbalSupplements-HealthProfessional/>

### 4. 🔴 Ginkgo Biloba (은행잎) ↔ 항응고제·항혈소판제
- **메커니즘**: 은행잎의 ginkgolide B는 platelet-activating factor(PAF) 길항 → 혈소판 응집 억제
- **영향 약물**: 와파린, 아스피린, 클로피도그렐(플라빅스), 리바록사반·아픽사반 등 DOAC, NSAID
- **임상 의미**: 출혈 위험 증가. 지주막하 출혈 사례 보고
- **대응**:
 - 수술 **최소 2주 전 은행잎 중단**
 - 항응고·항혈소판제 복용자 병용 주의, 의사 상담
- **출처**:
 - NIH NCCIH Ginkgo: <https://www.nccih.nih.gov/health/ginkgo>
 - PubMed: <https://pubmed.ncbi.nlm.nih.gov/?term=ginkgo+bleeding+warfarin>

### 5. 🟠 Calcium ↔ 퀴놀론·테트라사이클린 항생제
- **메커니즘**: Ca²⁺가 항생제와 chelate(킬레이트) 형성 → 위장관 흡수 저하
- **영향 약물**: 시프로플록사신, 레보플록사신, 독시사이클린, 미노사이클린
- **임상 의미**: 항생제 효과 저하 → 치료 실패 가능
- **대응**: **항생제 복용 전 2시간 / 후 6시간 이내 칼슘 섭취 피하기**
- **출처**:
 - NIH ODS Calcium: <https://ods.od.nih.gov/factsheets/Calcium-HealthProfessional/>
 - Drugs.com: <https://www.drugs.com/drug-interactions/calcium.html>

### 6. 🟠 Magnesium ↔ 비스포스포네이트·레보티록신
- **메커니즘**: Mg²⁺가 약물과 복합체 형성 → 흡수 저하
- **영향 약물**: 알렌드로네이트(포사맥스), 이반드로네이트 등 비스포스포네이트; 레보티록신(신지로이드)
- **임상 의미**: 골다공증·갑상선약 효과 감소
- **대응**:
 - 비스포스포네이트: 복용 후 **최소 30분~2시간** 간격
 - 레보티록신: 복용 후 **최소 4시간** 간격
- **출처**:
 - NIH ODS Magnesium: <https://ods.od.nih.gov/factsheets/Magnesium-HealthProfessional/>

### 7. 🟠 Iron ↔ 레보티록신·퀴놀론
- **메커니즘**: Fe²⁺·Fe³⁺가 약물과 킬레이트 형성
- **영향 약물**: 레보티록신, 시프로플록사신·레보플록사신, 미코페놀레이트
- **임상 의미**: 약물 흡수 큰 폭 감소
- **대응**:
 - 레보티록신: **최소 4시간 간격**
 - 퀴놀론: **복용 전 2시간 / 후 6시간** 간격
- **출처**:
 - NIH ODS Iron: <https://ods.od.nih.gov/factsheets/Iron-HealthProfessional/>

### 8. 🟠 Vitamin B6 고용량 ↔ Levodopa (레보도파, 파킨슨약)
- **메커니즘**: B6(pyridoxine)는 levodopa의 주변부 decarboxylase를 활성화 → 뇌 진입 전 분해 증가
- **영향**: 단독 levodopa 복용 시 효과 저하. 단, **carbidopa 병용(신지메트)** 시 이 상호작용은 거의 소실
- **임상 의미**: 순수 levodopa는 드묾 (현대 처방은 대부분 carbidopa 복합제)
- **대응**:
 - 파킨슨 환자 B6 고용량(>10 mg/day) 복용 전 의사 상담
 - 대부분의 복합제에서는 문제없음
- **출처**:
 - NIH ODS Vitamin B6: <https://ods.od.nih.gov/factsheets/VitaminB6-HealthProfessional/>

### 9. 🟠 CoQ10 ↔ Warfarin
- **메커니즘**: CoQ10 구조가 비타민 K와 유사 → 항응고 효과 저하 가능
- **임상 의미**: INR 감소 보고 있음 (경미~중등)
- **대응**:
 - 와파린 복용자 CoQ10 추가·중단 시 **INR 주기 모니터링**
 - 의사 상담 권고
- **출처**:
 - NIH ODS CoQ10: <https://ods.od.nih.gov/factsheets/CoenzymeQ10-HealthProfessional/>

### 10. 🟠 Omega-3 고용량 ↔ 항응고제·항혈소판제
- **메커니즘**: EPA·DHA가 thromboxane A2 생성 억제 → 혈소판 응집 감소
- **영향 약물**: 와파린, 아스피린, 클로피도그렐, DOAC
- **임상 의미**: EPA+DHA 3 g/day 이상에서 출혈 시간 연장. 임상 출혈 사건 증가는 근거 약하지만 고위험군 주의
- **대응**:
 - 3 g/day 이하 보충제 일반 섭취는 비교적 안전
 - 수술 전 7-14일 중단 권고
 - 항응고제 복용자는 의사 상담
- **출처**:
 - NIH ODS Omega-3: <https://ods.od.nih.gov/factsheets/Omega3FattyAcids-HealthProfessional/>

### 11. 🔴 Potassium ↔ ACE억제제·ARB·칼륨보존 이뇨제
- **메커니즘**: 이들 약물은 모두 신장 K⁺ 배설 감소 → 고칼륨혈증 위험
- **영향 약물**: 리시노프릴·에날라프릴 등 ACE억제제; 로사르탄·발사르탄 등 ARB; 스피로노락톤·에플레레논
- **임상 의미**: 고칼륨혈증 → 심부정맥·심정지. 특히 신기능 저하·당뇨 환자 위험
- **대응**:
 - **칼륨 보충제 병용 원칙적 금기** (의사 처방 하에서만)
 - 저염소금(K 함유) 과다 섭취 주의
- **출처**:
 - NIH ODS Potassium: <https://ods.od.nih.gov/factsheets/Potassium-HealthProfessional/>

### 12. 🟠 Ashwagandha ↔ 면역억제제·갑상선약·진정제
- **메커니즘**:
 - 면역조절 → 면역억제제 효과 길항
 - 갑상선 기능 자극 → 레보티록신 복용자 과잉
 - GABA 작용 → 진정제와 상가 효과
- **영향 약물**: 시클로스포린·타크로리무스, 레보티록신, 벤조디아제핀·알코올
- **임상 의미**: 이론적 근거 다수, 임상 사례 일부
- **대응**:
 - 해당 약물 복용자 복용 전 의사 상담
 - 임신·수유부 권장 안 함 (낙태 유도 이력)
- **출처**:
 - NIH NCCIH Ashwagandha: <https://www.nccih.nih.gov/health/ashwagandha>
 - PubMed: <https://pubmed.ncbi.nlm.nih.gov/?term=ashwagandha+interaction>

---

## 3. 카테고리 B — 보충제 ↔ 보충제 (5개)

### 13. 🟠 Calcium ↔ Iron (흡수 경쟁)
- **메커니즘**: Ca²⁺가 Fe²⁺·Fe³⁺ 수송체(DMT1) 경쟁 → Fe 흡수 저하 (고용량 Ca에서 최대 50-60% 저하 보고)
- **영향**: 빈혈 치료 중 철분제 효과 감소
- **대응**:
 - 철분제와 칼슘 **최소 2시간 간격**
 - 식사 중 Ca 300 mg 이하는 영향 적음
- **출처**: NIH ODS Iron — Interactions 섹션

### 14. 🟡 Calcium ↔ Magnesium (고용량 동시 시)
- **메커니즘**: 동일 흡수 경로 경쟁 (소장 투과성 채널)
- **임상 의미**: 저용량(<500 mg each)은 문제없음. 고용량 동시 복용 시 상호 흡수↓
- **대응**: 각 성분 **1회 500 mg 이하로 분할** 또는 시간 분리
- **출처**: NIH ODS Magnesium

### 15. 🟠 Zinc 고용량 장기 ↔ Copper 결핍
- **메커니즘**: Zn이 장세포 metallothionein 유도 → Cu를 세포 내로 격리 → 대변 배출
- **임상 의미**: Zn ≥50 mg/day 장기 복용 시 Cu 결핍 → 빈혈·신경병증. AREDS2(80 mg Zn) 장기 보충 시 Cu 2 mg 병용 권고된 이유
- **대응**:
 - Zn ≥40 mg/day 장기 복용 시 **Cu 1-2 mg/day 병용 권장**
 - 단기(<30일) 고용량은 문제없음
- **출처**: NIH ODS Zinc — Interactions

### 16. 🟡 Zinc ↔ Iron (흡수 경쟁)
- **메커니즘**: Zn²⁺·Fe²⁺ 모두 DMT1 경유 → 경쟁
- **임상 의미**: 공복 단독 복용 시 경쟁 뚜렷, 식사 중 일반적 용량은 경미
- **대응**: 보충제로 병용 시 **시간 분리 권장** (2시간 간격)
- **출처**: NIH ODS Zinc

### 17. 🟠 Vitamin E 고용량 ↔ Vitamin K (항응고 상가)
- **메커니즘**: Vit E 고용량(>400 IU)은 vit K 의존성 응고인자 합성 감소 가능
- **영향**: 와파린 병용 시 출혈 위험 증가, 비타민 K 길항 작용 강화
- **대응**:
 - 항응고제 복용자 비타민 E >400 IU 피하기
 - 일반인도 고용량 장기 복용 주의
- **출처**: NIH ODS Vitamin E

---

## 4. 카테고리 C — 보충제 ↔ 음식·물질 (3개)

### 18. 🟠 Iron ↔ 탄닌(차·커피)·칼슘 식품 (흡수 저하) / 비타민 C (흡수↑)
- **메커니즘**:
 - 탄닌·피틴산·폴리페놀이 non-heme Fe와 복합체 → 흡수 저하 (최대 60-70%)
 - Ca 식품·보충제: DMT1 경쟁
 - 비타민 C: Fe³⁺ → Fe²⁺ 환원, ascorbate-Fe 킬레이트 형성 → 흡수↑ (최대 3-4배)
- **임상 의미**: 빈혈 치료 중 식사 구성이 치료 성과에 큰 영향
- **대응**:
 - 철분제 복용 전후 **1시간 차·커피·우유 피하기**
 - **비타민 C 100-200 mg 또는 감귤류 동반** 권장
- **출처**: NIH ODS Iron — Absorption 섹션

### 19. 🟠 Folate 고용량 ↔ B12 결핍 마스킹
- **메커니즘**: Folate 고용량은 B12 결핍의 **혈액학적 증상(거대적아구성 빈혈)만 개선**. 신경학적 손상(말초신경병증·척수 변성)은 계속 진행
- **임상 의미**: B12 결핍이 조기 발견 안 되고 진행 → 비가역 신경 손상. 특히 노인·채식주의자
- **대응**:
 - Folate ≥1000 mcg/day 보충 시 **B12 상태 확인**(혈액검사) 권고
 - 보충제 구매 시 B12와 함께 있는 B-complex 선택 권장
- **출처**: NIH ODS Folate — Concerns 섹션

### 20. 🟠 Green Tea Extract (EGCG) 고용량 ↔ 간독성
- **메커니즘**: 고용량 EGCG(특히 공복)는 간세포에 산화 스트레스 유발 → 드물지만 급성 간독성·간부전 사례 보고 (Hepatotoxicity)
- **임상 의미**: EFSA 및 USP 경고. 녹차 음료는 안전하나 **농축 추출물 보충제**가 주 원인
- **대응**:
 - 성인 **EGCG 800 mg/day 이하** 권장 (EFSA)
 - 보충제는 **식사와 함께** 복용
 - 간질환 과거력·간독성 약물 병용자 회피
- **출처**:
 - NIH ODS Green Tea: <https://ods.od.nih.gov/factsheets/GreenTea-Consumer/>
 - EFSA Scientific Opinion on EGCG (2018)

---

## 5. 매칭 알고리즘 (Claude 참조용)

사용자 입력 분석 순서:

1. **프로필 파싱**: 복용 약물 리스트 추출
2. **각 처방약에 대해 §2 (카테고리 A) 12개 매칭**:
 - 약물명 정확 매칭 (예: "와파린", "warfarin", "쿠마딘")
 - 매칭 시 해당 항목 전체 리포트 상단 경고 삽입
3. **복수 보충제 입력 시 §3 (카테고리 B) 5개 매칭**:
 - 예: Ca + Fe + Zn 조합 → 13, 15, 16번 모두 트리거
4. **식이 습관(차·커피·채식 등) 입력 시 §4 (카테고리 C) 매칭**
5. **UL 근접(70%+)·초과 성분이 있으면**:
 - 해당 성분의 상호작용도 자동 확인 (예: Vit E 1000 mg 초과 → 17번)

---

## 6. 리포트 삽입 형식

```markdown
## ⚠️ 상호작용 경고

🔴 **심각** — [성분 A] × [약물/성분 B]
- 메커니즘: [요약]
- 대응: [간격/금기/모니터링]
- [출처: URL]

🟠 **중등도** — [...]

🟡 **경미** — [...]

> 처방약 복용 중이면 반드시 **의사·약사와 상담** 후 보충제 복용을 결정하세요.
```

---

## 7. 절대 금지

1. 본 파일에 없는 상호작용을 **추측으로 생성 금지** — "일반적으로 상호작용 가능" 식 표현 금지
2. 처방약의 **용량 조정·중단 권고 절대 금지** → "의사 상담 권고" 고정
3. **출처 URL 환각 금지** — 위 URL 목록에 있거나 WebFetch로 재확인된 URL만
4. **"자주 발생" "많은 환자에서" 같은 근거 없는 빈도 표현 금지** → 임상 연구 출처 있을 때만 수치 인용

---

## 8. 확장 정책 

새 상호작용 추가 조건:
- Tier 1 근거 1건 이상 (PubMed systematic review 또는 NIH ODS 팩트시트 언급)
- 실사용 빈도 or 안전 중요도 상
- 세션 3 테스트에서 실제 URL 검증 완료

