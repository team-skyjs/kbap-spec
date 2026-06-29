# FE 핸드오프 — K-Bap MVP (목업 → React Native 구현)

이 문서는 **kbap-fe 레포에서 FE를 구현하는 세션/개발자**를 위한 단일 안내서다. 목표:
**API 없이(다음 주 연결), high-fi 목업대로 전 화면을 mock 데이터로 보이게** 만든다.

> 읽는 순서: 1) 이름 규칙 → 2) 출처 → 3) RN 변환 규칙 → 4) 디자인 토큰 →
> 5) mock 데이터 seam(★핵심) → 6) 화면별 매핑 → 7) 헌법 게이트 → 8) 빌드 순서 →
> 9) FE 세션에 붙여넣을 프롬프트.

---

## 1. 이름 규칙 (중요)

- **제품 표시명 = `K-Bap`** (로고/타이틀/스토어). 기술 슬러그 = `kbap`.
- 목업(디자인 Direction G)은 브랜드명이 **"KLens"** 로 되어 있다 → **전부 `K-Bap`으로 치환**.

| 목업에 나오는 것 | FE에서 쓸 것 |
|---|---|
| 헤더 마크 `K` + `KLens` | `K-Bap` 워드마크 (마크 글자는 `K` 유지 가능) |
| "Welcome to KLens" 등 문구 | "Welcome to K-Bap" (단, 문구는 i18n 키로 — 2번/7번 참조) |
| 파일/컴포넌트명 `KLens*` | 자유롭게 `KBap*` 또는 기능명으로 |

---

## 2. 출처 (SSOT)

| 무엇 | 위치 |
|---|---|
| **high-fi 목업** | Claude Design 프로젝트 `43cc8ad0-38d8-4afa-a66f-a7412eae2a49`, 파일 `KLens Hi-Fi (Direction G).html` + `hifi-*.jsx` + `hifi-g.css` + `screenshots/*hifi-g*` |
| **API 계약(SSOT)** | spec 레포 `specs/001-personalized-menu-mvp/contracts/openapi.yaml` |
| **헌법(제약)** | spec 레포 `.specify/memory/constitution.md` |
| **디자인 브리프** | spec 레포 `specs/001-personalized-menu-mvp/design-brief.md` |
| **유저 스토리/FR** | spec 레포 `specs/001-personalized-menu-mvp/spec.md` |

원칙: **계약·디자인이 바뀌면 spec 레포에서 먼저** 바꾼다(헌법 VI). FE는 소비만.

---

## 3. 목업은 웹, 우리는 React Native — 변환 규칙

목업은 `<div>`/CSS/`<IOSDevice>` 기반 **웹 코드**다. **복붙 금지.** 시각적 정답지로만 쓰고 RN으로 재구현.

| 목업(웹) | RN 구현 |
|---|---|
| `<div>` / `<header>` / `<nav>` | `<View>` |
| 텍스트 | `<Text>` (RN은 텍스트가 반드시 `<Text>` 안) |
| `hifi-g.css` 클래스 | `StyleSheet.create` 토큰(4번)으로 변환 |
| `<IOSDevice>` 베젤 프레임(402×874) | **무시** — 기기 자체가 화면 |
| CSS `radial-gradient` 배경 | `expo-linear-gradient` |
| SVG glyph(icons.jsx) | `react-native-svg`로 동일 패스 이식 |
| `:hover` / 마우스 | 터치(`Pressable`)로 대체 |

---

## 4. 디자인 토큰 (Direction G "Bright & Appetizing")

`hifi-g.css`의 `:root` 값. RN 테마 상수로 그대로 옮긴다. **위험도 4색은 의미 고정 — 변경 금지.**

```ts
// kbap-fe/lib/theme.ts
export const color = {
  primary: '#E2580C', primaryPress: '#c44a08',
  accent: '#0E9AA7',
  surface: '#FCF5EF', surface2: '#F7ECE1',
  panel: '#FFFFFF', card: '#FFFFFF',
  ink: '#2A211B', ink2: '#7C6B5E', ink3: '#B0A395',
  hair: '#EFE5D9', line: '#E7DACB',
  // 위험도 4상태 (FIXED semantic — 헌법 III)
  riskSafe: '#2f8f5b', riskCaution: '#d28a12', riskDanger: '#cf3a2c', riskUnable: '#5b6470',
} as const;
export const radius = { lg: 20, sm: 15, xs: 11 } as const;
export const font = {
  display: 'Baloo2',          // 디스플레이/타이틀
  body: 'NunitoSans',         // 본문 (영어 우선)
  ko: 'NotoSansKR',           // 한국어(사장님 확인 등 place=ko)
} as const;
```
> 폰트는 `expo-font`로 로드(Baloo 2, Nunito Sans, Noto Sans KR). 나머지 그림자·간격은 `hifi-g.css` 참고.

---

## 5. ★ Mock 데이터 Seam — 드리프트 방지의 핵심

**목표: 지금은 mock JSON, 다음 주 실제 API. 화면 코드는 한 줄도 안 바뀌게.**

원리: 화면은 데이터 출처를 모른다. **계약 타입을 반환하는 훅**만 호출한다. 훅 뒤에서 mock↔real을 토글.

```
[화면]  →  [useXxx() 훅 (계약 타입 반환)]  →  MOCK_MODE ? mock JSON : API client
```

### 5-1. 계약에서 타입 생성 (다음 주 sync 전까지 임시 가능)
```bash
# 다음 주: spec 레포 openapi.yaml → 타입 생성 (sync 스크립트는 그때 셋업)
npx openapi-typescript <openapi.yaml> -o kbap-fe/lib/api/schema.ts
```
지금 당장 타입이 없으면, **계약의 스키마 이름과 동일하게** 임시 `types.ts`를 손으로 만들되,
필드명을 openapi.yaml과 **정확히 일치**시킨다(다음 주 생성 타입으로 교체).

### 5-2. mock JSON은 계약 타입을 만족해야 함 (컴파일러가 드리프트를 잡음)
```ts
// kbap-fe/lib/mocks/foods.ts
import type { components } from '../api/schema';
type FoodCard = components['schemas']['FoodCard'];
export const MOCK_FOODS: FoodCard[] = [ /* 타입 안 맞으면 빌드 에러 */ ];
```

### 5-3. 훅 seam
```ts
// kbap-fe/lib/data/useHome.ts
export const MOCK_MODE = true;   // 다음 주 false
export function useHome() {
  if (MOCK_MODE) return { data: MOCK_HOME, isLoading: false };
  return useQuery({ queryKey: ['home'], queryFn: () => api.get('/home') }); // 다음 주
}
```
화면은 `const { data } = useHome()` 만 쓴다. → 다음 주 `MOCK_MODE=false` 한 줄로 실제 연결.

> **왜 이렇게?** mock 데이터를 계약 타입에 묶어두면, 네가 일일이 안 봐도 "목업과 실제 API가
> 다른 모양"이 컴파일 단계에서 막힌다. 이게 폴리레포에서 FE/BE 어긋남을 막는 장치다.

---

## 6. 화면별 매핑 (목업 → 라우트 → 계약 → mock)

각 화면은 `hifi-*.jsx` + 스크린샷 참조. 라우트는 Expo Router 기준. 타입은 openapi.yaml `components/schemas`.

| 화면 | 목업 파일 | 라우트(제안) | 계약 엔드포인트 / 타입 | mock 메모 |
|---|---|---|---|---|
| **앱 셸/탭바** | `hifi-shell.jsx` | `app/(tabs)/_layout.tsx` | — | **5-탭**: Home·Food·**Scan(중앙 FAB)**·**Community(잠금)**·Profile. Scan FAB→카메라. Community는 "coming soon" 오버레이(phase 2). |
| **온보딩** | `hifi-onboarding.jsx` | `app/(onboarding)/*` | `POST /auth/*`, `PATCH /me`(`User`,`DietaryRestriction`), `POST /me/consent` | 5-스텝 진행바(`TopBar seg/of`). 제약 **건너뛰기 불가**. 가입은 mock 토큰. |
| **스파이스 선택** | `SpicePicker-Explore` / `Spice Picker.html` | 온보딩 내 or 프로필 | `User.spiceTolerance`(0–10) | 매운맛 선호 입력 UI. |
| **홈** | `hifi-home.jsx`, `klens-home.jsx`, `HomeAlt-HiFi-G` | `app/(tabs)/home` | `GET /home` → `recent[]` + `recommended[]`(위험도 포함) | **빈 상태**(신규 0건) mock도 준비(`screenshots/home-empty-rec.png`). |
| **카메라/스캔** | `hifi-camera.jsx` | `app/(tabs)/scan` (FAB) | `POST /scan/menu` → `ScanResultItem[]`(키=`rawText`) | 온디바이스 OCR은 실제 ML Kit 필요 → MVP UI 단계선 **mock OCR 텍스트**로 오버레이 시연. **원본↔번역 토글** 포함(FR-037). |
| **음식 탐색** | `hifi-food.jsx` | `app/(tabs)/food` | `GET /foods` → `FoodCard[]` | 검색 오버레이(shell B2). 식당 개념 없음(음식 단위). |
| **음식 디테일** | `hifi-detail.jsx` | `app/(tabs)/food/[id]` | `GET /foods/{id}` → `FoodDetail`,`IngredientRisk[]`,`RiskBasis` | 성분 **위험순** 정렬, `riskBasis` 노출. '주의' 성분 탭→사장님 확인. |
| **사장님 확인** | `hifi-owner.jsx` | `features/owner-confirm` (모달/스택) | `GET /foods/{id}/owner-confirmation` → `OwnerConfirmation`(place=ko) | 한국어 문구 카드. `menuNameKo` == 스캔 메뉴명(SC-006). 한글 폰트(Noto Sans KR). |
| **리뷰 작성** | `hifi-review.jsx` | `features/reviews/compose` | `POST /foods/{id}/reviews` → `Review` | 별점 1–5 정수(`Stars`). |
| **프로필** | `hifi-profile.jsx` | `app/(tabs)/profile` | `GET /me`,`GET /me/reviews`,`GET /me/ranking`(`Ranking` tier), `DELETE /me` | 랭킹 티어 표시. 계정삭제 확인 모달. |
| **상태(빈/로딩/에러)** | `hifi-states.jsx` | 공통 컴포넌트 | — | 모든 화면 공통 빈/로딩/에러. `States.html` 참조. |

### 공유 컴포넌트(키트 → RN 재구현)
`hifi-kit.jsx` / `icons.jsx` / `components.jsx` 기준:
- `Header`(브랜드+검색+알림) · `SubHeader`(뒤로+타이틀) · `TabBar`(5-탭+FAB) · `Btn`(variant/sm) ·
  `TopBar`(온보딩 진행바) · `ObTitle` · `Stars`(별점) · **`RiskMark`**(위험도 4상태 SVG: safe=원+체크 / caution=삼각+! / danger=팔각+X / unable=마름모+?) · `Icon*` 세트.
- **전부 SVG**(`react-native-svg`). **기본 이모지 절대 금지**(헌법).

### 헤더는 무조건 scroll-aware sticky (필수)
모든 스크롤 화면의 상단 헤더(`Header`/`SubHeader`)는 **scroll-aware sticky header**로 구현한다:
- 스크롤해도 **상단 고정**(sticky), 콘텐츠는 헤더 아래로 흐른다.
- 스크롤 위치에 따라 **축약/그림자 전환**: 최상단=투명/큰 타이틀, 스크롤 시작 시 배경 채움 +
  하단 hairline/그림자(`--sh-1`) + (해당 시) 타이틀 축약. `Animated`/`react-native-reanimated`의
  scrollY로 보간. iOS large-title 느낌.
- 구현 권장: 화면별 `ScrollView`/`FlatList`의 `onScroll`(또는 reanimated `useAnimatedScrollHandler`)로
  공유 `StickyHeader` 컴포넌트의 상태를 구동. 화면마다 재발명하지 말고 **공유 컴포넌트 1개**로.

---

## 7. 헌법 게이트 (구현 중 상시 체크)

- [ ] **기본 이모지 0** — 모든 아이콘은 SVG(`RiskMark`/`Icon*`). 🍜🚫 같은 유니코드 이모지 금지.
- [ ] **i18n 하드코딩 0** — 모든 reader 텍스트는 `t('key')`. MVP는 영어 리소스 우선. (place=ko 문구는 데이터/`OwnerConfirmation`에서 옴, UI 하드코딩 아님.)
- [ ] **위험도 4상태 일관** — 색/실루엣/글리프 고정. safe를 임의로 표시하지 않기(데이터가 safe일 때만).
- [ ] **음식 중심** — '식당' 화면/필드 없음.
- [ ] **안전 고지** — 위험 화면에 상시 고지 배너(FR-030).

---

## 8. 빌드 순서 (실수 최소화)

1. **기반**: Expo 프로젝트, 폰트 로드, `lib/theme.ts`(토큰), `react-native-svg`, Expo Router.
2. **디자인 시스템**: `RiskMark` + `Icon*` + `Btn`/`Header`/`SubHeader`/`TabBar`/`TopBar`/`Stars`/상태 컴포넌트.
3. **mock seam**: `lib/api/types`(계약 일치) + `lib/mocks/*` + `lib/data/useXxx()` 훅.
4. **화면 1개씩** (목업 대조하며): 앱 셸/탭바 → 온보딩 → 홈 → 카메라(mock) → 음식 탐색 → 디테일 → 사장님 확인 → 리뷰 → 프로필 → 상태.
5. 각 화면 완성 시 **해당 목업 스크린샷과 1:1 대조**.

> **한 번에 다 시키지 말 것.** 화면 단위로 만들고 대조해야 어긋남을 작게 잡는다.

---

## 9. FE 세션에 붙여넣을 프롬프트 (템플릿)

```
너는 kbap-fe(React Native + Expo, TypeScript) 레포에서 K-Bap 앱의 UI를 구현한다.
지금은 백엔드 API 없이, high-fi 목업대로 화면을 mock 데이터로 보이게 만드는 단계다.

[필독]
- 이 문서 전체: specs/001-personalized-menu-mvp/fe-handoff.md (spec 레포)
- API 계약(데이터 모양 SSOT): .../contracts/openapi.yaml
- 헌법(제약): .specify/memory/constitution.md
- 목업: Claude Design 프로젝트, "KLens Hi-Fi (Direction G)" + hifi-*.jsx + hifi-g.css

[규칙]
1. 목업은 웹 코드 → 복붙 금지, RN으로 재구현(핸드오프 3번 표).
2. 브랜드명 KLens → K-Bap 치환.
3. 데이터는 useXxx() 훅으로만 접근, 훅 뒤에 mock JSON(계약 타입 일치). MOCK_MODE=true.
4. 기본 이모지 금지(SVG만), 모든 텍스트 i18n, 위험도 4색 고정.
5. 디자인 토큰은 핸드오프 4번(theme.ts) 사용.

[이번 작업]
"<화면 이름> 화면"을 구현해줘. 목업 파일 hifi-<x>.jsx 참조.
끝나면 어떤 mock 타입/필드를 썼는지, 목업과 다른 점이 있으면 알려줘.
```

---

## 10. 결정 기록 (해소됨)

- ✅ **네비 5-탭 + Community(잠김)**: Scan은 중앙 FAB, Community 탭은 phase 2 예약(잠금) — plan/tasks T019 반영 완료.
- ✅ **알림(notifications) 패널**: MVP에 **정적 UI로 구현**(hifi-shell 그대로). 푸시 알림 기능은 2차.

## 11. 진행 중 — API 계약 재조정 (FE는 무관)

BE 실제 구현(`meogo.handev.site` swagger)이 우리 `openapi.yaml`과 일부 다름(경로/래퍼/enum/음식 식별).
**현재 재조정 논의 중.** FE는 mock 데이터로 UI만 만드는 단계라 **이 논의와 독립** — 그대로 진행.
단, mock 타입은 "재조정 중인 계약"이므로 **음식 식별자(foodId vs menuName) 같은 미확정 필드에
화면 로직을 강하게 묶지 말 것**. UI 렌더링에는 영향 없음. 확정 후 sync 시 타입 교체.

---

## 12. 작업 진행 방식 — 체크리스트 기반 (필수)

큰 작업을 한 세션에 통째로 던지면 컨텍스트가 비대해져 실수가 는다. 그래서 **진행 상황을
파일로 남기고, 세션이 그걸 보며 한 단위씩** 진행한다. 두 겹으로 운영:

1. **`kbap-fe/PROGRESS.md`** (durable, 커밋됨) — FE 세션이 **가장 먼저 생성**한다. 아래 빌드
   순서(§8)를 체크박스로 옮기고, 각 항목을 `[ ]`→`[~]`(진행중)→`[x]`(완료)로 갱신한다.
   세션이 리셋돼도 여기서 이어간다. 항목 완료 시 커밋.
2. **세션 내 todo**(Claude Code 기본 todo) — 현재 항목을 더 잘게 쪼갠 단기 추적.

규칙:
- **한 번에 1개 단위만** 진행(예: "디자인시스템 컴포넌트", 다음 "앱 셸", 다음 "온보딩 화면").
  여러 화면을 한 프롬프트에 몰지 않는다.
- 각 단위 시작 시 PROGRESS.md에서 해당 항목 `[~]` 표시 + 무엇을 할지 1줄 메모, 끝나면 `[x]` +
  결과/이탈점 1줄. **다음 단위로 넘어가기 전에 목업과 1:1 대조**.
- 막히거나 결정이 필요하면 PROGRESS.md에 `❓`로 적고 사용자에게 질문(임의 가정 금지).

### PROGRESS.md 초기 템플릿(FE 세션이 생성)
```markdown
# kbap-fe Build Progress
> 규칙: 한 번에 1개. [ ]→[~]→[x]. 각 화면 완료 시 목업 대조 + 커밋.

## 기반
- [ ] Expo(TS+expo-router) 초기화 + 의존성
- [ ] lib/theme.ts (토큰/폰트)
- [ ] mock seam (types / mocks / useXxx, MOCK_MODE=true)
## 디자인시스템
- [ ] RiskMark + Icon* (SVG)
- [ ] Btn / StickyHeader(scroll-aware) / SubHeader / TabBar / TopBar / Stars / 상태 컴포넌트
## 앱 셸
- [ ] 5-탭 + Scan FAB + Community(잠김) + 검색/알림 패널
## 화면 (각: 목업 대조 후 [x])
- [ ] 온보딩(+스파이스)  - [ ] 홈  - [ ] 카메라/스캔(mock OCR)  - [ ] 음식 탐색
- [ ] 음식 디테일  - [ ] 사장님 확인  - [ ] 리뷰 작성  - [ ] 프로필  - [ ] 상태(빈/로딩/에러)
```

---

## 13. 스캔 스파이크 — FE↔BE 실연결 + 온디바이스 OCR (카메라 화면 단계에서만)

> ⚠️ **기반/디자인시스템/앱 셸 단계에선 무시.** 이 절은 **카메라/스캔 화면을 만들 때** 적용한다.
> 목적: 스캔 기능(앱 핵심)을 실제 BE에 한 번 붙여 흐름 + OCR 패키지를 검증하는 **스파이크**다.
> 이 API는 **확정 계약이 아니라 잠정**이다(재조정 예정). 그래도 mock seam 덕에 갈아끼우기 쉽다.

### 13-1. 한 훅만 실연결
- `useScan()` 만 `MOCK_MODE=false`로 실제 BE에 연결. 나머지 화면 훅은 mock 유지.
- BE 베이스: `https://meogo.handev.site` · 응답은 `BaseResponse{success,payload,message}` 래퍼.

### 13-2. BE 실제 모양 (이대로 코딩, 우리 옛 계약과 다름)
```
POST /api/v1/menu-scans
req:  { items: [{ itemId:int, rawMenuName:string, boundingBox:{x,y,width,height} }] }   // 1~100개
res:  payload { scanId, results: [{ id, itemId, riskLevel, reason }] }
      riskLevel: "SAFE" | "CAUTION" | "DANGER" | "UNKNOWN"

GET /api/v1/foods/detail?menuName=<한국어명>&lang=<reader lang>
res:  payload { name, imageRef, ingredients:[{ name, iconRef, inclusionPercent, riskStatus }] }
```
- **enum 매핑은 어댑터에서**: BE `UNKNOWN` → 앱 내부 `unable`(판정불가). 화면은 내부 4상태만 안다.
- 스캔 응답에는 **번역 이름이 없다** → 오버레이 전략은 13-4.

### 13-3. 온디바이스 OCR — dev build 필수
- `@react-native-ml-kit/text-recognition`은 네이티브 모듈 → **Expo Go에서 안 돈다.**
- **EAS dev build(custom dev client)** 로 빌드해서 실기기/시뮬레이터에서 테스트. (`npx expo run:ios` 또는 EAS dev build)
- OCR로 텍스트 + bounding box 추출 → box는 **기기 보관**(오버레이용), 텍스트만 `menu-scans`로 전송.

### 13-4. 오버레이 전략 (객관적 권장 = A, 느리면 B)
- **A (기본)**: 스캔 응답의 `riskLevel`로 박스에 **위험도 색만 즉시** 표시. 사용자가 박스를 **탭하면** 그때 `GET /foods/detail`로 번역명+성분. → 첫 결과 가장 빠름. **전부 미리 부르지 말 것.**
- **B (이름이 꼭 보여야 하면)**: BE에 "스캔 응답에 displayName 1필드 추가" 요청. → 1호출로 이름+위험도.

### 13-5. 반드시 테스트 (안전, 헌법 III)
- 테스트 케이스에 **미등록 음식**(예: "신상 화채")을 꼭 넣어, 스캔 결과가 `UNKNOWN`(→unable)으로
  나오고 **절대 SAFE가 아님**을 확인(false-safe 0, SC-003). SAFE로 나오면 BE에 즉시 보고.
```
