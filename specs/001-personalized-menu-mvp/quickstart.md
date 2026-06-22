# Quickstart / 검증 가이드 — 001-personalized-menu-mvp

이 문서는 MVP가 end-to-end로 동작함을 증명하는 **검증 시나리오**다. 구현 코드는 포함하지 않는다
(상세는 tasks.md/구현 단계). 계약은 [`contracts/openapi.yaml`](./contracts/openapi.yaml), 데이터는
[`data-model.md`](./data-model.md) 참조.

## 전제

- FE: Expo 앱(개발 빌드), BE: Spring Boot 로컬 + MongoDB(로컬/DocumentDB), 큐레이션 카탈로그
  데이터(최소 표본) 접근 가능(D5 가정).
- 계약 기반: BE는 openapi.yaml 구현, FE는 openapi.yaml에서 타입 생성.

## 핵심 흐름 검증 (US1~US3 = 7/10 필수)

### 1) 온보딩 → 프로필 (US1)
- 이메일/애플/구글로 가입 → `POST /auth/*` 토큰 수신.
- 식이 제약 **필수** 입력(건너뛰기 불가) → `PATCH /me` 반영.
- 안전 고지 동의 → `POST /me/consent` 기록.
- 기대: `GET /me`가 입력한 제약/국적/언어를 반환.

### 2) 메뉴판 스캔 → 위험도 오버레이 (US2)
- 앱에서 메뉴판 촬영 → 온디바이스 OCR로 원문+위치 추출.
- `POST /scan/menu`에 원문 배열 전송 → 응답의 각 항목에 reader 언어명 + `risk` 포함.
- 기대:
  - 사용자 제약과 충돌하는 메뉴 = `danger`.
  - 매장 변동 성분 = `caution`.
  - 카탈로그 미등록 메뉴(예: "신상 화채") = `unable`(절대 safe 아님) + 미등록 로그 적재.
  - 앱이 응답 `rawText`로 온디바이스 bounding box에 오버레이, 원본↔번역 토글 동작(FR-037).

### 3) 음식 디테일 → 사장님 확인 (US2/US3)
- 메뉴 탭 → `GET /foods/{id}`: 이름/평점/설명(≤150자)/사진/성분(위험순) 반환, `riskBasis` 존재.
- '주의' 성분 선택 → `GET /foods/{id}/owner-confirmation?ingredientCode=...&scannedMenuName=...`.
- 기대: `menuNameKo`가 스캔 인식 메뉴명과 **정확히 일치**(FR-019), `questionKo`는 한국어.

## 보조 흐름 (P2/P3)

### 4) 탐색 & 리뷰 (US4)
- `GET /foods?q=...` 탐색 → `GET /foods/{id}/reviews`(전체/같은국적 집계, `translateTo`로 번역,
  `nationality`로 같은 국적 필터).
- `POST /foods/{id}/reviews`(rating 1–5) → 작성 반영, 집계 갱신.

### 5) 홈 (US6)
- `GET /home`: `recent`(최근 스캔/조회 + 위험도) + `recommended`(개인화 위험도) 반환.
- 신규 사용자: 빈 상태(0건) 정상 표기.

### 6) 프로필 & 계정 삭제 (US5)
- `GET /me/reviews`, `GET /me/ranking`(복합 점수 tier) 확인.
- `DELETE /me` → 민감정보 즉시 완전 삭제, 본인 리뷰는 `anonymized=true`로 보존(평점 유지).

## 합격 기준(명세 SC 연계)
- false-safe(위험한데 safe로 표시) 0건 — 미확실/미등록은 caution/unable로 강등(SC-003).
- 스캔 메뉴명 ↔ 사장님 확인 메뉴명 불일치 0건(SC-006).
- 가입→프로필 완료 5분 이내(SC-001), 스캔 결과 표시 평균 10초 이내(SC-002).

## 헌법 게이트 확인 포인트
- UI에 기본 이모지 없음(디자인 아이콘만). 모든 reader 텍스트 i18n. place 언어=ko 데이터화.
- 검수 안 된 음식 '안전' 미표시. 위험도 `riskBasis`로 근거 추적 가능.
