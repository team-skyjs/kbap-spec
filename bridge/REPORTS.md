# FE 완료 보고

> 운영: FE 세션이 작업 완료 시 **최상단에** 해당 ID로 보고 작성 (이 파일의 쓰기 주인은 FE 세션).
> 분량은 채팅으로 하던 수준: 커밋 해시, 변경 파일, 항목별 결과, 검증(tsc/jest), 특이사항·질의.
> 커맨드 센터가 검증 후 피드백/반려는 [PROMPTS.md](PROMPTS.md) 최상단에 새 항목으로 남긴다.
> Jira 전환은 여기서 하지 않는다 (검토 게이트 몫).

---

## [P-061] 7/23 QA 묶음 (2026-07-23)

**커밋**: `1034691` (main, 푸시·CI) · **검증**: tsc 0 · jest 243/243 (+2, 강등 잠금 1건 반전) · **preview OTA 발행**(양 채널 — ⚠️ P-060 1차분 동반 탑승, 아래)

- **① 셔터 중복 방지**: `capturing` 상태 — 첫 탭 즉시 비활성(+opacity 피드백), 처리 종료 시 finally 복구. 갤러리 버튼 동일 가드(권장 채택). 전 플랫폼
- **② 미설정=BE 그대로**: 착수 전 **헌법 v2.1.0 개정을 spec 정본에서 실확인** 후 `personalRisk` 강등 폐지 — SAFETY INVARIANT 주석 v2.1.0으로 개정, chokepoint·시그니처는 유지(정책 재변경 대비 한 곳). 스캔 '기피 설정' 배너 안전망 유지. orderCta 강등 잠금 반전 + risk.test 신설(미설정 4상태 그대로)
- **③ 안전 고지 행**: `safety.html` 열기(Linking.openURL) 연결. **실측**: "기존 약관·개인정보 행"은 코드에 부재 — 안전 고지 행만 존재(미연결 상태)했음, openURL 표준 방식 채택
- **④ 알림 UI 전면 삭제**: NotificationsPanel.tsx 삭제 · StickyHeader bell/bellDot/패널/게이트 제거 · 홈/음식/프로필 진입점 제거 · 프로필 알림 행 삭제 · GateContext 'notifications' 제거 · i18n(notifications.*, profile.notifications, gate.notif*) ×10 정리 — 주석 아웃 아닌 삭제
- **OTA 참고**: main에 P-060 1차분(피커 철거·OS 정본화)이 있어 동반 탑승 — preview 기기의 언어 확인은 **기기 전체 언어 변경**으로 가능(앱별 언어 항목은 최종 빌드부터). P-061 지시의 OTA를 우선함

## [P-060] 언어 OS 정본화 — 1차 중간 보고 (2026-07-23, 🔄 진행 중)

**커밋**: `0b7c02b` (main, 푸시·CI) · **검증**: tsc 0 · jest 241/241 (+2, P-015 라이브 전환 테스트 폐기) · **OTA 미발행(사유 아래)**

즉시 가능분(1·4·5·6) 완료:
- **① LocaleProvider 단순화**: kbap.lang 저장·복원/`setLang`/라이브 전환·언어 쿼리 무효화(P-015·P-018 계열) 전부 제거 — 언어 = `getLocales()` 기기 언어(미지원=en), `resolveInitialLang` 시그니처 유지(bootGate 무변). 온보딩 국적→언어 제안도 소멸
- **④ 피커 철거**: `LanguagePicker.tsx` 삭제 · 온보딩 언어 입력 삭제 — **실측: 별도 스텝이 아니라 프로필 스텝 내 필드**였음 → 필드 행 삭제, 스텝 수·진행 표시 무변 · 프로필 탭+수정 화면 언어 행 → `Linking.openSettings()`(iOS 앱 설정·안드13+ 앱 정보의 언어 항목), **안드12- 행 숨김**(Platform.Version<33)
- **⑤ OS 로케일 선언**: expo-localization plugin `supportedLocales`(ios/android ×10종) — `CFBundleLocalizations` 생성은 expo config로 확인, 안드 localeConfig는 빌드 시 생성. **네이티브 — 최종 빌드(iOS8/안드3) 탑승, OS 설정 노출은 빌드 후 확인 항목**. app.json 회전은 기존 발행 스왑이 커버(추가 항목 없음)
- **⑥ 테스트**: 기기 언어 추종·en 폴백·resolveInitialLang=기기 잠금 +2

잔여(스웨거 게이트):
- **②appLanguage 전송 중단·③/scans lang** — 착수 시점 실측: dev 스웨거에 /scans lang 파라미터 없음·appLanguage 11회 잔존(마감 시점 재확인도 동일). **BE 갱신 신호 오면 즉시 이어서**

**OTA 보류 판단(보고 요망 사항)**: 피커 제거를 OTA로 선발행하면 ⑤(OS 언어 항목)가 빌드 전이라 **언어 변경 수단 공백** 구간이 생김 — 브리프 타이밍(7/24 제출 빌드 일괄)에 맞춰 빌드 탑승이 정합. Metro(dev client)에선 지금 즉시 확인 가능

## [P-059] KB-175 API 도메인 전환 1단계 (2026-07-23)

**커밋**: `64d1be6` (main, 푸시·CI) · **검증**: tsc 0 · jest 240/240 (+2) · 발행 불요(빌드/OTA 무변 — dev 전용 배선)

- 확정 배선 그대로: **dev(Metro/.env)=dev.kbap.site** (.env는 untracked+gitignore, `.env.example` 팀 문서화) / **preview·production=eas.json 각 프로파일 env에 meogo 명시 고정** — dev 도메인이 빌드에 스밀 경로 차단. config.ts에 도메인 로드맵 주석(prod는 시딩 신호 후 후속 P, meogo는 그때 폐기+fallback 정리)
- **dev 계약 동일 실증(curl)**: `/home`·`/foods`·`/foods/{id}` — 엔벨로프(code/message/payload/success)·payload 키·아이템 필드 **dev=meogo 완전 동일**, dev 20종 시딩 확인. Metro 실기 스모크(스캔 POST 포함)는 예진 확인 요청
- ⚠️ **fingerprint 영향(지시 2 확인 항목)**: eas.json 편집이 회전 유발 — P-017 선례 재확인. **추가 발견: .gitignore 파일 자체도 해시 대상**. 발행 스왑 목록 확장(app.json·eas.json·.gitignore 되돌림 + 안드 스플래시 에셋·.env 치우기)으로 베이스라인(f96ae4f7/c43664ed) 복원을 실측 검증 — FE 메모리에 절차 갱신, 빌드3 재베이스라인 때 전체 소멸
- 테스트 +2: BE_BASE env 분기 / meogo fallback

## [P-058] KB-125 후속 — 다양성 행 dim 예고 (2026-07-22)

**커밋**: `762f3b7` (main, 푸시·CI) · **검증**: tsc 0 · jest 238/238 (+1) · **preview OTA 발행**(양 채널 도달)

- 다양성 BreakRow를 리뷰 행과 **동일 처리**: 공용 `ComingRow` 컴포넌트로 추출(P-048 인라인 리뷰 행도 이전 — 중복 제거) — opacity 0.55 + IconLock + `ranking.reviewsComing` 재사용, 탭 무반응. 점수 로직 무변
- 카드 시각 균형(지시 2): dim 2행(리뷰·다양성) → 활성 스캔 1행 — 행 간 breakRowBorder 유지로 리듬 정상, 첫 행(리뷰)만 border 없음(기존 규칙)
- 테스트 +1: 다양성 라벨 렌더·실적 detail 부재·예고 문구 2행·스캔 활성 잠금

## [P-057] KB-212 후속 — 스캔 배너 리스트 첫 카드 편입 (2026-07-22, A안)

**커밋**: `6f07859` (main, 푸시·CI) · **검증**: tsc 0 · jest 237/237 (+1) · **preview OTA 발행**(양 채널 도달)

- 확정 스펙 그대로: absolute 오버레이 폐기 → **결과 리스트 첫 아이템**(메뉴 카드와 같은 폭·radius·마진 리듬, 스태거 대열 첫 항목 — delay 0으로 P-032와 간섭 없음)
- 스타일: 밝은 브랜드 틴트 `#fdf0e6`/테두리 `#f0d9c4` · 좌측 원형 primary **IconBulb**(신규 SVG — 기호=SVG 규칙) · 본문 진갈색(ink2) + **"기피 재료를 설정"만 primary 강조**(i18n을 nudgeAction/nudgeRest 2키로 분리 ×10 — 어순 안전) · 우측 ×
- **목록 뷰 전용**(지시 4): 위험도/원본 토글 시 미렌더 — 테스트로 잠금
- 동작·노출 조건 무변(탭→기피 설정, ×→세션 억제, 회원&&기피0) — 기존 4케이스를 카드 구조로 이전 + 목록 전용 1케이스 추가

## [P-056] 🔴 KB-176 안드 스캔 오버레이 좌표 2배 오프셋 (2026-07-22)

**커밋**: `c44028f` (main, 푸시·CI) · **검증**: tsc 0 · jest 236/236 (+5) · **preview OTA 발행**(양 채널 도달 — Android `f96ae4f7`·iOS `c43664ed`)

- 실측 원인 그대로 구현: `chooseDenominator` — **frame 최대 좌표(maxRight/maxBottom)를 수용하는 치수 채택**. 우선순위 measured(iOS 정답 — KB-141 무회귀) → reported(안드형: 절반 저장 파일과 ML Kit 원치수 기준 불일치 케이스) → 둘 다 못 담으면 큰 쪽(클램프 손실 최소화). 반올림 여유 2%. 플랫폼 하드코딩 없음 — 두 치수가 정수배 관계면 스케일 보정과 동치
- **채택 근거 로그**(지시 1): `[ocr] denominator = {pxW, pxH, basis, maxRight, maxBottom}` — 실기 adb로 basis=reported 확인 가능
- **부수 확인(지시 3 — 조사만)**: 절반 저장은 안드 저장 경로(P-025 ImageManipulator saveAsync 추정). OCR 인식은 정상(부대찌개 등 전 라인 검출) — 2040px면 메뉴판 텍스트 해상도 여유, 화질 이슈 미관측. 필요 시 별도 과제 의견
- 테스트 +5 (chooseDenominator.test): **adb 실측 케이스 그대로**(top=2051>2040→reported) · iOS형 measured 유지 · measured null 폴백 · larger-fallback · 2% 경계 여유
- 실기 확인 요청: 안드에서 6개 마커 전부 해당 메뉴 옆 정위치([ocr] denominator basis=reported 로그 동반) + iOS 마커 무회귀

## [P-055] KB-225 안드 하단 내비바 클리어런스 전수 (2026-07-22)

**커밋**: `1c8a09c` (main, 푸시·CI) · **검증**: tsc 0 · jest 231/231 (+2) · **preview OTA 발행**(P-054 스왑 절차로 — Android `f96ae4f7`=build2·iOS `c43664ed`=빌드7 도달 확인)

- **공용 승격**: `useBottomInset()`(안드 `max(bottom, 48)` floor / iOS 실측 통과) — P-021 온보딩 국소 처치를 훅으로 추출, 온보딩도 교체
- **전수 적용(지시 목록 그대로)**: `AuthGateSheet`(이번 짤림 — **원인은 인셋 미반영 고정 paddingBottom 34**; 안드만 18+보정 인셋, iOS 34 유지=무회귀) · `login` · `intro` · `scan` 2곳(카메라/결과 하단 바) · `owner` · `order`. `TabBar`는 실기 정상 전제 무변(과보정 회피 — 짤림 확인 시 동일 유틸 적용)
- **실기 확인 목록(3버튼 내비 — 에뮬 설정 전환)**: ① 게이트 시트 '나중에 하기'(비로그인 알림/스캔 진입) ② 로그인 하단 약관/버튼 ③ 인트로 CTA ④ 스캔 카메라·결과 하단 바 ⑤ owner/order Done ⑥ 온보딩 CTA(기존 정상 확인분 — 회귀 체크)
- 테스트 +2 (bottomInsetFloor 분기 잠금)
- 발행 절차 참고: P-054 회전분 때문에 app.json+안드 스플래시 에셋 2파일 임시 되돌림으로 발행(절차 정상 동작 확인) — 빌드3 후 소멸

## [P-054] KB-194 후속 — Android 12+ 스플래시 원형 마스크 (2026-07-22)

**커밋**: `18de75a` (main, 푸시·CI) · **검증**: tsc 0 · jest 229/229 · ⚠️ **OTA 불가 — 안드 빌드3/차기 iOS 빌드 합류**(제출용 최종 빌드 배치, 커맨드 센터 의견대로 단독 빌드 없음)

- 안드 전용 에셋 분리: 기존 png의 심볼 밴드(300×300, 알파 행 분석으로 추출) → **450 정사각 캔버스 중앙 배치 = 원형 안전영역 2/3** (`assets/images/splash-icon-android.png`). app.json expo-splash-screen에 `android: { image, imageWidth: 200 }` 오버라이드 — iOS는 기존 조합 이미지(230) 무변, 배경 `#E2580C` 무변
- **미리보기 첨부**: `bridge/assets/p054-android-splash-preview.png` — 원형 마스크 시뮬, 원 안에 K 볼 심볼 온전
- ⚠️ **특이(운영)**: 이 커밋으로 fingerprint 재회전(안드 `c71456e3` ≠ build2 / iOS `584a401b` ≠ 빌드7) — **빌드3 전 OTA 발행 시 app.json+안드 에셋 2파일 임시 되돌림 필요**(축소판 스왑 부활, FE 세션 메모리에 절차 기록). 빌드3 재베이스라인으로 소멸 예정

## [P-053] KB-206 반려 — 게이트 시트 애니 원복 (2026-07-22)

**커밋**: `3ef625f` (main, 푸시·CI) · **검증**: tsc 0 · jest 229/229 무회귀 · **preview OTA 발행**

- AuthGateSheet 등장의 `SlideInDown.springify()`(P-031 B5) 제거 — **Modal fade 단독 원복**(P-031 직전 상태, git 이력 대조). 퇴장·backdrop 무변
- Snackbar 등장 스프링은 유지(지시 — 반려 대상 아님, 시트만)
- 제거 사유 주석 축적(P-035 기피 칩·P-047 헤더와 같은 절제 계열 — 모멘텀 없는 게이트 시트에 스프링 부적합)

## [P-052] 🔴 KB-215 반려 — 사장님 질문 원문 노출 봉쇄 (2026-07-22)

**커밋**: `82d54b9` (main, 푸시·CI) · **검증**: tsc 0 · jest 229/229 (+3분기 5케이스) · **preview OTA 즉시 발행**

- 반려 분석 그대로: 상세 IngredientResponse엔 81종 코드가 없어 어댑터 합성 라우트 키(`ing:{i}:{name}`)가 P-045의 `ownerQuestionKo`에 코드로 흘러 매핑 실패 → 원문 통과
- ① **원천 봉쇄**: `resolveIngredientKo` 실해석 실패(또는 미지 형식)면 **일반 질문 강등** — 어떤 입력이든 내부 식별자가 사장님 화면에 뜨지 않음(false-safe 계열)
- ② **실해석**: 합성 키에서 name 추출 → 81종 **en 카탈로그명·현재(요청) 언어 라벨·ko 라벨** 역인덱스 검색(소문자·공백 정규화) → 코드 → ko 라벨. 예: `ing:0:Egg`→"달걀이", `ing:3:shrimp`→"새우가"(대소문자), `ing:2:달걀`→ko 역매핑
- ③ 역매핑 실패(서버 번역 상이)도 일반 질문 — 지시의 "원문보단 덜 구체적인 게 항상 낫다"
- 테스트: 합성 키 3형(en/대소문자/ko) · 미지 name·미지 형식 → 일반 질문 · 직접 코드(SHRIMP) 기존 동작 — 3분기 잠금

## [P-051] KB-195 후속 — 맵기 UI 원복: 화면=전송 일치 (2026-07-22)

**커밋**: `13be68e` (main, 푸시·CI 재실행) · **검증**: tsc 0 · jest 226/226 · **preview OTA 발행**

- 확정 원칙 그대로: 기본 **5 표시**(P-039 미선택 UI — 대시·"불꽃을 눌러 선택" 힌트·회색 불꽃·노브 숨김 — 전부 제거), **Continue(미조작 포함) = 화면값 그대로 제출**(기본이면 5), **Skip = -1**(P-019 경로)
- **P-039의 stale closure 수정은 유지**(지시): 제출은 호출측 인자 — Continue=`finish(spice)`(화면의 그 값), Skip=`finish(null)`(UNSET). skipped 플래그는 draft 기록용으로만
- draft 호환: 저장값 있으면 그 값, **null(구 스킵분)이면 5 표시** — 테스트 하네스가 null draft 복귀로 이 경로를 겸검증
- i18n `onboarding.spiceUnsetHint` ×10 제거(미사용화 정리)
- 테스트 교체(지시 3케이스): 미조작+Continue→**5** / 조작 7+Continue→7 / Skip→UNSET(body -1은 submit.test 체인)

## [P-050] KB-219 CI — GitHub Actions tsc+jest (2026-07-22)

**커밋**: `b5832b9` (main, 푸시됨) · **1회 green 확인**: https://github.com/team-skyjs/kbap-fe/actions/runs/29887022483 (run #1 success — 로컬과 동일 결과 226/226)

- `.github/workflows/ci.yml`: push(main)+PR → `npm ci` → `tsc --noEmit` → `jest --ci`. node 20(로컬 v20 일치)·npm 캐시. 신뢰 불가 이벤트 입력 미사용(정적 명령만)
- **브랜치 보호 안내(예진 콘솔 몫)**: GitHub → Settings → Branches → main 보호 규칙에서 "Require status checks to pass" 켜고 `check` 잡 지정 — 이후 실패 시 병합 차단
- 참고: Actions 런타임 Node20 deprecation 어노테이션은 액션 자체 경고(우리 스텝의 node 20과 무관, 무해)

## [P-049] KB-218 프로필 사진 촬영 (2026-07-22)

**커밋**: `7fcfb35` (main) · **검증**: tsc 0 · jest 226/226 (+6) · **preview OTA 발행** (재빌드 불요 확인 — 기설치 expo-image-picker)

- **시스템 UI만(지시)**: `choosePhotoSource` — iOS `ActionSheetIOS`(촬영/갤러리/[사진 있으면]**삭제 destructive**/취소), Android **시스템 Alert 3버튼**(촬영/갤러리/취소). ⚠️ Android Alert는 3버튼 상한이라 삭제는 시트에 못 실림 — **기존 화면 삭제 링크 존치**로 커버(iOS는 시트+링크 양쪽)
- 촬영: `launchCameraAsync` 1:1 크롭(갤러리 경로와 동일 옵션) → 기존 업로드 파이프라인 합류. **권한 거부 = 정직한 안내 + 설정 유도 Alert**(Linking.openSettings) 후 null — 가입/수정 흐름 불막음
- 적용처: 프로필 수정 + 온보딩 사진 스텝(공용 `pickBySource` 경로 — 온보딩은 삭제 옵션 없음, 재선택 대체 기존과 동일). 삭제 P-016 로직 재사용, busy/에러 표시 무변
- i18n `photo.*` 6키 ×10. iOS NSCameraUsageDescription은 expo-camera 플러그인 문구 기존재 — **실기 권한 플로우 확인은 예진 요청**(DoD 항목)
- 테스트 +6 (photoSheet.test): iOS 시트 옵션·인덱스 매핑(사진 유무별) / 안드 3버튼 / 권한 거부·허용·취소

## [P-048] KB-125 랭킹 리뷰 흔적 정리 (2026-07-22)

**커밋**: `4ec6823` (main) · **검증**: tsc 0 · jest 220/220 (+2) · **preview OTA 발행**

- ① 하단 **리뷰 쓰기 CTA 삭제** — 스캔 CTA를 단독 filled로 정리 ② **breakTotal("리뷰 하나 더 +10점") 행 삭제** — P-037의 하단 radius 보정도 행 제거로 자연 소멸, 마지막 행 카드 마감 정상(패딩 내 종결)
- ③ 리뷰 팩터를 FLAGS 무관 **dim 예고 행** 상시 노출: opacity 0.55 + IconLock + `ranking.reviewsComing`("다음 업데이트에 제공돼요") ×10 — BreakRow와 같은 골격, 탭 요소 아님(무반응). 점수 로직(ranking.ts) 무변
- 음식 상세·프로필 리뷰 흔적은 범위 외(지시) — FLAGS 숨김 유지
- 테스트 +2: oneMore·리뷰 CTA 부재/스캔 CTA 유지 · dim 예고 렌더

## [P-047] KB-217 헤더 바운싱 — timing 복귀 (2026-07-22)

**커밋**: `ea552db` (main) · **검증**: tsc 0 · jest 218/218 · **preview OTA 발행**

- **원인 실조사(요청 항목)**: 의심 그대로 P-031 소행 — 숨김/복귀를 withSpring으로 바꾼 뒤, 빠른 방향 반복 스크롤에서 **방향 전환마다 스프링이 미정착 상태에서 재타겟되며 이전 속도를 승계** → 0↔1 목표 사이 진동이 translateY로 노출. timing 시절엔 고정 200ms라 재타겟 즉시 새 궤적으로 수렴해 무증상
- **수정**: 과제의 유력안 중 **timing 복귀** 채택(히스테리시스 추가보다 단순·검증된 원상) — 이 헤더의 숨김/복귀만 withTiming(200ms ease-out), "스크롤 크롬은 즉답 우선" 절제 원칙 그대로. 북마크 팝 스프링·타 화면 스프링(P-031 나머지)은 무변
- StickyHeader 전 사용처(홈·음식탭·상세) 공통 반영 — 훅 한 곳 수정
- 기존 스위트 무회귀로 검증(모션 파라미터 자체는 테스트 대상 아님 — 실기 재확인 요청)

## [P-046] KB-216 스캔 진입 오프라인 게이트 (2026-07-22)

**커밋**: `e7ee492` (main) · **검증**: tsc 0 · jest 218/218 (+3) · **preview OTA 발행**(Android build2 도달)

- 카메라 phase 진입 시 오프라인이면 **카메라 미기동 + 전체 J4**(QueryErrorBlock)+Retry — 판정은 P-027/028과 동일 메커니즘(useInfiniteFoods 프로브, 캐시 공유). Retry 성공 시 카메라 기동
- 촬영 화면 자체를 대체하므로 **갤러리·샘플 스캔 진입로도 자연 차단**(하단 바 미렌더). 서버 5xx(J3)는 게이트 미발동 — 스캔은 오프라인만 불가(테스트로 잠금). J4는 밝은 배경으로 타 탭과 톤 통일
- 테스트 +3: 오프라인→J4·카메라/갤러리/샘플 미렌더 / 온라인 무회귀 / 5xx 미발동. 기존 scan 테스트 2본에 프로브 mock 보강

## [P-045] KB-215 사장님 카드 실데이터 조립 (2026-07-22)

**커밋**: `23a447a` (main) · **검증**: tsc 0 · jest 215/215 (+3) · **preview OTA 발행(스왑 소멸 후 첫 발행 — Android build2 `f96ae4f7` 도달)**

- mock `mocks/owner.ts` **삭제** → `useOwnerConfirmation`을 클라 조립으로 재작성: 상세 캐시 `nameKo`(스캔 미등록 흐름도 decode된 실명) + `ownerQuestionKo()`(81종 코드→ko 라벨 + 신규 **이/가** 조사 유틸 — P-030 을/를과 동일 산술) — 어느 음식이든 `{실명}에 {재료ko}이(가) 들어가나요?`, 재료 없는 진입은 `{실명}에 제가 못 먹는 재료가 들어가나요?`
- explanationKo 정적 유지(지시대로). renderQuestion 하이라이트는 menuNameKo=실명이라 자연 동작
- **전수 결과**: 사장님 노출 화면에서 음식명·재료명 mock/하드코딩 잔재는 owner 경로의 PHRASES 사전이 유일했음(삭제) — order.tsx는 기왕 실데이터(nameKo·81종 라벨). MOCK_FOOD_DETAILS는 mock-era 상세 흐름용으로 사장님 화면 자체와 무관(무변)
- 테스트 +3: iGa 분기 · 재료 있음("김치찌개에 새우가"/"비빔밥에 달걀이") · 재료 없음 일반 질문 — UI 언어 en 상태에서 ko 고정 겸 잠금

## [P-043] KB-176/198/202 — Android 빌드2 (2026-07-21)

**빌드**: `24277e9c` FINISHED · **runtime `f96ae4f7`** · 코드 무변(기록 커밋 `9b1a431`) · keystore 기존 EAS 관리

- **설치**: 빌드 페이지(QR) https://expo.dev/accounts/rocher/projects/kbap/builds/24277e9c-2b9e-401b-853d-353c7fc403ea · apk 직링크 https://expo.dev/artifacts/eas/Km94RGF5sYIm3DeTRYTxLhzFyisKvKVFZMSRohvfwYk.apk
- **합류분**: expo-sensors 가로 감지(P-022 — 안드 실동작 첫 탑재), WYSIWYG 캡처 크롭(P-025), 스캔 세그먼트 글라이드·스태거(P-032), 빈 프로필 스캔 배너(P-038) + 최신 JS 전량(P-042까지) — 공기계에서 Q-14 잔여 전부 확인 가능
- **스왑 절차 완전 소멸 선언**: iOS(7/21 빌드7 `c43664ed`)에 이어 Android도 재베이스라인 — **이후 양 채널 OTA는 스왑 없이 발행**. 구 build1(cbbec117)은 orphan — 공기계는 새 apk 재설치가 기준(구 apk 위 OTA는 더 안 감)
- 세션 메모리의 스왑 절차 노트 2건 삭제(소멸 반영)

## [P-042] Q-18 후속 3건 — press 확산 · 스테퍼 위치 · 노드 팝 제거 (2026-07-21)

**커밋**: `1455b96` (main) · **검증**: tsc 0 · jest 212/212 (+1) · **preview OTA 발행**

- ① **press 확산**: 공용 `PressScale` 래퍼 신설(Btn과 동일 프리셋 spring.press·0.97·onPressIn 즉발) 후 생 Pressable 버튼류 전수 적용 — **StickyHeader**(back·search·bell·bookmark·**Sign in pill**), **SubHeader** back, **search** back, **owner/order** close·Done, 주문 **StepBtn**. **제외 판단(과적용 금지)**: 리스트 행/카드 내비(지시), ranking Cta(기존 pressed 배경 반응 보유), 텍스트 링크류(saveLink 등 — 버튼 모양 아님), 시트 내부 요소, 스캔 화면 버튼(OTA 스왑 표면 밖 — 빌드2 합류 시 별도 1건으로)
- ② **스테퍼 위치**: 주문카드 수량 스테퍼를 문장 아래 → **Done 검정 버튼 바로 위**(foot 내부)로 이동. 문장 내 {n}개 갱신·값 팝 무변 — 기존 orderStepper 잠금 테스트 통과로 확인
- ③ **스텝 노드 팝 제거**: TopBar Seg를 plain View로 원복(P-032 ⑦ 폐기) — 제거 사유 주석(P-035 기피 칩 선례와 동일 절제 사례). 성공 체크 드로잉(⑤)은 무변 유지
- 테스트 +1 (uiPolish — PressScale 즉발·복귀 잠금)
- **preview OTA**: Android runtime `cbbec117` = build1(스왑 후 복원·트리 클린). 프로덕션은 다음 묶음 후보

## [P-041] 🔴 KB-152 재수정 — 부트 레이스: 정리→프리페치 직렬화 (2026-07-21, 프라이버시)

**커밋**: `3010a8d` (main) · **검증**: tsc 0 · jest 211/211 (+2) · **preview+production 양 채널 OTA 발행**

- 원인 분석 그대로: `cleanupIfFreshInstall()`과 `prefetchBootData()`가 같은 틱 병렬 발사 — 신규 설치 첫 부팅에서 프리페치의 `hasBeSession()`이 아직 안 지워진 옛 토큰으로 /home·/me 인증 프리페치 → 홈 캐시 오염
- 수정: bootGate에 `prefetchAfterCleanup(cleanupDone, prefetch)` — **cleanup settle 후에만 프리페치 시작**(순서 불변식의 단일 구현, cleanup 실패해도 프리페치는 진행 — 부트 불막음). `_layout` 배선 + 주석 "정리가 프리페치·렌더 모두보다 선행"으로 갱신(렌더는 entryChecked, 프리페치는 직렬화가 담당). gateSplash min/cap 시맨틱 무변 — cleanup은 AsyncStorage 체크 1회라 비신규 설치 체감 무변
- 테스트 +2 (bootGate.test): cleanup 완료 전 prefetch 미시작(순서 검증 모킹 — 지시 그대로) / cleanup reject여도 진행. 기존 게이트 타이밍 4케이스 무회귀
- **발행(프라이버시 — 프로덕션 우선)**: production iOS `019f83b9-e332-7044` (runtime `8d0d5504` = 테플 빌드) + preview Android (runtime `cbbec117` = build1). ⚠️ 두 채널은 gitignore 스왑 여부가 달라 **2단 발행**(production=gitignore 제거 상태, preview=복원 후) — Android fingerprint가 gitignore에도 반응함을 실측, 절차 노트로 남김. 스왑 전량 복원·트리 클린
- 예진: 재설치→게스트 진입 실기 재확인 부탁 (테플은 재실행 2번 후)

## [P-040] KB-205 재수정 — 스테퍼 −/+ SVG 교체 (2026-07-21, Q-17 대응)

**커밋**: `eb30502` (main) · **검증**: tsc 0 · jest 209/209 · **preview OTA 발행**

- `order.tsx` StepBtn: 텍스트 글리프 제거 → `IconMinus`(신규, 수평선 1개) + `IconPlus`(재사용). 크기 20·C.ink·비활성(1/5 경계) opacity 표현 무변. `stepText` 스타일 삭제 — SVG는 자체 viewBox 중심이라 컨테이너 center 정렬이 그대로 시각 중심
- **전수 점검 결과(보고만)**: 기호를 아이콘 대용으로 텍스트 렌더하는 곳은 **order.tsx −/+가 유일**했음. 나머지 검출분은 활자 구두점으로 판단 — `·`(saved/reviews 메타 구분점), `—`(평점 빈값 대시), `≈`(맵기 비유 접두), `–/10`(P-039 미선택 대시): 전부 문장 흐름 내 텍스트라 정렬 문제 무관, 교체 불요 의견
- 영구 규칙(기호=SVG) CLAUDE.md 반영은 커맨드 센터 몫으로 확인
- 동작 무변 — 기존 orderStepper 잠금 테스트 통과로 검증
- **preview OTA**: Android runtime `cbbec117` = build1(스왑 후 복원·트리 클린). 프로덕션은 다음 묶음 후보. 예진: 인스펙터로 원 정중앙 재확인 부탁

## [P-039] 🔴 KB-195 재수정 — 맵기 미조작=미설정 (2026-07-21)

**커밋**: `4d3e9d8` (main) · **검증**: tsc 0 · jest 209/209 (+3) · **preview OTA 발행**

- ① `useState(5)` → `useState<number|null>(null)`: **기본 미선택**. 불꽃을 탭해야만 값 확정, 미조작+계속 = UNSET(→body -1, 건너뛰기와 동일)
- ② 미선택 UI: 큰 숫자 자리 대시(–/10) + "불꽃을 눌러 선택하세요" 힌트(`onboarding.spiceUnsetHint` ×10 — 실제 조작이 불꽃 탭이라 문구도 탭 기준), 불꽃 전열 회색·트랙 노브 미표시 — "기본 5로 보이는 착시" 제거
- ③ **기저 버그 추가 발견·수정**: 제출식이 `skipped.spice` 플래그를 참조했는데, 같은 탭에서 setState 직후 finish()가 실행되면 closure가 stale — (a) draft 복귀(skipped=true) 후 **조작해도 UNSET 제출**, (b) 조작 후 건너뛰기가 **실값 제출** 경로. 수정: 제출식에서 skipped 제거하고 **값의 null 여부를 단일 진실**로, `finish(spiceFinal)` 인자화로 계속=화면의 그 값/건너뛰기=null을 명시 전달. 조작 즉시 skip 해제(setLevel 래퍼)로 draft 저장도 일관
- ④ draft null 왕복 유지(`d.spice ?? null`), 명시 건너뛰기 버튼 존치, 프로필 수정 화면 무변
- 테스트 +3 (onboardingSpiceUnset.test.tsx — draft 직행 하네스): 미조작+계속→UNSET / 불꽃 7 조작+계속→7 / 건너뛰기→UNSET. **UNSET→body -1은 기존 submit.test가 잠금**(체인 완결)
- **preview OTA**: Android runtime `cbbec117` = build1(스왑 후 복원·트리 클린). **Metro body 실측(DoD)은 예진 확인 요청** — 미조작+계속 시 `spicinessPreference: -1` 나가는지

## [P-038] KB-212 빈 프로필 첫 스캔 배너 (2026-07-21)

**커밋**: `acedcd1` (main) · **검증**: tsc 0 · jest 206/206 (+4) · **preview OTA 발행**

- 스캔 결과 상단 비차단 1줄 배너: **회원 && 기피 0 && 세션 내 미닫음**일 때만. 게스트는 라우트 가드로 결과 화면 진입 자체가 없어 자연 제외(테스트로 확인). absolute 오버레이(Close 버튼 우측, 리스트 상단 여백 위)라 리스트/오버레이 토글 레이아웃 불변 — "안 밀게" 충족
- 탭 → `/profile/restrictions`(기피 설정) · × → **세션 내 억제**(신규 `nudgeSession.ts` 모듈 플래그 — AsyncStorage 불사용, 재실행 시 재노출이 사양)
- i18n `scan.nudge` ×10("기피 재료를 설정하면 이 메뉴판에서 피할 걸 짚어드려요" 톤 통일), SVG 아이콘만
- 테스트 +4 (scanNudge.test.tsx): 기피0 회원=노출 / 기피1+=미노출 / 게스트=미노출 / 닫기 후 재마운트 억제 — 지시 4케이스 그대로
- **preview OTA**: Android runtime `cbbec117` = build1(스왑 후 복원·트리 클린). ⚠️ 스캔 화면 변경이라 **build1 preview에선 확인 불가**(OTA 스왑이 scan.tsx를 pre-P022로 되돌림 — P-032 스캔 효과와 동일 트레이드오프) — 실확인은 dev 빌드 또는 빌드2/차기 iOS 빌드에서

## [P-037] KB-125 랭킹 UI 2건 — 현재 카드 정렬 · 점수내역 코너 (2026-07-21, Q-15 대응)

**커밋**: `31e77e9` (main) · **검증**: tsc 0 · jest 202/202 · **preview OTA 발행**

- ① 현재 등급 카드(`nodeBodyCur`)에 `marginHorizontal: -14.5` — 카드 자체 패딩 13 + 테두리 1.5가 안쪽으로 밀던 것을 정확히 상쇄. 내부 텍스트("맛보기"·"30점 이상")가 비카드 행들과 같은 좌/우 기준선, 카드 테두리는 기준선 바깥으로(지시의 음수 마진 방식 그대로). 우측 확장분은 화면 패딩 18 안에 수용(3.5px 여유)
- ② `breakTotal`에 `borderBottom(Left/Right)Radius: 19`(카드 radius 20 − 테두리 1) — 음수 마진으로 확장된 배경의 하단 코너 뚫림 봉합. 지시대로 overflow:'hidden' 회피(iOS 그림자 클립 + 안드 elevation 회색 P-024 선례)
- 스타일 상수 2건뿐(신규 로직 없음) — 기존 스위트 무회귀로 검증, 상수 자체의 테스트는 동어반복이라 생략
- **preview OTA**: Android runtime `cbbec117` = build1 일치(스왑 후 복원·트리 클린). 프로덕션은 다음 묶음 후보. 예진: 랭킹 화면에서 포토샵 기준선 재확인 부탁

## [P-036] KB-174 재수정 — 음식탭 캐시+오프라인 J4 · 스피너 공전 (2026-07-21, Q-08 대응)

**커밋**: `dee7567` (main) · **검증**: tsc 0 · jest 202/202 (+2) · **preview OTA 발행**

- ① `food.tsx`: 커맨드 센터 원인 분석 그대로 — `isError`면 **FlatList 자체를 전체화면 QueryErrorBlock으로 조건부 대체**(ListEmptyComponent 방식 폐기). 캐시 유무 무관 J3/J4, 리스트 미렌더라 onEndReached·푸터 스피너 경로도 원천 차단. 헤더 미렌더(P-027)·Retry refetch 유지
- ② `Spinner.tsx`: 회전 래퍼에 `width/height = size` + `alignSelf: 'center'` 고정 — 전폭 stretch 띠 회전(공전) 원천 봉쇄. 사용처 전수(스캔·검색·상세 푸터·저장 목록 등) 확인 — 기존 좁은 컨테이너에선 렌더 결과 동일(무회귀)
- 테스트 +2: **캐시 존재+NETWORK 에러 → J4 렌더·캐시 카드/헤더 미렌더**(tabStates — Q-08 ① 재현 케이스 그대로 잠금) · Spinner 래퍼 크기·center 고정(uiPolish)
- **preview OTA**: Android runtime `cbbec117` = build1 일치(스왑 후 복원·트리 클린). 프로덕션은 다음 묶음 후보

## [P-035] KB-207 재수정 — 기피 칩 애니메이션 제거 (2026-07-21, Q-11 대응)

**커밋**: `c0c485f` (main) · **검증**: tsc 0 · jest 200/200 · **preview OTA 발행**

- IngredientFilter 요약 칩의 entering/exiting(P-032 ① ZoomIn/ZoomOut) 완전 제거 — 즉시 나타남/사라짐. 공용 컴포넌트 한 곳이라 온보딩·프로필 동시 반영
- P-026 고정높이 36·placeholder 무변(기존 잠금 테스트 통과 유지), 타 화면 P-032 효과(북마크·스테퍼·세그먼트·스태거 등) 무변 — 이 화면 칩만 원복
- 제거 사유를 코드 주석으로 남김(Q-11 절제 사례) — 재도입 방지
- **preview OTA**: Android runtime `cbbec117` = build1 일치(스왑 후 복원·트리 클린). 프로덕션은 다음 묶음 후보

## [P-034] KB-203 재수정 — 계정 연동 아이콘 공식 로고 (2026-07-21, Q-16 대응)

**커밋**: `46a14d7` (main) · **검증**: tsc 0 · jest 200/200 (+2) · **preview OTA 발행**

- **방식**: 지시의 "export 또는 icons로 이동" 중 **icons.tsx로 이동** 채택 — edit.tsx가 SocialAuthButtons를 import하면 expo-apple-authentication·소셜 인증 훅까지 끌려와 테스트/의존 오염. icons.tsx의 기존 `IconApple`/`IconGoogleG` 스트로크판을 **이름 그대로 공식 로고로 교체**(사용처 코드 무변): 애플 = 필드(채움) 모노크롬 마크(기본 ink), 구글 = 공식 4색 G(브랜드 고정색 — color prop 무시)
- SocialAuthButtons의 로컬 GoogleG 삭제 → 공용 `IconGoogleG` 사용(**SSOT** — 로그인 버튼과 연동 행이 같은 마크, 경로/색 완전 동일). 로그인 화면 렌더 결과 무변(애플은 OS 네이티브 버튼 그대로)
- 스트로크판 다른 사용처 전수: edit.tsx 연동 행뿐 → 제거(교체)로 처리. 중립 폴백(IconProfile) 무변
- 테스트 +2 (brandIcons.test.tsx): 구글 4색 fill을 RNSVG payload로 고정 잠금 + 애플 채움·스트로크 회귀 방지
- **preview OTA**: Android runtime `cbbec117` = build1 일치(스왑 후 복원·트리 클린). 프로덕션은 다음 묶음 후보

## [P-033] KB-205 정정 — 주문카드 종교·식이 문단 제거 (2026-07-21)

**커밋**: `e24c664` (main) · **검증**: tsc 0 · jest 198/198 (−1) · 다음 OTA 편승(단독 발행 없음)

- `orderCard.ts`의 lifestyleLinesKo(③) 조립 로직 + `order.tsx` 렌더 + 테스트 2케이스 제거. 평면 81종 정본 주석으로 대체 — 재발 방지
- "미지 코드만이면 null" 테스트는 유지(입력 방어 잠금) — 명칭만 ③ 담당 표현 제거
- P-030 보고의 "BE 논의 필요(종교·식이 코드 왕복)" 질의는 **철회** — 평면 81종 확정으로 논의 불요

## [P-032] KB-207 UI 폴리시 2 — kinetics 마이크로 인터랙션 (2026-07-21)

**커밋**: `d692bcb` (main) · **검증**: tsc 0 · jest 199/199 (+1) · **preview OTA 발행**

9종 처리 내역 (전부 JS-only·reanimated 기설치분·절제 톤 — 바운스는 탭 모멘텀 있는 곳만 `spring.pop`):
- ① **칩 팝인/아웃**: IngredientFilter 요약 칩 ZoomIn.springify/ZoomOut — scale 변형이라 **P-026 고정높이 36 불변**(기존 잠금 테스트 통과로 확인)
- ② **Bookmark Toggle**: StickyHeader 기존 timing 3단 시퀀스 → 움츠림+스프링 팝으로 통일(상세 헤더가 유일한 토글 아이콘 — 목록엔 토글 없음, 저장 목록은 삭제 행)
- ③ **Tab Pill Glide**: 스캔 세그먼트 — 라벨 폭이 i18n 가변이라 onLayout 실측 → 흰 인디케이터 스프링 글라이드. 첫 배치는 무애니메이션(등장 시 글라이드 금지)
- ④ **Error Shake**: 공용 `useShake` 훅(감쇠 진동·재트리거) — 로그인 실패(SocialAuthButtons, phase 복귀 기준이라 같은 에러 재시도도 재트리거) + 온보딩 제출 실패
- ⑤ **Success Check**: `SuccessCheck` 컴포넌트 — SVG strokeDashoffset 체크 드로잉. 온보딩 제출 성공 시 0.9s 오버레이 후 홈 진입(reduced-motion이면 정적 체크)
- ⑥ **Stagger Entrance**: 스캔 결과 리스트 FadeInDown 60ms 스태거(지연 상한 8행 — 긴 메뉴판에서 총 대기 무한 증가 방지). reduced-motion 시 전역 config가 생략
- ⑦ **Step Progress**: TopBar 세그먼트 — 새로 채워지는 노드만 팝
- ⑧ **Shimmer**: **기구현 확인** — P-009 때 sweep 하이라이트가 이미 구현돼 있어 무변(위치: Skeleton.tsx Shimmer). P-031의 ReducedMotionConfig로 reduce-motion 시 루프도 정지
- ⑨ **Quantity Stepper 값 팝**: 주문카드(P-030에서 이관) — bump 래퍼로 값 변경 시 소량 바운스 펄스
- 테스트: +1(orderStepper — bump 클램프 1~5·경계 비활성·문장 갱신). 무회귀 = 기존 전 스위트 통과(칩 고정높이·스캔 기본 뷰·CTA 등). 테스트 reanimated mock 표면 2차 보강(9본 공통)
- **preview OTA**: Android runtime `cbbec117` = build1(스왑 후 복원·트리 클린). ⚠️ 스캔 화면 효과(③⑥)는 OTA 스왑이 scan.tsx를 pre-P022로 되돌리므로 **build1 preview에는 미포함** — 빌드2부터 보임(기존 스캔 스왑 절차의 알려진 트레이드오프, P-012 이후 동일)

## [P-031] KB-206 UI 폴리시 1 — 대비·큰글씨·press·시트 스프링·reduced-motion (2026-07-21)

**커밋**: `dc7ddd7` (main) · **검증**: tsc 0 · jest 198/198 (+6) · **preview OTA 발행**

- **A1 대비 — 방식 명시(요청 항목)**: ① `ink3` 토큰을 #B0A395(2.28:1)→**#837363**으로 어둡게 — 사용처 ~40곳 일괄 해결. white(카드 — 소형 텍스트 대부분의 실배경) **4.57:1 ✓**, surface 4.23:1(목표 근접). **전수 후 판단 근거**: 4.5-on-surface까지 어둡게 하면 ink2(#7C6B5E)와 시각 구분이 소멸해 3단 위계가 죽음 — 여기서 절충, 위계는 크기·굵기 담당. ② 소형 주황: 신규 `primaryText`(#c44a08, white 4.85:1) 토큰 추가, **fontSize ≤14 primary 텍스트 19곳 전환**(링크·라벨). 15px+ 디스플레이 숫자·owner/order 카드 34px 메뉴명은 brand primary 유지(대형 텍스트 3:1 충족). ③ **위험도 4색 불변** — 테스트로 hex 고정 잠금
- **A2**: `Txt`에 maxFontSizeMultiplier **1.3 기본**(명시 prop 우선) — 전 화면이 Txt 경유라 한 곳으로 관통 · **A3**: 스캔 unmatched 안내 numberOfLines 1→2
- **B4**: 공용 `Btn` — onPressIn **즉시** scale 0.97 damped 스프링(AnimatedPressable, 릴리스 대기 금지). 프리셋은 신규 `src/lib/motion.ts` 일원화(press/sheet/move — apple-design 실측치 직역)
- **B5**: AuthGateSheet 시트 `SlideInDown.springify()`(바운스 0, backdrop·퇴장은 Modal fade 유지) · Snackbar 등장 스프링+퇴장 페이드 · StickyHeader 숨김/복귀 timing→withSpring. **판단**: LanguagePicker는 OS 네이티브 slide 유지 — 플랫폼 표준 전환이지 우리가 쓴 timing이 아님(재구현 시 퇴장 애니메이션 상실 리스크만 생김)
- **B6**: 루트 `ReducedMotionConfig(System)` — reduce-motion 시 reanimated 전역 비활성, Modal fade 경로만 남아 크로스페이드 폴백이 자연 성립
- 테스트 +6 (uiPolish.test.tsx): **WCAG 대비 수식을 테스트에 내장**(ink3·primaryText·ink2 ≥4.5 on card — 토큰 되돌리면 jest가 잡음)·위험도 4색 hex 불변·Txt 1.3 기본/override·press 스프링 즉발. 기존 테스트 9본의 reanimated mock 표면 보강(withSpring·entering 체인 빌더)
- **preview OTA**: Android runtime `cbbec117` = build1 일치(스왑 후 복원·트리 클린). iOS 그룹은 상시 발생하는 무해 orphan

## [P-030] KB-205 음식 상세 주문+고지 원카드 (2026-07-21)

**커밋**: `0d7aa99` (main) · **검증**: tsc 0 · jest 192/192 (+16) · **preview OTA 발행** (JS-only 확인)

- ① CTA(상세 최하단): `order.cta` — **개인화 판정** safe/caution만 노출, danger·unable·게스트 미노출. 빈 프로필은 false-safe 강등으로 safe→caution이라 CTA 유지(고지 없는 순수 주문 카드로 이어짐 — 테스트로 잠금)
- ② `order.tsx`: owner.tsx 동일 패밀리(닫기 X·중앙 큰 글자·하단 done 버튼, done은 기존 `owner.done` 재사용). 메뉴명 primary 하이라이트 + 수량 스테퍼 1~5(텍스트 ±, 이모지·신규 에셋 없음), 상세·프로필 **캐시 재사용 — BE 호출 0**
- ③ 문장 조립은 `orderCard.ts` 순수 함수: ko 라벨 `i18n.getFixedT('ko')` **고정**(UI 언어 무관 — place=ko 헌법), 을/를은 한글 종성 산술 분기(비한글 폴백 '을(를)'), 기피 6개 초과 "외 n개" 접기, **기피 0개 → ②③ 생략**(순수 주문 카드)
- ④ i18n `order.cta`/`order.caption` ×10 (본문은 i18n 아님 — ko 데이터)
- 테스트 +16: 을/를 3 · ko 고정 2 · 주문 문장 2 · 고지 생략/접기 4 · lifestyle 2 · CTA 노출 조건 5케이스 + 카드 문단 유무 2 (스위트 orderCard.test.ts / orderCta.test.tsx)
- **preview OTA**: Android `019f829f-3778-7747` (runtime `cbbec117` = build1, 스왑 후 복원·트리 클린). P-025의 네이티브 추가와 무관 — 이 건 자체는 JS-only(스왑이 package.json·scan.tsx를 되돌려 build1 호환 유지)
- **특이·BE 질의**: 기획 ③(종교·식이 한 줄)은 구현했으나 **현행 와이어(avoidanceSubstanceCodes)가 재료 81종만 왕복이라 live 프로필에선 도달 불가** — 온보딩이 수집한 religion:/diet: 코드는 전송 시 드롭됨(KB-75 체계). 서버가 종교·식이 코드를 프로필에 실어주면 FE는 수정 없이 자동 활성. BE 논의 필요: 종교·식이 코드 프로필 왕복 지원 여부

## [P-025] KB-202 스캔 캡처 WYSIWYG — 과다 캡처 크롭 (2026-07-21)

**커밋**: `4bf857a` (main) · **검증**: tsc 0 · jest 176/176 (+5) · ⚠️ **OTA 아님 — 재빌드 필요**

- ① `coverCropRect()` 순수 함수 (src/lib/scan/coverCrop.ts): cover 표시 역산 — 사진/뷰 비율 비교로 잘린 축만 중앙 크롭 rect 산출. 비율 이미 일치하면 null(무의미한 JPEG 재인코딩 생략), 무효 치수도 null(크롭 생략이 안전 폴백)
- ② capture 경로 배선: CameraView `onLayout` 실측 → `takePictureAsync` 직후 `expo-image-manipulator`(SDK 56 새 API manipulate→renderAsync→saveAsync)로 크롭. **업로드·OCR·결과 "원본 사진" 전부 크롭본** — 미리보기 밖은 어디에도 안 감. 원본(과다 캡처)은 크롭 직후 삭제(KB-137 캐시 위생 연장). 오버레이 마커는 preview 정규화 공간이라 크롭 후 정합 유지(무변). iOS·안드 공통. 갤러리 경로는 대상 아님(미리보기 없던 이미지)
- ③ expo-image-manipulator는 **지연 require** — expo-sensors(P-022) 크래시 선례와 동일 사유. 미탑재 빌드(현행 전 빌드)에선 크롭만 생략하고 스캔 정상 → 재빌드 전 OTA에 섞여도 무해
- 테스트 +5 (coverCrop.test.ts): 좌우 크롭(4:3 센서×세로 폰) / 상하 크롭 / 비율 일치 null / 무효 입력 null / rect 경계 불변식
- ⚠️ **발행: OTA 불가** — 네이티브 모듈 추가. **expo-sensors(P-022)와 같은 빌드(안드 빌드2/차기 iOS 빌드)에 배치** 요청. 빌드 발주는 검토 게이트 판단에 맡김. package.json 변경으로 fingerprint 재회전 — 기존 OTA 스왑 절차(package.json+scan.tsx 복원)가 그대로 커버함

## [P-028] KB-174 후속 — 검색 화면 오프라인 전체화면 J4 (2026-07-21)

**커밋**: `f8ea807` (main) · **검증**: tsc 0 · jest 171/171 (+4) · **preview OTA 발행**

- ① 오프라인 판정: 프롬프트의 "useFoods().isError 또는 기존 netinfo 훅"에서 — netinfo 훅은 없고(P-007 때 리빌드 회피로 미도입), `useFoods()`는 검색 empty의 인기 블록이 MOCK이라 에러가 안 섬. **음식탭과 같은 live 쿼리 `useInfiniteFoods()`를 프로브로 재사용**(같은 쿼리 키 → 캐시 공유, 추가 트래픽 없음) — P-027과 동일 메커니즘·동일 `classifyQueryError` 분류
- ② 오프라인(NETWORK→J4)일 때만 empty(최근+인기)를 전체화면 `QueryErrorBlock`으로 대체. **서버 5xx(J3)는 empty 유지** — 최근 검색·인기는 로컬/mock이라 서버 장애가 가릴 이유가 없음 (홈·음식탭과 다른 점: 거기는 화면 콘텐츠 자체가 그 쿼리)
- ③ 온라인 동작 무변(인기 6종·최근 검색 로직 그대로 — "변경 금지" 준수, 테스트로 잠금)
- ④ 재량 항목 채택: 제출 검색 에러(L142)의 StateBlock → `QueryErrorBlock` 교체. 오프라인 중 검색 제출 시에도 J4가 뜨고, 홈·음식탭과 톤 통일 — 코드도 더 짧음
- 테스트 +4 (`searchOffline.test.tsx`): 오프라인→J4+최근/인기 미렌더 / 온라인→무변 / 프로브 500→empty 유지 / 제출 검색 NETWORK→J4
- **preview OTA**: Android `019f8292-e2ff-7aa0` (runtime `cbbec117` = build1 일치, build1 호환 스왑 후 복원·트리 클린). production은 검토 후 묶음

## [P-029] KB-203 프로필 계정 연동 — provider 표시 (2026-07-21)

**커밋**: `2633291` (main) · **검증**: tsc 0 · jest 167/167 (+2) · **preview OTA 발행**

- 실측: `MyProfileResponse.provider` required 배포 확인. Wire→adaptProfile→User 매핑(구서버 방어 옵셔널). 기존 email 행은 계약에 없어 항상 undefined 표시였음 — provider 행으로 교체
- edit.tsx 연동 행: APPLE→`IconApple` / GOOGLE→`IconGoogleG`(**로그인 화면 SVG 재사용** — 신규 에셋 없음, 헌법 준수) / 미지원·누락→IconProfile+중립 폴백("소셜 계정"). `providerLabelKey()` 순수 함수 — 빈 값 금지
- i18n `linkedVia/linkedApple/linkedGoogle/linkedSocial` ×10
- 테스트 +2: providerLabelKey 4케이스 + adaptProfile provider 매핑
- **preview OTA**: Android `019f828b-21bf-7230` (runtime `cbbec117` = build1 일치, build1 호환 스왑 후 복원). production은 검토 통과 후 묶음 발행 권장(긴급 아님). 예진: 프로필>수정 최하단 — 가입 수단대로 Apple/Google 표시

## [P-023] 🔴긴급 KB-199 홈 lang 파라미터 — /home?lang= (2026-07-21)

**커밋**: `005ed31` (main) · **검증**: tsc 0 · jest 165/165 · **preview + production OTA 발행**

- BE가 lang을 모든 대상 API에서 **필수 승격**(7/20 밤) — /home만 미전송이라 홈 조회가 400날 상태였음. `fetchHome()`을 `/home?lang=${apiLang()}`로(/foods 동일 패턴), apiLang import 추가. bootGate 프리페치 키는 이미 `['home', lang]`이라 정합
- 표시 언어를 파라미터가 담당 → **Q-13 언어 전환 잔상·게스트/미설정 영어도 동시 해결**(회원 DB appLanguage 기반 → 요청 언어 기반). appLanguage PATCH(선호 저장) 무변
- 테스트 +1: /home 요청 URL에 lang 포함 잠금
- **발행 (최우선, 양 채널)**:
  - preview(공기계): Android update `019f827f-1b3d-7727` (runtime `cbbec117` = build1 일치)
  - **production(테플)**: iOS update `019f8283-3088-74f7` (runtime `8d0d5504` = 테플 빌드 일치). [대시보드](https://expo.dev/accounts/rocher/projects/kbap/updates/bc302411-001b-4f8a-b3a0-7d927cdad217)
  - 프로덕션 스왑 = `_photostyle` gitignore 제거 + app.json·splash·package.json·scan.tsx를 pre-P018/P022(`0e9f884`/`eecbe52~1`)로 (fingerprint 8d0d5504 매칭 검증 후) → 발행 → 전량 복원
- 테스터: 앱 완전 종료 후 재실행 2번 → 홈 정상 로드 + 언어 전환 즉시 반영

## [P-027] KB-174 후속 — 음식 탭 오프라인 전체화면 에러 (2026-07-20)

**커밋**: `a762280` (main) · **검증**: tsc 0 · jest 164/164 · **preview OTA 발행(build1 호환)**

- Q-08 실기: 음식 탭 오프라인 시 헤더(제목·부제·검색바)가 그대로 뜨고 그 아래 오프라인 에러 겹침. 원인: FlatList가 `ListHeaderComponent`(Header)를 항상 렌더 + `ListEmptyComponent`로 QueryErrorBlock을 아래 깖
- 수정: `ListHeaderComponent={isError ? null : Header}` — 에러/오프라인 시 헤더 미렌더 → QueryErrorBlock 전체화면(홈 탭과 톤 통일). 로딩(스켈레톤)·정상·빈 상태는 헤더 유지, StickyHeader(브랜드 크롬)는 홈 탭과 동일 유지
- 테스트 +1: 에러→헤더(food.title·searchPlaceholder) 미렌더 / 정상→헤더 렌더 잠금
- **preview OTA**: Android update `019f7f49-e9ad-74bf` (runtime `cbbec117` = build1 일치, build1 호환 스왑 후 복원). 예진: 강제중지→2회 실행 후 음식 탭 기내모드 = 전체화면 오프라인(헤더·검색바 미노출)

## [P-026] KB-178 재수정 — 기피 재료 요약 줄 고정 높이 (2026-07-20)

**커밋**: `0ea62e6` (main) · **검증**: tsc 0 · jest 163/163 · **preview OTA 발행(build1 호환)**

- P-011 보고에서 유보했던 "0↔1 전환 1회 높이 변화 — placeholder 판단 요망"이 실기(Q-11 2·6)로 확정됨. 원인: activeCard 안 칩 ScrollView가 selected>0일 때만 렌더 → 첫 선택 시 칩영역+gap이 새로 생기며 아래 목록 1회 밀어냄
- 수정: 칩 영역을 **항상 고정 높이(36 = rmChip 높이) View**로 — 빈 상태엔 placeholder(`restrictionsEdit.chipPlaceholder` ×10), 선택 시 기존 horizontal ScrollView. 빈↔선택 높이 불변 → 목록 밀림 0. **공용 IngredientFilter라 온보딩·프로필 동시 반영**
- 테스트: 0건 placeholder+ScrollView 미렌더 / 0↔1 칩영역 높이 36 불변 잠금(밀림 0 근거)
- **preview OTA**: Android update `019f7f46-3946-7b5a` (runtime `cbbec117` = build1 일치). P-022(expo-sensors) 이후라 build1 호환 스왑(app.json·splash·package.json·scan.tsx pre-P022) 후 복원. 예진: 강제중지→2회 실행 후 첫 재료 선택 시 목록 밀림 없는지

## [P-024] KB-197 재수정 — 언어 선택 회색 = elevation (2026-07-20)

**커밋**: `567d461` (main) · **검증**: tsc 0 · jest 162/162 · **preview OTA 발행(build1 호환)**

### 수정 (내 P-021 오진 정정 — 지적 정확)
- P-021의 android_ripple은 **원인 오진**이었음. 실제 회색 padding = LanguagePicker `row`의 **`...shadow.sh1`(elevation:1) + `overflow:'hidden'`(P-021이 추가) 공존** — 안드에서 elevation 그림자가 클립되며 padding을 회색으로 채우는 조합. (증거: 같은 파일 closeBtn은 sh1 있지만 overflow 없어 멀쩡)
- `row`에서 `...shadow.sh1` 제거 — borderWidth 1(C.hair)로 이미 구분, sh1 opacity 0.04라 시각 손실 미미. overflow·android_ripple은 유지(무해·정상), rowOn 무변

### ⚠️ preview OTA — P-022 상호작용 (중요)
- **Android update `019f7f2f-05a0-72f9`** (runtime `cbbec117` = 공기계 build1 일치). [대시보드](https://expo.dev/accounts/rocher/projects/kbap/updates/a2b6e8e2-e555-4fb4-b7c6-201597403744)
- **함정 발견·해결**: P-022가 expo-sensors(네이티브)를 트리에 추가한 뒤라, HEAD를 그냥 OTA하면 ① fingerprint 불일치로 build1 미도달(orphan) ② 번들의 scan.tsx가 DeviceMotion(build1엔 없는 네이티브) 호출 → **Android 스캔 크래시**. → 발행 순간 app.json·splash·**package.json**을 `0e9f884`(preview 빌드 시점)로 + **scan.tsx를 `eecbe52~1`(P-022 직전, 센서 없음)로** 스왑해 build1 안전 번들 생성(JS엔 P-024만 실림), 후 전량 복원. (첫 발행분 runtime `0251c0a0`은 orphan — 무해, 폐기)
- FE 세션 메모리에 절차 기록. **build2(P-018·P-022 포함) 나오면 이 스왑 전부 폐기**
- 예진: 강제중지→2회 실행 후 언어 선택 화면 재확인 (padding 회색 없음)

## [P-022] KB-198 Android 스캔 가로 감지 — expo-sensors 포팅 (2026-07-20)

**커밋**: `eecbe52` (main) · **검증**: tsc 0 · jest 162/162 (+5) · ⚠️ **OTA 아님 — 재빌드 필요**

### 작업 결과
1. **expo-sensors 추가**(~56.0.6, autolink) — app.json plugins에 `["expo-sensors", {"motionPermission": false}]`: Android DeviceMotion(가속도)은 런타임 권한 불요, iOS는 Platform 가드로 미사용이라 불필요한 NSMotionUsageDescription 선언을 끔(App Store 리뷰 클린). **네이티브 모듈이라 다음 빌드에만 실림**
2. **`deviceOrientation.ts` 순수 함수**: 중력 벡터 → portrait/landscapeLeft/landscapeRight. **임계각 35° + flat(화면 위/아래 향함) 가드로 45° 근처 떨림 방지**. 좌표계 부호를 KB-141 오버레이 rotate와 일치(landscapeLeft→+90°)
3. **scan.tsx**: Android 전용 useEffect로 `DeviceMotion.addListener`(200ms) → 기존 `camOrientation` state 공급(두 소스 같은 state, 오버레이·가드 로직 전부 무변). iOS는 expo-camera 콜백 현행 유지 — **안드만 추가(회귀 최소화)**, 언마운트 시 remove
4. 테스트 +5: 방향 판정 경계(세워듦/임계 미만·이상/flat/거꾸로)

### 발행 (지시대로 명시)
**OTA 불가** — 네이티브 모듈이라 preview OTA로 못 감(P-021과 발행 경로 다름). **다음 Android 빌드(빌드2) + 다음 iOS 빌드(빌드7과 함께)에 포함**. 실기기 확인은 재빌드 후: 스캔 중 안드 기기를 가로로 돌리면 세로 유도 오버레이(iOS와 동일)·세로 복귀 시 사라짐

### 🔧 후속 보강 — dev 빌드 크래시 수정 (`1795d3b`)
실기(iOS dev 빌드, 로컬 metro) 확인 시 **앱 전체 크래시**: `Cannot find native module 'ExponentPedometer'`. 원인 — scan.tsx 최상단 `import { DeviceMotion } from 'expo-sensors'`가 파일 로드 시점에 네이티브 모듈을 즉시 require. **현 dev 빌드는 expo-sensors 추가(P-022) 전에 빌드돼 그 모듈이 없음** → import 자체가 iOS에서도 실행돼(라우트 파일이라) 앱 전체가 죽음. Android 가드는 사용부에만 있어 import는 못 막았음.
- **수정**: 최상단 import 제거 → Android 가드 **안에서 try-require + catch 폴백**. iOS는 expo-sensors 무접촉(현 dev 빌드 **metro 리로드만으로 즉시 정상**, 재빌드 불필요), Android 네이티브 미탑재(재빌드 전)면 가로 힌트만 조용히 비활성·스캔 정상.
- 교훈: 특정 플랫폼 전용 네이티브 모듈은 최상단 import 금지 — 해당 플랫폼 코드 경로 안에서 lazy require해야 다른 플랫폼/구버전 빌드를 안 깨뜨림. tsc 0 · jest 162/162.
- **정리**: iOS 개발은 재빌드 불필요(리로드). Android 가로 감지 실동작만 빌드2 대기(변함없음).

## [P-021] KB-197 Android UI 정리 — 온보딩 CTA 짤림 + 언어 리플 (2026-07-20)

**커밋**: `81deecf` (main) · **검증**: tsc 0 · jest 157/157 · **preview OTA 발행**

### 원인 정정 + 수정
1. **온보딩 제출 버튼 짤림**: 지시가 가리킨 stickyFoot(restrictions 전용)은 이미 insets 적용 상태였고, **실제 짤린 "제출" 버튼은 마지막 spice 스텝의 in-scroll `foot`**(marginTop:auto, body paddingBottom 28뿐 — 내비바 클리어런스 전무). 안드 edge-to-edge에서 insets.bottom 과소보고(0) 기기 대비 `bottomInset = android ? Math.max(insets.bottom, 48) : insets.bottom` — in-scroll foot 스텝은 body를 `28+bottomInset`, restrictions stickyFoot는 `bottomInset+14`. iOS는 실측값 그대로(무회귀)
2. **언어 선택 리플**: LanguagePicker row에 브랜드 `android_ripple({color: rgba(226,88,12,0.12)})` 명시(회색 기본 리플이 padding까지 번지던 것 대체) + row `overflow:'hidden'`로 라운드 코너 클립. 선택 상태는 기존 테두리+틴트(rowOn) 무변. **공용 컴포넌트라 프로필 수정·프로필 탭 동시 반영**. iOS는 android_ripple 무시

### preview OTA 발행
- **Android update `019f7e9c-bb69-76cd`** (runtime `cbbec117` = 공기계 preview 빌드 일치). [대시보드](https://expo.dev/accounts/rocher/projects/kbap/updates/cba0600c-438b-4a41-a4df-19e832f0abf9)
- P-020과 동일하게 발행 순간 P-018 이전 에셋(app.json·splash) `096a954~1` 스왑으로 fingerprint 매칭, 후 복원
- 스타일-only라 신규 테스트 없음(기존 렌더 스위트가 회귀 커버). **예진**: 강제중지→2회 실행 후 온보딩 마지막 스텝 제출 버튼·언어 선택 화면 재확인

## [P-020] KB-196 Android 구글 로그인 accessToken 누락 (2026-07-20)

**커밋**: `dc984a3` (main) · **검증**: tsc 0 · jest 157/157 (+1) · **preview OTA 발행 완료**

### 수정
- 원인 실측 그대로: `signInWithGoogle`이 `GoogleAuthProvider.credential(idToken)`로 idToken만 전달 — RNFB Android 네이티브는 accessToken도 요구("accessToken cannot be empty"). iOS는 idToken만으로 통과해 iOS만 동작해왔음
- `GoogleSignin.getTokens()`로 accessToken 받아 `credential(idToken, accessToken)` 2인자 전달. 취소·에러 분기 무변, iOS 무회귀(병행 무해)
- 테스트 +1: 네이티브 모킹으로 getTokens 호출 + credential 2인자(idToken, accessToken) 잠금

### preview OTA 발행 (지시대로)
- **Android update `019f7e7b-929e-7b31`** (branch preview, runtime `cbbec117` = 공기계 preview 빌드와 **정확히 일치** — 도달 확인). [대시보드](https://expo.dev/accounts/rocher/projects/kbap/updates/0b6e51b9-903d-4945-96b5-dec9b478304a)
- ⚠️ **발행 순간 P-018 이전 에셋(app.json·splash-icon.png)을 `096a954~1`로 스왑**: 공기계 preview 빌드(00ba0336)가 P-018 네이티브 변경 전에 만들어져 fingerprint 매칭 필요(스킬 3-1 선례 — iOS `_photostyle` 스왑과 별개). JS 번들은 에셋 무관이라 셔먼 동일, 발행 후 즉시 복원
- **예진**: 공기계 앱 강제중지 → 2회 실행 후 구글 재로그인 (계정 선택→온보딩/홈)

## [P-019] KB-195 온보딩 맵기 스킵 = -1 명시 전송 (2026-07-20)

**커밋**: `8135d3e` (main) · **검증**: tsc 0 · jest 156/156 (+1 test) · **JS-only — OTA 가능**

- 재실측 확인: `spicinessPreference` required 승격 ✓ — P-003의 전환 예약 주석대로 스킵(UNSET) 시 `-1` 명시 전송으로 전환(생략 시 가입 400). KB-150 센티널 정책·로컬 보관 무변
- **겸사 확인**: `profileImageUrl`도 required로 승격돼 있음 — P-016이 이미 항상 값 전송이라 추가 대응 불필요(기록만)
- 테스트: 스킵→-1 / 설정→실값 잠금

## [P-018] KB-194 스플래시 개선 — 태그라인 제거 + 부트 게이팅 (2026-07-20)

**커밋**: `096a954` (main) · **검증**: tsc 0 · jest 155/155 (31 suites, +4 tests)

### 에셋 (네이티브 — 빌드 7 확인 항목)
- 원본 소스 부재 → png 직접 처리: 알파 밴드 분석으로 태그라인 스트립(838..857, h20) 식별·제거, 콘텐츠 bbox 크롭+여백 정리(900×1200 → 391×535, 패딩 24·하단은 태그라인 직전 캡)
- **미리보기 첨부**: `dropbox/yj/splash-v2-preview.png` (주황 배경 합성본, 푸시됨)
- app.json `imageWidth` 280→230 원복 — `_comment`가 예약해둔 그 원복, 주석 갱신

### 부트 게이팅 (JS — OTA 검증 가능)
- `gateSplash`: **min 1200ms**(조기 완료여도 채움 — 반짝임 소멸) / **cap 4000ms**(초과 시 진입 — KB-174 스켈레톤/J4 인계). 프리페치 reject=settle 취급 — **오프라인이 스플래시를 붙잡지 않음**
- `prefetchBootData`: 홈·음식 목록(+세션 시 me). **실키 일치 보장** — 훅 queryFn 3개를 fetchHome/fetchFoodsPage/fetchMe로 추출 공유하고, `resolveInitialLang()`로 저장 언어를 LocaleProvider 마운트 전에 선적용(캐시 키·Accept-Language 정합 — 키 불일치로 프리페치가 무의미해지는 함정 차단)
- 게이트 유닛 테스트 4종: min 전 hide 절대 없음 / min~cap settle 즉시 / cap 강제 / reject 무지연

### ⚠️ 다음 OTA 주의
이 커밋은 네이티브(에셋·app.json)+JS 혼합. **빌드 7 전에 게이팅만 OTA로 내려면 발행 순간 app.json·splash-icon.png를 직전 커밋 상태로 스왑**(기존 _photostyle 스왑에 추가 — 스킬 3-1 선례). 빌드 7 재베이스라인 후 스왑 전부 소멸.

### 실기기 확인 포인트 (DoD)
(OTA 후) 빠른 네트워크: ~1.2초 균일 스플래시 후 진입 / 기내모드: 최대 4초 후 스켈레톤·J4 / (빌드 7) 태그라인 없는 스플래시·230 크기

## [P-017] KB-176 Android 첫 빌드 — 공기계 스모크 apk (2026-07-20)

**빌드 성공** (첫 빌드 ~26분, 콜드 캐시) · **코드 무변경** (PROGRESS 기록만 `0e9f884`)

### 예진 설치·후속 작업
- **설치**: [빌드 페이지(QR)](https://expo.dev/accounts/rocher/projects/kbap/builds/00ba0336-8fcf-45d0-b6de-505e6c4bc64b) — 공기계 브라우저로 열어 Install / [apk 직링크](https://expo.dev/artifacts/eas/i7TZ7TsjquTfoCclaxe6SFybG-RFcPk-b98pQR-yoPM.apk) (184MB)
- **⚠️ 구글 로그인 사전 작업**: EAS keystore SHA-1을 Firebase 콘솔(Android 앱 com.rocher.kbap)에 등록해야 동작 —
  `SHA1: D2:F7:2B:1A:F4:5C:C4:23:13:18:CB:98:D4:65:7D:6D:F8:2A:A3:35`
  (SHA256: 39:45:A3:20:B4:F5:72:D4:51:34:C4:72:F4:C3:11:71:EA:6F:F4:A8:D2:32:06:77:A3:63:78:01:53:E4:35:94 — App Check용 대비 병기)
- 애플 로그인 버튼은 Android **미노출이 정상** (appleAvailable = iOS 전용, 기구현 확인)

### 판단·특이사항
1. **eas.json 무변경**: preview(distribution: internal)는 Android 기본 산출물이 apk — buildType 명시는 동작 동일하면서 **eas.json이 fingerprint 소스라 다음 iOS OTA에 eas.json 스왑 부담 추가**(스킬 3-1 함정 계열) → 생략 판단, 산출물 `.apk` 확장자로 검증 완료
2. 사전 점검 통과: google-services.json 존재(Firebase Android 기등록) — 즉시 보고 사유 없음
3. keystore EAS 자동 생성(Build Credentials 3TdEsf5DKE) · preview 채널/브랜치 자동 생성됨
4. SHA-1 추출은 `eas credentials`가 인터랙티브라 **apk v2 서명 블록 직접 파싱**으로 확보(비대화형)

## [OPS] 프로덕션 OTA 발행 — P-011~P-016 묶음 테플 배포 (2026-07-20)

**요청 경로**: 예진 직접 지시. **결과**: iOS 정상 발행, runtime `8d0d5504`(설치 빌드 일치), update `019f7de6-afc5`. [대시보드](https://expo.dev/accounts/rocher/projects/kbap/updates/7a6cb4e2-fc8a-4882-8897-eba653416b06)

- 포함: `79ace75`(P-011 선택 UI B안) · `d134d1f`(P-012 상세 가격 param) · `9e67bf4`(P-013 사진 삭제) · `3ebbc03`(P-014 null 교체) · `60c043f`(P-015 언어 전환) · `847f3d3`(P-016 기본 path 최종) — 전부 JS-only 확인(스왑 시 fingerprint 일치)
- P-014의 서버 게이트는 P-016(기본 path = 항상 유효 값)으로 소멸 — 서버 배포와 무관하게 안전. 스왑 절차 적용, 트리 클린. 적용법: 앱 완전 종료 후 재실행 2번
- 확인 대기 핵심: 81종 선택 밀림 없음·CTA 상시 / 스캔→상세 ₩ / 사진 삭제→기본 사진(서버 배포 후) / 언어 전환 잔상 0·맵기 라벨 언어 추종

## [P-016] KB-149 최종 — 프로필 사진 기본 path 전송 (2026-07-20)

**커밋**: `847f3d3` (main) · **검증**: tsc 0 · jest 151/151 (30 suites)

- **3차 최종 확정 반영** (''→null→기본 path): `PROFILE_IMAGE_DEFAULT_PATH = 'images/default/profile/profile-default-512.png'` — CLEAR 상수명 유지(화면 무변), 경위 주석. 타입 `string` 원복(null 폐기)
- 온보딩 미선택도 기본 path **명시 전송**(필드 생략 폐기 — 필드는 항상 값)
- 삭제 액션 노출: `isDefaultProfileImage(url)` — 기본 사진=사진 없음 취급(미노출), 커스텀만 노출. 렌더는 서버 URL(기본 포함) 그대로, svg 플레이스홀더는 URL 부재·비-http 방어용 유지
- 착수 시 실측: 스펙 표기 여전히 optional — 서버 "항상 값" 배포 여부는 스펙만으론 확인 불가(진행로그 기록). 코드는 완성 — 서버가 기본 URL을 주기 시작하면 그대로 동작
- 테스트: 삭제·미선택 전송값 기본 path 잠금 / isDefault 판별 5케이스 / 기본 URL 시 삭제 미노출
- ✅ **P-014의 OTA 게이트 소멸**: 기본 path는 항상 유효한 string — 이 커밋부터 서버 무관 OTA 가능. 실기기 확인: 삭제→기본 사진 렌더→재시작 유지 / 온보딩 미선택 가입→기본 사진

## [P-015] KB-187 언어 전환 즉시 반영 + 맵기 라벨 i18n (2026-07-20)

**커밋**: `60c043f` (main) · **검증**: tsc 0 · jest 149/149 (30 suites, +12 tests)

### 잔상 원인 실조사 결과 (지시 요구분)
- placeholderData/keepPreviousData류 **아님** — 코드베이스 미사용 확인
- 실원인: `setLang`이 컨텍스트 리렌더(동기)와 `i18n.changeLanguage`(**비동기**)를 분리 실행 → 완료 전 리렌더 시점엔 쿼리 키가 읽는 `i18n.language`가 아직 이전 언어 → **키 불변 = 재조회 없음**, staleTime 60s 동안 구언어 데이터가 fresh 판정으로 유지(잔상). 완료 후 리렌더가 와야 새 키로 갈아탐 — "한참 뒤/다른 행동 후 정상" 정황과 일치

### 최소 수술
changeLanguage **완료 직후** lang 종속 루트 키(home/foods/food/me)만 1회 invalidate — 활성 쿼리 즉시 재조회, 새 키는 fresh 마운트라 P-009 탭별 스켈레톤이 그대로 노출. 매 탭 진입 재호출 아님(금지 준수). 비언어 쿼리 무접촉을 테스트로 잠금

### SPICE_SCALE i18n 이전
영어 하드코딩 11개 → `spice.scale.0~10` 키 ×10 로케일(110 문구 번역), 사용처 4파일 5곳 전부 `t()` 래핑(온보딩 spice 스텝·프로필 칩·수정 칩·상세 detail.spice). 스캔 리스트 name 건은 범위 제외 준수(무접촉)

### 테스트 (신규 12건)
전환→4키 무효화+비언어 무접촉 / SPICE_SCALE=키 11개 / 로케일별 패리티 ×10

### 실기기 확인 포인트 (DoD)
언어 변경 직후 홈·음식 탭이 스켈레톤 경유 새 언어로만(이전 언어 잔상 0) / 맵기 설명 전 사용처 언어 추종. ※ JS-only — OTA 가능(스왑)

## [P-014] KB-149 후속 — 기본 프로필 사진 서버 일원화 (2026-07-20)

**커밋**: `3ebbc03` (main) · **검증**: tsc 0 · jest 137/137

### 작업 결과
1. **png 전달 완료**: `dropbox/yj/profile-default/profile-default-512.png` (+원본 svg) — 종한 확인 요청. 사양: 512×512, 배경 단색 `#FDF1EC`(흰 배경 위 앱 아바타 버블 rgba(226,88,12,0.08) 합성값 — 투명 대신 단색 채택: 원형 크롭·어느 배경에서든 동일 룩), 글리프 = 앱 IconProfile 재현(primary `#E2580C`, stroke 2/24 round)
2. **삭제 전송값 null 교체**: `PROFILE_IMAGE_CLEAR` '' → null (상수 한 곳 — P-013 설계 그대로 한 줄), 타입 `string | null`. 생략=유지 보존. **스펙 스키마는 아직 비-nullable 표기** — 진행로그 기록, 종한 배포와 동시 적용 전제
3. **조회 렌더**: 서버 기본 URL은 기존 절대 URL 경로로 그대로 렌더됨. FE svg 플레이스홀더 분기 + 비-http 방어(adaptProfileImageUrl) **유지** — 지시대로 제거 안 함
4. 테스트: 삭제 잠금 ''→null 교체 (PATCH body·화면 mutate·invalidate)

### ⚠️ OTA 게이트 (순서 주의 반영)
**서버가 null 삭제를 배포하기 전 이 커밋이 OTA에 실리면 삭제가 무동작/400 가능** — `3ebbc03`은 **서버 배포 확인 후 OTA 포함**할 것. (P-011~P-013까지는 서버 무관 — 먼저 내보내려면 `d134d1f`까지로 끊어서 발행)

### 실기기 확인 포인트 (서버 배포 후)
삭제 → 기본 사진(서버 png) 렌더 → 재시작 유지 (Q-02 6번) / 미설정 신규 계정도 기본 사진

## [P-013] KB-149 후속 — 프로필 사진 삭제 (2026-07-20)

**커밋**: `9e67bf4` (main) · **검증**: tsc 0 · jest 137/137 (28 suites, +3 tests)

### 착수 게이트 처리 (null vs '' — 미확정 상태에서 진행)
- 재실측: 스펙에 확정 단서 없음(`type: string`, nullable 명시 없음). **잠정 `''` 채택** — 근거: ① 비-nullable string 스키마라 null은 검증 위반 소지 ② imagePath `''` 선례(required string에 '' 허용 확정)와 동일 계열. `PROFILE_IMAGE_CLEAR` 상수 한 곳 격리 — **종한 확정 시 한 줄 교체**
- 서버 반응 실측은 로그인 계정 필요 → 세션에서 불가, **실기기 확인 포인트로 이관**: 삭제 후 사진이 남으면(서버가 '' 무시/거부) null 확정 필요 신호 — 공유 요망
- "생략=유지" 보존: undefined만 미전송, ''는 명시적 삭제 의사

### 작업 결과
1. 수정 화면 아바타 아래 "사진 삭제"(danger 톤 링크) — **설정 상태에서만 노출**, 업로드 중 미노출. 확인 얼럿 없이 즉시 실행 판단(재선택으로 복구 용이 — "과하지 않게" 지시 반영)
2. 삭제 → PATCH `profileImageUrl: ''` → 기존 `['me']` invalidate 재조회 → 탭·수정 화면 플레이스홀더 복귀(재시작 후에도 서버 상태 기준)
3. 온보딩 무변. i18n `editProfile.removePhoto` ×10

### 테스트 (신규 3건)
전송값 '' 잠금+invalidate / 설정 시 노출·탭→mutate / 미설정 시 미노출

### 실기기 확인 포인트 (DoD)
삭제 → 플레이스홀더 복귀 + **앱 재시작 후 유지** (사진이 되살아나면 서버 미반영 — null 확정 요청 신호) / 사진 없으면 액션 없음

## [P-012] KB-179 상세 가격 — 스캔 진입 param 전달 (2026-07-20)

**커밋**: `d134d1f` (main) · **검증**: tsc 0 · jest 134/134 (27 suites, +5 tests)

- **첨부**: 스캔 결과의 상세 이동은 리스트 행·오버레이 마커·사진 전용 항목 전부 `openDish()` 한 곳을 지나감 — 거기서 `scanPriceParam()`으로 양의 정수만 `?price=n` 첨부(null/0/비정수 미첨부). 이동 지점 누락 없음
- **표시**: 상세 metaRow(위험도·스파이스) 아래 한 줄 `₩9,000 · 스캔한 메뉴판 기준`(신규 i18n `detail.scannedPrice` ×10). ₩ 포맷만 — 환율·추정 없음
- **방어**: `parseScanPrice()` — 정수 파싱 실패·음수·0·지수 표기·안전 정수 초과 전부 미표시. 표시 전용, 저장 안 함
- 타 경로(홈·목록·검색·북마크) 진입은 param이 없어 자연 미표시 — 코드 분기 불필요
- 테스트 +5: 첨부 잠금(양의 정수만) + 파싱 게이트(조작값 포함) + 중복 param 첫 값
- 실기기 확인: 스캔→상세 ₩ 표시 / 같은 음식 목록 진입 시 미표시

### 참고 — 보류분 소멸
직전의 "[P-012] 착수 보류(BE price 미배포)" 항목은 본문 개정(param 방향 확정)으로 대체됨.

## [P-011] KB-178 기피 재료 선택 UI — B안 (2026-07-20)

**커밋**: `79ace75` (main) · **검증**: tsc 0 · jest 129/129 (26 suites, +3 tests)

### 작업 결과
1. **요약 1줄 가로 스크롤**: 공용 IngredientFilter 한 곳 수정 → 온보딩·프로필 **양쪽 자동 적용**(복붙 없음). 칩 탭=제거 유지, 선택 순서 유지(기존엔 카탈로그 순 재정렬이었음 — B안의 "줄 끝에 추가"를 위해 삽입 순서로 교체), 새 선택 시 자동 끝 스크롤(제거 시엔 안 움직임 — 위치 감각 유지)
2. **CTA 하단 고정**: 온보딩 restrictions 스텝 foot을 ScrollView 밖 스티키 바로("계속 · n개" + skip), 목록 하단 패딩 130. **프로필 수정 화면은 savebar가 이미 하단 고정**이라 추가 작업 없음
3. **0건 판단(보고 요청분)**: 줄 자체 미표시 유지 — 1→n은 높이 불변이나 **0↔1 전환 1회 높이 변화** 남음. placeholder(빈 줄 고정 높이) 유지가 나은지 실물 체감 후 판단 요청
4. 기존 토큰·칩 스타일 그대로 — 신규 디자인 결정 없음

### 테스트 (신규 3건)
요약 줄 horizontal 1줄 + 선택 순서 잠금 / 0건 미표시 / 온보딩 CTA가 ScrollView 밖(스티키) 잠금 — 스크롤 콘텐츠로 돌아가는 회귀 방지

### 실기기 확인 포인트 (DoD)
81종에서 선택/해제 시 목록 스크롤 위치 유지·밀림 없음 / 새 선택이 요약 줄 끝에 보임 / CTA 항상 노출·동작(온보딩+프로필) / 0↔1 전환 점프 체감되면 placeholder 논의

## [P-012] 착수 보류 — BE price 필드 미배포 (2026-07-20 실측)

착수 게이트 실측: `FoodDetailResponse`에 `price` **없음** (스펙 v3/api-docs 기준). 지시대로 착수 보류 — 상태 ⬜ 유지, BE 배포 후 재착수. (스캔 쪽 `ItemRiskResponse.price` 매핑·₩ 포맷 규칙이 이미 있어 배포되면 소규모 작업)

## [OPS] 프로덕션 OTA 발행 — P-007~P-010 묶음 테플 배포 (2026-07-20)

**요청 경로**: 예진 직접 지시. **결과**: iOS 정상 발행, runtime `8d0d5504`(설치 빌드 일치), update `019f7d8d-f684`. [대시보드](https://expo.dev/accounts/rocher/projects/kbap/updates/7d2266c9-9be8-4ad5-9d4a-811b17d68711)

- 포함: `9d64f43`(P-007 공통 상태 UI) · `b8b8df2`(P-008 401 수정) · `4ba7106`(P-009 탭별 스켈레톤) · `b178e1c`(P-010 탈퇴 잔재 정리) — 전부 JS-only 확인(스왑 시 fingerprint 일치)
- `_photostyle` gitignore 스왑 절차 적용, 트리 클린. 테스터 적용법: 앱 완전 종료 후 재실행 2번
- 확인 대기 핵심: 기내모드 세 탭 J4 / 탭별 스켈레톤 골격·시프트 / 탈퇴→둘러보기 재개 모달 없음

## [P-010] KB-177 탈퇴 후 로컬 잔재 정리 (2026-07-20)

**커밋**: `b178e1c` (main) · **검증**: tsc 0 · jest 126/126 (24 suites, +5 tests)

### 작업 결과
1. **일괄 정리** `clearMemberLocalState()` 신규 — **지운 것**: 온보딩 draft(`kbap.onboardingDraft.v1`) · 맵기 로컬(`kbap.profile.spice.v1`). **유지한 것(기기 귀속 판단)**: introSeen · lang · installed 센티널 · recentSearches(서버 미전송 기기 검색기록 — 계정 비귀속). 세션(토큰·서버캐시)=withdrawBe, Keychain=freshInstall 각자 담당이라 접점 없음 확인 — 이 함수는 AsyncStorage 회원 데이터 계층 전담(계층 분리 주석 + 새 회원 키 등록 지점 명시). 겸사: SPICE_KEY 중복 정의 3곳 → submit.ts export 1곳으로 해소
2. **doWithdraw 배선**: withdrawBe → 로컬 정리 → logOut → /login. (기존 `await import`가 jest 미지원이라 파일 내 관례인 lazy require로 통일)
3. **방어 분기(이중 방어)**: `shouldShowResume()` 순수 함수 추출 — 게스트는 draft/서버 플래그가 어떻든 미노출. 회원은 KB-75 규칙 무변(완료=미노출 / 미완=draft 없어도 노출 / 플래그 없으면 draft 기준)
4. **로그아웃 draft 판단 기록(범위 확장 안 함)**: 타계정 재로그인 시 이전 draft가 이어질 수 있는 창은 "서버 플래그가 없는 로그인 회원"뿐인데 실회원은 항상 플래그가 오므로 실害 제한적 — 방어 분기가 게스트 노출은 이미 차단. 필요해지면 draft에 계정 ID 태깅이 정석

### 테스트 (신규 5건)
정리 키 2종 소거+타 키 미접촉 / 게스트 미노출 3케이스(어떤 잔재 조합이든) / 회원 흐름 무변 4케이스 / 탈퇴 배선 통합(동의→확정→withdrawBe+정리+/login 호출 잠금)

### 실기기 확인 포인트 (DoD)
탈퇴 완주(애플 revoke 포함) → "둘러보기" 게스트 진입 시 재개 모달 **없음** / 정상 회원 온보딩 중단 → 재진입 시 재개 모달 기존대로. ※ JS-only — OTA 가능(스왑)

## [P-009] KB-174 후속 — 탭별 스켈레톤 (실제 레이아웃 미러) (2026-07-20)

**커밋**: `4ba7106` (main) · **검증**: tsc 0 · jest 121/121 (21 suites, +2 tests·1 강화)

- Skeleton.tsx에 화면별 조립 3종: **SkeletonHome**(인사말→배너→히어로→카드 행) / **SkeletonFoodGrid**(2열 카드 ×6 — 폭 48.5%·사진 높이 102 실측 일치) / **SkeletonProfile**(아바타 56 원→랭킹 카드→섹션 행). Shimmer 원자·톤·애니메이션 재사용 — 새 디자인 결정 없음
- **시프트 0**: 패딩(18/4/20)·치수를 각 화면 styles 실측값으로 미러. "화면 레이아웃 변경 시 동반 갱신" 주석 명시
- 범용 SkeletonList는 리스트형(검색·카탈로그)에 유지. 탭별 로딩→전용 스켈레톤 렌더를 테스트로 잠금
- 실기기 확인: 각 탭 첫 로드 골격 일치 + 로딩→렌더 전환 점프 없음 (품질 기준)

## [P-008] KB-174 후속 — useFoods 401 삼키기 제거 (2026-07-20)

**커밋**: `b8b8df2` (main) · **검증**: tsc 0 · jest 119/119 (21 suites, +3 tests)

- 실측 원인 그대로: `useInfiniteFoods`·`useSearchFoods`의 401→빈 목록 특례 제거 (catch가 그것뿐이라 try/catch 통째 삭제). 남는 401(죽은 토큰)은 isError로 서서 P-007 에러 블록이 뜸 — 음식 탭 백지 소멸
- 겸사 grep 재확인: 같은 패턴 다른 훅 없음 (beAuth/client의 401은 refresh 재시도 로직 — 별개 목적)
- 테스트 +3: browse·search 401→isError(빈 목록 위장 없음) + 게스트(무토큰) 정상 목록 무변
- ※ P-009와 같은 세션에서 처리 — P-008(실기기 버그)을 선행. 실기기 확인: 죽은 토큰 상태에서 음식 탭 J3 표시(백지 아님), 게스트 목록 정상

## [P-007] KB-174 공통 상태 UI — false-empty 제거 (2026-07-20)

**커밋**: `9d64f43` (main) · **검증**: tsc 0 · jest 116/116 (20 suites, +6 tests)

### 핵심 발견
J 시리즈 원자(StateBlock·SkeletonList·아이콘·states.* i18n ×10)는 **이미 전부 구현돼 있었음** (states.tsx 카탈로그 화면 포함) — 문제는 화면 미적용. 신규 i18n 문구 0건(패리티 스크립트 검증), 순수 배선 작업으로 수렴.

### 작업 결과
1. **공용 렌더 1개**: `QueryErrorBlock` — J3(err 톤, Try again[+Go back]) / J4(오프라인, Retry) 분기. 화면별 복붙 없음
2. **오프라인 판별(JS-only, 리빌드 회피)**: 공용 클라이언트가 fetch 거부에 붙이는 `NETWORK:` 프리픽스 = J4, 서버 응답 4xx/5xx = J3. 한계(연결 있으나 DNS만 죽는 경우 offline 묶임) 주석 명시 — 정밀 판별은 NetInfo 도입(리빌드) 때
3. **적용**: 홈 — isError 분기 신설(**7/20 장애의 false-empty 그 경로 제거**) · 음식 탭 — 자체 에러 블록을 공용으로 교체(오프라인 분기 획득) · 프로필 — 로딩 스켈레톤+에러 블록(백지 제거) · 음식 상세 — 톤 통일+Go back. Try again은 해당 쿼리 `refetch()`만(전면 invalidate 아님)
4. 빈 상태("아직 스캔이 없어요")는 **성공+0건일 때만** — 테스트로 잠금

### 테스트 (신규 6건 — 탭 풀스크린 렌더)
홈 에러→J3+빈상태 미렌더(false-empty 잠금) / 홈 성공 0건→빈 상태(오버슈트 방지) / NETWORK→J4 / 음식 탭 에러→J3 / 프로필 에러→J3 / 프로필 로딩→스켈레톤

### 실기기 확인 포인트 (DoD 그대로)
기내모드: 세 탭 모두 J4+Retry 동작 / BE 5xx(재현 시): 세 탭 J3+Try again 재요청 / 정상 0건 계정: 기존 빈 상태 유지. ※ JS-only — OTA 가능(스왑)

## [OPS] 프로덕션 OTA 발행 — P-006 테플 배포 (2026-07-20)

**요청 경로**: 예진 직접 지시. **결과**: iOS 정상 발행, runtime `8d0d5504`(설치 빌드 일치), update `019f7d42-ac08`. [대시보드](https://expo.dev/accounts/rocher/projects/kbap/updates/f741357d-00aa-40b5-a571-00c4b8028008)

- 포함: `28e5bae`(P-006 profileImageUrl path 전송 교체) — JS-only 확인(스왑 시 fingerprint 일치)
- `_photostyle` gitignore 스왑 절차 적용, 트리 클린. 테스터 적용법: 앱 완전 종료 후 재실행 2번
- 확인 대기: 프로필 사진 교체→렌더(서버 조합 URL) / "절대 URL이 아님" 로그 여부

## [P-006] KB-149 후속 — profileImageUrl 전송값 path(objectKey) 교체 (2026-07-20)

**커밋**: `28e5bae` (main) · **검증**: tsc 0 · jest 110/110 (19 suites, +3 tests)

### 작업 결과
1. **전송값 교체**: `uploadProfileImage()` 반환 publicUrl → path(objectKey) — P-004에 명시해둔 전환 지점 한 곳 교체로 온보딩·수정 PATCH 동시 반영. purpose는 확정값 `PROFILE_IMAGE` 그대로(P-004 질의 ① 소멸, 질의 ②도 이 확정으로 소멸)
2. **조회 방어**: `adaptProfileImageUrl()` — 비-http 값은 플레이스홀더 처리 + Metro 감지 로그(`조회 profileImageUrl이 절대 URL이 아님 — BE 확인 필요`). 실기기에서 이 로그가 보이면 조회 응답이 path로 오는 것 → BE 확인 요망
3. **온보딩 미리보기 분리**: 제출용 path는 Image 렌더가 불가라 미리보기는 로컬 파일 uri로 — draft 복귀 시 미리보기는 소실돼도 path는 유지·제출됨(한계 명시). 수정 화면은 기존 PATCH→`['me']` invalidate 재조회 경로 그대로 서버 조합 URL 렌더

### 테스트 (신규/갱신 3건 + 기존 전체 통과)
uploadProfileImage path 반환·purpose 확정값 잠금(신규) · 조회 비-http → null 방어 · PATCH/온보딩 body 잠금 값 path로 갱신

### 실기기 확인 포인트
사진 교체 → 탭·수정 화면 렌더(서버 조합 URL) / "절대 URL이 아님" 로그 발견 시 공유. ※ JS-only — 검토 통과 후 OTA 가능(gitignore 스왑 필요)

## [OPS] 프로덕션 OTA 발행 — P-002~P-005 묶음 테플 배포 (2026-07-16)

**요청 경로**: 예진 직접 지시. **결과**: iOS 정상 발행, runtime `8d0d5504`(설치 빌드 a6aebbf8 일치), update `019f6a21-8998`. [대시보드](https://expo.dev/accounts/rocher/projects/kbap/updates/3c048736-0ed1-4752-864c-2c8d571646d8)

- 포함: `be9aae5`(P-002 스캔 신계약+가격) · `bc2d051`(P-005 애플 revoke) · `db95536`(P-003 맵기 -1+presigned) · `4afd89e`(P-004 프로필 이미지) — 전부 JS-only 확인(스왑 시 fingerprint 일치)
- `_photostyle` gitignore 스왑 절차 적용(7/16 OPS 선례) — 발행 순간만 라인 제거 후 복원, 트리 클린
- 실기기 확인 대기 항목은 각 P 보고 참조. 특히: 스캔 업로드 imagePath 실경로 / 프로필 purpose 추정값(발급 400 가능) / 애플 탈퇴 revoke
- 테스터 적용법: 앱 완전 종료 후 재실행 2번

## [P-004] KB-149 프로필 이미지 업로드 — 온보딩·수정·조회 실연결 (2026-07-16)

**커밋**: `4afd89e` (main) · **검증**: tsc 0 · jest 107/107 (18 suites, +4 tests)

### 사전 확인
- **expo-image-picker 이미 설치**(스캔 갤러리에서 사용 중) → **네이티브 변경 없음, 리빌드 불필요** — 7/24 전 빌드에 태울 항목 아님, OTA로 나감
- profileImageUrl: Onboarding·Update·MyProfile 3계약 전부 optional string 배포 확인

### 작업 결과
1. **업로드 공용화 이행**: P-003의 `uploadImage(file, purpose)` 재사용 — 반환만 `{ path, publicUrl }`로 확장(스캔=path, 프로필=publicUrl). 프로필 전용 로직은 `profileImage.ts` 40줄(1:1 크롭 픽커 + 업로드 래퍼)뿐
2. **온보딩 profile 스텝**: 아바타+카메라 뱃지(선택 사항) — 선택 즉시 업로드, **draft에 URL 보존**(중단 복귀 시 사진 유지), 제출 body 포함/미선택 시 필드 생략
3. **프로필 수정**: 사진은 국적 행과 같은 **즉시 적용** 시맨틱(선택→업로드→PATCH) — Save 대기 없음. 조회(탭·수정 화면)는 profileImageUrl 렌더, 없으면 기존 플레이스홀더
4. **실패 폴백**: 픽커/업로드 실패 → 정직한 에러 문구(전용 i18n 키 ×10) + 사진 없이 가입/수정 계속 가능

### BE 질의 2건 (지시대로 값 제외 나머지 완성)
1. **프로필용 purpose 값 미명시** (스펙 예시 MENU_SCAN뿐, enum 없음) — `PROFILE_IMAGE` 추정 사용. 상수 한 곳(profileImage.ts)이라 확정 시 한 줄 교체. 계정 필요라 에러 메시지 실측 불가 — 실기기에서 발급 400 나면 그 메시지로 역산 가능
2. **profileImageUrl에 publicUrl vs objectKey** — 지시대로 **publicUrl 우선** 채택(필드명이 Url + "만료 없는 표시용" 설명 부합). 반증되면 path로 전환

### 테스트 (신규/확장 4건)
uploadImage `{path, publicUrl}` 반환 · adaptProfile 매핑(URL 그대로/누락·빈문자열→null) · PATCH body 포함+`['me']` invalidate · 온보딩 body 포함/미선택 생략

### 실기기 확인 포인트
온보딩 사진 선택→가입 후 프로필 탭 표시 / 수정 교체 즉시 반영·앱 재시작 후 유지 / 미설정 플레이스홀더 / 비행기 모드 업로드 실패 시 에러 문구+진행 가능 / ⚠️ **purpose 추정값 — 발급 400 시 에러 메시지 공유 요망**

## [P-003] 맵기 -1 센티널 + presigned 발급 실연동 (2026-07-16)

**커밋**: `db95536` (main) · **검증**: tsc 0 · jest 103/103 (18 suites, +12 tests)

### 계약 재실측 (착수 시 — 서버 재배포 완료 상태)
- `POST /images/upload-url` 배포 확인: req 3필드 전부 required / res 6필드 전부 required — 프롬프트의 계약 그대로
- `spicinessPreference`: MyProfileResponse required int / ProfileUpdateRequest·OnboardingRequest optional

### 작업 1 — 맵기 -1 센티널
1. `adaptSpice()` 신설: **-1 포함 0..10 밖·비정수 → null(미설정)** — 칩 "-1/10" 노출 오작동 차단. 서버가 항상 값을 주므로 서버값이 진실 — -1 수신 시 로컬 fallback도 타지 않음(fallback은 필드 누락=구서버일 때만). P-001의 BE 질의 2건(미설정 표현/해제 전달) 이것으로 소멸
2. 해제: PATCH `spicinessPreference: -1` 전송 — "해제 후 재조회 시 값 되살아남" 한계 소멸(기존 로컬-only 처리 폐기)
3. **온보딩 스킵 = 필드 생략 유지 (판단)**: 계약상 optional + BE 정책이 "미설정 유저 DB -1 저장"이라 생략이 미설정으로 수렴 — 실측으로 반증되면 -1 명시 전송 전환(submit.ts 주석에 전환 지점). 계정 생성이 필요해 서버 왕복 실측은 못 함 → 실기기 확인 포인트에 포함

### 작업 2 — presigned 실연동 (scanImage.ts TODO(KB-72) 해소)
- `uploadImage(file, purpose)`: 발급 → PUT(**requiredHeaders 발급값 그대로**, BINARY_CONTENT, contentLength는 getInfoAsync 정확값) → complete(objectKey). contentType 확장자 매핑(png/heic/webp/gif, 기본 jpeg)
- **purpose 파라미터화 완료** — P-004(프로필, purpose만 다름)가 이 함수 재사용하면 됨 (중복 구현 금지 이행)
- 실패 폴백: 발급 400/PUT 비2xx/파일 소실 어디서든 null → imagePath `''`(텍스트-only, BE 허용 확정 반영). PUT 실패 시 complete 미호출(IMAGE-002 신고값 불일치 방지). KB-137 파일 삭제보다 업로드 선행 유지

### 테스트 (신규/갱신 12건)
adaptSpice 경계 4건(-1→null, **0 유효값 유지**, 10/11/3.5, 누락 시만 fallback) · 해제→-1 전송 + 0 전송 경계(useUpdateMe) · 업로드 성공 경로(발급 body·requiredHeaders·complete body 잠금) + 실패 3경로 폴백(scanImage 신규) · 검증 path 전송/실패 '' 폴백(useScan)

### 실기기 확인 포인트
스캔 → Metro 로그 `[scan] upload-url issued → image upload complete → POST /scans | imagePath = scan/...` 실경로 / 맵기 해제 → 앱 재시작 후에도 미설정 / 미설정 계정 프로필 칩에 "-1" 없음 / **온보딩 맵기 스킵 → 프로필 미설정 표시** (생략=미설정 수렴 검증 — 아니면 보고 요망)

## [P-005] KB-162 탈퇴 시 애플 재인증 + 토큰 revoke — 경로 A (2026-07-16)

**커밋**: `bc2d051` (main) · **검증**: tsc 0 · jest 91/91 (16 suites, +11 tests)

### 작업 결과
1. **재인증 게이트**: `appleRevoke.ts` 신규 — 기존 expo-apple-authentication 경로 재사용(새 라이브러리 없음). 로그인과 달리 Firebase credential을 만들지 않아 nonce/scope 불필요, `signInWithCredential` 미호출(타계정 인증 시 앱 세션이 갈아타지는 사고 차단)
2. **revoke**: `revokeToken(auth, authorizationCode)` — RNFB v25 modular API(iOS 네이티브 브릿지) 실측 확인. code 수명 5분 → 본인 대조 직후 즉시 호출, 성공 시에만 기존 탈퇴 흐름(withdrawBe→logOut→/login) 진행
3. **엣지 3종**: 시트 취소 / 타계정(credential.user ≠ 현재 apple.com providerData.uid — **식별자 부재로 대조 불가여도 거부**) / revoke·시트 실패 — 전부 탈퇴 중단 + 사유별 안내 카드, 계정 유지. revoke 없는 탈퇴 경로 없음
4. **구글 회원·웹·안드로이드 무변**: 분기 판별은 `currentIsAppleUser()`(providerData 기준) 한 곳 — iOS에서만 require(웹 번들 네이티브 모듈 격리 유지)
5. i18n +5키 ×10 로케일

### 테스트 (신규 11건)
provider 판별 4단(google→false 잠금 포함, 미로그인 크래시 없음) · 타계정/식별자 부재/코드 부재 거부 · 시트 취소/타계정/실패 시 revokeToken 미호출 + 중단 신호 · 성공 경로 code 전달 잠금

### 특이사항
- Firebase 유저 자체(auth 계정)는 기존 탈퇴 흐름대로 로컬 로그아웃만 — Firebase 계정 삭제는 기존 범위 밖 유지(변경 없음). revoke가 심사 요건의 핵심이라 여기까지가 경로 A
- withdrawBe 실패 시에도 로컬 세션 정리하는 기존 동작 유지(변경 없음) — revoke는 그보다 앞 단계라 "revoke 성공 없이 탈퇴 API 진행"은 없음

### 실기기 확인 포인트
애플 계정 탈퇴 완주 → iOS 설정→Apple 계정→로그인 및 보안→Apple로 로그인 목록에서 K-Bap **소멸** / 재가입 시 이메일 선택(가리기 포함) 화면 **재등장** / 구글 계정 탈퇴는 게이트 없이 기존과 동일 / 시트 취소·다른 Apple ID 선택 시 계정 유지 + 안내 카드

## [P-002] KB-72 스캔 실연결 마무리 — imagePath + 가격 (2026-07-16)

**커밋**: `be9aae5` (main) · **검증**: tsc 0 · jest 80/80 (15 suites, +1 suite/+7 tests)

### 계약 재실측 (착수 시)
- **presigned 발급 API 미배포 확인** → 시나리오 ② 적용 (업로드 파트만 스텁, 나머지 완성)
- `ScanRequest.imagePath` **required** 확정 (단 minLength 0 · 패턴 `^(?!https?://)` — `''` 통과 가능 추정)
- 응답 `idx` nullable화, `price`(KRW 정수·미표기 null·응답 전용) 추가 확인

### 작업 결과
1. **요청 전환**: `{ imagePath, items }` — 온디바이스 OCR·박스 유지(7/16 합의 준수). 발급 API 대기 동안 `imagePath: ''`(텍스트-only 폴백) — 스캔 무중단, 가짜 결과 없음(서버 거부 시 기존 에러 화면으로 정직하게 표면화)
2. **업로드 흐름**: `scanImage.ts` 신규 — `completeImageUpload()`(실코드, 발급 API 배포 시 그대로 사용) + `resolveScanImagePath()`(발급 파트만 TODO(KB-72) 스텁). KB-137 파일 정리 순서 확인: 업로드 해석이 삭제 트리거(사진 교체/언마운트)보다 항상 선행 — 구조상 업로드 전 삭제 경로 없음
3. **가격 표시**: 리스트 행 + 오버레이 pill에 서버 price 그대로(₩ 포맷만), null·비숫자 방어=미표시. OCR 추정가를 서버값으로 대체
4. **안전 게이트(개정판 반영)**: 응답에 없는 idx=드롭 유지(기존 테스트 3건 보존) · **idx=null 결과는 photoOnlyResults로 리스트 전용 노출**(오버레이 마커 없음—좌표 부재, 위험도 규칙 동일, matched=false→unable 강등, 결과 카운트 포함)

### 테스트 (신규/확장 7건)
가격 매핑 정상/null/비숫자 · idx=null 조인 제외(크래시 없음) · photoOnlyResults 2건(danger 유지/unable 강등·이름 폴백) · 요청 body imagePath 잠금 2경로(useScan.test 신규 — 샘플/사진, box 미전송 포함) · idx=null 리스트 노출 회귀(scanDefaultView 확장)

### 특이사항 · BE 질의 1건
- **imagePath `''` 허용 여부 확정 필요** — required인데 발급 API가 없어 클라이언트가 유효한 path를 만들 방법이 없음. 스키마상 `''`는 통과 추정이나 서버 의미(@NotBlank 여부) 미확인. 거부로 확정되면 정직한 에러 표시로 전환(scanImage.ts 주석에 전환 지점 명시). **발급 API 배포 일정도 공유 요망** — 배포되면 TODO 1곳만 채우면 됨
- 진행로그(Jira KB-72)에 "발급 API 대기" 기록함

### 실기기 확인 포인트
스캔 정상 왕복(서버가 `''` 거부하면 에러 화면 — 발생 시 공유) · 가격 표기 메뉴 ₩ 표시/미표기 미표시 · 사진 전용(박스 없는) 항목 리스트 노출

## [P-002] 🔄 착수 질의 — 응답에 없는 idx 처리 (지시 ↔ 기존 결정 충돌) (2026-07-16)

**질문**: 안전 게이트의 "응답에 없는 idx는 unable 처리"가 기존 7/10 결정과 충돌합니다.
- **기존(7/10 신계약, 테스트로 잠금)**: 응답에 없는 idx = 서버가 비음식 판정(원산지·가격·UI 문구) → **드롭**(마커·행 없음). Swagger도 여전히 "메뉴가 아닌 항목은 결과에서 제외" 명시.
- **이번 지시**: 응답에 없는 idx → **unable 처리**(표시 유지).
- 참고: 이번 계약에서 응답 `idx`가 nullable로 바뀜("사진 추출됐지만 대응 OCR 항목 없으면 null") — 사진이 판정 소스가 되면서 "응답에 없음=비음식" 등식이 약해진 건 사실이라 의도된 안전 강화일 수 있어 되묻습니다. 어느 쪽인지 확정해 주세요 (unable 전환이면 기존 테스트 3건 수정 포함해 반영).

**부수 관찰**: idx=null 결과 항목(사진에서만 추출된 메뉴)은 현재 FE에서 렌더 대상이 없어 버려집니다. DANGER 메뉴가 여기 실리면 사용자에게 안 보이는 갭 — 리스트 뷰에만 박스 없이 노출하는 안이 가능하나 이번 지시 범위 밖이라 미구현. 판단 요청.

**진행**: 이 건만 보류, 나머지(imagePath 전환·가격 표시·complete 연동·테스트)는 진행 중.

## [OPS] 프로덕션 OTA 발행 — P-001(3494917) 테플 배포 (2026-07-16)

**요청 경로**: 브릿지 큐 아님 — 예진 직접 지시(OTA 배포). **결과**: iOS 정상 발행, runtime `8d0d5504`(설치 빌드 a6aebbf8과 일치), update `019f6702-46d7`. 테스터는 앱 완전 종료 후 재실행 2번이면 적용.

- P-001의 "다음 OTA로 배포 가능" 항목 이행 — 상세 bookmarked·맵기 실연결이 테플에 반영됨
- 사전 정리 커밋 `427a172`: CLAUDE.md 추가 + `assets/_photostyle/`(디자인 참고 PNG 9.6MB) gitignore
- **특이사항**: 그 gitignore 라인이 expo fingerprint를 회전시켜(fingerprint가 .gitignore 존중) 1차 발행이 orphan runtime `d42be1d5`로 나감 — 어떤 빌드에도 도달 안 함(무해). 발행 순간만 라인을 빼는 스왑(ota-prod 스킬 3-1 eas.json 선례와 동일)으로 재발행해 해결
- **다음 네이티브 리빌드까지 유효한 운영 제약**: 프로덕션 OTA마다 같은 스왑 필요 + `_photostyle` PNG는 디스크에 남아 있어야 함(삭제 시 또 회전). 다음 리빌드 때 새 fingerprint로 재베이스라인하면 소멸 — FE 세션 메모리에 기록됨

## [P-001] KB-142·150 후속 — 상세 bookmarked 서버 필드 교체 + 맵기 실연결 (2026-07-15)

**커밋**: `3494917` (main, push 완료) · **검증**: tsc 0 · jest 73/73 (14 suites)

### 작업 1 — 상세 저장 상태를 서버 필드로 교체 (KB-142 계약 갭 해소)
- Swagger 실측 재확인: `FoodDetailResponse.bookmarked` (required, "비회원 조회는 항상 false") 배포 확인
- 어댑터 매핑 추가(누락/비불리언 → false 방어). `FoodDetail.bookmarked`는 옵셔널 — mock 상세 11건은 게스트/개발 경로라 미설정=false 취급, 라이브 어댑터는 항상 세팅
- 상세 화면 saved: 목록 캐시 유도 → `food.bookmarked` 기반 교체. 다른 기기·재설치에서도 저장 상태 정확(기존 한계 소멸)
- `useToggleBookmark`: onMutate가 목록 캐시(유지) + 상세 캐시(`['food', id, lang]`) bookmarked 동시 반전, onError 양쪽 롤백, onSettled 양쪽 invalidate. saved 리스트 해제/Undo도 상세 invalidate 동기화
- `useIsBookmarked` 제거(사용처 상세뿐), 계약 갭 주석·PROGRESS 한계 항목 "해소됨" 갱신

### 작업 2 — 맵기 서버 실연결 (TODO(KB-150) 3곳 해소)
- `useMe.ts`: PATCH body에 `spicinessPreference`(값 있을 때만), 로컬 보관은 마이그레이션 fallback으로 유지
- `memberAdapter.ts`: `wire.spicinessPreference` 숫자만 신뢰 → 아니면 로컬 fallback
- `submit.ts`: 온보딩 body 포함, 스킵 시 필드 생략(not required 실측 확인)
- profileImageUrl은 비범위 준수(KB-149 — presigned API 대기)

### 테스트
- 신규 `foodAdapter.test.ts`: bookmarked true/false 매핑 + 누락 방어 2건
- `toggleBookmark.test.tsx` 확장: 낙관 토글이 상세 캐시 bookmarked를 반전하는 단언 2건
- `useUpdateMe.test.tsx` 갱신(기존 spice 로컬 테스트 → 새 동작): PATCH body 포함 1건 + 해제(null)는 서버 미전송·로컬만 1건

### 변경 파일 (12)
`foodAdapter.ts` `foodDetailTypes.ts` `types.ts` `memberAdapter.ts` `bookmarks.ts` `useMe.ts` `submit.ts` `food/[id]/index.tsx` `foodAdapter.test.ts`(신규) `toggleBookmark.test.tsx` `useUpdateMe.test.tsx` `PROGRESS.md`

### 특이사항 · BE 질의 2건 (코드 주석 + PROGRESS 병기)
1. **spicinessPreference "미설정" 표현 미정** — MyProfileResponse에서 required인데 0은 SPICE_SCALE상 "맵지 않음"이라 미설정과 의미가 다름. 미설정 사용자의 응답 값(null 허용/필드 누락/센티널?) 확정 필요. FE는 null/누락 시 로컬 fallback으로 방어 중.
2. **"설정 해제"(null) PATCH 전달 방법 미정** — null 전송 vs 필드 생략 의미가 계약에 없음. 확정 전까지 해제는 로컬만 반영(필드 생략=서버 유지) → 서버에 값이 이미 있는 사용자는 해제 후 재조회 시 값이 되살아나는 한계.

### 실기기 확인 포인트
상세 저장 → 재설치/다른 기기에서도 표시 정확 · 맵기 수정→저장→재조회(서버 왕복) 값 유지 · 온보딩 spice가 요청 body에 포함. ※ JS-only 변경 — 방금 제출된 테플 빌드(a6aebbf8)에는 미포함, 다음 OTA로 배포 가능.
