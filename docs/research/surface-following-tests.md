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

### 2.3 Checks (d) and (e): two seeds' disagreement where the shell is, and colour between cameras

*Ticket #20's section.* `A_default_s0` against `A_default_s1` where the shell is; and the colour difference at the two ends of section 2.5's admitted matches, per camera pair, against `kitchen`'s floor.

**What was measured, and with what.** On 2026-11-16, by uncommitted scratch scripts under `<scratch>/ticket-20` ("Runs", at the end of this section, lists every command), with the worktree's package at the integration branch's 9bd213c (`src/` as `main`'s 02e3686), Python 3.14.10, torch 2.15.0+cu132, gsplat 1.5.3, NumPy 2.5.4, SciPy 1.18.1, Pillow 13.0.0 and safetensors 0.8.0, within four CPU threads, and the RTX 4090 through the shared lock for the GPU runs. Results are in `<surface-runs>/ticket-20`. The scripts read private scene A's checkpoints, init points, images and renders and wrote numbers; no image, render or sheet was looked at. Camera names are the manifests', as in section 2.5.

**Check (d): two seeds' disagreement where the shell is.**

**The two models.** `A_default_s0` and `A_default_s1` (section 2.6): the default configuration from the stride-1 init points file both trained from, with the centre camera `cam0` as the test view, 30,000 iterations and 3,000,000 primitives each, differing by `seed` alone (row T20.3).

**The populations,** as section 2.6 defines them, taken in each model by its own primitives:

- the **near side**, positions nearer than 2.0 along `cam0`'s axis, and the **far side**, the others;
- **far from init**: positions more than 0.05 from every point of the init points file, all frames pooled;
- **under two frames**: durations under 2/24 s.

The near side's two populations overlap, so the near side is also split into its four cells, and **the rest** is the near side's primitives in neither population: near an init point and not under two frames. The script first reproduces section 2.6's figures for both checkpoints, to every digit given there: the far-5 cm share of opacity 0.259 and 0.257, the under-two-frames share 0.517 and 0.519, the near side's share of opacity 0.749 and 0.755, the far-5 cm share within it 0.128 and 0.131, and the far-from-init share of its primitives 0.140 and 0.144.

**The measure.** For every primitive of one model, by its position μx:

- **cross**, its distance to the other model's nearest primitive;
- **own**, its distance to its own model's nearest other primitive, the spacing of its neighbourhood;
- their ratio, cross / own, primitive by primitive. Raw centimetres can't be compared between populations, since the primitives far from init are sparse and those near it dense. The ratio can: it is about 1 when the other seed has a primitive about as near as the model's own neighbour, and it grows when the other seed has put none there.

Both directions are taken, `s0` against `s1` and `s1` against `s0`, and they agree closely. A variant restricts both searches to primitives whose time centres lie within 6 frames of each other (the same frame bin or within 6 bins, so 6 to 7 frames), since the person moves about 0.5 over the clip (section 2.6); it gives the same ordering, with larger ratios. Every distance is also given weighted by base opacity, which moves no median by more than 0.3 cm.

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
- **The primitives under two frames are placed alike, by the rule.** Their median ratio is 1.52 and 1.53, 20 % above the rest's, and their share with cross under own is 0.288 and 0.286, 0.074 below it; neither margin is reached. Their cell near init, 93 % of them, reads 1.46 and 1.47. At 5 cm their overlap is 0.83, against the rest's 0.87.
- **The far side is placed differently too.** Its ratio is 2.05 and 2.01, its share with cross under own 0.22, its overlap at 5 cm 0.37, against 0.62 for a random split. It holds the background, where init points are thin (section 2.6). So the seeds' disagreement does not mark the shell alone: it marks every region that the init points and the cameras pin down loosely. On the near side, the shell's far-from-init population is the only one it singles out.
- **Within ±6 frames** the ordering holds: 6.14 and 6.17 for the far-from-init primitives, 1.74 and 1.76 under two frames, 1.47 for the rest, 4.65 and 4.52 on the far side.

**The renders.** Both checkpoints were rendered from the sheet's request, `<surface-runs>/ticket-5/side_request.json` (SHA-256 `1b88fd859d64028c9ff40a4624a6bce76edb0be95b8f4129ccfbdc5933d98821`, checked): 39 items, the 13 poses at frames 0, 60 and 119 (section 2.6, "The sheet"). On each item the two renders are clamped to [0, 1] and compared in 8-bit levels, on the pixels where both alphas are at least 0.5. A tile's pixels are also sorted by where the shell is in it. Each model's near-side primitives are projected into the item's camera, as section 2.6's framing check did (the position by E1 at the item's time, the weight σ_eff by E3 and E4). Their weight is summed in 16 px cells, as is the weight of the far-from-init ones among them. A **shell cell** is one where the far-from-init primitives hold at least a quarter of the near-side weight, in the mean of the two models; a **rest cell**, one where they hold under 5 %. No sheet was made, and the renders were looked at by no one (row T20.5).

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
- **So the renders do not show the shell** apart from the rest. The seeds' difference in an image is a diffuse growth away from the training cameras, not a place in the picture; the primitives say more than the renders do.

**The reading** (section 5.1, row (d)). The two seeds place the shell's primitives that lie far from every init point differently, and the near side's rest alike, by the rule fixed beforehand. That is the row's first case: two models' disagreement is a signal for that part of the person, and rows E11 (RIGI's two-model uncertainty) and E22 (CoR-GS' co-pruning and co-regularization) come back beside finalist 2. Four limits go with it:

- **It sees one of the shell's two populations.** The primitives under two frames, which make up most of the near side (0.47 of the model), are placed as alike as the rest, so the signal cannot see that population.
- **It marks the region, not a misplacement of it.** Both seeds build a shell in the same place: they disagree on its arrangement at a few millimetres to a centimetre or two, while its primitives lie some 15 cm from the init points. A term that penalized where two models disagree would push on the shell's fine arrangement. That it would remove the shell is not shown.
- **It is not the shell's alone.** The far side is placed as differently, so on its own the signal would also act on the background, where the init points are thin.
- **The renders do not localize it.** Rows E11 and E22 act on renders, and from the sheet's request the seeds' renders differ about as much in the shell's cells as elsewhere. A signal taken from renders would be weaker than the one taken from the primitives here.

No finalist's place changes; rows E11 and E22 come back as companions to test after finalist 2, each still with a second model's cost (section 4.8).

**Check (e): colour between the cameras.**

**The matches.** On private scene A, section 2.5's own matches: the grids its run saved at offset 0, D-22's 7 pairs and all 120 frames, read again without matching anything (row T20.1). On `kitchen`, which gives the floor, ticket #4 saved no grids, so its matches were made again with the same script and settings, unchanged: D-22's 13 pairs and the six pairs about 15° apart of section 2.5's floor, every tenth frame, 570 evaluations (row T20.4). The rerun reproduces section 2.5's residuals: a median of the per-evaluation static medians of 0.027 px over the 13 pairs and 0.022 px over the six.

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
| `python d_null.py <A_s0> <A_s1> <init> <private-scene-A>/scene.json --out $O/d_null.json` | 4; CPU | 1 min 27 s | The same measures within each population, for the two seeds and for one random split of their pooled primitives (seed 20) |
| `ftgs render <A_s0> --request <surface-runs>/ticket-5/side_request.json --out $O/renders/A_default_s0.safetensors`, and the same for `<A_s1>` | GPU | 39 renders each; 8.3 s and 8.4 s | `ftgs` is `python -m openftgs.cli`. The card held 1,293 MiB of other work when they ended |
| `python d_render.py $O/renders/A_default_s0.safetensors $O/renders/A_default_s1.safetensors <surface-runs>/ticket-5/side_request.json <private-scene-A>/scene.json <A_s0> <A_s1> <init> --out $O/d_render.json` | 2; CPU | 39 items; 26 s | The render differences, against `cam0`'s images on `sC00`, and by shell and rest cells |
| `python match_residuals.py <m15-runs>/kitchen/scene.json $O/kitchen --label kitchen --tag d22 --gt $G --frames 0:300:10 --save-grids --device cuda:0` | 4; GPU | 390 evaluations; 4 min 10 s | `kitchen`'s matches made again for check (e), on D-22's 13 pairs; `$G` is `$OPENFTGS_DATASETS/synth/ground_truth/kitchen`; grids of 2.8 GB |
| `python match_residuals.py <m15-runs>/kitchen/scene.json $O/kitchen --label kitchen --tag wide --gt $G --pairs cam06:cam10,cam05:cam09,cam03:cam07,cam08:cam12,cam12:cam16,cam02:cam06 --frames 0:300:10 --save-grids --device cuda:0` | 4; GPU | 180 evaluations; 1 min 56 s | The six pairs about 15° apart; grids of 1.4 GB |
| `python e_colour.py --selftest` | 1; CPU | under 1 s | The colour script's checks on synthetic data |
| `python e_colour.py <private-scene-A>/scene.json <surface-runs>/ticket-4/private-scene-A/d22 $O/e_colour_A.json --mode depth --workers 2` | 2 processes; CPU | 840 evaluations' grids; 2 min 59 s | Private scene A's colours at the saved matches |
| `python e_colour.py <m15-runs>/kitchen/scene.json $O/kitchen/d22 $O/e_colour_kitchen_d22.json --mode gt --gt $G --workers 4`; the same for `$O/kitchen/wide` | 4 processes; CPU | 3 min 5 s and 1 min 17 s | The floor; `PYTHONPATH` adds `<worktree>/scripts` for `gt_depth.py`'s frame reader |

The GPU ran for about 6 minutes in all, 17 s of it for the two renders.

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
