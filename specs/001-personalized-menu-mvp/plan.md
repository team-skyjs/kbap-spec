# Implementation Plan: 개인화 한식 메뉴 안전 안내 (MVP)

**Branch**: `001-personalized-menu-mvp` | **Date**: 2026-06-22 | **Spec**: [spec.md](./spec.md)

**Input**: Feature specification from `specs/001-personalized-menu-mvp/spec.md`

## Summary

한국 방문 외국인이 식이 프로필을 등록하고 한식 메뉴판을 촬영하면, 온디바이스 OCR로 메뉴를
인식하고 BE가 큐레이션 카탈로그에 매칭·번역·개인화 위험도(안전/주의/위험/판정불가)를 산출해
앱이 촬영 이미지 위에 오버레이로 안내한다. '주의' 성분은 사장님께 한국어로 확인하고, 음식 단위
리뷰를 본다. 3-레포(spec/fe/be) 구조이며 본 spec 레포의 OpenAPI 계약이 FE/BE의 SSOT다.

## Technical Context

**Language/Version**: FE TypeScript (React Native + Expo) · BE Kotlin (Spring Boot 3.x, Java 21)

**Primary Dependencies**: FE — Expo Router, React Query, i18next + expo-localization,
`@react-native-ml-kit/text-recognition`(온디바이스 OCR). BE — Spring Web, Spring Security(JWT),
Spring Data MongoDB, springdoc-openapi, 외부 번역 API 어댑터(기본 Google Cloud Translation, 교체 가능)

**Storage**: MongoDB(AWS DocumentDB) — 앱 데이터. AWS S3 — 음식 사진 등 정적 자산.
메뉴 사진은 서버 미저장(온디바이스 OCR). 위험도/성분/음식 카탈로그 = 외부 큐레이션 DB(읽기 소비)

**Testing**: FE — Jest + React Native Testing Library. BE — JUnit5 + Testcontainers(Mongo).
계약 — openapi.yaml 기준 contract 검증

**Target Platform**: iOS 15+ / Android 8+ (모바일 앱) · BE: AWS(컨테이너)

**Project Type**: Mobile app + REST API (3-레포 폴리레포: spec/fe/be)

**Performance Goals**: 스캔 결과 표시 평균 10초 이내(SC-002), 가입→프로필 5분 이내(SC-001)

**Constraints**: i18n 필수(하드코딩 금지), 기본 이모지 금지, false-safe 0(SC-003),
스캔/사장님 확인 메뉴명 일치(SC-006), MVP 마감 2026-07-10

**Scale/Scope**: MVP 1차. 화면 ~10개(A~J), 핵심 엔티티 ~8, REST 엔드포인트 ~16

## Constitution Check

*GATE: Phase 0 전 통과 필수. Phase 1 설계 후 재확인.*

| 원칙/제약 | 계획상 준수 방법 | 상태 |
|-----------|-----------------|------|
| I. 국제화 우선 | reader 텍스트 전부 i18next, place 언어(ko)는 데이터(OwnerConfirmation.placeLanguage)로 모델링, BE 번역 일원화 | ✅ |
| II. 음식 중심 | API/데이터가 canonical Food ID 기준, 식당 엔티티 없음 | ✅ |
| III. 안전 정확성(NON-NEGOTIABLE) | 위험도는 큐레이션 데이터 기반, 미매칭/불확실=unable·caution(safe 단정 금지), `riskBasis`로 근거 추적, 온보딩 동의+상시 고지 | ✅ |
| IV. 개인화 | 위험도=User.restrictions×성분, 민감정보 즉시 완전삭제+리뷰 익명화 | ✅ |
| V. 단계별 출시 | MVP만(가공식품/커뮤니티/푸시 제외), 미래 비가역 결정 회피 | ✅ |
| VI. 스펙=SSOT | contract-first openapi.yaml을 spec 레포에 두고 FE/BE가 소비 | ✅ |
| 추가: 이모지 금지 | 디자인 아이콘 세트(와이어프레임 검증 완료) | ✅ |
| 추가: 장소 일반화 | locale로 모델링, MVP 값 ko 고정, 위치감지·다국가 DB는 미구현 | ✅ |

**위반 없음 → Complexity Tracking 비움.**

## Project Structure

### Documentation (this feature)

```text
specs/001-personalized-menu-mvp/
├── plan.md              # 본 파일
├── research.md          # Phase 0 결정
├── data-model.md        # Phase 1 엔티티(소유권 분리)
├── quickstart.md        # Phase 1 검증 가이드
├── contracts/
│   └── openapi.yaml     # API 계약(SSOT)
├── design-brief.md      # 디자인 핸드오프(이미 작성)
└── tasks.md             # /speckit-tasks 산출(아직 없음)
```

### Source Code (3-레포 폴리레포)

```text
# 레포 1: spec (현재, SSOT)
specs/001-personalized-menu-mvp/contracts/openapi.yaml   # 계약 원본
.specify/memory/constitution.md                          # 원칙

# 레포 2: kbap-fe (React Native + Expo)
app/                      # Expo Router 화면
├── (onboarding)/
├── (tabs)/                # 5-탭: home · food · scan(중앙 FAB) · community(잠김/phase2) · profile
components/               # 디자인 시스템(위험도 4상태 등, 와이어프레임 기반)
features/                 # scan(OCR), risk-overlay, reviews ...
lib/api/                  # openapi.yaml에서 생성한 타입 + 클라이언트
lib/i18n/                 # 언어 리소스(영어 우선)
tests/

# 레포 3: kbap-be (Kotlin + Spring Boot)
src/main/kotlin/.../
├── auth/                 # JWT, Apple/Google OAuth, 이메일 인증
├── user/                 # 프로필, 제약, 동의, 계정삭제/익명화
├── food/                 # 카탈로그 소비, 디테일, 위험도 계산
├── scan/                 # /scan/menu (매칭+번역+위험도)
├── review/               # 리뷰/평점/집계
├── ranking/              # 복합 점수
├── curation/             # 외부 큐레이션 DB 접근 어댑터(가정)
└── translation/          # 번역 API 어댑터(교체 가능)
src/test/kotlin/
```

**Structure Decision**: 3-레포 폴리레포(헌법 VI). spec 레포가 계약·원칙의 SSOT. fe/be는 계약을
생성/구현해 소비. fe/be 레포는 `/speckit-tasks` 직전/직후 생성 권장.

## 통합 지점(주의)

1. **큐레이션 DB**(외부 팀): 음식 카탈로그·성분·태그를 BE `curation/` 어댑터로 읽기 소비.
   접근 방식(공유 DB vs 내부 API)은 그 팀과 확정 필요 — MVP는 접근 가능 가정.
2. **번역 API**(외부): BE `translation/` 어댑터. 기본 제공자 1개로 시작, 교체 가능.
3. **소셜 로그인**: Apple/Google idToken 검증(BE), 이메일 인증코드.
4. **온디바이스 OCR**: FE가 텍스트+위치 추출, 위치는 기기 보관(오버레이용).

## Complexity Tracking

해당 없음(헌법 위반 없음).
