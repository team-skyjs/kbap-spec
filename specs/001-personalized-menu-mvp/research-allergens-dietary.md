# 알레르기 · 식이 분류 리서치 (기획 공유용)

> **목적**: 개인화 메뉴 MVP의 핵심 도메인인 "알레르기"와 "식이 제한"을 팀(기획·FE·BE·디자인)이
> 공통 언어로 논의하기 위한 레퍼런스. 규제 출처(US/EU/한국 식약처) 기반으로 정리했으며,
> 이후 알레르겐/식이 태그 데이터 모델과 API 컨트랙트의 입력 자료가 된다.
>
> **상태**: 리서치/논의용 초안. 확정 시 `data-model.md` · `contracts/openapi.yaml`에 반영.
> **작성 기준일**: 2026-06-25

---

## 0. 한눈에 보는 요약

- "상위 N개 알레르겐"의 **공식 글로벌 표준은 없다.** 각국 규제기관이 표시 의무화한 건 9~19종.
- 본 문서는 **규제 핵심(US Big-9 / EU-14 / 한국 식약처 19종)을 기반**으로 깔고, 임상에서 흔히
  보고되는 항목까지 50개로 확장했다.
- 한국 음식 앱이므로 **서구 리스트엔 없지만 한식에서 치명적인 항목**(메밀·고등어·잣·복숭아·아황산류)을
  반드시 포함한다.
- 식이 제한은 "고기 안 먹음" 단일 축이 아니라 **① 동물성 허용 범위 축**과 **② 종교/문화 축**의
  2차원으로 모델링해야 정확하다.
- 한식의 진짜 난이도는 **'숨은 동물성'**(육수·액젓·젓갈·젤라틴 등)이다. 메뉴명만으로는 판정 불가.

범례: 🇺🇸 US Big-9 · 🇪🇺 EU-14 · 🇰🇷 한국 식약처 표시의무

---

## 1. 알레르겐 50종

### 1-1. 유제품 · 난류
| # | 알레르겐 | EN | 규제 |
|---|---|---|---|
| 1 | 우유 | Milk (cow) | 🇺🇸🇪🇺🇰🇷 |
| 2 | 달걀(난류) | Egg | 🇺🇸🇪🇺🇰🇷 |
| 3 | 산양유 | Goat's milk | |

### 1-2. 땅콩 · 견과 · 씨앗
| # | 알레르겐 | EN | 규제 |
|---|---|---|---|
| 4 | 땅콩 | Peanut | 🇺🇸🇪🇺🇰🇷 |
| 5 | 호두 | Walnut | 🇺🇸🇪🇺🇰🇷 |
| 6 | 잣 | Pine nut | 🇰🇷 |
| 7 | 아몬드 | Almond | 🇺🇸🇪🇺 |
| 8 | 캐슈넛 | Cashew | 🇺🇸🇪🇺 |
| 9 | 피스타치오 | Pistachio | 🇺🇸🇪🇺 |
| 10 | 헤이즐넛 | Hazelnut | 🇺🇸🇪🇺 |
| 11 | 마카다미아 | Macadamia | 🇺🇸🇪🇺 |
| 12 | 피칸 | Pecan | 🇺🇸🇪🇺 |
| 13 | 브라질너트 | Brazil nut | 🇺🇸🇪🇺 |
| 14 | 밤 | Chestnut | |
| 15 | 참깨 | Sesame | 🇺🇸🇪🇺 |
| 16 | 해바라기씨 | Sunflower seed | |
| 17 | 겨자 | Mustard | 🇪🇺 |

### 1-3. 곡물 · 글루텐
| # | 알레르겐 | EN | 규제 |
|---|---|---|---|
| 18 | 밀 | Wheat | 🇺🇸🇪🇺🇰🇷 |
| 19 | 메밀 | Buckwheat | 🇰🇷 |
| 20 | 보리 | Barley | 🇪🇺 (gluten) |
| 21 | 호밀 | Rye | 🇪🇺 (gluten) |
| 22 | 귀리 | Oat | 🇪🇺 (gluten) |
| 23 | 옥수수 | Corn | |

### 1-4. 콩류
| # | 알레르겐 | EN | 규제 |
|---|---|---|---|
| 24 | 대두 | Soybean | 🇺🇸🇪🇺🇰🇷 |
| 25 | 루핀 | Lupin | 🇪🇺 |
| 26 | 완두콩 | Pea | |
| 27 | 병아리콩 | Chickpea | |
| 28 | 렌틸콩 | Lentil | |

### 1-5. 갑각류 · 연체류(조개류)
| # | 알레르겐 | EN | 규제 |
|---|---|---|---|
| 29 | 새우 | Shrimp | 🇺🇸🇪🇺🇰🇷 |
| 30 | 게 | Crab | 🇺🇸🇪🇺🇰🇷 |
| 31 | 가재/랍스터 | Lobster | 🇺🇸🇪🇺 |
| 32 | 오징어 | Squid | 🇪🇺🇰🇷 |
| 33 | 문어 | Octopus | 🇪🇺 |
| 34 | 굴 | Oyster | 🇪🇺🇰🇷 |
| 35 | 전복 | Abalone | 🇪🇺🇰🇷 |
| 36 | 홍합 | Mussel | 🇪🇺🇰🇷 |
| 37 | 조개(바지락 등) | Clam | 🇪🇺 |
| 38 | 가리비 | Scallop | 🇪🇺 |

### 1-6. 어류
| # | 알레르겐 | EN | 규제 |
|---|---|---|---|
| 39 | 고등어 | Mackerel | 🇪🇺🇰🇷 |
| 40 | 연어 | Salmon | 🇺🇸🇪🇺 |
| 41 | 참치 | Tuna | 🇺🇸🇪🇺 |
| 42 | 대구 | Cod | 🇺🇸🇪🇺 |
| 43 | 멸치 | Anchovy | 🇪🇺 |

### 1-7. 육류
| # | 알레르겐 | EN | 규제 |
|---|---|---|---|
| 44 | 쇠고기 | Beef | 🇰🇷 |
| 45 | 돼지고기 | Pork | 🇰🇷 |
| 46 | 닭고기(가금류) | Chicken | 🇰🇷 |

### 1-8. 과일 · 채소 · 첨가물
| # | 알레르겐 | EN | 규제 |
|---|---|---|---|
| 47 | 복숭아 | Peach | 🇰🇷 |
| 48 | 토마토 | Tomato | 🇰🇷 |
| 49 | 셀러리 | Celery | 🇪🇺 |
| 50 | 아황산류(이산화황) | Sulfites | 🇪🇺🇰🇷 |

---

## 2. 식이(채식·비건·종교) 분류

### 2-1. 채식 스펙트럼 (제한 적음 → 많음)

| 분류 | 붉은고기 | 가금류 | 생선·해산물 | 달걀 | 유제품 | 꿀 |
|---|:---:|:---:|:---:|:---:|:---:|:---:|
| **플렉시테리언** Flexitarian | △ 가끔 | △ | ✅ | ✅ | ✅ | ✅ |
| **폴로테리언** Pollotarian | ❌ | ✅ | ❌ | ✅ | ✅ | ✅ |
| **페스카테리언** Pescatarian | ❌ | ❌ | ✅ | ✅ | ✅ | ✅ |
| **락토-오보 베지테리언** (가장 일반적) | ❌ | ❌ | ❌ | ✅ | ✅ | ✅ |
| **락토 베지테리언** Lacto | ❌ | ❌ | ❌ | ❌ | ✅ | ✅ |
| **오보 베지테리언** Ovo | ❌ | ❌ | ❌ | ✅ | ❌ | ✅ |
| **비건** Vegan | ❌ | ❌ | ❌ | ❌ | ❌ | ❌ |

> △ = 빈도/상황에 따라 가끔 허용. ✅ 허용 / ❌ 제외.

### 2-2. 더 엄격하거나 특수한 유형
| 분류 | 핵심 정의 / 추가로 제외하는 것 |
|---|---|
| **로 비건** Raw vegan | 비건 + 약 46°C 이상 **가열 조리한 음식** 제외 |
| **프루테리언** Fruitarian | 식물을 죽이지 않고 얻는 것(과일·견과·씨앗) 위주, **뿌리·잎채소 제외** |
| **자이나교 채식** Jain | 비건에 가깝고 + **뿌리채소(양파·마늘·감자·당근) 전부 제외** |

### 2-3. 종교 · 문화 기반 식이 (한식 앱에 특히 중요)
| 분류 | 주로 제외하는 것 |
|---|---|
| **할랄** Halal (이슬람) | **돼지고기**, **알코올(요리술·미림 포함)**, 할랄 비도축 육류, 젤라틴·라드 등 돼지 유래 첨가물 |
| **코셔** Kosher (유대교) | 돼지, **갑각류·조개·오징어**, **육류+유제품 한 끼 혼합**, 비늘/지느러미 없는 어류 |
| **힌두교** | **소고기**(절대), 다수가 채식·달걀까지 회피 |
| **불교 사찰음식** | 모든 육류 + **오신채(파·마늘·부추·달래·흥거)** 제외 |

---

## 3. '숨은 동물성' 체크리스트 (한식 판정의 핵심 함정)

겉보기엔 채식/비건처럼 보여도 동물성이 숨어 있어 **놓치기 쉬운** 항목. 앱이 반드시 잡아야 함.

| 숨은 성분 | 어디에 들어가나 | 누가 걸리나 |
|---|---|---|
| **젤라틴** | 젤리·마시멜로·일부 요구르트 | 비건·채식·할랄(돼지)·코셔 |
| **육수·다시** | 멸치/사골/가다랑어(가쓰오부시) 베이스 국·찌개 | 비건·채식·페스카테리언(사골) |
| **액젓·새우젓·젓갈** | 김치·찌개·나물무침 → **김치 대부분 비건 아님** ⚠️ | 비건·채식 |
| **굴소스** | 볶음·잡채 양념 | 비건·채식 |
| **라드·우지** | 볶음·튀김 기름 | 비건·채식·할랄(돼지) |
| **버터·기버터** | 구이·볶음·베이커리 | 비건 |
| **카민(코치닐, E120)** | 붉은 색소 (곤충 유래) | 비건·코셔 |
| **레닛** | 치즈 응고제 (동물성) | 비건 |
| **꿀** | 디저트·음료 | 비건만 |
| **알코올·미림·맛술** | 조림·볶음 양념 | 할랄 |
| **오신채(파·마늘 등)** | 거의 모든 한식 양념 | 사찰음식·자이나교 |

---

## 4. 통합 식재료 마스터 리스트 (81종)

위 1~3장에서 도출된 식재료를 중복 제거해 한 목록으로 정리.

```
계란, 우유, 유제품, 산양유, 버터, 기버터, 치즈, 젤라틴, 레닛, 꿀, 카민,
땅콩, 호두, 잣, 아몬드, 캐슈넛, 피스타치오, 헤이즐넛, 마카다미아, 피칸,
브라질너트, 밤, 참깨, 해바라기씨, 겨자,
밀, 메밀, 보리, 호밀, 귀리, 옥수수,
대두, 루핀, 완두콩, 병아리콩, 렌틸콩,
새우, 새우젓, 게, 가재, 랍스터, 오징어, 문어, 굴, 굴소스, 전복, 홍합, 조개,
바지락, 가리비, 해산물,
생선, 고등어, 연어, 참치, 대구, 멸치, 액젓, 육수, 다시,
소고기, 돼지고기, 라드, 우지, 닭고기, 가금류,
복숭아, 토마토, 셀러리, 감자, 당근, 양파, 마늘, 파, 부추, 달래, 흥거,
알코올, 미림, 맛술, 아황산류
```

---

## 5. 기피 항목 3-카테고리 분류 (ALLERGEN / DIETARY_RULE / PERSONAL_AVOIDANCE)

### 5-1. 왜 나누나 (판정·안내 관점)
81종은 성격이 제각각이라 하나의 "기피 항목"으로만 묶으면 판정 강도와 안내 문구가 모호해진다.
세 가지 성격으로 나눠 안내 톤과 위험도를 다르게 가져간다.

- **ALLERGEN** — 건강 위험과 직결 → 가장 보수적으로 경고
  ("알러지 유발 성분이 포함될 가능성이 있습니다")
- **DIETARY_RULE** — 종교·윤리·채식·건강 식단 기준 → 기준 불일치로 안내
  ("설정한 식이 기준에 맞지 않는 성분이 포함될 수 있습니다")
- **PERSONAL_AVOIDANCE** — 개인 취향 → 부드럽게 안내
  ("개인적으로 피하는 재료가 포함될 가능성이 있습니다")

### 5-2. 한 항목이 여러 카테고리에 속함 → 선택 이유 수집
같은 **돼지고기**라도 알러지/종교/취향에 따라 메시지와 위험도가 달라진다. 따라서 사용자가
항목을 선택할 때 "왜 피하는지"를 함께 받아야 맞춤 판정이 가능하다. 단, UX 부담을 줄이기 위해:

- **단일 카테고리 항목** → 기본 이유 자동 적용, 추가 질문 없음 (예: 대두 = 알러지)
- **복수 카테고리 항목** → 이유 선택 (예: 돼지고기 → ① 알러지 / ② 식이·종교·건강 / ③ 개인 취향)

### 5-3. 선택 이유별 판정·안내 기준
| 선택 이유 | 예상 판정 | 안내 톤 |
|---|---|---|
| ALLERGEN | DANGER / CAUTION | 가장 강하게 경고 (보수적) |
| DIETARY_RULE | DANGER / CAUTION | 식이 기준 불일치 안내 |
| PERSONAL_AVOIDANCE | CAUTION | 부드럽게 안내 |

> 공통: 성분 포함 여부가 확실치 않으면 **UNKNOWN**. 안전 도메인이므로 모호할 땐 경고 쪽으로 기운다.

### 5-4. 전체 81종 분류
> 범례: `ALLERGEN`(건강위험) · `DIETARY_RULE`(식이·종교·건강기준) · `PERSONAL_AVOIDANCE`(개인취향)
> `PERSONAL_AVOIDANCE`는 기본 제안값이며, 선택 UX에선 다른 항목에도 개인 기피 사유를 허용할 수 있다.

```yaml
# ── 유제품 · 난류 · 동물성 가공 ──
- { code: EGG,             name: 계란,        categories: [ALLERGEN, DIETARY_RULE] }
- { code: MILK,            name: 우유,        categories: [ALLERGEN, DIETARY_RULE] }
- { code: DAIRY,           name: 유제품,      categories: [ALLERGEN, DIETARY_RULE] }
- { code: GOAT_MILK,       name: 산양유,      categories: [ALLERGEN, DIETARY_RULE] }
- { code: BUTTER,          name: 버터,        categories: [ALLERGEN, DIETARY_RULE] }
- { code: GHEE,            name: 기버터,      categories: [ALLERGEN, DIETARY_RULE] }
- { code: CHEESE,          name: 치즈,        categories: [ALLERGEN, DIETARY_RULE] }
- { code: GELATIN,         name: 젤라틴,      categories: [DIETARY_RULE] }
- { code: RENNET,          name: 레닛,        categories: [DIETARY_RULE] }
- { code: HONEY,           name: 꿀,          categories: [DIETARY_RULE] }
- { code: CARMINE,         name: 카민,        categories: [DIETARY_RULE] }

# ── 견과 · 씨앗 ──
- { code: PEANUT,          name: 땅콩,        categories: [ALLERGEN] }
- { code: WALNUT,          name: 호두,        categories: [ALLERGEN] }
- { code: PINE_NUT,        name: 잣,          categories: [ALLERGEN] }
- { code: ALMOND,          name: 아몬드,      categories: [ALLERGEN] }
- { code: CASHEW,          name: 캐슈넛,      categories: [ALLERGEN] }
- { code: PISTACHIO,       name: 피스타치오,  categories: [ALLERGEN] }
- { code: HAZELNUT,        name: 헤이즐넛,    categories: [ALLERGEN] }
- { code: MACADAMIA,       name: 마카다미아,  categories: [ALLERGEN] }
- { code: PECAN,           name: 피칸,        categories: [ALLERGEN] }
- { code: BRAZIL_NUT,      name: 브라질너트,  categories: [ALLERGEN] }
- { code: CHESTNUT,        name: 밤,          categories: [ALLERGEN] }
- { code: SESAME,          name: 참깨,        categories: [ALLERGEN] }
- { code: SUNFLOWER_SEED,  name: 해바라기씨,  categories: [ALLERGEN] }
- { code: MUSTARD,         name: 겨자,        categories: [ALLERGEN, PERSONAL_AVOIDANCE] }

# ── 곡물 ──
- { code: WHEAT,           name: 밀,          categories: [ALLERGEN, DIETARY_RULE] }   # 글루텐프리
- { code: BUCKWHEAT,       name: 메밀,        categories: [ALLERGEN] }                 # 메밀 자체는 글루텐프리
- { code: BARLEY,          name: 보리,        categories: [ALLERGEN, DIETARY_RULE] }
- { code: RYE,             name: 호밀,        categories: [ALLERGEN, DIETARY_RULE] }
- { code: OAT,             name: 귀리,        categories: [ALLERGEN, DIETARY_RULE] }
- { code: CORN,            name: 옥수수,      categories: [ALLERGEN] }

# ── 콩류 ──
- { code: SOY,             name: 대두,        categories: [ALLERGEN] }
- { code: LUPIN,           name: 루핀,        categories: [ALLERGEN] }
- { code: PEA,             name: 완두콩,      categories: [ALLERGEN] }
- { code: CHICKPEA,        name: 병아리콩,    categories: [ALLERGEN] }
- { code: LENTIL,          name: 렌틸콩,      categories: [ALLERGEN] }

# ── 갑각 · 연체 · 조개 ──
- { code: SHRIMP,          name: 새우,        categories: [ALLERGEN, DIETARY_RULE] }
- { code: SALTED_SHRIMP,   name: 새우젓,      categories: [ALLERGEN, DIETARY_RULE, PERSONAL_AVOIDANCE] }
- { code: CRAB,            name: 게,          categories: [ALLERGEN, DIETARY_RULE] }
- { code: CRAYFISH,        name: 가재,        categories: [ALLERGEN, DIETARY_RULE] }
- { code: LOBSTER,         name: 랍스터,      categories: [ALLERGEN, DIETARY_RULE] }
- { code: SQUID,           name: 오징어,      categories: [ALLERGEN, DIETARY_RULE, PERSONAL_AVOIDANCE] }
- { code: OCTOPUS,         name: 문어,        categories: [ALLERGEN, DIETARY_RULE, PERSONAL_AVOIDANCE] }
- { code: OYSTER,          name: 굴,          categories: [ALLERGEN, DIETARY_RULE, PERSONAL_AVOIDANCE] }
- { code: OYSTER_SAUCE,    name: 굴소스,      categories: [ALLERGEN, DIETARY_RULE] }
- { code: ABALONE,         name: 전복,        categories: [ALLERGEN, DIETARY_RULE] }
- { code: MUSSEL,          name: 홍합,        categories: [ALLERGEN, DIETARY_RULE, PERSONAL_AVOIDANCE] }
- { code: CLAM,            name: 조개,        categories: [ALLERGEN, DIETARY_RULE] }
- { code: SHORT_NECK_CLAM, name: 바지락,      categories: [ALLERGEN, DIETARY_RULE] }
- { code: SCALLOP,         name: 가리비,      categories: [ALLERGEN, DIETARY_RULE] }
- { code: SEAFOOD,         name: 해산물,      categories: [ALLERGEN, DIETARY_RULE, PERSONAL_AVOIDANCE] }  # 상위 카테고리

# ── 어류 ──
- { code: FISH,            name: 생선,        categories: [ALLERGEN, DIETARY_RULE, PERSONAL_AVOIDANCE] }  # 상위 카테고리
- { code: MACKEREL,        name: 고등어,      categories: [ALLERGEN, DIETARY_RULE, PERSONAL_AVOIDANCE] }
- { code: SALMON,          name: 연어,        categories: [ALLERGEN, DIETARY_RULE] }
- { code: TUNA,            name: 참치,        categories: [ALLERGEN, DIETARY_RULE] }
- { code: COD,             name: 대구,        categories: [ALLERGEN, DIETARY_RULE] }
- { code: ANCHOVY,         name: 멸치,        categories: [ALLERGEN, DIETARY_RULE, PERSONAL_AVOIDANCE] }
- { code: FISH_SAUCE,      name: 액젓,        categories: [ALLERGEN, DIETARY_RULE, PERSONAL_AVOIDANCE] }
- { code: BROTH,           name: 육수,        categories: [ALLERGEN, DIETARY_RULE] }   # 멸치/사골 등 숨은 알러지원 → 보수적 경고
- { code: DASHI,           name: 다시,        categories: [ALLERGEN, DIETARY_RULE] }   # 가다랑어 등 숨은 알러지원 → 보수적 경고

# ── 육류 ──
- { code: BEEF,            name: 소고기,      categories: [ALLERGEN, DIETARY_RULE, PERSONAL_AVOIDANCE] }
- { code: PORK,            name: 돼지고기,    categories: [ALLERGEN, DIETARY_RULE, PERSONAL_AVOIDANCE] }
- { code: LARD,            name: 라드,        categories: [DIETARY_RULE] }
- { code: TALLOW,          name: 우지,        categories: [DIETARY_RULE] }
- { code: CHICKEN,         name: 닭고기,      categories: [ALLERGEN, DIETARY_RULE, PERSONAL_AVOIDANCE] }
- { code: POULTRY,         name: 가금류,      categories: [ALLERGEN, DIETARY_RULE, PERSONAL_AVOIDANCE] }  # 상위 카테고리

# ── 과일 · 채소 · 향신 · 첨가물 ──
- { code: PEACH,           name: 복숭아,      categories: [ALLERGEN] }
- { code: TOMATO,          name: 토마토,      categories: [ALLERGEN, PERSONAL_AVOIDANCE] }
- { code: CELERY,          name: 셀러리,      categories: [ALLERGEN] }
- { code: POTATO,          name: 감자,        categories: [DIETARY_RULE] }   # 자이나교 뿌리채소 제외
- { code: CARROT,          name: 당근,        categories: [DIETARY_RULE] }   # 자이나교 뿌리채소 제외
- { code: ONION,           name: 양파,        categories: [DIETARY_RULE, PERSONAL_AVOIDANCE] }
- { code: GARLIC,          name: 마늘,        categories: [DIETARY_RULE, PERSONAL_AVOIDANCE] }  # 오신채
- { code: SCALLION,        name: 파,          categories: [DIETARY_RULE, PERSONAL_AVOIDANCE] }  # 오신채
- { code: CHIVE,           name: 부추,        categories: [DIETARY_RULE, PERSONAL_AVOIDANCE] }  # 오신채
- { code: WILD_CHIVE,      name: 달래,        categories: [DIETARY_RULE] }   # 오신채
- { code: ASAFOETIDA,      name: 흥거,        categories: [DIETARY_RULE] }   # 오신채
- { code: ALCOHOL,         name: 알코올,      categories: [DIETARY_RULE, PERSONAL_AVOIDANCE] }
- { code: MIRIN,           name: 미림,        categories: [DIETARY_RULE, PERSONAL_AVOIDANCE] }
- { code: COOKING_WINE,    name: 맛술,        categories: [DIETARY_RULE, PERSONAL_AVOIDANCE] }
- { code: SULFITES,        name: 아황산류,    categories: [ALLERGEN] }
```

**분류 통계**: ALLERGEN 64종 · DIETARY_RULE 56종 · PERSONAL_AVOIDANCE 23종
(중복 포함 — 한 항목이 복수 카테고리에 속하므로 합계는 81을 넘는다)

> **경계 케이스 확정(2026-06-29)**:
> - 육수·다시 → `ALLERGEN` 추가(숨은 멸치·사골·가다랑어 대비, 보수적 경고)
> - 알코올·미림·맛술 → `PERSONAL_AVOIDANCE` 추가(취향·라이프스타일 금주 포함)
> - 굴소스·전복·조개·바지락·가리비 → 현행 유지(`PERSONAL` 미부여, 태그 일관성 우선)

---

## 6. 데이터 모델·기획 시사점 (논의 포인트)

1. **알레르기 ≠ 식이, 데이터 구조가 다르다.**
   - 알레르기 = "이 성분이 들었나?"(boolean 포함 여부).
   - 식이 = "이 카테고리 전체를 배제"(rule set). 메뉴 1개에 여러 동물성 태그를 달고,
     식이 유형별 규칙으로 자동 판정하는 구조가 맞다.

2. **식재료는 계층(부모-자식) 관계가 필요하다.**
   - 리스트엔 추상화 레벨이 섞여 있다: `생선`(상위) ⊃ `고등어`(개별), `해산물` ⊃ `새우`.
   - "생선 알레르기 → 고등어 자동 포함" 추론을 위해 계층 트리로 모델링한다.

3. **식이는 단일 축이 아니라 2차원이다.**
   - ① 동물성 허용 범위 축(플렉시 → 비건) + ② 종교/문화 축(할랄·코셔·사찰).
   - 비건 ⊄ 사찰음식(사찰음식은 오신채 추가 제외) — 별도 축으로 둬야 정확.

4. **'숨은 동물성'은 별도 플래그가 필요하다.**
   - 육수·액젓·젤라틴 등은 메뉴명에 안 드러난다. OCR 메뉴명만으로는 판정 불가 →
     **재료 단위 데이터** 또는 **'확실치 않음(unknown)' 상태**가 필요.
   - 안전 도메인이라 **false-OK(안전하다고 잘못 표시)가 가장 위험**. 모호하면 경고 쪽으로.

5. **규제 출처는 '순위'가 아니라 '태그'로 관리한다.**
   - 50개를 순위로 박으면 규제 변경 시 깨진다. `US/EU/KR` 출처 태그로 두면
     규제 업데이트·지역 확장에 안전하다.

---

## 7. 다음 단계 (제안)

- [ ] 식재료 마스터 리스트 → **계층 구조 + 영문 코드값(snake_case) + 다국어 라벨** 테이블화
- [ ] 알레르겐 enum / 식이 유형 enum → `data-model.md` 반영
- [ ] 식이 유형별 **제외 규칙 테이블**(어떤 재료 태그를 배제하는지) 정의
- [ ] `contracts/openapi.yaml`에 알레르겐/식이/재료 스키마 추가

> 출처 참고: US FDA FALCPA(Big-9, 2023 참깨 추가) · EU Regulation 1169/2011 Annex II(14종) ·
> 한국 식품의약품안전처 「식품등의 표시기준」 알레르기 유발물질(19종). 50종 확장분은 임상 보고
> 빈도 기반이며 출처마다 순위가 다를 수 있음.
