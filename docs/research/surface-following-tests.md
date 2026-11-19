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

**Sheets on `kitchen`** (added on 2026-11-18 at the owner's request of that day, PRD #16; for steps 3 to 5). Every `kitchen` model trained by then has a labelled sheet of novel views from the side, `<surface-runs>/kitchen-sheets/sheets/<run>.jpg`, with its coverage numbers in `<run>.coverage.json` beside it, and each finalist and seed a side-by-side sheet in `<surface-runs>/kitchen-sheets/compare/`. They are made from one fixed request, `<surface-runs>/kitchen-sheets/requests/kitchen_side_request.json` (27 items; SHA-256 `5560b8a317157d825b53561a519f755498f286feb78bd82298f88e0e856fd27c`), built by section 2.6's rule from M15's manifest (row 880). The choices `kitchen` needed:

- **The centre camera** is `cam00`, the test view (M17's split; `docs/prd/validation-runs.md`). P is the point nearest to all 20 optical axes, which pass it within 1.4 to 8.4 cm, not within 2·10⁻⁷ as on private scene A. The training cameras lie 3° to 65° round from `cam00`, all on its +x side, so the poses at negative yaw (`sL…`) are beyond the rig, and `sR30` to `sR60` lie among training cameras.
- **The yaws** are section 2.6's nine, −90° to 90°. Ticket #5's script was copied with one change: the turn is applied to the manifest's own `world_to_camera`, because `kitchen`'s rotations are orthonormal only to 10⁻⁷. Yaw 0 is then `cam00` exactly, and the other poses agree with the original's to 2.4·10⁻⁷.
- **The times** are frames 33, 114 and 262 of the 300 (1.1 s, 3.8 s and 8.733 s). In each third of the clip, each is the frame with the largest share of dynamic pixels in `cam00`'s ground-truth masks, among the frames whose dynamic points stay at least 80 % inside all nine poses' images. The points are the masked pixels, back-projected by the ground truth's depth (row 880). At those frames that share is 0.81 to 1.00 per pose. Frames 0, 150 and 299 kept 0.46 of them at frame 150, and frames 0 and 299 are near the clip's least dynamic share (`<surface-runs>/kitchen-sheets/requests/kitchen_frame_sweep.json` per frame, `kitchen_side_request_framing.json` for the chosen request).
- **No poses between cameras.** Section 2.6's four were copied from the notes on private scene A, which have no `kitchen` counterpart. In their place, the ground truth's image of `cam00` at each frame is a last column, `gt`.
- **The rest is section 2.6's:** a black background, the model's full SH degree, tiles at 0.4 scale. The labels are shown, since `kitchen` is synthetic.
- **Finalist 1's control** is rendered at both poses at which this section scores it. The exact pose uses the fixed request (sheets `K_<turned or corrected>_s<n>`). Each run's own manifest's pose uses the same rule on ticket #25's turned or corrected manifest: `kitchen_side_request_turned.json` (SHA-256 `d4de5ea450f2130a8fd596e2f18fe93f639f359550ae12c2ee6485c214ae1f3a`) and `kitchen_side_request_corrected.json` (`5d133c796cc35412553a703b083618e756e5c8d410881d090f9309cc76e7b5cc`); those sheets are `…_ownpose` and have no `gt` column, which belongs to the exact pose. The plain baseline at seeds 0 and 1 is the exact-camera reference. The side-by-side sheets `finalist1_kitchen_s<0 or 1>.jpg` stack, top to bottom: corrected at the exact pose and at its own pose, turned likewise, then the plain baseline.

| Command | Device | Time | Note |
| --- | --- | --- | --- |
| `python $T/make_side_request.py $K/scene.json $O/requests/kitchen_side_request.json --centre cam00 --frames 33,114,262`, and the same on ticket #25's `turned` and `corrected` manifests | CPU | under 1 s | The fixed request and finalist 1's own-pose requests; ticket #5's script, copied, with yaw 0 exact |
| `python $T/sweep_frames.py $K/scene.json $G/kitchen scripts/gt_depth.py …`; `python $T/framing_kitchen.py $K/scene.json $G/kitchen $O/requests/kitchen_side_request.json --gt-depth-script scripts/gt_depth.py --out …_framing.json` | CPU | about 1 min | The frame rule's figures over the 300 frames, and the chosen request's framing |
| `FTGS_WORKTREE=<worktree> bash $T/kitchen_sheets.sh <run-dir> <run> [turned or corrected]`: `ftgs render <run-dir>/checkpoint --request <request> --out $O/renders/<run>.safetensors` under the shared GPU lock, then `python $T/sheet_gt.py <renders> <request> $O/sheets/<run>.jpg --label <run> --stats $O/sheets/<run>.coverage.json --gt $G/kitchen` | GPU, then CPU | 6.4 to 8.9 s per render, apart from the wait for the lock | One render per lock; ticket #5's `side_sheet.py`, copied, with the `gt` column added; the renders file is deleted once its sheet is made |
| `python $T/compare_sheets.py $O/compare/finalist<1, 2 or 3>_kitchen_s<n>.jpg --title … <sheets>` | CPU | seconds | The side-by-side sheets, from the per-run sheets at 0.6 scale |

`$T` is `<scratch>/kitchen-sheets`, `$O` is `<surface-runs>/kitchen-sheets`, `$K` is `<m15-runs>/kitchen` and `$G` is `$OPENFTGS_DATASETS/synth/ground_truth`. Each command ran with three CPU threads, with this branch's code at 96659a0 and the shared `.venv`. The 33 renders ran from 21:39 on 2026-11-18 to 01:07 on 2026-11-19, between other runs. Every rendered value is finite; each tile's alpha is in its sheet's coverage file. The plan for the models not trained yet (finalist 4's, and any `kitchen` runs of finalist 2's variants) is `$T/PLAN.md`, applying `$T/kitchen_sheets.sh` to each. Section 7 here records those sheets.

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

**The patch side** (the PRD's owner item 2). AligNeRF's paper gives it, re-read on arXiv (v1; row 865): a rendered patch of 32 × 32 is matched against every sub-patch of a 48 × 48 patch of the training image, "in all of our experiments", so the shift reaches 8 px each way, the window section 5.4 names. **The arm's patch side is 32 px.**

**The term, as the arm runs it** (`<scratch>/ticket-26/align_term.py`, not committed). The paper's Eq. 4 is the minimum over sub-patches of D + λ·√(Δx² + Δy²), with D the squared L2 norm of the patch's difference and λ = 0.01 (row 865). E7's L1 replaces D here, as section 5.4 asks:

| | AligNeRF (row 865) | The arm |
| --- | --- | --- |
| Patches | 32 × 32, 32 per batch, sampled anew each iteration, in a fine-tuning stage after a plain pre-training stage | The whole render tiled by 32 × 32 from its top-left corner at every iteration, from the first. Private scene A's 736 × 1280 tiles exactly; on `kitchen` (1352 × 1014) the last row of patches is 22 px high and the last column 8 px wide, and on `studio` (1920 × 1080) the last row is 24 px high |
| The shifts | Every sub-patch of the 48 × 48 patch, which lies inside the image | Every shift with \|Δy\|, \|Δx\| ≤ 8 px, 289, for which the patch's whole target lies inside the image. A patch at an edge shifts inwards only, so every cost is a mean over a whole patch |
| The distance | Squared L2, its normalization not stated | E7's L1: the mean over the patch's pixels and channels |
| The penalty | 0.01 · √(Δx² + Δy²) | The same, added to that mean (but see question 1 below) |
| The choice | The minimum, inside the loss, at every iteration | The same: per patch at every iteration, without a gradient; ties go to the smaller shift, the zero shift first |
| The value | L_PM, with a feature loss on VGG-19's first block that reads the aligned patch, weight 0.05 | The patches' costs at their shifts, averaged with weights equal to their pixel counts. At window 0 it is E7's L1 itself, and nothing else is computed (section 5.4) |
| SSIM and the perceptual term | – | Unchanged: D-8's SSIM and D-89's ꟻLIP read the unshifted image |

- **In the wrapper.** It is ticket #24's wrapper v2 copied (`<scratch>/ticket-26/wrap_train.py`), with one term, `align`, registered: λ_img · (L_align − L1), at weight 1. The total is then E7 with L_img's L1 replaced by the aligned one, and the package's own logged `l1` stays the plain L1. With the weight at 0, the off arm, nothing of the term runs. In both arms, the off arm included, the copy also records where D-51's `auto` placed the frame cache, through a pass-through wrapper of a private method called once per run.
- **E10's g.** The term is on the colour render, so its gradient reaches the 2D means through `out.image` before `Trainer.step` reads their gradient. Growth's test against τ_pos and relocation's score read the new loss with no further step, as section 5.4 says.
- **What the test settles for a revision.** SSIM and ꟻLIP do not read the shifted target in this arm. A shift is chosen per patch at every iteration, as the paper does, and is not held.

**The scratch check** (`check_term.py`, on the CPU and on the GPU). It passes:

- **A render that is the image shifted by k pixels costs k times the penalty.** The check covers k = 1 to 8 down, and k = 1, 4 and 8 up, left and right. The diagonals (1, 1), (3, 3) and (5, 5) cost k√2 times it, AligNeRF's Euclidean offset. It holds within 10⁻⁶ on random images of 1280 × 736 and 301 × 247, the second with partial patches. It holds for every patch whose target at the true shift lies inside the image, which then takes that shift: 858 to 897 of the 920 patches, and 63 to 72 of the 80. The patches at the edge the shift points to cannot take it, and cost more. Beyond the window (k = 10) no patch costs k times the penalty, and no shift exceeds the window.
- **On a real frame it holds as a bound.** On a frame of `kitchen`, every patch that can reach the true shift costs at most k times the penalty. The minimum can take a smaller shift where a patch is smooth: at a shift of 8 px, only 13 % to 20 % of those patches cost exactly k times the penalty, and at (5, 5) 22 %. At the paper's relative scale (question 1), all of them do.
- **A window of 0 gives E7's L1.** The short cut returns `l1_loss`'s value exactly. The search forced at window 0 equals it within 10⁻⁷ relative, with the same gradient. The wrapped term λ_img · (L_align − L1) is 0 with a zero gradient.
- **The batched search that the arm runs** makes the same choice per patch as a plain loop over the 289 shifts, on all 12 baseline renders below, at both scales, and on two random pairs. Its costs agree within 2·10⁻⁶ relative. Replayed as a CUDA graph, it gives the same values bit for bit, but the graph saves only 0.8 to 1.4 ms and has never run inside `Trainer.step`, so the arm runs the batched search without it.

**The term on the baseline's own renders of training cameras.** Each render is the baseline checkpoint at a training camera's pose and time, against that camera's training image: `A_h0_s0` at `cam1`, frames 0, 30, 60, 90 and 119; `kitchen_s0` at `cam05`, frames 0, 100, 200 and 299; `studio_s0` at `0003`, frames 149, 179 and 208 (row 866). The table gives the share of pixels whose patch takes a shift other than 0, their mean shift, and how much the term's L1 part lies below E7's L1. Two scales of the penalty are shown: the ticket's, 0.01 against the patch's mean L1, and the paper's relative scale, 0.01 against a sum over the patch's 32 × 32 × 3 values, that is 0.01 / 3,072 ≈ 3.3·10⁻⁶ against the mean.

| Run, frames | E7's L1 | Ticket's scale: pixels moved / mean shift / L1 lower by | Paper's relative scale: pixels moved / mean shift / L1 lower by | Paper's scale: share of all patches shifted by more than 3.5 px |
| --- | --- | --- | --- | --- |
| `A_h0_s0`, `cam1`, 5 frames | 0.0098 to 0.0126 | 0.000 to 0.001 / 0.00 px / 0.1 % at most | 0.38 to 0.57 / 1.1 to 2.1 px / 2.4 % to 6.4 % | 15.6 % |
| `kitchen_s0`, `cam05`, 4 frames | 0.0127 to 0.0131 | 0 / 0 / 0 | 0.07 to 0.14 / 0.1 to 0.3 px / 0.2 % to 0.9 % | 2.3 % |
| `studio_s0`, `0003`, 3 frames | 0.0138 to 0.0153 | 0 / 0 / 0 | 0.02 to 0.04 / 0.04 to 0.07 px / 0.1 % to 0.2 % | 0.3 % |

- **At the ticket's scale nothing moves at the end of training.** A trained model's L1 at a training camera is about 0.01 per patch, so a shift of one pixel costs as much as the whole patch's error. A run at this scale can choose shifts only while its renders are far from the images, early in training, and these renders cannot say how long that lasts.
- **At the paper's relative scale, private scene A stands apart from openftgs-synth.** Between 38 % and 57 % of its pixels' patches take a shift, and 15.6 % of all its patches shift by 3.5 px or more. On `kitchen` the figures are 7 % to 14 % and 2.3 %, and on `studio` 2 % to 4 % and 0.3 %, although nothing is misaligned there. The gain is small: the L1 part lies 2.4 % to 6.4 % below E7's L1 on private scene A. These are renders against the same camera's own image, which the baseline has learnt to fit (the shell), so they are no measure of the disagreement between cameras that section 2.5 measured. They show only that at this scale the term finds shifts where the images and the model disagree, more on private scene A than where the cameras agree.
- **An edge effect, removed.** A first form let a patch at an edge shift outwards and compared it on its remaining pixels, down to 1 of 8 columns on `kitchen`'s last column. At the paper's scale, the edge patches then moved two to four times as often as the interior ones on `kitchen` (25 % to 39 % against 7 % to 13 %). With the target kept inside the image they move 23 % to 33 %, and the interior figures are unchanged.

**The search, timed on one training image.** It was timed on the first frame of each set above, synchronized, after 5 warm-up calls, as the median of 100 calls. The term's time includes its value and its backward to a render that requires a gradient, against E7's L1 and its backward:

| Image | The term (median; 90th percentile) | Of which the search | Graphed, for comparison | E7's L1 | Extra memory at peak | A baseline iteration (30,000 in the run's time) | The term's share of it |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Private scene A, 736 × 1280 | 9.0 ms; 9.1 ms | 8.8 ms | 8.1 ms | 0.06 ms | 69 MiB | 26 ms | +34 % |
| `kitchen`, 1352 × 1014 | 12.9 ms; 12.9 ms | 12.5 ms | 11.5 ms | 0.12 ms | 102 MiB | 34 ms | +38 % |
| `studio`, 1920 × 1080 | 22.5 ms; 22.7 ms | 22.0 ms | 21.8 ms | 0.19 ms | 154 MiB | 48 ms | +47 % |

- **The first form was slower.** A loop over the 289 shifts took 120 ms, 141 ms and 160 ms on the same images. Batching the shifts by threes along x brought it to the figures above, which are bound by memory traffic. A CUDA graph, which removes the kernel launches, saves 0.8 to 1.4 ms more and is not used (above).
- **What this time is.** It is a one-line note, not a benchmark, and it is the term timed alone, not inside a training run. Section 5.4's assumption of 1 to 2 times a default run holds: the arm's runs should take 1.3 to 1.5 times the baseline's.
- **Memory.** The baseline's runs on private scene A peaked at 23,394 to 23,982 MiB of the card's 24,564 MiB in `nvidia-smi` (section 3 here). The term adds up to 70 MiB there, and it may change where D-51's `auto` puts the frame cache. Both arms record the placement.

**The owner's answers at the first step** (2026-11-17, in session, relayed by the orchestrating session, after the first step's report):

- **The penalty's scale** (question 1 below): option (c), both. Two arms, window 8 px and patch side 32 px each: `align_rel` at the paper's relative scale as this section translates it (0.01 / 3,072 against the patch's mean L1), and `align_abs` at the ticket's (0.01 against it). The 300-iteration probes at both scales are cleared, as the smoke run.
- **The hours** (questions 2 and 3): about 8.5 GPU hours cleared in all, for the off arm and both arms on both copies of private scene A at seeds 0 and 1, the left copy's second seed included, `kitchen` at seeds 0 to 2 and `studio` at seed 0. Ticket #24's `A_h0_s0_v2` is reused as the off arm's run on the centre copy at seed 0, since it is wrapper v2 at every weight 0 on the same manifest, init and seed. Past about 9.5 hours, the work stops and reports first.
- **The frame-cache record** on the off arm is accepted: it records only.

**The arm and the off arm, as planned for the owner's go.**

- **The arm.** The default training configuration, 30,000 iterations, through the wrapper copy with `--term align=1`: window 8 px, patch side 32 px, penalty 0.01 at the scale question 1 settles. Nothing else differs.
- **The off arm.** The same script with `--term align=0`, which is wrapper v2 with the frame-cache record.
- **First, a smoke run.** The wrapper copy has not run end to end: a 300-iteration run on `kitchen` at weight 0 and at weight 1 comes first, after the go.
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
  | The arm, at 1.34, 1.38 and 1.47 times those | 71 min + 70 min + 35 min, about 2.9 hours |
  | Renders, scores, `gt_depth.py` and `reference_flaws.py`, about 16 runs | about 0.6 hours |
  | In all | **about 5.6 hours** |

  That is beyond the ticket's 2 to 4 hours, which counted no off arm through the script. The off arm alone is 2.1 hours, and finalists 3 and 4 reuse it.

  Two ways to save time, neither assumed: ticket #24's `A_h0_s0_v2` is already a full v2 run of the centre copy at seed 0, and reusing it as the off arm's run saves 13 minutes. Running the arm on the left copy at one seed, as section 5.2 plans, saves 17 minutes more, but leaves the left copy unreadable by the rule.

**The questions for the owner at the first step.**

1. **The penalty's scale.** The paper's 0.01 is set against a sum of squared differences over a patch, with no normalization stated (row 865). The ticket's check, "a shift of k pixels costs k times the penalty, and a window of 0 gives E7's L1", puts the same 0.01 against the patch's mean L1. Against the mean, one pixel of shift costs about as much as a trained model's whole error in a patch.
   - (a) The ticket's scale: 0.01 against the patch's mean L1. On the baseline's renders nothing moves (above). The arm can then differ from the off arm only early in training. If it ends near the off arm, that cannot be told from section 7.2's drop clause, "the share does not move".
   - (b) The paper's relative scale, as this section translates it: 0.01 against the patch's sum over its 32 × 32 × 3 values, 3.3·10⁻⁶ against the mean. This assumes the paper's sum is unnormalized, and it puts L1 where the paper has squared L2, so it is a reading, not the paper's number. The penalty is then what the paper calls it, a tie-breaker for blurry patches. On the baseline's renders 38 % to 57 % of private scene A's pixels move, against 2 % to 14 % on openftgs-synth. The check holds as stated, with the penalty in the mean's units.
   - (c) Both, as two arms: about 2.9 GPU hours more.

   **Recommended: (b).** It is the nearest reading of the paper's setting (section 1 here: one arm per finalist at its paper's settings), and at the end of training it is the only one of the two at which the term acts at all. Either way, the penalty's unit is a thing a revision would have to word, beside section 5.4's list. A 300-iteration run of each scale on `kitchen` would show how early (a) stops choosing shifts. It takes about 2 minutes and was not run before the go.
2. **The hours:** about 5.6 GPU hours (above), or less by the two savings.
3. **The left copy's second seed for the arm** (above): run it, recommended, or keep section 5.2's one seed.

**The first step's commands.** Each ran with `OMP_NUM_THREADS=16 MKL_NUM_THREADS=16 OPENBLAS_NUM_THREADS=16`, `PYTHONPATH=<worktree>/src:<worktree>/harness/src` and the shared `.venv`'s Python. The CPU ones ran with `CUDA_VISIBLE_DEVICES=` empty, the GPU ones under the shared lock. `$T` is `<scratch>/ticket-26`, `$O` is `<surface-runs>/ticket-26`, `$B` is `<surface-runs>/ticket-24/runs`, `$X` is `<private-scene-A-exp>`, `$K` is `<m15-runs>/kitchen`, `$M` is `<m17-runs>`. The scripts are not committed.

| Command | Device | Time | Note |
| --- | --- | --- | --- |
| `curl -sSL -o $T/dl/aligner.html https://arxiv.org/html/2211.09682`, then a text extraction | CPU | seconds | AligNeRF re-read (row 865) |
| `python $T/check_term.py --penalty <0.01 or 0.01/3072> --image $K/images/cam05/000100.png` | CPU, and GPU with `--device cuda` | 20 to 50 s | The scratch check; passes at both scales |
| `python $T/probe.py request <manifest> <camera> <frames> $O/probe/req_<set>.json` | CPU | seconds | Requests at training cameras' poses and times: `$X/scene_h0` `cam1`, `$K` `cam05`, `$M/studio/scene` `0003` |
| `ftgs render $B/<run>/checkpoint --request $O/probe/req_<set>.json --out $O/probe/render_<set>.safetensors`, for `A_h0_s0`, `kitchen_s0`, `studio_s0` | GPU | seconds each | The baseline's renders of training images |
| `python $T/probe.py measure $O/probe/req_<set>.json $O/probe/render_<set>.safetensors $O/probe/probe_<set>.json` | GPU | about 1 min each | The shares moved at both scales; run first with the first form (its timing, 120 to 160 ms, and the edge effect), then again with the arm's |
| `python $T/equiv.py <the three pairs of request and renders>` | GPU | about 1 min | The batched and graphed searches against the loop |
| `python $T/border.py <the same>` | GPU | about 1 min | The edge patches against the interior ones, with the target kept inside the image and without |
| `python $T/prof.py <the same>` | GPU | about 1 min | The table of times |

**The runs** were made on 2026-11-17 and 2026-11-18, at the owner's answers above, with the same code, versions and hardware as the first step. Everything ran under the shared GPU lock with 16 CPU threads, one training at a time. They took about 8.2 GPU hours, from the smoke runs to the last diagnostic, and the first step about 0.3: about 8.5 in all, the hours cleared. Scripts read tensors, JSON, images and mask files and wrote numbers and files, and no image, render or sheet of private scene A was looked at (row 867). Every run wrote only under `<surface-runs>/ticket-26`: a search afterwards found no file newer than the first step under `<private-scene-A-exp>`, `<m15-runs>/kitchen`, `<m17-runs>`'s `kitchen` and `studio`, or tickets #24's and #25's folders.

- **The runs, 23 of them.**
  - The off arm: 7 runs, with `A_h0_s0_v2` reused as the eighth.
  - `align_rel` (runs `REL_…`) and `align_abs` (runs `ABS_…`): 8 runs each.
  - All are 30,000 iterations of the default training configuration, from the kept stride-4 files on private scene A and M17's init files on openftgs-synth.
- **Every run's wrappers** ran 30,000 times each.
- **D-51's `auto` placed the frame cache alike in all three arms,** in the 23 runs made here. It went on the GPU on private scene A and was memory-mapped on `kitchen` and `studio`, so the term's extra memory changed no placement. The reused `A_h0_s0_v2` recorded none.
- **GPU memory** peaked at 23,391 to 23,985 MiB on private scene A, the same as the baseline's 23,394 to 23,982 MiB.
- **Times.**

  | Arm | Private scene A | `kitchen` | `studio` |
  | --- | --- | --- | --- |
  | The off arm | 13 min 2 s to 13 min 29 s | 16 min 38 s to 17 min 3 s | 23 min 16 s |
  | `align_rel` | 17 min 25 s to 17 min 52 s | 23 min 9 s to 23 min 37 s | 34 min 36 s |
  | `align_abs` | 17 min 34 s to 18 min 56 s | 23 min 7 s to 23 min 25 s | 34 min 44 s |

  That is 1.3 to 1.5 times the off arm, as the first step estimated.

**The smoke runs** were 300 iterations on `kitchen` at seed 0 in each arm, with `--config` holding `{"optim": {"iterations": 300}}`. All three ran to the end, with each wrapper's count equal to the iterations. They also showed what each scale does at the start, and the full runs logged it every 100 iterations:

| Iteration | `align_rel`, private scene A: pixels moved / mean shift / L1 lower by | `align_rel`, `kitchen` | `align_rel`, `studio` | `align_abs`, private scene A | `align_abs`, `kitchen` | `align_abs`, `studio` |
| --- | --- | --- | --- | --- | --- | --- |
| 100 | 0.997 / 9.8 to 9.9 px / 14 % | 0.999 / 10.0 px / 14 % | 0.998 / 9.9 px / 11 % | 0.022 to 0.024 / 0.13 to 0.15 px / 0.3 % | 0.018 / 0.11 px / 0.1 % | 0.015 / 0.10 px / 0.2 % |
| 1,000 | 0.99 / 8.4 to 8.7 px / 24 % | 1.00 / 9.4 to 9.5 px / 20 % | 1.00 / 9.7 px / 15 % | 0.025 to 0.027 / 0.11 px / 0.6 % | 0.02 / 0.10 to 0.11 px / 0.3 % | 0.016 / 0.10 px / 0.4 % |
| 3,000 | 0.97 / 6.5 to 6.8 px / 24 % to 27 % | 0.99 / 7.4 to 7.5 px / 19 % | 1.00 / 8.6 px / 18 % | 0.023 to 0.029 / 0.07 to 0.09 px / 0.6 % to 0.7 % | 0.011 / 0.03 px / 0.2 % | 0.013 / 0.07 px / 0.6 % |
| 10,000 | 0.82 to 0.83 / 3.8 to 4.0 px / 19 % to 21 % | 0.78 to 0.79 / 3.1 to 3.2 px / 7 % | 0.89 / 4.5 px / 11 % | 0.013 to 0.016 / 0.04 px / 0.6 % to 0.7 % | 0.001 to 0.002 / 0.00 px / 0 % | 0.008 / 0.04 px / 0.9 % |
| 30,000 | 0.52 / 1.8 to 1.9 px / 10 % to 12 % | 0.10 / 0.23 to 0.24 px / 0.6 % to 0.7 % | 0.12 / 0.42 px / 1.5 % | 0.003 to 0.004 / 0.01 px / 0.2 % to 0.3 % | 0 / 0 / 0 % | 0 / 0 / 0 % |

- **`align_rel` starts with arbitrary shifts.** While the renders are blurred, nearly every patch takes a shift near the window's corner, 9.8 to 10.0 px against 8√2 = 11.3 px at most, since a blurred patch matches the image about equally well at any shift. The paper meets this with a stage of plain training first, at 60 % of the iterations (row 865). This arm aligns from the first iteration, the choice the first step stated, and it shifts 97 % to 100 % of the pixels until iteration 3,000.
- **Later, the two scenes part.** By the end, `kitchen` and `studio` keep 10 % to 12 % of pixels moved, by 0.2 to 0.4 px. Private scene A keeps 52 %, by 1.9 px: the order the first step found on the baseline's renders.
- **`align_abs` is near the off arm throughout.** It moves 1.5 % to 3 % of the pixels by about 0.1 px at the start, and none on openftgs-synth after iteration 10,000.

**The named measure: the near-side far-5 cm share,** with the tail beyond 25 cm beside it, against the kept stride-4 files (each cell is 5 cm / beyond 25 cm), and the other shell measures. Each pair is near side / whole model where two figures are given. The plain baseline is section 3 here's, with `A_h1_s1` from section 4 here.

| Run | Near side: far-5 cm / beyond 25 cm | Near side: far-10 cm | Under two frames: near / whole | Whole model: far-5 cm | Far side: far-5 cm | Cap reached at |
| --- | --- | --- | --- | --- | --- | --- |
| `A_h0_s0`, plain | 0.1401 / 0.0349 | 0.0634 | 0.5856 / 0.452 | 0.224 | 0.535 | 10,800 |
| `A_h0_s1`, plain | 0.1409 / 0.0267 | 0.0583 | 0.5945 / 0.459 | 0.229 | 0.541 | 11,100 |
| `OFF_A_h0_s0` (ticket #24's `A_h0_s0_v2`) | 0.1308 / 0.0461 | 0.0754 | 0.5857 / 0.454 | 0.218 | 0.537 | 10,900 |
| `OFF_A_h0_s1` | 0.1380 / 0.0333 | 0.0644 | 0.5836 / 0.451 | 0.225 | 0.536 | 10,800 |
| `REL_A_h0_s0` | 0.1339 / 0.0479 | 0.0820 | 0.5857 / 0.449 | 0.220 | 0.536 | 14,200 |
| `REL_A_h0_s1` | 0.1278 / 0.0367 | 0.0654 | 0.6001 / 0.462 | 0.212 | 0.515 | 14,600 |
| `ABS_A_h0_s0` | 0.1413 / 0.0349 | 0.0688 | 0.5855 / 0.452 | 0.227 | 0.539 | 11,000 |
| `ABS_A_h0_s1` | 0.1284 / 0.0296 | 0.0618 | 0.5911 / 0.455 | 0.217 | 0.544 | 11,000 |
| `A_h1_s0`, plain | 0.1632 / 0.0438 | 0.0746 | 0.5854 / 0.449 | 0.246 | 0.541 | 9,700 |
| `A_h1_s1`, plain | 0.1526 / 0.0366 | 0.0658 | 0.5882 / 0.448 | 0.239 | 0.540 | 9,900 |
| `OFF_A_h1_s0` | 0.1411 / 0.0379 | 0.0662 | 0.5808 / 0.445 | 0.227 | 0.536 | 10,000 |
| `OFF_A_h1_s1` | 0.1337 / 0.0264 | 0.0561 | 0.5893 / 0.449 | 0.225 | 0.540 | 10,000 |
| `REL_A_h1_s0` | 0.1300 / 0.0235 | 0.0553 | 0.6040 / 0.457 | 0.223 | 0.540 | 12,800 |
| `REL_A_h1_s1` | 0.1402 / 0.0400 | 0.0674 | 0.6015 / 0.455 | 0.228 | 0.537 | 12,700 |
| `ABS_A_h1_s0` | 0.1381 / 0.0481 | 0.0730 | 0.5739 / 0.439 | 0.226 | 0.542 | 9,600 |
| `ABS_A_h1_s1` | 0.1232 / 0.0277 | 0.0538 | 0.5906 / 0.450 | 0.219 | 0.554 | 10,100 |

**Judged by section 4.4's rule** against the off arm, the owner's comparison: both of the arm's seeds beyond both of the off arm's, and the means more than 0.01 apart (0.021 for the tail).

| Copy | `align_rel` against the off arm | `align_abs` against the off arm | The off arm against the plain baseline |
| --- | --- | --- | --- |
| Centre, named measure | not moved (seeds 0.134, 0.128 against 0.131, 0.138; mean 0.004 lower) | not moved (0.141, 0.128; mean 0.000) | not moved (0.131, 0.138 against 0.140, 0.141; both below, mean 0.006 lower) |
| Centre, tail | not moved (mean 0.003 higher) | not moved (mean 0.008 lower) | not moved |
| Left, named measure | not moved (0.130, 0.140 against 0.141, 0.134; mean 0.002 lower) | not moved (0.138, 0.123; mean 0.007 lower) | **lower** by 0.021 (0.141, 0.134 against 0.163, 0.153) |
| Left, tail | not moved | not moved | not moved |
| Under two frames, near side (reported, not the criterion) | centre not moved (mean 0.008 higher); left **higher** by 0.018 | not moved on either copy | not moved |

- **Neither arm moves the named measure,** on either copy, nor the tail.
- **The off arm was needed.** Against the plain baseline both arms count as lower on the left copy: `align_rel` by 0.023 and `align_abs` by 0.027. But the off arm, the same script with the term at 0, is lower by 0.021 too. That is the script's own effect, which section 3 here found on the centre copy and the off arm takes out. On the left copy it is now measured by the rule, with two seeds a side.
- **`align_rel` raises the near side's under-two-frames share on the left copy,** by 0.018 by the rule, and by 0.008 on the centre copy. This is the measure section 5.4 first named for finalist 2, and it moves the wrong way for H2's prediction of fewer short-lived primitives.
- **`align_rel` delays growth's cap,** to 14,200 to 14,600 on the centre copy and 12,700 to 12,800 on the left one, against 10,800 to 10,900 and 10,000 for the off arm. It does the same on `kitchen` (2,300 against 2,100) and `studio` (7,700 against 6,900). The term's gradient reaches E10's g (above), so with the target moved probably fewer primitives pass τ_pos early; that was not measured. `align_abs`'s caps are the off arm's.

**The test-view scores, the sharpness ratio and the tiles' coverage,** as section 3 here takes them. Kept-pixel PSNR is Q-233's, on the centre copy only. Every rendered value is finite.

| Run | Crop PSNR | Crop SSIM₁ | Sharpness ratio | Late | Training cameras: PSNR / ratio | E16 PSNR / SSIM₁ / ꟻLIP | Kept-pixel PSNR | Coverage: between cameras to ±45°; ±60°, ±90° |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `A_h0_s0`, plain | 24.36 | 0.8149 | 0.7262 | 0.7504 | 32.90 / 0.7755 | 24.79 / 0.8187 / 0.1531 | 25.52 | 0.84 to 1.00; 0.50 to 0.90 |
| `A_h0_s1`, plain | 24.31 | 0.8153 | 0.7249 | 0.7467 | 32.79 / 0.7742 | 24.75 / 0.8189 / 0.1545 | 25.47 | 0.80 to 1.00; 0.50 to 1.00 |
| `OFF_A_h0_s0` (ticket #24's `A_h0_s0_v2`) | 24.35 | 0.8148 | 0.7313 | 0.7536 | 32.99 / 0.7733 | 24.80 / 0.8180 / 0.1531 | 25.54 | 0.86 to 1.00; 0.53 to 1.00 |
| `OFF_A_h0_s1` | 24.36 | 0.8153 | 0.7258 | 0.7497 | 32.84 / 0.7740 | 24.66 / 0.8176 / 0.1538 | 25.37 | 0.82 to 1.00; 0.47 to 0.87 |
| `REL_A_h0_s0` | 24.38 | 0.8174 | 0.7220 | 0.7431 | 32.61 / 0.7730 | 24.71 / 0.8196 / 0.1532 | 25.45 | 0.81 to 1.00; 0.51 to 0.87 |
| `REL_A_h0_s1` | 24.36 | 0.8182 | 0.7198 | 0.7422 | 32.42 / 0.7678 | 24.61 / 0.8209 / 0.1529 | 25.32 | 0.82 to 1.00; 0.51 to 0.92 |
| `ABS_A_h0_s0` | 24.40 | 0.8160 | 0.7293 | 0.7493 | 32.93 / 0.7736 | 24.77 / 0.8189 / 0.1525 | 25.51 | 0.87 to 1.00; 0.55 to 0.89 |
| `ABS_A_h0_s1` | 24.40 | 0.8157 | 0.7247 | 0.7469 | 32.78 / 0.7717 | 24.70 / 0.8187 / 0.1539 | 25.42 | 0.82 to 1.00; 0.44 to 0.96 |
| `A_h1_s0`, plain | 23.90 | 0.7473 | 0.7403 | 0.7938 | 31.31 / 0.7719 | 21.92 / 0.7392 / 0.1951 | – | 0.78 to 1.00; 0.55 to 1.00 |
| `A_h1_s1`, plain | 23.71 | 0.7472 | 0.7457 | 0.7969 | 31.59 / 0.7686 | 21.66 / 0.7384 / 0.1976 | – | 0.82 to 1.00; 0.49 to 0.86 |
| `OFF_A_h1_s0` | 23.92 | 0.7487 | 0.7323 | 0.7804 | 31.57 / 0.7727 | 21.77 / 0.7403 / 0.1952 | – | 0.81 to 1.00; 0.53 to 0.89 |
| `OFF_A_h1_s1` | 23.98 | 0.7489 | 0.7353 | 0.7874 | 31.36 / 0.7706 | 21.89 / 0.7387 / 0.1947 | – | 0.80 to 1.00; 0.51 to 0.87 |
| `REL_A_h1_s0` | 24.00 | 0.7502 | 0.7272 | 0.7782 | 31.08 / 0.7682 | 21.83 / 0.7414 / 0.1938 | – | 0.80 to 1.00; 0.47 to 0.89 |
| `REL_A_h1_s1` | 24.07 | 0.7512 | 0.7347 | 0.7907 | 31.22 / 0.7672 | 21.96 / 0.7401 / 0.1940 | – | 0.85 to 1.00; 0.52 to 1.00 |
| `ABS_A_h1_s0` | 24.07 | 0.7492 | 0.7298 | 0.7857 | 31.31 / 0.7729 | 21.91 / 0.7394 / 0.1959 | – | 0.80 to 1.00; 0.55 to 0.94 |
| `ABS_A_h1_s1` | 23.91 | 0.7476 | 0.7422 | 0.7993 | 31.42 / 0.7729 | 21.79 / 0.7405 / 0.1945 | – | 0.80 to 1.00; 0.46 to 0.88 |

- **No test-view score falls by more than 0.3 dB** against the off arm, in either arm on either copy. For `align_rel`, E16 PSNR's mean is 0.06 dB lower on the centre copy and 0.06 dB higher on the left. The kept-pixel PSNR is 0.07 dB lower, and the crop PSNR 0.01 and 0.08 dB higher. For `align_abs` every mean lies within 0.05 dB. Against the plain baseline, `align_rel`'s E16 PSNR is 0.11 dB lower on the centre copy.
- **The sharpness ratio falls for `align_rel` on the centre copy,** beyond what two seeds give. Both seeds, 0.722 and 0.720, lie below both of the off arm's, 0.731 and 0.726. The means are 0.008 apart, against the off arm's spread of 0.0055 and the plain seeds' 0.0013. The late ratio falls with it, 0.743 and 0.742 against 0.754 and 0.750. On the left copy it does not separate (0.727, 0.735 against 0.732, 0.735). `align_abs` does not fall on either copy.
- **The training cameras' PSNR falls for `align_rel`,** by 0.40 dB on the centre copy and 0.31 dB on the left one. That is what the term asks: training images matched up to a shift. It is not a test-view score, so the guard does not apply.
- **The coverage.** Between cameras to ±45° the arms' tiles cover 0.80 to 1.00, against 0.80 to 1.00 for the off arm and 0.78 to 1.00 for the plain baseline. On the side tiles they cover 0.44 to 1.00, against 0.47 to 1.00 and 0.49 to 1.00. The lowest side tile, `ABS_A_h0_s1`'s 0.44, lies 0.03 below the off arm's lowest; every other tile lies within the off arm's or the baseline's range. No tile collapses. (Corrected on 2026-11-18, at the owner's answer below, item 5: the table's last column first held each run's cap, wall time and peak memory, and this line gave the arms' ranges as within the others'.)

**The sheets, for judging with their labels hidden.** There are four sets: the centre copy at seeds 0 and 1, then the left copy at seeds 0 and 1. Each holds the off arm and both arms in an order from Python's `random.Random(26)`, labelled `set<n>-A` to `-C` only. They are `<surface-runs>/ticket-26/sheets/blind/set<n>-<letter>.jpg`, from section 2.6's fixed request, with the key apart in `KEY.json` beside them. Nobody here has looked at them. The owner's judgement is the criterion, and it is open (question 2 below).

**openftgs-synth.** The columns are as in section 3 here: E16 on the test view; `scripts/reference_flaws.py`'s kept-pixel PSNR; `scripts/gt_depth.py`'s residual shares at δ = 0.02·S and the median error of expected depth; section 2.1 here's far shares and under-two-frames share against M17's init, whole model / dynamic side.

| Run | E16 PSNR / SSIM₁ / ꟻLIP | Kept-pixel PSNR | Residual, all pixels: in front / behind | Dynamic pixels: in front / behind | Expected depth, median error, all / dynamic | Far share at 0.12·S | Beyond 25 cm | Under two frames | Cap reached at |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `kitchen_s0`, plain | 29.25 / 0.8787 / 0.0881 | 30.42 | 0.041 / 0.291 | 0.015 / 0.613 | 251 / 56 mm | 0.036 / 0.002 | 0.023 / 0.001 | 0.381 / 0.660 | 2,100 |
| `kitchen_s1`, plain | 30.11 / 0.8803 / 0.0856 | 31.34 | 0.039 / 0.297 | 0.015 / 0.612 | 448 / 304 mm | 0.035 / 0.003 | 0.022 / 0.001 | 0.377 / 0.662 | 2,100 |
| `kitchen_s2`, plain | 29.25 / 0.8800 / 0.0885 | 31.15 | 0.040 / 0.295 | 0.015 / 0.618 | 277 / 102 mm | 0.036 / 0.002 | 0.023 / 0.001 | 0.385 / 0.661 | 2,100 |
| `OFF_kitchen_s0` | 24.88 / 0.8753 / 0.0949 | 29.16 | 0.042 / 0.294 | 0.015 / 0.616 | 247 / 73 mm | 0.035 / 0.002 | 0.023 / 0.001 | 0.382 / 0.662 | 2,100 |
| `OFF_kitchen_s1` | 26.97 / 0.8753 / 0.0934 | 29.73 | 0.041 / 0.295 | 0.016 / 0.616 | 206 / 58 mm | 0.036 / 0.002 | 0.024 / 0.001 | 0.381 / 0.659 | 2,100 |
| `OFF_kitchen_s2` | 27.58 / 0.8772 / 0.0910 | 30.08 | 0.041 / 0.294 | 0.016 / 0.615 | 217 / 44 mm | 0.035 / 0.003 | 0.023 / 0.001 | 0.383 / 0.655 | 2,100 |
| `REL_kitchen_s0` | 30.30 / 0.8801 / 0.0846 | 31.31 | 0.041 / 0.293 | 0.016 / 0.609 | 223 / 56 mm | 0.035 / 0.002 | 0.022 / 0.001 | 0.377 / 0.657 | 2,300 |
| `REL_kitchen_s1` | 27.93 / 0.8778 / 0.0907 | 29.88 | 0.040 / 0.295 | 0.016 / 0.613 | 257 / 78 mm | 0.035 / 0.003 | 0.023 / 0.001 | 0.376 / 0.655 | 2,300 |
| `REL_kitchen_s2` | 28.27 / 0.8780 / 0.0905 | 30.59 | 0.041 / 0.295 | 0.016 / 0.611 | 193 / 44 mm | 0.034 / 0.002 | 0.023 / 0.001 | 0.378 / 0.654 | 2,300 |
| `ABS_kitchen_s0` | 27.45 / 0.8783 / 0.0903 | 30.46 | 0.040 / 0.297 | 0.015 / 0.616 | 179 / 35 mm | 0.037 / 0.002 | 0.024 / 0.001 | 0.381 / 0.659 | 2,100 |
| `ABS_kitchen_s1` | 26.52 / 0.8765 / 0.0917 | 30.61 | 0.042 / 0.295 | 0.016 / 0.612 | 272 / 126 mm | 0.037 / 0.003 | 0.024 / 0.001 | 0.380 / 0.661 | 2,100 |
| `ABS_kitchen_s2` | 28.11 / 0.8790 / 0.0891 | 30.62 | 0.041 / 0.293 | 0.015 / 0.614 | 327 / 103 mm | 0.037 / 0.002 | 0.024 / 0.001 | 0.381 / 0.655 | 2,100 |
| `studio_s0`, plain | 22.24 / 0.7991 / 0.1928; dynamic 24.85 | 22.30 | 0.014 / 0.412 | 0.007 / 0.647 | 144 / 87 mm | 0.041 / 0.001 | 0.068 / 0.001 | 0.387 / 0.710 | 6,800 |
| `OFF_studio_s0` | 22.83 / 0.7986 / 0.1899; dynamic 25.02 | 23.02 | 0.014 / 0.414 | 0.009 / 0.649 | 172 / 102 mm | 0.044 / 0.001 | 0.071 / 0.001 | 0.384 / 0.707 | 6,900 |
| `REL_studio_s0` | 22.36 / 0.7945 / 0.1913; dynamic 24.99 | 22.44 | 0.014 / 0.412 | 0.008 / 0.642 | 148 / 86 mm | 0.042 / 0.001 | 0.069 / 0.001 | 0.385 / 0.707 | 7,700 |
| `ABS_studio_s0` | 22.28 / 0.7991 / 0.1913; dynamic 24.70 | 22.43 | 0.014 / 0.412 | 0.008 / 0.648 | 147 / 87 mm | 0.042 / 0.001 | 0.070 / 0.001 | 0.386 / 0.712 | 6,800 |

- **On `kitchen`, no harm by the guards against the off arm.**
  - **SSIM₁:** `align_rel`'s mean is 0.0027 above the off arm's, and `align_abs`'s 0.0020 above.
  - **The residual's shares:** every mean lies within 0.005 of the off arm's. The largest difference is `align_rel`'s share behind the surface on dynamic pixels, 0.0045 lower.
  - **PSNR, kept-pixel PSNR and ꟻLIP:** no arm has all three seeds below all three of the off arm's.
- **The off arm's own effect on `kitchen`.** Its SSIM₁ is 0.8753 to 0.8772, against the plain baseline's 0.8787 to 0.8803: a mean 0.0037 lower, beyond the 0.002 guard. Its PSNR is 24.9 to 27.6 dB against 29.3 to 30.1. So the script with the term at 0 is itself what section 4.4 would call harm on `kitchen` against plain `ftgs train`, while section 3 here's acceptance, on loss means up to iteration 2,200, did not see it.
  - Against the plain baseline, `align_rel`'s SSIM₁ is 0.0010 lower and `align_abs`'s 0.0017 lower, both within the guard.
  - `align_abs`'s PSNR and ꟻLIP are flagged: all three seeds lie below or above all three plain ones.
- **On `studio`, one seed.**
  - `align_rel`'s dynamic-region PSNR is 24.99 dB against the off arm's 25.02 dB, 0.03 lower, within the 0.1 dB flag.
  - `align_abs`'s is 24.70 dB, 0.31 lower, and is flagged. With one seed, and two plain trainings already 0.34 dB apart (section 3 here), the flag cannot be told from the spread.
- **The far shares and the under-two-frames share** are the off arm's on both scenes, within 0.003.

**The reading by section 7.2's row for finalist 2,** with its measure as the owner renamed it (section 2.6 here).

- **Adopt when** the named measure moves with no harm by the other criteria. **Not met:** neither arm moves it, on either copy.
- **Drop when** "the share does not move: the shell is not a reconciliation, or the disagreement is not a shift of a few pixels rigid over a patch". **Met for both arms.** What the drop rests on differs between them:
  - **`align_abs`, the ticket's scale,** is close to an arm that was never on. Its term shifts under 3 % of the pixels by about 0.1 px early in training, and almost none after. Its result says that this setting does nothing, not that alignment does nothing.
  - **`align_rel`, the paper's relative scale,** did act. To the end of training it shifts half of private scene A's pixels by about 2 px, five times as many as on openftgs-synth. Its aligned L1 lies a tenth below the plain L1 at the end, and the shell's measure stays where the off arm has it. By the row's letter that is the drop clause's case. It is weaker evidence than the clause's wording, for two reasons:
    - **Only L1 tolerated the shift.** At iteration 30,000 the L1 term is 16 % to 17 % of the loss on private scene A. SSIM, 23 % to 24 %, and ꟻLIP, 59 %, still read the unshifted image, so most of the loss kept asking for the cameras' versions as they are.
    - **The start was arbitrary.** Nearly every patch sat near the window's corner until iteration 3,000, and 82 % to 83 % still moved at 10,000, before the cap.

    So it is a null for this form of the loss, not evidence against H2.
  - **The named measure's spread.** Over the off arm's and `align_abs`'s four runs per copy, which are near-identical arms, the named measure ranges over 0.013 on the centre copy and 0.018 on the left. `align_rel`'s seeds lie inside or at the edge of that range. This supports "not moved", and shows that two seeds understate the off arm's spread against the 0.01 margin.
  - **The renders turn softer** on the centre copy with `align_rel`: sharpness 0.008 lower, beyond two seeds. The under-two-frames share rises on the left copy. Section 7.2's last column, a measure that moves while the renders turn soft, does not arise, since the measure did not move.
- **What this test does not cover.**
  - The paper's staged form, alignment only after a plain pre-training stage. The arm here aligned from the first iteration and spent its first 3,000 iterations shifting nearly every patch to the window's corner.
  - A shift held for several iterations.
  - SSIM and ꟻLIP on the shifted target.
  - A window wider than 8 px.
- **Recommended: drop this form of finalist 2, by the row.** The named measure did not move at either scale, and the paper's scale softened the centre copy's renders. On openftgs-synth neither arm harmed by the guards, apart from `align_abs`'s one-seed flag on `studio`. What is dropped is L1 alone made shift-tolerant, aligned from the first iteration; the paper's staged form is untested. Rows E11, E22 and E27, which section 2.6 here brought back beside finalist 2, keep their own evidence (sections 2.3 and 2.6 here) and are not tested by this result. Testing the staged form would be a new arm at the owner's call, about 3 GPU hours on both sets.

**Against the ticket's acceptance criteria.**

- **The patch side and the search's time, reported before the arm ran:** done (the first step, above).
- **The scratch check:** passes (above).
- **The arm on both sets, against the baseline that matches its manifests.** Finalist 1 was dropped, so it trained on the manifest's cameras. It ran as two arms at the owner's answer, against the off arm, with the plain baseline beside it.
- **Section 4.4's criteria:**
  - the named measure by the rule, with the tail and the under-two-frames share beside it, near side and whole model;
  - the sharpness ratio;
  - the sheets, keyed but not judged (question 2);
  - the test-view scores against the 0.3 dB guard;
  - on openftgs-synth, the residual's shares and SSIM₁ against the off arm.
- **The reading by section 7.2's row:** above. The owner's go or stop for step 5: below, not yet given.
- **Sources, commands, files changed, privacy:**
  - sources: rows 865 to 867;
  - commands: below;
  - files changed: only this document and the source log;
  - privacy: nothing names private scene A or shows its images.

**The questions for the owner at step 4.**

1. **Finalist 2: dropped, as recommended, or kept** for its staged form (alignment after a plain first stage, as the paper trains), as one more arm?
2. **The sheets** (`<surface-runs>/ticket-26/sheets/blind`, key apart): the owner's judgement with the labels hidden, which this ticket cannot give.
3. **The off arm's effect on `kitchen`.** Wrapper v2 at weight 0 lowers `kitchen`'s SSIM₁ by 0.0037 against plain `ftgs train` over three seeds, beyond the 0.002 guard. On the left copy it lowers the named measure by 0.021, by the rule. Finalists 3 and 4 reuse this off arm, which cancels the script's effect only as far as the off arm's three seeds are a fair draw of it. They may be a low draw: `align_abs`, close to a second off arm through the same script, lies 0.0020 above it in SSIM₁ and 0.9 dB above in PSNR on `kitchen`, so a finalist could gain that much against it for nothing. The finalists' adopted forms, inside the package, would not have the script's effect. The cause is not found: section 3 here's diagnostics located only version 1's shift.

**The phase's commands** (the same conventions as the first step's table, with `$I` for `<surface-runs>/ticket-24/init`, `$G` for `$OPENFTGS_DATASETS/synth/ground_truth` and `$S` for `<surface-runs>/ticket-5/side_request.json`):

| Command | Device | Time | Note |
| --- | --- | --- | --- |
| `bash $T/run_train.sh smoke_kitchen_<off, rel or abs> <arm> $K/scene.json $M/kitchen/init.safetensors 0 --config $O/smoke/cfg300.json` | GPU | 33 to 38 s | The smoke runs |
| `bash $T/run_train.sh <run> off $X/scene_h<c>/scene.json $I/ref_h<c>_s4.safetensors <seed>`, i.e. `python $T/wrap_train.py --term align=0 --stats … -- <scene> --init <init> --out $O/runs/<run> --seed <seed>`; the same with `$K/scene.json` and `$M/kitchen/init.safetensors`, and `$M/studio/scene/scene.json` and `$M/studio/init.safetensors` | GPU | 13 to 23 min | The off arm, 7 runs; `/usr/bin/time -v` and `nvidia-smi` each second |
| The same with `rel` (`--term align=1 --align-window 8 --align-patch 32 --align-penalty 3.2552083333333335e-06`) and `abs` (`… --align-penalty 0.01`) | GPU | 17 to 35 min | The two arms, 8 runs each |
| `bash $T/post_a.sh <run> h<c>`: `ftgs render <checkpoint> --request $S`; `ftgs render <checkpoint> --request $X/requests/heldout_novel.json` (the left copy: `heldout1_novel.json`); `python $T/score_a.py <renders> $X/scene_h<c>/scene.json --heldout <cam0 or cam1> --crop <rect> --stored-masks <surface-runs>/ticket-4/private-scene-A/placeholder_masks.npz`; `python $T/measures_a.py $X/scene_h0/scene.json --out $O/measures/a_runs.json <run>=<checkpoint>=$I/ref_h<c>_s4.safetensors`; `python $T/side_sheet.py …`; `python scripts/run_summary.py …` | GPU, then CPU | about 1.5 min | Ticket #24's post-processing, repointed |
| `bash $T/post_synth.sh <run> <scene name> <scene.json> <init>`: `python scripts/gt_depth.py <checkpoint> <scene.json> $G/<scene>`; `python scripts/reference_flaws.py run <checkpoint> <scene.json> $M/reference/<scene>/flags`; `python $T/synth_measures.py <checkpoint> <init> $G/<scene> --scene-name <scene>`; `python scripts/run_summary.py …` | GPU, then CPU | 2 to 3 min | The same |
| `bash $T/blind.sh` | CPU | seconds | The twelve sheets with their labels hidden, and their key |
| `python $T/tables.py` | CPU | seconds | The tables of this section |

The private scene's folders, `<m15-runs>`, `<m17-runs>` and tickets #24's and #25's folders were read and never written.

**Sheets on `kitchen`** (added on 2026-11-18 at the owner's request of that day). This step's `kitchen` models have labelled sheets in `<surface-runs>/kitchen-sheets/sheets/`: `OFF_kitchen_s0` to `s4`, `PLAIN_kitchen_s0` to `s4`, `REL_`, `ABS_` and `STG_kitchen_s0` to `s2`, the staged arm's included although it is described below. They are made from the fixed request of section 4 here's "Sheets on `kitchen`" (SHA-256 `5560b8a317157d825b53561a519f755498f286feb78bd82298f88e0e856fd27c`): `cam00` turned by section 2.6's nine yaws, at frames 33, 114 and 262, with the ground truth's `cam00` image as a last column. The side-by-side sheets `<surface-runs>/kitchen-sheets/compare/finalist2_kitchen_s<0, 1 or 2>.jpg` stack, top to bottom: `align_rel`, `align_abs`, `align_stg`, the off arm, the plain baseline. Seeds 3 and 4 have no arm, so they have only per-run sheets.

| Command | Device | Time | Note |
| --- | --- | --- | --- |
| `bash $T/kitchen_sheets.sh <run-dir> <run>`, per run; then `python $T/compare_sheets.py $O/compare/finalist2_kitchen_s<n>.jpg --title … <sheets>` | GPU (one render under the lock), then CPU | 6.4 to 8.9 s per render, apart from the wait for the lock | As in section 4 here's table, with the same `$T` and `$O` |

**The owner's answer on step 4** (2026-11-18, in session, relayed by the orchestrating session):

1. **Finalist 2: test the staged form** as one more arm, before step 5's arms. The alignment term is switched on only after a plain first stage, at the paper's relative scale, with `align_rel`'s settings otherwise. The switch-on iteration comes from AligNeRF's own staging if it gives one, or is proposed and stated. It runs on the same runs as the other arms (both copies of private scene A at seeds 0 and 1, `kitchen` at seeds 0 to 2, `studio` at seed 0), and is measured, scored and sheeted blind the same way, against the off arm. About 3 GPU hours.
2. **The off arm's own effect on `kitchen`** (SSIM₁ 0.0037 lower than plain over three seeds): **diagnose first,** before step 5's arms, in about 1 GPU hour, by more off-arm and plain `kitchen` seeds and/or a check of the frame cache's placement, to tell a low draw from a real effect. The rule it is read by is stated before the runs are looked at.
3. **Step 5: go.** Its first step runs meanwhile; its arms wait for the result here.
4. **On the diagnosis's result** (2026-11-18, in session, relayed by the orchestrating session, after the verdict below, undetermined): **the off arm stays the comparison for finalists 3 and 4,** with the SSIM₁ guard on `kitchen` also read against the five plain `kitchen` seeds. **No more diagnosis runs.** The go for step 5's arms stands as given in item 3.
5. **After the staged arm** (2026-11-18, in session, relayed by the orchestrating session):
   - **Finalist 2 is not decided yet.** First its three untested variants are tested, each as a staged arm like `align_stg`: the switch after 18,000, the paper's relative scale, window 8 px and patch side 32 px, unless the variant changes it.
     - V1: a shift held for several iterations instead of re-chosen at every iteration, with the hold's length picked and stated, with its reason, before running.
     - V2: SSIM and ꟻLIP also read on the shifted target.
     - V3: a window wider than 8 px, picked and stated, with the patch and the penalty kept.
   - **On private scene A only:** both copies at seeds 0 and 1, 4 runs per variant, 12 in all, about 3.6 GPU hours. `kitchen` and `studio` are added only for a variant that moves the named measure, and if one does, the work stops and reports rather than running them.
   - **Each variant's term is scratch-checked** as the first step's was, and smoke-run for 300 iterations before the full runs.
   - **Each is judged** by section 4.4's criteria against the off arm, with the plain baseline beside, as the staged arm was, with blind sheets as before (key apart), and read by section 7.2's row, with a combined recommendation for finalist 2. The owner's decision on finalist 2 stays open.
   - **Step 4's first test-view table is corrected:** its coverage column held run metadata (the cap, wall time and peak memory), and now holds the coverage, from each run's records.
6. **After the variants** (2026-11-19, in session, relayed by the orchestrating session), on question 3 below: **V2, `align_full`, is run on openftgs-synth,** `kitchen` at seeds 0 to 2 and `studio` at seed 0, about 1.2 GPU hours, to see its effect there. The decision on finalist 2 follows it. Each `kitchen` run gets its labelled sheet and a side-by-side with `align_stg`, the off arm and the plain baseline.

The diagnosis ran first, then the staged arm, both on 2026-11-18, with this branch's code at a77b838 (`src/` and `scripts/` as at ticket #24's 163bd39 and the first step's ef7864b), the same versions and hardware as above, under the shared GPU lock with 12 CPU threads, one training at a time. Another ticket's work held the lock between commands for seconds. `$T` is now `<scratch>/ticket-26b`, holding copies of `$T`'s scripts above, and `$O` is `<surface-runs>/ticket-26b`. Every new run wrote only there.

**The diagnosis of the off arm's effect on `kitchen`: the design, fixed before any new run had ended.**

- **What it costs nothing to check.** These are records already made (row 869).
  - **The frame cache.** D-51's `auto` placed it by memory-mapping in every off-arm and arm run on `kitchen`. Each record holds `auto`, then `mmap` at the automatic choice. Section 3 here found that `kitchen`'s training frames take 21.8 GiB, more than half the card, so `auto` always resolves to `mmap` there. The plain runs record no placement, but their whole-card memory peaked at 21,474 to 21,814 MiB, which leaves no room for 21.8 GiB of frames beside the model. So the plain runs and the off arm placed the cache alike, and the placement cannot be the cause. No GPU time is spent on it.
  - **Training itself.** Over iterations 29,001 to 30,000, the logged L1 of the off arm's three runs is 0.013183 to 0.013243, against 0.013111 to 0.013241 for the three plain runs. The total is 0.056323 to 0.056662 against 0.055958 to 0.056609. The training images are fitted alike, so any difference lies in the test view.
  - **Earlier runs of the default configuration.** Of M17's `kitchen` runs, only `default-395` (openftgs's eaeb791) has today's default configuration; the others differ in the perceptual loss. Its SSIM₁ is 0.8789. Ticket #24's wrapper v1 run at seed 0, the script with every weight at 0 in its first version, has 0.8794. Both are other code, so they are reported beside the comparison and not pooled.
- **The new runs.** Two plain `ftgs train` runs and two off-arm runs on `kitchen`, at seeds 3 and 4, in the order plain s3, off s3, off s4, plain s4. Interleaving them means that a difference between this session and the earlier ones (another day, 16 threads, the machine otherwise idle) falls on both arms alike. That is about 68 minutes. SSIM₁ is read from each run's own `METRICS.json`, `ftgs train`'s final evaluation on the test view, as in the tables above.
- **The rule.** The statistic is E16's SSIM₁ on `kitchen`'s test view. The plain arm pools section 3 here's seeds 0 to 2 with the new seeds 3 and 4, five runs, and the off arm pools this section's seeds 0 to 2 with the new 3 and 4, five runs, all at the same code. `align_abs`, `align_rel`, `default-395` and wrapper v1 are reported beside the comparison, never pooled into it. Let d be the off arm's mean minus the plain arm's, and p the exact two-sided Mann–Whitney p of the five against the five (SciPy), as section 3 here tested parity.
  - **A real effect** if d < −0.002 (beyond the guard) and p < 0.05.
  - **A low draw** if d ≥ −0.002: over five seeds a side, the off arm's deficit lies within the guard.
  - **Undetermined** otherwise: d < −0.002 with p ≥ 0.05.
  - PSNR and ꟻLIP are reported beside the verdict, as are the new runs alone, two against two. They do not decide it.
- **What each verdict means for the comparisons on `kitchen`.**
  - **A real effect:** the script itself costs `kitchen` SSIM₁. Comparing against the off arm still cancels the effect, so the comparison stands. The cause should be found before a finalist's form goes into the package.
  - **A low draw:** the off arm's three seeds understate it. Finalists 3 and 4 then judge `kitchen`'s guards against all five off-arm seeds, and finalist 2's arms are reread against them beside the three.
  - **Undetermined:** the same as a low draw for the five seeds, with the effect stated as open.
  - The staged arm is judged against the off arm's seeds 0 to 2, as the other arms were, with the five seeds beside.

**The staged arm, `align_stg`: the design.**

- **The switch-on iteration comes from AligNeRF's staging** (row 868). Its pre-training stage is standard per-pixel training "with 60% as many iterations". Its alignment stage follows with 32 patches of 32 × 32 per batch, twice the rays of a standard batch, for "20% as many" iterations, so that both stages together cost about one standard training. The split is therefore 60 % to 40 % of the rays seen. A training iteration here renders one whole training image, so the rays per iteration are constant, and 60 % of the 30,000 iterations is **18,000**.
  - **This is a reading,** since AligNeRF counts a NeRF's rays and this arm counts whole images. Counting iterations instead of rays, the stages are 60 and 20, three quarters to one quarter, which would put the switch at 22,500. That alternative was not run.
  - The total stays at 30,000 iterations, the default, as in every other arm.
- **The arm.** It is `align_rel`'s run with the term gated: window 8 px, patch side 32 px, penalty 0.01 / 3,072 against the patch's mean L1, weight 1. The term acts only at iterations it > 18,000, through the wrapper's own path for an inactive term, so iterations 1 to 18,000 run the off arm's code exactly. The switch falls on a boundary of the log's 100-iteration windows, so every logged mean covers a single stage. The wrapper copy takes a new flag, `--align-start 18000`, and nothing else in it changed.
- **What the default schedule has done by iteration 18,000.**
  - Growth stops at 15,000 (`growth.stop`), and its cap is reached between 2,100 (`kitchen`) and about 11,000 (private scene A) in the off arm. The staged term therefore cannot delay the cap, as `align_rel` did, nor change growth's test against τ_pos.
  - Relocation runs to 25,000, so its score reads the new loss for 7,000 iterations.
  - The means' learning rate has decayed exponentially to 0.01^0.6 ≈ 6 % of its start.
  - AligNeRF's fine-tuning stage also adds convolutional layers and a feature loss. Neither is part of this arm, as in `align_rel`.
- **The smoke run.** A 300-iteration run on `kitchen` at seed 0, with the switch moved to 200, ran to the end with each wrapper's count equal to 300. Its logged rows at iterations 100 and 200 equal the off arm's smoke run to six figures and carry no alignment keys. The keys appear at 300, where 99.9 % of pixels moved by 9.8 px on that still-blurred render, as in `align_rel`'s own smoke run.
- **The runs** are the other arms' eight, in one queue after the diagnosis: `STG_A_h0_s0`, `STG_A_h0_s1`, `STG_A_h1_s0` and `STG_A_h1_s1` from the kept stride-4 files, then `STG_kitchen_s0` to `s2` and `STG_studio_s0` from M17's init files.
- **The measures** are the same, by the same copied scripts, against the off arm (section 4.4's rule) with the plain baseline and `align_rel` beside. The blind sheets pair the off arm and the staged arm for each copy and seed, in an order from `random.Random(2602)`, with the key apart.
- **Estimate.** At the off arm's speed for 60 % of each run and `align_rel`'s for 40 %, the runs take about 15, 19 and 28 minutes each on private scene A, `kitchen` and `studio`, so about 2.4 GPU hours. Post-processing adds about 0.3. With the diagnosis that is about 3.9 of the 4 hours cleared.

**What ran.** The diagnosis's four runs took 16 min 33 s to 16 min 36 s each, 68 minutes in all from the first start to the last end. The staged arm's smoke run, 33 s, ran between the diagnosis's first and second runs. Its eight runs and their post-processing then took 2 hours 36 minutes from the diagnosis's end. That is about 3.7 GPU hours of the 4 cleared. The blind sheets were made afterwards on the CPU from renders already made, so they took no GPU time.

- **Every staged run's wrappers** ran 30,000 times each, and each record holds window 8 px, patch 32 px, penalty 0.01 / 3,072 and start 18,000.
- **The term was off to iteration 18,000.** In all eight runs' logs no row up to iteration 18,000 carries the alignment keys, and every row from 18,100 on does, 120 rows each.
- **D-51's `auto`** put the frame cache on the GPU on private scene A and memory-mapped it on `kitchen` and `studio`, as in the other arms.
- **Times.** The staged runs took 14 min 50 s to 14 min 55 s on private scene A, 19 min 12 s to 19 min 17 s on `kitchen` and 27 min 54 s on `studio`, the estimate's 15, 19 and 28 minutes.
- **GPU memory** peaked at 23,446 to 23,948 MiB on private scene A.

**The diagnosis's result, by its rule.** Each value is E16's SSIM₁ on `kitchen`'s test view, from each run's `METRICS.json`:

| Arm | Seeds 0 to 2 | Seeds 3 and 4 (new) | Mean of five (sample s.d.) |
| --- | --- | --- | --- |
| Plain `ftgs train` | 0.8787, 0.8803, 0.8800 | 0.8787, 0.8767 | 0.8789 (0.0014) |
| The off arm | 0.8753, 0.8753, 0.8772 | 0.8779, 0.8781 | 0.8768 (0.0014) |

- **d = −0.00208** (the off arm's mean minus plain's), and the exact two-sided Mann–Whitney p of five against five is **0.056**.
- **Verdict: undetermined.** d lies below −0.002, by 0.00008, and p ≥ 0.05. By the rule's meaning above, that is treated like a low draw for the five seeds, with the effect stated as open.
- **Beside it, deciding nothing:**
  - PSNR: d = −1.57 dB, p = 0.15. The plain seeds run from 26.48 to 30.11 dB and the off arm's from 24.88 to 28.33 dB.
  - ꟻLIP: d = +0.0036, p = 0.056.
  - **The new runs alone** go the other way. Off 0.8779 and 0.8781 against plain 0.8787 and 0.8767, d = +0.0003. Off-arm seeds 3 and 4 lie above the three earlier ones, and plain seed 4 lies below every earlier plain seed.
  - `default-395`'s 0.8789 and wrapper v1's 0.8794 lie at the plain mean.
- **What it leaves.** Two of the off arm's first three seeds are the lowest of the ten runs, and its seeds 3 and 4 do not repeat the gap. Five seeds a side cannot tell a cost of about 0.002 from a low draw. The owner's answer (item 4 above) settles what follows: the off arm stays the comparison, the guard is read against the five plain seeds as well, and nothing more is run.

**Finalist 2's arms reread against five seeds.** Each figure is the arm's three-seed SSIM₁ mean minus the reference's:

| Arm | Against the off arm's seeds 0 to 2 | Against the off arm's five seeds | Against plain seeds 0 to 2 | Against plain's five seeds |
| --- | --- | --- | --- | --- |
| `align_rel` | +0.0027 | +0.0019 | −0.0010 | −0.0002 |
| `align_abs` | +0.0020 | +0.0012 | −0.0017 | −0.0009 |
| `align_stg` | +0.0028 | +0.0020 | −0.0009 | −0.0001 |

- **No arm breaks the guard** against any of the four references.
- **No arm's PSNR or ꟻLIP is flagged against the five seeds,** off or plain. `align_abs`'s PSNR and ꟻLIP flags and the staged arm's PSNR flag against plain seeds 0 to 2 (below) do not survive: plain seed 4's PSNR, 26.48 dB, lies below at least one seed of every arm, and its ꟻLIP, 0.0938, above every arm's seeds.
- **The reread covers E16's three metrics only.** The new seeds 3 and 4 have `METRICS.json` alone: `post_synth.sh` was not run on them, so they have no kept-pixel PSNR, residual shares or expected depth.

**The staged arm, as it acted.** The log's alignment figures from the switch on, with `align_rel`'s at the same iterations (pixels moved / mean shift / aligned L1 lower than the plain L1 by):

| Iteration | Private scene A, staged | Private scene A, `align_rel` | `kitchen`, staged | `kitchen`, `align_rel` | `studio`, staged | `studio`, `align_rel` |
| --- | --- | --- | --- | --- | --- | --- |
| 18,100 | 0.60 to 0.61 / 2.2 to 2.4 px / 10 % | 0.62 to 0.63 / 2.4 to 2.6 px / 13 % to 14 % | 0.32 to 0.33 / 0.8 to 0.9 px / 2 % | 0.33 to 0.34 / 0.9 to 1.0 px / 2 % | 0.45 / 1.2 px / 4 % | 0.47 / 1.3 px / 4 % |
| 20,000 | 0.57 to 0.59 / 2.2 px / 10 % to 11 % | 0.59 to 0.61 / 2.4 px / 13 % to 14 % | 0.27 / 0.7 px / 2 % | 0.27 / 0.7 px / 2 % | 0.34 / 0.9 px / 3 % | 0.35 / 1.0 px / 3 % |
| 25,000 | 0.54 to 0.56 / 1.9 to 2.0 px / 9 % to 11 % | 0.55 to 0.57 / 2.0 to 2.1 px / 11 % to 13 % | 0.15 to 0.17 / 0.4 px / 1 % | 0.15 to 0.17 / 0.4 px / 1 % | 0.16 / 0.4 px / 2 % | 0.16 / 0.4 px / 2 % |
| 30,000 | 0.50 to 0.51 / 1.7 to 1.8 px / 8 % to 10 % | 0.52 / 1.8 to 1.9 px / 10 % to 12 % | 0.10 / 0.2 px / 0.6 % to 0.7 % | 0.10 / 0.2 px / 0.6 % to 0.7 % | 0.12 / 0.4 px / 1.3 % | 0.12 / 0.4 px / 1.5 % |

- **The staged start is not arbitrary.** At the switch the renders are sharp, and the first logged window moves 60 % of private scene A's pixels by 2.2 to 2.4 px, not 99.7 % by 9.9 px as `align_rel` did at its start.
- **From the switch on, the staged arm moves what `align_rel` moves,** within a few hundredths, and ends where it ends: half of private scene A's pixels by under 2 px, a tenth of `kitchen`'s and `studio`'s by a quarter to half a pixel.
- **Growth's cap** is reached within 200 iterations of the off arm's: 10,700 and 10,900 on the centre copy (the off arm's 10,900 and 10,800), 9,900 and 10,000 on the left one (10,000), 2,100 on `kitchen` (2,100) and 6,800 on `studio` (6,900). `align_rel` delayed it by 200 (`kitchen`) to 3,800 iterations.

**The named measure, the near-side far-5 cm share,** with the tail and the other shell measures, as in the table above. The off arm's and the plain baseline's rows are repeated from it:

| Run | Near side: far-5 cm / beyond 25 cm | Near side: far-10 cm | Under two frames: near / whole | Whole model: far-5 cm | Far side: far-5 cm | Cap reached at |
| --- | --- | --- | --- | --- | --- | --- |
| `A_h0_s0`, plain | 0.1401 / 0.0349 | 0.0634 | 0.5856 / 0.452 | 0.224 | 0.535 | 10,800 |
| `A_h0_s1`, plain | 0.1409 / 0.0267 | 0.0583 | 0.5945 / 0.459 | 0.229 | 0.541 | 11,100 |
| `OFF_A_h0_s0` | 0.1308 / 0.0461 | 0.0754 | 0.5857 / 0.454 | 0.218 | 0.537 | 10,900 |
| `OFF_A_h0_s1` | 0.1380 / 0.0333 | 0.0644 | 0.5836 / 0.451 | 0.225 | 0.536 | 10,800 |
| `STG_A_h0_s0` | 0.1290 / 0.0398 | 0.0744 | 0.5718 / 0.442 | 0.219 | 0.544 | 10,700 |
| `STG_A_h0_s1` | 0.1404 / 0.0387 | 0.0712 | 0.5795 / 0.447 | 0.226 | 0.533 | 10,900 |
| `A_h1_s0`, plain | 0.1632 / 0.0438 | 0.0746 | 0.5854 / 0.449 | 0.246 | 0.541 | 9,700 |
| `A_h1_s1`, plain | 0.1526 / 0.0366 | 0.0658 | 0.5882 / 0.448 | 0.239 | 0.540 | 9,900 |
| `OFF_A_h1_s0` | 0.1411 / 0.0379 | 0.0662 | 0.5808 / 0.445 | 0.227 | 0.536 | 10,000 |
| `OFF_A_h1_s1` | 0.1337 / 0.0264 | 0.0561 | 0.5893 / 0.449 | 0.225 | 0.540 | 10,000 |
| `STG_A_h1_s0` | 0.1355 / 0.0336 | 0.0641 | 0.5798 / 0.442 | 0.227 | 0.541 | 9,900 |
| `STG_A_h1_s1` | 0.1369 / 0.0320 | 0.0595 | 0.5862 / 0.448 | 0.227 | 0.539 | 10,000 |

**Judged by section 4.4's rule** against the off arm, with the plain baseline beside:

| Copy | `align_stg` against the off arm | `align_stg` against the plain baseline |
| --- | --- | --- |
| Centre, named measure | not moved (seeds 0.129, 0.140 against 0.131, 0.138; mean 0.000 higher) | not moved (mean 0.006 lower) |
| Centre, tail | not moved (mean 0.000) | not moved (mean 0.008 higher) |
| Left, named measure | not moved (0.136, 0.137 against 0.141, 0.134; mean 0.001 lower) | **lower** by 0.022 (against 0.163, 0.153), the off arm's own 0.021 again |
| Left, tail | not moved (mean 0.001 higher) | not moved |
| Under two frames, near side (reported, not the criterion) | centre: both seeds (0.572, 0.580) below both of the off arm's (0.586, 0.584), but the mean only 0.0089 lower, inside the 0.01 margin, so not moved; left not moved (mean 0.002 lower) | centre **lower** by 0.014; left not moved |

- **The staged arm does not move the named measure** on either copy, nor the tail.
- **The under-two-frames share on the centre copy** leans the way H2 predicts, the reverse of `align_rel`'s rise, but stays inside the margin against the off arm. Against the plain baseline it counts as lower, and the off arm alone does not (0.586, 0.584 against 0.586, 0.595). With two seeds a side that is reported, not read as a result.

**The test-view scores, the sharpness ratio and the tiles' coverage,** as in the table above. The off arm's rows are repeated from it:

| Run | Crop PSNR | Crop SSIM₁ | Sharpness ratio | Late | Training cameras: PSNR / ratio | E16 PSNR / SSIM₁ / ꟻLIP | Kept-pixel PSNR | Coverage: between cameras to ±45°; ±60°, ±90° |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `OFF_A_h0_s0` | 24.35 | 0.8148 | 0.7313 | 0.7536 | 32.99 / 0.7733 | 24.80 / 0.8180 / 0.1531 | 25.54 | 0.86 to 1.00; 0.53 to 1.00 |
| `OFF_A_h0_s1` | 24.36 | 0.8153 | 0.7258 | 0.7497 | 32.84 / 0.7740 | 24.66 / 0.8176 / 0.1538 | 25.37 | 0.82 to 1.00; 0.47 to 0.87 |
| `STG_A_h0_s0` | 24.30 | 0.8151 | 0.7285 | 0.7438 | 32.59 / 0.7674 | 24.75 / 0.8190 / 0.1527 | 25.48 | 0.84 to 1.00; 0.56 to 0.90 |
| `STG_A_h0_s1` | 24.30 | 0.8161 | 0.7240 | 0.7443 | 32.69 / 0.7674 | 24.75 / 0.8194 / 0.1543 | 25.48 | 0.82 to 1.00; 0.49 to 0.88 |
| `OFF_A_h1_s0` | 23.92 | 0.7487 | 0.7323 | 0.7804 | 31.57 / 0.7727 | 21.77 / 0.7403 / 0.1952 | – | 0.81 to 1.00; 0.53 to 0.89 |
| `OFF_A_h1_s1` | 23.98 | 0.7489 | 0.7353 | 0.7874 | 31.36 / 0.7706 | 21.89 / 0.7387 / 0.1947 | – | 0.80 to 1.00; 0.51 to 0.87 |
| `STG_A_h1_s0` | 23.90 | 0.7490 | 0.7346 | 0.7904 | 31.49 / 0.7729 | 21.80 / 0.7400 / 0.1953 | – | 0.82 to 1.00; 0.55 to 0.98 |
| `STG_A_h1_s1` | 23.94 | 0.7494 | 0.7339 | 0.7861 | 31.24 / 0.7677 | 21.86 / 0.7404 / 0.1949 | – | 0.81 to 1.00; 0.49 to 0.92 |

- **No test-view score falls by more than 0.3 dB** against the off arm. E16 PSNR's mean is 0.02 dB higher on the centre copy and 0.005 dB higher on the left. Kept-pixel PSNR is 0.03 dB higher. Crop PSNR is 0.06 dB lower on the centre copy, both seeds below both of the off arm's, and 0.03 dB lower on the left.
- **The sharpness ratio does not fall beyond two seeds.** On the centre copy it is 0.7285 and 0.7240 against 0.7313 and 0.7258, a mean 0.002 lower, inside the off arm's spread of 0.0055. On the left copy it is 0.0005 higher.
- **The late ratio falls on the centre copy,** as it did for `align_rel`. Both seeds, 0.744 and 0.744, lie below both of the off arm's, 0.754 and 0.750. The mean is 0.008 lower, against the off arm's spread of 0.004. On the left copy it does not separate. The late ratio is reported beside the criterion and is not the criterion.
- **The training cameras' PSNR falls,** by 0.28 dB on the centre copy and 0.10 dB on the left, less than `align_rel`'s 0.40 and 0.31. Their ratio is lower on the centre copy too (0.767 against 0.773 and 0.774). That is what the term asks, as for `align_rel`, and the guard does not apply.
- **The coverage** stays within the off arm's and the baseline's ranges. Every rendered value is finite.

**The sheets, for judging with their labels hidden.** Four sets: the centre copy at seeds 0 and 1, then the left copy at seeds 0 and 1. Each holds the off arm and the staged arm in an order from Python's `random.Random(2602)`, labelled `set<n>-A` and `-B` only. They are `<surface-runs>/ticket-26b/sheets/blind/set<n>-<letter>.jpg`, from section 2.6's fixed request, with the key apart in `KEY.json` beside them. Nobody here has looked at them, and the owner's judgement of them is open (question 2 below).

**openftgs-synth.** The columns are as in the table above. The off arm's and plain baseline's rows are repeated from it:

| Run | E16 PSNR / SSIM₁ / ꟻLIP | Kept-pixel PSNR | Residual, all pixels: in front / behind | Dynamic pixels: in front / behind | Expected depth, median error, all / dynamic | Far share at 0.12·S | Beyond 25 cm | Under two frames | Cap reached at |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `kitchen_s0`, plain | 29.25 / 0.8787 / 0.0881 | 30.42 | 0.041 / 0.291 | 0.015 / 0.613 | 251 / 56 mm | 0.036 / 0.002 | 0.023 / 0.001 | 0.381 / 0.660 | 2,100 |
| `kitchen_s1`, plain | 30.11 / 0.8803 / 0.0856 | 31.34 | 0.039 / 0.297 | 0.015 / 0.612 | 448 / 304 mm | 0.035 / 0.003 | 0.022 / 0.001 | 0.377 / 0.662 | 2,100 |
| `kitchen_s2`, plain | 29.25 / 0.8800 / 0.0885 | 31.15 | 0.040 / 0.295 | 0.015 / 0.618 | 277 / 102 mm | 0.036 / 0.002 | 0.023 / 0.001 | 0.385 / 0.661 | 2,100 |
| `OFF_kitchen_s0` | 24.88 / 0.8753 / 0.0949 | 29.16 | 0.042 / 0.294 | 0.015 / 0.616 | 247 / 73 mm | 0.035 / 0.002 | 0.023 / 0.001 | 0.382 / 0.662 | 2,100 |
| `OFF_kitchen_s1` | 26.97 / 0.8753 / 0.0934 | 29.73 | 0.041 / 0.295 | 0.016 / 0.616 | 206 / 58 mm | 0.036 / 0.002 | 0.024 / 0.001 | 0.381 / 0.659 | 2,100 |
| `OFF_kitchen_s2` | 27.58 / 0.8772 / 0.0910 | 30.08 | 0.041 / 0.294 | 0.016 / 0.615 | 217 / 44 mm | 0.035 / 0.003 | 0.023 / 0.001 | 0.383 / 0.655 | 2,100 |
| `STG_kitchen_s0` | 28.26 / 0.8798 / 0.0878 | 30.58 | 0.041 / 0.295 | 0.016 / 0.611 | 218 / 56 mm | 0.036 / 0.002 | 0.024 / 0.001 | 0.381 / 0.662 | 2,100 |
| `STG_kitchen_s1` | 28.49 / 0.8786 / 0.0896 | 29.69 | 0.040 / 0.297 | 0.016 / 0.613 | 298 / 152 mm | 0.036 / 0.003 | 0.023 / 0.001 | 0.380 / 0.663 | 2,100 |
| `STG_kitchen_s2` | 27.69 / 0.8780 / 0.0916 | 31.21 | 0.041 / 0.294 | 0.016 / 0.618 | 195 / 36 mm | 0.035 / 0.003 | 0.023 / 0.001 | 0.381 / 0.657 | 2,100 |
| `studio_s0`, plain | 22.24 / 0.7991 / 0.1928; dynamic 24.85 | 22.30 | 0.014 / 0.412 | 0.007 / 0.647 | 144 / 87 mm | 0.041 / 0.001 | 0.068 / 0.001 | 0.387 / 0.710 | 6,800 |
| `OFF_studio_s0` | 22.83 / 0.7986 / 0.1899; dynamic 25.02 | 23.02 | 0.014 / 0.414 | 0.009 / 0.649 | 172 / 102 mm | 0.044 / 0.001 | 0.071 / 0.001 | 0.384 / 0.707 | 6,900 |
| `STG_studio_s0` | 21.59 / 0.7948 / 0.1971; dynamic 24.88 | 21.65 | 0.015 / 0.416 | 0.009 / 0.654 | 151 / 96 mm | 0.042 / 0.001 | 0.069 / 0.001 | 0.384 / 0.708 | 6,800 |

- **On `kitchen`, no harm by the guards.**
  - **SSIM₁:** the mean is 0.0028 above the off arm's seeds 0 to 2 and 0.0020 above its five. It is 0.0009 below plain seeds 0 to 2 and 0.0001 below plain's five, the reading the owner's answer adds (item 4 above).
  - **The residual's shares:** every mean lies within 0.0014 of the off arm's.
  - **PSNR, kept-pixel PSNR and ꟻLIP:** none is flagged against the off arm. Against plain seeds 0 to 2 the PSNR is flagged, all three seeds below all three plain ones and the mean 1.39 dB lower. Against plain's five it is not.
- **On `studio`, one seed, the dynamic-region PSNR is flagged.** It is 24.88 dB against the off arm's 25.02 dB, 0.14 dB lower, beyond the 0.1 dB flag. It is 0.03 dB above the plain run's 24.85 dB. As for `align_abs`, one seed cannot tell the flag from the spread between trainings: two plain trainings of `studio` lie 0.34 dB apart (section 3 here). The whole view's PSNR is 1.24 dB below the off arm's and 0.65 dB below plain. On `kitchen` the same column spreads over 3.4 dB between the off arm's five seeds.
- **On `studio`, the residual's shares** lie within 0.005 of the off arm's. The largest gap is on dynamic pixels behind the surface: 0.6539 against 0.6490, 0.0049, on the line but not over it, so it is not worse by the criterion.
- **The far shares and the under-two-frames share** are the off arm's on both scenes, within 0.003.

**The reading by section 7.2's row for finalist 2,** for the staged form, with its measure as the owner renamed it (section 2.6 here).

- **Adopt when** the named measure moves with no harm by the other criteria. **Not met:** the staged arm does not move it on either copy.
- **Drop when** "the share does not move: the shell is not a reconciliation, or the disagreement is not a shift of a few pixels rigid over a patch". **Met.** The staged form removes one of the two reasons `align_rel`'s drop was weak evidence:
  - **The start is no longer arbitrary.** The term met sharp renders, chose shifts of about 2 px on private scene A and about 1 px or less on openftgs-synth, and held them in the same order between the scenes as `align_rel` did. The 12,000 aligned iterations did not move the named measure.
  - **Only L1 tolerated the shift, still.** SSIM and ꟻLIP read the unshifted image throughout, as in `align_rel`.
  - **A limit of its own.** Growth stops at 15,000, before the switch. Only relocation, to 25,000, and the decayed learning rate of the means could respond to the aligned loss. So the staged arm tests whether alignment reshapes a model already grown, not whether it changes where primitives are added.
- **Other criteria.** The sharpness ratio holds. No test-view score falls by more than 0.3 dB. On `kitchen` nothing is harmed. On `studio` the one seed's dynamic-region PSNR is flagged, as `align_abs`'s was. The late ratio on the centre copy falls as it did for `align_rel`. The sheets are open.
- **Recommended: drop finalist 2,** both forms, by the row. Neither L1 aligned from the first iteration nor L1 aligned after AligNeRF's plain first stage moves the named measure. What stays untested is the list above, apart from the staged form: a shift held for several iterations, SSIM and ꟻLIP on the shifted target, and a window wider than 8 px. Rows E11, E22 and E27 keep their own evidence, as above.

**The phase's commands** (the same conventions as above; `$T` is `<scratch>/ticket-26b`, `$O` is `<surface-runs>/ticket-26b`):

| Command | Device | Time | Note |
| --- | --- | --- | --- |
| `ftgs train $K/scene.json --init $M/kitchen/init.safetensors --out $O/runs/PLAIN_kitchen_s<3 or 4> --seed <3 or 4>` and `python $T/wrap_train.py --term align=0 --stats … -- $K/scene.json --init $M/kitchen/init.safetensors --out $O/runs/OFF_kitchen_s<3 or 4> --seed <3 or 4>`, each by `$T/run_train.sh` in the order plain s3, off s3, off s4, plain s4 | GPU | 16 min 33 s to 16 min 36 s | The diagnosis's four runs |
| `python $T/diag.py` | CPU | seconds | The verdict by the rule, PSNR and ꟻLIP beside, and the new runs alone |
| `bash $T/run_train.sh smoke_kitchen_stg stg $K/scene.json $M/kitchen/init.safetensors 0 --config $O/smoke/cfg300.json`, with the switch at 200 | GPU | 33 s | The staged arm's smoke run |
| `bash $T/run_train.sh <run> stg <scene> <init> <seed>`, i.e. `python $T/wrap_train.py --term align=1 --align-window 8 --align-patch 32 --align-penalty 3.2552083333333335e-06 --align-start 18000 --stats … -- <scene> --init <init> --out $O/runs/<run> --seed <seed>`, on the same scenes and init files as the other arms | GPU | 14 min 50 s to 27 min 54 s | The staged arm, 8 runs |
| `bash $T/post_a.sh <run> h<c>` and `bash $T/post_synth.sh <run> <scene name> <scene.json> <init>`, as above | GPU, then CPU | 1.5 to 3 min | The same post-processing |
| `bash $T/blind.sh` | CPU | seconds | The eight sheets with their labels hidden, and their key |
| `python $T/tables.py` | CPU | seconds | The tables here, and the five-seed rereads with a short inline script reading the same `METRICS.json` files |

The private scene's folders, `<m15-runs>`, `<m17-runs>` and tickets #24's, #25's and #26's first folders were read and never written (row 870).

**The questions for the owner after the staged arm.**

1. **Finalist 2: dropped, as recommended,** now that both forms have run, or kept for one of the untested variants above?
2. **The sheets** (`<surface-runs>/ticket-26b/sheets/blind`, key apart), beside the first set (`<surface-runs>/ticket-26/sheets/blind`): the owner's judgement with the labels hidden, which this ticket cannot give.
3. **The off arm's seeds for finalists 3 and 4.** The owner's answer keeps the off arm as the comparison. The diagnosis's rule, fixed beforehand, said that an undetermined verdict means judging `kitchen`'s guards against all five off-arm seeds. The answer does not say whether finalists 3 and 4 read the off arm's three seeds, as the arms here were, or its five. Reading the five is the rule's consequence. For SSIM₁, PSNR and ꟻLIP it costs nothing, since the runs exist. For the guards' other columns, kept-pixel PSNR and the residual's shares, seeds 3 and 4 of both arms would first need `post_synth.sh`, a few GPU minutes each, which this ticket did not run. Either way, the guard is also read against the five plain seeds. Recommended: the five off-arm seeds for SSIM₁, PSNR and ꟻLIP, and seeds 0 to 2 for the other columns unless the owner clears the post-processing. This blocks nothing here; it sets how finalists 3 and 4 read `kitchen`.

**The three variants: the design, fixed before any of their runs** (2026-11-18, at the owner's answer above, item 5). Each is the staged arm, `align_stg`, with one thing changed: the switch after iteration 18,000, the penalty at the paper's relative scale (0.01 / 3,072 against the patch's mean L1), the patch side 32 px, the window 8 px unless the variant changes it, weight 1. Iterations 1 to 18,000 run the off arm's code exactly, as in the staged arm. The wrapper copy (`<scratch>/ticket-26c/wrap_train.py`) takes two new forms of the term, `--align-mode hold` and `--align-mode full`; the window is the existing flag. Nothing else in it changed.

| Variant | Runs | What changes from `align_stg` | The choice, and why |
| --- | --- | --- | --- |
| V1, `align_hold` | `HLD_…` | Each training image's shifts are chosen at its first draw in each block of 4,000 iterations from the switch (iterations 18,001 to 22,000, 22,001 to 26,000 and 26,001 to 30,000) and held for that image's later draws in the block | **The hold is per training image,** keyed by camera and time. Every iteration renders a different image, so a shift field can only be held across draws of the same image. D-26's sampler draws each of a copy's 960 training images (8 cameras × 120 frames) once per pass of 960 iterations, so each image is drawn about 4 times per block: its shifts are chosen once and held for about 3 draws, and about three quarters of the aligned iterations use a held shift. **4,000 iterations** are long enough for the model to meet the same target on several passes, so that the target stops following the render from one draw to the next. Three blocks renew the choice twice while the means' learning rate decays, and the last block starts after relocation ends at 25,000. Not run: a hold per camera across its frames, which would suit a fixed calibration error but not a moving person, and one choice per image for the whole aligned stage |
| V2, `align_full` | `FUL_…` | The whole of E9 reads the shifted target: L1, D-8's SSIM and D-89's ꟻLIP are taken between the render and I(p + d), d the patch's shift, plus λ_img times the penalty, a constant of the choice | **The shifts are chosen as in `align_stg`,** by the aligned L1's cost, then the package's own `total_loss` runs against the shifted target. Under the inside rule every pixel's target lies inside the image, so the L1 part is the aligned L1 exactly. **A limit:** SSIM's window and ꟻLIP's filters straddle the seams between patches with different shifts, where the target is discontinuous. The unshifted terms are computed without a gradient, for the log only, so the package's logged `l1`, `one_minus_ssim` and `perceptual` stay the unshifted values, as in the other arms |
| V3, `align_w16` | `W16_…` | The window is **16 px**, 1,089 shifts against 289; the patch side and the penalty are kept | **The window binds at 8 px on the staged arm's own renders.** Rendered at training cameras' poses (`STG_A_h0_s0` at `cam1` and `STG_A_h1_s0` at `cam2`, frames 0, 30, 60, 90 and 119) against their training images, 4.1 % to 14.1 % of the patches take a shift on the 8 px window's edge. With the window at 16 px, 4.3 % to 14.8 % shift beyond 8 px, the mean shift rises from 1.2 to 3.0 px to 1.7 to 3.9 px, and the aligned L1 lies 3.4 % to 16.9 % below the plain L1 instead of 3.1 % to 15.7 %. On the person, section 2.2 here's epipolar residual has its 90th percentile at 7.0 to 7.6 px over the seven pairs, at the old window's edge; 16 px doubles it. The patch's target is then 64 × 64 |

**The scratch check** (`check_v.py`, on the CPU and on the GPU). It passes:

- **V1, through the wrapper's own function.** In one block, an image's first draw chooses its shifts. A later draw reuses them after the render has changed, at the cost the held shifts give, 0.30 to 0.32 above a fresh choice on random images. Another image chooses its own, and the next block chooses anew. With a block of one iteration every draw chooses, and the value equals `align_stg`'s term exactly.
- **V2, through the wrapper's own function, against the package's `total_loss` with SSIM and ꟻLIP.** The loss equals `total_loss` against the shifted target plus λ_img times the penalty, with the same gradient. The logged aligned L1 equals the search's value within 2·10⁻⁸. At window 0 the loss is the package's own, value equal and gradient within 2·10⁻⁷ of its largest element on the GPU, the rounding of SSIM's convolution there, and exactly equal on the CPU with contiguous inputs, as training images are. For a render that is the image shifted by 2 to 5 px, 1 − SSIM falls from 0.72 to 1.10 against the unshifted image to 0.06 to 0.13 against the shifted target, and ꟻLIP from 0.55 to 0.67 to 0.09 to 0.14. On random images of 1280 × 736 and 301 × 247.
- **V3, at window 16.** A render that is the image shifted by k pixels costs k times the penalty at k = 1, 4, 8, 9, 12 and 16 down, 16 up, 16 right and 12 left, and k√2 times it on the diagonals (5, 5) and (11, 11), for every patch whose target at the true shift lies inside the image. At k = 20 no patch costs k times the penalty and no shift exceeds 16. The batched search makes the plain loop's choice per patch on random and shifted pairs, its costs within 2·10⁻⁶ relative.
- **Time and memory,** on one training image of private scene A (1280 × 736), as the median of 30 synchronized calls after 5 warm-up calls, with the backward, a one-line note: the term at window 8 takes 9.1 ms and at window 16 32.7 ms, the same 70 MiB extra at peak. V2's whole loss takes 27.3 ms against the package's 12.2 ms, at 495 MiB extra at peak against the package's 469 MiB. The baseline's runs on private scene A peak at 23,394 to 23,982 MiB of the card's 24,564 MiB, so V2's 26 MiB more fits.

**The runs, as planned.** A 300-iteration smoke run of each variant and of the off arm on the centre copy at seed 0, with the switch at 200, compared row by row up to iteration 200. Then the 12 runs, one of each variant first (the centre copy at seed 0), so that a failure after the switch shows early, then the other nine. They are measured, scored and sheeted by the same copied scripts against the off arm, with the plain baseline and `align_stg` beside. The blind sheets hold the off arm and the three variants for each copy and seed, in an order from `random.Random(2603)`, labelled `set<n>-A` to `-D`, with the key apart. **Estimate:** about 14, 16 and 20 minutes per run for V1, V2 and V3 (V1 searches on about a quarter of the aligned iterations, V2 adds 15 ms and V3 24 ms per aligned iteration), 3.3 hours for the 12, with 0.3 for post-processing and the smoke runs: about 3.7 GPU hours, against the 3.6 cleared and the 4.5 to stop at.

**Read this first: V2, `align_full`, moves the named measure by section 4.4's rule, and upward, on the left copy.** Its two seeds, 0.1462 and 0.1490, both lie above both of the off arm's, 0.1411 and 0.1337. Their mean is 0.0102 higher, 0.0002 over the 0.01 margin. More of the near side's opacity lies more than 5 cm from the init points, which is the shell growing, not shrinking. On the centre copy it does not move. Against the plain baseline the same runs count as lower by 0.0103, but that is the script's own effect: the off arm lies 0.021 below plain on that copy (above). V1 and V3 do not move the measure on either copy. The owner's answer (item 5 above) adds `kitchen` and `studio` only for a variant that moves the named measure and, if one does, stops the work to report rather than run them. So they were not run, and whether V2 goes on to them is the owner's (question 3 below). A rise is not section 7.2's adopt condition.

**What ran** (2026-11-18 to 2026-11-19). The four smoke runs ran at c8e2d83 and the 12 runs at 87e8718, the commit that holds the design above. Its commit time is the queue's start to the second, so the design was committed before any full run and after the smoke runs, which use the same wrapper copy. The code was the shared `.venv`'s (as in the first step above), on one RTX 4090 under the shared lock, with 16 CPU threads for training and post-processing and 2 for the tables and sheets. Scripts read tensors, JSON and images and wrote numbers and files. No image, render or sheet of private scene A was looked at.

- **The smoke runs,** 300 iterations on the centre copy at seed 0 with the switch at 200, took 17 to 19 s each, and each wrapper count was 300. Every variant's logged rows at iterations 100 and 200 match the off arm's smoke run within 2·10⁻⁷ in every loss, with the same primitive count, and carry no alignment keys. The keys appear at 300, on that still-blurred render: 99.7 % to 99.8 % of pixels moved, by 9.6 px (V1, V2) and 17.4 px (V3, 89 % of patches beyond 8 px). V1's log has its reuse share (0 at the first draw) and V2's has the shifted target's 1 − SSIM and ꟻLIP.
- **The 12 runs** ran in one queue, from 20:34 to 00:03, 3 hours 28 minutes with their post-processing. With the smoke runs that is about 3.5 GPU hours, and the probe and the check above add minutes, against the 3.6 cleared and the 4.5 to stop at. The blind sheets were made afterwards on the CPU from renders already made.
- **Every run's wrappers** ran 30,000 times each, and each record holds window 8 px (V3: 16 px), patch 32 px, penalty 0.01 / 3,072, start 18,000, and the mode, `hold` (with 4,000) or `full`. No row up to iteration 18,000 carries the alignment keys, every row from 18,100 on does, 120 rows each, and every logged value is finite.
- **V1's hold, as counted by the wrapper:** 960 training images, 2,880 choices (each image once per block, in each of the three blocks) and 9,120 reuses, so 76 % of the aligned iterations used a held shift, the design's "about three quarters".
- **Times:** V1 13 min 28 s to 13 min 39 s, V2 16 min 3 s to 16 min 23 s, V3 19 min 35 s to 19 min 42 s, against the estimate's 14, 16 and 20 minutes. D-51's `auto` put the frame cache on the GPU in every run. GPU memory peaked at 23,294 to 23,970 MiB.

**The variants, as they acted.** The log's alignment figures from the switch on, four runs each (pixels moved / mean shift / aligned L1 lower than the plain L1 by), with the staged arm's from the table above:

| Iteration | `align_stg` | V1, `align_hold` | V2, `align_full` | V3, `align_w16` (patches beyond 8 px) |
| --- | --- | --- | --- | --- |
| 18,100 | 0.60 to 0.61 / 2.2 to 2.4 px / 10 % | 0.60 to 0.61 / 2.2 to 2.4 px / 10 % (none reused) | 0.61 to 0.62 / 2.3 to 2.4 px / 11 % | 0.60 to 0.61 / 3.1 to 3.3 px / 11 % (11 % to 12 %) |
| 20,000 | 0.57 to 0.59 / 2.2 px / 10 % to 11 % | 0.58 to 0.60 / 2.2 to 2.3 px / 6 % to 7 % (all reused) | 0.68 to 0.70 / 2.9 to 3.0 px / 21 % to 23 % | 0.58 to 0.61 / 3.0 to 3.4 px / 11 % to 13 % (11 % to 13 %) |
| 25,000 | 0.54 to 0.56 / 1.9 to 2.0 px / 9 % to 11 % | 0.56 to 0.58 / 2.0 to 2.1 px / 6 % to 7 % (all reused) | 0.71 to 0.73 / 3.1 to 3.2 px / 31 % to 33 % | 0.55 to 0.56 / 2.6 to 2.7 px / 11 % to 12 % (9 % to 10 %) |
| 30,000 | 0.50 to 0.51 / 1.7 to 1.8 px / 8 % to 10 % | 0.53 / 1.9 px / 5.5 % to 7 % (all reused) | 0.71 to 0.72 / 3.1 to 3.3 px / 35 % to 37 % | 0.51 / 2.4 to 2.5 px / 10 % to 12 % (8 % to 9 %) |

- **V1 held what it chose.** After each image's first draw in a block, every logged window reuses its shifts. Held shifts lower the L1 by 5.5 % to 7 % against the fresh choice's 10 %. The pixels moved and the shift stay where the staged arm's are, a little higher at the end (1.9 against 1.7 to 1.8 px).
- **V3 used the wider window.** 11 % to 13 % of patches took a shift beyond 8 px after the switch and 8 % to 9 % at the end, as the probe above found on the staged arm's renders. The mean shift is about 1 px larger than the staged arm's throughout, and the L1 lies 1 to 2 points lower.
- **V2 ran away.** Where the other forms' shifts shrink as training goes on, V2's grow, from 2.4 to 3.2 px, over 71 % of pixels, and its aligned L1 ends 35 % to 37 % below the plain L1. The model stops fitting the unshifted image. Its logged unshifted 1 − SSIM rises from 0.072 to 0.074 at the switch to 0.075 to 0.079 at 30,000, where the staged arm's falls to 0.057 (`STG_A_h0_s0`), while its 1 − SSIM against the shifted target falls to 0.048 to 0.050. ꟻLIP goes the same way: unshifted 0.080 to 0.082, shifted 0.057, the staged arm's 0.066. When all three terms read the shifted target, nothing in the loss pins the render to the image, and the search finds larger shifts as the render drifts.
- **Growth's cap** is reached within 200 iterations of the off arm's in every run: 10,700 to 11,000 on the centre copy (the off arm's 10,800 and 10,900) and 9,800 to 10,100 on the left (10,000). Growth stops at 15,000, before the switch, so no variant could change it.

**The named measure, the near-side far-5 cm share,** with the tail and the other shell measures, as in the tables above. The off arm's, the plain baseline's and the staged arm's rows are repeated from them:

| Run | Near side: far-5 cm / beyond 25 cm | Near side: far-10 cm | Under two frames: near / whole | Whole model: far-5 cm | Far side: far-5 cm | Cap reached at |
| --- | --- | --- | --- | --- | --- | --- |
| `A_h0_s0`, plain | 0.1401 / 0.0349 | 0.0634 | 0.5856 / 0.452 | 0.224 | 0.535 | 10,800 |
| `A_h0_s1`, plain | 0.1409 / 0.0267 | 0.0583 | 0.5945 / 0.459 | 0.229 | 0.541 | 11,100 |
| `OFF_A_h0_s0` | 0.1308 / 0.0461 | 0.0754 | 0.5857 / 0.454 | 0.218 | 0.537 | 10,900 |
| `OFF_A_h0_s1` | 0.1380 / 0.0333 | 0.0644 | 0.5836 / 0.451 | 0.225 | 0.536 | 10,800 |
| `STG_A_h0_s0` | 0.1290 / 0.0398 | 0.0744 | 0.5718 / 0.442 | 0.219 | 0.544 | 10,700 |
| `STG_A_h0_s1` | 0.1404 / 0.0387 | 0.0712 | 0.5795 / 0.447 | 0.226 | 0.533 | 10,900 |
| `HLD_A_h0_s0` | 0.1398 / 0.0425 | 0.0791 | 0.5864 / 0.456 | 0.223 | 0.532 | 10,700 |
| `HLD_A_h0_s1` | 0.1357 / 0.0358 | 0.0734 | 0.5871 / 0.454 | 0.224 | 0.542 | 10,700 |
| `FUL_A_h0_s0` | 0.1403 / 0.0378 | 0.0737 | 0.5852 / 0.447 | 0.231 | 0.529 | 10,900 |
| `FUL_A_h0_s1` | 0.1347 / 0.0360 | 0.0750 | 0.5849 / 0.449 | 0.226 | 0.529 | 11,000 |
| `W16_A_h0_s0` | 0.1311 / 0.0355 | 0.0694 | 0.5823 / 0.452 | 0.217 | 0.533 | 10,900 |
| `W16_A_h0_s1` | 0.1567 / 0.0311 | 0.0654 | 0.5910 / 0.459 | 0.239 | 0.536 | 11,000 |
| `A_h1_s0`, plain | 0.1632 / 0.0438 | 0.0746 | 0.5854 / 0.449 | 0.246 | 0.541 | 9,700 |
| `A_h1_s1`, plain | 0.1526 / 0.0366 | 0.0658 | 0.5882 / 0.448 | 0.239 | 0.540 | 9,900 |
| `OFF_A_h1_s0` | 0.1411 / 0.0379 | 0.0662 | 0.5808 / 0.445 | 0.227 | 0.536 | 10,000 |
| `OFF_A_h1_s1` | 0.1337 / 0.0264 | 0.0561 | 0.5893 / 0.449 | 0.225 | 0.540 | 10,000 |
| `STG_A_h1_s0` | 0.1355 / 0.0336 | 0.0641 | 0.5798 / 0.442 | 0.227 | 0.541 | 9,900 |
| `STG_A_h1_s1` | 0.1369 / 0.0320 | 0.0595 | 0.5862 / 0.448 | 0.227 | 0.539 | 10,000 |
| `HLD_A_h1_s0` | 0.1368 / 0.0367 | 0.0604 | 0.5801 / 0.443 | 0.229 | 0.544 | 9,900 |
| `HLD_A_h1_s1` | 0.1365 / 0.0345 | 0.0610 | 0.5788 / 0.442 | 0.226 | 0.536 | 10,100 |
| `FUL_A_h1_s0` | 0.1462 / 0.0349 | 0.0701 | 0.5870 / 0.446 | 0.236 | 0.533 | 9,800 |
| `FUL_A_h1_s1` | 0.1490 / 0.0419 | 0.0661 | 0.5857 / 0.445 | 0.238 | 0.530 | 9,900 |
| `W16_A_h1_s0` | 0.1317 / 0.0268 | 0.0616 | 0.5810 / 0.444 | 0.221 | 0.541 | 9,800 |
| `W16_A_h1_s1` | 0.1305 / 0.0291 | 0.0561 | 0.5880 / 0.449 | 0.222 | 0.540 | 10,100 |

**Judged by section 4.4's rule against the off arm** (both of the variant's seeds beyond both of the off arm's, the means more than 0.01 apart; 0.021 for the tail), with the plain baseline beside:

| Copy and measure | V1, `align_hold` | V2, `align_full` | V3, `align_w16` | Against the plain baseline |
| --- | --- | --- | --- | --- |
| Centre, named measure | not moved (0.140, 0.136 against 0.131, 0.138; mean 0.003 higher) | not moved (0.140, 0.135; mean 0.003 higher) | not moved (0.131, 0.157; mean 0.010 higher, the seeds 0.026 apart) | none moved (V1 0.003 lower, V2 0.003 lower, V3 0.003 higher) |
| Centre, tail | not moved (mean 0.000) | not moved (mean 0.003 lower) | not moved (mean 0.006 lower) | none moved |
| Left, named measure | not moved (0.137, 0.137 against 0.141, 0.134; mean 0.001 lower) | **higher** by 0.0102 (0.146, 0.149) | not moved (0.132, 0.131: both below both, but the mean only 0.006 lower) | all **lower**: V1 by 0.021, V2 by 0.010, V3 by 0.027 (against 0.163, 0.153); the off arm's own fall is 0.021 |
| Left, tail | not moved (mean 0.004 higher) | not moved (mean 0.006 higher) | not moved (mean 0.004 lower) | none moved |
| Under two frames, near side (reported, not the criterion) | not moved on either copy (mean 0.002 higher, 0.006 lower) | not moved (0.000, 0.001 higher) | not moved (0.002 higher, 0.001 lower) | none moved |

- **Only V2 moves the named measure,** on the left copy and the wrong way, by 0.0002 over the margin, with the tail not moved. On the centre copy its mean is 0.003 higher too, inside the margin.
- **V3's left copy** leans the hoped way: both seeds below both of the off arm's, as `align_stg`'s under-two-frames share did on the centre copy, but the mean only 0.006 lower, inside the 0.01 margin, so not moved. On the centre copy its seed 1, 0.157, is the highest of the centre copy's 12 rows, and its two seeds' 0.026 apart is the widest pair there.
- **V1 moves nothing,** within 0.006 of the off arm's mean on every row.

**The test-view scores, the sharpness ratio and the tiles' coverage,** as in the tables above. The off arm's and the staged arm's rows are repeated from them:

| Run | Crop PSNR | Crop SSIM₁ | Sharpness ratio | Late | Training cameras: PSNR / ratio | E16 PSNR / SSIM₁ / ꟻLIP | Kept-pixel PSNR | Coverage: between cameras to ±45°; ±60°, ±90° |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `OFF_A_h0_s0` | 24.35 | 0.8148 | 0.7313 | 0.7536 | 32.99 / 0.7733 | 24.80 / 0.8180 / 0.1531 | 25.54 | 0.86 to 1.00; 0.53 to 1.00 |
| `OFF_A_h0_s1` | 24.36 | 0.8153 | 0.7258 | 0.7497 | 32.84 / 0.7740 | 24.66 / 0.8176 / 0.1538 | 25.37 | 0.82 to 1.00; 0.47 to 0.87 |
| `STG_A_h0_s0` | 24.30 | 0.8151 | 0.7285 | 0.7438 | 32.59 / 0.7674 | 24.75 / 0.8190 / 0.1527 | 25.48 | 0.84 to 1.00; 0.56 to 0.90 |
| `STG_A_h0_s1` | 24.30 | 0.8161 | 0.7240 | 0.7443 | 32.69 / 0.7674 | 24.75 / 0.8194 / 0.1543 | 25.48 | 0.82 to 1.00; 0.49 to 0.88 |
| `HLD_A_h0_s0` | 24.29 | 0.8164 | 0.7171 | 0.7419 | 32.54 / 0.7672 | 24.73 / 0.8191 / 0.1548 | 25.46 | 0.83 to 1.00; 0.51 to 0.92 |
| `HLD_A_h0_s1` | 24.29 | 0.8158 | 0.7197 | 0.7473 | 32.45 / 0.7696 | 24.58 / 0.8185 / 0.1545 | 25.29 | 0.79 to 1.00; 0.46 to 0.87 |
| `FUL_A_h0_s0` | 23.73 | 0.8097 | 0.7000 | 0.7116 | 29.61 / 0.7541 | 24.08 / 0.8157 / 0.1569 | 24.70 | 0.82 to 1.00; 0.57 to 1.00 |
| `FUL_A_h0_s1` | 23.77 | 0.8110 | 0.6995 | 0.7116 | 29.52 / 0.7573 | 24.28 / 0.8174 / 0.1560 | 24.93 | 0.83 to 1.00; 0.52 to 0.87 |
| `W16_A_h0_s0` | 24.29 | 0.8142 | 0.7250 | 0.7458 | 32.84 / 0.7705 | 24.80 / 0.8182 / 0.1528 | 25.54 | 0.84 to 1.00; 0.49 to 0.86 |
| `W16_A_h0_s1` | 24.39 | 0.8161 | 0.7266 | 0.7447 | 32.52 / 0.7696 | 24.78 / 0.8194 / 0.1531 | 25.51 | 0.80 to 1.00; 0.46 to 0.91 |
| `OFF_A_h1_s0` | 23.92 | 0.7487 | 0.7323 | 0.7804 | 31.57 / 0.7727 | 21.77 / 0.7403 / 0.1952 | – | 0.81 to 1.00; 0.53 to 0.89 |
| `OFF_A_h1_s1` | 23.98 | 0.7489 | 0.7353 | 0.7874 | 31.36 / 0.7706 | 21.89 / 0.7387 / 0.1947 | – | 0.80 to 1.00; 0.51 to 0.87 |
| `STG_A_h1_s0` | 23.90 | 0.7490 | 0.7346 | 0.7904 | 31.49 / 0.7729 | 21.80 / 0.7400 / 0.1953 | – | 0.82 to 1.00; 0.55 to 0.98 |
| `STG_A_h1_s1` | 23.94 | 0.7494 | 0.7339 | 0.7861 | 31.24 / 0.7677 | 21.86 / 0.7404 / 0.1949 | – | 0.81 to 1.00; 0.49 to 0.92 |
| `HLD_A_h1_s0` | 23.83 | 0.7479 | 0.7301 | 0.7816 | 31.30 / 0.7684 | 21.63 / 0.7389 / 0.1960 | – | 0.80 to 1.00; 0.51 to 0.88 |
| `HLD_A_h1_s1` | 23.90 | 0.7493 | 0.7365 | 0.7942 | 31.30 / 0.7664 | 21.52 / 0.7400 / 0.1969 | – | 0.83 to 1.00; 0.51 to 0.98 |
| `FUL_A_h1_s0` | 24.11 | 0.7576 | 0.7036 | 0.7490 | 28.23 / 0.7587 | 21.88 / 0.7463 / 0.1913 | – | 0.80 to 1.00; 0.54 to 0.90 |
| `FUL_A_h1_s1` | 24.14 | 0.7583 | 0.7041 | 0.7446 | 28.52 / 0.7588 | 22.25 / 0.7453 / 0.1891 | – | 0.83 to 1.00; 0.53 to 0.90 |
| `W16_A_h1_s0` | 23.82 | 0.7498 | 0.7274 | 0.7800 | 31.18 / 0.7692 | 21.70 / 0.7411 / 0.1948 | – | 0.80 to 1.00; 0.49 to 0.87 |
| `W16_A_h1_s1` | 23.82 | 0.7480 | 0.7323 | 0.7758 | 31.37 / 0.7677 | 21.72 / 0.7393 / 0.1961 | – | 0.84 to 1.00; 0.46 to 0.90 |

- **V2 harms, by section 4.4's criteria.**
  - **On the centre copy its test-view scores fall by more than 0.3 dB:** crop PSNR by 0.61 dB, E16 PSNR by 0.55 dB and kept-pixel PSNR by 0.64 dB, both seeds below both of the off arm's in each.
  - **The sharpness ratio falls beyond two seeds on both copies:** 0.700 and 0.700 against 0.731 and 0.726 (a mean 0.029 lower, the off arm's spread 0.0055), and 0.704 and 0.704 against 0.732 and 0.735 (0.030 lower, spread 0.0030). The late ratio falls by 0.040 and 0.037.
  - **On the left copy its test-view scores rise:** crop PSNR by 0.17 dB, E16 PSNR by 0.24 dB, SSIM₁ by 0.006 and ꟻLIP by 0.005 lower. That copy's test view is a generated camera (decision 4), and section 4.4 counts only falls. With the sharpness ratio down on the same views, it reads as a softer render that matches that camera better. With the named measure rising, not falling, it is not a pass by section 4.4.
  - **The training cameras' PSNR falls by 3.4 and 3.1 dB,** the runaway above: the model fits the shifted targets, not the images.
- **V1's sharpness ratio falls beyond two seeds on the centre copy.** 0.717 and 0.720 lie below both of the off arm's 0.731 and 0.726, a mean 0.010 lower against the spread of 0.0055, and below both of the plain baseline's 0.726 and 0.725 as well. On the left copy it holds (0.0005 lower). No test-view score falls by more than 0.3 dB: the largest fall is E16 PSNR on the left copy, 0.26 dB, both seeds below both. The training cameras' PSNR falls by 0.42 and 0.16 dB.
- **The late ratio,** reported beside the criterion, falls on the centre copy for V1 and V3 as it did for `align_stg`: both seeds below both of the off arm's 0.754 and 0.750 (V1 0.742 and 0.747, V3 0.746 and 0.745), means 0.007 and 0.006 lower against the off arm's spread of 0.004. On the left copy V1's is higher, and V3's two seeds lie below both of the off arm's but within their spread of 0.007.
- **Against the plain baseline,** no variant but V2 falls by more than 0.3 dB in any test-view score; V2's centre-copy falls are 0.58 to 0.68 dB against it too.
- **V3 harms nothing by the criteria.** The sharpness ratio's mean is 0.003 lower on the centre copy, inside the spread. On the left copy it is 0.004 lower against a spread of 0.0030, on the line: seed 1, 0.73229, lies 0.00004 above the off arm's seed 0, so not both below both. No test-view score falls by more than 0.3 dB. The largest fall is crop PSNR on the left copy, 0.13 dB. The training cameras' PSNR falls by 0.24 and 0.19 dB, as the staged arm's did.
- **The coverage** stays within 0.01 of the off arm's and the baseline's lowest values: the variants' lowest are 0.79 to 0.84 between cameras and 0.46 to 0.57 at ±60° and ±90°, against 0.78 and 0.47, and within what the earlier arms showed (`ABS_A_h0_s1`'s 0.44). No tile collapses, and every rendered value is finite.

**The sheets, for judging with their labels hidden.** Four sets: the centre copy at seeds 0 and 1, then the left copy at seeds 0 and 1. Each holds the off arm and the three variants in an order from Python's `random.Random(2603)`, labelled `set<n>-A` to `-D` only. They are `<surface-runs>/ticket-26c/sheets/blind/set<n>-<letter>.jpg`, from section 2.6's fixed request, with the key apart in `KEY.json` beside them. Nobody here has looked at them, and the owner's judgement of them is open (question 2 below).

**The reading by section 7.2's row for finalist 2,** for each variant, with its measure as the owner renamed it (section 2.6 here):

| Criterion | V1, `align_hold` | V2, `align_full` | V3, `align_w16` |
| --- | --- | --- | --- |
| Adopt: the named measure moves, with no harm by the other criteria | **Not met.** It does not move | **Not met.** It moves the wrong way, up, on the left copy, and the variant harms | **Not met.** It does not move. The left copy leans lower, inside the margin |
| Drop: the share does not move | **Met** | **Not met as written:** the share moves, but up. The row's drop reasons still apply, and the harm settles it | **Met** |
| The sharpness ratio | Falls beyond two seeds on the centre copy | Falls beyond two seeds on both copies | Holds. The left copy is on the line |
| Test-view scores, harm at more than 0.3 dB | None | Centre copy: crop, E16 and kept-pixel PSNR fall by 0.55 to 0.64 dB. Left copy: they rise | None |
| What it tests | Whether a target that stops following the render lets alignment reshape the model | Whether SSIM and ꟻLIP, read on the shifted target, let it act | Whether the 8 px window cut off the disagreement |
| The answer | No: a held shift lowers the loss less and moves nothing | No: the loss drifts with the shifts, and the render softens | No: 8 % to 13 % of patches use more than 8 px, and the share does not move |

**The combined recommendation for finalist 2: drop it, all six arms.** `align_rel`, `align_abs`, `align_stg` and the three variants all leave the near-side far-5 cm share where the off arm has it, except V2, which raises it on one copy while softening the render.

- **By the row's reason, the shell is not a rigid shift of a few pixels.** Every variant removes one reason the earlier drops were weak evidence:
  - V1 lets the model meet a fixed target several times.
  - V2 lets SSIM and ꟻLIP tolerate the shift.
  - V3 lets shifts reach the person's epipolar residual and twice it.
- **Read by 7.2's row:**
  - V1 and V3 meet the drop condition as written.
  - V2 meets neither condition. Its rise of 0.0002 over the margin on one copy, with its harm on the centre copy and its softer renders on both, is not a reason to keep the finalist.
- **What stays untested** is narrow:
  - a hold per camera across its frames;
  - the switch at 22,500 (the other reading of AligNeRF's staging, above);
  - V2 with an anchor that keeps the render on the unshifted image;
  - combinations of the variants.

  Each is a smaller step from an arm that moved nothing the hoped way.
- **Rows E11, E22 and E27** keep their own evidence, as above.
- **The decision is the owner's** (question 1 below).

**The variants' commands** (the same conventions as above; `$T` is `<scratch>/ticket-26c`, `$O` is `<surface-runs>/ticket-26c`, `$I` is `<surface-runs>/ticket-24/init`):

| Command | Device | Time | Note |
| --- | --- | --- | --- |
| Requests at `cam1` and `cam2`'s poses and times (frames 0, 30, 60, 90 and 119), `$O/probe/req_A_h<c>.json`, and `ftgs render` of `STG_A_h0_s0` and `STG_A_h1_s0` at them into `$O/probe/render_stg_h<c>.safetensors`, as the first step's probe made them; then `python $T/probe_w.py $O/probe/req_A_h<c>.json $O/probe/render_stg_h<c>.safetensors $O/probe/probe_h<c>.json`, with `--time` on the centre copy, whose output holds the timings | GPU | minutes | The window's use on the staged arm's renders, in the design above |
| `python $T/check_v.py --device cpu` and `--device cuda` | CPU; GPU | minutes | The scratch check above, through the wrapper's own functions |
| `bash $T/queue.sh smoke`, i.e. `ALIGN_START=200 bash $T/run_train.sh smoke_A_h0_<off, v1, v2 or v3> <mode> $X/scene_h0/scene.json $I/ref_h0_s4.safetensors 0 --config $O/smoke/cfg300.json` | GPU | 17 to 19 s each | The four smoke runs |
| `bash $T/queue.sh`, i.e. for each run `bash $T/run_train.sh <run> <v1, v2 or v3> $X/scene_h<c>/scene.json $I/ref_h<c>_s4.safetensors <seed>`, then `bash $T/post_a.sh <run> h<c>`. The run is `python $T/wrap_train.py --term align=1 --align-window <8 or 16> --align-patch 32 --align-penalty 3.2552083333333335e-06 --align-start 18000` plus `--align-mode hold --align-hold 4000` (V1) or `--align-mode full` (V2), `--stats … -- <scene> --init <init> --out $O/runs/<run> --seed <seed>` | GPU, then CPU | 13 min 28 s to 19 min 42 s per run; about 30 s post-processing | The 12 runs, one per variant on the centre copy at seed 0 first, then the other nine; post-processing as above |
| `python $T/tables.py` | CPU, 2 threads | seconds | The tables here, against the off arm with the plain baseline and `align_stg` beside |
| `python $T/facts.py` | CPU, 2 threads | seconds | The smoke runs' rows against the off arm's, the wrappers' records, V1's hold counts, the logs' alignment keys and times |
| `bash $T/blind.sh` | CPU, 2 threads | seconds | The 16 sheets with their labels hidden, and their key |

The private scene's folders and the folders of tickets #24, #25, #26 and #26's first follow-up were read and never written (rows 877, 878).

**The questions for the owner after the variants.**

1. **Finalist 2: dropped, as recommended,** now that the staged arm's three variants have run, or kept for one of the narrower forms above? This decides whether section 7.2's combined arm has a disagreement side from finalist 2.
2. **The sheets** (`<surface-runs>/ticket-26c/sheets/blind`, key apart), beside the first two sets (`<surface-runs>/ticket-26/sheets/blind` and `<surface-runs>/ticket-26b/sheets/blind`): the owner's judgement with the labels hidden, which this ticket cannot give.
3. **V2 on `kitchen` and `studio`.** By the owner's answer they wait, since V2 moved the named measure. Recommended: not run. It moved the measure up, by 0.0002 over the margin on one copy, and harmed on private scene A, so neither outcome on openftgs-synth would make it an adoption. About 1.7 GPU hours with post-processing (three `kitchen` seeds and one `studio`, at the staged arm's times and V2's 15 ms more per aligned iteration) if the owner wants them.

**V2 on openftgs-synth: the design, fixed before its runs** (2026-11-19, at the owner's answer above, item 6).

- **The runs.** `FUL_kitchen_s0` to `s2` on M15's `kitchen` and `FUL_studio_s0` on M17's `studio`, with M17's init files, as the staged arm's were. Each runs V2 exactly as on private scene A: the wrapper copy and the term unchanged (`<scratch>/ticket-26d`, copied from `<scratch>/ticket-26c` with only its paths changed), `--align-mode full`, window 8 px, patch 32 px, penalty 0.01 / 3,072, the switch after 18,000. Each is post-processed as the staged arm's `kitchen` and `studio` runs were: E16's `METRICS.json`, `scripts/gt_depth.py`, `scripts/reference_flaws.py` and the same measures script.
- **A smoke run first,** 300 iterations on `kitchen` at seed 0 with the switch at 200, because V2 had not yet met `kitchen`'s 1352 × 1014 frames, the edge patches they leave, or the memory-mapped frame cache. It ran on 2026-11-19 in 35 s, its wrappers counted 300 each, rows 100 and 200 carry no alignment key, and row 300 carries them all, finite, V2's shifted-target 1 − SSIM and ꟻLIP included.
- **The guards, section 4.4's for openftgs-synth, against the off arm:**
  - On `kitchen`, SSIM₁: V2's three-seed mean not below the off arm's five-seed mean by more than 0.002, with the plain baseline's five seeds beside.
  - On `kitchen`, PSNR and ꟻLIP are flagged when all three of V2's values are worse than all five of the off arm's. Kept-pixel PSNR is flagged against the off arm's seeds 0 to 2, the only ones it was scored on.
  - On both scenes, a residual share is worse when V2's mean differs from the off arm's (seeds 0 to 2 on `kitchen`) by more than 0.005, in front and behind, on all pixels and on dynamic pixels.
  - On `studio`, one seed: dynamic-region PSNR is flagged when it is lower than the off arm's by more than 0.1 dB.
- **What the runs can settle.** Section 7.2's adopt condition is read on private scene A, where V2 raised the named measure instead of lowering it. So openftgs-synth can show harm or no harm, not an adoption. The alignment log is read as well, to see whether the runaway on private scene A happens on `kitchen` and `studio`, where the staged arm's shifts were about 1 px.
- **The estimate.** From the staged arm's times and V2's overhead on private scene A, scaled by the frames' pixels: about 21 minutes per `kitchen` run and 31 minutes for `studio`, with about 1.5 minutes of post-processing each, about 1.6 GPU hours with the smoke run. That is over the 1.2 cleared and under the 1.8 to stop at. `studio` runs last and starts only if its expected end falls before 1.8 hours.

## 6. Step 5: finalist 3, surface terms without a prior

*Ticket #27's section.* Row A5's plane-distance depth checked on one flat, slanted primitive, then finalist 3's arms, flattening, then depth–normal consistency, run as the research document's section 5.5 says and judged by section 4.4's criteria.

**The first step** was run on 2026-11-18, at the owner's go of that day, before any calibration or arm run, with this branch's code at 31b1a2c (`src/`, `scripts/` and `tests/` as `main`'s at 02e3686), Python 3.14.10, torch 2.15.0+cu132, gsplat 1.5.3, NumPy 2.5.4 and SciPy 1.18.1 from the shared `.venv`. The hardware was one RTX 4090 (driver 595.99.02), shared under the GPU lock with another ticket's trainings, so every GPU command here waited its turn between them; the CPU work ran on 4 threads. Scripts read tensors, JSON, the scene manifests and, for the device's probe on trained models, one training image per scene, and wrote numbers (row 875). No image or render of private scene A was looked at. The calibration and the arms ran after it, at the owner's answers below ("The calibration, run" and "The arms, run").

**The owner's answers at step 5's go** (2026-11-18, in session, relayed by the orchestrating session, after section 5 here's questions):

- **Finalist 1 stays dropped,** so both arms train on the manifest's cameras.
- **The off arm** is section 5 here's, wrapper v2 at every added weight 0, reused unchanged: its eight runs, with ticket #24's `A_h0_s0_v2` as the centre copy's seed 0. The plain baseline is reported beside it.
- **The named measure** is the near-side far-5 cm share, with the tail beyond 25 cm beside it, read by section 4.4's rule, the tail with the margin 0.021.
- **Two seeds per arm** on each copy of private scene A, as section 5 here ran them.

**The owner's answers at the first step** (2026-11-18, in session, relayed by the orchestrating session, after the first step's report; questions 1 to 6 below):

- **The calibration:** as proposed, stages 1 and 2 (question 1, option (a)).
- **The hours:** about 8.5 GPU hours cleared for the calibration and both arms; past about 9.5 the work stops and reports first (question 2).
- **The settings:** every recommendation taken. L_flat is σt-weighted and in world units (open choice 1; question 6, option (a)); L_nc reads the plane distance (open choice 2); no oriented start (open choice 3); L_nc's pixels as fixed below, with no image-edge weight (question 4).
- **Memory:** the frame cache's `auto` placement, with the fallbacks cleared in advance if arm 2 does not fit: `perf.frame_cache` `"mmap"` on private scene A, `PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True` on `studio` (question 5, option (a) with (b) and (c)).
- **The comparison** stays the off arm of section 5 here. On `kitchen`, SSIM₁'s guard is also read against the five plain seeds (section 3 here's three, and `PLAIN_kitchen_s3` and `s4` under `<surface-runs>/ticket-26b/runs`), since the off arm's own effect on SSIM₁ against plain runs came out undetermined at that ticket's diagnosis (−0.0021, p = 0.056).
- **After the calibration** the work goes on to the arms without a further stop, unless a rule fails to pick a weight.
- **How `kitchen` is read** (2026-11-18, a later answer, for finalists 3 and 4): against the off arm's five seeds, `OFF_kitchen_s0` to `s2` under `<surface-runs>/ticket-26/runs` and `s3` and `s4` under `<surface-runs>/ticket-26b/runs`, for SSIM₁, PSNR and ꟻLIP; against its seeds 0 to 2 for the kept-pixel PSNR and the residual's shares. The plain baseline's five seeds stay beside it for SSIM₁. This replaces the three-seed off arm of the item above for `kitchen`.

**What the papers give for the settings** (re-read on arXiv; rows 871 to 874, with CoMVS-GS's row 646 not reopened):

| Setting | PGSR (row 871) | 2DGS (row 872) | Gaussian Opacity Fields (row 873) | RaDe-GS (row 874) | CoMVS-GS (row 646) |
| --- | --- | --- | --- | --- | --- |
| Flattening | ‖min(s₁, s₂, s₃)‖₁, weight λ₁ = 100. Whether the norm is a sum or a mean over the primitives is not stated | – (disks) | – | – | PGSR's, at the same weight, from the first iteration |
| The normal | The smallest scale's direction, turned so that its angle with the viewing direction exceeds 90° | The disk's normal, turned to the camera | The ray–Gaussian plane's (row 633) | A plane per primitive and camera (row 632) | PGSR's |
| The depth the term reads | The plane-distance depth, 𝒟 / (N·K⁻¹p̃) (Eqs. 2 to 4) | The median intersection | The intersection depth (row 633) | Its rasterized depth (row 632) | PGSR's |
| The consistency term | An L1 between the composited normal and the depth map's normal, weighted by (1 − the normalized image gradient)², λ₂ = 0.015 | Σᵢ wᵢ(1 − nᵢᵀN), β = 0.05 | 2DGS's, β = 0.05 | w_n = 5 | – |
| Its start | Not stated | Not stated | Not stated | Iteration 15,000 of 30,000, after a stage of rendering losses alone | Iteration 7,000 |

So section 5.5's range, 7,000 to 15,000 of 30,000, rests on CoMVS-GS and RaDe-GS; PGSR states no start, and its weight of 100 does not say what it multiplies. The consistency weights span 0.015 to 5 between papers whose terms are normalized differently, so the papers fix a form and an order of magnitude, not a number for E7 beside them. That is what the calibration below is for.

**The device for the surface terms** (`<scratch>/ticket-27/surface_device.py`, not committed; Q-226's device as ticket #3's script builds it, copied, with two passes). Per primitive, at the image's time and in the training camera's space:

- **The normal** nᵢ is the column of E5's R(q) on the smallest of the three scales, turned so that nᵢ·xᵢ < 0, xᵢ the primitive's camera-space centre: PGSR's (row 871). Ticket #3's carried the rotation's third column, 2DGS's; the smallest scale need not be on it.
- **The plane distance** is dᵢ = nᵢ·xᵢ, PGSR's dᵢ; with the normal turned to the camera it is never positive.
- **Pass N** carries the normal coded as (n/1.25 + 1)/2; **pass D** carries (d/s + 1)/2 with s = 1.25·max |d| (a constant per call), the centre's depth z, and a constant. Decoded with the pixel's alpha: N(p) = 1.25·(2·image − α) = Σwn, 𝒟(p) = s·(2·image − α) = Σwd, and Z(p) = Σwz. The plane depth is D(p) = 𝒟(p) / (N(p)·K⁻¹p̃) at the pixel's centre (u + ½, v + ½), and expected depth Z(p)/α(p).
- **The geometry** is computed from the stored tensors by E1 and E5 directly, as `at_time` computes them, without its concatenation of the SH coefficients over every row; on the finite-difference scene below its values and gradients equal `at_time`'s exactly in float64 (`eq_geometry.py`). The first three smoke runs and `prof_device.py` below ran the first form, through `at_time`; the later probes and the last smoke run ran this one.
- **The margin of 1.25** is the one change to section 2.4's code. With the code (v + 1)/2 of a unit value, the primitive whose |d| is the largest, or a normal of −1 in a component, is coded as exactly 0. E2's clamp at 0 passes that value and stops its gradient. The first finite-difference run below used s = max |d| and failed for every tensor that the plane distance reaches (`means` 1.45 relative, `quats` 2.71). With the margin every code lies in [0.1, 0.9], and every tensor passes. The script asserts that no code is at or below 0. Section 2.4's frontal hand case, the normal (0, 0, −1) coded as 0 in z, had the same edge, which a forward check cannot see.
- **The gradient** is the device's two parts, (i) through the copies' weights and (ii) through the colours, carried back to the model's stored tensors by a scalar on the model's graph whose value is the term's and whose gradient is the two parts' sum. A training step's `backward` then reaches the model, as section 5.2 asks.

**The first step: the plane-distance depth on one flat, slanted primitive.** One opaque primitive at (0.1, −0.05, 4.0) in front of a 160 × 120 camera (f = 100 px), scales (0.6, 0.002, 0.45), so the smallest is on R's second column, the plane turned 40° about x and 25° about y: its camera-space normal is (−0.033, −0.812, −0.583), 54° from the axis. Each depth is held to the depth at which the pixel's ray meets the primitive's plane, at every pixel of the cut, on the gsplat backend and the reference backend in float32 on the GPU and on the reference backend in float64 on the CPU. A second case gives the same primitive an opacity of 0.6, and a control turns the smallest scale to the third column at the identity rotation, facing the camera.

| Case | Alpha cut: pixels, ray–plane depths | Plane depth, largest error: gsplat / reference / float64 | Expected depth, largest error / median (all three alike) |
| --- | --- | --- | --- |
| Slanted, opaque | ≥ 0.5: 421 px, 3.59 to 4.49 | 1.44·10⁻⁶ / 1.52·10⁻⁶ / 1.1·10⁻⁹ (3.5·10⁻⁷ relative in float32) | 0.487 / 0.189 |
| | ≥ 0.1: 1,407 px, 3.29 to 5.07 | 3.1·10⁻⁶ / 3.0·10⁻⁶ / 1.2·10⁻⁹ | 1.067 / 0.348 |
| | ≥ 0.01: 2,809 px, 3.07 to 5.70 | 2.2·10⁻⁵ / 2.1·10⁻⁵ / 1.4·10⁻⁹ | 1.702 / 0.488 |
| Slanted, opacity 0.6 (alpha at most 0.600) | ≥ 0.5: 110 px | 1.45·10⁻⁶ / 1.45·10⁻⁶ / 1.0·10⁻⁹ | 0.212 / 0.095 |
| | ≥ 0.01: 2,500 px | 2.2·10⁻⁵ / 2.2·10⁻⁵ / 1.4·10⁻⁹ | 1.580 / 0.461 |
| Frontal, opaque (control) | ≥ 0.5: 733 px, 4.0 | 0 / 0 / 0 | 0 |
| | ≥ 0.01: 4,894 px | 8.1·10⁻⁶ / 7.6·10⁻⁶ / 1.5·10⁻¹⁴ | 0 |

- **The plane-distance depth returns the plane's depth,** at every pixel of the footprint and on both backends: within 1.5·10⁻⁶ at Q-227's cut in float32, the order of float32's rounding at a depth of 4, and within 1.4·10⁻⁹ in float64, where the rest is the float32 of the stored centre. Row A5's depth passes the check section 5.5 set it.
- **It does not depend on alpha,** as PGSR says (row 871): at an opacity of 0.6 its errors are the opaque case's, since 𝒟 and N are both scaled by the accumulated weight and the ratio cancels it.
- **Expected depth is the centre's depth at every pixel,** E6's one value per primitive, so on this primitive it is wrong by up to 0.49 at Q-227's pixels and 1.7 at the cut 0.01, a quarter and more of the depth. On the frontal control the two depths agree.
- **The decoded normal over alpha** equals the primitive's normal within 1.9·10⁻⁷ in float32 and 2·10⁻¹⁶ in float64.

**The consistency term on the same primitive.** The term is 2DGS's form on the composited normal: per pixel, Σᵢ wᵢ(1 − nᵢᵀN_d) = α − N(p)·N_d(p), N_d the unit normal of the depth map by the cross product of its central differences, PGSR's four neighbours (Eq. 5), turned to the camera; its mean over the pixels whose alpha and whose four neighbours' alphas are at least 0.5 (Q-227's cut). On the opaque slanted primitive, over 349 such pixels:

- **With the plane depth,** N_d agrees with the primitive's normal at every pixel: cosines at least 1 − 1·10⁻⁷ in float32, 1 in float64; the term is 0 within 2·10⁻⁸.
- **With expected depth,** N_d is the camera's axis at every pixel, cosine 0.583 with the normal, the cosine of the slant; the term is 0.318, and 0.236 at an opacity of 0.6. Read on expected depth, the term asks a slanted flat primitive to turn to face the camera, or to give up its weight: the technical risk section 4.3 named.
- **On the frontal control** both forms give 0.

**The term's gradient against finite differences.** On seven flat primitives in front of a 48 × 36 camera, overlapping, at different slants, opacities, times and velocities, in float64 on the reference backend on the CPU: the device's two parts against central differences (step 10⁻⁶) of the whole term, along three random directions per stored tensor, with the term's pixels held fixed. The term's value is 0.315 on the plane depth and 0.468 on expected depth, over 52 pixels.

| Stored tensor | Plane depth: largest relative error / gradient norm | Expected depth: largest relative error / gradient norm |
| --- | --- | --- |
| `means` | 1.9·10⁻⁹ / 0.364 | 4.4·10⁻¹⁰ / 0.453 |
| `times` | 2.5·10⁻⁸ / 0.0103 | 4.6·10⁻⁹ / 0.0127 |
| `log_durations` | 2.9·10⁻⁵ / 1.0·10⁻⁵ | 5.5·10⁻⁶ / 2.5·10⁻⁵ |
| `velocities` | 4.5·10⁻⁸ / 0.0195 | 1.4·10⁻⁸ / 0.0237 |
| `log_scales` | 4.1·10⁻⁹ / 0.100 | 2.7·10⁻⁸ / 0.0609 |
| `quats` | 1.6·10⁻⁹ / 0.581 | 8.9·10⁻¹⁰ / 0.551 |
| `opacity_logits` | 3.3·10⁻⁸ / 0.0108 | 2.0·10⁻⁷ / 0.0189 |

The largest figure, `log_durations`' 2.9·10⁻⁵, is on a gradient five orders below the others, where the differences' own rounding is that size. The term's gradient reaches `quats` through the normal and the plane distance, `means`, `velocities` and `times` through the plane distance and z (E1), and the four other tensors through the weights.

**L_flat, and its check** (`check_terms.py`, float64 on the CPU). L_flat is the mean over all N of each primitive's smallest scale, PGSR's Eq. 1 read as a mean, as E8 is one; or, weighted as E8 weights σ, the mean over all N of sg[σt,i(t)]·minₖ sₖ,ᵢ. Both forms equal their hand computation on three primitives exactly, their gradient is λ·σt·s_min/N on the smallest axis's `log_scales` alone within 4·10⁻¹⁸, and every other stored tensor gets none.

**L_flat under Adam** (`adam_check.py`, on the CPU). The package's own optimizer (`build_optimizer` with §1.3's rates, default fused Adam, ε = 10⁻¹⁵) was run for 30,000 steps on L_flat alone, scaled as a mean over 3,000,000 rows. Three primitives get no other gradient: one lives the whole sequence, one a tenth of it, one two frames of 120; t is drawn from the 120 frames. Each starts at a scale of 0.02, a log-scale of −3.9.

| λ_flat | Over all N: log of the smallest scale at the end, the three rows / first step 1 lower | Weighted by σt(t): the same |
| --- | --- | --- |
| 100 | −22.0, −22.0, −22.0 / 253 | −22.0, −20.1, −16.5 / 253, 553, 1,223 |
| 1 | −21.6 for all three / 253 | −21.6, −19.6, −16.4 / 253, 553, 1,223 |
| 0.01 | −18.9 / 253 | −18.9, −16.8, −14.6 / 253, 553, 1,224 |
| 10⁻⁴ | −14.9 / 254 | −14.9, −12.9, −11.0 / 254, 555, 1,295 |

- **Where nothing else pulls, the weight does not set the rate.** Adam divides a constant gradient by its own size, so the smallest log-scale falls by about the learning rate, 5·10⁻³, a step, whatever λ_flat is. Over all N, six decades of λ move the first unit of fall by one step (253 to 254); σt-weighted, they move the two-frame row's from 1,223 to 1,295. Every row ends flat, its smallest scale below 10⁻³ of its largest (8.8·10⁻⁴ at most).
- **What λ_flat does set** is the balance against the rendering gradient on the rows that have one, in Adam's second moment. That balance is what the calibration below has to read, from outcomes (the scales' ratios, the scores), not from the term's magnitude.
- **Over all N, every row flattens at every iteration,** alive at the image's time or not. Weighted by σt(t), a row flattens only while it is alive, so the short-lived rows of the near side flatten more slowly, as E8 lowers only the opacities of the rows alive at t.

**Today's primitives are not spheres at the end.** In the off arm's two runs of the centre copy (section 5 here), the ratio of the smallest to the largest scale has a median of 0.075 on the near side, and 57 % of the near side's primitives are below 0.1; the smallest to the middle scale has a median of 0.20 (`ratio_stats.py`; the seeds agree within 0.002). Among the near side's primitives farther than 5 cm from the reference file, the shell, the median ratios are 0.051 and 0.166. So the smallest scale is already distinct for most primitives by iteration 30,000 without any term. When it is distinct by iteration 7,000, the earliest start, is not known: no off-arm run kept a checkpoint there.

**The terms in the wrapper** (`<scratch>/ticket-27/wrap_train.py`, ticket #26's copy with two terms registered; its weight-0 path, the frame-cache record included, unchanged):

- **`flat`**, λ_flat · L_flat, with `--flat-weighting all` or `sigma_t`. It reads the stored tensors only.
- **`nc`**, λ_nc · L_nc through the device's two passes, with `--nc-depth plane` or `expected` and `--nc-from <iteration>`. In the final form the render wrapper computes the term and its whole gradient before the colour pass, with the package's `render` called directly, so that the device's memory is never held beside the colour pass's graph and the wrappers' call counts stay the iterations'; the loss wrapper then adds it with that gradient (below, "The device's cost"). Before `--nc-from` the term reports 0, so that `train_log.jsonl`'s keys stay fixed for the run, and computes nothing. It also logs the share of the image's pixels it read.
- **Settings fixed before the smoke runs:** the composited normal unnormalized (Σwn, 2DGS's form, which weights each primitive by its blending weight); the pixel cut alpha ≥ 0.5 on the pixel and its four neighbours; no image-edge weight (PGSR's reads the training image, a step this arm does not take); the term a mean over the cut's pixels.
- **E10's g.** Neither term enters it (section 5.2): L_flat has no gradient on the 2D means, and L_nc's reaches the device's copies, not the colour pass's 2D means. Growth and relocation read the rendering loss alone, as in the off arm.

**The smoke runs** (300 iterations on `kitchen` at seed 0, `--config` holding `{"optim": {"iterations": 300}}`, as section 5 here ran its own). Each wrapper ran 300 times in every run.

- **At every weight 0** the logged losses equal ticket #26's off-arm smoke run's at iterations 100, 200 and 300 to six figures, one ꟻLIP digit apart at 300 (0.413692 against 0.413693). The rows lie before the first growth event, where section 3 here found two runs of one configuration to agree within rounding. The copy changes nothing at weight 0.
- **`flat` at λ_flat = 100, σt-weighted:** L_flat 9.96·10⁻⁴ at iteration 100 and 9.79·10⁻⁴ at 300, so the term adds about 0.1 to a total of 0.45; L1 at 300 is 0.1148 against 0.1132.
- **`flat` with `nc` at λ_nc = 0.05, plane depth, from iteration 100** (moved early so that the term runs in a smoke run): the device's two passes ran at every iteration from 100 to 300 and the run ended normally. L_nc was 0.68 at iterations 101 to 200 and 0.69 at 201 to 300, read on 96 % and 98 % of the pixels: at iteration 300 the primitives are still near D-25's spheres, so their smallest-scale normals do not agree with the depth map, which is why the term starts late. L1 at 300 is 0.1157. The run took 34 s like the others: at 300 iterations `kitchen` holds 300,000 primitives, so the device's cost shows only on a trained model (below). A first try of this run stopped at iteration 100 on the script's own assertion: it counted the depth channel's clamp at 0 for the rows behind the camera, which the depth cull removes before compositing (E6). The assertion now covers the normal's and the plane distance's codes, the two that can reach a rendered row, and the run was repeated.
- **The same run in the wrapper's final form,** the device's term computed before the colour pass with the geometry taken without `at_time` (above): its logged losses equal the previous run's to five or six figures at iterations 100 to 300 (L_nc 0.679549 against 0.679538 at 200, 0.693289 against 0.693300 at 300), the agreement of two runs of one configuration before the first growth event. The arms would run this form.

**The device's cost on a trained model** (`prof_device.py`, then `prof3.py`): ticket #24's trained checkpoints of 3,000,000 primitives at a training camera's size, synchronized, the median of 15 (10 for `prof3.py`) calls after warm-ups, and the peak allocated memory above the model's own. It is a one-line note taken while another ticket's trainings shared the machine between locked commands, not a benchmark.

- **First,** `prof_device.py` timed the device alone, forward and backward, the geometry through `at_time`: 38.3 ms on `kitchen_s0` at `cam05` (1352 × 1014) and 36.0 ms on `A_h0_s0` at `cam1` (736 × 1280), against 9.5 and 9.2 ms for the colour pass; 38.2 and 35.9 ms on expected depth, which reads the same two passes.
- **Then, as a training step runs it** (`prof2.py`, `prof3.py`): the colour pass with λ_img·L1 and its backward, against the same with the device's term at λ_nc = 0.05, in two orders. In the first, the device runs after the colour pass, whose graph is still alive, as `total_loss` sees it. In the second, the wrapper's, the device's term and its whole gradient are computed first and its graphs freed, then the colour pass runs and one backward takes both (`device_term_grads` in `surface_device.py`; the finite-difference figures above are unchanged by it, to every digit shown).

| Model, camera | Colour pass and backward: time / peak | Device after the colour pass: time / peak | Device first (the wrapper's order): time / peak |
| --- | --- | --- | --- |
| `A_h0_s0`, `cam1` | 10.2 ms / 900 MiB | 46.9 ms / 1,847 MiB | 48.1 ms / 1,566 MiB |
| `kitchen_s0`, `cam05` | 10.3 ms / 896 MiB | 48.4 ms / 1,850 MiB | 49.5 ms / 1,579 MiB |
| `studio_s0`, `0003`, 1920 × 1080 | 15.3 ms / 925 MiB | 63.4 ms / 1,960 MiB | 64.4 ms / 1,612 MiB |

- **The device adds about 38 ms an iteration on private scene A and `kitchen`, and 49 ms on `studio`:** about 1.5 times a baseline iteration on private scene A (26 ms, section 5 here) and 1.1 times one on `kitchen` (34 ms). Section 2.4's figure for one pass was 14.8 ms on `kitchen`; two passes, the geometry and the term's own work come to about 38 ms.
- **Its memory.** After the colour pass it adds about 0.95 GiB to the step's peak; computed first, about 0.65 GiB (666 to 687 MiB), the device's own peak, its two copies of the model and their gradients. The baseline's and the off arm's runs on private scene A peaked at 23,391 to 23,985 MiB of the card's 24,564 MiB in `nvidia-smi`, with the frame cache on the GPU, and `studio`'s at 23,868 MiB with the cache memory-mapped already (sections 3 and 5 here). The margins, 0.57 and 0.68 GiB, are about the device's 0.65 GiB. The two counts are not of the same kind (`nvidia-smi` includes the allocator's reserve), so only a run can tell whether arm 2 fits there; question 5 below.

**The two terms on trained models** (`prof2.py`, at one training image each: `A_h0_s0` at `cam1`, `kitchen_s0` at `cam05`, `studio_s0` at `0003`, the middle frame of each camera; ticket #24's baseline checkpoints, 3,000,000 primitives, trained without either term). The training image is read by the script; λ_img·L1, E7's L1 part, stands for the rendering loss, since the comparison needs only an order of magnitude.

| | Private scene A | `kitchen` | `studio` |
| --- | --- | --- | --- |
| The spatial scale S (D-27) | 0.416 | 1.597 | 4.663 |
| L_nc's pixels: share of the image | 1.00 | 1.00 | 1.00 |
| The plane depth's denominator N·K⁻¹p̃/α: median; 0.1 % and 99 % quantiles | −0.67; −1.21, −0.28 | −0.59; −0.95, −0.24 | −0.61; −1.03, −0.34 |
| Pixels with the denominator above −0.2 / with a plane depth ≤ 0 | 0.16 % / 0 | 0.34 % / 0 | 0.02 % / 0 |
| Plane depth against expected depth, relative: median; 99 % quantile | 0.014; 0.17 | 0.043; 0.12 | 0.014; 0.060 |
| L_nc on the plane depth / on expected depth | 0.796 / 0.798 | 0.642 / 0.593 | 0.785 / 0.708 |
| Gradient norm on `means`: L_nc (plane) / λ_img·L1 | 0.337 / 0.0149 | 0.363 / 0.0157 | 0.226 / 0.0114 |
| Gradient norm on `quats`: L_nc (plane) / λ_img·L1 | 0.0313 / 0.0019 | 0.168 / 0.0030 | 0.0396 / 0.0010 |
| L_flat against λ_img·L1 on the smallest log-scale, visible rows: median ratio at λ_flat = 1 / 100 / 10⁴ | 3.8 / 381 / 38,111 | 1.4 / 142 / 14,235 | 5.8 / 577 / 57,696 |
| The share of those rows where L_flat's gradient is the larger, at λ_flat = 1 / 100 | 63 % / 98 % | 55 % / 99 % | 71 % / 99.5 % |

- **The plane depth is well conditioned on trained models.** No pixel has a denominator near 0 or a plane depth at or below 0, and fewer than 0.4 % lie above −0.2, so the term's mask needs no condition on it. Plane and expected depth differ by 1 % to 4 % at the median.
- **L_nc is large on a model trained without it,** 0.59 to 0.80 on either depth: the smallest-scale directions of today's trained primitives do not follow the depth map. Its unweighted gradient is about 20 times λ_img·L1's on `means` and 17 to 55 times on `quats`, so at 2DGS's λ_nc = 0.05 it is about as strong as E7's L1 part on positions and up to three times on orientations. All gradients were finite.
- **L_flat outweighs the rendering gradient from λ_flat ≈ 1.** At λ_flat = 1 its gradient on the smallest log-scale already exceeds λ_img·L1's on more than half the visible rows; at PGSR's 100, on 98 % to 99.5 % of them, and at 10⁴ on all. E7's SSIM and ꟻLIP parts add to the rendering side, so the balance point against the whole loss lies somewhat higher, but within a decade or two of 1. Stage 1's grid below is set around it.
- **The spatial scale differs elevenfold** between the scenes, and L_flat is in world units: one λ_flat pulls an absolute scale. The balance measured above differs by a factor of four between them, not eleven, so the scale is not the whole story; whether a revision words L_flat in world units or as s_min/S is question 6 below.

**The calibration, proposed** (the PRD's owner item 3; section 5.5: "a short calibration on one scene before the arms"). It has not run.

- **The scene and the runs.** The centre copy of private scene A, seed 0, from `ref_h0_s4`, through the wrapper, on the default 30,000-iteration schedule, stopped by SIGTERM once the log passes iteration 15,000, as section 3 here stopped its short repeats (the checkpoint is saved at the stop). `--checkpoint-every 7000` keeps checkpoints at 7,000 and 14,000 beside it. A stop at 15,000 is the end of growth (D-18), so flattening is read after every clone, split and relocation of the run; a run with `iterations` set to 15,000 would change every schedule and is not used.
- **Stage 1, L_flat.** Six runs: λ_flat = 0.1, 1, 10 and 100 σt-weighted, around the balance point the trained models show (above), up to PGSR's 100; 100 over all N; and one at every weight 0 to the same stop, the comparison for this stage.
- **Stage 2, L_nc,** with stage 1's λ_flat and form, plane depth, from stage 1's start: λ_nc = 0.005, 0.05 and 0.5, around 2DGS's 0.05 (rows 872, 873), and one run of 0.05 on expected depth, the evidence for which depth L_nc reads. If the start is 15,000, these four runs stop at 23,000 instead, so that the term acts for 8,000 iterations in both cases, and one more run, stage 1's chosen λ_flat alone to 23,000, is their comparison.
- **What is read,** at 7,000 and at the stop:
  - the ratio of the smallest to the middle and to the largest scale, near side and its far-5 cm part (`ratio_stats.py`);
  - the means of L1, 1 − SSIM and ꟻLIP over the last 1,000 iterations, from `train_log.jsonl`;
  - the crop PSNR and the sharpness ratio on the test view (`score_a.py`), at the stop;
  - for stage 2, L_nc's logged value over the run and the share of pixels it reads;
  - the named measure and the tail at the stop, reported but not used to choose: choosing a weight by the measure the arm is then judged on would decide the test in its calibration.
- **The rules, fixed before any calibration run:**
  - **The L1 guard's width.** Over eight centre-copy runs of near-identical configurations already made (three plain, the off arm's two, `align_abs`'s two and wrapper v1's), the mean L1 over iterations 14,001 to 15,000 has a standard deviation of 0.40 % of its value and a range of 1.08 %; over 22,001 to 23,000, 0.47 % and 1.27 %. A single run against another then differs by about 0.6 % for nothing, so the guard is set at 1.5 %, about 2.5 such differences.
  - **λ_flat** is the smallest of the four σt-weighted weights whose near-side median ratio of the smallest to the middle scale at the stop is at most half the weight-0 run's, while its last-1,000 L1 mean lies within 1.5 % of the weight-0 run's and its crop PSNR within 0.3 dB (section 4.4's harm guard). If none meets it, PGSR's 100 is used and the calibration says so.
  - **The form decides nothing here:** the arms run σt-weighted unless the owner answers otherwise (open choice 1 below), and the all-N run at 100 is evidence beside the σt-weighted one at 100, read by the same three figures.
  - **The start** is 7,000 if the chosen run's near-side median ratio of the smallest to the middle scale is at most 0.1 at iteration 7,000, the level at which the smallest axis is a tenth of the next and its direction defined; otherwise 15,000, RaDe-GS's.
  - **λ_nc** is the largest of the three whose last-1,000 L1 mean lies within 1.5 % of stage 1's chosen run at the same stop and whose crop PSNR lies within 0.3 dB of it: the strongest term that does no harm by these guards. If none does, the calibration stops and reports before the arms.
- **What it costs, estimated** from the off arm's times (13 min 2 s to 13 min 29 s for 30,000 iterations on the centre copy, fewer primitives before the cap) and the device's timing above:

  | Part | Estimate |
  | --- | --- |
  | Stage 1: 6 runs of 15,000 iterations | about 42 min |
  | Stage 2: 4 runs of 15,000 (or 23,000) iterations, 8,000 of them with the device | about 45 min (4 runs of about 11 min: 6 without the device and 8,000 iterations at 38 ms); about 70 min if the start is 15,000, the runs stop at 23,000 and the comparison run is added |
  | Ratios (CPU), the 142 renders and scores of 10 to 11 checkpoints | about 10 min |
  | In all | about 1.6 GPU hours (about 2.0 if the start is 15,000) |

**The calibration, run** on 2026-11-18, at the owner's answers above, after the other ticket's queue had ended, with this branch's code, the same versions and the RTX 4090 alone under the lock, with 14 CPU threads. Every run is the centre copy at seed 0 from `ref_h0_s4`, through the wrapper copy, stopped at exactly its iteration by the wrapper's `--stop-at`, which sends the process SIGTERM after that iteration's loss (`ftgs train` then saves its checkpoint and exits 143); `--checkpoint-every 7000 --keep-checkpoints 3` kept the checkpoints at 7,000 and 14,000. Each wrapper ran once per iteration in every run. D-51's `auto` put the frame cache on the GPU in all of them.

**Stage 1, L_flat, at iteration 15,000.** The ratios are medians of the smallest scale over the middle one (min/mid) or over the largest (min/max); the near side's far-5 cm part is the shell. L1 is the mean over iterations 14,001 to 15,000; the scores are the test view's crop at 15,000. The named measure and the tail are reported and were not used.

| Run | Near side min/mid at 7,000 / 15,000 | Near side min/max at 15,000 | Whole model / shell min/mid at 15,000 | L1, against the weight-0 run | Crop PSNR | Sharpness ratio (late) | Named measure / tail at 15,000 | Under two frames, near side |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `C1_off`, every weight 0 | 0.457 / 0.323 | 0.156 | 0.338 / 0.225 | 0.014533 | 24.23 | 0.656 (0.669) | 0.159 / 0.068 | 0.577 |
| `C1_s0.1`, λ_flat = 0.1 | 0.326 / 0.196 | 0.098 | 0.139 / 0.000 | +0.11 % | 24.29 | 0.652 (0.670) | 0.237 / 0.103 | 0.562 |
| `C1_s1`, λ_flat = 1 | 0.188 / 0.061 | 0.032 | 0.018 / 0.000 | +1.35 % | 24.25 | 0.655 (0.669) | 0.229 / 0.077 | 0.564 |
| `C1_s10`, λ_flat = 10 | 0.002 / 0.000 | 0.000 | 0.000 / 0.000 | +0.13 % | 24.24 | 0.660 (0.677) | 0.215 / 0.086 | 0.552 |
| `C1_s100`, λ_flat = 100 | 0.000 / 0.000 | 0.000 | 0.000 / 0.000 | +0.48 % | 24.26 | 0.666 (0.686) | 0.207 / 0.087 | 0.564 |
| `C1_all100`, λ_flat = 100 over all N | 0.000 / 0.000 | 0.000 | 0.000 / 0.000 | −0.49 % | 24.32 | 0.665 (0.683) | 0.230 / 0.104 | 0.565 |

- **The rule picks λ_flat = 1.** λ_flat = 0.1 lowers the near side's min/mid ratio to 0.196, short of half the weight-0 run's 0.323 (0.161); λ_flat = 1 lowers it to 0.061, with L1 1.35 % high, inside the 1.5 % guard, and the crop PSNR 0.02 dB higher. It is the smallest weight that passes.
- **The rule puts the start at 15,000.** At iteration 7,000 the chosen run's near-side min/mid ratio is 0.188, above the 0.1 that defines a settled smallest axis; it would have been met from λ_flat = 10 on.
- **From λ_flat = 10 the flattening is total.** The median smallest scale is below 10⁻³ of the middle one everywhere, at 7,000 already; no guard is crossed by it. The shell's primitives are flattened to the same degree at every weight from 0.1 on. These are the rows with little rendering gradient, as the Adam check predicted.
- **All N against σt-weighted at 100:** the same flattening; L1 0.49 % lower and the crop PSNR 0.09 dB higher than the weight-0 run, against +0.48 % and 0.03 dB σt-weighted; the named measure and the tail at 15,000 higher (0.230, 0.104 against 0.207, 0.087). Neither form is harmed by the guards. The arms run σt-weighted, as the owner answered.
- **At 15,000 every flattening run has a higher named measure than the weight-0 run,** 0.207 to 0.237 against 0.159, one seed each. It was not used to choose, and the arms will read it at 30,000 against the off arm's two seeds.

**Stage 2, L_nc, at iteration 23,000.** Since the start is 15,000, the four runs and the comparison run, λ_flat = 1 alone (`C2_flat_only`), went to 23,000, so that L_nc acted for 8,000 iterations. L1 is the mean over 22,001 to 23,000; L_nc is its logged mean over the same iterations, against 0.70 over iterations 15,001 to 15,100 in each run.

| Run | L1, against `C2_flat_only` | L_nc at the end | Near side min/mid at 23,000 | Crop PSNR | Sharpness ratio (late) | Named measure / tail at 23,000 | Under two frames, near side |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `C2_flat_only`, λ_flat = 1 | 0.012007 | – | 0.015 | 24.25 | 0.705 (0.722) | 0.144 / 0.054 | 0.583 |
| `C2_nc0.005`, plane depth | −0.23 % | 0.590 | 0.015 | 24.29 | 0.697 (0.716) | 0.153 / 0.047 | 0.579 |
| `C2_nc0.05`, plane depth | **+9.75 %** | 0.210 | 0.012 | 24.31 | 0.674 (0.686) | 0.153 / 0.062 | 0.580 |
| `C2_nc0.5`, plane depth | **+50.3 %** | 0.039 | 0.011 | 24.03 | 0.594 (0.595) | 0.125 / 0.036 | 0.610 |
| `C2_exp0.05`, expected depth | +10.2 % | 0.226 | 0.016 | 24.30 | 0.668 (0.684) | 0.128 / 0.046 | 0.601 |

- **The rule picks λ_nc = 0.005.** It is the only weight within the 1.5 % L1 guard; 0.05, 2DGS's weight, raises L1 by 9.75 % and 0.5 by 50 %. The crop PSNR does not fall beyond 0.3 dB at 0.005 or 0.05 (+0.04, +0.06 dB) and falls by 0.22 dB at 0.5. The rule was met, so the work went on to the arms without a stop, as the owner answered.
- **At the chosen weight the term barely acts.** L_nc falls from 0.70 to 0.59 in 8,000 iterations at 0.005, against 0.21 at 0.05 and 0.04 at 0.5. Arm 2 is therefore a test of a weak consistency term; what the guard rules out is the strength at which the term reshapes the model.
- **Where it acts, the renders turn soft.** The sharpness ratio falls with λ_nc: 0.705 without the term, 0.697 at 0.005, 0.674 at 0.05, 0.594 at 0.5, with L1 rising as the term fits the depth map's normals instead of the images. That is H2's first outcome, seen here on one seed of one copy.
- **The two depths at 0.05** behave alike in L1 (+9.75 % and +10.2 %), L_nc (0.210 and 0.226) and sharpness (0.674 and 0.668); the named measure at 23,000 is lower on expected depth (0.128 against 0.153), one seed each. The plane distance stays the arm's, as the owner answered.
- **Memory.** Every stage-2 run fit beside the frame cache on the GPU at the cap, peaking at 23,662 to 23,978 MiB in `nvidia-smi`, as the weight-0 run's 23,832. So arm 2 on private scene A keeps `auto`.
- **Times:** 6 min for 15,000 iterations without the device; 10 min for 23,000; 14.5 to 14.8 min for 23,000 with the device from 15,000. Stage 1 and 2 took about 1.7 GPU hours with their scores and ratios, within the 2.0 estimated for the late start.

**The arms' settings, from the calibration:** λ_flat = 1, σt-weighted, in world units, from iteration 1; λ_nc = 0.005 on the plane distance from iteration 15,000 (arm 2 only).

**The arms, as planned for the owner's go.**

- **Arm 1, `flat`:** the default training configuration, 30,000 iterations, through the wrapper copy with `--term flat=<λ_flat>` and the calibration's form.
- **Arm 2, `flat_nc`:** the same with `--term nc=<λ_nc> --nc-depth plane --nc-from <start>`.
- **The off arm** is section 5 here's, reused; the plain baseline beside it.
- **The runs, eight per arm,** mirrored on section 5 here's: the centre and left copies at seeds 0 and 1 from `ref_h0_s4` and `ref_h1_s4`; `kitchen` at seeds 0 to 2 and `studio` at seed 0 from M17's init files.
- **The measures** are section 5 here's, by the same copied scripts: the named measure by the rule against the off arm, with the tail by its margin; the under-two-frames share; the sharpness ratio, the tiles' coverage and the sheets, relabelled blind; the test-view scores and Q-233's kept-pixel PSNR on the centre copy; on openftgs-synth, `scripts/gt_depth.py`'s shares behind and in front of the surface, where a surface term is expected to lower the share behind by more than 0.005, SSIM₁ against the 0.002 guard, and `scripts/reference_flaws.py`'s kept-pixel PSNR. Beside them, for row A1, the distribution of the ratios of the smallest scale to the middle and the largest at the end of every run, by `ratio_stats.py`, whole model, near side and its far-5 cm part.
- **GPU hours, estimated** from the off arm's times and the device's timing:

  | Part | Estimate |
  | --- | --- |
  | Arm 1: 4 runs on private scene A, 3 on `kitchen`, 1 on `studio`; L_flat adds no render | 53 + 51 + 23 min, about 2.1 hours |
  | Arm 2: the same with the device from iteration 7,000 | 4 × (13 + 15) + 3 × (17 + 15) + (23 + 19) min, about 4.2 hours with the device from 7,000 (38 to 39 ms an iteration over 23,000 iterations, 49 ms on `studio`); about 3.5 hours from 15,000 |
  | Renders, scores, `gt_depth.py`, `reference_flaws.py` and the ratios, 16 runs | about 0.6 hours |
  | In all, the arms | about 6.9 GPU hours (6.2 with the start at 15,000) |

  That is beyond section 5.5's 6 hours by the device's cost, which section 5.2 says a prototype pays and an adopted form, channels of the colour pass, would not (about 3.2 ms for depth and a normal, section 2.4). With the calibration, about 8.5 hours in all.

**The open choices the ticket lists, with this step's recommendations.**

1. **L_flat over all N, or weighted by σt(t) as E8 is.** Recommended: **σt-weighted.** Under the default Adam the weight does not control how fast a row with no rendering gradient flattens (above): over all N every primitive flattens at about the learning rate at every iteration, whether or not it is alive at the image's time, and the weight then matters only where the rendering gradient competes. Weighted by σt(t), a primitive is flattened while it is alive, the time-local form E8 already has, and a revision would word it as E8 is worded. The calibration's run over all N at λ_flat = 100 is the evidence beside it; if it lowers the ratios as much with no harm, the choice is open again for the revision.
2. **Which depth L_nc reads.** Recommended: **the plane distance**, row A5. On a slanted flat primitive it returns the plane's depth on both backends, and with it the consistency term is 0 for a lone primitive, so it acts only where neighbouring primitives' planes disagree. Expected depth is the centre's depth across the footprint, so the term on it penalizes every slanted primitive by the cosine of its slant (0.318 on the 54° plane above), pushing primitives to face the camera. The calibration's one expected-depth run measures what that costs.
3. **An oriented start (row A10), or today's spheres.** Recommended: **no oriented start for these arms.** At the off arm's end the smallest scale is already a fifth of the middle one at the near side's median, with no term, and L_flat sharpens it from the first iteration; the calibration's 7,000-iteration checkpoints show whether it is distinct by the consistency term's start, and the start rule above moves it to 15,000 if not. An oriented start would need normals for the init points, which `roma_init` does not give, a key in `ftgs-init` and a field in `InitPoints` (section 5.5): a step of its own if the arms show that orientations never settle, which the end-of-run ratios and the consistency term's logged value would show.

**The questions for the owner at the first step.**

1. **The calibration** as proposed above: its form, its two stages and ten runs (six, then four, and one more if the start is 15,000), its rules, and about 1.6 GPU hours (about 2.0 if the start is 15,000). Options: (a) as proposed, recommended; (b) stage 1 alone, with λ_nc set to 2DGS's 0.05 and the start to 7,000 without a stage 2; (c) no calibration, PGSR's 100 and 2DGS's 0.05 from 7,000.
2. **The arms' hours:** about 6.9 GPU hours (6.2 with the start at 15,000) for both arms on both sets, against section 5.5's 6 hours, the calibration apart.
3. **The open choices:** the three recommendations above, σt-weighted, the plane distance, and no oriented start, or others.
4. **Where L_nc's pixels come from:** fixed above before any training (alpha ≥ 0.5 on the pixel and its four neighbours, no edge weight, the unnormalized composited normal). PGSR's image-edge weight is the one setting a paper gives that this arm leaves out; adding it would read the training image in a geometric term.
5. **Memory for arm 2 on private scene A and `studio`** (above). Options: (a) keep the frame cache's `auto` placement, as the off arm had it, and let the calibration's stage 2, which runs the device on the centre copy at the cap, show whether it fits there; if a run fails for memory, stop and report; (b) set `perf.frame_cache` to `"mmap"` for arm 2 on private scene A, which frees about 2.5 GiB but differs from the off arm in the cache's placement, a change section 3 here's diagnostic (iii) found indistinguishable from noise on `kitchen` and did not test on private scene A; `studio`'s cache is memory-mapped already, so (b) does nothing there; (c) for `studio`, and for private scene A if (b) is not wanted, PyTorch's allocator setting `PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True`, which changes how memory is reserved and no value computed. Recommended: (a), with (b) for private scene A and (c) for `studio` as fallbacks the owner clears beforehand, so that a failure does not wait for a second go.
6. **L_flat's unit.** In world units, as PGSR writes it, one λ_flat pulls harder on a scene of larger scale; as s_min/S, D-27's normalization of the positions' learning rate, it pulls alike in each. The arms run on scales of 0.42 (private scene A), 1.6 (`kitchen`) and 4.7 (`studio`), and the calibration is on private scene A alone. Options: (a) world units, recommended: it is the paper's form, and the balance against the rendering gradient measured above differs by a factor of four between the scenes, not eleven, so dividing by S would not equalize it either; (b) s_min/S, with λ_flat calibrated in that unit. Either is a wording a revision would need.

**The first step's commands.** Each ran with `OMP_NUM_THREADS=4 MKL_NUM_THREADS=4 OPENBLAS_NUM_THREADS=4`, `PYTHONPATH=<worktree>/src:<worktree>/harness/src:<worktree>/tests:<scratch>/ticket-27` and the shared `.venv`'s Python, the CPU ones with `CUDA_VISIBLE_DEVICES=` empty and the GPU ones under the shared lock. `$T` is `<scratch>/ticket-27`, `$O` is `<surface-runs>/ticket-27`, `$B` is `<surface-runs>/ticket-24/runs`, `$X` is `<private-scene-A-exp>`, `$K` is `<m15-runs>/kitchen`, `$M` is `<m17-runs>`. The scripts are not committed.

| Command | Device | Time | Note |
| --- | --- | --- | --- |
| `curl -sSL -o $T/dl/<paper>/page.html https://arxiv.org/html/<id>`, for 2406.06521, 2403.17888, 2404.10772 and 2406.01467, then a text extraction | CPU | seconds | The four papers re-read (rows 871 to 874) |
| `python $T/plane_check.py fd $O/fd.json` | CPU | 1 s | The term's two-part gradient against finite differences, float64; run twice, the first with the code's margin at 1, which failed (above) |
| `python $T/plane_check.py values $O/values_cpu.json cpu` | CPU | seconds | The flat, slanted primitive in float64 alone; run after a first GPU run that stopped on a device mismatch in the script's own check |
| `python $T/plane_check.py values $O/values.json` | GPU and CPU | seconds, after the lock | The flat, slanted primitive on both backends in float32, and in float64 |
| `python $T/check_terms.py $O/check_terms.json` | CPU | seconds | L_flat's value and gradient, and the wrapper's weight-0 path |
| `python $T/adam_check.py $O/adam_check.json` | CPU, 2 threads | 33 s | L_flat alone under the package's optimizer, 30,000 steps, eight settings |
| `python $T/ratio_stats.py $X/scene_h0/scene.json <surface-runs>/ticket-24/init/ref_h0_s4.safetensors $O/ratios_off.json OFF_A_h0_s0=$B/A_h0_s0_v2/checkpoint OFF_A_h0_s1=<surface-runs>/ticket-26/runs/OFF_A_h0_s1/checkpoint` | CPU | 11 s | The off arm's scale ratios on the centre copy |
| `bash $T/run_train.sh smoke_kitchen_<off, flat or nc> $K/scene.json $M/kitchen/init.safetensors 0 <terms> -- --config $O/smoke/cfg300.json`, i.e. `python $T/wrap_train.py <terms> --stats … -- $K/scene.json --init $M/kitchen/init.safetensors --out $O/runs/smoke_kitchen_<…> --seed 0 --config …`, with `<terms>` `--term flat=0 --term nc=0`; `--term flat=100 --flat-weighting sigma_t`; and that with `--term nc=0.05 --nc-from 100 --nc-depth plane` | GPU | 33 to 35 s each | The smoke runs |
| `python $T/prof_device.py <checkpoint> <scene.json> <camera> $O/prof/<set>.json`, for `$B/kitchen_s0` at `cam05` and `$B/A_h0_s0` at `cam1` | GPU | about 1 min each, most of it loading the checkpoint | The device's timing alone |
| `python $T/eq_geometry.py`; `python $T/plane_check.py fd $O/fd_lean.json` and `… $O/fd_precomputed.json precomputed` | CPU | seconds | The geometry without `at_time` against the first form, and the finite differences again with it and with the gradient computed first |
| `python $T/prof2.py $O/prof/prof2.json <label>=<checkpoint>=<scene.json>=<camera> …`, for `$B/A_h0_s0` (`$X/scene_h0/scene.json`, `cam1`), `$B/kitchen_s0` (`$K/scene.json`, `cam05`) and `$B/studio_s0` (`$M/studio/scene/scene.json`, `0003`) | GPU | about 2 min | The two terms on trained models and the step in the first order; run twice, the first stopping on the script's reuse of a freed graph |
| `python $T/prof3.py $O/prof/prof3.json <the same>` | GPU | about 2 min | The step in the wrapper's order |
| `bash $T/run_train.sh smoke_kitchen_nc2 … <the third smoke run's terms> -- --config $O/smoke/cfg300.json` | GPU | 34 s | The smoke run in the wrapper's final form |
| A short inline read of `train_log.jsonl` of `$B/A_h0_s0`, `A_h0_s1`, `A_h0_s0_rep`, `A_h0_s0_v2`, `A_h0_s0_wrap` and `<surface-runs>/ticket-26/runs/` `OFF_A_h0_s1`, `ABS_A_h0_s0`, `ABS_A_h0_s1` | CPU | seconds | The L1 guard's noise |

**In short, for the owner.** Neither arm lowers the shell. With λ_flat = 1 from the calibration, flattening alone makes almost every primitive flat, the shell's included (the median smallest scale goes from a fifth of the middle one to a thousandth on the near side, and to 0 within the shell), and leaves the near-side far-5 cm share where the off arm has it on both copies. The renders keep their sharpness and no guard is crossed on openftgs-synth, but the share behind the surface does not fall. Adding the consistency term, at the only weight the calibration's L1 guard allowed (λ_nc = 0.005 from iteration 15,000), raises the named measure on the centre copy by 0.011, by the rule. On `kitchen` it moves the residual from in front of the surface to behind it by 0.006, beyond the 0.005 guard, while the error of expected depth falls. By section 7.2's row this is the drop clause, the shell staying and turning flat: H2's second outcome. At 2DGS's weight of 0.05, where the term reshapes the model, the calibration's renders turned softer instead: H2's first outcome, on one seed. Recommended: drop finalist 3 as tested. Finalist 4's flattening weight, should it run: λ_flat = 1.

**The arms, run** on 2026-11-18, at the owner's answers above, after the calibration, with the same code, versions and hardware: the RTX 4090 alone under the lock, 14 CPU threads, one training at a time. Every run wrote only under `<surface-runs>/ticket-27`. Scripts read tensors, JSON, images and mask files, wrote numbers and files, and no image, render or sheet of private scene A was looked at (row 876).

- **The runs:** 16, eight per arm, all 30,000 iterations of the default training configuration through the wrapper copy: `FLAT_…` with `--term flat=1 --flat-weighting sigma_t`, and `FLATNC_…` with that and `--term nc=0.005 --nc-depth plane --nc-from 15000`. Private scene A's two copies at seeds 0 and 1 from the kept stride-4 files; `kitchen` at seeds 0 to 2 and `studio` at seed 0 from M17's init files. Every wrapper ran 30,000 times in every run.
- **Memory.** No run needed a fallback: arm 2 fit beside the frame cache on the GPU on private scene A (peaks of 23,575 to 23,924 MiB in `nvidia-smi`) and with the cache memory-mapped on `studio` (23,775 MiB). D-51's `auto` placed the cache as in the off arm: on the GPU on private scene A, memory-mapped on openftgs-synth.
- **Times.** Arm 1: 13 min 14 s to 13 min 18 s on private scene A, 16 min 51 s to 17 min 14 s on `kitchen`, 23 min 29 s on `studio`, the off arm's. Arm 2: 22 min 33 s to 23 min 7 s, 26 min 16 s to 26 min 17 s and 35 min 11 s, the device from 15,000 as estimated.
- **GPU hours.** The arms with their post-processing took about 5.9 hours, the calibration about 1.9 and the first step about 0.3: about 8.1 hours in all, inside the 8.5 cleared.

**The named measure,** the near-side far-5 cm share against the kept files, with the tail beyond 25 cm (each cell 5 cm / tail), and the other shell measures. The off arm and plain baseline are section 5 here's.

| Run | Near side: far-5 cm / beyond 25 cm | Near side: far-10 cm | Under two frames: near / whole | Whole model: far-5 cm | Far side: far-5 cm | Cap reached at |
| --- | --- | --- | --- | --- | --- | --- |
| `OFF_A_h0_s0` / `s1` | 0.1308 / 0.0461; 0.1380 / 0.0333 | 0.0754; 0.0644 | 0.5857 / 0.454; 0.5836 / 0.451 | 0.218; 0.225 | 0.537; 0.536 | 10,900; 10,800 |
| `FLAT_A_h0_s0` | 0.1243 / 0.0411 | 0.0706 | 0.5827 / 0.450 | 0.210 | 0.511 | 10,900 |
| `FLAT_A_h0_s1` | 0.1463 / 0.0477 | 0.0837 | 0.5815 / 0.448 | 0.226 | 0.516 | 10,600 |
| `FLATNC_A_h0_s0` | 0.1465 / 0.0574 | 0.0831 | 0.5823 / 0.448 | 0.225 | 0.514 | 10,500 |
| `FLATNC_A_h0_s1` | 0.1448 / 0.0328 | 0.0721 | 0.5907 / 0.455 | 0.231 | 0.531 | 10,800 |
| `OFF_A_h1_s0` / `s1` | 0.1411 / 0.0379; 0.1337 / 0.0264 | 0.0662; 0.0561 | 0.5808 / 0.445; 0.5893 / 0.449 | 0.227; 0.225 | 0.536; 0.540 | 10,000; 10,000 |
| `FLAT_A_h1_s0` | 0.1372 / 0.0338 | 0.0650 | 0.5792 / 0.437 | 0.225 | 0.517 | 9,800 |
| `FLAT_A_h1_s1` | 0.1467 / 0.0388 | 0.0670 | 0.5838 / 0.442 | 0.231 | 0.519 | 9,900 |
| `FLATNC_A_h1_s0` | 0.1350 / 0.0326 | 0.0683 | 0.5927 / 0.451 | 0.221 | 0.529 | 9,700 |
| `FLATNC_A_h1_s1` | 0.1338 / 0.0313 | 0.0589 | 0.5847 / 0.444 | 0.223 | 0.526 | 9,800 |

**Judged by section 4.4's rule against the off arm** (both of the arm's seeds beyond both of the off arm's, the means more than 0.01 apart; 0.021 for the tail):

| Copy | Arm 1, `flat` | Arm 2, `flat` with `nc` | Against the plain baseline, for reference |
| --- | --- | --- | --- |
| Centre, named measure | not moved (0.124, 0.146 against 0.131, 0.138; mean 0.001 higher) | **higher by 0.011** (0.147, 0.145 against 0.131, 0.138) | arm 1 not moved; arm 2 not moved (mean 0.005 higher) |
| Centre, tail | not moved | not moved (mean 0.005 higher) | not moved |
| Left, named measure | not moved (0.137, 0.147 against 0.141, 0.134; mean 0.005 higher) | not moved (0.135, 0.134; mean 0.003 lower) | arm 1 lower by 0.016, arm 2 by 0.024: the script's own effect, which the off arm takes out (section 5 here) |
| Left, tail | not moved | not moved | not moved |
| Under two frames, near side (reported) | not moved on either copy | not moved on either copy | not moved |

- **Neither arm lowers the named measure** on either copy. Arm 2 raises it on the centre copy by the rule, the wrong way for H1.
- **The tail does not move** in either arm on either copy.
- **The far side's far-5 cm share** is lower in every arm run, 0.511 to 0.531 against the off arm's 0.536 to 0.540; the rule is not applied to it.

**Row A1: does flattening survive growth and relocation?** The medians of the ratio of the smallest scale to the middle one (min/mid) and to the largest (min/max) at iteration 30,000, by `ratio_stats.py`, with the share of primitives whose min/max is below 0.01:

| Runs | Whole model: min/mid, min/max, share below 0.01 | Near side | Near side's far-5 cm part (the shell) |
| --- | --- | --- | --- |
| Off arm, centre copy, 2 seeds | 0.224, 0.061, 0.21 | 0.198 to 0.200, 0.075, 0.17 | 0.166, 0.051, 0.24 to 0.25 |
| Off arm, left copy, 2 seeds | 0.217 to 0.221, 0.058 to 0.059, 0.21 to 0.22 | 0.188 to 0.190, 0.071, 0.18 | 0.150 to 0.172, 0.053 to 0.058, 0.22 to 0.25 |
| Plain baseline, centre copy, 2 seeds | 0.226, 0.062, 0.21 | 0.202, 0.076, 0.17 | 0.164 to 0.166, 0.055 to 0.056, 0.24 to 0.25 |
| Arm 1, private scene A, 4 runs | 0.0001, 2·10⁻⁵ to 5·10⁻⁵, 0.72 to 0.74 | 0.0016 to 0.0026, 0.0007 to 0.0011, 0.64 to 0.66 | 0.0000, 0.0000, 0.87 to 0.88 |
| Arm 2, private scene A, 4 runs | 0.0001 to 0.0002, 3·10⁻⁵ to 5·10⁻⁵, 0.73 to 0.74 | 0.0018 to 0.0025, 0.0008 to 0.0011, 0.65 to 0.66 | 0.0000, 0.0000, 0.86 to 0.88 |
| Off arm / arm 1 / arm 2, `kitchen`, 3 seeds each | 0.561, 0.168, 0.06 / 0.090, 0.022, 0.41 / 0.085 to 0.087, 0.020, 0.41 to 0.42 | – | – |
| Off arm / arm 1 / arm 2, `studio` | 0.330, 0.082, 0.14 / 0.0000, 0.0000, 0.86 / 0.0000, 0.0000, 0.86 | – | – |

- **Flattening survives growth and relocation.** At the end of every arm run the near side's median smallest scale is about 1/500 of its middle one, against a fifth in the off arm, and the shell's is 0 to the printed digit; E11's and E17's copies keep it, as row A1 read them.
- **The shell flattens most.** Its primitives get the least rendering gradient, so under Adam L_flat drives them at the learning rate's pace (the Adam check above); the share of the shell below 0.01 is 0.86 to 0.88, against 0.64 to 0.66 on the near side.
- **`kitchen` flattens far less** (min/mid 0.09) than private scene A and `studio` (about 0), at the same λ_flat in world units. Its rendering gradients balance the term at a larger ratio, as the trained models showed (above).
- **The smallest-scale direction did settle without an oriented start:** in the calibration by iteration 15,000 at λ_flat = 1 and by 7,000 at 10, and in every arm run. Row A10's remedy is not needed for the term to act. It is still not shown that the settled direction is the surface's.
- **The consistency term moved little at λ_nc = 0.005:** L_nc ended at 0.51 to 0.57 on private scene A, 0.24 to 0.25 on `kitchen` and 0.35 on `studio`, from about 0.7 at its start.

**The test-view scores, the sharpness ratio and the tiles' coverage,** as section 3 here takes them. Kept-pixel PSNR is Q-233's, on the centre copy only. Every rendered value is finite.

| Run | Crop PSNR | Crop SSIM₁ | Sharpness ratio | Late | Training cameras: PSNR / ratio | E16 PSNR / SSIM₁ / ꟻLIP | Kept-pixel PSNR | Coverage: between cameras to ±45°; ±60°, ±90° |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `FLAT_A_h0_s0` | 24.37 | 0.8152 | 0.7232 | 0.7449 | 32.86 / 0.7764 | 24.83 / 0.8189 / 0.1535 | 25.57 | 0.81 to 1.00; 0.49 to 0.90 |
| `FLAT_A_h0_s1` | 24.41 | 0.8163 | 0.7278 | 0.7519 | 32.76 / 0.7704 | 24.96 / 0.8196 / 0.1533 | 25.70 | 0.82 to 1.00; 0.50 to 0.88 |
| `FLATNC_A_h0_s0` | 24.35 | 0.8153 | 0.7252 | 0.7419 | 32.76 / 0.7713 | 24.84 / 0.8183 / 0.1521 | 25.57 | 0.89 to 1.00; 0.45 to 0.93 |
| `FLATNC_A_h0_s1` | 24.36 | 0.8164 | 0.7236 | 0.7445 | 32.71 / 0.7685 | 24.73 / 0.8203 / 0.1533 | 25.47 | 0.81 to 1.00; 0.44 to 0.88 |
| `FLAT_A_h1_s0` | 23.86 | 0.7461 | 0.7463 | 0.8011 | 31.50 / 0.7745 | 21.86 / 0.7382 / 0.1957 | – | 0.81 to 1.00; 0.62 to 0.99 |
| `FLAT_A_h1_s1` | 23.85 | 0.7464 | 0.7418 | 0.7912 | 31.49 / 0.7724 | 21.67 / 0.7381 / 0.1959 | – | 0.79 to 1.00; 0.48 to 0.87 |
| `FLATNC_A_h1_s0` | 23.82 | 0.7485 | 0.7317 | 0.7856 | 31.46 / 0.7700 | 21.62 / 0.7395 / 0.1965 | – | 0.80 to 1.00; 0.51 to 0.87 |
| `FLATNC_A_h1_s1` | 24.00 | 0.7494 | 0.7292 | 0.7822 | 31.40 / 0.7679 | 21.79 / 0.7402 / 0.1940 | – | 0.78 to 1.00; 0.48 to 0.89 |

- **No test-view score falls by more than 0.3 dB** against the off arm in either arm on either copy. The largest falls are arm 1's crop PSNR on the left copy (−0.10 dB) and arm 2's E16 PSNR there (−0.12 dB). On the centre copy arm 1 is higher: E16 +0.16 dB and kept-pixel PSNR +0.18 dB, about the off arm's own seed spread (0.14 and 0.17).
- **The sharpness ratio does not fall in arm 1:** −0.003 on the centre copy, within the off arm's spread of 0.0055, and +0.010 on the left copy, both seeds above both of the off arm's.
- **In arm 2 it falls slightly on the left copy:** 0.732 and 0.729 against 0.732 and 0.735, a mean 0.003 lower, about the off arm's spread of 0.003. On the centre copy it is 0.004 lower, within the spread of 0.0055. Section 7.2's last column, a measure that falls while the renders soften, does not arise.
- **The coverage** stays within the off arm's and the baseline's ranges; no tile collapses.

**The sheets, for judging with their labels hidden.** There are four sets, the centre copy at seeds 0 and 1, then the left copy at seeds 0 and 1. Each holds the off arm and both arms in an order from Python's `random.Random(27)`, labelled `set<n>-A` to `-C` only. They are `<surface-runs>/ticket-27/sheets/blind/set<n>-<letter>.jpg`, from section 2.6's fixed request, with the key apart in `KEY.json` beside them. Nobody here has looked at them, and the owner's judgement is open.

**openftgs-synth.** The columns are as in section 3 here. On `kitchen`, at the owner's answer, SSIM₁, PSNR and ꟻLIP are read against the off arm's five seeds; the kept-pixel PSNR and the residual's shares against its seeds 0 to 2; and the plain baseline's five seeds are beside them for SSIM₁.

| Run | E16 PSNR / SSIM₁ / ꟻLIP | Kept-pixel PSNR | Residual, all pixels: in front / behind | Dynamic pixels: in front / behind | Expected depth, median error, all / dynamic | Far share at 0.12·S | Beyond 25 cm | Under two frames | Cap reached at |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `OFF_kitchen_s0` to `s2` | 24.88 to 27.58 / 0.8753 to 0.8772 / 0.0910 to 0.0949 | 29.16 to 30.08 | 0.041 to 0.042 / 0.294 to 0.295 | 0.015 to 0.016 / 0.615 to 0.616 | 206 to 247 / 44 to 73 mm | 0.035 to 0.036 / 0.002 to 0.003 | 0.023 to 0.024 / 0.001 | 0.381 to 0.383 / 0.655 to 0.662 | 2,100 |
| `OFF_kitchen_s3`, `s4` | 28.33 / 0.8779 / 0.0895; 27.69 / 0.8781 / 0.0945 | – | – | – | – | – | – | – | 2,100 |
| `FLAT_kitchen_s0` | 29.10 / 0.8792 / 0.0887 | 31.02 | 0.041 / 0.295 | 0.016 / 0.618 | 192 / 52 mm | 0.038 / 0.003 | 0.024 / 0.001 | 0.384 / 0.659 | 2,000 |
| `FLAT_kitchen_s1` | 27.42 / 0.8785 / 0.0916 | 30.44 | 0.041 / 0.298 | 0.016 / 0.617 | 244 / 107 mm | 0.036 / 0.003 | 0.023 / 0.001 | 0.383 / 0.662 | 2,000 |
| `FLAT_kitchen_s2` | 29.96 / 0.8799 / 0.0861 | 31.42 | 0.041 / 0.294 | 0.016 / 0.619 | 186 / 37 mm | 0.037 / 0.003 | 0.024 / 0.001 | 0.385 / 0.658 | 2,000 |
| `FLATNC_kitchen_s0` | 27.41 / 0.8785 / 0.0897 | 30.83 | 0.035 / 0.299 | 0.014 / 0.627 | 169 / 29 mm | 0.032 / 0.002 | 0.020 / 0.001 | 0.380 / 0.660 | 2,000 |
| `FLATNC_kitchen_s1` | 27.41 / 0.8768 / 0.0923 | 29.81 | 0.036 / 0.299 | 0.016 / 0.622 | 182 / 39 mm | 0.032 / 0.003 | 0.020 / 0.001 | 0.381 / 0.663 | 2,000 |
| `FLATNC_kitchen_s2` | 26.15 / 0.8776 / 0.0911 | 30.52 | 0.036 / 0.301 | 0.014 / 0.627 | 203 / 32 mm | 0.032 / 0.003 | 0.019 / 0.001 | 0.379 / 0.657 | 2,000 |
| `OFF_studio_s0` | 22.83 / 0.7986 / 0.1899; dynamic 25.02 | 23.02 | 0.014 / 0.414 | 0.009 / 0.649 | 172 / 102 mm | 0.044 / 0.001 | 0.071 / 0.001 | 0.384 / 0.707 | 6,900 |
| `FLAT_studio_s0` | 22.63 / 0.7942 / 0.1919; dynamic 25.29 | 22.57 | 0.013 / 0.414 | 0.007 / 0.645 | 140 / 82 mm | 0.040 / 0.001 | 0.068 / 0.001 | 0.389 / 0.716 | 6,600 |
| `FLATNC_studio_s0` | 22.69 / 0.7963 / 0.1886; dynamic 25.36 | 22.68 | 0.011 / 0.436 | 0.007 / 0.648 | 159 / 91 mm | 0.036 / 0.001 | 0.064 / 0.001 | 0.390 / 0.712 | 6,600 |

- **Arm 1 on `kitchen` does no harm by the guards.**
  - SSIM₁ is 0.0024 above the off arm's five seeds, and 0.0003 above the plain baseline's five.
  - The residual's shares are within 0.003 of the off arm's: behind the surface +0.0016 on all pixels and +0.0026 on dynamic pixels.
  - PSNR, kept-pixel PSNR and ꟻLIP are not flagged.
  - **The share behind the surface does not fall,** so the better result section 5.5 expected of a surface term on openftgs-synth does not appear.
- **Arm 2 on `kitchen` moves the residual behind the surface.**
  - Its share behind is 0.0057 above the off arm's on all pixels and 0.0097 on dynamic pixels, beyond the 0.005 by which section 4.4 calls a share worse. Its share in front is 0.0056 lower, beyond the same margin the other way.
  - The median error of expected depth falls: 169 to 203 mm against 206 to 247 mm, and 29 to 39 against 44 to 73 mm on dynamic pixels, ranges that do not overlap, so the criterion applies and reads better.
  - SSIM₁ is 0.0008 above the off arm's five seeds and 0.0012 below the plain baseline's five, within the guard. PSNR, kept-pixel PSNR and ꟻLIP are not flagged against the off arm.
  - So the consistency term moves weight from in front of the surface to behind it and brings expected depth closer: the guard on the share behind counts it as harm, the depth error as a gain.
- **On `studio`, one seed.** The dynamic-region PSNR is 25.29 dB in arm 1 and 25.36 dB in arm 2, against the off arm's 25.02 dB; neither is flagged. Arm 2's share behind the surface is 0.436 against 0.414, 0.022 higher, the direction `kitchen` shows; arm 1's is the off arm's.
- **The far shares and the under-two-frames share** are within 0.004 of the off arm's on `kitchen`; on `studio`'s one seed within 0.009, arm 2's far share at 0.12·S 0.036 against 0.044.

**The reading by section 7.2's row for finalist 3.**

- **Adopt when** "the near-side far-5 cm share moves with the renders sharp, with no harm on openftgs-synth, where a lower share behind the surface is expected". **Not met:** neither arm lowers it, and arm 2 raises it on the centre copy.
- **Drop when** "the shell stays and turns flat, thin and aligned with its own depth, H2's other outcome". **Met, in part for the last word.**
  - **Arm 1** keeps the shell where the off arm has it and makes its primitives flat, thinner than any other population. The row says the first arm predicts nothing about the shell; it was run to see what flattening does to today's spheres, and it shows that L_flat alone shapes the shell and does not move it, as row A1 read.
  - **Arm 2**, at the weight the guard allowed, aligns the primitives with their own depth only a little (L_nc from 0.7 to about 0.5 on private scene A). It keeps the shell, raises it on the centre copy by 0.011, and on `kitchen` moves the residual behind the surface.
  - **At 2DGS's weight, 0.05,** the calibration's centre-copy run aligned them more (L_nc 0.21), raised L1 by 9.75 % and lowered the sharpness ratio from 0.705 to 0.674; at 0.5, to 0.594. That is H2's first outcome, the renders turning soft, on one seed at iteration 23,000; the arms did not run it.
- **What this test does not cover.**
  - λ_nc between 0.005 and 0.05, and any weight above 0.005 at 30,000 iterations on both sets.
  - The consistency term from iteration 7,000 (the start rule moved it to 15,000 at λ_flat = 1).
  - PGSR's image-edge weight and its multi-view terms.
  - λ_flat expressed as s_min/S.
- **Recommended: drop finalist 3 in the form tested,** by the row. Flattening survives growth and does no harm, but does not move the shell. The consistency term either barely acts, at the weight the guard allows, or softens the renders, at the papers' weight. For finalist 4, which the PRD runs on top of finalist 3's flattening, the calibrated λ_flat = 1 is the weight this step found.

**Section 5.5's open choices, as answered** (by the owner, at this step's recommendations, and by the runs):

1. **L_flat over all N or σt-weighted:** σt-weighted, at the owner's answer. The calibration's all-N run at 100 flattened alike and crossed no guard, so the choice stays open for a revision on evidence: neither form lowered the shell.
2. **Which depth L_nc reads:** the plane distance, at the owner's answer. At 0.05 the two depths behaved alike in L1, L_nc and sharpness; the calibration's one expected-depth run had a lower named measure at 23,000 (0.128 against 0.153), one seed each. Not settled by these runs.
3. **An oriented start:** not needed for the smallest axis to settle: today's spheres flatten by iteration 7,000 at λ_flat ≥ 10, and by 15,000 at 1. Whether an oriented start would put the flat primitives on the surface is not tested.

**Against the ticket's acceptance criteria.**

- **The plane-distance check, before the arms:** done (the first step, above).
- **The calibration's proposal, the owner's answer and its result:** recorded above.
- **The two arms on both sets:** 16 runs, against the off arm, the plain baseline beside it.
- **The named measure by section 4.4's rule, with the sharpness ratio and the sheets, against H2's two outcomes:** above; the sheets are keyed and await the owner's judgement.
- **On openftgs-synth, the share behind the surface and the guards:** above. Arm 1 does no harm and does not lower the share; arm 2 raises it by 0.006, beyond the guard.
- **The ratio distribution at the end of each arm:** above, row A1's table.
- **Section 5.5's open choices:** answered or stated as open, above.
- **The reading by section 7.2's row, and the owner's answer:** the reading above; the owner's answer below, not yet given, with the flattening weight for finalist 4 to be recorded there.
- **Sources, commands, files changed, privacy:**
  - sources: rows 871 to 876;
  - commands: below;
  - files changed: only this document and the source log;
  - privacy: nothing names private scene A or shows its images.

**The questions for the owner at step 5.**

1. **Finalist 3:** dropped, as recommended, or kept for a further arm (λ_nc between 0.005 and 0.05, or from 7,000 at λ_flat = 10, about 3.5 to 4 GPU hours each on both sets)?
2. **Step 6, finalist 4:** go or stop. If go, its flattening weight: λ_flat = 1, σt-weighted, as calibrated here (recommended), or another.
3. **The sheets** (`<surface-runs>/ticket-27/sheets/blind`, key apart): the owner's judgement with the labels hidden.

**The calibration's and the arms' commands** (the conventions of the first step's table, with 14 threads, `$I` for `<surface-runs>/ticket-24/init`, `$G` for `$OPENFTGS_DATASETS/synth/ground_truth` and `$S` for `<surface-runs>/ticket-5/side_request.json`):

| Command | Device | Time | Note |
| --- | --- | --- | --- |
| `bash $T/run_train.sh smoke_stop150 $K/scene.json $M/kitchen/init.safetensors 0 --stop-at 150 --term flat=0 -- --config $O/smoke/cfg300.json --checkpoint-every 100 --keep-checkpoints 3` | GPU | 30 s | The exact stop checked: exit 143 at iteration 150, its checkpoint saved; then removed |
| `bash $T/cal_stage1.sh`: `bash $T/run_train.sh C1_<…> $X/scene_h0/scene.json $I/ref_h0_s4.safetensors 0 --stop-at 15000 <terms> -- --checkpoint-every 7000 --keep-checkpoints 3`, six runs | GPU | 6 min each | Stage 1 |
| `bash $T/cal_post.sh <run>`: `ftgs render <checkpoint> --request $X/requests/heldout_novel.json`; `python $T/score_a.py … --heldout cam0 --crop 96,256,640,1280 --stored-masks <surface-runs>/ticket-4/private-scene-A/placeholder_masks.npz`; `python $T/ratio_stats.py` at 7,000, 14,000 and the stop; `python $T/measures_a.py` | GPU, then CPU | about 35 s | The calibration's figures |
| `python $T/cal_decide.py stage1` | CPU | seconds | The stage-1 rule: λ_flat = 1, start 15,000 |
| `bash $T/cal_stage2.sh 1 15000 23000`: `C2_flat_only`, then `C2_nc<0.05, 0.5, 0.005>` and `C2_exp0.05` with `--term nc=<…> --nc-depth <plane or expected> --nc-from 15000 --stop-at 23000` | GPU | 10 min, then 14.5 to 14.8 min each | Stage 2 |
| `python $T/cal_decide.py stage2 C2_flat_only 23000` | CPU | seconds | The stage-2 rule: λ_nc = 0.005 |
| `bash $T/arms.sh 1 0.005 15000 both`: per arm, `bash $T/run_train.sh <run> <scene.json> <init> <seed> <terms>` for the eight runs, each followed by `bash $T/post_a.sh <run> h<c>` or `bash $T/post_synth.sh <run> <scene> <scene.json> <init>` (section 5 here's, repointed, with `ratio_stats.py` added) | GPU, then CPU | 13 to 35 min a run; 1 to 2 min a post | The arms; the script's memory fallbacks were never taken |
| `python $T/ratio_stats.py` on the off arm's and the plain baseline's checkpoints | CPU | seconds each | The off arm's ratios |
| `bash $T/blind.sh` | CPU | seconds | The twelve sheets with their labels hidden, and their key |
| `python $T/tables27.py` | CPU | seconds | The tables of this section, with the five-seed reads on `kitchen` |

The private scene's folders, `<m15-runs>`, `<m17-runs>` and tickets #24's, #26's and #26b's folders were read and never written.

**Sheets on `kitchen`** (added on 2026-11-18 at the owner's request of that day). This step's `kitchen` models, `FLAT_` and `FLATNC_kitchen_s0` to `s2`, have labelled sheets in `<surface-runs>/kitchen-sheets/sheets/`, beside the off arm's and the plain baseline's of section 5 here. They are made from the fixed request of section 4 here's "Sheets on `kitchen`" (SHA-256 `5560b8a317157d825b53561a519f755498f286feb78bd82298f88e0e856fd27c`): `cam00` turned by section 2.6's nine yaws, at frames 33, 114 and 262, with the ground truth's `cam00` image as a last column. The side-by-side sheets `<surface-runs>/kitchen-sheets/compare/finalist3_kitchen_s<0, 1 or 2>.jpg` stack, top to bottom: `FLAT`, `FLATNC`, the off arm, the plain baseline.

| Command | Device | Time | Note |
| --- | --- | --- | --- |
| `bash $T/kitchen_sheets.sh <run-dir> <run>`, per run; then `python $T/compare_sheets.py $O/compare/finalist3_kitchen_s<n>.jpg --title … <sheets>` | GPU (one render under the lock), then CPU | 6.4 to 8.9 s per render, apart from the wait for the lock | As in section 4 here's table, with the same `$T` and `$O` |

**The owner's answer on step 5** (the go for step 6 or a stop), given in session:
- **Finalist 3: dropped as tested** (question 1). The options left untested stay recorded: λ_nc between 0.005 and 0.05, the term from iteration 7,000 at λ_flat = 10, and an oriented start.
- **Step 6: go, with DAViD's optional arm** (question 2). Finalist 4 uses the calibrated flattening weight, λ_flat = 1, σt-weighted. Its first step runs while finalist 2's variants hold the GPU, and its arms after them.
- **The sheets** (question 3) await the owner's judgement with their labels hidden.

**Date:** 2026-11-18.

## 7. Step 6: finalist 4, a normal prior on top of finalist 3's flattening

*Ticket #28's section.* Section 2.7's measurement repeated for normals, with `kitchen` for the floor, then finalist 4's arm, run as the research document's section 5.6 says and judged by section 4.4's criteria.

**The first step** was run on 2026-11-18, at the owner's go of that day (section 6 here, "The owner's answer on step 5"), while finalist 2's variant queue held the GPU, with this branch's code at 96659a0 (`src/`, `scripts/` and `tests/` as `main`'s at 02e3686), Python 3.14.10, torch 2.15.0+cu132, gsplat 1.5.3, NumPy 2.5.4 and Pillow 13.0.0 from the shared `.venv`, MoGe's code at 74fbce0 and the ONNX runtime 1.32.0 from `<scratch>/ticket-28` (below). The hardware was one RTX 4090 (driver 595.99.02), shared under the GPU lock, so every GPU command here waited its turn between the queue's trainings; the CPU work ran on 4 threads. Scripts read the scene manifests, the images, the match grids, the ground truth and checkpoints, and wrote numbers and map files (row 888). Private scene A's images went only to the two estimators, running offline on this machine; no image, map or render was looked at. No arm was trained.

**The owner's answers at step 6's go** (2026-11-18, in session, relayed by the orchestrating session):

- **Finalists 1 to 3 are dropped,** so the arms train on the manifest's cameras, and the prior is added on top of finalist 3's flattening alone, without its consistency term.
- **The flattening weight** is finalist 3's calibrated one, λ_flat = 1, σt-weighted, in world units, from iteration 1 (section 6 here, "The arms' settings").
- **The off arm** is section 5 here's, wrapper v2 at every added weight 0, with the plain baseline beside it. On `kitchen` it is read against the off arm's five seeds for SSIM₁, PSNR and ꟻLIP, and its seeds 0 to 2 for the rest, as section 6 here read it.
- **The named measure** is the near-side far-5 cm share, with the tail beyond 25 cm beside it, read by section 4.4's rule, the tail with the margin 0.021.
- **Two seeds per arm** on each copy of private scene A.
- **DAViD's optional arm is cleared** (2026-11-18): one arm on private scene A with the maps of `normal-model-vitl16_384.onnx`.
- **Labelled `kitchen` sheets** (the owner's request of 2026-11-18): every `kitchen` model gets a labelled sheet from a fixed `kitchen` request, by `<scratch>/kitchen-sheets/kitchen_sheets.sh <run-dir> <label>`, run after each `kitchen` run's post-processing.

**The owner's answers at the first step** (2026-11-18, in session, relayed by the orchestrating session, after the first step's report; questions 1 to 4 below):

- **The start:** both prior terms from iteration 15,000 (question 1).
- **The settings:** as fixed below, C7 summed over the three components on the unnormalized Σwn, and C8 on the plane-distance depth with its weight read from the composited normal, without a gradient (question 2).
- **DAViD's person:** section 2.7's depth rule, from ticket #12's Depth Anything V2 maps (question 3, option (a)).
- **The hours:** about 5.7 GPU hours cleared for both arms, with step 5's memory fallbacks cleared again (`perf.frame_cache` `"mmap"` on private scene A, `PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True` on `studio`); the work stops and asks before going about an hour past the estimate (question 4).

**The weights and the code, checked again at their publishers** (rows 569, 582, 583 and 830 hold for their dates; all fetched without credentials):

| What | Checked | Terms | The file used |
| --- | --- | --- | --- |
| MoGe-2's weights, `Ruicheng/moge-2-vitl-normal` (row 882) | The card's front matter and the Hub's public record | `license: mit`; base model `facebook/dinov2-large`; not gated; commit cb0e8bb, unchanged since rows 583 and 830 | `model.pt` at cb0e8bb: 1,323,815,904 bytes, SHA-256 `280741fd…cc1a01`, equal to the publisher's linked ETag |
| MoGe's code, `microsoft/MoGe` (row 883) | `LICENSE` first, then the README's "License" | MIT, copyright Microsoft Corporation; the bundled DINOv2 code under `moge/model/modules/dinov2` is Apache-2.0 (Meta AI) | The source archive at 74fbce0, used unchanged from the scratch directory |
| DAViD's normal model and runtime, `microsoft/DAViD` (row 884) | The README's "Model License", `LICENSE-MIT.txt`, the normal model's card | Models and runtime code under MIT, copyright Microsoft Corporation; human subjects only | `normal-model-vitl16_384.onnx`: 1,377,959,705 bytes, SHA-256 `759cdfe0…e461d3`, its MD5 the host's Content-MD5 |
| The ONNX runtime and its dependencies (row 885) | The licenses inside the wheels, read before installing | onnxruntime 1.32.0 MIT; protobuf 7.36.2 BSD-3-Clause; flatbuffers 25.12.19 Apache-2.0. The runtime's notices list Eigen (MPL-2.0) compiled into the binary, and Mbed TLS taken under Apache-2.0: a binary used as a program in a scratch environment, its source not read | Installed with `pip install --no-deps --target <scratch>/ticket-28/env-ort`; numpy and packaging from the shared `.venv`, which was not written |

All four pass the bar as section 4.9 found them. The weights live under `<scratch>/ticket-28/weights`, downloaded outside the test suite (Q-7), and each loads without a network call: MoGe's with `torch.load(..., weights_only=True)`, its two imports of `utils3d` and `huggingface_hub` given empty stand-ins, since only the point map's post-processing and downloads use them.

**The estimators as run** (`<scratch>/ticket-28/normal_maps.py`, not committed):

- **MoGe-2:** the model's `forward` at `infer`'s defaults, 3,600 tokens (level 9), under float16 autocast, on RGB in [0, 1]; the normal head's output, which `forward` resizes to the image bilinearly and normalizes; the mask head's sigmoid above 0.5 marks the pixels with a prior. 56 ms per `kitchen` image and 54 ms per image of private scene A, at a peak of 2.3 GiB. Its mask kept every pixel of all 201 images.
- **DAViD:** the runtime's `estimate_normal`, its OpenCV steps done in NumPy and PyTorch: BGR in [0, 1], padded to a square by replicating the border, resized bilinearly to 512, the model, the output resized back bicubically, cropped and normalized. PyTorch's bilinear and bicubic kernels stand in for OpenCV's; the two were not compared. On the CPU, 4 threads: 1.36 s per image. Its maps mean something on the person only, so a training map would mark every other pixel as carrying no prior (below, question 3).
- **The maps' format:** one file per image, `<surface-runs>/ticket-28/<scene>/<estimator>/maps/<camera>/<frame>.npz`, the normal as int8 (127·n) in the estimator's own axes with a packed validity mask. The code costs 0.14° at the median (0.23° at the 90th percentile; measured on `kitchen`'s ground-truth normals). A file is 4.1 MB for `kitchen` and 2.9 MB for private scene A.
- **The images:** on `kitchen`, frames 0, 1, 100, 101, 200 and 201 of all 20 cameras (120 images); on private scene A, frames 0, 1, 20, 40, 60, 80, 81, 100 and 119 of the nine cameras (81 images), by both estimators.

**The measure** (`<scratch>/ticket-28/normals_agree.py`). It is section 2.7's, with what a normal does not need taken out.

- **Pairs and matches:** section 2.7's. On private scene A, D-22's 7 pairs, 14.5° or 15.0° apart. On `kitchen`, the six wide pairs, about 15° apart; D-22's 13 pairs with `cam00`–`cam01`, 2.8° to 6.6° apart; at frames 0, 100 and 200. The matches are the dense matcher's admitted matches from tickets #4's and #12's grid files: both ends inside their images, certainty at least 0.5.
- **The comparison:** at each match, each camera's map is read at its end of the match (bilinear, renormalized), turned into the world by the manifest's rotation (n_w = Rᵀn_c, §1.4's world-to-camera R), and the two are compared by their angle. A normal has no scale, shift or anchor, so nothing is aligned and no depth enters it. The cost is that only matched points are compared: section 2.7's unmatched pixels, where its tail lay, are not reached.
- **Regions:**
  - On `kitchen`, static and dynamic, by the ground truth's masks at the first end. Figures there are on matches whose two ends are away from depth edges by Q-228's rule and that the ground truth calls seen: the first end, lifted by its ground-truth depth, lands within 2 px of the second, with the depth agreeing within 1 %.
  - On private scene A, section 2.5's split by the match's triangulated depth in the first camera: the **person** is rule a's crop at 0.5 to 1.8 units, the **background** 2.2 units or more. The loose rule's placeholder pixels are left out, and person figures are on matches whose second end also lies in the other camera's crop ("both").
- **Against the ground truth,** on `kitchen`: the estimator's camera-space normal against the normal of the ground truth's depth (planar z; the wrapper's `depth_normals`, (P_down − P_up) × (P_right − P_left), facing the camera), at pixels away from depth edges, all 120 images.
- **One camera, two frames:** each camera's normals at frames k and k + 1 at the same pixels, the first frame's match ends (frames 0 and 1, 100 and 101, 200 and 201 on `kitchen`; 0 and 1, 80 and 81 on private scene A). A moving surface moves under a fixed pixel, so on the person and on `kitchen`'s dynamic objects this counts motion as well as the estimator's jitter.

**Checks of the measure,** with the ground truth's own normals as the maps, on `kitchen`'s wide pairs:

- **The conventions hold.** Between cameras 15° apart the static surfaces read 0.002° at the median and 0.007° at the 90th percentile; the dynamic objects 0.12° and 5.0°, the matches' placement on small curved objects.
- **A camera that is slightly off.** Camera B turned about its own centre by 2° about its y axis reads 1.93° at the median (1.97° at the 90th percentile), and by 0.3° about its x axis 0.299°, as a rotation of that size should: a normal turns by less than the rotation as far as it lies along the rotation's axis. The rotations' errors of tenths of a degree that section 2.5 measured on private scene A therefore reach this measure as tenths of a degree.

**On `kitchen`: the axes, and the estimator against the ground truth.**

| | MoGe-2's normals against the ground truth's, static pixels |
| --- | --- |
| As given, §1.4's axes (x right, y down, z forward), facing the camera | **3.00°** at the median, 7.07° at the 90th percentile; 82 % under 5° |
| The next-best of the 48 signed permutations, (−x, y, z) | 14.27° at the median |
| (x, −y, z) | 31.1° |

So MoGe-2's normals are in §1.4's axes as the README says (row 883), and face the camera as the device's do: the maps go to the terms unchanged. On the dynamic objects the estimator is 13.7° from the ground truth at the median (32.3° at the 90th percentile; 12 % under 5°).

**On `kitchen`: between cameras, the floor.** One rendered scene through exact, synchronized cameras, so any angle between two cameras' normals is the estimator's own view-dependence.

| Pairs | Static: median / 90th percentile | Dynamic: median / 90th percentile | Ground truth's normals, static / dynamic, medians |
| --- | --- | --- | --- |
| Six wide pairs, about 15° | **1.81° / 4.35°** (pairs 1.64° to 1.94°) | **6.93° / 16.1°** (pairs 6.08° to 7.84°) | 0.003° / 0.11° |
| The same without the edge and "seen" filters, as private scene A is measured | 1.83° / 4.55° | 7.12° / 17.3° | |
| D-22's pairs and `cam00`–`cam01`, 2.8° to 6.6° | 1.01° / 3.18° | 4.86° / 12.1° | |
| One camera, two frames | 0.07° / 0.44° | 8.2° / 34.6°, motion included | |

The angle between cameras grows with their separation, and it is about half the estimator's error against the ground truth (1.81° against 3.00° static, 6.93° against 13.7° dynamic): most of each camera's error is common to two cameras 15° apart, as section 2.7 found for depth.

**Private scene A: between cameras.** All seven pairs at frames 0, 20, 40, 60, 80, 100 and 119 (49 pair-frames).

| | MoGe-2: median / 90th percentile | Under 5° / 10° | DAViD, in (x, −y, −z): median / 90th percentile | Under 5° / 10° |
| --- | --- | --- | --- | --- |
| The person, both ends | **6.71° / 14.7°** | 34 % / 74 % | **7.12° / 16.6°** | 31 % / 70 % |
| Frames 0 to 60 / 80 to 119 | 6.46° / 6.94° | | 7.53° / 6.72° | |
| By pair, medians | 6.02° (`cam1`–`cam4`) to 7.87° (`cam6`–`cam8`) | | 6.07° (`cam1`–`cam4`) to 8.30° (`cam6`–`cam8`) | |
| The two pairs one above the other, `cam1`–`cam4` and `cam2`–`cam5` | 6.02°, 6.15° | | 6.07°, 6.53° | |
| The background | 1.79° / 8.19° | 82 % / 93 % | – | |
| The background by pair, median / 90th percentile | 1.11° / 3.3° (`cam0`–`cam1`) to 3.99° / 15.3° (`cam6`–`cam8`); `cam2`–`cam5` 2.79° / 10.4°, `cam3`–`cam5` 2.91° / 11.1° | | – | |
| One camera, two frames: person / background | 2.65° / 0.39° (medians) | | 2.53° / – | |

- **DAViD's axes.** DAViD has no ground truth here, so its axes were searched against MoGe-2's, whose axes `kitchen` fixed, on the person's matched pixels. (x, −y, −z) is best at 11.9° at the median, against 28.4° for the next candidate and 135° as given. Its output is in the y-up, z-towards-the-viewer convention, and a map would be turned by (x, −y, −z) before use. The 11.9° between the two estimators at the same pixel is larger than either's angle between cameras.
- **What it shows.** On the person, two cameras 15° apart see normals 6.7° (MoGe-2) and 7.1° (DAViD) apart at the median. That is `kitchen`'s floor for its dynamic objects at the same separation, with exact cameras: 6.9°, or 7.1° without the edge and "seen" filters, which private scene A's figures do not have. On the person the pairs read alike, and the two pairs one above the other are not worse. The background varies more by pair, 1.1° to 4.0° at the median against `kitchen`'s static 1.6° to 1.9°, and is widest on `cam6`–`cam8`, `cam3`–`cam5` and `cam2`–`cam5`. That is not read further here: the background's matches are fewer and at 2.6 to 6 units. Section 2.7's depth found those two pairs worst (4.6 % and 4.8 % against 0.9 % to 2.1 %), from the cameras' disagreement. A normal carries none of what made them so: no scale, no shift, no anchor, and a rotation error of tenths of a degree. At the matched points this measure reaches, the estimators' normals of the person disagree between cameras about as much as an estimator's do on exact, synchronized cameras, and no excess shows that would come from each camera seeing its own version of the person.
- **What it cannot show.**
  - The person's comparison is on matched points only, where the two images agree enough to be matched. Section 2.7 found the larger gaps among unmatched pixels, and a pose difference between cameras that the matcher refuses is not seen here.
  - `kitchen`'s dynamic objects are small rendered objects, not a person, so the floor is the nearest one available, not a like-for-like one.
  - The prior's own error is about twice its disagreement between cameras on `kitchen` (13.7° on the dynamic objects). An error the cameras share does not cancel in the loss, and section 3.4's within-one-image reading holds: the prior can shape a surface and cannot place it.
- **For H2's prediction** that the loss settles between the cameras' versions and smooths: the measure finds no versions to settle between beyond the estimator's own spread, which an L1 on 6° to 7° of disagreement would still average over. Whether that smooths the renders is for the arm to show.

**The terms in the wrapper** (`<scratch>/ticket-28/wrap_train.py` and `surface_device.py`, ticket #27's copies with one term registered; their weight-0 path unchanged). The term **`prior`** is registered at weight 1 and carries its own weights: λ₇·L₇ + λ₈·L₈ through the device's two passes, computed with its whole gradient before the colour pass, as `nc` is, from iteration `--prior-from` on. Before that iteration it reports 0 and computes nothing.

- **L₇, row C7** (DN-Splatter's Eq. 10, row 886): the mean over the pixels with a prior of ‖N(p) − P(p)‖₁, with N(p) = Σwn the composited smallest-scale normal, unnormalized as its Eq. 9 composites it, and P the prior; λ₇ = 0.1. No alpha cut, since the paper's mean is over the image. The total-variation term is left out (the owner's answer at review 2).
- **L₈, row C8** (VCR-GauS's Eq. 14, row 887): the mean over the consistency term's pixels of w·(‖N_d − P‖₁ + (1 − N_d·P)); λ₈ = 0.015 and γ = 0.005.
  - N_d is the normal of the plane-distance depth map (row A5, the depth finalist 3's arm 2 read, which is PGSR's intersection depth).
  - The confidence weight is w = exp((N̂·P − 1)/γ), with N̂ the composited normal normalized (VCR-GauS's Eq. 9). It is taken without a gradient: the paper does not say, and a gradient through it would pull the rendered normal towards the prior a third time.
  - The pixels are finalist 3's: alpha ≥ 0.5 on the pixel and its four neighbours, among the pixels with a prior.
- **The maps** are read per iteration from the files, one image at a time, so nothing of them is held on the GPU. An image is found by the render's camera name and the record whose timestamp equals the model's `time.to_tau(t)`, within 10⁻⁶. A missing map, a time that matches no record or a size that differs from the camera's stops the run. `--prior-axes` turns a map into §1.4's axes: (x, y, z) for MoGe-2, (x, −y, −z) for DAViD.
- **Logged:** L₇, L₈, their pixel shares, the mean of w and the share of pixels with w above e⁻¹: at γ = 0.005, w falls to e⁻¹ at 5.7° between N̂ and P.
- **E10's g.** The term does not enter it (section 5.2): its gradient reaches the device's copies, not the colour pass's 2D means.

**Checks of the terms** (`<scratch>/ticket-28/prior_check.py`, float64 on the CPU, the reference backend; ticket #27's seven flat primitives in front of a 48 × 36 camera, a smooth synthetic prior on 80 % of the pixels):

- **Against finite differences,** the wrapper's form of the gradient, λ₇·L₇ + λ₈·L₈ with the confidence weight held at its value: plane and expected depth, γ = 0.005 and 0.5. Every tensor the device reaches passes: the worst relative error is 1·10⁻⁹ to 1·10⁻⁷ on `means`, `quats`, `log_scales`, `velocities`, `times` and `opacity_logits`. On `log_durations`, whose gradient is about 10⁻⁷ here, it is up to 9·10⁻⁴.
- **The confidence weight:** with the model's own composited normal as the prior, w = 1 at every pixel of the cut; with that prior turned by 90°, w ≈ 10⁻¹⁵ and L₈ ≈ 0. At γ = 0.005 the term thus acts only where the rendered normal already lies within a few degrees of the prior, which makes its start a setting in its own right (below).

**The terms on trained models** (`<scratch>/ticket-28/prior_on_model.py`, under the lock, at one training image each: finalist 3's `FLAT_kitchen_s0` and the off arm's `OFF_kitchen_s0` at `cam05`, frame 100; `FLAT_A_h0_s0` at `cam1`, frame 60; each 3,000,000 primitives). The priors: the image's own MoGe-2 map; on `kitchen`, the ground truth's normals; on private scene A, DAViD's map over the whole image; a wrong prior, the same frame's map of another camera (`cam12`, `cam7`); and the model's own composited normal. The angle is between the model's normalized composited normal and the prior, at the pixels with alpha above 0.5.

| Model, prior | Model against prior: median / 90th percentile | L₇ | L₈ | Pixels C8's weight passes (w > e⁻¹) |
| --- | --- | --- | --- | --- |
| `FLAT_kitchen_s0`, MoGe-2 | 17.8° / 46.7° | 0.641 | 0.173 | 11.3 % |
| `FLAT_kitchen_s0`, the ground truth | 18.6° / 49.1° | 0.661 | 0.167 | 11.1 % |
| `FLAT_kitchen_s0`, another camera's map | 32.2° / 70.3° | 0.935 | 0.071 | 4.2 % |
| `FLAT_kitchen_s0`, its own normal | 0° | 0.352 | 1.850 | 100 % |
| `OFF_kitchen_s0`, MoGe-2 / the ground truth / another camera | 17.9° / 18.6° / 32.5° (medians) | 0.670 / 0.681 / 0.954 | 0.156 / 0.166 / 0.080 | 10.0 % / 11.3 % / 5.1 % |
| `FLAT_A_h0_s0`, MoGe-2 | 42.8° / 67.0° | 1.045 | 0.035 | 1.8 % |
| `FLAT_A_h0_s0`, DAViD, whole image | 37.8° / 69.2° | 1.012 | 0.032 | 1.3 % |
| `FLAT_A_h0_s0`, another camera's map | 44.0° / 70.8° | 1.080 | 0.016 | 0.7 % |

- **The terms read the right prior as right.** On `kitchen`, L₇ is lower with the image's own map than with another camera's, and MoGe-2's map reads as the ground truth does (17.8° against 18.6°): on a trained model the estimator's 3° of error is small beside the model's own departure.
- **On private scene A, today's model is not oriented by its surface.** Its primitives' normals lie 43° from the image's prior at the median, barely nearer than another camera's map (44°): there is a great deal for row C7 to turn. On `kitchen` it is 18°. Flattening does not change this (`FLAT_` against `OFF_`, 17.8° against 17.9°).
- **C8's weight passes few pixels on a trained model:** 10 % to 11 % on `kitchen`, under 2 % on private scene A, at γ = 0.005. Unless C7 first brings the rendered normals within a few degrees of the prior, the regularizer acts on a small share of the image. That is VCR-GauS's design, where its own rendered-normal loss runs beside it.
- **The model's own normal against its depth map's normal** (the `own` rows' L₈, 1.85 to 2.50) is far from consistent, as finalist 3's L_nc of about 0.7 found.
- **Cost:** the two device passes with both terms and their whole gradient took 40 ms per call on these models (the first call 0.21 s), at a peak allocation of 2.4 GiB, the model included, the device's cost of section 6 here.

**The smoke runs** (300 iterations on `kitchen` at seed 0, `--config` holding `{"optim": {"iterations": 300}}`, as sections 5 and 6 here ran theirs). Each wrapper ran 300 times in every run.

- **At every weight 0** (`--term flat=0 --term prior=0`), the logged losses equal ticket #27's weight-0 smoke run's at iterations 100, 200 and 300 to every digit logged (total 0.445250, 0.390724, 0.345167). The copy changes nothing at weight 0. The run also recorded its 300 draws: 300 distinct images.
- **The maps for exactly those images:** MoGe-2 on the 300 drawn images, 32 s under the lock, into `<surface-runs>/ticket-28/kitchen/moge2_smoke`.
- **`flat` at 1, σt-weighted, with `prior` from iteration 1** (moved early so that the terms run in a smoke run). The run ended normally in 35 s, against 32 s at weight 0, at a peak of 3,450 MiB in `nvidia-smi` against 3,226. It drew the same 300 images as the run at weight 0, so the draws do not depend on the loss.

  | Iteration | L₇ | L₈ | Pixels with a prior / in C8's cut | C8's weight: mean / share above e⁻¹ | λ₇L₇ + λ₈L₈ | L1 (weight 0: 0.1724, 0.1397, 0.1132) |
  | --- | --- | --- | --- | --- | --- | --- |
  | 100 | 1.566 | 0.0458 | 100 % / 94.0 % | 0.040 / 5.2 % | 0.157 | 0.1740 |
  | 200 | 1.358 | 0.0345 | 100 % / 96.3 % | 0.036 / 4.5 % | 0.136 | 0.1419 |
  | 300 | 1.206 | 0.0331 | 100 % / 98.2 % | 0.039 / 4.6 % | 0.121 | 0.1147 |

  The figures are the logged rows, each the mean over the 100 iterations before it. L₇ falls as the term turns the primitives, while at 300 iterations they are still near D-25's spheres. C8's weight passes about 5 % of the cut's pixels, as on the trained models. The prior adds 0.12 to 0.16 to a total of 0.47 to 0.61.
- **The same with `--prior-from 100`,** the path a late start takes. It ended normally in 35 s, both wrappers 300 times, with the same logged keys in every row. The row at iteration 100, the mean over iterations 1 to 100, carries the term for its one active iteration: pixel share 0.01, L₇ 0.014, a hundredth of a full iteration's. The rows at 200 and 300 equal the first run's to three figures (L₇ 1.372 and 1.215).

**The arms, as planned for the owner's go.**

- **The main arm, `PRIOR`:** the default training configuration, 30,000 iterations, through the wrapper copy with `--term flat=1 --flat-weighting sigma_t --term prior=1 --prior-maps <MoGe-2's maps> --prior-axes x,y,z --prior-c7 0.1 --prior-c8 0.015 --prior-gamma 0.005 --prior-depth plane --prior-from <start>` (question 1). Eight runs, mirrored on finalist 3's: the centre and left copies at seeds 0 and 1 from `ref_h0_s4` and `ref_h1_s4`; `kitchen` at seeds 0 to 2 and `studio` at seed 0 from M17's init files.
- **DAViD's arm, `DAVID`** (cleared): the same on private scene A's two copies at seeds 0 and 1, with DAViD's maps, `--prior-axes x,-y,-z`, and every pixel off the person marked as carrying no prior (question 3).
- **Read against** the off arm (section 5 here; on `kitchen` as the owner answered), with finalist 3's `FLAT_…` (what the prior adds to flattening) and `FLATNC_…` (the prior against consistency without one) beside it, by section 6 here's copied scripts: the named measure by the rule, with the tail by its margin; the under-two-frames share; the sharpness ratio, the tiles' coverage and the sheets, relabelled blind; the test-view scores and Q-233's kept-pixel PSNR on the centre copy; on openftgs-synth `scripts/gt_depth.py`'s shares behind and in front of the surface (a surface term is expected to lower the share behind by more than 0.005), SSIM₁ against the 0.002 guard and `scripts/reference_flaws.py`'s kept-pixel PSNR; row A1's scale ratios by `ratio_stats.py`; and the logged L₇, L₈ and the share of pixels the confidence weight passes. Each `kitchen` run then gets the owner's labelled sheet (`kitchen_sheets.sh`).
- **The full maps,** before the arms: MoGe-2 on every image of private scene A (1,080), `kitchen` (6,000) and `studio` (1,440), and DAViD on private scene A's 1,080, by `normal_maps.py` without `--frames`. Private scene A's two copies and its full manifest name the same images (equal in content and timestamp at every camera and frame), so one map set, keyed by camera and frame, serves both copies. About 3.2 GB, 25.7 GB and 9.3 GB for MoGe-2 and 3.2 GB for DAViD, in `<surface-runs>/ticket-28`.
- **GPU hours, estimated** from finalist 3's measured times (section 6 here, "Times"), which ran with 14 CPU threads; the arms would run with the same, `THREADS=14` in the copied `run_train.sh`, whose default of 4 served this step's smoke runs: the device's two passes cost what they cost for `nc`, and the map read and the two terms are per-pixel work beside them.

  | Part | From iteration 15,000 | From iteration 1 |
  | --- | --- | --- |
  | MoGe-2's full maps (about 0.06 s an image in the network, decoding beside it) | 0.25 h | 0.25 h |
  | The main arm: 4 runs on private scene A, 3 on `kitchen`, 1 on `studio` | 4 × 23 + 3 × 26.3 + 35.2 min, about 3.4 h (`FLATNC_…`'s times) | at most 4 × 32.5 + 3 × 35.6 + 46.9 min, about 4.7 h (the device at every iteration, at its cost at the end of the run) |
  | Its post-processing, 8 runs, and 3 labelled `kitchen` sheets | about 0.35 h | about 0.35 h |
  | The main arm in all | **about 4.0 h** | about 5.3 h |
  | DAViD's arm: 4 runs on private scene A, with post-processing (its maps on the CPU, about 25 min, no GPU) | about 1.7 h | about 2.3 h |
  | Both arms | **about 5.7 h** | about 7.6 h |

  Section 5.6's 4.5 hours and 2 more for DAViD's arm. Memory: finalist 3's arm 2 peaked at 23,575 to 23,924 MiB on private scene A with the same two device passes, and the prior adds one image's map (11 to 25 MB) and per-pixel tensors of L_nc's size, so the arm would run with the fallbacks the owner cleared at step 5 (`perf.frame_cache` `"mmap"` on private scene A, `PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True` on `studio`) ready in advance (question 4).


**The open choices, with this step's recommendations.**

1. **The terms' start.** Neither paper states one (rows 886, 887), and DN-Splatter starts from oriented primitives, which this arm does not have. At λ_flat = 1 the smallest axis is distinct by iteration 15,000 and not yet at 7,000 (section 6 here, stage 1), and before that the normal of a primitive is a tie between near-equal scales. C8's confidence weight passes only pixels whose rendered normal lies within about 6° of the prior, so it does little until C7 has turned the primitives. Recommended: **iteration 15,000**, the start finalist 3's arm 2 had, so that the arm reads against `FLATNC_…` like for like; from iteration 1 costs about 1.3 hours more, and puts the prior on near-spheres.
2. **The settings fixed above, and two readings of the papers that change numbers.**
   - **C7:** on the unnormalized Σwn (DN-Splatter's form; it also pulls the alpha towards 1 where a prior is given), mean over the pixels with a prior, no alpha cut.
   - **C7's norm:** the L1 is the sum of the three components' differences per pixel, ‖·‖₁ as Eq. 10 writes it. A mean over the three components would be a third of it at the same λ₇ = 0.1.
   - **C8:** on the plane-distance depth (the owner's answer for finalist 3), on finalist 3's pixels, with the confidence weight taken without a gradient.
   - **C8's weight reads the composited normal.** VCR-GauS's Eq. 13, as fetched, writes the weight's normal with a subscript d, which would make it the depth map's normal; its text says the weight uses "the rendered normal learned from multi-view pseudo normals", the composited one. The text is followed here. With the depth map's normal, the weight would pass the pixels where L₈ is already small.
   - **The prior's size.** At the papers' weights the prior is not small beside the rendering loss: on the trained models above, λ₇L₇ + λ₈L₈ is 0.067 on `kitchen` and 0.105 on private scene A. The final total loss of the same runs is 0.0565 and 0.0476, so the prior would weigh 1.2 and 2.2 times as much. Finalist 3's consistency term at its paper's weight softened the renders (section 6 here). No calibration is proposed, since section 5.6 fixes the papers' weights, but this is the setting most likely to cost sharpness.
   - Recommended as fixed. The alternatives are C7 on the normalized N̂ (VCR-GauS's L_n form), C7 as a mean over components, C8 on expected depth (the screening row's 0.35 against 0.40, row 726), and the depth map's normal in C8's weight.
3. **DAViD's person.** DAViD's maps mean something on the person only (section 5.6), and the one model cleared is the normal model, so its mask cannot come from DAViD's segmentation model without a further check and clearance. Options: (a) the person as section 2.7 defines it, the pixels of rule a's crop that ticket #12's Depth Anything V2 depth, aligned on the pair's matches, lifts to 0.5 to 1.8 units, placeholder pixels left out: the rule this PRD's measures already use, from maps already made for all 1,080 images, a CPU job of minutes; (b) DAViD's `foreground-segmentation-model-vitl16_384.onnx` (MIT, row 569), its terms checked again and cleared by the owner; (c) rule a's crop alone, which leaves background inside the box with a meaningless prior. Recommended: (a).

**The questions for the owner at the first step.**

1. **The start** of the two prior terms: iteration 15,000 (recommended) or 1 (open choice 1).
2. **The settings** of open choice 2: as fixed (recommended) or another.
3. **DAViD's person mask:** (a) section 2.7's depth rule (recommended), (b) DAViD's segmentation model, or (c) rule a's crop.
4. **The hours:** about 5.7 GPU hours for both arms with the start at 15,000, the maps and the sheets included (about 7.6 from iteration 1), against section 5.6's 6.5; and the memory fallbacks of step 5 cleared again for these arms.

Questions 1, 2 and 4 block the main arm; question 3 blocks DAViD's arm only. Nothing of this step depends on them.


**The first step's commands.** Each ran with `OMP_NUM_THREADS=4 MKL_NUM_THREADS=4 OPENBLAS_NUM_THREADS=4` and the shared `.venv`'s Python, the CPU ones with `CUDA_VISIBLE_DEVICES=` empty and the GPU ones under the shared lock. Scripts that read downloaded code or weights ran with `python -I` and put their paths in themselves. Others ran with `PYTHONPATH=<worktree>/src:<worktree>/harness/src:<worktree>/tests:<scratch>/ticket-28`. `$T` is `<scratch>/ticket-28`, `$O` is `<surface-runs>/ticket-28`, `$K` is `<m15-runs>/kitchen/scene.json`, `$M` is `<m17-runs>`, `$G` is `$OPENFTGS_DATASETS/synth/ground_truth/kitchen`, `$R12` is `<surface-runs>/ticket-12/kitchen`, `$R4` is `<surface-runs>/ticket-4/private-scene-A`, `$X` is `<private-scene-A-exp>`, and `$A` is `<private-scene-A>/scene.json`. The scripts are not committed.

| Command | Device | Time | Note |
| --- | --- | --- | --- |
| `curl -sSL` of `https://huggingface.co/Ruicheng/moge-2-vitl-normal/raw/main/README.md`, `https://huggingface.co/api/models/Ruicheng/moge-2-vitl-normal` and `…/resolve/cb0e8bb…/model.pt` | CPU | about 1 min | Row 882; the hash against the linked ETag |
| `curl -sSL` of `https://api.github.com/repos/microsoft/MoGe/commits/main`, `https://raw.githubusercontent.com/microsoft/MoGe/74fbce0…/LICENSE`, then `https://codeload.github.com/microsoft/MoGe/tar.gz/74fbce0…` | CPU | seconds | Row 883; the license read before the archive was opened |
| `curl -sSL` of DAViD's `README.md`, `LICENSE-MIT.txt`, `model_cards/surface_normal_model.md`, file list and `runtime/` at 20a3eb6, then the README's link to `normal-model-vitl16_384.onnx` | CPU | about 2 min | Row 884; the MD5 against the host's header |
| `pip download --no-deps -d $T/dl/wheels onnxruntime==1.32.0 flatbuffers protobuf`; the wheels' license files read; `pip install --no-deps --no-index --target $T/env-ort $T/dl/wheels/*.whl` | CPU | seconds | Row 885; `pip freeze` of the shared `.venv` against the baseline: unchanged |
| `curl -sSL -o $T/dl/<paper>/page.html https://arxiv.org/html/<id>`, for 2403.17822 and 2406.05774, then a text extraction | CPU | seconds | Rows 886, 887 |
| `bash $T/maps_first.sh`: `python -I $T/normal_maps.py $K $O/kitchen/moge2 --estimator moge2 --weights $T/weights/moge2/model.pt --code <MoGe's archive> --frames 0,1,100,101,200,201`; the same on `$A` into `$O/private-scene-A/moge2` with `--frames 0,1,20,40,60,80,81,100,119` | GPU | 13 s and 7 s after loading | MoGe-2's maps, 120 and 81 images |
| `python -I $T/normal_maps.py $A $O/private-scene-A/david --estimator david --weights $T/weights/david/normal-model-vitl16_384.onnx --frames <the same> --device cpu --threads 4` | CPU | 1 min 53 s | DAViD's maps, 81 images; a one-image timing run before it |
| `python $T/normals_agree.py $K $O/checks/fake_wide.json --fake-gt --gt $G --grids $R12/wide --frames 0,100,200 --max-matches 200000`; again with `--turn-b 0,2,0` and `--turn-b 0.3,0,0` | CPU | 10 s each | The checks of the measure |
| `python $T/normals_agree.py $K $O/results/kitchen_wide_identity.json --maps $O/kitchen/moge2 --gt $G --grids $R12/wide --frames 0,100,200 --axes-search --quant-check --max-matches 200000` | CPU | 52 s | The axes, the estimator against the ground truth, the wide pairs, the code's cost |
| `python $T/normals_agree.py $K $O/results/kitchen_d22.json --maps $O/kitchen/moge2 --gt $G --grids $R12/d22 $R12/cam00 --frames 0,100,200 --max-matches 200000` | CPU | about 1 min | D-22's pairs |
| `python $T/normals_agree.py $A $O/results/A_moge2.json --maps $O/private-scene-A/moge2 --crops $R4/crops.json --placeholder $R4/placeholder_masks.npz --grids $R4/d22 --frames 0,20,40,60,80,100,119` | CPU | about 1 min | Private scene A with MoGe-2 |
| The same with `--maps $O/private-scene-A/david --compare-maps $O/private-scene-A/moge2 --axes-search`, then with `--axes x,-y,-z` | CPU | about 1 min each | DAViD's axes and its agreement |
| `python $T/prior_check.py values $O/checks/prior_values.json`; `python $T/prior_check.py fd $O/checks/prior_fd.json` | CPU | seconds | The terms' checks |
| `bash $T/prior_on_model.sh`: `python $T/prior_on_model.py $O/checks/prior_on_model.json FLAT_kitchen_s0=<surface-runs>/ticket-27/runs/FLAT_kitchen_s0/checkpoint=$K=cam05=100=<priors> OFF_kitchen_s0=<surface-runs>/ticket-26/runs/OFF_kitchen_s0/checkpoint=… FLAT_A_h0_s0=<surface-runs>/ticket-27/runs/FLAT_A_h0_s0/checkpoint=$X/scene_h0/scene.json=cam1=60=<priors>` | GPU | about 1 min, after the lock | The terms on trained models; run twice, the first stopping on the script's own parsing of the priors' list |
| `bash $T/smoke.sh`: `bash $T/run_train.sh smoke_kitchen_off $K $M/kitchen/init.safetensors 0 --term flat=0 --term prior=0 --record-draws $O/smoke/draws_s0.json -- --config <surface-runs>/ticket-27/smoke/cfg300.json`; `python -I $T/normal_maps.py $K $O/kitchen/moge2_smoke --estimator moge2 … --draws $O/smoke/draws_s0.json`; `bash $T/run_train.sh smoke_kitchen_prior … --term flat=1 --flat-weighting sigma_t --term prior=1 --prior-maps $O/kitchen/moge2_smoke --prior-axes x,y,z --prior-c7 0.1 --prior-c8 0.015 --prior-gamma 0.005 --prior-depth plane --prior-from 1 -- --config …` | GPU | 32 s, 32 s, 35 s | The smoke runs |
| `bash $T/run_train.sh smoke_kitchen_prior_from100 …` with the third run's terms and `--prior-from 100` | GPU | 35 s | The late start's path |

The private scene's folders, `<m15-runs>`, `<m17-runs>` and tickets #4's, #12's, #26's and #27's folders were read and never written.

**In short, for the owner, before the arms.**

- **The weights and code are as row 830 found them.** MoGe-2's `model.pt` (MIT, commit unchanged) and DAViD's normal model (MIT) still pass. MoGe's code is MIT with its DINOv2 part Apache-2.0, and the ONNX runtime is MIT (Eigen inside it MPL-2.0), in a scratch environment only.
- **MoGe-2's normals are in §1.4's axes as given,** 3.0° from `kitchen`'s ground truth on static surfaces. DAViD's need (x, −y, −z).
- **On private scene A's person the two estimators' normals disagree between cameras 15° apart by 6.7° and 7.1° at the median.** That is what MoGe-2 gives on `kitchen`'s dynamic objects through exact cameras (6.9°). On the person the pairs read alike, the two pairs one above the other included, which section 2.7's depth found worst. At the person's matched points, the normal prior shows no sign of the cameras' versions of the person that the depth priors inherited.
- **The terms are in the wrapper copy and checked:** finite differences, the weight-0 parity, and a 300-iteration smoke run with flattening and both terms.
- **On trained models today's primitives' normals lie 18° (`kitchen`) and 43° (private scene A) from the image's prior,** and C8's confidence weight at γ = 0.005 passes 2 % to 11 % of the pixels: row C7 has much to turn, and row C8 acts mostly after it has.
- **The plan:** one arm on both sets (8 runs) and DAViD's arm on private scene A (4 runs), about 5.7 GPU hours with the terms from iteration 15,000, the full maps and the `kitchen` sheets included. Four questions above; none trains anything until answered.
- **This step's GPU time** was about 6 minutes in all: the maps, the two passes of the checks on trained models and the four smoke commands. Everything else ran on 4 CPU threads.

**In short, for the owner, after the arms.**

- **The prior turns the primitives fast** and does not move the shell enough to adopt. With MoGe-2's maps, from iteration 15,000 at the papers' weights, the near-side far-5 cm share is lower than the off arm's and than flattening alone on the left copy only (by 0.011 and 0.016), and lower than finalist 3's consistency term's on the centre copy only.
- **The renders turn softer:** the sharpness ratio falls by 0.035 on both copies. No test-view score falls: crop PSNR rises by 0.02 and 0.11 dB, E16's PSNR by 0.17 and 0.15 dB, and the kept-pixel PSNR by 0.19 dB.
- **On `kitchen` it halves the expected depth's error** and removes most of what floated in front of the surface, with no harm to the image scores, but the share behind the surface does not fall; on `studio`'s one seed it rises beyond its guard.
- **DAViD's maps on the person do less** and cost up to 0.28 dB.
- **By section 7.2's row** this is its third column, a measure that moves in part with softer renders: not adopted alone and not dropped, waiting for a disagreement-side term or a lower, calibrated weight.

**The arms, run** from 2026-11-18 to 2026-11-19, at the owner's answers above, after finalist 2's variant queue had ended, with this branch's code, the same versions and hardware: the RTX 4090 alone under the lock, 14 CPU threads, one training at a time. Every run wrote only under `<surface-runs>/ticket-28`. Scripts read tensors, JSON, images, maps and mask files and wrote numbers and files; no image, render, map or sheet of private scene A was looked at (row 889).

- **The maps first.** MoGe-2 on every image: private scene A's 1,080 in 93 s, `kitchen`'s 6,000 in 10 min 34 s, `studio`'s 1,440 in 3 min 16 s (54 to 58 ms per image in the network), 3.0, 24 and 8.7 GB. DAViD on private scene A's 1,080 on the CPU, 4 threads, in 25 min 18 s.
- **DAViD's person**, by section 2.7's rule (`<scratch>/ticket-28/person_masks.py`, CPU, 3 min 33 s). Each camera is aligned on the first of D-22's pairs it belongs to, with 49,941 to 50,000 anchors per image. The person is 28.3 % of an image at the median (16.9 % at the least); section 2.7 found 28.8 %. Every other pixel carries no prior.
  - **One map set for both copies uses the held-out camera's images.** With the first pair, `cam1`'s mask is aligned on `cam0`–`cam1`, whose `cam0` is the centre copy's held-out camera. `cam0`'s mask comes from `cam0`–`cam1` and `cam4`'s from `cam1`–`cam4`, both with `cam1`, the left copy's.
  - What enters a training image's mask from the held-out camera is the scale and shift of one alignment, two numbers per image, through the triangulated anchors. That is D-66's concern in a small form.
  - Sized afterwards, with `cam1` aligned on `cam1`–`cam4` and `cam4` on `cam3`–`cam4`: 1.4 % and 2.2 % of their person pixels change at the median, at most 4.2 %. `cam0` has no other pair in D-22's set. The arms were not rerun.
- **The runs:** twelve, all 30,000 iterations of the default training configuration through the wrapper copy with `--term flat=1 --flat-weighting sigma_t --term prior=1 --prior-c7 0.1 --prior-c8 0.015 --prior-gamma 0.005 --prior-depth plane --prior-from 15000`.
  - `PRIOR_…`, MoGe-2's maps with `--prior-axes x,y,z`: private scene A's two copies at seeds 0 and 1 from the kept stride-4 files; `kitchen` at seeds 0 to 2 and `studio` at seed 0 from M17's init files.
  - `DAVID_…`, DAViD's person maps with `--prior-axes x,-y,-z`: private scene A's two copies at seeds 0 and 1.
  - Every wrapper ran 30,000 times in every run.
- **Memory.** No run needed a fallback. The arms fit beside the frame cache on the GPU on private scene A, at peaks of 23,416 to 24,006 MiB in `nvidia-smi`, and with the cache memory-mapped on openftgs-synth (22,446 to 23,740 MiB). D-51's `auto` placed the cache as in the off arm.
- **Times:** 22 min 44 s to 23 min 6 s on private scene A; 27 min 7 s to 27 min 11 s on `kitchen`; 36 min 24 s on `studio`.
- **GPU hours:** the maps, the arms and their post-processing took 5.49 hours (00:04 to 05:33 on 2026-11-19), the labelled `kitchen` sheets 20 s, and the first step 0.1 hours: about 5.6 hours in all, inside the 5.7 cleared.

**The named measure,** the near-side far-5 cm share against the kept files, with the tail beyond 25 cm (each cell 5 cm / tail), and the other shell measures. The off arm's and finalist 3's runs are section 6 here's.

| Run | Near side: far-5 cm / beyond 25 cm | Near side: far-10 cm | Under two frames: near / whole | Whole model: far-5 cm | Far side: far-5 cm | Cap reached at |
| --- | --- | --- | --- | --- | --- | --- |
| `OFF_A_h0_s0` / `s1` | 0.1308 / 0.0461; 0.1380 / 0.0333 | 0.0754; 0.0644 | 0.5857 / 0.454; 0.5836 / 0.451 | 0.218; 0.225 | 0.537; 0.536 | 10,900; 10,800 |
| `PRIOR_A_h0_s0` | 0.1279 / 0.0301 | 0.0573 | 0.6085 / 0.476 | 0.217 | 0.541 | 11,000 |
| `PRIOR_A_h0_s1` | 0.1360 / 0.0388 | 0.0704 | 0.6056 / 0.476 | 0.216 | 0.523 | 10,600 |
| `DAVID_A_h0_s0` | 0.1380 / 0.0386 | 0.0688 | 0.6057 / 0.448 | 0.226 | 0.523 | 10,600 |
| `DAVID_A_h0_s1` | 0.1300 / 0.0284 | 0.0560 | 0.6115 / 0.453 | 0.222 | 0.534 | 10,900 |
| `OFF_A_h1_s0` / `s1` | 0.1411 / 0.0379; 0.1337 / 0.0264 | 0.0662; 0.0561 | 0.5808 / 0.445; 0.5893 / 0.449 | 0.227; 0.225 | 0.536; 0.540 | 10,000; 10,000 |
| `PRIOR_A_h1_s0` | 0.1213 / 0.0269 | 0.0513 | 0.6127 / 0.473 | 0.213 | 0.551 | 9,800 |
| `PRIOR_A_h1_s1` | 0.1307 / 0.0276 | 0.0557 | 0.6151 / 0.472 | 0.219 | 0.535 | 9,700 |
| `DAVID_A_h1_s0` | 0.1287 / 0.0316 | 0.0568 | 0.6001 / 0.439 | 0.224 | 0.535 | 9,800 |
| `DAVID_A_h1_s1` | 0.1387 / 0.0251 | 0.0582 | 0.6092 / 0.441 | 0.234 | 0.537 | 9,800 |

**Judged by section 4.4's rule** (both of the arm's seeds beyond both of the reference's, the means more than 0.01 apart; 0.021 for the tail), against the off arm and against finalist 3's two arms, as section 5.6 asks:

| Copy, measure | `PRIOR` against the off arm | against `FLAT` (what the prior adds to flattening) | against `FLATNC` (the prior against consistency) | `DAVID` against the off arm / `FLAT` / `FLATNC` |
| --- | --- | --- | --- | --- |
| Centre, named measure | not moved (0.128, 0.136 against 0.131, 0.138; mean 0.003 lower) | not moved (against 0.124, 0.146) | **lower by 0.014** (against 0.147, 0.145) | not moved / not moved / **lower by 0.012** |
| Left, named measure | **lower by 0.011** (0.121, 0.131 against 0.141, 0.134) | **lower by 0.016** (against 0.137, 0.147) | not moved (against 0.135, 0.134; mean 0.008 lower) | not moved (mean 0.004 lower) / not moved / not moved |
| Either copy, tail | not moved (means 0.005 lower) | not moved | not moved | not moved |
| Under two frames, near side (reported) | **higher** by 0.022 and 0.029 | higher by 0.025, 0.032 | higher by 0.021, 0.025 | higher by 0.024, 0.020 against the off arm |

- **The prior lowers the named measure in one place against each reference, never in both copies.** Against the off arm and against flattening alone, it is lower on the left copy and not on the centre copy. Against the consistency term it is the other way round, since `FLATNC` raised the centre copy (section 6 here).
- **DAViD's maps move it against `FLATNC` alone, on the centre copy.** On the person they do less than MoGe-2's over the whole image.
- **The near side's under-two-frames share rises** by 0.02 to 0.03 in all eight runs, beyond every reference's seeds. The prior's runs keep more short-lived primitives near the camera, which section 4.4 names as finalist 2's measure, not this one's.
- **The tail does not move.**
- **How thin these moves are.** With two seeds a side, both seeds beyond both happens by chance about one time in six whichever way. Twelve comparisons were read on the named measure (two copies, three references, two arms) and four moved, all four lower. That is above chance but not by much. What keeps finalist 4 off the drop clause is `kitchen`'s depth error on three seeds, not the named measure.

**Row A1: the scale ratios at iteration 30,000** (`ratio_stats.py`; medians of min/mid and min/max, and the share with min/max below 0.01):

| Runs | Whole model | Near side | Near side's far-5 cm part (the shell) |
| --- | --- | --- | --- |
| `FLAT_…`, private scene A, 4 runs (section 6 here) | 0.0001, 2·10⁻⁵ to 5·10⁻⁵, 0.72 to 0.74 | 0.0016 to 0.0026, 0.0007 to 0.0011, 0.64 to 0.66 | 0.0000, 0.0000, 0.87 to 0.88 |
| `PRIOR_…`, private scene A, 4 runs | 0.0001, 2·10⁻⁵ to 4·10⁻⁵, 0.73 to 0.75 | 0.0013 to 0.0017, 0.0006 to 0.0007, 0.66 to 0.67 | 0.0000, 0.0000, 0.83 to 0.89 |
| `DAVID_…`, private scene A, 4 runs | 0.0000 to 0.0001, 7·10⁻⁶ to 2·10⁻⁵, 0.73 to 0.75 | 0.0012 to 0.0019, 0.0005 to 0.0008, 0.64 to 0.66 | 0.0000, 0.0000, 0.86 to 0.88 |
| `FLAT_…` / `PRIOR_…`, `kitchen`, 3 seeds each | 0.090, 0.022, 0.41 / 0.041 to 0.042, 0.0096 to 0.0098, 0.50 | – | – |
| `FLAT_…` / `PRIOR_…`, `studio` | 0.0000, 0.0000, 0.86 / 0.0000, 0.0000, 0.87 | – | – |

The prior keeps the flattening and adds to it on `kitchen`, where flattening alone left most primitives thicker. Every run reached the cap by iteration 11,000, before the prior's start at 15,000, so these runs do not exercise growth under the prior. That E11's and E17's copies would keep its turns follows from their keeping the parent's rotation, not from these runs.

**The prior terms over the run** (the logged rows, each the mean of the 100 iterations before it):

| Runs | At iteration 15,100: L₇ / L₈ / share C8's weight passes | At 30,000: L₇ / L₈ / share | λ₇L₇ + λ₈L₈ at 30,000, of the total loss | L1 at 30,000 (off arm) |
| --- | --- | --- | --- | --- |
| `PRIOR_…`, private scene A | 0.665 to 0.680 / 0.259 to 0.270 / 16 % to 17 % | 0.138 to 0.142 / 0.438 to 0.467 / 78 % to 79 % | 0.020 to 0.021 of 0.073 to 0.075 | 0.0118 to 0.0122 (0.0104 to 0.0107) |
| `DAVID_…`, private scene A | 0.697 to 0.723 / 0.108 to 0.130 / 7 % to 9 % | 0.247 to 0.256 / 0.257 to 0.281 / 43 % to 46 % | 0.029 to 0.030 of 0.080 to 0.082 | 0.0117 to 0.0119 |
| `PRIOR_…`, `kitchen` | 0.458 to 0.543 / 0.255 to 0.348 / 22 % to 32 % | 0.076 / 0.194 to 0.198 / 91 % | 0.0105 of 0.068 | 0.0133 to 0.0135 |
| `PRIOR_…`, `studio` | 0.312 / 0.538 / 45 % | 0.089 / 0.258 / 89 % | 0.013 of 0.078 | 0.0139 |

- **Row C7 turns the primitives quickly:** L₇ falls from about 0.67 to 0.14 on private scene A, and C8's weight then passes 78 % of the pixels instead of 16 %, as VCR-GauS's design has it.
- **The prior ends at about 0.39 times the rest of the loss on private scene A** (0.021 against about 0.054; 0.57 times with DAViD's maps). The trained models gave 2.2 times before it acted: the primitives move to meet it.
- **L1 ends 11 % to 17 % higher than the off arm's on private scene A** (0.0117 to 0.0122 against 0.0104 to 0.0107).

**The test-view scores, the sharpness ratio and the tiles' coverage,** as section 3 here takes them. Kept-pixel PSNR is Q-233's, on the centre copy.

| Run | Crop PSNR / SSIM | Sharpness ratio (late frames) | Training views: PSNR / sharpness | E16 PSNR / SSIM₁ / ꟻLIP | Kept-pixel PSNR | Tiles with alpha ≥ 0.5: front; side |
| --- | --- | --- | --- | --- | --- | --- |
| `OFF_A_h0_s0` / `s1` | 24.35 / 0.8148; 24.36 / 0.8153 | 0.7313 (0.7536); 0.7258 (0.7497) | 32.99 / 0.7733; 32.84 / 0.7740 | 24.80 / 0.8180 / 0.1531; 24.66 / 0.8176 / 0.1538 | 25.54; 25.37 | 0.86 to 1.00; 0.53 to 1.00 / 0.82 to 1.00; 0.47 to 0.87 |
| `PRIOR_A_h0_s0` / `s1` | 24.37 / 0.8205; 24.38 / 0.8210 | **0.6913** (0.7090); **0.6935** (0.7212) | 32.14 / 0.7483; 31.96 / 0.7418 | 24.79 / 0.8256 / 0.1492; 25.00 / 0.8255 / 0.1492 | 25.53; 25.76 | 0.89 to 1.00; 0.94 to 1.00 / 0.83 to 1.00; 0.92 to 1.00 |
| `DAVID_A_h0_s0` / `s1` | 24.33 / 0.8185; 24.32 / 0.8191 | 0.6997 (0.7111); 0.6952 (0.7112) | 32.40 / 0.7604; 32.08 / 0.7565 | 24.81 / 0.8202 / 0.1534; 24.34 / 0.8189 / 0.1560 | 25.55; 25.02 | 0.93 to 1.00; 0.45 to 0.97 / 0.80 to 1.00; 0.46 to 0.85 |
| `OFF_A_h1_s0` / `s1` | 23.92 / 0.7487; 23.98 / 0.7489 | 0.7323 (0.7804); 0.7353 (0.7874) | 31.57 / 0.7727; 31.36 / 0.7706 | 21.77 / 0.7403 / 0.1952; 21.89 / 0.7387 / 0.1947 | – | 0.81 to 1.00; 0.53 to 0.89 / 0.80 to 1.00; 0.51 to 0.87 |
| `PRIOR_A_h1_s0` / `s1` | 24.04 / 0.7554; 24.08 / 0.7558 | **0.6989** (0.7477); **0.6990** (0.7486) | 30.97 / 0.7491; 30.99 / 0.7487 | 21.79 / 0.7444 / 0.1912; 22.17 / 0.7438 / 0.1890 | – | 0.86 to 1.00; 0.75 to 1.00 / 0.87 to 1.00; 0.88 to 1.00 |
| `DAVID_A_h1_s0` / `s1` | 23.75 / 0.7490; 23.85 / 0.7512 | 0.7119 (0.7559); 0.7113 (0.7565) | 31.12 / 0.7587; 30.98 / 0.7589 | 21.41 / 0.7396 / 0.1985; 21.70 / 0.7418 / 0.1947 | – | 0.78 to 1.00; 0.47 to 0.90 / 0.78 to 1.00; 0.40 to 0.86 |

- **The renders turn softer.** The sharpness ratio falls by 0.036 (centre) and 0.035 (left) with MoGe-2's maps and by 0.031 and 0.022 with DAViD's. The off arm's seeds lie 0.003 to 0.006 apart, and `FLAT`'s and `FLATNC`'s means lie 0.019 to 0.045 above the prior arms'. It is larger than the softening of finalist 3's calibration run at 2DGS's weight, 0.031 (0.705 to 0.674), on one seed. On the training views PSNR falls by 0.5 to 0.9 dB and sharpness by 0.025 to 0.03.
- **No test-view score falls by 0.3 dB.** `PRIOR`'s crop PSNR is 0.02 and 0.11 dB above the off arm's, its E16 PSNR 0.17 and 0.15 dB above, its kept-pixel PSNR 0.19 dB above, and its SSIM₁ and ꟻLIP better on both copies. `DAVID`'s are 0.03 to 0.28 dB below, the left copy's E16 PSNR closest to the guard (−0.28).
- **The side views fill:** with MoGe-2's maps the side tiles reach 0.75 to 1.00 of alpha ≥ 0.5, against 0.47 to 1.00 in the off arm. DAViD's, on the person alone, do not.

**The sheets.** For private scene A: four sets, the centre copy at seeds 0 and 1, then the left copy at seeds 0 and 1. Each holds five sheets, the off arm, `FLAT`, `FLATNC`, `PRIOR` and `DAVID` in a seeded order, labelled only by set and letter, in `<surface-runs>/ticket-28/sheets/blind`, with `KEY.json` apart. They await the owner's judgement with their labels hidden. For `kitchen` (the owner's request): the labelled sheet of each `PRIOR_kitchen_s<n>` from the fixed request (SHA-256 `5560b8a3…6fd27c`, checked), in `<surface-runs>/kitchen-sheets/sheets`, and one side-by-side per seed, `PRIOR`, `FLAT`, `OFF` and `PLAIN`, in `<surface-runs>/kitchen-sheets/compare/finalist4_kitchen_s<n>.jpg`. The lowest tile coverage is 0.79 to 0.81 against the off arm's 0.82 to 0.85 and `FLAT`'s 0.75 to 0.86. The labelled sheets were made after the arms, since the sheet script asked for its checkout (`FTGS_WORKTREE`) after it was first called.

**openftgs-synth.** The columns are as in section 6 here. On `kitchen`, at the owner's answer, SSIM₁, PSNR and ꟻLIP are read against the off arm's five seeds, and the rest against its seeds 0 to 2.

| Run | PSNR / SSIM₁ / ꟻLIP | Kept-pixel PSNR | Residual: in front / behind (all) | Dynamic: in front / behind | Expected depth's error, median: all / dynamic | Far share (0.12 S): whole / dynamic | Beyond 25 cm | Under two frames |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `OFF_kitchen_s0` to `s2` | 24.88 to 27.58 / 0.8753 to 0.8772 / 0.0910 to 0.0949 | 29.16 to 30.08 | 0.041 to 0.042 / 0.294 to 0.295 | 0.015 to 0.016 / 0.615 to 0.616 | 206 to 247 / 44 to 73 mm | 0.035 to 0.036 / 0.002 to 0.003 | 0.023 to 0.024 / 0.001 | 0.381 to 0.383 / 0.655 to 0.662 |
| `FLATNC_kitchen_s0` to `s2` | 26.15 to 27.41 / 0.8768 to 0.8785 / 0.0897 to 0.0923 | 29.81 to 30.83 | 0.035 to 0.036 / 0.299 to 0.301 | 0.014 to 0.016 / 0.622 to 0.627 | 169 to 203 / 29 to 39 mm | 0.032 / 0.002 to 0.003 | 0.019 to 0.020 / 0.001 | 0.379 to 0.381 / 0.657 to 0.663 |
| `PRIOR_kitchen_s0` to `s2` | 25.81 to 29.39 / 0.8761 to 0.8796 / 0.0909 to 0.0951 | 28.91 to 30.82 | **0.017 to 0.018** / 0.294 to 0.298 | 0.009 to 0.010 / 0.622 to 0.627 | **109 to 123 / 16 to 17 mm** | 0.022 / 0.002 | 0.012 to 0.013 / 0.001 | 0.358 to 0.361 / 0.659 to 0.666 |
| `OFF_studio_s0` | 22.83 / 0.7986 / 0.1899 | 23.02 | 0.014 / 0.414 | 0.009 / 0.649 | 172 / 102 mm | 0.044 / 0.001 | 0.071 / 0.001 | 0.384 / 0.707 |
| `PRIOR_studio_s0` | 23.17 / 0.7978 / 0.1863 | 23.13 | 0.005 / **0.423** | 0.006 / 0.650 | 189 / 91 mm | 0.026 / 0.000 | 0.043 / 0.001 | 0.377 / 0.692 |

- **No guard is crossed on `kitchen`.** Against the off arm's five seeds, SSIM₁ is 0.0010 higher (the guard is 0.002 the other way), PSNR 0.28 dB higher and ꟻLIP unchanged, none flagged; kept-pixel PSNR is 0.51 dB higher against seeds 0 to 2. Against the five plain seeds, SSIM₁ is 0.0010 lower, inside the guard.
- **The geometry moves on `kitchen`, but not as expected.** The expected depth's error halves (from 206 to 247 mm to 109 to 123 mm; on the dynamic objects from 44 to 73 mm to 16 to 17 mm), and the residual in front of the surface falls from 0.041 to 0.017. The share behind the surface, where a surface term is expected to fall by more than 0.005, does not: +0.002 overall and +0.009 on the dynamic objects. The prior draws in what floated in front of the surface, and the far shares fall by a third (0.035 to 0.022; beyond 25 cm 0.023 to 0.013).
- **`studio`, one seed:** in front falls from 0.014 to 0.005, behind rises from 0.414 to 0.423 (beyond the 0.005 guard, the wrong way), and the expected depth's error rises from 172 to 189 mm while the far shares fall. PSNR is 0.34 dB higher.

**The reading by section 7.2's row for finalist 4.**

- **Adopt when** "it moves the named measure beyond finalist 3's two arms, with the renders sharp and no harm on openftgs-synth". **Not met.**
  - The named measure is lower than `FLAT`'s on the left copy and than `FLATNC`'s on the centre copy, never beyond both on one copy.
  - The renders turn softer by 0.035.
  - `studio`'s share behind the surface rises beyond its guard on its one seed.
- **Drop when** "it adds nothing to finalist 3's flattening". **Not met in full.** Against `FLAT` it lowers the left copy's named measure by 0.016 by the rule, and on `kitchen` it halves the expected depth's error and the share in front where flattening alone moved neither. It adds something, and not what the adopt clause asks.
- **The row's third column,** "if its measure moves and the renders turn soft: H2's prediction for it, the loss settling between the cameras' versions of the person: the combined arm". **This is the outcome:** a measure that moves in part, with renders that turn soft.
  - The first step found the cameras' normals of the person no further apart than an estimator's on exact cameras. So the softening is not shown to come from the cameras' versions; the prior's own spread and its size beside the rendering loss (0.39 times the rest at the end) explain it as well.
  - On `kitchen`, with exact cameras, no image score was harmed, but the sharpness ratio is measured on private scene A only.
- **DAViD's arm** does less on the shell (one move, against `FLATNC` on the centre copy), softens less (0.022 to 0.031), and costs 0.03 to 0.28 dB of test-view PSNR. Made for people and given the person only, it is not better here than MoGe-2 over the whole image.
- **What this test does not cover:**
  - weights below the papers' (λ₇ = 0.1, λ₈ = 0.015) that would keep the renders sharp;
  - the start before iteration 15,000;
  - C7's total-variation term;
  - the combined arm, with finalist 2's term;
  - more than one seed on `studio`.
- **Recommended: not adopted alone, and not dropped.** The row's third column sends it to the combined arm, but that arm has a partner only if a term on the disagreement side exists. Finalist 2 was dropped as tested, and its variants are ticket #26c's to report. So finalist 4 waits:
  - for such a term, should one come out of ticket #26c or a later step;
  - or for a further arm of its own at a lower, calibrated weight.
  Dropping it outright would leave out the one surface effect any finalist has shown on `kitchen`, the halved depth error.

**Against the ticket's acceptance criteria.**

- **The normals' agreement on the person, against `kitchen`'s floor, and the axes check, before the arm:** done (the first step, above).
- **The weights' terms and each package's license, checked and logged before use:** rows 882 to 885.
- **The arm on both sets without the total-variation term, read against finalist 3's two arms:** 8 runs, judged against the off arm, `FLAT` and `FLATNC` (above).
- **Section 4.4's criteria, as for finalist 3:**
  - the named measure by the rule, with the tail by its margin;
  - the 0.3 dB guard, not crossed;
  - the sharpness ratio, lower;
  - the sheets, keyed and awaiting the owner's judgement;
  - on openftgs-synth, the share behind and the guards (above).
- **DAViD's arm, with the pixels without a prior left out:** 4 runs, on the person by section 2.7's rule.
- **The reading by section 7.2's row:** above; the owner's answer below, not yet given.
- **Sources, commands, files changed, privacy:**
  - sources: rows 882 to 889;
  - commands: in the tables of this section;
  - files changed: only this document and the source log;
  - privacy: nothing names private scene A or shows its images, and its images went only to the two estimators, offline on this machine.

**The questions for the owner at step 6.**

1. **Finalist 4:** not adopted alone and not dropped (recommended), waiting for a disagreement-side term to combine with; or dropped; or given a further arm at a lower, calibrated weight (about 1 GPU hour for a calibration on the centre copy, then about 4 for the arm on both sets)?
2. **Step 7, the recommendation (ticket #29):** go, or stop.
3. **The sheets** (`<surface-runs>/ticket-28/sheets/blind`, key apart): the owner's judgement with the labels hidden; and the labelled `kitchen` side-by-sides.

**The arms' commands** (the conventions of the first step's table, with 14 threads, `$I` for `<surface-runs>/ticket-24/init`, `$O12` for `<surface-runs>/ticket-12/private-scene-A`, `$KS` for `<scratch>/kitchen-sheets`):

| Command | Device | Time | Note |
| --- | --- | --- | --- |
| `python -I $T/normal_maps.py $A $O/private-scene-A/david_full --estimator david --weights … --device cpu --threads 4` | CPU | 25 min 18 s | DAViD on every image of private scene A |
| `python $T/person_masks.py $A $O/private-scene-A/david_full $O/private-scene-A/david_person --da2 $O12/da2s/maps --grids $R4/d22 --crops $R4/crops.json --placeholder $R4/placeholder_masks.npz` | CPU, 4 threads | 3 min 33 s | DAViD's maps restricted to the person by section 2.7's rule |
| `bash $T/maps_full.sh`: `python -I $T/normal_maps.py <scene.json> $O/<scene>/moge2_full --estimator moge2 …` for private scene A, `kitchen` and `studio`, each under the lock | GPU | 16 min in all | MoGe-2 on every image |
| `bash $T/arms.sh both`: per run `bash $T/run_train.sh <run> <scene.json> <init> <seed> --term flat=1 --flat-weighting sigma_t --term prior=1 --prior-c7 0.1 --prior-c8 0.015 --prior-gamma 0.005 --prior-depth plane --prior-from 15000 --prior-maps <maps> --prior-axes <axes>`, each followed by `bash $T/post_a.sh <run> h<c>` or `bash $T/post_synth.sh <run> <scene> <scene.json> <init>` (section 6 here's, repointed) | GPU, then CPU | 23 to 36 min a run; under 2 min a post | The twelve runs; the memory fallbacks were never taken |
| `FTGS_WORKTREE=<worktree> bash $KS/kitchen_sheets.sh <surface-runs>/ticket-28/runs/PRIOR_kitchen_s<n> PRIOR_kitchen_s<n>`, then `python $KS/compare_sheets.py <surface-runs>/kitchen-sheets/compare/finalist4_kitchen_s<n>.jpg --title … PRIOR_… FLAT_… OFF_… PLAIN_…` | GPU, then CPU | about 7 s each | The labelled `kitchen` sheets and side-by-sides |
| `bash $T/blind.sh` | CPU | seconds | The twenty sheets with their labels hidden, and their key |
| `python $T/person_masks.py … --use cam1=cam1-cam4,cam4=cam3-cam4` into `$O/checks/david_person_alt` | CPU, 4 threads | 46 s | The held-out cameras' effect on the masks, sized |
| `python $T/tables28.py` and short inline reads of `ratios.json`, `train_log.jsonl` and the sheets' coverage | CPU | seconds | The tables of this section |

Tickets #4's, #12's, #24's, #26's, #26b's and #27's folders, `<m15-runs>`, `<m17-runs>` and the private scene's folders were read and never written.

**The owner's answer on step 6** (whether the work goes on to the recommendation, or stops): *not yet given.* **Date:** *not yet given.*

## 8. The recommendation

*Ticket #29's section.* Per finalist that ran: adopt, drop, or test in combination; a finalist the owner stopped before its step says so. Where it recommends adopting, the spec change's wording proposed for the owner, on an issue of its own.

## 9. The sources

*Ticket #29's section.* The clean-room record in full, completing the opening's: the source-log rows of every ticket of the PRD, what was read only as documentation, what was not opened, and any incident.
