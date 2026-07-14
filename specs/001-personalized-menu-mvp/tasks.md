---
description: "Task list for 001-personalized-menu-mvp"
---

# Tasks: 개인화 한식 메뉴 안전 안내 (MVP)

> ⚠️ **동결 (2026-07-14)** — 태스크 관리는 **Jira(KB)로 승계**됐다(2026-07-06 결정, 이슈 상태
> 정본 = Jira). 이 파일은 더 이상 갱신하지 않으며, FR↔태스크 추적 기록으로만 보존한다.
> 체크박스가 비어 있는 것은 미완이 아니라 **이 파일에서 추적을 중단했기 때문**이다.
> 현재 할 일 = Jira + `LAUNCH.md`(큰그림).

**Input**: `specs/001-personalized-menu-mvp/` (plan.md, spec.md, research.md, data-model.md, contracts/openapi.yaml, quickstart.md)

**Constitution**: `.specify/memory/constitution.md` (I 국제화 · II 음식중심 · III 안전정확성 NON-NEGOTIABLE · IV 개인화 · V 단계출시 · VI 스펙=SSOT)

## Format: `[ID] [P?] [Repo] [Story] Description`

- **[P]**: 병렬 가능(다른 파일, 의존성 없음)
- **[Repo]**: `[SPEC]` spec 레포 · `[BE]` kbap-be(Kotlin/Spring) · `[FE]` kbap-fe(RN/Expo)
- **[Story]**: US1~US6 추적용 라벨
- 모든 작업에 정확한 파일 경로 포함

## 레포 경로 규약 (plan.md 기준)

- **SPEC**: `specs/001-personalized-menu-mvp/contracts/openapi.yaml` (계약 SSOT)
- **BE**: `kbap-be/src/main/kotlin/com/kbap/...`, 테스트 `kbap-be/src/test/kotlin/...`
- **FE**: `kbap-fe/app/`, `kbap-fe/components/`, `kbap-fe/features/`, `kbap-fe/lib/`, 테스트 `kbap-fe/tests/`

## 테스트 정책 (C안: 확대)

헌법 III(안전 정확성, NON-NEGOTIABLE)·SC-003(false-safe 0)에 따라 **계약 테스트 + 위험도 엔진
단위 테스트 + 각 스토리 핵심 통합/E2E 테스트**를 포함한다. 테스트 작업은 `(test)`로 표기하며
구현 전에 작성해 실패를 확인한다. 안전 루프(US2)·계정삭제(US5)는 end-to-end로 못 박는다.

---

## Phase 1: Setup (공유 인프라)

**Purpose**: 3-레포 골격 + 계약 발행 + 코드젠 파이프라인

- [ ] T001 [SPEC] `contracts/openapi.yaml` lint/유효성 검증 통과 확인(redocly/spectral 등), 버전 태그 부여 → FE/BE 공통 SSOT 고정
- [ ] T002 [P] [BE] `kbap-be` Spring Boot 3.x(Java 21, Kotlin) 프로젝트 생성 + Gradle 의존성(Spring Web, Security, Data MongoDB, springdoc-openapi)
- [ ] T003 [P] [FE] `kbap-fe` Expo(TypeScript) 프로젝트 생성 + Expo Router, React Query, i18next, expo-localization, `@react-native-ml-kit/text-recognition` 설치
- [ ] T004 [P] [BE] ktlint/detekt + 포맷터 설정 (`kbap-be/`)
- [ ] T005 [P] [FE] ESLint + Prettier + tsconfig strict 설정 (`kbap-fe/`)
- [ ] T006 [FE] `openapi.yaml` → TS 타입 생성 스크립트(openapi-typescript) 구성 → `kbap-fe/lib/api/schema.ts` 산출 (T001 의존)
- [ ] T007 [P] [BE] `openapi.yaml`를 빌드 리소스로 동기화 + springdoc로 런타임 노출, 계약 일치 검증 베이스 구성 (T001 의존)

**Checkpoint**: 양 레포 빌드 가능 + 계약에서 타입 생성/노출 동작

---

## Phase 2: Foundational (차단성 선행 작업)

**⚠️ CRITICAL**: 이 단계가 끝나기 전에는 어떤 유저 스토리도 시작 불가

### BE 기반
- [ ] T008 [BE] MongoDB(DocumentDB) 연결 설정 + 환경 구성(`application.yml`, 프로파일 local/aws) in `kbap-be/src/main/resources/`
- [ ] T009 [BE] 전역 에러 처리 + 구조화 로깅 + 요청 ID in `kbap-be/.../common/`
- [ ] T010 [BE] Spring Security + JWT(access/refresh) 골격 + 인증 필터 in `kbap-be/.../auth/security/`
- [ ] T011 [P] [BE] 베이스 엔티티/도큐먼트: `User`(restrictions·consent 임베디드), `Review`, `ScanItemLog`, `UnregisteredFoodLog`, `UserActivityScore` in `kbap-be/.../<domain>/domain/` (data-model.md A)
- [ ] T012 [P] [BE] **큐레이션 DB 어댑터 인터페이스 + stub**(`CurationCatalogPort`, 읽기 전용: Food/Ingredient 조회) in `kbap-be/.../curation/` — 외부 팀 계약 미확정이므로 인터페이스+표본 stub로 격리(D5)
- [ ] T013 [P] [BE] **번역 API 어댑터 인터페이스 + 기본 구현**(`TranslationPort`, 기본 Google Cloud Translation, 교체 가능, **번역 대상 9개 언어** en/zh-Hans/zh-Hant/ja/vi/id/th/ru/es) in `kbap-be/.../translation/` (D4, FR-027)
- [ ] T014 [BE] **위험도 산출 엔진 골격** `RiskAssessmentService`(User.restrictions × Ingredient.tags/percentage → safe/caution/danger/unable + riskBasis) in `kbap-be/.../food/risk/` (헌법 III, data-model.md C)
- [ ] T015 [P] [BE] (test) 위험도 엔진 단위 테스트: 충돌→danger, 변동/저비율→caution, **미매칭/데이터부재→unable(절대 safe 아님)**, basis 존재 in `kbap-be/src/test/kotlin/.../risk/` (SC-003 false-safe 0)

### FE 기반
- [ ] T016 [FE] API 클라이언트(React Query + fetch, 인증 토큰 인터셉터, 401 refresh) in `kbap-fe/lib/api/client.ts` (T006 의존)
- [ ] T017 [FE] i18n 초기화(i18next + expo-localization, **9개 로케일 리소스** en/zh-Hans/zh-Hant/ja/vi/id/th/ru/es, 하드코딩 금지 규약) in `kbap-fe/lib/i18n/` (헌법 I, FR-027) — 번역 리소스 적재는 T070, 폰트는 T071
- [ ] T018 [P] [FE] 디자인 시스템 프리미티브 + **위험도 4상태 아이콘 컴포넌트**(safe=원+체크 / caution=삼각+! / danger=팔각+X / unable=마름모+? · SVG, 기본 이모지 금지) in `kbap-fe/components/` (와이어프레임 검증본 이식, 헌법 추가제약)
- [ ] T019 [P] [FE] 앱 셸 + **5-탭 네비게이션**(home · food · scan[중앙 FAB] · **community[잠김, phase2 예약]** · profile) + 알림 패널 UI(정적, 푸시는 2차) + 온보딩 스택 라우팅 in `kbap-fe/app/` (hifi-shell 목업)
- [ ] T020 [FE] 안전 고지 상시 배너 컴포넌트(위험 화면 공통) in `kbap-fe/components/` (헌법 III, FR-030)

**Checkpoint**: 인증·DB·위험엔진·어댑터·디자인시스템 준비 완료 → 스토리 병렬 착수 가능

---

## Phase 3: User Story 1 - 온보딩 & 프로필 (P1) 🎯 MVP

**Goal**: 가입(이메일/애플/구글) → 식이 제약 필수 입력 → 안전 동의 → 프로필 확립
**Independent Test**: 가입 후 `GET /me`가 입력한 제약/국적/언어를 반환, 동의 기록 존재

### Tests for US1
- [ ] T021 [P] [BE] (test) 계약 테스트: `/auth/email/request-code`, `/auth/email/verify`, `/auth/social`, `/auth/refresh` in `kbap-be/src/test/kotlin/.../auth/`
- [ ] T022 [BE] (test) **통합 테스트: 온보딩 플로우** 가입 → 제약 **필수**(미입력 거부) → 동의 → `GET /me` 반영 in `kbap-be/src/test/kotlin/.../user/` (FR-003/030)

### Implementation for US1
- [ ] T023 [BE] [US1] 이메일 인증코드 발급/검증 구현 `POST /auth/email/request-code`, `POST /auth/email/verify` in `kbap-be/.../auth/`
- [ ] T024 [BE] [US1] 소셜 로그인 `POST /auth/social`(Apple/Google idToken 검증) + `POST /auth/refresh` in `kbap-be/.../auth/`
- [ ] T025 [BE] [US1] `GET /me`, `PATCH /me`(국적·언어·제약·spiceTolerance) — 제약 **최소 입력 필수** 검증(FR-003) in `kbap-be/.../user/`
- [ ] T026 [BE] [US1] `POST /me/consent`(안전 고지 동의 이력) in `kbap-be/.../user/` (FR-030, 민감정보 취급)
- [ ] T027 [P] [FE] [US1] 온보딩 가입 화면(이메일/애플/구글) in `kbap-fe/app/(onboarding)/` (Onboarding 와이어프레임)
- [ ] T028 [FE] [US1] 식이 제약 입력 화면(알러지/종교/식이, **건너뛰기 불가**) in `kbap-fe/app/(onboarding)/` (FR-003)
- [ ] T029 [FE] [US1] 안전 고지 동의 화면 → `POST /me/consent` in `kbap-fe/app/(onboarding)/`
- [ ] T030 [FE] [US1] 프로필 상태 연동(React Query, 로그인 토큰 secure storage) in `kbap-fe/features/auth/`

**Checkpoint**: 가입~프로필 독립 동작(SC-001 5분 이내 목표)

---

## Phase 4: User Story 2 - 메뉴판 스캔 → 위험도 오버레이 (P1) 🎯 MVP

**Goal**: 메뉴판 촬영 → 온디바이스 OCR → BE 매칭/번역/위험도 → 촬영 이미지 위 오버레이 + 원본↔번역 토글
**Independent Test**: 충돌메뉴=danger, 변동성분=caution, 미등록=unable(+로그), 오버레이/토글 동작

### Tests for US2
- [ ] T031 [P] [BE] (test) 계약 테스트: `POST /scan/menu`, `GET /foods/{foodId}` in `kbap-be/src/test/kotlin/.../scan/`
- [ ] T032 [BE] (test) **E2E 테스트: 스캔→위험도** 원문 배열 → 매칭 → danger/caution/**unable** 분기 + 미등록 로그 적재 검증 in `kbap-be/src/test/kotlin/.../scan/` (SC-003, FR-033)

### Implementation for US2
- [ ] T033 [BE] [US2] `POST /scan/menu` 구현: 원문 배열 → 큐레이션 카탈로그 매칭(T012) → 표시명(매칭=카탈로그 현지화명, 미매칭=번역 T013) → 위험도(T014) → `rawText` 키로 결과 반환 in `kbap-be/.../scan/` (FR-008/031/037)
- [ ] T034 [BE] [US2] 미등록(미매칭) 메뉴 → `unable` + `UnregisteredFoodLog`·`ScanItemLog` 적재 in `kbap-be/.../scan/` (FR-033, 헌법 III)
- [ ] T035 [BE] [US2] `GET /foods/{foodId}` 디테일(이름/평점/설명≤150자/사진/성분 위험순 + riskBasis) in `kbap-be/.../food/` (FR-012/014)
- [ ] T036 [P] [FE] [US2] 카메라 촬영 + **온디바이스 OCR**(ML Kit text-recognition, 텍스트+bounding box 기기 보관) in `kbap-fe/features/scan/` (D4, 이미지 미전송)
- [ ] T037 [FE] [US2] OCR 원문 배열 → `POST /scan/menu` 전송 + 응답 매핑 in `kbap-fe/features/scan/`
- [ ] T038 [FE] [US2] **결과 오버레이**: 응답 `rawText`를 온디바이스 bounding box에 정합, 위험도 4상태 색/아이콘 표시 in `kbap-fe/features/scan/` (Camera 와이어프레임, FR-008)
- [ ] T039 [FE] [US2] **원본↔번역 토글** 버튼(파파고/렌즈식) in `kbap-fe/features/scan/` (FR-037)
- [ ] T040 [FE] [US2] 음식 디테일 화면(`GET /foods/{id}`, 성분 위험순, riskBasis 표시) in `kbap-fe/app/(tabs)/food/[id].tsx` (FoodDetail 와이어프레임)

**Checkpoint**: 스캔→위험 핵심 루프 동작(SC-002 평균 10초, SC-003 false-safe 0)

---

## Phase 5: User Story 3 - 사장님께 한국어 확인 (P2)

**Goal**: '주의' 성분을 한국어 문구로 사장님께 확인. 메뉴명은 스캔 인식명과 정확 일치
**Independent Test**: owner-confirmation의 `menuNameKo`가 스캔 메뉴명과 일치(SC-006), `questionKo` 한국어

### Tests for US3
- [ ] T041 [P] [BE] (test) 계약 + **메뉴명 일치** 테스트: `GET /foods/{foodId}/owner-confirmation` (`menuNameKo` == scannedMenuName) in `kbap-be/src/test/kotlin/.../food/` (SC-006)
- [ ] T042 [BE] (test) **통합 테스트: 주의 성분 → 한국어 확인 문구** 생성(질문/알러지 설명 ko, place=ko 데이터) in `kbap-be/src/test/kotlin/.../food/` (FR-018/019)

### Implementation for US3
- [ ] T043 [BE] [US3] `GET /foods/{foodId}/owner-confirmation?ingredientCode=&scannedMenuName=` → `OwnerConfirmation`(place=ko 데이터 모델, 질문/알러지 설명 ko) in `kbap-be/.../food/` (FR-018/019, 헌법 I place-as-data)
- [ ] T044 [FE] [US3] 사장님 확인 화면(디테일의 '주의' 성분 선택 → 한국어 문구 카드, place 언어 라벨링) in `kbap-fe/features/owner-confirm/` (OwnerConfirm 와이어프레임)

**Checkpoint**: US1~US3 = MVP 핵심 7/10 흐름 완성

---

## Phase 6: User Story 4 - 음식 탐색 & 리뷰 (P2)

**Goal**: 음식 검색/탐색, 음식 단위 리뷰 조회(전체/같은 국적 집계, 번역) 및 작성
**Independent Test**: `GET /foods?q=`, 리뷰 집계·번역·같은국적 필터, `POST` 리뷰 후 집계 갱신

### Tests for US4
- [ ] T045 [P] [BE] (test) 계약 테스트: `GET /foods`, `GET /foods/{foodId}/reviews`, `POST /foods/{foodId}/reviews` in `kbap-be/src/test/kotlin/.../review/`
- [ ] T046 [BE] (test) **통합 테스트: 리뷰 작성 → 집계 갱신** rating 1–5 작성 후 전체/같은국적 집계 반영 검증 in `kbap-be/src/test/kotlin/.../review/` (FR-022)

### Implementation for US4
- [ ] T047 [BE] [US4] `GET /foods?q=`(음식 중심 탐색, 식당 엔티티 없음 — 헌법 II) in `kbap-be/.../food/`
- [ ] T048 [BE] [US4] `GET /foods/{foodId}/reviews`(전체/같은국적 집계, `translateTo` 번역, `nationality` 필터) in `kbap-be/.../review/` (FR-022)
- [ ] T049 [BE] [US4] `POST /foods/{foodId}/reviews`(rating 1–5 정수, 국적/티어 스냅샷) + RatingAggregate 갱신 in `kbap-be/.../review/`
- [ ] T050 [P] [FE] [US4] 음식 탐색 탭/검색 화면(`GET /foods`) in `kbap-fe/app/(tabs)/food/index.tsx` (FoodTab 와이어프레임)
- [ ] T051 [FE] [US4] 리뷰 조회(전체/같은국적 토글, 번역 버튼) in `kbap-fe/features/reviews/`
- [ ] T052 [FE] [US4] 리뷰 작성 화면(별점 1–5, 본문) → `POST` in `kbap-fe/features/reviews/` (ReviewCompose 와이어프레임)

**Checkpoint**: 탐색/리뷰 독립 동작

---

## Phase 7: User Story 5 - 프로필 & 랭킹 & 계정삭제 (P3)

**Goal**: 내 리뷰/랭킹 티어 조회, 계정 삭제 시 민감정보 즉시 완전삭제 + 리뷰 익명화 보존
**Independent Test**: `GET /me/reviews`·`GET /me/ranking` 동작, `DELETE /me` 후 리뷰 `anonymized=true`(평점 유지)

### Tests for US5
- [ ] T053 [P] [BE] (test) 계약 테스트: `GET /me/reviews`, `GET /me/ranking`, `DELETE /me` in `kbap-be/src/test/kotlin/.../user/`
- [ ] T054 [BE] (test) **E2E 테스트: 계정삭제 → 익명화 보존** `DELETE /me` 후 개인·민감정보 완전삭제 + 본인 리뷰 `userId=null`·`anonymized=true`·평점/본문 보존 검증 in `kbap-be/src/test/kotlin/.../user/` (FR-032, 헌법 IV)

### Implementation for US5
- [ ] T055 [BE] [US5] `GET /me/reviews` in `kbap-be/.../user/`
- [ ] T056 [BE] [US5] `UserActivityScore` 복합 점수(리뷰+음식다양성+스캔 가중합) + tier 산출, `GET /me/ranking` in `kbap-be/.../ranking/` (FR-025, MVP 범위 — 커뮤니티 활성도 제외)
- [ ] T057 [BE] [US5] `DELETE /me`: 개인·민감정보 즉시 완전삭제 + 본인 리뷰 `userId=null`·식별필드 제거(평점/본문 보존) in `kbap-be/.../user/` (FR-032, 헌법 IV)
- [ ] T058 [P] [FE] [US5] 프로필 화면(내 리뷰, 랭킹 티어) in `kbap-fe/app/(tabs)/profile/` (Profile 와이어프레임)
- [ ] T059 [FE] [US5] 계정 삭제 플로우(확인 모달 → `DELETE /me`) in `kbap-fe/features/profile/`

**Checkpoint**: 프로필/랭킹/삭제 동작

---

## Phase 8: User Story 6 - 홈 개인화 허브 (P3)

**Goal**: 최근 스캔/조회 + 개인화 추천을 위험도와 함께 보여주는 홈
**Independent Test**: `GET /home`이 recent+recommended 반환, 신규 사용자 빈 상태 정상 표기

### Tests for US6
- [ ] T060 [P] [BE] (test) 계약 테스트: `GET /home` in `kbap-be/src/test/kotlin/.../home/`
- [ ] T061 [BE] (test) **통합 테스트: 홈 빈상태/채워진상태** 신규 사용자 0건 + 스캔 후 recent/recommended 반영 in `kbap-be/src/test/kotlin/.../home/` (FR-034)

### Implementation for US6
- [ ] T062 [BE] [US6] `GET /home`: `recent`(ScanItemLog 기반 최근 + 위험도) + `recommended`(개인화 위험도) in `kbap-be/.../home/` (FR-034)
- [ ] T063 [FE] [US6] 홈 화면(최근/추천 카드 + 위험도, 빈 상태) in `kbap-fe/app/(tabs)/home/` (Home 와이어프레임)

**Checkpoint**: 전 스토리 독립 동작

---

## Phase 9: Polish & 교차 관심사

- [ ] T064 [P] [SPEC] quickstart.md end-to-end 검증 시나리오 실행 + 합격 기준(SC) 확인
- [ ] T065 [BE] 위험도/번역 결과 캐싱(성능 SC-002) + 미등록 로그 집계 점검
- [ ] T066 [P] [FE] i18n 누락 키 점검(하드코딩 0) + 기본 이모지 사용 0 점검(헌법 추가제약) + **9개 언어 레이아웃/폰트 깨짐(두부) 0 QA**(긴 번역·CJK·태국·키릴 포함)
- [ ] T067 [P] [FE] 접근성/빈상태/에러상태(States 와이어프레임) 마감
- [ ] T068 [SPEC] 계약 변경분 openapi.yaml 역반영 확인(FE/BE drift 0, 헌법 VI)
- [ ] T069 보안 점검(JWT 만료/refresh, 민감정보 로그 마스킹, idToken 검증)
- [ ] T070 [FE] **9개 언어 i18n 리소스 번들 작성**(en/zh-Hans/zh-Hant/ja/vi/id/th/ru/es) — 기계번역 초안 + 핵심 화면(온보딩/위험도/사장님확인) 검수, 키 누락 0 in `kbap-fe/lib/i18n/locales/` (FR-027)
- [ ] T071 [FE] **다중 스크립트 폰트 통합**(라틴=Baloo2/NunitoSans, CJK=Noto Sans SC/TC/JP, 태국=Noto Sans Thai, 키릴=NunitoSans/Noto) + 로케일별 폰트 폴백 매핑 + 글자 깨짐(두부) 0 검증 in `kbap-fe/lib/theme.ts`·폰트 로더 (FR-027, 헌법 I)

---

## Dependencies & Execution Order

### Phase 의존성
- **Setup(P1)**: 즉시 시작. T006/T007은 T001 이후
- **Foundational(P2)**: Setup 완료 후, 모든 스토리 차단
- **User Stories(P3~P8)**: Foundational 완료 후 병렬 가능 / 또는 우선순위 순(US1→US2→US3→...)
- **Polish(P9)**: 원하는 스토리 완료 후

### 스토리 의존성
- **US1(P1)**: Foundational 후 독립
- **US2(P1)**: Foundational 후 독립(위험엔진 T014·어댑터 T012/T013 사용). US3가 US2 디테일을 활용
- **US3(P2)**: Foundational 후. US2의 디테일/스캔명과 통합되나 독립 테스트 가능
- **US4/US5/US6(P2/P3)**: Foundational 후 각각 독립

### 스토리 내부 순서
- (test) 작업 → 실패 확인 → 구현 (C안: 계약+단위+통합/E2E)
- 모델 → 서비스 → 엔드포인트 → 통합
- BE 엔드포인트와 FE 화면은 계약(openapi.yaml) 기준으로 **병렬** 진행 가능

### 병렬 기회
- [P] 표기 작업은 서로 다른 파일/레포 → 병렬 가능
- Foundational 완료 후, FE 개발자와 BE 개발자가 같은 스토리를 계약 기준으로 동시 진행
- BE T012(큐레이션 stub)/T013(번역)은 외부 팀 확정 전에도 인터페이스로 병렬 진행

---

## Implementation Strategy

### MVP First (권장, 마감 2026-07-10)
1. Phase 1 Setup → Phase 2 Foundational
2. **US1(온보딩) → US2(스캔→위험도)** = 안전 핵심 루프 → **여기서 검증/데모(MVP 코어)**
3. US3(사장님 확인) 추가 → US1~US3 = quickstart 7/10 필수 흐름 완성
4. 여유 시 US4 → US5 → US6

### 외부 통합 미확정 대응
- 큐레이션 DB 접근 방식(공유 DB vs 내부 API) 미확정 → T012 stub로 진행, 확정 시 어댑터 구현만 교체
- 번역 제공자 미확정 → T013 기본 구현 + 어댑터로 교체 가능

---

## 작업 요약

- 총 **71개** 작업(T001~T071) — 다국어 9개(B안): T070(리소스), T071(다중 스크립트 폰트)
- 테스트 **13개**: 단위 1(T015) · 계약 7(T021/T031/T041/T045/T053/T060 + auth) · 통합/E2E 5(T022/T032/T042/T046/T054/T061)
- Repo 분포: SPEC 3 · BE 다수 · FE 다수
- MVP 코어(P1) = US1+US2, 핵심 7/10 = US1~US3

## Notes
- [P] = 다른 파일/레포, 의존성 없음
- [Repo]/[Story] 라벨로 FE/BE 개발자 분담 + 추적
- 헌법 III(안전): unable/caution 강등 우선, 근거 없는 safe 금지 — T014/T015/T032에서 강제
- 헌법 VI(SSOT): 계약 변경은 spec 레포 openapi.yaml에서 먼저 → FE/BE 재생성
- 각 작업/논리 단위 후 커밋, 체크포인트에서 스토리 독립 검증
