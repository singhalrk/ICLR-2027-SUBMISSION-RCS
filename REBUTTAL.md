# ICLR 2027 Rebuttal — Risk-Controlled Safe Stopping

> **Working draft for author review.** Based on the revised manuscript, including additional baseline comparisons (Appendix A.3, Figures 6–8) and calibration sensitivity analyses (Appendix A.4, Figures 9–11). This file is intended for iterative editing before submission.

## Response to Reviewer R1

We thank the reviewer for recognizing the importance of conditional risk control and the applicability of RCS to existing acquisition pipelines.

**W1 — Limited novelty.** We agree that the finite-sample guarantee follows from Learn-Then-Test (LTT), and we do not claim a new concentration bound. Our contribution is casting sequential acquisition stopping as trajectory-level selective prediction with **absolute, ground-truth risk conditional on stopping**. Unlike agreement with full-acquisition predictions, absolute risk has no automatically safe configuration and need not be monotone across candidate policies. RCS therefore certifies candidates without assuming an ordering of their risks.

**W2 — “Certified infeasibility.”** We agree. An empty certified set means that no candidate could be certified with the available calibration data at the specified confidence level; it does not establish that no safe policy exists. We corrected this interpretation in the abstract, introduction, and methods.

**W3/Q5 — Additional baselines and acquisition efficiency.** We added Operating Point and Selective Inference comparisons across all three datasets (Appendix A.3, Figures 6–8). At each acquisition checkpoint, both methods choose a threshold on calibration data to target the same nominal task-specific risk: Operating Point uses all samples, whereas Selective Inference computes risk only among selected stopped samples. These empirical operating-point procedures do not furnish RCS's finite-sample certificate.

On **FastMRI Knee**, RCS stops 13% of slices at 5% acquisition with zero observed ACL FNR and meniscus FNR at most 0.012; Selective Inference also meets the empirical risk target, but stops no slices at 5% or 8% and stops more at later stages. On **prostate MRI**, RCS stops 36–53% of examinations with observed FNR 0.026–0.064, while Operating Point (0.14–0.17) and Selective Inference (0.105–0.14) exceed the nominal FNR target of 0.1. We view these as empirical risk–efficiency comparisons under a common calibration target, not proof of universal efficiency superiority.

**Q1 — Time-specific guarantees.** Corollary 1 controls conditional risk on the full population of eventually stopped cases, not simultaneously on cumulative stopped subsets at each checkpoint. We corrected the main-text and figure-caption claims accordingly. The early CIFAR-10 checkpoints with observed stopped-sample accuracies of approximately 0.83 and 0.88 are empirical diagnostics, not violations of a separately asserted time-specific certificate.

**Q2 — Calibration setup.** Appendix A.4 (Figures 9–11) now reports sensitivity to the significance level δ, grid size |Λ|, and calibration size n_cal. Default settings (δ, |Λ|, n_cal) are (0.05, 101, 4800) for CIFAR-10, (0.1, 15, 6180) for knee, and (0.1, 51, 745) for prostate. We report effective class-positive counts and regimes in which certification returns an empty set.

**Q3 — Patient/slice independence.** FastMRI Knee was partitioned at the **patient level**, with no patient overlap across training, calibration, and testing. Thus, held-out performance is measured on unseen patients. However, knee classification and risk calibration are **slice-level**, and slices from the same patient can be correlated. Patient-disjoint splitting therefore does not itself establish the i.i.d. slice assumption needed for the exact finite-sample slice-level certificate. The available independent knee volumes, especially positive volumes for rare ACL pathology, were insufficient for reliable volume-level classification and risk certification at the target FNR. Accordingly, we interpret the knee results as empirical risk and efficiency measurements on patient-disjoint test data and identify volume-level prediction/certification with larger cohorts as future work. **In contrast, prostate MRI uses patient-disjoint examination/volume-level prediction and calibration, with sufficient independent examinations.**

**Q4 — Full-acquisition fallback.** If no early-stopping policy is certified, acquisition proceeds through the existing full protocol. This is an operational fallback, not a statistical certificate for full-acquisition predictions or a proof of end-to-end clinical noninferiority. We revised the manuscript to distinguish these statements.

## Response to Reviewer R2

We thank the reviewer for recognizing the clinical motivation and the model-agnostic design.

**Additional risk-aware comparisons.** Appendix A.3 adds the two empirical risk-targeting baselines noted above. On **CIFAR-10**, Operating Point stops approximately 90% of samples but achieves 0.68–0.74 stopped-sample accuracy; Selective Inference ends near 0.84, whereas RCS reaches approximately 0.905 while stopping 52%. On **prostate MRI**, both additional baselines exceed the observed FNR target, while RCS remains below 0.1 at evaluated checkpoints. On **knee MRI**, Selective Inference meets the empirical target but does not stop at the earliest acquisition rates, unlike RCS. The comparisons emphasize the difference between calibrating to an empirical operating point and obtaining a formal conditional-risk certificate under the stated assumptions.

**Distribution shift.** RCS inherits LTT's requirement that calibration and deployed acquisition trajectories follow the same induced distribution. Changes in populations, scanners, or adaptive acquisition policies can invalidate the certificate; recalibration with representative data or a suitable shift-aware procedure may be necessary. We explicitly acknowledge this scope limitation.

**JAFA+RCS stopping efficiency.** JAFA optimizes expected reward trading prediction utility against measurement cost, rather than a hard conditional-risk constraint. RCS thresholds the STOP-versus-acquire Q-value margin and certifies only policies satisfying its risk test. Consequently, it defers some decisions JAFA would otherwise make early. This conservativeness follows from the statistical constraint and does not require nonsmooth Q-value estimates.

## Response to Reviewer R3

We thank the reviewer for the suggestions on prior work, statistical assumptions, and empirical analysis.

**Positioning.** Compared with Ringel et al., which control the discrepancy between early and full-data predictions, RCS controls **absolute ground-truth error conditional on stopping**. Compared with computational early-exit approaches such as Jazbec et al., RCS targets external data acquisition. We acknowledge that our guarantee is inherited from LTT; the contributions are the acquisition-specific risk formulation, policy-level certification, and medical-imaging evaluation. The paper also discusses Gauthier et al. in the broader sequential/conformal context.

**Scope and fallback.** The candidate family must be fixed independently of risk calibration, and the independent calibration trajectories must match the deployment-induced distribution. An empty certified set is a failure to establish safety from available data, not a proof of population infeasibility. Full acquisition is a fallback, not an automatically certified alternative. These distinctions have been corrected in the main text.

**Calibration sensitivity and power.** Appendix A.4 systematically varies δ, candidate-grid density, and calibration size. **CIFAR-10:** 21–501 grid values give similar behavior; an 11-point grid certifies no policy, and calibration samples of 250–1000 give no certificate while 2000 suffice for a conservative policy. **FastMRI Knee:** slice-level calibration subsets containing up to 93 ACL-positive slices yield no certificate, while the full set with 155 ACL-positive slices does under the reported procedure, subject to the within-patient dependence limitation described to R1. **Prostate MRI:** 41 or 84 PI-RADS-positive calibration examinations are insufficient, while certification is achieved from 168 positives onward. These experiments describe certification power and the empty-set regime, but do **not** measure the repeated-calibration probability of incorrectly certifying an unsafe policy.

**Design choices and assumptions.** Holm's step-down controls family-wise error without requiring a monotone ordering of candidate risks; exact binomial testing is appropriate for binary class-conditional FNR/FPR losses. The new analyses vary calibration parameters rather than directly ablating Holm against alternative corrections or p-value constructions. Policy-family design continues to influence achievable acquisition efficiency. We also clarify that distribution shift and adaptive-policy trajectory mismatch can invalidate the certificate.

## Closing

We thank the reviewers for the constructive feedback. The expanded evaluation introduces **two additional calibration-based baselines (Figures 6–8)** and **calibration sensitivity experiments (Figures 9–11)**, clarifying empirical risk–efficiency trade-offs, class-conditional statistical power, and empty certified sets. We also corrected the interpretation of failure to certify, the eventual-cohort scope of the guarantee, and the operational status of full acquisition.

The medical-imaging evaluation uses **patient-disjoint partitions** in both cohorts; knee classification remains slice-level with a corresponding within-patient dependence limitation, whereas prostate prediction and calibration operate at the examination/volume level. These distinctions more precisely delimit the claims supported by the experiments.

RCS remains a post-hoc, model-agnostic approach for certifying conditional prediction risk in sequential acquisition **under its stated statistical assumptions**, without retraining the underlying predictor or acquisition policy.

---

### Author notes (not part of submitted rebuttal)

- The independence limitation for knee MRI is explicitly acknowledged; do not characterize the existing slice-level knee results as formally patient-cluster-certified.
- The repeated-calibration violation probability requested by R3 was not measured.
- The knee significance-level sensitivity plot is retained unchanged. No claim of monotone threshold selection is made from that sweep.
- The prostate cohort has 945 examinations in the designated calibration partition; 745 were used in risk calibration and 200 were unused.
- This draft has no new experimental results beyond the current paper.
