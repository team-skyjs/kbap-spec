# Phase 0 Research — 001-personalized-menu-mvp

결정 사항과 근거. 모든 NEEDS CLARIFICATION 해소됨.

## D1. 프론트엔드: React Native + Expo (TypeScript)

- **Decision**: React Native + Expo, TypeScript. 네비게이션 Expo Router, 서버 상태 React Query,
  i18n은 i18next + expo-localization, 온디바이스 OCR은 `@react-native-ml-kit/text-recognition`.
- **Rationale**: iOS+Android 단일 코드베이스로 18일 일정에 유리, i18n·카메라·푸시 생태계 성숙,
  와이어프레임이 React 기반이라 이지성↑. 환경에 Expo 도구/스킬 존재.
- **Alternatives**: Flutter(React 자산 재사용 불가), 네이티브(공수 2배 — 일정 부적합).

## D2. 백엔드: Kotlin + Spring Boot + MongoDB(AWS DocumentDB)

- **Decision**: Kotlin + Spring Boot 3.x(Java 21), MongoDB(AWS DocumentDB). 인증은 Spring
  Security + JWT(access/refresh), 소셜 로그인 Apple/Google OAuth, 이메일은 인증코드 방식.
- **Rationale**: 전담 BE 개발자 결정. 별도 레포·API 계약 SSOT 구조에 부합.
- **Alternatives**: NestJS+Postgres / Supabase(BaaS) — 팀 역량/취향상 미채택.

## D3. API 계약: OpenAPI 3 / REST — contract-first

- **Decision**: `contracts/openapi.yaml`(이 spec 레포)을 **권위 원본(SSOT)** 으로 둔다. FE는 여기서
  TS 타입을 생성(openapi-typescript 등), BE는 이를 구현. 변경은 spec 레포에서 먼저.
- **Rationale**: FE/BE 분리 + 헌법 VI(SSOT). REST는 단순·보편, 툴링 풍부, 18일 MVP에 적합.
- **Alternatives**: GraphQL(초기 셋업·러닝커브 과함), code-first(계약이 코드에 종속 → SSOT 약화).

## D4. 메뉴 스캔 파이프라인: 온디바이스 OCR + BE 번역/판정

- **Decision**:
  1. 앱: ML Kit `text-recognition`으로 메뉴판에서 **텍스트 + bounding box** 추출(온디바이스).
  2. 앱 → BE: 인식된 원문 메뉴명 배열 + reader 언어 전송(이미지는 전송하지 않음).
  3. BE: 각 원문을 **큐레이션 카탈로그에 매칭** → 표시명 생성(매칭 시 카탈로그 현지화 이름,
     미매칭 시 **BE 서버측 번역 API**) → 사용자 프로필 기준 **위험도 계산** → 미등록은 로그.
  4. 앱: 응답을 온디바이스 bounding box 위에 **오버레이**(FR-008), 원본↔번역 토글(FR-037).
- **Rationale**: 메뉴 이미지가 서버로 안 올라가 프라이버시·통신량 유리. 위치정보는 기기에만
  존재하므로 오버레이가 빠름. 번역은 BE 일원화로 일관·캐싱 용이.
- **번역 제공자**: BE측 외부 MT(예: Google Cloud Translation)로 시작하되 **교체 가능한 어댑터**로
  추상화. 매칭된 음식은 카탈로그 현지화 이름을 우선(품질·안전).
- **Alternatives**: 전량 온디바이스 번역(품질 낮음), 서버 OCR(이미지 업로드 비용·프라이버시 손해).

## D5. 위험도 판정 데이터 = 외부 큐레이션 DB (가정)

- **Decision**: 음식 카탈로그(표준명·현지화 이름·설명·사진·성분)와 알러지/종교/식이 매핑은
  **별도 팀의 자체 큐레이션 DB**가 제공한다고 가정한다. BE는 이를 **읽기 전용으로 소비**한다.
- **Integration(TBD with that team)**: 접근 방식은 (a) 공유 데이터스토어 직접 읽기 또는
  (b) 큐레이션 팀이 노출하는 내부 API 중 하나. MVP는 BE가 접근 가능하다고 가정하고, 구체
  방식은 그 팀과 확정. 본 기능 범위는 "소비"까지.
- **Rationale**: 명세 FR-015/Q1 결정. 검수된 음식만 '안전' 허용, 미검수/미매칭은 '판정불가'.

## D6. 저장소·호스팅

- **Decision**: 음식 사진 등 정적 자산은 **AWS S3**. **메뉴 사진은 서버에 저장하지 않음**(온디바이스
  OCR). 앱 데이터(User/Review/Scan 로그 등)는 MongoDB(DocumentDB). BE는 AWS(ECS/Elastic
  Beanstalk 등)에 배포, FE는 Expo EAS 빌드 + OTA.
- **Rationale**: DocumentDB가 AWS이므로 인프라 일원화. 메뉴 이미지 미저장으로 비용·프라이버시 이점.

## D7. 안전 고지/동의 & 민감정보

- **Decision**: 온보딩 1회 동의(FR-030) + 위험 화면 상시 고지, 동의 이력 보관. 계정 삭제 시
  개인·민감정보 즉시 완전 삭제, 리뷰는 익명화 보존(FR-032).
- **Rationale**: 헌법 III/IV, 명세 결정 그대로.

## 미해소/후속(범위 밖, 추후)

- 큐레이션 DB 접근 방식의 최종 확정(공유 DB vs 내부 API) — 큐레이션 팀과 협의.
- 번역 MT 제공자 최종 선택(어댑터 뒤에서 교체 가능).
- 푸시 알림(2차), 커뮤니티(2차)는 본 계획 범위 밖.
