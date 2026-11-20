# HANDOFF — current status

Update your section at the end of every session (see AGENTS.md). Newest info on top.
Format: date · tool (Claude Code / Codex) · branch · Done · Next · Build status · Problems.

## Dev A — On-device AI + Live Coach
- 2026-11-19 · Claude Code · `docs/team-plan`
  - Done: AGENTS.md, docs/PLAN.md, this file, LICENSE (Apache-2.0), .gitignore secrets.
  - Next: M0 (Firebase setup + shared models), then M1 `feature/nano-checkpoint`.
  - Build: not changed (docs only).
  - Problems: need to confirm which test phone supports Gemini Nano 4.

## Dev B — Video import + lessons
- (not started) Next: M2 `feature/video-import` after M0 is merged.

## Decisions log
- 2026-11-19: Package stays `com.hackathon.skillecho` (Android Studio default, not `com.skillecho.app`).
- 2026-11-19: No Hilt; manual `AppContainer`.
- 2026-11-19: `google-services.json` is not committed; judges get a test APK.
