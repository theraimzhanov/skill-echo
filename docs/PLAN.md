# Skill Echo — Build Plan (Nov 19 → Nov 30, 2026)

Source: our idea doc (app.pdf) + official rules (firebasehybridaihackathon.devpost.com/rules).

## What the judges need (from the official rules)
- Android app using **Gemini Nano 4 via Firebase AI Logic**, with **App Check** (debug tokens OK).
- Other Firebase products encouraged: Firestore, Auth, Remote Config.
- Repo public on GitHub, **open-source LICENSE file** (Apache preferred), all source + setup
  instructions, Firebase AI Logic actually imported **and called** at runtime.
- **YouTube demo ≤ 3 min**, English, shows the app really running.
- Text description: how we used Firebase AI Logic, features, value proposition, what we built
  during the contest.
- Free test build (APK) for judges. No production keys / PII in repo or instructions.
- All team members added to the Devpost project.
- Submit by **Nov 30, 2:00 PM PT**. Commits after that are not evaluated.
- Judged equally on: Technological implementation (hybrid inference!), Design (complete
  product, not a demo), Potential impact, Quality of idea.

## Scope
**Core (must ship):** video → lesson via cloud Gemini · review/edit lesson · Live Coach camera
with automatic Nano checkpoint checks · spoken warnings (TTS) · offline coaching · consent-based
cloud fallback · inference-source badge · App Check.
**Include if time:** YouTube URL, record video in-app (local file upload comes first),
Firestore sync + anonymous Auth.
**Not now:** Gemini Live streaming, marketplace, iOS/Web, accounts/history beyond simple.

## AI routing (one place: `InferenceRouter`)
| Operation | Mode |
|---|---|
| Analyze expert video, generate lesson | `ONLY_IN_CLOUD` |
| Live checkpoint, private mode (default) | `ONLY_ON_DEVICE` |
| Live checkpoint, hybrid mode (user opted in) | `PREFER_ON_DEVICE` |
| "Detailed review" button | `ONLY_IN_CLOUD` |
| Short offline question about a step | `ONLY_ON_DEVICE` |

## Shared data contract (`domain/model/`) — agree before splitting work
```kotlin
data class Skill(val id: String, val title: String, val sourceType: SourceType,
                 val sourceUri: String?, val materials: List<String>, val steps: List<Step>)
enum class SourceType { YOUTUBE, LOCAL_VIDEO, RECORDED }
data class Step(val id: Int, val title: String, val startSeconds: Int, val endSeconds: Int,
                val instructions: String, val expectedVisual: String, val commonErrors: List<String>)
data class CheckResult(val stepId: Int, val status: CheckStatus, val errorType: String?,
                       val observed: String, val expected: String, val feedback: String,
                       val needsAnotherView: Boolean, val inferenceSource: String)
enum class CheckStatus { CORRECT, POSSIBLE_ERROR, UNCERTAIN }
```
Cloud Gemini returns `Skill` JSON (use a response schema). Nano returns `CheckResult` JSON.

## Milestones (each = one branch → PR → developer says "okay" → merge to master)

### M0 — Setup (Dev A, Nov 19) · `feature/firebase-setup`
- Firebase project "SkillEcho": register app `com.hackathon.skillecho`, enable AI Logic with
  Gemini Developer API, enable App Check (debug provider), download `google-services.json`
  (each dev keeps a local copy — never commit).
- Add google-services plugin + deps (versions from our doc, verify against the official
  Android hybrid setup guide before pinning):
  `firebase-ai:18.2.0`, `firebase-ai-ondevice:16.0.0-beta08`, `firebase-appcheck-debug:20.1.0`.
- Add `domain/model/` classes above + empty `AppContainer` + Compose Navigation with 5 empty
  routes (home, create, lesson, coach, summary).
- Exit: app builds and runs; App Check debug token registered.

### M1 — Prove Gemini Nano works (Dev A, Nov 19–20) · `feature/nano-checkpoint` ⚠️ highest risk
- Check model availability, trigger download, show status.
- `CheckpointAnalyzer.check(bitmap, step): CheckResult` — one image + prompt, JSON output.
- Debug screen: pick a photo → run check → show result + `inferenceSource`.
- Test 3 photos: correct part, reversed part, blurry view.
- Exit: on the real phone, source = ON_DEVICE and correct/reversed are told apart.
- **If Nano fails by end of Nov 20:** simplify the test object (big color-coded blocks) and use
  human-confirmed step progression. Do not sink more days into it.

### M2 — Video → lesson (Dev B, Nov 20–23) · `feature/video-import`, `feature/lesson-review`
- Create screen: pick local video (≤ ~10 MB, 2–5 min; inline limit is 20 MB incl. base64),
  then YouTube URL, then in-app CameraX recording (same pipeline).
- `SkillGenerator.generate(video): Skill` via cloud Gemini with JSON schema.
- Lesson review screen: edit steps/expected visuals/common errors, then Save.
- Room storage so lessons work offline. Home screen lists saved skills.
- Exit: team-owned 2–3 min video → 5-step lesson saved and visible offline.

### M3 — Live Coach (Dev A, Nov 21–25) · `feature/live-coach`
- CameraX preview full screen + current step + AI status + inference badge.
- `FrameSampler`: grab a frame every ~10 s (tune 8–15 s) only when no request is in flight.
- Rules: `POSSIBLE_ERROR` must repeat on a second frame before speaking a warning; dedupe the
  same warning; `UNCERTAIN` → "Show me a clearer view"; never auto-advance on one uncertain
  result; manual Recheck / Next / Prev / Pause buttons.
- TextToSpeech for warnings and "Step verified".
- Exit: offline (airplane mode), reversed part → spoken warning → fix → step verified.

### M4 — Hybrid + privacy (Dev A, Nov 26) · `feature/hybrid-fallback`
- Settings toggle Private / Hybrid; consent dialog before first cloud use.
- Private mode + Nano unavailable → "Local AI unavailable", nothing uploaded.
- "Detailed review" button (cloud) with consent.

### M5 — Integration & polish (both, Nov 26–28) · small `fix/*` branches
- Dev B: Summary screen (completed steps, corrections, unresolved checks), Firestore sync +
  anonymous Auth (optional), onboarding, empty/error states, consistent theme.
- Dev A: permissions, crash fixes, camera framing tips, performance.
- Both: run the full flow 5× on the real phone; fix anything that breaks.

### M6 — Submission (Nov 29) · `docs/submission`
- README: value proposition, how we use Firebase AI Logic (with file links), setup steps
  (create Firebase project, add own `google-services.json`, App Check debug token), test steps.
- Signed/debug APK in GitHub Releases. Set LICENSE visible in repo About.
- Record demo (script below), upload public to YouTube, fill Devpost, add both members.
- Nov 30 morning = buffer only. Submit before noon PT.

## Demo script (≤ 3:00)
0:00 problem → 0:20 import team-owned video (show YouTube/record options) → 0:45 cloud Gemini
builds 5 steps → 1:05 start Live Coach, turn internet off → 1:30 Nano catches reversed part,
spoken warning → 2:00 fix, checkpoint verified → 2:20 show ON_DEVICE badge, then cloud fallback
with consent → 2:40 finish + pitch. Only real model output on screen.

## Day-by-day
| Date | Dev A (Claude Code + Codex) | Dev B (Codex) |
|---|---|---|
| Nov 19 | M0 setup, start M1 | Read docs, design screens, Room + models after M0 merge |
| Nov 20 | M1 Nano proof (go/no-go) | M2 local video → cloud lesson |
| Nov 21–23 | M3 Live Coach | M2 lesson review, Room, Home, YouTube, recording |
| Nov 24–25 | M3 finish (TTS, dedupe, offline) | Summary screen, Firestore/Auth, UI polish |
| Nov 26 | M4 hybrid + privacy | Integrate lesson → coach → summary flow |
| Nov 27–28 | Test + fix | Test + fix |
| Nov 29 | Demo recording, APK | README, Devpost text |
| Nov 30 | Buffer, submit before 2 PM PT | Buffer |

## Compliance checklist (tick before submitting)
- [ ] Firebase AI Logic imported and called (cloud + on-device)
- [ ] Real Gemini Nano inference shown on a supported phone
- [ ] Consent-based cloud fallback works
- [ ] App Check configured
- [ ] English UI
- [ ] Public GitHub repo + Apache-2.0 LICENSE visible
- [ ] README with setup + test instructions, no secrets
- [ ] Public YouTube demo ≤ 3 min
- [ ] Test APK available free for judges
- [ ] Rights to every video/asset used (use team-owned videos in the demo)
- [ ] Both members on Devpost project
- [ ] Submitted before Nov 30, 2:00 PM PT
