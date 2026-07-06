---
name: jira
description: >
  K-Bap Jira(KB) 이슈 생성·관리 규격. 이슈를 만들거나 수정할 때 반드시 발동 —
  본문에 Background + DoD(Definition of Done)를 넣고, 에픽 parent를 배정하고,
  라벨(BE/FE)·담당·스프린트를 규격대로 채운다. 쓰기 전 사용자 승인.
---

# K-Bap Jira 이슈 규격

## 대상/접속
- 사이트 `simhani1.atlassian.net` · cloudId `50957656-c97c-4cc0-b1ca-f209cef7d5c9` · 프로젝트 **KB**(team-managed).
- 도구: `mcp__atlassian__createJiraIssue` / `editJiraIssue` / `searchJiraIssuesUsingJql` / `getTransitionsForJiraIssue` / `transitionJiraIssue`.
- 정본 포맷 예시: **KB-50** (BE 이슈) — 새 이슈 쓸 때 이 구조를 따른다.

## 1. 본문(description) 규격 — 필수
모든 **작업(Task)** 이슈 description은 markdown으로 아래 2섹션을 반드시 포함:

```
## Background

* 목표: <이 작업이 뭘 달성하는가>
* 현재 상태: <지금 어디까지 돼 있나 / 관련 코드·이슈>
* 범위:
    * <해야 할 것 세부>
* 비범위: <이 이슈에서 안 하는 것>
* 의존: <선행 이슈/조건, 예: KB-66, BE Swagger>
* 다음 단계: <이 다음에 이어지는 이슈>

## DoD

- [ ] <검증 가능한 완료 조건 — 테스트/실행 결과/화면으로 확인되는 것>
- [ ] ...
```
- DoD는 "구현했다"가 아니라 **관측 가능한 결과**로 쓴다(예: "jest 통과", "기기에서 X 동작 확인", "응답에 커서 포함").

## 2. 에픽(parent) — 필수
- **모든 태스크는 에픽 parent를 가진다.** 기존 에픽에 맞으면 거기, 없으면 새 에픽 생성.
- **기능 축으로 묶는다(담당 축 아님) — BE·FE가 같은 에픽을 공유**한다.
- 설정: 생성 시 `createJiraIssue(parent: "KB-<epic>")`, 수정 시 `editJiraIssue(fields: {"parent": {"key": "KB-<epic>"}})`.
- 현재 에픽(기능):
  - KB-29 메뉴 스캔 · KB-30 메뉴 조회 · KB-61 메뉴 검색 · KB-32 리뷰 · KB-51 회원가입
  - KB-31 메뉴/재료 DB설계 · KB-42 도메인 모델 · KB-33 데이터 수집 파이프라인 · KB-34 AWS 인프라 · KB-35 예외 처리 · KB-39 기타
  - (FE 전용이 불가피하면) KB-65 실 API 통합(FE)
- 새 에픽 생성: `createJiraIssue(issueTypeName: "에픽", summary: "<기능명>")` → 반환 key를 parent로.

## 3. 라벨·담당·스프린트
- 라벨: `additional_fields: {"labels": ["FE"]}` (BE/FE/기타 중).
- 담당 accountId: FE=**김예진** `6275ff7c7de1ec00695c019e` · BE=**hanjongsim** `63e3cb0586a66a7cc7a9a1c5`.
- 스프린트 필드: `customfield_10020` = 스프린트 id(숫자). 현재 활성 계획 스프린트 "KB 2차 스프린트" = **35**.
- 이슈 타입: 에픽=`에픽`, 작업=`작업`. 요약 접두 `[FE]`/`[BE]`/`[기타]`.

## 4. 생성 절차
1. 요약([접두] 포함) 정한다.
2. Background + DoD 본문 작성(§1).
3. 에픽 parent 결정(기존 우선, 없으면 생성)(§2).
4. 라벨·담당·스프린트 채운다(§3).
5. **쓰기 전 목록·매핑을 사용자에게 보여주고 승인**받는다(다건은 배치로).

## 5. 상태 전환 + 완료 게이트
- 흐름: **할 일 → 진행 중 → 검토 중 → 완료**.
- 전환 id (KB, 전역): 할 일=`11` · 진행 중=`21` · 검토 중=`31` · 완료=`41`. (`getTransitionsForJiraIssue`로 재확인 가능, `transitionJiraIssue`로 전환)
- **완료 게이트 (필수)**: 구현자(FE/BE 세션)는 작업을 끝내면 **완료가 아니라 `검토 중`으로** 전환하고 §6 진행로그 댓글을 남긴다.
  - **`완료` 전환은 리뷰어(예진+Claude)만** — 코드/DoD 확인 후. 이때 필요하면 §7 BE-논의 플래그도 설정.
  - 이유: 안전 크리티컬(false-safe=0). 자기인증 완료는 드리프트·오검증의 원인. 리뷰어는 `검토 중` 이슈를 모아 한 번에 확인→완료/반려하여 병목을 줄인다.
  - ⚠️ 구현 세션(kbap-fe/kbap-be)은 이 레포 스킬을 자동으로 읽지 않으므로, **프롬프트에 "완료 시 검토 중 전환 + 진행로그 댓글" 지침을 명시**해 전달할 것.

## 6. 진행 로그 댓글 — 앞으로 필수 (소급 X)
- **언제**: ① 이슈 하나를 완료할 때, ② 하루 작업을 마칠 때, ③ 막힌(블락) 지점이 생겼을 때.
- **어디**: 해당 이슈에 `mcp__atlassian__addCommentToJiraIssue`로 댓글.
- **형식**(정본 = KB-28 종한 댓글):
  ```
  <YYYY-MM-DD>

  진행한 것

  * <그날 처리한 일 간략히>

  블락된 것

  * <막힌 것 / 없으면 "없음">

  진행할 것

  * <다음에 할 것 / 없으면 "없음">
  ```
- 날짜는 **실제 `date` 출력**에서 가져온다(추측 금지). 간략하게, 관측 가능한 사실 위주.
- ⚠️ **이미 지난 작업엔 소급 적용하지 않는다.** 이 규칙 도입(2026-07-06) 이후 처리분부터.

## 7. BE 논의/공유 플래그 — 앞으로 필수 (소급 X)
- 백엔드와 **회의로 논의**해야 하거나 **공유**가 필요한 이슈에는 **Flag(Impediment)를 설정**한다.
- 설정: `editJiraIssue(fields: {"customfield_10021": [{"value": "Impediment"}]})` (팀 표준 Flag 필드). 필드 id 불확실 시 대상 이슈에 `fields:["*all"]`로 조회해 Flag 필드 확인 후 설정.
- 플래그 건 이유는 §6 진행 로그 댓글에 한 줄 남긴다(무엇을 논의/공유할지).
- ⚠️ 소급 적용 X — 2026-07-06 이후 발생분부터.

## 원칙
- 이슈는 **혼자 읽고 착수 가능**하도록: Background로 맥락, DoD로 끝 기준. 링크로 정책 문서 참조(risk-display-policy, auth-onboarding-policy 등).
- 쓰기(생성/수정/전환)는 항상 사용자 승인 후. 다건은 매핑 표로 먼저 확인.
