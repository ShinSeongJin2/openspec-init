# OpenSpec Init — Custom Claude Code Skill Pack

> 기존 코드에서 OpenSpec 스펙을 만들고, 그 스펙으로 E2E 테스트와 DOCX 사용자 매뉴얼까지 이어서 생성하는 커스텀 Claude Code skill pack.

---

## 이 skill pack은 무엇인가

Claude Code를 어느 정도 사용해 본 개발자가 **Spec-Driven Development(SDD)** 흐름을 가볍게 시작할 수 있도록 만든 OpenSpec 보조 도구입니다.

일반적인 흐름은 다음과 같습니다.

1. 기존 코드 폴더를 분석해 `openspec/specs/<spec-name>/spec.md`를 작성합니다.
2. 작성된 스펙을 기준으로 E2E 시나리오, Playwright 테스트, 실행 결과, 스크린샷, 커버리지 리포트를 만듭니다.
3. E2E에서 확보한 실제 화면 증거를 바탕으로 DOCX 사용자 매뉴얼을 생성합니다.

**핵심 특징**:

- OpenSpec 산출물을 한국어로 작성하도록 프로젝트 규칙을 제공합니다.
- 코드 요약이 아니라 외부에서 관찰 가능한 행위 계약 중심의 스펙을 만들도록 안내합니다.
- 스펙, E2E, 매뉴얼 산출물을 같은 `openspec/specs/<spec-name>/` 폴더 아래에 모아 추적성을 유지합니다.
- `openspec-code-to-spec`, `openspec-e2e`, `openspec-manual` 3개 스킬을 순차적으로 사용해 코드에서 매뉴얼까지 이어갈 수 있습니다.

---

## 설치

이 저장소는 OpenSpec 자체나 OpenSpec 기본 스킬/커맨드를 대체하지 않습니다. 이 pack의 스킬은 OpenSpec CLI를 호출하거나 OpenSpec 산출물 구조를 전제로 하므로, 먼저 OpenSpec CLI가 설치되어 있어야 합니다.

### 1. OpenSpec 설치

OpenSpec은 Node.js 20.19.0 이상을 요구합니다.

```bash
npm install -g @fission-ai/openspec@latest
```

설치 후 대상 프로젝트에서 다음 명령이 동작하는지 확인합니다.

```bash
openspec --version
```

### 2. 대상 프로젝트 OpenSpec 구조 준비

대상 프로젝트가 아직 OpenSpec 구조를 갖고 있지 않다면 대상 프로젝트에서 `openspec init`을 실행합니다.

```bash
cd your-project
openspec init
```

이미 OpenSpec 구조가 있는 프로젝트라면 기존 `openspec/` 내용을 유지하세요. 이 skill pack은 더 이상 별도의 `openspec/config.yaml`, `openspec/e2e/memories/`, `openspec/e2e/scripts/`를 설치하지 않습니다. 한국어 작성 규칙, feature-sized spec 규칙, spec-local 산출물 배치 규칙은 각 스킬의 `SKILL.md`에 포함되어 있습니다.

### 3. 커스텀 스킬 복사

이 저장소에서 대상 프로젝트에 복사할 항목은 커스텀 스킬 3개입니다.

```text
.claude/skills/openspec-code-to-spec/
.claude/skills/openspec-e2e/
.claude/skills/openspec-manual/
```

대상 프로젝트 루트로 복사합니다.

```bash
mkdir -p your-project/.claude/skills
cp -R .claude/skills/openspec-code-to-spec your-project/.claude/skills/
cp -R .claude/skills/openspec-e2e your-project/.claude/skills/
cp -R .claude/skills/openspec-manual your-project/.claude/skills/
```

Windows PowerShell 예시는 다음과 같습니다.

```powershell
New-Item -ItemType Directory -Force C:\path\to\your-project\.claude\skills
Copy-Item -Recurse -Force .\.claude\skills\openspec-code-to-spec C:\path\to\your-project\.claude\skills\
Copy-Item -Recurse -Force .\.claude\skills\openspec-e2e C:\path\to\your-project\.claude\skills\
Copy-Item -Recurse -Force .\.claude\skills\openspec-manual C:\path\to\your-project\.claude\skills\
```

복사 후 대상 프로젝트의 구조는 대략 다음과 같습니다.

```text
your-project/
├── openspec/
│   └── specs/
├── .claude/
│   └── skills/
│       ├── openspec-code-to-spec/
│       ├── openspec-e2e/
│       └── openspec-manual/
└── ...
```

**중요**: 이 저장소는 `.claude/commands`, upstream OpenSpec 기본 스킬, 프로젝트 `openspec/config.yaml`, 전역 E2E memory/script 폴더를 포함하지 않습니다. OpenSpec 기본 기능은 설치된 OpenSpec CLI와 upstream 자료를 그대로 사용하고, 이 pack은 `openspec-code-to-spec`, `openspec-e2e`, `openspec-manual`만 추가합니다.

---

## 빠른 시작

대상 프로젝트를 Claude Code에서 열고, 코드 폴더를 스펙으로 변환합니다.

```text
/openspec-code-to-spec services/billing
```

생성된 스펙 이름을 확인한 뒤 E2E 테스트를 만듭니다.

```text
/openspec-e2e billing_invoice-search
```

E2E 산출물과 스크린샷이 준비되면 DOCX 사용자 매뉴얼을 생성합니다.

```text
/openspec-manual billing_invoice-search
```

각 스킬은 인자를 생략할 수 있습니다. 다만 대상이 명확하지 않으면 스킬이 분석할 코드 폴더, 스펙 이름, 또는 스펙 경로를 다시 물어봅니다.

```text
/openspec-code-to-spec
/openspec-e2e
/openspec-manual
```

자연어로도 요청할 수 있습니다.

```text
services/billing 코드를 OpenSpec 스펙으로 만들어줘
```

```text
billing_invoice-search 스펙 기준으로 E2E 테스트를 만들어줘
```

```text
이 스펙의 E2E 스크린샷으로 사용자 매뉴얼 DOCX를 만들어줘
```

---

## 3단계 워크플로

| # | Skill | 입력 | 주요 산출물 |
|---|---|---|---|
| 1 | `openspec-code-to-spec` | 소스 폴더 하나 이상 | planning 문서, `spec.md` |
| 2 | `openspec-e2e` | OpenSpec 스펙 이름 | E2E 시나리오, Playwright 테스트, Docker 실행 환경, 결과/스크린샷/커버리지 리포트 |
| 3 | `openspec-manual` | OpenSpec 스펙 이름 | DOCX 사용자 매뉴얼, 재생성 스크립트 |

### Step 1 — Code To Spec

기존 코드에서 사용자, 클라이언트, 운영자가 관찰할 수 있는 행위를 추출해 OpenSpec main spec으로 정리합니다.

```text
/openspec-code-to-spec <source-folder...>
```

예시:

```text
/openspec-code-to-spec services/completion
```

이 스킬은 `SKILL.md`에 포함된 프로젝트 규칙과 템플릿을 읽고, 소스 폴더의 공개 API, UI 흐름, 이벤트, 작업, 저장 상태, 오류 처리 등을 확인합니다. 그 다음 바로 스펙을 쓰지 않고 `openspec/plannings/` 아래에 분할 계획을 먼저 만듭니다.

**산출물**:

```text
openspec/
├── plannings/
│   └── openspec-code-to-spec-<scope-slug>.md
└── specs/
    └── <spec-name>/
        └── spec.md
```

스펙 이름은 서비스 경계를 보존합니다.

```text
openspec/specs/<microservice>_<feature>/spec.md
openspec/specs/<microservice>_<domain>-<feature>/spec.md
```

예를 들어 `services/billing`의 invoice search 기능은 `billing_invoice-search`처럼 작성합니다. 여러 도메인을 가진 `services/completion`은 `completion_agent-memory-chat`, `completion_mcp-server-config`처럼 나눌 수 있습니다.

### Step 2 — Spec-Driven E2E Tests

OpenSpec 스펙을 기준으로 실제 사용자 흐름과 검증 가능한 시나리오를 만들고, Docker Compose와 Playwright 기반 E2E 테스트를 구성합니다.

```text
/openspec-e2e <spec-name>
```

예시:

```text
/openspec-e2e completion_agent-memory-chat
```

이 스킬은 스펙 이름을 그대로 E2E suite slug로 사용합니다. 언더스코어와 하이픈은 서비스, 도메인, 기능 경계를 나타내므로 바꾸지 않습니다.

**산출물**:

```text
openspec/specs/<spec-name>/
└── e2e/
    ├── scenarios/
    │   ├── 00-coverage-matrix.md
    │   └── <scenario-docs>.md
    ├── tests/
    │   └── <playwright-specs>
    ├── seed_files/
    ├── scripts/
    ├── docker/
    └── results/
        ├── screenshots/
        ├── coverage-summary.json
        └── spec-coverage-report.html
```

E2E 결과물은 단순 테스트 코드가 아니라 다음 단계의 사용자 매뉴얼을 만들기 위한 증거이기도 합니다. 특히 시나리오 문서와 스크린샷은 `openspec-manual`이 사용자 흐름과 화면 설명을 구성할 때 재사용합니다.

### Step 3 — DOCX User Manual

스펙과 E2E 산출물, 실제 화면 스크린샷을 바탕으로 최종 사용자가 읽을 수 있는 한국어 DOCX 매뉴얼을 생성합니다.

```text
/openspec-manual <spec-name>
```

예시:

```text
/openspec-manual completion_agent-memory-chat
```

이 스킬은 개발자용 검증 문서를 그대로 옮기지 않습니다. E2E, Playwright, API route, assertion 같은 내부 용어는 사용자에게 보이는 작업 흐름, 화면 항목, 오류 해결 방법으로 바꿔 설명합니다.

**산출물**:

```text
openspec/specs/<spec-name>/
└── docs/
    ├── <spec-name>-user-manual.docx
    └── generate_<spec-name>_user_manual.py
```

생성 스크립트가 함께 남기 때문에 스펙, 시나리오, 스크린샷이 바뀐 뒤에도 같은 스타일로 매뉴얼을 다시 만들 수 있습니다.

---

## 산출물 구조

전체 산출물은 스펙 이름을 중심으로 응집됩니다.

```text
openspec/
├── plannings/
│   └── openspec-code-to-spec-<scope-slug>.md
└── specs/
    └── <spec-name>/
        ├── spec.md
        ├── e2e/
        │   ├── scenarios/
        │   ├── tests/
        │   ├── seed_files/
        │   ├── scripts/
        │   ├── docker/
        │   └── results/
        │       ├── screenshots/
        │       ├── coverage-summary.json
        │       └── spec-coverage-report.html
        └── docs/
            ├── <spec-name>-user-manual.docx
            └── generate_<spec-name>_user_manual.py
```

이 구조를 유지하면 한 기능에 대한 요구사항, 테스트 증거, 사용자 문서를 한 폴더에서 추적할 수 있습니다.

---

## 워크스루 예시 — 결제 조회 기능

예를 들어 `services/billing`에 청구서 검색 기능이 이미 구현되어 있다고 가정합니다.

### 입력

```text
/openspec-code-to-spec services/billing
```

Claude Code는 코드 구조를 그대로 요약하지 않고, 외부에서 관찰 가능한 기능 단위로 나눕니다. 그 결과 `billing_invoice-search`라는 스펙이 적절하다고 판단할 수 있습니다.

### Step 1 결과

```text
openspec/plannings/openspec-code-to-spec-billing.md
openspec/specs/billing_invoice-search/spec.md
```

`spec.md`에는 "사용자가 조건으로 청구서를 검색한다", "권한이 없는 요청은 거부된다", "결과가 없을 때 빈 결과를 명확히 반환한다" 같은 행위 계약이 한국어 요구사항과 시나리오로 작성됩니다.

### Step 2 입력

```text
/openspec-e2e billing_invoice-search
```

### Step 2 결과

```text
openspec/specs/billing_invoice-search/e2e/scenarios/00-coverage-matrix.md
openspec/specs/billing_invoice-search/e2e/tests/
openspec/specs/billing_invoice-search/e2e/results/screenshots/
openspec/specs/billing_invoice-search/e2e/results/spec-coverage-report.html
```

이 단계에서는 스펙 요구사항이 어떤 E2E 시나리오로 검증되는지 연결하고, 실제 UI 또는 사용자 흐름을 통해 스크린샷을 남깁니다.

### Step 3 입력

```text
/openspec-manual billing_invoice-search
```

### Step 3 결과

```text
openspec/specs/billing_invoice-search/docs/billing_invoice-search-user-manual.docx
openspec/specs/billing_invoice-search/docs/generate_billing_invoice-search_user_manual.py
```

최종 매뉴얼은 테스트 리포트가 아니라 사용자 문서입니다. 기능 개요, 사용 전 확인 사항, 화면 구성, 따라 하기, 결과 확인, 오류 상황, FAQ 같은 형식으로 정리됩니다.

---

## 스펙 이름과 경로 전달

세 스킬 모두 대상 인자를 선택적으로 받을 수 있습니다.

| Skill | 인자 생략 시 | 인자 전달 시 |
|---|---|---|
| `openspec-code-to-spec` | 분석할 소스 폴더를 질문합니다. | 전달한 폴더들을 분석합니다. |
| `openspec-e2e` | 사용할 OpenSpec 스펙 이름을 질문합니다. | `openspec/specs/<spec-name>/spec.md`를 기준으로 진행합니다. |
| `openspec-manual` | 매뉴얼을 만들 스펙 이름과 E2E 증거를 질문합니다. | 해당 스펙 폴더의 `spec.md`, `e2e/` 산출물을 기준으로 진행합니다. |

스펙 이름은 폴더명 그대로 전달하는 것을 권장합니다.

```text
completion_agent-memory-chat
billing_invoice-search
auth_password-reset
```

경로를 말해도 됩니다.

```text
openspec/specs/billing_invoice-search/spec.md 기준으로 E2E를 만들어줘
```

다만 스킬 내부 규칙은 기본적으로 `openspec/specs/<spec-name>/` 구조를 기준으로 동작합니다.

---

## Spec-Driven Development가 낯선 분을 위한 설명

SDD는 "코드를 먼저 만들고 나중에 문서화"하는 방식과 반대로, 기능의 행위 계약을 먼저 명확히 하고 구현과 검증을 그 계약에 맞추는 흐름입니다.

여기서 스펙은 긴 기획서가 아닙니다. 다음 질문에 답하는 가벼운 기준 문서입니다.

- 사용자는 무엇을 할 수 있어야 하는가?
- 클라이언트나 운영자는 어떤 입력, 출력, 상태 변화를 관찰할 수 있는가?
- 오류, 권한, 빈 결과, 재시도 같은 경계 상황은 어떻게 보여야 하는가?
- 이 요구사항은 어떤 E2E 시나리오로 검증할 수 있는가?

이 skill pack은 기존 코드에서 시작할 수 있도록 설계되어 있습니다. 이미 구현된 기능을 `openspec-code-to-spec`으로 역분석해 현재 시스템이 보장해야 하는 행위를 정리하고, 이후 테스트와 매뉴얼을 같은 스펙 기준으로 이어갑니다.

---

## 팁과 주의사항

### 잘 사용하는 방법

- 한 번에 전체 서비스를 스펙 하나로 만들지 말고, 사용자 또는 클라이언트가 이해할 수 있는 기능 단위로 나누세요.
- `openspec-code-to-spec` 결과를 바로 확정하지 말고 `openspec/plannings/`의 분할 계획을 먼저 검토하세요.
- E2E를 만들기 전에 `spec.md`가 실제로 테스트 가능한 요구사항과 시나리오를 담고 있는지 확인하세요.
- 매뉴얼 품질은 E2E 스크린샷 품질에 크게 좌우됩니다. 사용자에게 의미 있는 화면 상태를 캡처하도록 시나리오 단계에서 정리하세요.
- 산출물은 사람이 직접 수정해도 됩니다. 다음 Claude Code 세션에서 수정된 파일을 기준으로 이어갈 수 있습니다.

### 피해야 할 패턴

- `frontend_*`, `ui_*`, `react_*`처럼 구현 레이어 이름으로 main spec을 만들지 마세요.
- 내부 함수명, 클래스명, 파일 라인 번호를 `spec.md` 요구사항에 넣지 마세요.
- E2E 산출물을 루트 `e2e/`나 `docs/`로 흩뜨리지 말고 해당 스펙 폴더 아래에 두세요.
- 사용자 매뉴얼에 E2E, Playwright, coverage, API route 같은 검증/개발 용어를 그대로 노출하지 마세요.
- 기존 `openspec/specs/`가 있는 프로젝트에 이 저장소의 `openspec/` 폴더를 무심코 덮어쓰지 마세요.

---

## 자주 묻는 질문

**Q. OpenSpec을 설치하지 않고 이 폴더만 복사해도 되나요?**

A. 권장하지 않습니다. 대상 프로젝트를 `openspec init`으로 초기화하지 않는 것은 가능하지만, 이 pack의 스킬은 `openspec validate` 같은 CLI 기능을 활용하므로 OpenSpec CLI 자체는 설치되어 있어야 합니다.

**Q. 기존 프로젝트에 이미 `.claude/` 폴더가 있으면 어떻게 하나요?**

A. 폴더 전체를 덮어쓰기보다 `.claude/skills/openspec-code-to-spec`, `.claude/skills/openspec-e2e`, `.claude/skills/openspec-manual`만 기존 `.claude/skills/` 아래로 병합하세요. `.claude/commands`나 upstream OpenSpec 기본 스킬을 이 저장소에서 복사할 필요는 없습니다.

**Q. `openspec-code-to-spec`에 여러 폴더를 전달할 수 있나요?**

A. 가능합니다. 예를 들어 `/openspec-code-to-spec services/billing services/auth`처럼 전달할 수 있습니다. 다만 스킬은 서비스 경계와 기능 경계를 먼저 나눈 뒤 여러 스펙으로 분리하려고 합니다.

**Q. 스펙 없이 바로 E2E 테스트를 만들 수 있나요?**

A. 이 pack의 권장 흐름에서는 먼저 `openspec/specs/<spec-name>/spec.md`가 있어야 합니다. E2E는 스펙 요구사항과 시나리오를 검증하기 위한 산출물이기 때문입니다.

**Q. E2E 없이 DOCX 매뉴얼을 만들 수 있나요?**

A. 스킬은 기본적으로 스펙과 E2E 시나리오, 스크린샷을 근거로 매뉴얼을 만듭니다. E2E 증거가 없으면 어떤 화면과 사용자 흐름을 문서화해야 하는지 다시 확인해야 하므로, 먼저 `openspec-e2e`를 진행하는 것이 좋습니다.

**Q. 산출물은 모두 한국어로 작성되나요?**

A. 기본 규칙은 한국어입니다. API 경로, HTTP method, 필드명, 이벤트명, 환경 변수, 파일 경로, 코드 식별자처럼 정확히 유지해야 하는 기술 식별자만 원문을 유지합니다.

**Q. OpenSpec 업데이트는 어떻게 하나요?**

A. OpenSpec CLI는 전역 패키지를 업데이트합니다. 이 저장소는 OpenSpec 기본 스킬/커맨드를 vendoring하지 않으므로, upstream OpenSpec skill pack을 덮어쓰는 별도 갱신 작업은 없습니다.

```bash
npm install -g @fission-ai/openspec@latest
```

---

## 업데이트

이 skill pack 자체를 최신으로 반영하려면 이 저장소에서 최신 커스텀 스킬 3개 폴더만 다시 대상 프로젝트에 병합합니다.

```text
.claude/skills/openspec-code-to-spec/
.claude/skills/openspec-e2e/
.claude/skills/openspec-manual/
```

기존 프로젝트에 이미 작성된 `openspec/specs/` 산출물은 이 업데이트 과정에서 덮어쓸 필요가 없습니다. 이 pack의 규칙 변경은 스킬 폴더 안에 포함되므로, 대상 프로젝트에서 스킬 파일을 수정해 사용 중이었다면 수동 병합을 권장합니다.

---

## 제거

대상 프로젝트에서 이 skill pack을 제거하려면 추가했던 스킬 폴더를 삭제합니다.

```bash
rm -rf .claude/skills/openspec-code-to-spec
rm -rf .claude/skills/openspec-e2e
rm -rf .claude/skills/openspec-manual
```

OpenSpec 산출물까지 제거하려면 `openspec/` 폴더를 별도로 정리해야 합니다. 이미 생성한 스펙, E2E 결과, DOCX 매뉴얼이 함께 삭제될 수 있으니 주의하세요.

---

## 구성 파일

주요 파일은 다음과 같습니다.

```text
.claude/skills/openspec-code-to-spec/SKILL.md
.claude/skills/openspec-code-to-spec/TEMPLATES.md
.claude/skills/openspec-e2e/SKILL.md
.claude/skills/openspec-e2e/OUTPUT_CONTRACT.md
.claude/skills/openspec-e2e/TEMPLATES.md
.claude/skills/openspec-manual/SKILL.md
.claude/skills/openspec-manual/STYLE_REFERENCE.py
```

각 스킬의 `SKILL.md`는 한국어 작성 규칙, feature-sized spec 규칙, spec-local 산출물 배치 규칙, Claude Code가 해당 작업을 수행할 때 따라야 할 워크플로와 품질 게이트를 정의합니다.

이 저장소에는 `.claude/commands`나 upstream OpenSpec 기본 스킬이 포함되지 않습니다. 해당 기능은 OpenSpec CLI 같은 upstream 설치본을 사용하세요.
