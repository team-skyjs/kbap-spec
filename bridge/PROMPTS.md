# FE 작업 프롬프트 큐

> 운영: 커맨드 센터(스펙 레포 세션)가 항목 작성·수정, **새 항목은 최상단 추가**.
> FE 세션은 예진이 지목한 ID(지목 없으면 최상단 ⬜)를 처리하고 **상태만** 갱신:
> ⬜ 대기 → 🔄 착수 → ✅ 완료(커밋 해시 병기). 본문 수정은 커맨드 센터만.
> 완료 보고는 [REPORTS.md](REPORTS.md) 최상단에 같은 ID로 작성.

---

## [P-002] ⬜ KB-72 스캔 실연결 마무리 — 사진 전송(imagePath) + 가격 표시

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

### 안전 게이트 (스캔 = 안전 크리티컬)

- matched=false → UNKNOWN/unable 유지, 정렬 마지막, 숨김 금지 (기존 mapRisk 경로 보존)
- 서버 응답이 요청 items보다 적을 수 있음(비메뉴 항목 제외) — idx로 짝 맞추고, 응답에 없는 idx는 unable 처리

### DoD

- [ ] 새 계약(imagePath+items)으로 스캔 동작, 가격 표시(null 미표시 포함)
- [ ] 업로드 실패/발급 API 부재 시 안전한 폴백 (가짜 safe·크래시 없음)
- [ ] 테스트: 가격 매핑(정상/null) + 요청 body에 imagePath 포함 + idx 누락 항목 unable 처리
- [ ] tsc 0 · jest 통과 · 진행로그에 발급 API 존재 여부 기록

완료 시 이 항목 상태 ✅+커밋 해시로 갱신, 보고는 REPORTS.md 최상단에 [P-002]로.

## [P-001] ✅ KB-142·150 후속 — 상세 bookmarked 서버 필드 교체 + 맵기 실연결 — `3494917`

(브릿지 도입 전 채팅으로 전달·완료된 건 — 형식 참고용 사후 기록. 검토 게이트 통과 2026-07-15)

- Swagger 재배포 반영: FoodDetailResponse.bookmarked 기반으로 상세 저장 상태 교체(useIsBookmarked 제거), 낙관 토글이 목록+상세 캐시 동시 반전
- 맵기 TODO(KB-150) 3곳 실연결 (spicinessPreference), 스킵·해제는 필드 생략
- BE 질의 2건 기록: 미설정 표현(required int, 0≠미설정) / 해제(null) 전달 방법
