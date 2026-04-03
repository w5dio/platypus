# CLAUDE.md — Platform Constitution

## About This Repository

This repository contains the governing principles for the w5d.io platform — a shared platform that provides common services and resources for multiple first-class projects and products. The principles defined here apply to all repositories in the [w5d.io](https://github.com/w5dio) GitHub organisation.

This is a meta-level repository. It does not implement any platform system itself — it defines the framework that governs how all platform systems are built and maintained.

## Working on the Principles

### Keep principles broad

Principles must be broad enough to govern implementations across the full spectrum — from fully automatic (e.g. Terraform) to fully manual (e.g. a YAML file describing desired state, with a human performing the required actions). Do not narrow a principle to a specific tool, technology, or workflow.

### Keep principles actionable

Each principle should be concrete enough that a coding agent implementing a platform system can make decisions based on it. Avoid purely philosophical statements that do not translate into observable behaviour or constraints.

### Prefer fewer, more fundamental principles

Resist adding principles for every concern. Prefer a small set of principles that are genuinely governing over a longer list that includes implementation guidance or edge-case rules.

### Do not over-specify

The principles intentionally leave room for a variety of implementations. Do not add detail that constrains implementation choices that should remain open.

## Key Design Decisions

**Cross-repo structure is out of scope:** The principles govern what happens within each repository, not the relationship between repositories. Platform systems may live in the same or separate repositories. Do not add principles about how repositories relate to or interact with each other.

**The primary reader of these principles is a coding agent:** Principles should be written with this in mind — precise, unambiguous, and actionable. This also has implications for the document itself: it must be fully self-contained (readable without any external context) and concise (it will be injected into the context window of Claude Code sessions in platform system repositories via a session-start hook, so unnecessary length has a real cost).
