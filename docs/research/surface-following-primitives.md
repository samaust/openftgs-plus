# Primitives that follow the surface, with image-model priors: survey, probes and recommendation

A research document, written from 2026-11-14 on PRD #1 of openftgs-plus and its tickets #2 to #15, ahead of the first spec revision that departs from the paper (README, "Direction"). On the owner's private scene the cameras' images disagree about the person at the centre, and training answers with a shell of faint, short-lived primitives off the body's surface (section 2.1). The document surveys techniques that make primitives lie on the surface they represent and turn with it, and the prior maps from pretrained image networks that guide them; it measures how far the private scene's cameras disagree; and it recommends which techniques to adopt, each as a field of the training configuration that defaults to off, or that none should be. It proposes no code and changes no spec item: adopting a technique is a spec change the owner makes, as the next lettered revision ("Editing the spec" in `docs/agents/guidelines.md`), followed by its own implementation issue.

Sections 1 to 9 are the nine the PRD fixes ("The phases", phase 4). The probes are sections 2.4 to 2.7 and the survey's families sections 3.1 to 3.6. The opening and sections 1 to 2.3 were written first (ticket #2). Every later section opens with a line naming the ticket that writes it, and a section that holds only that line, and the empty tables under it, is not written yet. A ticket writes between its section's heading and the blank line before the next heading, and changes neither, so that sections written in parallel merge without conflict.

**The owner's instructions and decisions** (PRD #1; the owner's comment on it of 2026-11-14, which replaces the body where they differ; two answers the owner gave in session on 2026-11-14, which the orchestrating session recorded in a second comment on the PRD the same day). Each is marked **confirmed** (the owner accepted what the plan or the implementer proposed), **answered** (the owner's own instruction, or an answer that replaced a proposal) or **default** (proposed by the plan of 2026-11-13, which the owner published unchanged, and not confirmed in words; it holds until the owner says otherwise on the PRD).

- Instructions given in session:
  - 2026-11-13: the private dataset's name appears in no commit. **Answered.** The PRD applies the same to issues, branch and file names, source-log rows and search queries, and uses the alias "private scene A". **Default**, the PRD's extension; decision 5 confirms the alias for this document.
  - 2026-11-14: every `gh` command names the repository, `-R samaust/openftgs-plus`. openftgs's issues are read with `-R samaust/openftgs` and never written (CLAUDE.md, "Issues"). **Answered.**
  - 2026-11-14: the generator behind private scene A is named nowhere. It isn't relevant to the research. **Answered.**
- The plan's decisions, by its numbers:
  1. **Scope.** The core is primitives shaped and oriented to the surface, and prior maps from pretrained image networks. **Default**; the owner's comment names neither the core nor a change to it. Its three edges:
     - Training that tolerates cameras whose images disagree is an adjacent family (family E). It may take up to half of the survey, not the quarter the plan proposed, since no camera of private scene A is trusted (decision 4). **Answered** (2026-11-14). Its cap is 120 abstracts and 24 full reads, twice another family's, which makes the overall cap 360 abstracts and 72 full reads and family E a third of it; the figures are the implementer's. **Confirmed** (2026-11-14, in session; the PRD's second comment of that day records it, where the first still said the figures awaited the owner's nod).
     - The subject is a person, so depth and normal estimators made for people belong to the core. Body templates (a parametric body fitted to the person) are recorded if met, not searched for. **Default.**
     - gsplat stays at 1.5.3. A technique that needs rasterization kernels gsplat 1.5.3 lacks is recorded but cannot be a finalist. **Default.**
  2. **Depth of the work.** The survey, the probes and a test plan; training tests only after review 1. `RenderOutput` returns no depth or normals (Q-226), so a finalist that needs them can't be trained through the package as it is. How each finalist is tried is decided at review 1: a scratch script outside the package that builds on Q-226's device, a throwaway uncommitted worktree, or waiting for its spec revision and testing behind its field. Until then H1 rests on evidence from papers. **Confirmed** (2026-11-14).
  3. **The bar for pretrained weights.** As openftgs issue 373 set it: the weights' own publisher states a permissive license that allows commercial use, checked per weights file. An estimator that fails is recorded and screened out. The survey also records whether a technique would work from prior maps the user supplies as files, which puts no estimator in the package. **Confirmed** (2026-11-14).
  4. **The source video.** No camera of private scene A is a real video. The centre camera is the closest to the source video, but it was regenerated like the other eight and shares their artifacts, sharpness and colour temperature. It also carries severe magenta artifacts: magenta is the colour the generator was meant to replace. So no camera is ground truth, and none is trusted above the others. **Answered** (2026-11-14, in session).
  5. **Private scene A's numbers** may appear in this document under the alias. Its name, paths and images never do. **Confirmed** (2026-11-14).
  6. **Citation search** uses keyless routes unless the owner supplies a Semantic Scholar or SciX key in an environment variable. **Default.**
- From the owner's comment of 2026-11-14 and the session:
  - The ticket breakdown: 14 tickets, the PRD's sub-issues (#2 to #15). The training tests are ticketed after review 1, each blocking the last ticket, or become a PRD of their own. **Confirmed.**
  - Temporary source-log IDs on a ticket's branch use the ticket's number, `T<ticket>.<k>`, as the implement skills do, not the PRD's. **Answered.**
  - This work's run directories, prior maps and sheets go in a folder outside the repository, which the owner allowed the orchestrating session to create; this document writes it `<surface-runs>`. **Confirmed** (2026-11-14, in session).

**Hypotheses.** The probes test H2's premise. H1 is tested only by training with a technique.

- **H1 (the owner's).** Constraining primitives to lie on the surface and turn with it, with depth or normal prior maps as supervision, removes the shell and keeps the body's shape in novel views from the side.
- **H2 (from the owner's notes).** The shell is how training reconciles cameras whose images disagree. A surface constraint alone would then trade the shell for blur, and depth estimated on each generated camera would inherit the disagreement. If H2 holds, the recommendation pairs a surface constraint with a way to absorb or down-weight the disagreement.

**Words.** **Private scene A** is the owner's private scene; its name, its paths, its generator and its images appear nowhere in this document. The **shell** is the failure of section 2.1: the faint, short-lived primitives that training leaves 5 to 25 cm off the body's surface. A **prior map** is a per-image map (depth, normals, a point map, flow, a mask) predicted by a pretrained network, and an **estimator** is such a network. The **bar** is the license condition on an estimator's weights (decision 3). A **placeholder pixel** is a pixel of the colour the generator was meant to replace (magenta) and didn't. A **finalist** is a technique shortlisted for a test. A **review** is one of the two review points with the owner, review 1 after the survey and review 2 on the document; the word checkpoint keeps its glossary meaning. A **family** is one of the survey's five groups of techniques, A to E (section 3), and a **kind** is one of the eight kinds of prior map (section 3, "The estimators tables"). A **search seed** is a method name a family's search starts from; a seed, alone, is a training run's. Other words are the glossary's (`docs/GLOSSARY.md`).

**Local paths** are written as placeholders: `$OPENFTGS_DATASETS` (openftgs-synth is at `$OPENFTGS_DATASETS/synth`); `<m17-runs>` and `<m15-runs>`, as `docs/prd/validation-runs.md` uses them; `<surface-runs>`, this work's folder outside the repository; `<scratch>`, the PRD's gitignored scratch directory, for scripts, environments and downloads ("Exploration notes" in `docs/agents/guidelines.md`); `<private-scene-A>`, private scene A's location, which is written nowhere. Prototype scripts are not committed: the section that used one records its commands, the versions and a one-line note on each run.

**Clean-room record.** Ticket #15 completes this paragraph, with section 9. What holds from the start: the rules are CLAUDE.md's guidelines rules, which replace spec §0 on sources for this fork, and the PRD's "Clean room for the search" (section 3 summarizes it for the survey). The sources consulted while planning are rows 491 to 502 of `docs/prd/source-log.md`. Every ticket logs a source when it opens it, under a temporary ID `T<ticket>.<k>` on its branch, numbered at the merge, and the document cites a source by its row, as `(row 499)`. The opening and sections 1 to 2.3 rest on this repository, the spec and this repository's issues, which need no row, and on the owner's notes as the PRD's Notes report them (row 499); ticket #2 opened no other source and added no row.

## 1. Scope and the owner's decisions

**The problem.** Private scene A has nine cameras around one person. Every camera's images were generated from one monocular video, so each camera shows a slightly different pose and the cameras disagree about the person at the centre (decision 4). Training answers with the shell, and the body breaks up in novel views from the side. Changing the training configuration recovered little (section 2.1).

**What the work does.** It surveys techniques that make primitives lie on the surface they represent and turn with it, and the prior maps (depth, normals, point maps, flow) that guide them. It measures how much private scene A's cameras disagree. It recommends which techniques to adopt, each as a field of the training configuration that defaults to off. It studies `openftgs.model`, `openftgs.render`, `openftgs.losses`, `openftgs.density`, `openftgs.init`, `openftgs.train` and `openftgs.data`, and changes none of them; section 2.2 lists the spec items studied. The deliverable is this document. No code or spec change lands from the PRD.

**The scope** is decision 1's: the core, and the three edges. Two consequences run through the document. A technique that needs rasterization kernels gsplat 1.5.3 lacks gets its row in a screening table and cannot be a finalist. An estimator that fails the bar gets its row in an estimators table and is screened out, and the survey records, per technique, whether prior maps the user supplies as files would do in its place (decision 3).

**No camera is ground truth** (decision 4). On private scene A a test-view score measures agreement with one generated camera, not accuracy. In the PRD's starting proposal for the training tests' criteria, which review 1 settles, scores there only guard against harm, and better means lower shell measures (section 2.6) and novel-view sheets the owner judges better. openftgs-synth, whose ground truth is exact (§1.4 "Validation data", D-79), is where a geometry residual and an error of expected depth can be measured (Q-226 to Q-231).

**The questions the research answers**, and where:

1. Which techniques make primitives follow a surface (scale, orientation, base opacity, depth), and what does each need from the primitive and from the render backends? Families A and B (sections 3.1 and 3.2) and the render-outputs probe (section 2.4).
2. Which prior maps supervise or initialize them, with which losses, and which estimators clear the bar? Family C (sections 3.3 and 3.4) and the probe on the agreement of prior maps (section 2.7).
3. How do dynamic-scene methods keep a surface coherent over time, and what would that ask of primitives whose orientation and scale are constant and whose motion is E1's? Family D (section 3.5).
4. Adjacent: how do methods train on cameras whose images disagree, none of them trusted, and some of whose pixels are known to be wrong? Family E (section 3.6) and the probe on private scene A's images (section 2.5).
5. For each shortlisted technique: can it be a field of the training configuration that defaults to off and leaves E1 to E17 and the §3 tests untouched? If not, what does the spec change? Column 3 of every screening table, then sections 4, 5 and 8.

**The phases**, and where each lands:

| Phase | What | Sections | Tickets |
| --- | --- | --- | --- |
| 0 | Setup and the probes; the render-outputs probe and the survey don't wait for the others | 2.4 to 2.7 | #3, #4, #5, #12 |
| 1 | The survey, families A to E; review 1 follows | 3.1 to 3.6, then 4 | #6 to #11, then #13 |
| 2 | Integration sketches and the test plan, written before anything trains | 5 | #14 |
| 3 | Tests of the finalists the owner cleared, by the route chosen for each; a finalist left to its spec revision is tested later, behind its field | 6 | ticketed after review 1, or a PRD of their own; #15 writes their results up |
| 4 | The recommendation, what adopting it would change, the sources; review 2 follows | 7 to 9 | #15 |

**The reviews.** Review 1, after the survey, puts to the owner the screening table, two to four finalists (at least one of them only a loss on today's primitives), the proposed criteria for training tests, and the route and GPU hours for trying each finalist. Nothing trains before the owner confirms. Review 2 is on the document and its recommendation.

**Machine and data.** The survey needs no GPU. Run directories, prior maps and sheets go outside the repository, in `<surface-runs>`. Anything a probe or a test needs goes into a scratch environment under `<scratch>`, never the shared `.venv`, after each package's and each weights file's license is read. The owner gives private scene A's location in session, to the tickets that measure it (#4, #5 and #12) and to no other; it is never written down, and nothing derived from the scene's images leaves the machine.

**Q-n rows the work relies on.** Q-7 (tests that download weights run only with `OPENFTGS_ALLOW_DOWNLOADS=1`; the research downloads weights itself, outside the suite, and logs each file's terms). Q-106 (`roma_init`'s `matcher` port and the `DenseMatcher` protocol, a warp and a certainty per match, which the disagreement probe reads). Q-226 (no depth output; section 2.3 says how expected depth is read without one). Q-227 to Q-231 (which pixels have a meaningful expected depth, the edge rule, δ = 0.02·S, the sampling at a primitive's centre, the figures reported; `scripts/gt_depth.py`). Q-233 to Q-237 (flagged pixels and kept-pixel PSNR; `scripts/reference_flaws.py` warps with ground-truth depth, so it applies to openftgs-synth and not to private scene A). Q-220 (M17's run parameters on openftgs-synth, reused if a finalist trains there).

**Out of scope.** Adopting a technique: the owner changes the spec as the next lettered revision, and an implementation issue then lands it behind its field. Any change to `src/`, `tests/`, the harness, §2.3's table or the package metadata. Changing the gsplat pin (§2.3's note). The paper's datasets (D-79). Recording the privacy rule in CLAUDE.md, which the owner may ask for on a `workflow-` branch.

## 2. The failure and what the spec fixes

### 2.1 The failure

From the owner's notes on private scene A (openftgs 0.1.0, 2026-11-10 and 11; local, outside the repository), as the PRD's Notes report them (row 499). Ticket #2 read the PRD, not the notes or the scene.

- **The scene.** Nine cameras around one person. Every camera's images were generated from a single monocular video by a general video generation model, so each camera shows a slightly different pose and none is a real video (decision 4). The centre camera, the closest to the source video, also carries severe magenta artifacts.
- **The shell.** Nothing floats in free space. The shell sits 5 to 25 cm from the body: median base opacity 0.16, and durations of about two frames as the notes measure them. In novel views it shows as dark blobs on the face, smeared arms and several superimposed faces.
- **The scores.** The centre camera's crop scores 28.1 dB PSNR when that camera trains and 24.1 dB when it is the test view. The best training configuration recovered about 0.3 dB of that gap. These scores measure agreement with a generated camera, not accuracy.
- **Noise between seeds:** ±0.01 dB PSNR on the centre test view, about 0.2 dB on the second test view.

**Already tried**, so the survey doesn't propose them again. A technique that only moves one of these values is recorded as tried, not screened (section 3.2).

| What was changed | Spec item, default | Values tried | Outcome in the notes |
| --- | --- | --- | --- |
| `loss.lambda_reg` | E9, 10⁻² [P §3.3] | 0.1, 0.3, 1.0 | mixed at 0.1, softer at 0.3, collapses at 1.0 |
| `reloc.dead_opacity` | D-14, 0.005 | 0.05, 0.1 | no gain at 0.05, collapses at 0.1 |
| relocation off, with `reloc.prune_opacity` | §1.3 "Count-control modes", the "w/o periodic relocation" ablation | not in the PRD's Notes | worse |
| `sh_degree` | D-3, 3 | 0, 1 | no gain, so the blobs are geometry, not colour |
| `growth.n_max` | D-44, 3,000,000 | not in the PRD's Notes | trades sharpness on the training cameras against the test view |
| `scene_scale_factor` | D-27, 1.1 | × 3 | no gain |
| `loss.perc_net` | D-9 and D-89, `flip` | `vgg16_oxford` | no gain |
| `optim.log_durations_lr` | D-20, 5·10⁻³ | × 0.2, × 4 | worse at × 0.2; + 0.17 dB but softer at × 4 |

One change helped: `ftgs init roma --stride 4` (D-21's stride, default 1), by 0.2 to 0.6 dB, mostly through the longer initial duration.

### 2.2 The spec items

The items the PRD lists as studied, in its order. None changes. The spec is `docs/spec/openftgs-spec.md`, revision 549o; a row says what the item fixes and leaves the wording to the spec.

| Item | What it fixes | Tag |
| --- | --- | --- |
| §1.1 | The eight quantities of a primitive, stored as nine tensors, and the per-time arrays derived from them (`means_t`, `opacities_t`, `quats_n`, `scales`, `sh`). Scale, orientation and SH do not change with time; only position and opacity do | [P §3.1]; each tensor's row carries its own tag |
| D-5 | The activations and the initial values: sigmoid opacity from 0.1, exponential scales from the 3-NN distance, the identity rotation, which its rationale calls irrelevant while scales are isotropic | decision; the activations and the 3-NN are [L: 3DGS §5.1] |
| D-25 | The neighbours of the 3-NN scale: the kept points of the same init frame, with the mean floored at 10⁻⁷ | decision |
| E1 | Motion: the moved position is linear in t, for every t | [P §3.1, Eq. 1] |
| E3 | Space–time opacity: the renderer receives σ_eff(t) = σt(t)·σ as the primitive's opacity | [P §3.1, Eq. 3] |
| E4 | Temporal opacity: an unnormalized Gaussian in t around the time centre, whose standard deviation is the duration | [P §3.1, Eq. 4] |
| E5 | The covariance R S Sᵀ Rᵀ, from the normalized quaternion and three positive scales | [P §3.1, text under Eq. 3; L: 3DGS Eq. 6; quaternion convention: gsplat docs] |
| E6 | Splatting and compositing as gsplat 1.5.3's classic mode does them: the EWA projection, the constants (ε, α_max, the 1/255 skip, the transmittance stop), front to back by camera-space depth. The equation's output is the colour C(p) over the background | [P Fig. 2 and §3.2 ("similar to 3DGS"); L: 3DGS Eqs. 3 and 5; constants: gsplat 1.5.3, D-6, D-31] |
| E7 | The rendering loss: L1, 1 − SSIM and the perceptual network, weighted 0.8, 0.2 and the network's calibrated weight, on the unclamped render against the training image | [P §3.2, Eq. 5; λ_img, λ_ssim P §3.3; the perceptual network and λ_perc D-9] |
| E8 | The 4D regularization: the mean over all N primitives of σ·sg[σt(t)], whose only gradient reaches the opacity logits | [P §3.2, Eq. 6; weight P §3.3 and Table 5; "early stage": P §1] |
| E9 | The total loss: L_render + λ_reg·L_reg(t), with λ_reg = 10⁻² at every iteration | [weights P §3.3; composition D-37; schedule D-10] |
| D-10 | L_reg is on for the whole run at a constant weight; `LossConfig.reg_until` ends it earlier | decision (owner, 2026-10-06) |
| §1.3 Growth | A growth event: the candidates (mean NDC gradient above τ_pos), the cap, clone or split at 0.01·S, the new rows' Adam moments and statistics; nothing is deleted and opacities are never reset | [D-18; L: 3DGS §5.2] |
| D-18 | The count grows by 3DGS clone and split between iterations 500 and 15,000, with relocation instead of deletion and the cap N_max; the count-control modes | decision (owner, 2026-10-06) |
| E17 | The split: two children drawn from the parent's Gaussian, with scales a/1.6 and the parent's other tensors | [L: 3DGS §5.2; adoption D-18, D-43] |
| D-43 | The children of a split or a clone copy the time centre and the duration: the split is spatial only | decision |
| §1.3 Relocation | A relocation event: the dead set (base opacity below 0.005, D-14) is moved onto live targets drawn in proportion to ρ; N is unchanged | [P §3.2, Eq. 7; §3.3] |
| E10 | The sampling score ρ = λ_g·ĝ + λ_o·σ, from the normalized accumulated gradient and the base opacity | [P §3.2, Eq. 7; weights P §3.3; normalization D-12, D-13] |
| E11 | The relocation move: the new opacity and scales of a target and its copies; the copies take the target's other tensors and positions drawn from its Gaussian | [L: 3DGS-MCMC Eq. 9; adoption D-15] |
| §1.3 Initialization, steps 1 to 9 | From init frame slots to primitives: camera pairs and dense matches, triangulation, position and time centre, velocity, duration, the budget, scales, then opacity, rotation and colour | [P §3.2, "Initialization of our representation"]; each step carries its own tags |
| D-21 | The stride (1) and the initial duration s₀ = stride·Δt_f | decision |
| D-22 | The camera pairs (each training camera with the one whose optical axis is closest, each pair matched once) and the filters: 5,000 matches, certainty ≥ 0.5, DLT, positive depth, ≤ 2 px, ≥ 1° | decision |
| D-58 | The dense matcher: roma_outdoor at RoMa's defaults, with the matches drawn by the package from the raw certainty | decision (owner, 2026-10-07) |
| D-66 | Initialization reads training-split images only | decision |
| E15 | The initial velocity, from a point's nearest neighbour in the next init frame | [P §3.2, "Initialization of our representation"; formula D-23] |
| D-23 | 1-NN to the next init frame, backward on the last, no outlier filter | decision |
| §2.1 `openftgs.render`: `render`, `RenderOutput` | `render()`'s signature, and the output's fields: `image` and `alpha`, with `means2d`, `visible` and `gaussian_ids` for the gradient statistics. No field holds depth or normals | signatures |
| D-75 | The gsplat backend calls gsplat's stage functions (`fully_fused_projection`, `spherical_harmonics`, `isect_tiles`, `isect_offset_encode`, `rasterize_to_pixels`), not `rasterization()`, and returns `visible` in both layouts | decision (owner, 2026-10-12) |
| D-76 | The gsplat backend's remaining semantics: the packed rows, CUDA float32 only, the empty render, the background composited outside the kernel, the camera centre without a matrix inverse | decision (owner, 2026-10-12) |
| §2.1 `openftgs.losses`: `total_loss`, `reg4d_loss` | `total_loss(pred, gt, model, t, it, cfg, *, perceptual)`, which returns the loss and its terms as detached tensors (D-77), and `reg4d_loss(model, t)` | signatures |
| `LossConfig` | The loss's fields: the four weights, `reg_until`, the SSIM settings and `perc_net` | signature; each field carries its tag ([P §3.3], D-8, D-9, D-10) |
| §2.1 `openftgs.init`: `roma_init`, `RomaInitConfig` | `roma_init(dataset, cfg)`, which returns init points, and its settings: the matcher's model and resolutions, the matches per pair, the certainty threshold, the stride, the budget, the seed | signatures; the fields are D-21, D-22, D-24, D-56, D-58 |
| `InitConfig` | `points_per_frame`, `init_opacity`, `scale_knn` and `scale_floor` | signature; D-24, D-5, [L: 3DGS §5.1], D-25 |
| `GrowthConfig` | The mode, the window, τ_pos, the split scale, N_max and the score mode's rate | signature; D-18, D-44, [L: 3DGS §5.2], [L: 3DGS-MCMC §3.6] |
| `RelocConfig` | The switch, the period and the window, λ_g and λ_o, the gradient quantile, the dead threshold, the moment reset, where the copies go, the ablation's pruning threshold | signature; [P §3.3], D-13 to D-17 |
| §2.2 `ftgs-scene` v1 | The scene manifest: the cameras, and per image a camera, a timestamp, a path and a split. A test image may carry `mask`; nothing carries depth, normals or another prior map, and no training image a mask | file format; the mask is D-64's and D-67's |
| §2.2 `ftgs-if` v1 | The exchange model: physical values in raw time units (position, time centre, duration, velocity, scale, rotation, opacity, SH) and the `render_conventions` block | file format |
| D-73 | `render_conventions`, and the adapter's `info` reply, record the Jacobian clamp margin, the screen-extent rule and the rasterize mode | decision (owner, 2026-10-12) |
| D-84 | An exchange model records the background it was trained against | decision (owner, 2026-10-16) |
| §2.3 | gsplat ==1.5.3 and the note on the pin (moving it requires re-running U-7 and U-8); the note that pretrained weights are downloaded at run time and never redistributed, their terms recorded apart from the code licenses | table and notes |
| "Add nothing outside that table without asking" | A dependency, vendored file, dataset or pretrained model outside §2.3's table needs the owner's approval. The PRD lists the sentence under §2.3; it is CLAUDE.md's ("Working from the spec"), about §2.3's table | owner rule (CLAUDE.md) |
| §2.4 | The four rules of every hot path, among them that an approximate optimization is off by default and that an iteration makes no host–device synchronization beyond gsplat's own, and "Where the time goes", the stages of an iteration and their cost drivers | design; D-49 |
| §3.4 B-1 | The method for per-iteration cost: time per stage from CUDA events, and peak memory, on openftgs-synth's `kitchen` | benchmark; D-79 |
| U-7 | Render conformance: the two render backends agree within 10⁻⁴ in RGB and alpha on 50 random scenes, and on four fixed cases | test |
| U-8 | Culling: the images are the same with and without the cull, and a culled primitive's render gradient is exactly zero | test; D-32 |
| U-12 | Gradient agreement: per stored tensor, gsplat against the reference on 20 random scenes, cosine ≥ 0.999 and relative L2 error ≤ 10⁻³ | test |
| §3.3's adapter contract | The adapter's six commands, with data crossing only in §2.2's formats and the two harness formats; a setting an adapter cannot honour is reported as not comparable | test design; D-73 for `info` |
| §1.4 "Validation data" | The paper's three datasets are non-commercial, so the project validates on openftgs-synth, whose ground truth is exact: cameras, per-frame times, undistorted frames, depth, dynamic masks and background images | [D-79] |
| D-79 | Which data validates the converters and the method: openftgs-synth, for the training runs and for B-1, B-7 and B-8 | decision (owner, 2026-10-15) |
| D-72 | A D-n decision changes through a field of the training configuration where §2.1 defines one, otherwise by a logged change, and never silently | decision (owner, 2026-10-12) |
| §0 | The clean-room rules, the provenance tags and who may change what. On sources, CLAUDE.md's guidelines rules replace §0 for this fork | owner rules (2026-10-07; CLAUDE.md, 2026-11-13) |

What the tags mean for a technique. The constant orientation and scale of §1.1, E1, E3, E4, E8 and the form of E7 and E10 are [P]: the paper fixes them, so a technique that changes one departs from the paper, which is the owner's spec change (README, "Direction"), made behind a field where possible (D-72). §2.1's `RenderOutput`, `render` and `LossConfig` are listed signatures: changing one is a spec change too, not a Q-n gap (CLAUDE.md, "Working from the spec"). The [L] and [D-n] items (E11, E17, the growth rules, steps 8 and 9 of the initialization) are the spec's own choices, which §0 lets change with a logged reason or through a field; a technique this document recommends still reaches them through the owner's spec revision, since the PRD treats adopting one as a spec change. Section 8 says, per recommended technique, which of these it would touch.

### 2.3 The code

What the package does today where the failure touches it, read in `src/openftgs/` and `scripts/gt_depth.py` at the integration branch's base (`main` at 93eb58e).

- **`openftgs.render.render`** returns a `RenderOutput` with `image` ([C, H, W, 3], composited over the background, not clamped to 1) and `alpha` ([C, H, W, 1], 1 − T_final). With `return_aux` it adds the 2D means, with their gradient retained for the gradient statistics, the visibility mask and, in packed mode, the row behind each 2D mean (D-75; Q-32, Q-36). It returns no depth, normal or distortion map, and takes no argument that asks for one. Both render backends hold each primitive's camera-space depth and use it only to order the primitives front to back: the gsplat backend passes the depths that `fully_fused_projection` returns to `isect_tiles`, and its `rasterize_to_pixels` call composites the three colour channels; the reference backend sorts by a detached depth and composites colours with the weights α·T (E6). Section 2.4 says what else those stage functions can composite.
- **Expected depth is read without an API change (Q-226).** `scripts/gt_depth.py` copies the model with `shN` dropped and SH degree 0, sets `sh0 = (z − 0.5)/C₀` per primitive, z being its centre's camera-space depth at the frame's time, and renders the copy through `render()` over a black background; expected depth is image ÷ alpha. This works because E2 clamps colours only at 0 and the render isn't clamped above 1. As written, the copy's tensors are detached and the render runs under `torch.no_grad()`: the script reads values, for a diagnostic, and no gradient passes through it. It keeps pixels with alpha ≥ 0.5 (Q-227) away from depth edges (Q-228), and reports the geometry residual with δ = 0.02·S (Q-229 to Q-231). Whether the device keeps gradients to the stored tensors, and whether it carries normals, are the render-outputs probe's questions (section 2.4).
- **`openftgs.losses.total_loss`** adds four terms: λ_img·L1, λ_ssim·(1 − SSIM), λ_perc·L_perc and λ_reg·L_reg(t) (E7, E9; Q-41, Q-249), each skipped when its weight is 0. The first three read only the render's `image` and the training image, over the whole image: `total_loss` takes no mask and no per-pixel weight, the trainer passes it no alpha, and the manifest reader refuses a `mask` on a training image (§2.2), so no pixel can be left out of the loss today. L_reg reads only the base opacities, the time centres and the durations (E8): its gradient, (1/N)·σt(t)·σ(1 − σ), reaches only the opacity logit, and lowers it the more the nearer the training image's time is to the primitive's time centre, whether or not the primitive is visible. No term reads depth, a normal, the scales or the orientation except through the rendered colours, and none compares two cameras: an iteration renders one training image (D-26), so the cameras meet only in the primitives they share.
- **Growth** (`openftgs.density.grow`, mode "gradient", the default). The candidates are the primitives whose accumulated gradient g exceeds τ_pos, and a candidate is cloned or split by its largest scale (§1.3 Growth; Q-52). Neither the base opacity nor the duration is read. g is a mean over the iterations in which the primitive was visible (E10), so the iterations in which a short-lived primitive is not visible do not dilute it. A clone copies all nine tensors; a split's two children take scales a/1.6 and positions drawn from the parent's Gaussian (E17). Both keep the parent's opacity logit, time centre and duration (D-43), so a faint, short-lived candidate becomes two faint, short-lived primitives. Nothing is deleted for low opacity or large size (D-42). The "score" mode appends rows drawn from the live set in proportion to ρ, by E11; "none" adds nothing.
- **Relocation** (`openftgs.density.relocate`). The dead set is the primitives whose stored opacity logit is below logit(`dead_opacity`), 0.005 by default (D-14; Q-50, Q-51). The test reads the base opacity alone: the duration and the effective opacity do not enter, so a primitive at the shell's median base opacity of 0.16 is live, under the default threshold and under both thresholds the notes tried. Each dead primitive takes a live target drawn in proportion to ρ = 0.5·ĝ + 0.5·σ (E10). E11 sets the new opacity and scales of the target and its copies; the moved copies take the target's time centre, duration and velocity, and positions drawn from its Gaussian (D-15). Relocation never removes a primitive. The ablation's `prune` (relocation off, `reloc.prune_opacity` set; Q-53) removes primitives below that base opacity and, after iteration 3,000, those larger than 0.1·S, with opacities reset to at most 0.01 every 3,000 iterations.
- **Initialization.** `roma_init` matches each camera pair of D-22 with the dense matcher (D-58; the `DenseMatcher` protocol of Q-106 gives a warp and a certainty per match), keeps matches of certainty ≥ 0.5 and draws up to 5,000 of them, triangulates them and keeps the points that pass D-22's filters, and returns init points: positions, colours, timestamps, velocities (E15), durations (stride·Δt_f, D-21) and each point's init frame (Q-111). Init points hold no orientation, normal or scale, and `ftgs-init` v1 has no key for one (§2.2). The orientation and the scales are set afterwards, by `FreeTimeGaussians.from_init_points` in `openftgs.model`: one scale for the three axes, the mean distance to the `scale_knn` = 3 nearest kept points of the same init frame, floored at `scale_floor` (§1.3 step 8, D-25); the identity quaternion; and the opacity `init_opacity` = 0.1 (step 9, D-5). So every primitive starts as a sphere, with a duration of one frame interval at the default stride, and `log_scales`, `quats` and `log_durations` then train freely at constant rates (§1.3 "Optimizer and learning rates").

What follows for the survey. Nothing in today's method refers to a surface: the loss reads colours, growth reads the 2D-mean gradient and the largest scale, relocation reads the base opacity and ρ, and the initial primitives are spheres with no orientation. The two rules that act on faint primitives, E8 and relocation, both act on the base opacity, and the notes' changes to their settings (`loss.lambda_reg`, `reloc.dead_opacity`) recovered little (section 2.1). A technique that needs a depth or a normal per pixel has no render output to read it from: Q-226's device is the one route without an API change, and section 2.4 says how far it goes. A technique that leaves pixels out of the loss, or weights them, needs an input `total_loss` does not take.

### 2.4 Probe: the depth and normal outputs the render backends can supply

*Ticket #3's section.* Which depth and normal outputs gsplat 1.5.3's stage functions (D-75) can composite, and what its `rasterization` and `rasterization_2dgs` add; whether Q-226's device keeps gradients to the stored tensors on both render backends, what the second render pass costs, and whether it carries normals; what the reference backend would need for parity (U-7, U-12).

**What was read and run.** gsplat 1.5.3 as installed in the shared `.venv` (Apache-2.0, §2.3's pin): its license first, then `gsplat/rendering.py`, `gsplat/cuda/_wrapper.py`, the 3DGS and 2DGS kernels under `gsplat/cuda/csrc/`, `gsplat/utils.py` and `gsplat/cuda/_torch_impl.py` (rows 504, 505, 506, 507, 508, 509; row 497 holds what planning took from `rendering.py`). The installed package has no `contrib` directory, so the dynamic-scene code CLAUDE.md keeps closed was not there to open (row 504). Beside it, the package's two render backends in `src/openftgs/render.py` and Q-226's device in `scripts/gt_depth.py` (section 2.3), and one experiment, in two scratch scripts whose commands close this section (row 511).

**Notation.** z is a primitive's camera-space depth at the frame's time, the third component of E6's x_c. wᵢ = αᵢ·Πⱼ<ᵢ(1 − αⱼ) is the weight with which E6 composites primitive i's colour at a pixel, so alpha = Σwᵢ. Accumulated depth is Σwᵢzᵢ, and expected depth is accumulated depth over alpha (`docs/GLOSSARY.md`). The seven slice tensors are the stored tensors the temporal slice reads: `means`, `times`, `log_durations`, `velocities`, `log_scales`, `quats` and `opacity_logits`.

**The three answers.**

1. **What D-75's stage functions can composite.** Any vector per primitive, with E6's weights, as more colour channels of the colour pass: accumulated depth, hence expected depth; a normal per primitive; the second moment of depth. The colour pass cannot give median depth or the pairwise distortion: gsplat 1.5.3 computes those only in its 2DGS kernel, which renders flat primitives with another α than E6's. For E6's render its one other route is the list of (primitive, pixel) pairs, composited outside the kernels, which this probe did not measure. `rasterization()` adds nothing to the stage functions. `rasterization_2dgs()` adds composited normals, normals from depth, distortion and median depth, for its flat primitives only.
2. **Q-226's device.** It keeps gradients to the stored tensors on both render backends, in two parts that the caller adds up, because the copy is a model of its own. Between the backends, U-12's two figures for a loss on expected depth at Q-227's pixels are a cosine above 1 − 10⁻¹¹ and a relative L2 error of at most 1.63·10⁻⁶ for `means` (20 scenes), and above 1 − 10⁻⁹ and at most 5.41·10⁻⁵ for `opacity_logits` (the 17 scenes where that gradient is not zero). The same device carries normals, coded so that E2's clamp never acts, in a pass of their own. One pass of the device, as the probe ran it, costs 14.8 ms on the trained kitchen model, 1.3 times the colour pass's forward and backward, against 0.6 ms for a depth channel added to the colour pass.
3. **The reference backend.** It already forms the weights of every pixel in PyTorch, so every output of the table is a few tensor operations there, once z stays on the graph. The limit on parity is the gsplat backend, and U-7's and U-12's wording: U-7's 10⁻⁴ is absolute and names RGB and alpha, and U-12 presumes a gradient that is not zero.

**1. What can be composited.**

- **The kernel.** `rasterize_to_pixels` (`gsplat/cuda/_wrapper.py`), the last stage function the gsplat backend calls (D-75), composites `colors` of any width from 1 to 513 channels. Its forward kernel, `rasterize_to_pixels_3dgs_fwd_kernel` (`gsplat/cuda/csrc/RasterizeToPixels3DGSFwd.cu`), is a template over the channel count. Per pixel it adds colour·α·T for each channel, and writes the composite, 1 − T and the index of the last primitive it composited, which the Python wrapper keeps for the backward pass and doesn't return. It holds no depth, median, normal or distortion of its own. The widths compiled are 1 to 5, 8, 9, and 2ᵏ and 2ᵏ + 1 up to 513; any other is zero-padded to the next one and cut afterwards. Its backward (`_RasterizeToPixels.backward`) returns gradients for the 2D means, the conics, the colours of every channel and the opacities (rows 506, 507).
- **Depth is already there.** `fully_fused_projection` returns each primitive's z in both layouts, and its backward takes a gradient on z back to the means (row 506). The gsplat backend receives those depths and passes them to `isect_tiles` only (section 2.3). Appended to the colours as a fourth channel, they come out of `rasterize_to_pixels` as accumulated depth, and a division by alpha gives expected depth. A normal per primitive, computed in PyTorch from the per-time arrays, is three more channels, and z² one more.
- **What `rasterization()` adds: nothing.** Its render modes "D", "ED", "RGB+D" and "RGB+ED" do exactly that: they append the projection's depths to the colours, or pass them alone, call the same stage functions, and for the two expected-depth modes divide the last channel by alpha clamped at 10⁻¹⁰. It has no normal, distortion or median output (row 505). Calling it would give up what D-75 and D-76 keep: the 2D means as the gradient anchor in every layout, `visible` in packed mode, and the camera centre without a matrix inverse. On the trained kitchen model its "ED" mode and the device agree: the largest difference is 1.9·10⁻⁶ m over the 1,370,875 pixels with alpha ≥ 0.5, at depths of 2.06 to 4.32 m, with the same alpha (row 511).
- **What `rasterization_2dgs()` adds, and for which primitive.** It returns the colours with the same depth modes, alpha, the composited normals in world space, normals derived from the depth map, the distortion map and the median depth (row 505). Its kernels are separate ones (`fully_fused_projection_2dgs`, `rasterize_to_pixels_2dgs`; `RasterizeToPixels2DGSFwd.cu`), and they render another primitive (row 507):
  - The projection reads two of the three scales and takes the third column of the rotation, turned to face the camera, as the normal: a flat disk. It takes no opacity, so D-32's opacity cull and E6's opacity-scaled screen radii have no place in it.
  - α comes from where the pixel's ray meets the disk's plane, limited by a fixed screen-space filter. It is not E6's α, so the image is not E6's and U-7 could not hold against today's reference backend.
  - The median depth is the depth of the last primitive composited while the transmittance before it is above 0.5, and its gradient goes to that primitive's depth. The distortion is the sum over pairs of 2·wᵢwⱼ(zᵢ − zⱼ), i behind j, accumulated in the kernel with running sums; the docstring calls it the L1 form and says the 2DGS paper's is the L2 form (rows 505, 507).
  - Its backward calls `torch.cuda.synchronize()`, and the entry point inverts the view matrices (rows 505, 506): host–device synchronizations §2.4's rule 3 doesn't list.
  - These kernels are in gsplat 1.5.3. A technique that uses them changes the primitive's shape (§1.1, E5, E6), and with it U-7, U-8, U-12, D-75's list of stage functions and the reference backend, which would need the same ray–plane intersection. Whether decision 1's third edge, which names the kernels gsplat 1.5.3 lacks, leaves such a technique eligible as a finalist is the owner's to say.
- **The pair list.** `rasterize_to_indices_in_range` lists the (primitive, pixel) pairs of the 3DGS compositing, without gradients, and `accumulate` composites such pairs in PyTorch, with autograd. From the pairs' weights a median or the pairwise distortion of E6's render could be formed: this is the only route gsplat 1.5.3 has to rows 3 and 6 of the table for today's primitives. `accumulate` needs the `nerfacc` package, which §2.3 doesn't list and the `.venv` doesn't hold, and its docstring says it is much slower and takes much more memory than the kernels (rows 506, 508, 509). Whether plain PyTorch on the pairs would do in its place, what the route costs at a training image's size, and whether it would agree with the reference backend within U-7 and U-12 were not tried.
- **Beside the composite.** `depth_to_normal` (`gsplat/utils.py`) is plain PyTorch on a depth map: it lifts the pixels to 3D points and takes the normalized cross product of central differences, zero on a one-pixel border (row 508), so it applies to any depth map of this table. The two `…_inria_wrapper` functions call rasterizers gsplat doesn't ship, under their own license; they were not followed (row 509).

**The table.** A survey row cites a cell as "section 2.4, cell 4d". "Yes" means the output of E6's render, for today's primitives. Costs are the kitchen figures of "The cost", below.

| Output | a. D-75's stage functions, in the colour pass | b. `rasterization()` | c. `rasterization_2dgs()` | d. Q-226's device through `render()` | e. The reference backend |
| --- | --- | --- | --- | --- | --- |
| **1. Accumulated depth**, Σwᵢzᵢ | Yes: z from `fully_fused_projection` as a fourth channel of `rasterize_to_pixels`. Differentiable in one graph, through z to `means`, `velocities` and `times` and through the weights to the seven slice tensors. + 0.6 ms. Needs a new `RenderOutput` field, a §2.1 change | Yes, modes "D" and "RGB+D": the same kernels as 1a and nothing more, without D-75's and D-76's guarantees | Modes "D" and "RGB+D", for flat primitives with its own α: not E6's render | Yes: the copy's image over black. Differentiable in two parts the caller adds up. U-12's bounds hold between the backends on all seven slice tensors. One more pass, 14.8 ms | Not today. `_rasterize` holds the weights; z, which `_project` detaches for the sort, would stay on the graph and be composited like a colour |
| **2. Expected depth**, Σwᵢzᵢ / Σwᵢ | Yes: 1a divided by alpha in PyTorch, on pixels with enough alpha (Q-227) | Yes, modes "ED" and "RGB+ED": 1b over alpha clamped at 10⁻¹⁰ | Modes "ED" and "RGB+ED"; as 1c | Yes: Q-226 as confirmed. U-12's bounds hold at Q-227's pixels wherever the gradient is not zero | As 1e, then the division |
| **3. Median depth** | No: the 3DGS kernel keeps the composite, alpha and the last primitive's index, nothing per depth. Only the pair list could give it ("The pair list"), not measured | No | Yes, for flat primitives: the depth of the last primitive composited while the transmittance before it is above 0.5; its gradient goes to that depth | No: a median is not a composite | Possible in PyTorch from its weights. The gsplat backend would have only the pair list to compare it with (3a), so U-7 and U-12 could cover it by that route alone |
| **4. Composited normals**, Σwᵢnᵢ for a vector n per primitive | Yes: three more channels, n computed in PyTorch from the per-time arrays, so differentiable to `quats`. Depth and a normal together are seven channels, run by the 8-channel kernel: + 3.2 ms, computing n not included. §2.1 change as 1a | Yes, as 4a: it takes colours of any width | Built in: the rotation's third column turned to the camera, returned in world space; flat primitives | Yes, coded as (n + 1)/2 and decoded as 2·image − alpha, since E2 clamps at 0. A pass of its own: depth and normals are two more passes, about 30 ms | As 1e |
| **5. Normals from the depth map** | PyTorch on 2a's map (`depth_to_normal` or the like), differentiable through autograd. Not measured | As 5a; `rasterization()` doesn't call it | Built in for "RGB+D" and "RGB+ED", from the expected or the median depth | The same PyTorch on 2d's map; its gradient then takes the device's two parts | The same PyTorch on 2e's map |
| **6. Depth distortion, pairwise L1**, Σᵢ Σⱼ<ᵢ 2wᵢwⱼ(zᵢ − zⱼ) | No: it needs running sums over the primitives in front. Only the pair list, as 3a | No | Yes, with `distloss=True` and a depth mode; flat primitives | No | Possible with two cumulative sums over its sorted primitives; comparable with the gsplat backend by the pair list alone, as 3e |
| **7. Depth distortion from moments**, Σᵢ Σⱼ wᵢwⱼ(zᵢ − zⱼ)² = 2[(Σw)(Σwz²) − (Σwz)²] | Yes: z and z² as two channels (five with RGB, a compiled width), then the identity in PyTorch. Between 1a's cost and 4a's, not measured | Yes, as 7a | Not built in: its distortion is row 6's | Yes: z and z² fit one pass, with a channel to spare. Checked on two stacked primitives | As 1e |
| **8. Any other vector per primitive** | Up to 513 channels a call; a width with no compiled kernel is zero-padded. 3 → 4 channels + 8 % of `rasterization()`'s forward and backward, 3 → 7 + 44 % | The same, in chunks of 32 channels | Up to 512 channels; flat primitives | Three channels a pass; values at or above 0, or coded affinely | Any, with its weights |

Column a's and column b's differentiability is read from the source (rows 506, 507), and their passes were timed forward and backward, but their gradients were not compared with the reference backend's, which has no such output: the path from a depth channel back to `means` through `fully_fused_projection`'s backward is the one this probe did not run against anything. Column d's gradients were run on both backends. Row 7's identity is this ticket's own algebra, checked numerically, not a form read in a paper (row 512): which distortion a technique uses is for its survey row to say. Which vector a technique takes as a primitive's normal is its survey row's matter too (section 3.1); the probe carried the rotation's third column turned to the camera, the normal gsplat's 2DGS projection takes (row 507).

**2. Q-226's device.**

**Why two parts.** `scripts/gt_depth.py` reads values: its copy is detached and rendered under `torch.no_grad()` (section 2.3). A copy built for gradients is still a model of its own, because the constructor of `FreeTimeGaussians` detaches and clones its tensors (§2.1), so through the public API `sh0` cannot be a function of the model's `means`. `render()` is differentiable with respect to the copy's tensors, `sh0` among them. The gradient of a loss L on the copy's render with respect to the model's stored tensors is then the sum of:

- (i) the weights' part: ∂L/∂ each of the copy's seven slice tensors, the colours held fixed;
- (ii) the colours' part: ∂L/∂z = (1/C₀)·Σ over the channels that hold z of ∂L/∂`sh0`, pushed back through z = R_w[2]·μ(t) + t_w[2], which reaches `means`, `velocities` and `times` (E1). E2's clamp passes it, since a visible primitive has z ≥ 0.01.

On the gsplat backend both parts rest on `_RasterizeToPixels.backward` (`gsplat/cuda/_wrapper.py`; the kernel is in `RasterizeToPixels3DGSBwd.cu`). It returns the gradient of every colour channel, which part (ii) takes through `spherical_harmonics` at degree 0 to `sh0` (`_SphericalHarmonics`, which row 27 read), and the gradients of the opacities, the conics and the 2D means, which part (i) takes on through `fully_fused_projection`'s backward to the per-time arrays (rows 506, 507).

`opacity_logits`, `log_scales`, `quats` and `log_durations` have part (i) only. A prototype that keeps only (i) drops the direct effect of moving a primitive in depth: for `means`, part (ii)'s norm is 0.12 to 39 times part (i)'s over the 17 scenes below where (i) is not zero (median 0.82), and (i) alone misses the finite differences by a relative 0.50, 0.57 and 1.00 in the three scenes checked.

**The gradient experiment: the configuration.** The source of truth is U-12, so the configuration is U-12's own, read from `tests/test_render_gsplat.py` and its helpers: the 20 random scenes of its generator (seeds 100 to 119; 8 to 62 primitives, SH degree 3), its two ring cameras of 64×48 pixels, t = 0.37, both backends rendering the same float32 CUDA model, Q-39's excluded pixels (at most 0.16 % of a camera's), the four layouts (packed or unpacked, opacity cull inside gsplat or by compaction), and its two figures per stored tensor, in float64: the cosine between the backends' gradients and the L2 norm of their difference over the reference's norm, against 0.999 and 10⁻³. Three things differ, each forced by the device:

- z is per camera, so each camera is rendered apart, where U-12 renders both in one call; the two cameras' losses are added.
- The loss is U-12's random weighting, applied to expected depth (channel 0 of the copy's image over alpha) instead of to image and alpha, on the pixels Q-227 keeps, alpha ≥ 0.5. The pixels are fixed once per scene from the reference backend's alpha, and the same pixels and weights serve both backends and every layout.
- The gradient is the sum of parts (i) and (ii).

Two controls ran in the same loop: the same weighting of accumulated depth, with no division, on every pixel Q-39 keeps, and of the decoded normals. All of this was fixed before the first run. After it, two more alpha cuts, 0.1 and 0.01, were run, because Q-227's cut keeps few pixels of these sparse scenes; all three cuts are reported, and nothing else was changed.

**The gradient experiment: the figures at Q-227's pixels.** The cut keeps 2,647 of the 122,880 pixels: 0 to 264 per camera, median 48, and none in one camera of scene 114. In scenes 102, 108 and 111 every kept pixel is covered by one primitive, so its expected depth is that primitive's z whatever the weight, and the gradient with respect to the four tensors that have part (i) only is zero: at most 1.3·10⁻¹⁵ in float64 on the reference backend, where the other 17 scenes give 6.9·10⁻² and more for `log_scales` and `quats`, 2.4·10⁻³ and more for `opacity_logits` and 3·10⁻⁴ and more for `log_durations` (the float64 run draws its weights on the CPU, so its norms are not the table's). On those three scenes both float32 backends return rounding, norms of 5·10⁻⁸ to 1.1·10⁻⁶, and U-12's figures mean nothing (U-12's own test asserts a reference norm above zero). The table leaves them out for those four tensors and says so. In packed mode with the opacity cull inside gsplat (the default layout):

| Stored tensor | Scenes | Gradient norm on the reference backend: least / median / largest | The same on the gsplat backend | Lowest cosine | Largest relative L2 error |
| --- | --- | --- | --- | --- | --- |
| `means` | 20 | 3.78465 / 32.3102 / 115.152 | 3.78465 / 32.3102 / 115.152 | 1 − 1.2·10⁻¹² | 1.63·10⁻⁶ |
| `opacity_logits` | 17 | 0.00215004 / 2.78788 / 17.2819 | 0.00214998 / 2.78788 / 17.2819 | 1 − 9.6·10⁻¹⁰ | 5.41·10⁻⁵ |
| `times` | 20 | 1.04023 / 15.2178 / 66.5263 | 1.04023 / 15.2178 / 66.5263 | 1 − 2.3·10⁻¹² | 2.17·10⁻⁶ |
| `velocities` | 20 | 0.284468 / 7.27065 / 25.6786 | 0.284468 / 7.27065 / 25.6786 | 1 − 2.3·10⁻¹² | 2.17·10⁻⁶ |
| `log_durations` | 17 | 0.000276065 / 3.16328 / 13.7959 | 0.000276046 / 3.16328 / 13.7959 | 1 − 1.5·10⁻¹² | 6.62·10⁻⁵ |
| `log_scales` | 17 | 0.0605742 / 11.4854 / 44.2033 | 0.0605742 / 11.4854 / 44.2033 | 1 − 2.9·10⁻¹² | 2.44·10⁻⁶ |
| `quats` | 17 | 0.0743027 / 10.5384 / 49.1699 | 0.0743026 / 10.5384 / 49.1698 | 1 − 1.5·10⁻¹¹ | 5.56·10⁻⁶ |

The cosines are the lowest over the four layouts. The other three layouts give the same relative errors to three digits, but for `times` (2.18·10⁻⁶ with compaction). Every figure is inside U-12's bounds. The largest errors, for `opacity_logits` and `log_durations`, are scene 114's, where those gradients are the smallest (2·10⁻³ and 3·10⁻⁴).

**The gradient experiment: the controls and the other cuts.** The worst figures over the four layouts and the seven slice tensors:

| Loss | Pixels | Scenes left out for the four tensors with part (i) only | `means`: largest relative error | `opacity_logits`: lowest cosine, largest relative error | Largest relative error over the other five tensors |
| --- | --- | --- | --- | --- | --- |
| Expected depth | alpha ≥ 0.5 (Q-227): 0 / 48 / 264 a camera | 102, 108, 111 | 1.63·10⁻⁶ | 1 − 9.6·10⁻¹⁰, 5.41·10⁻⁵ | 6.62·10⁻⁵ (`log_durations`) |
| Expected depth | alpha ≥ 0.1: 13 / 426 / 1,719 a camera | 111 | 7.63·10⁻⁶ | 1 − 2.6·10⁻⁷, 7.37·10⁻⁴ | **1.20·10⁻³** (`log_durations`, cosine 1 − 7.0·10⁻⁷) |
| Expected depth | alpha ≥ 0.01: 67 / 1,256 / 3,070 a camera | 111 | 9.19·10⁻⁶ | 1 − 4.8·10⁻¹⁰, 3.19·10⁻⁵ | 1.11·10⁻⁴ (`quats`) |
| Accumulated depth | every pixel Q-39 keeps | none | 4.63·10⁻⁶ | 1 − 1.4·10⁻¹³, 7.80·10⁻⁷ | 2.85·10⁻⁶ (`quats`) |
| Decoded normals | every pixel Q-39 keeps | none | 4.05·10⁻⁶ | 1 − 1.3·10⁻¹³, 7.15·10⁻⁷ | 2.91·10⁻⁶ (`times`) |

One figure is outside U-12's bounds: at the cut 0.1, `log_durations` in scene 102, 1.20·10⁻³ against 10⁻³, with `opacity_logits` in the same scene close under it. That scene's gradients are nearly zero there (norms 3.3·10⁻³ and 1.0·10⁻², against medians of 15 and 23), and the backends differ by 4·10⁻⁶ and 8·10⁻⁶ in absolute terms, the order of the rounding that scene 111, whose gradient is zero, shows at the same cut (norms of 3·10⁻⁷ to 6·10⁻⁶). So the backends agree on the device as they do on colour, and what expected depth adds is its own property: where one primitive dominates the kept pixels its gradient vanishes, and a relative figure then compares rounding. Accumulated depth has no such case.

**The check against finite differences.** In float64 on the reference backend, on the CPU, the sum of the two parts was compared with central differences (step 10⁻⁶) of the whole device, from the stored tensors through the copy and `render()` to the loss, on scenes 103, 107 and 111. The relative L2 error is at most 3.6·10⁻⁹ for `means`, 2.8·10⁻⁹ for `times`, 1.0·10⁻⁸ for `velocities`, and 1.7·10⁻⁸ for `opacity_logits` in the two scenes where that gradient is not zero; in scene 111 its norm is 1.6·10⁻¹⁶ by the two parts and 3.6·10⁻⁹ by differences, the differences' own rounding.

**U-7's figure on the device's image.** Outside Q-39's pixels, the backends' accumulated depth differs by at most 1.45·10⁻⁵ where it reaches 7.97, and alpha by 1.9·10⁻⁶. Expected depth differs by at most 3.3·10⁻⁶ at Q-227's pixels (depths 0.84 to 8.84), 1.45·10⁻⁵ at alpha ≥ 0.1 and 4.3·10⁻⁵ at alpha ≥ 0.01. The four layouts give the same figures. These scenes' depths stay under 10, so U-7's 10⁻⁴ happened to hold; the difference is about 2·10⁻⁶ of the depth, and it grows with the scene's extent and as the alpha cut falls.

**Normals, and E2's clamp.** Compositing is linear in the colour, so an affine code survives it and alpha decodes it: with colour (n + 1)/2, that is `sh0` = n/(2C₀), the image is (Σwᵢnᵢ + alpha)/2, and Σwᵢnᵢ = 2·image − alpha. The colour stays in [0, 1] for a unit vector, so E2's clamp at 0 never acts. On one opaque primitive, both backends return its normal (0.384, 0.869, −0.313) to 6·10⁻⁸ once the decoded vector is divided by the pixel's alpha, 0.999, and at the identity rotation (0, 0, −1). The plain code n + 0.5, decoded as image − alpha/2, returns −0.5 for that last component on both backends: E2's clamp. On U-12's scenes the coded images of the two backends differ by at most 1.3·10⁻⁶, and a loss on the decoded normals passes U-12's bounds on all seven tensors (the table above); `quats` gets a part (ii) there, through n, 0.8 to 7.9 times its part (i). A pass holds three channels, and z is per camera, so depth and normals take two more passes, and one render call serves one camera.

**The second moment.** With the colours (z, z², 1) the device returns Σwz, Σwz² and Σw in one pass. On two stacked primitives of α 0.6 and 0.7 at z 3 and 5, the weights are 0.6 and 0.28, and both backends return 2[(Σw)(Σwz²) − (Σwz)²] = 1.343997 against row 7's pairwise sum of 1.344.

**The cost.** On the trained kitchen model (M17's default run: 3,000,000 primitives, SH degree 3; camera `cam10` at τ = 5.0 s, 1352×1014, 311,206 visible primitives; row 510), forward plus backward on the gsplat backend, in wall time with a synchronization before and after, the median of 15 repetitions after 3 warm-ups (row 511):

| What is timed | Median, ms (least to largest) |
| --- | --- |
| The colour pass as a training iteration renders it: `render()` at SH degree 3, packed, with `return_aux`, and the backward of a weighted sum of the image | 11.46 (10.88 to 12.77) |
| The device's pass alone: the prepared copy rendered, and the backward of the expected-depth loss to the copy's tensors | 10.14 (10.04 to 11.44) |
| The device in full: z from the model (a second temporal slice, on the model's graph), the copy's seven tensors and `sh0` refreshed in place, the pass, and parts (i) and (ii) brought back to the model's tensors | 14.83 (14.62 to 15.94) |
| One pass of `rasterization()` from the per-time arrays and precomputed colours as leaves (no temporal slice, no SH evaluation): 3 channels, "RGB" | 7.30 (7.24 to 8.53) |
| The same with depth, 4 channels, "RGB+ED" | 7.91 (7.86 to 9.79) |
| The same with depth and a normal per primitive, 7 channels (the 8-channel kernel), "RGB+ED" | 10.49 (10.40 to 11.49) |

A first run of the same script gave medians within 5 % of these. Three things qualify them:

- **The device in full times more than a prototype must pay.** Its z comes from `at_time`, which also concatenates `sh0` and `shN` over all 3,000,000 rows, about 1.9 ms at that size (`docs/modules/cli/benchmarks.md`; #423 took that concat out of the training render). A prototype that computes z from E1 itself would save about that: near 13 ms, not measured.
- **The normal's own cost is not in the 7-channel figure.** The normals there are random unit vectors given as leaves; computing n from `quats`, and its backward, would add to it.
- **B-1's iteration.** The figure recorded on the trained kitchen model at 549o is 30.9 ms without an event (`docs/prd/validation-runs.md`, step 38). It predates #423, after which B-1's synthesized model of 3,000,000 rows went from 28.1 to 25.45 ms (`docs/modules/cli/benchmarks.md`), and the probe ran the code after #423. B-1 also times its stages with CUDA events inside the trainer, on another checkpoint of the same scene. So the shares below are of size only, given against both figures.

The device in full costs 1.3 times the colour pass: about half an iteration more (48 % of 30.9 ms, 58 % of 25.45 ms), and depth with normals, two passes, about one iteration more. A depth channel in the colour pass costs 0.61 ms: 8 % of `rasterization()`'s forward and backward, and 2.0 % or 2.4 % of an iteration. Depth and a normal cost 3.2 ms: 44 %, and 10 % or 13 %. Peak allocated memory was 1.61 GiB in the colour pass and 1.70 GiB with the device in full, the copy included.

**What the device does not give a prototype.** The 2D-mean gradient of a loss on the copy's render lands on the copy's render, not on the colour pass's `means2d`, so the accumulated gradient g of E10 (§1.3 "One training iteration", step 5) doesn't see that loss unless the prototype adds it; a channel in the colour pass would put it there with no further step. And the device doubles the synchronizations inside gsplat's stage functions (§2.4 rule 3), one set per pass.

**3. The reference backend.**

- **What parity is with.** What the gsplat backend would composite: the sum of colour·α·T per channel in `rasterize_to_pixels_3dgs_fwd_kernel` (`RasterizeToPixels3DGSFwd.cu`), applied to the z that `fully_fused_projection` (`gsplat/cuda/_wrapper.py`) returns (rows 506, 507). That z is the third component of E6's x_c, the value the reference backend's `_project` computes.
- **What it has.** `_rasterize` evaluates every pixel against every visible primitive, sorted front to back, and forms the weights wᵢ, E6's stop included, and T_final; the image is the weights times the colours. `_project` computes z, replaces it where the depth cull removes a primitive, and returns it detached, for the sort only (`src/openftgs/render.py`).
- **What each output would need there.** Accumulated depth, composited normals and the second moment (rows 1, 4, 7 and 8): z kept on the graph beside the detached sort key, and the per-primitive vector composited with the same weights, as the colours are. Expected depth: the division, and a rule for the pixels with too little alpha. Median depth and the pairwise distortion (rows 3 and 6): a selection, or two cumulative sums, over the sorted primitives. All of it is plain PyTorch, as §2.4's rule 4 wants, and differentiable by autograd.
- **What parity means for U-7.** U-7 compares RGB and alpha within 10⁻⁴ on 50 random scenes and fixes four hand cases. A depth output would need its own line: the tolerance, absolute for colours in [0, 1], has to be relative to the depth or to the scene's scale for an output in scene units (the probe saw about 2·10⁻⁶ of the depth on accumulated depth, and up to 4.3·10⁻⁵ in absolute terms on expected depth at the lowest alpha cut); expected depth needs its alpha cut stated (Q-227's 0.5 kept 2 % of the pixels of these sparse scenes); and a hand case, such as Q-226's detecting test, one opaque primitive at a known z, which `tests/test_gt_depth.py` already holds for the device (the probe's own run gave an expected depth of 4.0 at z = 4 on both backends).
- **What parity means for U-12.** A loss on accumulated depth or on composited normals passes U-12's bounds on all seven slice tensors in every scene and layout. A loss on expected depth passes them wherever its gradient is not zero, but it is zero for four tensors in 3 of the 20 scenes at Q-227's cut, and one near-zero case at a lower cut exceeds the relative bound by rounding alone. So a U-12 case for a depth output would weight accumulated depth, or expected depth together with image and alpha, to keep U-12's presumption of a gradient that is not zero.
- **What stays out of reach.** Rows 3 and 6 can be written in the reference backend, but the gsplat backend cannot supply them in the colour pass (cells 3a and 6a). Unless the pair list proves usable, no U-7 or U-12 comparison would cover them, and the production backend would not have them. For flat primitives the reference backend would need the 2DGS kernel's ray–plane intersection and its screen-space filter, which is a second E6, not an output.
- **Whose change it is.** A new `RenderOutput` field or `render` argument is a change to a §2.1 signature, and a new line in U-7 or U-12 a change to §3.1: the owner's spec revision, not a Q-n gap. This section opens no Q-n row.

**What follows for the survey and the test plan.**

- **Column 4.** A technique that needs accumulated or expected depth, normals composited from a vector per primitive, normals from the depth map, or a squared-difference distortion can be supplied both ways: by the device with no API change (cells 1d, 2d, 4d, 5d, 7d), which is the route a scratch prototype has (decision 2), or by channels of the colour pass (cells 1a, 2a, 4a, 5a, 7a), which is what its spec revision would add, with the reference backend following (column e).
- **Median depth and the pairwise distortion** are supplied for today's primitives neither in the colour pass nor by the device (cells 3a, 3d, 6a, 6d). The one route gsplat 1.5.3 has is the pair list, unmeasured, whose helper needs a package outside §2.3. Whether that route makes them outputs gsplat 1.5.3 has, under decision 1's third edge, is the owner's to say (asked of the owner with this ticket). A technique that works with expected depth, or with row 7's form, doesn't raise the question.
- **Flat primitives** have kernels in gsplat 1.5.3 (column c). In this ticket's reading, which the survey's rows and review 1 may overrule, their integration class is a change to a quantity, with a second E6 behind it, not a loss that needs new render outputs. Their eligibility as finalists is the owner's to say, as above.
- **Column 9.** Through the device, depth costs about half an iteration more on kitchen, and depth with normals about one more. As channels of the colour pass they cost about 2 % and 10 to 13 % of an iteration ("The cost"). A finalist tried through the device is therefore tried at a cost its spec revision would not have, and its test should not be read as a measure of training cost.
- **For the test plan (section 5).** A prototype on the device adds parts (i) and (ii) itself, and decides whether the new loss enters E10's g. On kitchen, where depths are 2 to 4 m, the device's expected depth matches gsplat's own "ED" mode to 2·10⁻⁶ m.

**The commands.** The package is the integration branch's at 841f1bc, whose `src/`, `tests/` and `scripts/` are `main`'s at 93eb58e. Python 3.14.10, PyTorch 2.15.0+cu132, gsplat 1.5.3; RTX 4090, CUDA 13.2, driver 595.99.02. The two scripts, `depth_gradients.py` and `depth_cost.py`, are in `<scratch>/ticket-3/`, not committed, and import the worktree's package and its test helpers (`tests/builders.py`, `tests/render_margins.py`) read-only. Every command ran in `<scratch>/ticket-3/` with `OMP_NUM_THREADS=4 MKL_NUM_THREADS=4` and `PYTHONPATH=<worktree>/src:<worktree>/harness/src:<worktree>/tests:<scratch>/ticket-3`; `<worktree>` is this ticket's worktree, and `$PY` the shared `.venv`'s Python.

| Command | Device | Wall time | What it gave |
| --- | --- | --- | --- |
| `$PY depth_gradients.py gradients gradients.json` | GPU | 1 min 59 s, a first run; the same loop took seconds afterwards | U-12's 20 scenes, both backends on the CUDA model, four layouts; the losses on expected depth at Q-227's cut, accumulated depth and decoded normals; U-7's figure on the device's images |
| `$PY depth_gradients.py gradients gradients-cut0.1.json 0.1`, and the same with `gradients-cut0.01.json 0.01` | GPU | 5 s each | The expected-depth loss at the two lower alpha cuts |
| `$PY depth_gradients.py values values.json` | GPU | seconds | The hand-built rows: one opaque primitive's depth and normal under both codes, and the two stacked primitives' moments. Run twice: the first run's rotated primitive had no component under −0.5, so an identity rotation was added to show the clamp |
| `$PY depth_cost.py <m17-runs>/kitchen/default/checkpoint <m15-runs>/kitchen/scene.json cost.json` | GPU | 7 s | The cost table, and `rasterization()`'s "ED" mode against the device. Run twice: the "ED" comparison was added for the second run, whose figures are the table's |
| `CUDA_VISIBLE_DEVICES= $PY depth_gradients.py norms64 norms64.json` | CPU | 2 s | In float64 on the reference backend, the norm of the expected-depth gradient per stored tensor, scene and cut: which scenes have a zero gradient |
| `CUDA_VISIBLE_DEVICES= $PY depth_gradients.py fd fd.json` | CPU | 6 s | The two-part gradient against central differences, scenes 103, 107 and 111 |

Every command after the first also set `OPENBLAS_NUM_THREADS=4`.

### 2.5 Probe: disagreement between cameras, and placeholder pixels

*Ticket #4's section.* The epipolar residual of the dense matcher's matches with certainty ≥ 0.5, per camera pair and frame: on openftgs-synth's `kitchen`, which gives the measure's floor, and on private scene A, inside and outside the person's crop, with one camera read 1 and 2 frames earlier and later; the share of placeholder pixels per camera and where they lie; what the figures mean for H2's premise.

### 2.6 Probe: the shell's measures

*Ticket #5's section.* On private scene A's checkpoints, the notes' three measures (the share of opacity farther than 5 cm from every init point, the share of primitives whose duration is under two frames, the distribution of base opacity) and the fixed sheet of novel views from the side, with its render request; on openftgs-synth's M17 checkpoints, the geometry residual and the error of expected depth from `scripts/gt_depth.py` (Q-226 to Q-231).

### 2.7 Probe: agreement of prior maps between cameras

*Ticket #12's section.* One estimator of depth that clears the bar, cited from section 3.3: its depth lifted to 3D with the scene manifest's cameras, against the ground-truth depth on `kitchen`, which gives the estimator's own error, and between the cameras of each camera pair on private scene A's person; how scale and shift are set, the same way on both scenes; what the figures mean for H2's premise.

## 3. The screening tables

Six sections, one per ticket of the survey, with the same tables and the same columns throughout, so that section 4 can put them together. This introduction fixes the method, the columns and how a cell is written; it is the PRD's "The search map", "Method", "Screening columns" and "Clean room for the search", with what tickets #6 to #11 add to them, and the PRD and the tickets win where this differs.

**The families and their search seeds.** Names not marked as the owner's seeds are search terms from the implementer's memory (row 502): none is a finding until a section reads it. Third-party method names are spelled as their authors spell them.

| Family | Section | What to find | Search seeds |
| --- | --- | --- | --- |
| A. Surface-following primitives | 3.1 | Flat or planar primitives; the shortest axis as the normal; flattening and scale losses; consistency between depth and normals; depth distortion; geometric consistency between cameras; oriented initialization | 2D Gaussian Splatting, Gaussian Surfels, SuGaR, PGSR, RaDe-GS, Gaussian Opacity Fields |
| B. Opacity and floater regularizers | 3.2 | Opacity entropy or binarization; opacity decay; anisotropy and rank constraints; other regularizers aimed at faint primitives off the surface | 4C4D (the owner's seed, arXiv 2604.04063, row 493); 3DGS-MCMC's regularizers |
| C. Prior maps from image networks | 3.3 (the estimators), 3.4 (the techniques) | Monocular and video depth; normal estimators, including those made for people; feed-forward point maps; 4D geometry with scene flow; the losses that use them (scale-invariant, ranking, correlation); position and velocity initialized from them | MotionCrafter (the owner's seed, arXiv 2602.08961, row 491); DN-Splatter, DNGaussian, the DUSt3R and VGGT line |
| D. Dynamic-scene surfaces | 3.5 | Dynamic surfels and meshes; local rigidity; shared motion; orientation that changes with time; training from few cameras; FreeTimeGS follow-ups | Split4D (DOI 10.1145/3763343, row 494) and the survey arXiv 2609.39960 with its list (rows 492, 495), the owner's seeds; Dynamic 3D Gaussians, SC-GS, 4D-Rotor GS, Vidu4D, MonoFusion |
| E. Cameras whose images disagree (adjacent) | 3.6 | Reconstruction from generated images; robust and uncertainty-weighted losses; per-camera correction; weighting cameras or pixels by trust; leaving pixels known to be wrong out of the loss; feature-space losses | CAT4D, SpotLessSplats, WildGaussians |

The list's table of Gaussian-splatting methods has columns for flow, normals, uncertainty and extra priors, but only two of its 60 rows tick normals (row 495), so families A to C are expected to come mostly from static-scene work and from citations.

**Method.**

- **Routes:** arXiv abstract pages and listings; web search; proceedings and project pages; Hugging Face paper pages; backward references through Crossref and the papers' own lists; forward citations through Semantic Scholar or SciX. Google Scholar has no API: it is reached through web search, or by the owner checking a paper's citing list by hand. On 2026-11-13 the keyless arXiv API, Semantic Scholar and OpenAlex returned rate-limit errors, and arXiv abstract pages and Crossref answered (rows 491 to 494, 500).
- **Funnel:** title, then abstract, then a full read of the method, the experiments that isolate it and the limitations, then a row of the screening table.
- **Stop rule, per family:** two consecutive search rounds that add no technique passing the abstract screen, or the cap of 60 abstracts and 12 full reads; 120 and 24 for family E; 360 and 72 overall (decision 1). The owner can move the caps. A search that failed or was rate-limited is not a round for the stop rule.
- **Recency round:** one round per family restricted to 2025-10 onward, including the papers that cite FreeTimeGS.
- **The estimators** (section 3.3) are screened by kind, not by this funnel: at most six estimators per kind, those the Gaussian-splatting literature of 2024 to 2026 uses most and the newest that pass the bar; a kind stops after two consecutive search rounds that add no estimator, or at six. Reading a paper's abstract to place an estimator is logged but doesn't count toward a family's cap.
- **Hand-overs.** A technique that belongs to another family is named where it was met, with that family, and its row is left to that family's section. Losses that need a prior map go to section 3.4; flattening, and consistency between depth and normals, that need no prior map go to section 3.1; opacity regularizers go to section 3.2. 4C4D's row is section 3.2's, and section 3.5 names it for its training from few cameras. MotionCrafter the estimator is screened in section 3.3, and its point maps and scene flow as an initialization of position and velocity in section 3.4.
- **Besides its tables,** a family's section says for each search seed where its row is or why it has none, records the recency round, lists the techniques handed to other families, and says how the family reached its stop rule.

**Clean room for the search.** Searches may name FreeTimeGS, and papers that cite it are read. Code that implements or extends it is never opened, Split4D's included; a hit that looks like such code gets a "not opened" source-log row. EasyVolcap is never opened. Each project's license file is read before anything else of it; source code is read only under a permissive license that allows commercial use, and a repository under a copyleft, non-commercial or missing license is documentation only, its method taken from its paper. gsplat is read at the pinned 1.5.3 only; `gsplat/contrib/dynamic` and dynamic-scene code on newer branches stay unopened. Weights' terms come only from the weights' publisher: its README, model card, license file or release page. No repository-indexing tools. Third-party text is paraphrased, never pasted. Keys live in environment variables and are never committed. No request carries the owner's name or email, this project's name or anything about private scene A.

**The screening columns**, tightest first. Every screening table has these ten columns, the technique and the PRD's nine, under these headers:

| Header | What the cell holds |
| --- | --- |
| Technique | The technique's name, its paper and year |
| 1. Weights' license | For each estimator the technique uses: its weights' license per weights file, from its publisher, with the training data recorded, and its verdict against the bar. The verdict is cited from the estimators table that holds it (section 3.3, or the family's own) |
| 2. Code license | The code's license, and whether the code is readable or the technique is taken from its paper only |
| 3. Integration class | One of five classes, in these words: a loss on today's primitives; a loss that needs new render outputs; a change to growth, relocation or split; a change to a quantity or to motion; new rasterization kernels. With it, whether the field's default leaves E1 to E17 and the §3 tests untouched |
| 4. Render backends | What it needs from the render backends (which depth, normals, distortion), and whether gsplat 1.5.3's stage functions and the reference backend can supply it, cited from section 2.4 |
| 5. Fit with the primitives | Its fit with constant orientation and scale, E1's linear motion, short durations, relocation (E10, E11) and split (E17) |
| 6. Cameras that disagree | Whether it assumes cameras whose images agree and are right in every pixel, and any evidence with few cameras, inconsistent images or pixels known to be wrong |
| 7. Prior maps | The prior maps to precompute (depth, normals, masks, flow, point maps), their cost, and whether maps the user supplies would do |
| 8. Evidence | The data, the metric, and an experiment that isolates the technique |
| 9. Training cost | Its training cost against the default training configuration, whose per-iteration cost by stage is B-1's (§3.4) |

**The estimators tables.** Section 3.3 holds family C's estimators table. An estimator a technique names that section 3.3 lacks is screened in its family's own table, "Estimators first met in this family", with the same columns and the same bar. One row per weights file:

| Header | What the cell holds |
| --- | --- |
| Kind | The kind of prior map: monocular depth; video depth; surface normals; depth and normals made for people; feed-forward point maps; 4D geometry with scene flow; optical flow and point tracks; masks (segmentation and matting) |
| Estimator | The estimator's name, its paper and year |
| Weights file | The file, or the release, the row's terms apply to |
| Code license | The license of the estimator's code |
| Weights: the publisher's statement, with its URL | What the weights' own publisher states about that file's license, and where |
| Training data | The data the weights were trained on, recorded and not judged |
| Bar | Pass or fail against the bar, with the reason |
| Size; cost per frame | The parameters and the cost per frame, as the publisher states them |
| Supplied maps | Whether prior maps the user supplies as files would do in the estimator's place |

**The search-round logs.** One row per search round. In a family's log: the round's number, its date, its route, its query, the number of results seen, how many passed the title screen (their abstracts were read, and count toward the family's cap), how many passed the abstract screen (techniques new to the survey; a round that has none counts toward the stop rule), how many were read in full (they count toward the cap of full reads), how many rows of the screening table came of it, and a note: the recency round, a failed or rate-limited search, a hand-over. Section 3.3's log is by kind: the kind, the round, its date, its route, its query, the results seen, the estimators added and a note.

**How a cell is written.**

- Every cell is filled. Where a column doesn't apply the cell says "none" and why, as "none: no estimator"; where the source read doesn't say, "not stated".
- Every fact in a cell carries its source-log row, as `(row 499)` or `(rows 492, 495)`. On a ticket's branch a new row is cited by its temporary ID in the same form, `T`, the ticket's number, a point and the row's counter, and written in no other way, so that the merge can replace every one with its number.
- A verdict or an answer taken from this document is cited by its section: the bar's verdict from section 3.3 or the family's own estimators table, the render backends' answer from section 2.4.
- A cell that is the ticket's own reading of the spec or of this repository's code (columns 3 and 5, and parts of 4 and 9) cites the spec item or the Q-n row it rests on, which need no source-log row, beside the row of the paper the technique's needs were read from.
- Nothing is filled from memory. A claim with no source read is left out, or logged as "No source read: general knowledge" and marked so in the cell.
- Third-party text is paraphrased, never pasted.

### 3.1 Family A: surface-following primitives

*Ticket #7's section.* The survey of family A, with the cap of 60 abstracts and 12 full reads.

**Screening table.**

| Technique | 1. Weights' license | 2. Code license | 3. Integration class | 4. Render backends | 5. Fit with the primitives | 6. Cameras that disagree | 7. Prior maps | 8. Evidence | 9. Training cost |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

**Estimators first met in this family.**

| Kind | Estimator | Weights file | Code license | Weights: the publisher's statement, with its URL | Training data | Bar | Size; cost per frame | Supplied maps |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

**Search-round log.**

| Round | Date | Route | Query | Results seen | Passed the title screen | Passed the abstract screen | Read in full | Rows added | Note |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

### 3.2 Family B: opacity and floater regularizers

*Ticket #8's section.* The survey of family B, with the cap of 60 abstracts and 12 full reads; a technique that only moves a value the notes already tried (section 2.1) is recorded as tried, not screened.

**Screening table.**

| Technique | 1. Weights' license | 2. Code license | 3. Integration class | 4. Render backends | 5. Fit with the primitives | 6. Cameras that disagree | 7. Prior maps | 8. Evidence | 9. Training cost |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

**Estimators first met in this family.**

| Kind | Estimator | Weights file | Code license | Weights: the publisher's statement, with its URL | Training data | Bar | Size; cost per frame | Supplied maps |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

**Search-round log.**

| Round | Date | Route | Query | Results seen | Passed the title screen | Passed the abstract screen | Read in full | Rows added | Note |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

### 3.3 Family C, part 1: the estimators and the bar

*Ticket #6's section.* Family C's estimators table: for each of the eight kinds of prior map, the estimators a technique could use, each judged against the bar before anything else of it is read. This section screens estimators, not techniques, so it has no screening table; family C's is in section 3.4.

**Estimators table.**

| Kind | Estimator | Weights file | Code license | Weights: the publisher's statement, with its URL | Training data | Bar | Size; cost per frame | Supplied maps |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

**Search-round log.**

| Kind | Round | Date | Route | Query | Results seen | Estimators added | Note |
| --- | --- | --- | --- | --- | --- | --- | --- |

### 3.4 Family C, part 2: techniques that use prior maps

*Ticket #9's section.* The survey of family C's techniques: the losses that use prior maps, and positions, velocities and orientations initialized from them, with the cap of 60 abstracts and 12 full reads.

**Screening table.**

| Technique | 1. Weights' license | 2. Code license | 3. Integration class | 4. Render backends | 5. Fit with the primitives | 6. Cameras that disagree | 7. Prior maps | 8. Evidence | 9. Training cost |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

**Estimators first met in this family.**

| Kind | Estimator | Weights file | Code license | Weights: the publisher's statement, with its URL | Training data | Bar | Size; cost per frame | Supplied maps |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

**Search-round log.**

| Round | Date | Route | Query | Results seen | Passed the title screen | Passed the abstract screen | Read in full | Rows added | Note |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

### 3.5 Family D: dynamic-scene surfaces

*Ticket #10's section.* The survey of family D, with the cap of 60 abstracts and 12 full reads; the rows of the list of row 495 are search terms, and Split4D's paper is read, its code never.

**Screening table.**

| Technique | 1. Weights' license | 2. Code license | 3. Integration class | 4. Render backends | 5. Fit with the primitives | 6. Cameras that disagree | 7. Prior maps | 8. Evidence | 9. Training cost |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

**Estimators first met in this family.**

| Kind | Estimator | Weights file | Code license | Weights: the publisher's statement, with its URL | Training data | Bar | Size; cost per frame | Supplied maps |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

**Search-round log.**

| Round | Date | Route | Query | Results seen | Passed the title screen | Passed the abstract screen | Read in full | Rows added | Note |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

### 3.6 Family E: cameras whose images disagree

*Ticket #11's section.* The survey of the adjacent family E, with the cap of 120 abstracts and 24 full reads; the setting is decision 4's: no camera of private scene A is a real video or trusted above the others, and some of its pixels are placeholder pixels.

**Screening table.**

| Technique | 1. Weights' license | 2. Code license | 3. Integration class | 4. Render backends | 5. Fit with the primitives | 6. Cameras that disagree | 7. Prior maps | 8. Evidence | 9. Training cost |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

**Estimators first met in this family.**

| Kind | Estimator | Weights file | Code license | Weights: the publisher's statement, with its URL | Training data | Bar | Size; cost per frame | Supplied maps |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |

**Search-round log.**

| Round | Date | Route | Query | Results seen | Passed the title screen | Passed the abstract screen | Read in full | Rows added | Note |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |

## 4. The finalists and review 1

*Ticket #13's section.* One screening table and one estimators table, consolidated from sections 3.1 to 3.6; H2's premise, answered with the probes' figures (sections 2.5 to 2.7); two to four finalists, at least one of them only a loss on today's primitives, each with the reason it was chosen; the proposed criteria for training tests, starting from the PRD's proposal; per finalist, the route for trying it and its GPU hours; the owner's answers at review 1, with their date.

## 5. The test plan

*Ticket #14's section.* For each finalist the owner kept at review 1, before anything trains: the spec items it would touch, by identifier, and how; the fields of the training configuration it would add, with their defaults, and whether the default leaves E1 to E17 and the §3 tests untouched; the prior maps it needs and how they would reach training; the runs, with the criteria agreed at review 1. A sketched signature or field is a proposal for the owner's spec revision, not a Q-n candidate.

## 6. The results

*Ticket #15's section.* For every finalist review 1 cleared to train, its results against the agreed criteria; or the record that none was cleared, or that a finalist waits for its spec revision and is tested later behind its field.

## 7. The recommendation

*Ticket #15's section.* Which techniques to adopt, each as a field of the training configuration that defaults to off, or that none should be adopted.

## 8. What adopting it would change

*Ticket #15's section.* For each recommended technique, over the PRD's list: §1.1 and E5 to E6 if a quantity or the primitive's shape changes, and E1 to E4 if orientation varies with time; E7 and E9 for new loss terms; §1.3 Growth, E10, E11 and E17 for constrained primitives; §1.3 Initialization and E15 for initialization from prior maps; §2.1's `RenderOutput`, `render` and `LossConfig`; §2.2's `ftgs-scene` and `ftgs-if`; §2.3 for any estimator, weights or dependency; U-7, U-8 and U-12; §3.3's adapter contract; §2.4 for cost.

## 9. The sources

*Ticket #15's section.* The clean-room record in full, completing the opening's: the source-log rows of every ticket of the PRD, what was read only as documentation, what was not opened, and any incident.
