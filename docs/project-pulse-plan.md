# Project Pulse — Implementation Plan

## Summary

Project Pulse is a small static dashboard that renders a list of projects (name, owner, status, priority, recent activity, last-updated date) as visually distinct cards with status badges. It will live under `app/` in this repo and be launchable from VS Code via a `.vscode/launch.json` entry named **Run Project Pulse Dashboard**. The build is a learning exercise for orchestrating four custom Copilot CLI agents (Orchestrator → Planner → Designer + Coder). The Planner (this document) defines scopes so the Designer and Coder can work in parallel without file conflicts.

Deliverables:

- `app/index.html` — dashboard markup with deterministic hooks the Designer can style and the Coder can target from JS.
- `app/styles.css` — visual design, layout, badges, responsive behavior, accessibility.
- `app/project-data.json` — sample project data (top-level `projects` array) matching the brief.
- `.vscode/launch.json` — a launch configuration serving `app/` and opening `index.html` (cwd `${workspaceFolder}/app`).

Rendering approach: vanilla JS `fetch('./project-data.json')` on page load, iterate `projects`, render one `.project-card` per entry into a `.dashboard` container. No build tooling, no frameworks — matches the repo's static-app pattern and the Coder agent's "consistent, predictable layout" principle.

Launch approach: use VS Code's built-in Node `npx` runner to start a tiny static server rooted at `app/` (e.g. `npx --yes serve -l 5173 .` with `cwd: ${workspaceFolder}/app`) and set `serverReadyAction` to open `http://localhost:5173/index.html` in the browser. This satisfies the brief's requirement that learners see the dashboard, not a directory listing, and keeps the config to strict JSON with no comments.

---

## Ordered Implementation Steps

1. **Freeze the data contract** — decide the exact JSON shape for `app/project-data.json` so both markup and rendering agree. (Planner output; no file yet.)
2. **Designer: visual/UX spec** — produce the CSS hook contract and card anatomy the Coder must render (class names, states, badge variants). (Planner-mediated, written into this doc; no file yet.)
3. **Coder: author `app/project-data.json`** — 5–7 sample projects covering every status and priority value so Designer has real content to style against.
4. **Coder: author `app/index.html`** — semantic markup shell with the agreed CSS hooks, an empty `.dashboard` container, an inline or linked `<script type="module">` (or plain `<script defer>`) that fetches the JSON and renders `.project-card` nodes. Include header, empty-state element, and error element.
5. **Designer: author `app/styles.css`** — style `.dashboard`, `.project-card`, `.status-badge` (variants per status), `.priority-*`, header, empty state, responsive breakpoints, focus states, and color contrast.
6. **Coder: author `.vscode/launch.json`** — add **Run Project Pulse Dashboard** launch config (Node type, `npx serve`, `cwd: ${workspaceFolder}/app`, `serverReadyAction` opens `index.html`).
7. **Orchestrator: integration check** — open dashboard via launch config, verify cards render, badges reflect data, layout is responsive, no console errors.
8. **Iterate** on any gaps surfaced in step 7 (Designer for visual, Coder for logic/config).

Steps 3–5 can proceed in parallel once step 2 is agreed. Step 6 can proceed in parallel with 3–5 because it touches only `.vscode/`. Step 4 depends on step 3 only for the field names, not the file contents.

---

## Data Contract (frozen before implementation)

`app/project-data.json`:

```json
{
  "projects": [
    {
      "id": "string, stable slug",
      "name": "string",
      "owner": "string (display name or handle)",
      "status": "On Track | At Risk | Delayed",
      "priority": "High | Medium | Low",
      "recentActivity": "string, short sentence",
      "lastUpdated": "ISO 8601 date, e.g. 2026-08-28"
    }
  ]
}
```

Rules:

- `status` values are a closed set: `On Track`, `At Risk`, `Delayed`. The Designer maps each to a badge modifier class (see below). The Coder must slugify (`On Track` → `on-track`) when building class names.
- `priority` values are a closed set: `High`, `Medium`, `Low`.
- `lastUpdated` is required and displayed human-friendly (e.g. `Aug 28, 2026`) via `Intl.DateTimeFormat`.
- `id` is required so the Coder can key list rendering and avoid duplicate DOM ids.

---

## CSS Hook Contract (frozen before implementation)

The Coder must emit exactly these hooks; the Designer must style exactly these hooks.

- `.dashboard` — grid container for cards.
- `.dashboard__header` — title + subtitle + project count.
- `.dashboard__empty` — empty-state message, hidden unless list is empty.
- `.dashboard__error` — error message, hidden unless fetch fails.
- `.project-card` — single card root.
- `.project-card__name`
- `.project-card__owner`
- `.project-card__activity`
- `.project-card__meta` — wraps last-updated
- `.status-badge` + one of `.status-badge--on-track`, `.status-badge--at-risk`, `.status-badge--delayed`
- `.priority` + one of `.priority--high`, `.priority--medium`, `.priority--low`

Accessibility hooks:

- Each card is an `<article>` with `aria-labelledby` pointing at the name element's id.
- Status badge includes visually hidden `Status:` prefix (`.visually-hidden`).
- Dashboard root is `<main>` with a landmark; header is `<h1>`.

---

## File Assignments per Step

| Step | Agent | Files created / modified | Notes |
|------|-------|--------------------------|-------|
| 1 | Planner | (none — spec in this doc) | Data contract frozen above. |
| 2 | Planner (Designer input) | (none — spec in this doc) | CSS hook contract frozen above. |
| 3 | Coder | `app/project-data.json` | 5–7 sample projects, cover every status + priority. |
| 4 | Coder | `app/index.html` | Semantic shell + fetch/render script. May inline JS in a `<script defer>` block to keep to three `app/` files. |
| 5 | Designer | `app/styles.css` | Owns entire file. |
| 6 | Coder | `.vscode/launch.json` | Strict JSON, no comments. |
| 7 | Orchestrator | (none) | Integration verification only. |
| 8 | Designer or Coder | Their own file only | Fix-forward within existing ownership. |

Ownership is exclusive: no file is written by two agents. This is the key conflict-avoidance rule.

---

## Dependencies Between Steps

- Step 3 (`project-data.json`) depends on Step 1 (data contract).
- Step 4 (`index.html`) depends on Step 1 (needs field names) and Step 2 (needs CSS hook names).
- Step 4's JS rendering depends on Step 3's file *existing at runtime*, but not for authoring — the Coder can write the render code against the frozen contract before the sample file is finalized.
- Step 5 (`styles.css`) depends on Step 2 (CSS hook contract). It does *not* need `index.html` to exist because the hooks are frozen; however, visual QA of Step 5 depends on Step 4 being complete so the Designer can see real cards.
- Step 6 (`launch.json`) depends only on knowing the entry file (`index.html`) and cwd (`app/`). It does not depend on Steps 3–5 for authoring, but end-to-end verification does.
- Step 7 depends on Steps 3, 4, 5, and 6 all being complete.

---

## Work That Can Run in Parallel

Once Steps 1 and 2 (the contracts) are frozen, the following can run concurrently because file scopes and data are disjoint:

- **Coder track A:** Step 3 (`app/project-data.json`) → Step 4 (`app/index.html`) → Step 6 (`.vscode/launch.json`).
- **Designer track:** Step 5 (`app/styles.css`).

The Designer authoring `styles.css` against the frozen hook contract in parallel with the Coder authoring markup is the primary parallelism win. Neither agent touches the other's file.

Within the Coder track, Step 6 (`launch.json`) can be done at any time in parallel with 3 and 4 because it lives in a different directory and has no dependency on the other files' contents.

---

## Work That Must Run Sequentially

- Steps 1 → 2 → (3, 4, 5, 6): contracts must exist before any implementation.
- Step 4 must be authored after Step 3's schema is agreed (not necessarily its contents). Since the Planner freezes the schema in Step 1, this reduces to "Step 4 after Step 1."
- Step 7 (integration check) must run after 3, 4, 5, 6 land.
- Step 8 iteration runs after Step 7 findings.
- Any change to the data contract or CSS hook contract must be Planner-approved and then reapplied sequentially to all affected files to prevent divergence.

---

## Agent Responsibility Separation

### Designer owns
- `app/styles.css` (entire file).
- Visual system: typography scale, color palette (with WCAG AA contrast on badge text vs. badge background), spacing scale, elevation/shadows, rounded corners.
- Status badge visual language — three distinct treatments for On Track / At Risk / Delayed (color + optional icon glyph via CSS, not img).
- Priority treatment — high/medium/low visual weight (e.g., border-left accent color or pill).
- Responsive layout — CSS grid `.dashboard` with `repeat(auto-fill, minmax(280px, 1fr))` or equivalent; single-column below ~480px.
- Accessibility: focus-visible outlines, `.visually-hidden` utility, prefers-reduced-motion handling for any transitions, prefers-color-scheme if in scope.
- Empty-state and error-state visual design.

### Designer does NOT
- Modify `index.html`, `project-data.json`, or `launch.json`.
- Change class names once the hook contract is frozen without Planner sign-off.

### Coder owns
- `app/index.html` — semantic structure, exact CSS hooks from the contract, and the JS that fetches `./project-data.json`, renders cards, handles empty and error states, formats `lastUpdated`, and slugifies `status` for the badge modifier class.
- `app/project-data.json` — sample content only, no schema changes without Planner sign-off.
- `.vscode/launch.json` — strict JSON, `Run Project Pulse Dashboard`, `cwd: ${workspaceFolder}/app`, launches static server and opens `index.html` via `serverReadyAction`.

### Coder does NOT
- Add or change styles beyond attaching frozen class names.
- Invent new status or priority values.
- Add build tooling, package.json, or dependencies (repo pattern is static, zero-install; `npx --yes serve` runs on-demand).

---

## `launch.json` Shape (spec for Coder)

Strict JSON, no comments. Suggested content:

- `version`: `"0.2.0"`
- One configuration:
  - `name`: `"Run Project Pulse Dashboard"`
  - `type`: `"node"`
  - `request`: `"launch"`
  - `cwd`: `"${workspaceFolder}/app"`
  - `runtimeExecutable`: `"npx"`
  - `runtimeArgs`: `["--yes", "serve", "-l", "5173", "."]`
  - `serverReadyAction`: `{ "pattern": "Accepting connections at (https?://\\S+)", "uriFormat": "http://localhost:5173/index.html", "action": "openExternally" }`
  - `console`: `"integratedTerminal"`

The Coder must verify the chosen static server's ready-log pattern matches; if `serve`'s output differs, adjust `pattern` accordingly. Alternative: `http-server` via `npx --yes http-server . -p 5173 -o /index.html`, which handles opening the browser itself and does not need `serverReadyAction`. Coder picks one and documents choice in the PR description / handoff, not in the JSON (no comments allowed).

---

## Edge Cases to Handle

1. **Empty project list** — `projects: []` must show `.dashboard__empty` with a friendly message ("No projects yet"). Coder toggles visibility; Designer styles it.
2. **Missing optional fields** — if `recentActivity` is absent, the Coder must omit the element (not render `undefined`). If `lastUpdated` is absent, show `—`.
3. **Missing required fields** (`name`, `status`, `priority`, `owner`) — skip the card and `console.warn` with the offending `id`. Do not throw.
4. **Unknown `status` value** — fall back to a neutral badge (e.g., `.status-badge--unknown`) and warn. Designer provides a neutral style; Coder implements the fallback.
5. **Long project names** — `.project-card__name` must wrap gracefully; no horizontal overflow. Consider `overflow-wrap: anywhere;` and a two-line clamp only if design approves.
6. **Long `recentActivity` strings** — clamp to 2–3 lines with `-webkit-line-clamp` and provide `title` attribute for full text on hover.
7. **Many projects (50+)** — grid must remain performant; avoid layout thrash. No virtualization required at this scale, but avoid per-card box-shadow animation on scroll.
8. **Fetch failure** (JSON missing / malformed) — show `.dashboard__error` with the error message, do not leave the page blank. Coder catches; Designer styles.
9. **Accessibility contrast** — every badge and priority color must meet WCAG AA (4.5:1 for normal text, 3:1 for large text). Designer verifies with a contrast checker.
10. **Keyboard navigation** — cards are not interactive by default; if they become interactive (link to detail), they must have visible `:focus-visible` state.
11. **Date localization** — use `Intl.DateTimeFormat(undefined, { dateStyle: 'medium' })` so runner's locale is respected without hard-coding.
12. **Codespaces port forwarding** — `serverReadyAction` opens `http://localhost:5173/...`. In Codespaces, VS Code auto-forwards; confirm the Simple Browser or forwarded URL opens `index.html` and not a directory listing.
13. **Directory listing risk** — verify the static server serves `index.html` at `/` by default; if not, always route through `/index.html` in the launch URL (already the case above).
14. **JSON cache** — during iteration, browser may cache `project-data.json`. Coder should append `?v=${Date.now()}` on fetch during development, or set `cache: 'no-store'`.

---

## Validation Expectations

Manual (learner-run):

1. Run **Run Project Pulse Dashboard** from VS Code's Run and Debug panel.
2. Confirm the browser (or Simple Browser in Codespaces) opens `http://localhost:5173/index.html`, not a directory listing.
3. Confirm one `.project-card` appears per entry in `app/project-data.json` (count check).
4. Confirm each card shows: name, owner, status badge, priority, recent activity, formatted last-updated date.
5. Confirm status badges visually differ across On Track / At Risk / Delayed and match each card's data.
6. Confirm priority treatment differs across High / Medium / Low.
7. Resize the window to ~360px wide — confirm cards reflow to a single column with no horizontal scroll.
8. Open DevTools console — confirm zero errors and zero warnings (except intentional warnings from the missing-field edge cases if being tested).
9. Tab through the page — confirm visible focus states on any focusable element.
10. Temporarily replace `projects` with `[]` — confirm empty state renders. Restore afterward.
11. Temporarily break the JSON — confirm error state renders. Restore afterward.
12. Run an accessibility spot-check: use browser devtools contrast checker on each badge variant.

Automated checks are out of scope for this exercise; the brief prioritizes learners practicing orchestration, not test authoring.

Orchestrator's integration report should confirm each of the above and name the agent responsible for any follow-up.

---

## Open Questions

1. **Static server choice** — `serve` vs. `http-server`? Both are `npx`-runnable. Recommend `http-server` because `-o /index.html` opens the file directly and removes the need for `serverReadyAction` regex tuning. Confirm with Orchestrator before Coder writes `launch.json`.
2. **Dark mode** — should Designer implement `prefers-color-scheme: dark`? Brief doesn't require it. Recommend deferring unless the learner asks.
3. **Card interactivity** — are cards clickable (link to a detail view)? Brief implies read-only. Assuming non-interactive; revisit if learner requests.
4. **Sort / filter controls** — not in brief. Out of scope for v1.
5. **Field name: `lastUpdated`** — the exercise prompt lists "last-updated date" as a required display field, but the repo brief (`project-pulse-brief.md`) does *not* list it in the required JSON fields. Recommendation: include `lastUpdated` in `project-data.json` anyway to satisfy the display requirement; this is additive and does not violate the brief. Flag to Orchestrator for confirmation.
6. **Number of sample projects** — recommend 6 so every status × priority pair appears at least once without bloating the fixture. Confirm.
7. **Icon system** — pure CSS glyphs (e.g., a colored dot) vs. inline SVG in the badge? Recommend pure CSS to keep to the three `app/` files. Designer decides within `styles.css`.
8. **Port 5173 collision** — if the Codespace is already using 5173 for something else, Coder should fall back to 4173. Not blocking.
