# Primitives that follow the surface: the tests, from the checks that train nothing to the four finalists

**Contents**

- [1. Scope and the owner's decisions](#1-scope-and-the-owners-decisions)
- [2. Step 1: the work that trains nothing](#2-step-1-the-work-that-trains-nothing)
  - [2.1 Checks (a) and (b), and the fourth nine-camera model](#21-checks-a-and-b-and-the-fourth-nine-camera-model)
  - [2.2 Check (c): one camera fit per held-out copy, on the background](#22-check-c-one-camera-fit-per-held-out-copy-on-the-background)
  - [2.3 Checks (d) and (e): two seeds' disagreement where the shell is, and colour between cameras](#23-checks-d-and-e-two-seeds-disagreement-where-the-shell-is-and-colour-between-cameras)
  - [2.4 Check (f): a depth channel's gradient in the colour pass against Q-226's device](#24-check-f-a-depth-channels-gradient-in-the-colour-pass-against-q-226s-device)
  - [2.5 The three reads: FreeTimeGS++, SparseCam4D, SSA-3DGS](#25-the-three-reads-freetimegs-sparsecam4d-ssa-3dgs)
  - [2.6 Step 1's review: the checks' readings and the owner's go for the baseline](#26-step-1s-review-the-checks-readings-and-the-owners-go-for-the-baseline)
- [3. Step 2: the baseline, the wrapper script and its parity runs](#3-step-2-the-baseline-the-wrapper-script-and-its-parity-runs)
- [4. Step 3: finalist 1, the cameras corrected once](#4-step-3-finalist-1-the-cameras-corrected-once)
- [5. Step 4: finalist 2, the alignment-tolerant pixel loss](#5-step-4-finalist-2-the-alignment-tolerant-pixel-loss)
- [6. Step 5: finalist 3, surface terms without a prior](#6-step-5-finalist-3-surface-terms-without-a-prior)
- [7. Step 6: finalist 4, a normal prior on top of finalist 3's flattening](#7-step-6-finalist-4-a-normal-prior-on-top-of-finalist-3s-flattening)
- [8. The recommendation](#8-the-recommendation)
- [9. The sources](#9-the-sources)

A results document, written from 2026-11-16 on PRD #16 of openftgs-plus and its tickets #17 to #29. It is the run of the test plan in section 5 of `docs/research/surface-following-primitives.md`, which this document calls *the research document*: the record of PRD #1, which surveyed techniques against the shell on private scene A, chose four finalists and tested none of them (its sections 4.3, 5 and 6). The owner asked for this run at review 2 (the research document, section 7.7). It runs the plan in section 5.7's order: first the work that trains nothing (section 5.1), then the baseline, then the four finalists one at a time, and it ends in a recommendation per finalist that ran: adopt, drop, or test in combination. It proposes no code and changes no spec item: the prototypes drive `openftgs.train`, `openftgs.render`, `openftgs.losses`, `openftgs.init` and `openftgs.eval` from outside the package and change none of them, and adopting a finalist is a spec change the owner makes, as the next lettered revision ("Editing the spec" in `docs/agents/guidelines.md`), followed by its own implementation issue.

The nine sections here are the nine the PRD fixes ("How the work runs"). Section 1 here states the scope and the owner's decisions. Sections 2 to 7 here are the six steps of section 5.7's table, one per section: step 1, the work that trains nothing, in six subsections (sections 2.1 to 2.6 here); step 2, the baseline and the parity runs (section 3 here); steps 3 to 6, the four finalists (sections 4 to 7 here). Section 8 here holds the recommendation, and section 9 here the sources. The opening and section 1 here were written first (ticket #17); every later section opens with a line naming the ticket that writes it, and every step's section ends with a place for the owner's answer and its date, which the ticket that writes the section fills in.

**Words.** As in the research document's opening ("Words"): **private scene A**, the **shell**, a **prior map**, an **estimator**, the **bar**, a **placeholder pixel**, a **finalist**, and a **baseline**, which is the run of the default training configuration a test arm is compared with, not the glossary's Baseline. Also, from the PRD: an **arm** is one technique at one setting, trained on both sets of runs (the research document, section 5.2); a **check** is one of the eight rows of the research document's section 5.1, (a) to (f), the reads and the model; a **step** is a row of its section 5.7's table. The **near side** is the primitives nearer than 2.0 along the centre camera's axis (section 4.4). The **far-5 cm share** and the **under-two-frames share** are the notes' measures as section 2.6 defines them. A row label such as "row E16" is a screening row of the research document's section 4.8; a bare E16 is the spec's equation. **A bare section number is the research document's**, as in "section 2.6" or "section 4.4's criteria"; this document's own sections are written with "here", as in "section 3 here". Other words are the glossary's (`docs/GLOSSARY.md`).

**Local paths** are written as placeholders: `$OPENFTGS_DATASETS` (openftgs-synth is at `$OPENFTGS_DATASETS/synth`); `<m17-runs>`, as `docs/prd/validation-runs.md` uses it; `<surface-runs>`, the folder outside the repository that holds this work's run directories, checkpoints, prior maps, sheets and large JSON, one subfolder per ticket (section 1 here, "Machine and data"); `<scratch>`, the PRD's gitignored scratch folder `.scratch/surface-following-tests/`, for scripts, scratch environments and downloads; `<private-scene-A>`, private scene A's location, which is written nowhere. Prototype scripts are not committed: the section that used one records its commands, the versions and a one-line note on each run.

**Clean-room record.** Section 9 gives it in full, and ticket #29 completes this paragraph. The rules are CLAUDE.md's guidelines rules, which replace spec §0 on sources for this fork, with the PRD's "What the run must license-check and log before using it": FreeTimeGS++ is read as its arXiv paper only, its project page and code unopened (source-log row 767); any repository of SparseCam4D or SSA-3DGS is opened at its license file first; MoGe-2's and DAViD's weights have their terms checked again at the publisher before use, and the MoGe code's license file is read before its code; the ONNX runtime goes into a scratch environment only. Every source opened gets a row in `docs/prd/source-log.md`, written when it is opened, under a temporary ID of the form `T<ticket>.<k>` on the ticket's branch, numbered at its merge and cited by number. Reads of this repository and of its own tracker need no row (the owner's answer of 2026-11-16, PRD #16). Ticket #17 opened no source outside them and added no row.

## 1. Scope and the owner's decisions

*Ticket #17's section.* The scope of PRD #16 and every owner decision it rests on, from the PRD and the owner's answers in session, each marked confirmed, answered or default.

**The problem.** It is PRD #1's. On private scene A the cameras' images disagree about the person, and training answers with the shell (research document, section 2.1). PRD #1 measured the disagreement and screened 85 techniques, but tested nothing: H1 and H2 are both untested, and the evidence that the shell is a reconciliation is absent, not thin (research document, sections 4.2 and 7.5). This PRD supplies the tests.

**How the decisions are marked.** The sources are PRD #16, published by the owner on 2026-11-16 and without a comment when this section was written, and the owner's answers given in session on 2026-11-16, which the orchestrating session relayed to ticket #17. Each decision is marked **confirmed** (the owner accepted what a plan or a review proposed), **answered** (the owner's own instruction, or an answer that replaced a proposal) or **default** (proposed by the PRD, which the owner published unchanged, and not confirmed in words; it holds until the owner says otherwise on the PRD or in session). Where the research document already records a decision and its mark, the mark is carried from there.

**The gates.** Step 1, the work that trains nothing (section 2 here), runs when the implement run starts: the PRD's `ready-for-agent` label clears it and its under two GPU hours, and nothing else. Every later step starts only on the owner's go, given on the PRD or in session after the owner has seen the previous step's result, and the owner can stop after any step (research document, section 5.7). **Confirmed** (2026-11-16): at review 2 the owner accepted that section 5's plan runs as a later PRD, starting with the work that trains nothing (research document, section 7.7), and the PRD states the gates as section 5.7 does.

**The seam.** **Confirmed** by the owner on 2026-11-16, in session, before the PRD was published; the alternatives were a throwaway uncommitted worktree and waiting for the spec revisions. Finalists 2 to 4 and the parity runs go through a scratch script outside the package.

- **How it works.** `Trainer.step` calls `render` and `total_loss` by their names in `openftgs.train`, and a `Callback` runs only after a step is complete, so the public API has no place to add a term before the backward pass. The script drives `Trainer` and replaces those two names at run time with wrappers: one adds the new terms, and the other hands it the camera that Q-226's device needs (research document, section 4.5). Finalist 1 needs no wrapper: a corrected scene manifest, then `ftgs init roma` and `ftgs train` unchanged (section 5.2).
- **What it catches.** What the terms do to training, with the package's code unchanged, as the run with every added weight at 0 shows (the parity runs, section 3 here).
- **What it misses.**
  - The adopted form's cost: the device adds a render pass where an adopted output would be channels of the colour pass (research document, section 2.4), so no run here measures B-1.
  - Whether the new terms should reach E10's g.
  - A change in `Trainer.step` that stops it calling those two names, which the script would not notice; the parity run catches it.

**The criteria** are the research document's section 4.4's, accepted as drafted at review 1, the 0.3 dB guard on a test-view score and the sheets judged with their labels hidden included. **Confirmed** (2026-11-16; research document, section 4.7, line C). The PRD restates them under "Tests that must pass": each finalist names beforehand the one measure it should move, reported apart and never combined with the other; a measure has moved when both of the arm's seeds lie beyond both of the baseline's and the means differ by more than 0.01; and they are read by section 4.4's "How a result would be read" and section 7.2's table of when to adopt or drop.

**The two answers at review 2** (research document, section 7.7):

- **Finalist 1's fit** is made once per held-out copy, without the held-out camera's images, for D-66's reason; section 4.5's fit of the nine cameras as one set is not the plan's. A fit of all nine as one set may still be added to check (c), to see whether one set reconciles the pairs, and it never trains (the PRD, check (c)). **Answered** (2026-11-16).
- **Finalist 4's arm** leaves out row C7's total-variation term, so that its result reads as the prior's effect. **Confirmed** (2026-11-16).

**From section 5's plan, which review 2 accepted.** **Confirmed** (2026-11-16, with the plan; research document, section 7.7):

- **No arm combines two finalists:** one arm per finalist at its paper's settings. The combined arm H2 calls for, and weight sweeps, are steps the owner may add after a result (section 5.7).
- **The order** is section 5.7's.
- **Terms read through the device don't enter E10's g** (section 5.2). Finalist 2's term does, since it is on the colour render (section 5.4).

**The routes and the hours.** Section 4.5's routes, hours and `kitchen` control were a proposal at review 1 that cleared no run (research document, section 4.7, lines R, H and K; **answered**, 2026-11-16). The PRD adopts them as its plan, and each step's hours are cleared at that step's go. **Default.**

| Step | Section | GPU hours |
| --- | --- | --- |
| Step 1, the work that trains nothing | 2 | under 2 |
| Step 2, the baseline | 3 | 2.5 |
| Step 3, finalist 1 | 4 | 2, and 2 more with its optional control on `kitchen` |
| Step 4, finalist 2 | 5 | 2 to 4 |
| Step 5, finalist 3 | 6 | 6 |
| Step 6, finalist 4 | 7 | 4.5, and 2 more with DAViD's optional arm |
| All four with the baseline | | 17 to 19, or 25 to 27 if a run on private scene A takes twice the notes' median of 18 minutes |

The hours are on one RTX 4090, one run at a time, as the PRD counts them; section 5.7's table counts finalist 1 as 1 to 3, leaving check (c) to step 1. Finalist 3's weight calibration is not in that count (section 5.5).

**The owner's six items**, the PRD's "For the owner": not spec gaps, each with a default that holds until the owner answers.

1. **The notes' sharpness ratio.** Section 4.4 names it as a criterion and no section defines it. It is the notes' scorer's own, read from the owner's tools, which the owner points to in session; this document writes its definition down, and its spread between seeds is taken from the baseline's two seeds on the centre copy. **Default.**
2. **Finalist 2's patch side** (`loss.align_patch` in section 5.4's sketch), which row E16 does not record. It is taken from AligNeRF's paper, re-read and logged; otherwise it is set in finalist 2's first step and reported before the arm runs. **Default.**
3. **Finalist 3's weight calibration**, whose form and hours are unsettled (section 5.5). It is proposed to the owner at step 5's go, with its runs and hours, before any arm. **Default.**
4. **The two optional runs:** finalist 1's control on `kitchen`, and finalist 4's arm with DAViD's normals. Each is decided at its step's go, as section 4.6 recommends for the control. **Default.**
5. **The folder outside the repository.** It is PRD #1's, with one subfolder per ticket; this document writes it `<surface-runs>`. **Answered** (2026-11-16, in session).
6. **Subagents.** PRD #1's run used one at a time from 2026-11-15, at the owner's request. For step 1, implementer subagents run in parallel, with their CPU threads budgeted and one subagent at a time on the GPU. **Answered** (2026-11-16, in session).

**How the work runs.**

- **The deliverable** is this document, a document of its own rather than section 6 of the research document, which section 5.7 left open: the research document is long already (PRD #1, item 18) and stays the record of PRD #1. Its section 6 gains one line pointing here. The branch is `docs-surface-following-tests` from `main`, in its own worktree, and it merges with `git merge --no-ff` once the owner approves. **Default** (the PRD, "How the work runs").
- **Every `gh` command** names the repository, `-R samaust/openftgs-plus`; openftgs's issues (374, 395) are read with `-R samaust/openftgs` and never written. **Answered** (2026-11-14, in session; the research document's opening).
- **Source-log rows** are written as each source is opened, under temporary IDs `T<ticket>.<k>` that use the ticket's number, and numbered at the merge. **Answered** (2026-11-14, for PRD #1; the research document's opening), and carried by the PRD. Reads of this repository's own tracker need no row. **Answered** (2026-11-16, the PRD's "Sources consulted while planning").
- **FreeTimeGS++** (arXiv 2605.03337) may be read whole, and papers that cite FreeTimeGS and describe their authors' own reproduction of it may be read and cited; code that implements or extends FreeTimeGS stays unopened. **Answered** (2026-11-16, in session; the research document's opening, and section 4.7, items 10, 14 and 15).
- **The fourth nine-camera model** sits behind a link that points outside the folders given, and the link may be followed. **Answered** (2026-11-16; PRD #1, item 14, and section 4.7, items 10, 14 and 15).

**Machine and data.** Run outputs (run directories, checkpoints, prior maps, sheets, large JSON) go in `<surface-runs>`, one subfolder per ticket (item 5). Scripts, scratch environments and downloads go in `<scratch>`, one subfolder per ticket, never in the shared `.venv` or the repository; PRD #1's scripts under `.scratch/surface-following-primitives/` may be copied from, never edited, and prototype code is not committed. Each package's and each weights file's license is read and logged before it is used. **Default** (the PRD, "How the work runs"), apart from the folder of item 5.

**Private scene A.**

- Its name appears in no commit, issue, branch or file name, source-log row or search query, and the alias "private scene A" stands for it. **Answered** (2026-11-13, in session), with the extension to all of these **confirmed** at PRD #1's review 1 (the research document's opening, and section 4.7, item 17).
- Its generator is named nowhere. **Answered** (2026-11-14, in session).
- Its numbers may appear under the alias (decision 5 of PRD #1, **confirmed** 2026-11-14), image content given as numbers included (**answered**, 2026-11-16, in session).
- Its paths and its images never appear. **Confirmed** (decision 5 of PRD #1, 2026-11-14).
- Its images are read by scripts only, and nothing derived from them leaves the machine. **Default** (the PRD, "How the work runs", and its branch criteria), as PRD #1 did (research document, section 1, "Machine and data").
- Its location is given in session to the tickets that measure it, and written nowhere. **Default** (the PRD, "How the work runs"), as PRD #1 did.

**The spec items.** None changes. The prototypes exercise the items of the PRD's "Spec items", and the finalists' sections say, per finalist, how each behaved; the research document's section 8 gives what adopting each finalist would change. For every finalist: E9 (composition D-37); E10's g [P §3.2, Eq. 7], which new terms reach or don't; D-26; D-72; §1.4 "Validation data" and D-79; E16 and D-67; §2.3, which gains nothing; §2.4, whose cost rules a prototype does not measure. For finalist 1: §2.2 `ftgs-scene` v1, §1.3 Initialization with D-21, D-22, D-58 and E15, D-27 and D-66. For finalist 2: E7 and D-7's L1, with D-8 and D-9/D-89 left for the test to settle. For finalists 3 and 4: E6, D-75 and D-76; §1.3 Growth, E11 and E17; E8; finalist 4's maps are files written outside the package, so neither `ftgs-scene` nor D-51 changes. The §2.1 signatures adoption would change and the prototypes leave alone: `RenderOutput`, `render`, `LossConfig`, `total_loss`. §3.1 U-12, whose bounds check (f) and the device are held to. §0 and CLAUDE.md's guidelines rules decide what may be read.

**Q-n rows the work relies on** (all on `main`; their decision issues are openftgs's): Q-7, Q-64 and Q-65, Q-106, Q-220, Q-226, Q-227 to Q-231 and Q-233 to Q-237, as the PRD's "Open questions" gives them. The PRD records no Q-n candidate: the fields, keys and signatures sections 5.3 to 5.6 sketch are proposals for the owner's spec revision, not gaps.

**Out of scope.** Adopting any finalist, finalist 1's adopted form included (a script under `scripts/` or an `ftgs` command). Row E13, the known masks for the placeholder pixels, which come with the next related spec revision (research document, section 7.7). Any spec edit, and any change to `src/`, `tests/`, `harness/`, `scripts/`, §2.3's table or the package metadata. The combined arm and weight sweeps beyond one arm per finalist. The training cost of an adopted technique (B-1). Changing the gsplat pin; the paper's datasets (D-79); anything derived from SelfCap. Tightening the research document, and recording the privacy rule in CLAUDE.md.

## 2. Step 1: the work that trains nothing

*Tickets #18 to #23.* The eight rows of the research document's section 5.1: checks (a) to (f), the three reads and the fourth nine-camera model, each ending in its result and in the reading that section 5.1's last column gives for it; then step 1's review.

### 2.1 Checks (a) and (b), and the fourth nine-camera model

*Ticket #18's section.* The shell's measures where the cameras agree, on M17's `default-395` checkpoints of openftgs-synth; on private scene A's 45 kept checkpoints, split by time centre at frame 80; and on the fourth nine-camera model, at stride 1, with its sheet.

All measured on 2026-11-16 with this branch's code at 9bd213c (`src/` and `scripts/` as `main`'s), Python 3.14.10, torch 2.15.0+cu132, gsplat 1.5.3, SciPy 1.18.1, NumPy 2.5.4, Pillow from the shared `.venv`, five CPU threads and, for the renders only, one RTX 4090 under the shared lock. Scripts read tensors, JSON and mask files and wrote numbers and files; no image, render or sheet of private scene A was looked at. "The commands", at the end, lists every run. `<private-scene-A-exp>` is the folder of the owner's notes on private scene A, which holds their runs, init points files and tools; like `<private-scene-A>`, it is written nowhere.

**How the measures are taken.** As section 2.6 takes them (rows T18.2, T18.3): the exchange model's values from `load_checkpoint` and `FreeTimeGaussians.to_exchange()` (μx, E3's base opacity σ, E4's duration s and time centre μt, in seconds), the distance from each μx to the nearest point of the init points file the run trained from (SciPy's `cKDTree`, k = 1, all init frames together), and the scorer's expressions written anew. A primitive is *far* when that distance exceeds the threshold; the far share is the far primitives' share of the summed σ. The *under-two-frames share* is the share of primitives with s under two frame intervals. The *near side* on private scene A is section 2.6's: μx nearer than 2.0 along the centre camera's axis; the far side is the rest. Two checks hold the scripts to section 2.6's:

- On all 45 kept checkpoints the whole model's eight figures (N, the two far-5 cm shares, the under-two-frames share, the median duration and the three opacity figures) equal ticket #5's `<surface-runs>/ticket-5/shell_measures.json` in every digit, and the near side's far-5 cm share of `A_default_s0` is 0.128, as in section 2.6's second table.
- The fourth model's six figures that do not depend on an init points file (N, the median duration, the under-two-frames share and the three opacity figures) equal the notes' score file beside its link in every digit.

**Check (a): openftgs-synth, where the cameras agree.**

**The rules** (section 5.1, row (a); D-27 for S).

- **The checkpoints:** M17's `default-395` runs of `kitchen`, `courtyard` and `studio` (steps 35 to 37 of `docs/prd/validation-runs.md`), each against its scene's init points file `<m17-runs>/<scene>/init.safetensors`, which every default run of the scene trained from: 300,000, 300,000 and 63,000 points, against private scene A's 120,000 (row T18.4).
- **The 5 cm** is 0.12·S: S is 1.597, 6.343 and 4.663, so the threshold is 0.192, 0.761 and 0.560 m. Beside it: the share against a seeded subsample of the init file to 120,000 points (`kitchen` and `courtyard`; `studio`'s file is smaller), and the literal 5 cm.
- **Two frames** are 2/fps at each scene's frame rate (row T18.1): 2/30 s on `kitchen`, 2/60 s on `courtyard` and `studio`. Beside it, private scene A's absolute 2/24 s.
- **The dynamic side** stands in for the near side. Each primitive's μx, its position at its own time centre (E1), is projected into every ground-truth camera (`K_continuous` and `world_to_camera` of `cameras.json`; the masks are undistorted) at that camera's frame nearest μt within the checkpoint's [τ₀, τ₁], so `studio`'s sync offsets are taken camera by camera. A camera counts when μx is in front of it and inside its image; μx is in a camera's mask when the pixel it falls in is 255. **The main rule, "majority":** in the mask in at least half of the counting cameras. Beside it, "any camera" (looser: it takes in the static scene behind a moving object) and "every camera" (a visual hull: tighter). The static side is the rest of the primitives some camera counts; 0.04 % to 0.11 % of the opacity falls in no camera's image. The median primitive counts in 20, 17 and 16 of the 20, 18 and 24 cameras.

**The measures, per scene and side.** "Primitives / opacity" is the side's share of the model's primitives and of its summed σ. Far shares are of σ; σ's quartiles are the 25th, 50th and 75th percentiles; σ < 0.005 is D-14's dead threshold.

| Scene | Side | Primitives / opacity | Far share at 0.12·S | At 0.12·S, against 120,000 init points | At 5 cm | Under two frames, 2/fps | Under 2/24 s | Median duration, frames | Mean σ | σ < 0.05 | σ > 0.5 | σ quartiles | σ < 0.005 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `kitchen` | whole model | 1.000 / 1.000 | 0.035 | 0.038 | 0.237 | 0.382 | 0.467 | 2.74 | 0.234 | 0.514 | 0.202 | 0.008 / 0.045 / 0.315 | 0.177 |
| `kitchen` | dynamic side, majority (main) | 0.191 / 0.392 | 0.003 | 0.008 | 0.227 | 0.661 | 0.767 | 1.52 | 0.481 | 0.246 | 0.465 | 0.052 / 0.391 / 0.958 | 0.072 |
| `kitchen` | dynamic side, any camera | 0.267 / 0.460 | 0.013 | 0.018 | 0.253 | 0.574 | 0.676 | 1.73 | 0.404 | 0.322 | 0.382 | 0.027 / 0.197 / 0.907 | 0.098 |
| `kitchen` | dynamic side, every camera | 0.151 / 0.322 | 0.002 | 0.007 | 0.227 | 0.678 | 0.785 | 1.49 | 0.499 | 0.234 | 0.487 | 0.058 / 0.455 / 0.965 | 0.069 |
| `kitchen` | static side, majority | 0.807 / 0.607 | 0.055 | 0.057 | 0.242 | 0.316 | 0.396 | 3.41 | 0.176 | 0.577 | 0.140 | 0.007 / 0.031 / 0.173 | 0.202 |
| `courtyard` | whole model | 1.000 / 1.000 | 0.012 | 0.013 | 0.359 | 0.327 | 0.614 | 3.33 | 0.167 | 0.611 | 0.135 | 0.006 / 0.024 / 0.152 | 0.228 |
| `courtyard` | dynamic side, majority (main) | 0.205 / 0.362 | 0.000 | 0.001 | 0.457 | 0.486 | 0.885 | 2.05 | 0.294 | 0.432 | 0.269 | 0.012 / 0.083 / 0.570 | 0.137 |
| `courtyard` | dynamic side, any camera | 0.420 / 0.547 | 0.004 | 0.005 | 0.397 | 0.413 | 0.751 | 2.43 | 0.217 | 0.544 | 0.188 | 0.007 / 0.037 / 0.273 | 0.195 |
| `courtyard` | dynamic side, every camera | 0.181 / 0.336 | 0.000 | 0.001 | 0.458 | 0.479 | 0.887 | 2.08 | 0.309 | 0.409 | 0.284 | 0.014 / 0.097 / 0.624 | 0.124 |
| `courtyard` | static side, majority | 0.793 / 0.637 | 0.018 | 0.018 | 0.302 | 0.286 | 0.544 | 4.21 | 0.134 | 0.657 | 0.101 | 0.005 / 0.018 / 0.103 | 0.252 |
| `studio` | whole model | 1.000 / 1.000 | 0.042 | 0.042 | 0.333 | 0.386 | 0.585 | 3.15 | 0.189 | 0.487 | 0.142 | 0.010 / 0.054 / 0.238 | 0.174 |
| `studio` | dynamic side, majority (main) | 0.419 / 0.550 | 0.001 | 0.001 | 0.277 | 0.712 | 0.948 | 1.30 | 0.249 | 0.395 | 0.204 | 0.019 / 0.090 / 0.374 | 0.119 |
| `studio` | dynamic side, any camera | 0.805 / 0.842 | 0.017 | 0.017 | 0.304 | 0.457 | 0.666 | 2.30 | 0.198 | 0.476 | 0.152 | 0.011 / 0.058 / 0.254 | 0.165 |
| `studio` | dynamic side, every camera | 0.343 / 0.445 | 0.000 | 0.000 | 0.267 | 0.711 | 0.947 | 1.31 | 0.246 | 0.399 | 0.201 | 0.019 / 0.088 / 0.366 | 0.123 |
| `studio` | static side, majority | 0.580 / 0.449 | 0.092 | 0.092 | 0.400 | 0.151 | 0.324 | 11.23 | 0.147 | 0.553 | 0.097 | 0.007 / 0.036 / 0.168 | 0.213 |

Private scene A's figures, from section 2.6 and check (b)'s run below, for the same columns: over the whole model, far-5 cm shares of 0.15 to 0.37 over the 40 final checkpoints of runs with a test view and 0.259 for `A_default_s0`; under-two-frames shares of 0.45 to 0.52 for the default configuration and its nearest variants (section 5.1), 0.11 to 0.80 over the 40; mean σ 0.360, σ < 0.05 0.285, σ > 0.5 0.315 for `A_default_s0`. On the near side: a far-5 cm share of 0.12 to 0.13 for the default configuration (0.06 to 0.16 over the 40), an under-two-frames share of 0.647 for `A_default_s0` (0.15 to 0.88 over the 40), and 75 % of the model's opacity.

**The spread between trainings.** Each scene has one training at 549o; the earlier trainings of the default configuration (section 2.6, with LPIPS as the perceptual network) give the spread. "Dyn." is the dynamic side by the main rule.

| Scene | Checkpoint | Whole: far share at 0.12·S | Whole: under two frames | Whole: mean σ | Dyn.: share of opacity | Dyn.: far share at 0.12·S | Dyn.: under two frames | Dyn.: mean σ |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `kitchen` | `default-395` | 0.035 | 0.382 | 0.234 | 0.392 | 0.003 | 0.661 | 0.481 |
| `kitchen` | `default` | 0.025 | 0.425 | 0.199 | 0.483 | 0.002 | 0.685 | 0.451 |
| `kitchen` | `seed1` | 0.025 | 0.427 | 0.198 | 0.489 | 0.002 | 0.686 | 0.454 |
| `kitchen` | `default-326` | 0.025 | 0.429 | 0.202 | 0.492 | 0.001 | 0.691 | 0.459 |
| `courtyard` | `default-395` | 0.012 | 0.327 | 0.167 | 0.362 | 0.000 | 0.486 | 0.294 |
| `courtyard` | `default` | 0.013 | 0.488 | 0.145 | 0.473 | 0.000 | 0.622 | 0.267 |
| `studio` | `default-395` | 0.042 | 0.386 | 0.189 | 0.550 | 0.001 | 0.712 | 0.249 |
| `studio` | `default` | 0.046 | 0.388 | 0.145 | 0.605 | 0.001 | 0.695 | 0.188 |
| `studio` | `default-325` | 0.047 | 0.389 | 0.147 | 0.606 | 0.001 | 0.695 | 0.188 |
| `studio` | `default-341` | 0.045 | 0.389 | 0.145 | 0.608 | 0.001 | 0.694 | 0.186 |

**The far share at other multiples of S,** to see whether the reading rests on 0.12·S. The column is the far share of σ over the whole model, then on the near side (private scene A) or the dynamic side by the main rule (openftgs-synth).

| Checkpoint | 0.02·S | 0.05·S | 0.12·S | 0.25·S | 5 cm |
| --- | --- | --- | --- | --- | --- |
| `A_default_s0` | 0.751 / 0.675 | 0.446 / 0.307 | 0.259 / 0.128 | 0.175 / 0.076 | 0.259 / 0.128 |
| `A_default_s1` | 0.753 / 0.680 | 0.447 / 0.312 | 0.257 / 0.131 | 0.169 / 0.072 | 0.257 / 0.131 |
| `c3_s4` | 0.758 / 0.701 | 0.438 / 0.340 | 0.213 / 0.124 | 0.135 / 0.068 | 0.213 / 0.124 |
| `kitchen` `default-395` | 0.390 / 0.473 | 0.130 / 0.081 | 0.035 / 0.003 | 0.009 / 0.000 | 0.237 / 0.227 |
| `courtyard` `default-395` | 0.118 / 0.030 | 0.045 / 0.000 | 0.012 / 0.000 | 0.003 / 0.000 | 0.359 / 0.457 |
| `studio` `default-395` | 0.136 / 0.038 | 0.072 / 0.001 | 0.042 / 0.001 | 0.021 / 0.000 | 0.333 / 0.277 |

What the tables show:

- **The far share is clearly lower where the cameras agree,** at 0.12·S and at every other multiple of S. On the dynamic side it is 0.000 to 0.003 against private scene A's 0.12 to 0.13 on the near side, 0.008 at most against a 120,000-point sample; over the whole model 0.012 to 0.042 against 0.21 to 0.26, and on the static side 0.018 to 0.092 against the far side's 0.52 to 0.78. The earlier trainings' far shares are within 0.01 of the 549o ones. Only the literal 5 cm gives openftgs-synth higher shares (0.23 to 0.46): 5 cm is 0.008·S to 0.03·S there, finer than the init points' spacing.
- **The under-two-frames share is not lower on the moving parts.** On the dynamic side it is 0.66, 0.49 and 0.71 at 2/fps (0.68 to 0.69 and 0.62 and 0.69 in the earlier trainings), against 0.647 on private scene A's near side; at private scene A's absolute 2/24 s it is 0.77, 0.89 and 0.95. Over the whole model it is lower, 0.33 to 0.39 at 549o (0.39 to 0.49 before), against 0.45 to 0.52, because the static side, where it is 0.15 to 0.32, holds 45 % to 64 % of the opacity, against 25 % for private scene A's far side. On the dynamic side the primitives far from init are too few to compare (72 to 2,918 of 0.6 to 1.3 million).
- **The base opacity is lower over the whole model** on openftgs-synth (mean σ 0.17 to 0.23 against 0.36; σ < 0.05 for 0.49 to 0.61 of the primitives against 0.29; 17 % to 23 % dead against 7 %), while on the dynamic side `kitchen`'s is higher than private scene A's (mean 0.48, median 0.39) and the other two scenes' lower (0.29 and 0.25). The 549o and earlier trainings differ by up to 0.06 in mean σ, so the distribution is read only roughly.
- **The dynamic-side rule does not change the reading.** "Any camera" and "every camera" give far shares of 0.000 to 0.017 and under-two-frames shares of 0.41 to 0.71; "any camera" lowers the latter by taking in the static scene behind the moving objects, most on `studio`'s ring of 24 cameras, where it holds 84 % of the opacity.
- **FreeTimeGS++'s account of short-lived primitives** does not bear on these numbers directly: its "lifetime" depends on the base opacity as well as on s, and its reproduction starts from other initial values than §1.3 (section 2.5 here).

**The reading** (section 5.1, row (a)). The result is split by measure, so neither branch of the row's last column holds for both.

- **The far-5 cm share stands as a criterion.** Where the cameras agree it is near 0 on the moving parts and 5 to 20 times lower over the whole model, at every multiple of S tried. A fall in it on private scene A can be read as the shell shrinking, and finalist 1's control on `kitchen` has its baseline in the table above.
- **The under-two-frames share is the method's habit on moving content.** It is as high on openftgs-synth's moving objects, where the cameras agree exactly, as on private scene A's near side. A fall in it would not show that a disagreement was removed. As a test of H2 by removal, finalists 1 and 2 keep their reason through the far share and the sheet, not through the under-two-frames share.
- **Caveats.** One training per scene at 549o. The init points files are larger than private scene A's on two scenes (a 120,000-point sample changes the far share by 0.005 at most). The dynamic side is not private scene A's near side: it is the moving objects alone, while the near side holds the person, what lies near the person and nothing else nearer than 2.0. The reading is ticket #23's to confirm (section 2.6 here).

**Check (b): private scene A, split by time centre at frame 80.**

**The rules** (section 5.1, row (b)). The 45 kept checkpoints of section 2.6, each against its own init points file as section 2.6 assigns it, with the 5 cm, the 2/24 s and the near side unchanged. A primitive is *after* frame 80 when the frame nearest its time centre μt is 80 or later (μt ≥ 79.5/24 s, so frames 80 to 119, section 2.5's late part) and *before* otherwise; μt outside the clip goes with the nearer end. Every share below is taken within its half, so the two halves recombine to section 2.6's figures.

**The measures, per checkpoint.** Each cell is before / after. "Near share" is the near side's share of the half's opacity; "after" in the last column is the after half's share of the model's primitives and of its opacity. Runs are named as in section 2.6.

| Checkpoint | Whole: far-5 cm share | Whole: under two frames | Whole: mean σ | Near: far-5 cm share | Near: under two frames | Near: mean σ | Far side: far-5 cm share | Far side: under two frames | Near share of the half's opacity | After: primitives / opacity |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `stride4` | 0.228 / 0.183 | 0.382 / 0.564 | 0.330 / 0.346 | 0.115 / 0.130 | 0.533 / 0.659 | 0.346 / 0.358 | 0.522 / 0.548 | 0.047 / 0.051 | 0.722 / 0.872 | 0.404 / 0.415 |
| `stride4_zero_velocity` | 0.232 / 0.187 | 0.386 / 0.552 | 0.326 / 0.335 | 0.119 / 0.130 | 0.540 / 0.648 | 0.344 / 0.343 | 0.541 / 0.557 | 0.034 / 0.032 | 0.733 / 0.866 | 0.404 / 0.410 |
| `stride4_zero_velocity_reg0.1` | 0.174 / 0.128 | 0.313 / 0.475 | 0.110 / 0.121 | 0.058 / 0.082 | 0.424 / 0.531 | 0.120 / 0.125 | 0.647 / 0.677 | 0.001 / 0.008 | 0.804 / 0.924 | 0.404 / 0.426 |
| `A_default_s0` | 0.284 / 0.220 | 0.463 / 0.605 | 0.355 / 0.366 | 0.130 / 0.125 | 0.605 / 0.703 | 0.365 / 0.371 | 0.641 / 0.669 | 0.160 / 0.169 | 0.699 / 0.826 | 0.386 / 0.393 |
| `A_default_s1` | 0.281 / 0.221 | 0.462 / 0.610 | 0.355 / 0.364 | 0.130 / 0.132 | 0.605 / 0.705 | 0.364 / 0.370 | 0.638 / 0.670 | 0.149 / 0.172 | 0.703 / 0.835 | 0.387 / 0.393 |
| `c3_s4` | 0.249 / 0.163 | 0.380 / 0.545 | 0.351 / 0.374 | 0.138 / 0.108 | 0.530 / 0.639 | 0.369 / 0.388 | 0.540 / 0.534 | 0.051 / 0.065 | 0.724 / 0.869 | 0.407 / 0.422 |
| `c3_s4_reg0.1` | 0.174 / 0.144 | 0.316 / 0.464 | 0.119 / 0.130 | 0.065 / 0.103 | 0.422 / 0.519 | 0.129 / 0.135 | 0.640 / 0.668 | 0.004 / 0.013 | 0.811 / 0.928 | 0.407 / 0.429 |
| `c3_s4v0` | 0.240 / 0.216 | 0.388 / 0.545 | 0.349 / 0.360 | 0.136 / 0.159 | 0.530 / 0.642 | 0.367 / 0.370 | 0.536 / 0.576 | 0.047 / 0.033 | 0.740 / 0.864 | 0.411 / 0.418 |
| `c3_s4v0_reg0.1` | 0.171 / 0.155 | 0.321 / 0.461 | 0.120 / 0.130 | 0.063 / 0.109 | 0.426 / 0.520 | 0.130 / 0.135 | 0.655 / 0.694 | 0.002 / 0.011 | 0.819 / 0.921 | 0.413 / 0.432 |
| `sh0` | 0.292 / 0.219 | 0.452 / 0.598 | 0.327 / 0.325 | 0.133 / 0.106 | 0.602 / 0.703 | 0.331 / 0.322 | 0.624 / 0.671 | 0.150 / 0.166 | 0.676 / 0.799 | 0.380 / 0.379 |
| `sh1` | 0.290 / 0.227 | 0.450 / 0.606 | 0.337 / 0.338 | 0.125 / 0.123 | 0.601 / 0.709 | 0.340 / 0.338 | 0.628 / 0.642 | 0.147 / 0.193 | 0.673 / 0.800 | 0.382 / 0.383 |
| `B_usercfg_s0` | 0.247 / 0.205 | 0.398 / 0.592 | 0.410 / 0.418 | 0.101 / 0.131 | 0.549 / 0.686 | 0.436 / 0.432 | 0.632 / 0.639 | 0.074 / 0.138 | 0.724 / 0.854 | 0.423 / 0.427 |
| `B_usercfg_s1` | 0.244 / 0.206 | 0.405 / 0.593 | 0.407 / 0.419 | 0.105 / 0.126 | 0.548 / 0.687 | 0.433 / 0.431 | 0.611 / 0.656 | 0.095 / 0.155 | 0.726 / 0.848 | 0.416 / 0.423 |
| `b15_dead0.05` | 0.345 / 0.201 | 0.145 / 0.193 | 0.221 / 0.237 | 0.076 / 0.087 | 0.267 / 0.241 | 0.238 / 0.246 | 0.723 / 0.745 | 0.000 / 0.001 | 0.584 / 0.826 | 0.272 / 0.286 |
| `b15_durlr0.2` | 0.395 / 0.314 | 0.768 / 0.856 | 0.501 / 0.497 | 0.149 / 0.120 | 0.861 / 0.911 | 0.464 / 0.465 | 0.655 / 0.716 | 0.654 / 0.713 | 0.513 / 0.674 | 0.341 / 0.339 |
| `b15_durlr4` | 0.178 / 0.119 | 0.593 / 0.731 | 0.348 / 0.342 | 0.072 / 0.077 | 0.711 / 0.784 | 0.360 / 0.346 | 0.590 / 0.577 | 0.201 / 0.241 | 0.795 / 0.915 | 0.405 / 0.400 |
| `b15_init3k` | 0.203 / 0.141 | 0.447 / 0.577 | 0.366 / 0.379 | 0.067 / 0.052 | 0.589 / 0.667 | 0.388 / 0.389 | 0.519 / 0.525 | 0.171 / 0.241 | 0.699 / 0.813 | 0.379 / 0.387 |
| `b15_init_d4` | 0.219 / 0.152 | 0.333 / 0.453 | 0.361 / 0.379 | 0.085 / 0.077 | 0.462 / 0.536 | 0.386 / 0.394 | 0.613 / 0.617 | 0.034 / 0.051 | 0.747 / 0.862 | 0.394 / 0.405 |
| `b15_init_s4` | 0.218 / 0.180 | 0.328 / 0.479 | 0.369 / 0.396 | 0.091 / 0.121 | 0.466 / 0.570 | 0.394 / 0.415 | 0.558 / 0.554 | 0.032 / 0.053 | 0.728 / 0.864 | 0.393 / 0.410 |
| `b15_init_s8` | 0.223 / 0.178 | 0.294 / 0.439 | 0.367 / 0.420 | 0.093 / 0.135 | 0.429 / 0.518 | 0.395 / 0.445 | 0.545 / 0.510 | 0.029 / 0.030 | 0.713 / 0.886 | 0.392 / 0.424 |
| `b15_init_v0` | 0.263 / 0.208 | 0.404 / 0.553 | 0.382 / 0.394 | 0.098 / 0.101 | 0.551 / 0.655 | 0.401 / 0.402 | 0.695 / 0.727 | 0.078 / 0.109 | 0.723 / 0.829 | 0.395 / 0.402 |
| `b15_noreloc` | 0.292 / 0.166 | 0.306 / 0.509 | 0.163 / 0.200 | 0.193 / 0.126 | 0.393 / 0.567 | 0.167 / 0.208 | 0.680 / 0.720 | 0.003 / 0.005 | 0.798 / 0.932 | 0.415 / 0.465 |
| `b15_reg0.1` | 0.191 / 0.151 | 0.340 / 0.475 | 0.158 / 0.174 | 0.076 / 0.092 | 0.434 / 0.532 | 0.172 / 0.182 | 0.731 / 0.766 | 0.044 / 0.092 | 0.825 / 0.912 | 0.407 / 0.430 |
| `b15_reg0.3` | 0.194 / 0.130 | 0.199 / 0.336 | 0.056 / 0.061 | 0.080 / 0.092 | 0.260 / 0.363 | 0.061 / 0.063 | 0.773 / 0.798 | 0.003 / 0.012 | 0.836 / 0.946 | 0.398 / 0.418 |
| `b15_scale3` | 0.298 / 0.236 | 0.449 / 0.563 | 0.315 / 0.332 | 0.098 / 0.090 | 0.632 / 0.694 | 0.331 / 0.337 | 0.655 / 0.697 | 0.164 / 0.174 | 0.641 / 0.759 | 0.343 / 0.355 |
| `b15_vgg` | 0.196 / 0.177 | 0.474 / 0.608 | 0.330 / 0.346 | 0.082 / 0.107 | 0.612 / 0.698 | 0.353 / 0.359 | 0.582 / 0.644 | 0.114 / 0.139 | 0.772 / 0.869 | 0.390 / 0.401 |
| `f_s4_reg0.1` | 0.165 / 0.132 | 0.278 / 0.406 | 0.139 / 0.155 | 0.064 / 0.095 | 0.369 / 0.454 | 0.153 / 0.162 | 0.643 / 0.667 | 0.003 / 0.010 | 0.825 / 0.934 | 0.399 / 0.425 |
| `f_s4_reg0.3` | 0.240 / 0.121 | 0.079 / 0.166 | 0.034 / 0.040 | 0.088 / 0.080 | 0.120 / 0.184 | 0.040 / 0.042 | 0.748 / 0.796 | 0.000 / 0.000 | 0.770 / 0.943 | 0.334 / 0.370 |
| `f_s4v0` | 0.216 / 0.172 | 0.330 / 0.465 | 0.371 / 0.389 | 0.094 / 0.114 | 0.468 / 0.553 | 0.397 / 0.405 | 0.558 / 0.548 | 0.024 / 0.025 | 0.736 / 0.866 | 0.399 / 0.411 |
| `f_s4v0_reg0.1` | 0.162 / 0.150 | 0.285 / 0.414 | 0.142 / 0.156 | 0.066 / 0.102 | 0.373 / 0.468 | 0.156 / 0.163 | 0.642 / 0.708 | 0.003 / 0.008 | 0.834 / 0.922 | 0.416 / 0.438 |
| `f_v0_reg0.1` | 0.193 / 0.166 | 0.333 / 0.484 | 0.161 / 0.176 | 0.076 / 0.101 | 0.431 / 0.548 | 0.175 / 0.184 | 0.752 / 0.810 | 0.029 / 0.062 | 0.827 / 0.909 | 0.418 / 0.440 |
| `n1500k` | 0.261 / 0.194 | 0.426 / 0.560 | 0.377 / 0.387 | 0.104 / 0.089 | 0.557 / 0.647 | 0.394 / 0.395 | 0.663 / 0.713 | 0.134 / 0.183 | 0.719 / 0.831 | 0.383 / 0.389 |
| `n750k` | 0.226 / 0.179 | 0.391 / 0.514 | 0.401 / 0.414 | 0.071 / 0.081 | 0.504 / 0.583 | 0.429 / 0.428 | 0.670 / 0.708 | 0.133 / 0.202 | 0.742 / 0.844 | 0.381 / 0.389 |
| `c3_s4v0_reg0.1`, iteration 5,000 | 0.321 / 0.208 | 0.150 / 0.278 | 0.183 / 0.188 | 0.231 / 0.129 | 0.240 / 0.350 | 0.197 / 0.194 | 0.503 / 0.556 | 0.002 / 0.001 | 0.669 / 0.816 | 0.395 / 0.402 |
| `c3_s4v0_reg0.1`, iteration 10,000 | 0.254 / 0.198 | 0.276 / 0.432 | 0.136 / 0.138 | 0.142 / 0.132 | 0.399 / 0.517 | 0.144 / 0.141 | 0.553 / 0.583 | 0.004 / 0.002 | 0.728 / 0.853 | 0.414 / 0.417 |
| `c3_s4v0_reg0.1`, iteration 15,000 | 0.190 / 0.170 | 0.317 / 0.464 | 0.112 / 0.112 | 0.072 / 0.113 | 0.423 / 0.532 | 0.117 / 0.114 | 0.605 / 0.629 | 0.002 / 0.005 | 0.779 / 0.890 | 0.426 / 0.425 |
| `c3_s4v0_reg0.1`, iteration 20,000 | 0.182 / 0.160 | 0.318 / 0.457 | 0.113 / 0.116 | 0.065 / 0.107 | 0.420 / 0.518 | 0.119 / 0.119 | 0.637 / 0.664 | 0.002 / 0.008 | 0.795 / 0.904 | 0.419 / 0.425 |
| `c3_s4v0_reg0.1`, iteration 25,000 | 0.176 / 0.156 | 0.318 / 0.455 | 0.117 / 0.123 | 0.064 / 0.106 | 0.422 / 0.514 | 0.125 / 0.128 | 0.650 / 0.685 | 0.002 / 0.009 | 0.808 / 0.915 | 0.413 / 0.426 |
| `h1_base` | 0.259 / 0.201 | 0.430 / 0.567 | 0.375 / 0.382 | 0.097 / 0.095 | 0.571 / 0.659 | 0.393 / 0.390 | 0.644 / 0.712 | 0.143 / 0.168 | 0.704 / 0.828 | 0.377 / 0.381 |
| `h1_init_s4` | 0.220 / 0.191 | 0.331 / 0.456 | 0.365 / 0.388 | 0.099 / 0.129 | 0.470 / 0.546 | 0.388 / 0.404 | 0.544 / 0.569 | 0.030 / 0.038 | 0.729 / 0.859 | 0.386 / 0.400 |
| `h1_init_s4_seed1` | 0.220 / 0.188 | 0.333 / 0.462 | 0.363 / 0.383 | 0.099 / 0.125 | 0.474 / 0.555 | 0.387 / 0.399 | 0.551 / 0.565 | 0.029 / 0.034 | 0.731 / 0.858 | 0.386 / 0.398 |
| `h1_s4_reg0.1` | 0.164 / 0.146 | 0.283 / 0.402 | 0.139 / 0.153 | 0.060 / 0.100 | 0.377 / 0.454 | 0.153 / 0.161 | 0.651 / 0.709 | 0.003 / 0.010 | 0.825 / 0.925 | 0.403 / 0.427 |
| `h1_s4v0` | 0.222 / 0.188 | 0.337 / 0.467 | 0.364 / 0.382 | 0.101 / 0.120 | 0.475 / 0.562 | 0.389 / 0.397 | 0.563 / 0.589 | 0.026 / 0.031 | 0.739 / 0.853 | 0.391 / 0.402 |
| `h1_s4v0_reg0.1` | 0.159 / 0.151 | 0.290 / 0.405 | 0.140 / 0.153 | 0.059 / 0.098 | 0.381 / 0.463 | 0.153 / 0.161 | 0.659 / 0.734 | 0.003 / 0.010 | 0.832 / 0.916 | 0.413 / 0.435 |
| `h1_s4v0_seed1` | 0.205 / 0.190 | 0.337 / 0.459 | 0.361 / 0.381 | 0.086 / 0.119 | 0.477 / 0.554 | 0.388 / 0.395 | 0.546 / 0.594 | 0.024 / 0.024 | 0.742 / 0.852 | 0.393 / 0.406 |

Over the 45 checkpoints, after minus before:

| Measure | Whole model | Near side | Far side |
| --- | --- | --- | --- |
| Far-5 cm share | −0.143 to −0.008, median −0.045; 44 of 45 below −0.01 | −0.102 to +0.046, median +0.016; 28 above +0.01, 8 below −0.01 | −0.035 to +0.075, median +0.030; 37 above +0.01 |
| Under-two-frames share | +0.047 to +0.204, median +0.137; all 45 above +0.01 | −0.026 to +0.174, median +0.096; 44 above +0.01 | −0.014 to +0.070, median +0.009; 19 above +0.01 |
| Mean σ | −0.007 to +0.053, median +0.012 | −0.014 to +0.050, median +0.006 | −0.018 to +0.035, median +0.003 |
| σ < 0.05 | −0.050 to +0.044, median −0.005 | −0.068 to +0.019, median +0.002 | −0.029 to +0.045, median +0.014 |
| σ > 0.5 | −0.008 to +0.062, median +0.015 | −0.014 to +0.058, median +0.007 | −0.015 to +0.038, median +0.006 |

**Between seeds, per half** (the four pairs of section 2.6 that differ by seed alone): the far-5 cm share differs by up to 0.017 before and 0.003 after over the whole model, 0.015 and 0.007 on the near side; the under-two-frames share by up to 0.007 and 0.008, and 0.004 and 0.009 on the near side; mean σ by up to 0.005.

**A control by 20-frame bins of time centre,** on six checkpoints (`<surface-runs>/ticket-18/bins.json`): the near side's under-two-frames share / far-5 cm share per bin, then, for `A_default_s0`, the near side's median depth along the centre camera's axis, its share of the bin's opacity and the far side's under-two-frames share.

| Checkpoint | Frames 0–19 | 20–39 | 40–59 | 60–79 | 80–99 | 100–119 |
| --- | --- | --- | --- | --- | --- | --- |
| `A_default_s0` | 0.479 / 0.197 | 0.579 / 0.129 | 0.650 / 0.122 | 0.683 / 0.089 | 0.750 / 0.110 | 0.662 / 0.139 |
| `c3_s4` | 0.400 / 0.173 | 0.493 / 0.207 | 0.601 / 0.079 | 0.590 / 0.112 | 0.716 / 0.135 | 0.567 / 0.082 |
| `c3_s4_reg0.1` | 0.306 / 0.086 | 0.392 / 0.086 | 0.474 / 0.049 | 0.473 / 0.052 | 0.584 / 0.114 | 0.456 / 0.093 |
| `h1_init_s4` | 0.325 / 0.170 | 0.455 / 0.089 | 0.542 / 0.083 | 0.524 / 0.072 | 0.612 / 0.153 | 0.481 / 0.105 |
| `stride4` (nine cameras) | 0.397 / 0.164 | 0.495 / 0.174 | 0.605 / 0.074 | 0.599 / 0.073 | 0.700 / 0.119 | 0.620 / 0.140 |
| The fourth model (nine cameras) | 0.501 / 0.147 | 0.584 / 0.145 | 0.657 / 0.103 | 0.665 / 0.100 | 0.751 / 0.113 | 0.688 / 0.106 |
| `A_default_s0`: near side's median depth | 1.24 | 1.21 | 1.12 | 0.99 | 0.85 | 0.81 |
| `A_default_s0`: near side's share of the bin's opacity | 0.588 | 0.685 | 0.718 | 0.802 | 0.827 | 0.825 |
| `A_default_s0`: far side's under two frames | 0.171 | 0.172 | 0.139 | 0.148 | 0.152 | 0.184 |

What the tables show:

- **On the near side the under-two-frames share is higher after frame 80** in 44 of the 45 checkpoints, by 0.096 at the median, ten times the seeds' spread; for `A_default_s0` from 0.605 to 0.703. The far side's changes little (median +0.009).
- **The near side's far-5 cm share is not clearly higher after frame 80.** It rises by more than 0.01 in 28 checkpoints and falls by more than 0.01 in 8, median +0.016, about the seeds' spread before frame 80 (up to 0.015); for `A_default_s0` it goes from 0.130 to 0.125.
- **Over the whole model the far-5 cm share is lower after frame 80** (44 of 45 by more than 0.01), because the near side, where the share is low, holds more of the after half's opacity (0.83 against 0.70 for `A_default_s0`): the person is nearer and larger then. The whole model's figures mix the two sides, so the near and far sides are read apart.
- **The rise does not follow the split at frame 80.** By bins, the near side's under-two-frames share climbs from frames 0–19 to 80–99 as the person comes from 1.24 to 0.85 along the centre camera's axis, already from 0.48 to 0.68 before frame 80, and falls again in frames 100–119 (0.66), where the person is nearest and section 2.5's disagreement is still high. Both ends of the clip read lowest, which may be an effect of the clip's ends on durations as much as of the scene; this run does not separate it.

**The reading** (section 5.1, row (b)). The under-two-frames share is higher after frame 80; the far-5 cm share, the measure check (a) leaves standing as a criterion, is not clearly higher on the near side.

- **The far-5 cm share does not follow the disagreement in time.** Five of the seven camera pairs disagree 2.2 to 4 times more after frame 80, and the near side's far-5 cm share moves by a median of +0.016, inside the seeds' spread on most checkpoints. Per section 5.1, this weakens finalist 2 most, since it acts on the part that changes with the frame.
- **The under-two-frames share does rise,** by about 0.1 on the near side, which would be the size of effect to expect if it measured the disagreement. Check (a) found it the method's habit on moving content, and the bins show it rising with the person's approach from the first frames, so the person being closest, larger and faster in the image explains it as well as the disagreement does.
- **The two causes are not separated,** as section 5.1 warns: the person is closest in the same frames. The far side's shares, which see the same frames without the person, change little, which is all the control this split has.

**The fourth nine-camera model: the default configuration from a stride-1 init.**

**The checkpoint** is the one behind the link the owner allowed to be followed (PRD #1, item 14): 30,000 iterations of openftgs 0.1.0, the default configuration with the 3,000,000 cap written out, seed 0, every one of the nine cameras training (its manifest records the full scene manifest's SHA-256), S = 0.416. Only the checkpoint folder was opened (row T18.3).

**Its init points file is not among the kept ones.** The score file beside the link names none, and no file in `<private-scene-A-exp>` gives its far-5 cm share of 0.23426: the 15 init points files give 0.216 to 0.304, the stride-1 files of the two copies with a test view 0.2446 and 0.2444, the nine-camera stride-4 file 0.2469 (`<surface-runs>/ticket-18/default9_init_probe.json`). The file the model trained from would be `ftgs init roma` at stride 1 on all nine cameras, which differs from those copies' files, since D-66 keeps a test camera's images out of initialization. So the model has no far-5 cm share against its own init here; it is read against the files it shares with the other checkpoints, as section 2.6 requires for any comparison of the share.

**Its measures beside `A_default_s0`.** "Stride-1 file" is the stride-1 init of the copy with the centre camera as the test view, `A_default_s0`'s own; "nine-camera stride-4 file" is `stride4`'s own. The stride-4 pair section 4.2 read is repeated below them.

| Checkpoint | Cameras training | Far-5 cm share against the stride-1 file: whole / near / far side | Against the nine-camera stride-4 file: whole / near | Under two frames: whole / near / far side | Near side's share of opacity | Median duration, frames | Mean σ | σ < 0.05 | σ > 0.5 | σ quartiles | σ < 0.005 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| The fourth model | nine | 0.245 / 0.116 / 0.624 | 0.247 / 0.120 | 0.518 / 0.655 / 0.152 | 0.746 | 1.91 | 0.341 | 0.309 | 0.297 | 0.034 / 0.141 / 0.690 | 0.084 |
| `A_default_s0` | eight, not the centre | 0.259 / 0.128 / 0.649 | 0.264 / 0.134 | 0.517 / 0.647 / 0.162 | 0.749 | 1.92 | 0.360 | 0.285 | 0.315 | 0.040 / 0.160 / 0.755 | 0.074 |
| `A_default_s1` | eight, not the centre | 0.257 / 0.131 / 0.647 | 0.256 / 0.131 | 0.519 / 0.648 / 0.155 | 0.755 | 1.91 | 0.358 | 0.286 | 0.313 | 0.040 / 0.159 / 0.751 | 0.074 |
| `stride4` | nine | 0.200 / 0.105 / 0.545 | 0.209 / 0.122 | 0.456 / 0.590 / 0.048 | 0.784 | 2.25 | 0.337 | 0.327 | 0.294 | 0.030 / 0.133 / 0.686 | 0.103 |
| `c3_s4` | eight, not the centre | 0.200 / 0.106 / 0.542 | 0.206 / 0.116 | 0.447 / 0.580 / 0.055 | 0.785 | 2.29 | 0.360 | 0.301 | 0.317 | 0.035 / 0.154 / 0.783 | 0.092 |

Against the file with three times the points the fourth model reads 0.216 (near side 0.096) and `A_default_s0` 0.231 (0.110). Split at frame 80, the fourth model reads like the other default checkpoints (its row in check (b)'s table is not given there; `<surface-runs>/ticket-18/split80.json`, label `nine:default`): far-5 cm share 0.272 / 0.202 over the whole model and 0.120 / 0.109 on the near side, under two frames 0.457 / 0.615 and 0.608 / 0.717, before / after.

**Its sheet**, `<surface-runs>/ticket-18/sheets/side_nine-default.jpg`, from section 2.6's fixed request `<surface-runs>/ticket-5/side_request.json` (its SHA-256 checked as `1b88fd85…98821` before the render), laid out as section 2.6's sheets: 13 poses by 3 frames, every pose but `sC00` a novel view, as for the other nine-camera models. What could be checked without looking (`side_nine-default.coverage.json` beside it, and `<surface-runs>/ticket-18/diff_*.json`):

- Every rendered value is finite. A tile's share of pixels with alpha ≥ 0.5 is 0.98 to 1.00 in the poses between cameras and in `sC00`, and 0.54 to 0.89 at yaw ±60° and ±90°, as for `A_default_s0` (0.60 to 0.93); nothing like `stride4`'s fully covered, nearly black tiles at yaw +60° (section 2.6).
- `A_default_s0` was rendered again from the same request: its coverage figures equal ticket #5's to the last digit. Then, pose by pose, the mean absolute difference of the clamped renders from `A_default_s0`'s, for the fourth model and for `A_default_s1`, the seed yardstick:

| Pose | `nm01` | `nm03` | `nm05` | `nx02` | `sL90` | `sL60` | `sL45` | `sL30` | `sC00` | `sR30` | `sR45` | `sR60` | `sR90` |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| The fourth model against `A_default_s0` | 0.024 | 0.022 | 0.022 | 0.027 | 0.089 | 0.087 | 0.075 | 0.056 | 0.027 | 0.043 | 0.058 | 0.067 | 0.080 |
| `A_default_s1` against `A_default_s0` | 0.021 | 0.018 | 0.020 | 0.027 | 0.090 | 0.066 | 0.062 | 0.050 | 0.020 | 0.042 | 0.058 | 0.068 | 0.081 |

Over the 39 tiles the fourth model differs from `A_default_s0` by 1.09 times as much as the other seed does: most in `sC00` (1.4 times), where the fourth model trained on the centre camera and `A_default_s0` did not, and at yaw −45° and −60° (1.2 and 1.3 times); elsewhere within 0.006 of the seeds' difference.

**The reading** (section 5.1, row "Model"). It gives the nine-camera default its row, and repeats at stride 1 section 4.2's reading that the shell is the same with `cam0` in training or out of it, with a qualification.

- **The under-two-frames share is the same,** 0.518 against 0.517 and 0.519, inside the seeds' 0.004, on both sides; the median duration too (1.91 frames).
- **The far-5 cm share is a little lower with `cam0` in training:** 0.245 against 0.259 and 0.257 over the whole model, 0.116 against 0.128 and 0.131 on the near side, against the same file, and so against the other two files. That is 0.012 to 0.014, just beyond section 2.6's 0.01 between seeds; at stride 4 the pair differs by 0.000 to 0.003. Mean σ is 0.019 lower.
- **The renders differ from `A_default_s0`'s about as much as another seed's do,** except in the centre camera's own pose and on its left.
- **What the reading does not settle.** One run each, and the two also differ by one training camera, as section 4.2's pair did; and the fourth model's own init file is not kept, so its far-5 cm share against its own init stays unknown. No finalist's place changes (section 5.1).

**What is open.**

- **The fourth model's init points file.** Its own far-5 cm share needs the file it trained from, or the owner's leave to make it again by `ftgs init roma` at stride 1 on the full scene manifest, which would be checked against the score file's 0.23426 before use (one GPU run of minutes). Everything else about the model is measured.
- **The sheet** has not been seen by anyone yet.

**The commands.**

Each ran with `CUDA_VISIBLE_DEVICES=` empty unless it is marked GPU, `OMP_NUM_THREADS=5 MKL_NUM_THREADS=5 OPENBLAS_NUM_THREADS=5`, `PYTHONPATH=<worktree>/src:<worktree>/harness/src` and the shared `.venv`'s Python; the scripts under `<scratch>/ticket-18/` are not committed. `$T` is `<scratch>/ticket-18`, `$O` is `<surface-runs>/ticket-18`, `$X` is `<private-scene-A-exp>`, `$G` is `$OPENFTGS_DATASETS/synth/ground_truth`.

| Command | Where | Time | Note |
| --- | --- | --- | --- |
| `python $T/probe_init.py <fourth-model> $X/init*.safetensors --target 0.23425932313261924 --out $O/default9_init_probe.json` | CPU | 4 s | The fourth model's far-5 cm share against each of the 15 kept init points files; none gives the score file's. `<fourth-model>` is the checkpoint behind the link |
| `python $T/split80.py $X <private-scene-A>/scene.json <surface-runs>/ticket-5/shell_measures.json --out $O/split80.json --extra nine:default=<fourth-model>=<stride-1 file> --against <file>`, with `--against` for the two stride-1 files, the nine-camera stride-4 file and the stride-1 file with 3,000 points per frame | CPU | 4 min 12 s | Check (b) on the 45 checkpoints and the fourth model; each checkpoint's own init from ticket #5's file; the 45 whole-model rows equal ticket #5's |
| `python $T/bins.py $X <private-scene-A>/scene.json $O/split80.json --out $O/bins.json <label>=<checkpoint> …`, six checkpoints | CPU | 34 s | Check (b)'s control by 20-frame bins |
| `python $T/sweep_a.py $X <private-scene-A>/scene.json $O/split80.json --out $O/sweep_a.json <label>=<checkpoint> …`, three checkpoints | CPU | about 20 s | The far share at 0.02·S to 0.25·S on private scene A |
| `python $T/synth_measures.py <m17-runs>/<scene>/default-395/checkpoint <m17-runs>/<scene>/init.safetensors $G/<scene> --scene-name <scene> --out $O/synth_<scene>.json`, for the three scenes | CPU | 60 s, 60 s, 32 s | Check (a): 6,000, 5,400 and 1,517 mask files read. A first run without the 0.02·S to 0.25·S columns gave the same figures |
| The same for `kitchen`'s `default`, `seed1` and `default-326`, `courtyard`'s `default` and `studio`'s `default`, `default-325` and `default-341`, `--out $O/synth_<scene>-<run>.json` | CPU | 56 to 101 s each | The spread between trainings |
| `ftgs render <checkpoint> --request <surface-runs>/ticket-5/side_request.json --out $O/renders/<name>.safetensors`, for the fourth model, `A_default_s0` and `A_default_s1`, from `$T/run_gpu.sh` under the shared GPU lock | GPU | 6.9, 6.5 and 6.7 s | 39 renders each, between 18:07:41 and 18:08:01; the card's memory peaked at 6,741 MiB, 1,202 MiB of it in use before; the renders files were deleted once the sheet and the differences were made |
| `python $T/side_sheet.py $O/renders/nine-default.safetensors <surface-runs>/ticket-5/side_request.json $O/sheets/side_nine-default.jpg --label … --stats $O/sheets/side_nine-default.coverage.json` | CPU | about 1 s | The sheet, by ticket #5's script copied unchanged, and its coverage numbers |
| `python $T/pairdiff.py $O/renders/<a>.safetensors $O/renders/A_default_s0.safetensors <surface-runs>/ticket-5/side_request.json --out $O/diff_<a>_vs_A_default_s0.json`, for the fourth model and `A_default_s1` | CPU | about 2 s each | The differences of the renders, per tile |
| `python $T/tables_a.py $O`, `python $T/tables_b2.py $O/split80.json` | CPU | under 1 s | The rows of this section's tables, from the files above |

The private scene's folders were read and never written.

### 2.2 Check (c): one camera fit per held-out copy, on the background

*Ticket #19's section.* One correction per camera, fitted on the background once per held-out copy without that copy's test-view images, then sections 2.5's and 2.7's measures again on held-out frames, stated in section 5.3's two cases.

### 2.3 Checks (d) and (e): two seeds' disagreement where the shell is, and colour between cameras

*Ticket #20's section.* `A_default_s0` against `A_default_s1` where the shell is; and the colour difference at the two ends of section 2.5's admitted matches, per camera pair, against `kitchen`'s floor.

### 2.4 Check (f): a depth channel's gradient in the colour pass against Q-226's device

*Ticket #21's section.* The gradient of a depth channel in the colour pass against the same loss through Q-226's device, in U-12's configuration, held to U-12's bounds.

### 2.5 The three reads: FreeTimeGS++, SparseCam4D, SSA-3DGS

*Ticket #22's section.* FreeTimeGS++ in full (arXiv 2605.03337), SparseCam4D (arXiv 2603.26481) and SSA-3DGS (arXiv 2607.05598), each ending in a screening row of its family with section 3's nine columns.

### 2.6 Step 1's review: the checks' readings and the owner's go for the baseline

*Ticket #23's section.* Every result of step 1 read against section 5.1's last column, a recommendation per finalist, keep or drop, and the ask for step 2.

**The owner's answer on step 1** (each finalist kept or dropped, and the go for step 2 or a stop): *not yet given.* **Date:** *not yet given.*

## 3. Step 2: the baseline, the wrapper script and its parity runs

*Ticket #24's section.* The seven baseline runs by plain `ftgs train`, each with its measures, its sheet or `scripts/gt_depth.py`; and the scratch script's runs with every added weight at 0 against a plain run, on private scene A and on `kitchen`, within openftgs issue 395's tolerance.

**The owner's answer on step 2** (the go for step 3 or a stop): *not yet given.* **Date:** *not yet given.*

## 4. Step 3: finalist 1, the cameras corrected once

*Ticket #25's section.* Finalist 1's arms on the corrected cameras, run as the research document's section 5.3 says and judged by section 4.4's criteria.

**The owner's answer on step 3** (the go for step 4 or a stop): *not yet given.* **Date:** *not yet given.*

## 5. Step 4: finalist 2, the alignment-tolerant pixel loss

*Ticket #26's section.* Finalist 2's patch search, timed on one training image, then its arms, run as the research document's section 5.4 says and judged by section 4.4's criteria.

**The owner's answer on step 4** (the go for step 5 or a stop): *not yet given.* **Date:** *not yet given.*

## 6. Step 5: finalist 3, surface terms without a prior

*Ticket #27's section.* Row A5's plane-distance depth checked on one flat, slanted primitive, then finalist 3's arms, flattening, then depth–normal consistency, run as the research document's section 5.5 says and judged by section 4.4's criteria.

**The owner's answer on step 5** (the go for step 6 or a stop): *not yet given.* **Date:** *not yet given.*

## 7. Step 6: finalist 4, a normal prior on top of finalist 3's flattening

*Ticket #28's section.* Section 2.7's measurement repeated for normals, with `kitchen` for the floor, then finalist 4's arm, run as the research document's section 5.6 says and judged by section 4.4's criteria.

**The owner's answer on step 6** (whether the work goes on to the recommendation, or stops): *not yet given.* **Date:** *not yet given.*

## 8. The recommendation

*Ticket #29's section.* Per finalist that ran: adopt, drop, or test in combination; a finalist the owner stopped before its step says so. Where it recommends adopting, the spec change's wording proposed for the owner, on an issue of its own.

## 9. The sources

*Ticket #29's section.* The clean-room record in full, completing the opening's: the source-log rows of every ticket of the PRD, what was read only as documentation, what was not opened, and any incident.
