# Pre-release checklist

Reusable release checklist, prepared for the v2.17.0 July 30, 2026 plan.
This document does not certify that the release shipped or that these checks passed.
The three-runner acceptance split is a proposed configuration to validate before use.

Release: __________  
Candidate commit: __________  
Evidence location (access appropriate to its contents): __________  
Release owner: Bram; CI and security readiness: Yusuke

## 1. Confirm scope

- [ ] Include only merged, reviewed changes. For the July plan, confirm the
  gradient-hook refactor met the July 24 cutoff; otherwise defer it.
- [ ] Identify changed adapters and representative families affected by shared
  conversion, norm, or RoPE helpers. Obtain the required peer review.
- [ ] Record the candidate commit before collecting release evidence.

## 2. Conversion and acceptance

### CSV baseline: mandatory conversion checks

The original `weight_conversion_checklist.csv` lists these eight checks for each
of LLaMA-4, Qwen-3, Mistral-Small-3, Gemma-3, and Qwen-2.5-VL. Record results
separately for each family included in the release verification scope from
section 1. Selecting affected families is a release-scoping decision, not a
condition specified by the CSV.

- [ ] Read `head_dim` from the source model config; preserve its explicit value.
- [ ] Verify GQA KV-head repetition or equivalent broadcasting, including correct
  head ordering; matching tensor shapes alone is insufficient.
- [ ] Verify the RoPE variant and theta.
- [ ] Verify norm epsilon is inside `rsqrt`: equivalently, inside the
  normalization denominator's square root.
- [ ] Map QK-norm tensors **if present**.
- [ ] Apply embedding scaling **if present**.
- [ ] Verify tied/untied embeddings match the source model.
- [ ] Pass the `1e-4` acceptance threshold.

Only QK-norm mapping and embedding scaling are conditional in the CSV. Record
why either is absent when marking it not applicable; do not use that option to
waive the other six checks.

### Supplementary conversion and acceptance gates

These checks strengthen the CSV baseline with the agreed release-verification
requirements. They are additions, not requirements stated in the CSV. The scope,
CI, notebook, security, and sign-off steps elsewhere in this checklist are also
supplementary release-process gates.

- [ ] Verify epsilon values reach all relevant norms, including QK, extra, and
  final norms. Confirm the regression test fails with incorrect epsilon placement.
- [ ] Verify intentional dtype behavior for converted and loaded weights, synthetic
  biases, and outputs. Document intentional FP32 norm calculations; do not require
  all tensors to have identical dtypes.
- [ ] Test a configuration where `head_dim` differs from
  `hidden_size / num_attention_heads`. Check Q/O and GQA K/V dimensions.
- [ ] Verify local/global attention and any additional required norm tensors.
- [ ] Compare HF and the affected TransformerLens loading path on the smallest
  appropriate public checkpoint with matching FP32/eager settings, token IDs,
  evaluation mode, and pinned checkpoint revision.
- [ ] Test short and longer prompts, including the Gemma-3 regression
  (release-triage issue `#1121`). Apply the baseline threshold as maximum absolute
  raw-logit error <= `1e-4` on the unprocessed path, without a relative-tolerance
  allowance. Check selected intermediate activations against explicit tolerances.
- [ ] Separately test weight processing. When processing intentionally shifts raw
  logits, compare log-probabilities with an explicitly justified tolerance.
- [ ] If low-precision behavior changed, verify supported FP16/BF16 paths with
  documented tolerances and finite outputs.
- [ ] Save the commit, checkpoint revision, dependency versions, dtype/device,
  inputs, maximum errors, and executed/skipped status. Attach any required local
  or external-runner results for the same candidate.

References: [conversion tests](../tests/unit/pretrained_weight_conversions/),
[Gemma config tests](../tests/unit/model_bridge/supported_architectures/test_gemma3_config.py),
[numerical debugging](source/content/debugging_numerical_divergence.md), and
[compatibility mode](source/content/compatibility_mode.md).

## 3. Three-runner acceptance plan

- [ ] Validate a split into three runtime-balanced acceptance shards on separate
  runners. Preserve required Python 3.10, 3.11, and 3.12 coverage; this can require
  nine jobs if each version runs all three shards.
- [ ] Compare the sharded and unsharded test inventories on the same commit.
  Confirm no required cases are omitted and record skips separately.
- [ ] Keep tests sharing a checkpoint together where practical. Run model
  verification sequentially within each runner; do not share one accelerator
  across concurrent verification jobs.
- [ ] Require every expected shard to pass on the candidate commit. Missing,
  cancelled, skipped, or failed required jobs leave the release gate incomplete.
- [ ] Attach passing results for required slow, hardware-specific, or quarantined
  checks that standard CI does not execute. Fix or isolate quarantined tests;
  do not count their presence in the repository as coverage.
- [ ] Record total elapsed time, runner usage, and the slowest shard. Rebalance
  using measurements without removing checks or loosening tolerances.
- [ ] Confirm the publishing dependency waits for all required acceptance jobs.

References: [test commands](../makefile),
[CI workflow](../.github/workflows/checks.yml),
[publishing workflow](../.github/workflows/release.yml), and
[test quarantines](../tests/QUARANTINES.md).

## 4. Demo notebooks: Wednesday sweep

- [ ] Run the release demo inventory in clean kernels against both the locked
  release environment and the latest stable Transformers version available at
  the sweep. Record exact versions and keep the two environments separate.
- [ ] Explicitly reproduce and verify the previously reported
  Transformers-upgrade notebook failure; do not rely only on the CI subset.
- [ ] Arrange authenticated and hardware-appropriate runs for required notebooks
  that CI skips. Keep tokens out of logs and committed notebook outputs.
- [ ] Any failure blocks the cut until fixed and rerun, or the supported-version
  scope is explicitly revised and documented. A dependency pin alone does not
  demonstrate compatibility with the latest version.
- [ ] Review any dependency change as a separate compatibility change; do not
  introduce an unreviewed dependency bump into a patch release.

References: [demo notebooks](../demos/),
[notebook sanitization](../demos/doc_sanitize.cfg), and
[dependency configuration](../pyproject.toml).

## 5. Security readiness: restricted review

- [ ] The security owner checks the private advisory and patch queue and confirms
  readiness through the restricted maintainer channel.
- [ ] If a patch is included, verify review and regression results on the final
  candidate in an appropriately private environment.
- [ ] Keep vulnerability details, reproductions, private links, credentials, and
  sensitive logs out of public drafts, notebook outputs, and CI artifacts.
- [ ] Publish only approved security wording and advisory links at the agreed
  disclosure time. A late patch requires rerunning affected gates.

## 6. Final sign-off and follow-up

- [ ] All required CI, acceptance shards, external runs, and demo sweeps pass on
  the final candidate. Required checks have no unexplained omissions.
- [ ] Release notes reflect merged changes and tested dependency versions.
  Credit contributors without confidential attribution.
- [ ] Bram and Yusuke approve the exact commit to tag. Reassess and rerun affected
  checks after any subsequent code change.
- [ ] Confirm package publication succeeds and assign 48-hour monitoring and
  hotfix ownership. Record any regression accurately.

Decision: GO / HOLD  
Remaining blockers: __________  
Bram: __________; Yusuke: __________

See [contributor guidance](../AGENTS.md) and
[documentation build instructions](README.md).
