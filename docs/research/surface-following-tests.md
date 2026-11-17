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

**How the measures are taken.** As section 2.6 takes them (row 845; row 846; row 848): the exchange model's values from `load_checkpoint` and `FreeTimeGaussians.to_exchange()` (μx, E3's base opacity σ, E4's duration s and time centre μt, in seconds), the distance from each μx to the nearest point of the init points file the run trained from (SciPy's `cKDTree`, k = 1, all init frames together), and the scorer's expressions written anew. A primitive is *far* when that distance exceeds the threshold; the far share is the far primitives' share of the summed σ. The *under-two-frames share* is the share of primitives with s under two frame intervals. The *near side* on private scene A is section 2.6's: μx nearer than 2.0 along the centre camera's axis; the far side is the rest. Two checks hold the scripts to section 2.6's:

- On all 45 kept checkpoints the whole model's eight figures (N, the two far-5 cm shares, the under-two-frames share, the median duration and the three opacity figures) equal ticket #5's `<surface-runs>/ticket-5/shell_measures.json` in every digit, and the near side's far-5 cm share of `A_default_s0` is 0.128, as in section 2.6's second table.
- The fourth model's six figures that do not depend on an init points file (N, the median duration, the under-two-frames share and the three opacity figures) equal the notes' score file beside its link in every digit.

**Check (a): openftgs-synth, where the cameras agree.**

**The rules** (section 5.1, row (a); D-27 for S).

- **The checkpoints:** M17's `default-395` runs of `kitchen`, `courtyard` and `studio` (steps 35 to 37 of `docs/prd/validation-runs.md`), each against its scene's init points file `<m17-runs>/<scene>/init.safetensors`, which every default run of the scene trained from: 300,000, 300,000 and 63,000 points, against private scene A's 120,000 (row 847).
- **The 5 cm** is 0.12·S: S is 1.597, 6.343 and 4.663, so the threshold is 0.192, 0.761 and 0.560 m. Beside it: the share against a seeded subsample of the init file to 120,000 points (`kitchen` and `courtyard`; `studio`'s file is smaller), and the literal 5 cm.
- **Two frames** are 2/fps at each scene's frame rate (row 844): 2/30 s on `kitchen`, 2/60 s on `courtyard` and `studio`. Beside it, private scene A's absolute 2/24 s.
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

**The far share at other thresholds,** to see whether the reading rests on 0.12·S: other multiples of S, and distances in metres, since the moving objects are of human size in every scene while S follows the rig. Each cell is the far share of σ over the whole model, then on the near side (private scene A) or the dynamic side by the main rule (openftgs-synth).

| Checkpoint | 0.02·S | 0.05·S | 0.12·S | 0.25·S | 5 cm | 10 cm | 25 cm |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `A_default_s0` | 0.751 / 0.675 | 0.446 / 0.307 | 0.259 / 0.128 | 0.175 / 0.076 | 0.259 / 0.128 | 0.179 / 0.078 | 0.103 / 0.047 |
| `A_default_s1` | 0.753 / 0.680 | 0.447 / 0.312 | 0.257 / 0.131 | 0.169 / 0.072 | 0.257 / 0.131 | | |
| `c3_s4` | 0.758 / 0.701 | 0.438 / 0.340 | 0.213 / 0.124 | 0.135 / 0.068 | 0.213 / 0.124 | 0.137 / 0.069 | 0.081 / 0.043 |
| The fourth model, against the stride-1 file | | | | | 0.245 / 0.116 | 0.163 / 0.064 | 0.090 / 0.035 |
| `kitchen` `default-395` | 0.390 / 0.473 | 0.130 / 0.081 | 0.035 / 0.003 | 0.009 / 0.000 | 0.237 / 0.227 | 0.092 / 0.038 | 0.023 / 0.001 |
| `courtyard` `default-395` | 0.118 / 0.030 | 0.045 / 0.000 | 0.012 / 0.000 | 0.003 / 0.000 | 0.359 / 0.457 | 0.154 / 0.075 | 0.059 / 0.001 |
| `studio` `default-395` | 0.136 / 0.038 | 0.072 / 0.001 | 0.042 / 0.001 | 0.021 / 0.000 | 0.333 / 0.277 | 0.126 / 0.029 | 0.069 / 0.001 |

The init points' spacing, the distance from each to its nearest neighbour, is 5.1, 12 and 23 mm at the median on `kitchen`, `courtyard` and `studio`, and 8.8 mm on private scene A's stride-1 file. Blank cells were not run.

What the tables show:

- **Measured in S, the far share is clearly lower where the cameras agree,** at 0.12·S and at every other multiple of S. On the dynamic side it is 0.000 to 0.003 against private scene A's 0.12 to 0.13 on the near side, 0.008 at most against a 120,000-point sample; over the whole model 0.012 to 0.042 against 0.21 to 0.26, and on the static side 0.018 to 0.092 against the far side's 0.52 to 0.78. The earlier trainings' far shares are within 0.01 of the 549o ones.
- **Measured in metres, it is not.** 0.12·S is 19 to 76 cm on openftgs-synth, against 5 cm on private scene A, for moving objects of the same human size. At 5 cm openftgs-synth's dynamic side reads 0.23 to 0.46, more than private scene A's near side (0.12 to 0.13); at 10 cm 0.03 to 0.08, like it (0.06 to 0.08). Only beyond 25 cm does private scene A keep a tail that openftgs-synth lacks: 0.035 to 0.047 of the near side's opacity against 0.001. The init points are no sparser on openftgs-synth to explain it: their median spacing is 5 to 23 mm, against 8.8 mm.
- **The under-two-frames share is not lower on the moving parts.** On the dynamic side it is 0.66, 0.49 and 0.71 at 2/fps (0.68 to 0.69 and 0.62 and 0.69 in the earlier trainings), against 0.647 on private scene A's near side; at private scene A's absolute 2/24 s it is 0.77, 0.89 and 0.95. Over the whole model it is lower, 0.33 to 0.39 at 549o (0.39 to 0.49 before), against 0.45 to 0.52, because the static side, where it is 0.15 to 0.32, holds 45 % to 64 % of the opacity, against 25 % for private scene A's far side. On the dynamic side the primitives far from init are too few to compare (72 to 2,918 of 0.6 to 1.3 million).
- **The base opacity is lower over the whole model** on openftgs-synth (mean σ 0.17 to 0.23 against 0.36; σ < 0.05 for 0.49 to 0.61 of the primitives against 0.29; 17 % to 23 % dead against 7 %), while on the dynamic side `kitchen`'s is higher than private scene A's (mean 0.48, median 0.39) and the other two scenes' lower (0.29 and 0.25). The 549o and earlier trainings differ by up to 0.06 in mean σ, so the distribution is read only roughly.
- **The dynamic-side rule does not change the reading.** "Any camera" and "every camera" give far shares of 0.000 to 0.017 and under-two-frames shares of 0.41 to 0.71; "any camera" lowers the latter by taking in the static scene behind the moving objects, most on `studio`'s ring of 24 cameras, where it holds 84 % of the opacity.
- **FreeTimeGS++'s account of short-lived primitives** does not bear on these numbers directly: its "lifetime" depends on the base opacity as well as on s, and its reproduction starts from other initial values than §1.3 (section 2.5 here).

**The reading** (section 5.1, row (a)). The result is split by measure, so neither branch of the row's last column holds for both.

- **The far-5 cm share is split by its ruler.** Restated as 0.12·S, as the plan restates it, it is near 0 on openftgs-synth's moving parts, so by section 5.1's column it would stand as a criterion, and finalist 1's control on `kitchen` has its baseline in the tables above. Measured in metres, the 5 to 25 cm band the notes give the shell (section 2.6) is as full on openftgs-synth's moving objects as on private scene A's person, so that band is the method's habit too; what private scene A has and openftgs-synth lacks is the tail beyond about 25 cm. Which ruler is right is not settled here; the far share beyond 25 cm is the part of the measure that separates the two in both rulers, and ticket #23 may propose it.
- **The under-two-frames share is the method's habit on moving content.** It is as high on openftgs-synth's moving objects, where the cameras agree exactly, as on private scene A's near side. A fall in it would not show that a disagreement was removed. As a test of H2 by removal, finalists 1 and 2 keep their reason through the far share and the sheet, not through the under-two-frames share.
- **Caveats.** One training per scene at 549o. The init points files are larger than private scene A's on two scenes (a 120,000-point sample changes the far share by 0.005 at most). The dynamic side is not private scene A's near side: it is the moving objects alone, while the near side holds the person, what lies near the person and nothing else nearer than 2.0. The reading is ticket #23's to confirm (section 2.6 here).

**Check (b): private scene A, split by time centre at frame 80.**

**The rules** (section 5.1, row (b)). The 45 kept checkpoints of section 2.6, each against its own init points file as section 2.6 assigns it, with the 5 cm, the 2/24 s and the near side unchanged. A primitive is *after* frame 80 when the frame nearest its time centre μt is 80 or later (μt ≥ 79.5/24 s, so frames 80 to 119, section 2.5's late part) and *before* otherwise; μt outside the clip goes with the nearer end. Every share below is taken within its half, so the two halves recombine to section 2.6's figures.

**The measures, per checkpoint.** Each cell is before / after. "Near share" is the near side's share of the half's opacity; "after" in the last column is the after half's share of the model's primitives and of its opacity. Runs are named as in section 2.6.

| Checkpoint | Whole: far-5 cm share | Whole: under two frames | Whole: mean σ | Whole: σ < 0.05 | Whole: σ > 0.5 | Near: far-5 cm share | Near: under two frames | Near: mean σ | Near: σ < 0.05 | Near: σ > 0.5 | Far side: far-5 cm share | Far side: under two frames | Near share of the half's opacity | After: primitives / opacity |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `stride4` | 0.228 / 0.183 | 0.382 / 0.564 | 0.330 / 0.346 | 0.324 / 0.331 | 0.285 / 0.306 | 0.115 / 0.130 | 0.533 / 0.659 | 0.346 / 0.358 | 0.315 / 0.323 | 0.303 / 0.319 | 0.522 / 0.548 | 0.047 / 0.051 | 0.722 / 0.872 | 0.404 / 0.415 |
| `stride4_zero_velocity` | 0.232 / 0.187 | 0.386 / 0.552 | 0.326 / 0.335 | 0.325 / 0.329 | 0.281 / 0.293 | 0.119 / 0.130 | 0.540 / 0.648 | 0.344 / 0.343 | 0.316 / 0.325 | 0.301 / 0.303 | 0.541 / 0.557 | 0.034 / 0.032 | 0.733 / 0.866 | 0.404 / 0.410 |
| `stride4_zero_velocity_reg0.1` | 0.174 / 0.128 | 0.313 / 0.475 | 0.110 / 0.121 | 0.542 / 0.537 | 0.054 / 0.066 | 0.058 / 0.082 | 0.424 / 0.531 | 0.120 / 0.125 | 0.535 / 0.532 | 0.064 / 0.070 | 0.647 / 0.677 | 0.001 / 0.008 | 0.804 / 0.924 | 0.404 / 0.426 |
| `A_default_s0` | 0.284 / 0.220 | 0.463 / 0.605 | 0.355 / 0.366 | 0.287 / 0.282 | 0.310 / 0.323 | 0.130 / 0.125 | 0.605 / 0.703 | 0.365 / 0.371 | 0.269 / 0.274 | 0.319 / 0.327 | 0.641 / 0.669 | 0.160 / 0.169 | 0.699 / 0.826 | 0.386 / 0.393 |
| `A_default_s1` | 0.281 / 0.221 | 0.462 / 0.610 | 0.355 / 0.364 | 0.287 / 0.284 | 0.309 / 0.321 | 0.130 / 0.132 | 0.605 / 0.705 | 0.364 / 0.370 | 0.269 / 0.275 | 0.317 / 0.326 | 0.638 / 0.670 | 0.149 / 0.172 | 0.703 / 0.835 | 0.387 / 0.393 |
| `c3_s4` | 0.249 / 0.163 | 0.380 / 0.545 | 0.351 / 0.374 | 0.298 / 0.307 | 0.305 / 0.335 | 0.138 / 0.108 | 0.530 / 0.639 | 0.369 / 0.388 | 0.285 / 0.296 | 0.326 / 0.351 | 0.540 / 0.534 | 0.051 / 0.065 | 0.724 / 0.869 | 0.407 / 0.422 |
| `c3_s4_reg0.1` | 0.174 / 0.144 | 0.316 / 0.464 | 0.119 / 0.130 | 0.513 / 0.517 | 0.059 / 0.073 | 0.065 / 0.103 | 0.422 / 0.519 | 0.129 / 0.135 | 0.504 / 0.509 | 0.070 / 0.078 | 0.640 / 0.668 | 0.004 / 0.013 | 0.811 / 0.928 | 0.407 / 0.429 |
| `c3_s4v0` | 0.240 / 0.216 | 0.388 / 0.545 | 0.349 / 0.360 | 0.297 / 0.307 | 0.303 / 0.318 | 0.136 / 0.159 | 0.530 / 0.642 | 0.367 / 0.370 | 0.285 / 0.301 | 0.323 / 0.330 | 0.536 / 0.576 | 0.047 / 0.033 | 0.740 / 0.864 | 0.411 / 0.418 |
| `c3_s4v0_reg0.1` | 0.171 / 0.155 | 0.321 / 0.461 | 0.120 / 0.130 | 0.512 / 0.513 | 0.059 / 0.073 | 0.063 / 0.109 | 0.426 / 0.520 | 0.130 / 0.135 | 0.503 / 0.506 | 0.070 / 0.078 | 0.655 / 0.694 | 0.002 / 0.011 | 0.819 / 0.921 | 0.413 / 0.432 |
| `sh0` | 0.292 / 0.219 | 0.452 / 0.598 | 0.327 / 0.325 | 0.313 / 0.317 | 0.279 / 0.278 | 0.133 / 0.106 | 0.602 / 0.703 | 0.331 / 0.322 | 0.301 / 0.315 | 0.282 / 0.275 | 0.624 / 0.671 | 0.150 / 0.166 | 0.676 / 0.799 | 0.380 / 0.379 |
| `sh1` | 0.290 / 0.227 | 0.450 / 0.606 | 0.337 / 0.338 | 0.308 / 0.313 | 0.290 / 0.294 | 0.125 / 0.123 | 0.601 / 0.709 | 0.340 / 0.338 | 0.296 / 0.308 | 0.293 / 0.293 | 0.628 / 0.642 | 0.147 / 0.193 | 0.673 / 0.800 | 0.382 / 0.383 |
| `B_usercfg_s0` | 0.247 / 0.205 | 0.398 / 0.592 | 0.410 / 0.418 | 0.199 / 0.200 | 0.362 / 0.372 | 0.101 / 0.131 | 0.549 / 0.686 | 0.436 / 0.432 | 0.174 / 0.186 | 0.390 / 0.387 | 0.632 / 0.639 | 0.074 / 0.138 | 0.724 / 0.854 | 0.423 / 0.427 |
| `B_usercfg_s1` | 0.244 / 0.206 | 0.405 / 0.593 | 0.407 / 0.419 | 0.201 / 0.200 | 0.358 / 0.373 | 0.105 / 0.126 | 0.548 / 0.687 | 0.433 / 0.431 | 0.176 / 0.187 | 0.386 / 0.386 | 0.611 / 0.656 | 0.095 / 0.155 | 0.726 / 0.848 | 0.416 / 0.423 |
| `b15_dead0.05` | 0.345 / 0.201 | 0.145 / 0.193 | 0.221 / 0.237 | 0.365 / 0.409 | 0.155 / 0.189 | 0.076 / 0.087 | 0.267 / 0.241 | 0.238 / 0.246 | 0.429 / 0.432 | 0.195 / 0.207 | 0.723 / 0.745 | 0.000 / 0.001 | 0.584 / 0.826 | 0.272 / 0.286 |
| `b15_durlr0.2` | 0.395 / 0.314 | 0.768 / 0.856 | 0.501 / 0.497 | 0.191 / 0.173 | 0.470 / 0.462 | 0.149 / 0.120 | 0.861 / 0.911 | 0.464 / 0.465 | 0.181 / 0.171 | 0.423 / 0.422 | 0.655 / 0.716 | 0.654 / 0.713 | 0.513 / 0.674 | 0.341 / 0.339 |
| `b15_durlr4` | 0.178 / 0.119 | 0.593 / 0.731 | 0.348 / 0.342 | 0.298 / 0.313 | 0.305 / 0.300 | 0.072 / 0.077 | 0.711 / 0.784 | 0.360 / 0.346 | 0.291 / 0.310 | 0.319 / 0.305 | 0.590 / 0.577 | 0.201 / 0.241 | 0.795 / 0.915 | 0.405 / 0.400 |
| `b15_init3k` | 0.203 / 0.141 | 0.447 / 0.577 | 0.366 / 0.379 | 0.250 / 0.240 | 0.317 / 0.331 | 0.067 / 0.052 | 0.589 / 0.667 | 0.388 / 0.389 | 0.222 / 0.222 | 0.339 / 0.341 | 0.519 / 0.525 | 0.171 / 0.241 | 0.699 / 0.813 | 0.379 / 0.387 |
| `b15_init_d4` | 0.219 / 0.152 | 0.333 / 0.453 | 0.361 / 0.379 | 0.258 / 0.257 | 0.312 / 0.334 | 0.085 / 0.077 | 0.462 / 0.536 | 0.386 / 0.394 | 0.239 / 0.243 | 0.339 / 0.350 | 0.613 / 0.617 | 0.034 / 0.051 | 0.747 / 0.862 | 0.394 / 0.405 |
| `b15_init_s4` | 0.218 / 0.180 | 0.328 / 0.479 | 0.369 / 0.396 | 0.242 / 0.242 | 0.319 / 0.351 | 0.091 / 0.121 | 0.466 / 0.570 | 0.394 / 0.415 | 0.224 / 0.227 | 0.346 / 0.372 | 0.558 / 0.554 | 0.032 / 0.053 | 0.728 / 0.864 | 0.393 / 0.410 |
| `b15_init_s8` | 0.223 / 0.178 | 0.294 / 0.439 | 0.367 / 0.420 | 0.251 / 0.245 | 0.317 / 0.379 | 0.093 / 0.135 | 0.429 / 0.518 | 0.395 / 0.445 | 0.233 / 0.230 | 0.348 / 0.407 | 0.545 / 0.510 | 0.029 / 0.030 | 0.713 / 0.886 | 0.392 / 0.424 |
| `b15_init_v0` | 0.263 / 0.208 | 0.404 / 0.553 | 0.382 / 0.394 | 0.225 / 0.222 | 0.331 / 0.346 | 0.098 / 0.101 | 0.551 / 0.655 | 0.401 / 0.402 | 0.207 / 0.215 | 0.352 / 0.355 | 0.695 / 0.727 | 0.078 / 0.109 | 0.723 / 0.829 | 0.395 / 0.402 |
| `b15_noreloc` | 0.292 / 0.166 | 0.306 / 0.509 | 0.163 / 0.200 | 0.694 / 0.644 | 0.145 / 0.182 | 0.193 / 0.126 | 0.393 / 0.567 | 0.167 / 0.208 | 0.706 / 0.638 | 0.153 / 0.190 | 0.680 / 0.720 | 0.003 / 0.005 | 0.798 / 0.932 | 0.415 / 0.465 |
| `b15_reg0.1` | 0.191 / 0.151 | 0.340 / 0.475 | 0.158 / 0.174 | 0.419 / 0.412 | 0.090 / 0.108 | 0.076 / 0.092 | 0.434 / 0.532 | 0.172 / 0.182 | 0.405 / 0.399 | 0.105 / 0.116 | 0.731 / 0.766 | 0.044 / 0.092 | 0.825 / 0.912 | 0.407 / 0.430 |
| `b15_reg0.3` | 0.194 / 0.130 | 0.199 / 0.336 | 0.056 / 0.061 | 0.699 / 0.683 | 0.013 / 0.016 | 0.080 / 0.092 | 0.260 / 0.363 | 0.061 / 0.063 | 0.675 / 0.676 | 0.016 / 0.017 | 0.773 / 0.798 | 0.003 / 0.012 | 0.836 / 0.946 | 0.398 / 0.418 |
| `b15_scale3` | 0.298 / 0.236 | 0.449 / 0.563 | 0.315 / 0.332 | 0.322 / 0.304 | 0.268 / 0.287 | 0.098 / 0.090 | 0.632 / 0.694 | 0.331 / 0.337 | 0.304 / 0.299 | 0.286 / 0.292 | 0.655 / 0.697 | 0.164 / 0.174 | 0.641 / 0.759 | 0.343 / 0.355 |
| `b15_vgg` | 0.196 / 0.177 | 0.474 / 0.608 | 0.330 / 0.346 | 0.251 / 0.238 | 0.273 / 0.291 | 0.082 / 0.107 | 0.612 / 0.698 | 0.353 / 0.359 | 0.223 / 0.222 | 0.296 / 0.303 | 0.582 / 0.644 | 0.114 / 0.139 | 0.772 / 0.869 | 0.390 / 0.401 |
| `f_s4_reg0.1` | 0.165 / 0.132 | 0.278 / 0.406 | 0.139 / 0.155 | 0.452 / 0.445 | 0.073 / 0.092 | 0.064 / 0.095 | 0.369 / 0.454 | 0.153 / 0.162 | 0.438 / 0.434 | 0.087 / 0.099 | 0.643 / 0.667 | 0.003 / 0.010 | 0.825 / 0.934 | 0.399 / 0.425 |
| `f_s4_reg0.3` | 0.240 / 0.121 | 0.079 / 0.166 | 0.034 / 0.040 | 0.829 / 0.808 | 0.008 / 0.012 | 0.088 / 0.080 | 0.120 / 0.184 | 0.040 / 0.042 | 0.804 / 0.801 | 0.011 / 0.013 | 0.748 / 0.796 | 0.000 / 0.000 | 0.770 / 0.943 | 0.334 / 0.370 |
| `f_s4v0` | 0.216 / 0.172 | 0.330 / 0.465 | 0.371 / 0.389 | 0.240 / 0.242 | 0.320 / 0.343 | 0.094 / 0.114 | 0.468 / 0.553 | 0.397 / 0.405 | 0.223 / 0.232 | 0.349 / 0.360 | 0.558 / 0.548 | 0.024 / 0.025 | 0.736 / 0.866 | 0.399 / 0.411 |
| `f_s4v0_reg0.1` | 0.162 / 0.150 | 0.285 / 0.414 | 0.142 / 0.156 | 0.445 / 0.443 | 0.076 / 0.093 | 0.066 / 0.102 | 0.373 / 0.468 | 0.156 / 0.163 | 0.432 / 0.434 | 0.090 / 0.100 | 0.642 / 0.708 | 0.003 / 0.008 | 0.834 / 0.922 | 0.416 / 0.438 |
| `f_v0_reg0.1` | 0.193 / 0.166 | 0.333 / 0.484 | 0.161 / 0.176 | 0.413 / 0.405 | 0.092 / 0.111 | 0.076 / 0.101 | 0.431 / 0.548 | 0.175 / 0.184 | 0.397 / 0.394 | 0.107 / 0.119 | 0.752 / 0.810 | 0.029 / 0.062 | 0.827 / 0.909 | 0.418 / 0.440 |
| `n1500k` | 0.261 / 0.194 | 0.426 / 0.560 | 0.377 / 0.387 | 0.238 / 0.230 | 0.328 / 0.338 | 0.104 / 0.089 | 0.557 / 0.647 | 0.394 / 0.395 | 0.216 / 0.217 | 0.345 / 0.347 | 0.663 / 0.713 | 0.134 / 0.183 | 0.719 / 0.831 | 0.383 / 0.389 |
| `n750k` | 0.226 / 0.179 | 0.391 / 0.514 | 0.401 / 0.414 | 0.201 / 0.190 | 0.350 / 0.365 | 0.071 / 0.081 | 0.504 / 0.583 | 0.429 / 0.428 | 0.175 / 0.175 | 0.381 / 0.379 | 0.670 / 0.708 | 0.133 / 0.202 | 0.742 / 0.844 | 0.381 / 0.389 |
| `c3_s4v0_reg0.1`, iteration 5,000 | 0.321 / 0.208 | 0.150 / 0.278 | 0.183 / 0.188 | 0.473 / 0.433 | 0.129 / 0.132 | 0.231 / 0.129 | 0.240 / 0.350 | 0.197 / 0.194 | 0.422 / 0.407 | 0.138 / 0.135 | 0.503 / 0.556 | 0.002 / 0.001 | 0.669 / 0.816 | 0.395 / 0.402 |
| `c3_s4v0_reg0.1`, iteration 10,000 | 0.254 / 0.198 | 0.276 / 0.432 | 0.136 / 0.138 | 0.518 / 0.501 | 0.077 / 0.079 | 0.142 / 0.132 | 0.399 / 0.517 | 0.144 / 0.141 | 0.488 / 0.489 | 0.083 / 0.081 | 0.553 / 0.583 | 0.004 / 0.002 | 0.728 / 0.853 | 0.414 / 0.417 |
| `c3_s4v0_reg0.1`, iteration 15,000 | 0.190 / 0.170 | 0.317 / 0.464 | 0.112 / 0.112 | 0.518 / 0.529 | 0.047 / 0.049 | 0.072 / 0.113 | 0.423 / 0.532 | 0.117 / 0.114 | 0.505 / 0.524 | 0.051 / 0.052 | 0.605 / 0.629 | 0.002 / 0.005 | 0.779 / 0.890 | 0.426 / 0.425 |
| `c3_s4v0_reg0.1`, iteration 20,000 | 0.182 / 0.160 | 0.318 / 0.457 | 0.113 / 0.116 | 0.484 / 0.496 | 0.044 / 0.049 | 0.065 / 0.107 | 0.420 / 0.518 | 0.119 / 0.119 | 0.474 / 0.491 | 0.050 / 0.052 | 0.637 / 0.664 | 0.002 / 0.008 | 0.795 / 0.904 | 0.419 / 0.425 |
| `c3_s4v0_reg0.1`, iteration 25,000 | 0.176 / 0.156 | 0.318 / 0.455 | 0.117 / 0.123 | 0.465 / 0.468 | 0.046 / 0.055 | 0.064 / 0.106 | 0.422 / 0.514 | 0.125 / 0.128 | 0.451 / 0.460 | 0.054 / 0.059 | 0.650 / 0.685 | 0.002 / 0.009 | 0.808 / 0.915 | 0.413 / 0.426 |
| `h1_base` | 0.259 / 0.201 | 0.430 / 0.567 | 0.375 / 0.382 | 0.244 / 0.233 | 0.326 / 0.334 | 0.097 / 0.095 | 0.571 / 0.659 | 0.393 / 0.390 | 0.219 / 0.222 | 0.345 / 0.341 | 0.644 / 0.712 | 0.143 / 0.168 | 0.704 / 0.828 | 0.377 / 0.381 |
| `h1_init_s4` | 0.220 / 0.191 | 0.331 / 0.456 | 0.365 / 0.388 | 0.252 / 0.246 | 0.315 / 0.342 | 0.099 / 0.129 | 0.470 / 0.546 | 0.388 / 0.404 | 0.234 / 0.234 | 0.341 / 0.360 | 0.544 / 0.569 | 0.030 / 0.038 | 0.729 / 0.859 | 0.386 / 0.400 |
| `h1_init_s4_seed1` | 0.220 / 0.188 | 0.333 / 0.462 | 0.363 / 0.383 | 0.252 / 0.247 | 0.312 / 0.336 | 0.099 / 0.125 | 0.474 / 0.555 | 0.387 / 0.399 | 0.234 / 0.234 | 0.340 / 0.355 | 0.551 / 0.565 | 0.029 / 0.034 | 0.731 / 0.858 | 0.386 / 0.398 |
| `h1_s4_reg0.1` | 0.164 / 0.146 | 0.283 / 0.402 | 0.139 / 0.153 | 0.456 / 0.448 | 0.073 / 0.090 | 0.060 / 0.100 | 0.377 / 0.454 | 0.153 / 0.161 | 0.442 / 0.437 | 0.088 / 0.098 | 0.651 / 0.709 | 0.003 / 0.010 | 0.825 / 0.925 | 0.403 / 0.427 |
| `h1_s4v0` | 0.222 / 0.188 | 0.337 / 0.467 | 0.364 / 0.382 | 0.249 / 0.244 | 0.313 / 0.335 | 0.101 / 0.120 | 0.475 / 0.562 | 0.389 / 0.397 | 0.232 / 0.234 | 0.342 / 0.352 | 0.563 / 0.589 | 0.026 / 0.031 | 0.739 / 0.853 | 0.391 / 0.402 |
| `h1_s4v0_reg0.1` | 0.159 / 0.151 | 0.290 / 0.405 | 0.140 / 0.153 | 0.454 / 0.447 | 0.074 / 0.090 | 0.059 / 0.098 | 0.381 / 0.463 | 0.153 / 0.161 | 0.441 / 0.437 | 0.088 / 0.098 | 0.659 / 0.734 | 0.003 / 0.010 | 0.832 / 0.916 | 0.413 / 0.435 |
| `h1_s4v0_seed1` | 0.205 / 0.190 | 0.337 / 0.459 | 0.361 / 0.381 | 0.250 / 0.244 | 0.310 / 0.333 | 0.086 / 0.119 | 0.477 / 0.554 | 0.388 / 0.395 | 0.232 / 0.234 | 0.340 / 0.349 | 0.546 / 0.594 | 0.024 / 0.024 | 0.742 / 0.852 | 0.393 / 0.406 |

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

**The reading** (section 5.1, row (b)). The under-two-frames share is higher after frame 80; the far-5 cm share is not clearly higher on the near side, and this split cannot read it.

- **The far-5 cm share is not clearly higher after frame 80,** against the pooled init points: a median of +0.016 on the near side, with 28 checkpoints up and 8 down, while five of the seven camera pairs disagree 2.2 to 4 times more. Pooling the init frames biases this split, though: the middle of the clip is near init points from earlier and later frames, its ends only from one side. Compared only with the init points within 6 frames of each primitive's time centre, as section 2.6's windowed check did, the near side's share rises after frame 80 on all three checkpoints tried, 0.166 to 0.220 (`A_default_s0`), 0.167 to 0.264 (`c3_s4`) and 0.156 to 0.225 (the fourth model), but by bins it is highest at both ends of the clip (0.17 to 0.30 in frames 0–19 and 100–119 against 0.11 to 0.15 in frames 40–79), where the window is one-sided, and frames 20–39 and 80–99 read alike (0.163 and 0.185, 0.233 and 0.224, 0.175 and 0.198; `<surface-runs>/ticket-18/windowed_split.json`). So this split cannot read the far share: neither way of taking it is free of the clip's ends. Per section 5.1, a share no higher after frame 80 would weaken finalist 2 most; this result neither shows nor excludes that.
- **The under-two-frames share does rise,** by about 0.1 on the near side, which would be the size of effect to expect if it measured the disagreement. Check (a) found it the method's habit on moving content, and the bins show it rising with the person's approach from the first frames, so the person being closest and larger in the image explains it as well as the disagreement does.
- **The two causes are not separated,** as section 5.1 warns: the person is closest in the same frames. The far side's shares, which see the same frames without the person, change little, which is all the control this split has.

**The fourth nine-camera model: the default configuration from a stride-1 init.**

**The checkpoint** is the one behind the link the owner allowed to be followed (PRD #1, item 14): 30,000 iterations of openftgs 0.1.0, the default configuration with the 3,000,000 cap written out, seed 0, every one of the nine cameras training (its manifest records the full scene manifest's SHA-256), S = 0.416. Only the checkpoint folder was opened (row 846).

**Its init points file is not among the kept ones.** The score file beside the link names none, and no file in `<private-scene-A-exp>` gives its far-5 cm share of 0.23426: the 15 init points files give 0.216 to 0.304, the stride-1 files of the two copies with a test view 0.2446 and 0.2444, the nine-camera stride-4 file 0.2469 (`<surface-runs>/ticket-18/default9_init_probe.json`). The file the model trained from would be `ftgs init roma` at stride 1 on all nine cameras, which differs from those copies' files, since D-66 keeps a test camera's images out of initialization. So the model has no far-5 cm share against its own init here; it is read against the files it shares with the other checkpoints, as section 2.6 requires for any comparison of the share.

**The init made again, and not accepted.** At the owner's answer of 2026-11-16 (option 2 of this ticket's question), `ftgs init roma` was run at stride 1 on private scene A's full scene manifest, with the code at c70fe6b (`src/` as `main`'s) and the vendored RoMa's default weights (row 849): 120,000 points, `<surface-runs>/ticket-18/init9_s1_regenerated.safetensors`. Against it the fourth model's far-5 cm share is 0.23559, not the score file's 0.23426: 0.0013 more (`<surface-runs>/ticket-18/default9_init_probe_regenerated.json`). It is the nearest of every file tried, and nearer than the copies' stride-1 files (0.2446), but the owner's condition was that it reproduce the figure, so it is not taken as the model's own init and no figure against it enters the tables above. The likely cause is that the original was made by openftgs 0.1.0 with torch 2.14.1, and this one by openftgs-plus at c70fe6b with torch 2.15.0, where RoMa's matches and their sampling need not be bit-identical; this was not checked further.

**Its measures beside `A_default_s0`.** "Stride-1 file" is the stride-1 init of the copy with the centre camera as the test view, `A_default_s0`'s own; "nine-camera stride-4 file" is `stride4`'s own. The stride-4 pair section 4.2 read is repeated below them.

| Checkpoint | Cameras training | Far-5 cm share against the stride-1 file: whole / near / far side | Against the nine-camera stride-4 file: whole / near | Under two frames: whole / near / far side | Near side's share of opacity | Median duration, frames | Mean σ | σ < 0.05 | σ > 0.5 | σ quartiles | σ < 0.005 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| The fourth model | nine | 0.245 / 0.116 / 0.624 | 0.247 / 0.120 | 0.518 / 0.655 / 0.152 | 0.746 | 1.91 | 0.341 | 0.309 | 0.297 | 0.034 / 0.141 / 0.690 | 0.084 |
| `A_default_s0` | eight, not the centre | 0.259 / 0.128 / 0.649 | 0.264 / 0.134 | 0.517 / 0.647 / 0.162 | 0.749 | 1.92 | 0.360 | 0.285 | 0.315 | 0.040 / 0.160 / 0.755 | 0.074 |
| `A_default_s1` | eight, not the centre | 0.257 / 0.131 / 0.647 | 0.256 / 0.131 | 0.519 / 0.648 / 0.155 | 0.755 | 1.91 | 0.358 | 0.286 | 0.313 | 0.040 / 0.159 / 0.751 | 0.074 |
| `stride4` | nine | 0.200 / 0.105 / 0.545 | 0.209 / 0.122 | 0.456 / 0.590 / 0.048 | 0.784 | 2.25 | 0.337 | 0.327 | 0.294 | 0.030 / 0.133 / 0.686 | 0.103 |
| `c3_s4` | eight, not the centre | 0.200 / 0.106 / 0.542 | 0.206 / 0.116 | 0.447 / 0.580 / 0.055 | 0.785 | 2.29 | 0.360 | 0.301 | 0.317 | 0.035 / 0.154 / 0.783 | 0.092 |

N is 3,000,000 for all five. The far share of primitives against the stride-1 file is 0.265 for the fourth model and 0.273 and 0.274 for `A_default_s0` and `A_default_s1`. Against the file with three times the points the fourth model reads 0.216 (near side 0.096) and `A_default_s0` 0.231 (0.110). Split at frame 80, the fourth model reads like the other default checkpoints (its row in check (b)'s table is not given there; `<surface-runs>/ticket-18/split80.json`, label `nine:default`): far-5 cm share 0.272 / 0.202 over the whole model and 0.120 / 0.109 on the near side, under two frames 0.457 / 0.615 and 0.608 / 0.717, before / after.

**Its sheet**, `<surface-runs>/ticket-18/sheets/side_nine-default.jpg`, from section 2.6's fixed request `<surface-runs>/ticket-5/side_request.json` (its SHA-256 checked as `1b88fd85…98821` before the render), laid out as section 2.6's sheets: 13 poses by 3 frames, every pose but `sC00` a novel view, as for the other nine-camera models. What could be checked without looking (`side_nine-default.coverage.json` beside it, and `<surface-runs>/ticket-18/diff_*.json`):

- Every rendered value is finite. A tile's share of pixels with alpha ≥ 0.5 is 0.98 to 1.00 in the poses between cameras and in `sC00`, and 0.54 to 0.89 at yaw ±60° and ±90°, as for `A_default_s0` (0.60 to 0.93); nothing like `stride4`'s fully covered, nearly black tiles at yaw +60° (section 2.6).
- `A_default_s0` was rendered again from the same request: its coverage figures equal ticket #5's to the last digit. `A_default_s1`'s difference from it matches section 2.3 here, which measures the same seed pair from the same request in 8-bit levels on the pixels both renders cover: its 5.0 levels on `sC00` and 5.3 between cameras are 0.020 and 0.021, as below. Then, pose by pose, the mean absolute difference of the clamped renders from `A_default_s0`'s, for the fourth model and for `A_default_s1`, the seed yardstick:

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

- **The fourth model's init points file.** Made again at the owner's answer, it gives 0.23559 against the score file's 0.23426 and was not accepted (above), so the far-5 cm share against the model's own init stays unknown; it needs the original file. Everything else about the model is measured.
- **The far share's ruler.** Check (a) reads it as a criterion in S and as the method's habit in metres up to 10 cm; the share beyond 25 cm separates the scenes in both. Which form a later test uses is for step 1's review (section 2.6 here).
- **The sheet** has not been seen by anyone yet.

**The commands.**

Each ran with `CUDA_VISIBLE_DEVICES=` empty unless it is marked GPU, `OMP_NUM_THREADS=5 MKL_NUM_THREADS=5 OPENBLAS_NUM_THREADS=5`, `PYTHONPATH=<worktree>/src:<worktree>/harness/src` and the shared `.venv`'s Python; the scripts under `<scratch>/ticket-18/` are not committed. `$T` is `<scratch>/ticket-18`, `$O` is `<surface-runs>/ticket-18`, `$X` is `<private-scene-A-exp>`, `$G` is `$OPENFTGS_DATASETS/synth/ground_truth`.

| Command | Where | Time | Note |
| --- | --- | --- | --- |
| `python $T/probe_init.py <fourth-model> $X/init*.safetensors --target 0.23425932313261924 --out $O/default9_init_probe.json` | CPU | 4 s | The fourth model's far-5 cm share against each of the 15 kept init points files; none gives the score file's. `<fourth-model>` is the checkpoint behind the link |
| `python $T/split80.py $X <private-scene-A>/scene.json <surface-runs>/ticket-5/shell_measures.json --out $O/split80.json --extra nine:default=<fourth-model>=<stride-1 file> --against <file>`, with `--against` for the two stride-1 files, the nine-camera stride-4 file and the stride-1 file with 3,000 points per frame | CPU | 4 min 12 s | Check (b) on the 45 checkpoints and the fourth model; each checkpoint's own init from ticket #5's file; the 45 whole-model rows equal ticket #5's |
| `ftgs init roma <private-scene-A>/scene.json $O/init9_s1_regenerated.safetensors --stride 1`, with `PYTHONPATH` adding `<scratch>/ticket-18/env-init` (torchvision, einops and loguru, row 849), under the shared GPU lock | GPU | 6 min 25 s | 120,000 points; the shared `.venv` lacks the `[init]` extra, and a first try without the folder stopped at once on `loguru` |
| `python $T/probe_init.py <fourth-model> $O/init9_s1_regenerated.safetensors --target 0.23425932313261924 --out $O/default9_init_probe_regenerated.json` | CPU | 4 s | 0.23559: not the score file's figure |
| `python $T/bins.py $X <private-scene-A>/scene.json $O/split80.json --out $O/bins.json <label>=<checkpoint> …`, six checkpoints | CPU | 34 s | Check (b)'s control by 20-frame bins |
| `python $T/windowed_split.py $X <private-scene-A>/scene.json $O/split80.json --out $O/windowed_split.json <label>=<checkpoint> …`, three checkpoints | CPU | 21 s | The far share per half and per bin against the init points within 6 frames; private scene A's far share at 10 and 25 cm; the init points' spacing |
| `python $T/sweep_a.py $X <private-scene-A>/scene.json $O/split80.json --out $O/sweep_a.json <label>=<checkpoint> …`, three checkpoints | CPU | about 20 s | The far share at 0.02·S to 0.25·S on private scene A |
| `python $T/synth_measures.py <m17-runs>/<scene>/default-395/checkpoint <m17-runs>/<scene>/init.safetensors $G/<scene> --scene-name <scene> --out $O/synth_<scene>.json`, for the three scenes | CPU | 60 s, 60 s, 32 s | Check (a): 6,000, 5,400 and 1,517 mask files read. Two earlier runs without the other thresholds' columns gave the same figures |
| The same for `kitchen`'s `default`, `seed1` and `default-326`, `courtyard`'s `default` and `studio`'s `default`, `default-325` and `default-341`, `--out $O/synth_<scene>-<run>.json` | CPU | 56 to 101 s each | The spread between trainings |
| `ftgs render <checkpoint> --request <surface-runs>/ticket-5/side_request.json --out $O/renders/<name>.safetensors`, for the fourth model, `A_default_s0` and `A_default_s1`, from `$T/run_gpu.sh` under the shared GPU lock | GPU | 6.9, 6.5 and 6.7 s | 39 renders each, between 18:07:41 and 18:08:01; the card's memory peaked at 6,741 MiB, 1,202 MiB of it in use before; the renders files were deleted once the sheet and the differences were made |
| `python $T/side_sheet.py $O/renders/nine-default.safetensors <surface-runs>/ticket-5/side_request.json $O/sheets/side_nine-default.jpg --label … --stats $O/sheets/side_nine-default.coverage.json` | CPU | about 1 s | The sheet, by ticket #5's script copied unchanged, and its coverage numbers |
| `python $T/pairdiff.py $O/renders/<a>.safetensors $O/renders/A_default_s0.safetensors <surface-runs>/ticket-5/side_request.json --out $O/diff_<a>_vs_A_default_s0.json`, for the fourth model and `A_default_s1` | CPU | about 2 s each | The differences of the renders, per tile |
| `python $T/tables_a.py $O`, `python $T/tables_b2.py $O/split80.json` | CPU | under 1 s | The rows of this section's tables, from the files above |

The private scene's folders were read and never written.

### 2.2 Check (c): one camera fit per held-out copy, on the background

*Ticket #19's section.* One correction per camera, fitted on the background once per held-out copy without that copy's test-view images, then sections 2.5's and 2.7's measures again on held-out frames, stated in section 5.3's two cases.

**What was measured, and with what.** One correction per camera of private scene A, fitted on the background's matches, then sections 2.5's and 2.7's measures again with the corrected cameras, on 2026-11-16, by uncommitted scratch scripts under `<scratch>/ticket-19` ("Runs", at the end of this section, lists every command), with the worktree's package (`src/` as on `main` at 02e3686), Python 3.14.10, torch 2.15.0+cu132, NumPy 2.5.4 and SciPy 1.18.1, within four CPU threads, and the RTX 4090 for six minutes of matching under the shared lock; the dense matcher's packages are PRD #1's scratch folder of source-log row 612. Results are in `<surface-runs>/ticket-19`. The scripts read private scene A's matches and manifests and, for the matching alone, its images, and wrote numbers and files; no image was looked at. Camera names are the manifests', as in section 2.5: `cam0` is the centre camera and `cam1` the one left of it.

**The two copies, and the pairs the fit reads.** The *centre copy* holds `cam0` out and the *left copy* `cam1`: each is private scene A's manifest with that camera's 120 images in the test split, the same nine cameras bit for bit and the same images file for file (rows 850 and 851). Ticket #4's saved matches of D-22's seven pairs therefore serve both copies (row 852). D-22's pairing, run by the package on each manifest:

| Manifest | D-22's pairs |
| --- | --- |
| private scene A, nine training cameras | `cam0`–`cam1`, `cam1`–`cam4`, `cam2`–`cam5`, `cam3`–`cam4`, `cam3`–`cam5`, `cam6`–`cam7`, `cam6`–`cam8` |
| the centre copy | `cam1`–`cam4`, `cam2`–`cam5`, `cam3`–`cam4`, `cam3`–`cam5`, `cam6`–`cam7`, `cam6`–`cam8` |
| the left copy | `cam0`–`cam2`, `cam2`–`cam5`, `cam3`–`cam4`, `cam3`–`cam5`, `cam6`–`cam7`, `cam6`–`cam8` |

- **The rig's angles tie.** `cam0` is 15.0° from `cam1`, `cam2`, `cam3` and `cam6`; `cam1` from `cam0`, `cam4` and `cam7`; `cam2` from `cam0`, `cam5` and `cam8` (row 610). D-22 picks among them by the float32 values' last bits, so a turn of a tenth of a degree picks again ("The corrected manifests", below). A first pass fitted on D-22's own pairs only, and on the left copy the corrected manifest paired `cam0` with `cam3`, a pair that fit had not read.
- **So the fit reads D-22's pairs with their ties kept:** each training camera with every camera within 0.01° of its closest. On the centre copy 8 pairs (`cam1`–`cam4`, `cam1`–`cam7`, `cam2`–`cam5`, `cam2`–`cam8`, `cam3`–`cam4`, `cam3`–`cam5`, `cam6`–`cam7`, `cam6`–`cam8`), on the left copy 9 (`cam0` with `cam2`, `cam3` and `cam6`; `cam2`–`cam5`, `cam2`–`cam8`, `cam3`–`cam4`, `cam3`–`cam5`, `cam6`–`cam7`, `cam6`–`cam8`), for the nine as one set all 12. Five of them were not ticket #4's and were matched for this section, all 120 frames each: `cam0`–`cam2`, `cam0`–`cam3`, `cam0`–`cam6`, `cam1`–`cam7` and `cam2`–`cam8`, by ticket #12's `match_pair.py` with the package's matcher and settings (D-58, Q-106; row 851).

**The fit.**

- **What it changes.** Each camera is turned about its own centre by three numbers, R' = exp([w]×) R with w in the camera's frame. Centres and K are kept; a variant that also moves the centres is given apart ("The variant that moves the centres", below).
- **What it reads:** the background alone, the far matches of section 2.5's split by depth (the rays' closest approach 2.2 or more along camera A's axis, with the manifest's cameras), on the even frames.
- **Two terms, both in pixels.**
  - Each pair's Sampson error (section 2.5), on 1,500 far matches per pair and frame.
  - For each camera with two partners, section 2.7's agreement in depth: its grid points admitted in both pairs, each triangulated with each partner (closest approach), and the two inverse depths along its axis compared, times f·b (789.2 px × 0.274 units). That is how far, in pixels along the epipolar line in the second partner's image, the match lies from where the first partner's depth puts it: the direction section 2.5's residual does not read. 500 points per camera, pair of partners and frame.
  - Both through Cauchy's loss at a scale of 0.5 px, the one section 2.5's turn used, written into the residuals so that the two terms below stay quadratic.
- **The rig's common turn is held at zero.** Turning every camera by one rotation, centres kept, leaves the matches of a distant point almost as consistent as before, so the background barely reads it (row 853). In each fit of several cameras the three smallest singular values of the Jacobian, 34 to 121 px per degree, lie well below the others, 181 to 7,300; the smallest is the common turn about the optical axes, and without the hold the fit drifts along the common turn about the vertical axis (below). Only the smallest one's direction was kept, so that the other two are the common turn's other components is inferred, not read. The mean turn of the cameras fitted together is held at zero (1,000 px per degree), which keeps the manifest's orientation of the rig as a whole. Without that hold, on the centre copy's D-22 pairs, every camera turned 2.2° to 5.0°, mostly about its vertical axis, for about the same background residual on the fit's own matches (0.28 to 0.55 px at the median by pair, against 0.24 to 0.59 px with the hold). A damping of 1 px per degree on each component keeps any other direction the matches leave free at the manifest's value.
- **Solved** by SciPy's `least_squares` (trust region) from no turn, with the Jacobian by differences, block by block; every fit stopped on its cost or step tolerance, in 26 to 65 evaluations and 12 to 130 s.
- **Per copy, two steps** (section 5.3, D-66, and the owner's answer at review 2, section 7.7):
  1. the training cameras' turns, from the pairs between training cameras only, so that no image of the test view is read;
  2. the held-out camera's turn, with the training cameras fixed at theirs, from its own images' matches with every camera at its smallest angle: `cam0` with `cam1`, `cam2`, `cam3` and `cam6`, and its six pairs of partners; `cam1` with `cam0`, `cam4` and `cam7`, and its three.
- **The nine as one set:** one step over all 12 pairs and 22 pairs of partners; it never trains.
- **Measured on the odd frames,** which no fit read, with every match's region (person or background) fixed by the manifest's cameras before and after, and every figure after a correction read from the corrected manifest as written and read back by the package's reader.

**Checks of the fit.**

- **The measures reproduce the research document's.** Over all 120 frames with the manifest's cameras, the scripts give section 2.5's table by depth and section 2.7's table of one camera with two partners, and its fixed relation, to the digits printed there: `cam1`–`cam4` 5.23 and 9.62 px on the person, 2.24 and 3.44 px on the background; `cam3` 6.45% and 8.47%, a = 1.077, b = −0.0045. On the odd frames alone the figures barely differ (the five cameras' person 3.99% and 6.10%, against 4.02% and 6.07%), so the odd frames' "before" columns below stand for section 2.5's and 2.7's.
- **A turn put in is a turn taken out.** `cam4` was turned by (0.2°, −0.3°, 0.1°) in a manifest written by the same script, and fitted again on its two pairs and two partners: the camera so corrected lies 0.0035° from the one fitted on the manifest itself.
- **The file is the fit.** Read back, each corrected camera's rotation differs from the fit's by 3·10⁻⁸ at most, float32's rounding.
- **What the odd frames hold out.** The cameras and the background do not change from frame to frame, so the odd frames test the fit against new matches of the same scene, not against a new scene. The stronger hold-out is the test view's own pairs, which the first step never reads ("The held-out camera", below).

**The corrections,** in degrees: the turn's angle, then its components about the camera's x, y and z axes (x to the right, y down, z along the optical axis).

| Camera | The centre copy | The left copy | The nine as one set |
| --- | --- | --- | --- |
| `cam0` | 0.10° (−0.04, +0.03, +0.09) | 0.05° (−0.05, +0.00, +0.00) | 0.07° (−0.05, +0.04, +0.03) |
| `cam1` | 0.24° (−0.06, −0.14, +0.18) | 0.23° (−0.05, −0.22, −0.01) | 0.21° (−0.05, −0.20, +0.03) |
| `cam2` | 0.30° (−0.06, +0.18, −0.23) | 0.23° (−0.04, +0.22, −0.06) | 0.26° (−0.02, +0.25, −0.05) |
| `cam3` | 0.20° (−0.07, +0.06, −0.17) | 0.09° (+0.04, −0.04, −0.07) | 0.08° (+0.04, −0.00, −0.06) |
| `cam4` | 0.38° (+0.09, −0.37, +0.04) | 0.38° (+0.18, −0.33, −0.05) | 0.45° (+0.17, −0.40, −0.11) |
| `cam5` | 0.37° (−0.09, +0.21, −0.29) | 0.25° (+0.04, +0.22, −0.12) | 0.27° (+0.05, +0.24, −0.10) |
| `cam6` | 0.25° (+0.09, −0.01, +0.23) | 0.10° (−0.08, −0.03, +0.05) | 0.07° (−0.06, +0.02, +0.05) |
| `cam7` | 0.38° (+0.12, +0.08, +0.35) | 0.26° (+0.01, −0.09, +0.24) | 0.22° (+0.03, −0.02, +0.22) |
| `cam8` | 0.11° (−0.01, +0.03, −0.11) | 0.15° (−0.10, +0.11, +0.01) | 0.16° (−0.08, +0.14, −0.00) |

- **Their size** is section 2.5's: 0.05° to 0.45°, where its turn of one camera per pair was 0.10° to 0.41°.
- **The copies agree on most cameras:** `cam4` turns 0.38° to 0.45°, about −0.33° to −0.40° on its y axis, in all three fits; `cam2` and `cam5` +0.18° to +0.25° on theirs. They differ most where a copy reads a camera least: `cam3`, `cam6` and `cam7` on the centre copy, which lacks every pair with `cam0`, turn 0.20° to 0.38° there and 0.07° to 0.26° in the other two fits.

**Epipolar residuals on the odd frames,** with section 2.5's split by depth: the median and the 90th percentile in pixels over every admitted match of the 60 odd frames.

| Pair | The background: before | The centre copy | The left copy | The nine as one set |
| --- | --- | --- | --- | --- |
| `cam0`–`cam1` | 0.29, 0.89 | 0.28, 0.82 | 0.20, 0.87 | 0.20, 0.82 |
| `cam1`–`cam4` | 2.25, 3.45 | 0.51, 1.43 | 1.15, 2.38 | 0.53, 1.46 |
| `cam2`–`cam5` | 0.71, 2.63 | 0.48, 2.20 | 0.49, 2.06 | 0.49, 2.06 |
| `cam3`–`cam4` | 1.30, 3.27 | 0.61, 2.11 | 0.67, 1.79 | 0.68, 2.10 |
| `cam3`–`cam5` | 0.58, 1.62 | 0.52, 1.68 | 0.57, 1.57 | 0.58, 1.55 |
| `cam6`–`cam7` | 0.70, 1.43 | 0.26, 1.06 | 0.26, 1.28 | 0.26, 1.24 |
| `cam6`–`cam8` | 0.58, 2.78 | 0.48, 2.43 | 0.57, 2.25 | 0.59, 2.18 |
| The 7 pairs together | 0.87, 2.85 | 0.43, 1.67 | 0.55, 1.92 | 0.47, 1.68 |
| `cam0`–`cam2` | 0.42, 1.01 | 0.76, 1.59 | 0.37, 0.86 | 0.37, 0.88 |
| `cam0`–`cam3` | 0.61, 1.20 | 0.49, 1.22 | 0.24, 0.62 | 0.24, 0.60 |
| `cam0`–`cam6` | 0.43, 0.93 | 0.63, 1.07 | 0.31, 0.73 | 0.33, 0.74 |
| `cam1`–`cam7` | 1.77, 3.67 | 0.53, 2.00 | 0.61, 2.52 | 0.49, 2.07 |
| `cam2`–`cam8` | 1.25, 2.42 | 0.40, 2.36 | 0.43, 2.33 | 0.44, 2.40 |
| The 5 other pairs together | 0.69, 1.94 | 0.55, 1.41 | 0.33, 1.14 | 0.33, 1.04 |

| Pair | The person: before | The centre copy | The left copy | The nine as one set |
| --- | --- | --- | --- | --- |
| `cam0`–`cam1` | 0.95, 2.67 | 0.99, 2.71 | 0.95, 2.71 | 0.95, 2.71 |
| `cam1`–`cam4` | 5.23, 9.72 | 3.17, 7.35 | 4.17, 8.60 | 3.40, 7.71 |
| `cam2`–`cam5` | 1.10, 3.25 | 1.41, 3.57 | 1.24, 3.62 | 1.23, 3.62 |
| `cam3`–`cam4` | 3.07, 6.92 | 1.98, 6.35 | 2.07, 6.27 | 2.17, 6.31 |
| `cam3`–`cam5` | 3.81, 13.84 | 3.64, 13.28 | 3.71, 13.72 | 3.71, 13.74 |
| `cam6`–`cam7` | 3.35, 6.47 | 3.71, 6.72 | 4.37, 7.51 | 4.34, 7.46 |
| `cam6`–`cam8` | 1.88, 8.74 | 2.71, 10.06 | 2.02, 9.08 | 2.06, 9.17 |
| The 7 pairs together | 2.37, 7.64 | 2.29, 7.04 | 2.35, 7.50 | 2.30, 7.30 |
| `cam0`–`cam2` | 1.01, 3.20 | 1.24, 3.53 | 0.92, 2.97 | 0.89, 2.92 |
| `cam0`–`cam3` | 1.48, 3.20 | 2.09, 3.66 | 1.25, 2.80 | 1.27, 2.78 |
| `cam0`–`cam6` | 1.96, 3.92 | 1.45, 3.58 | 1.64, 3.62 | 1.74, 3.72 |
| `cam1`–`cam7` | 3.55, 6.35 | 1.86, 4.54 | 2.47, 5.22 | 2.10, 4.81 |
| `cam2`–`cam8` | 7.93, 14.71 | 6.25, 13.16 | 6.64, 13.60 | 6.63, 13.57 |
| The 5 other pairs together | 2.31, 7.99 | 2.12, 6.37 | 1.88, 6.73 | 1.84, 6.68 |

- **The background falls, not in every pair.** Over the seven pairs its median goes from 0.87 to 0.43 px on the centre copy, 0.55 px on the left copy and 0.47 px with the nine as one set, and its 90th percentile from 2.85 to 1.67 to 1.92 px. It falls most where it was most off: `cam1`–`cam4` from 2.25 to 0.51 px (1.15 px on the left copy, which reads `cam1` only in the second step), `cam3`–`cam4` from 1.30 to 0.61 to 0.68 px, `cam1`–`cam7` from 1.77 to 0.49 to 0.61 px, `cam2`–`cam8` from 1.25 to 0.40 to 0.44 px. It stays in `cam3`–`cam5` (0.58 to 0.52 to 0.58 px) and `cam6`–`cam8` (0.58 to 0.48 to 0.59 px), and on the centre copy it rises in `cam0`'s pairs with `cam2` and `cam6` (0.42 to 0.76 px, 0.43 to 0.63 px). Section 2.5's turn of one camera for one pair reached 0.28 to 0.60 px in every pair; one turn per camera for every pair at once reaches 0.20 to 0.76 px.
- **The person's does not follow.** Over the seven pairs its median is 2.37 px before and 2.29, 2.35 and 2.30 px after, its 90th percentile 7.64 and 7.04 to 7.50 px. By pair it moves as section 2.5 found for its turn: lower where the background was most off (`cam1`–`cam4` from 5.23 to 3.17 to 4.17 px, `cam3`–`cam4` from 3.07 to 1.98 to 2.17 px, `cam1`–`cam7` from 3.55 to 1.86 to 2.47 px) and higher in `cam2`–`cam5`, `cam6`–`cam7` and `cam6`–`cam8` (`cam6`–`cam7` from 3.35 to 3.71 to 4.37 px).
- **D-22's filters** (2 px in both views, in front of both, 1° between the rays), applied to a uniform sample of 2,000 admitted matches per pair, frame and region with each set of cameras. This is not the package's draw by certainty inside the crop, so it is not section 2.5's column. On the background they keep more where the residual fell (`cam1`–`cam4` from 69% to 93% to 97%, `cam3`–`cam4` from 74% to 92% to 94%). On the person, more in `cam1`–`cam4` (28% to 35% to 43%), `cam3`–`cam4` (47% to 61% to 64%) and `cam1`–`cam7` (34% to 53% to 64%), and fewer in `cam6`–`cam7` (45% to 25% to 37%) and `cam2`–`cam5` (87% to 80% to 83%).

**The held-out camera.** Its pairs are the ones no training camera's fit read: the background's median on the odd frames, at three stages.

| Copy | Pair | The manifest | The training cameras corrected, the held-out camera as in the manifest | The held-out camera corrected too | The nine as one set |
| --- | --- | --- | --- | --- | --- |
| the centre copy, `cam0` | `cam0`–`cam1` | 0.29, 0.89 | 0.92, 1.40 | 0.28, 0.82 | 0.20, 0.82 |
| the centre copy, `cam0` | `cam0`–`cam2` | 0.42, 1.01 | 0.70, 1.63 | 0.76, 1.59 | 0.37, 0.88 |
| the centre copy, `cam0` | `cam0`–`cam3` | 0.61, 1.20 | 0.79, 1.41 | 0.49, 1.22 | 0.24, 0.60 |
| the centre copy, `cam0` | `cam0`–`cam6` | 0.43, 0.93 | 0.44, 1.34 | 0.63, 1.07 | 0.33, 0.74 |
| the left copy, `cam1` | `cam0`–`cam1` | 0.29, 0.89 | 0.57, 1.21 | 0.20, 0.87 | 0.20, 0.82 |
| the left copy, `cam1` | `cam1`–`cam4` | 2.25, 3.45 | 1.30, 2.31 | 1.15, 2.38 | 0.53, 1.46 |
| the left copy, `cam1` | `cam1`–`cam7` | 1.77, 3.67 | 2.86, 4.82 | 0.61, 2.52 | 0.49, 2.07 |

- **The manifest's pose of the test view no longer agrees with the corrected training cameras.** With them, `cam0`'s four pairs read 0.44 to 0.92 px at the median, against 0.29 to 0.61 px with the manifest's cameras, and `cam1`'s three 0.57 to 2.86 px, against 0.29 to 2.25 px. A model trained on a corrected manifest and scored at the manifest's pose of the test view is scored at a pose that disagrees with its training cameras by that much, which is why section 5.3 scores both poses.
- **The held-out camera's own correction** brings `cam1`'s pairs to 0.20 to 1.15 px and `cam0`'s to 0.28 to 0.76 px. On the centre copy, `cam0`'s pairs with `cam1` and `cam3` end below the manifest's and those with `cam2` and `cam6` above: the training cameras' turns, fitted without `cam0`, leave no pose of `cam0` that agrees with its four partners as well as the manifest's did. The nine fitted as one set reach 0.20 to 0.37 px on `cam0`'s pairs and 0.20 to 0.53 px on `cam1`'s. That is what D-66's fit per copy costs here, against the nine fitted as one set: up to 0.4 px on the held-out camera's pairs, and 0.6 px on `cam1`–`cam4` on the left copy (1.15 against 0.53 px).

**Agreement in depth between the pairs that share a camera,** as section 2.7's "One camera, two partners": per camera and pair of partners, the median over the odd frames of each frame's median of |z₂/z₁ − 1|, and of its 90th percentile, then the signed median; the person by the first partner's depth (with the manifest's cameras) and rule a's crop, the background at 2.2 or more. The five cameras' row pools their 300 frames, as section 2.7's row of 600.

| Camera | Its two partners | The person: before | The centre copy | The left copy | The nine as one set |
| --- | --- | --- | --- | --- | --- |
| `cam1` | `cam0`, `cam4` | 1.37%, 2.83%; −0.87% | 1.48%, 3.02%; −0.82% | 1.39%, 2.79%; −1.15% | 1.46%, 2.84%; −0.95% |
| `cam3` | `cam4`, `cam5` | 6.45%, 8.47%; −6.45% | 4.38%, 6.44%; −4.37% | 6.24%, 8.08%; −6.24% | 5.47%, 7.38%; −5.47% |
| `cam4` | `cam1`, `cam3` | 3.90%, 5.67%; +3.90% | 1.67%, 3.38%; +1.53% | 3.51%, 5.19%; +3.51% | 2.54%, 4.24%; +2.53% |
| `cam5` | `cam2`, `cam3` | 1.58%, 4.10%; +0.41% | 1.69%, 4.67%; −1.09% | 1.58%, 4.36%; −0.86% | 1.57%, 4.33%; −0.84% |
| `cam6` | `cam7`, `cam8` | 5.85%, 7.57%; −5.85% | 6.87%, 8.51%; −6.87% | 6.15%, 7.91%; −6.15% | 6.30%, 8.04%; −6.30% |
| The five cameras | – | 3.99%, 6.10%; 4.1 cm | 2.96%, 5.34%; 3.4 cm | 3.80%, 6.01%; 4.1 cm | 3.19%, 5.51%; 3.4 cm |

| Camera | Its two partners | The background: before | The centre copy | The left copy | The nine as one set |
| --- | --- | --- | --- | --- | --- |
| `cam1` | `cam0`, `cam4` | 1.71%, 5.81%; −0.39% | 1.32%, 3.78%; −0.38% | 1.57%, 5.20%; −0.59% | 1.37%, 4.01%; −0.28% |
| `cam3` | `cam4`, `cam5` | 6.25%, 13.90%; −3.74% | 5.99%, 12.21%; +2.87% | 5.02%, 10.90%; −2.11% | 5.93%, 10.31%; +1.99% |
| `cam4` | `cam1`, `cam3` | 5.34%, 12.80%; +5.34% | 2.05%, 6.79%; +0.24% | 3.85%, 10.20%; +3.82% | 1.85%, 5.69%; +0.50% |
| `cam5` | `cam2`, `cam3` | 4.98%, 8.53%; +4.96% | 1.58%, 4.91%; −0.23% | 1.50%, 4.73%; −0.31% | 1.51%, 4.84%; −0.42% |
| `cam6` | `cam7`, `cam8` | 3.14%, 5.57%; +2.72% | 1.95%, 5.74%; +0.51% | 1.55%, 4.13%; +0.16% | 1.54%, 4.18%; −0.07% |
| The five cameras | – | 4.59%, 8.83%; 19.2 cm | 1.92%, 5.60%; 8.0 cm | 2.45%, 6.21%; 8.8 cm | 1.76%, 5.12%; 6.9 cm |

| Camera | Its two partners | Scale a: before; the centre copy, the left copy, the nine | Shift b (per unit of depth): before; after | The person, what the relation leaves: before; after |
| --- | --- | --- | --- | --- |
| `cam1` | `cam0`, `cam4` | 1.009; 1.006, 1.011, 1.008 | −0.0021; −0.0023, −0.0021, −0.0032 | 1.58%; 1.64%, 1.55%, 1.62% |
| `cam3` | `cam4`, `cam5` | 1.077; 1.076, 1.079, 1.082 | −0.0042; −0.0221, −0.0095, −0.0192 | 1.32%; 1.31%, 1.27%, 1.28% |
| `cam4` | `cam1`, `cam3` | 0.971; 0.977, 0.971, 0.968 | −0.0073; +0.0069, −0.0032, +0.0069 | 1.01%; 1.06%, 1.01%, 0.99% |
| `cam5` | `cam2`, `cam3` | 1.010; 1.016, 1.013, 1.013 | −0.0097; −0.0012, −0.0004, 0.0000 | 1.66%; 1.75%, 1.64%, 1.63% |
| `cam6` | `cam7`, `cam8` | 1.099; 1.107, 1.092, 1.094 | −0.0366; −0.0335, −0.0277, −0.0276 | 1.51%; 1.48%, 1.60%, 1.59% |

- **The person's 4.0% falls to 3.0% to 3.8%, not towards 1.0% to 1.6%.** Over the five cameras, 3.99% before (4.1 cm), 2.96% on the centre copy (3.4 cm), 3.80% on the left copy (4.1 cm) and 3.19% with the nine as one set. By camera: `cam4` from 3.90% to 1.67% to 3.51%, `cam3` from 6.45% to 4.38% to 6.24%; `cam1` and `cam5` about the same; `cam6` from 5.85% to 6.15% to 6.87%, higher. How much falls depends on which camera the copy leaves out.
- **The background's falls by half.** Over the five cameras, 4.59% before (19.2 cm), 1.92%, 2.45% and 1.76% after; by camera, `cam4`, `cam5` and `cam6` to 1.5% to 3.9%, and `cam3` hardly (6.25% to 5.0% to 6.0%).
- **Why the person's does not follow: the turn reaches b and not a.** Section 2.7 found one fixed relation per camera, 1/z₂ = a/z₁ + b, that takes the person's 4.0% to 1.0% to 1.6%. Fitted again over the odd frames, its scale a does not move with any fit (`cam3` 1.077, then 1.076 to 1.082; `cam6` 1.099, then 1.092 to 1.107; `cam4` 0.971, then 0.968 to 0.977), its shift b does, and it leaves 1.0% to 1.8% on the person after every fit as before. A turn along the epipolar lines adds one amount to every inverse depth, which is b; a scale of every depth, a, needs a baseline or a focal length that is not the manifest's (section 2.7). The turns took out part of b, and with it half of the background's figure; on the person the scale part, 8% to 10% for `cam3` and `cam6`, is what stays.
- **Pairs of partners beyond section 2.7's five** tell the same. Of the 17 more that the five other pairs make, the person's figure falls by more than half a point on both copies in 6 (`cam3` with `cam0` and `cam4` from 5.48% to 1.78% and 3.88%; `cam1` with `cam0` and `cam7` from 5.55% to 3.30% and 3.71%), rises as much in 4 (`cam0` with `cam2` and `cam3` from 2.27% to 3.96% and 3.03%) and stays in 7; the background's falls in 13 and rises in none.

**The variant that moves the centres.** A turn reaches section 2.7's shift b and not its scale a, which needs a baseline or a focal length. So the same fit was run again with each camera's centre free as well, three more numbers damped at 1 px per cm, the rig's common shift and its size (the mean distance of the centres from their centroid) held as its common turn is, on 600 and 250 points per pair or camera and frame. A first try that held the shift and not the size grew the rig 2.4 times, which shrinks every inverse depth and the depth term with it, and was stopped. The variant is evidence on what a centre move can reach, not a manifest proposed for training: it moves S.

- **The centres move** up to 2.4 cm (`cam4` and `cam8` on both copies; the held-out cameras 0.2 cm and 1.5 cm), and the turns stay 0.08° to 0.42°.
- **The background falls a little further:** 0.38 and 0.40 px over the seven pairs at the median, 1.33 and 1.40 px at the 90th percentile, against 0.43 and 0.55 px with turns alone.
- **The person's residual rises:** 3.23 and 5.65 px over the seven pairs at the median, 12.0 and 14.3 px at the 90th percentile, against 2.37 and 7.64 px before. A centre moved to suit the background is wrong for the person, whose matches it moves more since they are nearer (section 2.5, "What the turn does and does not show").
- **The person's 4.0% between pairs** is 3.58% and 3.09%. The scale a now moves, but not towards 1 in every camera: `cam3`'s from 1.077 to 0.967 and 1.005, so its person falls to 3.71% and 1.77%, while `cam4`'s goes from 0.971 to 1.109 and 1.089, and its person rises to 7.42% and 5.64%.
- **D-22's pairs flip on these too:** `cam1`–`cam7` on the centre copy, `cam0`–`cam6` on the left copy.

So this centre move, fitted on the background, does not reach the person's offset either, and it costs the person's epipolar residual and S. How well the background fixes the centres was not measured, and shifts of 2 cm from the background alone may be loose; a focal length per camera, the other cause section 2.7 names for a, was not tried.

**Centres and S (D-27).** The turns move no centre: read back from the corrected manifests, the largest change of a centre is 2.6·10⁻⁸ units, float32's rounding, and S by the package's `spatial_scale` (factor 1) is 0.378230496 before and 0.378230499 after on the centre copy, 0.402766772 and 0.402766771 on the left copy. E13's position rates and growth's split threshold of 0.01·S are unchanged. The variant that moves the centres changes S by +1.2% on the centre copy (0.378230 to 0.382828) and +0.37% on the left copy (0.402767 to 0.404241), and E13's rates and the split threshold with it; it stays in `<surface-runs>/ticket-19/manifests/V2`, not proposed for training.

**The corrected manifests.** In `<surface-runs>/ticket-19/manifests/P/h0` (the centre copy) and `P/h1` (the left copy), each `scene.json` beside a link `images` to its copy's images, not committed:

- **What they are:** the copy's manifest unchanged, records, splits, paths and metadata, except every camera's `world_to_camera`, turned as above, and a `metadata.camera_correction` that records the tool (this section's scratch fit, a prototype), the copy's SHA-256, both steps' settings, and per camera its turn, its centre's shift (none) and its `world_to_camera` before. The test view's two poses, the manifest's and the corrected one, are both in the file (section 5.3's measures row).
- **Valid `ftgs-scene` v1 files:** the package's `read_scene_manifest` and `MultiViewVideo.from_manifest` read each without error, 1,080 records, 120 of them the test view's, and load one training and one test image through the link.
- **D-22's pairs change on both,** since the turns break the 15.0° ties (above): on the centre copy `cam1`–`cam4` and `cam2`–`cam5` become `cam1`–`cam7` and `cam2`–`cam8`; on the left copy `cam0`–`cam2` and `cam2`–`cam5` become `cam0`–`cam6` and `cam2`–`cam8`. The fit read all four new pairs. But an init on a corrected manifest would take `cam2`'s person from `cam2`–`cam8`, where D-22's filters keep 0.8% to 2.1% of the person's sample against 80% to 83% in `cam2`–`cam5`, and where the person disagrees by 6.3 to 7.9 px at the median before and after; `cam1`–`cam7` keeps more of it than `cam1`–`cam4` (64% against 43% on the centre copy) and `cam0`–`cam6` a little less than `cam0`–`cam2` (82% against 89% on the left copy). So a run on a corrected manifest would differ from the baseline's in its init's pairs as well as in its cameras; whether finalist 1's inits keep the manifest's pairs is put to the owner (step 1's review, section 2.6 here).
- **The test view at the manifest's pose beside the corrected training cameras** is `P/h0-train-only` and `P/h1-train-only`: each copy's manifest with its first step's turns alone, the files that score the test view's first pose ("The held-out camera", above).
- **The nine as one set** has its manifest in `P/all9-never-trains`, for these measures only. The first pass's manifests, on D-22's own pairs, are set aside in `first-pass-not-for-training`.

**The outcome: section 5.3's second case.** The background's residual falls, over the seven pairs from 0.87 to 0.43 to 0.55 px, though not in every pair; the person's epipolar residual does not (2.37 to 2.29 to 2.35 px), and its 4.0% between pairs falls to 3.0% on one copy and 3.8% on the other, not towards the 1.0% to 1.6% that section 2.7's fitted relation leaves, which the turns leave in place. So finalist 1, with turns fitted on the background, tests the background's part of the disagreement alone, and little change is expected on the person (section 5.3, "The route, and the first step"). The variant that also moves the centres gives the same case, with the person's epipolar residual higher and S moved. The nine as one set does not reconcile the pairs either: it gives about the copies' figures (0.47 px and 2.30 px over the seven pairs, 3.19% on the person's depth), with `cam3`'s background still 5.9% and `cam6`'s person 6.3%.

**Runs.** `python` is the shared `.venv`'s, with `PYTHONPATH=<worktree>/src:<worktree>/harness/src` and `OMP_NUM_THREADS`, `MKL_NUM_THREADS` and `OPENBLAS_NUM_THREADS` at the thread count given, and `CUDA_VISIBLE_DEVICES` empty except for the matcher. The matcher's runs add `<worktree>/scripts` and PRD #1's `.scratch/surface-following-primitives/ticket-4/env-init` to `PYTHONPATH`, with `PYTHONDONTWRITEBYTECODE=1`, and hold the shared GPU lock. `$S` is `<scratch>/ticket-19`, `$O` is `<surface-runs>/ticket-19`, `$O4` is `<surface-runs>/ticket-4/private-scene-A`, `$G` is the grid folders `$O4/d22` and `$O/matches/<pair>` for the five pairs matched here, and `<copy>` is a held-out copy's folder, `<private-scene-A-exp>/scene_h0` or `scene_h1`, where `<private-scene-A-exp>` is the folder of the owner's notes on private scene A that holds the copies, written nowhere, as `<private-scene-A>` is. `run_fits2.sh`, `run_measure.sh` and `run_check.sh` chain the commands below with the pairs and pairs of partners given above.

| Command | Threads; device | What it ran; time | Note |
| --- | --- | --- | --- |
| `python -I $S/same_images.py <private-scene-A>/scene.json <private-scene-A-exp>/scene_h0/scene.json <private-scene-A-exp>/scene_h1/scene.json` | 4; CPU | 3,240 files; 1 s | the copies' images are the scene's |
| `python $S/tie_pairs.py <manifest>`, on the scene and the two copies | 1; CPU | seconds | D-22's pairs with their ties kept |
| `flock <scratch>/gpu.lock python $S/match_pair.py <private-scene-A>/scene.json $O/matches --label private-scene-A --tag cam0-cam2 --pair cam0:cam2 --device cuda:0`, and the same for `cam0:cam3`, `cam0:cam6`, `cam1:cam7` and `cam2:cam8` | 4; GPU | 120 evaluations each; 66 to 71 s each | ticket #12's script, unchanged: the five pairs' matches, in ticket #4's grid format |
| `python $S/measure.py <private-scene-A>/scene.json <private-scene-A>/scene.json $O/measures/manifest-all.json --grids $G --crops $O4/crops.json --frames all --pairs <D-22's 7> --triples <section 2.7's five> --d22-sample 2000` | 4 processes of 1 thread; CPU | 840 pair-frames and 600 images; 2 min 6 s | the reproduction of sections 2.5's and 2.7's tables |
| `python $S/fit.py <copy>/scene.json $O/fits/first-pass/h0-train-component.json --grids $G --pairs <D-22's pairs of the copy> --triples <their pairs of partners> --free <its training cameras>`, then the held-out camera's step, for both copies and the nine; and the centre copy's first step again with `--gauge none` | 4; CPU | 12 to 73 s each | the first pass, set aside: it showed the pairs' flip and the common turn |
| `python $S/fit.py <copy>/scene.json $O/fits/first-pass/h0-train-loss-on-hold.json …` on the centre copy's D-22 pairs, without and then with the hold, before the Cauchy loss was written into the residuals; the second run's output was replaced by the next row's | 4; CPU | 6 min 7 s and 1 min 47 s | set aside: the loss also took the hold and the damping, which then held nothing, and every camera turned 1.6° to 2.4°; a run without the hold under that code was stopped |
| `$S/run_fits2.sh P`: `python $S/fit.py <copy>/scene.json $O/fits/P/h0-train.json --grids $G --pairs <the copy's pairs, ties kept> --triples <their pairs of partners> --free <its training cameras>`, then `python $S/fit.py <copy>/scene.json $O/fits/P/h0.json --grids $G --pairs <the held-out camera's pairs> --triples <its pairs of partners> --free cam0 --fixed $O/fits/P/h0-train.json`; the same for the left copy; `python $S/fit.py <private-scene-A>/scene.json $O/fits/P/all9.json --grids $G --pairs <all 12> --triples <all 22> --free cam0,…,cam8` | 4; CPU | 12 to 130 s each | the corrections |
| `python $S/write_manifest.py <copy>/scene.json $O/fits/P/h0.json $O/manifests/P/h0 --label copy-h0 --train-fit $O/fits/P/h0-train.json --note "…"`; the same for the left copy; for the nine, into `P/all9-never-trains`; and from each copy's first step alone, into `P/h0-train-only` and `P/h1-train-only` | 4; CPU | seconds each | the corrected manifests, read back by the package, with S and D-22's pairs before and after |
| `$S/run_measure.sh <manifest> $O/measures/<name>-odd.json`, which runs `python $S/measure.py <manifest> <private-scene-A>/scene.json … --frames odd --pairs <the 12> --triples <the 22> --d22-sample 2000` | 4 processes of 1 thread; CPU | 720 pair-frames and 1,320 images; 4 min each | the measures before (the manifest) and with each corrected manifest; the scene's manifest fixes the regions |
| `python $S/measure.py $O/manifests/P/h0-train-only/scene.json <private-scene-A>/scene.json … --pairs <cam0's four> --triples <its six>`; the same for the left copy with `cam1`'s | 4 processes of 1 thread; CPU | 1 to 2 min each | the held-out camera at the manifest's pose beside the corrected training cameras |
| `$S/run_check.sh`: `write_manifest.py` with `cam4` turned by (0.2°, −0.3°, 0.1°), `fit.py` on the centre copy and on that manifest with `--free cam4`, then `python $S/compose.py` | 4; CPU | about 1 min | the conventions' check |
| `$S/run_v2.sh`: `run_fits2.sh V2 --centres --per-frame 600 --tri-per-frame 250` for the two copies, `write_manifest.py` into `$O/manifests/V2/h0` and `V2/h1`, and `run_measure.sh` on both | 4; CPU | fits 12 to 186 s; measures 4 min each | the variant that moves the centres; a first try without the rig's size held, stopped after 11 min, is set aside |
| `python $S/doc_tables2.py $O/measures $O/fits` | 1; CPU | seconds | the tables of this section |

The GPU ran for six minutes in all, for the five pairs' matching; everything else ran on the CPU. No output file holds private scene A's location or name: the scene is named by `--label`, and the corrected manifests, which hold the copies' own metadata, stay in `<surface-runs>`.

### 2.3 Checks (d) and (e): two seeds' disagreement where the shell is, and colour between cameras

*Ticket #20's section.* `A_default_s0` against `A_default_s1` where the shell is; and the colour difference at the two ends of section 2.5's admitted matches, per camera pair, against `kitchen`'s floor.

**What was measured, and with what.** On 2026-11-16, by uncommitted scratch scripts under `<scratch>/ticket-20` ("Runs", at the end of this section, lists every command), with the worktree's package at the integration branch's 9bd213c (`src/` as `main`'s 02e3686), Python 3.14.10, torch 2.15.0+cu132, gsplat 1.5.3, NumPy 2.5.4, SciPy 1.18.1, Pillow 13.0.0 and safetensors 0.8.0, within four CPU threads, and the RTX 4090 through the shared lock for the GPU runs. Results are in `<surface-runs>/ticket-20`. The scripts read private scene A's checkpoints, init points, images and renders and wrote numbers; no image, render or sheet was looked at. Camera names are the manifests', as in section 2.5.

**Check (d): two seeds' disagreement where the shell is.**

**The two models.** `A_default_s0` and `A_default_s1` (section 2.6): the default configuration from the stride-1 init points file both trained from, with the centre camera `cam0` as the test view, 30,000 iterations and 3,000,000 primitives each, differing by `seed` alone (row 841).

**The populations,** as section 2.6 defines them, taken in each model by its own primitives:

- the **near side**, positions nearer than 2.0 along `cam0`'s axis, and the **far side**, the others;
- **far from init**: positions more than 0.05 from every point of the init points file, all frames pooled;
- **under two frames**: durations under 2/24 s.

The near side's two populations overlap, so the near side is also split into its four cells, and **the rest** is the near side's primitives in neither population: near an init point and not under two frames. The script first reproduces section 2.6's figures for both checkpoints, to every digit given there: the far-5 cm share of opacity 0.259 and 0.257, the under-two-frames share 0.517 and 0.519, the near side's share of opacity 0.749 and 0.755, the far-5 cm share within it 0.128 and 0.131, and the far-from-init share of its primitives 0.140 and 0.144.

**The measure.** For every primitive of one model, by its position μx:

- **cross**, its distance to the other model's nearest primitive;
- **own**, its distance to its own model's nearest other primitive, the spacing of its neighbourhood;
- their ratio, cross / own, primitive by primitive. Raw centimetres can't be compared between populations, since the primitives far from init are sparse and those near it dense. The ratio can: it is about 1 when the other seed has a primitive about as near as the model's own neighbour, and it grows when the other seed has put none there.

Both directions are taken, `s0` against `s1` and `s1` against `s0`, and they agree closely. A variant restricts both searches to primitives whose time centres lie within 6 frames of each other (the same frame bin or within 6 bins, so 6 to 7 frames), since the person moves about 0.5 over the clip (section 2.6); it gives the same ordering, with larger ratios. Every distance is also given weighted by base opacity, which moves no median by more than 0.3 cm. Relocation's copies do not shrink the ratio's denominator: 0.3 % or fewer of a population's own spacings are under 10⁻⁴ (on `s0`; the far-from-init primitives' 5th percentile is 0.06 cm).

**The rule, fixed before the figures were read.** A population is placed differently by the two seeds when both of these hold in both directions:

- its median ratio exceeds the rest's by more than 25 %;
- its share of primitives with cross under own is lower than the rest's by more than 0.10.

Otherwise it is placed alike. The far side is a second reference.

| Population (in its model) | Share of the model's primitives | Cross: quartiles, cm | Own: median, cm | Ratio cross / own: median (quartiles) | Cross under own | Cross over 5 cm | Ratio, within ±6 frames: median | Distance to the nearest init point: median, cm |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| Near side, far from init | 0.102; 0.106 | 0.65 / 1.45 / 3.05; 0.63 / 1.29 / 2.54 | 0.36; 0.34 | 3.06 (1.41 to 8.25); 3.04 (1.43 to 7.51) | 0.155; 0.147 | 0.111; 0.072 | 6.14; 6.17 | 17.0; 14.8 |
| … of which under two frames | 0.034; 0.037 | 0.45 / 1.01 / 2.36; 0.44 / 0.91 / 1.76 | 0.19; 0.18 | 4.38; 4.26 | 0.124; 0.117 | 0.072; 0.042 | 6.20; 6.17 | 7.8; 7.1 |
| … of which not | 0.068; 0.069 | 0.80 / 1.71 / 3.34; 0.78 / 1.55 / 2.90 | 0.52; 0.49 | 2.67; 2.64 | 0.170; 0.163 | 0.131; 0.089 | 6.11; 6.17 | 22.9; 22.4 |
| Near side, under two frames | 0.474; 0.479 | 0.18 / 0.27 / 0.42; 0.18 / 0.28 / 0.43 | 0.18; 0.17 | 1.52 (0.92 to 2.67); 1.53 (0.92 to 2.72) | 0.288; 0.286 | 0.005; 0.003 | 1.74; 1.76 | 1.1; 1.1 |
| … of which near init | 0.440; 0.442 | 0.17 / 0.26 / 0.39; 0.17 / 0.26 / 0.39 | 0.17; 0.17 | 1.46; 1.47 | 0.301; 0.300 | 0.000; 0.000 | 1.66; 1.67 | 1.0; 1.1 |
| Near side, the rest | 0.190; 0.190 | 0.20 / 0.31 / 0.45; 0.20 / 0.31 / 0.45 | 0.24; 0.24 | 1.26 (0.81 to 2.04); 1.27 (0.81 to 2.04) | 0.362; 0.361 | 0.000; 0.000 | 1.47; 1.47 | 1.1; 1.1 |
| Near side, all | 0.733; 0.738 | 0.19 / 0.31 / 0.50; 0.19 / 0.31 / 0.50 | 0.20; 0.20 | 1.49; 1.51 | 0.296; 0.294 | 0.016; 0.010 | 1.77; 1.79 | 1.2; 1.3 |
| Far side | 0.267; 0.262 | 0.89 / 1.81 / 3.54; 0.87 / 1.76 / 3.47 | 0.79; 0.81 | 2.05 (1.10 to 4.55); 2.01 (1.08 to 4.28) | 0.216; 0.221 | 0.150; 0.150 | 4.65; 4.52 | 8.7; 8.6 |

Each cell gives `s0` against `s1`, then `s1` against `s0`; the quartiles are the 25th, 50th and 75th percentiles.

**The same, against two samples of one placement.** Within each population, the two seeds' primitives were also compared with each other alone (cross to the other seed's primitives of the same population, own within the population), and then pooled and split at random into two sets of the two models' sizes. The random split is what the measures would read if the two seeds were two samples of one placement: ratio 1, cross under own for half. The voxel overlap is Σ min(a, b) / ½ (Σ a + Σ b) over cubes of 1, 2 or 5 cm, with a and b the two sets' summed base opacities in a cube; its figures by count differ from these by at most 0.08.

| Population | Ratio: median, two seeds; random split | Cross under own: two seeds; random split | Overlap of opacity, 2 cm cubes: two seeds; random split | Overlap of opacity, 5 cm cubes: two seeds; random split |
| --- | --- | --- | --- | --- |
| Near side, far from init | 3.09; 0.99 | 0.155; 0.509 | 0.198; 0.612 | 0.377; 0.814 |
| Near side, under two frames | 1.56; 1.00 | 0.279; 0.503 | 0.650; 0.879 | 0.830; 0.961 |
| Near side, the rest | 1.32; 1.00 | 0.343; 0.501 | 0.680; 0.811 | 0.874; 0.945 |
| Near side, all | 1.49; 1.00 | 0.296; 0.502 | 0.653; 0.863 | 0.820; 0.952 |
| Far side | 2.05; 1.01 | 0.216; 0.495 | 0.158; 0.345 | 0.365; 0.623 |

- **No population is placed as one placement would be.** Every population's ratio is above the random split's 1, so two seeds never put their primitives where two samples of one placement would. The question is how far each population is from it.
- **The primitives far from init are placed differently, by the rule.** Their median ratio is 3.06 and 3.04, against the rest's 1.26 and 1.27, 2.4 times as high. Their share with cross under own is 0.155 and 0.147, against 0.362 and 0.361. At 5 cm the two seeds' opacity overlaps by 0.38, where a random split gives 0.81; the rest overlaps by 0.87, against 0.95. Both of the population's cells read so, and the cell that is also under two frames reads the more (4.38 and 4.26).
- **But they are placed in the same region.** For 96 % of one seed's far-from-init primitives (0.962 and 0.957), the other seed's nearest primitive is also far from init, where 14 % of its near side is. Half of them have it within 1.45 and 1.29 cm, while they lie 17.0 and 14.8 cm from the nearest init point at the median. So both seeds build a shell in the same place, to about a tenth of its distance from the surface. What differs is how its primitives are arranged at the scale of their own spacing, 0.3 to 0.5 cm: a tenth of them have nothing of the other seed within 5 cm (0.111 and 0.072), against none of the rest.
- **The primitives under two frames are placed alike, by the rule, but only just.** Their median ratio is 1.52 and 1.53, 20 % above the rest's, and their share with cross under own is 0.288 and 0.286, 0.074 below it; neither margin is reached. Their cell near init, 93 % of them, reads 1.46 and 1.47. At 5 cm their overlap is 0.83 against a random split's 0.96, a loss of 0.13, about twice the rest's 0.07 (0.87 against 0.945): a borderline case, nearer the rest than the far-from-init primitives.
- **The far side is placed differently too.** Its ratio is 2.05 and 2.01, its share with cross under own 0.22, its overlap at 5 cm 0.37, against 0.62 for a random split. It holds the background, where init points are thin (section 2.6). So the seeds' disagreement does not mark the shell alone: it marks every region that the init points and the cameras pin down loosely. On the near side, the shell's far-from-init population is the only one it singles out.
- **Within ±6 frames** the ordering holds: 6.14 and 6.17 for the far-from-init primitives, 1.74 and 1.76 under two frames, 1.47 for the rest, 4.65 and 4.52 on the far side.

**The renders.** Both checkpoints were rendered from the sheet's request, `<surface-runs>/ticket-5/side_request.json` (SHA-256 `1b88fd859d64028c9ff40a4624a6bce76edb0be95b8f4129ccfbdc5933d98821`, checked): 39 items, the 13 poses at frames 0, 60 and 119 (section 2.6, "The sheet"). On each item the two renders are clamped to [0, 1] and compared in 8-bit levels, on the pixels where both alphas are at least 0.5. A tile's pixels are also sorted by where the shell is in it. Each model's near-side primitives are projected into the item's camera, as section 2.6's framing check did (the position by E1 at the item's time, the weight σ_eff by E3 and E4). Their weight is summed in 16 px cells, as is the weight of the far-from-init ones among them. A **shell cell** is one where the far-from-init primitives hold at least a quarter of the near-side weight, in the mean of the two models; a **rest cell**, one where they hold under 5 %. No sheet was made, and the renders were looked at by no one (row 843).

| Poses | Items | Both alphas ≥ 0.5 | Mean absolute difference, levels | Its 90th percentile, levels | PSNR of one render against the other, dB | Mean absolute difference of alpha | Mean absolute difference, shell cells; rest cells, levels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `sC00`, the centre camera (the test view) | 3 | 1.00 | 4.5 to 5.9 (5.0) | 9.9 to 13.1 | 28.1 to 29.6 | 0.00 to 0.01 | 4.9; 6.0 |
| The four between cameras (`nm01`, `nm03`, `nm05`, `nx02`) | 12 | 0.97 to 1.00 | 4.1 to 6.8 (5.3) | 9.1 to 15.9 | 25.5 to 30.6 | 0.00 to 0.03 | 5.8; 6.2 |
| Yaw ±30° | 6 | 0.92 to 0.97 | 9.4 to 14.0 (10.9) | 23.1 to 34.2 | 21.0 to 23.5 | 0.03 to 0.05 | 11.9; 10.5 |
| Yaw ±45° | 6 | 0.78 to 0.94 | 13.0 to 17.0 (14.5) | 30.2 to 39.9 | 19.9 to 21.9 | 0.03 to 0.09 | 15.3; 13.9 |
| Yaw ±60° | 6 | 0.66 to 0.90 | 14.8 to 18.5 (17.0) | 33.0 to 41.8 | 19.2 to 21.1 | 0.06 to 0.10 | 18.2; 15.4 |
| Yaw ±90° | 6 | 0.49 to 0.82 | 19.5 to 31.3 (24.1) | 41.1 to 83.3 | 14.8 to 19.5 | 0.09 to 0.14 | 25.0; 22.3 |

The difference's ranges are over the items, the mean in brackets; the last column gives the means over the items. A shell cell holds 950 to 2,130 of a tile's 3,680 cells, a rest cell 230 to 1,190.

- **On the test view, the seeds differ less than either errs.** On `sC00` at frames 0, 60 and 119, the two renders lie 29.6, 29.5 and 28.1 dB from each other over the whole image. Each lies 24.8 and 24.8, 24.6 and 24.2, and 22.8 and 22.6 dB from `cam0`'s image (`s0` and `s1`), with a mean absolute error of 7.7 to 10.0 levels against their difference's 4.5 to 5.9. On the tools' crop (section 2.5, rule a) the seeds lie 30.6, 30.3 and 28.0 dB apart, and 24.8 and 24.9, 23.8 and 23.5, and 21.8 and 21.4 dB from the image.
- **Away from the cameras, the difference grows with the yaw everywhere,** fivefold from the test view to ±90°. At ±30° to ±90° it is 10 % to 18 % higher in the shell cells than in the rest cells, and in the centre and between poses it is lower there (4.9 against 6.0 levels). The frames hardly matter: 12.2, 11.6 and 13.0 levels on average over the poses at frames 0, 60 and 119.
- **So, by this split, the renders do not separate the shell** from the rest. The seeds' difference in an image grows away from the training cameras, more than it gathers where the shell is. The split was not checked against anything else, and none of the 13 poses is a training camera, so this holds for the held-out and novel poses of the request only.

**The reading** (section 5.1, row (d)). The two seeds place the shell's primitives that lie far from every init point differently, and the near side's rest alike, by the rule fixed beforehand. That is the row's first case: two models' disagreement is a signal for that part of the person, and rows E11 (RIGI's two-model uncertainty) and E22 (CoR-GS' co-pruning and co-regularization) come back beside finalist 2. They come back on unequal evidence, since they read the disagreement in different places (section 3.6):

- **Row E22's co-pruning** matches primitives by position within a distance, and the table above is its direct evidence. Beyond 1 cm lie 0.62 and 0.60 of the far-from-init primitives against 0.025 of the rest, and beyond 5 cm 0.111 and 0.072 against none. Its co-regularization, which pulls the two renders together between the cameras, rests on the renders, as row E11 does.
- **Row E11** weights the loss per pixel by the difference of the two renders, and by this section's split the renders do not separate the shell. Its evidence here is weak.

Four limits go with both:

- **It sees one of the shell's two populations.** The primitives under two frames, which make up most of the near side (0.47 of the model), are placed as alike as the rest, so the signal cannot see that population.
- **It marks the region, not a misplacement of it.** Both seeds build a shell in the same place: they disagree on its arrangement at a few millimetres to a centimetre or two, while its primitives lie some 15 cm from the init points. A term that penalized where two models disagree would push on the shell's fine arrangement. That it would remove the shell is not shown.
- **It is not the shell's alone.** The far side is placed as differently, so on its own the signal would also act on the background, where the init points are thin.
- **The renders do not localize it.** From the sheet's request the seeds' renders differ about as much in the shell's cells as elsewhere, so a signal taken from renders, row E11's and row E22's co-regularization, would be weaker than the one taken from the primitives here.

No finalist's place changes; rows E11 and E22 come back as companions to test after finalist 2, each still with a second model's cost (section 4.8).

**Check (e): colour between the cameras.**

**The matches.** On private scene A, section 2.5's own matches: the grids its run saved at offset 0, D-22's 7 pairs and all 120 frames, read again without matching anything (row 839). On `kitchen`, which gives the floor, ticket #4 saved no grids, so its matches were made again with the same script and settings, unchanged: D-22's 13 pairs and the six pairs about 15° apart of section 2.5's floor, every tenth frame, 570 evaluations (row 842). The rerun reproduces section 2.5's residuals: a median of the per-evaluation static medians of 0.027 px over the 13 pairs and 0.022 px over the six.

**The measure.** Every admitted match of both halves of the grid is read, as D-22 admits it (Q-110): both ends inside their images and a certainty of at least 0.5. Its own end lies at its grid cell's centre, and the other end where the grid stores it, to 1/16 px. Each end's colour is read bilinearly from its image as `MultiViewVideo.load_image` decodes it, pixel (i, j) covering [j, j + 1) × [i, i + 1) as in §1.4, and is given in 8-bit levels. The difference is camera B's colour minus camera A's, per channel, with the pair named (A, B) as in section 2.5. Per pair and region, the script gives:

- the mean of the signed difference, the shift that an offset per camera would remove;
- the median of the absolute difference, which also counts what a match slightly off its point reads on texture;
- the total-least-squares gain of B on A, the part a gain per camera would remove;
- the 10th to 90th percentile of the per-frame signed medians, as a range over the three channels: a difference of the cameras is steady from frame to frame, and one of the content is not.

The regions are section 2.5's. On private scene A they are split by depth along camera A's axis: the **person**, 0.5 to 1.8, and the **background**, 2.2 or more. On `kitchen`, by the ground truth's masks of dynamic objects: **static** when neither end is on a dynamic pixel, **dynamic** when both are. A **flat** subset keeps the matches where the local gradient (central differences of the three channels' mean) is under 2 levels per pixel at both ends, so a misplaced match reads little there. The script passed its checks on synthetic data. An image against itself gives 0. A copy with gains (1.1, 0.95, 1.0) and offsets (−5, 3, 2) returns them to 10⁻⁶ and 10⁻⁴. Bilinear reads at pixel centres return the pixel, the grid's ends are placed as section 2.5's script placed them, and the depth of a known point comes back to 10⁻⁹. The signed and absolute differences are read from histograms of 0.25-level bins, the means from exact sums.

**`kitchen`, the floor.** Over D-22's 13 pairs, a pair's mean signed difference is −0.8 to +0.4 levels per channel on static pixels, and −0.4 to +0.3 on dynamic ones. The absolute difference's median is 1.5 to 2.0 levels on static pixels (1.0 to 1.5 where flat) and 4.0 to 4.5 on dynamic ones. The gains are 0.993 to 1.005, and the per-frame signed medians' 10th to 90th percentiles lie within −0.8 and +0.6 levels. The six pairs 15° apart read the same: means −0.5 to −0.1 on static pixels and −0.8 to +0.3 on dynamic ones, absolute medians 1.5 to 2.0 and 4.0 to 5.0, gains 0.978 to 1.007, and per-frame percentiles within −1.4 and +0.2.

**Private scene A, per pair.** The difference, absolute-difference and gain cells give R, G and B, in levels; the counts are in millions of admitted matches.

| Pair | Person: admitted matches | Person: mean signed difference | Person: median absolute difference | Person: gain | Person: per-frame signed medians, 10th to 90th percentile | Background: admitted matches | Background: mean signed difference | Background: median absolute difference | Background: gain | Background: per-frame signed medians, 10th to 90th percentile |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `cam0`–`cam1` | 14.2 | −4.2, −4.7, −3.0 | 6.0, 6.0, 5.0 | 0.975, 0.968, 0.998 | −6.1 to −1.9 | 12.3 | −9.5, −9.9, −8.3 | 10.0, 10.5, 8.5 | 1.003, 1.004, 1.011 | −14.3 to −5.5 |
| `cam1`–`cam4` | 14.7 | +0.7, +1.2, +1.8 | 4.0, 4.0, 5.0 | 1.000, 1.013, 0.994 | −1.1 to +2.7 | 13.7 | +1.9, +1.7, +0.8 | 5.0, 4.5, 4.5 | 1.000, 1.007, 1.001 | +0.4 to +4.8 |
| `cam2`–`cam5` | 13.8 | −0.7, +0.3, +2.1 | 4.0, 4.0, 4.5 | 0.992, 0.996, 0.993 | −1.7 to +2.5 | 12.7 | +2.6, +2.3, +2.4 | 6.5, 6.5, 6.5 | 0.999, 1.003, 1.003 | +1.2 to +8.4 |
| `cam3`–`cam4` | 15.1 | −0.5, −0.6, −0.2 | 4.5, 4.0, 4.5 | 0.997, 1.005, 1.011 | −2.9 to +1.4 | 10.4 | −6.8, −6.6, −6.4 | 7.5, 7.0, 7.0 | 0.992, 1.003, 1.012 | −10.9 to −2.2 |
| `cam3`–`cam5` | 14.3 | −1.3, +0.2, +1.3 | 4.0, 4.0, 4.5 | 1.008, 1.023, 1.010 | −2.2 to +3.0 | 10.6 | +0.1, +1.6, +3.5 | 6.5, 6.0, 6.0 | 0.946, 0.957, 1.003 | −4.4 to +9.3 |
| `cam6`–`cam7` | 13.4 | −0.9, −1.3, −0.7 | 4.0, 4.0, 4.5 | 0.990, 0.984, 1.012 | −2.3 to +0.0 | 6.3 | −8.8, −8.3, −7.7 | 9.0, 8.5, 8.0 | 1.030, 1.065, 1.051 | −14.1 to −6.4 |
| `cam6`–`cam8` | 14.0 | −1.2, −0.7, −0.5 | 5.0, 4.0, 4.5 | 1.019, 1.041, 0.999 | −2.7 to +1.5 | 4.1 | +5.0, +4.3, +3.5 | 7.5, 7.0, 6.0 | 1.007, 1.012, 1.019 | −1.3 to +14.3 |
| The 7 pairs together | 99.4 | −1.2, −0.8, +0.1 | 4.5, 4.5, 4.5 | 0.998, 1.004, 1.001 | −4.7 to +2.3 | 70.0 | −2.3, −2.2, −1.8 | 7.0, 7.0, 6.5 | 0.984, 0.986, 0.989 | −11.6 to +7.0 |

The flat subset changes little. On the person, the seven pairs' means together are −1.3, −1.1 and 0.0 levels and the absolute median 3.0 to 3.5; on the background, −1.1 to −0.9 and 5.0 to 5.5. A pair's flat means stay within 1.5 levels of its means above, apart from `cam3`–`cam5`'s background red, +1.7 against +0.1.

- **Beyond the floor.** A pair's mean signed difference reaches −4.7 levels on the person (`cam0`–`cam1`, green) and −9.9 on the background (the same pair), where `kitchen`'s stays within 0.8 levels; six pairs of seven exceed 2 levels on the background in some channel, and `cam0`–`cam1` and `cam2`–`cam5` (blue) on the person. The person's absolute median, 4.0 to 6.0 levels, matches `kitchen`'s dynamic pixels, 4.0 to 5.0, but the background's, 4.5 to 10.5, is two to five times `kitchen`'s static pixels.
- **An offset more than a gain.** The gains are 0.95 to 1.07 against `kitchen`'s 0.98 to 1.01. The large shifts are additive and close to equal on the three channels: −9.5, −9.9 and −8.3 on `cam0`–`cam1`'s background, −8.8, −8.3 and −7.7 on `cam6`–`cam7`'s.
- **Not one per camera, and not steady.** Within a pair, the person and the background differ. On `cam0`–`cam1` the shift is −4.2 against −9.5 in red, on `cam3`–`cam4` −0.5 against −6.8, and on `cam6`–`cam8` it changes sign, −1.2 against +5.0. A colour correction per camera moves both alike. On the background the per-frame medians spread over 4 to 16 levels within a pair (`cam6`–`cam8`: −1.3 to +14.3), against under 2 levels on `kitchen`, so a large part of a pair's background difference changes with the frame. The person's spread is 2 to 5 levels.
- **What cannot be checked.** D-22's seven pairs form two trees, `cam0`–`cam1`–`cam4`–`cam3`–`cam5`–`cam2` and `cam7`–`cam6`–`cam8`, with no loop. Seven pair shifts can always be written as nine camera offsets, so whether they come from the cameras cannot be tested without pairs D-22 doesn't make.
- **Where it sits.** In `cam0`'s image, the camera most of these runs hold out, its pair reads the largest shift: −4.2 to −4.7 levels in red and green on the person, and −8 to −10 on the background, `cam1` darker than `cam0`. The placeholder pixels of section 2.5 do not enter: 19 of their grid points have an admitted match.

**The reading** (section 5.1, row (e)). The cameras' colours differ beyond `kitchen`'s floor, by a shift of up to 5 levels on the person and 10 on the background between neighbouring cameras. That brings back row E27, an affine colour correction in the training losses. Two things go with it:

- **It is not the whole difference.** The shift differs between the person and the background of one pair, and it changes with the frame on the background. A correction per camera, as row E27 describes it, would remove the part that is common to a camera and leave the rest; the size of that part cannot be measured from D-22's pairs alone.
- **It changes no finalist's place,** as section 5.1 says. None closes the part of section 3.6's setting that was left open. Row E27 would come back as a test of its own, after the finalists or beside finalist 2, which compares the colours of the same pixels.

**Runs.** `python` is the shared `.venv`'s, run in the worktree's checkout with `PYTHONPATH=<worktree>/src:<worktree>/harness/src` (and, for the matcher, `:<worktree>/scripts:<scratch-prd1>/ticket-4/env-init`, the scratch folder of row 612's packages, read and not written), `PYTHONDONTWRITEBYTECODE=1`, and `OMP_NUM_THREADS`, `MKL_NUM_THREADS` and `OPENBLAS_NUM_THREADS` at the thread count given; CPU runs with `CUDA_VISIBLE_DEVICES=` empty, GPU runs under the shared lock, `flock <scratch>/gpu.lock`, one command at a time. Scripts are in `<scratch>/ticket-20`, and `<scratch-prd1>` is PRD #1's scratch folder `.scratch/surface-following-primitives/`, whose `match_residuals.py` and `epi.py` were copied unchanged. `$O` is `<surface-runs>/ticket-20`, `<m15-runs>` M15's run folder that holds `kitchen`'s converted scene, as section 2.5 writes it, `<A_s0>` and `<A_s1>` the two checkpoints, and `<init>` the init points file both trained from; like `<private-scene-A>`, they are written nowhere.

| Command | Threads; device | What it ran; time | Note |
| --- | --- | --- | --- |
| `python d_seeds.py <A_s0> <A_s1> <init> <private-scene-A>/scene.json --out $O/d_seeds.json --workers 4` | 4; CPU | 2 × 3,000,000 primitives; 3 min 53 s | The populations, the cross and own distances in both directions, by position and within ±6 frames; reproduces section 2.6's figures first. A first run without the same-population and cube measures took 67 s |
| `python probe_own.py <A_s0> <init> <private-scene-A>/scene.json` | 4; CPU | 6 s | The own spacings' low quantiles on `s0`, to check that relocation's copies don't make near-zero spacings. Run with the locations in the script's arguments only |
| `python d_null.py <A_s0> <A_s1> <init> <private-scene-A>/scene.json --out $O/d_null.json` | 4; CPU | 1 min 27 s | The same measures within each population, for the two seeds and for one random split of their pooled primitives (seed 20) |
| `ftgs render <A_s0> --request <surface-runs>/ticket-5/side_request.json --out $O/renders/A_default_s0.safetensors`, and the same for `<A_s1>` | GPU | 39 renders each; 8.3 s and 8.4 s | `ftgs` is `python -m openftgs.cli`. The card held 1,293 MiB of other work when they ended |
| `python d_render.py $O/renders/A_default_s0.safetensors $O/renders/A_default_s1.safetensors <surface-runs>/ticket-5/side_request.json <private-scene-A>/scene.json <A_s0> <A_s1> <init> --out $O/d_render.json` | 2; CPU | 39 items; 26 s | The render differences, against `cam0`'s images on `sC00`, and by shell and rest cells |
| `python match_residuals.py <m15-runs>/kitchen/scene.json $O/kitchen --label kitchen --tag d22 --gt $G --frames 0:300:10 --save-grids --device cuda:0` | 4; GPU | 390 evaluations; 4 min 10 s | `kitchen`'s matches made again for check (e), on D-22's 13 pairs; `$G` is `$OPENFTGS_DATASETS/synth/ground_truth/kitchen`; grids of 2.8 GB |
| `python match_residuals.py <m15-runs>/kitchen/scene.json $O/kitchen --label kitchen --tag wide --gt $G --pairs cam06:cam10,cam05:cam09,cam03:cam07,cam08:cam12,cam12:cam16,cam02:cam06 --frames 0:300:10 --save-grids --device cuda:0` | 4; GPU | 180 evaluations; 1 min 56 s | The six pairs about 15° apart; grids of 1.4 GB |
| `python e_colour.py --selftest` | 1; CPU | under 1 s | The colour script's checks on synthetic data |
| `python e_colour.py <private-scene-A>/scene.json <surface-runs>/ticket-4/private-scene-A/d22 $O/e_colour_A.json --mode depth --workers 2` | 2 processes; CPU | 840 evaluations' grids; 2 min 59 s | Private scene A's colours at the saved matches |
| `python e_colour.py <m15-runs>/kitchen/scene.json $O/kitchen/d22 $O/e_colour_kitchen_d22.json --mode gt --gt $G --workers 4`; the same for `$O/kitchen/wide` | 4 processes; CPU | 3 min 5 s and 1 min 17 s | The floor; `PYTHONPATH` adds `<worktree>/scripts` for `gt_depth.py`'s frame reader |

The GPU ran for about 6 minutes in all, 17 s of it for the two renders and the rest for `kitchen`'s matches, which the ticket did not plan for. The `kitchen` matches (4 threads) overlapped with `d_render.py` (2) and private scene A's colour run (2 processes), so for those minutes the work held up to 8 threads, over the 4 budgeted.

### 2.4 Check (f): a depth channel's gradient in the colour pass against Q-226's device

*Ticket #21's section.* The gradient of a depth channel in the colour pass against the same loss through Q-226's device, in U-12's configuration, held to U-12's bounds.

**The result.** The depth channel passes U-12's bounds against the device on every stored tensor, in every scene where the gradient is not zero, in the four layouts. The loss is on expected depth at Q-227's pixels, the one fixed as the check's verdict. The worst cosine is 1 − 1.6·10⁻¹⁰ and the largest relative L2 error 1.79·10⁻⁵, both for `opacity_logits`, against 0.999 and 10⁻³. The same holds for accumulated depth on every pixel Q-39 keeps, where no gradient is zero, and at an alpha cut of 0.01. At the cut 0.1, one figure is outside the bounds: `log_durations` in scene 102, 1.11·10⁻³ against the device on the reference backend. It is section 2.4's case, a gradient near zero on which the two float32 backends differ by their rounding; the channel's figure against the device on the gsplat backend is 1.45·10⁻⁴, and against a float64 computation 7.28·10⁻⁴. So section 5.1's row (f) reads as passed. The adopted form of finalists 3 and 4, a depth output of `render` made of a channel of the colour pass, needs no second pass, and the way z reaches `means`, through `fully_fused_projection`'s backward, holds. Normals in the colour pass, which finalist 3's adopted form would also add, were not part of the check.

**What was run.** No gsplat file was opened, and no source outside the repository (row 832). The colour pass is the package's own `render()` on the gsplat backend, unchanged. A scratch script wraps two of gsplat's stage functions at run time, as the backend looks them up, so that `fully_fused_projection`'s depths go into `rasterize_to_pixels` as a fourth channel beside the colours (the 4-channel kernel, section 2.4, cell 1a). The wrapper keeps that channel, accumulated depth Σwᵢzᵢ, and hands the backend the first three. In unpacked mode the depths of the rows that fail the radius test are set to 0 before compositing, as the backend does for the 2D means (D-75). Every gradient was checked to be finite. The device, its two parts (i) and (ii), and U-12's scenes, cameras, time, weights and pixels are those of ticket #3's script, copied from section 2.4 ("The gradient experiment: the configuration"). One thing differs on the channel's side: both cameras are rendered in one call, as U-12 does, since the channel's z comes from the projection of each camera. The loss is the same function of accumulated depth and alpha on both sides. It reads no colour, so `sh0` and `shN` get a zero gradient on both sides: autograd returns exact zeros through the channel's graph, and the device never reads the model's SH tensors. U-12's figures cover the seven slice tensors, and its norm > 0 cannot hold for those two.

**What was fixed before the first run** (in `<scratch>/ticket-21/`, before any GPU command):

- **The verdict** is the channel against the device on the reference backend, U-12's reference. The device on the gsplat backend, same layout, is reported beside it, since it isolates the path from a depth channel back to `means`.
- **The losses** are four. Expected depth at Q-227's cut, alpha ≥ 0.5, is the verdict. Expected depth at 0.1 and at 0.01, and accumulated depth on every pixel Q-39 keeps, are reported as controls, as section 2.4 reported them. The masks come from the reference backend's alpha of the device's render, once per scene, for every comparison.
- **The zero-gradient rule** is section 2.4's. A (scene, tensor) is left out where the device's gradient on the reference backend in float64, on the CPU, has a norm below 10⁻⁹. That leaves out scenes 102, 108 and 111 at the cut 0.5, scene 111 at 0.1 and 0.01, and none for accumulated depth, always for the four tensors that have part (i) only. In that run the norms left out are at most 2.6·10⁻¹³ and those kept at least 3.1·10⁻⁴.
- **A localizer.** The channel's gradient with the depths detached should be the device's part (i), and the rest should be part (ii).

**The sanity check.** The device's gradient norms on the reference backend reproduce section 2.4's table to its six digits for every tensor: `means` 3.78465 / 32.3102 / 115.152, for example. The copied device has not drifted.

**At Q-227's pixels**, alpha ≥ 0.5: 2,647 pixels of 122,880, 0 to 264 a camera, median 48. These are the worst figures over the scenes kept and the four layouts:

| Stored tensor | Scenes | Gradient norm through the channel: least / median / largest | Against the device on the reference backend: lowest cosine, largest relative L2 error | Against the device on the gsplat backend: lowest cosine, largest relative L2 error |
| --- | --- | --- | --- | --- |
| `means` | 20 | 3.78465 / 32.3101 / 115.152 | 1 − 7.1·10⁻¹², 3.77·10⁻⁶ | 1 − 3.6·10⁻¹², 2.70·10⁻⁶ |
| `opacity_logits` | 17 | 0.00215005 / 2.78788 / 17.2819 | 1 − 1.6·10⁻¹⁰, 1.79·10⁻⁵ | 1 − 3.4·10⁻¹⁰, 4.22·10⁻⁵ |
| `times` | 20 | 1.04023 / 15.2178 / 66.5263 | 1 − 3.8·10⁻¹², 2.79·10⁻⁶ | 1 − 2.1·10⁻¹², 2.08·10⁻⁶ |
| `velocities` | 20 | 0.284468 / 7.27065 / 25.6786 | 1 − 2.3·10⁻¹², 2.21·10⁻⁶ | 1 − 9.9·10⁻¹⁴, 4.70·10⁻⁷ |
| `log_durations` | 17 | 0.000276061 / 3.16328 / 13.7959 | 1 − 1.3·10⁻¹², 1.27·10⁻⁵ | 1 − 7.6·10⁻¹³, 5.36·10⁻⁵ |
| `log_scales` | 17 | 0.0605743 / 11.4854 / 44.2033 | 1 − 6.1·10⁻¹², 3.50·10⁻⁶ | 1 − 2.9·10⁻¹², 2.92·10⁻⁶ |
| `quats` | 17 | 0.0743028 / 10.5384 / 49.1698 | 1 − 1.3·10⁻¹¹, 5.09·10⁻⁶ | 1 − 6.1·10⁻¹², 3.49·10⁻⁶ |
| `sh0`, `shN` | 20 | 0, exactly | zero on both sides, no figure | zero on both sides, no figure |

The norms are those of the packed layout with the opacity cull inside gsplat; the other layouts' agree to the digits shown. The largest errors, for `opacity_logits` and `log_durations`, are scene 114's, where those gradients are the smallest, as in section 2.4. In the scenes left out, the channel's norms are 3.6·10⁻⁸ to 8·10⁻⁷, the order of the device's float32 rounding there (7·10⁻⁸ to 10⁻⁶).

**The controls.** These are the worst figures over the four layouts and the seven slice tensors, against the device on the reference backend:

| Loss | Pixels a camera: least / median / largest | Scenes left out for the four tensors with part (i) only | `means`: largest relative error | Largest relative error over the other six tensors | Against the device on the gsplat backend: largest relative error |
| --- | --- | --- | --- | --- | --- |
| Expected depth, alpha ≥ 0.5 (Q-227; the verdict) | 0 / 48 / 264 | 102, 108, 111 | 3.77·10⁻⁶ | 1.79·10⁻⁵ (`opacity_logits`) | 5.36·10⁻⁵ (`log_durations`) |
| Expected depth, alpha ≥ 0.1 | 13 / 426 / 1,719 | 111 | 3.86·10⁻⁶ | **1.11·10⁻³** (`log_durations`, scene 102, cosine 1 − 6.0·10⁻⁷); `opacity_logits` 7.04·10⁻⁴, the same scene | 1.45·10⁻⁴ (`log_durations`, scene 102) |
| Expected depth, alpha ≥ 0.01 | 67 / 1,256 / 3,070 | 111 | 1.46·10⁻⁵ | 1.11·10⁻⁴ (`quats`, scene 108) | 1.25·10⁻⁵ (`times`) |
| Accumulated depth | 3,067 / 3,071 / 3,072, every pixel Q-39 keeps | none | 4.61·10⁻⁶ | 2.65·10⁻⁶ (`quats`) | 1.01·10⁻⁶ (`quats`) |

Every cosine in the table is at least 1 − 6.0·10⁻⁷.

**The one figure outside U-12's bounds.** At the cut 0.1, in scene 102, the gradients for `log_durations` and `opacity_logits` are near zero: norms of 3.3·10⁻³ and 1.0·10⁻², against medians of 15 and 22. The channel and the reference backend's device differ there by 3.7·10⁻⁶ and 7.2·10⁻⁶ absolute, in all four layouts alike. The two backends' devices give 1.20·10⁻³ and 7.37·10⁻⁴ between themselves, section 2.4's figures, and the channel lies near the gsplat device (1.45·10⁻⁴ and 5.5·10⁻⁵). To see which side the figure belongs to, a second script was written after the first run had shown the figure; nothing else was changed. It computes the device on the reference backend in float64, on the CPU, with the same CUDA-drawn weights and the same pixels. It then holds the three float32 gradients to that, over the 20 scenes and the four losses, in the default layout: the channel, the device on gsplat and the device on the reference backend. All three are inside U-12's bounds everywhere. At the cut 0.1 the largest relative errors are 7.28·10⁻⁴ for the channel, 8.34·10⁻⁴ for the gsplat device and 3.95·10⁻⁴ for the float32 reference, all three for `log_durations` in scene 102. At Q-227's cut they are at most 3.16·10⁻⁵, 2.35·10⁻⁵ and 4.43·10⁻⁵, and for accumulated depth at most 3.78·10⁻⁶, 2.02·10⁻⁶ and 2.22·10⁻⁶. So the figure outside the bounds is one float32 computation against another, on a gradient whose relative figure compares rounding. It is expected depth's own property, which section 2.4 named, not an error in the channel's path. Section 5.1 says what follows if a tensor fails: the adopted form keeps a second pass, or the way z reaches `means` is looked at first. Neither follows here, since every tensor is within the bounds against float64, and against the device at Q-227's cut.

**The localizer.** Away from the scenes where part (i) is zero, the channel's gradient with the depths detached matches the device's part (i). For `means`, `times` and `velocities` at Q-227's cut, the cosines are at least 1 − 1.1·10⁻¹⁰ and the relative errors at most 1.62·10⁻⁵. What the depths add matches part (ii), at least 1 − 1.7·10⁻¹¹ and at most 5.87·10⁻⁶. For accumulated depth the figures are at most 5.95·10⁻⁶ and 4.15·10⁻⁶. So the colour pass divides into the device's two parts: through the weights, and through z to `means`, `velocities` and `times` (E1). For the four tensors that have part (i) only, the detached channel is the channel itself.

**The forward pass.** Outside Q-39's pixels, the channel's accumulated depth differs from the device's image on the gsplat backend by at most 9.5·10⁻⁷, and from the reference backend's by at most 1.48·10⁻⁵, where it reaches 7.97. Section 2.4 found 1.45·10⁻⁵ between the two backends' devices. Alpha differs by at most 1.9·10⁻⁶. The channel's expected depth differs from the reference device's by at most 3.8·10⁻⁶ at Q-227's pixels, 1.48·10⁻⁵ at alpha ≥ 0.1 and 4.29·10⁻⁵ at alpha ≥ 0.01, alike in the four layouts.

**What it changes.** Section 5.1's row (f) reads as passed. The depth output that finalists 3's and 4's adopted form would add to `render` can be a fourth channel of the colour pass, about 0.6 ms on kitchen (section 2.4, "The cost"), without a second pass, and its gradient agrees with the device's within U-12's bounds. Three things stay as section 2.4 left them:

- **A U-12 line for a depth output** would still weight accumulated depth, or expected depth with image and alpha, so that U-12's norm > 0 holds. Expected depth's gradient is zero for four tensors wherever one primitive covers every kept pixel, and a relative figure there compares rounding.
- **The reference backend** has no depth channel. This check compares the gsplat backend's colour pass with the device on both backends, and the reference backend's side of such an output is still the spec revision's (section 2.4, part 3).
- **Normals** as channels of the colour pass (cell 4a) were not run against anything.

The finalists' tests go through the device, as section 5.1 says, and did not wait on this.

**The commands.** The package is this ticket's branch at 9bd213c (`src/`, `tests/` and `scripts/` as `main`'s at 02e3686); Python 3.14.10, PyTorch 2.15.0+cu132, gsplat 1.5.3; RTX 4090, CUDA 13.2, driver 595.99.02. The two scripts, `channel_gradients.py` and `tiebreak.py`, are in `<scratch>/ticket-21/` and are not committed, with a third, `tables.py`, which only reads `gradients.json` for the tables' worst figures and their scenes. The first two import the worktree's package and its test helpers (`tests/builders.py`, `tests/render_margins.py`) read-only. Every command but `tables.py`'s ran in `<scratch>/ticket-21/` with `OMP_NUM_THREADS=2 MKL_NUM_THREADS=2 OPENBLAS_NUM_THREADS=2` and `PYTHONPATH=<worktree>/src:<worktree>/harness/src:<worktree>/tests:<scratch>/ticket-21`. `<worktree>` is this ticket's worktree, `$PY` the shared `.venv`'s Python, and `$LOCK` the PRD's shared GPU lock under `<scratch>`. The GPU commands ran under `timeout 1200`. Each command ran once.

| Command | Device | Wall time | What it gave |
| --- | --- | --- | --- |
| `CUDA_VISIBLE_DEVICES= $PY channel_gradients.py norms64 <surface-runs>/ticket-21/norms64.json` | CPU | 2 s | In float64 on the reference backend, the device's gradient norm per scene, tensor and loss: the zero-gradient rule's input |
| `flock $LOCK $PY channel_gradients.py gradients <surface-runs>/ticket-21/gradients.json <surface-runs>/ticket-21/norms64.json` | GPU | 10 s | U-12's 20 scenes in the four layouts, the four losses; the channel against the device on both backends, the localizer, and the forward differences |
| `$PY -I tables.py <surface-runs>/ticket-21/gradients.json`, in `<scratch>/ticket-21/` with no other setting | CPU | seconds | The worst figures per tensor and loss over the scenes and layouts, with the scene and layout of each. The scene-102 figures, the localizer's, the zero-gradient margins and the SH tensors' zeros were read from `gradients.json` and `norms64.json` by short inline reads, and the worst scene of the float64 comparison and the rule's agreement with its float64 norms from `tiebreak.json` |
| `flock $LOCK $PY tiebreak.py <surface-runs>/ticket-21/tiebreak.json <surface-runs>/ticket-21/norms64.json` | GPU and CPU | 7 s | Written after the first run: the channel and both float32 devices against the device on the reference backend in float64, with the same weights and pixels, default layout |

### 2.5 The three reads: FreeTimeGS++, SparseCam4D, SSA-3DGS

*Ticket #22's section.* FreeTimeGS++ in full (arXiv 2605.03337), SparseCam4D (arXiv 2603.26481) and SSA-3DGS (arXiv 2607.05598), each ending in a screening row of its family with section 3's nine columns.

**What was read, and how.** The research document's section 5.1, row "Reads", and section 4.6, item 10. Each paper was fetched from arXiv without credentials into `<scratch>/ticket-22/` and read by the funnel's last step, the method, the experiments that isolate it and the limitations; FreeTimeGS++ whole, as the owner allowed on 2026-11-16 (section 1 here), its section 3.2 on how its authors reproduced FreeTimeGS and its appendices A to G included (row 833). The rows follow section 3's columns and its "How a cell is written"; family E's rows use section 3.6's readings 1 to 3, cited as "reading 1" and so on. Nothing was run: no GPU, no private scene A, no estimator.

| Look-up | Date | Route | What | Results seen | Read in full | Rows added | Note |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 2026-11-16 | arXiv abstract page and HTML, curl | FreeTimeGS++, arXiv 2605.03337 v3 | 1 | 1 | 1, family C part 2 | Its abstract and parts of it were read by families C, D and E (rows 711, 766, 829); by this ticket's reading this read counts against no family's cap, section 4.6's item 10 listing it apart. Its code address and project page were not opened (row 834) |
| 2 | 2026-11-16 | arXiv abstract page and HTML, curl; then the project page and the repository's license file and README | SparseCam4D, arXiv 2603.26481 v3 | 1 | 1 | 1, family E | Family E's 25th full read. Its abstract was read by family C (row 711), which handed it over, and is not counted again (rows 835, 836, 837) |
| 3 | 2026-11-16 | arXiv abstract page and HTML, curl; then one web search for its code | SSA-3DGS, arXiv 2607.05598 v2 | 1, and 10 in the search | 1 | 1, family E | Family E's 26th full read. Its abstract was read by family B (row 675), which named it for family E. The search found no code repository and no result was opened (row 838) |

**Family C, part 2: the row no family gave.** Section 3.4 left FreeTimeGS++'s initialization unread under the interim rule, and section 3.5 handed it over (section 3.4, "Handed to this family"). The row is laid out as rows C14 and C15, the initializations section 3.4 screened.

| Technique | 1. Weights' license | 2. Code license | 3. Integration class | 4. Render backends | 5. Fit with the primitives | 6. Cameras that disagree | 7. Prior maps | 8. Evidence | 9. Training cost |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **Initial velocity from optical flow, where the flow is valid** (FreeTimeGS++, Lee et al., 2026, arXiv 2605.03337, its section 5.2; a paper that cites FreeTimeGS): the init points are projected into the cameras, the flow of each camera is sampled there and back-projected, the estimates of several cameras are averaged by covisibility, and the result replaces the nearest-neighbour velocity where a validity mask from confidence and multi-view consistency allows; elsewhere the nearest-neighbour velocity stays (row 833) | UFM, which **fails** for each of its six files: non-commercial by both of its publisher's statements (section 3.5's estimators table, UFM; row 783). The paper names no file (row 833). A passing file exists for flow, SEA-RAFT's one file (section 3.4's estimators table), and for tracks, the TAPIR family and AllTracker; RAFT and WAFT are **unclear** (section 3.3) | Paper only. The paper reproduces and extends FreeTimeGS, so the code address its abstract gives and its project page stay unopened (rows 767, 834) | **None of the five classes: an initialization.** It changes only the velocities the training starts from, and training then moves them freely (E1; row 833). `velocities` is a key of `ftgs-init` v1, and points supplied through the public interface skip steps 1 to 6 of §1.3's initialization, so a file written outside the package, for instance a file of `ftgs init roma` with its velocities replaced, can be tried with no spec change (§1.3, §2.2; this reading). Inside the package it would change step 5, E15 [P], whose formula is D-23's. With `roma_init` as it is, E1 to E17 and the §3 tests are untouched | Nothing: an initialization reaches no render backend (row 833) | It keeps E15's structure and replaces its value point by point: a velocity over one init interval, as E1 and D-21's start ask, with the nearest-neighbour velocity as the fallback. E15 takes one nearest neighbour without an outlier filter (D-23); the paper's reproduction takes three (row 833). The paper's reason is its own finding that a clean render can hide a noisy velocity map, primitives switching roles over time; its Figure S1 shows the velocity maps cleaner with the flow, before and after optimization, on one scene (row 833). What it does to durations, relocation (E10, E11) or split (E17) is not stated | **Within one camera over time** for the flow, then **across cameras** by the average and by the mask's consistency test, whose thresholds the paper does not give (row 833). By this reading, on private scene A, where the cameras disagree about the person (section 2.5), the test would reject the person's points first, and those keep E15's velocity: it acts least where the shell is. Dense rigs only, 18 and 24 cameras; no evidence with few cameras, inconsistent images or pixels known to be wrong (row 833) | Flow per camera between sampled frames: 30 frames of 18 cameras and 60 frames of 24, which took 741 s and 1,708 s on one workstation GPU (row 833). Flow maps the user supplies would do, per pair of frames of one camera, as UFM's row says (section 3.5's estimators table); so would a supplied init file with velocities, which puts no estimator in the package (§2.2) | DyNeRF and SelfCap, PSNR, Table 5, one line each: adding it moves the reproduction from 32.62 to 32.73 dB and from 26.43 to 26.25 dB, against ten-run spreads of 0.22 and 0.36 dB in the same paper (rows 766, 833). Figure S1 is qualitative. Nothing measures geometry. This rests on the authors' reproduction of FreeTimeGS. The nearest measured initializations are rows C14 and C15 (section 3.4) | None at training time. Initialization takes 44 s and 81 s from cached flow, against 17.5 s and 20.7 s with nearest neighbours, on a CPU with one process of 16 threads (row 833) |

**Family E: the two rows family E had no read left for.** Section 3.4 handed SparseCam4D to section 3.6, and section 3.2 SSA-3DGS, after family E had reached its cap (section 4.6, item 10).

| Technique | 1. Weights' license | 2. Code license | 3. Integration class | 4. Render backends | 5. Fit with the primitives | 6. Cameras that disagree | 7. Prior maps | 8. Evidence | 9. Training cost |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| **SparseCam4D's Spatio-Temporal Distortion Field** (Pan et al., CVPR 2026, arXiv 2603.26481): 2 or 3 real cameras and 20 to 25 views generated by a video diffusion model; each generated view is rendered from the primitives displaced by a field over (x, y, z, t, s), s the index of the generated view, nine feature planes and a small MLP giving offsets of position, rotation and scale; the real views are rendered without it, and it is dropped after training (row 835) | The field uses none. Beside it: LPIPS on the generated views, backbone not stated (row 835), **fails** (section 3.6's estimators table, LPIPS); VGGT for the generator's conditioning (row 835) and, by the README, for the generated views' poses (row 837), **fails** for both of its files (section 3.3), as does DUSt3R, which it replaces (section 3.3; row 835). The repository's data layout holds depth maps from Video Depth Anything, size not named and use not stated in the paper (row 837): Small **passes**, Base and Large **fail** (section 3.3). The generators, ViewCrafter and ReCamMaster, are not estimators (section 3.6) | Apache-2.0 by the repository's license file, `inspatio/sparse-cam4d` (row 837). The README says the code builds on two other 4D Gaussian codebases whose licenses were not checked, so no source was read; the technique is taken from the paper (rows 835, 837) | Nearest class: a change to a quantity or to motion, as rows E8 and E10: where a primitive is rendered, and its orientation and scale, become functions of the view as well as of t, which E1 [P] and §1.1's constant orientation and scale do not allow. The class leaves out that the added parameters are a network outside the model, used in training only (reading 1); rendered without it the model is a plain one, so `render` and E16 keep their form. The paper's two other parts have rows already: poses optimized during training with a pull towards COLMAP's, as rows E6 and E15; a low L1 weight with LPIPS on generated views, as rows E1 and E2 (row 835). With the field off, untouched | Nothing new: the offsets move the primitives before rasterization, and the render is the colour image (section 2.3). No depth, normal or distortion output (row 835) | Its primitive is a 4D Gaussian sliced in time, not E1's, and the field offsets its 4D rotation and scale (row 835). Under §1.1 it would offset μx, the quaternion and the scales per view and time, in training only. The field is read at a primitive's position, not stored per primitive, so relocated copies (E10, E11) and split children (E17) need nothing copied and no primitive's identity has to last (this reading) | **It trusts the real cameras:** they are rendered without the field and anchor the model, and only the generated views are allowed to disagree, in space and in time (row 835). Private scene A has no trusted camera (decision 4). With the field on all nine cameras nothing anchors the model: the paper's terms on the field are total variation and a smoothness along the view axis, and no pull of the offsets towards zero is stated, where row E10's 4Real states that without one the model overfits (row 802; this reading). The smoothness is a second difference along the generated views' index, which the paper justifies by views along one trajectory; nine fixed cameras of a rig have no such order (this reading). Evidence with few cameras and inconsistent images: yes, 2 or 3 real cameras, and generated views, never real ones that disagree. Its failure case: poor generated images, a deformed body or ghosting, which modelling their inconsistency does not fix (row 835) | For the paper's setting, generated views per time and trajectory, 25 frames a sequence, poses and points from COLMAP at the first frame (row 835). For private scene A none: its views are given, and the field reads no map | Technicolor, Neural 3D Video and the Nvidia dynamic scenes, scored on every camera not trained on. Table 2, one Technicolor and one Nvidia scene, LPIPS and SSIM: without the field 0.608 and 0.426, 0.319 and 0.674; with it 0.264 and 0.656, 0.170 and 0.793; without its time axis or its view axis, between the two. Table 4, one DyNeRF scene, PSNR: + 2.51 dB with ViewCrafter's views and + 1.76 dB with ReCamMaster's. Its supplementary section 7 says PSNR barely moves without the field, the blur favouring it. Nothing measures geometry (row 835). **Test views:** the test poses are refined on the test image with the model fixed before scoring (row 835), a fit to the image being scored (reading 3) | Not stated: 30,000 iterations, one real and one generated image an iteration, on one A800; the field is queried for every primitive on a generated view's iteration. Rendering after training is unchanged, 11.9 against 11.5 frames a second (row 835) |
| **SSA-3DGS's screen-space overlay** (Overdulve, Jorissen and Michiels, 2026, arXiv 2607.05598): a colour map and an alpha matte of the image's size, shared by every training image, composited over the render before the loss, with an L1 sparsity term on the matte; the overlay starts from per-pixel statistics of the training images and is dropped at test time (row 838) | none: no estimator. The method uses no pretrained model (row 838) | Paper only: the paper names no repository (row 838), and this ticket's web search for its code on 2026-11-16 found none (look-up 3 above). It is built on gsplat (row 838) | Nearest class: a loss on today's primitives, as row E27: the render is composited under a learned image before E7 and only the composite enters the loss (row 838). On a rig it would be one overlay per training camera, since the paper's one overlay belongs to one moving camera (this reading). The class leaves out the four numbers per pixel and camera (reading 1). With the field off, untouched; the paper renders test views without the overlay (row 838), so E16 keeps its form | Nothing new: it is composited on the rendered colour image (row 838; section 2.3). No depth, normal or distortion output | The primitives are untouched. What it removes is primitives placed near a camera to mimic an artifact (row 838). Pixels the overlay explains send the render less gradient, so E10's g asks for fewer primitives there (reading 2's argument, this reading) | **It assumes the artifact stays fixed on the sensor in every image while the scene moves across it**, the camera moving, and a dense capture; its limitations say artifacts only nearly fixed are only partly captured (row 838). Private scene A meets neither part. Its cameras are fixed, so the background is also fixed on each sensor, and the statistics start would make the static background nearly opaque in the overlay, leaving only the other cameras and the sparsity term to argue it back (this reading). Its placeholder pixels move: over `cam0`'s 120 frames 26,478 different pixels are placeholder pixels at least once, and 280 in at least half of the frames (section 2.5); an overlay constant over a camera's frames could hold at most those. The paper leaves dead pixels out as too small to matter for plain 3DGS (row 838). No evidence with several fixed cameras or with pixels known to be wrong | None: the overlay is initialized from the training images' per-pixel mean and spread, computed in training (row 838) | Mip-NeRF 360 with five synthetic artifacts held identical in every training image, mean PSNR with ADC: 24.47 dB corrupted, 27.40 with the overlay, 27.76 trained clean; with 3DGS-MCMC 24.38, 29.00 and 29.46; DeSplat 25.51 and SpotLessSplats 25.74. Table 3 isolates its parts: sparsity + 1.93 dB on the synthetic set, little on 15 real scenes; the statistics start + 1.31 dB and + 1.56 dB over a constant one. Up to 3.3 dB lost with the wrong sparsity weight. Nothing measures geometry (row 838) | Not stated: one image-sized map per camera and one alpha blend per pixel, beside 30,000 iterations of gsplat's default configuration (row 838) |

**Estimators.** None is new. Each estimator the three papers name has its verdict already: UFM in section 3.5's estimators table; LPIPS in section 3.6's; VGGT, DUSt3R and Video Depth Anything in section 3.3, and the passing flow and track files the first table here names in sections 3.3 and 3.4. RoMa, which FreeTimeGS++'s reproduction and SparseCam4D's repository both use for initial points, is the spec's own matcher (D-58; rows 833, 837). ViewCrafter and ReCamMaster are generators, which section 3.6 gives no row; SSA-3DGS uses no pretrained model, and Difix3D+ and FLUX.1-Kontext appear in it as baselines only (rows 835, 838). So no weights' terms were looked up, and the reads add no estimators table.

**What the whole of FreeTimeGS++ adds to rows D2 and E27,** which stand as written (row 833):

- **Row D2, the gate.** Its settings: γ = 20, logits starting at −1 and a weight of 10⁻³ on the regularizer, whose form is still not given. So every gate starts near 2·10⁻⁹, which is E4 at the start, and training opens it. The gate's measured gain stays inside the paper's spread.
- **Row E27, the colour correction.** It starts at the identity at iteration 3,000, with a weight of 10⁻³. Appendix D adds a second method: on STGS, DyNeRF, six runs a setting, 31.77 ± 0.33 to 33.06 ± 0.19 dB, with LPIPS 0.047 to 0.045. It is the paper's one component measured over repeated runs on two methods. Its place still waits on check (e) (research document, section 5.1).
- **The reproduction it rests on:** 32.62 dB on DyNeRF and 26.43 on SelfCap over six runs, against the original paper's 32.97 and 27.27, with a standard deviation over runs of up to 2.78 dB on one SelfCap scene. Every FreeTimeGS++ figure cited in the research document and here, apart from appendix D's on STGS, is measured on that reproduction.

**What each read changes, as section 5.1 says it may.** No read removes a finalist (section 4.6, item 10), and none proposes an arm: arms beyond one per finalist are the owner's (section 1 here).

- **FreeTimeGS++, for check (a)'s reading.** The paper's account comes from real, synchronized rigs of 18 to 24 cameras, whose agreement it does not measure (its colour correction is there for differences between cameras). After training, the primitives that live long render the static background and the short-lived ones the moving subject; rendered alone, each group shows its part. As one possible cause of a noisy velocity map it names role-switching, different primitives explaining the same content at different times (row 833). By this reading, the paper leads one to expect many short-lived primitives wherever something moves, without any disagreement between cameras; it shows this on one figure of one scene. That is what check (a) measures. Three cautions bear on how its figures are compared:
  - **The paper's figure is not section 2.6's measure.** Section 2.6 counts primitives whose s, E4's standard deviation in time, is under two frame intervals. The paper's "duration" is 6s, and its "lifetime" is the time a primitive's rendered opacity stays above a threshold θ, 2s·√(2·ln(o/θ)), which grows with the base opacity o as well (row 833). At private scene A's median base opacity of 0.16 (section 2.6) and the paper's inactive threshold of 0.005, an s of two frames is a lifetime of about 10.5 frames, under a tenth of private scene A's 120. The paper's Figure 1(a) is lifetime over the span on a logarithmic axis for one DyNeRF scene, not named, its θ not stated and its vertical axis not evenly scaled: almost no primitive lives under a hundredth of the span, about four in ten live the whole span, and the rest spread around a tenth (row 833). It cannot be read as a share under two frames, so it is no figure to set beside the under-two-frames share. Its reproduction also starts elsewhere: initial opacity 0.5 and a temporal duration of 0.1 of the span (row 833), where §1.3 starts at opacity 0.1 (step 9) and s₀ = k_stride·Δt_f (step 6, D-21).
  - **The share over the whole model depends on how much of the scene is still.** The paper's long-lived group is the background. A share over the whole model therefore moves with the still part of the scene, and the comparison that speaks to H2 is the one inside the moving part, which check (a) takes with the ground truth's dynamic masks for the near side (research document, section 5.1).
  - What the paper does not say: how short the subject's primitives are in frames, or anything about cameras that disagree. Check (a) remains the control.
- **FreeTimeGS++'s velocity from flow, for the finalists:** none of them changes. It can be tried without a spec change, as a supplied init file, but UFM fails the bar, its one measured gain is inside the paper's spread and is negative on SelfCap, and by this reading its consistency test acts least on the person. It sits with rows C14 and C15 as an initialization that waits for a result that calls for it.
- **SparseCam4D, for finalist 2.** As an alternative it is the mechanism of rows E8 and E10, a deformation per view, which review 1 did not choose. The paper adds an ablation that isolates it with few cameras (Table 2) and a field that needs no lasting identity. It also adds two cautions for private scene A: it needs real views that it trusts, and its smoothness needs views in an order. As a companion to finalist 2's shift per patch, nothing measures how the two would combine. Its joint pose optimization is another instance of rows E6 and E15, finalist 1's family, with its own ablation (Table 3: LPIPS 0.336 to 0.264 and 0.217 to 0.170); finalist 1 does not change.
- **SparseCam4D and SSA-3DGS, for check (a):** nothing. Neither paper concerns durations or short-lived primitives (rows 835, 838).
- **SSA-3DGS, for the finalists:** nothing. It acts on no part of private scene A's disagreement that a finalist tests (its column 6).
- **SSA-3DGS, for the placeholder pixels.** It does not fit them: they move with the person and the cameras stand still, and both are against its premise. Row E13's known masks stay the answer, with nothing beside them (research document, section 3.6, item 3), and they are out of this PRD's scope (section 1 here).

**The counts.** Family E has spent 26 of its 26 full reads, two of them here, and still 55 of its 120 abstracts: the two abstracts were counted by families C and B (rows 711, 675). By this ticket's reading FreeTimeGS++ counts against no family's cap, family D having counted it (section 3.5) and section 4.6's item 10 listing it apart. One web search, and no forbidden source was met.

### 2.6 Step 1's review: the checks' readings and the owner's go for the baseline

*Ticket #23's section.* Every result of step 1 read against section 5.1's last column, a recommendation per finalist, keep or drop, and the ask for step 2.

Written on 2026-11-16 from sections 2.1 to 2.5 here and the research document's sections 4.4 to 4.6, 5.1 to 5.7 and 7.2; review 2's answers are cited as section 1 here records them. Nothing was trained, run or measured for it, and it opened no source outside this repository, so it adds no source-log row.

**Each result against section 5.1's last column.**

| Check | What section 5.1 said its result would change | What step 1 found | The reading |
| --- | --- | --- | --- |
| (a) | Whether the shell's measures stand as criteria, or are the method's habit on cameras that agree. The latter takes away finalists 1 and 2's reason as tests of H2 by removal | Split by measure (section 2.1 here). The under-two-frames share on openftgs-synth's moving parts is 0.49 to 0.71, against 0.647 on private scene A's near side. The far share in multiples of S is 0.000 to 0.003 on the dynamic side, against 0.12 to 0.13. In metres it is 0.23 to 0.46 at 5 cm and 0.03 to 0.08 at 10 cm, as high as private scene A's. Only beyond 25 cm do the two differ: 0.035 to 0.047 against 0.001 | **The under-two-frames share is the method's habit** on moving content. It falls under the row's branch in which the shares are like private scene A's, and finalist 2's named measure goes with it. **The far share stands in S and not in metres.** Its tail beyond 25 cm separates the two scenes under both rulers (below, "The far share's ruler") |
| (b) | Whether the shell follows the disagreement in time. A share no higher after frame 80 weakens finalist 2 most | The near side's under-two-frames share is higher after frame 80 in 44 of the 45 checkpoints (median +0.096), but by 20-frame bins it climbs with the person's approach from the first frames. The split cannot read the near side's far-5 cm share, because the clip's ends bias it (section 2.1 here) | **Neither branch.** The rise is as well explained by the person's approach as by the disagreement, and it is a rise in a measure check (a) found to be the method's habit. It neither weakens nor supports finalist 2 |
| (c) | What finalist 1 tests, by section 5.3's two cases | Section 5.3's **second case** (section 2.2 here). Over the seven pairs on held-out frames, the background's residual falls from 0.87 px to 0.43 and 0.55 px, but not in every pair: within the seven, `cam3`–`cam5` and `cam6`–`cam8` stay; of the held-out camera's pairs on the centre copy, `cam0`–`cam2` and `cam0`–`cam6` rise (0.42 to 0.76 px, 0.43 to 0.63 px). The person's residual stays at 2.29 to 2.35 px against 2.37 px. Its 4.0 % between pairs becomes 3.0 % and 3.8 %, not 1.0 % to 1.6 %: the turns reach section 2.7's shift b and not its scale a. Moving the centres gives the same case and moves S. The corrections also flip D-22's pairs at the 15.0° ties | **Finalist 1 would test the background's part of the disagreement alone,** and little change is expected on the person. By section 7.2, the drop clause is not triggered, since the background fell on held-out frames. The adopt clause's "for every pair" is not met. That leaves the owner's call of sections 4.5 and 5.7 |
| (d) | Whether rows E11 and E22 come back | The first case for the shell's far-from-init primitives. The two seeds place them differently (0.62 and 0.60 beyond 1 cm, against 0.025 for the rest). The primitives under two frames are placed alike. The far side is placed differently too, and the renders do not localize the shell (section 2.3 here) | **E11 and E22 come back** as companions to test after finalist 2, on unequal evidence: E22's co-pruning has direct evidence and E11's is weak. This is no arm of this PRD (section 1 here) |
| (e) | Whether row E27 comes back | Between neighbouring cameras, shifts of up to 5 levels on the person and 10 on the background, beyond `kitchen`'s floor. They are mostly an offset, not one per camera, and on the background they change with the frame (section 2.3 here) | **E27 comes back** as a test of its own, after the finalists or beside finalist 2. It changes no finalist's place. It does qualify finalist 2: part of what that loss sees between cameras is colour, which a shift does not absorb |
| (f) | Whether the adopted form of finalists 3 and 4 can rest on a depth output of the colour pass | Within U-12's bounds on every stored tensor at Q-227's cut. One figure at the cut 0.1 lies outside them. Against float64 it is one float32 computation against another (section 2.4 here) | **Passed**, which is ticket #21's reading and is put to the owner below. The adopted form can be a fourth channel without a second pass. Normals as channels, and the reference backend's side, stay open |
| Reads | FreeTimeGS++ on check (a)'s reading, and on a velocity row. SparseCam4D as an alternative to finalist 2. SSA-3DGS on the placeholder pixels | Section 2.5 here | **FreeTimeGS++ agrees with (a)'s reading** in kind: it expects short-lived primitives wherever something moves. Its lifetime is not the under-two-frames share, though. Its velocity from flow gets its row and waits. **SparseCam4D** is rows E8 and E10's mechanism and changes no finalist. **SSA-3DGS** does not fit the placeholder pixels, and row E13 stays. No read removes a finalist |
| Model | The nine-camera default's row, and section 4.2's reading at stride 1 | The under-two-frames share is the same with `cam0` in training (0.518, against 0.517 and 0.519). The far-5 cm share is 0.012 to 0.014 lower, just beyond the seeds' 0.01. Its share against its own init stays empty (section 2.1 here) | **Section 4.2's reading holds for the under-two-frames share.** For the far share it holds only with a qualification. No finalist's place changes |

**The far share's ruler, for the criteria finalists 1, 3 and 4 name.** All three name the near-side far-5 cm share (sections 5.3, 5.5 and 5.6).

- **What the ruler changes, and what it doesn't.** On private scene A the two rulers give the same figures, in section 2.1 here's table of other thresholds, so a run there reads the same number either way. The ruler matters in two places. One is how a fall is read: most of the 5 cm share on the near side lies in the 5 to 25 cm band, which openftgs-synth fills as fully with cameras that agree. A fall there may be a change in the method's habit rather than a disagreement removed. Finalist 3 is most exposed, since flattening could thin that band on any scene. The other place is openftgs-synth, finalist 1's control on `kitchen`: in S its baseline is near 0, in metres it is not.
- **The tail beyond 25 cm** is the one part of the share that separates the two scenes under both rulers: 0.035 to 0.047 of the near side's opacity against 0.001. It is thin evidence for a criterion:
  - It rests on three checkpoints of private scene A, `A_default_s0`, `c3_s4` and the fourth model.
  - Its spread between seeds was not measured (`A_default_s1`'s cells are blank).
  - At 0.035 to 0.047, section 4.4's margin of 0.01 asks it to fall by a quarter or more.
  - openftgs-synth's dynamic side is not private scene A's near side.
- **Recommended:**
  - Keep the near-side far-5 cm share as the named measure, with the threshold and seed rule that review 1 accepted.
  - Report the near-side far share beyond 25 cm beside it for every run, its spread between seeds taken from the baseline's two seeds on the centre copy, as the sharpness ratio's is. Read a fall in the 5 cm share that the tail does not share as a change in the method's habit, not as a disagreement removed. What counts as the tail sharing a fall is fixed at step 2 from the baseline's two seeds, before any arm runs.
  - On openftgs-synth, take the share at 0.12·S and at 25 cm.

  This changes how a confirmed criterion is read, so it is the owner's (question 5).

**A recommendation per finalist.**

| Finalist | Recommended | Why | What another answer would change |
| --- | --- | --- | --- |
| 1. The cameras corrected once | **Drop its runs, and keep the corrected manifests on file.** The correction does not become the manifest later arms train on | Check (c) gave section 5.3's second case. The named measure, on the near side, is not expected to move when the person's part is left in place. The adopt clause's "every pair" is not met, and on the centre copy the held-out camera agrees worse with two partners than at the manifest's pose. Kept, it would also carry D-22's changed pairs into every later arm's init (question 1) | **Kept:** step 3 runs (2 GPU hours, 2 more with `kitchen`'s control), reading the background's part alone, with question 1 to answer first. Finalist 2 then trains on the corrected manifests (section 5.7). The documents name no baseline for it there: section 5.2's baseline is on the manifest's cameras, so a finalist-2 arm on corrected cameras would differ from it in two things. That comparison would have to be fixed at step 4's go. **Dropped:** the cameras' background part stays in every result (section 4.6, line F1), and `kitchen`'s control goes with it |
| 2. The alignment-tolerant pixel loss | **Keep, with its named measure changed before step 4** to the near-side far-5 cm share, read as for finalists 1 and 3. The under-two-frames share is reported but is not its criterion | Check (a) found its named measure, the near-side under-two-frames share, to be the method's habit on moving content. Section 7.2's adopt rule rests on that measure, so as it stands, its moving would not show a disagreement removed. Section 2.1 here reads finalists 1 and 2 as keeping their reason through the far share and the sheet. It is still the one finalist that acts on the person's part, which check (c) left untouched | **Kept as it stands:** its result could speak against H1, but a fall in its measure could not be read as removal. **Dropped:** no finalist acts on the person's part (section 4.6, line F2), and rows E11, E22 and E27, which come back beside it, lose their anchor. Changing a named measure is the owner's (question 6) |
| 3. Surface terms without a prior | **Keep** | Check (f) passed, so its adopted form can rest on a depth channel of the colour pass. Its named measure stands in S, and its openftgs-synth criterion, the share behind the surface, is untouched by (a) | **Dropped:** a result of finalist 4 could not be split between the constraint and the prior (section 4.6, line F3). With the ruler left as it is, a fall in its measure could be flattening's habit and not H1's outcome |
| 4. A normal prior | **Keep, decided after finalist 3** (section 4.6, line F4) | Check (f) bears on it as on finalist 3. No check of step 1 touches its prior | **Dropped:** H1 as the owner worded it, with prior maps, is not tested |

**The ask for step 2.** The owner's go for section 3 here: the baseline and the script's parity runs, **2.5 GPU hours** as section 4.5 counts them, more if a run on private scene A takes twice the notes' median (section 1 here).

- **The runs** are section 5.2's seven, by plain `ftgs train` at the default training configuration and 30,000 iterations:
  - on private scene A, the centre copy at seeds 0 and 1 and the left copy at seed 0, from a stride-4 init on the manifest's cameras;
  - on openftgs-synth, `kitchen` at seeds 0, 1 and 2 and `studio` at seed 0, from M17's init files.

  Then two parity runs of the wrapper script with every added weight at 0, one on private scene A and one on `kitchen`, held to section 4.5's tolerance.
- **What the baseline records, after checks (a) and (b):**
  - section 4.4's measures, with the near-side far share at 5 cm, 10 cm and 25 cm;
  - on openftgs-synth, the dynamic side at 0.12·S and 25 cm;
  - the under-two-frames share, reported apart, for the near side and for the whole model;
  - the sharpness ratio, the tiles' coverage and the test-view scores.

  All of these are CPU work on the runs' checkpoints. The one fixed reference init points file, against which every arm's far share is taken (section 5.2), is fixed at step 2: proposed, the baseline's own stride-4 init of each copy.
- **Due at step 2 from the owner:** the sharpness ratio's definition (section 1 here, item 1). The owner points to the notes' scorer in session and this document writes it down. Its spread between seeds comes from the baseline's two seeds on the centre copy, so it is needed before the baseline is scored, not before it trains.
- **What blocks step 2:**
  - Nothing among the questions below blocks its training. Question 5 decides what its scoring reports, and recording every figure listed above keeps both answers open.
  - Questions 1 and 6 block later steps: step 3 if finalist 1 is kept, and step 4.
  - Questions 2 to 4 block no step.

**The questions for the owner.**

1. **Finalist 1's pairs (from section 2.2 here; only if finalist 1 is kept).** The corrections break D-22's exact 15.0° ties. A corrected manifest's init would then take `cam2`'s person from `cam2`–`cam8`, where D-22's filters keep 0.8 % to 2.1 % of it, so finalist 1's init would differ from the baseline's in its pairs as well as its cameras.
   - (a) Accept the new pairing as part of finalist 1.
   - (b) Keep the manifest's pairs for finalist 1's inits. This needs a scratch init or a pair override, since `ftgs init roma` unchanged cannot do it.
   - (c) Both, reporting the pairing's effect apart.

   **Recommended: (b),** with (c) as a side run if the hours allow (ticket #19's recommendation). Under (a), a result could not be split between the cameras and the pairs.
2. **The fourth model's far share against its own init (from section 2.1 here).** It stays empty, because the regenerated init gave 0.23559 against the score file's 0.23426.
   - (a) Leave it empty.
   - (b) The owner supplies the original file, if it is kept anywhere.
   - (c) Accept the regenerated file as a stand-in, labelled so.

   **Recommended: (a),** or (b) if the file exists. Nothing depends on the figure, and (c) would break the owner's condition that it reproduce.
3. **Four marks in section 1 here.** The gates, no combined arm, the order, and the device's terms outside E10's g are marked "Confirmed" because review 2 accepted section 5's plan.
   - (a) Keep "Confirmed".
   - (b) Relabel them "Default".

   **Recommended: (a).** Review 2 accepted the plan that states them (section 7.7). Under (b) they hold all the same until the owner speaks.
4. **Check (f)'s "passed" (from section 2.4 here).** One figure at the cut 0.1 lies outside U-12's bounds, but it is one float32 computation held against another. Against a float64 device, the channel and both float32 devices lie within the bounds, and ticket #21 attributes the figure to float32 rounding.
   - (a) Confirm "passed".
   - (b) "Passed at Q-227's cut": a U-12 line for an adopted depth output is then held at that cut.
   - (c) Not passed: the adopted form keeps a second pass (section 5.1).

   **Recommended: (a).** All three float32 gradients lie within bounds against float64. The tests go through the device either way, so no step waits on it.
5. **The far share's ruler (from section 2.1 here, check (a)).**
   - (a) Keep the near-side far-5 cm share as the named measure of finalists 1, 3 and 4, and report the share beyond 25 cm beside it, as above.
   - (b) Make the share beyond 25 cm the named measure, with a margin set from the baseline's seeds.
   - (c) Leave the criteria as they are.

   **Recommended: (a).** (b) rests on three checkpoints and an unmeasured spread. Under (c), a fall in the 5 to 25 cm band could not be told from a change in the method's habit. It decides what step 2's scoring reports.
6. **Finalist 2's named measure (from section 2.1 here, check (a)).**
   - (a) Name the near-side far-5 cm share, with the tail beside it as in question 5, before step 4, keeping the under-two-frames share as a reported figure.
   - (b) Keep the under-two-frames share, with the reading limited to "speaks against H1, cannot show removal".
   - (c) Drop finalist 2.

   **Recommended: (a),** on section 2.1 here's reading that finalists 1 and 2 keep their reason through the far share and the sheet. It blocks step 4.

**The owner's answer on step 1** (each finalist kept or dropped, and the go for step 2 or a stop), given in session:
- **Finalist 1: kept, with both pairings** (question 1, option (c)). Step 3 runs the corrected cameras with the manifest's pairs and, beside it, with the pairing the corrections give, and reports the pairing's effect apart from the correction's.
- **Finalists 2, 3 and 4: kept.** Finalist 2's named measure is the near-side far-5 cm share, with the tail beside it, and the under-two-frames share stays a reported figure (question 6, option (a)).
- **The ruler** (question 5): option (a). The near-side far-5 cm share stays the named measure of finalists 1, 3 and 4, and the share beyond 25 cm is reported beside it, with its reading rule fixed from the baseline's seeds.
- **Questions 2, 3 and 4:** the recommendations stand, since the owner gave no other answer: the fourth model's figure stays empty, the four marks stay "Confirmed", and check (f)'s "passed" stands.
- **Step 2: go.** The sharpness ratio's scorer is still to be pointed to in session, before the baseline is scored.

**Date:** 2026-11-16.

## 3. Step 2: the baseline, the wrapper script and its parity runs

*Ticket #24's section.* The seven baseline runs by plain `ftgs train`, each with its measures, its sheet or `scripts/gt_depth.py`; and the scratch script's runs with every added weight at 0 against a plain run, on private scene A and on `kitchen`, within openftgs issue 395's tolerance.

Run on 2026-11-16, at the owner's go of that day (section 2.6 here), with this branch's code at 163bd39 (`src/` and `scripts/` as `main`'s), Python 3.14.10, torch 2.15.0+cu132, gsplat 1.5.3, NumPy 2.5.4, SciPy 1.18.1 and Pillow 13.0.0 from the shared `.venv`. The hardware was one RTX 4090 (driver 595.99.02), alone on the machine, under the shared GPU lock, with 16 CPU threads. Scripts read tensors, JSON, images and mask files and wrote numbers and files. No image, render or sheet of private scene A was looked at. Every existing frame-cache file beside the scenes' manifests passed the package's own validity check before training, so the runs read them and wrote nothing beside a manifest. A search afterwards found no file under `<private-scene-A-exp>`, `<m15-runs>/kitchen`, or `<m17-runs>`'s `kitchen`, `studio` and `reference` newer than the first run. In all, about 4.3 GPU hours: the eleven full runs took 2.9, the parity control's 35 short runs about 1.3, and the renders, scores and diagnostics the rest. "The commands", at the end, lists every run.

**The owner's answers during this step** (2026-11-16, in session, relayed by the orchestrating session):

- **The sharpness ratio** (section 1 here, item 1). It is the notes' scorer's `crop.sharp`, with its `crop_late.sharp` and `train.sharp` reported beside it (row 854). The owner named it after a summary of what the scorer computes; this section writes the definition down, below.
- **Parity.** The PRD's criterion cannot be met by two plain runs, so it is replaced by the form of (a) below, judged with wrapper v2. When v2 failed that form on both scenes, the owner chose three diagnostics on `kitchen`: plain runs from the wrapper's own entry script with the wrappers not installed, four more plain repeats, and plain and v2 runs with `perf.frame_cache` set explicitly. Then, having seen them, the owner asked for four more plain and three more v2 repeats on the centre copy, and gave the rule for accepting v2: v2 is accepted if, on the centre copy's 8 plain runs, it is indistinguishable from plain as it is on `kitchen`, by the same comparison as the diagnostics (each arm's shift in plain standard deviations and a two-sided Mann–Whitney test, on the 1–1,000 means of L1, ꟻLIP and total). The reading of "indistinguishable" as p ≥ 0.05 is this section's, put to the owner by the orchestrating session.
- **Two more full runs on the centre copy** at seed 0, one through wrapper v2 and one plain, measured like the baseline, to tell the wrapper's effect on the named measure from its spread. The GPU time beyond the 2.5 hours was cleared for them and for the short repeats. Whether finalists 2 to 4 are compared with an off arm through the script (question 3) is decided at step 3's go.

**The sharpness ratio, defined.** It comes from the notes' scorer, and the scorer was not run. A scratch script `score_a.py` computes it anew, and its figures equal the score files the notes keep beside `c3_s4` and `h1_init_s4` in every printed digit (row 858).

- For each of the 120 frames of the held-out camera, take the render clamped to [0, 1] and the ground-truth image, both cut to the person's crop (`cam0`'s rectangle (96, 256, 640, 1280) on the centre copy, `cam1`'s (0, 256, 640, 1280) on the left copy; section 2.5).
- Turn each into luma, Y = 0.299 R + 0.587 G + 0.114 B. Take the forward differences across and down, cut to a common grid one pixel smaller each way. The image's gradient energy G is the mean over pixels of √(Δx² + Δy²).
- The frame's ratio is G(render) / G(ground truth): 1 means as sharp as the source, and below 1 softer. **The sharpness ratio** is its mean over the 120 frames. Beside it, `crop_late` is the mean over frames 80 to 119, and `train` the same ratio on whole frames of the training cameras in the notes' request: `cam1` and `cam2` on the centre copy, `cam0` and `cam2` on the left copy, at frames 0, 30, 60, 90 and 119.
- The notes give its seed noise on the centre copy as ±0.003.

**The wrapper script** (`<scratch>/ticket-24/wrap_train.py`, not committed) is the seam of section 1 here.

- **How it runs.** It calls `openftgs.cli.main(["train", …])` in its own process, so the seed, `train_log.jsonl`, the checkpoint and METRICS.json are `ftgs train`'s own. Before that call it puts two wrappers in place of the names `render` and `total_loss` in `openftgs.train`.
- **The `render` wrapper** calls the package's `render` unchanged and keeps the iteration's camera and time for Q-226's device.
- **The `total_loss` wrapper** calls the package's `total_loss` unchanged. It checks that its time is the render's, then adds weight × term for every registered term whose weight is not 0. With every weight at 0 it returns the package's own `(loss, terms)` objects with no key added.
- **The check that the patch took effect.** Both wrappers count their calls, and the script fails unless each count equals the iterations logged: 30,000 in every full run.
- **The two versions.** Version 1 (kept as `wrap_train_v1.py`) kept the iteration's render output until the next render. Version 2, used from the control on, drops it as soon as the loss is taken, so the wrapper keeps nothing alive that `Trainer.step` would have freed. A `--no-patch` switch runs the same script with no wrapper installed, for diagnostic (ii).

**Parity: the PRD's criterion, and why no two runs meet it.** The criterion has three parts (section 4.5; openftgs issue 395):

- the mean losses over iterations 1 to 1,000 equal to four figures;
- N at iteration 500 within 0.01 %;
- the cap reached at the same iteration.

"Four figures" is read here as a difference under one unit of the fourth significant figure. That is the reading issue 395's own example fits (L1 0.11465 against 0.11460); the rounded form was checked beside it, and every result below is the same under both. The plain side of each pair is the baseline's seed-0 run on that scene, trained by plain `ftgs train`.

| Scene, seed 0 | L1, 1–1,000 | 1 − SSIM | ꟻLIP | Total | N at 500 | Cap reached at | Final E16 PSNR / SSIM₁ on the test view |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `kitchen`, plain | 0.114396 | 0.416321 | 0.415922 | 0.347409 | 401,567 | 2,100 | 29.25 dB / 0.8787 |
| `kitchen`, wrapper v1 | 0.114580 | 0.416406 | 0.416155 | 0.347670 | 401,572 | 2,100 | 27.68 dB / 0.8794 |
| Centre copy, plain | 0.072663 | 0.250741 | 0.289568 | 0.228677 | 127,684 | 10,800 | 24.79 dB / 0.8187 |
| Centre copy, wrapper v1 | 0.073090 | 0.251104 | 0.290836 | 0.229615 | 127,680 | 10,800 | 24.78 dB / 0.8187 |

- **Both pairs miss "four figures" and meet the other two parts.** On `kitchen` L1 differs by 1.6·10⁻³ of its value, and on the centre copy by 5.9·10⁻³.
- **The rows before the first growth event agree.** The rows at iterations 100 to 500 agree within 4·10⁻⁶ of their values on `kitchen`, and 6·10⁻⁵ at most on the centre copy. The runs part after the event at the end of iteration 500, where N differs by 5 and 4 primitives.
- **Two plain runs miss by as much.** M17's `default-395`, another plain run of the same configuration (at openftgs's eaeb791), misses today's plain `kitchen` run by 2.2·10⁻³ in L1. Three plain repeats stopped after iteration 2,200 miss it too.
- **The control's figures.** Over 8 plain runs of `kitchen` (the baseline's and 7 repeats), L1's mean over iterations 1 to 1,000 runs from 0.114345 to 0.114576. None of the 28 pairs among the first 8 runs examined (4 plain, 4 v1) agrees to four figures, and 6 of those pairs are plain against plain. The cap falls at 2,000 in 2 of the 8 plain runs and at 2,100 in the other 6: in the plain runs that have not reached it by 2,000, N stands at 2,984,252 to 2,999,631 there.
- **The reason.** gsplat's backward is not bit-repeatable (Q-220), and growth's choices amplify the difference from the first event on. The final `kitchen` PSNRs, 29.25 against 27.68 dB, are two seed-0 runs, the second through wrapper v1, whose losses the diagnostics below find shifted. The gap lies inside the spread #374 recorded, a standard deviation of 0.55 to 1.54 dB (section 4.4), with SSIM₁ 0.0007 apart. Today's plain run and M17's `default-395` (28.32 dB) are 0.93 dB apart.

**Parity (a), the owner's replacement** (answered 2026-11-16, in session):

1. the rows before the first growth event (iterations 100 to 500) equal to five figures, read as a difference under one unit of the fifth significant figure;
2. every term's mean over iterations 1 to 1,000 inside the plain runs' range on that scene;
3. N at iteration 500 within 0.01 %;
4. the cap within one growth event, 100 iterations, where a run reaches it.

The plain ranges are 8 runs on `kitchen`, stopped after 2,200, and 4 on the centre copy: the baseline's run and 3 repeats stopped after 1,100, which do not reach the cap.

| Arm | Runs | Part 1 | Part 2 | Part 3 | Part 4 | (a) |
| --- | --- | --- | --- | --- | --- | --- |
| `kitchen`, v2 | 3 | 3 of 3 | 2 of 3 (r3 above the range in L1 0.114608, ꟻLIP 0.416342 and total 0.347771) | 3 of 3 | 3 of 3 | **fails**, 2 of 3 runs pass |
| Centre copy, v2 | 3 | 1 of 3 (row gaps up to 6.8·10⁻⁵) | 0 of 3 (all above the range in L1, 0.072964 to 0.073038 against 0.072663 to 0.072902) | 3 of 3 | not reached | **fails** |
| `kitchen`, each plain run against the other 7 | 8 | 8 of 8 | 3 of 8 | 8 of 8 | 8 of 8 | 3 of 8 pass |
| Centre copy, each plain run against the other 3 | 4 | 2 of 4 (gaps up to 5.2·10⁻⁵) | 0 of 4 | 4 of 4 | not reached | 0 of 4 pass |

**v2 fails (a) on both scenes, and plain runs fail it against each other as often.** A run drawn like the others falls outside the range of n others in a term with chance 2/(n + 1), and outside on any of five correlated terms more often. On the centre copy, part 1 fails between plain runs too. (a) was therefore not used to accept v2; the owner's rule below decides it.

**The diagnostics: which one finds the shift.** Each arm is a set of `kitchen` seed-0 runs stopped after 2,200. The table gives the arm's mean, over its runs, of the 1–1,000 loss means, against the 8 plain runs in units of their standard deviation, with a two-sided Mann–Whitney test against them (SciPy):

| Arm | Runs | L1 | 1 − SSIM | ꟻLIP | Total | Cap at |
| --- | --- | --- | --- | --- | --- | --- |
| Plain `ftgs train` | 8 | 0.114475 | 0.416362 | 0.416009 | 0.347516 | 2,100 (6), 2,000 (2) |
| (ii) The wrapper's script, nothing installed | 3 | +0.49 SD, p = 0.50 | +0.78, 0.38 | +0.16, 0.78 | +0.39, 0.63 | 2,000 (3) |
| Wrapper v1 | 4 | +1.33 SD, p = 0.004 | +0.69, 0.15 | +1.78, 0.03 | +1.52, 0.02 | 2,100 (1), 2,000 (3) |
| Wrapper v2 | 3 | +0.95 SD, p = 0.19 | +0.60, 0.28 | +1.14, 0.28 | +1.03, 0.28 | 2,100 (2), 2,000 (1) |
| (iii) Plain, `perf.frame_cache` "mmap" | 3 | +0.59 SD, p = 0.28 | +0.40, 0.63 | +0.65, 0.50 | +0.62, 0.38 | 2,100 (3) |
| (iii) v2, `perf.frame_cache` "mmap" | 3 | −0.61 SD, p = 0.63 | −0.68, 0.38 | −0.70, 0.28 | −0.68, 0.38 | 2,100 (2), 2,000 (1) |

- **The one shift located is version 1's.** Keeping the previous render output alive moved the losses up: L1 by 1.3 standard deviations of the plain runs, p = 0.004.
- **What is not the cause, on `kitchen`.** The entry script, with nothing installed (ii), moves nothing distinguishable from noise. Diagnostic (iii) tested less than its name says. `kitchen`'s training frames take 21.8 GiB, more than half the card, so "auto" always resolves to "mmap" there (D-51's rule). Fixing "mmap" removed only the PNG-decoding iterations before the cache opens, and with that, v2 lies below the plain mean. The placement itself can differ only where the cache may fit, as on the centre copy (2.5 GiB), and was not tested there.
- **Version 2 on `kitchen`** is not distinguishable from plain runs on these samples.
- **On the centre copy, first count.** The three v2 repeats lay above all four plain runs then in L1, ꟻLIP and total, 1.7 to 2.2 plain standard deviations high. That is complete separation; p = 0.057 is the smallest value a test of 3 runs against 4 can give.
- **On the centre copy, at the owner's 8 plain runs** (the baseline's and 7 repeats to 1,100) **and 6 v2 runs:**

  | Arm | Runs | L1 | ꟻLIP | Total |
  | --- | --- | --- | --- | --- |
  | Plain `ftgs train` | 8 | 0.072858 (0.072644 to 0.073186) | 0.290293 (0.289562 to 0.291249) | 0.229195 (0.228637 to 0.229972) |
  | Wrapper v2 | 6 | 0.072985 (0.072912 to 0.073080): +0.70 SD, p = 0.08 | 0.290653 (0.290310 to 0.290918): +0.60 SD, p = 0.18 | 0.229479 (0.229252 to 0.229683): +0.62 SD, p = 0.14 |

  On `kitchen` v2 lay +0.95 to +1.14 SD high, p = 0.19 to 0.28. **By the owner's rule, v2 is accepted:** on both scenes no test reaches p < 0.05, and the shifts are under 1.2 plain standard deviations. "Indistinguishable" is read here as p ≥ 0.05 in each of the three terms; the rule names no threshold, so that reading is this section's. Two facts sit beside the acceptance. All six v2 runs lie above the plain median in L1, and L1's p = 0.08 is close to the threshold. No v2 run went to 30,000 iterations on either scene, and no diagnostic ran on the centre copy.
- **The final scores of the full wrapper runs there** (v1, and v2 at the owner's answer) differ from plain about as much as two seeds do. E16 PSNR on `cam0` is 24.78 and 24.80 dB against 24.75 to 24.79, the crop's PSNR 24.36 and 24.35 against 24.31 to 24.36, the sharpness ratio 0.732 and 0.731 against 0.725 to 0.730. The near-side far-5 cm share is not: 0.126 and 0.131 against 0.140 to 0.149 (below, and question 3).

**The reference init points files, fixed for every later arm** (section 4.4's line on the far-5 cm share). The files are the copies' kept stride-4 inits from `<private-scene-A-exp>`, D-21's stride, which `c3_s4` and `h1_init_s4` trained from (row 856). Copied unchanged to `<surface-runs>/ticket-24/init/`, they are the files the baseline trained from:

| Copy | File | Points | SHA-256 |
| --- | --- | --- | --- |
| Centre camera held out | `ref_h0_s4.safetensors` | 120,000 | `dc61ffbe…1522c3e3` |
| Camera left of it held out | `ref_h1_s4.safetensors` | 120,000 | `0c7528c7…c5d51eb5` |

**They do not reproduce at this commit.** `ftgs init roma --stride 4` on the centre copy (with ticket #18's scratch folder for the `[init]` extra, row 849) gave 120,000 points with the same frames, times and durations but other positions: its points lie 8.4 mm from the kept file's at the median, about the points' own spacing. Measured against it rather than the kept file, `c3_s4`'s near-side far-5 cm share is 0.112 instead of 0.124, a change beyond section 4.4's margin of 0.01. Its share beyond 25 cm is 0.0436 against 0.0435. So every later arm is measured against these two files, never against an init of its own, and a finalist-1 arm initialised by current code differs from the baseline in its init's sample as well as its cameras (question 2 below).

**Private scene A: the shell's measures**, taken as section 2.1 here takes them (`measures_a.py`, after ticket #18's `split80.py`), against the copy's reference file. Each pair of cells is whole model / near side. The near side holds 78 % to 79 % of the opacity in every run. The scripts reproduce section 2.6's figures for `c3_s4` (0.213 / 0.124 at 5 cm) against its own file.

| Run | N | Far share, 5 cm | 10 cm | 25 cm | Under two frames | Median duration, frames | Mean σ | σ < 0.05 | σ > 0.5 | Far side: far-5 cm share | Cap at |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `A_h0_s0`, centre copy, seed 0 | 3,000,000 | 0.224 / 0.140 | 0.131 / 0.063 | 0.072 / 0.035 | 0.452 / 0.586 | 2.27 / 1.69 | 0.356 / 0.371 | 0.303 | 0.312 | 0.535 | 10,800 |
| `A_h0_s1`, centre copy, seed 1 | 3,000,000 | 0.229 / 0.141 | 0.132 / 0.058 | 0.073 / 0.027 | 0.459 / 0.594 | 2.23 / 1.66 | 0.358 / 0.374 | 0.301 | 0.315 | 0.541 | 11,100 |
| `A_h1_s0`, left copy, seed 0 | 3,000,000 | 0.246 / 0.163 | 0.144 / 0.075 | 0.085 / 0.044 | 0.449 / 0.585 | 2.29 / 1.69 | 0.349 / 0.365 | 0.309 | 0.306 | 0.541 | 9,700 |
| Centre copy, seed 0, wrapper v1 (the parity run; not a baseline run) | 3,000,000 | 0.214 / 0.126 | 0.135 / 0.066 | 0.081 / 0.040 | 0.444 / 0.577 | 2.31 / 1.72 | 0.353 / 0.370 | 0.304 | 0.309 | 0.533 | 10,800 |
| Centre copy, seed 0, wrapper v2 (at the owner's answer; not a baseline run) | 3,000,000 | 0.218 / 0.131 | 0.141 / 0.075 | 0.081 / 0.046 | 0.454 / 0.586 | 2.26 / 1.69 | 0.358 / 0.374 | 0.302 | 0.315 | 0.537 | 10,900 |
| Centre copy, seed 0, plain again (at the owner's answer; not a baseline run) | 3,000,000 | 0.233 / 0.148 | 0.147 / 0.081 | 0.085 / 0.047 | 0.450 / 0.579 | 2.29 / 1.71 | 0.359 / 0.375 | 0.302 | 0.315 | 0.537 | 10,700 |

**Private scene A: the test-view scores, the sharpness ratio and the tiles' coverage.**

- **Crop** is the notes' scorer's: PSNR, SSIM₁ and ꟻLIP on the person's crop, by `openftgs.eval.Evaluator`, averaged over the 120 frames.
- **E16** is METRICS.json's, over the whole test view.
- **Kept-pixel PSNR** is Q-233's, on the centre copy alone, whose test view is `cam0`. It leaves out the placeholder pixels by section 2.5's loose rule, rebuilt from `cam0`'s images; the rule equals ticket #4's stored mask on every pixel of the 120 frames and finds 0.300 % of them (0.507 % of the crop's).
- **Coverage** is each sheet tile's share of pixels with alpha ≥ 0.5, from section 2.6's fixed request `<surface-runs>/ticket-5/side_request.json`. The sheets are `<surface-runs>/ticket-24/sheets/side_<run>.jpg`, with their coverage files beside them. Every rendered value is finite.

| Run | Crop PSNR | Crop SSIM₁ | Crop ꟻLIP | Sharpness ratio | Late | Training cameras: PSNR / ratio | E16 PSNR / SSIM₁ / ꟻLIP | Kept-pixel PSNR | Coverage: between cameras, `sC00`, ±30°, ±45° | Coverage: ±60°, ±90° |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `A_h0_s0` | 24.36 | 0.8149 | 0.1656 | 0.7262 | 0.7504 | 32.90 / 0.7755 | 24.79 / 0.8187 / 0.1531 | 25.52 | 0.84 to 1.00 | 0.50 to 0.90 |
| `A_h0_s1` | 24.31 | 0.8153 | 0.1668 | 0.7249 | 0.7467 | 32.79 / 0.7742 | 24.75 / 0.8189 / 0.1545 | 25.47 | 0.80 to 1.00 | 0.50 to 1.00 |
| `A_h1_s0` | 23.90 | 0.7473 | 0.1790 | 0.7403 | 0.7938 | 31.31 / 0.7719 | 21.92 / 0.7392 / 0.1951 | (`cam1`: not applied) | 0.78 to 1.00 | 0.55 to 1.00 |
| Centre copy, seed 0, wrapper v1 | 24.36 | 0.8157 | 0.1656 | 0.7319 | 0.7540 | 33.19 / 0.7773 | 24.78 / 0.8187 / 0.1529 | 25.53 | 0.78 to 1.00 | 0.42 to 0.87 |
| Centre copy, seed 0, wrapper v2 | 24.35 | 0.8148 | 0.1658 | 0.7313 | 0.7536 | 32.99 / 0.7733 | 24.80 / 0.8180 / 0.1531 | 25.54 | 0.86 to 1.00 | 0.53 to 1.00 |
| Centre copy, seed 0, plain again | 24.32 | 0.8155 | 0.1658 | 0.7303 | 0.7544 | 33.05 / 0.7743 | 24.75 / 0.8183 / 0.1539 | 25.48 | 0.80 to 1.00 | 0.49 to 0.88 |

The notes' runs closest to these were trained by openftgs 0.1.0 with LPIPS, before revision 549o. `c3_s4` (the same copy, init and configuration then) scores 24.40 dB on the crop with a sharpness ratio of 0.7235, and its near-side far-5 cm share is 0.124 against 0.140 and 0.141 here.

**The spread between the centre copy's two seeds,** for every near-side measure and the sharpness ratio (section 4.4). The last column is the wrapper's seed-0 run: a third draw of the same seed. It is not a baseline run, and its wrapper is version 1, whose losses ran slightly high.

| Measure | Seed 0 | Seed 1 | Seeds 0 and 1 differ by | The notes' seed noise | Seed 0 under wrapper v1 |
| --- | --- | --- | --- | --- | --- |
| Near side: far share at 5 cm (the named measure) | 0.1401 | 0.1409 | 0.0008 | 0.003 (section 2.6, one pair) | 0.1257 |
| Near side: far share at 10 cm | 0.0634 | 0.0583 | 0.0051 | | 0.0660 |
| Near side: far share beyond 25 cm (the tail) | 0.0349 | 0.0267 | 0.0082 | | 0.0404 |
| Near side: under two frames | 0.5856 | 0.5945 | 0.0089 | | 0.5772 |
| Whole model: far share at 5 cm | 0.224 | 0.229 | 0.004 | 0.001 to 0.010 (section 2.6) | 0.214 |
| Whole model: under two frames | 0.452 | 0.459 | 0.007 | 0.002 to 0.004 | 0.444 |
| Sharpness ratio | 0.7262 | 0.7249 | 0.0013 | ±0.003 | 0.7319 |
| Late sharpness ratio | 0.7504 | 0.7467 | 0.0037 | | 0.7540 |
| Training cameras' sharpness ratio | 0.7755 | 0.7742 | 0.0013 | | 0.7773 |
| Crop PSNR | 24.358 dB | 24.305 dB | 0.053 dB | ±0.01 dB | 24.357 dB |

**The two full runs at the owner's answer.** They are a second plain draw of seed 0 and a full v2 run of seed 0; v2's wrappers ran 30,000 times each, and its cap fell at 10,900 against 10,700 and 10,800 for the plain seed-0 runs. With them the centre copy has three plain draws (seed 0 twice, seed 1) and two through the script (v1, v2):

| Measure | Plain draws | Their range | Through the script, v1 / v2 | Where the script's runs lie |
| --- | --- | --- | --- | --- |
| Near side: far share at 5 cm (the named measure) | 0.1401, 0.1409, 0.1485 | 0.0084 | 0.1257 / 0.1308 | below all three plain draws, by 0.009 and more |
| Near side: far share at 10 cm | 0.0634, 0.0583, 0.0811 | 0.023 | 0.0660 / 0.0754 | among them |
| Near side: far share beyond 25 cm (the tail) | 0.0349, 0.0267, 0.0475 | 0.021 | 0.0404 / 0.0461 | among them |
| Near side: under two frames | 0.5856, 0.5945, 0.5789 | 0.016 | 0.5772 / 0.5857 | among them |
| Sharpness ratio | 0.7262, 0.7249, 0.7303 | 0.0054 | 0.7319 / 0.7313 | above all three, by 0.001 |
| Crop PSNR | 24.358, 24.305, 24.324 dB | 0.053 dB | 24.357 / 24.352 dB | among them |

What the spread shows:

- **The named measure's spread is wider than two seeds say.** Seeds 0 and 1 agree within 0.0008, but the second draw of seed 0 lies 0.008 above them.
- **The two runs through the script both lie below every plain draw** in the near-side far-5 cm share: 0.126 and 0.131 against 0.140 to 0.149. Their mean is 0.128 against the two plain seeds' 0.1405, 0.012 lower. By section 4.4's rule, both of an arm's seeds beyond both of the baseline's and the means more than 0.01 apart, the script with every weight at 0 would count as having lowered the named measure against the plain baseline. On these few runs (two against three), that cannot be told from chance. Mann–Whitney's smallest p at 2 against 3 is 0.2. It is the measure each finalist through the script names, though, so question 3's off arm through the script (v2 at weight 0) is the comparison that removes it.
- **The other near-side measures** show no such split; the tail, the 10 cm share and the under-two-frames share of the script's runs lie among the plain draws.
- **The tail's spread** over the three plain draws is 0.021, more than half its level (0.027 to 0.048). The seeds' 0.008 understates it.
- **The sharpness ratio's spread** is 0.0054 over the three plain draws, 0.0013 between the seeds, against the notes' ±0.003. The two script runs lie 0.001 above the highest plain draw. The crop PSNR's spread, 0.053 dB, is five times the notes' ±0.01 dB, though still far inside section 4.4's 0.3 dB guard.

**The tail's reading rule, proposed and not fixed.** Section 2.6 here asks it to be fixed from these seeds before any arm runs. Three options, put to the owner (question 1 below):

- (a) the tail shares a fall when both of the arm's seeds lie below both of the baseline's (below 0.0267), with no margin;
- (b) as (a), and the means differ by more than the plain draws' range, 0.021;
- (c) section 4.4's rule unchanged, a margin of 0.01.

Recommended: (b). Its margin is the tail's own spread over three draws of the baseline, where (c)'s is the far-5 cm share's and the seeds' 0.008 understates it. Under (b) the tail can show only a large fall, about half its level; a fall in the 5 cm share that the tail does not share then reads as section 2.6 here says, as a change in the method's habit.

**openftgs-synth.** The runs start from M17's init files (`<m17-runs>/kitchen/init.safetensors` on M15's conversion of `kitchen`, `<m17-runs>/studio/init.safetensors`).

- **E16** is METRICS.json's on the test view (`cam00` × 300, and `0007` × 60).
- **Kept-pixel PSNR** is `scripts/reference_flaws.py`'s `overall.union.kept_psnr` against #329's flags, as M17 takes it.
- **The residual's shares** are `scripts/gt_depth.py`'s in front of and behind the surface at δ = 0.02·S, by σ_eff, on all pixels and on dynamic pixels.
- **The error of expected depth** is the median of its absolute value.
- **The far shares and the under-two-frames share** are section 2.1 here's, by ticket #18's `synth_measures.py` copied unchanged, against the scene's init file. They are given whole model / dynamic side by the majority rule, at 0.12·S and beyond 25 cm, with two frames at 2/fps.

| Run | E16 PSNR / SSIM₁ / ꟻLIP | Kept-pixel PSNR | Residual, all pixels: in front / behind | Dynamic pixels: in front / behind | Expected depth, median error, all / dynamic | Far share at 0.12·S | Beyond 25 cm | Under two frames | Cap at |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `kitchen_s0` | 29.25 / 0.8787 / 0.0881 | 30.42 | 0.041 / 0.291 | 0.015 / 0.613 | 252 / 57 mm | 0.036 / 0.002 | 0.023 / 0.001 | 0.381 / 0.660 | 2,100 |
| `kitchen_s1` | 30.11 / 0.8803 / 0.0856 | 31.34 | 0.039 / 0.297 | 0.015 / 0.612 | 448 / 304 mm | 0.035 / 0.003 | 0.022 / 0.001 | 0.377 / 0.662 | 2,100 |
| `kitchen_s2` | 29.25 / 0.8800 / 0.0885 | 31.15 | 0.040 / 0.295 | 0.015 / 0.618 | 277 / 103 mm | 0.036 / 0.002 | 0.023 / 0.001 | 0.385 / 0.661 | 2,100 |
| `studio_s0` | 22.24 / 0.7991 / 0.1928; dynamic 24.85 / 0.9219 / 0.1290 | 22.30 | 0.014 / 0.412 | 0.007 / 0.647 | 144 / 87 mm | 0.041 / 0.001 | 0.068 / 0.001 | 0.387 / 0.710 | 6,800 |
| `kitchen`, seed 0, wrapper v1 (not a baseline run) | 27.68 / 0.8794 / 0.0907 | 30.97 | 0.041 / 0.296 | 0.015 / 0.620 | 205 / 51 mm | 0.036 / 0.002 | 0.024 / 0.001 | 0.381 / 0.660 | 2,100 |

What these runs give the criteria (section 4.4):

- **SSIM₁ over the three seeds** runs from 0.8787 to 0.8803, inside #374's at most 0.0023.
- **PSNR** runs from 29.25 to 30.11 dB, and the kept-pixel PSNR from 30.42 to 31.34.
- **The residual's share behind the surface** runs from 0.291 to 0.297 over all pixels. Its spread, 0.006, is wider than section 4.4's 0.003 between trainings on `kitchen`, and wider than the 0.005 by which section 4.4 calls a share worse. In front the spread is 0.002, on dynamic pixels 0.006 behind and 0.0003 in front.
- **The median error of expected depth** runs from 252 to 448 mm, as unsteady as section 4.4 found it.
- **The dynamic side** is near 0 at 0.12·S and beyond 25 cm, as in section 2.1 here.
- **`studio`'s dynamic-region PSNR** is 24.85 dB, against 25.19 dB for M17's `default-395` (at openftgs's eaeb791): 0.34 dB apart. Section 4.4's flag on `studio` is a fall of more than 0.1 dB, and two trainings of the default configuration are already 0.34 dB apart.

**Against the ticket's acceptance criteria.**

- **Parity on both scenes.** Not met as the PRD wrote it, nor by the owner's first replacement (a), which plain runs meet no more reliably. Met by the owner's acceptance rule, on 8 plain runs per scene against 3 (`kitchen`) and 6 (centre copy) v2 runs, read at p ≥ 0.05 (above).
- **The seven baseline runs, each with its measures.** Done: the tables above, from the files under `<surface-runs>/ticket-24/`.
- **One reference init points file per copy.** Fixed and recorded above.
- **The sharpness ratio and its spread.** Defined above, at the owner's answer, with its spread between the centre copy's two seeds beside every near-side measure's.
- **Placeholder pixels.** Left out only where `cam0` is the test view, by the loose rule, checked against the stored mask first.
- **The owner's go or stop.** Below, not yet given.

**What is open, for the owner.**

1. **The tail's reading rule:** the options (a) to (c) above. It blocks reading finalists 1, 3 and 4's named measure.
2. **Finalist 1's init and the reference files.** Current code does not reproduce the kept stride-4 file, and a fresh sample alone moves `c3_s4`'s near-side far-5 cm share by 0.012. Options:
   - (a) also measure finalist 1 against a fresh init of the manifest's cameras, made by this commit, beside the kept file;
   - (b) measure against the kept file alone;
   - (c) also retrain the baseline from fresh inits.

   Recommended: (a). It is section 4.4's two-reference rule for finalist 1 with a fresh manifest-camera file, and costs one init.
3. **The off arm for finalists 2 to 4, decided at step 3's go** (the owner's answer of 2026-11-16). Both runs through the script, v1 and v2 at every weight 0, lie below all three plain draws in the named measure, by 0.009 and more (above). Options:
   - (a) compare finalists 2 to 4 with the plain baseline, as section 5.2 plans;
   - (b) compare them with an off arm through the script, v2 at weight 0 at two seeds, so that the script's own effect cancels out of the comparison (about 27 minutes more of GPU per scene set);
   - (c) both, reporting the difference.

   Recommended: (b). Under (a) the script alone moves the named measure by about section 4.4's margin. Finalist 1, which needs no script, is not affected.
4. **Two seeds and the named measure.** A second draw of seed 0 moved the near-side far-5 cm share by 0.008, and the tail by 0.013. Whether section 4.4's two seeds per arm stand is for the owner, and nothing here changes them.

**The commands.** Each ran with `OMP_NUM_THREADS=16 MKL_NUM_THREADS=16 OPENBLAS_NUM_THREADS=16`, `PYTHONPATH=<worktree>/src:<worktree>/harness/src` and the shared `.venv`'s Python, the CPU ones with `CUDA_VISIBLE_DEVICES=` empty and the GPU ones under the shared lock. Every timed training ran under `/usr/bin/time -v`, with `nvidia-smi` sampling memory once a second. `$T` is `<scratch>/ticket-24`, `$O` is `<surface-runs>/ticket-24`, `$X` is `<private-scene-A-exp>`, `$K` is `<m15-runs>/kitchen`, `$M` is `<m17-runs>`, `$G` is `$OPENFTGS_DATASETS/synth/ground_truth`, `$S` is `<surface-runs>/ticket-5/side_request.json`. The scripts are not committed.

| Command | Where | Time | Note |
| --- | --- | --- | --- |
| `ftgs train $K/scene.json --init $M/kitchen/init.safetensors --out $O/runs/kitchen_s<n> --seed <n>`, n = 0, 1, 2 | GPU | 16 min 51 s, 16 min 56 s, 16 min 51 s | The baseline on `kitchen`; peak 21,474 to 21,814 MiB |
| `ftgs train $M/studio/scene/scene.json --init $M/studio/init.safetensors --out $O/runs/studio_s0 --seed 0` | GPU | 23 min 45 s | The baseline on `studio`; peak 23,868 MiB |
| `ftgs train $X/scene_h<c>/scene.json --init $O/init/ref_h<c>_s4.safetensors --out $O/runs/A_h<c>_s<n> --seed <n>`, for (c, n) = (0, 0), (0, 1), (1, 0) | GPU | 13 min 9 s, 13 min 12 s, 13 min 24 s | The baseline on private scene A, the copies' manifests as `<private-scene-A-exp>` holds them; peak 23,394 to 23,982 MiB |
| The same for (0, 0) into `$O/runs/A_h0_s0_rep`, and `python $T/wrap_train.py --stats … -- <the same arguments>` into `$O/runs/A_h0_s0_v2` | GPU | 13 min 9 s, 13 min 29 s | The two full runs at the owner's answer, measured, scored and sheeted as the baseline |
| `python $T/wrap_train_v1.py --stats … -- <the same arguments>`, for `kitchen` and the centre copy at seed 0 | GPU | 17 min 20 s, 13 min 29 s | The parity runs, wrapper v1, every weight at 0, 30,000 calls of each wrapper |
| `bash $T/short_run.sh`-driven repeats of the same commands (`wrap_train.py` for v2 and `--no-patch`), stopped by SIGTERM once the log passed iteration 2,200 (`kitchen`) or 1,100 (the centre copy): 7 plain, 3 v1, 3 v2, 3 `--no-patch`, 3 plain and 3 v2 with `--config` holding `{"perf": {"frame_cache": "mmap"}}` on `kitchen`; 7 plain and 6 v2 on the centre copy | GPU | about 2.5 and 1.5 min each | The parity control and the diagnostics; each saved its checkpoint and exited 143 |
| `python $T/parity.py`, `python $T/parity2.py`, `python $T/arms.py` on the runs' `train_log.jsonl` | CPU | under 1 s each | The tables of parity, of (a) and of the diagnostics |
| `ftgs init roma $X/scene_h0/scene.json $O/init/regen_h0_s4.safetensors --stride 4`, with `PYTHONPATH` adding ticket #18's `env-init` | GPU | 1 min 23 s | The reproduction check: 120,000 points, other positions |
| `ftgs render <checkpoint> --request $S`; `ftgs render <checkpoint> --request $X/requests/heldout_novel.json` (the left copy: `heldout1_novel.json`) | GPU | seconds each | The sheet's 39 renders and the scorer's 142 |
| `python $T/score_a.py <renders> $X/scene_h<c>/scene.json --heldout cam0 --crop 96,256,640,1280 --stored-masks <surface-runs>/ticket-4/private-scene-A/placeholder_masks.npz --out $O/scores/<run>.json` (the left copy: `--heldout cam1 --crop 0,256,640,1280`) | GPU (the evaluator) | about 30 s | The test-view scores, the sharpness ratio and the kept-pixel PSNR; first run on `c3_s4` and `h1_init_s4`, equal to their score files |
| `python $T/measures_a.py $X/scene_h0/scene.json --out $O/measures/a_runs.json <run>=<checkpoint>=$O/init/ref_h<c>_s4.safetensors` | CPU | 2 to 4 s | The shell's measures; first on `c3_s4`, `h1_init_s4` and `h1_init_s4_seed1`, equal to section 2.6's |
| `python $T/side_sheet.py <renders> $S $O/sheets/side_<run>.jpg --label <run> --stats $O/sheets/side_<run>.coverage.json` | CPU | about 1 s | Ticket #5's sheet script, copied unchanged |
| `python scripts/gt_depth.py $O/runs/<run>/checkpoint <scene.json> $G/<scene> --out $O/gt_depth/<run>.json` | GPU | 60 to 65 s on `kitchen`, 24 s on `studio` | The geometry residual and expected depth |
| `python scripts/reference_flaws.py run $O/runs/<run>/checkpoint <scene.json> $M/reference/<scene>/flags --out $O/runs/<run>/REFERENCE_PSNR.json` | GPU | 20 to 21 s, 11 s | The kept-pixel PSNR |
| `python $T/synth_measures.py $O/runs/<run>/checkpoint <init> $G/<scene> --scene-name <scene> --out $O/measures/synth_<run>.json` | CPU | 42 to 45 s, 24 s | Section 2.1 here's measures, by ticket #18's script copied unchanged |
| `python scripts/run_summary.py $O/runs/<run> --time … --gpu-log …` | CPU | under 1 s | `SUMMARY.json` beside each run |

The private scene's folders, `<m15-runs>` and `<m17-runs>` were read and never written.

**The owner's answer on step 2** (the go for step 3 or a stop), given in session after seeing the two full runs through the script:
- **The off arm for finalists 2 to 4** (question 3): option (b). They are compared with an off arm through the script, v2 with every added weight at 0, at two seeds per copy of private scene A and on `kitchen` and `studio` as the baseline is, run once at step 4. The plain baseline stays the off arm of finalist 1, which needs no script.
- **The tail's reading rule** (question 1): option (b). The tail has moved when both of the arm's seeds lie beyond both of the off arm's and the means differ by more than the plain runs' range, 0.021.
- **Finalist 1's init** (question 2): option (a). Finalist 1 is measured against the kept stride-4 file and, beside it, against a fresh init from the manifest's cameras.
- **Two seeds per arm** (question 4): they stand, since the owner gave no other answer.
- **Step 3: go,** with finalist 1's optional control on `kitchen` (2 more GPU hours).

**Date:** 2026-11-16.

## 4. Step 3: finalist 1, the cameras corrected once

*Ticket #25's section.* Finalist 1's arms on the corrected cameras, run as the research document's section 5.3 says and judged by section 4.4's criteria.

Run on 2026-11-16 and 2026-11-17, at the owner's go of 2026-11-16 (section 3 here), with this branch's code at 5b0151a (`src/` and `scripts/` as `main`'s), Python 3.14.10, torch 2.15.0+cu132, gsplat 1.5.3, NumPy 2.5.4, SciPy 1.18.1 and Pillow 13.0.0 from the shared `.venv`, and, for the inits and the matcher, ticket #18's scratch packages (row 849). The hardware was one RTX 4090 (driver 595.99.02), alone on the machine, under the shared GPU lock, with 16 CPU threads. Scripts read tensors, JSON, images and mask files and wrote numbers and files. No image, render or sheet of private scene A was looked at. Every run wrote only under `<surface-runs>/ticket-25`: a search afterwards found no file under `<private-scene-A-exp>`, `<m15-runs>/kitchen` or `<m17-runs>/kitchen` newer than the first init. In all, about 4.2 GPU hours, from the first init to the last diagnostic: the six runs on private scene A with their inits, scores and renders about 1.8, the control on `kitchen` about 2.4. "The commands", at the end, lists every run.

**The owner's answers at step 3's go** (2026-11-16, in session, relayed by the orchestrating session; the ends of sections 2.6 and 3 here):

- **Both pairings** (section 2.6 here, question 1, option (c)). Arm `F1m` trains on the corrected cameras from an init made with the manifest's D-22 pairs; arm `F1c` from plain `ftgs init roma` on the corrected manifest, with the pairs the corrections give. The correction's effect is read from `F1m` against the baseline, the pairing's from `F1c` against `F1m`.
- **The off arm** is section 3 here's plain baseline, `A_h0_s0`, `A_h0_s1` and `A_h1_s0`, with no script.
- **The references** (section 3 here, question 2, option (a)): the named measure is taken against the kept stride-4 file and a fresh init of the manifest's cameras, beside the ticket's two files triangulated from the same matches with the manifest's cameras and with the corrected ones.
- **The tail beyond 25 cm** is reported beside the named measure and read by rule (b): both of the arm's seeds beyond both of the off arm's, and the means more than 0.021 apart.
- **Two seeds per arm** stand.
- **The optional control on `kitchen` is cleared,** with its 2 GPU hours.

**The manifests.** Ticket #19's corrected manifests `P/h0` and `P/h1` and their train-only forms (section 2.2 here), copied byte for byte into `<surface-runs>/ticket-25/manifests`, each with a link to its copy's images, so that the frame caches training writes beside a manifest land there (row 861).

**The inits: four per copy, one sample of matches per pairing.** Every init is `ftgs init roma --stride 4`, D-21's stride, as the baseline's: 120,000 points (row 862). A pair's drawn matches depend on its two images, the seed, the slot and the two cameras' names, not on the cameras' poses (Q-110). So two inits with one pairing and two sets of cameras triangulate the very same matches, and D-22's filters then keep different ones.

- **The scratch init** (`init_pairs.py`) runs `openftgs.cli.main(["init", "roma", …])` unchanged, with one private name of `openftgs.init` replaced at run time: `_pairs` returns the package's own pairing, computed on another manifest's cameras for the same admitted images. `_triangulated` is also wrapped, unchanged, to count the share D-22's filters keep per pair, which the written file no longer shows after the per-slot subset. No public function lets a caller name the pairs, so this is the seam of section 1 here applied to a private name, not the package's public API.
- **It was checked first.** On the centre copy's own manifest with its own pairing it writes a file equal tensor for tensor to ticket #24's `regen_h0_s4`, made by plain `ftgs init roma` (section 3 here). On each corrected manifest with its own pairing, its file equals plain `ftgs init roma`'s.
- **The four, per copy.** `MM`: the manifest's cameras and pairs, the fresh init of section 3 here's question 2. `CM`: the corrected cameras with the manifest's pairs, arm `F1m`'s init. `CC`: the corrected cameras with their own pairs, by plain `ftgs init roma`, arm `F1c`'s init. `MC`: the manifest's cameras with the corrected pairing. `MM` and `CM` are arm `F1m`'s two references from the same matches; `MC` and `CC` are arm `F1c`'s.

**D-22's kept share per pair,** of the 150,000 matches each pair draws over the 30 slots, beside section 2.7's column "Sampled matches D-22's filters would keep" (the manifest's cameras, all 120 frames, the scene's seven pairs):

| Pair | Section 2.7 | Centre copy: `MM` | `CM` | `MC` | `CC` | Left copy: `MM` | `CM` | `MC` | `CC` |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `cam0`–`cam2` | – | | | | | 89.9% | 91.7% | | |
| `cam0`–`cam6` | – | | | | | | | 87.0% | 90.2% |
| `cam1`–`cam4` | 47.4% | 47.7% | 67.2% | | | | | | |
| `cam1`–`cam7` | – | | | 53.2% | 76.6% | | | | |
| `cam2`–`cam5` | 87.5% | 88.1% | 86.5% | | | 88.1% | 88.3% | | |
| `cam2`–`cam8` | – | | | 32.3% | 33.3% | | | 32.3% | 32.6% |
| `cam3`–`cam4` | 57.8% | 58.1% | 73.6% | 58.1% | 73.6% | 58.1% | 73.2% | 58.1% | 73.2% |
| `cam3`–`cam5` | 60.2% | 62.0% | 63.0% | 62.0% | 63.0% | 62.0% | 62.4% | 62.0% | 62.4% |
| `cam6`–`cam7` | 57.8% | 58.2% | 52.8% | 58.2% | 52.8% | 58.2% | 44.3% | 58.2% | 44.3% |
| `cam6`–`cam8` | 67.9% | 68.5% | 60.4% | 68.5% | 60.4% | 68.5% | 67.9% | 68.5% | 67.9% |
| All pairs | 67.5% (seven) | 63.8% | 67.3% | 55.4% | 60.0% | 70.8% | 71.3% | 61.0% | 61.8% |

- **The manifest's figures reproduce section 2.7's** at stride 4, within 2 points per pair.
- **The corrections keep more where the background was most off,** `cam1`–`cam4` from 48% to 67% and `cam3`–`cam4` from 58% to 74%, and fewer in `cam6`–`cam7` (58% to 53% and 44%) and `cam6`–`cam8` on the centre copy (69% to 60%). That is the direction section 5.3 found with section 2.5's turns (67% from 47%, 73% from 58%; 47% from 58%, 54% from 68%).
- **The corrected pairing loses most of `cam2`–`cam8`:** 32% to 33% kept against 88% in `cam2`–`cam5`, as section 2.2 here foresaw for the person (0.8% to 2.1% there). `cam1`–`cam7` keeps 77% on the corrected cameras, more than `cam1`–`cam4`'s 67%. Over all pairs, `CC` keeps 60% and 62% against `CM`'s 67% and 71%.

**The runs.** Each is `ftgs train` unchanged, 30,000 iterations of the default training configuration, 3,000,000 primitives at the end.

| Run | Copy, seed | Init | Cap reached at | Time | GPU peak |
| --- | --- | --- | --- | --- | --- |
| `F1m_h0_s0` | centre, 0 | `CM_h0` | 10,400 | 13 min 12 s | 23,921 MiB |
| `F1m_h0_s1` | centre, 1 | `CM_h0` | 10,500 | 13 min 0 s | 23,788 MiB |
| `F1m_h1_s0` | left, 0 | `CM_h1` | 9,700 | 15 min 49 s | 23,785 MiB |
| `F1c_h0_s0` | centre, 0 | `CC_h0` | 10,300 | 13 min 9 s | 23,601 MiB |
| `F1c_h0_s1` | centre, 1 | `CC_h0` | 10,400 | 13 min 1 s | 23,788 MiB |
| `F1c_h1_s0` | left, 0 | `CC_h1` | 9,600 | 13 min 2 s | 23,794 MiB |

The baseline's caps fell at 10,800, 11,100 and 9,700 (section 3 here).

**The named measure: the near-side far-5 cm share,** with the tail beyond 25 cm beside it, against the five reference files: the kept stride-4 file (`kept`), and the copy's `MM`, `CM`, `MC` and `CC`. Each cell is 5 cm / beyond 25 cm. The baseline's checkpoints were measured again against every file; against the kept file they give section 3 here's figures. The near side is cut along the centre copy's manifest's `cam0` for every run, as in section 3 here.

| Run | `kept` | `MM` | `CM` | `MC` | `CC` |
| --- | --- | --- | --- | --- | --- |
| `A_h0_s0`, baseline | 0.1401 / 0.0349 | 0.1320 / 0.0316 | 0.1349 / 0.0310 | 0.1672 / 0.0385 | 0.1722 / 0.0380 |
| `A_h0_s1`, baseline | 0.1409 / 0.0267 | 0.1395 / 0.0257 | 0.1352 / 0.0248 | 0.1540 / 0.0325 | 0.1534 / 0.0325 |
| `A_h0_s0_rep`, plain, not a baseline run | 0.1485 / 0.0475 | 0.1426 / 0.0466 | 0.1529 / 0.0431 | 0.1726 / 0.0511 | 0.1805 / 0.0522 |
| `F1m_h0_s0` | 0.1347 / 0.0344 | 0.1316 / 0.0307 | 0.1288 / 0.0303 | 0.1594 / 0.0427 | 0.1525 / 0.0433 |
| `F1m_h0_s1` | 0.1481 / 0.0276 | 0.1456 / 0.0246 | 0.1448 / 0.0232 | 0.1742 / 0.0359 | 0.1702 / 0.0355 |
| `F1c_h0_s0` | 0.1301 / 0.0318 | 0.1298 / 0.0303 | 0.1318 / 0.0278 | 0.1595 / 0.0397 | 0.1599 / 0.0386 |
| `F1c_h0_s1` | 0.1208 / 0.0295 | 0.1111 / 0.0267 | 0.1183 / 0.0238 | 0.1457 / 0.0334 | 0.1458 / 0.0329 |
| `A_h1_s0`, baseline | 0.1632 / 0.0438 | 0.1541 / 0.0440 | 0.1546 / 0.0436 | 0.1768 / 0.0538 | 0.1699 / 0.0451 |
| `F1m_h1_s0` | 0.1333 / 0.0381 | 0.1302 / 0.0386 | 0.1277 / 0.0390 | 0.1561 / 0.0490 | 0.1475 / 0.0405 |
| `F1c_h1_s0` | 0.1301 / 0.0402 | 0.1172 / 0.0399 | 0.1164 / 0.0393 | 0.1365 / 0.0451 | 0.1302 / 0.0417 |

**Judged by section 4.4's rule** on the centre copy's two seeds: both of the arm's seeds below both of the baseline's, and the means more than 0.01 apart.

| Arm, against | `kept` | `MM` | `CM` | `MC` | `CC` | Its two references from the same matches |
| --- | --- | --- | --- | --- | --- | --- |
| `F1m` against the baseline | not moved (seeds 0.135, 0.148; mean 0.001 above) | not moved (0.003 above) | not moved (0.002 above) | not moved (0.006 above) | not moved (0.001 below) | `MM` and `CM`: **not lower** |
| `F1c` against the baseline | **lower** by 0.015 | **lower** by 0.015 | **lower** by 0.0100 (0.01005) | not moved: seed 0 at 0.160 above the baseline's 0.154 (mean 0.008 below) | not moved: 0.160 above 0.153 (mean 0.010 below) | `MC` and `CC`: **not lower** |
| `F1c` against `F1m` (the pairing) | **lower** by 0.016 | **lower** by 0.018 | not moved (mean 0.012 below) | not moved (0.014 below) | not moved (0.009 below) | – |

- **Neither arm counts as lower.** `F1m`, the correction with the baseline's pairs, does not move the named measure against any file: its seeds straddle the baseline's (0.135 and 0.148 against 0.140 and 0.141 on the kept file). `F1c` is lower against the kept file, `MM` and `CM`, by 0.010 to 0.015, but not against `MC` and `CC`, its two files from the same matches, where its seed 0 lies above the baseline's seed 1.
- **The third plain draw** of seed 0 (`A_h0_s0_rep`, section 3 here) lies at 0.149 on the kept file and above both arms' seeds against every file. Against the three plain draws, `F1c`'s seed 0 is below all three on the kept file and `MM`, and not on the others.
- **The left copy,** one seed a side: both arms lie below the baseline against every file, `F1m` by 0.021 to 0.030 and `F1c` by 0.033 to 0.040 (0.133 and 0.130 against 0.163 on the kept file), 2.5 to 5 times the three plain draws' range of 0.008 on the centre copy. It is the only place the named measure fell for `F1m`. Section 4.4's rule needs two seeds a side, and the left copy has one, so it is not a movement by the rule; it is the strongest sign in this step that the correction lowers the share (question 1 below).
- **The tail does not move** by rule (b): on the centre copy no arm has both seeds beyond both of the baseline's against any file, and the means lie within 0.005 of each other (the arms' seeds 0.023 to 0.043, the baseline's 0.025 to 0.039). So `F1c`'s fall against three files lies in the 5 to 25 cm band, which section 2.6 here reads as a change in the method's habit rather than a disagreement removed.
- **Each arm also differs from the baseline in its init's sample.** The baseline trained from the kept stride-4 files, which current code does not reproduce (section 3 here); both arms trained from fresh inits. So every difference between an arm and the baseline, `F1c`'s fall against three files and the left copy's included, carries the init's sample as well as the cameras. Only `F1c` against `F1m` compares two fresh inits.
- **The far share depends on the reference file** by as much as any arm moves it: the baseline's seed 0 reads 0.132 to 0.172 over the five files, and both corrected-pairing files read 0.01 to 0.04 higher than the others for every run, since they lack most of `cam2`–`cam8`'s points (above).

**The other shell measures** (against the kept file, whole model / near side) change little. The under-two-frames share is 0.443 to 0.460 / 0.583 to 0.599 for the arms, against 0.449 to 0.459 / 0.579 to 0.595 for the plain draws. The near side's far-10 cm share is 0.058 to 0.073, against 0.058 to 0.081. The far side's far-5 cm share rises, 0.555 to 0.581 on the centre copy and 0.627 and 0.636 on the left copy, against 0.535 to 0.541: the far side's points from the corrected cameras lie farther from the kept file's, which the manifest's cameras triangulated.

**The test-view scores, the sharpness ratio and the tiles' coverage.** As section 3 here takes them, at both poses of the test view: the manifest's (`mpose`: the notes' request unchanged for the held-out camera; E16 by `ftgs eval` on the copy's train-only manifest) and the corrected one (`cpose`: the corrected manifest's pose; E16 is the run's METRICS.json). In both requests the training-camera items take the corrected poses the arm trained on. The baseline has one pose, the manifest's. Kept-pixel PSNR is Q-233's, on the centre copy alone; the rule matched the stored mask on every pixel again.

| Run | Pose | Crop PSNR | Crop SSIM₁ | Sharpness ratio | Late | Training cameras: PSNR / ratio | E16 PSNR / SSIM₁ / ꟻLIP | Kept-pixel PSNR | Coverage: between cameras, `sC00`, ±30°, ±45°; ±60°, ±90° |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `A_h0_s0` | manifest | 24.36 | 0.8149 | 0.7262 | 0.7504 | 32.90 / 0.7755 | 24.79 / 0.8187 / 0.1531 | 25.52 | 0.84 to 1.00; 0.50 to 0.90 |
| `A_h0_s1` | manifest | 24.31 | 0.8153 | 0.7249 | 0.7467 | 32.79 / 0.7742 | 24.75 / 0.8189 / 0.1545 | 25.47 | 0.80 to 1.00; 0.50 to 1.00 |
| `F1m_h0_s0` | manifest | 24.27 | 0.8165 | 0.7117 | 0.7379 | 33.46 / 0.7745 | 24.36 / 0.8080 / 0.1556 | 25.03 | 0.90 to 1.00; 0.50 to 1.00 |
| | corrected | 24.50 | 0.8227 | 0.7122 | 0.7389 | | 24.61 / 0.8164 / 0.1523 | 25.32 | |
| `F1m_h0_s1` | manifest | 24.28 | 0.8163 | 0.7091 | 0.7398 | 33.15 / 0.7718 | 24.44 / 0.8079 / 0.1563 | 25.12 | 0.79 to 1.00; 0.45 to 0.94 |
| | corrected | 24.50 | 0.8224 | 0.7093 | 0.7404 | | 24.67 / 0.8157 / 0.1532 | 25.40 | |
| `F1c_h0_s0` | manifest | 24.27 | 0.8147 | 0.7102 | 0.7422 | 33.31 / 0.7748 | 24.31 / 0.8063 / 0.1565 | 24.97 | 0.84 to 1.00; 0.59 to 1.00 |
| | corrected | 24.48 | 0.8207 | 0.7106 | 0.7430 | | 24.57 / 0.8153 / 0.1532 | 25.28 | |
| `F1c_h0_s1` | manifest | 24.22 | 0.8149 | 0.7100 | 0.7370 | 33.34 / 0.7715 | 24.24 / 0.8059 / 0.1566 | 24.89 | 0.80 to 1.00; 0.44 to 0.90 |
| | corrected | 24.44 | 0.8210 | 0.7104 | 0.7378 | | 24.50 / 0.8148 / 0.1533 | 25.20 | |
| `A_h1_s0` | manifest | 23.90 | 0.7473 | 0.7403 | 0.7938 | 31.31 / 0.7719 | 21.92 / 0.7392 / 0.1951 | (`cam1`: not applied) | 0.78 to 1.00; 0.55 to 1.00 |
| `F1m_h1_s0` | manifest | 23.51 | 0.7421 | 0.7384 | 0.7949 | 31.93 / 0.7751 | 21.60 / 0.7331 / 0.1965 | | 0.80 to 1.00; 0.48 to 0.96 |
| | corrected | 24.75 | 0.7793 | 0.7376 | 0.7947 | | 22.07 / 0.7606 / 0.1872 | | |
| `F1c_h1_s0` | manifest | 23.45 | 0.7410 | 0.7441 | 0.8026 | 31.62 / 0.7750 | 21.41 / 0.7317 / 0.1985 | | 0.79 to 1.00; 0.49 to 0.89 |
| | corrected | 24.70 | 0.7785 | 0.7431 | 0.8016 | | 21.86 / 0.7602 / 0.1889 | | |

Every rendered value is finite. Against section 4.4's guard, a fall of more than 0.3 dB in a test-view score counts as harm:

- **At the manifest's pose, harm on both copies.** E16 PSNR falls by 0.31 to 0.55 dB on the centre copy (24.24 to 24.44 against 24.75 and 24.79), and by 0.32 to 0.51 dB on the left copy (21.41 to 21.60 against 21.92). The kept-pixel PSNR falls by 0.35 to 0.63 dB, and the left copy's crop PSNR by 0.39 and 0.45 dB. The centre copy's crop PSNR falls by 0.03 to 0.14 dB only. Section 2.2 here foresaw this: the manifest's pose of the test view no longer agrees with the corrected training cameras.
- **At the corrected pose, no harm by the guard,** with one value at its edge. The crop PSNR rises, by 0.08 to 0.19 dB on the centre copy and by 0.80 and 0.85 dB on the left copy, and the crop SSIM₁ with it. E16 PSNR lies within 0.08 to 0.29 dB below the centre baseline and 0.06 below to 0.15 dB above the left one. The kept-pixel PSNR of `F1c_h0_s1`, 25.20 dB, is 0.32 dB below `A_h0_s0`'s and 0.27 dB below `A_h0_s1`'s.
- **The training cameras' PSNR rises** by 0.1 to 0.7 dB on every run, as the cameras agree better with each other.
- **The sharpness ratio falls on the centre copy** at both poses: 0.709 to 0.712 for both arms, against 0.725 and 0.726 for the baseline's seeds and 0.730 for the third plain draw. The means are 0.015 apart, beyond what two seeds give (0.0013) and what three plain draws give (0.0054; section 3 here). The late ratio falls with it (0.737 to 0.743 against 0.747 to 0.754). On the left copy it does not fall: 0.738 and 0.744 against 0.740. The training cameras' ratio is unchanged (0.772 to 0.775 against 0.774 to 0.776). Both arms fall alike, and they share both the corrected cameras and a fresh init, so the fall is not shown to be the correction's: it could be the init's sample (above).
- **The tiles' coverage** stays within the baseline's ranges: 0.79 to 1.00 and 0.44 to 1.00 on the side tiles, against 0.78 to 1.00 and 0.49 to 1.00. No tile collapses.

**The sheets, for judging with their labels hidden.** Each set holds the baseline and both arms for one copy and seed, in a seeded order, labelled `set<n>-A` to `-C` only: set 1 the centre copy at seed 0, set 2 at seed 1, set 3 the left copy. They are `<surface-runs>/ticket-25/sheets/blind/set<n>-<letter>.jpg`, from section 2.6's fixed request, with the key apart in `KEY.json` beside them. Nobody here has looked at them; the owner's judgement is the criterion, and it is open (question 3 below).

**The pairing's effect, apart from the correction's.** `F1c` against `F1m`, same corrected cameras, other pairs:

- **On the named measure,** `F1c` lies below `F1m` against the kept file and `MM` (by 0.016 and 0.018, both seeds), and not against the files made on the corrected cameras (`CM`, `MC`, `CC`), where their seeds overlap. Against those files the pairing's effect is within the seeds' spread; against the manifest-camera files it is as large as section 4.4's margin.
- **On the scores** the two arms are alike: crop PSNR within 0.06 dB, E16 within 0.2 dB, the sharpness ratio within 0.003, at both poses. On the left copy `F1c` scores 0.05 to 0.2 dB below `F1m`.
- **So the correction's effect, with the init's sample** (`F1m` against the baseline), is: the named measure unmoved on the centre copy by the rule and lower on the left copy's one seed, the tail unmoved, the sharpness ratio down by 0.015 on the centre copy, the test-view scores up at the corrected pose and down at the manifest's. The pairing adds a fall of the named measure that only the manifest-camera references show.

**The control on `kitchen`** (section 5.3, cleared by the owner): openftgs-synth's exact cameras, the same turned by a seeded 0.1° to 0.4°, and those corrected by check (c)'s tool (row 863).

- **The turned manifest.** Each of the 20 cameras turned about its own centre by an axis uniform on the sphere and an angle uniform in [0.1°, 0.4°], from NumPy's `default_rng(25)` in the manifest's camera order: 0.12° to 0.38°. Their mean turn over the 19 training cameras is 0.11°, the common part the tool holds at zero (section 2.2 here) and cannot take out.
- **The tool, on `kitchen`.** Ticket #19's fit, copied, with three of private scene A's constants made arguments: every match in front of both cameras counts (`kitchen`'s cameras agree, so nothing is set aside as the person is), f·b = 1170.9 px × 0.232 = 271.6, and frames 0, 20, …, 280. D-22's pairs on `kitchen` form six separate groups of two to four cameras, each of which the hold at zero would leave its own common turn, so the fit reads each training camera with its two nearest by axis, on both manifests: 25 pairs, one connected set, and 45 pairs of partners; `cam00`, the test view, then with its three nearest. The 28 pairs were matched by ticket #12's script, 15 frames each.
- **On the exact cameras, the corrections stay at the fit's floor.** The 19 training cameras turn by 0.002° to 0.012° (median 0.005°) and `cam00` by 0.012°, against the 0.0035° by which section 2.2 here took a known turn back out; the background's epipolar residual is 0.023 to 0.034 px at the median per pair before and after, and the depth agreement 0.05% to 0.13% before and after. This is the acceptance criterion's check, and it holds.
- **On the turned cameras, the residual returns to the floor but the poses do not.** The residual goes from 0.50 to 7.96 px at the median per pair to 0.025 to 0.042 px, and the depth agreement from 0.9% to 9.9% to 0.05% to 0.12%. But each corrected camera still lies 0.106° to 0.120° from the exact one: 0.10° of it is the common turn, and what remains after removing it is 0.010° to 0.073° (median 0.041°), in the directions the fit's three smallest singular values (86 to 198 px per degree, against up to 8,519) say the matches barely fix. `cam00` lies 0.116° from the exact pose. D-22's pairs are the exact manifest's on all three manifests, so the control has no pairing effect. S changes by 1·10⁻⁷.
- **The inits:** plain `ftgs init roma` at its default stride, as M17's `kitchen` init, on each manifest: 32 min 1 s and 30 min 52 s.
- **The runs:** `ftgs train` at seeds 0 and 1 on each manifest, from its own init.

| Run | E16 at its own manifest's pose: PSNR / SSIM₁ | E16 at the exact pose: PSNR / SSIM₁ / ꟻLIP | Kept-pixel PSNR, own pose; exact pose | Residual, all pixels: in front / behind | Dynamic pixels: in front / behind | Expected depth, median error, all / dynamic | Far share at 0.12·S, against M17's init: whole / dynamic | Beyond 25 cm | Under two frames |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `kitchen_s0`, baseline | 29.25 / 0.8787 | the same | 30.42 | 0.041 / 0.291 | 0.015 / 0.613 | 252 / 57 mm | 0.036 / 0.002 | 0.023 / 0.001 | 0.381 / 0.660 |
| `kitchen_s1`, baseline | 30.11 / 0.8803 | the same | 31.34 | 0.039 / 0.297 | 0.015 / 0.612 | 448 / 304 mm | 0.035 / 0.003 | 0.022 / 0.001 | 0.377 / 0.662 |
| `kitchen_s2`, baseline | 29.25 / 0.8800 | the same | 31.15 | 0.040 / 0.295 | 0.015 / 0.618 | 277 / 103 mm | 0.036 / 0.002 | 0.023 / 0.001 | 0.385 / 0.661 |
| `K_turned_s0` | 21.80 / 0.6296 | 21.37 / 0.6201 / 0.2203 | 23.11; 22.55 | 0.175 / 0.556 | 0.066 / 0.709 | 808 / 599 mm | 0.075 / 0.012 | 0.050 / 0.005 | 0.407 / 0.577 |
| `K_turned_s1` | 22.00 / 0.6302 | 21.59 / 0.6214 / 0.2160 | 23.16; 22.65 | 0.171 / 0.564 | 0.062 / 0.720 | 519 / 279 mm | 0.075 / 0.012 | 0.049 / 0.005 | 0.404 / 0.580 |
| `K_corrected_s0` | 27.82 / 0.8743 | 23.19 / 0.6216 / 0.1598 | 30.41; 24.47 | 0.042 / 0.317 | 0.016 / 0.642 | 182 / 43 mm | 0.036 / 0.002 | 0.023 / 0.001 | 0.382 / 0.654 |
| `K_corrected_s1` | 28.25 / 0.8718 | 23.37 / 0.6231 / 0.1596 | 30.58; 24.57 | 0.040 / 0.320 | 0.015 / 0.644 | 181 / 42 mm | 0.036 / 0.003 | 0.023 / 0.001 | 0.381 / 0.656 |

The baseline's rows are section 3 here's. The residual's shares and expected depth are `scripts/gt_depth.py`'s at the exact cameras, the only ones it accepts (it refuses a pose that differs from the ground truth's), at δ = 0.02·S. The far shares and the under-two-frames share are section 2.1 here's, against M17's init file of the exact cameras; against each run's own init the far share at 0.12·S is 0.032 for the turned runs and 0.035 for the corrected ones, the baseline's 0.035 to 0.036. The four runs took 16 min 25 s to 17 min 2 s, their caps fell at 2,300 and 2,400 (turned) and 2,100 (corrected), against 2,100 for the baseline.

What the control shows:

- **The turned cameras do what private scene A's are suspected of, and the measures see it.** Against M17's init of the exact cameras, the far share at 0.12·S doubles (0.075 against 0.035 to 0.036) and the share beyond 25 cm with it (0.049 and 0.050 against 0.022 and 0.023); the residual's share behind the surface rises from 0.291 to 0.297 to 0.556 and 0.564; E16 falls by 7 to 8 dB. Against the runs' own init the far share stays at the baseline's (0.032): measured against an init made on the same wrong cameras, it does not see them.
- **The corrected cameras bring the notes' measures back to the baseline's:** the far share at 0.12·S 0.036, beyond 25 cm 0.023, the under-two-frames share 0.381 and 0.382, all within the three exact runs' ranges.
- **They do not bring back everything.** The residual's share behind the surface is 0.317 and 0.320, 0.020 to 0.029 above the exact runs', beyond section 4.4's 0.005, and on dynamic pixels 0.642 and 0.644 against 0.612 to 0.618; the share in front is unchanged. SSIM₁ at the runs' own pose is 0.8743 and 0.8718, 0.005 to 0.008 below the exact runs' 0.8787 to 0.8803, beyond the 0.002 guard; the kept-pixel PSNR at their own pose, 30.41 and 30.58 dB, is at the bottom of the exact runs' 30.42 to 31.34. At the exact pose of `cam00`, which the corrected `cam00` misses by 0.116°, E16 is 23.2 and 23.4 dB. The median error of expected depth is lower, 181 and 182 mm against 251 to 448, but section 4.4 counts it only where three-seed ranges part, and here there are two.
- **So on `kitchen` the tool takes back most of a 0.1° to 0.4° error but leaves about 0.1°,** the rig's common turn and the directions the background barely fixes. The corrected runs fall short of the exact runs by what section 4.4 calls harm in SSIM₁ and in the residual behind the surface, but those deficits are read, not attributed: the corrected runs also differ from the baseline in their init (a fresh one, where the baseline's is M17's), `gt_depth.py` scores a model whose frame is about 0.1° off at the exact cameras, and the own-pose scores are at a fitted `cam00` pose. Private scene A's corrections were of the same size (0.05° to 0.45°; section 2.2 here), with the common turn held at zero there too.

**The left copy's second seed** (at the owner's answer of 2026-11-17, question 1 below). The baseline (`A_h1_s1`: the copy's manifest, the kept stride-4 file) and arm `F1m` (`F1m_h1_s1`: the corrected manifest, `CM_h1`) were trained again at seed 1, by the same commands (`run_train.sh`, then `post_base.sh`, a copy of `post_a.sh` that scores the baseline at its one pose as section 3 here did, and `post_a.sh`), and measured and scored as above, in about 29 minutes of GPU (13 min 33 s and 13 min 15 s of training; caps at 9,900 and 9,900; peaks 23,970 and 23,504 MiB; row 864).

| Run | `kept` | `MM` | `CM` | `MC` | `CC` |
| --- | --- | --- | --- | --- | --- |
| `A_h1_s0`, baseline | 0.1632 / 0.0438 | 0.1541 / 0.0440 | 0.1546 / 0.0436 | 0.1768 / 0.0538 | 0.1699 / 0.0451 |
| `A_h1_s1`, baseline | 0.1526 / 0.0366 | 0.1429 / 0.0365 | 0.1500 / 0.0366 | 0.1724 / 0.0487 | 0.1695 / 0.0412 |
| `F1m_h1_s0` | 0.1333 / 0.0381 | 0.1302 / 0.0386 | 0.1277 / 0.0390 | 0.1561 / 0.0490 | 0.1475 / 0.0405 |
| `F1m_h1_s1` | 0.1398 / 0.0389 | 0.1269 / 0.0391 | 0.1322 / 0.0389 | 0.1553 / 0.0493 | 0.1450 / 0.0416 |

| Run | Pose | Crop PSNR | Crop SSIM₁ | Sharpness ratio | Late | Training cameras: PSNR / ratio | E16 PSNR / SSIM₁ / ꟻLIP | Coverage |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `A_h1_s1` | manifest | 23.71 | 0.7472 | 0.7457 | 0.7969 | 31.59 / 0.7686 | 21.66 / 0.7384 / 0.1976 | 0.82 to 1.00; 0.49 to 0.86 |
| `F1m_h1_s1` | manifest | 23.61 | 0.7421 | 0.7397 | 0.7840 | 31.70 / 0.7726 | 21.66 / 0.7325 / 0.1963 | 0.80 to 1.00; 0.49 to 0.89 |
| | corrected | 24.76 | 0.7792 | 0.7388 | 0.7832 | | 22.10 / 0.7604 / 0.1871 | |

- **On the left copy the named measure has moved, by section 4.4's rule.** Both of `F1m`'s seeds lie below both of the baseline's against every one of the five files, and the means differ by 0.019 to 0.023: 0.021 on the kept file (0.137 against 0.158), 0.020 on `MM` and 0.022 on `CM`, `F1m`'s two references from the same matches, so it counts as lower against both. The baseline's two seeds lie 0.0004 to 0.011 apart.
- **The tail does not move** there (`F1m` 0.038 and 0.039 against 0.044 and 0.037 on the kept file), so by section 2.6 here's reading the fall lies in the 5 to 25 cm band.
- **The other criteria on the left copy, two seeds a side.** At the manifest's pose the means fall by 0.16 dB in E16 PSNR and 0.25 dB in crop PSNR, within the 0.3 dB guard; at the corrected pose crop PSNR rises by 0.95 dB and E16 PSNR by 0.30 dB. The sharpness ratio falls by 0.004 (0.739 against 0.743), within the baseline's two seeds' 0.005.
- **This changes the reading of section 7.2's row for finalist 1** (below): the share moved on the left copy and did not on the centre copy, so the drop clause's "the share does not move" no longer holds on both copies. The arm still carries the init's sample (the baseline trained from the kept file, the arm from `CM_h1`), and the adopt clause stays unmet (check (c)'s "every pair", and on the centre copy the manifest-pose harm and the unmoved share). The owner's drop of 2026-11-17 was given before this result and stands unless the owner changes it.

**Against the ticket's acceptance criteria.**

- **Three runs on the corrected manifests, each from an init made on its own manifest.** Done, twice: arm `F1m` from the scratch init with the manifest's pairs, arm `F1c` from plain `ftgs init roma`, each on its copy's corrected manifest.
- **The near-side far-5 cm share against both reference files,** by section 4.4's rule. Done, against the ticket's two files per arm and the owner's two more: neither arm is lower against both of its files.
- **The other criteria of section 4.4.** The sharpness ratio, the coverage and the test-view scores at both poses: done (above). The sheets with their labels hidden: made and keyed, not judged (question 3).
- **The control on `kitchen`.** Done, with the tool's corrections at the fit's floor on the exact cameras.
- **The reading by section 7.2's row for finalist 1.** Below. The owner's answer: not yet given.
- **Sources, commands, files changed, privacy:** rows 861, 862 and 863; the commands below; only this document and the source log change; nothing names private scene A or shows its images.

**The reading by section 7.2's row for finalist 1.**

- **Adopt when** check (c) lowers the background's residual for every pair on held-out frames (it did not: section 2.2 here), the named measure is lower against both reference files (it is not, for either arm), and no other criterion shows harm at either pose (the manifest's pose shows harm of 0.31 to 0.63 dB, and the sharpness ratio falls on the centre copy beyond two seeds). **Not met, on all three counts.**
- **Drop when** check (c) does not lower the background's residual on held-out frames (it did lower it, over the seven pairs), **or the share does not move.** The share did not move for `F1m`, the correction's own arm, nor for `F1c` against its two files. **The drop clause is met by its second branch.** **With the left copy's second seed (above), this holds on the centre copy only:** on the left copy `F1m`'s share is lower against both its references by the rule. Neither clause of the row is then met on both copies; the reading is split by copy, and the decision is the owner's.
- **"A correction that holds on held-out frames is still a better manifest, and keeping it for that is the owner's call."** What argues for that: the background's residual halves (section 2.2 here); the training cameras' PSNR rises; at the corrected pose the crop PSNR rises, by 0.8 dB on the left copy; the control shows the tool takes 0.1° to 0.4° turns back to the residual's floor. What argues against it: the sharpness ratio falls by 0.015 on the centre copy, beyond the plain runs' spread, though not separated from the fresh init's sample; the named measure, the tail and the under-two-frames share are unmoved; the manifest's pose of the test view scores worse; the corrected pairing loses most of `cam2`–`cam8`; and the control shows the tool leaves each camera 0.1° from the truth, and on `kitchen` the corrected runs fall 0.005 to 0.008 short in SSIM₁ and 0.02 to 0.03 in the residual's share behind the surface, which this control does not separate from its fresh inits and its scoring at the exact cameras.
- **Recommended:** drop finalist 1 as a technique against the shell, and do not make the corrected manifests the ones finalists 2 to 4 train on: by the rule the correction moves none of the measures the finalists are judged by, the left copy's fall rests on one seed a side, and it would carry the changed pairing into every later init. Keep the corrected manifests on file, as section 2.6 here recommended. This leans on the drop clause's second branch, not on the sharpness fall, which is not separated from the init's sample.

**The questions for the owner.**

1. **The left copy's single seed.** Both arms lie 0.021 to 0.040 below the baseline there against every file, the only fall of the named measure in this step for `F1m`, and section 4.4's rule cannot read it with one seed a side. A second seed on the left copy for the baseline and `F1m` (about 30 minutes) would settle whether it is a movement. Recommended: run it before question 2 is answered, if the owner wants the drop to rest on both copies; otherwise the reading below stands on the centre copy.
2. **Finalist 1: kept or dropped, and is the correction kept** as the manifests every later arm trains on?
   - (a) Drop the finalist and keep the manifest's cameras for finalists 2 to 4; their off arm is the script's v2 at weight 0 on the manifest's cameras (section 3 here, question 3).
   - (b) Drop the finalist as a technique against the shell, but keep the corrected manifests as a better manifest for later arms. These runs then become finalists 2 to 4's baseline, but only as plain runs: the off arm the owner chose for them goes through the script, so it would have to be run again on the corrected manifests (about 27 minutes per scene set), with the pairing chosen with it (`F1m`'s or `F1c`'s).
   - (c) Keep finalist 1 for a combined arm later.

   **Recommended: (a).**
3. **The sheets** (`<surface-runs>/ticket-25/sheets/blind`, key apart): the owner's judgement with the labels hidden, which this ticket cannot give. Under (a) it is recorded only.
4. **The baseline's init sample.** The baseline trained from the kept stride-4 files and both arms from fresh inits (section 3 here), so the sharpness fall, `F1c`'s fall and the left copy's fall all carry the init's sample. A baseline retrained from `MM`, the fresh init of the manifest's cameras (about 40 minutes for three runs), would separate it. Recommended: run it if the owner leans to (b) or wants the sharpness fall attributed; under (a) the reading rests on the unmoved measure, which the confound does not create.

**What blocks what.** Question 2 blocks step 4: under (a) finalist 2 runs on the manifest's cameras, as section 3 here's off arm does; under (b) the off arm through the script is run again on the corrected manifests first. Questions 1 and 4 bear on question 2's answer and block nothing else; question 3 blocks nothing.

**The commands.** Each ran with `OMP_NUM_THREADS`, `MKL_NUM_THREADS` and `OPENBLAS_NUM_THREADS` at the thread count given, `PYTHONPATH=<worktree>/src:<worktree>/harness/src` and the shared `.venv`'s Python, the CPU ones with `CUDA_VISIBLE_DEVICES=` empty and the GPU ones under the shared lock. The inits and the matcher add ticket #18's `env-init` to `PYTHONPATH`, with `PYTHONDONTWRITEBYTECODE=1`. Every training ran under `/usr/bin/time -v`, with `nvidia-smi` sampling memory once a second. `$T` is `<scratch>/ticket-25`, `$O` is `<surface-runs>/ticket-25`, `$X` is `<private-scene-A-exp>`, `$K` is `<m15-runs>/kitchen`, `$M` is `<m17-runs>`, `$G` is `$OPENFTGS_DATASETS/synth/ground_truth`, `$S` is `<surface-runs>/ticket-5/side_request.json`. The scripts are not committed.

| Command | Threads; device | Time | Note |
| --- | --- | --- | --- |
| `cp` of `<surface-runs>/ticket-19/manifests/P/h<c>` and `P/h<c>-train-only` (`scene.json`) into `$O/manifests/`, each with a link `images` to its copy's images | –; CPU | seconds | byte for byte (`cmp`), so that the frame caches are written beside these copies and nowhere in ticket #19's folders or `<private-scene-A-exp>` |
| `python $T/init_pairs.py <manifest> $O/init/<name>.safetensors --pairs-from <manifest whose pairing is used> --counts $O/init/<name>.counts.json --stride 4` | 16; GPU | 1 min 24 s each | `ftgs init roma --stride 4` by `openftgs.cli.main`, with `openftgs.init._pairs` replaced by the package's own pairing on the other manifest's cameras and `_triangulated` counted per pair: `MM` (`$X/scene_h<c>` with its own pairs), `CM` (`$O/manifests/h<c>` with `$X/scene_h<c>`'s pairs), `MC` (`$X/scene_h<c>` with `$O/manifests/h<c>`'s pairs), `CC_counts` (`$O/manifests/h<c>` with its own) |
| `ftgs init roma $O/manifests/h<c>/scene.json $O/init/CC_h<c>.safetensors --stride 4` | 16; GPU | 1 min 24 s each | the second arm's init, by the CLI unchanged; equal bit for bit to `CC_h<c>_counts` |
| `python $T/same_init.py <a> <b>` | 1; CPU | seconds | the equality checks: `MM_h0` against ticket #24's `regen_h0_s4`, `CC_h<c>` against `CC_h<c>_counts` |
| `python $T/build_requests.py $X/requests/heldout_novel.json $X/scene_h0/scene.json $O/manifests/h0/scene.json $O/requests/score_h0 --heldout cam0` (the left copy: `heldout1_novel.json`, `cam1`) | 1; CPU | seconds | the notes' scoring request at both poses of the test view, the training-camera items at the corrected poses |
| `bash $T/run_train.sh <run> plain $O/manifests/h<c>/scene.json $O/init/<CM or CC>_h<c>.safetensors <seed>`, i.e. `ftgs train $O/manifests/h<c>/scene.json --init … --out $O/runs/<run> --seed <seed>` | 16; GPU | 13 min each | ticket #24's script, repointed |
| `bash $T/post_a.sh <run> h<c>`: `ftgs render <checkpoint> --request $S`; `ftgs render <checkpoint> --request $O/requests/score_h<c>_<mpose or cpose>.json`; `python $T/score_a.py <renders> $X/scene_h<c>/scene.json --heldout <cam> --crop <rect> --stored-masks <surface-runs>/ticket-4/private-scene-A/placeholder_masks.npz`; `ftgs eval <checkpoint> $O/manifests/h<c>-train-only/scene.json --out $O/metrics/<run>_mpose.json`; `python $T/side_sheet.py …`; `python scripts/run_summary.py …` | 16; GPU, then CPU | about 2 min | the sheet, the scores at both poses, E16 at the manifest's pose (the run's METRICS.json is the corrected pose) |
| `bash $T/measures.sh <runs>`: `python $T/measures_a.py $X/scene_h0/scene.json --out $O/measures/a_<ref>.json <run>=<checkpoint>=<reference file>` for the five references | 8; CPU | 2 to 4 s each | the shell's measures; the baseline's checkpoints under `<surface-runs>/ticket-24/runs`, read only |
| `python $T/kfit/turn_manifest.py $K/scene.json $O/kitchen/manifests/turned --seed 25` | 1; CPU | seconds | the control's turned manifest |
| `python $T/kfit/knn_pairs.py <manifest> --k 2 --test cam00 --ktest 3` | 1; CPU | seconds | the fit's pairs on `kitchen` |
| `flock <lock> bash $T/kfit/queue_match_body.sh`: `python $T/kfit/match_pair.py $K/scene.json $O/kitchen/matches --label kitchen --tag <pair> --pair <A>:<B> --frames 0:300:20` for 28 pairs | 8; GPU | 15 s each | ticket #12's script, unchanged; one lock for all, after one pair per lock lost every race to the trainings |
| `bash $T/kfit/run_kfit.sh <exact or turned> <manifest> <the 25 pairs> <their 45 pairs of partners>`: `python $T/kfit/fit.py <manifest> … --far 1e-9 --fb 271.6 --frame-list 0:300:20 --free <the 19 training cameras>`, then `--free cam00 --fixed <the first step>` on `cam00`'s three pairs | 8 processes of 1 thread; CPU | 2 min 40 s to 4 min 17 s, then 11 to 13 s | the floor check (exact) and the correction (turned) |
| `python $T/kfit/write_manifest.py $O/kitchen/manifests/turned/scene.json $O/kitchen/fits/turned.json $O/kitchen/manifests/corrected --train-fit … --label kitchen-turned`; `python $T/kfit/compare.py $K/scene.json <turned> <corrected>` | 4; CPU | seconds | the corrected manifest, read back by the package, and each camera's turn from the exact one |
| `bash $T/queue_kitchen.sh`: `ftgs init roma $O/kitchen/manifests/<turned or corrected>/scene.json $O/init/kitchen_<…>.safetensors` | 16; GPU | 32 min 1 s, 30 min 52 s | the control's inits, at the default stride as M17's |
| `bash $T/run_train.sh K_<turned or corrected>_s<n> plain $O/kitchen/manifests/<…>/scene.json $O/init/kitchen_<…>.safetensors <n>`, n = 0, 1 | 16; GPU | 16 min 25 s to 17 min 2 s | the control's runs; peak 21,822 to 23,838 MiB |
| `bash $T/post_synth.sh <run> <manifest> <init>`: `python scripts/gt_depth.py <checkpoint> $K/scene.json $G/kitchen --out …` (with the run's own manifest it stops: the pose differs from the ground truth's); `python scripts/reference_flaws.py run <checkpoint> <own manifest or $K/scene.json> $M/reference/kitchen/flags --out …`; `ftgs eval <checkpoint> $K/scene.json --out …`; `python $T/synth_measures.py <checkpoint> <own init or $M/kitchen/init.safetensors> $G/kitchen --scene-name kitchen --out …` | 16; GPU, then CPU | about 4 min | ticket #24's diagnostics, at the exact cameras and at the run's own |
| `bash $T/blind.sh` | 4; CPU | seconds | the nine sheets with their labels hidden, in an order from Python's `random.Random(25)`, and their key |
| `python $T/tables.py`, `python $T/judge.py`, `python $T/ktables.py` | 1; CPU | seconds | the tables of this section |

The private scene's folders, `<m15-runs>` and `<m17-runs>` were read and never written. The scene is named in no output file; the corrected manifests, which hold the copies' own metadata, and the runs' METRICS.json, which hold their manifests' paths, stay in `<surface-runs>`.

**The owner's answer on step 3** (the go for step 4 or a stop), given in session:
- **Finalist 1** (question 2): option (a). Finalist 1 is dropped. The corrected manifests stay on file, and finalists 2 to 4 train on the manifest's cameras.
- **The left copy's second seed** (question 1): run, at seed 1 for the baseline and `F1m`; its result is above ("The left copy's second seed"). It moved the named measure on the left copy by the rule, after the answer was given. Asked again with that result in hand, the owner confirmed the drop (2026-11-17, in session): finalist 1 stays dropped, and finalists 2 to 4 train on the manifest's cameras.
- **The sheets** (question 3) await the owner's judgement with their labels hidden; question 4 was not taken up.
- **Step 4: go.**

**Date:** 2026-11-17.

## 5. Step 4: finalist 2, the alignment-tolerant pixel loss

*Ticket #26's section.* Finalist 2's patch search, timed on one training image, then its arms, run as the research document's section 5.4 says and judged by section 4.4's criteria.

**The first step** was run on 2026-11-17, at the owner's go of that day (section 4 here), before any arm or off-arm run, with this branch's code at ef7864b (`src/` and `scripts/` as `main`'s), Python 3.14.10, torch 2.15.0+cu132, gsplat 1.5.3, NumPy 2.5.4 and Pillow 13.0.0 from the shared `.venv`. The hardware was one RTX 4090 (driver 595.99.02), under the shared GPU lock, with 16 CPU threads. During every timing no other compute process held the GPU: `nvidia-smi` listed one desktop application holding 384 MiB, at 0 % utilization. Scripts read tensors, JSON and images and wrote numbers. No image or render of private scene A was looked at.

**The owner's answers at step 4's go** (the ends of sections 2.6, 3 and 4 here):

- **The named measure** is the near-side far-5 cm share, with the tail beyond 25 cm beside it, read by rule (b) with the margin 0.021. The under-two-frames share is reported but is not the criterion (section 2.6 here, question 6, option (a); section 3 here, question 1).
- **Finalist 1 is dropped,** so the arm trains on the manifest's cameras (section 4 here, question 2, option (a)).
- **The off arm** is wrapper v2 with every added weight at 0, at two seeds per copy of private scene A and on `kitchen` (seeds 0 to 2) and `studio` (seed 0), as the baseline is. It is run once, here, and reused by finalists 3 and 4. Section 3 here's plain baseline is reported beside it (section 3 here, question 3, option (b)).
- **Two seeds per arm** stand.

**The patch side** (the PRD's owner item 2). AligNeRF's paper gives it, re-read on arXiv (v1; row T26.1): a rendered patch of 32 × 32 is matched against every sub-patch of a 48 × 48 patch of the training image, "in all of our experiments", so the shift reaches 8 px each way, the window section 5.4 names. **The arm's patch side is 32 px.**

**The term, as the arm runs it** (`<scratch>/ticket-26/align_term.py`, not committed). The paper's Eq. 4 is the minimum over sub-patches of D + λ·√(Δx² + Δy²), with D the squared L2 norm of the patch's difference and λ = 0.01 (row T26.1). E7's L1 replaces D here, as section 5.4 asks:

| | AligNeRF (row T26.1) | The arm |
| --- | --- | --- |
| Patches | 32 × 32, 32 per batch, sampled anew each iteration, in a fine-tuning stage after a plain pre-training stage | The whole render tiled by 32 × 32 from its top-left corner at every iteration, from the first. Private scene A's 736 × 1280 tiles exactly; on `kitchen` (1352 × 1014) and `studio` (1920 × 1080) the last row (and on `kitchen` the last column) of patches is narrower, 22 px or 24 px |
| The shifts | Every sub-patch of the 48 × 48 patch | Every shift with \|Δy\|, \|Δx\| ≤ 8 px: 289. At the image's edges a patch is compared on its pixels whose shifted target lies inside the image |
| The distance | Squared L2, its normalization not stated | E7's L1: the mean over the patch's compared pixels and the channels |
| The penalty | 0.01 · √(Δx² + Δy²) | The same, added to that mean (but see question 1 below) |
| The choice | The minimum, inside the loss, at every iteration | The same: per patch at every iteration, without a gradient; ties go to the smaller shift, the zero shift first |
| The value | L_PM, with a feature loss on VGG-19's first block that reads the aligned patch, weight 0.05 | The patches' costs at their shifts, averaged with weights equal to their pixel counts. At window 0 it is E7's L1 itself, and nothing else is computed (section 5.4) |
| SSIM and the perceptual term | – | Unchanged: D-8's SSIM and D-89's ꟻLIP read the unshifted image |

- **In the wrapper.** It is ticket #24's wrapper v2 copied (`<scratch>/ticket-26/wrap_train.py`), with one term, `align`, registered: λ_img · (L_align − L1), at weight 1. The total is then E7 with L_img's L1 replaced by the aligned one, and the package's own logged `l1` stays the plain L1. With the weight at 0, the off arm, nothing of the term runs. The copy also records, through a pass-through wrapper of a private method called once per run, where D-51's `auto` placed the frame cache.
- **E10's g.** The term is on the colour render, so its gradient reaches the 2D means through `out.image` before `Trainer.step` reads their gradient. Growth's test against τ_pos and relocation's score read the new loss with no further step, as section 5.4 says.
- **What the test settles for a revision.** SSIM and ꟻLIP do not read the shifted target in this arm. A shift is chosen per patch at every iteration, as the paper does, and is not held.

**The scratch check** (`check_term.py`, on the CPU and on the GPU). It passes:

- **A render that is the image shifted by k pixels costs k times the penalty.** The check covers k = 1 to 8 down, and k = 1, 4 and 8 up, left and right. The diagonals (1, 1), (3, 3) and (5, 5) cost k√2 times it, AligNeRF's Euclidean offset. It holds within 10⁻⁶ on random images of 1280 × 736 and 301 × 247, the second with partial patches, with every patch at the true shift. Beyond the window (k = 10) the value is not k times the penalty, and no shift exceeds the window.
- **On a real frame it holds as a bound.** On a frame of `kitchen` the value stays at or below k times the penalty: the minimum can take a smaller shift where a patch is smooth. At the paper's relative scale (question 1) every patch takes the true shift.
- **A window of 0 gives E7's L1.** The short cut returns `l1_loss`'s value exactly. The search forced at window 0 equals it within 10⁻⁷ relative, with the same gradient. The wrapped term λ_img · (L_align − L1) is 0 with a zero gradient.
- **The batched search that the arm runs** makes the same choice per patch as a plain loop over the 289 shifts, on all 12 baseline renders below, at both scales, and on two random pairs. Its costs agree within 2·10⁻⁶ relative. Replayed as a CUDA graph, it gives the same values bit for bit.

**The term on the baseline's own renders of training cameras.** Each render is the baseline checkpoint at a training camera's pose and time, against that camera's training image: `A_h0_s0` at `cam1`, frames 0, 30, 60, 90 and 119; `kitchen_s0` at `cam05`, frames 0, 100, 200 and 299; `studio_s0` at `0003`, frames 149, 179 and 208 (row T26.2). The table gives the share of pixels whose patch takes a shift other than 0, their mean shift, and how much the term's L1 part lies below E7's L1. Two scales of the penalty are shown: the ticket's, 0.01 against the patch's mean L1, and the paper's relative scale, 0.01 against a sum over the patch's 32 × 32 × 3 values, that is 0.01 / 3,072 ≈ 3.3·10⁻⁶ against the mean.

| Run, frames | E7's L1 | Ticket's scale: pixels moved / mean shift / L1 lower by | Paper's relative scale: pixels moved / mean shift / L1 lower by | Paper's scale: patches at a shift of 4 px or more |
| --- | --- | --- | --- | --- |
| `A_h0_s0`, `cam1`, 5 frames | 0.0098 to 0.0126 | 0.000 to 0.001 / 0.00 px / 0.1 % at most | 0.39 to 0.58 / 1.2 to 2.2 px / 2.5 % to 6.5 % | 16.7 % |
| `kitchen_s0`, `cam05`, 4 frames | 0.0127 to 0.0131 | 0 / 0 / 0 | 0.08 to 0.14 / 0.1 to 0.4 px / 0.3 % to 1.3 % | 2.8 % |
| `studio_s0`, `0003`, 3 frames | 0.0138 to 0.0153 | 0 / 0 / 0 | 0.02 to 0.04 / 0.05 to 0.08 px / 0.1 % to 0.2 % | 0.5 % |

- **At the ticket's scale nothing moves at the end of training.** A trained model's L1 at a training camera is about 0.01 per patch, so a shift of one pixel costs as much as the whole patch's error. A run at this scale can choose shifts only while its renders are far from the images, early in training, and these renders cannot say how long that lasts.
- **At the paper's relative scale, private scene A stands apart.** Between 39 % and 58 % of its pixels' patches take a shift, 17 % of them by 4 px or more, against 8 % to 14 % on `kitchen` and 2 % to 4 % on `studio`, whose cameras agree. That is the order section 2.5 measured for the person's residual after the turns, 0.9 to 4.0 px at the median.

**The search, timed on one training image.** It was timed on the first frame of each set above, synchronized, after 5 warm-up calls, as the median of 100 calls. The term's time includes its value and its backward to a render that requires a gradient, against E7's L1 and its backward:

| Image | The term, graphed (median; 90th percentile) | Of which the search | E7's L1 | Extra memory at peak | A baseline iteration (30,000 in the run's time) | The term's share of it |
| --- | --- | --- | --- | --- | --- | --- |
| Private scene A, 736 × 1280 | 8.1 ms; 8.2 ms | 7.9 ms | 0.06 ms | 69 MiB, and 22 MiB held by the graph | 26 ms | +31 % |
| `kitchen`, 1352 × 1014 | 11.3 ms; 11.3 ms | 11.0 ms | 0.12 ms | 102 MiB, and 31 MiB | 34 ms | +34 % |
| `studio`, 1920 × 1080 | 21.7 ms; 21.8 ms | 21.2 ms | 0.19 ms | 155 MiB, and 48 MiB | 48 ms | +46 % |

- **The first form was slower.** A loop over the 289 shifts took 120 ms, 141 ms and 160 ms on the same images. Batching the shifts and replaying them as one CUDA graph removed the launches; what remains is bound by memory traffic.
- **What this time is.** It is a one-line note, not a benchmark. Section 5.4's assumption of 1 to 2 times a default run holds: the arm's runs should take 1.3 to 1.5 times the baseline's.
- **Memory.** The baseline's runs on private scene A peaked at 23,394 to 23,982 MiB of the card's 24,564 MiB in `nvidia-smi` (section 3 here). The term adds up to 70 MiB there, and it may change where D-51's `auto` puts the frame cache. Both arms record the placement.

**The arm and the off arm, as planned for the owner's go.**

- **The arm.** The default training configuration, 30,000 iterations, through the wrapper copy with `--term align=1`: window 8 px, patch side 32 px, penalty 0.01 at the scale question 1 settles. Nothing else differs.
- **The off arm.** The same script with `--term align=0`, which is wrapper v2.
- **The runs, eight per arm, mirrored.**
  - On private scene A: the centre copy and the left copy at seeds 0 and 1, from the kept stride-4 files `ref_h0_s4` and `ref_h1_s4` (section 3 here).
  - On openftgs-synth: `kitchen` at seeds 0, 1 and 2, and `studio` at seed 0, from M17's init files.
  - The left copy's second seed is one run more than section 5.2's three per arm. The owner's off arm has it, and section 4.4's rule needs two seeds a side to read the left copy.
- **The measures** are section 3 here's, by ticket #24's scripts copied unchanged:
  - the named measure against the kept files, with the tail beside it;
  - the under-two-frames share, near side and whole model;
  - the sharpness ratio and the tiles' coverage;
  - the test-view scores with Q-233's kept-pixel PSNR on the centre copy;
  - on openftgs-synth, `scripts/gt_depth.py`'s shares, `scripts/reference_flaws.py`'s kept-pixel PSNR and the far shares.

  The arm's logs also carry, every 100 iterations, the share of pixels whose patch moved and the mean shift. The sheets of each copy and seed, off arm and arm, are relabelled blind as in section 4 here.
- **GPU hours, estimated from the baseline's times and the term's timing.**

  | Part | Estimate |
  | --- | --- |
  | The off arm: 4 runs on private scene A, 3 on `kitchen`, 1 on `studio` | 54 min + 51 min + 24 min, about 2.1 hours |
  | The arm, at 1.31, 1.34 and 1.46 times those | 70 min + 68 min + 35 min, about 2.9 hours |
  | Renders, scores, `gt_depth.py` and `reference_flaws.py`, about 16 runs | about 0.6 hours |
  | In all | **about 5.6 hours** |

  That is beyond the ticket's 2 to 4 hours, which counted no off arm through the script. The off arm alone is 2.1 hours, and finalists 3 and 4 reuse it.

  Two ways to save time, neither assumed: ticket #24's `A_h0_s0_v2` is already a full v2 run of the centre copy at seed 0, and reusing it as the off arm's run saves 13 minutes. Running the arm on the left copy at one seed, as section 5.2 plans, saves 17 minutes more, but leaves the left copy unreadable by the rule.

**The questions for the owner at the first step.**

1. **The penalty's scale.** The paper's 0.01 is set against a sum of squared differences over a patch, with no normalization stated (row T26.1). The ticket's check, "a shift of k pixels costs k times the penalty, and a window of 0 gives E7's L1", puts the same 0.01 against the patch's mean L1. Against the mean, one pixel of shift costs about as much as a trained model's whole error in a patch.
   - (a) The ticket's scale: 0.01 against the patch's mean L1. On the baseline's renders nothing moves (above). The arm can then differ from the off arm only early in training. If it ends near the off arm, that cannot be told from section 7.2's drop clause, "the share does not move".
   - (b) The paper's relative scale: 0.01 against the patch's sum over its 32 × 32 × 3 values, 3.3·10⁻⁶ against the mean. The penalty is then what the paper calls it, a tie-breaker for blurry patches. On the baseline's renders 39 % to 58 % of private scene A's pixels move, against 2 % to 14 % on openftgs-synth. The check holds as stated, with the penalty in the mean's units.
   - (c) Both, as two arms: about 2.9 GPU hours more.

   **Recommended: (b).** It is the paper's setting (section 1 here: one arm per finalist at its paper's settings), and it is the scale at which the term acts on what section 2.5 measured. A 300-iteration run of each scale on `kitchen` would show how early (a) stops choosing shifts. It takes about 2 minutes and was not run before the go.
2. **The hours:** about 5.6 GPU hours (above), or less by the two savings.
3. **The left copy's second seed for the arm** (above): run it, recommended, or keep section 5.2's one seed.

**The first step's commands.** Each ran with `OMP_NUM_THREADS=16 MKL_NUM_THREADS=16 OPENBLAS_NUM_THREADS=16`, `PYTHONPATH=<worktree>/src:<worktree>/harness/src` and the shared `.venv`'s Python. The CPU ones ran with `CUDA_VISIBLE_DEVICES=` empty, the GPU ones under the shared lock. `$T` is `<scratch>/ticket-26`, `$O` is `<surface-runs>/ticket-26`, `$B` is `<surface-runs>/ticket-24/runs`, `$X` is `<private-scene-A-exp>`, `$K` is `<m15-runs>/kitchen`, `$M` is `<m17-runs>`. The scripts are not committed.

| Command | Device | Time | Note |
| --- | --- | --- | --- |
| `curl -sSL -o $T/dl/aligner.html https://arxiv.org/html/2211.09682`, then a text extraction | CPU | seconds | AligNeRF re-read (row T26.1) |
| `python $T/check_term.py --penalty <0.01 or 0.01/3072> --image $K/images/cam05/000100.png` | CPU, and GPU with `--device cuda` | 20 to 50 s | The scratch check; passes at both scales |
| `python $T/probe.py request <manifest> <camera> <frames> $O/probe/req_<set>.json` | CPU | seconds | Requests at training cameras' poses and times: `$X/scene_h0` `cam1`, `$K` `cam05`, `$M/studio/scene` `0003` |
| `ftgs render $B/<run>/checkpoint --request $O/probe/req_<set>.json --out $O/probe/render_<set>.safetensors`, for `A_h0_s0`, `kitchen_s0`, `studio_s0` | GPU | seconds each | The baseline's renders of training images |
| `python $T/probe.py measure $O/probe/req_<set>.json $O/probe/render_<set>.safetensors $O/probe/probe_<set>.json` | GPU | about 1 min each | The shares moved at both scales; the first form's timing (120 to 160 ms) |
| `python $T/equiv.py <the three pairs of request and renders>` | GPU | about 1 min | The batched and graphed searches against the loop |
| `python $T/prof.py <the same>` | GPU | about 1 min | The table of times |

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
