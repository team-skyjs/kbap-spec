<!--
SYNC IMPACT REPORT
==================
Version change: (none) → 1.0.0  (initial ratification)
Bump rationale: First formal constitution authored from docs/최초-기획서.md. MAJOR baseline.

Principles defined (6):
  - I.   국제화 우선 (Internationalization-First) — reader/place 언어 2축 구분, place 언어는 데이터로 모델링(MVP=ko 고정)
  - II.  음식 중심 아키텍처 (Food-Centric Architecture)
  - III. 안전 정확성 우선 (Safety-First Accuracy) — NON-NEGOTIABLE
  - IV.  개인화 본질 (Personalization at the Core)
  - V.   단계별 출시 (Phased Delivery)
  - VI.  스펙이 진실의 원천 (Spec as Source of Truth)

Sections:
  - Added: 추가 제약 (Additional Constraints) — incl. 프론트엔드 기본 이모지 사용 금지(MUST NOT)
  - Added: 개발 워크플로 (Development Workflow)
  - Added: Governance

Template alignment:
  - .specify/templates/plan-template.md  ✅ aligned (Constitution Check reads this file dynamically; no hardcoded principles)
  - .specify/templates/spec-template.md  ✅ aligned (no constitution references)
  - .specify/templates/tasks-template.md ✅ aligned (no constitution references)
  - .specify/templates/checklist-template.md ✅ aligned (generic)

Version change: 1.0.0 → 1.0.1  (서비스명 확정: K-Bap)
Version change: 1.0.1 → 1.0.2  (MVP 지원 언어 9개 확정: en/zh-Hans/zh-Hant/ja/vi/id/th/ru/es)
Version change: 1.0.2 → 1.1.0  (API 계약 SSOT를 BE Swagger로 이관 — 원칙 VI·워크플로 §개정, 2026-07-02 회의 결정)
Version change: 1.1.0 → 2.0.0  (2026-07-14 레포 역할 전환 — MAJOR)
  - 원칙 VI 비호환 재정의: "스펙이 진실의 원천" → "정본 지도(SSOT Map)".
    본 레포 = FE 커맨드 센터(판단·조율·프롬프트 설계·검토 게이트). 직접 구현하지 않으며,
    정본은 도메인별 분산(Jira·Swagger·Claude Design·local-handoff)되고 본 레포가 그 지도를 유지.
  - 원칙 V 갱신: 회원가입 소셜 전용(2026-07-08 결정), 리뷰 조건부, 제출 목표 2026-07-16.
  - 개발 워크플로 §: spec-kit 흐름은 "신규 기능 명세 시"로 한정.

Deferred TODOs:
  - (해소됨) 서비스명 확정 = K-Bap (표시명) / 슬러그 kbap.
-->

# K-Bap Constitution

본 헌법은 한국 여행/거주 외국인을 위한 한식 안내 서비스 **K-Bap**의 모든 명세·설계·구현을 구속한다.
표시명(브랜드)은 **K-Bap**, 기술 슬러그는 `kbap`(레포 kbap-fe/kbap-be, 패키지 com.kbap)다.

## Core Principles

### I. 국제화 우선 (Internationalization-First)

- 언어에는 두 축이 있으며 구분하여 다룬다: **읽는 사람 언어(reader language)** = 사용자가
  읽는 UI·설명·번역, **장소 언어(place language)** = 음식이 제공되는 그 장소의 언어(메뉴판
  원문, 사장님께 보여줄 확인 문구 등).
- 사용자에게 노출되는 모든 UI 문자열(reader language)은 i18n 리소스로 관리한다. 한국어
  (또는 임의 단일 언어) 하드코딩을 **금지(MUST NOT)** 한다.
- "장소 언어로 출력하는 것이 기능 요구사항인 경우"(예: 사장님께 보여줄 확인 문구)는 reader
  i18n의 예외이며, 명세에 명시되어야 한다. 다만 이때도 장소 언어를 특정 언어로 하드코딩하지
  않고 **데이터/설정값으로 모델링한다(MUST)**. MVP에서 장소 언어는 한국어(`ko`)로 고정한다.
- 데이터 모델·API 응답은 다국어 확장이 가능한 구조여야 한다(MUST). 텍스트 필드는 언어
  코드와 함께 다룰 수 있어야 하며, 단일 언어 컬럼을 전제하지 않는다.
- 지원 언어: **MVP부터 9개를 모두 제공**한다 — `en`, `zh-Hans`, `zh-Hant`, `ja`, `vi`, `id`,
  `th`, `ru`, `es`(한국 인바운드 관광객 다수 언어). 폰트는 라틴·CJK·태국·키릴을 모두 커버해야
  한다(MUST, 글자 깨짐 0). 9개 이후 추가 확장 시 인구 규모 순을 우선순위로 한다. (Session 2026-06-29)
- 근거: 타겟 사용자가 한국어를 못 읽는 외국인이므로, 국제화는 부가 기능이 아니라 제품의
  전제 조건이다. 초기 설계에서 누락하면 전면 재작업 비용이 발생한다.

### II. 음식 중심 아키텍처 (Food-Centric Architecture)

- 정보와 리뷰의 1차 집계 단위는 **'음식(메뉴)'** 이다(MUST). '식당' 단위로 묶지 않는다.
- 지도·식당 엔티티는 1차 MVP의 핵심이 아니며, 도입하더라도 음식 중심 모델을 깨뜨려서는
  안 된다(MUST NOT). 추가 시 음식에 부속되는 형태여야 한다.
- 근거: 네이버맵·구글맵·파파고·구글렌즈와의 핵심 차별점이 "음식 단위 정보 공유"이다.
  이 축이 흔들리면 제품의 존재 이유가 사라진다.

### III. 안전 정확성 우선 (Safety-First Accuracy) — NON-NEGOTIABLE

- 알러지·종교·비건/채식 등 "못 먹는 음식" 판정과 위험도 표기(안전=초록 / 주의=노랑 /
  위험=빨강)는 사용자의 건강·신앙과 직결되므로 정확성과 신뢰성이 **다른 모든 목표에
  우선한다(MUST)**.
- 불확실성은 사용자에게 위험을 낮추는 방향으로 처리한다(MUST). 정보가 불완전하면 '안전'으로
  단정하지 않고 '주의'로 표기하고, 매장별로 달라질 수 있는 성분은 "사장님께 확인" 플로우로
  유도한다.
- 위험도 판정의 근거(어떤 성분/재료/확률에 기반했는지)는 추적 가능해야 한다(MUST). 근거
  없는 안전 단정을 금지한다(MUST NOT).
- "사장님께 확인" 화면의 한국어 문구는 메뉴판의 메뉴명과 정확히 일치해야 한다(MUST).
- 근거: 알러지 오판은 생명·법적 리스크다. 편의나 속도를 위해 안전 정확성을 타협할 수 없다.

### IV. 개인화 본질 (Personalization at the Core)

- 회원의 알러지·종교·비건·맵기 허용도·국적·언어 정보를 기준으로 한 개인화가 제품의 본질이며,
  핵심 기능(위험도 표기, 추천, 동일 국적 리뷰)은 이 프로필 위에서 동작해야 한다(MUST).
- 개인 정보(특히 알러지·종교 등 민감 정보)는 최소 수집·목적 내 사용 원칙을 따른다(MUST).
- 근거: 동일한 메뉴판도 사용자에 따라 안전/위험이 달라진다. 개인화 없는 번역은 기존 도구와
  차별화되지 않는다.

### V. 단계별 출시 (Phased Delivery)

- 출시는 다음 순서를 따른다(MUST): **1차 MVP** — 메뉴판 개인화(알러지/종교/비건) +
  회원가입(**소셜 전용: 애플/구글** — 이메일 가입 없음, 2026-07-08 결정) + 리뷰(**조건부** —
  2026-07-13 결정에 따라 미포함 시 2차 이월), 앱 제출 목표 **2026-07-16**, 7월 내 스토어 출시.
  **2차** — 가공식품, 커뮤니티. **3차** — 길거리 음식 사진 판별.
- 후순위 단계의 기능을 위해 현재 단계의 범위를 확장하지 않는다(MUST NOT). 단, 미래 단계를
  가로막는 비가역적 설계 결정은 피한다(SHOULD).
- 근거: 마감(2026-07-10)이 촉박하다. 범위 통제가 출시 가능성을 좌우한다.

### VI. 정본 지도 (SSOT Map) — 2026-07-14 재정의

- 본 레포(spec)는 **FE 트랙의 커맨드 센터**다: Jira 기반 할일 관리, 디자인/FE 세션 프롬프트
  설계, 검토 게이트(리뷰)를 담당한다. **직접 코드를 구현하지 않는다(MUST NOT)** — 구현은
  kbap-fe/kbap-server 세션에 프롬프트로 위임한다.
- 정본(SSOT)은 도메인별로 분산되며, 본 레포는 그 **지도**를 유지한다(MUST):
  - **이슈/태스크 상태** = Jira(KB) · **API 계약** = BE Swagger(런타임 생성) ·
    **디자인** = Claude Design 프로젝트 · **BE 공유 정책** = local-handoff/kbap ·
    **출시 큰그림** = LAUNCH.md · **제품 원칙/제약** = 본 헌법 · **기능 의도(FR)** = spec.md.
  - 본 레포의 `contracts/openapi.yaml`·`tasks.md`는 **참고/기록용**이며 정본이 아니다.
- 정본 간 어긋남을 발견하면 정본 쪽을 따르고 지도를 갱신한다. **침묵의 드리프트를 금지한다
  (MUST NOT)** — 특히 여기서 생성하는 프롬프트는 낡은 문서가 아니라 정본을 인용해야 한다(MUST).
- 근거: 태스크는 Jira, 계약은 Swagger, 디자인은 Claude Design에서 코드/도구로 생성·관리되는
  현실에서, 단일 레포 SSOT 선언은 유지 불가능한 허구가 된다. 커맨드 센터의 가치는 모든 정본을
  갖는 것이 아니라 **어디가 정본인지 항상 정확히 아는 것**이다.

## 추가 제약 (Additional Constraints)

- **프론트엔드 화면에 기본(시스템/유니코드) 이모지 사용을 금지한다(MUST NOT).** 아이콘이
  필요한 곳에는 일관된 아이콘 세트(아이콘 폰트 또는 SVG 등 디자인 시스템에서 정의한 자산)를
  사용한다. 위험도 표기(안전/주의/위험) 등 시각 신호도 기본 이모지가 아니라 디자인된 아이콘
  으로 구현한다. 근거: 기본 이모지는 OS/플랫폼/언어별 렌더링이 제각각이라 다국어 글로벌
  사용자에게 일관된 품질과 의미 전달을 보장하지 못한다(원칙 I과 직결).
- 기술 스택은 본 헌법에서 강제하지 않으며 `/speckit-plan` 단계에서 결정한다. 단, 어떤
  스택을 택하든 위 원칙(특히 I·III·VI)을 충족할 수 있어야 한다.
- 민감 개인정보(알러지·종교 등) 취급은 적용 가능한 개인정보 보호 규정을 준수한다(MUST).
- 외부 인식·번역(메뉴판 OCR/번역, 음식 인식) 결과는 항상 불확실성을 동반한다고 전제하고,
  원칙 III에 따라 보수적으로 처리한다.
- **장소 일반화 대비(데이터 모델링만, 구현은 단계적):** 메뉴·성분·확인 문구 등 장소에 묶인
  데이터는 `locale` 등으로 장소 언어를 식별할 수 있게 모델링한다(MUST). 단, 다음은 **MVP에서
  구현하지 않으며 3차 이후로 미룬다(MUST NOT in MVP):** 사용자 위치/국가 자동 감지, 여러
  국가의 메뉴·성분 DB, place↔reader 임의 언어 조합 번역 파이프라인. 근거: 장기적으로 한국
  방문 외국인뿐 아니라 해외의 사용자(예: 한국인의 해외여행)까지 확장할 수 있어야 하나(원칙
  V의 비가역적 결정 회피), 지금 구현까지 일반화하면 MVP 일정과 충돌한다. "변수로 부르되 값은
  하나(`ko`)만 채운다"는 선을 지킨다.

## 개발 워크플로 (Development Workflow)

- **신규 기능을 명세할 때만** spec-kit 흐름을 따른다: `/speckit-specify` → `/speckit-clarify`(선택)
  → `/speckit-plan` → `/speckit-tasks`. 일상 운영(할일·프롬프트·리뷰)은 커맨드 센터 스킬
  (`todo`·`jira`·`design-prompt`·`fe-prompt`·`review-gate`)로 한다. 태스크 실행 관리는 Jira(KB).
- `/speckit-plan`의 Constitution Check 게이트에서 본 헌법 위반이 발견되면, 명세 또는 설계를
  수정해 해소하거나 Complexity Tracking에 정당화 근거를 명시해야 한다.
- API 계약 변경은 **BE Swagger(정본)에 먼저 반영**하고 FE는 거기에 맞춘다. 본 레포
  `contracts/openapi.yaml`(참고용)은 필요 시 뒤따라 갱신한다(2026-07-02 결정).

## Governance

- 본 헌법은 다른 모든 관행에 우선한다. 충돌 시 헌법이 이긴다.
- 개정 절차: 변경 제안 → 영향 검토(의존 템플릿·진행 중 명세) → 버전 증가 → 커밋. 모든 개정은
  Sync Impact Report로 기록한다.
- 버전 정책(유의적 버전): MAJOR = 원칙의 제거/비호환 재정의, MINOR = 원칙/섹션 추가 또는
  실질적 확장, PATCH = 문구·오타·비의미적 정정.
- 준수 검토: 명세·계획·태스크 리뷰 시 본 헌법 준수 여부를 확인한다. 복잡성은 정당화되어야
  한다.

**Version**: 2.0.0 | **Ratified**: 2026-06-15 | **Last Amended**: 2026-07-14
