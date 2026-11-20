# AGENTS.md — rules for every AI agent and every human on Skill Echo

Read this file first, every session. It applies to Claude Code, Codex, and both developers.
Then read `docs/PLAN.md` (what to build) and `docs/HANDOFF.md` (where we are now).

## Project in one line
Android app (Kotlin + Jetpack Compose) for the Firebase #BuildHybridAI 2026 hackathon.
Cloud Gemini turns an instructional video into step checkpoints; Gemini Nano 4 watches the
learner through the camera (on-device, works offline) and warns about mistakes.
Goal: a premium-looking, fully working product — not a prototype. See "Quality bar" in `docs/PLAN.md`.

## Hard rules (never break)
1. **All AI calls go through Firebase AI Logic** (`com.google.firebase:firebase-ai` + the
   on-device / hybrid library). No direct Gemini REST calls, no other AI SDKs.
2. **No fake AI.** Never hardcode, mock, or prerecord model answers in app code. If the model is
   unavailable, show an honest error ("Local AI unavailable"). Test fakes live only in `src/test`.
3. **Never upload camera frames silently.** Private mode = `ONLY_ON_DEVICE`. Cloud fallback only
   after the user turns it on (consent). Always show the real `inferenceSource` in the UI.
4. **No secrets in git**: no `google-services.json`, App Check debug tokens, API keys,
   keystores, `local.properties`. They are in `.gitignore`; keep it that way.
5. **App must build before every commit**: `./gradlew :app:assembleDebug` (Windows:
   `.\gradlew.bat :app:assembleDebug`). Run `testDebugUnitTest` if you touched logic with tests.
6. **English only** in UI strings, code, commits, and docs. UI strings go in `strings.xml`.
7. **Versions come from official docs, then get pinned.** When adding a library, use the current
   stable version from the official Firebase / Android docs (Firebase via the BoM). Never copy
   versions from the idea PDF. Once it works, don't bump without both devs agreeing.
8. **Design is a deliverable.** Use the shared theme and components in `ui/theme` and
   `ui/components`; no hardcoded colors/sizes in screens. Every screen ships with loading,
   empty, error, and success states, dark mode, and accessibility (content descriptions,
   48dp targets). A screen that only works but looks unfinished is not done.
9. Only change files your task needs. Do not rewrite or "clean up" another person's area.

## Git workflow (mandatory)
- `master` = always working, demo-able code. **Nobody (human or agent) commits or pushes
  directly to `master`.**
- One branch per feature/function, cut from the latest `master`:
  `feature/<short-name>`, `fix/<short-name>`, `docs/<short-name>`.
  ```
  git checkout master && git pull
  git checkout -b feature/nano-checkpoint
  ```
- Commit small and often. Message format: `<area>: <what changed>` e.g.
  `coach: add frame sampler with 10s interval`.
- When the function is done and builds: push the branch and open a Pull Request into `master`.
  ```
  git push -u origin feature/nano-checkpoint
  gh pr create --base master --fill
  ```
- **Agents stop here.** Show the PR / diff to the developer and wait. Merge into `master`
  only after the developer explicitly says "okay". Agents never merge or force-push on their own.
- Before starting new work: `git checkout master && git pull`, then branch again.
- Merge conflicts: rebase your branch on `master`, fix, rebuild, push. Never `--force` to
  `master`. `--force-with-lease` only on your own feature branch.

## Who owns what (avoid editing the same files)
| Area | Owner | Paths |
|---|---|---|
| On-device AI, Live Coach, camera, TTS | **Dev A** (Claude Code + Codex) | `data/ai/CheckpointAnalyzer.kt`, `data/ai/InferenceRouter.kt`, `data/camera/`, `ui/coach/` |
| Video import, lesson generation, storage, Home/Lesson/Summary screens | **Dev B** (Codex) | `data/ai/SkillGenerator.kt`, `data/local/`, `data/firebase/`, `ui/home/`, `ui/create/`, `ui/lesson/`, `ui/summary/` |
| Shared (change only by agreement, small PRs) | both | `ui/theme/`, `ui/components/`, `domain/model/`, `navigation/`, `MainActivity.kt`, `app/build.gradle.kts`, `gradle/libs.versions.toml`, `AndroidManifest.xml` |

If you must touch a shared file, keep that change in its own small commit and mention it in
the PR description and `docs/HANDOFF.md`.

## Code conventions
- Package: `com.hackathon.skillecho`. Min SDK 28.
- MVVM: Compose screen → `ViewModel` (StateFlow) → repository/class in `data/`. No Hilt —
  construct dependencies by hand in one place (`AppContainer`) to save time.
- Coroutines for async work. Never block the main thread with AI calls.
- Live Coach loop: **one AI request in flight at a time.** Never queue checks while one runs.
- Model output is untrusted: parse JSON defensively; bad JSON = `UNCERTAIN`, never crash.
- Keep it simple: no new Gradle modules, no new libraries without a reason in the PR.

## Switching between Claude Code and Codex (or between developers)
**Start of every session (agent or human):**
1. Read `AGENTS.md`, `docs/PLAN.md`, `docs/HANDOFF.md`.
2. `git status`, `git branch --show-current`, `git log --oneline -10`.
3. Continue the task listed under "Next" in `docs/HANDOFF.md` for your branch — don't start
   something new.

**End of every session (before you switch tool or stop):**
1. Make sure it builds; commit work in progress on the feature branch (prefix `wip:` if unfinished).
2. Update your section in `docs/HANDOFF.md`: branch, done, next, build status, known problems.
3. Push the branch so the other tool/person can pull it.

Roles when both tools are used: Claude Code = spec, review, debugging hard problems;
Codex = writing feature code and fixing build errors. Either may write code, but only one agent
works on a branch at a time.
