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
> 폰트는 `expo-font`로 로드. **MVP 9개 언어 출시(B안)** 이므로 라틴만으론 부족 — **다중 스크립트
> 폰트 필수**: 라틴=Baloo 2/Nunito Sans, **CJK=Noto Sans SC/TC/JP**, **태국=Noto Sans Thai**,
> **키릴=Nunito Sans/Noto**(+ place=ko 표시용 Noto Sans KR). 로케일별 폰트 폴백을 매핑해 **글자
> 깨짐(두부 □) 0**. 나머지 그림자·간격은 `hifi-g.css` 참고. (상세: spec.md Session 2026-06-29, tasks T071)

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
- [ ] **i18n 하드코딩 0** — 모든 reader 텍스트는 `t('key')`. **MVP 9개 언어 출시(B안)**: en·zh-Hans·zh-Hant·ja·vi·id·th·ru·es 로케일 리소스 9개(영어로 먼저 짠 뒤 9개 채움, T070). (place=ko 문구는 데이터/`OwnerConfirmation`에서 옴, UI 하드코딩 아님.)
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

### 13-3. 온디바이스 OCR — dev build 필수 + 환경별 가능 여부
- `@react-native-ml-kit/text-recognition`은 네이티브 모듈 → **Expo Go에서 안 돈다.**
- **🔴 iOS 시뮬레이터에서 ML Kit OCR은 안 돌 가능성이 높다**(특히 애플 실리콘 arm64). 갤러리로
  이미지를 넣어줘도 **OCR 단계가 시뮬레이터에서 실패**할 수 있다. → **신뢰 가능한 OCR 검증 =
  실물폰 dev build.** 에뮬레이터로 보려면 **Android 에뮬레이터**가 iOS 시뮬레이터보다 호환이 낫다.
- 환경별 정리:
  | 환경 | 앱 실행 | 갤러리 선택 | **OCR** | BE 왕복+오버레이 |
  |---|---|---|---|---|
  | 웹/Expo Go | △/✅ | — | ❌(네이티브 없음) | ✅(sample scan) |
  | iOS 시뮬레이터 | ✅ | ✅ | ❌/불확실 | ✅ |
  | Android 에뮬레이터 | ✅ | ✅ | △(대체로 ✅) | ✅ |
  | **실물폰 + dev build** | ✅ | ✅ | **✅** | ✅ |
- **OCR 모듈은 lazy import**(호출 직전) 또는 Platform 가드 — 모듈 없는 환경에서 화면 진입만으로
  크래시하지 않게. (sample-scan·갤러리+BE 등 OCR 무관 경로 보호)
- OCR로 텍스트 + bounding box 추출 → box는 **기기 보관**(오버레이용), 텍스트만 `menu-scans`로 전송.
- **bbox 정렬**: 표시되는 이미지의 실제 크기·스케일(contain/cover) 기준으로 정규화/매핑.
  카메라 프리뷰 경로와 갤러리 이미지 경로의 스케일 처리를 분리 검증(letterbox 어긋남 주의).

### 13-3b. 갤러리 가져오기 (목업 D1 좌하단)
- `expo-image-picker`, `allowsMultipleSelection: false`(**1장만**). 선택 URI → `TextRecognition.recognize(uri)`
  → 동일 파이프라인. 카메라 셔터(중앙)·플립(우)와 함께 하단 행 구성.

### 13-3c. 실물폰 테스트 (TestFlight 불필요)
- **가장 간단(Android 폰)**: USB 디버깅 켜고 `npx expo run:android` → dev build가 폰에 바로 설치.
  계정/서명 불필요. (또는 `eas build -p android --profile development` → APK 링크로 설치)
- **iPhone + Mac**: `npx expo run:ios --device`(USB) → Xcode가 무료 Apple ID로 서명·설치(7일 재서명).
- 최초 빌드만 무겁고, 이후 JS 변경은 Metro로 핫리로드(재빌드 X). **TestFlight은 외부 배포용이라 개발 테스트엔 불필요.**

### 13-4. 오버레이 전략 (객관적 권장 = A, 느리면 B)
- **A (기본)**: 스캔 응답의 `riskLevel`로 박스에 **위험도 색만 즉시** 표시. 사용자가 박스를 **탭하면** 그때 `GET /foods/detail`로 번역명+성분. → 첫 결과 가장 빠름. **전부 미리 부르지 말 것.**
- **B (이름이 꼭 보여야 하면)**: BE에 "스캔 응답에 displayName 1필드 추가" 요청. → 1호출로 이름+위험도.

### 13-5. 반드시 테스트 (안전, 헌법 III)
- 테스트 케이스에 **미등록 음식**(예: "신상 화채")을 꼭 넣어, 스캔 결과가 `UNKNOWN`(→unable)으로
  나오고 **절대 SAFE가 아님**을 확인(false-safe 0, SC-003). SAFE로 나오면 BE에 즉시 보고.

### 13-6. 현재 BE 상태 (2026-06-29 테스트 기준) — 가정 금지
- **BE 스캔은 아직 mock(스텁)이다.** 위험도가 음식이 아니라 **itemId 순서로 순환**해서 나온다
  (0→SAFE, 1→CAUTION, 2→DANGER, 3→UNKNOWN, reason에 `"mock:"` 접두사). 실제 카탈로그 매칭·
  개인화 위험도는 미구현. → **FE는 "itemId=위험도" 매핑을 절대 가정하지 말고**, 응답 `riskLevel`을
  그대로 그린다. (지금은 4상태 UI를 모두 시험할 fixture로만 활용.)
- **detail 미등록 응답**: `GET /foods/detail`에 카탈로그에 없는 이름을 주면
  `{ success:false, payload:null, message:"..." }`(한국어)를 준다. FE 처리 규칙:
  ① **`success` 플래그로 분기**(HTTP 상태 아님), payload null 방어.
  ② **BE `message`를 사용자에게 표시 금지** — FE 자체 i18n "정보 없음/판정불가(unable)" 상태로 렌더.
- 안전(false-safe) 검증은 **BE 실제 판정 탑재 후 재테스트** 대상(된장찌개=콩 개인화, 미등록 음식 포함).

### 13-7. 스파이크 결과 (2026-06-30)
- ✅ **OCR 실물폰 작동 확인**: EAS dev build(실기기)에서 `@react-native-ml-kit/text-recognition`가
  텍스트 + bounding box를 실제로 추출함(김치찌개/된장찌개/순두부찌개 등 한국어 인식 양호).
- ✅ **BE 왕복 + 4상태 오버레이**: "Run sample scan"으로 검증됨.
- ✅ **bbox 정규화 버그 해결(2026-06-30) → 전 구간 통과**: 근본 원인 = bbox를 **포인트 크기
  (886×1920)로 정규화**했는데 ML Kit frame은 **픽셀 좌표**라 정확히 ×3(devicePixelRatio)만큼 초과
  (x·y 모두 ~3.0). 수정 = OCR이 처리한 이미지의 **실측 픽셀 크기(2658×5760)로 정규화**. 이후 모든
  bbox가 [0,1]에 들어오고 BE가 **200 success** 반환. **카메라→OCR→bbox→BE→오버레이 전 구간
  실물폰(EAS dev build)에서 검증 완료.** (교훈: ML Kit bbox는 픽셀 좌표 → 반드시 이미지 픽셀 크기로 정규화)
- ⏳ **남은 것**: ① 위험도는 아직 BE mock(itemId 순환) — 실제 카탈로그 매칭/개인화 후 재검증.
  ② 잡음/세그멘테이션(T072) — 화면-of-화면 촬영이라 키보드·브라우저·가격·원산지·로마자 파편까지
  다 잡힘. **실제 메뉴판 사진으로 한 번 더 테스트**해 현실 잡음 수준 확인 필요. ③ OCR 로마자 혼입
  ("· 김치찌개 Kmch lioae")은 매칭이 한국어 기준이라 무해하나, 표시 정제는 추후.
- 🟡 **메뉴 항목 분리/잡음 필터는 향후 과제(T072)**: 스파이크 단계선 만들지 않음. 잡음 텍스트는
  카탈로그 매칭에서 자연히 UNKNOWN(unable)로 떨어짐. **단 "UNKNOWN 일괄 숨김 금지"**(미등록 음식도
  UNKNOWN — 헌법 III). dish명 vs 원산지/카테고리/가격 분리는 안전 민감 과제로 별도 진행.
```

## 14. 스캔 세그멘테이션 T072 — 확정 설계 (2026-07-02, 실제 OCR 로그로 검증)

실제 메뉴판 OCR 로그(58줄, 3컬럼 그리드 + 2컬럼 리스트 혼재)로 검증한 설계. **핵심 원칙:
안전(위험도)은 "요리명" 하나에만 걸고, 어려운 기하(그룹핑)는 안전과 분리해 best-effort로 강등.**

### 14-1. 로그에서 확인된 사실 (설계 근거)
- **ML Kit Block은 절반만 묶어줌**: 로마자+설명은 한 Block(block9=`Bibimbap`+설명), 그러나
  한글 요리명(block8=`비빔밥`)·가격(block10=`W12,000`)은 별도 Block. 요리 1개 ≈ 3블록.
- **한 메뉴에 레이아웃 2종 혼재**: 식사류=3컬럼 그리드(이름→로마자→설명→가격 세로 쌓임),
  면류·음료=2컬럼(이름 | 가격 같은 행). → 단일 위치 규칙 불가.
- **요리명은 깨끗, 설명은 엉망**: `비빔밥·떡볶이·김치볶음밥·갈비구이·김치전·해물파전·라면·냉면`
  전부 정확 인식. 설명은 OCR 오타 다발(`열치국수`=멸치국수, `되지고기`=돼지고기, `플깃한`=쫄깃한).
  → **매칭은 짧은 요리명으로만. 설명은 매칭에 절대 쓰지 않음.**

### 14-2. 알고리즘 (FE 온디바이스)
1. **줄 분류** (정규식/키워드, 레이아웃 무관):
   - 가격: `/[Ww₩]\s?\d[\d,]*/` (W12,000 / w8,000 / ₩)
   - 원산지: `원산지|국산|수입산|산\)` 포함
   - 섹션헤더: 짧고 `~류`로 끝나거나 알려진 카테고리(음료 등)
   - 로마자/영문: 라틴 문자 위주 (표시용 영문명 후보)
   - 설명: 긴 한글 문장(대략 ≥12자 또는 공백/쉼표 다수)
   - **요리명**: 위 어디에도 안 걸리는 **짧은 한글** → 매칭 대상
   - 잡음: 단일문자(`Q &`), 파일확장자(`.jpg`), 기능키(`F4~F8`), 화면 가장자리(y<0.17 상단 / y>0.81 하단)의 라틴 UI 텍스트
2. **요리명만 BE 전송** (설명 전송 금지). BE는 **퍼지 매칭 필수**(OCR 오타 흡수).
3. **부가정보 붙이기 (best-effort, 안전 무관)**: 각 요리명 bbox 기준 **방사형 최근접
   (radial nearest-neighbor)** 으로 가장 가까운 가격·로마자를 연결. 그리드(가격 아래)·리스트
   (가격 같은 행 오른쪽) 둘 다 최근접이 정답. **틀려도 위험도는 이름에 직접 붙으므로 안전 피해 0.**
4. **표시 = 오버레이 마커 + 리스트 토글** (사용자 확정):
   - 오버레이: 위험도색 마커를 **요리명 bbox 위치**에 (그룹핑 불필요 → 레이아웃에 견고).
   - 토글: Original / Risk(마커) / **List**(파싱된 목록: 요리명·영문·가격·위험배지).
   - 마커/행 탭 → 음식 상세.
5. **미매칭 한글 요리명 = 판정불가(unable)로 표시**, 리스트 **맨 아래 정렬**. **절대 숨기지 않음**
   (미등록 음식과 잡음은 결과로 구분 불가 → 구조적 비음식만 버리고, 그럴싸한 한글 요리명은 항상 보임. 헌법 III).

### 14-3. 안전 불변식 (T072에서 반드시)
- 거르기는 **줄 타입(구조)** 로만. **매칭 결과(UNKNOWN 여부)로 거르지 않는다.**
- 빈 프로필 사용자에겐 스캔 결과에서도 개인화 '안전' 금지 → `personalRisk()` 경로 그대로 통과.
- 오버레이 마커 겹침: 마커는 점/작은 배지라 겹침 적음. 겹치면 살짝 오프셋 또는 List 토글로 회피.

## 15. 랭킹 디테일 화면 (신규, 2026-07-02 설계 확정)

프로필의 랭킹(Rosette ladder) 탭 → 랭킹 디테일. **P3라 정적 화면**(GET /me/ranking 데이터를 거의
그대로 표시). 계약: `Ranking`(tier/level/score/nextTier/pointsToNext/**breakdown**), `RankingFactor`(count/points).

### 15-1. 라우트·데이터
- 라우트: `app/profile/ranking` (또는 `(tabs)/profile/ranking`). 프로필 랭킹 카드 탭으로 진입.
- 훅: `useRanking()` + mock(계약 `Ranking` 타입 만족). MOCK_MODE seam 유지.

### 15-2. 화면 구성 (위→아래, 풀 버전)
1. **현재 등급 히어로**: 큰 Rosette(SVG) + 등급명(i18n) + level + 한 줄 플레이버.
2. **다음 등급까지**: 진행바 + "X점 남음 → [nextTier 등급명]" (pointsToNext/nextTier). 최고 등급이면 "최고 등급 달성".
3. **점수 내역**(breakdown): 리뷰 / 음식 다양성 / 스캔 — 각 `count` + `points` 표시. ("리뷰 하나 더 = +10점" 감각).
4. **전체 등급 사다리**: 7단계 전부. 현재 강조 / 지난 등급 unlock / 다음부터 잠금 + 진입 점수.
5. **등급 올리기**: "리뷰 쓰기"·"메뉴 스캔" 딜링크(각각 리뷰 작성/스캔 라우트).

### 15-3. 7단계 등급 (안정키 → i18n)
`newcomer`/`taster`/`explorer`/`regular`/`gourmet`/`kfood_master`/`korean_at_heart`.
누적점수 구간: 0 / 30 / 80 / 180 / 350 / 600 / 1000. 가중치(BE 산출, FE는 표시만):
리뷰=10, 고유음식=5, 스캔=2. **등급명은 tier 키로 i18n 매핑**(BE가 번역명 안 줌, 안정키만).

### 15-4. 규칙
- 이모지 금지(Rosette·아이콘 전부 SVG). 모든 텍스트 i18n 키(영어만 채움, 등급명도 키로).
- scroll-aware sticky 헤더(SubHeader 뒤로+타이틀). tsc 0 + web 대조.
- 위험도 렌더 없음(personalRisk 무관).
