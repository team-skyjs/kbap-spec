# FE 작업 프롬프트 큐

> 운영: 커맨드 센터(스펙 레포 세션)가 항목 작성·수정, **새 항목은 최상단 추가**.
> FE 세션은 예진이 지목한 ID(지목 없으면 최상단 ⬜)를 처리하고 **상태만** 갱신:
> ⬜ 대기 → 🔄 착수 → ✅ 완료(커밋 해시 병기). 본문 수정은 커맨드 센터만.
> 완료 보고는 [REPORTS.md](REPORTS.md) 최상단에 같은 ID로 작성.

---

## [P-009] ⬜ KB-174 후속 — 탭별 스켈레톤 (실제 레이아웃 미러링, 범용 리스트 스켈레톤 대체)

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

## [P-008] ⬜ KB-174 후속 — useFoods의 401 삼키기 제거 (음식 탭 백지 잔존 버그)

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
