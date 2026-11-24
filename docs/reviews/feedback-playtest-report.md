# DevLife — FeedbackPlaytest Report
## Data Routing & Validation (Chapter 1 rework)

**Type:** FeedbackPlaytest iteration (not a numbered Phase 4)
**Date:** 2026-11-24
**Branch:** `feedback-playtest` (base `260cb10`, final code `d59b1a6`, this report commit follows)
**Spec:** `E:\Download\DevLife_Data_Routing_Validation_Gameplay_Proposal.md`
**Plan:** `docs/superpowers/plans/2026-11-24-feedback-playtest.md`
**Engine:** Unity 6000.6.7f1

---

## 1. What was built

DevLife's node gameplay now supports practical data routing:

- A mission can author a **collection of data items** (file type, category, status, duplicate) and **multiple destinations** with acceptance rules.
- The graph runs **once per item**; each item carries its own properties and takes one TRUE/FALSE path.
- `IF` is configurable with **Field / Operator / Value** (equals, not equals, contains, empty, not empty), and chained IFs are exclusive (the condition port doubles as the flow signal).
- Routing is graded per item: correct destination, wrong destination, unprocessed, or delivered to more than one destination.
- Chapter 1 was reworked to teach the progression: 3 classic missions, then Type Sorting, Hidden Identity, Data Quality, and a routing Final Project.
- The game UI remains Thai (Noto Sans Thai), with a data inspector, an IF settings panel, destination labels, and the existing full-screen brief/victory flow.

## 2. Implemented vs the proposal

| Proposal area | Status |
|---|---|
| §3 Predefined Input/Output boundaries | Implemented for routing missions: one Input + one Output per destination, seeded on accept, non-deletable, preserved by Clear |
| §3.4 Per-item execution | Implemented via `ExecutionContext.Item`; the graph is evaluated once per item |
| §4 Data properties / misleading names | Implemented (type, category, status, duplicate); `hidden-identity` contains a `.mp4` whose actual type is Text |
| §5 Configurable IF | Implemented (field/operator/value); classic unconfigured IF keeps its old value semantics |
| §6 Misleading/invalid/duplicate data | Implemented in content; fairness data authored per item |
| §7 Per-item accuracy and mistakes | Implemented (correct / wrong / unprocessed / double, with per-item reports) |
| §8 Execution feedback | Per-item result list implemented; per-node current-item animation is not |
| §9 Nested modules | Boundaries/recursion/persistence preserved; ports remain boolean, nested configured IFs can read the current item (see limitations) |
| §10 Learning progression | Chapter 1 reworked through 6 main missions + Final Project |
| §11 UI/UX | Data panel, IF settings, destination labels, TRUE/FALSE port colours, Thai copy |
| §13 Acceptance | See §7 below for automated vs manual status |
| §14 Per-item default | Confirmed and implemented |

## 3. Architecture (smallest compatible change)

- **Ports stay boolean.** Per-item values are not pushed through ports; the current item is passed to every executor via `ExecutionContext.Item`. This preserves the Graph Core, validator, undo, modules and save format.
- **Configured IF** reads `field/operator/value` from the node's per-instance `Config`. In routing mode the condition input is the flow: `true = flow && match`, `false = flow && !match`, so an item cannot fall through multiple branches. A field-less IF fails the run with an actionable message instead of silently passing everything.
- **Boundaries** (`NodeInstance.Boundary`) are seeded by `RoutingGraphSeeder`; they are non-deletable, kept by Clear, and survive save/load (label, boundary, destination, config).
- **Evaluation** branches in `TestRunner`: classic missions keep the case runner; routing missions use `RoutingEvaluator`.
- **Save schema** is additive (`Label`, `Boundary`, node `Config`); classic saves load unchanged.
- **New node:** `OR` (two inputs, one result) is the minimum needed to merge Text/Image to one blog destination and multiple reject paths.

## 4. Content

| # | Id | Title | Type | Items | Destinations |
|---|---|---|---|---|---|
| 1 | `and-gate` | ประตู AND | Classic | 3 cases | — |
| 2 | `echo` | ส่งสัญญาณผ่าน | Classic | 2 cases | — |
| 3 | `assert-true` | ส่งผ่าน | Classic | 1 case | — |
| 4 | `smart-inbox` | คัดแยกไฟล์ | Routing | 4 | blog / video / reject |
| 5 | `hidden-identity` | ชื่อไฟล์หลอกตา | Routing | 5 (1 misleading) | blog / video / reject |
| 6 | `quality-check` | ตรวจคุณภาพข้อมูล | Routing | 5 (invalid + duplicate) | blog / reject |
| 7 | `checkout-flow` | ระบบรับเนื้อหาเว็บไซต์ | **Final routing** | 8 | blog / video / reject |
| — | `newsletter-filter` | ตรวจการส่งจดหมายข่าว | Optional classic | 3 cases | — |
| — | `login-check` | ตรวจหน้าล็อกอิน | Optional classic | 3 cases | — |

The Final Project exercises type, status, duplicate and a misleading filename in one pipeline.

## 5. Playtest guide (UX)

1. New game → room → computer → power on → Email → accept **ประตู AND**; press **รับทราบ** on the full-screen brief.
2. Complete the three classic missions (Input/IF/AND/Output).
3. Accept **คัดแยกไฟล์**: the Input and three destination nodes are already placed and cannot be deleted.
4. Press **ข้อมูล** to inspect the item table and the rules of each destination.
5. Add IFs; select one and press **ตั้งค่า IF**; cycle Field / Operator / Value; the node shows its rule and its ports read จริง/เท็จ in green/red.
6. Wire: `Input → IF(type=Video) → video`, false → `IF(type=Text)`, then Image, with an `OR` merging Text/Image to blog and the final false to reject.
7. Run: the result lists every item; a pass opens the victory panel (stars, next/replay/menu).
8. Continue through **ชื่อไฟล์หลอกตา** (do not trust filenames), **ตรวจคุณภาพข้อมูล** (status/duplicates), and the Final Project.
9. Check Clear keeps the boundaries, Undo restores the graph, and Save/reload restores configurations.

## 6. Tests

Run in an isolated copy at `fdeb42c`:

| Suite | Result | Exit | Evidence |
|---|---|---|---|
| EditMode | **204 / 204** | 0 | `Logs/editmode-results-playtest.xml` |
| PlayMode | **63 / 63** | 0 | `Logs/playmode-results-playtest.xml` |
New coverage includes: configured IF by field, chained exclusive branches, operators, item flow with/without an item, unconfigured IF failure, boundary seeding/promotion, boundary labels/config serialization, per-item correct/wrong/unprocessed/double, unassigned Output failure, node library hiding boundaries in routing mode, data panel/settings in the real scene, a full routing solve, Clear/Undo boundary survival, routing save/load, and chapter progression through all seven main missions.

## 7. Fresh-context review vs the docs — findings and fixes

A reviewer with no prior context compared the diff to the proposal. Dispositions:

**Important (fixed)**
1. **Unconfigured IF was an always-true switch** in routing missions → now the run fails with “IF ยังไม่ได้ตั้งค่า: เลือกฟิลด์/เงื่อนไข/ค่า”; test added.
2. **Player-added Outputs silently did not grade** → unassigned Outputs now fail the run up front; Input/Output entries are hidden from the routing palette; tests added.
3. **“Next Mission” bypassed jobs and boundary seeding** → the toolbar button was removed; the intended next-mission path is the victory button / job system. Tests were updated to switch missions directly.

**Minor (fixed)**
- Graph errors are appended to item failures; null authored items no longer break the pass condition.
- The IF settings panel refuses classic/no-field missions, and configuration changes are one undo entry.
- The brief shows **items/destinations** instead of “0 test cases” for routing.
- Checkout content now includes a valid Image and a corrupted Video so status is tested on both.
- Boundary seeding promotes an existing Input instead of adding a duplicate for legacy graphs.
- The data panel discloses that the graph highlight follows the last item while results list every item.

**Deferred / documented**
- Per-item, per-node execution animation (current item visible while it travels) is not implemented; only the post-run per-item list and last-item highlight exist.
- Accuracy is exposed as correct/total counts, not a percentage widget.
- Output cards show the destination label; acceptance rules live in the brief/data panel.
- A legacy Phase 3 save of the repurposed `smart-inbox`/`checkout-flow` may contain old nodes; boundaries are seeded and the evaluator explains unassigned Outputs, but the player should Clear once.
- Nested module ports remain boolean. A configured IF inside a Nest can see the current item because the item context is forwarded; this is not a media-collection contract.
- The Final Project's real graph solve and the chapter run are not both automated (the chapter test fabricates successful routing results; `smart-inbox` is solved for real in PlayMode).

## 8. Known limitations / NOT VERIFIED

- **Manual Game-view acceptance** of the data panel, settings card, Thai text fit, boundary layout and branch readability is outstanding.
- No playtest with a human beginner yet; the review predicts the two former traps (unconfigured IF, stray Output) were the first blockers.
- Visual execution still does not walk each item through nodes; it lists results and highlights the last run.
- Routing missions expose a fixed property set; free-text values and Contains on arbitrary strings are not available in the UI (dropdowns only).
- Star thresholds reuse the classic cost/accuracy formula; no per-mission routing-specific stars.
- Old saved graphs of repurposed missions need one Clear/restart.
- The `OR` node is a new type justified by routing merges; no other node types were added.

## 9. Playtest fix from the first screenshot

The first in-game screenshot showed the seeded boundaries still titled **“ส่งผลลัพธ์ (Output)”**, with delete (X) buttons visible and an editable `ค่า: จริง` control on the Input. Root cause: `NodeView` binds when the node is added, before the seeder/serializer applies `Label`, `Boundary` and `Config`.

Fix:
- `NodeView.RefreshIdentity()` re-applies the per-instance label, boundary delete visibility, and value control visibility.
- `GraphWorkspace.RefreshNodeViews()` is called after boundary seeding and after loading a job graph.
- Boundary nodes never show the close button or an editable value control.
- The PlayMode routing test now asserts the destination label on the Output card, hidden close buttons, and the hidden Input value control.

Verification after the fix: EditMode **204 / 204**, PlayMode **63 / 63**, exit 0. Re-open the job (or restart Play Mode) to see the corrected labels.

## 10. Verdict

**FeedbackPlaytest implementation complete for the automated scope.** The idea works end-to-end in tests and in the shipped scene flow, and the review's three Important traps are closed. It is ready for a human playtest focused on: can a beginner configure an IF without help, understand the TRUE/FALSE flow, and recover from Clear/restart; and whether the data panel and settings card are readable in the Game view.
