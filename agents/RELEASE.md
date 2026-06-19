# Release Phase Guide
v.0.0.01

## Table of Contents
- [1. Introduction](#1-introduction)
- [2. Goal](#2-goal)
- [3. Session Prompt](#3-session-prompt)
- [4. The Release Lifecycle](#4-the-release-lifecycle)
  - [4.1. Stage 1: Release Planning](#41-stage-1-release-planning)
  - [4.2. Stage 2: Deployment](#42-stage-2-deployment)
  - [4.3. Stage 3: Post-Release](#43-stage-3-post-release)
- [Appendix R - Revision History](#appendix-r---revision-history)

---

## 1. Introduction
This guide outlines the Release Phase, which bridges a verified product and its public launch. All session-level rules are defined in `AGENTS.md` and all script and command rules are in `agents/SCRIPT_RULES.md`. Both MUST be adhered to at all times.

## 2. Goal
The goal of this phase is to execute a smooth, controlled, and successful product launch. This includes publishing all documentation and effectively communicating new features.

## 3. Session Prompt
**MANDATE:** At the beginning of this phase, a re-invocable prompt file (e.g., `p_release.md`) must be created. This prompt will guide the agent through the release plan and checklist.

## 4. The Release Lifecycle

### 4.1. Stage 1: Release Planning
A formal plan is created, defining the release scope, target date, and a SemVer-compliant version number (e.g., `v0.1.0`).

### 4.2. Stage 2: Deployment
Once a release candidate is UAT-approved, this stage manages the final packaging and rollout.
-   **Crate Preparation:** The agent MUST ensure all crates intended for publishing have correct `Cargo.toml` metadata (`license`, `description`, `version`) and a `README.md`.
-   **Finalize Documentation:** All user-facing documentation, developer docs, and release notes are finalized and packaged.
-   **Marketing & Communications:** The agent creates and disseminates launch announcements, technical articles, and blog posts.
-   **Production Rollout:** The final package is deployed to the production environment.

### 4.3. Stage 3: Post-Release
After deployment, the team actively monitors system health, performance metrics, and user feedback.

## 5. Phase Completion Criteria
**MANDATE: Phase-End Quality Assurance.** Before finalizing, the agent MUST perform a final audit to ensure all release artifacts (docs, metadata, binaries) are complete and free of placeholder content.

This phase is complete when the final release package has been successfully deployed to production and all marketing and communication activities have been executed.

---

## Appendix R - Revision History
| Version | Date       | Author      | Changes                               |
|---------|------------|-------------|---------------------------------------|
| 0.0.01  | 2025-10-10 | Jules       | Initial creation and refactoring.     |