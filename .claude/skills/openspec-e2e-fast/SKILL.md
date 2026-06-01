---
name: openspec-e2e-fast
description: Rapidly produce screenshot-only Playwright E2E walkthroughs from an OpenSpec spec.md, with no expect/assertions, no coverage gates, and direct event dispatch for slow interactions. Use when speed matters more than verification and you only need runnable E2E code plus per-checkpoint screenshots as deliverables. Prefer openspec-e2e instead when you need assertions, coverage gates, traceability reports, or audit-quality evidence.
argument-hint: "<spec-name>"
arguments: [spec_name]
disable-model-invocation: true
license: MIT
compatibility: Requires Playwright and a way to run the repository frontend plus its owning backend (reuse the existing docker-compose.e2e.yml infrastructure and documented source-run commands). No coverage tooling is required.
metadata:
  author: project
  version: "1.0"
---

# Fast Spec-Driven E2E Screenshots

Produce a runnable Playwright walkthrough and per-checkpoint screenshots from OpenSpec requirements **as fast as possible**. The user reviews the screenshots; this skill does not verify behavior.

**Input**: One `spec.md`, the real frontend it exercises, and a way to start the owning backend.

**Deliverables (only these)**:
1. One Playwright/Node E2E script that drives the real frontend through the spec's flow.
2. Screenshots at each meaningful UI checkpoint.

Nothing else is required: no coverage matrix, no scenario markdown, no coverage reports, no traceability gate, no validators, no execution summary.

## Relationship to `openspec-e2e` (read this first)

`openspec-e2e-fast` is the speed-optimized sibling of [`../openspec-e2e/SKILL.md`](../openspec-e2e/SKILL.md). It deliberately drops the slow, high-assurance parts. The differences below are mandatory, not optional.

| Concern | `openspec-e2e` (slow, high quality) | `openspec-e2e-fast` (this skill) |
| --- | --- | --- |
| Assertions | `expect(...)`, status/body/UI checks | **None.** Never write `expect`, `assert`, or response validation. |
| Verification owner | The test verifies | **The user verifies** by looking at screenshots |
| Coverage | Backend + frontend coverage, Monocart, V8, thresholds | **None.** No coverage tooling, no `page.coverage`. |
| Reports/gates | `coverage-summary.json`, `spec-coverage-report.html`, traceability + output validators | **None.** Do not run `evaluate_spec_coverage.mjs` or `validate_e2e_outputs.py`. |
| Scenario docs | `00-coverage-matrix.md` + per-scenario `.md` + `execution-summary.md` | **None required.** A short inline checkpoint list is enough. |
| Slow user gestures (drag/drop, multi-step wizards, overlay-blocked clicks) | Mimic full real-user interaction | **Fire DOM events directly** when the real click path is slow or flaky (see Speed Techniques). |
| Runtime | Boot/validate infra, full Sanity Check, re-run compose pre-checks | **Reuse** what already runs; minimal smoke check only. |

**Hard rule — no validation code**: This skill must not generate `expect`, `assert`, `toBe*`, `toHaveURL`, `waitForResponse`-as-assertion, coverage capture, or report generation. If you feel the urge to assert, take a screenshot of that state instead and let the user judge it.

## Rules kept from `openspec-e2e` (do not drop these)

Speed does not mean faking the product. Keep these:

- **Spec resolution**: When invoked as `/openspec-e2e-fast <spec-name>`, use `$spec_name` as the exact OpenSpec folder name and read `openspec/specs/<spec-name>/spec.md`. If empty/ambiguous/missing, ask instead of guessing.
- **Suite slug**: Preserve the exact spec folder name as the suite slug (keep underscores/hyphens), e.g. `completion_process-definition-search`.
- **Spec-local output**: Write the script under `openspec/specs/<spec-name>/e2e/tests/` and screenshots under `openspec/specs/<spec-name>/e2e/results/screenshots/`. Do not scatter outputs elsewhere.
- **Real frontend**: Drive the repository's actual frontend served from source. Never serve synthetic tester HTML via `page.route()`, `page.setContent()`, or `file://`. Firing DOM events on the **real page** is allowed and encouraged; fabricating a fake page is not.
- **Real service path**: Run the real frontend against its owning backend (reuse existing `docker-compose.e2e.yml` infrastructure + documented source-run commands). Stub only true external boundaries (LLM, third-party APIs) when they block a screenshot.
- **Korean naming**: Use Korean for test titles, checkpoint names, and any short notes. Keep exact technical identifiers (routes, selectors, fields) verbatim.

## File Reference Budget (the speed lever — read only what you need)

The biggest time sink observed in practice is reading and restoring `openspec-e2e`'s heavy scaffolding (coverage wrappers, sourcemap builds, memory/script-promotion system, another suite's assertion-laden spec). **Do not touch those.** Keep your file footprint to the small allowlist below.

**Read / restore ONLY these (essential):**
- `openspec/specs/<spec-name>/spec.md` — the spec under test.
- The 1–2 real frontend files that trigger the behavior — locate them with a targeted `Grep` (the API route name, the button/menu label), then read only the relevant line range, not whole files. Typically one component (e.g. the page/dialog the user interacts with) plus the API client method it calls.
- Runtime bring-up files **only if the runtime is not already up** (see Step 3):
  - `docker-compose.e2e.yml` (infrastructure only).
  - The minimal source-run start scripts for the owning backend and the frontend (e.g. `start_completion.ps1`, `start_frontend.ps1`).
  - `seed_files/*.sql` only if the screenshots need seeded data that is not already present.
- An external-boundary mock (e.g. `mock_llm.py`) **only if** an external call would otherwise block a screenshot.

**Do NOT read, restore, run, or copy these (out of scope for fast):**
- Coverage scaffolding: `coverage_wrapper.py`, `monocart_report.mjs`, `coveragerc*`, `Dockerfile.*-coverage`, `sitecustomize.py`, and any `build_and_serve_frontend.*` whose only purpose is a sourcemap rebuild for coverage. Use the normal source-run frontend command instead.
- The `openspec-e2e` memory/script-promotion system: `openspec/e2e/memories/**`, `openspec/e2e/scripts/README.md`, and promotion bookkeeping.
- `openspec-e2e` contract/templates: `OUTPUT_CONTRACT.md`, `TEMPLATES.md`, `COVERAGE_HTML_TEMPLATE.html`, the validator/traceability scripts.
- Another suite's full `*.spec.mjs` or `playwright.config.mjs` as a template — they carry `expect`, coverage reporters, and long timeouts. Write a minimal config inline instead (Step 4).
- Scenario docs (`00-coverage-matrix.md`, per-scenario `.md`, `execution-summary.md`) — neither produced nor consumed by this skill.

When restoring scaffolding from git (if a previous run deleted it), restore **only** the essential runtime files above with explicit paths — never `git checkout` the whole `e2e/` folder, which would drag back coverage and scenario artifacts.

## Workflow

Copy this checklist and track progress:

```
- [ ] 1. 스펙 읽고 스위트 슬러그 확정
- [ ] 2. 실제 프론트엔드 진입 경로와 스크린샷 체크포인트 목록화
- [ ] 3. 런타임 기동(기존 것 재사용) + 1분 스모크
- [ ] 4. 검증 코드 없는 fast 스크립트 작성
- [ ] 5. 실행 후 스크린샷 산출물 확인
```

**Step 1 — Read spec, set slug**
- Read `spec.md`. Extract only the user-visible flow and its meaningful UI states. Ignore contract details you would normally assert.
- Set suite slug = spec folder name.

**Step 2 — Map the real entry path and checkpoints**
- Use a targeted `Grep` for the spec's API route / button label to find the real frontend route/component/action that triggers the behavior (including indirect paths, e.g. a search box that calls a backend route). Read only the matching line range — do not read whole files or browse unrelated suites.
- List checkpoints inline (no separate doc): initial screen, completed input, running/progress, result, empty result, error — only the ones that actually occur. Name them `NN-<korean-checkpoint>`.
- If there is genuinely no real user-visible surface, stop and tell the user (do not fabricate a page).

**Step 3 — Bring up runtime (reuse first, minimal bring-up only if down)**
- **Check what is already running first** (e.g. `docker ps`, then load the app URL). If the frontend + owning backend already respond, change nothing and skip to Step 4.
- Only if the runtime is down, bring up the minimum: `docker compose -f docker-compose.e2e.yml up -d` for infrastructure, then the source-run start scripts for the backend and frontend. Seed data only if the screenshots need it.
- Do one ~1 minute smoke check: the app URL loads and the entry screen renders. No formal Sanity Check, no compose pre-check scripts, no coverage build, no sourcemap rebuild.

**Step 4 — Write the fast script**
- Place at `openspec/specs/<spec-name>/e2e/tests/<suite-slug>.fast.spec.mjs` (Playwright runner) or a standalone `run.mjs` using `playwright`'s `chromium` directly when that boots faster.
- Write a **minimal inline config** (only a screenshot path + base URL + a single chromium project). Do not copy another suite's `playwright.config.mjs` — those carry coverage reporters and long timeouts.
- Reuse one browser context/page across checkpoints; log in once if needed. Do not repeat expensive setup per checkpoint.
- Drive real UI for cheap interactions (`goto`, `click`, `fill`, `press`, `selectOption`, `setInputFiles`).
- For slow/flaky interactions, **fire DOM events directly** (see Speed Techniques).
- Capture a screenshot at every checkpoint. **No `expect`, no coverage, no asserts.**

**Step 5 — Run and confirm screenshots exist**
- Run the script. Confirm one screenshot file per checkpoint exists under the screenshots folder.
- Done. Do not run coverage, validators, or report generators. Hand the screenshots to the user for review.

## Speed Techniques (fire events directly)

When a real user gesture is slow, multi-step, or hit-test-flaky, dispatch the underlying DOM event instead of choreographing the full interaction. This is the core speed lever.

**HTML5 drag-and-drop** — replace a slow `dragTo`/mouse choreography with one `page.evaluate`:

```js
async function html5Drop(page, srcHandle, tgtHandle) {
  await page.evaluate(({ s, t }) => {
    const dt = new DataTransfer()
    const fire = (el, type) => {
      const r = el.getBoundingClientRect()
      const ev = new DragEvent(type, {
        bubbles: true, cancelable: true,
        clientX: r.left + r.width / 2, clientY: r.top + r.height / 2,
      })
      Object.defineProperty(ev, 'dataTransfer', { value: dt })
      el.dispatchEvent(ev)
    }
    fire(s, 'dragstart'); fire(t, 'dragenter'); fire(t, 'dragover'); fire(t, 'drop'); fire(s, 'dragend')
  }, { s: srcHandle, t: tgtHandle })
}
```

**Overlay-blocked clicks** — bypass edge/overlay hit-testing with a direct dispatch:

```js
await page.locator('.vue-flow__node[data-id="..."]').dispatchEvent('dblclick')
```

**General rules**:
- Prefer `locator.dispatchEvent('click'|'dblclick'|'input'|'change')` over fighting overlays with `force: true` retries.
- Prefer `page.evaluate` to set input values or trigger custom events when a long typing/selection sequence only matters for reaching the next screen.
- Keep waits short and screenshot-driven: wait just long enough for the target UI to render, then capture. Use a quick visibility wait or a small fixed pause; do not wait on network assertions.
- It is fine to wrap each checkpoint phase in `try/catch` and screenshot the failure state so a single broken step never blocks the remaining screenshots.

## Anti-Patterns

- Writing `expect`/`assert`/coverage/report code "just to be safe" — it defeats the purpose; the user verifies.
- Fabricating a tester HTML page to make a screenshot easier — always drive the real frontend.
- Re-mimicking a 10-step drag or wizard with real mouse moves when a direct event reaches the same screen.
- Repeating login/boot for every checkpoint instead of reusing one page.
- Producing scenario docs, coverage matrices, or summaries — out of scope for this skill.
- Reading or restoring coverage scaffolding (`coverage_wrapper.py`, `monocart_report.mjs`, sourcemap build scripts, `coveragerc`, coverage Dockerfiles) — never needed for screenshots.
- Reading the `openspec/e2e/memories/**` or script-promotion `README.md` — that is the slow skill's bookkeeping.
- Copying another suite's `*.spec.mjs` / `playwright.config.mjs` as a starting template — they reintroduce asserts and coverage. Write minimal code inline.
- `git checkout` of the whole `e2e/` folder to restore deleted files — restore only the specific runtime files you need.

## When to escalate to `openspec-e2e`

If the user later needs assertions, coverage gates, traceability evidence, or audit-quality reports, switch to [`../openspec-e2e/SKILL.md`](../openspec-e2e/SKILL.md). This fast skill is for quick screenshot deliverables only.
