# CLAUDE.md

This file defines how Claude Code should work in this repository. Treat it as the *operating manual* for implementing this project.

---

## Authority & Precedence (Read First)

**`SPEC.md` and `PLAN.md` always take priority over this file.**

- `SPEC.md` defines **what** must be built
- `PLAN.md` defines **how and when** it should be built
- `CLAUDE.md` defines **how Claude should behave while building it**

If anything in `CLAUDE.md` conflicts with `SPEC.md` or `PLAN.md`, Claude must:
1. Update `CLAUDE.md` to be consistent with them
2. Proceed only after alignment is restored

Claude **must read and internalize both `SPEC.md` and `PLAN.md` before writing or modifying any code**.

If either file is ambiguous, incomplete, or contradictory, **pause and ask the user before proceeding**.

---

## Project Overview

This repository contains a **full-stack web application for preparing and choosing a wedding guest list**.

All product behavior, constraints, and workflows must come directly from `SPEC.md`.  
No assumptions should be made beyond what is explicitly stated.

---

## Core Principles (Follow These Strictly)

### 1. Ask Before Guessing

If you encounter:
- Edge cases not clearly specified
- UX or product decisions that materially affect behavior
- Data modeling decisions with long-term impact
- Multiple plausible interpretations of the spec

**Stop and ask the user before implementing.**  
Do not silently choose an approach.

---

### 2. Documentation Is Required and Ongoing

Claude must **write documentation continuously as code is written**, not at the end.

Documentation rules:
- Each major directory gets a short `README.md`
- Each non-obvious module should be documented via comments *and* referenced in docs
- High-level workflows and system behavior should be explained in `/docs`

Global documentation (setup, commands, deployment, testing, environment variables) belongs at the **repository root**.

---

### 3. Keep the Structure Simple and Intentional

Avoid both:
- Large, catch-all files
- Over-engineered directory trees

Favor:
- Small, purpose-driven files
- Clear ownership by domain
- Structures that make the codebase easy to navigate and reason about

When introducing new structure, prefer clarity over abstraction.

---

## Suggested Directory Structure

Claude may evolve this structure as needed, but should **start here** and keep it coherent.