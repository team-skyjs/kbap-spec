# Phase 1 Data Model — 001-personalized-menu-mvp

소유권 구분이 핵심이다. **App/BE 소유**(MongoDB)와 **큐레이션 팀 소유**(외부, 읽기 전용 소비)를
분리한다. 위험도는 둘을 결합해 계산한다.

## A. App/BE 소유 (MongoDB)

### User
- `id`, `email`(소셜 계정에서 수집, null 가능), `authProviders`[apple|google] — 소셜 전용,
  자체 이메일 로그인 없음(2026-07-08), `nickname`,
  `nationality`(ISO 3166-1 alpha-2), `readerLanguage`(BCP-47, MVP 영어 우선),
  `spiceTolerance`(0–10, optional), `createdAt`.
- `avoidedIngredients`: string[] — 안 먹는 재료 81종 중 부분집합(평면 재료코드, 카테고리 없음). 임베디드.
- `consent`: ConsentRecord (임베디드).
- 규칙: `nationality`는 **전체 국가(ISO 3166-1) 허용**, `readerLanguage`는 **MVP 9개로 제한**
  (en/zh-Hans/zh-Hant/ja/vi/id/th/ru/es). 가입 시 국적→언어 자동선택은 9개 내에서 매핑하고,
  매핑되는 언어가 없으면 en 폴백. (국적을 언어 9개에 맞춰 제한하지 말 것.)
- 규칙: `avoidedIngredients`는 온보딩에서 입력받되 **건너뛸 수 있다**(FR-003, skip 가능·강력 권장). 계정 삭제 시 본 문서 + 민감정보
  즉시 완전 삭제, 작성 리뷰는 익명화(FR-032).

### AvoidedIngredients (User.avoidedIngredients)
- 사용자가 안 먹는 **재료코드의 평면 목록**(81종 중 부분집합). 카테고리(알러지/종교/식이) 구분 없음 — 이유 불문 회피 대상.
- 재료코드는 BE 81종 카탈로그 기준. 민감정보로 취급(회피 이유가 알러지·종교일 수 있음). 위험도 계산의 입력.
- 판정 로직 상세: `local-handoff/kbap/risk-display-policy.md`.

### ConsentRecord (User 임베디드)
- `safetyNoticeAcceptedAt`, `version`. (FR-030)

### Review
- `id`, `foodId`(카탈로그 음식 ID 참조), `userId`(익명화 시 null),
  `authorNationalitySnapshot`, `authorRankTierSnapshot`,
  `rating`(정수 1–5, FR-022), `body`, `bodyLanguage`(BCP-47), `createdAt`,
  `anonymized`(bool).
- 규칙: 평점은 1–5 정수. 같은 국적 평점 집계를 위해 작성 시점 국적 스냅샷 저장.
- 익명화: 계정 삭제 시 `userId`=null, 개인식별 필드 제거, 평점/본문 보존(FR-032).

### ScanItemLog (홈 '최근' + 미등록 로깅 겸용)
- `id`, `userId`, `rawText`(인식 원문), `matchedFoodId`(nullable),
  `riskStateSnapshot`(safe|caution|danger|unable), `scannedAt`.
- 규칙: 홈의 "최근 스캔/조회" 피드(FR-034)와 디테일 조회 이력의 소스.

### UnregisteredFoodLog
- `id`, `rawText`, `readerLanguage`, `count`, `lastSeenAt`.
- 규칙: 카탈로그 미매칭(미등록 음식) 발생 기록 → 큐레이션 보강용(FR-033).

### UserActivityScore (랭킹, 파생/캐시)
- `userId`, `reviewCount`, `distinctFoodCount`, `scanCount`, `score`, `tier`.
- 규칙: score = 가중합(리뷰 + 음식 다양성 + 스캔) (FR-025). 2차에 커뮤니티 활성도 추가.
  tier: Rookie … "Korean to the bone".

## B. 큐레이션 팀 소유 (외부, 읽기 전용 소비)

> 스키마/적재는 본 기능 범위 밖. BE는 아래 형태로 **읽어서 사용**한다고 가정(D5).

### Food (canonical catalog) — 음식 ID 단일 기준 (FR-031, Q1)
- `id`(canonical), `names`{locale → name}, `descriptions`{locale → ≤150자(EN 기준)},
  `photoUrl`, `category`, `popularityRank`.
- 규칙: 스캔 원문은 이 카탈로그로 정규화. 없으면 '미등록'.

### Ingredient/Component
- `foodId`, `name`{locale}, `code`(81종 표준 재료코드), `inclusionPercent`(포함확률 0–100).
- 규칙: 위험도 판정의 근거(회피 재료 × 포함확률 밴드).

## C. 결합 계산 (저장 아님, 응답 시 산출 · 캐시 가능)

### FoodRiskAssessment (User × Food)
- 입력: User.avoidedIngredients(A, 회피 재료코드) + Food.Ingredient.code/inclusionPercent(B).
- 출력: `state`(safe|caution|danger|**unable**), `basis`[{ingredient, inclusionPercent, reason}].
- 규칙(헌법 III):
  - 회피 재료 포함확률: ≥60% → danger · 10–60% → caution · <10% → safe. 최악 재료 승자(risk-display-policy).
  - 매장별 변동/저비율/불확실 → caution(임의로 safe 단정 금지).
  - 카탈로그 미매칭(미등록) 또는 성분 데이터 부재 → **unable**(FR-011/033).
  - `basis`로 판정 근거 추적 가능(FR-012). 근거 없는 safe 금지.

### OwnerConfirmation (place language = ko, MVP 고정)
- 입력: Food.names.ko + 대상 ingredient.name.ko.
- 출력: 한국어 질문 문구("{메뉴명}에 {성분}이 들어가나요?") + 알러지 설명. 메뉴명은 스캔
  인식 메뉴명과 정확 일치(FR-018/019). place 언어는 데이터로 모델링(헌법 I), MVP 값 ko.

## 관계 요약
- User 1—N Review, User 1—N ScanItemLog, User 1—1 UserActivityScore.
- Review N—1 Food(외부), ScanItemLog N—0..1 Food(외부).
- FoodRiskAssessment = User ⋈ Food(외부 성분) (계산).
