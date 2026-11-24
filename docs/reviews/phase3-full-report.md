# DevLife Phase 3 — Full Implementation Report

**Project:** DevLife
**Phase:** 03 — Core Gameplay Loop & Learning Progression
**Engine:** Unity 6000.6.7f1 (URP 17.6.0, Test Framework 1.8.0, uGUI 2.6.0)
**Branch:** `main`
**Report date:** 2026-11-24
**Starting HEAD:** `8b61d44` (Phase 3 M0 baseline)
**Final code HEAD:** `33a1850` (documentation through `cde4f15`, this report commit follows)
**Spec:** `E:\Download\DevLife_Phase03_Core_Gameplay_Loop_AI_Agent_Specification.md`
**Roadmap:** `docs/superpowers/plans/2026-11-24-devlife-phase03-m0-audit-and-roadmap.md`

---

## 1. Executive summary

Phase 3 turned the existing prototype (graph editor, three missions, isometric room) into a persistent, motivating loop: accept work from a desktop, solve it with nodes, get scored and rewarded once, advance a chapter, spend money, and resume unfinished work after a restart.

All approved system work is implemented and covered by automated tests:

- versioned single-slot save/load with corruption recovery and migration hooks;
- job lifecycle with a dedicated Main Story slot, two Optional slots (third via career rank), cancellation policies, and per-job graphs;
- Email, Marketplace, Work, Profile and Store apps over one authoritative job/profile model;
- ordered Main Story with a gated chapter Final Project and a chapter completion summary;
- Money, XP, Player Level and Career Rank with distinct rules;
- mission-configurable evaluation, hint impact limited to stars/bonuses, replay XP policy;
- beginner-friendly node labels, execution-order visual highlighting, progressive feedback;
- create-from-scratch nested modules with manual ports, execution, recursion safety and persistence;
- a Full Developer Dashboard available from the desktop, the room HUD and Escape;
- a minimal hardware/software/cosmetic economy with a non-softlocking hardware gate.

**Verdict:** the Phase 3 **implementation** is complete against the approved scope. The **manual Unity Game-view acceptance** required by spec §20.3 has **not** been performed because the agent cannot see rendered output; that item is explicitly NOT VERIFIED and Phase 3 should not be signed off as fully accepted until a human completes it.

---

## 2. Repository state

- `main` at `33a1850`; working tree clean except untracked `Art/` (design art) and `graphify-out/` (local knowledge graph).
- `git diff --shortstat 8b61d44..HEAD`: **132 files changed, 18,920 insertions(+), 3,626 deletions(-)**.
  - 104 files added, 28 modified, 0 deleted.
  - Most deletions/insertions are the four full scene regenerations (`CleanRoom` ×3, `Room3D` ×1), which rewrite generated YAML.
- Runtime C# files under `Scripts/`: **86** (separate editor builders/tools live in `Assets/DevLife/Editor`); EditMode test files: **28**; PlayMode test files: **3**.
- Scene backups (git-ignored): `Backups/CleanRoom_20261124_155458.unity`, `_162027`, `_162808`, `Backups/Room3D_<timestamp>.unity`.

---

## 3. Test baseline vs final

| Suite | M0 baseline | Final | Failed | Skipped | Exit |
|---|---|---|---|---|---|
| EditMode | 123 / 123 | **190 / 190** | 0 | 0 | 0 |
| PlayMode | 43 / 43 | **58 / 58** | 0 | 0 | 0 |

Method: the tracked project was copied to an isolated directory and both suites were run with Unity batchmode:
`Unity -batchmode -projectPath <copy> -runTests -testPlatform <EditMode|PlayMode> -testResults <copy>/Logs/<platform>-results.xml`.
PlayMode tests additionally isolate saves with `GameSession.SaveDirectoryOverrideForTests`, so the developer's real save is never touched.

Evidence in `Logs/` (git-ignored, retained locally): `editmode-results-m0.xml`, `playmode-results-m0.xml`, `editmode-results-m1.xml`, `playmode-results-m1.xml`, `editmode-results-m2m3.xml`, `playmode-results-m2m3.xml`, `editmode-results-m4m7.xml`, `playmode-results-m4m7.xml`, `editmode-results-final.xml`, `playmode-results-final.xml`.

---

## 4. Scope status (spec §4.1)

| # | Approved scope item | Status |
|---|---|---|
| 1 | Persistent single-slot Auto Save/Load | Implemented (versioned, atomic, `.bak`, corrupt recovery) |
| 2 | Session continuity for jobs and graphs | Implemented (per-job graph, active job restore) |
| 3 | Main Story and Optional Job lifecycle | Implemented |
| 4 | Email + Marketplace over one job system | Implemented (one catalogue, one `JobService`) |
| 5 | Dedicated Main Story slot + limited Optional slots | Implemented (1 / 2 → 3) |
| 6 | Sequential Main Story + chapter gating | Implemented |
| 7 | Chapter Final Projects + completion summary | Implemented (one authored chapter) |
| 8 | Money + XP rewards, replay, career progression | Implemented |
| 9 | Mission-configurable multi-dimensional evaluation | Implemented (accuracy, cost, hints, per-mission config) |
| 10 | Beginner node cards + Visual Execution | Implemented (label + execution-order highlight) |
| 11 | Adaptive feedback + hint integration | Implemented (4-stage text + persisted hints) |
| 12 | Minimal Desktop + computer transition | Implemented (desktop in the desk view, existing transition) |
| 13 | Full Developer Dashboard (3 entry points) | Implemented (desktop, HUD, Escape) |
| 14 | Minimal hardware/software/cosmetic economy | Implemented (Store, gate, purchase persistence) |
| 15 | Nested Subgraph, create from scratch | Implemented (positional port mapping; see limitations) |
| 16 | Integration + regression tests | Implemented (190 EditMode + 58 PlayMode, all green) |

FUTURE-READY items were not built (certification, Convert Selection, automatic ports, shared modules, expanded OS, NPCs, advanced Profile, new data types, client review, multi-slot/cloud saves). OUT-OF-SCOPE items were not started.

---

## 5. Milestones

| Milestone | Commits | Outcome |
|---|---|---|
| M0 Audit | `8b61d44` (baseline) + roadmap doc | State verified: 123/43, reusable systems identified, gaps enumerated |
| M1 Save foundation | `7d59e72` … `4027836`, fixes `82dea98` | Profile/job/graph DTOs, atomic save, migration, corruption recovery, debounce; 26 new tests |
| M2 Jobs + Desktop | `df7b839`, `4926de3` | `JobService`, slots, Email/Marketplace/Work, per-job graphs, cancel/continue, scene rebuild #1 |
| M3 Chapters + Career | `df7b839`, `4926de3` (shared) | Chapter/final gating, Level, Rank, summary, next chapter unlock |
| M4 Evaluation/Rewards/Economy | `b79d5dc` | Rewards, replay XP, hints impact, Store, hardware gate, scene rebuild #2 |
| M5 Node learning UX | `b79d5dc` | Plain+technical node labels, visual execution order, 4-stage feedback, Hint button |
| M6 Nested modules | `d4fe8b0` | Create module, ports, breadcrumb, execution, cycle safety, save/load, scene rebuild #3 |
| M7 Dashboard + Room | `0848b2d`, `079cbe1` | Dashboard formatter, desktop Profile app, room HUD + Escape overlay, `Room3D` rebuild |
| Power workflow update + review fixes | `3026081`, `33a1850` | Monitor off/on, zoom, Stop Using vs Turn Off, desk exit, modal-safe Escape, raycast test |
| M8 Validation + report | `15c44eb`, `cde4f15` + this report | Full-suite validation, acceptance checklist, final report |

Review discipline: M1 had a fresh-context review that found 1 Critical + 4 Important data-loss paths (newer-save fallback, backup poisoning, unreadable primary, graph wipe on unknown definition, missing-primary backup). Each was fixed with a failing test first; five minors were deferred and recorded.

---

## 6. Complete gameplay loop (as built)

1. **Room hub** — `Room3D`: WASD sprite movement, orthographic isometric camera, prompt at the desk, `E` enters the computer.
2. **Desk transition and power** — `CleanRoom` loads in desk mode with the computer **off** (dark screen + “Turn On Computer”). Power on zooms into computer mode (the desktop). In computer mode, **Stop Using** zooms out with the computer still on, while **Turn Off Computer** zooms out and darkens the screen. At the desk view, **Stop Using** returns to the iso room.
3. **Desktop** — five apps: Email, Marketplace, Work, Profile, Store.
4. **Email** — full job feed (Main Story + Optional), accept from the list.
5. **Marketplace** — Optional jobs only, accept within slot limits.
6. **Work** — accepted jobs only; Open switches the active job graph; Continue/Restart dialog when a graph is saved; Cancel behind confirmation.
7. **Work Mode** — the existing Node Editor: add/drag/connect/knife/group/undo/zoom/pan, Mission Brief with Requirements for the Final Project, Hint button, Run.
8. **Run** — `TestRunner` evaluates every configured case against the real executor; results show Accuracy, Cost, Stars; Visual Execution highlights the actual evaluated order; failures append progressive feedback.
9. **Auto-complete** — a passing run completes an accepted job exactly once: rewards, best cost/stars, story progress, chapter check, save. Repeated runs cannot duplicate first-completion rewards.
10. **Chapter end** — the Final Project completing after every required Main mission shows the Chapter Summary, advances rank, unlocks the next chapter and the third Optional slot.
11. **Profile** — same dashboard from desktop, HUD or Escape; room reads the same save file.
12. **Persistence** — leave Work Mode or quit: profile, active job, every job graph, module data and purchases persist; relaunch resumes.

---

## 7. Job lifecycle and slots

### States
`Available`, `Accepted`, `Completed`, `Cancelled`, `Locked` (derived, never stored as a gate).

### Rules
- **Main Story:** one dedicated slot; strictly ordered by chapter and sequence; next mission locked until the previous is completed; Final Project locked until every required main mission in the chapter is complete.
- **Optional:** shared pool, capacity `PlayerProfile.OptionalSlotCount` (2 initially, 3 at the configured rank); independent of chapter gating.
- **No deadlines:** no dates, timers or expiry fields exist anywhere.
- **Cancellation:** confirmation dialog; the job's graph and hints are deleted and its slot freed. Main Story and `ReturnToMarketplace` jobs become Available again; `PermanentlyUnavailable` Optional jobs leave the pool. Completed work and rewards are never erased.
- **Continue/Restart:** opening an accepted job with a non-empty graph asks Continue or Restart; Restart clears only that job instance.
- **Replay:** replayable Optional jobs return to Available after completion and keep completion count and best results.

---

## 8. Chapters, career, level and rewards

### Authored content (Chapter 1 — Logic Fundamentals)

| Order | Id | Display | Type | Slots | Cases | Cost target |
|---|---|---|---|---|---|---|
| 1 | `and-gate` | AND Gate | Main | 2 | 3 | 5 |
| 2 | `echo` | Echo | Main | 1 | 2 | 2 |
| 3 | `assert-true` | Pass Through | Main | 1 | 1 | 2 |
| 4 | `smart-inbox` | Smart Inbox | Main | 2 | 4 | 5 |
| 5 | `checkout-flow` | Online Store Checkout | **Final Project** | 2 | 4 | 6 |
| — | `newsletter-filter` | Newsletter Send Check | Optional, replayable | 2 | 3 | 5 |
| — | `login-check` | Login Screen Check | Optional, permanently unavailable after cancel | 2 | 3 | 5 |

Final Project brief carries three explicit requirements (invalid cart, uncleared payment, valid orders only).

### Formulas
- **Stars:** 0 = fail; 1 = partial cases; 3 = all cases + cost ≤ target; 2 = all cases but over cost. Hint penalty = `min(hints × HintStarPenalty, MaxHintPenalty)` with a floor of 1 for a pass.
- **Money:** `BaseMoney + BonusMoney` only at 3 stars; Optional replays recalculate Money every completion.
- **XP:** `BaseXp + BonusXp` only at 3 stars. Optional replays use `ReplayXpPercent` (40%) of that total. Main Story completion rewards are granted once.
- **Level:** `1 + number of thresholds ≤ TotalXp` (thresholds 100/250/450/700). XP alone never changes ranks.
- **Rank:** advances one step per completed chapter if the rank's `RequiredCompletedChapter` is met (Beginner → Junior Developer after Chapter 1). Rank index ≥ 1 unlocks the third Optional slot.
- **Chapter complete:** all required Main missions + the Final Project passed. Completion is idempotent.

Hints never remove base Money/XP; they can only remove stars and bonuses. No Reputation, Energy or premium currency exists.

---

## 9. Save / Load

### Location and files
`Application.persistentDataPath/DevLife/`
- `save.json` — current save
- `save.json.bak` — previous save
- `save.corrupt-<UTC timestamp>.json` — preserved evidence of a failed load

### Format and versioning
- `JsonUtility` over public-field `[Serializable]` classes.
- `schemaVersion` = **1**; version 0 (unversioned legacy) migrates to 1 when a profile is present; newer versions are refused with a message and never overwritten or replaced by an older backup.
- Node graphs reference **definition ids**, not ScriptableObject references.

### Persisted inventory
- Profile: Money, TotalXp, RankIndex, ChapterIndex, OptionalSlotCount, HardwareLevel, CompletedMainMissions, UnlockedChapters, Purchases, ActiveJobId, Jobs.
- Job: MissionId, Status, graph, HintsUsed, BestCost, BestStars, CompletionCount, FirstRewardClaimed, TotalMoneyEarned, TotalXpEarned.
- Graph: node id, definition id, position, bool value, mission input-slot binding; connections by node/port ids.
- Module node: module name, port list (id/label/direction/max connections), nested graph (recursive).
- Not saved: selection, highlights, tooltips, animation state.

### Integrity
- Atomic write (temp file → `File.Replace` with a copy fallback), one `.bak`.
- Primary read failure → backup recovery; after recovery the good text is written back so the next save cannot poison `.bak`.
- Corrupt primary preserved before recovery; no silent reset.
- Debounced autosave (~2 s) plus saves on completion, rewards, chapter/rank changes, purchases, accept/cancel, focus loss and quit. Failed saves re-mark the debounce dirty.

---

## 10. Desktop, profile and room integration

**Desktop** (inside the monitor area of `CleanRoom`, visible only while zoomed into computer mode): header with Stop Using and Turn Off Computer, sidebar tabs, one content area. Email/Marketplace/Work list jobs from `JobService`; Profile shows the dashboard; Store lists purchasable items. **Work app** opens the job and enters Work Mode; Back returns to the desktop at the same zoom. At the zoomed-out desk view the room screen wallpaper is shown instead, keeping the desk exit visible.

**Computer state machine** (`DeskModeController`): `IsMonitorOn`, `IsZoomed`, `IsWorking` are independent flags.
- Entry from the room: monitor off, zoomed out, off panel + power button shown.
- Power On: monitor on, zoomed in, desktop shown.
- Stop Using (desktop): zoom out, monitor stays on, desktop visible at desk scale.
- Turn Off (desktop): zoom out, monitor off, dark screen + power button.
- Work app: node editor at the same zoom; Back returns to the desktop.
- Escape: editor → desktop; zoomed computer → stop using.
- Leave/Stop Using at the desk view: return to `Room3D`.

**Developer Dashboard** (one formatter, `ProfileDashboardText`): Developer Profile, Money, Level + XP to next, Career, Chapter, Missions, Optional slots, Hardware, Skills (distinct objectives of completed Main missions), Achievements (derived), Career Roadmap with done/current/next markers.

**Entry points:** desktop Profile app; room HUD “Profile” button; Escape (pause) in `Room3D`. The room overlay is read-only over the same save file; movement stops while it is open and resumes after closing.

**Room:** unchanged hub role — movement, computer interaction, no new rooms/NPCs/needs. `Room3D` was rebuilt only to add the HUD/overlay; the developer's hand-tuned monitor position and wall/monitor material colours were synced into `RoomSceneBuilder`.

---

## 11. Economy

- Hardware: “Second-hand Workstation” ($80, level 1), “Developer Workstation” ($200, level 2).
- Software: “Snap Connections Tool” ($60).
- Cosmetic: “Desk Plant” ($30) — no gameplay effect.
- Purchases deduct Money, are recorded in `Purchases`, persist, and hardware raises `HardwareLevel`.
- `MinHardwareLevel` is enforced in `JobService.CanAccept` (tested). Every shipped Main Story mission requires 0, so required progress cannot be softlocked; no Optional job requires hardware either (the gate exists and is ready for authored content).

---

## 12. Node learning experience

- **Cards:** plain meaning + technical label in the node header and library (`Emit a signal (Input)`, `Choose a path (IF)`, `Combine two checks (AND)`, `Deliver the result (Output)`); ports keep labels; the existing selection/connection/value/execution states remain.
- **Visual Execution:** after Run, the workspace clears all highlights and flashes every node in the executor's real `EvaluatedNodeIds` order; untaken branches stay dark. The result is never derived from the animation.
- **Adaptive feedback:** result summary → up to two mismatch cases → an authored concept hint → a guided checklist. The text explicitly states that an output mismatch cannot prove which node is wrong.
- **Hints:** authored per mission, revealed one at a time by the toolbar Hint button, counted on the job and persisted; stars/bonuses react, base rewards do not.

---

## 13. Nested Subgraph (M6)

- **Create from Scratch:** Module Bar → Create Module adds a module node and opens its empty body.
- **Ports:** + Input / + Output adds Bool ports with ids/labels; defined manually by the player.
- **Editing:** the existing workspace edits the nested model; “Back” returns to the parent via the breadcrumb “Project / Module N”. Nested modules are allowed; the history resets per model.
- **Execution:** `ModuleNodeExecutor` runs the nested graph with the parent executor set, maps module Input ports top-to-bottom to the nested Input nodes (by vertical position) and nested Output nodes back to module Output ports, and fails safely on recursive calls.
- **Persistence:** nested graph, ports and name are embedded in the job graph and restore with it. Capture always uses the root project model, so saving inside a module keeps the whole project.
- **Not implemented (future-ready):** Convert Selection to Module, automatic port generation, shared/cross-project modules.

---

## 14. Architecture and data responsibilities (spec §16)

| Responsibility | Implementation |
|---|---|
| Mission Definition | `MissionDefinition` ScriptableObject (+ `ChapterDefinition`, `ProgressionConfig`, `ShopCatalog`) |
| Job Instance | `JobState` (plain serializable, inside `PlayerProfile`) |
| Graph Model | `GraphModel` (unchanged core) |
| Node Definition / Instance | `NodeDefinition` / `NodeInstance` (unchanged) |
| Module Definition/Instance | `ModuleNodeDefinition` (runtime) + nested `GraphModel` |
| Execution | `GraphExecutor` + executor dictionary (+ `ModuleNodeExecutor`) |
| Evaluation | `TestRunner`, `StarEvaluator`, `FeedbackService` |
| Rewards | `RewardService` (idempotent grants, purchases) |
| Progression | `ProgressionService`, `JobService` |
| Profile | `PlayerProfile` + `ProfileDashboardText` |
| Save System | `SaveData`, `SaveStore`, `SaveMigration`, `GraphSerializer` |
| Desktop Apps | `DesktopController`, `JobListPanel`, `StorePanel` (thin views) |
| UI | Mission/Node/Job/Dashboard components; no business rules |

One `GameSession` scene object owns the profile, save store and job service; there is no new singleton, `DontDestroyOnLoad` or global manager.

---

## 15. Scene changes and editability

- `CleanRoom.unity` (regenerated): `DevLife_Core` now also holds `GameSession`; `Main Canvas` adds `Desktop` (header, sidebar tabs, Email/Marketplace/Work/Profile/Store windows, `JobRow` prefab rows), `Confirmation Dialog`, `Chapter Summary`, `Module Bar`, `Hint Button`. Existing workspace/monitor/toolbar/brief/output/debug objects remain.
- `Room3D.unity` (regenerated): adds `Interaction UI → Profile Button`, and a full-screen `Profile Overlay` (Dim, Card, Title, Body, Close). Existing room/desk/player/camera/prompt remain.
- The saved scenes remain the source of truth; builders are explicit and do not run automatically. Manual edits survive only by not rebuilding (documented), and this phase synced the developer's manual tweaks before rebuilding.

---

## 16. Files

### Created (source, assets, tests)
Persistence: `SaveData`, `GraphSerializer`, `SaveStore`, `SaveMigration`, `AutoSaveDebounce`.
Jobs/Progression/Economy: `JobTypes`, `JobService`, `ChapterDefinition`, `ProgressionConfig` (+ `RankDefinition`), `ChapterSummary`, `ProgressionService`, `ProfileDashboardText`, `ShopCatalog` (+ `ShopItem`), `RewardService`.
Session/Modules: `GameSession`, `ModuleNodeDefinition`, `ModuleNodeExecutor`.
Evaluation/UI: `FeedbackService`, `JobRowItem`, `JobListPanel`, `DesktopController`, `ConfirmationDialog`, `ChapterSummaryPanel`, `StorePanel`, `ModuleBarController`, `RoomProfileOverlay`.
Assets: `JobRow.prefab`, `SmartInbox`, `Checkout`, `Newsletter`, `LoginCheck`, `Chapter01`, `ProgressionConfig`, `Shop`.
Tests: `SaveDataTests`, `GraphSerializerTests`, `SaveStoreTests`, `AutoSaveDebounceTests`, `JobServiceTests`, `ProgressionServiceTests`, `RewardServiceTests`, `FeedbackServiceTests`, `ModuleNodeTests`, `ProfileDashboardTextTests`, `GameLoopSceneTests`.
Docs: M0 roadmap, M1 plan, M1/M2/M3/M4–M7 reports, this report.

### Modified (key)
`MissionDefinition`, `MissionLibrary`, `SaveData`, `NodeDefinition`, `GraphController`, `GraphExecutionRunner`, `StarEvaluator`, `ProgressionService`, `JobService`, `GraphSerializer`, `GraphWorkspace` (+ navigation/model swap), `NodeView`, `ToolbarController`, `MissionPanel`, `DesktopController`, `IsoPlayerController`, `CleanRoomBuilder`, `RoomSceneBuilder`, `CleanRoom.unity`, `Room3D.unity`, mission/node assets, `MissionAssetsTests`, `CleanRoomSceneTests`, `Room3DSceneTests`.

### Deleted
None.

Full machine-readable inventory: `git diff --name-status 8b61d44..33a1850` (104 added meta+asset files included).

---

## 17. Tests and verification

### Automated coverage (selected)
- Save: roundtrip, missing file, corrupt file, newer schema refusal (with and without backup), backup poisoning, unreadable primary, legacy migration, partial JSON, null save, per-job graph independence, career/chapter persistence.
- Jobs: ordering/locks, final gate, slot capacity 2→3, accept/cancel/complete, both cancel policies, replayable completion + best results, hardware gate, optional independence.
- Progression/rewards: level thresholds, chapter completion, rank/slot/unlock idempotency, summary objectives/stars/reward sums, money/XP/bonus, replay XP, hint star penalty keeps base rewards, purchases.
- Modules: execution + port mapping, recursion failure, nested serialization/roundtrip execution.
- Scene (PlayMode): desktop tabs, Email accept, Marketplace accept → Work Mode, job switching without graph bleed, cancel confirmation/deletion, Continue/Restart, chapter run to summary + rank, rewards/hint/store effects, module create/ports/open/back/save, room profile overlay from a save file, room↔desk↔work transition, computer power-off/on/stop/turn-off flow, and a raycast check that the desk exit is actually clickable with the computer on.

### Manual coverage
**Not performed.** Rendered UI quality (text fit, layout inside the 859×638 monitor area, glow readability, transition feel) and the §20.3 walkthrough with screenshots require a human in the Unity Game view. No screenshots are claimed.

---

## 18. Known limitations / NOT VERIFIED

1. **Manual Game-view acceptance and screenshots: NOT VERIFIED.**
2. Boolean-only data; no Number/String/Comparator nodes.
3. Visual Execution shows the evaluated node order and final output, not intermediate per-port values.
4. Job and Store lists do not scroll; the shipped content fits, larger catalogues will need a ScrollRect.
5. Module ports map positionally to nested Input/Output nodes; arbitrary per-port wiring is not supported. Port edits are not undoable and the undo history resets on model switches.
6. One chapter of content; later chapters/ranks exist as data/config only.
7. Reward/level numbers are plausible placeholders, not playtest-balanced.
8. Achievements are derived labels, not a separate achievement economy.
9. Save failure is surfaced in the console and the room overlay; the desktop has no guided recovery dialog.
10. No adaptive tutorial scaling beyond authored hints (spec §7.3 theme is only partially expressed).
11. The Final Project uses authored requirements plus one acceptance check, not separately submitted stages.
12. Certification, Convert Selection, automatic ports, shared modules, multi-slot/cloud saves, NPCs and expanded OS features are intentionally absent.
13. Unity prints a `Serialization depth limit 10` warning while scanning the nested module DTOs in PlayMode; module save/load tests pass and the warning is a console-cleanliness item.
14. The desktop is intentionally visible only in zoomed computer mode; the zoomed-out desk view shows the room screen so the desk exit stays free.

---

## 19. Deferred ideas (after Phase 3)

Certification/certificates; Convert Selection to Module with automatic ports; module assets shared across jobs; numeric/string data types and more node categories; scrolling lists and richer desktop polish; client review/revision workflow; cloud/multi-slot saves; NPC clients and interactive furniture; expanded Profile/portfolio; economy and hint balancing from playtests; automated screenshot pipeline for CI.

---

## 20. Acceptance checklist (spec §19)

### 20.1 Core loop
- [x] Enter the computer from the isometric room
- [x] Email receives client/story jobs
- [x] Marketplace browses/accepts Optional Jobs
- [x] Email and Marketplace use the same job system
- [x] Open an accepted job in Work Mode
- [x] Build and run a Node solution
- [x] Retry indefinitely without losing the graph or paying a penalty
- [x] A passing graph auto-completes the job exactly once
- [x] Completion grants rewards and updates progression
- [x] Leave Work Mode and resume the room without losing state

### 20.2 Job lifecycle
- [x] Dedicated Main Story slot
- [x] Main Story one sequential quest at a time
- [x] Two Optional slots initially
- [x] Third Optional slot unlocks from the career milestone
- [x] Multiple Optional Jobs accepted, one active in the editor
- [x] No deadlines or expiry penalties
- [x] Main Story cancellation requires confirmation and can be reaccepted from the beginning
- [x] Optional cancellation deletes that job's progress and returns the slot
- [x] Optional reavailability follows job configuration
- [x] Switching jobs does not mix or lose graphs

### 20.3 Learning and Node Editor
- [x] Mission design follows Learn → Practice → Apply where appropriate
- [ ] Early/mid/late support adapts as the player advances (authored hints exist; no scaled tutorial)
- [x] Node cards show plain meaning and technical label
- [x] Node interaction remains beginner-friendly and pure node-based
- [x] Visual Execution shows actual execution order (per-node values not shown)
- [x] Adaptive Feedback is progressive and does not invent a precise error source
- [x] Hint usage is recorded and affects Stars/bonus per mission configuration
- [x] Base completion Money/XP are not removed by hints
- [x] New Nodes/data types are justified by mission requirements (none added)

### 20.4 Evaluation and rewards
- [x] Mission evaluation is configurable by mission
- [x] Correctness criteria are explicit and deterministic
- [x] Performance metrics are not all mandatory for every mission
- [x] Money and XP are the only reward currencies
- [x] Base XP is granted for successful completion
- [x] Performance bonuses follow configured metrics
- [x] Player Level is XP-based
- [x] Career Rank requires knowledge and applied project completion
- [x] Main Story first-completion rewards cannot be duplicated
- [x] Optional replay Money is calculated on every successful replay
- [x] Optional replay XP is reduced after the first completion
- [x] Best Cost retains the best result

### 20.5 Chapter and Final Project
- [x] Main Story missions unlock in order
- [x] Final Project locked until all required Main Missions pass
- [x] Final Project uses a client brief and explicit requirements
- [x] At least one Final Project integrates more than one learned concept
- [x] Chapter Completion Summary shows results and learned concepts
- [x] Completion unlocks the next Chapter
- [x] Certification remains future-ready only

### 20.6 Save / Load
- [x] One Auto Save slot works
- [x] Save schema is versioned
- [x] Profile, rewards, progress, slots, mission results and upgrades persist
- [x] Multiple accepted Optional Jobs persist independently
- [x] In-progress graph and nested graph data persist
- [x] Continue and Restart behave as specified
- [x] Reload does not duplicate rewards or progression events
- [x] Missing/corrupt/old save data is handled safely

### 20.7 Computer, Profile and economy
- [x] Minimal Desktop provides Email, Marketplace, Work, Profile (plus Store)
- [x] Computer transition is brief and interrupt-safe
- [x] Profile Dashboard is accessible from Desktop, HUD and Escape
- [x] All Profile views use one authoritative data source (same formatter; room reads the save)
- [x] Dashboard displays Level/XP, Career Rank, Money, Chapter, skills, achievements, roadmap
- [x] Hardware/software purchases update persistent state
- [x] Hardware gates do not softlock required Main Story content
- [x] Room customization is cosmetic only
- [x] No Full OS Simulation

### 20.8 Nested Subgraph
- [x] Create an empty Module from scratch
- [x] Define Module Input/Output Ports manually
- [x] Enter/exit the nested graph via clear navigation
- [x] Parent graph executes the Module and receives results
- [x] Invalid/cyclic nesting is handled safely
- [x] Nested data survives save/load
- [x] Convert Selection remains unimplemented

### 20.9 Quality and maintainability
- [x] Existing Graph Core and working systems were reused
- [x] No duplicate job/mission/evaluation/reward/profile/save systems
- [x] Business logic lives outside UI scripts
- [x] `CleanRoom` remains inspectable/editable in Edit Mode
- [x] Builder does not silently overwrite the saved scene
- [x] EditMode and PlayMode suites pass; exact counts reported (190/190, 58/58)
- [ ] Manual test evidence covers the integrated loop (NOT VERIFIED)
- [x] Known limitations and unverified behavior disclosed

---

## 21. Post-report change: computer power workflow (2026-11-24)

### Requested flow
1. Press E at the PC → desk view, **computer off** (dark screen + power button).
2. Press the power button → screen on, **zoom into computer mode** (desktop).
3. In computer mode: **Stop Using** → zoom out with the computer still on; **Turn Off Computer** → zoom out and the screen goes dark.
4. At the zoomed-out desk view → **Leave Desk** returns to the iso room.
5. Work app → node editor; Back → desktop at the same zoom; Escape in the editor → desktop; Escape at computer mode → zoom out; Escape at the desk view → leave to the room.

### Implementation
- `DeskModeController` now has three independent flags: `IsMonitorOn`, `IsZoomed`, `IsWorking`. Power On sets on+zoomed, Stop Using zooms out and keeps the monitor on, Turn Off zooms out and darkens it, Leave returns to `Room3D`.
- `CleanRoom` gained a `Monitor Off` CanvasGroup (dark screen + “Computer is off” + hint), a `DesktopUi` CanvasGroup, and a two-action desktop header (Stop Using / Turn Off Computer). The room screen's leave button is labelled **Leave Desk** and is kept as the last sibling so it is never buried.
- The desktop is rendered **only while zoomed into computer mode**; at the zoomed-out desk view the room screen wallpaper is shown, so the desk exit cannot be covered by job rows. (This also keeps the “monitor on” desk view visually distinct from “monitor off”.)
- Escape is ignored while a `ConfirmationDialog` is open (`ConfirmationDialog.AnyOpen`), so a cancel dialog cannot be dismissed *and* zoom the desk at the same time.
- The obsolete `RoomFlowScenePatch` menu item now refuses to run and points to the builder instead of generating the old layout.

### Fresh-context review (sub-agent) and fixes
A reviewer with no prior context reviewed `cde4f15..3026081`:
1. **Important — desk exit covered:** the desktop (opaque, inside the room screen) sat above the Leave button at the monitor-on desk view; the player could be stranded. Fixed by showing the desktop only while zoomed and keeping Leave last; a raycast test now pins it.
2. **Important — Escape dead at the desk view:** Escape did nothing exactly where an exit was needed. Fixed: Escape at the desk view now leaves to the room; modal dialogs take priority.
3. **Important — test gap:** tests asserted flags but never that the exit was hit-testable. Fixed with `Desk_LeaveButtonStaysClickableWhileComputerIsOn`, which raycasts the button centre (and exposed Unity's hierarchy-depth ordering, driving the desktop-visibility fix).
- Minors applied: renamed the desk button to “Leave Desk”; disabled the stale scene patch; desk exit no longer depends on `SceneFlow.ReturnScene`.
- Declined to judge (rendered scale/feel, header spacing, off-panel contrast) remains manual.

### Verification
EditMode **190 / 190** and PlayMode **58 / 58**, exit 0, at `33a1850` (`Logs/editmode-results-final.xml`, `Logs/playmode-results-final.xml`). New PlayMode coverage: off-start → power on → editor → back → stop using → turn off → leave, and the leave-button raycast while the computer is on.

### Known warning
PlayMode output still contains Unity's `Serialization depth limit 10 exceeded at NodePortData.Direction` warning from scanning the nested module DTOs. Module save/load tests pass (JsonUtility handles the data), so this is a console-cleanliness item, not a functional failure; it is recorded for a later cleanup.

---

## 22. Final statement

Phase 3's approved implementation is delivered and green at `33a1850` (190 EditMode + 58 PlayMode, exit 0). The remaining gate is human acceptance of the rendered experience (spec §20.3): open the project, play the loop end-to-end, capture screenshots/video, and record any layout or clarity fixes. Until that is done, the correct status is **"Phase 3 implementation complete; manual acceptance outstanding"**, not an unconditional sign-off.
