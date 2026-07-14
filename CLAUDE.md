Product: K-Bap (display name) · slug `kbap` (package com.kbap) · codename kfood (this dir)

# 이 레포의 역할 — FE 커맨드 센터 (2026-07-14 전환)

spec-kit 스펙 레포로 출발했으나 지금은 **FE 트랙의 커맨드 센터**다.
**이 레포에서 직접 코드를 구현하지 않는다.** 하는 일 4가지:

1. **할일 관리** — Jira(KB) 정본 기반 종합 보고 (`todo` 스킬) · 이슈 생성/전환 (`jira` 스킬)
2. **디자인 프롬프트 설계** — 디자인 세션(Claude Design)에 보낼 프롬프트 (`design-prompt` 스킬)
3. **FE 프롬프트 설계** — kbap-fe 구현 세션에 보낼 프롬프트 (`fe-prompt` 스킬)
4. **검토 게이트** — Jira `검토 중` 이슈를 코드·DoD 대조 후 완료/반려 (`review-gate` 스킬)

# SSOT 지도 (어디가 정본인가 — 헌법 VI)

| 도메인 | 정본 |
|---|---|
| 이슈/태스크 상태 | **Jira KB** (simhani1.atlassian.net — 접속 규격은 `jira` 스킬) |
| API 계약 | **BE Swagger** https://meogo.handev.site/swagger-ui — 이 레포 `contracts/openapi.yaml`은 참고용 |
| 디자인 | **Claude Design 프로젝트** (fe-handoff §2 참조, Direction G) |
| BE 공유 정책 | `/Users/yejinkim/dev/local-handoff/kbap/` (be-handoff·risk-display-policy·auth-onboarding-policy·meeting-agenda) |
| 출시 큰그림 | `specs/001-personalized-menu-mvp/LAUNCH.md` (타임라인·크리티컬패스·리스크) |
| 제품 원칙/제약 | `.specify/memory/constitution.md` (v2.0.0) |
| 기능 의도(FR) | `specs/001-personalized-menu-mvp/spec.md` |

여기서 생성하는 모든 프롬프트는 **정본을 인용**한다. 낡은 문서 인용 = 드리프트 오염(헌법 VI 위반).

# 이웃 레포 (절대경로)

- FE: `/Users/yejinkim/dev/kfood/kbap-fe` — React Native + Expo (TypeScript)
- BE: `/Users/yejinkim/dev/kfood/kbap-server` — Kotlin + Spring Boot + MongoDB(AWS DocumentDB). 문서에 "kbap-be"로 표기된 곳 = 이 레포
- 파일 교환: `/Users/yejinkim/dev/kfood/dropbox` (yj/jh) · 정책 핸드오프: `/Users/yejinkim/dev/local-handoff/kbap/`
- OCR = on-device ML Kit · 번역 = BE 서버측 · 3-repo 폴리레포

# 레거시 (읽을 때 주의)

- `specs/001-personalized-menu-mvp/tasks.md` — **동결**. 태스크 관리는 Jira로 승계(2026-07-06). FR↔태스크 추적 기록용.
- `.specify/`(스크립트·템플릿) + `speckit-*` 스킬 — 신규 기능 명세 때만 사용.
- `fe-handoff.md` — 목업 단계(완료) 역사 문서. 유효 섹션은 상단 배너 참조.

<!-- SPECKIT START -->
Active feature: 001-personalized-menu-mvp
- Plan: specs/001-personalized-menu-mvp/plan.md
- Spec: specs/001-personalized-menu-mvp/spec.md
- API contract (SSOT): **BE Swagger** — https://meogo.handev.site/swagger-ui (meeting 2026-07-02). This repo's specs/001-personalized-menu-mvp/contracts/openapi.yaml is reference-only, NOT the contract SSOT.
- Constitution: .specify/memory/constitution.md
<!-- SPECKIT END -->
