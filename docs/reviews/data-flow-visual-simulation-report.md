# DevLife — Data Flow Visual Simulation Report

**Round:** Feedback Round 2 (Input icons + visual data flow)
**Date:** 2026-11-25
**Branch:** `feedback-playtest` — code `b6ae0c3`, docs follow
**Design:** `docs/superpowers/specs/2026-11-25-data-flow-visual-simulation-design.md`
**Plan:** `docs/superpowers/plans/2026-11-25-data-flow-visual-simulation.md`
**Brief:** `E:\Download\Devlife_Feedback_Round_2_Input_Icons_Visual_Simulation.md`
**Reference:** While True: learn() (the recording the developer supplied)
**Engine:** Unity 6000.6.7f1

---

## 1. What changed

- **Input card** shows one clean-room icon per data kind with a count badge (Text 2 · Image 1 · Video 1 · Unknown 1). Hovering a chip explains how that kind splits by the mission's other properties (`หมวด Cover 2 · สถานะ Corrupted 1`). Clicking a chip or **ดูข้อมูล** opens the existing full data list, so the four-property detail is unchanged. The old `N ไฟล์: …` text preview is gone.
- **Output cards** carry a vertical meter: one stacked segment per arrived item, coloured by kind, with `arrived / mission total` above it. It fills only as tokens arrive and resets on the next Run or mission change.
- **Run on a routing mission** grades as before, then replays every item's real path as a small icon token along the actual connection curve. Each node holds its token for its own `FlowSeconds`, so per-node speed differences are supported (visual only). A token that cannot continue stops at that node, turns red, and shows the reason the run produced. Delivered tokens drop into the Output meter.
- **Results** (text, mission panel, victory, save) appear only after the replay, or immediately on **ข้าม**/Esc. The graph is locked while replaying. Changing mission, leaving the work view, or disabling the workspace cancels playback without crediting the run to another mission.
- **Classic missions** keep the instant result and the old node-flash playback.

## 2. Data flow model

- `RoutingEvaluator` now keeps each item's `ExecutionResult` (`ItemRun`) from the graded loop; nothing is re-run for visuals.
- `ExecutionResult` exposes the per-node results the executor already cached.
- `DataFlowTrace` walks forward over the ports whose value was true, producing per-item steps and an end state: `Delivered`, `RejectedAtOutput` (wrong destination), `Blocked` (a node failed — its real error), or `DeadEnd` (no connection from the taken port). Fan-out produces several traces for one item, matching the grader's duplicate verdict.
- Because the trace reads the same values that produced the scoreboard, the animation cannot contradict the result, and no reason is invented (there is no "port accepts only text" message, because ports are boolean).

## 3. UI in words

- Input card: header, then a row of up to six 30 × 30 icon chips with an 18 px count badge at the top-right; the **ดูข้อมูล** button stays at the bottom.
- Output card: destination label as the header (as before), rules remain in the data panel/brief, and the meter sits against the right edge (14 × 110) with the count above it.
- While replaying, a **ข้าม** button appears at the top-right of the graph area; Esc does the same. Blocked tokens stay on the board with their reason until the next Run.

## 4. Verified cases

| Case | What the player sees |
|---|---|
| Passing run | Icons leave the Input, chips count down, meters reach 2/4 · 1/4 · 1/4 |
| Wrong destination | Token stops red at the Output with "ข้อมูลไปถึง … แต่โจทย์ต้องการ …" |
| Unconfigured IF | Token stops at that IF with "IF ยังไม่ได้ตั้งค่า: …" |
| Cut wire | Token stops at the node with "ไม่มี Connection จากพอร์ตนี้" |
| Duplicate delivery | Both arms fill, grade reports หลายปลายทาง, the Input badge drops once |
| Skip mid-run | Result appears at once, meters clear, a second skip changes nothing |
| Mission switch mid-run | Playback disappears with no result and no credit |
| Leaving the work view | Playback cancels instead of running off-screen |
| Classic mission | Result in the same frame, no tokens |

## 5. Tests

- EditMode **219 / 219**, PlayMode **72 / 72**, exit 0 at `b6ae0c3` (isolated copy, `p3-test.ps1 -Platform Both`).
- New coverage: kind mapping and counts, hover breakdown text, per-node results, per-item runs (including unprocessed), trace for correct/fan-out/blocked/dead-end/wrong-destination/dangling nodes, timing knobs, chips + meter + tooltip, run-before-result order, lock, skip once, mission-change cancel, leave-view cancel, blocked token position/reason, classic instant, queue with `MaxTokens = 1`.

## 6. Reviews

1. **Pre review (DeepSeek V4.1 Flash Free)** — design: per-item results must come from the graded loop (fixed), no invented port-kind reasons (removed), fan-out (added), Output-rejected (added), routing-only deferral, mutation gate, badge decrement, never-evaluated branches, module single hop.
2. **Pre review (same model)** — plan: task order could not compile, fake helper names/indices, impossible chip counts, an existing PlayMode assertion would break, and mission-change `onDone` could credit the new mission. All fixed before implementation.
3. **Main review (DeepSeek V4 Pro)** — one Important: Esc did not skip and leaving the work view kept the replay running invisibly (crediting a run the player thought they left). Fixed with a test. Also graded up: a blocked token had no test for its stop point/reason; added. Minors deferred, listed below.

## 7. Limitations kept

- Tokens are created and destroyed per item; `MaxTokens` caps concurrency, not allocations.
- Module nodes replay as a single hop; the nested path is not animated.
- A reached node whose own output port is false reports the dead-end reason (latent; module/AND only).
- The Output-value-false branch is unreachable while the walk follows only true ports.
- Only Text/Image/Video/Unknown have icons and slots; Audio/Data would need art and a slot.
- The brief's Input vertical meter is met by chips + badges on the Input with the meter moved to Output, as approved in the design.
- No per-item result-row replay; the token on the graph is the anchor.

## 8. Files

New: `DataKind.cs`, `DataKindCounts.cs`, `DataFlowTrace.cs`, `ItemRun.cs`, `FlowTiming.cs`, `DataChipView.cs`, `DataFlowPlayer.cs`, `Resources/DataKindLibrary.asset`, `Visuals/DataIcons/*.png`.
Changed: `ExecutionResult.cs`, `GraphExecutor.cs`, `RoutingEvaluator.cs`, `MissionResult.cs`, `NodeDefinition.cs`, `NodeView.cs`, `NodeView.Routing.cs`, `GraphWorkspace*.cs`, `ToolbarController.cs`, `DeskModeController.cs`, `CleanRoomBuilder.cs`, generated scene/prefabs/definitions, tests.
