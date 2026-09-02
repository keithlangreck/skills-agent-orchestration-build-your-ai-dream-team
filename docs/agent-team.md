# Agent team

To build Mona's Project Pulse dashboard, I'm using a four-agent custom team defined under `.github/agents/`, orchestrated with the GitHub Copilot CLI running in a Codespace.

## Orchestrator
- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Coordinates the Planner, Coder, and Designer. Breaks requests into phased, non-overlapping file scopes, decides what can run in parallel vs. sequentially, and reports integrated progress back to the learner. Does not implement work itself.
- **Definition:** `.github/agents/orchestrator.agent.md`

## Planner
- **Model:** Claude Opus 4.7 (copilot)
- **Responsibility:** Researches the codebase and docs, then produces an implementation plan with ordered steps, file assignments, dependencies, parallelizable work, edge cases, and validation expectations. Does not write code.
- **Definition:** `.github/agents/planner.agent.md`

## Coder
- **Model:** GPT-5.5 (copilot)
- **Responsibility:** Implements the dashboard logic and supporting files within its assigned scope, including creating `.vscode/launch.json` for Project Pulse so the app is easy to run and preview.
- **Definition:** `.github/agents/coder.agent.md`

## Designer
- **Model:** Gemini 3.1 Pro (copilot)
- **Responsibility:** Owns UI/UX, accessibility, and visual design for the dashboard — polished project cards, status badges, responsive layout, and deterministic CSS hooks like `.dashboard` and `.project-card`.
- **Definition:** `.github/agents/designer.agent.md`

All four agents avoid staging, committing, or pushing changes — git operations remain under the learner's control via Copilot CLI prompts.
