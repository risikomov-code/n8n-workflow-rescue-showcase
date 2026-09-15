# Workflow Rescue Architecture

This document describes a production-safe troubleshooting approach for n8n and AI automation systems without exposing any proprietary workflow.

## 1. Reproduce the failure

Before changing nodes, establish:

- which execution path failed;
- whether the failure is deterministic;
- the last valid state;
- the first invalid state;
- whether the external provider was actually called;
- whether paid credits or irreversible actions were triggered.

This prevents debugging by assumption.

## 2. Classify the failure

A useful rescue workflow distinguishes several failure classes.

### Contract failure

The workflow sends or expects data that no longer matches an external API or an upstream/downstream node.

Examples include:

- renamed fields;
- unsupported parameters;
- changed authentication;
- invalid model or action identifiers;
- unexpected response structure.

### State failure

The workflow technically runs but receives incomplete, stale or inconsistent data.

The correct fix may be a validation gate rather than a provider change.

### Routing failure

The correct data exists, but a conditional branch, mode selector or fallback routes it incorrectly.

### Asset failure

A required file, media segment or reference is missing, invalid or points to the wrong source.

### Recovery failure

The main action succeeds, but retry or fallback logic causes duplicate calls, duplicate records or unnecessary regeneration.

## 3. Establish the delivery contract

Each important workflow boundary should define what must exist before the next stage can run.

A generic contract can include:

```text
required input present
+ expected type / structure
+ valid temporal or logical range
+ source identity verified
+ no unsafe fallback
= branch allowed to continue
```

This turns implicit assumptions into explicit validation.

## 4. Apply a surgical patch

The preferred rescue approach is not to rewrite an entire automation.

A minimal patch should:

- change only the failing contract or route;
- preserve working branches;
- introduce the smallest new state possible;
- avoid provider calls during dry-run where practical;
- keep rollback straightforward.

## 5. Protect cost and side effects

For AI and external-service workflows, troubleshooting must account for paid or irreversible operations.

Useful controls include:

- pre-flight validation;
- dry-run mode;
- approval before generation;
- deduplication / idempotency markers;
- explicit regeneration rules;
- stopping instead of silently substituting an invalid asset.

## 6. Regression testing

A fix should be tested against adjacent behaviour.

Typical checks include:

- original failing mode;
- previously working mode;
- missing-input path;
- invalid-input path;
- rejected / retry path;
- cost-triggering branch;
- final persisted configuration.

The goal is not only “the error disappeared” but “the automation still behaves correctly as a system.”

## 7. Technical Delivery Gate

Before certifying a repair:

1. the workflow change must be persisted;
2. the repaired path must be reproducible;
3. required stop conditions must fire correctly;
4. no forbidden fallback may remain;
5. critical neighbouring modes must pass regression checks;
6. cost-triggering actions must use validated input;
7. remaining limitations must be documented.

## What is omitted

The public version intentionally excludes:

- real workflow names and IDs;
- production node graphs;
- credentials;
- provider payloads;
- exact validation expressions;
- internal schemas;
- server topology;
- production execution IDs;
- proprietary regression fixtures.
