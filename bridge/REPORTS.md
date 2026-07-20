# FE 완료 보고

> 운영: FE 세션이 작업 완료 시 **최상단에** 해당 ID로 보고 작성 (이 파일의 쓰기 주인은 FE 세션).
> 분량은 채팅으로 하던 수준: 커밋 해시, 변경 파일, 항목별 결과, 검증(tsc/jest), 특이사항·질의.
> 커맨드 센터가 검증 후 피드백/반려는 [PROMPTS.md](PROMPTS.md) 최상단에 새 항목으로 남긴다.
> Jira 전환은 여기서 하지 않는다 (검토 게이트 몫).

---

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
