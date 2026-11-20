# HANDOFF — current status

Update your section at the end of every session (see AGENTS.md). Newest entry on top.
Format: tool (Claude Code / Codex) · branch · Done · Next · Build status · Problems.

## Dev A — Foundation, on-device AI, Live Coach, hybrid
- Claude Code · `docs/team-plan`
  - Done: AGENTS.md, docs/PLAN.md, this file, LICENSE (Apache-2.0), .gitignore secrets.
  - Next: Step 0 `feature/foundation`, then Step 1 `feature/nano-checkpoint`.
  - Build: unchanged (docs only).
  - Problems: confirm the test phone supports Gemini Nano 4.

## Dev B — Video import, lessons, storage, summary, onboarding
- Not started. Next: Step 2 `feature/video-import` after Step 0 is merged.

## Decisions log
- Package stays `com.hackathon.skillecho` (Android Studio default).
- No Hilt; manual `AppContainer`.
- Library versions from official docs only (Firebase via BoM), pinned once working.
- `google-services.json` is not committed; judges get a test APK.
