# UI 폴리시 플랜 (2026-07-21 — 발주 전 계획, P-031 후보)

> 소스 3종: ① apple-design 스킬(kbap-fe/.claude/skills/apple-design — RN-MAPPING.md 포함)
> ② 정적 UI 감사(커맨드 센터 7/21 실측) ③ kinetics.colorion.co 117종 중 선별.
> 원칙: **절제**(apple-design §16 — 안전 앱 톤, 장식 금지) · 모멘텀 없는 UI에 바운스 금지 ·
> JS-only(reanimated 4.3 기설치, **새 네이티브 추가 금지**) · kinetics의 damping/stiffness
> 수치는 reanimated withSpring 파라미터로 직역 가능.

## A. 정적 UI (감사 확정 — 출시 전 필수 후보)

1. **대비 수정**: `ink3`(#B0A395) 2.28:1 → 어둡게 또는 사용처 ink2 승격 / 소형 주황 텍스트(3.73:1) 점검. WCAG 4.5:1 목표
2. **글자 크기 상한**: `maxFontSizeMultiplier`(×1.3) 전역 — 시스템 큰글씨에서 고정높이 33곳 잘림 방지 (정식 대응은 출시 후)
3. **스캔 unmatched 안내 2줄 허용** — 안전 행동 지시가 말줄임되지 않게

## B. 애니메이션 기본기 (apple-design — 출시 전 후보)

4. **press 즉시 피드백 통일** — 전 버튼/카드 onPressIn scale 0.97 (릴리스 대기 금지)
5. **시트/모달 스프링 통일** — 등장 damped(바운스 0), 기존 timing 대체
6. **reduced-motion** — `ReducedMotionConfig` + 크로스페이드 대체

## C. kinetics 선별 14종 (화면별 매핑)

### 출시 전 후보 (저위험·기존 화면 강화)
| kinetics 효과 | 우리 적용처 | 비고 |
|---|---|---|
| Quantity Stepper (값 팝) | **P-030 주문카드 수량 스테퍼** | P-030에 흡수 가능 |
| Choice Chips / Tag Input (칩 팝인/아웃) | 기피 재료 선택·요약 칩 (P-026 영역) | 칩 추가/제거 순간 |
| Bookmark Toggle (리본 채움) | 음식 상세·목록 북마크 | 1:1 대응 |
| Tab Pill Glide (인디케이터 글라이드) | 스캔 결과 리스트/리스크/원본 세그먼트 | 스크린샷의 그 토글 |
| Error Shake (감쇠 진동) | 로그인 실패·폼 검증 에러 | 재트리거 가능하게 |
| Success Check (스트로크 드로잉) | 온보딩 제출·업로드 완료 | SVG 스트로크 |
| Stagger Entrance (순차 등장) | **스캔 결과 리스트** — 분석 완료 리워드감 | 60ms 스태거, reduced-motion 시 생략 |
| Step Progress (노드 팝) | 온보딩 스텝 인디케이터 | |
| Shimmer Skeleton (스윕 하이라이트) | 기존 스켈레톤(KB-174)에 shimmer | 이미 골격 있음 |

### 출시 후 (제스처·플로우 변경 수반)
| 효과 | 적용처 | 왜 후순위 |
|---|---|---|
| Hold to Confirm (홀드 링) | **계정 삭제(탈퇴)** — destructive 확인의 정석 | 검증된 탈퇴 플로우 변경 = D-3 위험 |
| Swipe to Reveal | 북마크 목록 삭제 | 제스처 경로 신규 |
| Undo Snackbar | 북마크 해제 undo | 신규 패턴 |
| Number Counter/Odometer | 랭킹 점수 카운트업 | 랭킹 QA(Q-15) 후 |
| Status Pill (idle→loading→success) | 스캔 촬영 버튼 상태 | 스캔 크리티컬 경로라 신중히 |

### 명시적 제외 (절제 원칙)
hover·cursor 계열 전부(모바일 무의미) / Confetti·Glitch·Neon·Aurora·Liquid 등 장식(안전 앱 톤 훼손) / Magnetic·Parallax(과함) / Pulse Badge 상시 루프(0.2Hz 루프 — apple-design §14 금지 패턴)

## 발주 시나리오 (예진 결정 대기 — 아직 발주 안 함)

- **P-031 (출시 전)**: A1–3 + B4–6 + C 출시 전 9종 중 **엄선 4~5개**(주문카드 스테퍼는 P-030에, 칩 팝·북마크·세그먼트 글라이드·스캔 스태거 추천). 2~3pt, JS-only OTA
- **P-03x (출시 후)**: 나머지 kinetics + Dynamic Type 정식 + tracking 토큰화 + KB-192(프로필 UI)와 병합
