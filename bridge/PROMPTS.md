# FE 작업 프롬프트 큐

> 운영: 커맨드 센터(스펙 레포 세션)가 항목 작성·수정. **정렬 = 처리 순서** (최상단이 가장 먼저
> 할 일): 긴급하거나 선후 관계(1→2)가 있으면 먼저 할 것을 위로, 순서 무관하면 신규를 최상단에.
> FE 세션은 **위에서부터** ⬜ 항목을 처리하고(예진이 ID 지목 시 그것 우선) **상태만** 갱신:
> ⬜ 대기 → 🔄 착수 → ✅ 완료(커밋 해시 병기). 본문 수정은 커맨드 센터만.
> 완료 보고는 [REPORTS.md](REPORTS.md) 최상단에 같은 ID로 작성.

---

## [P-023] ⬜ 🔴긴급 KB-199 홈 조회 lang 파라미터 — /home?lang= (없으면 홈 400)

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

## [P-028] ⬜ KB-174 후속 — 검색 화면 오프라인 = 전체화면 J4

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

## [P-025] ⬜ (재빌드) KB-202 스캔 캡처 WYSIWYG — 과다 캡처 크롭

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
