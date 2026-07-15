# FE 완료 보고

> 운영: FE 세션이 작업 완료 시 **최상단에** 해당 ID로 보고 작성 (이 파일의 쓰기 주인은 FE 세션).
> 분량은 채팅으로 하던 수준: 커밋 해시, 변경 파일, 항목별 결과, 검증(tsc/jest), 특이사항·질의.
> 커맨드 센터가 검증 후 피드백/반려는 [PROMPTS.md](PROMPTS.md) 최상단에 새 항목으로 남긴다.
> Jira 전환은 여기서 하지 않는다 (검토 게이트 몫).

---

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
