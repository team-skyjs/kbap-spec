# FE 작업 프롬프트 큐

> 운영: 커맨드 센터(스펙 레포 세션)가 항목 작성·수정. **정렬 = 처리 순서** (최상단이 가장 먼저
> 할 일): 긴급하거나 선후 관계(1→2)가 있으면 먼저 할 것을 위로, 순서 무관하면 신규를 최상단에.
> FE 세션은 **위에서부터** ⬜ 항목을 처리하고(예진이 ID 지목 시 그것 우선) **상태만** 갱신:
> ⬜ 대기 → 🔄 착수 → ✅ 완료(커밋 해시 병기). 본문 수정은 커맨드 센터만.
> 완료 보고는 [REPORTS.md](REPORTS.md) 최상단에 같은 ID로 작성.

---

## [P-058] ✅ KB-125 랭킹 후속 — 음식 다양성(Food variety) 행도 dim 예고 처리 — `762f3b7` (preview OTA)

예진 지시(7/22): 다양성 점수는 **리뷰 작성에서 적립**되는 구조 — 리뷰 기능이 MVP 제외인 지금은 다양성도 죽은 지표. **P-048의 리뷰 팩터와 동일하게** dim 예고 처리.

### 할 일

1. `ranking.tsx` 다양성 BreakRow를 리뷰 행과 **똑같은 처리**: opacity 0.55 + IconLock + `ranking.reviewsComing`("다음 업데이트에 제공돼요") 라벨 재사용, 탭 무반응. 점수 로직(ranking.ts) 무변
2. 남는 활성 팩터 = **스캔 1개** — 카드 시각 균형 확인(첫 행 border 처리 등)
3. 테스트: 다양성 행 dim 렌더 잠금(리뷰 행 테스트와 동형)
4. JS-only → preview OTA

### DoD

- [ ] 다양성 행 dim+예고(리뷰 행과 동일 톤) · 스캔 행만 활성 · tsc 0 · jest · OTA

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-058].

## [P-057] ✅ KB-212 후속 — 스캔 배너를 리스트 첫 카드로 편입 (A안 확정) — `6f07859` (preview OTA)

실기(예진 7/22): P-038 배너가 어두운 오버레이라 배경에 묻히고 첫 메뉴 카드에 밀착 — 목업 비교 후 **A안 확정**: 오버레이 폐기, 리스트 첫 카드로 편입.

### 확정 스펙 (목업 기준 — scratchpad/scan-banner-mockup.html A안)

1. absolute 오버레이 제거 → 배너를 **결과 리스트의 첫 아이템**으로 렌더 (메뉴 카드와 같은 폭·radius·마진 리듬)
2. 스타일: **밝은 브랜드 틴트 카드** — 배경 `#fdf0e6`(primaryTint 계열)·테두리 `#f0d9c4`·본문 진갈색+**"기피 재료를 설정"만 primary 강조**·좌측 원형 primary 아이콘(SVG — 전구/설정 계열)·우측 ×(닫기)
3. 동작 무변: 탭→기피 설정, ×→세션 억제, 노출 조건(회원&&기피0) 그대로 — 기존 scanNudge 테스트 로직 유지, 렌더 위치만 이동
4. 리스트/위험도/원본 토글 시: 배너는 **목록 뷰에서만**(위험도·원본은 사진 위라 부적합)
5. 기존 잠금 테스트 갱신(오버레이→리스트 아이템) + 스태거(P-032)와 간섭 없는지(배너 포함 순차 등장 OK)

### DoD

- [ ] 배너=리스트 첫 카드(밝은 틴트·정상 간격) · 목록 뷰 전용 · 동작/조건 무변 · tsc 0 · jest · preview OTA

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-057].

## [P-056] ✅ 🔴 KB-176 안드 스캔 오버레이 좌표 2배 오프셋 — 정규화 분모 불일치 (adb 실측 확정) — `c44028f` (preview OTA)

실기+adb 로그(7/22)로 **원인 확정**:

```
[ocr] dims reported = 1883×4080 | measured(pixel) = 942×2040(→채택) 
[ocr] "부대찌개" frame top=2051   ← 분모 2040보다 큼 → norm y=1 클램프
```

- 크롭 실치수 1883×4080, 저장 파일은 **정확히 절반(942×2040)으로 다운스케일**됨(안드 저장 경로)
- **ML Kit 좌표는 원치수(1883×4080) 기준**인데 정규화 분모는 measured(942×2040) → **y 2배 뻥튀기**
- 결과: 김치(0.32→0.65 표시)·된장(0.39→0.77)만 화면 안, **나머지는 y≥1 클램프로 미표시** — "마커 2개"도 같은 버그(photoOnly 아님)
- iOS는 measured=ML Kit 기준이 일치해 정상. **KB-141의 교훈("reported 무시, measured 채택")이 안드에선 정반대** — 플랫폼별로 ML Kit이 참조하는 기준이 다름

### 할 일

1. 정규화 분모를 **ML Kit이 실제 참조한 치수**로: 안전한 방식 = reported vs measured 중 **frame 최대 좌표를 수용하는 쪽 채택**(플랫폼 하드코딩보다 견고 — 두 치수가 정수배 관계면 스케일 보정도 동치). 채택 근거를 로그로 남김
2. 안드 실기/에뮬에서 6개 마커 전부 정위치 확인 · **iOS 무회귀 필수**(KB-141 회귀 금지 — iOS는 measured가 정답이었음)
3. (부수 확인) 크롭 파일이 절반으로 저장되는 것 자체는 조사만 — OCR 인식은 되고 있으니 화질 이슈 여부만 보고
4. 테스트: 치수 선택 로직 잠금(안드형: frame>measured→reported / iOS형: measured 채택)
5. JS-only → preview OTA

### DoD

- [ ] 안드 오버레이 6개 마커 전부 해당 메뉴 옆 정위치 · iOS 무회귀 · tsc 0 · jest · OTA

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-056].

## [P-055] ✅ KB-225 Android 하단 내비바 클리어런스 전수 — 시트 '나중에 하기' 짤림 — `1c8a09c` (preview OTA)

실기(예진 7/22, 스크린샷): 게스트 게이트 시트(AuthGateSheet) 하단 "나중에 하기"가 **3버튼 내비바에 짤림**. P-021(온보딩 CTA)과 동일 원인 — 안드 edge-to-edge에서 insets.bottom 과소보고. P-021이 온보딩에만 국소 처치라 **다른 하단 고정 UI에 동일 병 잠복**.

### 할 일

1. **공용 유틸 승격**: P-021의 `Platform.OS==='android' ? Math.max(insets.bottom, 48) : insets.bottom` 을 훅/유틸(예: `useBottomInset`)로 추출 — 온보딩 것도 교체
2. **전수 적용** (insets.bottom 사용처 실측 목록 — 커맨드 센터 grep):
   - `AuthGateSheet`(이번 짤림 — 최우선) · `login.tsx` · `intro.tsx` · `food/[id]/owner.tsx` · `food/[id]/order.tsx` · `scan.tsx`(2곳)
   - `TabBar`는 기존 `Math.max(insets.bottom, 10)` — 실기 정상이면 무변(과보정 주의), 짤리면 동일 유틸
3. **화면별 실기 확인 목록을 보고에** — 3버튼 내비 기기 기준(에뮬 설정으로 3버튼 전환 가능)
4. iOS는 실측 인셋 그대로(무회귀). 테스트: 유틸 분기(안드 max48/iOS 통과) 잠금
5. JS-only → preview OTA

### DoD

- [ ] 게이트 시트 '나중에 하기' 포함 하단 고정 UI 전부 내비바 안 짤림(3버튼 기준) · iOS 무회귀 · tsc 0 · jest · OTA

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-055].

## [P-054] ✅ (재빌드·안드) KB-194 후속 — Android 12+ 스플래시 원형 마스크 잘림 — `18de75a` (OTA 불가·안드 빌드3 합류)

실기(예진 7/22, 안드 빌드2): 스플래시의 로고+워드마크가 **원형으로 클립**돼 "K-Bar"처럼 잘림. 원인 = **Android 12+ 시스템 스플래시 규칙**(앱 이미지가 OS의 원형 아이콘 마스크 안에 강제 배치 — windowSplashScreenAnimatedIcon). 세로 조합 이미지(로고+K-Bap)가 원을 벗어나는 부분이 잘리는 것. iOS는 마스크 없음 → 정상.

### 할 일

1. **안드 전용 스플래시 이미지 분리**: expo-splash-screen 플러그인의 플랫폼별 설정으로 `android.image`에 **로고 심볼만**(K 볼 글리프, 정사각·원형 안전영역 안에 여백 포함 — 대략 캔버스의 2/3 이내) 지정. 기존 `splash-icon.png`(로고+워드마크)는 iOS 전용으로
2. 에셋: 기존 png에서 로고 심볼부만 크롭해 정사각 캔버스+투명 여백으로 산출(P-018 방식). 배경색 `#E2580C` 무변
3. ⚠️ **네이티브 — 재빌드 필요(OTA 불가)**: 다음 안드 빌드(빌드3)에 실림. **제출용 최종 빌드 때 합류**(단독 빌드 불요 판단 — 쿼터 절약, 커맨드 센터 의견) — 단 코드·에셋은 지금 커밋
4. 결과 미리보기 png 보고 첨부(원형 마스크 시뮬 — 원 안에 로고 온전히)

### DoD

- [ ] app.json 플랫폼 분리·안드용 에셋 커밋 · iOS 스플래시 무변 · tsc 0 · jest · (실기 확인은 안드 빌드3에서)

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-054].

## [P-053] ✅ KB-206 반려 — 게스트 로그인 바텀시트 애니메이션 원복 — `3ef625f` (preview OTA)

실기(예진 7/22): 비로그인 헤더 알림 아이콘·스캔 진입 시 뜨는 **로그인 유도 바텀시트(AuthGateSheet)의 등장 애니메이션이 이상함 — 원래대로 롤백** 결정. P-031 B5의 `SlideInDown.springify()`가 원인(P-047 헤더 timing 복귀와 같은 계열 — 이 시트엔 스프링이 안 맞음).

### 할 일

1. AuthGateSheet 등장을 **P-031 이전 방식으로 원복**(기존 timing/Modal 전환 — git 이력에서 P-031 직전 상태 확인). 퇴장·backdrop 무변
2. 같은 B5에서 바꾼 **Snackbar 등장 스프링은 유지**(반려 대상 아님 — 시트만)
3. 제거 사유 주석(P-035·047 선례) — 절제 사례 축적
4. tsc 0 · jest 무회귀 · preview OTA

### DoD

- [ ] 알림 아이콘·스캔 진입 게이트 시트가 예전처럼 자연스럽게 등장 · 타 모션 무변 · OTA

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-053].

## [P-052] ✅ 🔴 KB-215 반려 — 사장님 질문에 재료 원문 노출 ("ing:0:Egg이(가) 들어가나요") — `82d54b9` (preview OTA)

Q 실기(예진 7/22, 스크린샷): "쫄면에 **ing:0:Egg**이(가) 들어가나요?" — 사장님 노출 문구에 내부 식별자 원문. **P-045의 구멍**: 상세 API IngredientResponse엔 **81종 코드가 없고 name(요청 언어 성분명)뿐** — 어댑터의 합성 라우트 키(`ing:{i}:{name}`, foodAdapter.ts L36)가 ownerQuestionKo에 코드로 전달돼 매핑 실패 → 원문 통과.

### 할 일

1. **원문 통과 원천 봉쇄(안전)**: `ownerQuestionKo`에서 라벨 해석 실패 시 **일반 질문**("{음식명}에 제가 못 먹는 재료가 들어가나요?")으로 강등 — 어떤 입력이든 내부 식별자가 사장님 화면에 뜨지 않게 (false-safe 계열: 불확실→일반 질문)
2. **ko 라벨 실해석**: 합성 키에서 name 추출(`ing:{i}:{name}` 파싱) 후 **역매핑** — 81종의 현재 언어 라벨·en 라벨·ko 라벨 사전에서 name 일치 검색 → 코드 → ko 라벨. (상세 name은 요청 언어라 en/현지어로 옴 — FE i18n 81종 라벨로 역인덱스 구축, 대소문자·공백 정규화)
3. 역매핑도 실패(서버 번역이 FE 라벨과 상이)하면 1의 일반 질문 — **원문보단 덜 구체적인 게 항상 낫다**
4. 테스트: `ing:0:Egg`→"달걀이(가)" / 미지 name→일반 질문 / 직접 코드(EGG)→기존 동작 — 3분기 잠금
5. JS-only → preview OTA 즉시 (사장님 노출 문구라 우선)

### DoD

- [ ] 어떤 재료든 사장님 질문에 원문/식별자 노출 0 (ko 라벨 or 일반 질문) · tsc 0 · jest · OTA

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-052].

## [P-051] ✅ KB-195 후속 — 맵기 스텝 UI 원복: 기본 5 표시, Skip만 -1 (P-039 UI 롤백) — `13be68e` (preview OTA)

Q-03 실기 피드백(예진 7/22): P-039의 미선택 UI("–/10"+회색 불꽃+힌트)가 **이상함** — 원복 결정. **화면 그대로 제출** 원칙으로 확정(예진 7/22):

- 기본 상태: 슬라이더 **5 표시**(P-039 이전처럼 불꽃 5개 활성, "–/10"·"Tap a flame to choose" 힌트 제거)
- **Continue(미조작 포함) = 화면 값 그대로 제출**(기본이면 5) — 표시=전송 일치
- **Skip(건너뛰기) = -1** (P-019 경로 유지)

### 할 일

1. `onboarding/index.tsx` spice 스텝: state 기본값 5 복원, 미선택 전용 UI(대시·힌트·회색 불꽃) 제거. P-039의 **stale closure 수정(finish 인자화·값 단일 진실)은 유지** — UI만 롤백
2. draft 왕복: 저장된 값 있으면 그 값, 없으면 5 표시 (null draft 호환)
3. 테스트 갱신: 미조작+Continue→**5** / 조작(7)+Continue→7 / Skip→-1 — P-039 테스트 교체
4. i18n 힌트 키(spiceUnsetHint 등) 미사용화 정리
5. JS-only → preview OTA

### DoD

- [ ] 기본 5 표시·힌트 없음 · Continue=화면값(기본 5)·Skip=-1 · tsc 0 · jest · OTA

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-051].

## [P-045] ✅ KB-215 사장님 확인 카드 실데이터 조립 — '이 음식' mock 잔재 (안전 우선) — `23a447a` (preview OTA)

멘토링 발견(7/21): danger 재료 ask owner → "이 음식에 제가 못 먹는 재료가 들어가나요" — 원인은 owner 카드가 **mock**(`src/lib/mocks/owner.ts` PHRASES 사전 2개 음식만 실명, 나머지 DEFAULT '이 음식').

### 할 일
1. mock 폐기 → **실데이터 클라 조립**: 상세 캐시의 `koreanName` + `?ingredient=` 코드→ko 라벨(81종 매핑)로 `{음식명}에 {재료ko}이(가) 들어가나요?` 생성 (P-030 을/를 유틸 재사용). ingredient 없는 진입(unregistered)은 `{음식명}에 제가 못 먹는 재료가 들어가나요?` — koreanName은 항상 실명
2. explanationKo(알레르기 설명)는 정적 유지 OK. renderQuestion 하이라이트는 실명 기준으로 자연 동작 확인
3. **사장님 노출 화면 전수**: owner.tsx·order.tsx에서 음식명·재료명이 mock/하드코딩으로 남은 곳 grep — 목록+처리 보고
4. 테스트: 조립 함수(재료 있음/없음/을·를) 잠금
### DoD
- [ ] 어느 음식이든 실명 질문 · 전수 결과 보고 · tsc 0 · jest · OTA(스왑 불요)

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-045].

## [P-046] ✅ KB-216 스캔 오프라인 — 진입 시 전체 J4 — `e7ee492` (preview OTA)

멘토링 발견: 오프라인에도 스캔 진입·촬영 가능. 결정(예진): **진입 시 전체 오프라인 화면**(다른 탭과 동일 J4, 카메라 미기동).

### 할 일
1. `scan.tsx` 진입 시 오프라인이면 카메라 대신 전체 QueryErrorBlock(J4)+Retry — 판정은 P-027/028과 같은 메커니즘 재사용. Retry로 온라인 복귀 시 카메라 기동
2. 갤러리 스캔 경로도 동일 차단. 테스트: 오프라인→J4·카메라 미렌더 잠금
### DoD
- [ ] 기내모드 스캔 진입=J4(촬영 불가) · 복귀 정상 · tsc 0 · jest · OTA

완료 시 상태 ✅+커밋 해시, 보고는 [P-046].

## [P-047] ✅ KB-217 헤더 스크롤 바운싱 — 빠른 방향 반복 시 이상 진동 — `ea552db` (preview OTA)

멘토링 발견: 스크롤 올리고/내리고 빠르게 반복하면 헤더가 이상 바운싱. **P-031이 StickyHeader 숨김/복귀를 timing→withSpring으로 바꾼 것과 연관 의심**(방향 전환마다 스프링 재타겟 → 미정착 상태 연속 재타겟 진동).

### 할 일
1. 원인 실조사(보고에 명시) 후 수정 — 유력안: 방향 전환 히스테리시스(임계 스크롤량)로 재타겟 빈도 억제, 또는 이 헤더만 damping 상향/timing 복귀(절제 원칙 — 스크롤 크롬은 즉답이 우선, 바운스 불필요)
2. 홈·음식탭 등 StickyHeader 전 사용처 확인. 테스트: 기존 스위트 무회귀
### DoD
- [ ] 빠른 반복 스크롤에서 진동 없음 · 정상 숨김/복귀 무변 · tsc 0 · jest · OTA

완료 시 상태 ✅+커밋 해시, 보고는 [P-047].

## [P-048] ✅ KB-125 랭킹 리뷰 흔적 정리 — '하나 더' 제거 + 리뷰 팩터 dim 예고 — `4ec6823` (preview OTA)

결정(7/22 예진): ① **하단 "리뷰 쓰기" CTA 버튼(ranking.tsx L163 ctaReview — /food로 보내는 그것) 제거** ② breakTotal "리뷰 하나 더 +10점" 행 **제거** ③ 숨김 처리된 리뷰 팩터 행(FLAGS.reviewsEnabled)을 **dim으로 노출**: 흐림+자물쇠+"다음 업데이트에 제공" 라벨(i18n ×10) — 리뷰 기능 예고. 점수 로직 무변. 스캔 CTA는 유지.

### 할 일
1. ranking.tsx: **ctaReview CTA 삭제**(스캔 CTA 레이아웃 정리) + oneMore 행 삭제(하단 radius P-037 처리와 충돌 없게 — 마지막 행 카드 마감 확인) / 리뷰 BreakRow를 FLAGS 무관 상시 렌더하되 disabled 스타일(opacity+IconLock)+예고 라벨, 탭 무반응
2. 음식 상세·프로필의 리뷰 흔적은 **범위 외**(이미 FLAGS 숨김 유지 — 랭킹만)
3. 테스트: oneMore 부재·리뷰 행 dim 렌더 잠금
### DoD
- [ ] 리뷰 쓰기 CTA 없음 · 하나 더 행 없음 · 리뷰 팩터 dim+예고 · i18n ×10 · tsc 0 · jest · OTA

완료 시 상태 ✅+커밋 해시, 보고는 [P-048].

## [P-049] ✅ KB-218 프로필 사진 촬영 — 시스템 카메라 + 네이티브 선택 시트 — `7fcfb35` (preview OTA)

멘토링 요청: 업로드만 → **촬영 추가**. 직접 UI 제작 금지 — 시스템 것만.

### 할 일
1. 사진 변경 탭 → **네이티브 선택 시트**: iOS `ActionSheetIOS`(촬영/갤러리에서 선택/[사진 있으면]사진 삭제/취소), 안드는 `Alert` 3버튼(시스템 대화상자) — RN 내장만, 신규 라이브러리 금지
2. 촬영 = `ImagePicker.launchCameraAsync`(**기설치 expo-image-picker — 재빌드 불요 확인됨**, allowsEditing 1:1 크롭 기존 갤러리 경로와 동일 옵션) → 기존 업로드 파이프라인 합류. 권한 거부 시 정직한 안내(설정 유도)
3. 적용처: 프로필 수정 + 온보딩 사진 스텝(공용 경로 확인). 삭제는 기존 P-016 로직 재사용
4. iOS NSCameraUsageDescription은 expo-camera 플러그인 문구가 기존재 — 시뮬 아닌 실기서 권한 플로우 확인 명시
5. 테스트: 시트 분기(사진 유무별 옵션)·촬영 결과 업로드 호출 잠금
### DoD
- [ ] 촬영→크롭→업로드 동작(실기) · 시트 네이티브 · 갤러리/삭제 무변 · tsc 0 · jest · OTA

완료 시 상태 ✅+커밋 해시, 보고는 [P-049].

## [P-050] ✅ KB-219 CI — GitHub Actions: push/PR 시 tsc+jest — `b5832b9` (run #1 green)

결정(7/22): 출시 전 CI만(빌드/OTA 자동화는 출시 후 — 검토 게이트 원칙 유지).

### 할 일
1. `.github/workflows/ci.yml`: push(main)+PR에서 `npm ci` → `tsc --noEmit` → `jest --ci` (node 버전은 로컬과 일치, npm 캐시)
2. 실패가 병합을 막게(브랜치 보호는 예진 콘솔 몫 — 보고에 안내만)
### DoD
- [ ] 워크플로 1회 green 확인(푸시 후 Actions 링크 보고) · 로컬과 동일 결과

완료 시 상태 ✅+커밋 해시, 보고는 [P-050].

## [P-043] ✅ KB-176/198/202 — Android 빌드2 (preview apk, 네이티브 합류분 탑재) — 빌드 `24277e9c` (runtime f96ae4f7, 코드 무변)

iOS 빌드7 제출·재베이스라인(09093dd, runtime c43664ed) 완료 — **Android만 빌드2 미완**(보고 명시: build1용 preview 스왑이 빌드2 전까지 잔존). 공기계/에뮬 확인 대기분(P-022 가로 감지·P-025 캡처 크롭·P-032 스캔 효과·P-038 스캔 배너)이 전부 이 빌드에 물림.

### 할 일

1. `eas build --platform android --profile preview` — 현 main(재베이스라인 포함)으로. keystore 기존 EAS 관리 그대로
2. 완료 보고: 빌드 페이지 링크(설치 QR)·apk 직링크·runtime — **이후 preview OTA도 스왑 소멸** 확인 명시
3. 빌드 완료 후 preview 채널 기준 재정리(구 build1 runtime cbbec117은 orphan 처리 — 공기계는 새 apk 재설치가 기준)

### DoD

- [ ] 안드 빌드2 FINISHED + 설치 경로 보고 · 스왑 절차 완전 소멸 선언 · 예진 공기계/에뮬 재설치 → Q-14 잔여(가로·크롭·스캔 배너·스캔 효과) 확인 가능

완료 시 상태 ✅+커밋 해시(코드 무변이면 기록만), 보고는 REPORTS.md 최상단 [P-043].

## [P-042] ✅ Q-18 후속 3건 — press 피드백 확산 · 주문카드 스테퍼 위치 · 스텝 노드 팝 제거 — `1455b96` (preview OTA)

Q-18 실기(예진 7/21): 대비·큰글씨·칩 즉시·북마크/스테퍼 펄스·reduced-motion 통과. 후속 3건:

### 할 일

1. **press 피드백 확산** (Q-18 2번): 공용 Btn은 눌림 반응이 있는데 **헤더 게스트 "Sign in" pill**(StickyHeader signIn — 생 Pressable) 등 미적용 버튼 잔존. 생 Pressable로 만든 **탭 가능한 버튼류 전수 확인** 후 동일한 onPressIn scale(공용 프리셋 spring.press) 적용 — 리스트 행/카드 네비게이션은 제외(버튼 모양만), 과적용 금지
2. **주문카드 스테퍼 위치 이동** (Q-18 5번, 예진 기존 요청 — 기록 유실분 정식 등록): `order.tsx` 수량 스테퍼(− n +)를 현 위치(문장 아래)에서 **하단 "Done" 검정 버튼 바로 위**로 이동. 사장님 카드의 시선 흐름(문장은 위, 조작은 아래 손 근처) 개선. 문장 내 숫자 갱신 무변
3. **온보딩 스텝 노드 팝 제거** (Q-18 6번 "이상함"): P-032 ⑦(TopBar 세그먼트 노드 팝) 애니메이션 제거 — 즉시 전환. 성공 체크 드로잉(⑤)은 유지(통과). 기피 칩 P-035 선례와 동일하게 제거 사유 주석
4. tsc 0 · jest(스테퍼·노드 기존 테스트 갱신) · preview OTA (프로덕션 묶음 후보)

### 참고 (수정 아님)

- Q-18 6번 "셰이크 진동": 셰이크는 **화면 흔들림(시각)**이고 햅틱 진동은 미구현·계획 없음 — 예진 "없어도 됨" 확인으로 현행 유지

### DoD

- [ ] 헤더 Sign in 등 버튼류 press 반응 일관 · 스테퍼가 Done 바로 위 · 스텝 노드 즉시 전환 · 무회귀 · OTA

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-042].

## [P-041] ✅ 🔴 KB-152 재수정 — 신규 설치 부트 레이스: 프리페치가 토큰 정리보다 먼저 발사 (이전 사용자 홈 잔상) — `3010a8d` (preview+production OTA)

Q-05 실기(예진 7/21): 로그인→삭제→재설치→게스트 진입 시 **프로필·음식탭은 게스트 정상인데 홈에만 이전 계정(Hi Mina + avoid 8 things)이 표시**. 재시작하면 완전 게스트. **프라이버시 문제**(중고폰·공용폰에서 타인 데이터 노출).

**원인 확정** (`_layout.tsx` L53-56): `gateSplash({ ready, prefetch: prefetchBootData() })` — `cleanupIfFreshInstall()`(Keychain 정리)와 `prefetchBootData()`가 **같은 틱에 병렬 시작**. 신규 설치 첫 부팅에서 프리페치의 `hasBeSession()`이 아직 안 지워진 옛 토큰을 보고 /home·/me를 인증 상태로 프리페치 → 홈 캐시 오염. 정리 완료 후 다른 화면은 게스트(세션 체크), 홈만 오염 캐시 표시. 기존 주석("정리가 항상 먼저")은 렌더 기준 — P-018 프리페치가 네트워크 경로로 그 가정을 깸.

### 할 일

1. **프리페치를 정리 뒤로 직렬화**: `cleanupIfFreshInstall()` 완료 **후에** `prefetchBootData()` 시작 (예: cleanup promise를 분리해 `prefetch: cleanupDone.then(() => prefetchBootData())`). gateSplash의 min/cap 시맨틱 무변 — cleanup은 AsyncStorage 체크 1회라 비신규 설치에서도 지연 무시 가능
2. 주석 갱신: "정리가 프리페치·렌더 모두보다 선행" 으로 — 이번 레이스 경위 명시
3. 테스트: **신규 설치 시나리오 — cleanup 완료 전 prefetch 미시작** 잠금(순서 검증 모킹) + 기존 게이트 타이밍 테스트 무회귀
4. JS-only → preview+**production 즉시** OTA (프라이버시 건 — 프로덕션 우선 발행)

### DoD

- [ ] 재설치→게스트 진입 시 홈 포함 전 화면 게스트(이전 계정 잔상 0) · 정상 부팅 체감 무변 · tsc 0 · jest · 양 채널 OTA

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-041].

## [P-040] ✅ KB-205 재수정 — 스테퍼 −/+ 텍스트 글리프 → SVG 아이콘 (수직 중심 이탈) — `eb30502` (preview OTA)

Q-17 실기(예진 7/21, 인스펙터 실측): 주문카드 스테퍼의 −/+가 원(44×44) **정중앙보다 위**에 붙음. 컨테이너 정렬은 정확(center/center) — 원인은 **글리프가 아이콘이 아니라 텍스트**(order.tsx `stepText`: Baloo2_700Bold 22/lineHeight 26). 디스플레이 폰트의 어센트 편향이 시각 중심을 위로 밀어냄. **컨테이너 정렬로는 못 고치는 폰트 메트릭 문제.**

### 할 일

1. `order.tsx` StepBtn의 −/+ 를 **SVG 아이콘으로 교체**: `IconPlus` 기존 재사용 + `IconMinus` 신규(수평선 하나 — icons.tsx 한 줄). `stepText` 스타일 제거
2. 크기·색 기존 톤 유지(≈20, C.ink), 비활성(1·5 경계) 상태 표현 무변
3. **레포 전수 점검**: 문자를 아이콘처럼 쓰는 곳이 더 있는지(× · ± · ✓ 류 텍스트 렌더) grep — 있으면 목록만 보고(교체는 별도)
4. tsc 0 · jest(스테퍼 기존 테스트 통과) · preview OTA

### ⚠️ 영구 규칙 (이번에 CLAUDE.md에도 박음)

**기호를 텍스트로 렌더해 아이콘 대용하지 말 것.** 폰트 메트릭(어센트/디센트)이 시각 중심을 틀어서 컨테이너 정렬이 무력화됨 — 기호는 전부 SVG(icons.tsx). 헌법의 "이모지 금지·SVG only"와 같은 계열 규칙.

### DoD

- [ ] −/+ 원 정중앙(인스펙터 재확인 가능 수준) · SVG 교체 · 전수 점검 보고 · tsc 0 · jest · OTA

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-040].

## [P-039] ✅ 🔴 KB-195 재수정 — 맵기 스텝 기본값 5가 '선택'으로 제출됨 (미조작=미설정으로) — `4d3e9d8` (preview OTA)

Metro 실측(예진 7/21): 맵기 **슬라이더 미조작 + "계속"** → body에 `spicinessPreference: 5` 전송. 원인: `onboarding/index.tsx` `useState(5)` 기본값 + 계속(answerStep)이 skip 해제 → 기본 5가 선택으로 확정. 명시적 "건너뛰기"를 눌러야만 UNSET→-1. **사용자 멘탈모델(안 건드림=미설정)과 불일치** — 그간 "미설정=5" 혼선의 진짜 뿌리. P-019(-1 전송)는 UNSET일 때만 정상 — 플로우가 UNSET을 안 만들던 것.

### 할 일

1. 맵기 스텝을 **기본 미선택 상태**로: 값 state를 UNSET/null로 시작, **슬라이더를 조작해야만** 값 확정(트랙 탭/드래그 시 setLevel). 미조작 상태의 "계속" = **UNSET 제출**(건너뛰기와 동일 결과)
2. 미선택 상태 UI: 노브 미표시 or 중립 표시 + "밀어서 선택" 힌트 등 FE 재량 (기본 5로 보이는 착시 제거가 목표)
3. draft 저장/복원도 미선택(null) 왕복 유지. 명시적 "건너뛰기" 버튼은 존치(동작 동일해도 접근성·명료성)
4. 테스트: **미조작+계속 → body -1** (이번 구멍 그대로 잠금) / 조작 후 계속 → 실값 / 건너뛰기 → -1
5. 프로필 수정 화면은 무변(해제 UX 별도 — Q-03 1·2 통과 확인됨)

### DoD

- [ ] 미조작+계속=-1 · 조작=실값 · dev Metro 실측으로 body 확인 · tsc 0 · jest · preview OTA

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-039].

## [P-038] ✅ KB-212 빈 프로필 첫 스캔 배너 — 1줄 넛지 — `acedcd1` (preview OTA)

기획 확정(7/21 — **정본: `specs/001-personalized-menu-mvp/empty-profile-nudge.md`** 필독). 스킵-완료자(기피 0)의 가치 증명 순간(스캔 직후)에 비차단 1줄 배너.

### 할 일

1. 스캔 결과 상단: **회원 && 프로필 기피 0개**일 때만 1줄 배너 — "기피 재료를 설정하면 이 메뉴판에서 피할 걸 짚어드려요" 류 + 기피 설정 화면 링크. **게스트 제외**(게스트는 기존 로그인 게이트 흐름), 기피 1개 이상 미노출
2. 닫기(×) 가능 — 닫으면 **세션 내 재노출 억제**(영구 아님, 재실행 시 다시). 반복 모달 금지 원칙 준수(배너는 비차단이라 OK)
3. i18n ×10 · 이모지 금지(SVG) · 스캔 결과 기존 레이아웃(리스트/오버레이 토글) 안 밀게
4. 테스트: 노출 조건 3케이스(기피0 회원=노출/기피1+=미노출/게스트=미노출) + 닫기 억제
5. JS-only → preview OTA (프로덕션 묶음 후보)

### DoD

- [ ] 조건부 노출 정확 · 링크→기피 설정 · 비차단·닫기 가능 · i18n ×10 · tsc 0 · jest · OTA

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-038].

## [P-037] ✅ KB-125 랭킹 화면 UI 2건 — 현재등급 카드 정렬 + 점수내역 하단 코너 — `31e77e9` (preview OTA)

Q-15 실기(예진 7/21, 포토샵 기준선 스크린샷): 기능(실데이터·점수 증가·BE 반영)은 전부 통과. UI 2건:

**① 현재 등급 카드 정렬 어긋남** (`src/app/profile/ranking.tsx` 전체 등급 섹션): "맛보기(현재)" 하이라이트 카드가 자체 테두리·패딩을 가져 내부 컨텐츠가 다른 행들과 기준선 불일치 — 우측 "30점 이상"이 다른 행 "80점"들의 우측 끝보다 안쪽(카드 우측 패딩 과다), 좌측 "맛보기"도 "입문자/탐험가" 좌측 정렬과 어긋남. **수정**: 카드의 좌우 패딩을 조정해 내부 텍스트가 비카드 행들과 같은 좌/우 기준선에 앉게(카드 테두리는 그 바깥으로 나가는 형태 — 필요시 카드에 음수 마진). 스크린샷의 검정(우측 기준)·주황(현재 이탈) 세로선 참조

**② 점수 내역 카드 하단 코너 뚫림**: `breakTotal`("리뷰 하나 더/+10점" 행)이 `marginHorizontal:-16, marginBottom:-6`으로 카드 가장자리까지 배경(C.surface)을 확장하는데 **네모 배경이 breakCard의 borderRadius 밖으로 삐져나옴** → 하단 양 코너가 뚫려 보임. **수정**: breakTotal에 `borderBottomLeftRadius/borderBottomRightRadius`(카드 radius − border 두께) 부여. ⚠️ breakCard에 overflow:'hidden'은 금지 — iOS 그림자 클립 + **안드 elevation 회색(P-024 선례)** 조합 재발 위험

### DoD

- [ ] 현재 카드 내부 텍스트가 타 행들과 좌/우 기준선 일치 · 점수내역 하단 코너 라운드 닫힘 · iOS/안드 무회귀 · tsc 0 · jest · preview OTA

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-037].

## [P-036] ✅ KB-174 재수정 — 음식탭 오프라인 캐시 잔존 + 푸터 스피너 궤도 버그 (Q-08 반려) — `dee7567` (preview OTA)

Q-08 실기(예진 7/21, 스크린샷 4장): ① 연결됐다가 기내모드 시 음식탭이 J4로 안 바뀌고 **캐시 목록+무헤더** 상태(홈·프로필은 정상 J4) ② 그 상태에서 스크롤하면 푸터 로딩 스피너가 제자리 회전이 아니라 **화면을 궤도로 돌아다님**.

**원인 (커맨드 센터 실조사)**:
- ①: P-027의 전체화면 에러가 `ListEmptyComponent` 경유 — **캐시가 있으면 리스트가 비어있지 않아 에러 화면 미표시**, 헤더만 숨는 어중간 상태. (Q-08 1번이 통과했던 건 재시작 직후=캐시 없음이었기 때문)
- ②: `Spinner.tsx`의 회전 `Animated.View`가 **너비 미지정** → FlatList 푸터 직속에서 **전폭 stretch**(화면폭×16 띠) → 띠를 360° 회전하니 좌측의 16px 아이콘이 띠 중심 축으로 공전. 좁은 컨테이너 사용처에선 무증상이던 잠재 버그.

### 할 일

1. `food.tsx`: 오프라인/에러 시 **캐시 유무와 무관하게 전체화면 QueryErrorBlock**(홈과 동일) — ListEmptyComponent 방식 대신 `isError`면 리스트 자체를 J4로 대체(조건부 렌더). 리스트 미렌더로 onEndReached·푸터 스피너 경로도 원천 차단. 온라인 복귀 Retry 정상
2. `Spinner.tsx`: 회전 래퍼에 `width: size, height: size`(필요시 alignSelf:'center') 고정 — 전폭 stretch 원천 봉쇄. 사용처 전수(다른 화면 무회귀) 확인
3. 테스트: ①캐시 존재+isError → J4 렌더·리스트 미렌더 잠금 ②Spinner 래퍼 크기 고정 잠금
4. JS-only → preview OTA (프로덕션 묶음 후보)

### DoD

- [ ] 기내모드(캐시 있음)에서 음식탭 = 전체화면 J4(홈과 동일) · 스피너 제자리 회전(전 사용처) · tsc 0 · jest · OTA

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-036].

## [P-035] ✅ KB-207 재수정 — 기피 재료 칩 애니메이션 제거 (Q-11 피드백) — `c0c485f` (preview OTA)

Q-11 실기(예진 7/21): 밀림(P-026)은 해결 확인 ✅ / **P-032 ①의 칩 팝인/아웃(ZoomIn.springify/ZoomOut)이 이 화면에선 오히려 정신없음 — "아예 빼달라"**. 기피 재료는 다다다 연속 선택하는 화면이라 매 칩 바운스가 소음이 됨(절제 원칙의 정확한 사례).

### 할 일

1. `IngredientFilter` 요약 칩의 entering/exiting 애니메이션 **완전 제거**(즉시 나타남/사라짐) — 온보딩·프로필 공용 컴포넌트라 동시 반영
2. P-026 고정높이 36·placeholder는 무변 (그건 통과 확인됨)
3. 다른 화면의 P-032 효과(북마크·스테퍼·세그먼트 등)는 건드리지 말 것 — 이 화면 칩만
4. tsc 0 · jest(칩 고정높이 기존 잠금 유지) · preview OTA

### DoD

- [ ] 기피 재료 칩 추가/제거 시 애니메이션 없음(즉시) · 밀림 여전히 0 · 타 화면 효과 무변 · OTA

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-035].

## [P-034] ✅ KB-203 재수정 — 계정 연동 아이콘을 공식 로고로 (Q-16 반려) — `46a14d7` (preview OTA)

Q-16 실기(예진): 프로필>수정 연동 행의 애플·구글 아이콘이 **둘 다 이상함** — P-029가 쓴 `icons.tsx`의 `IconApple`/`IconGoogleG`는 공식 로고가 아니라 손그림 스트로크 근사치. **공식 구글 4색 로고는 이미 레포에 있음** — `SocialAuthButtons.tsx` 내부 로컬 함수 `GoogleG`(L28, #4285F4/#34A853/#FBBC05/#EA4335) — export가 안 돼 있어서 P-029가 못 찾은 것.

### 할 일

1. **구글**: `SocialAuthButtons.tsx`의 `GoogleG`를 export(또는 icons로 이동)해 `edit.tsx` 연동 행에서 사용 — 로그인 버튼과 완전 동일한 공식 4색 마크
2. **애플**: 공식 **필드(채움형) 애플 마크** SVG Path를 icons에 추가(단색 `C.ink` 채움 — 스트로크 아님, 애플 브랜드 가이드의 모노크롬 로고 형태). 로그인 화면은 OS 네이티브 버튼이라 무변 — 이 마크는 연동 행 전용
3. 기존 조악한 `IconApple`/`IconGoogleG`(스트로크판)는 다른 사용처 없으면 제거, 있으면 유지하되 연동 행만 교체
4. 미지원/누락 provider 중립 폴백은 무변

### DoD

- [ ] 연동 행 아이콘 = 공식 구글 4색 + 공식 애플 필드 마크 · 로그인 화면 무변 · tsc 0 · jest · preview OTA(프로덕션 묶음 후보)

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-034].

## [P-033] ✅ KB-205 소형 정정 — 주문카드 종교·식이 문단 제거 (평면 81종 정본) — `e24c664` (다음 OTA 편승)

P-030의 ③문단(종교·식이 한 줄 "무슬림 식단입니다")은 **낡은 전제였음** — 회피 모델은 **평면 81종 재료로 통일**(religion:/diet: 코드 폐기, 7월 초 확정·7/21 예진 재확인). 커맨드 센터 기획 드리프트로 들어간 것. 해당 코드는 와이어에 영원히 안 오므로 죽은 코드.

### 할 일

1. `orderCard.ts`의 lifestyle(③) 문단 조립 로직 + 관련 테스트(lifestyle 2케이스) **제거**. ②(기피 재료 고지)는 무변 — 종교·식이는 해당 재료가 기피 목록에 포함되는 것으로 이미 커버됨
2. tsc 0 · jest (테스트 수 감소 OK)
3. JS-only — 다음 OTA에 편승(단독 발행 불요)

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-033].

## [P-030] ✅ KB-205 음식 상세 주문+고지 원카드 — "이 메뉴 주문하기" — `0d7aa99` (preview OTA)

기획 확정(7/21 디스커버리, **정본: `specs/001-personalized-menu-mvp/order-card-brief.md`** — 착수 전 필독). 안전 음식 상세 최하단 CTA → 사장님용 큰 한국어 카드(주문+알레르기 고지). BE 호출 0.

### 할 일

1. **진입 CTA** (`src/app/food/[id]/index.tsx` 최하단): "이 메뉴 주문하기" 버튼 — **overallRiskStatus ∈ {safe, caution}일 때만** (danger 미노출, unable=기존 owner 카드 자리). caution은 기존 주의 문구 유지 위에
2. **카드 화면** — 기존 `src/app/food/[id]/owner.tsx`(사장님께 확인)와 **같은 패밀리/패턴**으로 형제 라우트(예: `order.tsx`):
   - place=**한국어 고정** 본문: ① `{koreanName} {n}개 주세요.` (수량 스테퍼 1~5, 기본 1) ② 프로필 기피 **전체**: `저는 {재료 ko 나열}을(를) 못 먹어요. 들어가면 알려주세요.` (81종 코드→ko 라벨 — UI 언어 무관 ko 강제는 `i18n.getFixedT('ko')` 류로) ③ 종교·식이 코드 보유 시 한 줄(`무슬림 식단입니다` 등)
   - 기피 0개 → ②③ 생략(순수 주문 카드) / 재료 다수 → 주요 노출+"외 n개"
   - 하단 작게(reader 언어): 사용자용 요약+닫기. 조사(을/를) 받침 분기 유틸
3. i18n ×10(버튼·요약) · SVG 아이콘 · 이모지 금지
4. 테스트: 노출 조건(safe/caution만)·기피 0개 생략·을/를 분기·ko 고정 잠금

### DoD

- [ ] 기획안 스펙 충족(safe/caution CTA·카드 3문단·0개 생략) · owner.tsx와 시각 일관 · tsc 0 · jest · JS-only OTA 가능 확인

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-030].

## [P-031] ✅ KB-206 UI 폴리시 1 — 대비·글자크기·press 피드백·시트 스프링·reduced-motion — `dc7ddd7` (preview OTA)

**정본: `specs/001-personalized-menu-mvp/ui-polish-plan.md` A·B 섹션 + kbap-fe `.claude/skills/apple-design/`(SKILL.md·RN-MAPPING.md) — 착수 전 필독.** 전부 JS-only(신규 네이티브 금지 — reanimated 4.3 기설치로 충분).

### 할 일 (플랜 A1~3 + B4~6)

1. **대비**: theme `ink3`(#B0A395, 실측 2.28:1) → 4.5:1 목표로 어둡게 or 소형 텍스트 사용처를 ink2로 승격(사용처 전수 후 판단·보고에 방식 명시). 소형 주황(primary on white 3.73:1) 텍스트 점검. **위험도 4색은 불변(헌법 III — 건드리지 말 것)**
2. **maxFontSizeMultiplier ×1.3**: Txt 컴포넌트에 기본 적용(전 화면 관통) — 시스템 큰글씨에서 고정높이 잘림 방지. 정식 Dynamic Type 대응은 범위 외
3. **스캔 unmatched 안내 numberOfLines 2** — 안전 행동 지시 말줄임 제거
4. **press 즉시 피드백**: 공용 Btn/Pressable 계열 onPressIn scale 0.97(withSpring damped) — 릴리스 대기 금지. 개별 화면 산발 구현 말고 **공용 컴포넌트에서**
5. **시트/모달 등장 스프링**: timing → withSpring(damped, 바운스 0) 통일
6. **reduced-motion**: reanimated ReducedMotionConfig + 스태거/슬라이드는 크로스페이드 폴백

### DoD

- [ ] A·B 전 항목 · 위험도 4색 불변 · iOS/안드 무회귀 · tsc 0 · jest · preview OTA (프로덕션은 검토 후 묶음)

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-031].

## [P-032] ✅ KB-207 UI 폴리시 2 — kinetics 마이크로 인터랙션 9종 — `d692bcb` (preview OTA)

**정본: `ui-polish-plan.md` C 섹션(출시 전 표)** — 수량 스테퍼는 P-030에 흡수, **나머지 8종**. kinetics 각 효과의 damping/stiffness 수치를 reanimated withSpring으로 직역(RN-MAPPING.md 치환표). **P-031 후 착수**(공용 press 피드백·reduced-motion 위에 쌓기).

### 할 일 (9종 — 전부 절제 톤: 바운스는 탭/모멘텀 있는 곳만 소량)

1. **칩 팝인/아웃** — 기피 재료 선택·요약 칩 추가/제거 시 scale 팝 (P-026 고정높이 불변 유지)
2. **Bookmark Toggle** — 북마크 아이콘 채움+드롭인 (상세·목록 공용)
3. **Tab Pill Glide** — 스캔 결과 리스트/리스크/원본 세그먼트 인디케이터 글라이드
4. **Error Shake** — 로그인 실패·폼 검증 에러, 감쇠 진동·재트리거 가능
5. **Success Check** — 온보딩 제출 완료 SVG 스트로크 드로잉
6. **Stagger Entrance** — 스캔 결과 리스트 순차 등장(~60ms 스태거) · **reduced-motion 시 생략**
7. **Step Progress** — 온보딩 스텝 인디케이터 노드 팝
8. **Shimmer** — 기존 스켈레톤(KB-174)에 스윕 하이라이트
9. **Quantity Stepper 값 팝** — P-030 주문카드 수량 변경 시 scale 펄스(P-030이 먼저 완료돼 이쪽으로 이관 — 탭 모멘텀 있어 바운스 소량 허용)

### DoD

- [ ] 9종 적용 · 절제(장식 금지·상시 루프 금지) · reduced-motion 폴백 · 무회귀 · tsc 0 · jest · preview OTA

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-032].

## [P-023] ✅ 🔴긴급 KB-199 홈 조회 lang 파라미터 — /home?lang= (없으면 홈 400) — `005ed31` (preview+production OTA)

🔴 **BE 배포 완료(7/20 밤) — lang이 모든 대상 API에서 *필수*로 승격**(종한: 메뉴리스트·검색·북마크·상세·홈 모두 lang 필수, 미지원 언어=영어). **FE는 /home만 lang 미전송(useHome.ts:32 `api.get('/home')`) → 지금 홈 조회가 400날 상태.** 최우선.

전수 확인: /foods·/foods/search·/foods/{id}·/bookmarks(GET)는 이미 lang 전송 ✅ / **/home만 누락 ❌**.

### 할 일

1. `src/lib/data/useHome.ts` `fetchHome()`: `/home` → `/home?lang=${apiLang()}` (useFoods L77 동일 패턴, 템플릿 리터럴). bootGate 프리페치 키는 이미 ['home', lang]이라 정합
2. appLanguage PATCH 유지(선호 저장) — 표시는 파라미터가 담당 → Q-13 언어 잔상·미설정 영어도 동시 해결
3. 테스트: /home 요청 URL에 lang 포함 잠금
4. JS-only → preview/production OTA. **최우선 발행**

### DoD

- [ ] /home에 lang 전송(400 해소) · 언어 전환 즉시 홈 새 언어(Q-13) · 게스트/미설정 UI 언어 · tsc 0 · jest · OTA

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-023].

## [P-029] ✅ KB-203 프로필 계정 연동 정보 — provider(Apple/Google) 표시 — `2633291` (preview OTA)

프로필 탭>수정 최하단 계정 연동 행이 현재 email 라벨+값(`edit.tsx:185-186` `editProfile.email` / `me?.email`)인데, **email은 프로필 계약에 없어 사실상 undefined**(memberAdapter 주석 L12 "email ← 계약에 없음, 소셜 전용"). BE 프로필 응답에 **`provider`(APPLE/GOOGLE) 추가됨** → 연동 수단으로 교체.

### 할 일

1. **provider 매핑 추가**: MyProfileWire에 `provider: string` → `adaptProfile`에서 User로 태우기(현재 nickname·appLanguage만 매핑, provider 누락). User 타입에 `provider?: 'APPLE'|'GOOGLE'|string` 추가
2. `edit.tsx` 계정 연동 행: **provider에 따라 "Apple로 로그인" / "Google로 로그인"** 표시(+아이콘 — 애플/구글). 아이콘은 SVG(헌법: 기본 이모지 금지). 이메일 행은 값이 없으니 대체(애플 privaterelay는 무의미)
3. i18n ×10: `editProfile.linkedVia` + `{apple|google}` 라벨(또는 "{provider}로 로그인" 보간). 미지원/누락 provider면 안전한 폴백(빈 값 금지)
4. 테스트: provider=APPLE→애플 라벨 / GOOGLE→구글 라벨 잠금

### DoD

- [ ] 프로필 수정 최하단이 provider대로(Apple/Google) 표시 · SVG 아이콘 · i18n ×10 · tsc 0 · jest · OTA

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-029].

## [P-020] ✅ KB-196 Android 구글 로그인 실패 — accessToken 누락 (getTokens 병행) — `dc984a3` (preview OTA 발행)

안드 공기계 QA(adb logcat 실측): 구글 로그인이 `Exception in HostFunction: accessToken cannot be empty`로 실패. (콘솔 측 SHA-1·Android OAuth 클라이언트는 예진이 해결 — code 10 소멸 확인됨. 이건 순수 FE 버그.)

**원인**: `src/lib/auth/useSocialAuth.ts` signInWithGoogle이 `GoogleAuthProvider.credential(idToken)`으로 **idToken만** 전달. @react-native-firebase **Android 네이티브는 google 자격증명에 accessToken도 요구**(iOS 네이티브는 idToken만으로 통과 → iOS만 동작해왔음). `GoogleSignin.signIn()` 반환엔 accessToken 없음 → `getTokens()`로 받아야 함.

### 할 일

1. signInWithGoogle에서 idToken 확보(`res.data?.idToken`, 기존 가드 유지) 직후:
   `const { accessToken } = await GoogleSignin.getTokens();`
   → `await signInWithCredential(getAuth(), GoogleAuthProvider.credential(idToken, accessToken));`
2. 취소(`type==='cancelled'`)·에러 분기 무변. iOS는 accessToken 추가돼도 무해(회귀 없음)
3. 테스트: 자격증명 생성에 accessToken 포함되는지 잠금(모킹 수준) — 무리면 생략 가능, tsc/jest 통과가 우선

### 발행 (중요)

JS-only라 **preview 채널 OTA로 공기계 반영 가능**(재빌드 불요). 완료 보고에 `eas update --channel preview` 발행 여부/ID 기재. 예진: 공기계 앱 강제중지→2회 실행 후 재로그인.

### DoD

- [ ] 안드 실기기 구글 로그인 성공(계정 선택→온보딩/홈) · iOS 무회귀 · tsc 0 · jest 통과 · preview OTA 발행

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-020].

## [P-021] ✅ KB-197 Android UI 정리 — 온보딩 제출 버튼 짤림 + 언어 선택 리플 — `81deecf` (preview OTA)

안드 첫 스모크(Q-14) 발견. iOS 정상, 안드만 렌더 이슈 2건. JS-only(스타일) → preview 채널 OTA 가능.

### 할 일

1. **온보딩 제출 버튼 하단 짤림** — `src/app/onboarding/index.tsx` stickyFoot(L273 `paddingBottom: insets.bottom + 14`)에도 안드 하단 내비바에 버튼이 가림. 안드 edge-to-edge/safe-area 하단 인셋 보강(실기기로 3버튼·제스처 내비 둘 다 확인). 필요 시 app.json androidNavigationBar/edge-to-edge 설정 점검
2. **언어 선택 항목 배경 리플** — `src/components/LanguagePicker.tsx` `rowOn`(L82 `backgroundColor:'rgba(226,88,12,0.06)'`)가 안드 Pressable 눌림 하이라이트/리플과 겹쳐 padding까지 회색 번짐. `android_ripple` 명시 or 눌림 스타일 정리 → 선택 상태는 테두리+옅은 배경만 깔끔히. 프로필 수정 화면 동일 컴포넌트도 확인
3. iOS 무회귀 확인(스타일 변경이 iOS 렌더 안 바꾸는지)

### 발행

JS-only → **preview 채널 OTA**로 공기계 반영. 완료 보고에 OTA ID. 예진: 강제중지→2회 실행 후 온보딩·언어 화면 재확인.

### DoD

- [ ] 안드 온보딩 제출 버튼 내비바 안 가림 · 언어 선택 배경 깔끔 · iOS 무회귀 · tsc 0 · jest 통과 · preview OTA

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-021].

## [P-022] ✅ KB-198 Android 스캔 가로 감지 — expo-sensors 세로 유도 오버레이 포팅 — `eecbe52` (재빌드 필요, OTA 아님)

안드 스모크(Q-14): 스캔의 "가로로 돌리면 세로 유도 힌트"가 iOS 전용(expo-camera `responsiveOrientationWhenOrientationLocked`·`onResponsiveOrientationChanged` = @platform ios). 안드는 콜백 미발생 → 미동작. **안드 동시 출시라 대응.**

### 방식 (비교 확정 — expo-sensors)

DeviceMotion으로 **기기 물리 방향 직접 감지** → 앱은 세로 고정 유지, 힌트만 반응(iOS와 동일). expo-screen-orientation은 **잠금 상태에서 회전 이벤트가 안 와** 부적합(감지하려면 잠금 해제=가로 스캔 허용, 목표 반대). → 세로 고정+감지가 동시에 되는 건 센서뿐.

### 할 일

1. **`expo-sensors` 추가** — 네이티브 모듈이라 **재빌드 필요(OTA 불가)**. app.json plugins 반영. ⚠️ 이 건은 다음 빌드에 실림 — preview OTA로 못 감(P-021과 발행 경로 다름)
2. `src/app/scan.tsx`: 안드에서 `DeviceMotion.addListener`로 중력 벡터 → 방향 판정(**임계각+디바운스**로 45° 근처 떨림 방지) → 기존 `camOrientation` state에 공급. 마운트 구독/언마운트 해제. iOS는 현행 expo-camera 콜백 유지 — **안드만 센서 추가**(회귀 최소화, 두 소스가 같은 state 먹임)
3. 기존 오버레이(`isLandscape`→rotateOverlay, transform rotate) **무변 재사용**
4. 테스트: 중력 벡터→orientation 판정 순수 함수 단위 테스트(portrait/landscapeLeft/landscapeRight 경계)

### DoD

- [ ] 안드 실기기: 스캔 중 가로로 돌리면 세로 유도 오버레이 표시(iOS와 동일) · 세로 복귀 시 사라짐 · iOS 무회귀 · tsc 0 · jest 통과
- [ ] 재빌드 산출물에 포함(안드 빌드2 / 다음 iOS 빌드) — OTA 아님을 보고에 명시

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-022].

## [P-024] ✅ KB-197 재수정 — 언어 선택 회색은 Android elevation(리플 아님) — `567d461` (preview OTA)

P-021 실기 확인(예진): 온보딩 버튼 짤림 ✅ 해결 / **언어 선택 회색 padding 그대로** ❌. P-021의 android_ripple 수정이 **원인을 오진**함.

**진짜 원인**: `LanguagePicker.tsx` `row` 스타일에 **`overflow:'hidden'`(P-021 추가) + `...shadow.sh1`(elevation:1) 공존**. 안드에서 이 조합은 elevation 그림자가 클립되며 **padding이 회색으로 채워지는** 알려진 버그. 리플이 아니라 elevation이 회색의 정체. (증거: 같은 파일 `closeBtn`도 sh1 쓰지만 overflow 없어 멀쩡 → **row만 둘 다 가져 row만 회색**.)

### 할 일

1. `src/components/LanguagePicker.tsx` `row` 스타일에서 **`...shadow.sh1` 제거**. row는 이미 `borderWidth:1 + borderColor:C.hair`로 구분되고 sh1은 opacity 0.04라 시각 손실 거의 없음 → 회색 소멸
2. `overflow:'hidden'`은 **유지 가능**(elevation 사라지면 무해, 리플 라운드 클립 역할). 단 이제 안 겹치니 필요 판단은 FE 재량
3. android_ripple(P-021)은 그대로 두어도 됨(정상 동작). rowOn 틴트/테두리 무변
4. **회귀 주의**: 혹시 회색이 남으면 차선 용의자는 rowOn `rgba(226,88,12,0.06)` 틴트지만 그건 주황이라 회색 아님 — elevation이 1순위

### DoD

- [ ] 안드 실기: 언어 선택 항목 padding 회색 없음(선택 행은 주황 테두리+옅은 틴트만) · iOS 무회귀 · tsc 0 · jest
- [ ] preview OTA 발행(JS-only)

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-024].

## [P-027] ✅ KB-174 후속 — 음식 탭 오프라인 = 헤더 숨기고 전체화면 에러 — `a762280` (preview OTA)

Q-08 실기(예진): 핵심 3개 통과 / 음식 탭 오프라인 시 **헤더(제목 "음식 둘러보기"+부제+검색바)가 그대로 뜨고 그 아래 오프라인 에러**가 겹쳐 보임. 홈탭처럼 오프라인=전체화면 에러로 통일 요청.

**원인**: `src/app/(tabs)/food.tsx` FlatList가 `ListHeaderComponent={Header}`를 항상 렌더 + `ListEmptyComponent`로 QueryErrorBlock을 아래에 깖 → 헤더+에러 공존.

### 할 일

1. `isError`(오프라인/에러)일 때 **Header 미렌더** → QueryErrorBlock이 화면 전체(J4). 예: `ListHeaderComponent={isError ? null : Header}` + Empty가 전체높이 차지하도록(홈탭 J4 렌더와 톤 통일). 로딩(isLoading)은 기존 스켈레톤 유지
2. 정상/빈 응답 상태는 무변(헤더+그리드·빈 상태)
3. iOS·안드 공통

### DoD

- [ ] 오프라인 시 음식 탭 = 전체화면 오프라인(헤더·검색바 미노출, 홈탭과 동일 톤) · 온라인 복귀 시 정상 · tsc 0 · jest
- [ ] preview/production OTA (JS-only)

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-027].

## [P-026] ✅ KB-178 재수정 — 기피 재료 요약 줄 0→1 밀림 (고정 높이) — `0ea62e6` (preview OTA)

Q-11 실기 확인(예진): 3·4·5 통과 / **2번 첫 재료 선택 시 목록 밀림 + 6번 "완전 거슬려"** = 같은 원인. 칩 요약 줄이 0개 땐 높이 0 → 첫 칩에서 높이가 생기며 아래 목록을 1회 밀어냄.

### 할 일

1. 기피 재료 요약(칩) 줄에 **고정 최소 높이 예약** — 빈 상태에도 칩 한 줄 높이 유지. 빈 상태엔 placeholder 텍스트("선택한 재료가 여기 표시돼요" 류, i18n ×10) 또는 동일 높이 빈 컨테이너 → 0→1 전환에 레이아웃 높이 불변
2. 온보딩 스텝 + 프로필 기피 재료 수정 **공용 컴포넌트 동시 반영**
3. 스크롤 위치 점프 0 확인(2번), 첫 선택 체감 매끄러움(6번)
4. JS-only → preview/production OTA

### DoD

- [ ] 첫 재료 선택 시 목록 밀림·점프 0 · 빈↔선택 높이 불변 · 온보딩·프로필 동일 · iOS/안드 무회귀 · tsc 0 · jest

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-026].

## [P-028] ✅ KB-174 후속 — 검색 화면 오프라인 = 전체화면 J4 (f8ea807)

Q-08 이슈1 결정(예진 7/20): **평소(온라인)엔 지금인기 6개 그대로 유지**, 단 **오프라인이면 검색 화면도 다른 탭과 동일한 전체화면 오프라인(J4)**. (P-027이 음식탭은 처리 완료 — 이건 검색 화면 몫.)

**현재**: `src/app/search.tsx` empty 상태(미제출)가 오프라인에도 최근 검색+지금인기를 렌더 → 셋 다 오프라인엔 무의미(지금인기=네트워크 stale, 최근 탭=실검색 실패).

### 할 일

1. `search.tsx`: **오프라인이면 empty 상태(최근+지금인기) 대신 전체화면 J4**(홈·음식탭이 쓰는 QueryErrorBlock/offline 렌더 재사용 — 톤 통일). 오프라인 판정은 **음식탭/홈이 쓰는 그 메커니즘 그대로**(`useFoods().isError` 또는 기존 netinfo 훅 — P-027에서 쓴 것과 동일)
2. **온라인 평소엔 무변** — 지금인기 6개(popularityRank 상위)·최근 검색 그대로. **변경 금지(예진 명시)**
3. 제출 검색 중 오프라인(기존 `search.isError` L142)은 현행 유지 or 같은 J4로 톤 맞춤(FE 판단)
4. iOS·안드 공통

### DoD

- [ ] 오프라인 시 검색 화면 = 전체화면 오프라인(최근·지금인기 미노출, 음식탭과 동일 톤) · 온라인 평소 지금인기 6개 그대로 · tsc 0 · jest
- [ ] preview/production OTA (JS-only)

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-028].

## [P-025] ✅ (재빌드) KB-202 스캔 캡처 WYSIWYG — 과다 캡처 크롭 — `4bf857a` (OTA 불가·빌드2 배치)

Q-12 규명: 미리보기 밖인데 캡처본은 더 넓게(가격 줄까지) 찍힘 → BE가 그 가격을 정상 판독(BE 무혐의). 원인: `scan.tsx` `CameraView style=absoluteFill`(ratio 미지정) 미리보기는 센서 cover-크롭(중앙만), `takePictureAsync`는 센서 전체 → 미리보기 밖이 캡처·업로드에 포함. 과다 캡처가 브라우저 탭·상호명 혼입 → 제외 오판 배경.

### 할 일

1. **캡처본을 미리보기 뷰포트로 크롭** 후 업로드/표시(WYSIWYG). `expo-image-manipulator` 추가(**네이티브 → 재빌드**), cover-크롭 역산으로 crop rect 산출(preview WxH ↔ pic WxH). 오버레이 마커는 이미 preview 정규화 공간이라 크롭 후 정합 유지
2. iOS·안드 공통. 결과 "원본 사진"도 크롭본
3. 테스트: crop rect 계산 순수 함수 단위(센서>화면 세로/가로 케이스)

### 발행 (중요)

⚠️ **재빌드 필요 — OTA 불가.** `expo-sensors`(P-022)와 **같은 빌드(안드 빌드2/다음 iOS 빌드)에 배치**. 완료 보고에 OTA 아님·빌드 포함 명시

### DoD

- [ ] 미리보기 프레임 = 업로드/표시 이미지(밖 영역 미포함) · 마커 정합 · 재빌드 산출물 포함 · tsc 0 · jest

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-025].


## [P-019] ✅ KB-195 온보딩 맵기 스킵 = -1 명시 전송 (스웨거 required 승격) — `8135d3e`

스웨거 재배포(7/20) 대조 결과: `OnboardingRequest.spicinessPreference`가 **required로 승격**. 현행 스킵=필드 생략(P-003)이 서버 검증에 걸리면 온보딩 가입이 400으로 깨진다 — submit.ts의 전환 예약 주석("다르게 저장되는 게 실측되면 -1 명시 전송으로 전환")이 바로 이 시점.

### 할 일

1. `src/lib/onboarding/submit.ts`: 조건부 스프레드 제거 → `spicinessPreference: spiceTolerance !== UNSET ? spiceTolerance : -1` 명시 전송. 예약 주석을 "required 승격으로 전환 완료(7/20)"로 갱신
2. 로컬 보관(SPICE_KEY)·맵기 UNSET 처리 등 나머지는 무변 — KB-150 -1 센티널 정책 그대로
3. 테스트: 스킵 payload에 `spicinessPreference: -1` 포함 잠금 (기존 생략 검증 테스트 있으면 교체)

### DoD

- [ ] 스킵 온보딩 body에 -1 포함 · 설정 시 실값 · tsc 0 · jest 통과 (JS-only — OTA 가능 확인)

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-019].

## [P-018] ✅ KB-194 스플래시 개선 — 태그라인 제거 + 최소 노출/프리페치 게이팅 — `096a954`

실기기 QA Q-07(7/20, 빌드 6): ① 하단 "SCAN · KNOW · EAT" 태그라인이 너무 작아 가독 불가 → **제거 확정** ② 스플래시가 ~0.1초 반짝 후 진입해 렉처럼 보임 → 부트 게이팅 도입.

### 할 일

1. **태그라인 제거 (네이티브 — 빌드 7 반영)**: `assets/images/splash-icon.png` 하단 문구 스트립 제거 — 원본 소스 없으면 png 직접 크롭(투명 여백 정리 포함). app.json `expo-splash-screen` 플러그인 `imageWidth` 280→230 **원복** (그 자리 `_comment`가 이 원복을 예약해둔 것 — 주석도 정리). 결과물 미리보기 png를 보고에 첨부 경로로 남길 것
2. **부트 게이팅 (JS — OTA 가능)**: `src/app/_layout.tsx`의 `SplashScreen.hideAsync()` 게이트 확장:
   - 기존 준비 조건 + **핵심 데이터 프리페치**: queryClient.prefetchQuery로 홈·음식 목록, BE 세션 있으면 me — **쿼리 키는 lang 종속 실키와 일치**시킬 것 (LocaleProvider의 LANG_DEPENDENT_KEYS 참조 — 키 불일치 시 프리페치가 무의미해짐)
   - **최소 노출 1200ms**: 프리페치가 일찍 끝나도 min 채운 뒤 hide
   - **상한 4000ms**: 초과 시 프리페치 대기 중단하고 진입 — KB-174 스켈레톤/J4가 이어받음. **프리페치 실패·오프라인은 스플래시를 붙잡지 않는다** (reject여도 cap 전 진입)
3. 게이트 로직은 순수 함수로 분리해 유닛 테스트: 조기 완료→min까지 대기 / min~cap 사이 완료→즉시 hide / cap 초과→강제 hide / 프리페치 reject→hide 지연 없음

### DoD

- [ ] 빠른 네트워크: 스플래시 약 1.2초 균일 노출 후 진입 (반짝임 소멸) — OTA로 즉시 검증 가능
- [ ] 느린 네트워크: 최대 4초 후 스켈레톤 진입 (무한 스플래시 없음)
- [ ] 에셋: 태그라인 없는 splash-icon.png + imageWidth 230 (빌드 7 확인 항목임을 보고에 명시)
- [ ] tsc 0 · jest 통과 (게이트 유닛 테스트 포함)

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-018].

## [P-015] ✅ KB-187 언어 전환 즉시 반영 + 맵기 라벨 i18n (QA Q-04 피드백) — `60c043f`

실기기 QA(7/20): ① 맵기 설명이 전 언어에서 영어 고정 ② 언어 변경 후 음식 탭·홈 인기 요리가 이전 언어 음식명으로 표시되다가 한참 뒤에야 교체 ("다른 행동 후 홈 복귀하니 정상" — 재조회 완료 후 교체된 정황).

### 할 일

1. **SPICE_SCALE 라벨 i18n 이전**: onboarding/data.ts의 영어 하드코딩 배열 → i18n 키 ×10 로케일. 사용처 전부(온보딩 spice 스텝·프로필 조회/수정 칩·음식 상세 detail.spice) 언어 추종 확인
2. **언어 전환 잔상 원인 실조사 후 최소 수술**: 쿼리 키에 lang이 이미 있으니 재조회 자체는 됨 — 의심 지점은 이전 데이터가 로딩 중 유지되는 동작(placeholderData/keepPreviousData류)이 **언어가 바뀐 새 키에도 이전 키 데이터를 보여주는 것**. 조사 결과를 보고에 명시하고:
   - 언어 전환 순간 서버 데이터 쿼리 무효화(선택적 — foods/home/search 등 lang 종속만)
   - 새 언어 로딩 중엔 **이전 언어 데이터 대신 스켈레톤**(P-009 기구현 재사용)
   - 매 탭 진입마다 무조건 재호출하는 방식은 금지 (예진 제안의 의도는 "즉시 갱신" — 전환 시점 1회 처리로 달성)
3. 스캔 리스트 name 한국어 고정은 **BE 계약 문제로 범위 제외** (질의 진행 중 — 건드리지 말 것)
4. 테스트: 언어 전환 → lang 종속 쿼리 무효화 호출 잠금 / SPICE 라벨 키 존재 ×10 패리티

### DoD

- [ ] 언어 전환 직후 홈·음식 탭이 새 언어로만 (이전 언어 잔상 0) — 로딩은 스켈레톤
- [ ] 맵기 라벨 전 사용처 언어 추종 · i18n 패리티 · tsc 0 · jest 통과

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-015].

## [P-017] ✅ KB-176 Android 첫 빌드 — 공기계 스모크용 apk — 코드 무변경 (기록 `0e9f884`)

예진이 안드 공기계 확보 — 설치용 apk가 필요하다. EAS 안드 쿼터 15회 전량 미사용.

### 할 일

1. **eas.json 확인**: 공기계 직설치용 apk 프로파일 — `preview`에 `"android": {"buildType": "apk"}` 없으면 추가 (⚠️ **preview 프로파일 사용** — dev client는 Metro 연결이 필요해서 단독 스모크에 부적합. preview는 임베디드 JS가 빌드 시점 main 최신이라 standalone으로 돎)
2. **Firebase Android 사전 점검**: `google-services.json` 존재 여부 확인 — 없으면 빌드 실패/로그인 불가이니 **즉시 보고**(예진이 Firebase 콘솔에서 Android 앱 등록 필요). 있으면 진행
3. `eas build --platform android --profile preview` — 첫 빌드라 **keystore 생성 질문에 EAS 관리(자동 생성)로 답**
4. 빌드 완료 후 보고: **설치 링크/QR(expo.dev 빌드 페이지 URL)** + apk 다운로드 URL — 예진이 공기계에서 바로 설치
5. ⚠️ 예상 이슈 미리 기록: 구글 로그인은 EAS keystore의 **SHA-1을 Firebase 콘솔에 등록해야 동작** — 빌드 후 `eas credentials`로 SHA-1 확인해 보고에 포함 (등록은 예진 콘솔 작업). 애플 로그인 버튼은 Android에서 미노출/무동작이 정상인지도 확인 포인트로 명시

### DoD

- [ ] apk 빌드 성공 + 설치 URL·SHA-1 보고
- [ ] 코드 변경이 있었다면(eas.json 등) 커밋 해시 병기

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-017]. (공기계 스모크 자체는 예진 — QA 항목으로 별도 추가 예정)

## [P-016] ✅ KB-149 후속 — 프로필 사진 최종 컨벤션: 삭제·미설정 = 기본 path 전송 (null 폐기) — `847f3d3`

종한 최종 확정(7/20, Q-02 답변) — **P-014의 null 방식 대체**:
- 삭제하거나 온보딩에서 사진 미설정 시 **`images/default/profile/profile-default-512.png`를 값으로 전송** (null 금지)
- 응답도 null이 내려가지 않도록 서버 처리 중 — 프로필 사진 필드는 **항상 값이 있는** 설계로 확정 ("미설정" 상태 소멸, 기본 path가 그 역할)

### 할 일

1. `PROFILE_IMAGE_CLEAR`: null → **`'images/default/profile/profile-default-512.png'`** (상수 한 곳 — 세 번째 교체이니 상수명 유지, 값·주석에 확정 경위 명시)
2. **온보딩 미선택 시에도 이 path를 명시 전송** (기존 필드 생략 → 기본 path 전송으로 변경 — submit.ts)
3. **삭제 액션 노출 판별 변경**: 지금은 "사진 설정 시에만 노출"인데, 앞으로 서버가 항상 URL(기본 포함)을 주므로 — `isDefaultProfileImage(url)` (URL에 `images/default/profile/` 포함 여부) 판별 추가: 기본 사진이면 삭제 액션 미노출·플레이스홀더 취급, 커스텀 사진이면 노출
4. 렌더: 서버가 주는 URL(기본 포함) 그대로 렌더 — FE svg 플레이스홀더는 URL 부재·비-http 방어용으로만 유지
5. 착수 시 실측: 미설정 계정 조회 응답이 기본 절대 URL로 오는지 (서버 배포 확인 — 안 왔으면 진행로그 기록 후 코드만 완성)
6. 테스트 갱신: 삭제·온보딩 미선택 전송값 잠금(기본 path) / isDefault 판별 / 기본 사진 시 삭제 액션 미노출

### DoD

- [ ] 삭제·온보딩 미선택 → 기본 path 전송, 서버 기본 URL 렌더, 삭제 액션 노출 로직 정확
- [ ] tsc 0 · jest 통과. ※ 이 커밋부터는 서버 무관하게 OTA 가능 (기본 path는 항상 유효한 값)

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-016].

## [P-014] ✅ KB-149 후속 — 기본 프로필 사진 서버 일원화 (png 산출 + 삭제값 null 교체) — `3ebbc03`

종한×예진 확정(7/20 채팅): ① 미설정/삭제 시 서버가 **기본 프로필 사진 URL을 응답**하는 방식으로 변경 (FE svg 플레이스홀더 → 서버 제공 png) ② 삭제 전송 컨벤션은 **null** ("null로 내려간다" — P-013의 잠정 ''를 교체).

### 할 일

1. **기본 아바타 png 산출**: 현재 플레이스홀더로 쓰는 svg를 png로 export (512×512 권장, 앱 배경과 어울리는 단색/투명 판단) → `dropbox/yj/`에 파일 전달 + 종한에게 공유 (서버가 이걸 기본 URL로 서빙)
2. **삭제 전송값 교체**: `PROFILE_IMAGE_CLEAR` '' → **null** (상수 한 곳). ProfileUpdateRequest 타입이 null 허용인지 확인 — 스펙상 비-nullable이면 진행로그에 기록하고 종한 배포와 맞출 것
3. **조회 렌더 확인**: 서버가 기본 URL을 주기 시작하면 미설정도 절대 URL이 옴 — 기존 렌더 그대로 동작 확인. FE svg 플레이스홀더 분기는 **방어로 유지** (서버 배포 전·비-http 값 대비), 제거하지 말 것
4. 테스트 갱신: 삭제 전송 null 잠금 (기존 '' 테스트 교체)

### 순서 주의

png 전달(1)이 종한의 서버 작업 선행 조건. 2·4는 독립적으로 지금 가능 — 단 **서버가 null 삭제를 배포하기 전 OTA에 실리면 삭제가 무동작할 수 있으니**, 보고에 "서버 배포 확인 후 OTA 포함" 명시.

### DoD

- [ ] png 전달 완료 · 삭제 전송 null·테스트 교체 · tsc 0 · jest 통과
- [ ] (서버 배포 후) 실기기: 삭제→기본 사진 렌더→재시작 유지 (Q-02 6번)

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-014].

## [P-013] ✅ KB-149 후속 — 프로필 사진 삭제 (기본 플레이스홀더 복귀) — `9e67bf4`

QA Q-02 피드백(7/20): 사진 교체만 가능하고 **삭제가 없음**. "프로필 사진 삭제" 액션 추가 — 삭제 시 기존 기본 플레이스홀더(svg)로 복귀.

### 할 일

1. 프로필 수정 화면에 "프로필 사진 삭제" 액션 — 사진이 설정된 상태에서만 노출 (없을 땐 미노출). 위치·톤은 기존 화면 관용구 (파괴적 액션이지만 계정 삭제급은 아님 — 확인 얼럿은 과하지 않게 판단)
2. **삭제 전송 표현 확인 (착수 게이트)**: ProfileUpdateRequest.profileImageUrl에 null vs 빈 문자열 — 종한 확정 대기 중. 착수 시 확정값 반영, 미확정이면 빈 문자열 실측(서버 반응) 후 진행로그 기록. 맵기 -1 선례처럼 "생략=유지"는 보존
3. 삭제 성공 시: ['me'] invalidate → 탭·수정 화면 즉시 플레이스홀더 복귀, 앱 재시작 후에도 유지
4. 온보딩 쪽은 무변 (선택 사항이라 이미 미선택 = 없음)
5. 테스트: 삭제 전송값 잠금 / 삭제 후 렌더 플레이스홀더 / 사진 없을 때 삭제 액션 미노출

### DoD

- [ ] 삭제 → 플레이스홀더 복귀 + 재시작 후 유지 / 사진 없으면 삭제 액션 없음
- [ ] i18n ×10 · tsc 0 · jest 통과

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-013].

## [P-012] ✅ KB-179 음식 상세 가격 — 스캔 결과에서 진입 시 파라미터 전달 (방향 확정 7/20) — `d134d1f`

QA Q-01 피드백 후속. BE 확인: **detail API에선 가격 제공 불가(로직상)** — 가격은 음식이 아니라 그 메뉴판의 속성. 확정 방향: **스캔 결과 → 상세 이동 시에만 FE가 가격을 route param으로 전달해 표시.** 홈·목록·검색·북마크 진입은 가격 미표시가 정상(어느 식당 가격인지 알 수 없으므로).

### 할 일

1. 스캔 결과의 상세 이동 지점 전부(리스트 행·오버레이 마커 탭·사진 전용 항목)에서 해당 항목 price가 있으면 `/food/{id}?price=9000` 형태로 첨부 (null이면 미첨부)
2. 상세 화면: `price` 파라미터가 유효한 정수일 때만 가격 행 표시 — 스캔 결과와 동일 규칙(₩ 포맷만, 환율·추정 금지), 기존 레이아웃 안 깨지게. "스캔한 메뉴판 기준"임을 암시하는 짧은 레이블(i18n ×10)
3. 방어: 파라미터 조작 가능성 — 정수 파싱 실패·음수는 미표시 (신뢰 경계는 낮음: 표시 전용, 저장 안 함)
4. 테스트: param 유→표시 / 무·비정수→미표시 / 스캔 결과 이동 시 param 첨부 잠금

### DoD

- [ ] 스캔→상세 ₩ 표시 / 타 경로 진입 미표시 · tsc 0 · jest 통과

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-012].

## [P-011] ✅ KB-178 기피 재료 선택 UI 개선 — 요약 가로 스크롤 1줄 + CTA 하단 고정 (B안 확정) — `79ace75`

실기기 QA 피드백(7/20): ① 선택 요약 박스가 자라며 목록을 밀어냄 ② 계속 버튼이 81종 아래. 3안 비교 후 **B안 확정** (예진).

### 할 일

1. **선택 요약 → 고정 높이 1줄 가로 스크롤**: 세로 wrap 박스를 horizontal ScrollView 칩 줄로 교체 — 칩 탭=제거 유지, 새 선택 시 줄 끝에 추가되고 **자동으로 끝까지 스크롤**(방금 추가된 게 보이게). "n가지를 피하고 있어요" 카운트는 줄 위 소제목 or 칩 줄 앞 고정 요소로 축약. 0건이면 줄 자체 미표시(높이 0 아님 — 레이아웃 점프 방지 위해 placeholder 유지 여부는 실물 보고 판단·보고)
2. **CTA 하단 고정**: "계속 · n개 추가됨"을 화면 하단 스티키로 (목록 스크롤과 무관하게 항상 노출). 목록 하단 패딩으로 마지막 칩이 CTA에 가리지 않게
3. **프로필 기피 재료 수정 화면(restrictions)도 동일 적용** — 같은 컴포넌트로 공용화(복붙 금지)
4. 핵심 검증: 재료 선택/해제 시 **목록 스크롤 위치가 유지**되고 화면이 밀리지 않음
5. 기존 토큰·칩 스타일 재사용 — 새 디자인 결정 금지 (B안 패턴 선택은 이미 확정)

### DoD

- [ ] 선택/해제 시 밀림·점프 없음 (요약 줄 높이 불변)
- [ ] CTA 항상 노출·동작, 온보딩+프로필 양쪽 적용
- [ ] tsc 0 · jest 통과 (요약 줄 렌더·CTA 고정 테스트)

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-011].

## [P-010] ✅ KB-177 탈퇴 후 로컬 잔재 정리 — 게스트에게 온보딩 재개 모달 노출 버그 — `b178e1c`

실기기 발견(7/20): 탈퇴 완주(애플 revoke 포함) 후 "둘러보기"로 게스트 진입 → 홈에서 **"음식 프로필 설정을 마무리하세요" 재개 모달**이 뜸. 탈퇴한 계정의 온보딩 로컬 흔적이 남아 게스트를 온보딩 미완 회원으로 오인. KB-152(Keychain 잔존)와 같은 "탈퇴 후 로컬 잔재" 계열.

### 할 일

1. **탈퇴 성공 시 회원 로컬 상태 일괄 정리** — 온보딩 draft(중단 저장분) 포함. freshInstall.ts의 정리 로직과 접점이 있으면 공용 정리 함수로 묶을 것(중복 구현 금지). 무엇을 지웠는지 목록을 보고에 명시
2. **방어 분기**: 온보딩 재개 모달 노출 조건을 "로그인 + 온보딩 미완 회원"으로 제한 — 게스트에겐 draft가 있어도 미노출. ①이 있어도 이 방어는 별도로 (미래의 다른 잔재로부터도 보호)
3. 로그아웃(탈퇴 아님)의 draft 처리 방침은 현행 유지하되, 다른 계정 재로그인 시 이전 draft가 이어지는 문제가 보이면 판단·기록만 (범위 확장 금지)
4. 테스트: 탈퇴 후 draft 소거 / 게스트 상태 재개 모달 미노출 / 회원 중단→재개 흐름 무변

### DoD

- [ ] 탈퇴→둘러보기 진입 시 재개 모달 없음 (실기기 확인 포인트)
- [ ] 정상 회원 온보딩 재개 무변 · tsc 0 · jest 통과

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-010].

## [P-009] ✅ KB-174 후속 — 탭별 스켈레톤 (실제 레이아웃 미러링, 범용 리스트 스켈레톤 대체) — `4ba7106`

실기기 피드백(예진 7/20): 범용 SkeletonList(리스트형)가 프로필 탭에도 그대로 떠서 어색 — **스켈레톤은 그 화면의 렌더 완료 레이아웃과 같은 골격이어야** 자연스럽다. 디자인 발주 불필요 — 각 화면의 확정 레이아웃이 곧 스펙. J1의 톤(회색 블록·라운드·기존 토큰)만 재사용.

### 할 일

1. 화면별 스켈레톤 변형 — **각 탭의 실제 렌더 결과와 블록 위치·크기 일치**:
   - 홈: 인사말/히어로 카드/최근 스캔 행 등 실제 홈 구성 그대로 회색 블록화
   - 음식 탭: **2열 그리드 카드** (사진 블록 + 이름/뱃지 줄) × 6개 내외
   - 프로필: 아바타 원 + 닉네임 줄 / 랭킹 행 / 섹션 행들 — 프로필 실제 구조 미러
2. 구현 원칙: 기존 SkeletonList의 톤·애니메이션 재사용, 화면별로는 **블록 구성만** 다르게 (Skeleton 원자 + 화면별 조립 — 새 디자인 결정 금지)
3. **레이아웃 시프트 0 목표**: 스켈레톤과 실제 콘텐츠의 패딩·크기를 같게 — 로딩→렌더 전환 시 점프 없음 (이게 품질 기준)
4. 범용 SkeletonList는 마땅한 화면(검색 등 리스트형)에만 유지

### DoD

- [ ] 홈·음식·프로필 각각 자기 레이아웃 모양의 스켈레톤 (전환 시 점프 없음)
- [ ] tsc 0 · jest 통과 (기존 스켈레톤 렌더 테스트 갱신)

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-009].

## [P-008] ✅ KB-174 후속 — useFoods의 401 삼키기 제거 (음식 탭 백지 잔존 버그) — `b8b8df2`

실기기 확인(7/20): 홈·프로필은 에러 화면이 뜨는데 **음식 탭만 백지**. 원인 실측 완료 — P-007의 화면 레벨 수정은 정상인데, **데이터 레벨에서 401을 "성공+빈 목록"으로 위장**하는 레거시가 남아 있음.

### 원인 (검토 게이트 실측)

`useFoods.ts` 2곳 — `useInfiniteFoods`(:82)·검색 훅(:111)의 queryFn catch:
`if (e instanceof ApiError && e.status === 401) return { items: [], hasNext: false }`
→ BE가 foods를 인증 필수로 걸었던 시절의 게스트 정숙 임시책. 지금은:
- BE foods 3종 **인증-선택 전환 완료** (무토큰 200 실측 7/20) — 게스트는 애초에 토큰을 안 붙여 401이 없음
- 남는 401 = **죽은 토큰(로그인 유저)뿐** — 이걸 빈 목록으로 위장하면 isError가 안 서서 에러 블록(P-007)이 무력화됨. 주석의 "전환되면 자동 소생" 조건이 충족됐으니 제거 시점

### 할 일

1. 두 catch의 401 특례 제거 — 401도 다른 에러처럼 throw (에러 블록 표면화). catch 자체가 그것뿐이면 try/catch 제거
2. 회귀 테스트: 401 응답 → 음식 탭 isError → 에러 블록 렌더 (빈 목록 위장 없음) — browse·search 각 1건
3. 겸사 확인: 다른 훅에 같은 패턴 없음 (검토 게이트 grep 완료 — 이 2곳뿐. 재확인만)

### DoD

- [ ] 401 시 음식 탭·검색 에러 상태 렌더 (백지/false-empty 없음)
- [ ] 게스트(무토큰) 정상 목록 무변 · tsc 0 · jest 통과

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-008].

## [P-007] ✅ KB-174 공통 상태 UI — 에러·오프라인·스켈레톤 적용 (디자인 J 시리즈, false-empty 제거) — `9d64f43`

배경: 7/20 BE `/auth/refresh` 500 장애 때 실기기 확인 — 홈이 에러인데 "아직 스캔이 없어요"(빈 상태)로 표시(**false-empty**), 음식 탭 텅 빈 배경, 프로필 백지. 디자인의 공통 상태 원자가 미적용.

### 디자인 (Claude Design "KLens Hi-Fi Direction G" — J·Common states 시트)

- **J1 Loading**: 스켈레톤 (카드 자리 회색 블록)
- **J2 Empty**: 아이콘+제목+설명+주 CTA — 진짜 0건일 때만
- **J3 Error**: ⚠️ 아이콘 + "Something went wrong" + "We couldn't load this right now. It's on our side, not yours." + [Try again] 주버튼 + [Go back] 보조 (탭 루트에선 Go back 생략)
- **J4 Offline**: 플러그 아이콘 + "You're offline" + 설명 + [Retry] (+ 탭 맥락 보조 CTA)
- J5 Unable-to-judge: 기구현 위험도 unable 상태와 동일 개념 — 이번 범위에선 확인만
- 톤·토큰은 기존 Direction G 화면들과 동일 (신규 색·폰트 금지)

### 할 일

1. 공통 상태 컴포넌트 1개(변형 loading/error/offline/empty)로 J1·J3·J4 구현 — 화면별 복붙 금지
2. 적용: 홈·음식 탭·프로필 탭(+음식 상세는 기존 에러 처리 있으면 톤만 통일). **React Query `isError` 분기 — 에러를 빈 상태로 위장하는 경로 전수 제거** (특히 홈의 "아직 스캔이 없어요"는 데이터가 성공적으로 비었을 때만)
3. Try again = `refetch()` (전 화면 invalidate 아님 — 해당 쿼리만)
4. 오프라인 판별: **신규 네이티브 라이브러리 금지** (NetInfo 추가 = 리빌드 → 7/24 전 회피). fetch 실패의 TypeError('Network request failed') 계열 = offline(J4), 서버 응답 있는 5xx/4xx = error(J3)로 JS-only 분류. 정확도 한계는 주석으로
5. 로딩: 각 탭 첫 로드에 J1 스켈레톤 (기존 스피너 대체는 홈·음식·프로필만, 과확장 금지)
6. i18n 신규 문구 ×10 로케일

### DoD

- [ ] (기내모드) 세 탭 모두 J4 + Retry 동작 / (서버 5xx — mock으로) 세 탭 J3 + Try again 재요청
- [ ] 진짜 0건일 때만 빈 상태 — false-empty 경로 제거를 테스트로 잠금 (에러 → 에러 렌더 단언, 탭별)
- [ ] tsc 0 · jest 통과 · i18n 패리티

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-007].

## [P-006] ✅ KB-149 후속 — profileImageUrl 전송값 publicUrl → path(objectKey) 교체 (BE 확정 반영) — `28e5bae`

종한 확정(7/16 저녁): ① purpose `PROFILE_IMAGE` 그대로 확정 (BE가 그 값으로 개발) ② **profileImageUrl 필드에는 전체 URL이 아니라 path(objectKey)만 전송** — CDN distribution 교체 대응을 위해 도메인 조합은 백엔드가 담당.

### 할 일

1. profileImage.ts: 업로드 후 전송값을 `publicUrl` → **`path`(objectKey)**로 교체 (온보딩·프로필 수정 PATCH 동일). P-004에서 명시해둔 전환 지점 그대로
2. 조회 렌더: 설계상 MyProfileResponse.profileImageUrl은 **서버가 도메인을 조합한 절대 URL**이어야 FE가 렌더 가능 (FE는 CDN 도메인을 모름 — 그게 이 설계의 목적). 방어: 값이 `http`로 시작하지 않으면 플레이스홀더 처리 + 진행로그에 "조회 응답이 path로 옴 — BE 확인 필요" 기록
3. 업로드 직후 화면 반영: PATCH 후 재조회(['me'] invalidate)로 서버 조합 URL을 받아 렌더 — 로컬 미리보기는 가능하나 재조회 값이 진실
4. 테스트 갱신: PATCH/온보딩 body에 path 전송 잠금 · 조회 비-http 값 방어 1건

### DoD

- [ ] 전송=path, 렌더=서버 절대 URL(비-http 방어 포함) · 기존 테스트 전체 통과 · tsc 0

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-006].

## [P-005] ✅ KB-162 탈퇴 시 애플 재인증 + 토큰 revoke (경로 A — 심사 요건) — `bc2d051`

7/16 결정: 경로 A(클라이언트) 확정. BE 무변. 배경은 Jira KB-162·KB-122 참조.

### 할 일

1. delete-account 화면 분기: **애플 가입 회원**이면 탈퇴 확정 전에 "탈퇴를 완료하려면 Apple로 본인 확인이 필요해요" 안내 → 애플 로그인 시트 재호출 (기존 useSocialAuth의 애플 플로우 재사용 — 새 라이브러리 금지, 기존 경로에서 authorizationCode만 뽑아올 것)
2. 받은 `authorizationCode`(수명 5분)로 즉시 `auth().revokeToken(code)` (RNFB) → 성공 시 기존 탈퇴 API 진행
3. 엣지 처리:
   - 시트 취소 → 탈퇴 중단 + 안내 (계정 유지)
   - **다른 애플 계정으로 인증** → 거부 (현재 로그인 사용자와 대조 — credential의 user/sub 비교)
   - revoke 실패(네트워크 등) → 탈퇴 진행하지 말고 안내 후 중단 (심사 요건이라 revoke 없는 탈퇴를 만들지 않는다)
4. 구글 가입 회원: 기존 흐름 무변 (분기 누락 주의 — provider 판별 근거를 테스트로 잠글 것)
5. i18n: 신규 문구 ×10 로케일

### DoD

- [ ] 애플 회원 탈퇴 플로우에 재인증 게이트, 구글 회원 무변
- [ ] 엣지 3종(취소/타계정/실패) 처리 + provider 분기 테스트
- [ ] tsc 0 · jest 통과. 실기기 확인 포인트 보고에 명시 (설정→Apple로 로그인 목록에서 앱 소멸 / 재가입 시 이메일 선택 재등장)

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-005].

## [P-004] ✅ KB-149 프로필 이미지 업로드 — `4afd89e` — 온보딩·프로필 수정/조회 실연결

presigned 파이프라인(P-003과 동일: upload-url → PUT → complete)을 프로필 사진에 적용. 착수 시 Swagger 실측.

### 계약 참고 (7/16 실측)

- `POST /images/upload-url` `{ purpose, contentType, contentLength }` — purpose 예시는 "MENU_SCAN"이었음. **프로필용 purpose 값은 실측/에러 메시지로 확인**하고, 스펙에 없으면 진행로그에 질의 기록 후 그 값 제외 나머지 완성
- 응답의 `publicUrl`(만료 없는 표시용) vs `objectKey` — `profileImageUrl` 필드에 어느 쪽을 넣는지 계약 명시가 없으면 **publicUrl 우선 시도**(필드명이 Url) + 진행로그 기록
- 회원 API: OnboardingRequest·ProfileUpdateRequest·MyProfileResponse 모두 `profileImageUrl` 보유 (배포 확인됨)

### 할 일

1. 사진 선택: **expo-image-picker 설치 여부 먼저 확인** — 미설치면 추가가 네이티브 변경이라 **리빌드 필요를 보고에 명시** (7/24 전 빌드에 태울 것)
2. 업로드 공용화: scanImage.ts의 발급→PUT→complete 흐름을 프로필과 공유 (purpose 파라미터화 — 중복 구현 금지)
3. 온보딩 profile 스텝: 사진 선택(선택 사항) → 업로드 → OnboardingRequest.profileImageUrl 포함. 스킵 시 필드 생략
4. 프로필 수정: 이미지 교체 → ProfileUpdateRequest.profileImageUrl. 조회(프로필 탭): MyProfileResponse.profileImageUrl 렌더 (없으면 기존 플레이스홀더)
5. 업로드 실패: 정직한 에러 + 사진 없이 진행 가능 (가입/수정 자체를 막지 않음)

### DoD

- [ ] 온보딩 사진 선택→가입 후 프로필 표시 / 수정에서 교체 / 미설정 플레이스홀더
- [ ] 테스트: 업로드 흐름(발급 body·헤더) + profileImageUrl 왕복 + 실패 폴백
- [ ] tsc 0 · jest 통과 · purpose/URL 필드 실측 결과 진행로그 기록

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-004].

## [P-003] ✅ 맵기 -1 센티널 반영 + presigned 발급 연동 (KB-150 후속·KB-72 마무리) — `db95536`

7/16 회의 확정 + presigned 발급 API 배포 직후 상황. ⚠️ 착수 시 Swagger 실측 필수 — 지금 서버 재배포 중(502)이라 계약 세부는 실측으로 확인할 것.

### 작업 1 — 맵기 "미설정 = -1" 정책 반영 (회의 확정, Swagger 무관)

BE 확정: 미설정 유저는 DB에 **int `-1`** 저장, 응답도 `-1`로 옴. 해제도 **PATCH에 `-1` 전송**.
현재 FE는 null/누락 방어라 **-1이 오면 "맵기 -1"이라는 값으로 오작동** (SPICE_SCALE[-1]=undefined, 칩에 "-1/10" 노출 위험).

1. memberAdapter: `spicinessPreference === -1` (또는 0~10 범위 밖) → FE 내부 표현 null(미설정). 로컬 fallback은 서버가 -1 정책으로 항상 값을 주므로 서버값 우선으로 단순화 가능 — 마이그레이션 주석 갱신
2. useMe PATCH: "설정 해제" 시 필드 생략(로컬만) → **`spicinessPreference: -1` 전송**으로 전환. 해제 후 재조회 시 값 되살아나던 한계 소멸 — 관련 주석·PROGRESS 갱신
3. 온보딩 submit: 스킵 시 필드 생략 유지 여부를 스펙/종한 확정에 맞춤 — 생략이 미설정으로 저장되면 유지, 아니면 -1 전송 (실측·판단 후 진행로그에 기록)
4. 테스트 갱신: -1 수신→미설정 렌더 / 해제→-1 전송 / 0은 유효값 유지 (0과 -1 경계 잠금)

### 작업 2 — presigned 발급 연동 (scanImage.ts TODO(KB-72) 채우기) — 계약 실측 완료(7/16 오후)

**`POST /api/v1/images/upload-url`** (로그인 필수) `{ purpose: "MENU_SCAN", contentType, contentLength(정확한 바이트) }`
→ `{ uploadUrl, method(PUT), requiredHeaders, publicUrl(만료 없음), objectKey, expiresAt }`

흐름: 발급 → `uploadUrl`로 PUT (⚠️ **requiredHeaders를 그대로 실을 것** — Content-Type·Content-Length 불일치 시 스토리지 거절) → `completeImageUpload({ path: objectKey, contentType, size })` → 반환 path를 imagePath로.

- 발급 단계에서 용도·형식·크기 검증 — 400이면 텍스트-only 폴백(null 반환) 유지. imagePath `''` 허용은 BE 확정됨(7/16 체크리스트 3번)
- contentLength는 "정확값" — expo FileSystem.getInfoAsync의 size 사용
- KB-137 파일 정리보다 업로드 선행 순서 유지
- 테스트: 발급→업로드→complete 성공 경로 1건(요청 body·헤더 잠금) + 발급/업로드 실패 시 null 폴백 1건

### DoD

- [ ] -1 수신 시 프로필/수정 화면 "미설정" 표시 (칩에 -1 노출 없음), 해제→서버 왕복 후에도 미설정 유지
- [ ] 실기기 스캔 시 사진이 업로드되어 imagePath가 실경로로 전송됨 (progress 로그 등으로 확인 가능하게)
- [ ] tsc 0 · jest 통과 (0/-1 경계 테스트 포함)

완료 시 상태 ✅+커밋 해시, 보고는 REPORTS.md 최상단 [P-003].

## [P-002] ✅ KB-72 스캔 실연결 마무리 — 사진 전송(imagePath) + 가격 표시 — `be9aae5`

Swagger 재배포(7/16 새벽) 실측 기반. 착수 시 https://meogo.handev.site/v3/api-docs 를 다시 실측할 것 — 특히 **presigned 발급 API 존재 여부** (아래 참조).

### 계약 (7/16 새벽 실측)

- `ScanRequest`: `{ imagePath: string, items: [{idx, rawMenuName}] }` — imagePath는 "CDN 도메인 제외 오브젝트 경로, **업로드 완료 신고가 검증한 본인 소유 이미지**"(예: `scan/123/20260715-abc123.jpg`). required 여부는 착수 시 스펙에서 확인
- `POST /api/v1/images/complete` `{ path, contentType, size }` → 서버가 실제 이미지인지 검증 후 `{ path }` 반환 — **이 path를 imagePath로 사용**. 멱등. 400: IMAGE-001(이미지 아님)/002(신고값 불일치)/003(오브젝트 없음)
- `ItemRiskResponse.price`: 원 단위 정수(축약 "1.6"→1600은 서버가 복원), 가격 미표기 메뉴는 **null**, 응답 전용
- ⚠️ **presigned 발급 API가 스펙에 아직 없음** (complete만 배포됨). 착수 시 재확인: ① 있으면 → 촬영 사진을 발급 URL로 PUT 업로드 → complete 신고 → imagePath로 스캔, 전체 흐름 구현 ② 없으면 → 업로드 파트만 어댑터 경계에 스텁(TODO(KB-72))으로 남기고 나머지(요청 형태 전환·가격 표시·complete 연동 코드) 완성, 진행로그에 "발급 API 대기" 기록

### 할 일

1. 스캔 요청을 새 계약으로 전환: 기존 items(idx+rawMenuName) 유지 + imagePath 추가. 온디바이스 OCR은 유지한다(좌표=오버레이, 7/16 예진×종한 합의) — OCR 제거 금지
2. 업로드 흐름: 촬영 원본 → (발급 API) → PUT 업로드 → `/images/complete` → path를 스캔 요청에. 실패 경로 주의: 업로드/신고 실패 시 스캔을 통째로 실패시키기보다, imagePath가 optional이면 텍스트-only로 폴백하고 optional이 아니면 정직한 에러 표시 (가짜 결과 금지 — 헌법 III)
3. 가격 표시: 결과 리스트/오버레이의 메뉴 항목에 price 표시 — **제공값 그대로**(포맷팅만, 환율·추정 금지), null이면 가격 영역 미표시
4. 스캔 후 촬영 파일 정리(KB-137의 `cleaned photo file`)가 업로드 완료 이후에 돌도록 순서 확인 — 업로드 전에 지우면 안 됨

### 안전 게이트 (스캔 = 안전 크리티컬) — ⚠️ 7/16 착수 질의 반영 개정

- matched=false → UNKNOWN/unable 유지, 정렬 마지막, 숨김 금지 (기존 mapRisk 경로 보존)
- **응답에 없는 idx → 기존 7/10 결정대로 드롭 유지** (서버의 비음식 판정 신뢰 — 계약 명시 "메뉴 아닌 항목 제외". 이전 판 "unable 처리" 지시는 철회 — 기존 결정 몰랐던 커맨드 센터 오류, 기존 테스트 3건 유지)
- **idx=null 결과(사진에서만 추출된 메뉴) → 버리지 말 것**: 리스트 뷰에 박스 없이 노출(오버레이 마커는 없음 — 좌표 부재로 정상), 위험도 표시는 동일 규칙. DANGER가 사용자에게 안 보이는 갭 차단(헌법 III). FE 부수 관찰 수용 — 범위 편입

### DoD

- [ ] 새 계약(imagePath+items)으로 스캔 동작, 가격 표시(null 미표시 포함)
- [ ] 업로드 실패/발급 API 부재 시 안전한 폴백 (가짜 safe·크래시 없음)
- [ ] 테스트: 가격 매핑(정상/null) + 요청 body에 imagePath 포함 + idx=null 항목 리스트 노출(드롭 아님)
- [ ] tsc 0 · jest 통과 · 진행로그에 발급 API 존재 여부 기록

완료 시 이 항목 상태 ✅+커밋 해시로 갱신, 보고는 REPORTS.md 최상단에 [P-002]로.

## [P-001] ✅ KB-142·150 후속 — 상세 bookmarked 서버 필드 교체 + 맵기 실연결 — `3494917`

(브릿지 도입 전 채팅으로 전달·완료된 건 — 형식 참고용 사후 기록. 검토 게이트 통과 2026-07-15)

- Swagger 재배포 반영: FoodDetailResponse.bookmarked 기반으로 상세 저장 상태 교체(useIsBookmarked 제거), 낙관 토글이 목록+상세 캐시 동시 반전
- 맵기 TODO(KB-150) 3곳 실연결 (spicinessPreference), 스킵·해제는 필드 생략
- BE 질의 2건 기록: 미설정 표현(required int, 0≠미설정) / 해제(null) 전달 방법
