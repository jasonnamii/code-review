# code-review

> 🇰🇷 [한국어 README](./README.ko.md)

**One precise review engine for every part — backend, frontend, Android, iOS, AI/ML. Turn a code change into a report a developer can fix from, with false positives filtered out.**

## Problem

Authors write the intended happy path well. Bugs live not in the changed lines but in the **surrounding assumptions the change breaks**. And generic linters / LLM reviews drown signal with false positives. Different parts use different languages and roles — yet most review tooling is part-specific.

## Solution — universal 8-axis spine + per-part lens

- **Reverse TDD**: like writing tests first, **generate the acceptance criteria first** (what must hold for the intent to be correct), then reverse-validate the code against them. The diff isn't under review — it's the raw material.
- **8 universal axes** (correctness · state/concurrency · error/failure · contract · security · performance · maintainability · test) are the same for every part. Each **part lens** (backend / frontend / Android / iOS / AI-ML) only changes *which axis is critical and how it manifests here*.
- **Adversarial verification**: every finding must survive a refutation attempt before it's reported. False positives kill a review's credibility.
- **Knowledge-based, not pattern matching**: model the code's meaning first; catalogs are priors, not triggers.

## Flow

```
Input: a diff/PR or target files
   ↓ Phase 0    normalize + identify part(s) + load lens + red-flag pre-scan
   ↓ Phase 0.5  code semantic model (intent · contract · resources · state · invariants)
   ↓ Phase 1    reverse-generate acceptance criteria (8 axes + part lens + detectors)
   ↓ Phase 1.5  system ripple (callers · callees · migration · cross-cutting)
   ↓ Phase 2    reverse-validate → findings + adversarial verify (drop false positives) + confidence
   ↓ Phase 3    prescribe → patch + rationale (security/license → route, don't generate)
   ↓ Phase 3.5  elevate (aesthetics mode) → higher-level structure suggestions
Output: scannable review report (.md) + optional inline PR comments
```

## Parts (one engine, many lenses)

| Part | Lens emphasis |
|---|---|
| Backend | concurrency · transactions · API contract · N+1 · authz · idempotency |
| Frontend | render/state · effect cleanup leaks · accessibility · bundle · XSS |
| Android | lifecycle · Context leaks · main-thread (ANR) · coroutine scope |
| iOS | retain cycles · main-thread UI · force unwrap · async isolation |
| AI/ML | data leakage · reproducibility · metric misuse · prompt injection · cost |

Multi-part PRs apply all lenses and add **part-boundary contract** checks (e.g. backend API shape change breaking the app).

## Modes × Targets × Weights

- **Action**: diagnose / prescribe (default) / elevate (aesthetics).
- **Target**: diff review (default) / full audit.
- **Weight**: Light (critical only) / Standard (full 8-axis, default) / Heavy (security·concurrency·perf·tests·elevate).

## Prerequisites

- Claude Code / Codex app environment, a git repo or target files.
