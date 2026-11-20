# Skill Echo — Build Plan

**Watch. Practice. Get corrected. Master it.**
Upload or record an instructional video → cloud Gemini turns it into an interactive, step-by-step
skill → the learner starts **Live Coach**, the phone camera watches the work, and **Gemini Nano 4
on-device** checks each checkpoint, speaks warnings about mistakes, and keeps working offline.

We build continuously, step by step. Each step = one branch → PR → developer says "okay" → merge
to `master`. Do not start the next step until the current one builds and works on a real phone.

## Quality bar (applies to every step)
We are building a **finished, premium product**, not a tech demo. Judges score Design and
Technological Implementation equally with Impact and Idea.

**Design**
- Distinct visual identity: custom Material 3 theme (own color scheme, typography scale, shapes),
  not the default purple template. Light + dark mode.
- Every screen designed before coding (Claude proposes layout + states; Figma optional).
- Every screen has all states done: loading (skeletons / progress with real stage text),
  empty, error with a retry action, success.
- Motion with purpose: animated step transitions, checkpoint "verified" feedback, warning
  pulse on the camera overlay, smooth navigation transitions.
- Live Coach is the hero screen: full-bleed camera, clear step card, large readable status,
  inference badge (ON-DEVICE / CLOUD), big touch targets usable with dirty/busy hands.
- Accessibility: content descriptions, 48dp targets, contrast AA, works with TalkBack,
  font scaling; spoken feedback via TextToSpeech.
- Onboarding that explains the camera setup (tripod, framing) and privacy mode in 2–3 screens.

**Functionality**
- Real AI only, real inference source shown, honest errors.
- Never crashes on bad model output, no network, denied permissions, or missing Nano model.
- Fast and smooth: no work on the main thread, one AI request in flight, camera never freezes.
- Works offline for coaching once a lesson is saved.
- Unit tests for parsing, routing, and the coach decision logic.

## Contest requirements (from the official rules)
- Android app using **Gemini Nano 4 via Firebase AI Logic**, with **App Check** (debug tokens OK).
- Other Firebase products encouraged: Firestore, Auth, Remote Config.
- Public GitHub repo, open-source LICENSE (Apache) visible in About, all source + setup
  instructions; Firebase AI Logic must be imported **and called** at runtime.
- Public YouTube demo ≤ 3 minutes, English, showing the app really running.
- Text description: how we use Firebase AI Logic, features, value proposition.
- Free test build (APK) for judges. No production keys or personal data in repo.
- Both team members added to the Devpost project.
- Judging (equal weight): Technological implementation (hybrid inference), Design (complete
  product), Potential impact, Quality of idea.

## Libraries & versions
- Do **not** copy versions from the idea PDF. When adding a library, take the current stable
  version from the official docs (Firebase Android setup / Firebase AI Logic hybrid guide,
  Android Developers) and add it to `gradle/libs.versions.toml`.
- Use the **Firebase BoM** for Firebase libraries. The on-device AI library is experimental:
  follow the official hybrid inference guide exactly for its artifact and version.
- Once a version works, it is pinned. Upgrades only with both developers' agreement.

## Scope
**Core:** video → lesson via cloud Gemini · review/edit lesson · Live Coach camera with automatic
Nano checkpoint checks · spoken warnings · offline coaching · consent-based cloud fallback ·
inference-source badge · App Check · polished UI on every screen.
**Also in:** local video upload, YouTube URL, in-app recording, session summary, simple anonymous
Auth + Firestore sync.
**Later (only if everything above is excellent):** Gemini Live streaming, marketplace, iOS/Web.

## AI routing (single place: `InferenceRouter`)
| Operation | Mode |
|---|---|
| Analyze expert video, generate lesson | `ONLY_IN_CLOUD` |
| Live checkpoint, private mode (default) | `ONLY_ON_DEVICE` |
| Live checkpoint, hybrid mode (user opted in) | `PREFER_ON_DEVICE` |
| "Detailed review" button | `ONLY_IN_CLOUD` |
| Short offline question about a step | `ONLY_ON_DEVICE` |

Privacy: private mode never uploads frames — if Nano is unavailable, show "Local AI unavailable".
Hybrid mode asks for consent before the first cloud use.

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

## Screens
1. **Onboarding** — what Skill Echo does, camera setup tips, privacy mode choice.
2. **Home** — continue lesson, saved skills (available offline), create new.
3. **Create Skill** — YouTube link / upload video / record video; staged progress
   ("Uploading → Watching video → Building steps").
4. **Review Lesson** — edit steps, expected visuals, common mistakes; publish.
5. **Skill Details** — materials, difficulty, steps, source link + timestamps, start coaching.
6. **Live Coach** — camera, step card, AI status, inference badge, warnings, pause / recheck /
   next / previous.
7. **Session Summary** — completed steps, corrections made, unresolved checks, user-confirmed
   completion (never claim "mastery" from AI checks alone).
8. **Settings** — Private / Hybrid mode, voice on/off, check interval.

## Steps

### Step 0 — Foundation (Dev A) · `feature/foundation`
- Firebase project: register `com.hackathon.skillecho`, enable AI Logic (Gemini Developer API),
  enable App Check with debug provider. Each dev keeps `google-services.json` locally.
- Add Firebase BoM, AI Logic, on-device AI, App Check, Navigation Compose, CameraX, Room
  (current official versions).
- `domain/model/` classes, `AppContainer`, navigation graph with all screen routes.
- **Design system:** theme (colors, type, shapes, dark mode), shared components
  (buttons, step card, status chip, inference badge, loading/empty/error views).
- Exit: app runs, navigates between placeholder screens in the final theme.

### Step 1 — Prove Gemini Nano works (Dev A) · `feature/nano-checkpoint` ⚠️ highest risk
- Model availability check, download with progress, status UI.
- `CheckpointAnalyzer.check(bitmap, step): CheckResult` — one image + prompt, JSON output.
- Debug screen: pick photo → check → result + `inferenceSource`.
- Test: correct part, reversed part, blurry view.
- Exit: on the real phone, source = ON_DEVICE and correct vs reversed are told apart.
- If Nano cannot do it reliably: simplify the training object (large color-coded blocks) and
  use human-confirmed step progression. Don't sink more time than this step deserves.

### Step 2 — Video → lesson (Dev B) · `feature/video-import`
- Create Skill screen with the three sources: local video first (keep files small — inline
  request limit is 20 MB including base64), then YouTube URL, then CameraX recording.
- `SkillGenerator.generate(source): Skill` via cloud Gemini with JSON response schema.
- Exit: team-owned 2–5 min video → structured 5-step skill.

### Step 3 — Lesson review & storage (Dev B) · `feature/lesson-review`
- Review Lesson editor, Skill Details, Home list.
- Room persistence so saved skills work offline.
- Exit: create → edit → save → reopen offline.

### Step 4 — Live Coach (Dev A) · `feature/live-coach`
- CameraX preview + step card + status + inference badge.
- `FrameSampler`: one frame every ~8–15 s (tune on device), only when no request is in flight,
  preferably when the image is stable.
- Decision logic: `POSSIBLE_ERROR` must repeat on a second frame before a spoken warning;
  dedupe repeated warnings; `UNCERTAIN` → "Show me a clearer view"; never auto-advance on one
  uncertain result; manual Recheck / Next / Previous / Pause.
- TextToSpeech for warnings and "Step verified".
- Exit: in airplane mode, reversed part → spoken warning → fix → step verified.

### Step 5 — Hybrid & privacy (Dev A) · `feature/hybrid-fallback`
- Settings: Private / Hybrid; consent dialog; "Detailed review" in cloud.
- Clear handling for: Nano not supported, model downloading, quota hit, no network.

### Step 6 — Summary, onboarding, sync (Dev B) · `feature/summary-onboarding`
- Session Summary, Onboarding, Settings screen UI.
- Anonymous Auth + Firestore sync of skills (optional Remote Config for check interval).

### Step 7 — Polish pass (both) · small `fix/*` and `design/*` branches
- Walk every screen in light/dark, small/large font, TalkBack; fix anything that looks unfinished.
- Animations and transitions, haptics on warnings, final app icon and splash.
- Run the full flow (import → review → coach → summary) many times on the real phone; fix
  every crash, wrong warning, permission and framing issue.

### Step 8 — Submission (both) · `docs/submission`
- README: value proposition, screenshots, how we use Firebase AI Logic (with file links),
  setup (own Firebase project + `google-services.json`, App Check debug token), test steps.
- APK in GitHub Releases; LICENSE visible in repo About.
- Record demo (script below), upload public to YouTube, fill Devpost, add both members.

## Demo script (≤ 3:00)
Problem → import team-owned video (show YouTube/record options) → cloud Gemini builds the skill →
start Live Coach, turn internet off → Nano catches a reversed part, spoken warning → learner fixes
it, checkpoint verified → show ON-DEVICE badge, then cloud fallback with consent → finish,
summary, pitch. Only real model output on screen.

## Compliance checklist
- [ ] Firebase AI Logic imported and called (cloud + on-device)
- [ ] Real Gemini Nano inference shown on a supported phone
- [ ] Consent-based cloud fallback works
- [ ] App Check configured
- [ ] English UI
- [ ] Public GitHub repo + Apache-2.0 LICENSE visible
- [ ] README with setup + test instructions, no secrets
- [ ] Public YouTube demo ≤ 3 min
- [ ] Test APK available free for judges
- [ ] Rights to every video/asset used (team-owned videos in the demo)
- [ ] Both members on Devpost project
- [ ] Submitted before the contest deadline
