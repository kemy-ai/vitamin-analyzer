# Ingredient Aliases — 성분 동의어 & 단위 표준화

> **용도**: 원시 성분명(한/영/라틴/약어)을 canonical 영문명으로 정규화하고, 용량 단위를 표준화하는 참조 테이블.
> **범위**: 상위 100종 (비타민 13 + 미네랄 15 + 아미노산 10 + 지방산 4 + 식물추출물 20 + 프로바이오틱스·프리바이오틱스 10 + 기타 28 = 100).
> **정렬**: 카테고리순 → 카테고리 내 canonical_name 알파벳/중요도 순.
> **미매칭 시**: Claude 추론 fallback 사용 + 리포트에 `자동 매칭됨 (검증 필요)` 플래그 필수.

---

## 0. 단위 변환식 (필수 참조)

### 0-1. 비타민 A
- **1 IU Retinol** = 0.3 mcg RAE (Retinol Activity Equivalent)
- **1 IU Beta-carotene (식이)** = 0.05 mcg RAE
- **1 IU Beta-carotene (보충제)** = 0.15 mcg RAE
- **1 mcg RAE** ≈ 3.33 IU (retinol 기준)
- 공식: `mcg_RAE = IU × 0.3` (retinol 기준, 보충제 기본)

### 0-2. 비타민 D
- **1 IU Vitamin D** = 0.025 mcg
- **1 mcg Vitamin D** = 40 IU
- 공식: `mcg = IU × 0.025` / `IU = mcg × 40`
- 예: 5000 IU = 125 mcg (UL 100 mcg 초과 → 🔴 플래그)

### 0-3. 비타민 E
- **1 IU d-alpha-tocopherol (천연)** = 0.67 mg alpha-TE
- **1 IU dl-alpha-tocopherol (합성)** = 0.45 mg alpha-TE
- 공식(천연): `mg = IU × 0.67` / 공식(합성): `mg = IU × 0.45`

### 0-4. 나이아신 (Niacin, B3)
- **1 mg NE (Niacin Equivalent)** = 1 mg 나이아신 = 60 mg 트립토판
- 라벨 표기가 `mg`인 경우 대부분 NE 기준으로 간주
- **주의**: 니코틴산(nicotinic acid)과 니코틴아미드(nicotinamide)는 UL이 다름 — 니코틴산 UL 35 mg/day, 니코틴아미드 UL 900 mg/day (NIH)

### 0-5. 엽산 (Folate, B9)
- **1 mcg DFE (Dietary Folate Equivalent)** = 1 mcg 식이 엽산 = 0.6 mcg 보충제 엽산(식사와 함께) = 0.5 mcg 보충제 엽산(공복)
- 공식(보충제 공복): `mcg_DFE = mcg_보충제 × 2.0`
- 공식(보충제 식사 중): `mcg_DFE = mcg_보충제 × 1.7`
- 라벨 `mcg DFE` 우선, 없으면 `mcg`로 간주하되 플래그

### 0-6. 비타민 K
- **1 mcg Vitamin K1 (phylloquinone)** = 1 mcg Vitamin K2 (menaquinone, MK-7) 기능 동등 근사(NIH 단일 DRI)
- 단, 반감기 차이(K1 수시간 / MK-7 약 72시간)로 임상 반응은 다름 → 리포트에서 형태 병기

### 0-7. 오메가-3 (EPA + DHA)
- 라벨의 "Fish Oil 1000 mg" ≠ EPA+DHA 1000 mg. 반드시 EPA, DHA 개별 용량 확인
- 공식: `실제_EPA_DHA = 어유_총량 × (EPA%+DHA%)` — 보통 30~60%

### 0-8. 미네랄 (원소량 vs 화합물량)
- **"Calcium Carbonate 1250 mg"**에서 원소 Ca = 1250 × 0.40 = 500 mg
- 주요 변환 계수 (원소%):
  - Calcium Carbonate → Ca 40%
  - Calcium Citrate → Ca 21%
  - Magnesium Oxide → Mg 60%
  - Magnesium Citrate → Mg 16%
  - Magnesium Glycinate → Mg 14%
  - Iron Fumarate → Fe 33%
  - Iron Sulfate → Fe 20%
  - Iron Bisglycinate → Fe 20%
  - Zinc Gluconate → Zn 14%
  - Zinc Picolinate → Zn 20%
- 라벨이 원소량("Calcium 500 mg")으로 표시되었으면 변환 불필요

### 0-9. 정규화 원칙
1. 라벨이 원소량/활성형으로 표시되었으면 **그 값을 그대로 사용**
2. 화합물량만 있으면 위 계수로 환산 → 리포트에 "환산됨" 표시
3. 형태 불명(단순 "Iron 15 mg")은 가장 흔한 형태로 가정 + 플래그

---

## 1. 비타민 (13종)

| canonical_name | kor_aliases | eng_aliases | latin_aliases | forms | unit_default |
|---|---|---|---|---|---|
| vitamin_a | 비타민 A, 레티놀, 베타카로틴 | Vitamin A, Retinol, Beta-carotene | Retinol | retinyl palmitate, retinyl acetate, beta-carotene | mcg RAE |
| vitamin_d | 비타민 D, 비타민 D3, 콜레칼시페롤, 에르고칼시페롤 | Vitamin D, Vitamin D3, D2, Cholecalciferol, Ergocalciferol | Cholecalciferol, Ergocalciferol | D3 (cholecalciferol), D2 (ergocalciferol) | mcg |
| vitamin_e | 비타민 E, 토코페롤, d-알파 토코페롤, 혼합 토코페롤 | Vitamin E, Tocopherol, d-alpha-tocopherol, Mixed tocopherols | Tocopherol | d-alpha, dl-alpha, mixed tocopherols/tocotrienols | mg alpha-TE |
| vitamin_k | 비타민 K, 비타민 K1, 비타민 K2, 필로퀴논, 메나퀴논, MK-4, MK-7 | Vitamin K, K1, K2, Phylloquinone, Menaquinone | Phylloquinone, Menaquinone | K1, K2 (MK-4, MK-7) | mcg |
| thiamine | 티아민, 비타민 B1, B1 | Thiamine, Vitamin B1, Thiamin | Thiamina | HCl, mononitrate, benfotiamine | mg |
| riboflavin | 리보플라빈, 비타민 B2, B2 | Riboflavin, Vitamin B2 | Riboflavinum | riboflavin, R5P (riboflavin-5-phosphate) | mg |
| niacin | 나이아신, 니아신, 비타민 B3, B3, 니코틴산, 니코틴아미드, 니아신아마이드 | Niacin, Vitamin B3, Nicotinic acid, Nicotinamide, Niacinamide | Acidum nicotinicum | nicotinic acid, nicotinamide, inositol hexanicotinate | mg NE |
| pantothenic_acid | 판토텐산, 비타민 B5, B5 | Pantothenic acid, Vitamin B5, Pantothenate | Acidum pantothenicum | calcium pantothenate, pantethine | mg |
| pyridoxine | 피리독신, 비타민 B6, B6, P5P | Pyridoxine, Vitamin B6, Pyridoxal-5-phosphate (P5P) | Pyridoxinum | HCl, P5P (활성형) | mg |
| biotin | 비오틴, 비타민 B7, B7, 비타민 H | Biotin, Vitamin B7, Vitamin H | Biotinum | d-biotin | mcg |
| folate | 엽산, 폴산, 비타민 B9, B9, 5-MTHF, 메틸폴레이트 | Folate, Folic acid, Vitamin B9, 5-MTHF, L-methylfolate | Acidum folicum | folic acid, 5-MTHF (활성형) | mcg DFE |
| cobalamin | 코발라민, 비타민 B12, B12, 시아노코발라민, 메틸코발라민, 히드록소코발라민 | Cobalamin, Vitamin B12, Cyanocobalamin, Methylcobalamin, Hydroxocobalamin | Cobalaminum | cyano-, methyl-, hydroxo-, adenosyl- | mcg |
| vitamin_c | 비타민 C, 아스코르브산, 아스코르빈산 | Vitamin C, Ascorbic acid, Ascorbate | Acidum ascorbicum | ascorbic acid, sodium ascorbate, liposomal | mg |

---

## 2. 미네랄 (15종)

| canonical_name | kor_aliases | eng_aliases | latin_aliases | forms | unit_default |
|---|---|---|---|---|---|
| calcium | 칼슘 | Calcium, Ca | Calcium | carbonate, citrate, malate, lactate, gluconate | mg (원소) |
| magnesium | 마그네슘 | Magnesium, Mg | Magnesium | oxide, citrate, glycinate, malate, threonate, taurate | mg (원소) |
| iron | 철, 철분, 아이언 | Iron, Fe | Ferrum | fumarate, sulfate, bisglycinate, gluconate, heme | mg (원소) |
| zinc | 아연, 징크 | Zinc, Zn | Zincum | gluconate, picolinate, citrate, bisglycinate, oxide | mg (원소) |
| selenium | 셀레늄, 셀레니움 | Selenium, Se | Selenium | selenomethionine, sodium selenite, yeast | mcg |
| copper | 구리 | Copper, Cu | Cuprum | gluconate, bisglycinate, sulfate | mg (원소) |
| manganese | 망간 | Manganese, Mn | Manganum | gluconate, bisglycinate, sulfate | mg (원소) |
| iodine | 요오드, 아이오딘, 옥소 | Iodine, I | Iodum | potassium iodide, kelp | mcg |
| chromium | 크롬, 크로뮴 | Chromium, Cr | Chromium | picolinate, polynicotinate, GTF | mcg |
| molybdenum | 몰리브덴 | Molybdenum, Mo | Molybdenum | sodium molybdate, glycinate | mcg |
| potassium | 칼륨, 포타슘 | Potassium, K | Kalium | chloride, citrate, gluconate | mg (원소) |
| sodium | 나트륨, 소듐 | Sodium, Na | Natrium | chloride, citrate | mg (원소) |
| phosphorus | 인, 인산 | Phosphorus, P | Phosphorus | dicalcium phosphate, sodium phosphate | mg (원소) |
| boron | 붕소 | Boron, B | Borum | boric acid, sodium borate, glycinate | mg (원소) |
| vanadium | 바나듐 | Vanadium, V | Vanadium | vanadyl sulfate | mcg |

---

## 3. 아미노산 (10종)

| canonical_name | kor_aliases | eng_aliases | latin_aliases | forms | unit_default |
|---|---|---|---|---|---|
| l_theanine | L-테아닌, 테아닌, 엘테아닌 | L-Theanine, Theanine, Suntheanine | - | L-form (생리활성) | mg |
| l_arginine | L-아르기닌, 아르기닌 | L-Arginine, Arginine | - | HCl, AAKG (alpha-ketoglutarate) | mg |
| l_cysteine | L-시스테인, 시스테인 | L-Cysteine, Cysteine | - | L-cysteine, NAC (별도 엔트리) | mg |
| l_glutamine | L-글루타민, 글루타민 | L-Glutamine, Glutamine | - | free-form | mg |
| l_lysine | L-라이신, 라이신 | L-Lysine, Lysine | - | HCl | mg |
| l_carnitine | L-카르니틴, 카르니틴, 아세틸카르니틴, ALCAR | L-Carnitine, Acetyl-L-carnitine (ALCAR) | - | L-tartrate, ALCAR, propionyl | mg |
| taurine | 타우린 | Taurine | - | free-form | mg |
| bcaa | BCAA, 분지사슬아미노산, 류신, 이소류신, 발린 | BCAA, Branched-chain amino acids, Leucine, Isoleucine, Valine | - | 2:1:1 또는 4:1:1 비율 | mg |
| glycine | 글리신, 글라이신 | Glycine | - | free-form | mg |
| l_tryptophan | L-트립토판, 트립토판 | L-Tryptophan, Tryptophan | - | free-form | mg |

---

## 4. 지방산 (4종)

| canonical_name | kor_aliases | eng_aliases | latin_aliases | forms | unit_default |
|---|---|---|---|---|---|
| epa | EPA, 에이코사펜타엔산 | EPA, Eicosapentaenoic acid | Acidum eicosapentaenoicum | triglyceride, ethyl ester, phospholipid (krill) | mg |
| dha | DHA, 도코사헥사엔산 | DHA, Docosahexaenoic acid | Acidum docosahexaenoicum | triglyceride, ethyl ester, phospholipid (krill), algae | mg |
| ala_omega3 | 알파-리놀렌산, ALA (오메가-3) | Alpha-linolenic acid, ALA | Acidum alpha-linolenicum | flaxseed oil, chia, perilla | mg |
| gla | 감마-리놀렌산, GLA | Gamma-linolenic acid, GLA | Acidum gamma-linolenicum | evening primrose, borage, blackcurrant | mg |

> **주의**: `ALA`는 오메가-3의 알파-리놀렌산(`ala_omega3`)과 항산화제 알파-리포산(`alpha_lipoic_acid`, §7)이 약어가 같음. 입력 맥락으로 구분 필요.

---

## 5. 식물추출물 (20종)

| canonical_name | kor_aliases | eng_aliases | latin_aliases | forms | unit_default |
|---|---|---|---|---|---|
| silymarin | 실리마린, 밀크씨슬, 흰무늬엉겅퀴 | Silymarin, Milk thistle | Silybum marianum | 표준화 추출물 (실리마린 70~80%), 실리빈 | mg |
| curcumin | 커큐민, 강황, 울금 | Curcumin, Turmeric | Curcuma longa | 표준화 (95% curcuminoids), BCM-95, Meriva, 나노/리포솜 | mg |
| boswellia | 보스웰리아, 유향 | Boswellia, Frankincense | Boswellia serrata | AKBA 표준화 (10~30%) | mg |
| ashwagandha | 아슈와간다, 인도인삼 | Ashwagandha, Indian ginseng, KSM-66, Sensoril | Withania somnifera | KSM-66 (뿌리), Sensoril (뿌리+잎) | mg |
| rhodiola | 홍경천, 로디올라 | Rhodiola, Golden root | Rhodiola rosea | 표준화 (rosavins 3%, salidroside 1%) | mg |
| ginkgo_biloba | 은행잎, 은행 | Ginkgo biloba | Ginkgo biloba | EGb 761, 표준화 (flavone 24%, terpene 6%) | mg |
| panax_ginseng | 인삼, 파낙스 진생 | Panax ginseng, Asian ginseng | Panax ginseng | 뿌리 추출물, 진세노사이드 표준화 | mg |
| korean_red_ginseng | 홍삼, 고려홍삼 | Korean red ginseng | Panax ginseng (red processed) | 농축액, 정, 진세노사이드 Rg1+Rb1+Rg3 | mg |
| maca | 마카 | Maca, Peruvian ginseng | Lepidium meyenii | 뿌리 분말/추출물, gelatinized | mg |
| saw_palmetto | 쏘팔메토, 톱야자 | Saw palmetto | Serenoa repens | 지용성 추출물 (free fatty acids 85~95%) | mg |
| green_tea_extract | 녹차추출물, 녹차카테킨, EGCG | Green tea extract, EGCG | Camellia sinensis | EGCG 표준화, 카테킨 총량 | mg |
| grape_seed_extract | 포도씨추출물, OPC, 프로안토시아니딘 | Grape seed extract, OPC, Proanthocyanidin | Vitis vinifera | OPC 95% 표준화 | mg |
| resveratrol | 레스베라트롤 | Resveratrol, trans-Resveratrol | Polygonum cuspidatum, Vitis vinifera | trans-resveratrol 98% | mg |
| cranberry | 크랜베리 | Cranberry | Vaccinium macrocarpon | PAC 표준화 (36 mg PAC/일 기준) | mg |
| bilberry | 빌베리 | Bilberry | Vaccinium myrtillus | 안토시아닌 25~36% 표준화 | mg |
| acai | 아사이, 아사이베리 | Acai berry | Euterpe oleracea | 분말, 추출물 | mg |
| elderberry | 엘더베리, 블랙 엘더베리 | Elderberry, Black elderberry, Sambucus | Sambucus nigra | 안토시아닌 표준화 시럽/추출물 | mg |
| echinacea | 에키네시아, 자주루드베키아 | Echinacea, Purple coneflower | Echinacea purpurea / angustifolia | 전초/뿌리 추출물 | mg |
| ginger | 생강 | Ginger | Zingiber officinale | gingerol 표준화 | mg |
| garlic_extract | 마늘 추출물, 흑마늘, 숙성마늘추출물 | Garlic extract, Aged garlic extract, Allicin | Allium sativum | allicin 표준화, AGE (aged) | mg |

---

## 6. 프로바이오틱스·프리바이오틱스 (10종)

| canonical_name | kor_aliases | eng_aliases | latin_aliases | forms | unit_default |
|---|---|---|---|---|---|
| l_acidophilus | 락토바실러스 아시도필러스, L.아시도필러스 | Lactobacillus acidophilus, L. acidophilus | Lactobacillus acidophilus | NCFM, La-5, DDS-1 균주 | CFU |
| b_longum | 비피도박테리움 롱검, B.롱검 | Bifidobacterium longum, B. longum | Bifidobacterium longum | BB536, Rosell-175 균주 | CFU |
| l_rhamnosus | 락토바실러스 람노서스, L.람노서스 | Lactobacillus rhamnosus, L. rhamnosus, LGG | Lacticaseibacillus rhamnosus | LGG, HN001 균주 | CFU |
| l_plantarum | 락토바실러스 플란타룸, L.플란타룸 | Lactobacillus plantarum, L. plantarum | Lactiplantibacillus plantarum | 299v, CJLP133 균주 | CFU |
| l_casei | 락토바실러스 카제이, L.카제이 | Lactobacillus casei, L. casei | Lacticaseibacillus casei | Shirota, DN-114001 균주 | CFU |
| b_lactis | 비피도박테리움 락티스, B.락티스 | Bifidobacterium lactis, B. lactis | Bifidobacterium animalis subsp. lactis | BB-12, HN019 균주 | CFU |
| s_boulardii | 사카로마이세스 보울라디, S.보울라디 | Saccharomyces boulardii, S. boulardii | Saccharomyces cerevisiae var. boulardii | CNCM I-745 | CFU |
| inulin | 이눌린 | Inulin | Cichorium intybus (chicory root) | 치커리 유래, 장쇄/단쇄 | g |
| fos | FOS, 프락토올리고당 | FOS, Fructooligosaccharides | - | short-chain FOS, scFOS | g |
| gos | GOS, 갈락토올리고당 | GOS, Galactooligosaccharides | - | 유당 유래 | g |

> **CFU 표기**: 억 (CFU × 10^8) / 천억 (10^11) 단위 혼재. 라벨 확인 필수.

---

## 7. 기타 (지질·항산화·관절·기능성, 28종)

| canonical_name | kor_aliases | eng_aliases | latin_aliases | forms | unit_default |
|---|---|---|---|---|---|
| coenzyme_q10 | 코엔자임 Q10, 코큐텐, CoQ10, 유비퀴논, 유비퀴놀 | Coenzyme Q10, CoQ10, Ubiquinone, Ubiquinol | - | ubiquinone (산화형), ubiquinol (환원형) | mg |
| lutein | 루테인 | Lutein | Tagetes erecta (marigold) | free / esterified | mg |
| zeaxanthin | 지아잔틴, 제아잔틴 | Zeaxanthin | - | 루테인과 10:2 비율 흔함 | mg |
| astaxanthin | 아스타잔틴 | Astaxanthin | Haematococcus pluvialis | 헤마토코쿠스 유래 | mg |
| collagen | 콜라겐, 가수분해 콜라겐, 콜라겐 펩타이드 | Collagen, Hydrolyzed collagen, Collagen peptides | - | Type I (피부/뼈), II (관절), III | g |
| glucosamine | 글루코사민 | Glucosamine | - | sulfate, HCl | mg |
| chondroitin | 콘드로이틴 | Chondroitin sulfate | - | bovine, shark, porcine | mg |
| msm | MSM, 메틸설포닐메탄 | MSM, Methylsulfonylmethane | - | OptiMSM | mg |
| hyaluronic_acid | 히알루론산 | Hyaluronic acid, HA | - | 저분자, 고분자 | mg |
| lecithin | 레시틴, 포스파티딜콜린 | Lecithin, Phosphatidylcholine | - | soy, sunflower | mg |
| phosphatidylserine | 포스파티딜세린, PS | Phosphatidylserine, PS | - | soy, sunflower | mg |
| mct | MCT, 중쇄중성지방, 중쇄지방산 | MCT, Medium-chain triglyceride, C8, C10 | - | C8 (caprylic), C10 (capric) | g |
| quercetin | 케르세틴, 퀘르세틴 | Quercetin | - | aglycone, phytosome | mg |
| alpha_lipoic_acid | 알파리포산, ALA (알파-리포산) | Alpha-lipoic acid, ALA, R-ALA | - | racemic, R-ALA | mg |
| melatonin | 멜라토닌 | Melatonin | - | IR, SR (서방형) | mg |
| hydroxy_tryptophan | 5-HTP, 5-하이드록시트립토판 | 5-HTP, 5-Hydroxytryptophan | Griffonia simplicifolia | griffonia seed 유래 | mg |
| same | SAMe, 사메, S-아데노실메티오닌 | SAMe, S-Adenosylmethionine | - | disulfate tosylate | mg |
| nac | NAC, N-아세틸시스테인 | NAC, N-Acetyl cysteine, N-Acetyl-L-cysteine | - | free-form | mg |
| nmn | NMN, 니코틴아미드 모노뉴클레오타이드 | NMN, Nicotinamide mononucleotide | - | β-NMN | mg |
| creatine | 크레아틴, 크레아틴 모노하이드레이트 | Creatine, Creatine monohydrate | - | monohydrate, HCl | g |
| hmb | HMB, 베타-하이드록시 베타-메틸 부티레이트 | HMB, Beta-hydroxy beta-methylbutyrate | - | Ca-HMB, free acid | g |
| propolis | 프로폴리스 | Propolis | - | 플라보노이드 표준화 | mg |
| royal_jelly | 로얄젤리 | Royal jelly | - | 10-HDA 표준화 | mg |
| spirulina | 스피룰리나 | Spirulina | Arthrospira platensis | 분말/정 | g |
| chlorella | 클로렐라 | Chlorella | Chlorella vulgaris | 세포벽 파쇄 | g |
| psyllium_husk | 차전자피, 질경이씨, 사일리엄 | Psyllium husk | Plantago ovata | husk 분말 | g |
| choline | 콜린, 알파GPC, CDP콜린, 시티콜린 | Choline, Alpha-GPC, CDP-choline, Citicoline | - | bitartrate, Alpha-GPC, CDP-choline | mg |
| inositol | 이노시톨, 미오이노시톨 | Inositol, Myo-inositol, D-chiro-inositol | - | myo, d-chiro (40:1 흔함) | mg |

---

## 8. 매칭 알고리즘 (분석 시 적용 순서)

1. **정확 일치**: 입력을 소문자·공백 제거·하이픈 통일 후 `canonical_name` / `kor_aliases` / `eng_aliases` / `latin_aliases` 각 필드 완전 일치 검색
2. **부분 포함**: 정확 일치 실패 시 aliases 내 부분 문자열 포함 검색 (예: "비타민B1" → "비타민 B1" 매칭)
3. **약어/숫자**: "B1" "B12" "K2" "D3" 같은 약어는 상위 비타민과 함께 매칭
4. **화합물 → 원소**: "Calcium Carbonate 1250 mg" 같은 입력은 `calcium` 매칭 + forms 에 carbonate 기록 + §0-8 계수로 원소량 환산
5. **실패 → Claude 추론**: 위 4단계 모두 실패 시 Claude가 추론. 리포트에 `canonical_name: unknown_*, 자동 매칭됨 (검증 필요)` 플래그
6. **중복 병합**: 동일 canonical_name이 여러 형태로 라벨에 있으면 원소량(또는 활성형 기준)으로 합산 후 "N개 형태 병합" 메모

---

## 9. 확장 정책 (v2+)

- 본 리스트에 없는 성분이 실사용에서 3회 이상 등장하면 추가 후보.
- 추가 시: 출처(NIH ODS 또는 식약처 원료DB URL) 확인 → canonical_name 부여 → 카테고리 내 알파벳 순 삽입.
- 단위/forms 변경은 § 0 단위 변환식과 일관성 검증 필수.
