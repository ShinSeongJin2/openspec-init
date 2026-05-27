# OpenSpec E2E Memory Index

Cross-suite E2E knowledge — pitfalls, workarounds, and quirks discovered while
building suites under `openspec/specs/*/e2e/`. The E2E skill at
`.claude/skills/e2e-tests/SKILL.md` reads this index in Phase A and writes
back to it in Phase F.

## 사용 방법

1. Phase A에서 이 인덱스를 먼저 읽습니다. 각 항목의 `applies-to`가 현재 작업
   중인 스펙의 스택과 일치하면 해당 메모리 파일을 읽습니다.
2. 메모리를 적용하기 전에 항상 최신 여부를 확인합니다. 파일 경로, 이미지
   태그, 명령어는 코드 변경으로 부패할 수 있습니다. 확인할 수 없으면 사실이
   아니라 힌트로 취급하고 현재 코드 상태를 우선합니다.
3. 메모리는 단문, 의미 단위, 프로젝트 고유 지식만 담습니다. 스킬은 일반론을,
   메모리는 이 저장소에서만 통용되는 함정을 담는 분담입니다.

## 메모리 추가 기준 (Phase F)

다음 조건을 **모두** 만족할 때만 메모리를 추가합니다:

- 이번 스위트에서 약 30분 이상을 잡아먹은 함정/우회/환경 문제였음
- 미래의 작업자가 코드/`git log`를 읽어 유도할 수 없는 지식임
- 스킬의 일반 워크플로에 포함하기에는 프로젝트 특수성이 강함

다음은 메모리에 적지 않습니다:

- 코드를 보면 알 수 있는 사항 (파일 경로, 함수 구조)
- 잘 알려진 명령어/관용구
- 이미 SKILL.md, OUTPUT_CONTRACT.md, TEMPLATES.md에 적힌 내용
- 한 번 발생하고 다시 재현될 가능성이 낮은 일회성 이슈

## 메모리 파일 포맷

```markdown
---
name: <short-kebab-case>
description: <한 줄 요약 — 이 메모리가 어떤 상황에 적용되는지>
applies-to:
  - <stack-tag-1>
  - <stack-tag-2>
last-verified: YYYY-MM-DD
metadata:
  type: pitfall | quirk | workaround | reference
---

# <제목>

<상황 설명 — 어떤 작업 중에 발생했는지>

## What works

<해결책 — 명령어, 패치, 우회 절차>

## Why

<원인 — 가능하면 1~2 문장으로>

## How to apply

- Triggered when: <적용 조건>
- Skip if: <적용 제외 조건>

Related: [[other-memory-name]]
```

## Index


