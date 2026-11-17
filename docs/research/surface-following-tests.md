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

### 2.2 Check (c): one camera fit per held-out copy, on the background

*Ticket #19's section.* One correction per camera, fitted on the background once per held-out copy without that copy's test-view images, then sections 2.5's and 2.7's measures again on held-out frames, stated in section 5.3's two cases.

**What was measured, and with what.** One correction per camera of private scene A, fitted on the background's matches, then sections 2.5's and 2.7's measures again with the corrected cameras, on 2026-11-16, by uncommitted scratch scripts under `<scratch>/ticket-19` ("Runs", at the end of this section, lists every command), with the worktree's package (`src/` as on `main` at 02e3686), Python 3.14.10, torch 2.15.0+cu132, NumPy 2.5.4 and SciPy 1.18.1, within four CPU threads, and the RTX 4090 for six minutes of matching under the shared lock; the dense matcher's packages are the scratch folder of the research document's row 612. Results are in `<surface-runs>/ticket-19`. The scripts read private scene A's matches and manifests and, for the matching alone, its images, and wrote numbers and files; no image was looked at. Camera names are the manifests', as in section 2.5: `cam0` is the centre camera and `cam1` the one left of it.

**The two copies, and the pairs the fit reads.** The *centre copy* holds `cam0` out and the *left copy* `cam1`: each is private scene A's manifest with that camera's 120 images in the test split, the same nine cameras bit for bit and the same images file for file (rows T19.1 and T19.2). Ticket #4's saved matches of D-22's seven pairs therefore serve both copies (row T19.3). D-22's pairing, run by the package on each manifest:

| Manifest | D-22's pairs |
| --- | --- |
| private scene A, nine training cameras | `cam0`–`cam1`, `cam1`–`cam4`, `cam2`–`cam5`, `cam3`–`cam4`, `cam3`–`cam5`, `cam6`–`cam7`, `cam6`–`cam8` |
| the centre copy | `cam1`–`cam4`, `cam2`–`cam5`, `cam3`–`cam4`, `cam3`–`cam5`, `cam6`–`cam7`, `cam6`–`cam8` |
| the left copy | `cam0`–`cam2`, `cam2`–`cam5`, `cam3`–`cam4`, `cam3`–`cam5`, `cam6`–`cam7`, `cam6`–`cam8` |

- **The rig's angles tie.** `cam0` is 15.0° from `cam1`, `cam2`, `cam3` and `cam6`; `cam1` from `cam0`, `cam4` and `cam7`; `cam2` from `cam0`, `cam5` and `cam8` (row 610). D-22 picks among them by the float32 values' last bits, so a turn of a tenth of a degree picks again ("The corrected manifests", below). A first pass fitted on D-22's own pairs only, and on the left copy the corrected manifest paired `cam0` with `cam3`, a pair that fit had not read.
- **So the fit reads D-22's pairs with their ties kept:** each training camera with every camera within 0.01° of its closest. On the centre copy 8 pairs (`cam1`–`cam4`, `cam1`–`cam7`, `cam2`–`cam5`, `cam2`–`cam8`, `cam3`–`cam4`, `cam3`–`cam5`, `cam6`–`cam7`, `cam6`–`cam8`), on the left copy 9 (`cam0` with `cam2`, `cam3` and `cam6`; `cam2`–`cam5`, `cam2`–`cam8`, `cam3`–`cam4`, `cam3`–`cam5`, `cam6`–`cam7`, `cam6`–`cam8`), for the nine as one set all 12. Five of them were not ticket #4's and were matched for this section, all 120 frames each: `cam0`–`cam2`, `cam0`–`cam3`, `cam0`–`cam6`, `cam1`–`cam7` and `cam2`–`cam8`, by ticket #12's `match_pair.py` with the package's matcher and settings (D-58, Q-106; row T19.2).

**The fit.**

- **What it changes.** Each camera is turned about its own centre by three numbers, R' = exp([w]×) R with w in the camera's frame. Centres and K are kept; a variant that also moves the centres is given apart ("The variant that moves the centres", below).
- **What it reads:** the background alone, the far matches of section 2.5's split by depth (the rays' closest approach 2.2 or more along camera A's axis, with the manifest's cameras), on the even frames.
- **Two terms, both in pixels.**
  - Each pair's Sampson error (section 2.5), on 1,500 far matches per pair and frame.
  - For each camera with two partners, section 2.7's agreement in depth: its grid points admitted in both pairs, each triangulated with each partner (closest approach), and the two inverse depths along its axis compared, times f·b (789.2 px × 0.274 units). That is how far, in pixels along the epipolar line in the second partner's image, the match lies from where the first partner's depth puts it: the direction section 2.5's residual does not read. 500 points per camera, pair of partners and frame.
  - Both through Cauchy's loss at a scale of 0.5 px, the one section 2.5's turn used, written into the residuals so that the two terms below stay quadratic.
- **The rig's common turn is held at zero.** Turning every camera by one rotation, centres kept, leaves the matches of a distant point almost as consistent as before, so the background barely reads it: the three smallest singular values of the Jacobian of each fit of several cameras, 34 to 121 px per degree against 181 to 7,300 for all the others, are that common turn. The mean turn of the cameras fitted together is held at zero (1,000 px per degree), which keeps the manifest's orientation of the rig as a whole. Without that hold, on the centre copy's D-22 pairs, every camera turned 2.2° to 5.0°, mostly about its vertical axis, for about the same background residual on the fit's own matches (0.28 to 0.55 px at the median by pair, against 0.24 to 0.59 px with the hold). A damping of 1 px per degree on each component keeps any other direction the matches leave free at the manifest's value.
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
- **Why the person's does not follow: the turn reaches b and not a.** Section 2.7 found one fixed relation per camera, 1/z₂ = a/z₁ + b, that takes the person's 4.0% to 1.0% to 1.6%. Fitted again over the odd frames, its scale a does not move with any fit (`cam3` 1.077, then 1.076 to 1.082; `cam6` 1.099, then 1.092 to 1.107; `cam4` 0.971, then 0.968 to 0.977), its shift b does, and it leaves 1.0% to 1.8% on the person after every fit as before. A turn along the epipolar lines adds one amount to every inverse depth, which is b; a scale of every depth, a, needs a baseline or a focal length that is not the manifest's (section 2.7). The turns took out part of b, which on the background is most of the error; on the person the scale part, 8% to 10% for `cam3` and `cam6`, is what stays.
- **Pairs of partners beyond section 2.7's five** tell the same. Of the 17 more that the five other pairs make, the person's figure falls by more than half a point on both copies in 6 (`cam3` with `cam0` and `cam4` from 5.48% to 1.78% and 3.88%; `cam1` with `cam0` and `cam7` from 5.55% to 3.30% and 3.71%), rises as much in 4 (`cam0` with `cam2` and `cam3` from 2.27% to 3.96% and 3.03%) and stays in 7; the background's falls in 13 and rises in none.


**The variant that moves the centres.** A turn reaches section 2.7's shift b and not its scale a, which needs a baseline or a focal length. So the same fit was run again with each camera's centre free as well, three more numbers damped at 1 px per cm, the rig's common shift and its size (the mean distance of the centres from their centroid) held as its common turn is, on 600 and 250 points per pair or camera and frame. A first try that held the shift and not the size grew the rig 2.4 times, which shrinks every inverse depth and the depth term with it, and was stopped. The variant is evidence on what a centre move can reach, not a manifest proposed for training: it moves S.

- **The centres move** up to 2.4 cm (`cam4` and `cam8` on both copies; the held-out cameras 0.2 cm and 1.5 cm), and the turns stay 0.08° to 0.42°.
- **The background falls a little further:** 0.38 and 0.40 px over the seven pairs at the median, 1.33 and 1.40 px at the 90th percentile, against 0.43 and 0.55 px with turns alone.
- **The person's residual rises:** 3.23 and 5.65 px over the seven pairs at the median, 12.0 and 14.3 px at the 90th percentile, against 2.37 and 7.64 px before. A centre moved to suit the background is wrong for the person, whose matches it moves more since they are nearer (section 2.5, "What the turn does and does not show").
- **The person's 4.0% between pairs** is 3.58% and 3.09%. The scale a now moves, but not towards 1 in every camera: `cam3`'s from 1.077 to 0.967 and 1.005, so its person falls to 3.71% and 1.77%, while `cam4`'s goes from 0.971 to 1.109 and 1.089, and its person rises to 7.42% and 5.64%.
- **D-22's pairs flip on these too:** `cam1`–`cam7` on the centre copy, `cam0`–`cam6` on the left copy.

So a centre move fitted on the background does not reach the person's offset either, and it costs the person's epipolar residual and S.

**Centres and S (D-27).** The turns move no centre: read back from the corrected manifests, the largest change of a centre is 2.6·10⁻⁸ units, float32's rounding, and S by the package's `spatial_scale` (factor 1) is 0.378230496 before and 0.378230499 after on the centre copy, 0.402766772 and 0.402766771 on the left copy. E13's position rates and growth's split threshold of 0.01·S are unchanged. The variant that moves the centres changes S by +1.2% on the centre copy (0.378230 to 0.382828) and +0.37% on the left copy (0.402767 to 0.404241), and E13's rates and the split threshold with it; it stays in `<surface-runs>/ticket-19/manifests/V2`, not proposed for training.

**The corrected manifests.** In `<surface-runs>/ticket-19/manifests/P/h0` (the centre copy) and `P/h1` (the left copy), each `scene.json` beside a link `images` to its copy's images, not committed:

- **What they are:** the copy's manifest unchanged, records, splits, paths and metadata, except every camera's `world_to_camera`, turned as above, and a `metadata.camera_correction` that records the tool (this section's scratch fit, a prototype), the copy's SHA-256, both steps' settings, and per camera its turn, its centre's shift (none) and its `world_to_camera` before. The test view's two poses, the manifest's and the corrected one, are both in the file (section 5.3's measures row).
- **Valid `ftgs-scene` v1 files:** the package's `read_scene_manifest` and `MultiViewVideo.from_manifest` read each without error, 1,080 records, 120 of them the test view's, and load one training and one test image through the link.
- **D-22's pairs change on both,** since the turns break the 15.0° ties (above): on the centre copy `cam1`–`cam4` and `cam2`–`cam5` become `cam1`–`cam7` and `cam2`–`cam8`; on the left copy `cam0`–`cam2` and `cam2`–`cam5` become `cam0`–`cam6` and `cam2`–`cam8`. The fit read all four new pairs. But an init on a corrected manifest would take `cam2`'s person from `cam2`–`cam8`, where D-22's filters keep 1% to 2% of the person's sample against 80% to 83% in `cam2`–`cam5`, and where the person disagrees by 6.3 to 7.9 px at the median before and after; `cam1`–`cam7` keeps more of it than `cam1`–`cam4` (64% against 43% on the centre copy) and `cam0`–`cam6` a little less than `cam0`–`cam2` (82% against 89% on the left copy). So a run on a corrected manifest would differ from the baseline's in its init's pairs as well as in its cameras; whether finalist 1's inits keep the manifest's pairs is put to the owner (step 1's review, section 2.6 here).
- **The nine as one set** has its manifest in `P/all9-never-trains`, for these measures only. The first pass's manifests, on D-22's own pairs, are set aside in `first-pass-not-for-training`.

**The outcome: section 5.3's second case.** The background's residual falls, over the seven pairs from 0.87 to 0.43 to 0.55 px, though not in every pair; the person's epipolar residual does not (2.37 to 2.29 to 2.35 px), and its 4.0% between pairs falls to 3.0% on one copy and 3.8% on the other, not towards the 1.0% to 1.6% that section 2.7's fitted relation leaves, which the turns leave in place. So finalist 1, with turns fitted on the background, tests the background's part of the disagreement alone, and little change is expected on the person (section 5.3, "The route, and the first step"). The variant that also moves the centres gives the same case, with the person's epipolar residual higher and S moved. The nine as one set does not reconcile the pairs either: it gives about the copies' figures (0.47 px and 2.30 px over the seven pairs, 3.19% on the person's depth), with `cam3`'s background still 5.9% and `cam6`'s person 6.3%.

**Runs.** `python` is the shared `.venv`'s, with `PYTHONPATH=<worktree>/src:<worktree>/harness/src` and `OMP_NUM_THREADS`, `MKL_NUM_THREADS` and `OPENBLAS_NUM_THREADS` at the thread count given, and `CUDA_VISIBLE_DEVICES` empty except for the matcher. The matcher's runs add `<worktree>/scripts` and the research document's `.scratch/surface-following-primitives/ticket-4/env-init` to `PYTHONPATH`, with `PYTHONDONTWRITEBYTECODE=1`, and hold the shared GPU lock. `$S` is `<scratch>/ticket-19`, `$O` is `<surface-runs>/ticket-19`, `$O4` is `<surface-runs>/ticket-4/private-scene-A`, `$G` is the grid folders `$O4/d22` and `$O/matches/<pair>` for the five pairs matched here, and `<copy>` is a held-out copy's folder, `<private-scene-A-exp>/scene_h0` or `scene_h1`. `run_fits2.sh`, `run_measure.sh` and `run_check.sh` chain the commands below with the pairs and pairs of partners given above.

| Command | Threads; device | What it ran; time | Note |
| --- | --- | --- | --- |
| `python -I $S/same_images.py <private-scene-A>/scene.json <private-scene-A-exp>/scene_h0/scene.json <private-scene-A-exp>/scene_h1/scene.json` | 4; CPU | 3,240 files; 1 s | the copies' images are the scene's |
| `python $S/tie_pairs.py <manifest>`, on the scene and the two copies | 1; CPU | seconds | D-22's pairs with their ties kept |
| `flock <scratch>/gpu.lock python $S/match_pair.py <private-scene-A>/scene.json $O/matches --label private-scene-A --tag cam0-cam2 --pair cam0:cam2 --device cuda:0`, and the same for `cam0:cam3`, `cam0:cam6`, `cam1:cam7` and `cam2:cam8` | 4; GPU | 120 evaluations each; 66 to 71 s each | ticket #12's script, unchanged: the five pairs' matches, in ticket #4's grid format |
| `python $S/measure.py <private-scene-A>/scene.json <private-scene-A>/scene.json $O/measures/manifest-all.json --grids $G --crops $O4/crops.json --frames all --pairs <D-22's 7> --triples <section 2.7's five> --d22-sample 2000` | 4 processes of 1 thread; CPU | 840 pair-frames and 600 images; 2 min 6 s | the reproduction of sections 2.5's and 2.7's tables |
| `python $S/fit.py <copy>/scene.json $O/fits/first-pass/h0-train-component.json --grids $G --pairs <D-22's pairs of the copy> --triples <their pairs of partners> --free <its training cameras>`, then the held-out camera's step, for both copies and the nine; and the centre copy's first step again with `--gauge none` | 4; CPU | 12 to 73 s each | the first pass, set aside: it showed the pairs' flip and the common turn |
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
