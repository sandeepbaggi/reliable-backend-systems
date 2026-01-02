# Designing Crash-Safe Bulk Import Pipelines

## Context
This system was designed and owned in a multi-tenant B2B SaaS platform where enterprise customers upload large configuration datasets (XML, CSV, ZIP).

Failures in this pipeline had direct customer impact: corrupted configuration state, broken integrations, and manual recovery effort.

The primary constraint was **correctness and recoverability under partial failure**, not raw throughput.

## What This Demonstrates
- Designing long-running backend workflows that survive crashes without manual intervention
- Making explicit tradeoffs between correctness, performance, and operability
- Treating bulk ingestion as a **recovery problem**, not a parsing problem

---

## Executive Summary
Bulk import and export pipelines are among the highest-risk operations in enterprise B2B systems. They combine long-running execution, untrusted input, large data volumes, and user-initiated retries under uncertainty. This artifact documents a production-tested approach focused on correctness, recoverability, and operational stability, shaped by real incidents rather than theoretical best practices.

## What Actually Broke Before This Design
- Long-running imports failed mid-way due to unpredictable stream closures, forcing full restarts
- Single invalid records caused entire batches to roll back, wasting hours of work
- Partial failures left orphaned resources, leading to name collisions and manual cleanup
- Lack of reliable progress visibility caused users to retry blindly, creating duplicate state

## Design Goals
- Never load entire files into memory
- Ensure every record is independently recoverable
- Survive crashes without manual intervention
- Provide clear, user-visible progress
- Prevent orphaned or inconsistent state

Correctness and predictability were explicitly prioritized over architectural elegance.

## Key Design Decisions & Tradeoffs

### Decoupling Network Streams from Processing
Incoming streams are copied to durable temporary storage immediately and closed early. All downstream processing operates on local files.

**Why:** Network streams are unreliable for long-running work  
**Tradeoff:** Requires explicit disk cleanup logic

### Two-Pass Processing for Progress Accuracy
A first pass determines scope and progress boundaries; a second pass performs actual work.

**Why:** Reliable progress prevents unsafe retries  
**Tradeoff:** Additional I/O overhead

### Record-Level Isolation
Each record is processed and persisted independently.

**Why:** Real datasets contain partial errors  
**Tradeoff:** More granular state tracking

### Cleanup-on-Failure as a Rule
Any partially created resource is explicitly cleaned up on failure.

**Why:** Orphaned state accumulates silently and blocks future operations

## Outcomes
- Eliminated an entire class of non-deterministic import failures
- Reduced manual cleanup and support escalations
- Increased user trust in long-running operations

## What I Would Improve Next Time
- Stronger abstraction around progress derivation
- More granular retry strategies per failure class
- Clearer separation between validation and execution

## Transferable Principle
Bulk import is not a parsing problem; it is a recovery and correctness problem. Systems that survive crashes without manual recovery earn user trust and scale operationally.
