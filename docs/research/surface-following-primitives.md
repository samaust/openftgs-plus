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

**What was measured, and with what.** Two measurements on the images of private scene A, on 2026-11-14 and 15, by scratch scripts under `<scratch>/ticket-4`, which are not committed: "Runs", at the end of this section, lists every command. The package is the worktree's, whose `src/` and `scripts/` are `main`'s at 93eb58e, on Python 3.14, torch 2.15.0+cu132, NumPy 2.5.4, SciPy 1.18.1 and one RTX 4090, within six CPU threads. The dense matcher's packages, which the shared `.venv` lacks, sit in a scratch folder: torchvision 0.29.2+cu132, einops 0.8.2 and loguru 0.8.0 (row T4.5). The results are in `<surface-runs>/ticket-4`. No image of private scene A was opened, and no sheet made from one was looked at: scripts read the images and wrote numbers and files. Camera names are the manifests': `cam0` to `cam8` on private scene A, where `cam0` is the centre camera and `cam1` the one left of it (row T4.3), and `cam00` to `cam19` on `kitchen`.

**The person's crop.** The owner's tools score a run on a fixed rectangle per held-out camera, in pixels as (x0, y0, x1, y1) with x1 and y1 exclusive: (96, 256, 640, 1280) on `cam0`, which is also the tools' default and the one they apply to poses between cameras, and (0, 256, 640, 1280) on `cam1` (row T4.2). The notes call it the crop that follows the person; it is the same for every frame. The other seven cameras have none, and both measurements need one per camera, so two rules were run:

- **Rule a,** which every figure below uses unless it says otherwise: the tools' rectangle where they give one, and their default, `cam0`'s, on the other seven cameras.
- **Rule b,** by camera maths alone: `cam0`'s rectangle carried to each camera through the two planes that face `cam0` at 1 and 1.5 times the depth of the point the nine optical axes meet at (1.05 units in front of `cam0`); the box around both outlines, with the side that lies on `cam0`'s lower image border kept on the border, rounded outward to 16 px and clipped to the image. On `cam1` it gives (0, 208, 624, 1280), against the tools' (0, 256, 640, 1280): that agreement is why the two depths were chosen. On the others: `cam2` (112, 208, 736, 1280), `cam3` (48, 128, 688, 1280), `cam4` (0, 80, 656, 1280), `cam5` (80, 80, 736, 1280), `cam6` (16, 304, 720, 1280), `cam7` (0, 256, 672, 1280), `cam8` (64, 256, 736, 1280).

How far rule a can be off, by the same maths: carried through the plane at the meeting depth, the outline of `cam0`'s rectangle lands up to 101 px from the default's left, right and top sides (the left side on `cam7`), and through the plane at 1.5 times that depth up to 213 px; the bottom side lies on the image border and is left out of that comparison. A pixel (x, y), counted from the image's top-left corner, is inside its camera's crop when x0 ≤ x < x1 and y0 ≤ y < y1. Which rule the owner means is a question for the owner; the sheet below draws rule a's rectangle on every camera.

**Disagreement between cameras: the measure.**

- **Pairs.** The camera pairs are D-22's, made by the package's own pairing, the one `roma_init` plans with: each training camera with the one whose optical axis is closest, each pair once. `kitchen` has 13 pairs among its 19 training cameras (`cam00` is its test camera), 2.8° to 6.6° between the axes. Private scene A has 7, all 14.5° or 15.0° apart: `cam0`–`cam1`, `cam3`–`cam4`, `cam3`–`cam5`, `cam6`–`cam7` and `cam6`–`cam8`, whose cameras sit side by side, and `cam1`–`cam4` and `cam2`–`cam5`, whose cameras sit one above the other.
- **Matches.** For a pair (A, B), a frame k and an offset d, A's image of frame k is matched to B's image of frame k + d by the package's dense matcher: the builder, the settings and the deterministic context of `ftgs init roma` (roma_outdoor at 560 px, upsampled to 864; D-58, Q-106), on the weights already in the cache (row T4.6; nothing was downloaded, Q-7). The matcher gives a match for every point of an 864 × 864 grid on each of the two images, 1,492,992 per evaluation. A match is admitted as D-22 admits it (Q-110): both ends inside their images and a certainty of at least 0.5. Every admitted match is measured, not only D-22's draw of 5,000.
- **The residual** of a match (x_A, x_B) is its Sampson error, in pixels of §1.4's continuous convention, with the fundamental matrix F of the manifest's two cameras: r = |x_Bᵀ F x_A| / √((F x_A)₁² + (F x_A)₂² + (Fᵀ x_B)₁² + (Fᵀ x_B)₂²). It is the measure of the spec's converter checks C-1 and C-4, whose bar is a median of 1 px. It is also given in matcher cells, after scaling each image's axes to the 864 grid, because the matcher's precision is a matter of its grid: a cell is 1.56 × 1.17 px on `kitchen` and 0.85 × 1.48 px on private scene A.
- **What the residual sees.** Only the part of a disagreement that lies across the epipolar line. A point that moved along the line triangulates to another depth and leaves no residual, so the residual is a lower bound on how far two images disagree. On private scene A the five side-by-side pairs have horizontal epipolar lines and see vertical disagreement only, and the two stacked pairs see horizontal disagreement only. For two cameras with one K that share the error evenly, a residual r is a distance of r√2 from the epipolar line and, after triangulation, a reprojection error of r/√2 in each view, so D-22's 2 px filter sits near r = 2.83 px.
- **Regions.** On `kitchen`, by the ground truth's masks of dynamic objects (row T4.4), read at each end's pixel in its own image and frame: a match is dynamic when both ends are on dynamic pixels, static when neither is, and mixed otherwise (one match in 2,000). On private scene A, by the person's crop: inside when both ends lie in their cameras' crops, outside when neither does, and "one end inside" otherwise.
- **By depth,** on private scene A, as a second split that needs no rectangle. Each admitted match is given the depth, along camera A's optical axis, of the point where its two rays pass closest. The depths fall in two groups with almost nothing between them: 0.6 to 1.4 units, around the point the optical axes meet, where the person stands, and 2.2 units and beyond, the background. A match is near at a depth of 0.5 to 1.8 and far at 2.2 or more; 0.6% of the admitted matches lie between and are left out. The split holds where the two images disagree, because depth is read along the epipolar line and the residual across it.
- **Coverage** of a region is the share of the matcher's grid points lying in it that have an admitted match. It stands beside every residual, because the residual speaks only for the matches the matcher is certain of: where two images disagree most, the matcher may admit nothing.
- **Init points.** Each evaluation also makes D-22's draw (up to 5,000 admitted matches, in proportion to certainty) and applies the triangulation and its filters, with the package's own functions; the share kept is the share that would become init points.
- **Frame offsets.** d = 0 is the manifest's pairing. With d = ±1 and ±2, camera B is read one and two frames later or earlier than camera A. If part of the disagreement were timing, some d ≠ 0 would give a lower residual than d = 0. The comparison is made per frame: the median residual of frame k at offset d against the same frame's at offset 0.
- **One camera, two frames.** As a control with no epipolar geometry, each camera's image of frame k is also matched to its own image of frame k + 1, on every tenth frame. It gives the coverage of a region when the two images do agree, and how far the region moves in one frame.
- **Figures.** Per pair and region, the median and the 90th percentile over the admitted matches of all frames together, from histograms with bins of 0.001 px up to 2 px and of 0.01 px up to 20 px; and the median of each frame alone.

**Checks of the measure.**

- **Synthetic correspondences whose residual is known** (`check_epi.py`, 32 checks, all passing). Exact projections of 3D points give a residual under 10⁻⁹ px. On a rectified pair a shift of δ px across the epipolar line gives δ/√2, and a shift along it gives 0. On a general pair (two different K, 15° apart, an oblique baseline) a shift of 0.7, 3 and 40 px across the line gives (d_A⁻² + d_B⁻²)^(−1/2), where the two distances from the epipolar lines are built from 3D points and not from F. The figure in cells equals the residual computed with cameras rescaled to the grid. A point that moves across the baseline, seen one or two frames apart, gives f·v·|d|/(z√2); one that moves along the baseline gives 0. Triangulated by the package's `triangulate_dlt`, a shift of 3.8 px across the line reprojects 1.900 px off in each view, and one of 4 px reprojects 2 px off, D-22's filter.
- **Exact correspondences through the whole script.** With the matcher replaced by correspondences from `kitchen`'s ground-truth depth (two pairs, three frames each), 90% of the residuals are under 2.2·10⁻⁵ px, the precision of the manifest's float32 cameras, and all 5,000 drawn matches pass D-22's filters.
- **The matcher against the ground truth,** on `kitchen`, every tenth frame of the 13 pairs: each admitted match's end in camera B against the place the ground-truth depth at its end in camera A puts it. The other camera's depth map confirms 99.4% of the admitted matches as points it sees. Their error is 0.080 px at the median and 0.245 px at the 90th percentile on static pixels, and 0.129 and 0.379 px on dynamic ones (each the median over the evaluations). Of the static error, 0.039 px lies across the epipolar line and 0.054 px along it; the residual reads the first part only, about half of the matcher's own error.

**The floor on `kitchen`.** One rendered scene through exact, synchronized cameras, so everything the residual reads there is the measure's own: the matcher's error, the video compression of the converted frames, and the manifest's float32 cameras. D-22's 13 pairs, all 300 frames, 3,900 evaluations at offset 0. The matcher admits 79.9% of the grid points, and D-22's filters keep every drawn match, 5,000 of 5,000 in every evaluation.

| Pair | Angle between the axes | Static: median, 90th percentile (px) | Dynamic: median, 90th percentile (px) | Median in cells: static, dynamic | Coverage: static, dynamic | Drawn matches kept by D-22's filters |
| --- | --- | --- | --- | --- | --- | --- |
| `cam01`–`cam02` | 2.8° | 0.029, 0.091 | 0.040, 0.119 | 0.020, 0.028 | 82.6%, 63.9% | 100.00% |
| `cam02`–`cam03` | 4.9° | 0.027, 0.078 | 0.040, 0.118 | 0.021, 0.030 | 79.3%, 63.0% | 100.00% |
| `cam04`–`cam05` | 5.3° | 0.032, 0.091 | 0.045, 0.132 | 0.022, 0.031 | 80.4%, 64.5% | 100.00% |
| `cam06`–`cam07` | 4.4° | 0.030, 0.085 | 0.043, 0.120 | 0.022, 0.032 | 79.3%, 67.8% | 100.00% |
| `cam07`–`cam08` | 3.4° | 0.026, 0.083 | 0.040, 0.117 | 0.019, 0.029 | 82.7%, 69.1% | 100.00% |
| `cam07`–`cam09` | 6.6° | 0.023, 0.066 | 0.033, 0.091 | 0.019, 0.028 | 77.5%, 67.9% | 100.00% |
| `cam10`–`cam11` | 4.1° | 0.031, 0.087 | 0.045, 0.128 | 0.022, 0.032 | 80.4%, 65.9% | 100.00% |
| `cam11`–`cam12` | 4.2° | 0.027, 0.084 | 0.042, 0.125 | 0.020, 0.030 | 83.0%, 68.1% | 100.00% |
| `cam13`–`cam14` | 5.9° | 0.025, 0.086 | 0.044, 0.132 | 0.019, 0.032 | 78.8%, 64.9% | 100.00% |
| `cam14`–`cam15` | 4.7° | 0.031, 0.093 | 0.045, 0.132 | 0.022, 0.032 | 80.9%, 65.1% | 100.00% |
| `cam15`–`cam16` | 3.8° | 0.024, 0.073 | 0.038, 0.102 | 0.018, 0.029 | 82.0%, 69.2% | 100.00% |
| `cam17`–`cam18` | 5.3° | 0.025, 0.085 | 0.041, 0.117 | 0.019, 0.030 | 79.7%, 67.6% | 100.00% |
| `cam18`–`cam19` | 4.8° | 0.027, 0.081 | 0.036, 0.103 | 0.021, 0.029 | 79.5%, 68.0% | 100.00% |
| The 13 pairs together | 2.8° to 6.6° | 0.027, 0.084 | 0.041, 0.118 | 0.020, 0.030 | 80.5%, 66.6% | 100.00% |
| Six pairs beyond D-22, 30 frames each | 14.5° to 15.3° | 0.022, 0.075 | 0.033, 0.100 | 0.019, 0.028 | 64.6%, 56.4% | 100.00% |

So the floor is a median of 0.02 to 0.05 px and a 90th percentile of 0.07 to 0.13 px, about one and a half times higher inside the masks of dynamic objects than outside them, and about a thirtieth of C-1's bar. The last row is not D-22's: private scene A's pairs are 14.5° or 15° apart and `kitchen`'s at most 6.6°, so six pairs of `kitchen` cameras four apart on its arc (`cam02`–`cam06`, `cam03`–`cam07`, `cam05`–`cam09`, `cam06`–`cam10`, `cam08`–`cam12`, `cam12`–`cam16`) were matched on every tenth frame. At 15° the residual is no higher; only the coverage falls, from 80.5% to 64.6% on static pixels and from 66.6% to 56.4% on dynamic ones.

What a timing error looks like in the measure, on `kitchen`'s every tenth frame with camera B read one and two frames off:

| Offset of camera B, frames | Static: median of the per-frame medians (px) | Dynamic: median of the per-frame medians (px) | Dynamic: change against offset 0, per frame: median [10th, 90th percentile] | Pair-frames whose dynamic median is lower than at offset 0 | Dynamic: drawn matches kept by D-22's filters |
| --- | --- | --- | --- | --- | --- |
| −2 | 0.027 | 7.297 | +7.249 [+0.132, +13.576] | 3% of 377 | 36.1% |
| −1 | 0.027 | 3.695 | +3.648 [+0.264, +7.068] | 3% of 377 | 46.7% |
| 0 | 0.027 | 0.041 | 0 | – of 390 | 100.0% |
| +1 | 0.027 | 3.493 | +3.452 [+0.155, +6.575] | 1% of 390 | 47.6% |
| +2 | 0.027 | 6.706 | +6.665 [+0.170, +13.521] | 1% of 390 | 36.9% |

The static residual does not move. The dynamic one rises about ninety-fold with one frame of offset, and is lower than at offset 0 in 1% to 3% of the pair-frames. The control of one camera at two consecutive frames (the 19 training cameras, every tenth frame) says how far they move: 8.9 px per frame at the median, 8.4 px of it along x. So on this scene one frame of offset turns 8.9 px of motion into 3.5 to 3.7 px of residual. The same control gives the coverage when two images agree but for one frame of motion: 87.4% on static pixels and 57.3% on dynamic ones.

**Private scene A: per pair, inside and outside the crop.** D-22's 7 pairs, all 120 frames, 840 evaluations at offset 0. The matcher admits 13.6% of the grid points, against 79.9% on `kitchen` and 64.3% on `kitchen`'s pairs 15° apart. The crop is rule a's.

| Pair | The residual sees | Inside: median, 90th percentile (px) | Outside: median, 90th percentile (px) | One end inside: median, 90th percentile (px) | Inside: median in cells | Coverage: inside, outside | Admitted matches: inside, outside, one end inside | Inside: above 2.83 px | Drawn matches kept by D-22's filters: inside, outside |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `cam0`–`cam1` | vertical disagreement | 0.68, 2.28 | 0.92, 3.08 | 0.21, 0.43 | 0.46 | 19.0%, 7.5% | 72%, 7%, 21% | 5.6% | 93.8%, 84.4% |
| `cam1`–`cam4` | horizontal disagreement | 3.27, 8.51 | 4.75, 10.96 | 2.26, 3.45 | 3.84 | 20.6%, 7.4% | 73%, 4%, 23% | 55.9% | 40.5%, 11.8% |
| `cam2`–`cam5` | horizontal disagreement | 1.05, 2.77 | 0.40, 3.34 | 1.01, 2.87 | 1.23 | 19.1%, 8.9% | 70%, 18%, 12% | 9.4% | 89.4%, 83.6% |
| `cam3`–`cam4` | vertical disagreement | 2.45, 6.18 | 2.74, 6.74 | 1.13, 3.42 | 1.66 | 18.8%, 7.6% | 69%, 13%, 19% | 43.4% | 53.0%, 47.4% |
| `cam3`–`cam5` | vertical disagreement | 2.89, 10.06 | 1.13, 6.21 | 0.46, 1.39 | 1.96 | 18.0%, 8.6% | 64%, 10%, 26% | 50.7% | 46.4%, 66.2% |
| `cam6`–`cam7` | vertical disagreement | 2.70, 6.13 | 4.36, 9.08 | 0.60, 1.97 | 1.83 | 16.1%, 3.6% | 79%, 1%, 20% | 48.2% | 48.8%, 33.7% |
| `cam6`–`cam8` | vertical disagreement | 1.86, 7.69 | 14.50, 25.87 | 0.46, 2.62 | 1.26 | 15.3%, 2.7% | 82%, 1%, 17% | 35.2% | 63.9%, 6.8% |
| The 7 pairs together | – | 1.84, 6.61 | 1.71, 6.19 | 0.76, 2.80 | 1.52 | 18.2%, 6.6% | 72%, 8%, 20% | 35.0% | 62.4%, 64.4% |

- **Inside the crop** the median is 0.68 to 3.27 px per pair and 1.84 px over the seven; the 90th percentile is 2.28 to 10.06 px and 6.61 px. It is not steady. A pair's per-frame median ranges up to eightfold (`cam1`–`cam4`: 1.29 to 10.59 px). In five pairs the frames 80 to 119, where the person is closest, read 2.2 to 4 times the frames 0 to 79: 2.47 against 5.41 px for `cam1`–`cam4`, 1.78 against 3.99 for `cam3`–`cam4`, 1.83 against 6.07 for `cam3`–`cam5`, 1.62 against 5.05 for `cam6`–`cam7` and 1.04 against 4.21 for `cam6`–`cam8`. `cam0`–`cam1` and `cam2`–`cam5` read about 0.65 and 1.1 px in both parts.
- **Coverage** inside is 15% to 21%: the matcher is certain of under a fifth of the crop.
- **Init points.** D-22's draw is 5,000 in every evaluation. Its filters keep 67.4% of them over the seven pairs, from 47% for `cam1`–`cam4` to 94% for `cam0`–`cam1`, and every match they drop fails the 2 px reprojection filter. Inside the crop they keep 62.4%.
- **Outside, and one end inside.** The rectangles fence the background badly. Matches with neither end inside are 8% of the admitted ones, on a coverage of 2.7% to 8.9%, and their median runs from 0.40 px (`cam2`–`cam5`) to 14.5 px (`cam6`–`cam8`, on 1% of that pair's matches). Matches with one end inside are 20%: background that parallax carries across a side of one camera's rectangle. Their median is 0.21 to 2.26 px.
- **Rule b** moves the inside figures little: 1.63 and 6.23 px over the seven pairs, 0.68 to 2.70 px per pair at the median, on a coverage of 16.8%. It moves the outside figures a lot, to 3.24 and 7.99 px on 2% of the admitted matches. So the figures inside the crop do not hinge on which rule the owner means; the figures outside it do, which is why the next split uses depth.

**Private scene A: the person and the background, by depth.** The same 840 evaluations. Near matches are the person's: they are 58% of the admitted matches, and 97.5% of them have their end in camera A inside rule a's crop. Of the matches whose end in camera A lies in the crop, 30% are far: background seen past the person.

| Pair | Near matches, share of the admitted | Near: median, 90th percentile (px) | Far: median, 90th percentile (px) | Far: per-frame medians, least to most (px) | Turn fitted to the far matches | Far, odd frames: median before → after the turn; 90th percentile after | Near, odd frames: median before → after the turn; 90th percentile after |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `cam0`–`cam1` | 53% | 0.94, 2.67 | 0.29, 0.89 | 0.24 to 0.55 | 0.24° | 0.32 → 0.21; 0.78 | 0.96 → 0.87; 2.61 |
| `cam1`–`cam4` | 51% | 5.23, 9.62 | 2.24, 3.44 | 2.07 to 2.87 | 0.27° | 2.35 → 0.47; 1.45 | 5.12 → 3.28; 7.36 |
| `cam2`–`cam5` | 52% | 1.10, 3.21 | 0.71, 2.63 | 0.36 to 1.04 | 0.26° | 0.64 → 0.44; 2.03 | 1.10 → 1.41; 3.70 |
| `cam3`–`cam4` | 59% | 3.04, 6.93 | 1.30, 3.26 | 1.02 to 3.25 | 0.68° | 1.64 → 0.42; 1.21 | 2.95 → 2.02; 6.13 |
| `cam3`–`cam5` | 56% | 3.77, 13.75 | 0.58, 1.62 | 0.43 to 1.06 | 0.66° | 0.64 → 0.59; 2.11 | 3.57 → 3.62; 12.45 |
| `cam6`–`cam7` | 68% | 3.30, 6.43 | 0.70, 1.43 | 0.53 to 1.40 | 0.59° | 0.71 → 0.26; 0.75 | 3.05 → 3.84; 6.90 |
| `cam6`–`cam8` | 77% | 1.87, 8.65 | 0.59, 2.77 | 0.24 to 2.35 | 0.41° | 0.65 → 0.37; 2.11 | 1.64 → 2.96; 9.49 |
| The 7 pairs together | 58% | 2.35, 7.60 | 0.87, 2.84 | – | – | – | – |

- **The person.** A median of 0.94 to 5.23 px per pair and 2.35 px over the seven; a 90th percentile of 2.67 to 13.75 px and 7.60 px.
- **The background** is not at the floor either: 0.29 to 2.24 px at the median, 0.87 px over the seven, where `kitchen`'s static pixels read 0.022 px at the same angle. Unlike the person's, its residual is steady: for `cam1`–`cam4` every one of the 120 frames reads between 2.07 and 2.87 px, where the person's per-frame median goes from 0.77 to 10.67 px. A residual that does not change with the frame, on a background that does not move, is a fault of the pair's geometry, not of the scene's content.
- **A turn of one camera removes most of it.** Camera B is turned about its own centre by a small rotation (three numbers; both centres and both K kept), fitted to a sample of the far matches of the even frames by least squares on the residual with a Cauchy loss, and tried on the odd frames. A turn moves every image point alike, whatever its depth, so one fitted to the background holds for the person too. The fitted turns are 0.24° to 0.68°. After them the background's median is 0.21 to 0.59 px in every pair (from 2.35 to 0.47 px for `cam1`–`cam4`, from 1.64 to 0.42 px for `cam3`–`cam4`).
- **The person's residual does not follow.** After the turn its median is 0.87 to 3.84 px and its 90th percentile 2.61 to 12.45 px: lower than before in `cam1`–`cam4` and `cam3`–`cam4`, about the same in `cam0`–`cam1` and `cam3`–`cam5`, higher in `cam2`–`cam5`, `cam6`–`cam7` and `cam6`–`cam8`, and 3 to 15 times the background's in every pair.
- **What the turn does and does not show.** On synthetic matches of a camera turned by 0.3°, the fit returns the turn, the background's residual falls to the noise put in, and a 3 px vertical disagreement of the near points reads 2.12 px after the turn (3/√2) and 0.33 px before it: a camera that is slightly off can hide a disagreement as well as feign one. Only the part of a turn that moves points across the epipolar lines is fixed by the fit. A shift of a camera's centre is not tested: its effect grows as the point comes nearer, so the background cannot tell it from the person's own disagreement. A whole fundamental matrix fitted to the far matches was tried and set aside: it lowers the background's residual to 0.24 to 0.39 px in six pairs, but it moves the baseline's direction by 4° to 17°, which far matches leave loose, and so says nothing of the person.

**Private scene A: frame offsets.** The 7 pairs and every frame again, with camera B read one and two frames earlier and later: 3,318 more evaluations. Inside the crop (rule a):

| Pair | Median of the per-frame medians (px) at offset −2, −1, 0, +1, +2 | Change against offset 0, per frame, at −2, −1, +1, +2: median (px) | Frames whose median is lower than at offset 0, at −2, −1, +1, +2 | Coverage inside at −2, −1, 0, +1, +2 |
| --- | --- | --- | --- | --- |
| `cam0`–`cam1` | 1.18, 0.74, 0.66, 1.04, 1.49 | +0.414, +0.033, +0.392, +0.802 | 16%, 41%, 13%, 8% | 18.5%, 18.9%, 19.0%, 18.6%, 17.9% |
| `cam1`–`cam4` | 4.34, 4.03, 3.58, 3.01, 3.23 | +0.804, +0.683, −0.146, −0.333 | 26%, 25%, 60%, 58% | 19.7%, 20.3%, 20.6%, 20.2%, 19.5% |
| `cam2`–`cam5` | 1.83, 1.48, 1.10, 1.15, 1.54 | +0.732, +0.347, +0.085, +0.417 | 7%, 13%, 34%, 21% | 18.5%, 18.9%, 19.1%, 18.8%, 18.2% |
| `cam3`–`cam4` | 4.40, 3.48, 2.25, 2.36, 2.38 | +1.277, +0.673, −0.127, +0.054 | 22%, 24%, 59%, 49% | 17.9%, 18.6%, 18.8%, 18.3%, 17.6% |
| `cam3`–`cam5` | 3.68, 2.66, 2.04, 2.06, 2.75 | +0.376, +0.184, −0.057, +0.008 | 29%, 31%, 59%, 50% | 17.2%, 17.7%, 18.0%, 17.7%, 17.2% |
| `cam6`–`cam7` | 3.70, 2.97, 2.49, 2.35, 2.82 | +0.392, +0.140, −0.138, −0.050 | 35%, 37%, 64%, 55% | 15.3%, 15.8%, 16.1%, 15.7%, 15.2% |
| `cam6`–`cam8` | 2.36, 1.65, 1.92, 2.80, 3.67 | +0.144, −0.098, +0.697, +1.574 | 42%, 60%, 10%, 6% | 14.6%, 15.1%, 15.3%, 15.0%, 14.5% |
| The 7 pairs together | 2.64, 2.05, 1.78, 1.94, 2.40 | +0.559, +0.192, +0.083, +0.367 | 25%, 33%, 43%, 35% | 17.4%, 17.9%, 18.2%, 17.8%, 17.2% |

- **No pair's residual has its low point more than one frame from offset 0.** Four pairs are lowest at 0, `cam1`–`cam4` and `cam6`–`cam7` at +1, and `cam6`–`cam8` at −1.
- **Where an offset helps, it helps a little.** Reading camera B one frame later lowers the per-frame median in 59% to 64% of the frames of `cam1`–`cam4`, `cam3`–`cam4`, `cam3`–`cam5` and `cam6`–`cam7`, by 0.06 to 0.15 px at the median; reading it one frame earlier does so in 60% of the frames of `cam6`–`cam8`, by 0.10 px. The largest gain is `cam1`–`cam4`'s, from 3.58 to 3.01 px at +1, a sixth of that pair's residual. In `cam0`–`cam1` and `cam2`–`cam5` every offset raises the residual in most frames.
- **Nothing else improves.** Coverage inside the crop is highest at offset 0 in every pair. D-22's filters keep 62.4% of the drawn matches inside the crop at offset 0, 60.0% and 56.6% at +1 and −1, and 54.0% and 50.5% at +2 and −2. Outside the crop, over the seven pairs together, the offsets change the per-frame median by under 0.002 px at the median.
- **How much grip the test has.** The control of one camera at two consecutive frames (the nine cameras, every tenth frame) gives the person's motion: inside the crop the admitted matches move 0.28 px per frame at the median and 4.2 px at the 90th percentile, where `kitchen`'s dynamic objects move 8.9 px. So an offset can move the median only a little, and it does move it, upward: over the seven pairs from 1.78 px to 1.94 and 2.05 px one frame off, and to 2.40 and 2.64 px two frames off.

So the cameras are in step to within a frame. A lag of a fraction of a frame between the two cameras is not excluded in five of the pairs, but it would account for a sixth of one pair's residual at most and a few percent of the others'. Timing is not what the disagreement is made of.

**Placeholder pixels: the rule.** On a pixel's 8-bit values, m = min(R, B) − G, how far the pixel leans to magenta. A pixel is a placeholder pixel when m ≥ 64. Each pixel is judged alone: no smoothing, no neighbourhood and no other frame enters. Two more rules bracket it: tight, m ≥ 128; loose, m ≥ 64, or m ≥ 32 with R − B < 32.

Why 64: the nine cameras show one scene, so a colour that only one camera has is not the scene's. Over all 120 frames, 2,630 of every million pixels of `cam0` have m ≥ 64, and 0.2 of every million pixels of the other eight cameras together (192 pixels in 904 million, never more than 12 in a frame). On `cam0` those pixels are magenta: their median colour is (R, G, B) = (206, 38, 219), half of them have R between 184 and 219, G between 25 and 54 and B between 199 and 230, and R is within 32 of B for 79% of them. Below 64 every camera holds a second, pink population, 1.6% to 2.1% of its pixels, around (R, G, B) = (240, 112, 158), with R above B by about 56 to 95; it is as large on each of the nine cameras, so it is taken as the scene's own colour and is not counted. On `cam0` alone a ramp of the magenta hue runs on below 64, at 75 to 114 pixels per million for every 8 levels of m down to m = 32, where the other cameras together have 20 per million at m 32 to 39 and under 2 from 48 up. These are probably the blended edges of the magenta regions, and the rule at 64 leaves them out. The loose rule takes them in: it adds 365 pixels per million on `cam0`, 14% more, and finds 20 to 52 per million on the other cameras, where the rule at 64 finds under 1.

**Placeholder pixels: the shares.** Per camera, over its 120 frames of 736 × 1280 pixels. The crop is rule a's; on `cam0` and `cam1` that is the tools' own rectangle, and rule b changes no cell by more than 0.1 pixels per million.

| Camera | Frames with a placeholder pixel | Placeholder pixels, all frames | Share of all pixels | Tight rule; loose rule | Per frame: least, median, most | Share of the crop's pixels | Placeholder pixels inside the crop |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `cam0` | 120 of 120 | 297,359 | 0.263% | 0.195%; 0.300% | 0.060%, 0.256%, 0.471% (frame 93) | 0.445% | 100% |
| `cam1` | 12 | 80 | 0.7 per million | 0; 52 per million | 0, 0, 12 pixels | 1.0 per million | 100% |
| `cam2` | 7 | 36 | 0.3 per million | 0; 20 per million | 0, 0, 8 pixels | 0.5 per million | 100% |
| `cam3` | 4 | 28 | 0.2 per million | 0; 25 per million | 0, 0, 12 pixels | 0.4 per million | 100% |
| `cam4` | 1 | 4 | under 0.1 per million | 0; 40 per million | 0, 0, 4 pixels | 0.1 per million | 100% |
| `cam5` | 1 | 4 | under 0.1 per million | 0; 30 per million | 0, 0, 4 pixels | 0.1 per million | 100% |
| `cam6` | 1 | 4 | under 0.1 per million | 0; 23 per million | 0, 0, 4 pixels | 0.1 per million | 100% |
| `cam7` | 1 | 4 | under 0.1 per million | 0; 22 per million | 0, 0, 4 pixels | 0.1 per million | 100% |
| `cam8` | 7 | 32 | 0.3 per million | 0; 20 per million | 0, 0, 8 pixels | 0.5 per million | 100% |

So the placeholder pixels are `cam0`'s: one pixel in 380 of the camera, one in 225 of its crop, in every frame. No other camera has more than 12 in a frame.

**Placeholder pixels: where they lie.** On `cam0` every one of the 297,359 lies inside the crop, and none within 102 px of its left, right or top side: they span x 198 to 501 and y 430 to 857, the crop's middle columns, between 17% and 59% of its height from the top. They move: 26,478 different pixels are a placeholder pixel in at least one frame (4.75% of the crop), and 280 in at least half of the frames. They are densest in the row of 32 px cells at y 512 to 544, x 320 to 480, where 15% to 26% of a cell's pixels, counted over all frames, are placeholder pixels. Over time the share dips to 0.06% at frame 91 and jumps to 0.449% at frame 92; its mean is 0.238% over frames 0 to 79 and 0.313% over frames 80 to 119, the notes' last third, where the person is closest (row T4.1).

The matcher admits nothing there. Over the 120 frames of `cam0`–`cam1`, 233,628 of `cam0`'s grid points fall on placeholder pixels, and 19 of them have an admitted match. So placeholder pixels enter neither the residuals above nor the points `roma_init` would make. They reach training only as pixels of `cam0`'s images, where no term of the loss can leave a pixel out today (section 2.3).

**Placeholder pixels: the sheet,** for the owner, in `<surface-runs>/ticket-4/private-scene-A/`. Green paint is a placeholder pixel, yellow a pixel only the loose rule adds, and the white box is rule a's crop on that camera.

- `placeholder_sheet_overview.jpg`: the nine cameras, one per column, at frames 0, 30, 60, 90 and 119, the frames of the notes' own sheets (row T4.1), and in the last row each camera at its frame with the most placeholder pixels (`cam0`: frame 93). Half size, where a painted pixel stands for any painted pixel of its 2 × 2 block.
- `placeholder_sheet_centre.png`: `cam0` at frames 0, 30, 60, 90, 119 and 93, full size, the image beside the painted image.
- `placeholder_sheet_zoom.png`: the same six frames, the region x 144 to 544, y 384 to 1136, which holds every pixel the loose rule finds in any frame with 32 px to spare, enlarged twice without interpolation, the image beside the painted image.
- `placeholder_sheet_pink.jpg`: the nine cameras at frame 60, the image above and, below it, the pink population painted blue, so the owner can say whether it is the scene's own colour.

On the six full-size panels the green pixels number exactly the placeholder pixels of the frame (2,352, 1,400, 3,336, 1,076, 2,518 and 4,440). The implementer has not seen the sheets or any image: the detections are judged by the numbers above, and **the owner's check of the sheet is pending**. What the owner is asked: whether the green regions are the magenta artifacts, and all of them; whether the yellow pixels belong to them; whether the blue population is the scene's own colour; and whether the white box holds the person on every camera.

**What the figures mean for H2's premise.** H2's premise is that the cameras' images disagree about the person. The figures bear it out, show a second disagreement beside it, and bound both from below only.

- **Against the floor.** On `kitchen` the admitted matches lie 0.02 to 0.04 px from epipolar agreement at the median and 0.08 to 0.12 px at the 90th percentile, at D-22's angles and at 15° alike. On private scene A the matches inside the person's crop lie 1.84 px and 6.61 px from it: 56 and 66 times `kitchen`'s dynamic objects at 15°, and 54 times at the median when both are counted in matcher cells. Per pair the median is 21 to 99 times that floor.
- **Against the scene's own background.** Not all of that is about the person. The background of private scene A reads 0.29 to 2.24 px, steady from frame to frame, and a turn of one camera of the pair by 0.24° to 0.68° brings it to 0.21 to 0.59 px: the manifest's cameras are not quite the cameras the generated images show. That is a disagreement of its own kind, which concerns the whole image and which no change to the person's surface could absorb. With it taken out, the person's matches still lie 0.87 to 3.84 px from agreement at the median and 2.6 to 12.5 px at the 90th percentile, 3 to 15 times the background's. They do so unevenly: the residual swings up to eightfold between frames and is two to four times higher in five pairs once the person is closest.
- **How large that is in the scene.** Reading the manifest's unit as a metre, as the notes do, 1 px of residual at the person's depth of 1.1 is 2 mm across the epipolar line. With the manifest's cameras the person's median is then 2 to 10 mm per pair and the 90th percentile 5 to 27 mm; after the turn, 2 to 8 mm and 5 to 25 mm. The shell sits 5 to 25 cm from the body (section 2.1). So where the matcher can match the person, the two images disagree by about a tenth of the shell's distance, or less.
- **What the residual cannot see.** It is a lower bound twice over. It reads only the part of a disagreement across the epipolar line, one image direction per pair. And it reads only the matches the matcher is certain of: 18% of the grid points of the crop, where two consecutive frames of one camera, which do agree, give 48.5%. So about three fifths of what can be matched on the person when two images agree cannot be matched between two cameras. `kitchen` at the same angle loses far less: 56.4% against 57.3% on its dynamic objects, 64.6% against 87.4% on its static pixels. Whether the cameras disagree more where the matcher gives up than where it matches, the probe cannot say; a disagreement the size of the shell's distance, if there is one, would be there.
- **Initialization already sees it.** D-22's filters turn away 38% of the matches drawn inside the crop, and none on `kitchen`: the init points come from the part of the person the two cameras agree on within 2 px.
- **Timing is not the cause.** No frame offset lowers a pair's residual by more than a sixth, and the cameras are in step to within a frame.
- **Placeholder pixels are a separate defect:** one camera, one pixel in 225 of its crop, in every frame, all on the person, and none of them matched. They are pixels known to be wrong, the case family E's survey looks for (section 3.6).

So the premise holds in kind: the cameras disagree about the person, well beyond the measure's floor and beyond the scene's own background, and more as the person comes closer. In size, what the probe can measure is millimetres to a few centimetres, short of the shell's 5 to 25 cm, on the fifth of the crop that can be matched at all. The probe does not test H2 itself, that the shell is how training reconciles the disagreement. Two things follow for the other sections. A depth estimated per camera and lifted with the manifest's cameras (section 2.7) inherits the fraction of a degree by which those cameras are off, before any disagreement about the person. And a technique that corrects each camera's pose (section 3.6) could remove the background's part of the residual, and not the person's.

**Runs.** `python` is the shared `.venv`'s, run with `PYTHONPATH=<worktree>/src:<worktree>/harness/src:<worktree>/scripts:<scratch>/ticket-4/env-init` and `OMP_NUM_THREADS`, `MKL_NUM_THREADS` and `OPENBLAS_NUM_THREADS` at the thread count given (the three chained GPU runs of 2026-11-14 were started before the third was required, and held 2.4 cores as measured). Scripts are in `<scratch>/ticket-4`, `<out>` is `<surface-runs>/ticket-4`, and `<gt>` is `$OPENFTGS_DATASETS/synth/ground_truth/kitchen`. An evaluation is one pair, frame and offset.

| Command | Threads; device | What it ran; time | Note |
| --- | --- | --- | --- |
| `python colour_stats.py <private-scene-A>/scene.json <out>/colour_stats_A.npz --workers 6` | 6 processes; CPU | 1,080 images; 10 s | the colour histograms behind the placeholder rule |
| `python placeholder.py <private-scene-A>/scene.json <out>/private-scene-A --workers 6` | 6 processes; CPU | 1,080 images; 7 s | the masks and counts of the three rules |
| `python crops.py <private-scene-A>/scene.json <out>/private-scene-A/crops.json` | 1; CPU | under 1 s | the rectangles of rules a and b, from the manifest's cameras |
| `python placeholder_sheet.py <private-scene-A>/scene.json <out>/private-scene-A --crops <out>/private-scene-A/crops.json` | 6; CPU | 7 s | the four sheets |
| `python placeholder_table.py <out>/private-scene-A` | 6; CPU | 2 s | the shares' table |
| `python check_epi.py` | 6; CPU | 2 s | the 32 checks on synthetic correspondences |
| `python match_residuals.py <m15-runs>/kitchen/scene.json <out>/checks --label kitchen --tag gt-matcher-cpu --gt <gt> --matcher gt --device cpu --pairs cam01:cam02,cam06:cam10 --frames 0:300:100 --offsets=-1,1 --offset-frames 0:300:100 --gt-error 0:300:100` | 6; CPU | 16 evaluations; 14 s | exact correspondences through the whole script |
| `python match_residuals.py <m15-runs>/kitchen/scene.json <out>/checks --label kitchen --tag roma-cpu --gt <gt> --device cpu --frames 150:151 --gt-error 150:151 --limit 1`; the same on private scene A with `--crops`, `--frames 60:61 --save-grids` | 6; CPU | 1 evaluation each; 35 s and 37 s | the matcher on the CPU, end to end, before the GPU was free: 0.029 px on `kitchen` |
| `python match_residuals.py <m15-runs>/kitchen/scene.json <out>/checks --label kitchen --tag roma-gpu-smoke --gt <gt> --device cuda:0 --frames 150:152 --offsets=-1,1 --offset-frames 150:151 --gt-error 150:151 --limit 4` | 6; GPU | 4 evaluations; 9 s | the same pair on the GPU: 0.029 px |
| `python match_residuals.py <m15-runs>/kitchen/scene.json <out>/kitchen --label kitchen --tag d22 --gt <gt> --offsets=-2,-1,1,2 --offset-frames 0:300:10 --gt-error 0:300:10 --device cuda:0` | 6; GPU | 5,434 evaluations (3,900 at offset 0); 40 min 38 s; 0.39 s of matching per evaluation; peak 4,494 MiB allocated | the floor, its offsets, and the matcher against the ground truth |
| `python match_residuals.py <m15-runs>/kitchen/scene.json <out>/kitchen --label kitchen --tag wide --gt <gt> --pairs cam06:cam10,cam05:cam09,cam03:cam07,cam08:cam12,cam12:cam16,cam02:cam06 --frames 0:300:10 --gt-error 0:300:10 --device cuda:0` | 6; GPU | 180 evaluations; 1 min 42 s | the six pairs about 15° apart, beyond D-22 |
| `python match_residuals.py <private-scene-A>/scene.json <out>/private-scene-A --label private-scene-A --tag d22 --crops <out>/private-scene-A/crops.json --offsets=-2,-1,1,2 --save-grids --device cuda:0` | 4, and 2 writing; GPU | 4,158 evaluations (840 at offset 0); 25 min 40 s; 0.33 s of matching per evaluation; peak 4,483 MiB allocated | private scene A, every frame and offset; the offset-0 matches kept as grids (6.0 GB) |
| `python match_same_camera.py <private-scene-A>/scene.json <out>/private-scene-A/same_camera.json --label private-scene-A --crops <out>/private-scene-A/crops.json --frames 0:119:10 --device cuda:0` | 6; GPU | 108 evaluations; 46 s | one camera, two consecutive frames |
| `python match_same_camera.py <m15-runs>/kitchen/scene.json <out>/kitchen/same_camera.json --label kitchen --gt <gt> --frames 0:299:10 --device cuda:0` | 6; GPU | 570 evaluations; 3 min 58 s | the same on `kitchen` |
| `python summarize.py <out>/kitchen/d22`; the same for `<out>/kitchen/wide` and `<out>/private-scene-A/d22`, the last also with `--split rule_b`; `python doc_tables.py <out>` | 6; CPU | seconds | the tables |
| `python analyze_grids.py <out>/private-scene-A --tag d22` | 2; CPU | 47 s | the matches on placeholder pixels |
| `python analyze_depth.py <private-scene-A>/scene.json <out>/private-scene-A --tag d22 --workers 4` | 4 processes; CPU | 27 s | the split by depth |
| `python analyze_rotfit.py <private-scene-A>/scene.json <out>/private-scene-A --tag d22`; `python analyze_refit.py` with the same arguments | 6; CPU | 54 s each | the turn fitted to the background; the whole fundamental matrix, set aside |

The GPU ran for 73 minutes in all. The matcher is the vendored RoMa at 77f8d68 with `use_custom_corr` off (fused-local-corr is not installed), as `roma_provenance` records it in each run's `meta.json`. No output file holds private scene A's location or name: the scene is named by `--label`.

### 2.6 Probe: the shell's measures

*Ticket #5's section.* On private scene A's checkpoints, the notes' three measures (the share of opacity farther than 5 cm from every init point, the share of primitives whose duration is under two frames, the distribution of base opacity) and the fixed sheet of novel views from the side, with its render request; on openftgs-synth's M17 checkpoints, the geometry residual and the error of expected depth from `scripts/gt_depth.py` (Q-226 to Q-231).

These are the figures a later training test is compared with (section 1, "No camera is ground truth"). All were measured on 2026-11-14 with the integration branch's code at 841f1bc, whose `src/` and `scripts/` are `main`'s at 93eb58e: Python 3.14, torch 2.15.0+cu132, gsplat 1.5.3, SciPy 1.18.1, NumPy 2.5.4, four CPU threads, one RTX 4090. "The commands", at the section's end, lists every run. No image of private scene A was opened, and no render or sheet of it was looked at: scripts read tensors and JSON, and wrote numbers and files. `<notes>` below is the folder of the owner's notes on private scene A, which holds their runs; like `<private-scene-A>`, it is written nowhere.

**The notes' three measures** (rows 513, 514). The notes' scorer reads two files per run: the exchange model `ftgs export` writes from the checkpoint (§2.2 `ftgs-if`: `position` is μx, `opacity` is E3's base opacity σ, `duration` is E4's duration s in seconds), and the init points file the run trained from.

- **The far-5 cm share.** A primitive is far when its position is more than 0.05 world units, which the notes read as 5 cm, from every init point of that file, all init frames taken together. The measure is the share of the summed base opacity that the far primitives hold. The scorer also keeps the share of primitives that are far.
- **The under-two-frames share.** The share of primitives whose duration is under 2/24 s, two frame intervals at the scene's 24 fps. The duration is the standard deviation of E4's Gaussian in time, not a length of life: a primitive at that bound still has σt ≥ 0.61 over four frame intervals.
- **The distribution of base opacity.** Its mean, and the shares of primitives under 0.05 and over 0.5. This section adds the quartiles and the share under 0.005, D-14's dead threshold.

Every checkpoint of the scene has the spatial scale S = 0.416 (D-27), and the nine cameras are 1.05 from the point where their optical axes meet (rows 515, 517). So 5 cm is 0.12·S, six times the δ = 0.02·S that the geometry residual uses on openftgs-synth (Q-229).

A scratch script computes the three as the scorer does, without the exported file: `load_checkpoint`, the model on the CPU, `FreeTimeGaussians.to_exchange()`, `read_init_points`, an exact nearest-neighbour search (SciPy's `cKDTree`, k = 1), then the scorer's expressions, written anew (row 517).

**The checkpoints the owner still has: 45**, all from 30,000-iteration runs of openftgs 0.1.0 (row 513).

- The final checkpoint of each of the notes' 37 scored runs: 30 keep the centre camera as the test view, and 7 the camera left of it. The notes' three failed or stopped runs left none.
- Five periodic checkpoints, at iterations 5,000 to 25,000, of one of those runs.
- The three models the notes trained on all nine cameras.

A fourth nine-camera model, the default configuration from a stride-1 init, which the notes cite for the artifacts between cameras, is not among them. It is a link from `<notes>` to a place this ticket was not given, so it was not opened ("What is open", below).

**Against the notes' own figures.** The 42 checkpoints of the runs that keep one camera as the test view can be checked, and all 42 agree. The eight figures the scorer writes per checkpoint (N, the two far-5 cm shares, the under-two-frames share, the median duration and the three opacity figures) equal the notes' score files in every digit, though those were written with torch 2.14.1 and these with torch 2.15.0. The 37 rows of the notes' table of all runs are the same figures, rounded. The three nine-camera models have no figure in the notes, so they cannot be checked that way; they are measured by the same code, against the init points file the note beside them names.

**The measures, per checkpoint.** A run is named as the notes name it. "Init points" says how `ftgs init roma` was run (D-21's stride; 1,000 points per init frame, 120,000 in all, unless the cell says otherwise) and how the notes then edited the file, which keeps its positions. The two far-5 cm shares are taken against that file. "Differs" lists what the run's training configuration changes from the default. σ is the base opacity, and its quartiles are the 25th, 50th and 75th percentiles.

| Checkpoint | Test view | Init points | Differs | N | Far-5 cm share of opacity | Far-5 cm share of primitives | Under-two-frames share | Median duration, frames | Mean σ | σ < 0.05 | σ > 0.5 | σ quartiles | σ < 0.005 | Equal to the notes' figures |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `stride4` | none | stride 4 | nothing | 3,000,000 | 0.209 | 0.225 | 0.456 | 2.25 | 0.337 | 0.327 | 0.294 | 0.030 / 0.133 / 0.686 | 0.103 | no figure in the notes |
| `stride4_zero_velocity` | none | stride 4, velocities zeroed | nothing | 3,000,000 | 0.213 | 0.225 | 0.453 | 2.26 | 0.330 | 0.327 | 0.285 | 0.030 / 0.130 / 0.646 | 0.100 | no figure in the notes |
| `stride4_zero_velocity_reg0.1` | none | stride 4, velocities zeroed | `loss.lambda_reg` 0.1 | 3,000,000 | 0.154 | 0.183 | 0.378 | 2.64 | 0.115 | 0.540 | 0.058 | 0.012 / 0.043 / 0.116 | 0.171 | no figure in the notes |
| `A_default_s0` | centre | stride 1 | nothing | 3,000,000 | 0.259 | 0.273 | 0.517 | 1.92 | 0.360 | 0.285 | 0.315 | 0.040 / 0.160 / 0.755 | 0.074 | yes |
| `A_default_s1` | centre | stride 1 | seed 1 | 3,000,000 | 0.257 | 0.274 | 0.519 | 1.91 | 0.358 | 0.286 | 0.313 | 0.040 / 0.159 / 0.751 | 0.074 | yes |
| `c3_s4` | centre | stride 4 | nothing | 3,000,000 | 0.213 | 0.226 | 0.447 | 2.29 | 0.360 | 0.301 | 0.317 | 0.035 / 0.154 / 0.783 | 0.092 | yes |
| `c3_s4_reg0.1` | centre | stride 4 | `loss.lambda_reg` 0.1 | 3,000,000 | 0.161 | 0.186 | 0.376 | 2.65 | 0.123 | 0.515 | 0.065 | 0.014 / 0.047 / 0.126 | 0.156 | yes |
| `c3_s4v0` | centre | stride 4, velocities zeroed | nothing | 3,000,000 | 0.230 | 0.241 | 0.452 | 2.27 | 0.353 | 0.301 | 0.309 | 0.036 / 0.151 / 0.745 | 0.090 | yes |
| `c3_s4v0_reg0.1` | centre | stride 4, velocities zeroed | `loss.lambda_reg` 0.1 | 3,000,000 | 0.164 | 0.192 | 0.379 | 2.63 | 0.124 | 0.513 | 0.065 | 0.014 / 0.048 / 0.126 | 0.153 | yes |
| `sh0` | centre | stride 1 | `sh_degree` 0 | 3,000,000 | 0.264 | 0.268 | 0.508 | 1.96 | 0.326 | 0.315 | 0.278 | 0.034 / 0.129 / 0.616 | 0.086 | yes |
| `sh1` | centre | stride 1 | `sh_degree` 1 | 3,000,000 | 0.266 | 0.272 | 0.509 | 1.95 | 0.337 | 0.310 | 0.291 | 0.035 / 0.136 / 0.671 | 0.084 | yes |
| `B_usercfg_s0` | centre | stride 1 | `growth.n_max` 1,500,000; `reloc.dead_opacity` 0.01; `growth.grad_threshold` 0.0004, growth from 100 to 30,000 | 1,500,000 | 0.229 | 0.255 | 0.480 | 2.10 | 0.414 | 0.199 | 0.366 | 0.068 / 0.244 / 0.868 | 0.030 | yes |
| `B_usercfg_s1` | centre | stride 1 | `growth.n_max` 1,500,000; `reloc.dead_opacity` 0.01; `growth.grad_threshold` 0.0004, growth from 100 to 30,000; seed 1 | 1,500,000 | 0.228 | 0.254 | 0.483 | 2.08 | 0.412 | 0.201 | 0.365 | 0.068 / 0.242 / 0.863 | 0.030 | yes |
| `b15_dead0.05` | centre | stride 1 | `growth.n_max` 1,500,000; `reloc.dead_opacity` 0.05 | 1,500,000 | 0.304 | 0.324 | 0.158 | 5.81 | 0.225 | 0.377 | 0.164 | 0.021 / 0.095 / 0.298 | 0.112 | yes |
| `b15_durlr0.2` | centre | stride 1 | `growth.n_max` 1,500,000; `optim.log_durations_lr` 0.001 | 1,500,000 | 0.368 | 0.366 | 0.798 | 1.45 | 0.500 | 0.185 | 0.467 | 0.081 / 0.405 / 0.992 | 0.039 | yes |
| `b15_durlr4` | centre | stride 1 | `growth.n_max` 1,500,000; `optim.log_durations_lr` 0.02 | 1,500,000 | 0.154 | 0.164 | 0.649 | 1.19 | 0.346 | 0.304 | 0.303 | 0.034 / 0.154 / 0.689 | 0.096 | yes |
| `b15_init3k` | centre | stride 1, 3,000 points per frame | `growth.n_max` 1,500,000 | 1,500,000 | 0.179 | 0.199 | 0.496 | 2.01 | 0.371 | 0.246 | 0.322 | 0.051 / 0.188 / 0.747 | 0.055 | yes |
| `b15_init_d4` | centre | stride 1, durations × 4 | `growth.n_max` 1,500,000 | 1,500,000 | 0.191 | 0.219 | 0.380 | 2.65 | 0.368 | 0.258 | 0.320 | 0.048 / 0.179 / 0.757 | 0.067 | yes |
| `b15_init_s4` | centre | stride 4 | `growth.n_max` 1,500,000 | 1,500,000 | 0.203 | 0.219 | 0.388 | 2.64 | 0.380 | 0.242 | 0.331 | 0.052 / 0.194 / 0.792 | 0.061 | yes |
| `b15_init_s8` | centre | stride 8 | `growth.n_max` 1,500,000 | 1,500,000 | 0.204 | 0.219 | 0.351 | 2.89 | 0.388 | 0.249 | 0.341 | 0.050 / 0.196 / 0.842 | 0.069 | yes |
| `b15_init_v0` | centre | stride 1, velocities zeroed | `growth.n_max` 1,500,000 | 1,500,000 | 0.241 | 0.264 | 0.463 | 2.18 | 0.387 | 0.224 | 0.337 | 0.058 / 0.205 / 0.799 | 0.045 | yes |
| `b15_noreloc` | centre | stride 1 | `growth.n_max` 1,500,000; relocation off, `reloc.prune_opacity` 0.005 | 1,258,673 | 0.233 | 0.293 | 0.390 | 2.74 | 0.178 | 0.674 | 0.160 | 0.000 / 0.000 / 0.147 | 0.605 | yes |
| `b15_reg0.1` | centre | stride 1 | `growth.n_max` 1,500,000; `loss.lambda_reg` 0.1 | 1,500,000 | 0.174 | 0.211 | 0.395 | 2.49 | 0.164 | 0.416 | 0.098 | 0.024 / 0.068 / 0.182 | 0.106 | yes |
| `b15_reg0.3` | centre | stride 1 | `growth.n_max` 1,500,000; `loss.lambda_reg` 0.3 | 1,500,000 | 0.167 | 0.203 | 0.253 | 3.39 | 0.058 | 0.693 | 0.014 | 0.003 / 0.023 / 0.063 | 0.305 | yes |
| `b15_scale3` | centre | stride 1 | `growth.n_max` 1,500,000; `scene_scale_factor` 3.3 | 1,500,000 | 0.276 | 0.297 | 0.488 | 2.06 | 0.321 | 0.316 | 0.274 | 0.033 / 0.134 / 0.584 | 0.080 | yes |
| `b15_vgg` | centre | stride 1 | `growth.n_max` 1,500,000; `loss.perc_net` `vgg16_oxford` | 1,500,000 | 0.189 | 0.213 | 0.526 | 1.88 | 0.336 | 0.246 | 0.280 | 0.051 / 0.169 / 0.589 | 0.063 | yes |
| `f_s4_reg0.1` | centre | stride 4 | `growth.n_max` 1,500,000; `loss.lambda_reg` 0.1 | 1,500,000 | 0.151 | 0.182 | 0.329 | 2.92 | 0.146 | 0.449 | 0.080 | 0.020 / 0.060 / 0.156 | 0.121 | yes |
| `f_s4_reg0.3` | centre | stride 4 | `growth.n_max` 1,500,000; `loss.lambda_reg` 0.3 | 1,500,000 | 0.196 | 0.259 | 0.108 | 5.30 | 0.036 | 0.822 | 0.009 | 0.000 / 0.004 / 0.033 | 0.509 | yes |
| `f_s4v0` | centre | stride 4, velocities zeroed | `growth.n_max` 1,500,000 | 1,500,000 | 0.198 | 0.216 | 0.384 | 2.66 | 0.378 | 0.241 | 0.329 | 0.053 / 0.193 / 0.785 | 0.060 | yes |
| `f_s4v0_reg0.1` | centre | stride 4, velocities zeroed | `growth.n_max` 1,500,000; `loss.lambda_reg` 0.1 | 1,500,000 | 0.156 | 0.188 | 0.338 | 2.88 | 0.148 | 0.444 | 0.083 | 0.021 / 0.061 / 0.159 | 0.118 | yes |
| `f_v0_reg0.1` | centre | stride 1, velocities zeroed | `growth.n_max` 1,500,000; `loss.lambda_reg` 0.1 | 1,500,000 | 0.181 | 0.221 | 0.396 | 2.50 | 0.167 | 0.410 | 0.100 | 0.025 / 0.069 / 0.184 | 0.100 | yes |
| `n1500k` | centre | stride 1 | `growth.n_max` 1,500,000 | 1,500,000 | 0.235 | 0.257 | 0.477 | 2.10 | 0.381 | 0.235 | 0.332 | 0.055 / 0.197 / 0.785 | 0.050 | yes |
| `n750k` | centre | stride 1 | `growth.n_max` 750,000 | 750,000 | 0.208 | 0.241 | 0.437 | 2.26 | 0.406 | 0.197 | 0.356 | 0.069 / 0.240 / 0.830 | 0.037 | yes |
| `c3_s4v0_reg0.1`, iteration 5,000 | centre | stride 4, velocities zeroed | `loss.lambda_reg` 0.1 | 1,357,226 | 0.275 | 0.242 | 0.200 | 4.05 | 0.185 | 0.457 | 0.130 | 0.020 / 0.060 / 0.211 | 0.000 | yes |
| `c3_s4v0_reg0.1`, iteration 10,000 | centre | stride 4, velocities zeroed | `loss.lambda_reg` 0.1 | 2,730,872 | 0.231 | 0.227 | 0.340 | 2.92 | 0.137 | 0.511 | 0.078 | 0.017 / 0.048 / 0.145 | 0.000 | yes |
| `c3_s4v0_reg0.1`, iteration 15,000 | centre | stride 4, velocities zeroed | `loss.lambda_reg` 0.1 | 3,000,000 | 0.181 | 0.196 | 0.379 | 2.59 | 0.112 | 0.523 | 0.048 | 0.017 / 0.046 / 0.122 | 0.000 | yes |
| `c3_s4v0_reg0.1`, iteration 20,000 | centre | stride 4, velocities zeroed | `loss.lambda_reg` 0.1 | 3,000,000 | 0.173 | 0.191 | 0.376 | 2.61 | 0.114 | 0.489 | 0.047 | 0.021 / 0.052 / 0.126 | 0.000 | yes |
| `c3_s4v0_reg0.1`, iteration 25,000 | centre | stride 4, velocities zeroed | `loss.lambda_reg` 0.1 | 3,000,000 | 0.168 | 0.192 | 0.375 | 2.64 | 0.119 | 0.466 | 0.050 | 0.023 / 0.056 / 0.131 | 0.000 | yes |
| `h1_base` | left of centre | stride 1 | `growth.n_max` 1,500,000 | 1,500,000 | 0.237 | 0.260 | 0.482 | 2.08 | 0.378 | 0.240 | 0.329 | 0.053 / 0.192 / 0.778 | 0.051 | yes |
| `h1_init_s4` | left of centre | stride 4 | `growth.n_max` 1,500,000 | 1,500,000 | 0.208 | 0.222 | 0.379 | 2.68 | 0.374 | 0.250 | 0.325 | 0.050 / 0.186 / 0.772 | 0.065 | yes |
| `h1_init_s4_seed1` | left of centre | stride 4 | `growth.n_max` 1,500,000; seed 1 | 1,500,000 | 0.207 | 0.222 | 0.383 | 2.67 | 0.370 | 0.250 | 0.321 | 0.050 / 0.184 / 0.757 | 0.063 | yes |
| `h1_s4_reg0.1` | left of centre | stride 4 | `growth.n_max` 1,500,000; `loss.lambda_reg` 0.1 | 1,500,000 | 0.156 | 0.187 | 0.331 | 2.92 | 0.145 | 0.452 | 0.080 | 0.020 / 0.059 / 0.154 | 0.123 | yes |
| `h1_s4v0` | left of centre | stride 4, velocities zeroed | `growth.n_max` 1,500,000 | 1,500,000 | 0.208 | 0.223 | 0.388 | 2.64 | 0.371 | 0.247 | 0.322 | 0.051 / 0.186 / 0.759 | 0.062 | yes |
| `h1_s4v0_reg0.1` | left of centre | stride 4, velocities zeroed | `growth.n_max` 1,500,000; `loss.lambda_reg` 0.1 | 1,500,000 | 0.156 | 0.187 | 0.337 | 2.87 | 0.145 | 0.451 | 0.081 | 0.020 / 0.060 / 0.155 | 0.122 | yes |
| `h1_s4v0_seed1` | left of centre | stride 4, velocities zeroed | `growth.n_max` 1,500,000; seed 1 | 1,500,000 | 0.199 | 0.217 | 0.385 | 2.65 | 0.369 | 0.248 | 0.319 | 0.051 / 0.183 / 0.749 | 0.061 | yes |

**What the measures contain.** This ticket's reading, on seven of the checkpoints (`<surface-runs>/ticket-5/shell_supplement.json`); the notes make none of these splits. The init points fall into two groups by depth along the centre camera's axis: in each of the five files checked, 52 to 58 % lie nearer than 2.0, about 1 % between 1.5 and 2.3, and the rest farther, to 14. The person is taken to be in the nearer group, at 0.7 to 1.5: that is where the init points that project inside the notes' crop of the centre camera lie, and no image was seen to confirm it. "The near side" below is the primitives whose position is nearer than 2.0, and "the far side", or the background, the others. The four init points files named in the header are those of the runs with the centre camera as the test view, and the nine-camera models' file.

| Checkpoint | Far-5 cm share against its own init | Against the stride-1 file | Against the stride-4 file | Against the nine-camera stride-4 file | Against the file with 3,000 points per frame | Share of the opacity on the near side | Far-5 cm share within the near side | Far-5 cm share within the far side | The far-5 cm share's parts: near side + far side |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `A_default_s0` | 0.259 | 0.259 | 0.269 | 0.264 | 0.231 | 0.749 | 0.128 | 0.649 | 0.096 + 0.163 |
| `A_default_s1` | 0.257 | 0.257 | 0.265 | 0.256 | 0.228 | 0.755 | 0.131 | 0.647 | 0.099 + 0.159 |
| `n1500k` | 0.235 | 0.235 | 0.243 | 0.236 | 0.209 | 0.763 | 0.097 | 0.677 | 0.074 + 0.161 |
| `c3_s4` | 0.213 | 0.200 | 0.213 | 0.206 | 0.176 | 0.785 | 0.124 | 0.538 | 0.097 + 0.116 |
| `b15_init_s4` | 0.203 | 0.185 | 0.203 | 0.194 | 0.160 | 0.784 | 0.105 | 0.557 | 0.082 + 0.120 |
| `c3_s4_reg0.1` | 0.161 | 0.150 | 0.161 | 0.155 | 0.129 | 0.861 | 0.083 | 0.647 | 0.071 + 0.090 |
| `stride4` | 0.209 | 0.200 | 0.220 | 0.209 | 0.174 | 0.784 | 0.122 | 0.528 | 0.095 + 0.114 |

- **The far-5 cm share depends on the init points it is taken against.** Against another 120,000-point file of the scene it moves by up to 0.02, and against the file with three times the points it is 0.03 to 0.04 lower: it is a share against a sample of the surface, and it falls as the sample grows. The gain the notes found for a stride-4 init holds against any one file: from `n1500k` to `b15_init_s4` the share goes from 0.235 to 0.203 against each run's own init, from 0.235 to 0.185 against the stride-1 file, and from 0.243 to 0.203 against the stride-4 file.
- **More than half of it is the background.** The far side holds 14 to 25 % of the opacity, and 53 to 68 % of that opacity is far from every init point, against 8 to 13 % on the near side. So 54 to 68 % of the far-5 cm share comes from the far side, where the init points are thin, and the rest from the side the person is on. The near side's own share is 0.128 for the default configuration (0.131 with seed 1), 0.124 with the stride-4 init (`c3_s4`) and 0.122 for the nine-camera model `stride4`.
- **Pooling the init frames was checked once, without a clear answer.** The person comes about 0.5 nearer over the clip, so the init points of all frames together fill the volume the surface sweeps, and a primitive off the surface at its own time can lie within 5 cm of a point from another time. Comparing each primitive only with the init points within 6 frames of its time centre raises the near side's share of the default checkpoint from 0.128 to 0.189. Against the file with three times the points the same window gives 0.161, so much of the rise is the thinner sample, and the check does not separate the two (`<surface-runs>/ticket-5/shell_windowed.json`).

On the near side, the primitives far from every init point against those within 5 cm of one:

| Checkpoint | Far from init: share of the near side's primitives | Their distance to the nearest init point, quartiles, cm | Share of their opacity within 25 cm | Median duration (quartiles), frames: far from init | Median duration (quartiles), frames: near init | Under two frames: far from init | Under two frames: near init | Median σ: far from init | Median σ: near init | Under two frames, on the far side |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `A_default_s0` | 0.140 | 7.4 / 17.0 / 37.6 | 0.64 | 3.90 (1.43 to 11.47) | 1.43 (0.91 to 2.22) | 0.332 | 0.698 | 0.122 | 0.177 | 0.162 |
| `A_default_s1` | 0.144 | 6.9 / 14.8 / 33.8 | 0.67 | 3.62 (1.39 to 11.02) | 1.42 (0.91 to 2.21) | 0.346 | 0.699 | 0.121 | 0.177 | 0.155 |
| `n1500k` | 0.110 | 6.8 / 14.9 / 34.0 | 0.68 | 4.41 (1.75 to 13.83) | 1.62 (1.06 to 2.47) | 0.286 | 0.633 | 0.148 | 0.223 | 0.147 |
| `c3_s4` | 0.127 | 6.7 / 13.4 / 36.7 | 0.65 | 5.24 (1.41 to 17.27) | 1.63 (1.01 to 2.59) | 0.316 | 0.618 | 0.135 | 0.174 | 0.055 |
| `b15_init_s4` | 0.107 | 6.3 / 9.1 / 28.6 | 0.78 | 5.04 (1.44 to 19.14) | 1.89 (1.22 to 3.00) | 0.329 | 0.534 | 0.185 | 0.225 | 0.037 |
| `c3_s4_reg0.1` | 0.087 | 6.2 / 8.9 / 21.4 | 0.84 | 4.82 (1.39 to 30.18) | 2.08 (1.33 to 3.31) | 0.335 | 0.478 | 0.047 | 0.049 | 0.006 |
| `stride4` | 0.130 | 6.7 / 10.8 / 27.2 | 0.75 | 4.21 (1.03 to 16.17) | 1.61 (1.00 to 2.59) | 0.379 | 0.622 | 0.104 | 0.148 | 0.048 |

- **The far primitives are not the short-lived ones.** On the near side of the default checkpoint, the primitives far from every init point have a median duration of 3.9 frames, and 33 % of them are under two frames; those within 5 cm of an init point have 1.4 frames and 70 %. Their base opacities differ less (medians 0.12 and 0.18). So the two shares count different primitives. The far-5 cm share counts longer-lived primitives away from the triangulated points, more than half of them by opacity in the background. The under-two-frames share counts short-lived primitives close to the triangulated points, on the near side: 65 % of the near side's primitives are under two frames, against 16 % of the far side's. Across the 30 runs with the centre camera as the test view, the rank correlation of the two shares is 0.44.
- **The notes' two medians are the whole model's.** The notes give the shell a median opacity of 0.16 and a median lifetime of two frames (section 2.1). Those are the medians over all 3,000,000 primitives of the default checkpoints: 0.160 and 1.92 frames, and 0.159 and 1.91 with seed 1. Over the primitives far from every init point they are 0.135 and 6.2 frames. The script behind the notes' sentence is not among the kept tools (row 514), so the population it took cannot be checked. The far primitives' distances to the nearest init point have quartiles of 8.5, 18 and 37 cm, wider than the notes' 5 to 25 cm.
- **Between seeds.** Four pairs of runs differ only by seed. The far-5 cm share differs by 0.001 to 0.002 in three pairs and by 0.010 in the fourth, the under-two-frames share by 0.002 to 0.004, and the mean base opacity by 0.001 to 0.003.
- **Against the notes' test-view scores.** Across the 30 runs with the centre camera as the test view, a lower far-5 cm share goes with a higher PSNR on that camera's crop (Spearman's ρ = −0.72) and a lower ꟻLIP (ρ = 0.65), and so does a lower under-two-frames share (−0.60 and 0.69). Neither goes with the notes' sharpness ratio (0.07 and 0.13). Each far-5 cm share is against its run's own init, and the runs differ by configuration, not by seed: the measures moved with the scores under the notes' changes, which is less than saying that lowering them raises a score.
- **Over training.** In the run with periodic checkpoints (`loss.lambda_reg` 0.1), the far-5 cm share falls from 0.275 at iteration 5,000 to 0.164 at 30,000. The under-two-frames share rises from 0.200 to 0.340 at iteration 10,000, and stays near 0.38 from 15,000 on.
- **Dead primitives at the end.** At iteration 30,000, 7 % of the default checkpoints' primitives are under D-14's dead threshold of 0.005; 9 to 10 % for the stride-4 runs at the default cap, and 15 to 17 % for those that also set `loss.lambda_reg` 0.1. At the periodic checkpoints up to iteration 25,000 the share is at most 0.02 %, and that run's final checkpoint has 15 %. Relocation stops at iteration 25,000 (`reloc.stop`), and the primitives that fall under the threshold after it stay where they are.

**The sheet** (row 517). One render request, `<surface-runs>/ticket-5/side_request.json` (39 items; SHA-256 `1b88fd859d64028c9ff40a4624a6bce76edb0be95b8f4129ccfbdc5933d98821`), fixes the cameras and the times. It holds cameras and timestamps and no path, and since the scene manifest and its two copies with a test view share cameras and timestamps, it serves every checkpoint of the scene: `ftgs render CHECKPOINT --request <surface-runs>/ticket-5/side_request.json --out RENDERS.safetensors`.

- **The times:** frames 0, 60 and 119 of the 120 (0 s, 2.5 s and 4.958 s), the three the notes' novel views use.
- **The side poses,** by a rule that reads the scene manifest alone (row 515). P is the point nearest to the nine optical axes, which pass it within 2·10⁻⁷; seen from P, the four cameras beside, above and below the centre camera are 15° from it and the four corner cameras 21.1°. The centre camera is turned rigidly about the axis through P that is parallel to its image's vertical axis, by a yaw of −90°, −60°, −45°, −30°, 0°, 30°, 45°, 60° and 90°, negative toward the left. It keeps its distance to P, its K and its image size, and looks at P. Yaw 0° is the centre camera itself, and yaw ∓15° gives the cameras left and right of it to 10⁻⁷, so 30° is twice as far round as they are. The ids are `sL90`, `sL60`, `sL45`, `sL30`, `sC00`, `sR30`, `sR45`, `sR60` and `sR90`, followed by the frame.
- **The notes' four poses between cameras,** kept so that the sheet joins on to the notes' own pictures: halfway from the centre camera to the camera left of it, to the one above it and to a corner one, and 40 % beyond the camera right of it, with the centre interpolated linearly and the rotation along the shortest arc. Their items are copied from the notes' request (ids `nm01`, `nm03`, `nm05`, `nx02`), after a check against that rule to 10⁻⁹.
- **The rest of the request:** a black background and the model's full SH degree, as in the notes' requests.
- **The sheet's layout:** one JPEG per checkpoint. The 13 poses are the columns, in the request's order, and the three frames the rows. A tile is the render clamped to [0, 1] and scaled by 0.4, with its id; a strip above the tiles carries the checkpoint's name.

Seven checkpoints have a sheet, `<surface-runs>/ticket-5/sheets/side_<name>.jpg`:

- the three nine-camera models, `stride4`, `stride4_zero_velocity` and `stride4_zero_velocity_reg0.1` (files `side_nine-<name>.jpg`), where every pose but `sC00` is a novel view;
- `A_default_s0` and `A_default_s1`, the default configuration at two seeds, which show how much two seeds differ to the eye;
- `c3_s4`, the notes' recommended configuration with the centre camera as the test view, where `sC00` is the test view;
- `h1_init_s4`, the same init with the camera left of the centre as the test view, at a cap of 1,500,000.

Any other kept checkpoint is one `ftgs render` away. What could be checked without looking:

- The request parses with `read_render_request`.
- The model `stride4`'s near-side primitives, weighted by σ_eff at the item's time (E3, E4) and moved by E1, were projected into each item's camera. Their median lands between 0.23 and 0.75 of the frame's width and between 0.58 and 0.65 of its height in every side pose, and 61 to 89 % of their weight falls inside the frame, against 62 to 81 % in the centre camera's own pose, where the frame's lower edge already cuts the person (`<surface-runs>/ticket-5/side_request_framing.json`).
- Every rendered value is finite. The share of a tile's pixels with alpha ≥ 0.5 is 0.96 to 1.00 in the poses between cameras, 1.00 in the centre pose, and 0.88 to 1.00, 0.69 to 0.96, 0.52 to 1.00 and 0.37 to 0.95 at yaw ±30°, ±45°, ±60° and ±90° (`side_<name>.coverage.json` beside each sheet).
- One model stands out in those numbers. In `stride4`, the tiles at yaw +60° are fully covered at 2.5 s and 4.958 s, and the last is nearly black (mean of the clamped render 0.09, against 0.16 to 0.40 in every other tile of the seven sheets); its tile at yaw −90° and 2.5 s is 95 % covered, against 50 to 64 % in the other six. Something opaque stands between those poses and the person in that model; the other six sheets' numbers show nothing like it.

The sheets are for the owner to look at; this ticket reports no judgement of them.

**On openftgs-synth: the geometry residual and the error of expected depth** (rows 516, 518), from `scripts/gt_depth.py` on the gsplat backend, over every test frame of the scene's test view, with δ = 0.02·S (Q-226 to Q-231). The checkpoints measured, and why:

- **`default-395` of each scene** (steps 35 to 37 of `docs/prd/validation-runs.md`): the default training configuration at spec revision 549o, with ꟻLIP as the perceptual network. It is what a later test's run "with the field off" repeats, and none of the three had a `GT_DEPTH.json`.
- **`default` of each scene** (steps 2, 12 and 16, trained before 549o with LPIPS): openftgs issue 328 left their figures beside the runs on 2026-10-22. Run again here, each file equals the stored one byte for byte, so the script at this commit reproduces that record.
- **The other trainings of the default configuration before 549o:** kitchen's `seed1` (step 5; issue 328's stored file, not run again) and `default-326` (step 20), studio's `default-325` and `default-341` (steps 23 and 27). They give the spread between trainings of one configuration.
- **Not measured:** `tau4e-4`, `reg15k` and their periodic checkpoints, which are other configurations; issue 328's files for them stay beside the runs.

The geometry residual r is a primitive's camera-space depth minus the ground-truth depth at the pixel its centre projects to, over the primitives the render reports visible with σ_eff ≥ 1/255, weighted by σ_eff (Q-230). A share "in front" is the weight with r < −kδ, and "behind" with r > kδ.

| Scene | Checkpoint | Figures from | δ, m | In front at ½δ / δ / 2δ | Behind at ½δ / δ / 2δ | Static pixels at δ: in front / behind | Dynamic pixels at δ: in front / behind | Weighted median / 90th percentile of the absolute residual, m | Weighted median of the absolute residual, static / dynamic, m |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `kitchen` | `default-395` (step 35) | this ticket | 0.0319 | 0.067 / 0.040 / 0.022 | 0.362 / 0.294 / 0.227 | 0.043 / 0.257 | 0.016 / 0.619 | 0.0097 / 0.386 | 0.0078 / 0.0764 |
| `kitchen` | `default` (step 2) | this ticket; equal to issue 328's stored file | 0.0319 | 0.080 / 0.050 / 0.028 | 0.367 / 0.298 / 0.228 | 0.054 / 0.261 | 0.018 / 0.570 | 0.0112 / 0.402 | 0.0091 / 0.0548 |
| `kitchen` | `seed1` (step 5) | issue 328's stored file | 0.0319 | 0.080 / 0.050 / 0.028 | 0.370 / 0.300 / 0.229 | 0.054 / 0.263 | 0.017 / 0.568 | 0.0115 / 0.405 | 0.0092 / 0.0534 |
| `kitchen` | `default-326` (step 20) | this ticket | 0.0319 | 0.080 / 0.050 / 0.028 | 0.369 / 0.298 / 0.228 | 0.054 / 0.262 | 0.017 / 0.565 | 0.0114 / 0.407 | 0.0092 / 0.0529 |
| `courtyard` | `default-395` (step 36) | this ticket | 0.1269 | 0.030 / 0.015 / 0.007 | 0.221 / 0.161 / 0.117 | 0.017 / 0.125 | 0.002 / 0.436 | 0.0127 / 0.396 | 0.0103 / 0.0857 |
| `courtyard` | `default` (step 12) | this ticket; equal to issue 328's stored file | 0.1269 | 0.036 / 0.018 / 0.009 | 0.234 / 0.170 / 0.124 | 0.021 / 0.135 | 0.002 / 0.385 | 0.0144 / 0.461 | 0.0112 / 0.0643 |
| `studio` | `default-395` (step 37) | this ticket | 0.0933 | 0.023 / 0.013 / 0.007 | 0.466 / 0.412 / 0.366 | 0.019 / 0.174 | 0.007 / 0.648 | 0.0411 / 6.719 | 0.0052 / 0.5121 |
| `studio` | `default` (step 16) | this ticket; equal to issue 328's stored file | 0.0933 | 0.022 / 0.013 / 0.008 | 0.478 / 0.422 / 0.372 | 0.023 / 0.213 | 0.003 / 0.616 | 0.0469 / 5.810 | 0.0066 / 0.3439 |
| `studio` | `default-325` (step 23) | this ticket | 0.0933 | 0.023 / 0.013 / 0.008 | 0.483 / 0.426 / 0.375 | 0.024 / 0.224 | 0.003 / 0.611 | 0.0496 / 5.922 | 0.0070 / 0.3310 |
| `studio` | `default-341` (step 27) | this ticket | 0.0933 | 0.022 / 0.013 / 0.007 | 0.481 / 0.426 / 0.375 | 0.023 / 0.225 | 0.003 / 0.611 | 0.0483 / 5.965 | 0.0067 / 0.3278 |

The error of expected depth is expected depth minus ground-truth depth, negative in front of the surface, over the surface pixels with alpha ≥ 0.5 (Q-227) away from depth edges (Q-228). Each cell gives the median absolute error, the mean absolute error, the root mean square error and the mean error, in metres.

| Scene | Checkpoint | Test frames | Share of the surface pixels kept | All pixels | Static pixels | Dynamic pixels |
| --- | --- | --- | --- | --- | --- | --- |
| `kitchen` | `default-395` | 300 | 0.9827 | 0.214 / 0.278 / 0.377 / −0.269 | 0.222 / 0.286 / 0.383 / −0.277 | 0.045 / 0.078 / 0.114 / −0.063 |
| `kitchen` | `default` | 300 | 0.9826 | 0.207 / 0.313 / 0.442 / −0.287 | 0.216 / 0.320 / 0.449 / −0.293 | 0.112 / 0.127 / 0.157 / −0.119 |
| `kitchen` | `seed1` | 300 | 0.9821 | 0.219 / 0.308 / 0.425 / −0.268 | 0.227 / 0.314 / 0.431 / −0.272 | 0.154 / 0.158 / 0.184 / −0.153 |
| `kitchen` | `default-326` | 300 | 0.9831 | 0.295 / 0.414 / 0.551 / −0.396 | 0.302 / 0.421 / 0.560 / −0.403 | 0.203 / 0.215 / 0.241 / −0.212 |
| `courtyard` | `default-395` | 300 | 0.9859 | 0.897 / 0.990 / 1.146 / −0.975 | 0.924 / 1.005 / 1.162 / −0.991 | 0.679 / 0.658 / 0.730 / −0.625 |
| `courtyard` | `default` | 300 | 0.9857 | 0.692 / 0.722 / 0.842 / −0.678 | 0.722 / 0.739 / 0.856 / −0.695 | 0.323 / 0.344 / 0.422 / −0.300 |
| `studio` | `default-395` | 60 | 0.9853 | 0.137 / 0.305 / 0.537 / −0.188 | 0.199 / 0.382 / 0.615 / −0.236 | 0.084 / 0.097 / 0.205 / −0.058 |
| `studio` | `default` | 60 | 0.9852 | 0.115 / 0.229 / 0.420 / −0.065 | 0.184 / 0.287 / 0.474 / −0.082 | 0.048 / 0.074 / 0.215 / −0.021 |
| `studio` | `default-325` | 60 | 0.9852 | 0.162 / 0.410 / 0.644 / −0.313 | 0.309 / 0.532 / 0.744 / −0.412 | 0.064 / 0.082 / 0.195 / −0.043 |
| `studio` | `default-341` | 60 | 0.9852 | 0.111 / 0.223 / 0.419 / −0.050 | 0.181 / 0.281 / 0.475 / −0.060 | 0.045 / 0.069 / 0.204 / −0.023 |

- **The residual's shares are steady between trainings, the error of expected depth is not.** Over the three trainings of the default configuration before 549o, the six shares differ by at most 0.003 on kitchen and 0.004 on studio. The median absolute error of expected depth runs from 0.207 to 0.295 m on kitchen and from 0.111 to 0.162 m on studio, up to 47 % above its lowest value, and by more within a region: from 0.112 to 0.203 m on kitchen's dynamic pixels, and from 0.181 to 0.309 m on studio's static pixels.
- **At 549o the shares are lower than before it,** by up to 0.017, which is more than the earlier trainings differ by: in front on kitchen (0.040 against 0.050 at δ) and courtyard (0.015 against 0.018), behind on studio (0.412 against 0.422 to 0.426) and courtyard (0.161 against 0.170). The error of expected depth is inside the earlier runs' range on kitchen and on studio's pixels as a whole, below it on kitchen's dynamic pixels (0.045 m), above it on studio's dynamic pixels (0.084 m), and higher on courtyard (0.897 m against 0.692 m). Each scene has one training at 549o, so the spread there is not known.
- **Most of the weight off the surface is behind it,** not in front. In the 549o runs, at δ, 29 % of the weight is behind on kitchen, 16 % on courtyard and 41 % on studio, against 1 to 4 % in front; on dynamic pixels 44 to 65 % is behind. The residual counts a primitive whether or not something hides it. Expected depth, in which a hidden primitive weighs little, errs toward the camera on every scene (mean error −0.19 to −0.98 m).

**What a later test compares with.**

- **On private scene A:** the row of the first table with its test view, its cap and its init, or a baseline it trains itself. The far-5 cm share compares only against the same init points file, and it says more with its near-side part beside it. A difference under 0.01 in the far-5 cm share, or under 0.004 in the under-two-frames share, is within what a second seed gave. The sheet is rendered from the same request.
- **On openftgs-synth:** the `default-395` rows. On the residual's shares a difference above 0.005 is more than the earlier trainings differed by. On the error of expected depth one run per arm cannot show "no worse": the earlier trainings of one configuration differ by up to 47 %.

**What is open.**

- **The fourth nine-camera model.** The notes' scorer gave it a far-5 cm share of 0.234 and an under-two-frames share of 0.518, in the score file beside the link, against an init points file that file does not name. Measuring it here, and rendering its sheet, needs the owner's word that the link's target may be opened.
- **The sheets** have not been seen by anyone yet.

**The commands.** Each ran in the ticket's worktree with `OMP_NUM_THREADS=4 MKL_NUM_THREADS=4 PYTHONPATH=<worktree>/src:<worktree>/harness/src` and the shared `.venv`'s Python; the CPU runs also had `CUDA_VISIBLE_DEVICES=` empty. The scripts under `<scratch>/ticket-5/` are not committed. `$T` is `<scratch>/ticket-5`, `$O` is `<surface-runs>/ticket-5`, `$G` is `$OPENFTGS_DATASETS/synth/ground_truth`.

| Command | Where | Time | Note |
| --- | --- | --- | --- |
| `python $T/shell_measures.py <notes> <private-scene-A>/scene.json <manifest-centre> <manifest-left> --out $O/shell_measures.json --nine <model>=<init>`, with one `--nine` per nine-camera model | CPU | 2 min 15 s | The three measures on the 45 checkpoints. `<manifest-centre>` and `<manifest-left>` are the notes' two copies of the scene manifest that keep a test view, and `<init>` the init points file a nine-camera model trained from; a run's own init is read from the notes' results. Each checkpoint is assigned to its scene manifest by the SHA-256 its manifest records. 42 equal the notes' score files. Two of the 45 came from a first run of the same command with `--only` |
| `python $T/supplement.py <notes> <private-scene-A>/scene.json $O/shell_measures.json --out $O/shell_supplement.json` | CPU | 41 s | What the measures contain, on seven checkpoints; the share under 0.005 on all 45; the rank correlations with the notes' scores |
| `python $T/windowed.py <notes> <private-scene-A>/scene.json --out $O/shell_windowed.json` | CPU | 59 s | The far-5 cm share against the init points within 2, 6 and 12 frames of each primitive's time centre, on four checkpoints |
| `python $T/make_side_request.py <private-scene-A>/scene.json $O/side_request.json --between <notes-request>` | CPU | under 1 s | The sheet's render request. `<notes-request>` is the notes' render request for its poses between cameras. The rule gives the cameras beside the centre one at yaw ∓15° to 10⁻⁷ |
| `python $T/framing_check.py <checkpoint> $O/side_request.json <private-scene-A>/scene.json --out $O/side_request_framing.json` | CPU | 4 s | Where the near-side primitives of the nine-camera model `stride4`, whose checkpoint `<checkpoint>` is, land in each of the 39 cameras |
| `python scripts/gt_depth.py <m17-runs>/kitchen/<run>/checkpoint <m15-runs>/kitchen/scene.json $G/kitchen --out $O/gt_depth/kitchen-<run>.json`, for `default-395`, `default` and `default-326` | GPU | 64 s, 61 s, 71 s | 300 test frames each; `default` equals issue 328's stored file |
| `python scripts/gt_depth.py <m17-runs>/courtyard/<run>/checkpoint <m17-runs>/courtyard/scene/scene.json $G/courtyard --out $O/gt_depth/courtyard-<run>.json`, for `default-395` and `default` | GPU | 83 s, 83 s | 300 test frames each; `default` equals issue 328's stored file |
| `python scripts/gt_depth.py <m17-runs>/studio/<run>/checkpoint <m17-runs>/studio/scene/scene.json $G/studio --out $O/gt_depth/studio-<run>.json`, for `default-395`, `default`, `default-325` and `default-341` | GPU | 25 s, 31 s, 23 s, 24 s | 60 test frames each; `default` equals issue 328's stored file |
| `ftgs render <checkpoint> --request $O/side_request.json --out $O/renders/<name>.safetensors`, for the seven checkpoints of the sheet | GPU | 5.8 to 7.0 s each | 39 renders per checkpoint; the renders file is deleted once its sheet is made |
| `python $T/side_sheet.py $O/renders/<name>.safetensors $O/side_request.json $O/sheets/side_<name>.jpg --label <name> --stats $O/sheets/side_<name>.coverage.json`, after each render | CPU | about 1 s each | The sheet and its coverage numbers |
| `python $T/tables.py …` | CPU | under 1 s | The rows of this section's tables, from the files above |

The GPU commands ran one at a time, from `$T/run_gpu.sh`, between 01:44:29 and 01:53:05; the card's memory peaked at 5,422 MiB during `gt_depth.py` and 7,737 MiB during `ftgs render`, about 1,400 MiB of it in use before they started. The private scene's folders were read and never written.

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
