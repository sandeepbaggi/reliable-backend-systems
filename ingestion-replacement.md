# Replacing a Production Bulk Ingestion System Without Breaking Customers

## Context
This work involved replacing a production bulk ingestion system that had become operationally fragile as features accumulated.

The system processed real customer data, and downtime or corruption was unacceptable.

Incremental refactoring increased risk instead of reducing it, so a **parallel shadow replacement strategy** was chosen.

## What This Demonstrates
- Recognizing when a core abstraction has become the primary source of risk
- Safely replacing production systems using evidence, not debate
- Designing migration paths with atomic cutover and rollback

---

## Executive Summary
This artifact describes how a production bulk ingestion system was replaced safely when its core abstraction—a long-running, stateful job—became the dominant source of complexity, recovery risk, and scaling friction. Rather than incremental patching, a parallel, shadow implementation was used to deliver a clean, event-driven architecture with an atomic cutover.

## What Actually Broke Before the Replacement
- Recovery paths multiplied with each new feature, making failures hard to reason about
- Scaling validation, persistence, and cleanup independently was impossible under a single job abstraction
- Debugging required reconstructing complex job state histories
- Team velocity slowed due to fear of regressions in orchestration logic

These were structural signals, not performance bugs.

## First-Principles Reframing
A bulk ingestion “job” was decomposed into:

1. **Fact:** A user submitted a file (immutable)
2. **Process:** Records are validated and persisted (idempotent)
3. **View:** User-visible progress and reports (derived)

Conflating these concerns created most operational pain.

## Options Considered (and Rejected)

### Harden the Existing Job Model
Preserved the root cause of complexity.

### Hybrid Job + Event Model
Created a distributed monolith with worse failure modes.

Both were rejected for compounding complexity.

## Chosen Strategy — Parallel Shadow Replacement
- Build a new ingestion system in complete isolation
- No shared databases or dual writes
- Route a small percentage of real traffic to validate equivalence
- Measure discrepancies instead of debating correctness
- Cut over traffic atomically; rollback is a switch

## Outcomes
- Reduced recovery logic complexity significantly
- Enabled independent scaling of ingestion stages
- Increased team confidence to evolve the system

## Transferable Principle
When the core abstraction becomes the bottleneck, parallel replacement with evidence is safer than incremental evolution.
